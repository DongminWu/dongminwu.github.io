---
layout: post
title: Linux学习笔记1
date: 2015-02-28 10:22 +0008
categories: Linux
---


这两天都在配置编译服务器,

### 1. 服务器的IP地址的问题

虚拟机的IP地址的问题搞定了.
修改/etc/sysconfig/network-scripts/ifcfg-ens32

```
TYPE="Ethernet"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
NAME="ens32"
UUID="476287bc-8643-404f-984e-8fd719a2832f"
ONBOOT="yes"
HWADDR="00:50:56:8C:6E:82"
PEERDNS="yes"
PEERROUTES="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPADDR=10.79.183.125
NETMASK=255.255.255.0
GATEWAY=10.79.183.1
```
又遇到了一个dns的问题

在这个文件里面还需要添加

```
DNS1="64.104.123.144"
DNS2="171.70.168.183"
```


### 2. Sudo权限设置.

主要是看了一个blog [How To Edit the Sudoers File on Ubuntu and CentOS](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos)

/etc/sudoers里面  

`root        ALL=(ALL:ALL) ALL`

这一行下面添加

`demo        ALL=(ALL:ALL) ALL`

就ok

### 3. 64位系统的交叉编译环境 arm-linux-gcc 不能使用的问题.

有两个库找不到, `libz.so.1`和`ld-linux.so.2`

google了一下, 问题是出在这个服务器是64位的, 有些32位的lib没有

```
yum install glibc.i686
yum install zlib.i686
```


### 4. fedora 设置环境变量

修改/etc/profile文件

```
PATH=/opt/tool:$PATH
export PATH
```


立即执行 环境变量

```
. /etc/profile
```

注意 点 `.` 和 `/` 之间有个空格


### 5. tftp Server on Fedora 21

这个弄了我好久.

tftpServer的本质是用tftpd来作为server

但是可以用不同的服务来启动.

Fedora 18的文档是这样写的.

>
On the DHCP server, verify that the tftp-server package is installed with the command rpm -q tftp-server.
tftp is an xinetd-based service. Configure xinetd to process tftp requests by editing /etc/xinetd.d/tftp so that disable = no.
Start the tftp service with the following commands:
```
systemctl start xinetd.service
systemctl enable xinetd.service
```
These commands configure the tftp and xinetd services to immediately turn on and also configure them to start at boot.

但是到了Fedora 21的文档就变成了

>
1. Install the tftp server package.
    yum install tftp-server
2. Start and enable the tftp socket. systemd will automatically start the tftpd service when required.
    systemctl start tftp.socket
    systemctl enable tftp.socket

恩其实是两个不同的服务调用了同一个程序: /usr/sbin/in.tftpd

这个程序的帮助: man in.tftpd

#### xinet

配置文件位置: /etc/xinet.d/tftp   没有的话就自己建立一个
配置如下:

```
service tftp
{
	disable		=	no
	socket_type	=	dgram
	protocol	=	udp
	wait		=	yes
	user		=	root
	server		=	/usr/sbin/in.tftpd
	server_args	=	-s /tftpboot
	per_source	=	11
	csp		    =	100 2
	flags		=	IPv4
}
```

注意的是:

1. server_args 这一项是 in.tftpd的参数, 意思是把/tftpboot作为根目录
2. flags=IPv4是采用IPv4的端口,否则默认用的是IPv6的.


##### tftp.service / tftp.socket

配置文件位置: `/usr/lib/systemd/system/tftp.service`
内容:

```
[Unit]
Description=Tftp Server
Requires=tftp.socket
Documentation=man:in.tftpd

[Service]
ExecStart=/usr/sbin/in.tftpd -s /tftpboot
StandardInput=socket

[Install]
Also=tftp.socket
```

IPV4的配置在 `/usr/lib/systemd/system/tftp.service`
要配置成这样:

```
[Unit]
Description=Tftp Server Activation Socket

[Socket]
ListenDatagram=0.0.0.0:69

[Install]
WantedBy=sockets.target
```

重要的是ListenDatagram这一项

**配置成功!!!!**

要把selinux配置成自由模式(放纵模式?)

`setenforce 0`

linux的帮助这样写的

>
setenforce(8)                               SELinux Command Line documentation                              setenforce(8)
NAME
       setenforce - modify the mode SELinux is running in
SYNOPSIS
       setenforce [Enforcing|Permissive|1|0]
DESCRIPTION
       Use Enforcing or 1 to put SELinux in enforcing mode.
       Use Permissive or 0 to put SELinux in permissive mode.
       If  SELinux  is  disabled  and you want to enable it, or SELinux is enabled and you want to disable it, please see
       selinux(8).
AUTHOR
       Dan Walsh, <dwalsh@redhat.com>
SEE ALSO
       selinux(8), getenforce(8), selinuxenabled(8)
dwalsh@redhat.com                                      7 April 2004                                         setenforce(8)


##### 查看SELinux状态：

1、/usr/sbin/sestatus -v      ##如果SELinux status参数为enabled即为开启状态

SELinux status:                 enabled

2、getenforce                 ##也可以用这个命令检查

##### 关闭SELinux：

1、临时关闭（不用重启机器）：

```bash
setenforce 0                  ##设置SELinux 成为permissive模式

                              ##setenforce 1 设置SELinux 成为enforcing模式
```

2、修改配置文件需要重启机器：

修改/etc/selinux/config 文件

将SELINUX=enforcing改为SELINUX=disabled


#### 服务的启动

命令

```
systemctl status xinetd.service  //查看状态

systemctl start xinetd.service   //启动

systemctl stop xinetd.service    //停止

systemctl restart xinetd.service //重启

systemctl enable xinetd.service  //开机自启动

```

####  Samba 服务

[Fedora Samba配置文档](http://docs.fedoraproject.org/en-US/Fedora/21/html/System_Administrators_Guide/ch-File_and_Print_Servers.html#samba-rgs-overview)

**1. 改/etc/samba/smb.conf 文件**

在最下面加上

```
[dongmwu]
path = /home/dongmwu
writeable = yes
public = yes
guest ok = no
```

**2. 设置密码**

`smbpasswd -a username`

**3. 重启samba服务**

`systemctl restart smb.service`


**4. 记住关防火墙!!!**

`systemctl stop firewalld.service`
