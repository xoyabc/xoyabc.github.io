---
layout: post
title: 理解IP及DNS协议中的TTL
categories: linux
description: 理解IP及DNS协议中的TTL
keywords: IP, DNS, TTL
---

本文主要介绍了IP及DNS协议中的TTL含义

# 1. TTL

## 1.1 IP协议中的TTL

- 定义

TTL是IP协议包中的一个值，指定数据报被路由器丢弃之前允许通过的网段数量。(IP数据包在计算机网络中可以转发的最大跳数)

在很多情况下数据包在一定时间内不能被传递到目的地。解决方法就是在一段时间后丢弃这个包，然后给发送者一个报文，由发送者决定是否要重发。

TTL 是由**发送主机**设置的，以防止数据包不断在 IP 互联网络上永不终止地循环。转发 IP 数据包时，每经过一个路由器，路由器会修改TTL值， 即将改值减小1。当记数到0时，路由器决定丢弃该包，并发送一个*ICMP Type 11 and Code 0 message(Time to live exceeded)* 报文给最初的发送者，由发送者决定是否要重发。

###  1.1.1 常见操作系统的TTL值

```
UNIX 及类 UNIX 操作系统       ICMP 回显应答的 TTL 字段值为 255
Compaq Tru64 5.0             ICMP 回显应答的 TTL 字段值为 64
微软 Windows NT/2K操作系统    ICMP 回显应答的 TTL 字段值为 128
微软 Windows 95 操作系统      ICMP 回显应答的 TTL 字段值为 32
LINUX Kernel 2.2.x & 2.4.x   ICMP 回显应答的 TTL 字段值为 64
```

### 1.1.2 linux系统TTL值修改

TTL值在文件/proc/sys/net/ipv4/ip_default_ttl中定义,可通过执行`echo 128 > /proc/sys/net/ipv4/ip_default_ttl`命令修改
(这是短暂性的)若要永久生效可修改/etc/sysctl.conf配置文件，添加`net.ipv4.ip_default_ttl=128`，接着执行`sysctl -p`即可。

### 1.1.3 理解发送主机

在本机(windows 10)ping本地的VMware虚拟主机(操作系统为CentOS release 6.8)，其IP为192.168.10.128，可见TTL为64：

[![1.jpg](https://i.loli.net/2018/05/05/5aec99300e4f5.jpg)](https://i.loli.net/2018/05/05/5aec99300e4f5.jpg)

在CentOS上执行`echo 168 > /proc/sys/net/ipv4/ip_default_ttl`修改TTL值为**168**，接着再次在本机(windows 10)ping 192.168.10.128，发现TTL由64变为168

[![2.jpg](https://i.loli.net/2018/05/05/5aec9930178c5.jpg)](https://i.loli.net/2018/05/05/5aec9930178c5.jpg)

[![3.jpg](https://i.loli.net/2018/05/05/5aec99301b752.jpg)](https://i.loli.net/2018/05/05/5aec99301b752.jpg)

综上可知，这里的**发送主机**指的是**ping后面IP对应的主机**。


## 1.2 DNS中的TTL


- 定义


- 定义1

TTL(Time- To-Live)，简单的说它表示一条域名解析记录在DNS服务器上的缓存时间。

- 定义2

TTL值全称是“生存时间（Time To Live)”，简单的说它表示DNS记录在DNS服务器上缓存时间，数值越小，修改记录各地生效时间越快。

当各地的DNS(LDNS)服务器接受到解析请求时，就会向域名指定的授权DNS服务器发出解析请求从而获得解析记录；该解析记录会在DNS(LDNS)服务器中保存一段时间，这段时间内如果再接到这个域名的解析请求，DNS服务器将不再向授权DNS服务器发出请求，而是直接返回刚才获得的记录；而这个记录在DNS服务器上保留的时间，就是TTL值。

### 1.2.1 合理设置域名TTL值


#### 1.2.1.1 增大TTL值，以节约域名解析时间
 
通常情况下域名解析记录是很少更改的。我们可以通过增大域名记录的TTL值让记录在各地DNS服务器中缓存的时间加长，这样在更长的时间段内，我们访问这个网站时，本地ISP的DNS服务器就不需要向域名的NS服务器发出解析请求，而直接从本地缓存中返回域名解析记录,从而提高解析效率。
TTL值是以秒为单位的，通常的默认值都是3600，也就是默认缓存1小时。我们可以根据实际需要把TTL值扩大，例如要缓存一天就设置成86400。

#### 1.2.1.2 减小TTL值，减少更新域名记录时的不可访问时间

因为DNS记录缓存的问题，新的域名记录在有的地方可能生效了，但在有的地方可能等上一两天甚至更久才生效(部分省份运营商调大了TTL值)，这样就会就导致部分用户在一段时间内无法访问网站。

为了尽可能的减小各地的解析时间差，可参考以下步骤执行：


1.先查看当前域名的TTL值。

2.修改TTL值为可设定的最小值，建议为60秒。

3.等待一天，保证各地的DNS服务器缓存都过期并更新了记录，可使用[cloudxns全国DNS查询](http://tools.cloudxns.net/index/around)

4.设置并修改DNS解析到新的记录，这样各地的DNS就能以最快的速度更新到新的记录。

5.确认各地的DNS已经更新完成后，再将TTL值设置成常用的值(如: TTL=86400，一般解析商提供的默认值为600秒)。


# 2.参考链接

[https://www.cnblogs.com/tian4837/p/4178662.html](https://www.cnblogs.com/tian4837/p/4178662.html)

[https://osqa-ask.wireshark.org/questions/22337/ttl-time-to-live](https://osqa-ask.wireshark.org/questions/22337/ttl-time-to-live)

[https://blog.csdn.net/ysdaniel/article/details/6922097](https://blog.csdn.net/ysdaniel/article/details/6922097)
