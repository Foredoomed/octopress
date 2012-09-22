---
layout: post
title: "NoSQL数据库比较"
date: 2011-06-03 13:59
comments: true
categories: nosql
---
随着Web2.0时代的来临以及宽带技术的迅猛发展，许多Web应用都感受到了来自高并发以及高数据量的挑战。而大多数情况下这种挑战来自于数据库，即二维关系型数据库的缺陷导致了RDBMS成为了高并发和高数据量下的瓶颈。正是为了消除这种瓶颈，才诞生了NoSQL。NoSQL是当下最热的技术话题，而且越来越多的公司开始选择NoSQL产品来提高Web应用的性能及伸缩性，我们完全可以想象在不远的将来NoSQL取代现在的关系型数据库，成为数据存储的第一选择。 <!--more-->

**一.NoSQL数据库的分类**

NoSQL按存储方式主要分为四类：

1.Key-values Stores

主要的想法是用hash表，里面包括一个unique key和指向特定数据的指针。Key-value 模型是最简单和最容易实现的. 但是它对查询和更新部分数据时效率不高。

<table>
<tr>
<td>Examples</td>
<td>Tokyo Cabinet/Tyrant, Redis, Voldemort, Oracle BDB</td>
</tr>
<tr>
<td>典型应用场景</td>
<td>内容缓存，主要用于处理大量数据的高访问负载，也用于一些日志系统等等。</td>
</tr>
<tr>
<td>数据模型</td>
<td>Key 指向 Value 的键值对，通常用hash table来实现</td>
</tr>
<tr>
<td>强项</td>
<td>查找速度快</td>
</tr>
<tr>
<td>弱项</td>
<td>数据无结构化，通常只被当作字符串或者二进制数据</td>
</tr>
</table>

<br/>
2.Document Databases

这个模型是包含key-value集合的文档，并且这个文档被标记过版本的。这些半结构文档是以类似于JSON的形式存储。Document databases基本上是更高级的Key-value形式，允许每个key的value可以多级嵌套。Document databases 支持更有效的查询。

<table>
<tr>
<td>Examples</td>
<td>CouchDB, MongoDB</td>
</tr>
<tr>
<td>典型应用场景</td>
<td>Web应用（与Key-Value类似，Value是结构化的，不同的是数据库能够了解Value的内容）</td>
</tr>
<tr>
<td>数据模型</td>
<td>Key-Value对应的键值对，Value为结构化数据</td>
</tr>
<tr>
<td>强项</td>
<td>数据结构要求不严格，表结构可变，不需要像关系型数据库一样需要预先定义表结构</td>
</tr>
<tr>
<td>弱项</td>
<td>查询性能不高，而且缺乏统一的查询语法。</td>
</tr>
</table>

<br/>
3.Column Family Stores

这类NoSQL主要是用来存储和处理大容量分布式数据。仍然有key，但指向的是多列，并且列是由column family来组织。

<table>
<tr>
<td>Examples</td>
<td>Cassandra, HBase</td>
</tr>
<tr>
<td>典型应用场景</td>
<td>分布式的文件系统</td>
</tr>
<tr>
<td>数据模型</td>
<td>以列簇式存储，将同一列数据存在一起</td>
</tr>
<tr>
<td>强项</td>
<td>查找速度快，可扩展性强，更容易进行分布式扩展</td>
</tr>
<tr>
<td>弱项</td>
<td>功能相对局限</td>
</tr>
</table>

<br/>
4.Graph Databases

这类NoSQL的结构不是一般的二维表结构，而是更有弹性的图形模型。并且需要指定数据模型来查询这类数据库。

<table>
<tr>
<td>Examples</td>
<td>Neo4J, InfoGrid, Infinite Graph</td>
</tr>
<tr>
<td>典型应用场景</td>
<td>社交网络，推荐系统等。专注于构建关系图谱</td>
</tr>
<tr>
<td>数据模型</td>
<td>图结构</td>
</tr>
<tr>
<td>强项</td>
<td>利用图结构相关算法。比如最短路径寻址，N度关系查找等</td>
</tr>
<tr>
<td>弱项</td>
<td>很多时候需要对整个图做计算才能得出需要的信息，而且这种结构不太好做分布式的集群方案。</td>
</tr>
</table>

<br/>
**二.RDBMS和NoSQL的选择:**

NoSQL: 
 
* 实时状态或分析
* 日志或归档
* 社交计算
* 外部数据Feed集成
* 前端顺序处理系统
* 企业内容管理服务
* 处理非常高负载的存储能力
* 对于持久化数据有非常多的写操作
* 希望具有水平扩展的能力
* 如果你需要动态查询
* 简单的查询语句(没有“join”)

RDBMS:

* 希望有高负责情况下的存储能力，但是主要的是读操作
* 比起优雅的数据结构，你更看重性能
* 你需要强大的SQL查询语言


**三.MongoDB和Redis的比较:**

1.MongoDB

MongoDB的特性：  

* 用C++实现的
* 保留了一些SQL的友好的特性（查询，索引）
* 在AGPL许可下发布
* 自定义的二进制协议（BSON）
* Master/slave形式的数据同步
* 查询语句是javascript表达式
* 在服务器端执行任意的javascript函数
* 比起CouchDB，有更好的即时更新（update-in-place）性能
* 内建分片
* 用内存映射存储数据
* 追求性能而不是特性
* 一旦崩溃，它需要去修复表
* 在1.8版本中将有更好的持久性

MongoDB的主要事项:

* 对于分开的读和写，MongoDB会用一个该死的全局锁。
* MongoDB没有一个统计计划优化程序。
* 在分片的时候才需要Mongos进程。
* MongoDB支持Master-Slave和Replica-Sets的数据同步模式。
* 在第二次复制时，MongoDB支持“slave-delay”
* MongoDB不应该在32位环境下运行。
* MongoDB会自动记录任何超过100ms的查询语句。
* 如果一个查询语句要消耗较长的一段时间，MongoDB可以用来做性能提高工具。
* MongoDB不支持multi-master数据同步。他们认为这样保持逻辑简单很好。而且，系统因为不用担心在多master的情况下发生写冲突而简单很多。
* 在复制安装情况下，MongoDB支持一个叫做Arbiter，它的工作就是把结打开。

2.Redis

Redis的特性：

* 用C++实现的
* 主要特点：Blazing fast
* 在BSD许可下发布
* 协议：Telnet-like
* 以硬盘为后备的内存数据库
* 从2.0后,可以交换到磁盘上
* Master/slave形式的数据同步
* 简单的keys和values
* 复杂的操作，像ZREVRANGEBYSCORE
* 增长和压缩（对比率和统计有好处）
* Sets(also union/diff/inter)
* lists(also a queue; blocking pop)
* hashes(多属性的对象)
* 在所有这些数据之中，只有Redis支持事务
* 数据可以设置为过期（因为在cache中）
* Sorted sets（高分数表，对处理查询有好处）
* 发布/订阅和看管数据变化

最佳应用: 对于在可预见的数据库容量下快速改变的数据（绝大部分在内存）。例如：股价，分析，实时数据收集，实时交流。 

参考文章：

[1][picking-the-right-nosql-database-tool](http://blog.monitis.com/index.php/2011/05/22/picking-the-right-nosql-database-tool "picking-the-right-nosql-database-tool")  
[2][cassandra-vs-mongodb-vs-couchdb-vs-redis](http://kkovacs.eu/cassandra-vs-mongodb-vs-couchdb-vs-redis "cassandra-vs-mongodb-vs-couchdb-vs-redis")  
[3][如何选择最适合你的NoSQL数据库](http://blog.nosqlfan.com/html/1727.html "如何选择最适合你的NoSQL数据库")

