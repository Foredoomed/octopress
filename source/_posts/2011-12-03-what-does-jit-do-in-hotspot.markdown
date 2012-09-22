---
layout: post
title: "JIT到底做了些什么事"
date: 2011-12-03 15:15
comments: true
categories: java
---
最近在网上看到一篇关JIT(Just in Time)的文章[Just in Time Compiler (JIT) in Hotspot](http://java.dzone.com/articles/just-time-compiler-jit-hotspot?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+javalobby%2Ffrontpage+%28Javalobby+%2F+Java+Zone%29&utm_content=Google+Reader "Just in Time Compiler (JIT) in Hotspot"),这篇文章很短，但是对JIT的作用基本上说的比较清楚的，可以作为JIT的学习参考，所以我就决定把这篇文章翻译成中文，供以后复习参考用。

<!--more-->

================正文开始==================

什么是JIT编译器

JIT和更普便的自适应优化概念是在包括Java在内的许多编程语言，例如.Net, Lua, JRuby中所为人熟知的概念。

为了解释什么是JIT编译器，我想从编译器的定义开始。根据维基百科上对于编译器的定义：
> 一个计算机程序是把源程序语言转换成目标语言。

我们对静态Java编译器(javac)都很熟悉，它的作用是将人类能够阅读的.java文件编译成为能够被JVM解释的.class文件。现在问题来了，那么JIT用来编译什么东西呢？答案将在解释完什么是Just in Time后给出。

根据许多研究得出的结果，20%的代码的执行时间要占到全部时间的80%。如果这时候有一种方法决定那些20%的代码并且优化他们，那将是令人兴奋的事。这就是JIT做的事：在运行时收集统计数据，找出那些经常执行的代码，把他们从字节码转换成操作系统能够直接执行的本地代码，并且重度优化他们。最小的编译单元是单个方法。编译和统计数据收集是靠特殊线程并行处理的。在统计数据收集期间，编译器会推测代码功能，并且通过时间的推移来验证这个推测。如果这个推测是错误的话，优化过的代码会被还原并且重新编译。

Hotspot JVM这个名字的由来就是因为这个JVM的作用就是找到代码中的“hot spot(热点)”。

JIT会做哪些优化？

* 内敛方法 - 方法不是直接在对象的实例上调用，而是拷贝到调用代码里去。为了防止产生任何开销，热门方法应该尽可能靠近调用着
* 如果监视器从别的线程是不可到达的，那就消除锁
* 用直接的方法调用代替接口以消除调用虚函数的开销
* 合并临近的在同一对象上的锁
* 消除不会被执行的代码(dead code)
* 对非volatile变量，忽略内存写
* 删除预先检查的NullPointerException和IndexOutOfBoundsException
* 等等

当JVM调用一个Java方法，它会使用在已经加载类的方法块中的定义的调用方法。虚拟机有好几个调用方法，比如，如果方法是同步的，或者是本地方法的话，不同的调用者会被使用。JIT编译器使用的是它自己的调用者。虚拟机会检查ACC_MACHINE_COMPILED这个值，来通知解释器这个值对应的方法已经编译并且保持在已加载的类中。JIT编译器把方法块编译成本地代码，并且把它保存在那个方法的代码块中。一旦ACC_MACHINE_COMPILED位被设置，那么这个代码就被编译完成了。那我们怎么知道JIT在我们的程序中正在做什么，怎么样才能控制它？

首先就是取消JIT的功能：Djava.compiler=NONE

Hotspot中有2种JI编译器，一种是为客户端程序使用，另一种是服务器端程序。运行在服务器上的程序通常来说需要更多的资源，并且程序的吞吐量是非常重要的。因此，服务器版的JIT会消耗更多的资源，并且为了统计数据的准确，要花更多的时间来收集统计数据。而客户端版的JIT为一个方法收集统计数据会持续1500次方法调用，服务器版JIT则是15000次。这些默认值可以通过JVM参数:-XX:CompileThreshold=XXX来修改。

JIT的不足

JIT增加了Java程序的不可预测性和复杂性。它增加了一层程序员不是正真理解的另外一层。

一个可能bug的例子：并发中的”happens before relations”。JIT会简单的记录在单线程中代码的改变是否安全。解决这个问题的方法是加上**synchronized**或着显式的加锁。

JIT的优势

* 对于GC来说，程序必须到达安全点。为了这个目的，JIT通过在本地代码的相同间隔内注入**yieldpoints**。
* 除了扫描堆栈来找出根引用，寄存器也必须被扫描，因为他们可能会持有JIT创建的对象。

