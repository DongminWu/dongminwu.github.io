---
layout: post
title: Some notes about Linux Device Tree
date: 2016-03-17
categories: Linux
---

# Part 1 Kernel 相关文档

找到kernel的这个文档:

[https://www.kernel.org/doc/Documentation/devicetree/bindings/mtd/partition.txt]()

这里面说的很清楚怎么给nand分区


# Part 2 关于 address-cells, size-cells 是什么意思


这得从 `reg` 字串开始说起,

 `reg` 字串其实就是一个用来存某个空间的首地址和空间大小的存储器

但是呢, 受制于32bits的兼容问题, 所以首地址和空间大小必须以8byte=32bits的大小来存储

这就像英美数字表示法里面的逗号comma, 如果是999以下的数字用英美数字表示法就是纯粹的999, 但是大于等于1000的数字, 用英美数字表示法就变成了"1,000".

这就可以说是在英美数字表示法里面用3位的10进制作为单位存储一个大数字.

类比在 Linux device tree里面, 它用 8位的16进制(=32位的二进制)作为单位存储一个大数字.

比如我们要表达一个8GB的数字, 十进制的8GB = 0x100000000 (8个0) , 用device tree 的表达方法就是应该为 8GB = 0x1 0x00000000

这样问题又来了, 作为程序, 我怎么知道一串数字 <0x1 0x00000001 0x1000000>

哪一部分是空间首地址, 哪一部分是空间大小呢?

所以, 就有了 `#address-cells` , 和 `#size-cells` 这两个字段

这两个东西都是用来描述 `reg` 这个字串的, 他们描述了, 在描述首地址或者是空间大小时,使用了多少个8bytes 字符

举个栗子

![example](/img/2016-03-17/1.png)


这个是kernel文档里面关于flash分区的一段描述

这个flash的分区里面的 #address-cell =1, #size-cells=2 所以, 在他的 `reg` 字段里面, 第一个8bytes 字符表示的是这个分区的起始地址, 第二和第三个8bytes字符表示的是这个分区的大小




关于一些其他的知识, 在kernel的文档里面说的很清楚: https://www.kernel.org/doc/Documentation/devicetree/booting-without-of.txt


> In general, the format of an address for a device is defined by the parent bus type, based on the `#address-cells` and `#size-cells` properties.  Note that the parent's parent definitions of `#address-cells` and `#size-cells` are not inherited so every node with children must specify them.  The kernel requires the root node to have those properties defining addresses format for devices directly mapped on the processor bus.

> Pasted from <https://www.kernel.org/doc/Documentation/devicetree/booting-without-of.txt> 

另外这里提到的OF, 也就是Open Firmware, https://en.wikipedia.org/wiki/Open_Firmware

这个是基于PowerPC的一个bootloader啦


# Part 3 博通的NAND的device tree的描述格式

在这里, 讲的很清楚, 连例子都有

[https://www.kernel.org/doc/Documentation/devicetree/bindings/mtd/brcm,brcmnand.txt]()
