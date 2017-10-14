---
layout: post
title: Linux学习笔记2
date: 2015-03-04 10:11 +0008
categories: Linux
---

### 1. 建立linux账号

需要在ROOT权限下执行

```
useradd -m username
```

新建一个用户名为`username`的账号

```
passwd username
```

为`username`用户创建密码

### 2. nfs server

 In Fedora, the nfs-utils package is required for full NFS support. Run the following command to see if the nfs-utils is installed:

```
~]$ rpm -q nfs-utils
package nfs-utils is not installed
```

If it is not installed and you want to use NFS, use the yum utility as root to install it:

```
~]# yum install nfs-utils
```
检查是否安装了nfs

        rpm -qa |grep nfs

    [kuaile@localhost ~]$ rpm -qa |grep nfs
    nfs-utils-1.2.8-6.0.fc20.x86_64
    libnfsidmap-0.25-7.fc20.x86_64

输出这就是已经安装了.

fedora 20 默认是已经安装的.

配置nfs

打开nfs配置文件exports

sudo vi /etc/exports

输入以下

	/home/kuaile 192.168.122.1(insecure,rw,sync,no_root_squash)

/home/kuaile       是共享的目录

192.168.122.1    是允许挂在该目录的主机的IP地址, 如果不能确定 ,请使用 * ,表示任意IP 都可以

insecure               是一个安全选项, 如果nfs服务端口号小于1024则可以不添加这个选项, 否则不添加的话, 是无法访问的.其他主机访问的话就会被拒绝.

rw                          是共享目录的权限,rw 是可读写的权限,只读的权限是ro.

sync                      是同步的选项, 可选的还有 async. sync是不使用缓存,随时写入同步, async是使用缓存的.

no_root_squash   是NFS 服务共享的目录的属性, 如果用户是root, 那么对这个目录就有root的权限.

如果有多个目录, 每个目录一行,可添加多个目录.
