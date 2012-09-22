---
layout: post
title: "Stack Overflow的系统架构"
date: 2011-07-14 15:31
comments: true
categories: architecture
---
[Stack Overflow](http://stackoverflow.com "Stack Overflow")是我最喜欢的网站之一，无论是他的问题和回答的质量，内容相关度还是用户体验都是非常优秀的。就我自己而言，如果在编程中遇到问题，我会第一时间上去搜索。如果没有搜到相关的问题，我就会自己提问让老外来回答。一般来说，只要是主流技术，或者不是太小众的，都会很快得到回答。最主要的是，老外会非常认真详细地回答你的问题，由于国内没有与之完全对应的网站，就这点来说可以把国内具有类似功能的网站（例如CSDN，ItEye等）暴出屎来。<!--more-->

今天无意中看到一篇介绍Stack Overflow架构的文章后觉得非常有价值，所以决定翻译成中文，供以后参考。

<a href="http://www.flickr.com/photos/60110479@N08/5940332610/" title="Flickr 上 Foredoomed 的 stackoverflow"><img src="http://farm7.static.flickr.com/6128/5940332610_b983b2c9a6.jpg" width="368" height="104" alt="stackoverflow"/></a>

Stack Overflow目前已经有超过1600万用户，并且每月有将近9500万PV。Stack Overflow已经发展并扩大成为了[Stack Exchange Network](http://stackexchange.com "Stack Exchange Network")。Stack Exchange Network现在包括**Stack Overflow**, **Server Fault**, 和 **Super User**，旗下拥有43个网站,并且还在以成倍的速度增长。

Stack Overflow不变的是对他们做的事保有开放的态度，这也是写这篇的原因。最近有一系列关于其成长的文章：
<ul>
    <li><a href="http://blog.serverfault.com/post/stack-exchanges-architecture-in-bullet-points/">Stack Exchange’s Architecture in Bullet Points</a>
    </li>
    <li><a href="http://blog.serverfault.com/post/1432571770/">Stack Overflow’s New York Data Center</a></li>
    <li><a href="http://blog.serverfault.com/post/1097492931/">Designing For Scalability of Management and Fault Tolerance</a></li>
    <li><a href="http://blog.stackoverflow.com/2011/01/stack-overflow-search-now-81-less-crappy/">Stack Overflow Search &mdash; Now 81% Less</a></li>
    <li><a href="http://blog.stackoverflow.com/2010/01/stack-overflow-network-configuration/">Stack Overflow Network Configuration</a></li>
    <li><a href="http://meta.stackoverflow.com/questions/69164/does-stackoverflow-use-caching-and-if-so-how">Does StackOverflow use caching and if so, how?</a></li>
    <li><a href="http://meta.stackoverflow.com/questions/10369/which-tools-and-technologies-build-the-stack-exchange-network">Which tools and technologies build the Stack Exchange Network?</a></li>
</ul>

StackOverflow在这段时间里一些明显的变化有：

<ul>
    <li>快速增长. 更多的用户，更多的PV，更多的数据中心，更多的网站，更多的开发人员，更多的操作系统，更多的数据库，更多的服务器。很多的<a href="http://blog.stackoverflow.com/2011/01/state-of-the-stack-2010-a-message-from-your-ceo/">更多</a>。
    </li>
    <li>Linux. Stack Overflow被人熟知的是它大量使用部署了Windows的服务器, 但是他们现在使用很多部署了Linux的服务器来跑 HAProxy, Redis, Bacula, Nagios, 日志和路由。 所有需要<a href="http://blog.serverfault.com/post/1097492931/">并行处理</a>的功能都会交给Linux处理。</li>
    <li>容错. Stack Overflow现在<a href="http://blog.stackoverflow.com/2010/01/stack-overflow-network-configuration/">使用两个不同网络线路接入的交换机</a> 他们增加了冗余的服务器，一些功能也被转移到了第二个数据中心。</li>
    <li>NoSQL. Redis现在作为整站的<a href="http://meta.stackoverflow.com/questions/69164/does-stackoverflow-use-caching-and-if-so-how">缓存层</a>在使用。以前并没有一个单独的缓存层，平且在Linux服务器上使用NoSQL，所以这是一个很大的变化。</li>
</ul>

遗憾的是，我没有找到我所关心的问题的答案，比如他们是怎么在这么多不同的特性中处理<a href="http://en.wikipedia.org/wiki/Multitenancy">Multitenancy（多租户）</a>的问题的，但是还是有很多东西可以学习。下面是一些数据汇总：

**统计信息**
<ul>
    <li>每月9500万次浏览量</li>
    <li>每秒800个HTTP请求</li>
    <li>每秒180个DNS请求</li>
    <li>每秒55M流量</li>
    <li>1600万个用户（Stack Overflow的流量在2010年增长了131%，全球每月不重复访客增至1660万人）。</li>
</ul>

**数据中心**
<ul>
    <li>1个机架放在俄勒冈州的Peak Internet（用于放置chat和Data Explorer）</li>
    <li>2个机架放在纽约州的Peer 1（用于放置Stack Exchange Network的其余部分）</li>
</ul>

**硬件设备**
<ul>
    <li>10台戴尔R610 IIS Web服务器（3台专门用于Stack Overflow）：1个英特尔至强处理器E5640，2.66 GHz四核，8线程；16 GB内存；Windows Server 2008 R2</li>
    <li>2台戴尔R710数据库服务器：2个英特尔至强处理器X5680，3.33 GHz；64 GB内存；8个硬盘；SQL Server 2008 R2</li>
    <li>2台戴尔R610 HAProxy服务器：1个英特尔至强处理器E5640，2.66 GHz；4 GB内存；Ubuntu Server</li>
    <li>2台戴尔R610 Redis服务器：2个英特尔至强处理器E5640，2.66 GHz；16 GB内存；CentOS</li>
    <li>1台戴尔R610 Linux备份服务器，运行Bacula：1个英特尔至强处理器E5640，2.66 GHz；32 GB内存</li>
    <li>1台戴尔R610 Linux管理服务器，用于Nagios和日志：1个英特尔至强处理器E5640，2.66 GHz；32 GB内存</li>
    <li>2个戴尔R610 VMWare ESXi域控制器：1个英特尔至强处理器E5640，2.66 GHz；16 GB内存；2只Linux路由器；5台戴尔Power Connect交换机</li>
</ul>

**开发工具**
<ul>
    <li>编程语言：C#（ASP.NET）</li>
    <li>开发环境：Visual Studio 2010 Team Suite</li>
    <li>开发框架：Microsoft ASP.NET Framework 4.0</li>
    <li>Web框架：ASP.NET MVC 3</li>
    <li>视图引擎：Razor</li>
    <li>Ajax框架：jQuery 1.4.2</li>
    <li>数据访问：LINQ to SQL，一些原生SQL</li>
    <li>版本控制：Mercurial和Kiln</li>
    <li>代码比较：Beyond Compare 3</li>
</ul>

**软件与技术**
<ul>
    <li>堆栈技术：<a href="http://blog.stackoverflow.com/2009/03/stack-overflow-and-bizspark/">BizSpark</a> 的 <a href="http://stackoverflow.com/questions/177901/what-does-wisc-stack-mean">WISC</a></li>
    <li>操作系统：<a href="http://www.microsoft.com/windowsserver2008/en/us/default.aspx">Windows Server 2008 R2 x64</a>，<a href="http://www.ubuntu.com/server">Ubuntu Server</a>，<a href="http://www.centos.org/">CentOS</a></li>
    <li>数据库：运行于Microsoft Windows Server 2008 Enterprise Edition x64 的<a href="http://www.microsoft.com/sqlserver/en/us/default.aspx">SQL Server 2008 R2</a></li>
    <li>WEB服务器：IIS7.0</li>
    <li>负载均衡：<a href="http://haproxy.1wt.eu/">HAProxy</a></li>
    <li>缓存：&nbsp;<a href="http://redis.io/">Redis</a>（分布式缓存）</li>
    <li>代码部署：<a href="http://sourceforge.net/projects/ccnet/">CruiseControl.NET</a></li>
    <li>搜索系统：<a href="http://incubator.apache.org/lucene.net/">Lucene.NET</a></li>
    <li>备份系统：<a href="http://www.bacula.org/en/">Bacula</a></li>
    <li>监控系统：<a href="http://www.nagios.org/">Nagios</a>&nbsp;(使用 n2rrd 和 drraw 插件)</li>
    <li>日志分析：<a href="http://www.splunk.com/">Splunk</a></li>
    <li>SQL Server监控：<a href="http://www.red-gate.com/products/dba/sql-monitor/">SQL Monitor from Red Gate</a></li>
    <li>DNS解析：<a href="http://www.isc.org/software/bind">Bind</a></li>
    <li>远程控制：<a href="http://www.wowwee.com/en/products/tech/telepresence/rovio/rovio">Rovio</a></li>
    <li>外部监控：<a href="http://tools.pingdom.com/">Pingdom</a></li>
</ul>

**外部组件**（作为一部分开发工具并没有包含在代码中）
<ul>
    <li>reCAPTCHA</li>
    <li>DotNetOpenId</li>
    <li>WMD - Now developed as open source. See github network graph</li>
    <li>Prettify</li>
    <li>Google Analytics</li>
    <li>Cruise Control .NET</li>
    <li>HAProxy</li>
    <li>Cacti</li>
    <li>MarkdownSharp</li>
    <li>Flot</li>
    <li>Nginx</li>
    <li>Kiln</li>
    <li>CDN: 没有使用CDN，所有静态内容由<a href="http://sstatic.net">sstatic.net</a>提供。sstatic.net是一个快速的、无cookie的域，用于将静态内容分发到Stack Exchange系列网站。</li>
</ul>

**开发人员和系统管理员**
<ul>
    <li>14名开发人员</li>
    <li>2名系统管理员</li>
</ul>

**内容**
<ul>
    <li>协议: Creative Commons Attribution-Share Alike 2.5 Generic</li>
    <li>标准: OpenSearch, Atom</li>
    <li>托管: PEAK Internet</li>
</ul>

**每个站点有三种不同的缓存：本地缓存、站点缓存和全局缓存。**
<ul>本地缓存：只能被一对服务器或站点访问。</ul>
<ul>
    
        <li>为了限制网络等待时间，他们使用本地“L1”缓存。基本上是在一台服务器上用HttpRuntime.Cache缓存最近设置或读取的数据。这样就可以使在网络上的缓存查找开销减少到0字节。
        </li>
        <li>本地缓存包含用户session和待定的查看计数更新。</li>
        <li>本地缓存只在内存中，没有网络或数据库的访问。</li>
</ul>

<ul>站点缓存：可以被单个站点的任何一个服务器上的实例访问。</ul> 
<ul>
        <li>绝大多数的数据被缓存在这里，比如热门问题的id列表和用户的回答被接受的百分比是很好的例子。</li>
        <li>站点缓存使用的是Redis（在一个不同的数据库里，为了更方便调试）。</li>
        <li>Redis非常快，导致缓存查找中最慢的部分变成了从网络读字节或写字节到网络中去。
        </li>
        <li>
        数据是被压缩过后再发送到Redis的。有很多的CPU，并且大多数数据是字符串类型的，所以他们可以得到很好的压缩比率。
        </li>
        <li>
        Redis所在服务器的CPU使用率是0%。
        </li>
    </ul>
  
<ul>全局缓存：所有站点和服务器共享。</ul>
    
<ul>
        <li>包括收件箱，API使用的引用和一些其他全局数据。</li>
        <li>全局缓存使用Redis（在DB 0，同样为了调试方便）。</li>
</ul>
   

**更多的架构和经验**

<ul>
    <li>使用HAProxy替代Windows NLB 的原因是：HAProxy不仅使用简单而且还是免费的，通过Hyper-V可以成为优秀的网络上的一个512M虚拟机“设备”。它运行在服务器的前端，所以对服务器来说完全透明，而且作为一个不同的网络层，比起和windows的配置混在一起，更容易故障检测，</li>
    <li>没有使用CDN，因为就连亚马逊这样“便宜的”CDN，如果考虑捆绑到现有主机方案中的带宽，其费用都相当昂贵的。根据亚马逊的CDN费用和Stack Overflow使用的带宽，每月至少要支付1000美元。</li>
    <li>为了快速恢复，把数据备份到磁盘上。为了历史归档，把数据备份到磁带（tape）上。</li>
    <li>SQL Server对的全文搜索的集成相当差，bug多，功能少，所以改用了Lucene。</li>
    <li>最让人感兴趣的是他们能够确保可以处理的HTTP请求峰值。</li>
    <li>所有特性都运行在Stack Exchange平台上。Stack Overflow、Super User、Server Fault、Meta、WebApps和Meta Web Apps都运行同一软件上。</li>
    <li>对StackExchange的用户进行区分是因为拥有不同专业技能的人不应该在不同主题的站点之间穿梭。<a href="http://meta.stackoverflow.com/questions/69422/why-separate-stack-exchange-accounts">你可以是世界上最出色的大厨，但这并不能说明修复服务器的能力</a>。</li>
    <li>尽可能缓存所有数据。</li>
    <li>所有匿名用户访问的页面都由<a href="http://learn.iis.net/page.aspx/154/walkthrough-iis-70-output-caching/">Output Caching</a>缓存。</li>
    
    <li>
    大多数数据会在超时时间段后过期（通常是几分钟），并且他们不会被显示地删除。
    当需要特定的缓存失效时，使用<a href="http://code.google.com/p/redis/wiki/PublishSubscribe">Redis messaging</a>向“L1”缓存发送删除通知。
    </li>
    <li>
    Joel Spolsky并不是微软的忠实用户，他并不对Stack Overflow做技术决策，他认为微软许可发放是个错误。认为你自己是正确的<a href="http://news.ycombinator.com/item?id=2284900">Hacker News commentor</a>。
    </li>
    <li>
    他们在<a href="http://www.intel.com/design/flash/nand/extreme/index.htm">Intel X25 solid state drives</a>上做RAID 10来作为IO系统。RAID解决了可靠性的顾虑，并且对比FusionIO，SSD性能更好，价格更便宜。
    </li>
    <li>
    全部的微软许可<a href="http://news.ycombinator.com/item?id=2285931">费用</a>大约是$242K，因为Stack Overflow使用的是Bizspark，所以并没有支付全价，但这已经是他们可能支付的最大费用。
    </li>
    <li>
    <a href="http://blog.serverfault.com/post/broadcom-die-mutha/">Intel NICs are replacing Broadcom NICs</a>和他们主要的生产环境服务器。这样就解决了连接丢失，包丢失和ARP表冲突。
    </li>

</ul>

**相关文章**

<ul>
    <li><a href="http://news.ycombinator.com/item?id=2284900">Hacker News Thread on this Post</a> / <a href="http://www.reddit.com/r/programming/comments/fwpik/stackoverflow_scales_using_a_mixture_of_linux_and/">Reddit Thread</a></li>
    <li><a href="http://blog.serverfault.com/post/stack-exchanges-architecture-in-bullet-points/">Stack Exchange’s Architecture in Bullet Points</a> / <a href="http://news.ycombinator.com/item?id=2207789">HackerNews Thread</a></li>
    <li><a href="http://blog.serverfault.com/post/1432571770/">Stack Overflow’s New York Data Center</a> - <span style="font-family: 'Trebuchet MS',Arial,'Bitstream Vera Sans',sans-serif; line-height: 17px; color: rgb(0, 0, 0);">hardware of the various machines?</span></li>
    <li><a href="http://blog.serverfault.com/post/1097492931/">Designing For Scalability of Management and Fault Tolerance</a></li>
    <li><a href="http://blog.stackoverflow.com/">Stack Overflow Blog</a></li>
    <li><a href="http://blog.stackoverflow.com/2011/01/stack-overflow-search-now-81-less-crappy/">Stack Overflow Search &mdash; Now 81% Less Crappy</a>&nbsp;- Lucene is now running on an underused cluster.</li>
    <li><a href="http://blog.stackoverflow.com/2011/01/state-of-the-stack-2010-a-message-from-your-ceo/">State of the Stack 2010 (a message from your CEO)</a></li>
    <li><a href="http://blog.stackoverflow.com/2010/01/stack-overflow-network-configuration/">Stack Overflow Network Configuration</a></li>
    <li><a href="http://meta.stackoverflow.com/questions/69164/does-stackoverflow-use-caching-and-if-so-how">Does StackOverflow use caching and if so, how?</a></li>
    <li><a href="http://meta.stackoverflow.com/">Meta StackOverflow</a></li>
    <li><a href="http://meta.stackoverflow.com/questions/6435/how-does-stackoverflow-handle-cache-invalidation">How does StackOverflow handle cache invalidation?</a></li>
    <li><a href="http://meta.stackoverflow.com/questions/10369/which-tools-and-technologies-build-the-stack-exchange-network">Which tools and technologies build the Stack Exchange Network?</a></li>
    <li><a href="http://meta.stackoverflow.com/questions/2765/how-does-stack-overflow-handle-spam">How does Stack Overflow handle spam?</a></li>
    <li><a href="http://blog.serverfault.com/post/our-storage-decision/">Our Storage Decision</a></li>
    <li><a href="http://meta.stackoverflow.com/questions/4766/how-are-hot-questions-selected">How are “Hot” Questions Selected?</a>&nbsp;</li>
    <li><a href="http://meta.stackoverflow.com/questions/20473/how-are-related-questions-selected">How are “related” questions selected?</a>&nbsp;- the title, the question body, and the tags.&nbsp;</li>
    <li><a href="http://blog.stackoverflow.com/2010/04/stack-overflow-and-dvcs/">Stack Overflow and DVCS</a>&nbsp;- Stack Overflow selects Mercurial for source code control.</li>
    <li><a href="http://chat.stackexchange.com/rooms/127/the-comms-room">Server Fault Chat Room</a>&nbsp;</li>
    <li><a href="https://github.com/ServiceStack/ServiceStack.Redis">C# Redis Client</a></li>
    <li><a href="http://blog.serverfault.com/post/broadcom-die-mutha/">Broadcom, Die Mutha</a></li>
</ul>


英文原文地址：

[stack-overflow-architecture-update-now-at-95-million-page-vi](http://highscalability.com/blog/2011/3/3/stack-overflow-architecture-update-now-at-95-million-page-vi.html "stack-overflow-architecture-update-now-at-95-million-page-vi")

