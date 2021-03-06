---
layout: post
title:  Lua中的协程
date:   2014-09-07 03:01:00 +0800
categories: code
tags: lua
---
Lua中的协程和其他变量一样，都是第一类值（first-class alue），可以被保存在变量中，可以被作为参数传递，可以被函数返回。  
协程有4种状态：挂起（suspended），运行（running），死亡（dead）和正常（normal）。
Lua为协程提供了3个基础接口：create，resume和yield。  

#coroutine.create

- 创建一个新的协程，并为它的运行分配一个独立的栈
- 协程处于挂起状态（suspended）
- 接受一个函数作为参数，这个函数就是协程的主程序块
- 返回这个协程
- 挂起点被设置为主程序块的第一句

#coroutine.resume

- 启动一个协程（第一次启动或从暂停状态启动）
- 自身（如果是协程的话）处于正常状态，被启动的协程处于运行状态
- 第一个参数为所要启动的协程
- 协程从它的挂起点开始执行
- 一直执行到被挂起或终止
- 导致协程终止的情况有两种：它的主程序块正常返回、运行过程中出错
- 执行结束后，控制权递交给此协程被激活的地方

#coroutine.yield

- 挂起一个协程
- 协程处于挂起状态
- 协程的运行状态被记录
- 激活它的那个coroutine.resume返回

----------

#协程间通信

- 协程第一次被启动时，传递给coroutine.resume的参数将传递给协程的主程序
- 协程挂起时，传递给coroutine.yield的参数将作为上次启动它的coroutine.resume的返回值返回
- 协程被再次启动时，传递给coroutine.resume的参数将作为上次挂起它的coroutine.yield的返回值返回
- 协程死亡时，主程序返回的值将作为上次启动它的coroutine.resume的返回值返回

----------

#实验

##状态

{% highlight lua %}
local function status(str, c)
    print(str, coroutine.status(c))
end

local c1,c2
c1 = coroutine.create(function()
    status("<c2>", c2)
    print("before c1 yield")
    coroutine.yield()
    print("after c1 yield")
end)
c2 = coroutine.create(function()
    status("<c2>", c2)
    print("before c2 resume c1")
    coroutine.resume(c1)
    print("after c2 resume c1")
end)

status("<c2>", c2)
coroutine.resume(c2)
status("<c1>", c1)
status("<c2>", c2)
coroutine.resume(c1)
status("<c1>", c1)
{% endhighlight %}

输出：

    outsky@x201:~/tmp$ lua test.lua 
	<c2>	suspended
	<c2>	running
	before c2 resume c1
	<c2>	normal
	before c1 yield
	after c2 resume c1
	<c1>	suspended
	<c2>	dead
	after c1 yield
	<c1>	dead


----------

##通信

{% highlight lua %}
local c = coroutine.create(function(...)
    print("c start:", ...)
    print("c yield return:", coroutine.yield("c yield"))
    return "c dead"
end)

print("main start c")
local _,r = coroutine.resume(c, "main start c")
print("main resume return:", r)

print("--------")

print("main resume c")
_,r = coroutine.resume(c, "main resume c")
print("main resume return again:", r)
{% endhighlight %}

输出：

    outsky@x201:~/tmp$ lua test.lua 
    main start c
    c start:	main start c
    main resume return:	c yield
    --------
    main resume c
    c yield return:	main resume c
    main resume return again:	c dead

