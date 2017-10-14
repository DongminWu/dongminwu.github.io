---
layout: post
title: 虚拟AP和基础网模式
date: 2014-05-16
categories: Openwrt
---


今天的又在做毕设。

昨天把网页设置的部分完了一点。

今天想添加新功能，虚拟AP，AP模式和基础网模式互换

## Part.1 虚拟AP

就是用一个路由器广播多个ssid

这样的可以完成不同用户的独立控制。

比如呢。我给家里的wifi分隔一下，一个是`Myself`，我自己用的。另一个是`Guest`,用来给访客用

给自己的流量自然要多分一点，给访客的就让他们能够看网页就好啦~~~

这个功能在`/etc/config/wireless`中有设置

先看wireless里面的config信息

```
config wifi-device 'radio0'
        option type 'mac80211'
        option hwmode '11ng'
        option path 'platform/ar933x_wmac'
        list ht_capab 'SHORT-GI-20'
        list ht_capab 'SHORT-GI-40'
        list ht_capab 'RX-STBC1'
        list ht_capab 'DSSS_CCK-40'
        option disabled '0'
        option channel 'auto'
        option htmode 'HT40+'
        option country 'CN'
        option noscan '1'

config wifi-iface
        option device 'radio0'
        option network 'lan'
        option mode 'ap'
        option encryption 'none'
        option ssid 'pipi1net'

config wifi-iface
        option device 'radio0'
        option network 'lan'
        option mode 'ap'
        option encryption 'none'
        option ssid 'pipi2net'
```

参考资料在这里：[Wireless configuration](http://wiki.openwrt.org/doc/uci/wireless) | [利用nodogsplash 打造多节点超强无线广告机 ](http://blog.sina.com.cn/s/blog_6838386a0101d7gh.html)

我这里就是建立了两个不同的ap

一个是`pipi1net`，另一个是`pipi2net`

用uci实现


#### 添加一个虚拟ap
```
uci add wireless wifi-iface
uci set wireless.@wifi-iface[1].device=radio0
uci set wireless.@wifi-iface[1].network=lan
uci set wireless.@wifi-iface[1].mode=ap
uci set wireless.@wifi-iface[1].encryption=none
uci set wireless.@wifi-iface[1].ssid=pipi2net
uci commit
```

#### 删除虚拟ap

```
uci delete wireless.@wifi-iface[1]
uci commit
```

记得如果要立即看到效果的话要添加`/etc/init.d/network restart`啊啊啊

## Part.2 基础网和AP模式互换

基础网模式就是wifi-client，就是像手机一样用wifi连接在某个ap上作为从机

AP模式是wifi-AP

主要参考资料：[Routed AP](http://wiki.openwrt.org/doc/recipes/routedap) | [Routed Client](http://wiki.openwrt.org/doc/recipes/routedclient)

我的这个模块拿到手里面的时候就是默认的AP模式

改为基础网（sta）模式要修改的地方
（修改第一个wifi ap）

```
 uci del wireless.@wifi-device[0].disabled
uci del wireless.@wifi-iface[0].network
uci set wireless.@wifi-iface[0].mode=sta
uci commit wireless
wifi
```
**PS：wifi命令相当于原来的/etc/init.d/network restart**

然后就开始扫描空间中的wifi信号

    iwlist scan

官网资料上面说如果出现了Devic Busy或者资源被占用之类的事情

可以执行

    killall -9 wpa_supplicant
    iwlist scan

我在图书馆的扫描结果如下：

```

wlan0     Scan completed :
          Cell 01 - Address: 00:1F:64:EB:4F:78
                    Channel:1
                    Frequency:2.412 GHz (Channel 1)
                    Quality=29/70  Signal level=-81 dBm
                    Encryption key:off
                    ESSID:"Cert_Download"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00000011157e1d06
                    Extra: Last beacon: 740ms ago
                    IE: Unknown: 000D436572745F446F776E6C6F6164
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030101
                    IE: Unknown: 0706434E20010D14
                    IE: Unknown: 2A0100
                    IE: Unknown: 32043048606C
                    IE: Unknown: 2D1AAD0103FFFF000000000000000000000000000000040
                    IE: Unknown: 331AAD0103FFFF000000000000000000000000000000040
                    IE: Unknown: 3D160100170000000000000000000000000000000000000
                    IE: Unknown: 34160100170000000000000000000000000000000000000

                    IE: Unknown: 4A0E14000A002C01C800140005001900
                    IE: Unknown: 7F0101
                    IE: Unknown: DD180050F2020101060003A4000027A4000042435E00623
                    IE: Unknown: DD0900037F01010000FF7F
          Cell 02 - Address: 00:1F:64:EC:4F:78
                    Channel:1
                    Frequency:2.412 GHz (Channel 1)
                    Quality=31/70  Signal level=-79 dBm
                    Encryption key:off
                    ESSID:"BUPT-3"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=4b000011157e26e8
                    Extra: Last beacon: 710ms ago
                    IE: Unknown: 0006425550542D33
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030101
                    IE: Unknown: 0706434E20010D14
                    IE: Unknown: 2A0100
                    IE: Unknown: 32043048606C
                    IE: Unknown: 2D1AAD0103FFFF000000000000000000000000000000040

                    IE: Unknown: 331AAD0103FFFF000000000000000000000000000000040
                    IE: Unknown: 3D160100170000000000000000000000000000000000000
                    IE: Unknown: 34160100170000000000000000000000000000000000000
                    IE: Unknown: 4A0E14000A002C01C800140005001900
                    IE: Unknown: 7F0101
                    IE: Unknown: DD180050F2020101060003A4000027A4000042435E00623
                    IE: Unknown: DD0900037F01010000FF7F
          Cell 03 - Address: 00:1F:64:ED:4F:78
                    Channel:1
                    Frequency:2.412 GHz (Channel 1)
                    Quality=30/70  Signal level=-80 dBm
                    Encryption key:on
                    ESSID:"eID_WAPI"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=32000011157e379a
                    Extra: Last beacon: 2000ms ago
                    IE: Unknown: 00086549445F57415049
                    IE: Unknown: 010882848B960C121824
                    IE: Unknown: 030101
                    IE: Unknown: 0706434E20010D14
                    IE: Unknown: 2A0100
                    IE: Unknown: 44140100010000147201010000147201001472010000
                    IE: Unknown: 32043048606C
                    IE: Unknown: 2D1AAD0103FFFF000000000000000000000000000000040
                    IE: Unknown: 331AAD0103FFFF000000000000000000000000000000040
                    IE: Unknown: 3D160100170000000000000000000000000000000000000
                    IE: Unknown: 34160100170000000000000000000000000000000000000
                    IE: Unknown: 4A0E14000A002C01C800140005001900
                    IE: Unknown: 7F0101
                    IE: Unknown: DD180050F2020101060003A4000027A4000042435E00623
                    IE: Unknown: DD0900037F01010000FF7F
          Cell 04 - Address: 3E:4B:D6:A1:34:28
                    Channel:6
                    Frequency:2.437 GHz (Channel 6)
                    Quality=26/70  Signal level=-84 dBm
                    Encryption key:on
                    ESSID:"AWIN-JPMT21CIJMJ"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master
                    Extra:tsf=00000002ea7a06c0
                    Extra: Last beacon: 410ms ago

.........................实在太长了就截取到这里吧

```

于是呢我们就能找到想要加入的wfi网络的SSID和加密协议了~

根据这个信息使用UCI命令再次修改`/etc/config/wireless`

尝试连接至**BUPT-3**

```
uci set wireless.@wifi-iface[0].ssid=BUPT-3
uci set wireless.@wifi-iface[0].network=wan
uci commit

```

设置之后的`/etc/cofig/wireless`文件`wifi-iface[0]`部分内容如下

```
config wifi-iface
        option device 'radio0'
        option encryption 'none'
        option mode 'sta'
        option network 'wan'
        option ssid 'BUPT-3'
```

执行`wifi`命令之后出现一下信息

```
[ 1275.460000] wlan0: authenticate with 00:1f:64:ec:4f:78
[ 1275.470000] wlan0: send auth to 00:1f:64:ec:4f:78 (try 1/3)
[ 1275.530000] wlan0: authenticated
[ 1275.540000] wlan0: associate with 00:1f:64:ec:4f:78 (try 1/3)
[ 1275.550000] wlan0: RX AssocResp from 00:1f:64:ec:4f:78 (capab=0x421 status=0
aid=9)
[ 1275.560000] wlan0: associated
[ 1275.560000] IPv6: ADDRCONF(NETDEV_CHANGE): wlan0: link becomes ready
```

成功变为基础网模式

再次切换为AP模式只需恢复原有config即可

```
uci set wireless.@wifi-iface[0].network=lan
uci set wireless.@wifi-iface[0].ssid=pipinet
uci set wireless.@wifi-iface[0].mode=ap
uci commit
```
