---
layout: post
title: Study of Linux Driver
date: 2015-02-17 00:00 +0008
categories: Blog
---

今天开始复习?
学习Linux设备驱动, 就是为了新项目嘛..

大概项目要用到的就只有最简单的字符设备就好.

所以慢慢学


## 1. Hello world模块

printf 不能用, 要用printk

**用户空间和内核空间**

内核空间是最高用户态, 拥有最多的权限可以进行所有的操作.
用户空间是运行在最低用户态, 处理器控制着对硬件的直接访问以及对内存的非授权访问.

每当应用程序执行系统调用或者被硬件中断挂起时, Unix将执行模式从用户空间切换到内核空间.

内核中的并发要注意, 所有驱动程序可能被很多个程序调用.

**当前进程查看:**

1. `#include <linux/sched.h>`
2. 'printk(KERN_INFO "The process is %s (pid %i)", current->comm, current->pid);'


双下划线是接口的底层组件. 最好不要修改


## 2. 编译和装载
