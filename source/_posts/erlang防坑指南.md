---
    title: erlang 防坑指南
    date: 2018-12-16 18:35:26
    updated: 2018-12-16 18:35:26
    categories: erlang
    tags:
---

任何语言在使用中都会遇到这样那样的问题，erlang也是。这里整理下我遇到的一些问题，避免继续踩坑。说实话，“防坑指南”这个标题有点过于标新立异，不过还是希望能引起重视，避免在实际开发中重复犯这些问题。
<!--more-->

### '--' 运算与 '++'运算

```
1> [1,2,3,4] -- [1] -- [2]. 
[2,3,4]
算是erlang经典的问题了。这是从后面算起的，先算 [1] -- [2] ，得到 [1] 后被 [1,2,3,4] --，最后得到 [2,3,4]
 '++'运算也是一样的，也是从后面开始算起。
2> [1,2,3,4] -- [1] ++ [2,3,4].
[]
另外，以下这种情况也要注意，只会减去遇到的第一个元素。
3> [1,2,3,2] -- [2].
[1,3,2]
```

### erlang:function_exported()

这个接口是用来检查模块函数是否导出，但是，如果模块没加载过，这个函数返回值就是false

```
3> erlang:function_exported(odbc,start,0).
false
4> odbc:start().
ok
5> erlang:function_exported(odbc,start,0).
true
```

### erlang:list_to_binary()

如果参数是多层嵌套结构，就会被扁平化掉，使用 binary_to_list 不能转成原来的数据，也就是不可逆的。

```
6> list_to_binary([1,2,[3,4],5]) .
<<1,2,3,4,5>>
如果想可逆，可以使用 erlang:term_to_binary
7> binary_to_term(term_to_binary([1,2,[3,4],5])).
[1,2,[3,4],5]
```

### random:uniform()

这个用于生成随机数，返回一个随机数浮点数。但是，这个函数的随机初始种子是个定值，而且种子就放在进程字典，就是说每个进程生成的随机数都是一样的。坑爹啊。

```
10> spawn(fun() -> io:format("~w ~w~n",[random:uniform(),random:uniform()]) end)
,ok.
0.4435846174457203 0.7230402056221108
ok
11> spawn(fun() -> io:format("~w ~w~n",[random:uniform(),random:uniform()]) end)
,ok.
0.4435846174457203 0.7230402056221108
ok
```

所以，解决的方法就是进程启动后要重置随机数种子，然后再使用这个函数。

```
13> random:seed(erlang:now()), random:uniform().
0.4691405130019146
```

### io_lib:char_list()

这个函数在R15和R16下运行结果可能是相反的。

```
R15下：
Eshell V5.9.3.1 (abort with ^G)
1> io_lib:char_list([10100,10600,20100]).
false

R16下：
Eshell V5.10.1 (abort with ^G) 
1> io_lib:char_list([10100,10600,20100]). 
true
```

### 不同类型数据比较

比较公式为：number < atom < reference < fun < port < pid < tuple < list < bit string

### 系统限制

```
1. mnesia或dets有2G限制（无法定制）
2. ets表最大数量默认1400（可用 ERL_MAX_ETS_TABLES 定制）
3. 原子最大数量默认 1048576 （可用 +t 定制）
4. 进程最大数量默认 32768   （可用  +P 定制， 范围1024-134217727）
5. 端口/文件句柄最大数量默认 16384  （可用  +Q  定制， 范围1024-134217727）
```
