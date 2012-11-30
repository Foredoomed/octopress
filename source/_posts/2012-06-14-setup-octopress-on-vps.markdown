---
layout: post
title: "快速搭建Octopress博客环境"
date: 2012-06-14 23:04
comments: true
categories: octopress
---
这次为一个新买的VPS安装博客环境，特此记录如下。

**1.删除和替换已有软件**

{% codeblock lang:bash %}
servive sendmail stop
yum remove sendmail

service httpd stop
yum remove httpd

service rsyslog stop
yum remove rsyslog

# 添加EPEL软件库
rpm -Uvh http:#dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-7.noarch.rpm

# 安装syslog-ng代替rsyslog
yum install syslog-ng
chkconfig syslog-ng on

# 安装dropbear代替openssh
yum install dropbear
chkconfig dropbear on

# 等dropbear配置好后
service sshd stop
yum remove openssh

# 安装vsftpd作为ftp服务器
yum install vsftpd
chkconfig vsftpd on

# 添加用户
groupadd www
useradd -g www username
passwd username

# 设定ftp登陆目录权限
chown -R usernmae /path/to/ftp/folder

# 安装yum-utils
yum install yum-utils

# 更新软件
yum upgrade

# 安装ppp
rpm -ivh http:#poptop.sourceforge.net/yum/stable/rhel6/i386/ppp-2.4.5-17.0.rhel6.i686.rpm

# 安装pptpd
rpm -ivh http:#poptop.sourceforge.net/yum/stable/rhel6/i386/pptpd-1.3.4-2.el6.i686.rpm
{% endcodeblock %}

<!-- more -->

**2.安装Octopress所需软件**

{% codeblock lang:bash %}
# 安装git
yum install git

# 安装RVM
curl -L get.rvm.io | bash -s stable

# 安装ruby的依赖包
yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison

# 添加nginx软件库
rpm -Uvh http:#nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm

# 安装nginx
yum install nginx

# 安装ruby
rvm install 1.9.2

# 安装Octopress的依赖包
yum install python-devel

# 安装Octopress
mkdir /data
cd /data
git clone git:#github.com/imathis/octopress.git octopress

gem install bundler
rbenv rehash #If you use rbenv, rehash to be able to run the bundle command
bundle install
rake install

cd octopress
rake generate
{% endcodeblock %}

**3.VPS设置**

{% codeblock lang:bash %}
# 配置vim
vim /etc/vimrc

# 加入下面几行
syntax on
set nu
set mouse=a
set tabstop=4

# 禁用IPv6
vim /etc/sysconfig/network
NETWORKING_IPV6="no"

# 设置文件最大打开数
vim /etc/security/limits.conf
# 加入下面两行
* soft nofile 65535
* hard nofile 65535

# 把服务器时间改为上海时间
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 关闭不需要的服务
ntsysv  #按空格选择和取消，按F12退出
{% endcodeblock %}

**4.其他设置**

因为众所周知的原因在天朝twitter是无法访问的，所以最好去掉内置的twitter插件，这样就不会去请求twitter的js文件。方法是修改source/_includes文件夹下_twitter_sharing.html的文件名。

关于dropbear/vsftp/pptp的配置可以参考[这里](http://liuxuan.info/blog/2011/08/21/install-dropbear-vsftp-pptpd-on-vps/)，nginx的配置则可以参考[这里](http://liuxuan.info/blog/2011/11/08/switching-from-wordpress-to-octopress/)。
