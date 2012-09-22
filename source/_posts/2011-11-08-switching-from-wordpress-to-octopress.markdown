---
layout: post
title: "告别Wordpress,拥抱Octopress"
date: 2011-11-08 22:49
comments: true
categories: octopress
---
至今为止我开博有差不多1年多时间了，期间一直是用Wordpress搭建的博客系统。Wordpress虽然是目前最流行的博客系统，但是如果要在自己的VPS上搭建Wordpress环境还是有一定难度的(我当初搭建的时候就花了蛮长的时间，搭建完了还要优化)。当然最主要的还是用下来有许多的不方便的地方：

* 因为不懂PHP，所以修改功能起来比较麻烦
* 对“<”等特殊符号需要手动转义
* 需要安装很多插件才能满足需要
* Nginx+PHP+mySql的组合在小内存的VPS上内存占用率过高(在我256MB内存的VPS上达到了30%)

直到上个星期在Google Reader上看到有人提到[Octopress]{http://octopress.org/ "Octopress"}这个用Ruby写的博客系统后，就一下子被它的简单轻量所吸引(只要一个Web服务器就行了，不需要mySql)，最终要的是我会Ruby，所以我毫不犹豫的决定从Wordpress转向Octopress。而且在Octopress的主页上还写着这样一句话：A blogging framework for hackers.恩，我就是个hacker，哈哈。

整个安装过程还是比较简单的，不过在安装之前要先把之前的博客导出来(不管是从后台还是数据库导)，然后就可以按照[Octopress Setup](http://octopress.org/docs/setup/ "Octopress Setup")来安装了。注意，如果没有装过Ruby的话，那么在安装完[RVM](http://beginrescueend.com/ "RVM")后需要安装Ruby的依赖库：

{% codeblock lang:bash %}
yum install -y gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison iconv-devel
{% endcodeblock %}

git的话只需要运行命令：

{% codeblock lang:bash %}
yum install git-core
{% endcodeblock %}

我的VPS的操作系统是CentOS 6，并且我把预装的Apache给卸载了，改用Nginx。顺便提一下，在编译Nginx的时候如果碰到编译错误，那就试试下面这句命令行吧：

{% codeblock lang:bash %}
yum install pcre-devel
{% endcodeblock %}

然后修改Nginx的配置文件nginx.conf
{% codeblock %}

user  root;
worker_processes  1;

error_log  logs/error.log crit;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

pid        logs/nginx.pid;

worker_rlimit_nofile 65535;

events {
    use epoll;
    worker_connections  65535;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    server_names_hash_bucket_size 128;
    large_client_header_buffers 4 32k;
    client_header_buffer_size 32k; 
    client_max_body_size 8m;
      
    sendfile on;
    keepalive_timeout 10 10;
    tcp_nopush on;
    tcp_nodelay on;
    
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 32k;
    fastcgi_buffers 4 32k;
    fastcgi_busy_buffers_size 32k;
    fastcgi_temp_file_write_size 32k;
    fastcgi_intercept_errors on;

    gzip  on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 1;
    gzip_types       text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    server {
        listen       80;
        server_name  liuxuan.info www.liuxuan.info;
        
        log_format  access  '$remote_addr - $remote_user [$time_local] "$request" '
                            '$status $body_bytes_sent "$http_referer" '
                            '"$http_user_agent" $http_x_forwarded_for';
        
        access_log  logs/host.access.log  access;

        location / {
            root   /path/to/octopress/public;
            index  index.html;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
  
    }

}
{% endcodeblock %}

创建Nginx自动启动脚本
{% codeblock lang:bash %}
vim /etc/init.d/nginx
{% endcodeblock %}

然后加入下面的内容
{% codeblock lang:bash %}
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemin
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:     /usr/local/nginx/logs/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
{% endcodeblock %}

添加可执行权限
{% codeblock lang:bash %}
chmod +x /etc/init.d/nginx
{% endcodeblock %}

配置开机自启动
{% codeblock lang:bash %}
chkconfig nginx on
{% endcodeblock %}

安装完成后顺便对CentOS做一下优化，用[syslog-ng](http://www.balabit.com/network-security/syslog-ng/opensource-logging-system "syslog-ng")替换rsyslog，删除不需要的进程。然后调整Linux的最大文件打开数：

{% codeblock lang:bash %}
vim /etc/security/limits.conf
在最后一行添加
* soft nofile 65535
* hard nofile 65535
{% endcodeblock %}

整个搭建过程在30分钟之内就可以搞定，大大低于搭建Wordpress环境所需要的时间。而且通过top命令可以看到由于只有一个Nginx进程，所以整个内存占用率在4%左右，大大低于Wordpress的30%，这点对于小内存的VPS是非常有优势的。

