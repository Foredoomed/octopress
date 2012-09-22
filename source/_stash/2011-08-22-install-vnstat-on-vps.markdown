---
layout: post
title: "在VPS上安装网络流量监控工具vnStat"
date: 2011-08-22 18:04
comments: true
categories: vps
---
用上VPS以后博客的访问速度是比以前快了很多，但是也有烦心的事，就是担心每个月的流量有没有超标。你可以通过VPS提供给你的Control Panel的地址登录，然后查看流量数据。但是这种方法始终有点复杂，最好有种直接输入一个URL就能查看流量数据的方法。在Google上搜索了一会儿之后，终于找到了一个好工具：[vnStat](http://humdi.net/vnstat/ "vnStat")。 

<!--more-->

vnStat是Linux和BSD平台上基于控制台的网络流量监控工具。它使用的是Linux内核提供的网络接口统计作为信息源。vnStat不会嗅探网络并且保证占系统资源少，需要注意的是内核版本要在2.2之上。

vnStat只是一个基于控制台的工具，我们还需要安装一个它的PHP扩展[vnStat PHP frontend](href="http://www.sqweek.com/sqweek/index.php?p=1 "vnStat PHP frontend")，那么我们就能直接通过流量器来查看流量数据了。

速度安装起来：

1.安装vnStat:

{% codeblock lang:bash %}
wget http://humdi.net/vnstat/vnstat-1.11.tar.gz
tar zxvf vnstat-1.11.tar.gz
cd vnstat-1.11
make && make install
{% endcodeblock %}

2.安装vnStat PHP frontend：

{% codeblock lang:bash %}
wget http://www.sqweek.com/sqweek/files/vnstat_php_frontend-1.5.1.tar.gz
cp -r vnstat_php_frontend-1.5.1 /你的Nginx的根目录/vnstat
{% endcodeblock %}

然后就可以通过访问`http://www.yourdomain.com/vnstat`，应该就可以看到流量数据页面了，但是现在还没有数据，接下来我们来给系统生成数据。

3.建立流量数据库:

首先查看你是哪种外网网卡：

{% codeblock lang:bash %}
ifconfig    //Xen一般是eth0；OpenVZ一般是venet0
{% endcodeblock %}

然后生成数据库：

{% codeblock lang:bash %}
/usr/bin/vnstat -u -i venet0  //因为我的VPS是基于OpenVZ的，所以就是venet0
{% endcodeblock %}

配置通过cron的方式定时更新数据库,编辑`/etc/cron.d/vnstat`文件，加入下面的内容：

{% codeblock lang:bash %}
0-55/5 * * * *   root   vnstat -u -i venet0
0-55/5 * * * *   root   vnstat --dumpdb -i venet0 >/var/lib/vnstat/vnstat_dump_venet0
{% endcodeblock %}

注意：第二行是为了更新venet0的数据后，dump出来一个文件提供给PHP访问.这里dump出来的vnstat_dump_venet0文件名是有规定的。

4.修改配置文件,编辑`/你的Nginx的根目录/vnstat/config.php`：

{% codeblock lang:bash %}
$language = 'en';
$iface_list = array('venet0');
$iface_title['venet0'] = 'VPS';
$data_dir = '/var/lib/vnstat/';
$graph_format='png';
{% endcodeblock %}

到这里安装就都完成了，就这么简单。现在访问www.yourdomain.com/vnstat就会发现有流量统计了,统计数据更新是５分钟刷新一次。

