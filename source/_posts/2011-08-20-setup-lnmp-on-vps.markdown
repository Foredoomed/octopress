---
layout: post
title: "在VPS上搭建LNMP(Linux/Nginx/MySQL/PHP)"
date: 2011-08-20 17:12
comments: true
categories: vps
---
**一.写在最前**

从我开博到现在也有半年有余了，之前一直是放在一个共享空间里。当初选择共享空间主要是因为价格便宜，而且我只在空间里放一个WordPress，不需要很大的空间和流量。但是在这半年多时间里，这个空间经常出现响应速度变得很慢，FTP和CPANEL无法登录等问题，真让我的忍耐到达了极点；再加上我们不知道什么原因访问一些网站经常返回错误页面，所以我决定抛弃这个空间(一年的时间还没到，浪费劳资的钱啊。。。)，转投VPS的怀抱。 <!--more-->

其实在几年之前就知道VPS这样东西了，但是真正当我要去搞一个的时候，却发现有太多的选择。经过一番思想斗争，我最终选择了<a href="http://buyvm.net/" title="buyvm">buyvm</a>这个VPS。如果你要问我为什么选择这个VPS，我想有三个原因：1.便宜 2.便宜 3.还是便宜。谁不想要被称为最好的VPS之称的<a href="http://www.linode.com/" title="Linode">Linode</a>啊，但是穷逼只能捡便宜的用。当然，并不是便宜的就不好，buyvm的VPS虽然便宜，质量和服务也是非常好的，并且它还在<a href="http://www.lowendbox.com" title="lowendbox">lowendbox</a>的评选中获得过第三名的好成绩。

重要的来了，你需要搞清楚在VPS上放些什么东西，因为这关系着你选OpenVZ还是Xen。提前告知一下：**如果你想在VPS上装L2TP/IPSec VPN的话，就不要选OpenVZ**。以我的经验来看，OpenVZ还是有很多限制的，如果手头宽裕的话还是首选Xen吧。好了，扯蛋完毕，开工搭建。

**二.安装LNMP+Dropbear+vsFTP**

本次搭建必要的是：
<ul>
<li>1台VPS</li>
<li>CentOS操作系统(我的是CentOS 5.6，其他Linux发行版应该差不多)</li>
<li>能够以root权限从SSH登录</li>
<li>有足够的耐心和时间，因为接下来的工作还是蛮多的</li>
</ul>

**1.安装必要的软件包**

首先以root权限从SSH登录，然后运行下面的命令：

{% codeblock lang:bash %}
yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers
{% endcodeblock %}

**2.在home目录新建一个software目录用来存放这次搭建所需的软件：**

{% codeblock lang:bash %}
cd ~
mkdir software
cd software
{% endcodeblock %}

然后下载我们需要的软件：

{% codeblock lang:bash %}
wget http://nginx.org/download/nginx-1.0.5.tar.gz
wget http://www.php.net/get/php-5.3.7.tar.gz/from/a/mirror
wget http://dev.mysql.com/downloads/mirror.php?id=403104#mirrors
wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
wget http://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.bz2/download
wget http://sourceforge.net/projects/mcrypt/files/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz/download
wget http://pecl.php.net/get/memcache-3.0.6.tgz
wget http://sourceforge.net/projects/mhash/files/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz/download
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/
wget http://pecl.php.net/get/APC-3.1.9.tgz
wget http://pecl.php.net/get/PDO_MYSQL-1.0.2.tgz
wget ftp://ftp.imagemagick.org/pub/ImageMagick/ImageMagick.tar.gz
wget http://pecl.php.net/get/imagick-3.0.1.tgz
{% endcodeblock %}

**3.编译安装MySQL**

首先添加mysql用户组和用户：

{% codeblock lang:bash %}
/usr/sbin/groupadd mysql
/usr/sbin/useradd -g mysql mysql
{% endcodeblock %}

编译安装mysql：

{% codeblock lang:bash %}
tar zxvf mysql-5.1.58.tar.gz
cd mysql-5.1.58
./configure --prefix=/usr/local/webserver/mysql/ --localstatedir=/usr/webserver/local/mysql/data --with-unix-socket-path=/tmp/mysql.sock --with-charset=utf8 --with-collation=utf8_general_ci --enable-assembler --with-extra-charsets=complex --enable-thread-safe-client --with-big-tables --with-readline --with-ssl --with-embedded-server --enable-local-infile --with-client-ldflags=-all-static --with-mysqld-ldflags=-all-static --with-plugins=partition,innobase,myisammrg
make && make install
cp support-files/my-medium.cnf /usr/local/webserver/mysql/my.cnf
{% endcodeblock %}

赋予mysql权限：

{% codeblock lang:bash %}
chmod +w /usr/local/webserver/mysql
chown -R mysql:mysql /usr/local/webserver/mysql
{% endcodeblock %}

以mysql用户建立数据库表：

{% codeblock lang:bash %}
/usr/local/webserver/mysql/bin/mysql_install_db --basedir=/usr/local/webserver/mysql --datadir=/usr/local/webserver/mysql/data --user=mysql
{% endcodeblock %}

修改mysql的配置文件为：

{% codeblock lang:bash %}
[client]
character-set-server = utf8
port		= 3306
socket		= /tmp/mysql.sock

[mysqld]
character-set-server = utf8
port		= 3306
socket		= /tmp/mysql.sock
skip-locking
key_buffer_size = 16M
max_allowed_packet = 16M
table_open_cache = 64
sort_buffer_size = 128K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M

basedir = /usr/local/webserver/mysql
datadir = /usr/local/webserver/mysql/data
open_files_limit = 600
back_log = 20
max_connections = 100
max_connect_errors = 200
table_cache = 60
external-locking = FALSE
join_buffer_size = 128K
thread_cache_size = 10
thread_concurrency = 8
query_cache_size = 0M
query_cache_limit = 2M
query_cache_min_res_unit = 2k
default_table_type = MyISAM
thread_stack = 192K
tmp_table_size = 512K
max_heap_table_size = 32M
long_query_time = 1
log_long_format
binlog_cache_size = 2M
max_binlog_cache_size = 4M
max_binlog_size = 512M
expire_logs_days = 7

[myisamchk]
key_buffer_size = 4M
sort_buffer_size = 1M
read_buffer = 2M
write_buffer = 2M

read_rnd_buffer_size = 2M
bulk_insert_buffer_size = 2M
myisam_sort_buffer_size = 4M
myisam_max_sort_file_size = 10G
myisam_max_extra_sort_file_size = 10G
myisam_repair_threads = 1
myisam_recover
{% endcodeblock %}

创建一个脚本，方便管理mysql：

{% codeblock lang:bash %}
vim /usr/local/webserver/mysql/mysql
{% endcodeblock %}

在其中添加下面的脚本命令：

{% codeblock lang:bash %}
#!/bin/sh

function_start_mysql()
{
   printf "Starting MySQL...\n"
   /bin/sh /usr/local/webserver/mysql/bin/mysqld_safe --defaults-file=/usr/local/webserver/mysql/my.cnf &
}

function_stop_mysql()
{
   printf "Stopping MySQL...\n"
   /usr/local/webserver/mysql/bin/mysqladmin -u mysql -pmysql shutdown
}

function_restart_mysql()
{
   printf "Restarting MySQL...\n"
   function_stop_mysql
   sleep 5
   function_start_mysql
}

function_kill_mysql()
{
   kill -9 $(ps -ef | grep 'bin/mysqld_safe' | grep 3306 | awk '{printf $2}')
   kill -9 $(ps -ef | grep 'libexec/mysqld' | grep 3306 | awk '{printf $2}')
}

if [ "$1" = "start" ]; then
function_start_mysql
elif [ "$1" = "stop" ]; then
function_stop_mysql
elif [ "$1" = "restart" ]; then
function_restart_mysql
elif [ "$1" = "kill" ]; then
function_kill_mysql
else
printf "Usage: /usr/local/webserver/mysql {start|stop|restart|kill}\n"
fi
{% endcodeblock %}

然后赋予脚本执行权限：

{% codeblock lang:bash %}
chmod +x /usr/local/webserver/mysql/mysql
{% endcodeblock %}

启动mysql：

{% codeblock lang:bash %}
/usr/local/webserver/mysql/mysql start
{% endcodeblock %}

给mysql用户赋予数据库的所有权限：

{% codeblock lang:bash %}
GRANT ALL PRIVILEGES ON *.* TO 'mysql'@'localhost' IDENTIFIED BY 'mysql';
GRANT ALL PRIVILEGES ON *.* TO 'mysql'@'127.0.0.1' IDENTIFIED BY 'mysql';
{% endcodeblock %}

配置开机自启动mysql

{% codeblock lang:bash %}
vim /etc/rc.local
/usr/local/webserver/mysql/bin/mysqld_safe --defaults-file=/usr/local/webserver/mysql/my.cnf &
{% endcodeblock %}

**4.编译安装PHP**

首先创建一个用户和组(用来管理PHP/Nginx/Wordpress)
创建用户和组

{% codeblock lang:bash %}
groupadd www
useradd -g www www
mkdir -p /data/htdocs/blog  //blog目录是Wordpress的根目录
chmod +w /data/htdocs/blog
chown -R www:www /data/htdocs/blog
{% endcodeblock %}

编译安装PHP所需的支持库：

{% codeblock lang:bash %}
tar zxvf libiconv-1.14.tar.gz
cd libiconv-1.13.1/
./configure --prefix=/usr/local
make && make install

tar zxvf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8/
./configure
make && make install
/sbin/ldconfig
cd libltdl
./configure --enable-ltdl-install
make && make install

tar zxvf mhash-0.9.9.9.tar.gz
cd mhash-0.9.9.9/
./configure
make && make install

tar zxvf mcrypt-2.6.8.tar.gz
cd mcrypt-2.6.8/
/sbin/ldconfig
./configure
make && make install

ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la
ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so
ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4
ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8
ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a
ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la
ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so
ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2
ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1
ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config
{% endcodeblock %}
{% codeblock lang:bash %}
tar zxvf php-5.3.7.tar.gz
cd php-5.3.7
./configure --prefix=/usr/local/webserver/php --with-config-file-path=/usr/local/webserver/php/etc --with-mysql=/usr/local/webserver/mysql --with-mysqli=/usr/local/webserver/mysql/bin/mysql_config --with-iconv-dir=/usr/local --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath  --enable-safe-mode --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --with-curlwrappers --enable-mbregex  --enable-fpm  --enable-mbstring --with-mcrypt --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-ldap --with-ldap-sasl --with-xmlrpc --enable-ftp --enable-zip --enable-exif --enable-soap --without-pear --with-fpm-user=www --with-fpm-group=www
make ZEND_EXTRA_LIBS='-liconv'
{% endcodeblock %}

如果遇到下面的问题：

{% codeblock lang:bash %}
/root/source/php-5.3.7/sapi/cli/php: error while loading shared libraries: libmysqlclient.so.18: cannot open shared object file: No such file or directory
make: *** [ext/phar/phar.php] Error 127
{% endcodeblock %}

解决方法：

{% codeblock lang:bash %}
ln -s /usr/local/webserver/mysql/lib/libmysqlclient.so.18 /usr/lib/
{% endcodeblock %}

然后再执行：

{% codeblock lang:bash %}
make install
cp php.ini-production /usr/local/webserver/php/etc/php.ini
{% endcodeblock %}

安装PHP扩展包：

{% codeblock lang:bash %}
tar zxvf memcache-3.0.6.tgz
cd memcache-3.0.6
/usr/local/webserver/php/bin/phpize
./configure --with-php-config=/usr/local/webserver/php/bin/php-config
make && make install

tar jxvf APC-3.1.9.tgz
cd APC-3.1.9
/usr/local/webserver/php/bin/phpize
./configure --with-php-config=/usr/local/webserver/php/bin/php-config --prefix=/usr/local/webserver/ --enable-apc --enable-apc-mmap --enable-apc-spinlocks
make && make install

tar zxvf PDO_MYSQL-1.0.2.tgz
cd PDO_MYSQL-1.0.2
/usr/local/webserver/php/bin/phpize
./configure --with-php-config=/usr/local/webserver/php/bin/php-config --with-pdo-mysql=/usr/local/webserver/mysql
make && make install

tar zxvf ImageMagick.tar.gz
cd ImageMagick-6.7.1-7
./configure
make && make install

tar zxvf imagick-3.0.1.tgz
cd imagick-3.0.1
/usr/local/webserver/php/bin/phpize
./configure --with-php-config=/usr/local/webserver/php/bin/php-config
make && make install
{% endcodeblock %}

修改php.ini文件:

{% codeblock lang:bash %}
output_buffering = On
cgi.fix_pathinfo = 0

extension_dir = "/usr/local/webserver/php/lib/php/extensions/no-debug-non-zts-20090626/"

//并在此行后增加以下几行
extension = "memcache.so"
extension = "pdo_mysql.so"
extension = "imagick.so"

[APC]
apc.enabled = 1
apc.shm_segments = 1
apc.shm_size = 8M
apc.ttl = 7200
apc.user_ttl = 7200
apc.optimization = 1
apc.num_files_hint = 1024
apc.mmap_file_mask =/tmp/apc.XXXXXX 
apc.enable_cli = 1
{% endcodeblock %}

打开php-fpm.conf文件:

{% codeblock lang:bash %}
vim /usr/local/webserver/php/etc/php-fpm.conf
{% endcodeblock %}

修改添加如下配置：

{% codeblock lang:bash %}
pm.max_children = 1
pid = run/php-fpm.pid
error_log = log/php-fpm.log
log_level = notice
emergency_restart_threshold :10
emergency_restart_interval:1m
process_control_timeout:5s
daemonize:yes

backlog:-1
listen.owner = www
listen.group = www
listen.mode = 0666
user = www
group = www
pm = static
request_terminate_timeout = 0
request_slowlog_timeout = 0s
slowlog = log/$pool.log.slow
rlimit_files = 51200
rlimit_core = 0
catch_workers_output = yes
pm.max_requests = 500
listen.allowed_clients = 127.0.0.1
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
env[OSTYPE] = $OSTYPE
env[MACHTYPE] = $MACHTYPE
env[MALLOC_CHECK_] = 2
{% endcodeblock %}

启动php-fpm：

{% codeblock lang:bash %}
/usr/local/webserver/php/sbin/php-fpm
{% endcodeblock %}

配置开机自动启动php-fpm:

{% codeblock lang:bash %}
vim /etc/rc.local
/usr/local/webserver/php/sbin/php-fpm
{% endcodeblock %}

**5.编译安装Nginx**
首先编译安装Nginx所需的pcre库：

{% codeblock lang:bash %}
tar zxvf pcre-8.12.tar.gz
cd pcre-8.12
./configure
make && make install
{% endcodeblock %}

编译安装Nginx:

{% codeblock lang:bash %}
tar zxvf nginx-1.0.5.tar.gz
cd nginx-1.0.5
./configure --user=www --group=www --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module
make && make install
{% endcodeblock %}

创建Nginx日志目录:

{% codeblock lang:bash %}
mkdir -p /data/log
chmod +w /data/log
chown -R www:www /data/log
{% endcodeblock %}

修改Nginx配置文件:

{% codeblock lang:bash %}
user  www www;

worker_processes 1;

error_log  /data/log/nginx_error.log  crit;

pid        /usr/local/webserver/nginx/nginx.pid;

#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 65535;

events
{
  use epoll;
  worker_connections 65535;
}

http
{
  include       mime.types;
  default_type  application/octet-stream;

  #charset  gb2312;
      
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

  gzip on;
  gzip_min_length  1k;
  gzip_buffers     4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 1;
  gzip_types       text/plain application/x-javascript text/css application/xml;
  gzip_vary on;

  #limit_zone  crawler  $binary_remote_addr  10m;
  

  server
  {
    listen       80;
    server_name  liuxuan.info www.liuxuan.info;
    index index.html index.htm index.php;
    root  /data/htdocs/blog;
    
    #limit_conn   crawler  20;
    
    location / {          
       try_files $uri $uri/ /index.php?q=$uri&$args;    
    }

    # if file exists return it right away
    if (-f $request_filename) {      
       break;
    }
 
    # otherwise rewrite the request
    if (!-e $request_filename) {
       rewrite ^(.+)$ /index.php$1 last;
       break;
    }    
    
     
    location ~ .*\.(php|php5)?$
    {      
      fastcgi_pass  unix:/tmp/php-cgi.sock;
      #fastcgi_pass  127.0.0.1:9000;
      fastcgi_index index.php;
      include fastcgi.conf;

    }
    
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
      access_log off;
      expires      30d;
    }

    location ~ .*\.(js|css)?$
    {
      access_log off;
      expires      1d;
    }    
    
    # Only allow search engine to access pics
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv)$
    {
	valid_referers none blocked *.liuxuan.info *.google.com *.baidu.com;
	if ($invalid_referer)
	{
	   return 403;
	}
    }
  

    log_format  access  '$remote_addr - $remote_user [$time_local] "$request" '
              '$status $body_bytes_sent "$http_referer" '
              '"$http_user_agent" $http_x_forwarded_for';
    access_log  /data/log/access.log  access;
  }

 
}
{% endcodeblock %}

检查Nginx配置是否正确：

{% codeblock lang:bash %}
/usr/local/webserver/nginx/sbin/nginx -t
{% endcodeblock %}

启动Nginx：

{% codeblock lang:bash %}
/usr/local/webserver/nginx/sbin/nginx
{% endcodeblock %}

配置开机自动启动Nginx：

{% codeblock lang:bash %}
vim /etc/rc.local
/usr/local/webserver/nginx/sbin/nginx
{% endcodeblock %}

在不停止Nginx服务的情况下平滑变更Nginx配置：

{% codeblock lang:bash %}
kill -HUP `cat /usr/local/webserver/nginx/nginx.pid`
{% endcodeblock %}

**6.Linux设置：**

修改服务器时间为上海时间：

{% codeblock lang:bash %}
cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
{% endcodeblock %}

关闭不需要的服务：

{% codeblock lang:bash %}
ntsysv  //按空格选择和取消，按F12退出
{% endcodeblock %}

以下仅列出需要启动的服务，未列出的服务一律关闭：

{% codeblock lang:bash %}
crond
iptables
network
syslog
vsftpd
{% endcodeblock %}

**四.总结**

回顾这段搭建的过程可谓历经挫折和痛苦，其中碰到了很多问题，为此我还到StackOverflow上发贴求助，但是我还是坚持了下来，直到搭建成功，所以说坚持和永不放弃的精神是非常重要的。搭建完成后，通过top命令发现虽然只有19个进程，但是内存占用还是蛮高的，特别是系统启动后需要占用的内存。好了，希望能给看到这篇博文的朋友带来帮助，缩短他们搭建的时间。

参考文章：

[1] [http://blog.s135.com/nginx_php_v6/](http://blog.s135.com/nginx_php_v6/ "http://blog.s135.com/nginx_php_v6/")  
[2] [http://www.fallday.org/archives/551](http://www.fallday.org/archives/551 "http://www.fallday.org/archives/551")

