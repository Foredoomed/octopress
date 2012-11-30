---
layout: post
title: "JVM内存管理和垃圾收集"
date: 2011-04-03 18:52
comments: true
categories: java
---
**0.Java基本类型和类的大小**
                                                                       
Java手册上的类型大小

byte : 8-bit                             
short : 16-bit                             
Int : 32-bit                                
long : 64-bit                                 
char : 16-bit unsigned integer           
float : 32-bit                               
double: 64-bit                                
boolean: 1-bit                             
                                                  
实际存储大小(根据JVM实现而定)

Byte : 16 bytes  
Short : 16 bytes  
Integer : 16 bytes  
Long : 16 bytes  
Character : 16 bytes  
Float : 16 bytes  
Double : 16 bytes  
Boolean : 16 bytes  
Object ：8 bytes  

<!--more-->

**1.JVM的堆结构**

{% img http://farm6.static.flickr.com/5136/5504939947_19ed9a8dbc.jpg  "JVMHeapStructure" "JVMHeapStructure"%}

运行时数据区域，所有类实例和数组的内存均从此处分配，由Java 虚拟机启动时创建。对象的堆内存由称为垃圾回收器的自动内存管理系统回收。

堆由两部分组成:

Eden + From Space(S0) + To Space(S1): 也叫做Young Generation(年轻代)

Tenured Space: 也叫做Old Generation(老年代)

Survivor Space: 包括S0和S1

permanent space(方法区)： JVM具有一个由所有线程共享的方法区。它存储每个类结构，如运行时常数池、字段和方法数据，以及方法和构造方法的代码，它是在Java虚拟机启动时创建的，**不包括在JVM堆内**，默认为4M。 

这个区域主要存放：

类的信息

* 包名
* 父类的包名
* 类或接口
* 类型修饰符
* 父接口包名
 
其它信息

* 类型的常量池
* 属性 
* 方法
* 类里的静态变量(除了常量)
* 一个指向`ClassLoader`的引用 
* 一个指向`Class`类的引用


**2.GC的工作流程**

(0) 绝大多数情况下对象初始化被分配在`Eden`中(一个非常大的对象会被分配在老年代中)。  
(1) 如果`Eden`空间占满了， 会触发 minor GC。 Minor GC 后仍然存活的对象会被复制到`S0`中去。这样`Eden`就被清空可以分配给新的对象。  
(2) 又触发了一次 Minor GC ， `S0`和`Eden`中存活的对象被复制到`S1`中， 并且`S0`和`Eden`被清空。 在同一时刻, 只有`Eden`和一个`Survivor Space`同时被操作。  
(3) 当每次对象从`Eden`复制到`Survivor Space`或者从`Survivor Space`中的一个复制到另外一个，有一个计数器会自动增加值。 默认情况下如果复制发生超过16次， JVM 会停止复制并把他们移到老年代中去。  
(4) 如果一个对象不能在`Eden`中被创建，它会直接被创建在老年代中。 如果老年代的空间被占满会触发老年代的 GC，也被称为 Full GC。full GC 是一个压缩处理过程，所以它比`Minor GC`要慢很多。  

**3.GC算法**

Serial Collector

大部分平台或者强制`java -client`默认会使用这种。
Young Generation  = Serial
Old Generation  = Serial (Mark-Sweep-Compact)
这种方法的缺点很明显，Stop-the-world, 速度慢。服务器应用不推荐使用。

Parallel Collector

在Linux x64上默认是这种算法，其他平台要加`java -server`参数。
Young Generation = parallel，多个thread同时copy
Old Generation = Mark-Sweep-Compact = 1
优点：新生代回收更快。因为系统大部分时间做的GC都是新生代的，这样提高了Throughput(CPU用于非GC时间)。
缺点：当运行在8G/16G Server 上 Old Generation 存活对象太多的时候暂停时间(Pause Time)过长。

Parallel Compacting Collector (ParallelOld)

Young Generation = parallel 
Old Generation = parallel
优点：Old Generation 上性能较 Parallel Collector 方式有提高。
缺点：大部分Server系统Old Generation内存占用会达到60%-80%, Compact方面开销比起Parallel Collector并没明显减少。

Concurent Mark-Sweep(CMS) Collector (low-latency collector)

Young Generation = parallel 
Old Generation = CMS
同时不做 compact 操作。
优点：Pause Time会降低, Pause Time敏感但CPU有空闲的场景需要建议使用策略4。
缺点：CPU占用过多，CPU密集型服务器不适合。需要更大的堆空间，会造成很多碎片。


| GC算法 | 年代区 | 特点 |
|----
| Serial | Young | Stop The World, Copying Collector, Single GC thread. |
| Serial Old (MSC) | Old | Stop the World, Mark Sweep Compact (MSC), Single GC thread |
| Parallel Scavenge | Young | Stop the World, copying collector, multiple GC threads. Provides higher throughput by executing GC tasks in parallel with each other (but not the app). Cannot run during concurrent phases of the CMS. | 
| Parallel New | Young | As Parallel Scavenge, but can run during the concurrent phases of the CMS |
| Parallel Old/ Parallel Compacting | Old | Similar to Parallel Scavenge, but operates on the old generation. Uses multiple GC threads to speed up the work of Serial Old (MSC). STW collector, but higher throughput for old generation collections. |
| Concurrent Mark-Sweep (CMS) | Old | Breaks up its work into phases, and executes most of its phases concurrently with the application thread – resulting in low latency. However, it introduces substantial management overhead and results in a fragmented heap.|
{: class="datalist" }

**垃圾收集器的选择**

| JVM参数 | 年轻代 | 老年代 |
|----
| UseSerialGC | Serial | Serial Old (MSC) |
| UseParNewGC | ParNew | Serial Old (MSC) |
| UseConcMarkSweepGC | ParNew | CMS (mostly used)， Serial Old (used when concurrent mode failure occurs) | 
| +UseParallelGC | Parallel Scavenge | Serial Old |
| UseParallelOldGC | Parallel Scavenge | Parallel Old |
| +UseConcMarkSweepGC/-UseParNewGC | Serial | CMS/Serial Old |
{: class="datalist" }



**4.JVM的默认设置**

堆（heap）即（New Generation 和Old Generaion 之和）的设置

* 初始分配的内存由`-Xms`指定，默认是物理内存的1/64但小于1G。

* 最大分配的内存由`-Xmx`指定，默认是物理内存的1/4但小于1G。

* 默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制，可以由`-XX:MinHeapFreeRatio`指定。

* 默认空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制，可以由`-XX:MaxHeapFreeRatio`指定。

* 服务器一般设置`-Xms`、`-Xmx`相等以避免在每次GC后调整堆的大小，所以上面的两个参数没啥用。 

* `-Xmn`设置Young Generation的heap大小

* `-XX:MinHeapFreeRatio`与`-XX:MaxHeapFreeRatio`设定空闲内存占总内存的比例范围，这两个参数会影响GC的频率和单次GC的耗时。`-XX:NewRatio`决定Young与Old Generation的比例。Young generation空间越大，Minor GC频率越低，但是Old Generation空间小了，又可能导致Major GC频率增加。`-XX:NewSize`和`-XX:MaxNewSize`直接指定了Young Generation的缺省大小和最大大小。

非堆内存的设置

默认分配为64M

`-XX:PermSize`设置最小分配空间，`-XX:MaxPermSize`设置最大分配空间。一般把这两个数值设为相同，以减少申请内存空间的时间。

**5.GC调优**

参考：[GC调优例子](http://java.sun.com/docs/hotspot/gc1.4.2/example.html "GC调优例子")

*设置Xms=Xmx=3/4物理内存(或50%-70%)，-Xmn为1/4的-Xmx值，新生代必须小于堆的一半大
*如果是CPU密集型服务器，使用–XX:+UseParallelOldGC, 否则–XX:+UseConcMarkSweepGC
*新生代,Parallel/ParallelOld可设大于Xmx1/4，CMS可设小，小于Xmx1/4
*经验之谈：通常情况下，JVM堆的大小应为物理内存的80%
*使用-verbose:gc和-XX:+PrintGCDetails观察GC性能

**6.Dump heap**

Linux下：`jmap -dump:file=xxx.hprof pid` (其中`pid`通过`ps -aux`命令查看)

命令行:`jstat -gcutil pid p1 p2` (其中`p1`表示每多少毫秒打印一次；`p2`表示一共打印几次)

输出的参数含义：

* S0：Heap上的 Survivor space 0 段已使用空间的百分比
* S1：Heap上的 Survivor space 1 段已使用空间的百分比
* E： Heap上的 Eden space 段已使用空间的百分比
* O： Heap上的 Old space 段已使用空间的百分比
* P： Perm space 已使用空间的百分比
* YGC：从程序启动到采样时发生Young GC的次数
* YGCT：Young GC所用的时间(单位秒)
* FGC：从程序启动到采样时发生Full GC的次数
* FGCT：Full GC所用的时间(单位秒)
* GCT：用于垃圾回收的总时间(单位秒)

jstat命令其他参数含义：
      
* jstat -class pid:显示加载class的数量，及所占空间等信息。 
* jstat -compiler pid:显示VM实时编译的数量等信息。 
* jstat -gc pid:可以显示gc的信息，查看gc的次数，及时间。其中最后五项，分别是young gc的次数，young gc的时间，full gc的次数，full  gc的时间，gc的总时间。 
* jstat -gccapacity:可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小，如：PGCMN显示的是最小perm的内存使用量，PGCMX显示的是perm的内存最大使用量，PGC是当前新生成的perm内存占用量，PC是但前perm内存占用量。其他的可以根据这个类推， OC是old内纯的占用量。 
* jstat -gcnew pid:new对象的信息。 
* jstat -gcnewcapacity pid:new对象的信息及其占用量。 
* jstat -gcold pid:old对象的信息。 
* jstat -gcoldcapacity pid:old对象的信息及其占用量。 
* jstat -gcpermcapacity pid: perm对象的信息及其占用量。 
* jstat -util pid:统计gc信息统计。 
* jstat -printcompilation pid:当前VM执行的信息。 

GC日志

-Xloggc:d:\gc.log
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps – add time stamp 

**7.内存泄漏-java.lang.OutOfMemoryError**

Java heap space

* Configuration issue: -Xmx
* Memory Leak: The excessive use of finalizers
* PermGen space: Too many classes`-XX:MaxPermSize`
* Requested array size exceeds VM limit: Need a so big array?
* Request <size> bytes for <reason>: Out of swap space?
* Native Memory Leak
* <reason> <stack trace> (Native method)
* Native Memory Allocation Issue 

Perm Memory Leak

Too Many Interned String

* String.intern()
* Constant String will be interned implicitly
* No Enough Info provided by Heap Dump on Interned String
* If Perm Memory increased dynamically, be careful

Too Many Classes or Class Load Leak

* Enlarge the perm generation
* Avoid duplicated class loader

Out of Swap Space

* Enlarge the swap space
* Systems with 4GB of ram or less require a minimum of 2GB of swap space
* Systems with 4GB to 16GB of ram require a minimum of 4GB of swap space
* For Unix Family OS, use pmdump or pmap, libumem for Solaris

