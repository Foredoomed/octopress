---
layout: post
title: "Facebook的系统架构"
date: 2011-05-10 22:46
comments: true
categories: architecture
---
原文地址：[What is Facebooks architecture](http://www.quora.com/What-is-Facebooks-architecture "What is Facebooks architecture") 

* Web前端用PHP写的。然后，Facebook的HipHop[1]把它转换成C++,用g++编译, 这样做可以提供高性能模板和Web逻辑执行层。
* 业务逻辑用Thrift[2]作为服务暴露处理。这其中的一些服务是用PHP实现的，C++或Java要看服务的需求（也可能使用其他一些语言...)。
* 用Java实现的服务不使用任何一个普通的企业级应用服务器，而是用Facebook为自己定制开发的应用服务器。一开始这样做是被看作重复造轮子，但是当这些服务只（或绝大多数）使用Thrift暴露和消费，Tomcat甚至Jetty的开销可能会非常大，而且还没有加入他们需要的重要的作用。
* 持久化使用MySQL, Memcached[3], Facebook's Cassandra[4], Hadoop's HBase[5]。Memcached被用作MySQL的缓存和一般目的的缓存。Facebook的工程师们承认他们目前对于Cassandra的使用正在减少，他们更喜欢使用HBase,因为HBase有简单的一致性模型和MapReduce功能。
* 线下处理使用Hadoop和Hive处理。
* 像日志，点击和feeds等数据用Scribe[6]处理，这些数据被聚集和储存在Hadoop分布式文件系统Scribe-HDFS[7]中, 这样可以让MapReduce做扩展分析。
* BigPipe[8]是他们定制的技术，这个技术使用流水线逻辑可以加速网页渲染速度。
* Varnish Cache[9]被用作HTTP代理。他们更喜欢使用Varnish，因为它的高性能表现[10]。
* 使用Haystack处理数10亿的照片的存储，Facebook自己开发的点对点存储解决方案带来的是低等级的优化和只能增加的写操作[11]。
* Facebook Messages使用的是他们自己的架构，这个架构是构建在基础设施分区和动态集群管理。业务逻辑和持久化被包装在所谓的'单元'（cell）里。每个单元处理一部分用户；随着用户的增长新的单元能够被增加[12]。持久化数据用HBase归档[13]。
* Facebook Messages的搜索引擎是用存储在HBase里倒转的索引构建的[14]。
* Facebook搜索引擎的实现细节还不清楚，自动完成搜索使用一个自定义的存储和检索逻辑[15]。
* 聊天功能是在用Erlang开发的Epoll服务器上，而且可以被Thrift访问[16]。

关于为这些组件中的每个提供的资源，一些信息和数据:

* Facebook被估计拥有超过60,000台服务器[17]。他们最近新的在普赖恩维尔和俄勒冈州的数据中心的硬件都是自己他们自己设计的[18]，而且最近作为Open Compute Project[19]公诸于众。
* 300TB的数据被存储在memcached processes[20]里。
* Hadoop和Hive集群是用3000台8核，32G内存，12TB硬盘的服务器里[20]。
* 每天1亿的点击量，500亿的照片，缓存着3兆的对象，到2010年7月为止，每天有130TB的日志[21]。

[1] *HipHop for PHP*: [http://developers.facebook.com/b...](http://developers.facebook.com/blog/post/358 "http://developers.facebook.com/b...")  
[2] *Thrift*: [http://thrift.apache.org](http://thrift.apache.org/ "http://thrift.apache.org")  
[3] *Memcached*: [http://memcached.org](http://memcached.org/ "http://memcached.org")  
[4] *Cassandra*: [http://cassandra.apache.org](http://cassandra.apache.org/ "http://cassandra.apache.org")  
[5] *HBase*: [http://hbase.apache.org](http://hbase.apache.org/ "http://hbase.apache.org")  
[6] *Scribe*: [https://github.com/facebook/scribe](https://github.com/facebook/scribe "https://github.com/facebook/scribe")  
[7] *Scribe-HDFS*: [http://hadoopblog.blogspot.com/2...](http://hadoopblog.blogspot.com/2009/06/hdfs-scribe-integration.html "http://hadoopblog.blogspot.com/2...")  
[8] *BigPipe*: [http://www.facebook.com/notes/fa...](http://www.facebook.com/notes/facebook-engineering/bigpipe-pipelining-web-pages-for-high-performance/389414033919 "http://www.facebook.com/notes/fa...")  
[9] *Varnish Cache*: [http://www.varnish-cache.org](http://www.varnish-cache.org/ "http://www.varnish-cache.org")  
[10] *Facebook goes for Varnish*: [http://www.varnish-software.com/...](http://www.varnish-software.com/customers/facebook "http://www.varnish-software.com/...")  
[11] *Needle in a haystack*: efficient storage of billions of photos: [http://www.facebook.com/note.php...](http://www.facebook.com/note.php?note_id=76191543919 "http://www.facebook.com/note.php...")  
[12] *Scaling the Messages Application Back End*: [http://www.facebook.com/note.php...](http://www.facebook.com/note.php?note_id=10150148835363920 "http://www.facebook.com/note.php...")  
[13] *The Underlying Technology of Messages*: [https://www.facebook.com/note.ph...](https://www.facebook.com/note.php?note_id=454991608919 "https://www.facebook.com/note.ph...")  
[14] *The Underlying Technology of Messages Tech Talk*: [http://www.facebook.com/video/vi...](http://www.facebook.com/video/video.php?v=690851516105 "http://www.facebook.com/video/vi...")  
[15] *Facebook's typeahead search architecture*: [http://www.facebook.com/video/vi...](http://www.facebook.com/video/video.php?v=432864835468 "http://www.facebook.com/video/vi...")  
[16] *Facebook Chat*: [http://www.facebook.com/note.php...](http://www.facebook.com/note.php?note_id=14218138919 "http://www.facebook.com/note.php...")  
[17] *Who has the most Web Servers?*: [http://www.datacenterknowledge.c...](http://www.datacenterknowledge.com/archives/2009/05/14/whos-got-the-most-web-servers/ "http://www.datacenterknowledge.c...")  
[18] *Building Efficient Data Centers with the Open Compute Project*: [http://www.facebook.com/note.php...](http://www.facebook.com/note.php?note_id=10150144039563920 "http://www.facebook.com/note.php...")  
[19] *Open Compute Project*: [http://opencompute.org](http://opencompute.org/ "http://opencompute.org")  
[20] *Facebook's architecture presentation at Devoxx 2010*: [http://www.devoxx.com](http://www.devoxx.com "http://www.devoxx.com")  
[21] *Scaling Facebook to 500 millions users and beyond*: [http://www.facebook.com/note.php...](http://www.facebook.com/note.php?note_id=409881258919 "http://www.facebook.com/note.php...")  

