---
layout: post
title: 网站接入安全设备遇到的一些问题及其解决办法
categories: python
description: 网站接入 防火墙，WAF时 遇到的一些问题及其解决办法
keywords: linux
---


最近公司要申请三级等保证，需要接入安全设备，测试过程中遇到了几个典型问题，这里总结记录下。

## 主防火墙 telnet 无法连通，备防火墙正常

### 环境及拓扑

环境：防火墙及 nginx 均部署在 AWS 上，在同一个 VPC 下。

网络拓扑：用户-->外网ALB-->防火墙 eth1-->防火墙 eth2-->nginx

拓扑说明：
 - nginx 前实际上还挂有 NLB，为简化结构分析问题，在防火墙处直接将流量转给了 nginx

两台防火墙为主备工作模式，内部网络结构均为 NAT 模式，具体为 DNAT。外部流量进入防火墙时从 eth1 网口流入，之后会将数据包中的源地址转换为防火墙 eth2 地址，目标地址转换为 nginx，即"源地址--防火墙 eth1" -->"防火墙-->nginx"


### 问题

在一台 server 上使用 `nc -zv -w 5 IP 80` 命令分别测试到两台防火墙的 80 端口连通性，一个通，一个不通，通的为备机。

### 排查

 - 安全组限制
 无限制
 
 - 防火墙配置
 配置无异常，安全策略无限制
 
 - 抓包分析
还是要使大招。同时在防火墙和后端 nginx server 上抓包。防火墙上的包文件名为 firewall.pcap，nginx 上的包文件为 nginx.pcap。

1，wireshark 打开 firewall.pcap ,使用 `ip.addr == server IP` 过滤包，之后展开 IP 层详情信息，找到 `Identification` 的值，这里为 `32886`。

![firewall-1.png](https://i.loli.net/2020/07/16/TQVcyNqHz85LfPm.png)

2，wireshark 打开 nginx.pcap，使用 `ip.id == 32886` 找到对应的包，这里序号为 80，查看 info 列发现无 `TSval` 值，由于服务器端同时开启了 `tcp_tw_recycle` 及 `tcp_timestamps`，linux 会丢弃所有来自远端的 timestamp 时间戳小于上次记录的时间戳(由同一个远端发出的)的任何数据包。换句话说，就是必须要保证数据包的时间戳是递增的。

![firewall-2.png](https://i.loli.net/2020/07/16/O7m81WNLdjK2REH.png)

进一步分析，上图为抓包截图，由于发出的 syn 包中的时间戳无 `TSval` 值，时间戳不是递增，于是服务器就会丢弃该数据包，不返会 syn-ack 包，表现为用户无法正常完成 tcp 3 次握手，`nc` 命令测试连接超时。

备防火墙由于流量较小（没有高并发，基本是串行），TIME_WAIT 状态连接下同一个远端使用的最后时间戳，基本都是单调递增的，内核也就不会将其丢弃。

主防火墙由于流量较大（高并发），同一时间有多个并发连接，TIME_WAIT 状态连接下同一个远端使用的最后时间戳，会有一部分不是单调递增的，内核就会将其丢弃。

可以使用 `netstat -st | egrep -i "drop|reject|overflowed|listen|filter"` 命令查看因不符合时间戳递增规则被丢弃的数据包

```shell
# netstat -st | egrep -i "drop|reject|overflowed|listen|filter" 

    8 ICMP packets dropped because they were out-of-window 

    826164 passive connections rejected because of time stamp  # 由于 timestamp 丢弃的数据包

    14 packets rejects in established connections because of timestamp 

    826963 SYNs to LISTEN sockets dropped 
```

timestamps 说明：

```
timestamps一个双向的选项，当一方不开启时，两方都将停用timestamps。
比如client端发送的SYN包中带有timestamp选项，但server端并没有开启该选项。
则回复的SYN-ACK将不带timestamp选项，同时client后续回复的ACK也不会带有timestamp选项。
当然，如果client发送的SYN包中就不带timestamp，双向都将停用timestamp。
```

### 解决

禁用 net.ipv4.tcp_timestamps 或 net.ipv4.tcp_tw_recycle , 使用 `sysctl -w 内核参数=0`将值改为 0 即可。


## REF

[被抛弃的tcp_recycle ](https://mp.weixin.qq.com/s/uwykopNnkcRL5JXTVufyBw)

[net.ipv4.tcp_timestamps参数设置](https://www.jianshu.com/p/a66ecd12927e)

[网站响应时快时慢的真相？只有 1% 的人知道 ](https://mp.weixin.qq.com/s/zoiZhS1wJTm28tBRSzZREg)

[一场由tcp_timestamps 引发的无解追击案](https://blog.51cto.com/fuyuan2016/1795998)

[TCP timestamp](https://perthcharles.github.io/2015/08/27/timestamp-intro/)

[net.ipv4.tcp_timestamps引发的tcp syn无响应案](https://blog.csdn.net/pyxllq/article/details/80351827)

[The TCP Timestamp Option](https://cloudshark.io/articles/tcp-timestamp-option/)

