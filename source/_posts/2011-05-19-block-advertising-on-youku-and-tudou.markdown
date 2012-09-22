---
layout: post
title: "屏蔽优酷和土豆广告的方法"
date: 2011-05-19 22:58
comments: true
categories: life
---
**Windows:** C:\WINDOWS\system32\drivers\etc\hosts

**Linux:** /etc/hosts

根据你所用的操作系统，用文本编辑工具打开上面的文件，然后添加下面的代码：

\#优酷  
127.0.0.1 atm.youku.com  
127.0.0.1 Fvid.atm.youku.com  
127.0.0.1 html.atm.youku.com  
127.0.0.1 valb.atm.youku.com  
127.0.0.1 valf.atm.youku.com  
127.0.0.1 valo.atm.youku.com  
127.0.0.1 valp.atm.youku.com  
127.0.0.1 lstat.youku.com  
127.0.0.1 speed.lstat.youku.com  
127.0.0.1 urchin.lstat.youku.com  
127.0.0.1 stat.youku.com  
127.0.0.1 static.lstat.youku.com  
127.0.0.1 valc.atm.youku.com  
127.0.0.1 vid.atm.youku.com  
127.0.0.1 walp.atm.youku.com  

\#土豆  
127.0.0.1 adextensioncontrol.tudou.com  
127.0.0.1 iwstat.tudou.com  
127.0.0.1 nstat.tudou.com  
127.0.0.1 stats.tudou.com  
127.0.0.1 \*.p2v.tudou.com\*  
127.0.0.1 at-img1.tdimg.com  
127.0.0.1 at-img2.tdimg.com  
127.0.0.1 at-img3.tdimg.com  
127.0.0.1 adplay.tudou.com  
127.0.0.1 adcontrol.tudou.com  
127.0.0.1 stat.tudou.com  
