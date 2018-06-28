---
layout: post
title: 理解wireshark中的length字段含义
categories: network
description: 理解wireshark中的length字段含义
keywords: network, wireshark
---

主要介绍wireshark中的length计算方法，主要涉及MTU、MSS、常见协议头大小。


## length含义

Wireshark is showing you the length of the Ethernet frame that is handed to it, which may or may not include the FCS.
wireshark显示的长度为以太网帧的长度，不包括FCS(Frame check sequence)[帧校验序列]

## 常见协议头大小

``` bash
Ethernet header: 14 bytes # 数据链路层帧头大小，一般为14bytes,其为源MAC(6bytes/48bits)+目的MAC(6bytes/48bits)+类型/长度(2bytes)
IP header (standard): 20 bytes # IP头部大小
ICMP header: 8 bytes     # ICMP头部大小
ICMP payload: 32 bytes   # ICMP缓冲区大小，可以通过ping的-l选项指定大小，默认为32。
FCS 4 bytes of Ethernet checksum # 帧尾CRC校验
```

## length计算

MTU=MSS+IP header(20 bytes)+tcp header(20 bytes)
length=MTU+Ethernet header(14bytes)

其中MSS为Maximum Segment Size，即最大报文段长度，其受MTU大小影响，这里的MTU指的是三层的，二层的MTU固定为1500，不能修改。
MTU为Maximum Transmission Unit,即最大传输单元，需要注意MTU如果太小会影收到数据包的速度，表现为下载过慢。

```
三层收到大数据包时，要将数据包分片后再往下层传输，这就是IP分片原理。既然IP分片可以改变一个IP数据包的大小 ，那么IP分片怎么设置呢？
这也是网上人们所谓的修改MTU值达到最佳网速的方法，而这里所说的修改“MTU”大小其实是IP分片的大小。
```
![2.png](https://i.loli.net/2018/06/29/5b351f365f656.png)
![1.png](https://i.loli.net/2018/06/29/5b351195a1754.png)

从截图中可以看到MSS为1460，MTU计算后为1500，1460 + 20(IP header) + 20(tcp header) = 1500

## 参考

[length field](https://osqa-ask.wireshark.org/questions/44865/length-field)

[Understanding Guide to ICMP Protocol with Wireshark](http://www.hackingarticles.in/understanding-guide-icmp-protocol-wireshark/)

[ireshark实战之：MTU、MSS及计算方法](https://blog.csdn.net/chiyuwei1766/article/details/50762894)

