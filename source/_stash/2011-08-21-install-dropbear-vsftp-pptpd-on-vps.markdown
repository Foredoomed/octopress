---
layout: post
title: "在VPS上安装dropbear,vsftp和pptpd"
date: 2011-08-21 17:31
comments: true
categories: vps
---
**编译安装Dropbear**

首先修改OpenSSH的端口,编辑`/etc/ssh/sshd_config`文件，找到`Port 22`这一行，并修改为`Port 2222`。

然后重启OpenSSH：

{% codeblock lang:bash %}
service sshd restart
{% endcodeblock %}

编译安装Dropbear:

{% codeblock lang:bash %}
cd /data/software
wget http://matt.ucc.asn.au/dropbear/dropbear-0.53.1.tar.gz
tar -zxvf dropbear-0.53.1.tar.gz
cd dropbear-0.53.1
./configure
make && make install
{% endcodeblock %}

生成公钥：

{% codeblock lang:bash %}
mkdir /etc/dropbear
/usr/local/bin/dropbearkey -t dss -f /etc/dropbear/dropbear_dss_host_key
/usr/local/bin/dropbearkey -t rsa -s 4096 -f /etc/dropbear/dropbear_rsa_host_key
{% endcodeblock %}

启动Dropbear：

{% codeblock lang:bash %}
/usr/local/sbin/dropbear start
{% endcodeblock %}

配置开机自动启动：

{% codeblock lang:bash %}
chkconfig --add dropbear
chkconfig dropbear on
{% endcodeblock %}

Dropbear正确安装完成后，如果能够用ssh登录，那么就可以删除OpenSSH了:

{% codeblock lang:bash %}
yum remove openssh
{% endcodeblock %}

<!--more-->

**编译安装vsftp**

如果编译安装的话会报错，要自己手动复制文件，所以我就从软件库自动安装了：

{% codeblock lang:bash %}
yum install vsftpd
{% endcodeblock %}

创建FTP用户：

{% codeblock lang:bash %}
/usr/sbin/groupadd ftp
/usr/sbin/useradd -g ftp ftp
{% endcodeblock %}

建立FTP默认目录:

{% codeblock lang:bash %}
mkdir /var/ftp
{% endcodeblock %}

设定FTP的home目录为/var/ftp：

{% codeblock lang:bash %}
useradd -d /var/ftp -s /sbin/nologin ftp
{% endcodeblock %}

设定home目录所有者和权限:

{% codeblock lang:bash %}
chown ftp:ftp /var/ftp
chmod 755 /var/ftp
{% endcodeblock %}

编辑vsftp配置文件`/etc/vsftpd/vsftpd.conf`,修改为：

{% codeblock lang:bash %}
anonymous_enable=no
listen=yes

//并在文件末尾添加：
local_root=/var/ftp/pub
use_localtime=yes
connect_timeout=60
accept_timeout=60
max_clients=10
max_per_ip=10
{% endcodeblock %}

配置开机自动启动
{% codeblock lang:bash %}
chkconfig --add vsftpd
chkconfig vsftpd on
{% endcodeblock %}

**安装pptpd**

1.首先用ssh登录VPS，检查VPS是否有必要的支持，否则将导致无法安装(如果是buyvm的VPS可以跳过这步)

{% codeblock lang:bash %}
modprobe ppp-compress-18 && echo ok
{% endcodeblock %}

用模块方式支持MPPE加密模式浏览，如果内核支持则检测不到。如果显示“ok”表明通过，不过还需要做另一个检查：

{% codeblock lang:bash %}
cat /dev/net/tun
{% endcodeblock %}

如果显示结果为下面的文本就表明通过：

{% codeblock lang:bash %}
cat: /dev/net/tun: File descriptor in bad state
{% endcodeblock %}

上述两条检测只需一条通过，即可安装pptpd。如果还有其他问题，就提Ticket给服务商替你解决。

2.安装ppp：

{% codeblock lang:bash %}
rpm -ivh http://poptop.sourceforge.net/yum/stable/rhel6/i386/ppp-2.4.5-17.0.rhel6.i686.rpm
{% endcodeblock %}

3.安装pptpd:

{% codeblock lang:bash %}
rpm -ivh http://poptop.sourceforge.net/yum/stable/rhel6/i386/pptpd-1.3.4-2.el6.i686.rpm
{% endcodeblock %}

4.配置pptpd:

{% codeblock lang:bash %}
vim /etc/pptpd.conf
{% endcodeblock %}

把下面字段前面的#去掉即可：

{% codeblock lang:bash %}
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
{% endcodeblock %}

接下来再编辑`/etc/ppp/options.pptpd`文件：

{% codeblock lang:bash %}
vim /etc/ppp/options.pptpd
{% endcodeblock %}

去掉`ms-dns`前面的#，并修改成如下字段：

{% codeblock lang:bash %}
ms-dns 8.8.8.8
ms-dns 8.8.4.4
{% endcodeblock %}

5.设置pptpd账号和密码：

编辑`/etc/ppp/chap-secrets`文件：

{% codeblock lang:bash %}
vim /etc/ppp/chap-secrets
{% endcodeblock %}

直接输入如下字段,username和password就是你要登录VPN的用户名和密码：

{% codeblock lang:bash %}
username pptpd password *
{% endcodeblock %}

6.修改内核设置，使其支持转发:

编辑`/etc/sysctl.conf`文件：

{% codeblock lang:bash %}
vim /etc/sysctl.conf
{% endcodeblock %}

做下面的修改：

{% codeblock lang:bash %}
net.ipv4.ip_forward=1
# net.ipv4.tcp_syncookies = 1  //注释这行
{% endcodeblock %}

保存退出，并执行下面的命令来使它生效：

{% codeblock lang:bash %}
sysctl -p
{% endcodeblock %}

7.添加iptables转发规则：

{% codeblock lang:bash %}
iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source xxx.xxx.xxx.xxx  //xxx.xxx.xxx.xxx为你的VPS的公网IP地址
{% endcodeblock %}

保存iptables转发规则：

{% codeblock lang:bash %}
/etc/init.d/iptables save
{% endcodeblock %}

重启iptables：

{% codeblock lang:bash %}
/etc/init.d/iptables restart
{% endcodeblock %}

8.重启pptpd服务：

{% codeblock lang:bash %}
/etc/init.d/pptpd restart
{% endcodeblock %}

9.设置开机自动启动

{% codeblock lang:bash %}
chkconfig --add pptpd
chkconfig pptpd on
{% endcodeblock %}

