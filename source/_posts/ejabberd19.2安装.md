---
title: ejabberd源码分析
date: 2019-03-07 12:11:20
updated: 2019-03-07 12:11:20
categories: 开源框架
tags: ejabberd

---

### ejabberd19.2源码分析

#### 部署

- ./autogen.sh

  ./autogen.sh: line 2: aclocal: command not found

  ./autogen.sh: line 3: autoconf: command not found

  解决：yum -y install automake

- ./configure --enable-user=ejabberd --enable-lager

- error: libexpat header file expat.h was not found

  解决：yum install expat-devel

- configure: error: libyaml header file yaml.h was not found

  解决：yum install libyaml-devel

#### ejabberd认证

<<"DIGEST-MD5">>,<<"PLAIN">>,<<"SCRAM-SHA-1">>,<<"X-OAUTH2">>

xml流处理：xmpp_stream_in:process_element:589

```erlang
-spec process_element(xmpp_element(), state()) -> state().
process_element(Pkt, #{stream_state := StateName, lang := Lang} = State) ->
    case Pkt of
    #starttls{} when StateName == wait_for_starttls;
             StateName == wait_for_sasl_request ->
        process_starttls(State);
    #starttls{} ->
        process_starttls_failure(unexpected_starttls_request, State);
    #sasl_auth{} when StateName == wait_for_starttls ->
        send_pkt(State, #sasl_failure{reason = 'encryption-required'});
    #sasl_auth{} when StateName == wait_for_sasl_request ->
        process_sasl_request(Pkt, State);
    #sasl_auth{} when StateName == wait_for_sasl_response ->
        process_sasl_request(Pkt, maps:remove(sasl_state, State));
    #sasl_auth{} ->
        Txt = <<"SASL negotiation is not allowed in this state">>,
        send_pkt(State, #sasl_failure{reason = 'not-authorized',
                          text = xmpp:mk_text(Txt, Lang)});
    #sasl_response{} when StateName == wait_for_starttls ->
        send_pkt(State, #sasl_failure{reason = 'encryption-required'});
    #sasl_response{} when StateName == wait_for_sasl_response ->
        process_sasl_response(Pkt, State);
    #sasl_response{} ->
        Txt = <<"SASL negotiation is not allowed in this state">>,
        send_pkt(State, #sasl_failure{reason = 'not-authorized',
                          text = xmpp:mk_text(Txt, Lang)});
    #sasl_abort{} when StateName == wait_for_sasl_response ->
        process_sasl_abort(State);
    #sasl_abort{} ->
        send_pkt(State, #sasl_failure{reason = 'aborted'});
    #sasl_success{} ->
        State;
    #compress{} ->
        process_compress(Pkt, State);
    #handshake{} when StateName == wait_for_handshake ->
        process_handshake(Pkt, State);
    #handshake{} ->
        State;
    #stream_error{} ->
        process_stream_end({stream, {in, Pkt}}, State);
    _ when StateName == wait_for_sasl_request;
           StateName == wait_for_handshake;
           StateName == wait_for_sasl_response ->
        process_unauthenticated_packet(Pkt, State);
    _ when StateName == wait_for_starttls ->
        Txt = <<"Use of STARTTLS required">>,
        Err = xmpp:serr_policy_violation(Txt, Lang),
        send_pkt(State, Err);
    _ when StateName == wait_for_bind ->
        process_bind(Pkt, State);
    _ when StateName == established ->
        process_authenticated_packet(Pkt, State)
    end.
```

认证逻辑：xmpp_stream_in:process_sasl_request:817

```erlang
-spec process_sasl_request(sasl_auth(), state()) -> state().
process_sasl_request(#sasl_auth{mechanism = Mech, text = ClientIn},
             #{lserver := LServer} = State) ->
    State1 = State#{sasl_mech => Mech},
    Mechs = get_sasl_mechanisms(State1),
    case lists:member(Mech, Mechs) of
    true when Mech == <<"EXTERNAL">> ->
        Res = case xmpp_stream_pkix:authenticate(State1, ClientIn) of
              {ok, Peer} ->
              {ok, [{auth_module, pkix}, {username, Peer}]};
              {error, Reason, Peer} ->
              {error, Reason, Peer}
          end,
        process_sasl_result(Res, State1);
    true ->
        GetPW = get_password_fun(Mech, State1),
        CheckPW = check_password_fun(Mech, State1),
        CheckPWDigest = check_password_digest_fun(Mech, State1),
        SASLState = xmpp_sasl:server_new(LServer, GetPW, CheckPW, CheckPWDigest),
        Res = xmpp_sasl:server_start(SASLState, Mech, ClientIn),
        process_sasl_result(Res, State1#{sasl_state => SASLState});
    false ->
        process_sasl_result({error, unsupported_mechanism, <<"">>}, State1)
    end.
```

#### 资源绑定

xmpp_stream_in:process_bind:666

ejabberd_c2s:bind:393

```erlang
bind(R, #{user := U, server := S, access := Access, lang := Lang,
      lserver := LServer, socket := Socket,
      ip := IP} = State) ->
    case resource_conflict_action(U, S, R) of
    closenew ->
        {error, xmpp:err_conflict(), State};
    {accept_resource, Resource} ->
        JID = jid:make(U, S, Resource),
        case acl:access_matches(Access,
                    #{usr => jid:split(JID), ip => IP},
                    LServer) of
        allow ->
            State1 = open_session(State#{resource => Resource,
                         sid => ejabberd_sm:make_sid()}),
            State2 = ejabberd_hooks:run_fold(
                   c2s_session_opened, LServer, State1, []),
            ?INFO_MSG("(~s) Opened c2s session for ~s",
                  [xmpp_socket:pp(Socket), jid:encode(JID)]),
            {ok, State2};
        deny ->
            ejabberd_hooks:run(forbidden_session_hook, LServer, [JID]),
            ?WARNING_MSG("(~s) Forbidden c2s session for ~s",
                 [xmpp_socket:pp(Socket), jid:encode(JID)]),
            Txt = <<"Access denied by service policy">>,
            {error, xmpp:err_not_allowed(Txt, Lang), State}
        end
    end.
```

ejabberd_c2s:open_session:198 -> ejabberd_sm:open_session:154

```erlang
open_session(SID, User, Server, Resource, Priority, Info) ->
    set_session(SID, User, Server, Resource, Priority, Info),
    check_for_sessions_to_replace(User, Server, Resource),
    JID = jid:make(User, Server, Resource),
    ejabberd_hooks:run(sm_register_connection_hook,
               JID#jid.lserver, [SID, JID, Info]).
```

set_session函数可以根据配置把session信息存入不同介质（mnesia，redis，sql）配置选项：sm_db_type

#### 数据流

- 监听进程创建c2s事例

  ```erlang
  %ejabberd_listener:accept:238 module:ejabberd_c2s, socket:#Port<0.129638>, option:[{access,c2s},{shaper,c2s_shaper},{max_stanza_size,262144}],sup:ejabberd_c2s_sup
  start_connection(Module, Socket, Opts, Sup) ->
      Res = case Sup of
            undefined -> Module:start({gen_tcp, Socket}, Opts);
            _ -> supervisor:start_child(Sup, [{gen_tcp, Socket}, Opts])
        end,
      case Res of
      {ok, Pid} ->
          %移交对socket的控制权
          case gen_tcp:controlling_process(Socket, Pid) of
          ok ->
              %调用ejabberd_c2s:appept -> xmpp_stream_in:accept(Ref).
              Module:accept(Pid),
              {ok, Pid};
          Err ->
              exit(Pid, kill),
              Err
          end;
      Err ->
          Err
      end.
  ```

c2s事例创建分析：使用-behaviour(xmpp_stream_in).

xmpp_stream_in又使用-behaviour(p1_server).

在p1_server模块loop -> process_message -> collect_messages -> decode_msg -> handle_msg

handle_msg有三种类型：

```erlang
dispatch({'$gen_cast', Msg}, Mod, State) ->
    Mod:handle_cast(Msg, State);
dispatch(Info, Mod, State) -> %这里会接收到Tcp类型的消息，事例：{tcp,#Port<0.129638>,<<"<stream:stream to=\"localhost\" xmlns=\"jabber:client\" xmlns:stream=\"http://etherx.jabber.org/streams\" version=\"1.0\">">>}，这里的Mod:xmpp_stream_in
    Mod:handle_info(Info, State).

handle_msg({'$gen_call', From, Msg}, Parent, Name, State, Mod,
       Limits, Queue, QueueLen) ->
    case catch Mod:handle_call(Msg, From, State) of
    {reply, Reply, NState} ->
        reply(From, Reply),
        loop(Parent, Name, NState, Mod, infinity, [],
                 Limits, Queue, QueueLen);
    {reply, Reply, NState, Time1} ->
        reply(From, Reply),
        loop(Parent, Name, NState, Mod, Time1, [],
                 Limits, Queue, QueueLen);
    {noreply, NState} ->
        loop(Parent, Name, NState, Mod, infinity, [],
                 Limits, Queue, QueueLen);
    {noreply, NState, Time1} ->
        loop(Parent, Name, NState, Mod, Time1, [],
                 Limits, Queue, QueueLen);
    {stop, Reason, Reply, NState} ->
        {'EXIT', R} = 
        (catch terminate(Reason, Name, Msg, Mod, NState, [], Queue)),
        reply(From, Reply),
        exit(R);
    Other -> handle_common_reply(Other, Parent, Name, Msg, Mod, State,
                                     Limits, Queue, QueueLen)
    end;
handle_msg(Msg, Parent, Name, State, Mod,
       Limits, Queue, QueueLen) ->
    Reply = (catch dispatch(Msg, Mod, State)),
    handle_common_reply(Reply, Parent, Name, Msg, Mod, State,
                        Limits, Queue, QueueLen).
```

xmpp_stream_in:handle_info:381

```erlang
%完成对Tcp消息的解析xmpp_socket:recv -> parse:383行 -> fxml_stream:parse  这里使用了c的代码，会将tcp消息转成gen_event，这条转换的消息最早也是在p1_server模块loop -> process_message -> collect_messages -> decode_msg -> handle_msg接收的，事例：{'$gen_event',{xmlstreamstart,<<"stream:stream">>,[{<<"xmlns">>,<<"jabber:client">>},{<<"xmlns:stream">>,<<"http://etherx.jabber.org/streams">>},{<<"to">>,<<"localhost">>},{<<"version">>,<<"1.0">>}]}} 
handle_info({tcp, _, Data}, #{socket := Socket} = State) ->
    noreply(
      case xmpp_socket:recv(Socket, Data) of
      {ok, NewSocket} ->
          State#{socket => NewSocket};
      {error, Reason} when is_atom(Reason) ->
          process_stream_end({socket, Reason}, State);
      {error, Reason} ->
          %% TODO: make fast_tls return atoms
          process_stream_end({tls, Reason}, State)
      end);
```

当gen_event消息再次到达xmpp_stream_in:handle_info:343

```erlang
%gen_event消息在执行钩子函数后->process_element ->process_element(Pkt, #{stream_state := StateName, lang := Lang} = State):589 在这里完成认证bind等操作，当StateName == established时，iq_session、iq_roster 协议都会调用process_authenticated_packet -> ejabberd_c2s:handle_authenticated_packet：469  
handle_info({'$gen_event', {xmlstreamelement, El}},
        #{xmlns := NS, codec_options := Opts} = State) ->
    noreply(
      try xmpp:decode(El, NS, Opts) of
      Pkt ->
          %回调钩子函数进入c2s模块的handle_recv
          State1 = try callback(handle_recv, El, Pkt, State)
               catch _:{?MODULE, undef} -> State
               end,
          case is_disconnected(State1) of
          true -> State1;
          false -> process_element(Pkt, State1)
          end
      catch _:{xmpp_codec, Why} ->
          State1 = try callback(handle_recv, El, {error, Why}, State)
               catch _:{?MODULE, undef} -> State
               end,
          case is_disconnected(State1) of
          true -> State1;
          false -> process_invalid_xml(State1, El, Why)
          end
      end);
```

ejabberd_c2s:handle_authenticated_packet

```erlang
handle_authenticated_packet(Pkt, #{lserver := LServer, jid := JID,
                   ip := {IP, _}} = State) ->
    Pkt1 = xmpp:put_meta(Pkt, ip, IP),
    State1 = ejabberd_hooks:run_fold(c2s_authenticated_packet,
                     LServer, State, [Pkt1]),
    #jid{luser = LUser} = JID,
    {Pkt2, State2} = ejabberd_hooks:run_fold(
               user_send_packet, LServer, {Pkt1, State1}, []),
    case Pkt2 of
    drop ->
        State2;
    #iq{type = set, sub_els = [_]} ->
        try xmpp:try_subtag(Pkt2, #xmpp_session{}) of
        #xmpp_session{} ->
            send(State2, xmpp:make_iq_result(Pkt2));
        _ ->
            check_privacy_then_route(State2, Pkt2)
        catch _:{xmpp_codec, Why} ->
            Txt = xmpp:io_format_error(Why),
            Lang = maps:get(lang, State),
            Err = xmpp:err_bad_request(Txt, Lang),
            send_error(State2, Pkt2, Err)
        end;
    #presence{to = #jid{luser = LUser, lserver = LServer,
                lresource = <<"">>}} ->
        process_self_presence(State2, Pkt2);
    #presence{} ->
        process_presence_out(State2, Pkt2);
    _ ->
        %路由逻辑 -> ejabberd_router:route -> ejabberd_router:do_route:353 -> ejabberd_local:do_route:142 local route  -> ejabberd_sm:do_route:693 processing IQ to bare JID ->  gen_iq_handler:handle
        check_privacy_then_route(State2, Pkt2)
    end.
```

gen_iq_handler:handle:76

```erlang
handle(Component,
       #iq{to = To, type = T, lang = Lang, sub_els = [El]} = Packet)
  when T == get; T == set ->
    XMLNS = xmpp:get_ns(El),
    Host = To#jid.lserver,
    case ets:lookup(Component, {Host, XMLNS}) of
    [{_, Module, Function}] ->
        %根据iq注册的名字空间，调用响应的mod模块实现，可以参照mod_echo，最终将type为set、get的请求处理结果发送到ejabberd_router:route(ResIQ)
        process_iq(Host, Module, Function, Packet);
    [] ->
        Txt = <<"No module is handling this query">>,
        Err = xmpp:err_service_unavailable(Txt, Lang),
        ejabberd_router:route_error(Packet, Err)
    end;
```
