---
layout: post
title: "深入浅出Coroutine"
date: 2012-08-18 22:55
comments: true
categories: programming 
---
说到coroutine(中文一般翻译成**协程**)，对于Java程序员来说可能有点陌生，因为Java语言本身并不支持coroutine，但是早在1963年这个想法就被提出来了。到目前为止，已经有很多语言提供了对coroutine的支持，比如Ruby，Python，Go，Erlang等。所以，理解coroutine是很有必要的。Lua是原生支持coroutine的语言之一，下面关于coroutine的例子都将使用Lua。

<!-- more -->

维基上给出的Coroutine定义是：

> Coroutines are computer program components that generalize subroutines to allow multiple entry points for suspending and resuming execution at certain locations.

上面这段中包含了另一个名词：subroutine，所以先来看一下subroutine的定义：

> In computer science, a subroutine, also termed procedure, function, routine, method, or subprogram, is a part of source code within a larger computer program that performs a specific task and is relatively independent of the remaining code.

也就是说subroutine就是方法或代码片段，方法或代码片段就是subroutine。现在就能翻译coroutine的定义了：

**Coroutine是允许多入口，即使方法或代码片段能够在某个地方挂起和继续执行的特性的程序组件。**

这句话对于Java程序员来说是具有颠覆性作用的，因为对于Java和那些不支持coroutine的语言来说，方法只有一个入口，也就是从方法体的大括号开始到结束的大括号为止。比如有个方法：

{% codeblock lang:java %}
public void foo(){
	System.out.print("hello");
	System.out.print("world");
}
{% endcodeblock %}

每当foo方法被调用的时候，方法体中的语句都会被执行，即每次都会打印“helloworld”。然而coroutine允许方法执行到一半就挂起(通常是碰到yield关键字)，然后上下文状态会被保存下来，并且可以在之后恢复这个coroutine。考虑下面这段lua代码：

{% codeblock lang:lua %}
function foo()
	print("hello")
	coroutine.yield() -- 在这里挂起coroutine
	print("world") 
end

local co = coroutine.create(foo) -- co保存的是上下文变量
coroutine.resume(co)         -- 只打印出 hello
coroutine.resume(co)         -- 只打印出 world
{% endcodeblock %}

第一次执行coroutine.resume(co)的时候，会从头开始执行foo函数，当遇到yield关键字时就会挂起coroutine，等到第二次执行coroutine.resume(co)的时候，会从上次coroutine挂起的地方继续执行，直到再次遇到yield或着函数结束。

coroutine在yield时还可以指定另一个coroutine，下面是维基上的例子：

{% codeblock %}
var q := new queue

coroutine produce
    loop
        while q is not full
            create some new items
            add the items to q
        yield to consume

coroutine consume
    loop
        while q is not empty
            remove some items from q
            use the items
        yield to produce
{% endcodeblock %}

注意到Lua的coroutine不支持上面的这种指定另外coroutine的特性，所以类似Lua的coroutine实现又被称为asymmetric coroutine(非对称协程)。

其实也可以使用多线程来实现上面例子中的生产者-消费者问题，但线程相比于协程的实现又有哪些区别呢？一般来说有以下几点不同：

* 线程需要有操作系统来调度，而coroutine则是由程序自己来调度
* 线程的上下文切换开销较大，coroutine则要小的多
* 线程会有竞争，coroutine则没有
* 多线程的实现在某一时刻会有多个线程在运行，而coroutine则是只有一个


[1] [Coroutine From Wikipedia](http://en.wikipedia.org/wiki/Coroutine "Coroutine From Wikipedia")  
[2] [Lua Coroutines Documentation](http://www.lua.org/manual/5.2/manual.html#2.6 "Lua Coroutines Documentation")

