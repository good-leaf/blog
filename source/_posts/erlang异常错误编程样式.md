---
title: erlang异常错误编程样式
date: 2019-02-23 14:22:53
updated: 2019-02-23 14:22:53
categories: 开发语言
tags: erlang异常错误编程样式

---

### try catch end和catch区别

- catch在try catch end引入之前就时erlang语言的一部分了。

- 异常错误如果发生在catch语句中，就会被转换成一个描述此错误的{'EXIT', ...}元组。

### 异常错误编程样式

1. 改进错误信息

   ```erlang
   sqrt(X) when X < 0 ->
       error({argument_error, X});
   sqrt(X) ->
       match:sqrt(X).
   ```

2. 经常返回错误时代码

   如果函数没有什么“通常的情形”，那么多半返回{ok, Val}或者{error, Reason}，这样迫使所有调用者必须对返回值做什么。一种代码编写：

   ```erlang
   case f(X) of
       {ok, Val} ->
           do_some_thine_with(Val);
       {error, Why} ->
           %% ...处理错误....
   end.
   ```

   两种返回都要处理。

   另一种代码编写：

   ```erlang
   {ok, Val} = f(X),
   do_some_thing_with(Val).
   ```

   这样，如果f(X)返回错误，就会抛出错误。

3. 错误可能有但罕见时的代码

   这种情况下，通常要编写能够处理错误的代码。

   ```erlang
   try f(X)
   catch
       throw:{thisError, X} -> ....
       throe:{someOtherError, X} -> ....
   end
   ```

   f(X) 函数实现：

   ```erlang
   f(X) ->
       case ... of
           ... ->
              Result;
           ... ->
              throw({thisError,...});
           ... ->
              throw({someOtherError,...}) 
       end.             
   ```

4. 捕捉一切可能的异常错误

   ```erlang
   try Expr 
   catch
       _:_ ...  匹配处理所有异常
   end       
   或者：
   try Expr
   catch
       _ ... 默认只能处理throw:_ 类型的错误
   end    
   ```

### 事例

```erlang
generate_exception(1) -> a;
generate_exception(2) -> throw(a);
generate_exception(3) -> exit(a);
generate_exception(4) -> {'EXIT', a};  %%模拟返回错误
generate_exception(5) -> error(a).

catcher(N) ->
    try generate_exception(N) of
        Val -> {N, normal, Val}
    catch 
        throw:X -> {N, caught, throw, X};
        exit:X -> {N, caught, exited, X};
        error:X -> {N, caucaughtgth, error, X}
    end.       
    
demo1() ->
    [catcher(I) || I <- lists:seq(1,5)].
    
demo2() ->
[{I, (catch generate_exception(I))} || I <- lists:seq(1,5)].
    
```

结果：

```erlang
2> demo1().
[{1,normal,a},
 {2,caught,throw,a},
 {3,caught,exited,a},
 {4,normal,{'EXIT',a}},
 {5,caught,error,a}]
3> demo2().
[{1,a},
 {2,a},
 {3,{'EXIT',a}},
 {4,{'EXIT',a}},
 {5,
  {'EXIT',{a,[{a,generate_exception,1,
                 [{file,"a.erl"},{line,7}]},
              {a,'-demo2/0-lc$^0/1-0-',1,[{file,"a.erl"},{line,19}]},
              {a,'-demo2/0-lc$^0/1-0-',1,[{file,"a.erl"},{line,19}]},
              {erl_eval,do_apply,6,[{file,"erl_eval.erl"},{line,674}]},
              {shell,exprs,7,[{file,"shell.erl"},{line,686}]},
              {shell,eval_exprs,7,[{file,"shell.erl"},{line,641}]},
              {shell,eval_loop,3,[{file,"shell.erl"},{line,626}]}]}}}]
```

通过catch捕捉这三种异常返回的结果分别是：

  throw(Any) -> Term

  exit(Reason) -> {'EXIT',Reason}

  error(Reason) -> {'EXIT',{Reason,erlang:get_stacktrace()}}

使用这三种异常的场景为：

exit(Reason): 当想要终止当前进程时，用这个函数。如果这个消息未被捕获，那么系统会向所有与当前进程连接的进程广告{'EXIT',Pid,Reason}消息

throw(Any): 用于抛出一个调用者可能会捕获的异常。针对throw，必须为函数添加注释，说明他会抛出这个异常。调用者可以选择：忽略这些异常/对异常进行处理。

error(Reason): 用于抛出那些“崩溃错误“。这种异常应该是调用者不会真正意识到要去处理的那些致命错误。










