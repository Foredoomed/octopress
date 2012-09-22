---
layout: post
title: "Quora使用的技术"
date: 2011-07-16 15:43
comments: true
categories: architecture
---

如果提起[Quora](http://www.quora.com "Quora"),国人首先想到的可能会是它在国内的山寨版[知乎](http://www.zhihu.com "知乎")。想当初知乎还因为它的条款问题引来网络上的一片骂声，这个事件倒是给我提了个醒，因为我以前基本上不看网站的条款声明，现在看来以后要看明白了再用啊（特别是山寨货），不然怎么死的都不知道。当然我关注的不是山寨货，而是它的本源Quora。 <!--more-->

<a href="http://www.flickr.com/photos/60110479@N08/5943445558/" title="Flickr 上 Foredoomed 的 quora_stackoverflow_zhihu"><img src="http://farm7.static.flickr.com/6001/5943445558_12a1fc3061.jpg" width="500" height="375" alt="quora_stackoverflow_zhihu"/></a>

Quora由Facebook前CTO亚当·德安杰洛（Adam D'Angelo）以及Facebook工程技术经理查理·切沃（Charlie Cheever）于2008年创立。2010年初开始内部测试的Quora在3月获得风险投资公司Benchmark Capital的1400万美元投资。据《华尔街日报》报道，估值为8750万美元。但是Quora目前还是执行着邀请制度，这导致它的PV相对于同类型的社交问答网站[Stack Overflow](http://stackoverflow.com "Stack Overflow")要少了很多。但这并不能阻止Quora的火爆，其高质量的回答和优秀的用户体验都是它的优势所在。

作为我自己来讲，更多的还是关心Quora使用到的技术。它的性能为什么这么棒，它的架构是怎样的，它使用的是哪种语言和框架，它的搜索性能为什么这么好等。那就让我们来看看Quora到底用到了哪些技术。

**The Search-Box**

Quora只能搜索问题，主题标签，用户名，和主题标题。没有全文搜索，所以，你无法搜索问题和答案的内容。而搜索中使用前缀搜索方式，比如你输入mi，则Microsoft会马上出来。其搜索还会有一些非常简单的模糊匹配的算法。另外，如果有重复的问题，其中一个问题会自动跳转到另一个问题，但是在搜索中还是会出现。搜索中没有拼写检查。

一开始，他们使用的是一个开源的搜索服务器，叫<a href="http://sphinxsearch.com/">Sphinx</a>，其支持上述的那些功能。现在他们不用这个服务器了，因为<a href="http://www.quora.com/What-is-the-best-open-source-solution-for-implementing-fast-auto-complete">受到了一些限制</a>。他们做了一个新的解决方案，这个算法由Python实现。


> <a href="http://www.quora.com/What-libraries-does-Quora-use-for-search"><img width="16" height="16" style="padding-right: 10px;" src="http://www.quora.com/favicon.ico"/>What libraries does Quora use for search?</a><br />
<small>Adam D’Angelo, Quora Founder (Nov 13, 2010)</small><br />
Our search is custom-written. It doesn’t use any libraries aside from Thrift, and Python’s unicode library, which we use for unicode normalization.


**Speedy Queries**

Quora的查询是非常高速的，其查询请求是通过AJAX的GET请求发送的，结果返回用的是JSON数据格式，但他们解析JSON是在服务器端，而不是通过浏览器的javascript。这么做的原因可能是他们想高亮搜索关键词，似乎使用Client端的Javascript非常不好做。

Quora的即时搜索好像比较暴力，如果你输入Microsoft（一共9个字符），你会看到其会像后端发送9次查询——每按一个键一次，无论你敲这个单词的速底有多快，每输入一个字符都会发一个请求给后台。对于这样的看上去没有效率的对后台的请求，后台的服务器端会来控制相关的前台请求，所以，就算是前台这样做，也不会增加服务器端的负载，因为后台会做相关的处理。

Quora的搜索使用HTTP长连接，当你开始敲查询的时候，连接就建立了，这个连接会持续在那里，你下次搜索的时候会继续使用这个连接，除非你60秒没有动作了。

> <a href="http://www.quora.com/Quora-product/Is-Quora-going-to-implement-full-text-search"><img width="16" height="16" style="padding-right: 10px;" src="http://www.quora.com/favicon.ico"/>Is Quora going to implement full-text search?</a><br />
<small>Adam D’Angelo, I made a lot of the early Quora … (Sep 1, 2010)</small><br />
Yes, eventually. We haven’t implemented this yet because we’ve prioritized other things, but we will definitely do it in the future.

**Webnode2 And LiveNode**

Webnode2 和 LiveNode 是 Quora 内部的系统，其用来管理内容。Webnode2 生成 HTML, CSS 和 JavaScript 并且和 LiveNode 紧紧地耦合在一起，Webnode2主要是用来管理内容在网页上显示的，LiveNode主要是用来做动态网页内容更新的。Charlie Cheever 说，如果他可以从新开始，他<a href="http://www.quora.com/What-limitations-has-Quora-encountered-due-to-LiveNode-WebNode#answers">第一件事要做的就是重写整个LiveNode</a>。

Quora的工程师看上去对他们搞的这些东西非常的满意，并且<a href="http://www.quora.com/What-limitations-has-Quora-encountered-due-to-LiveNode-WebNode#answers">他们也在努力地找到这些东西的弱点</a>。有一个有意思的关于LiveNode的问题是，如果A和B同时正在看相当的一个问题，那么用户A的一些交互动作会影响B的页面。例如，如果A顶了一下某个答案，那么这个答案可能会往上移动。这样的一个显示变化会通过AJAX更新B的浏览器。如果B此时展开了评论，可能会受到影响。

<a href="http://www.quora.com/What-is-LiveNode-written-in">LiveNode 由这些东西写成：</a>Python, C++, 和 JavaScript。<a href="http://jquery.com/">jQuery</a> ，<a href="http://cython.org/">Cython</a>也用到了。
因为Quora<a href="http://www.quora.com/Is-Quora-planning-on-open-sourcing-LiveNode">想要对他们的LiveNode开源</a> 并准备把他们的代码分开，做这个事可能需要太多的工作和时间。
Charlie Cheever 指出 WebNode2 和<a href="http://www.quora.com/Quora-Infrastructure/What-is-webnode2">有一个叫做 “free and easy website builder” 的 Webnode 的 webnode.com</a> 没有任何的关系。

**Amazon Web Services**

Quora全部托管在Amazon的EC2和S3上，因为这样就不像在相当长的时间内自己运营服务器那样成本高昂，他们就是为像Quora这样快速增长的公司量身定做的。

**Ubuntu Linux<**

Quora使用Ubuntu Linux作为操作系统。没有什么大惊喜。在Amazon EC2上部署和管理都非常容易。Adam D’Angelo <a href="http://www.quora.com/Quora-Infrastructure/Which-Linux-flavor-does-Quora-use-Why">指出</a>他在高中时就使用Debian Linux，并且一直坚持到大学时还在用。原因就是<a href="http://www.quora.com/Quora-Infrastructure/Which-Linux-flavor-does-Quora-use-Why">“它工作的很好，而且没有令人信服的理由用别的操作系统去替代它”</a>。

**Static Content**

你只需要看一下Quora任何一个HTML的源代码，就可以看到他们在使用Amozon的分布式CDN, <a href="http://aws.amazon.com/cloudfront/">Cloudfront</a>。URLs在form里：
<pre lang="html">http://d2o7bfz2il9cb7.cloudfront.net/main-thumb-670336-25-7kmigSSkkdusoE6gHRkdQsXfjuTCaxQs.jpeg</pre>
CloudFront用来处理所有的静态页面,CSS和JavaScript (除了Google的 Analytics JavaScript,这是被Google托管)。<a href="http://www.quora.com/How-is-Quora-doing-image-uploads-to-Amazon-S3">图片被上传到EC2服务器,然后调整大小后上传到S3服务器</a>。这些都是用<a href="http://aws.amazon.com/code/134">Python S3 API</a>来管理的。

**HAProxy Load-Balancing**

Quora把<a href="http://haproxy.1wt.eu/">HAProxy</a>放在最前端，作为在它后面的分布式Nginx服务器的负载均衡服务器。

**Nginx**

反向代理服务器是Nginx，如果要了解更多这种设置方式，我推荐阅读<a href="http://kovyrin.net/2006/05/18/nginx-as-reverse-proxy/">Using Nginx As Reverse-Proxy Server On High-Loaded Sites</a>。

**Pylons And Paste**

<a href="http://pylonshq.com/">Pylons</a>是一个轻量级的Web框架，通常都是在Nginx之后作为主要的web服务器。他们使用默认的<a href="http://spacepants.org/blog/pylons-paste-stack">Pylon + Paste stack</a>方式。选择Pylons就像在万圣节选择南瓜一样。他们把Pylons中的template和ORM用他们自己的用Python写成的技术替换掉，这就是<a href="http://www.quora.com/What-languages-and-frameworks-were-used-to-code-Quora">LiveNode和WebNode2所在的地方</a>。
<a href="http://www.mochimedia.com/">MochiMedia</a>也是使用Pylons的启发中的一个，因为他们自己正在用它。

**Python**

从Facebook出来的Charlie 和 Adam选用了Python而不是PHP。正如Adam指出的 “<a href="http://www.quora.com/Why-did-Quora-choose-Python-for-its-development">Facebook is stuck on that for legacy reasons, not because it is the best choice right now</a>”（Facebook使用PHP并不是因为其好，而是因为历史原因的问题），当然他们也不会使用C#，因为那样一来就会引入一堆微软的东西。当然，也不会是Java，因为Python要比Java更容易写出代码，Scala太年轻了，还需要考验。Ruby看上来很像Python，但是他们对Ruby没有过多的经验。最终还是Python胜出。当然，他们知道Python的弱点是性能和速度，所以，他们在需要速度和性能的地方使用了C/C++。 他们使用Python的版本是2.6。

使用Python的另一个原因是Python的数据结构和JSON可以很好的映射起来。代码易读性很高。而且有很多的库，调试器和重载器。Quora的B/S结构几乎完全通过JSON进行数据交互。

他们<a href="http://www.quora.com/Adam-DAngelo/What-version-of-Python-are-you-programming-in-and-what-IDE-do-you-use">没有使用IDE</a>，他们使用得最多的是Emacs，一看就知道这是一个个人的选择，随着他们开发团队的扩大，这种情况会得到改变的。

另外，他们提到了<a href="http://pypy.org/">PyPy</a>，一个让Python更快更灵活的项目。

**Thrift**

<a href="http://incubator.apache.org/thrift/">Thrift</a> 用于后端服务器间的通讯。Thrift &nbsp;服务由 C++开发。<a href="http://coolshell.cn/articles/4549.html">Facebook同样使用了这个技术</a>。

> <a href="http://www.quora.com/Why-would-you-write-a-Thrift-service-in-C"><img width="16" height="16" style="padding-right: 10px;" src="http://www.quora.com/favicon.ico"/>Why would you write a Thrift service in C++?</a><br />
<small>Adam D’Angelo, I’ve written a lot of Python, in… (Sep 4, 2010)</small><br />
Mainly if you want to keep data in memory between requests, and want to keep your Python code stateless. Writing a Python wrapper around a C library involves some memory management with reference counting that requires some understanding of the Python internals, but writing a thrift interface is simple. You also isolate failures this way &ndash; if the service goes down it won’t take the Python code down with it.


**Tornado**

<a href="http://www.tornadoweb.org/">Tornado</a> web 框架用于实时更新，其运行在Comet服务器上，其用来处理大量的需要长时间poll和push更新的网络连接。

**Long Polling (Comet)**

Quora的网页并不是简单的显示，每一个页面都需要更新，或是创建问题，答案和评论。所以，他们使用了Long Polling而不是传统的Polling，传统的Polling需要浏览器一端不停地重复地向服务器询问——“有更新吗？”，服务器说没有，于是过一会浏览大再问，现在呢？服务器说，还是没有，浏览器过一会又问，现在呢？服务器说，还没好。这样一来，就好像让我们的客户端放到了驾驶室里，这显然是有问题的，因为只有服务器知道什么时候会有更新。而且浏览器这么干，很快会让服务器的负载加上去。

Long polling也就是我们熟知的<a href="http://en.wikipedia.org/wiki/Comet_(programming)">Comet</a>，其让服务器来控制这些事，让客服端等在那里听服务器的响应。在client和server的会话对于两者是是相同的，而不是client需要等着然后向服务器查询。服务器端可以把一个连接打开很长时间（比如：60秒），在这段时间里，服务器会查看是否有相应的东西需要更新，如果有的话，就发给浏览器。如果没有的话，就等下一次的client询问。可见，这种服务器等一会再响应的方法可以让浏览器少发几次查询。

对于long-polling的最好的地方是，可以降低浏览器和客户端间来来回回的次数。让服务器端来控制时间，所以，内容更新可能会只是几个毫秒，或是几十秒。 服务器端也可以积攒一堆更新后，一次发给浏览器。这样做会更有效率。

但是，这个方法的黑暗面是——这会让服务器端出现大量的TCP链接，想一想，Quora也是百万级用户的应用了，只需要10%的在线用户，你就需要一个可以处理10万并发量的架构。注意，如果一个用户在其浏览器里打开了多个Quora网页的话，那么，这个链接器会是非常致命的。

当然，好的消息是已经有一些技术专门为Long Polling设计，这些技术可以让你在那些等待的连接中只会消耗非常非常少的内存（因为那些等待连接并不需要所有的资源）。例如：Nginx是一个单线程的事件驱动的小型服务器，每一个链接只花非常小的内存。每一个Nginx的进程只会在一个时候处理一个连接。这意味着其很容易扩展成一个可以处理成千上的并发量的服务架构。

> <a onclick="javascript:_gaq.push(['_trackEvent','outbound-article','www.quora.com/How-do-you-push-messages-back-to-a-web-browser-client-through-AJAX-Is-there-any-way-to-do-this-without-having-the-client-constantly-polling-the-server-for-updates']);" href="http://www.quora.com/How-do-you-push-messages-back-to-a-web-browser-client-through-AJAX-Is-there-any-way-to-do-this-without-having-the-client-constantly-polling-the-server-for-updates"><img width="16" height="16" style="padding-right: 10px;" src="http://www.quora.com/favicon.ico"/>How do you push messages back to a web-browser client through AJAX?  Is there any way to do this without having the client constantly polling the server for updates?</a><br />
<small>Adam D’Angelo, Quora (Sep 29, 2010)</small><br />
There is no reliable way to do this without having the client polling the server. However, you can make the server stall its responses (50 seconds is a safe bet) and then complete them when a message is ready for the client. This is called “long polling” and it’s how Quora, Gmail, Meebo, etc all handle the problem.
If you have a specialized server that uses epoll or kqueue, you should be able to hold on the order of 100k users per server (depending on how many messages are going). This is called the “c10k” problem. http://www.kegel.com/c10k.html

**MySQL**

就像Adam D’Angelo的老东家Facebook一样，Quora同样重度使用MySQL。在回答Quora的问题“<a href="http://www.quora.com/When-Adam-DAngelo-says-partition-your-data-at-the-application-level-what-exactly-does-he-mean">When Adam D’Angelo says “partition your data at the application level”, what exactly does he mean?</a>“, D’Angelo详细描述了怎样使用MySQL(或者一般的关系型数据库）来做分布式数据存储。

基本建议是，如果需要的话把数据库里的数据分区。尽可能的把数据放在一台机器上，使用hash主键对多数据库中的大规模的数据进行分区。必须避免使用表连接（join）。Adam参考了FriendFeed的一篇文章<a href="http://bret.appspot.com/entry/how-friendfeed-uses-mysql">How FriendFeed uses MySQL to store schema-less data</a>，他<a href="http://www.quora.com/NoSQL/In-what-parts-of-a-social-site-with-concert-listings-should-one-use-a-NoSQL-DB-versus-a-SQL-DB">还说</a>在还没有100万用户之前，不要使用NoSQL。

并不只是Quora和FriendFeed重度使用MySQL，Google也在一些与搜索无关的应用上使用MySQL。Google已经为MySQL的复制，同步，监控和更快的速度提升发布了<a href="http://code.google.com/p/google-mysql-tools/wiki/Mysql4Patches">补丁</a>。

> <a onclick="javascript:_gaq.push(['_trackEvent','outbound-article','www.quora.com/How-does-one-evaluate-if-a-database-is-efficient-enough-to-not-crash-as-its-put-under-increasing-load']);" href="http://www.quora.com/How-does-one-evaluate-if-a-database-is-efficient-enough-to-not-crash-as-its-put-under-increasing-load"><img width="16" height="16" style="padding-right: 10px;" src="http://www.quora.com/favicon.ico"/>How does one evaluate if a database is efficient enough to not crash as it’s put under increasing load?</a><br />
<small>Adam D’Angelo, Quora (Oct 10, 2010)</small><br />
One option is to simulate some load. Write a script that mimics the kinds of queries your application will be doing, and make sure it can handle the amount of load you want it to be ready for (especially as the size of the dataset changes).

**Memcached**

<a href="http://memcached.org/">Memcached</a>作为MySQL的前端缓存。

**Git**

用<a href="http://git-scm.com/">Git</a>作为<a href="http://www.quora.com/What-languages-and-frameworks-were-used-to-code-Quora">版本控制工具</a>。

**JavaScript Placement**

如果你看一下Quora的网页源码，你会看到其JavaScript总是在页面的最后。 Charlie Cheever<a href="http://www.quora.com/Why-is-the-Quora-website-so-fast">建议</a>这样做会让页面加载变快，因为会先渲染网页内容，然后再加载JavaScript。

**Charlie Cheever Follows “14 Rules for Faster-Loading Web Sites”**

Steve Souders,《High Performance Web Sites》和《Even Faster Web Sites》的作者，列出了<a href="http://stevesouders.com/hpws/rules.php">让你网页更快的原则</a>。Charlie Cheever提到过这个列表，这是Quora的速度快的原因之一。

**Steve Souders的14条规则是：**
<ul>
<li>尽可能少的HTTP请求</li>
<li>使用CDN</li>
<li>添加过期头</li>
<li>用Gzip压缩组件</li>
<li>把CSS放在页面的顶部</li>
<li>把JavaScript放在页面的底部</li>
<li>避免CSS表达式</li>
<li>JavaScript,CSS和HTML分离</li>
<li>减少DNS查询</li>
<li>最小化JavaScript</li>
<li>避免重定向</li>
<li>减少重复的脚本</li>
<li>定义ETags</li>
<li>使AJAX可缓存化</li>
</ul>

参考资料

[1] [quoras technology examined](http://www.philwhln.com/quoras-technology-examined "quoras technology examined")  
[2] [http://coolshell.cn/articles/4936.html](http://coolshell.cn/articles/4936.html "http:coolshell.cn/articles/4936.html")

