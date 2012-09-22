---
layout: post
title: "Java的四种引用类型"
date: 2011-03-05 14:58
comments: true
categories: java
---
在Java中有四种引用类型，他们是：强引用(Strong Reference)，软引用(Soft Reference)，弱引用(Weak Reference) 和 虚引用(Phantom Reference)。

**一.四种引用类型的解释：**
<ul>
<li>JVM会持有一般对象直到他们不再是可触及的状态。换句话说，当没有任何有效引用指向他们的时候会被垃圾回收，无效引用不会被计算在内。</li>
<li>软引用指向的对象会在不存在任何指向他们的引用并且内存空间不足情况下被垃圾收集。大多数情况下被用来实现内存敏感的缓存。没有GC的时间限制，会在OOM发生之前清理完毕。</li>
<li>弱引用指向的对象会在没有任何引用指向他们的时候立即被垃圾收集。如果一个对象只有弱引用的话，那么这个对象是不可触及的。这些对象会在任何时候被垃圾收集并且会在下一个GC周期里被丢弃。</li>
<li>虚引用指向的是已经执行finalize方法，但是还没有回收内存的对象。</li>
</ul>


**二.四种引用类型的比较：**

<table>
<tr>
<td colspan="5" style="text-align:center;">Strong vs Soft vs Weak vs Phantom References
</td>
</tr>
<tr>
<td style="text-align:center">类型</td>
<td style="text-align:center">目的</td>
<td style="text-align:center">作用</td>
<td style="text-align:center">触发GC条件</td>
<td style="text-align:center">实现类</td>
</tr>
<tr>
<td>强引用</td>
<td>普通引用类型，只要对象的引用是强引用，他们就不会被垃圾收集</td>
<td>普通引用</td>
<td>任何对象如果不是强引用都可以被垃圾收集</td>
<td>默认类型</td>
</tr>
<tr>
<td>软引用</td>
<td>在内存足够的时候，对象不会被垃圾收集</td>
<td>为了保证即使对象没有任何引用指向它的时候也不会被垃圾收集，防止有引用再次指向这个对象</td>
<td>在第一次GC后，JVM需要回收更多的空间</td>
<td>java.lang.ref.SoftReference</td>
</tr>
<tr>
<td>弱引用</td>
<td>在对象可触及的状态下不会被垃圾收集</td>
<td>如果对象不再被引用会被自动回收</td>
<td>GC后对象只有弱引用</td>
<td>java.lang.ref.WeakReference
<br />
java.util.WeakHashMap</td>
</tr>
<tr>
<td>虚引用</td>
<td>让你可以清理已经执行finalize方法，但是还没有回收内存的对象</td>
<td>特殊清理</td>
<td>finalize方法执行之后</td>
<td>java.lang.ref.PhantomReference</td>
</tr>
</tbody>
</table>
