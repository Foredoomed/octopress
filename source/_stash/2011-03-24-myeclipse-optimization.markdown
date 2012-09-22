---
layout: post
title: "MyEclipse优化设置"
date: 2011-03-24 18:27
comments: true
categories: programming
---
原文地址：[MyEclipse优化设置](http://www.javaeye.com/topic/789541 "MyEclipse优化设置") 

**第一步: 取消自动validation**

validation有一堆，什么xml、jsp、jsf、js等等，我们没有必要全部都去自动校验一下，只是需要的时候才会手工校验一下！
取消方法：
windows–>perferences–>myeclipse–>validation
除开Manual下面的复选框全部选中之外，其他全部不选

手工验证方法：
在要验证的文件上，单击鼠标右键–>myeclipse–>run validation
 
**第二步：取消Eclipse拼写检查**

1、拼写检查会给我们带来不少的麻烦，我们的方法命名都会是单词的缩写，他也会提示有错，所以最好去掉，没有多大的用处
windows–>perferences–>general–>validation->editors->Text Editors->spelling
 
**第三步：取消myeclipse的启动项**

myeclipse会有很多的启动项，而其中很多我们都用不着，或者只用一两个，取消前面不用的就可以
windows–>perferences–>general–>startup and shutdown  (详见底端介绍)

<!--more-->

**第四步：更改jsp默认打开的方式**

安装了myeclipse后，编辑jsp页面，会打开他的编辑页面，同时也有预览页面，速度很慢，不适合开发。
修改windows–>perferences–>general–>editors->file associations
在下方选择一种编辑器，然后点击左边的default按钮

**第五步：更改代码提示快捷键(不建议使用增强提示,使用Ctrl+/在自己需要的时候提示更佳)**

现在的代码提示快捷键，默认为`ctrl+space`，而我们输入法切换也是，所以会有冲突。
找到windows–>perferences–>general–>Keys
更改`content assist`为`alt+/`
同时由于`alt+/`已经被`word completion`占用，所以得同时修改`word completion`的快捷键值。
 
**第六步: 更改内存使用文件**

1、打开 eclipse.ini
{% codeblock eclipse.ini lang:java %}
-showsplash 
com.genuitec.myeclipse.product
--launcher.XXMaxPermSize 256M
-vmargs
-Dosgi.requiredJavaVersion=1.5
-Xms256m
-Xmx1024m   
-Dosgi.splashLocation=e:MyEclipse 6.0eclipseMyEclipseSplash.bmp
-Duser.language=en
-XX:PermSize=128M
-XX:MaxPermSize=256M
{% endcodeblock %}

把下面的那个`-XX:MaxPermSize`调大，比如`-XX:MaxPermSize=512M`，再把`-XX:PermSize`调成跟`-XX:MaxPermSize`一样大
原因：大家一定对这个画面很熟悉吧：
几乎每次 eclipse 卡到当都是因为这个非堆内存不足造成的，把最大跟最小调成一样是因为不让myeclipse频繁的换内存区域大小
注意：`XX:MaxPermSize`和`Xmx`的大小之和不能超过你的电脑内存大小
 
**第七步: 修改Struts-config.xml文件打开错误**

有时点击myeclipse里的struts的xml配置文件，会报错：
Error opening the editorUnable to open the editor ,unknow the editor id…..
把这个窗口关闭后才出正确的xml文件显示，这个我们这样改：
windows–>perferences–>general–>editors->file associations选择`*.xml`，选择`myeclipse xml editor`点`default`
 
**第八步: 取消自动验证,该成手动验证**

windows-->perferences-->myeclipse-->validation
将Build下全部勾取消,保留Manual(手动) 如果你需要验证某个文件的时候，我们可以单独去验证它。方法是，在需要验证的文件上(右键 -> MyEclipse -> Run Validation)
 
**第九步: 取消Maven更新(启动更新)**

Window > Preferences > Myeclipse Enterprise Workbench > Maven4Myeclipse > Maven>禁用Download repository index updates on startup
 
**附件:**
 
第一项: 启动功能介绍和样例(红色为需要保留的文件,此为本人样例,请按需选择)
<ol>
<li><span><span>Automatic&nbsp;Updates&nbsp;Scheduler&nbsp;</span><span><span style="color: rgb(0, 130, 0);">//自动升级调度</span></span> <span>&nbsp;&nbsp;</span></span></li>
<li>
<span>MyEclipse&nbsp;QuickSetup&nbsp;</span><span><span style="color: rgb(0, 130, 0);">//快速启动</span></span> <span>&nbsp;&nbsp;</span> </li>
<li>
<span>MyEclipse&nbsp;Derby&nbsp;</span><span><span style="color: rgb(0, 130, 0);">//derby是一个开源数据库的名字</span></span> </li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Geronimo&nbsp;</span><span>1</span><span>&nbsp;</span><span>//同色都是应用服务器的名字</span> <span>&nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Geronimo&nbsp;</span><span>2</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;JBOSS&nbsp;</span><span>2</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;JBOSS&nbsp;</span><span>3</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;JBOSS&nbsp;</span><span>4</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;JBOSS&nbsp;</span><span>5</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);">MyEclipse&nbsp;EASIE&nbsp;JBOSS&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Jetty&nbsp;</span><span>4</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Jetty&nbsp;</span><span>5</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Jetty&nbsp;</span><span>6</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);">MyEclipse&nbsp;EASIE&nbsp;Jetty&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;JOnAS&nbsp;</span><span>3</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;JOnAS&nbsp;</span><span>4</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);">MyEclipse&nbsp;EASIE&nbsp;JOnAS&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;JRun&nbsp;</span><span>4</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);">MyEclipse&nbsp;EASIE&nbsp;JRun&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Oracle&nbsp;</span><span>10</span><span>&nbsp;AS&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span style="color: rgb(237, 28, 36);"><span>MyEclipse&nbsp;EASIE&nbsp;Oracle&nbsp;</span><span>9</span></span><span><span style="color: rgb(237, 28, 36);">&nbsp;AS</span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);">MyEclipse&nbsp;EASIE&nbsp;Oracle&nbsp;AS&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Orion&nbsp;</span><span>1</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Orion&nbsp;</span><span>2</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Resin&nbsp;</span><span>2</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Resin&nbsp;</span><span>3</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);">MyEclipse&nbsp;EASIE&nbsp;Resin&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Sun&nbsp;</span><span>8</span><span>.x&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Sun&nbsp;</span><span>8</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Sun&nbsp;</span><span>9</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Glassfish&nbsp;</span><span>2</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Glassfish&nbsp;</span><span>1</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);">MyEclipse&nbsp;EASIE&nbsp;Sun&nbsp;One&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span style="color: rgb(255, 0, 0);"><span>MyEclipse&nbsp;EASIE&nbsp;MyEclipse&nbsp;Tomcat&nbsp;</span><span>6</span></span><span><span style="color: rgb(255, 0, 0);">&nbsp;Server</span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Tomcat&nbsp;</span><span>4</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;Tomcat&nbsp;</span><span>5</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(237, 28, 36);"><span>MyEclipse&nbsp;EASIE&nbsp;Tomcat&nbsp;</span><span>6</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);"><span style="color: rgb(237, 28, 36);">MyEclipse&nbsp;EASIE&nbsp;Tomcat</span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;WebLogic&nbsp;</span><span>10</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;WebLogic&nbsp;</span><span>6</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;WebLogic&nbsp;</span><span>7</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;WebLogic&nbsp;</span><span>8</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span style="color: rgb(237, 28, 36);"><span>MyEclipse&nbsp;EASIE&nbsp;WebLogic&nbsp;</span><span>9</span></span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span><span style="color: rgb(34, 177, 76);">MyEclipse&nbsp;EASIE&nbsp;WebLogic&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;WebSphere&nbsp;</span><span>5</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;WebSphere&nbsp;</span><span>6.1</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;WebSphere&nbsp;</span><span>6</span><span>&nbsp; &nbsp;&nbsp;</span></span></li>
<li>
<span style="color: rgb(34, 177, 76);"><span>MyEclipse&nbsp;EASIE&nbsp;WebSphere&nbsp;</span><span>4</span></span><span>&nbsp; &nbsp;&nbsp;</span> </li>
<li>
<span>MyEclipse&nbsp;Examples&nbsp;</span><span><span style="color: rgb(0, 130, 0);">//样例</span></span> <span>&nbsp;&nbsp;</span> </li>
<li><span style="color: rgb(237, 28, 36);"><span>MyEclipse&nbsp;Memory&nbsp;Monitor&nbsp;</span><span>//内存监控</span> <span>&nbsp;&nbsp;</span></span></li>
<li>
<span><span style="color: rgb(237, 28, 36);">MyEclipse&nbsp;Tapestry&nbsp;Integration&nbsp;</span></span><span><span style="color: rgb(0, 130, 0);"><span style="color: rgb(237, 28, 36);">//插件集成</span></span></span> <span>&nbsp;&nbsp;</span> </li>
<li><span style="color: rgb(237, 28, 36);"><span>MyEclipse&nbsp;JSP&nbsp;Debug&nbsp;Tooling&nbsp;</span><span>//jsp调试插件</span> <span>&nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(237, 28, 36);"><span>MyEclipse&nbsp;File&nbsp;Creation&nbsp;Wizards&nbsp;</span><span>//文件创建程序&nbsp;</span><span>&nbsp;&nbsp;</span></span></li>
<li><span style="color: rgb(237, 28, 36);"><span>ICEfaces Integration for MyEclipse //基于Ajax的JSF开发框架()</span></span></li>
<li><span style="color: rgb(237, 28, 36);"><span>MyEclipse&nbsp;Backward&nbsp;Compatibility&nbsp;</span><span>//后台功能</span> <span>&nbsp;&nbsp;</span></span></li>
<li>
<span><span style="color: rgb(237, 28, 36);">MyEclipse&nbsp;Perspective&nbsp;Plug-in&nbsp;</span></span><span><span style="color: rgb(0, 130, 0);"><span style="color: rgb(237, 28, 36);">//透视图插件</span>&nbsp;</span></span><span>&nbsp;&nbsp;</span> </li>
<li><span>Pluse Collaboration Control Center //Eclipse的网页管理中心</span></li>
<li><span><span style="color: rgb(237, 28, 36);">eclipse-cs 4.x.x -&gt; 5.0.0 Migration Plug-in&nbsp; //Eclipse插件兼容组件</span></span></li>
<li>
<span>Mozilla&nbsp;Debug&nbsp;UI&nbsp;Plug-in(Incubation)&nbsp;</span><span><span style="color: rgb(0, 130, 0);">//Mozilla调试插件（Mozilla是一款浏览器)&nbsp;</span></span><span>&nbsp;&nbsp;</span> </li>
<li><span><span style="color: rgb(237, 28, 36);">Dynamic Languages ToolKit Core UI //对入PHP等动态语言支持的用户接口</span></span></li>
<li>
<span>WTP&nbsp;Webservice&nbsp;UI&nbsp;Plug-in&nbsp;</span><span><span style="color: rgb(0, 130, 0);">//Web&nbsp;服务视图插件</span></span> <span>&nbsp;&nbsp;</span> </li>
<li><span style="color: rgb(237, 28, 36);"><span>JavaServer&nbsp;Faces&nbsp;Tools&nbsp;-&nbsp;Core&nbsp;</span><span>//jsf工具核心包&nbsp;</span><span>&nbsp;&nbsp;</span></span></li>
<li><span>Automatic Updates Scheduler //自动更新</span></li>
<li><span><span style="color: rgb(237, 28, 36);">Service policy&nbsp; //Web提供的服务性能目标定义,自动管理</span></span></li>
<li><span><span style="color: rgb(237, 28, 36);">Atfdebug Plug-in(Incubation)&nbsp; //动态语言的调试工具</span></span></li>
<li><span><span style="color: rgb(237, 28, 36);">Auxiliary Web Module Support for MeEclipse// 辅助的Web模块支持.(可能是Struts等文件自动添加)</span></span></li>
<li>
<span style="color: rgb(237, 28, 36);"><span>JSF&nbsp;Editor&nbsp;Preview&nbsp;Support&nbsp;</span><span>**for**</span><span>&nbsp;MyEclipse</span><span>//jsf编辑器</span></span><span>&nbsp;&nbsp;</span> </li>
</ol>

第二项: MyEclipse Validation
由于文件导入的时候，不能保证文件的正确性，所以在启动服务前需要做一下验证，包括语法等。另外可以自己添加需要的验证模块．如checkStyle的验证。
