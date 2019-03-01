---

title: ejabberd编码风格
date: 2019-03-01 17:22:53
updated: 2019-03-01 17:22:53
categories: 开发语言
tags: erlang

---

### ejabberd编码风格

[Erlang编码规则](http://www.erlang.se/doc/programming_rules.shtml)

[ejabberd编码风格](https://github.com/inaka/erlang_guidelines/blob/master/README.md)

- 更改别人的代码时，保持现有风格。

- 2个空格缩进

- 参数间放置空格

  ```erlang
  -module(spaces).
  -export([bad/3, good/3]).
  % @doc no spaces
  bad(_My,_Space,_Bar)->[is,'not',working].
  % @doc spaces!!
  good(_Hey, _Now, _It) -> ["works " ++ "again, " | [hooray]].
  ```

- 行尾不留空白

- 每行100列，100字符为最大值

- case表达式执行小函数

  ```erlang
  -module(smaller_functions).
  -export([bad/0, bad/1, good/0, good/1]).
  %% @doc function with just a case
  bad(Arg) ->
    case Arg of
      this_one -> should:be(a, function, clause);
      and_this_one -> should:be(another, function, clause)
    end.
  %% @doc usage of pattern matching
  good(this_one) -> is:a(function, clause);
  good(and_this_one) -> is:another(function, clause).
  %% @doc function with an internal case
  bad() ->
    InitialArg = some:initial_arg(),
    InternalResult =
    case InitialArg of
      this_one -> should:be(a, function, clause);
        and_this_one -> should:be(another, function, clause)
      end,
    some:modification(InternalResult).
  %% @doc usage of function clauses instead of an internal case
  good() ->
    InitialArg = some:initial_arg(),
    InternalResult = good(InitialArg),
    some:modification(InternalResult).
  ```

- 模块功能单一，不要把无关功能放在一个模块中

- 单元测试，一个函数测试一个点，不要把多个测试点写一起

- 按功能为模块分组，设置子目录

- 头文件

  不应包含类型定义、记录定义、函数定义；

  可能包含红定义，但应避免使用宏；

  在头文件中包含记录定义可促进跨模块共享这些记录的内部细节，增加耦合并防止封装，从而使更改和维护代码变得更加困难。记录应该在他们自己的模块中定义，这些模块应该提供不透明的数据类型和访问和操作记录的功能。

  函数定义绝对不应包含在头文件中，因为它会导致代码重复。

- 程序的函数调用图应该努力成为有向无环图

  ```erlang
  -module(spaghetti).
  -export([bad/0, good/0]).
  bad() ->
    Client = active_user:get_current_client(),
    [binary_to_list(Org)
     || Org <- autocomplete_db:members(
                case Client of
                  home_client ->
                    <<"our:organizations">>;
                  aperture_science ->
                    <<"client:", (prefix_for(aperture_science))/binary, ":orgs">>;
                  wayne_ents ->
                    <<"client:", (prefix_for(wayne_ents))/binary, ":orgs">>
                end)].
  
  good() ->
    Client = active_user:get_current_client(),
    RawOrgs = autocomplete_db:members(client_ac_key(Client)),
    [binary_to_list(Org) || Org <- RawOrgs].
  
  client_ac_key(home_client) -> <<"our:organizations">>;
  client_ac_key(Client) ->
    Prefix = prefix_for(Client),
    <<"client:", Prefix/binary, ":orgs">>.
  
  prefix_for(aperture_science) -> <<"as">>;
  prefix_for(wayne_ents) -> <<"we">>.
  ```

- 避免动态调用

  动态调用语法，不能使用xref对代码进行检查。

  ```erlang
  -module(dyn_calls).
  
  -export([bad/1, good/1]).
  
  bad(Arg) ->
    Mods = [module_1, module_2, module_3],
    Fun = my_function,
    lists:foreach(
      fun(Mod) ->
        Mod:Fun(Arg)
      end, Mods).
  
  good(Arg) ->
    mdoule_1:my_function(Arg),
    module_2:my_function(Arg),
    module_3:my_function(Arg).
  ```

- 避免深层嵌套

  嵌套级别表示函数中的逻辑深度。

- 避免if表达式

  if在代码中引入了静态布尔逻辑，从而降低了代码的灵活性。

  ```erlang
  
  -module(no_if).
  
  -export([bad/1, better/1, good/1]).
  
  bad(Connection) ->
  
    {Transport, Version} = other_place:get_http_params(),
  
    if
  
      Transport =/= cowboy_spdy, Version =:= 'HTTP/1.1' ->
        [{<<"connection">>, utils:atom_to_connection(Connection)}];
      true ->
        []
  
    end.
  
  better(Connection) ->
  
    {Transport, Version} = other_place:get_http_params(),
  
    case {Transport, Version} of
  
      {cowboy_spdy, 'HTTP/1.1'} ->
        [{<<"connection">>, utils:atom_to_connection(Connection)}];
      {_, _} ->
        []
  
    end.
  
  good(Connection) ->
  
    {Transport, Version} = other_place:get_http_params(),
  
    connection_headers(Transport, Version, Connection).
  
  connection_headers(cowboy_spdy, 'HTTP/1.1', Connection) ->
  
      [{<<"connection">>, utils:atom_to_connection(Connection)}];
  
  connection_headers(_, _, _) ->
  
      [].
  ```

- 避免嵌套try....catch

  ```erlang
  -module(nested_try_catch).
  
  -export([bad/0, good1/0, good2/0]).
  
  bad() ->
    try
      maybe:throw(exception1),
      try
        maybe:throw(exception2),
        "We are safe!"
      catch
        _:exception2 ->
          "Oh, no! Exception #2"
      end
    catch
      _:exception1 -> "Bummer! Exception #1"
    end.
  
  good1() ->
    try
      maybe:throw(exception1),
      maybe:throw(exception2),
      "We are safe!"
    catch
      _:exception1 ->
        "Bummer! Exception #1";
      _:exception2 ->
        "Oh, no! Exception #2"
    end.
  
  good2() ->
    try
      maybe:throw(exception1),
      a_function:that_deals(with, exception2),
      "We are safe!"
    catch
      _:exception1 ->
        "Bummer! Exception #1"
    end.
  ```

- 模块命名要统一

- 函数名称

    函数名称只能使用小写字符和数字。函数名中单词必须用"_" 

- 变量名称 

  CameCase必须用于变量。

- iolists 尽可能使用

  ```erlang
  -module(iolists).
  
  -export([good/1, bad/1]).
  
  bad(Param) -> "Hello " ++ binary_to_list(Param) ++ "! Have a nice day!".
  
  good(Param) -> ["Hello ", Param, "! Have a nice day!"].
  ```

- 大写宏

    宏使代码更难调试。

- 尽量不要把宏作为模块或者函数名

  ```erlang
  -module(macro_mod_names).
  
  -define(SERVER, ?MODULE). % Oh, god! Why??
  -define(TM, another_module).
  
  -export([bad/1, good/1]).
  
  bad(Arg) ->
    Parsed = gen_server:call(?SERVER, {parse, Arg}),
    ?TM:handle(Parsed).
  
  good(Arg) ->
    Parsed = gen_server:call(?MODULE, {parse, Arg}),
    another_module:handle(Parsed).
  ```

- 记录名称

  记录名称只能使用小写字符，记录名称中的单词必须用"_"分隔。同样的规则适用于记录字段名称。

  ```erlang
  -module(record_names).
  
  -export([records/0]).
  
  -record(badName, {}).
  -record(bad_field_name, {badFieldName :: any()}).
  -record('UPPERCASE', {'THIS_IS_BAD' :: any()}).
  
  -record(good_name, {good_field_name :: any()}).
  
  records() -> [#badName{}, #bad_field_name{}, #'UPPERCASE'{}, #good_name{}].
  ```

- 不要分享记录

  记录不应在多个模块之间共享。如果需要共享表示为记录的*对象*，请使用opaque导出类型并在模块中提供足够的访问者函数。

  ```erlang
  -module(record_sharing).
  
  -include("record_sharing.hrl").
  
  -export([bad/0, good/0, good_field/1, good_field/2]).
  
  -record(good, {good_field :: string()}).
  -opaque good() :: #good{}.
  -export_type([good/0]).
  
  -spec good() -> good().
  good() -> #good{}.
  
  -spec good_field(good()) -> string().
  good_field(#good{} = Good) -> Good#good.good_field.
  
  -spec good_field(good(), string()) -> good().
  good_field(#good{} = Good, Value) -> Good#good{good_field = Value}.
  
  -spec bad() -> #bad{}.
  bad() -> #bad{}.
  ```
