---
layout: post
title: 月末的毕业设计
date: 2014-04-30
categories: Openwrt
---

恩一转眼就月末了呢。

这个星期大概就没什么进展，从上次编译出SDK以后就一直没有coding关于毕设的事情

前几天看了一下教程[【整理】如何在OpenWRT环境下做开发](http://hi.baidu.com/gouooo/item/6932bfa97d23d1981410736a)

发现自己还是要学一些关于**Makefile**的东西，于是就找了个**跟我一起学Makefile**看了看，

恩目前还没看完但是觉得还是挺有收获的。

于是今天早上就按照教程上的写了个helloworld程序

遇到了一下的问题恩

一开始我的Makefile是这样写的。。（这里只是最后的一些执行语句）

{%highlight makefile%}
<!@@make@@!>

#prepare

define Build/Prepare
mkdir -p $(PKG_BUILD_DIR)
$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

#install will be call

define Package/$(PKG_NAME)/install
$(INSTALL_DIR) $(1)/bin
$(INSTALL_BIN) $(PKG_BUILD_DIR)/$(PKG_NAME) $(1)/bin/
endef

#the function eval will add the text into makefile

$(eval $(call BuildPackage,helloworld))


{% endhighlight %}


但是每次make的时候总是报错说是最后一行少了一个分隔符（separator）

百度了一下说是命令之前要加上一个`TAB`字符什么的

但是Makefile直接调用函数什么的根本不用加`TAB`啊啊啊啊。。。

于是琢磨了好久发现是在之前的`define`里面有问题

Makefile的define就是宏。。就相当于是文本替换。所以在

{%highlight makefile%}
<!@@make@@!>
define Package/$(PKG_NAME)/install
$(INSTALL_DIR) $(1)/bin
$(INSTALL_BIN) $(PKG_BUILD_DIR)/$(PKG_NAME) $(1)/bin/
endef

{% endhighlight %}


这样打出的命令之前是没有`TAB`字符的。。于是在最后一行调用的时候就出现了缺少分隔符错误

后来修改成这样就正确了

{%highlight makefile%}
<!@@make@@!>
define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef


define Package/helloworld/install
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/bin/
endef

$(eval $(call BuildPackage,helloworld))
{% endhighlight %}
**注意上面这两个define中的行命令的行首都有TAB**

恩。小问题但是要注意细节恩~
