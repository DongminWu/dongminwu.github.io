---
layout: post
title: Platform Builder 恢复调试方法
date: 2015-03-09
categories: wince
---

转: [platform builder was unable to read the device settings from the datastore问题解决](http://blog.csdn.net/wirror800/article/details/4087968)




本文章主要讲的是当安装完CE6后，会发现Platform Builder for CE5的Connectivity Options不好用了，即使修改设置后点Apply也无法把新设置保存住在完全卸载CE6和VS2005后，点PB5的Connectivity Options会弹出这个错误： Platform Builder was unable to read the device settings from the Datastore.

即使把CE5和PB5完全卸载，然后再重装，改错误仍然存在。

 

方案一：

1、关闭pb的所有进程，确保Cepb.exe, CeSvcHost.exe (CeSvcH~1.exe) and DeviceEmulator.exe 都已停止运行。

2、备份C:/Documents and Settings/<user>/local settings/application data/microsoft/corecon文件夹。(可以不用备份，直接删除)

3、备份C:/Documents and Settings/all users/application data/microsoft/corecon/1.0/addons文件夹。(可以不用备份，直接删除)

4、在C:/Documents and Settings/all users/application data/microsoft/corecon/1.0目录下创建一个addons文件夹。

5、将C:/Program Files/<PB Install folder>/CoreCon/SDK/XSL/Addons目录下的所有.xsl问价复制到第四步中新建的文件夹下。

6、将C:/Program Files/<PB Install folder>/CEPB/bin/microsoft.platformbuilder500.servicecategory.xsl也复制到第四步中新建的文件夹下。

 

注意：你可能找不到/local settings目录，需要在文件夹选项中选择不隐藏系统文件即可。

 

方案二：

1、在命令行中进入到你的platform目录

如：C:/Program Files/Windows CE Platform Builder/5.00/CORECON/SCRIPTS/

2、依次运行以下命令：

disablecorecon <path>
unregister <path>
register <path>
enablecorecon <path>

 

注意：这里的path格式应该为"C:/Program Files/Windows CE Platform Builder/5.00"，别忘了加引号。

 

总结：

    第一种方法我试验成功了，第二种方法不知道为什么没成功，但大家也可以借鉴一下

pipicold: 第二种方法我实验成功了, 我的case是, 不知道为什么我的Documents and Settings/user里面的文件被重置了, 然后直接用第二个方法就ok了
