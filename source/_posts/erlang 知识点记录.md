---
title: erlang知识点记录
date: 2019-01-08 14:22:53
updated: 2019-01-08 14:22:53
categories: develop-language
tags: erlang
---

1. gen_server:cast和erlang:send()都可以向指定进程发送消息，两者有什么区别？

   gen_server:cast 内部调用erlang:send使用noconnect

   ```erlang
   case catch erlang:send(Dest, Msg, [noconnect]) of
       noconnect ->
           spawn(erlang, send, [Dest,Msg]);
       Other ->
           Other
       end.
   ```

   ```erlang
   -spec erlang:send(Dest, Msg, Options) -> Res when
         Dest :: dst(),
         Msg :: term(),
         Options :: [nosuspend | noconnect],
         Res :: ok | nosuspend | noconnect.
   ```

   nosuspend：遇到会挂起进程时不挂起进程，直接返回nosuspend

   noconnect：遇到远程节点没有连接时不自动连接发送消息，直接返回noconnect

2. gen_server:call

3. rpc:call和gen_server:call 区别？

   rpc模块本身就是gen_server进程，随kernel模块启动，rpc进程启动时通过local注册一个rex的名字。

   rpc:call内部调用gen_server:call({Name,Node},Request)，Name是rex，所以说rpc:call是调用远程节点的rex进程做事情，而gen_server:call可以调用任意进程做事情。

4. erlang:now()和os:timestamp()区别？

   erlang:now()获取erlang虚拟机时间，os:timestamp()获取操作系统时间，对于erlang:start_time函数如果调快系统时间，此定时不会提前收到消息。因为erlang:start_time内部使用erlang虚拟机时间。

5. erlang:send_after和erlang:start_time区别？

   主要是TimerRef，超时消息进入邮箱时，start_time函数的消息携带了TimeRef标识。

6. ref数据类型

   Erlang 虚拟机会创建一个新的 ref。由于全局当前 ref 值是用多个变量表示的，所以 make_ref() 会通过一个自旋锁保护对这些变量的操作，递增全局 ref 的值，然后根据新的 ref 值创建新的 ref 对象并返回对应的 Eterm。递增操作针对 word 0 递增，如果 word 0 超过了218218，则进位到 word 1，word 1 归零的话则进位到 word 2。

   我们打印 ref 的时候，得到的是类似 #Ref<0.0.0.2055> 这样的输出，通过 3 个句点将输出结果分为 4 段。第 1 段，和 pid 和 port 的第一段是一样的，表示节点，在本地节点总是为 0，后面 3 段分别为上面的 word 2、1 和 0。所以 ref 较少的时候前面几段都为 0。

7. ets表

   ```erlang
   -spec new(Name, Options) -> tid() | atom() when
         Name :: atom(),
         Options :: [Option],
         Option :: Type | Access | named_table | {keypos,Pos}
                 | {heir, Pid :: pid(), HeirData} | {heir, none} | Tweaks,
         Type :: type(),
         Access :: access(),
         Tweaks :: {write_concurrency, boolean()}
                 | {read_concurrency, boolean()}
                 | compressed,
         Pos :: pos_integer(),
         HeirData :: term().
   ```

   write_concurrency、read_concurrency是用来提升读写性能的，代价是额外的内存。并不是支持读和写的并发控制的，因为ets本身的读写操作就是原子的。通常来说，**ets写数据时整张表是锁定的**，其他进程不能进行读写直到前面的操作完成。并发写可以改变这个情况，**同一个表中的不同记录可以被多个进程并发读写**。有了这个参数，使得ets写记录时表读写锁变成了读锁，就是说，只要不是同一条记录，还可以继续往这个ets表写入数据，提高了并发写效率。但并发写也有弊端，降低数据连续写入的效率和性能。如果有且只有一个进程在读写数据，将会带来一定的开销。而测试发现这个开销比较小，可以忽略。而且，只有一个进程在读写数据的场合比较小。

8. Pid <A,B,C>

   A对应节点信息（0代表本地节点，其他数字代表远程节点）

   B低15字节代表进程表索引

   C16～18字节代表进程唯一标识

9. erlang:dbg、trace、火焰图

10. receive的理解

    receive会检查遍历进程的邮箱一次，如果匹配到条件，就执行条件后的代码，并去掉邮箱中对应消息，停止匹配过程。等待下一条消息到达时触发再次匹配逻辑。




