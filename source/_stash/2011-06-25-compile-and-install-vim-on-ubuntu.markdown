---
layout: post
title: "Ubuntu下编译安装Vim"
date: 2011-06-25 14:56
comments: true
categories: vim
---
**一.编译安装最新版本Vim**

我们都知道在Ubuntu下如果直接在终端中用命令安装Vim，那么安装的都不是最新版本。如果要想安装最新版本只有下载最新版源代码包，然后再编译安装。

首先，Vim最新版源代码包可以在 [官网](http://www.vim.org/ "官网") 下载，下载源代码包到本地Ubuntu主机后，解压缩，进入该目录，进行配置，编译和安装：
{% codeblock lang:shell %}
tar xzvf vim-7.3.tar.bz2
cd vim73
./configure --prefix=/usr/local/vim --enable-rubyinterp --enable-multibyte --enable-gui=gnome2 --enable-xim --enable-fontset --with-features=huge
make && make install
{% endcodeblock %}
<!--more-->

如果安装出现错误则有可能是目录的权限问题，试试下面的命令吧：
{% codeblock lang:shell %}
sudo chmod 777 /usr/bin
sudo chmod 777 /usr/local/
{% endcodeblock %}

好了，如果不出什么意外的话，Vim应该已经安装好了，接下来就是要安装和配置插件了。首先，要在主目录里新建**.vimrc**文件，然后在这个文件里配置Vim。
配置可以参考[这里](https://github.com/panweizeng/env "这里")

如果运行出现下面的错误：  
    (gvim:6667): Gtk-WARNING **: Invalid input string

解决办法是在主目录里`CTRL+H`显示`.bashrc`文件，然后在这里文件里添加如下代码：
{% codeblock lang:bash %}
export  LANG=zh_CN.UTF-8;
export  LC_CTYPE="zh_CN.UTF-8";
export  LC_NUMERIC="zh_CN.UTF-8";
export  LC_TIME="zh_CN.UTF-8";
export  LC_COLLATE="zh_CN.UTF-8";
export  LC_MONETARY="zh_CN.UTF-8";
export  LC_MESSAGES="zh_CN.UTF-8";
export  LC_PAPER="zh_CN.UTF-8";
export  LC_NAME="zh_CN.UTF-8";
export  LC_ADDRESS="zh_CN.UTF-8";
export  LC_TELEPHONE="zh_CN.UTF-8";
export  LC_MEASUREMENT="zh_CN.UTF-8";
export  LC_IDENTIFICATION="zh_CN.UTF-8";
{% endcodeblock %}

**二.配置Vim对Ruby的支持**

我主要用Vim来写Ruby代码，所以比如代码高亮，自动补全等都是必须的:
  
1.安装Ruby也可以去[官网](http://www.ruby-lang.org/ "官网")上下载最新版本，然后编译安装。  
2.设置环境变量。在`.bashrc`文件里添加如下代码：
{% codeblock lang:bash %}
export PATH=/usr/local/ruby/bin:$PATH
{% endcodeblock %}
3.下载[vim-ruby](http://rubyforge.org/projects/vim-ruby/ "vim-ruby")插件，解压后复制道Vim安装目录里。现在开始写Ruby代码吧，然后按F5看下运行结果是不是出来了呢。

最后Show一下我的Vim：

<a href="http://www.flickr.com/photos/60110479@N08/5868623217/" title="Flickr 上 Foredoomed 的 vim_screenshot"><img src="http://farm6.static.flickr.com/5067/5868623217_70b3bcae76_z.jpg" width="640" height="480" alt="vim_screenshot"/></a>

