---
layout: post
title: 学习一下norflash
date: 2015-06-18
categories: Embedded
---

最近公司项目遇到了两个关于Nor Flash的问题, 于是就学习了一下NorFlash是怎么操作的.

项目的平台是博通(boardcom)的, 不过应该nor的操作是通用而相似的.

首先是找了一个特别好的文章 ===> [Nor Flash](http://wiki.dzsc.com/info/5353.html)


文章讲的很清楚, 接下来讲下遇到的问题

### 1. flash设备的时序

>We have been able to track the problem and that’s caused by the fact that factory used slower flash devices on some production batch (100ns Vs 110ns).

遇到的问题是工厂换了一块和现在的IO速度不同的Nor Flash. 然后让我们看看是不是能够修改一下我们这边的设定

发现了可以提供修改的寄存器

### 2. 查看新的Nor Flash是否合适

用了两款新的Nor Flash, 但是不知道现有的代码是否支持?

主要的配置是有两种方式,

第一种方式, 是查询一个配置表, 里面把现在在产品上面用到的FLASH芯片的配置都列了出来

第二种方式, 是通过查询CFI信息, 获取芯片内部的配置, 关于CFI信息是什么东西

这里有份spansion的文档介绍  ====> [Quick Guide to Common Flash Interface](http://www.spansion.com/Support/Application%20Notes/Quick_Guide_to_CFI_AN.pdf)

然后还涉及到一个命令集的事情, 其实就是Nor 不仅仅有CFI命令, 还有自己的比如读数据, 写数据, 擦除数据的时候的指令集

之前提到的文章[Nor Flash](http://wiki.dzsc.com/info/5353.html)里面有详细的说法.

ok that is all
