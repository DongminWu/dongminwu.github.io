---
layout: post
title: Using Jenkins to Build Wince
date: 2016-03-06
categories: WinCE
---


# Part 1 历史由来

好了, 这次让我们说中文, 就不瞎掰扯自己的垃圾英语水平了.

公司项目里面用的是Windows CE 5.0 嵌入式系统, 这是个非常老旧的操作系统, 没记错的话整个系统只能有32个用户进程, 每个进程还不能完整使用全部的虚拟内存地址.

关于这套系统: [https://en.wikipedia.org/wiki/Windows_CE]()

这套古老的系统我维护了一年了, 工作是维护HAL, 也就是硬件外设层的代码.

因为系统的古老所以一直以来这个项目的版本发布都是依赖于每个开发者自己编译和发布.

有个10年前的子项目的发布步骤繁琐到令人发指, 照着文档做都很难完全做对.

这样带来的问题可想而知, 去年我就因为发布的时候漏掉了一步导致被投诉...

做维护的事情就是很烦而且还要负责任.

那次事故之后, team里面就申请了一台服务器上的虚拟机用作专门版本发布.

我就在工作之余慢慢研究整个的工作流程怎么移植到这台专用机器上面.


# Part 2 Jenkins

Jenkins, [http://jenkins-ci.org]()

这应该是一个主流的自动化编译服务器框架吧.官网上说他的好处如下:

>Jenkins offers the following major features out of the box, and many more can be added through plugins:

>Easy installation: Just run java -jar jenkins.war, deploy it in a servlet container. No additional install, no database. Prefer an installer or native package? We have those as well.

>Easy configuration: Jenkins can be configured entirely from its friendly web GUI with extensive on-the-fly error checks and inline help.

>Rich plugin ecosystem: Jenkins integrates with virtually every SCM or build tool that exists. View plugins.

>Extensibility: Most parts of Jenkins can be extended and modified, and it's easy to create new Jenkins plugins. This allows you to customize Jenkins to your needs.

>Distributed builds: Jenkins can distribute build/test loads to multiple computers with different operating systems. Building software for OS X, Linux, and Windows? No problem.

在公司的项目中确实也有很多在用他来管理的, 所以大家的学习使用成本很低.

但是问题是这是第一次在WINCE, 或者说是windows平台上进行这种automation的工作.

宿主机器是Windows XP系统!

PS: WINDOWS不死, 程序员的世界永远不会更美好.

# Part 3 Jenkins 用户权限管理问题

这是安装Jenkins一开始就遇到的问题..

注册了一个编译用的账号然而权限不足, 我是用直接改 Jenkins目录下 `config.xml` 的方法解决的额.

可以参考这个: [http://www.cnblogs.com/gao241/archive/2013/03/20/2971416.html]()

关于怎么在Windows XP 下面重启Jenkins, 我机器上面的是以服务模式运行的, 所以用的方法是直接在 CMD里面 输入:

```
sc stop jenkins
sc start jenkins
```

就直接吧编译账号变为最高权限吧, 反正没几个人会用的.

# Part 4 Windows CE 通常的编译方式

Wince的编译有两个模式, 一个是编译kernel, 第二个是编译应用,

因为有IDE的存在, 所以如果按照Wince 正常的工作模式, 编译也就是点两个按钮的问题.

但是因为我们发布的是库文件, 所以要用到命令行下的编译工具

 [Build.exe](https://msdn.microsoft.com/en-us/library/aa448614.aspx)

 但是通过先人传承下来的workflow, 我们需要

 1. 将对应的硬件的 [BSP](https://msdn.microsoft.com/en-us/library/ms901757.aspx) 放到 `C:\WINCE500`
 2. 打开IDE
 3. 通过IDE打开对应硬件平台的工程项目文件
 4. 点击菜单栏的 `Build Tools -> Open Build Source`
 5. 在弹出的命令行用`cd` 命令定位到我们项目的Git根目录
 6. 输入 `build -c` 编译


 这需要极强的人工参与, 这也是为什么每次发版本效率极低



 # Part 5 使用Windows Batch Command进行编译


 通过网上查找资料, 还有自己的对比, 发现其实

 > 点击菜单栏的 `Build Tools -> Open Build Source`

 这一步弹出的窗口, 是一个已经预设过环境变量的CMD窗口而已,

 之所以能通过输入`build`命令, 调用Build.exe程序, 就是因为在这个窗口的`PATH`环境变量里面吧build.exe所在的目录包含了.

 所以通过使用`set`命令比对 原始的CMD窗口的环境变量 和从IDE里面打开的窗口的环境变量, 就能得到在编译时候的窗口环境了.

 经过试验, 这样做是可行的, 于是进行下一步



 # Part 6 Python进行流程控制

 如果是单纯的编译就没那么麻烦了,

 但是发布版本还涉及到文件的移动拷贝, 还有打包的工作

 试图完全用Windows Batch Command 脚本来写整套逻辑的, 发现BAT脚本在逻辑处理上面真的是很糟糕, 于是就决定用python来进行一些逻辑层的处理

 写了一个auto.py脚本和几个作为实施者的BAT脚本.

 想到几个地方要注意的:

 1. 必须要是同步的, 在实施者执行命令的时候必须等待, 否则会出编译错误
 2. 最好每一步打好时间戳, 方便调试

 不得不承认这是我第一次用Python写工具类应用, 之前都是解一些算法题什么的所以功力不够.

> 必须要是同步的, 在实施者执行命令的时候必须等待, 否则会出编译错误

这一个的解决方案是用 模块subprocess 的`Pipe.open()` 和 `Pipe.wait()` 实现的.

可以参考这里:

[小心subprocess的PIPE卡住你的python程序](http://www.aikaiyuan.com/4705.html)

[避免python Popen阻塞](http://backend.blog.163.com/blog/static/2022941262014016710912/)


> 最好每一步打好时间戳, 方便调试

这个就很简单的调用 time 模块就行了

教程: [python 获取当前时间](http://www.cnblogs.com/wanpython/archive/2010/08/07/1794598.html)

# Part 7 SSH 的问题

这个问题是在尝试吧 GIT 仓库管理的部分集成在 `auto.py` 里面的时候遇到的, 虽然现在把这一块的管理移出到使用Jenkins插件了, 但是还是可以记录一下.

ssh实现免密码登陆的话, 需要把私钥存在 `~/.ssh/id_rsa` 文件里面, 于是我可以用Git bash里面的小型的cygwin来运行这个 `auto.py` 并且把准备好的私钥文件copy到`~/.ssh/` 里面就好.

然后在Jenkins的 Jenkins设置里面, 把shell的路径指定到 `C:\Program Files\Git\bin\bash.exe` 也就是GIT bash的可执行文件, 这样Jenkins在执行shell 命令的时候就会调用我们配置过SSH 私钥的Terminal了.
