---
layout: post
title: "用Munin监控VPS的运行情况"
date: 2012-06-17 13:42
comments: true
categories: vps 
---
因为换了vps，所以还要重新安装监控软件。而Linux平台上的系统监控软件还是很多的，比如大名顶顶的[Nagios](http://www.nagios.org/)，[Cacti](http://www.cacti.net/)和[Zabbix](http://www.zabbix.com/),还有[MRTG](http://oss.oetiker.ch/mrtg/)。Nagios是一个比较重量级的软件，而且还必须搭配许多插件才能满足需求，所以在小内存vps上不适用；Cacti是通过[SNMP](http://en.wikipedia.org/wiki/Simple_Network_Management_Protocol)协议来监控系统，这意味着还要为它在后台开启SNMP进程，这对于小内存vps也不可取，还有cacti需要数据库的支持，web前端还需要php，所以综上也放弃；Zabbix虽然不错，但它也需要有数据库的支持，所以也放弃；而MRTG只能够监控网络使用情况，不能够监控CPU，磁盘等设备，所以还是放弃。最后我选择的是[Munin](http://munin-monitoring.org/),它的有点是不仅轻量，不需要数据库，可以直接生成静态页面，应该说Munin是小内存vps上的理想选择。

<!-- more -->

如果想要安装Munin的最新版本就需要自己编译安装，软件库里的版本还是有点旧。但是自己编译安装会比较麻烦，因为要安装编译所需要的很多依赖库，所以我还是选择从软件库里直接安装。好了，下面就开始安装Munin，操作系统是CentOS 6 32位。

首先安装EPEL软件库，如果已经装过就跳过此步

{% codeblock lang:bash %}
rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-7.noarch.rpm
{% endcodeblock %}

然后安装munin

{% codeblock lang:bash %}
yum -y install munin munin-node
{% endcodeblock %}

设置开机启动和目录权限

{% codeblock lang:bash %}
chkconfig munin-node on

chown -R munin:munin /var/www/html/munin/
{% endcodeblock %}

接下来就是配置munin

{% codeblock lang:bash %}
cd /etc/munin

vim munin.conf

// 打开下面几行的注释
dbdir   /var/lib/munin
htmldir /var/www/html/munin
logdir  /var/log/munin
rundir  /var/run/munin

tmpldir /etc/munin/templates

// 修改munin页面上显示的munin
[liuxuan.info]
    address 127.0.0.1
    use_node_name yes
{% endcodeblock %}

配置munin-node

{% codeblock lang:bash %}
// 修改hostname
host_name   liuxuan.info

// 如果有多台机器需要监控的话还需要加上他们的IP地址
allow ^xxx\.xxx\.xxx\.xxx$
{% endcodeblock %}

配置对nginx的监控

{% codeblock lang:bash %}
cd /etc/munin/plugins/
ln -s /usr/share/munin/plugins/nginx_request nginx_request
ln -s /usr/share/munin/plugins/nginx_status nginx_status

// 修改nginx_request里的hostname，搜索$URL,把其中的$fqdn改为localhost
// 然后输入下面的命令，如果输出yes则说明配置成功
perl nginx_request autoconf

// 配置nginx
cd /etc/nginx/conf.d
vim munin.conf
// 然后加入下面的配置
server {
        listen 127.0.0.1;
        server_name localhost;
        location /nginx_status {
                stub_status on;
                access_log   off;
                allow 127.0.0.1;
                deny all;
        }
        
	    location /munin {
        	root /var/www/html/munin;
  			expires off;
  			access_log   off;
  			auth_basic "Munin";
  			auth_basic_user_file /etc/nginx/.htpasswd;
		 }
}

// 重启nginx使配置生效
nginx -s reload

// 设置访问munin的用户名和秘密
htpasswd -c /etc/nginx/.htpasswd username
{% endcodeblock %}

// 最后设置计划任务每5分钟获取数据
{% codeblock lang:bash %}
sudo -u munin crontab -e
*/5 * * * *  /usr/bin/munin-cron
// 一定要先执行下面这行命令，否则不会生成index.html，访问会报403错误
sudo -u munin munin-cron
{% endcodeblock %}

到现在为止配置基本完成了，启动munin试下吧

{% codeblock lang:bash %}
/etc/init.d/munin-node restart
{% endcodeblock %}

然后打开浏览器访问`http://your.domain/munin/`，输入用户名和密码后就应可以看到munin的首页了，一开始是没有数据的，耐心等待几分钟后数据就会更新了。

参考文章

1. [Setting up Munin on Ubuntu](http://www.darkcoding.net/software/setting-up-munin-on-ubuntu/)
2. [Munin Documentation](http://munin-monitoring.org/wiki/Documentation)




















