---
layout: post
title: linux 用户组配置, 挂载新硬盘
date: 2015-05-28
categories: Linux
---


今天上班没事做, 一点事都没...

于是把自己的笔记搬上来

### 建立Liunx账号

需要在ROOT权限下执行

    useradd -m username

新建一个用户名为username的账号

    passwd username

为username用户创建密码


### Samba 服务

[Fedora Samba配置文档](http://docs.fedoraproject.org/en-US/Fedora/21/html/System_Administrators_Guide/ch-File_and_Print_Servers.html#samba-rgs-overview)

1. 改/etc/samba/smb.conf 文件, 在最下面加上
```
[dongmwu]
path = /home/dongmwu
writeable = yes
public = yes
guest ok = no
```

2. 设置密码
```
smbpasswd -a username
```

3. 重启samba服务
```
systemctl restart smb.service
```
4. 记住关防火墙!!!
```
systemctl stop firewalld.service
```


### 修改用户权限

用到的是`chgrp`和`chown`这两个命令

一开始是这样的

```
/home1                                               
 $ ll                                                
total 20K                                            
drwxr-xr-x. 2 root root 4.0K May 27 10:11 dongmwu    
drwx------. 2 root root  16K May 26 23:29 lost+found
```

用 chgrp修改用户组

```
/home1                                                   
 $ sudo chgrp -R dongmwu dongmwu                       
/home1                                                   
 $ ll                                                 
total 20K                                                
drwxr-xr-x. 2 root dongmwu 4.0K May 27 10:11 dongmwu     
drwx------. 2 root root     16K May 26 23:29 lost+found  
```


用chown修改用户

```
/home1                                                           
 $ sudo chown -R dongmwu dongmwu                       [10:15:21]
/home1                                                           
 $ ll                                                  [10:15:27]
total 20K                                                        
drwxr-xr-x. 2 dongmwu dongmwu 4.0K May 27 10:11 dongmwu          
drwx------. 2 root    root     16K May 26 23:29 lost+found       
```
