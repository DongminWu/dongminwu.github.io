---
layout: post
title: Lua语言中pairs和ipairs的区别
date: 2014-09-01 00:00 +0008
categories: Coding
tags: Lua
---

tbl = {"alpha", "beta", ["one"] = "uno", ["two"] = "dos"}

    for key, value in ipairs(tbl) do

            print(key, value)

    end


-pairs()函数基本和ipairs()函数用法相同， 区别在于：

pairs()可以遍历整个table，即包括数组及非数组部分。

-->如有pairs迭代输出如下：

-->1 alpha

-->2 beta

-->one uno

-->two dos


ipairs()函数用于遍历table中的数组部分。

-->如有ipairs迭代输出如下：

-->1 alpha

-->2 beta


###&&&

将键值对插入一个表就直接在空表里面符值就好

###关于串口监听的问题

原来的想法是用串口发送AT指令什么的。
现在暂时取消，采用单纯web来控制，以后再添加。先以完成demo为主
