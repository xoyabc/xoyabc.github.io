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

拓扑图：

![防火墙部署架构.png](https://i.loli.net/2020/08/04/XF7HGhxzQrdDLmu.png)

拓扑说明：
 - nginx 前实际上还挂有 NLB，为简化结构分析问题，在防火墙处直接将流量转给了 nginx

两台防火墙为主备工作模式，内部网络结构均为 NAT 模式，具体为 DNAT。外部流量进入防火墙时从 eth1 网口流入，之后会将数据包中的源地址转换为防火墙 eth2 地址，目标地址转换为 nginx，即"源地址--防火墙 eth1" --> "防火墙 eth2--nginx"


### 问题描述

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

进一步分析，上图为抓包截图，由于发出的 syn 包中的时间戳无 `TSval` 值，时间戳不是递增，于是服务器就会丢弃该数据包，不返回 syn-ack 包，表现为用户无法正常完成 tcp 3 次握手，`nc` 命令测试连接超时。

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


## ping 两台防火墙，其中一个不通

### 问题描述

源地址：192.168.69.94  该设备为 nginx

两个目标地址均为防火墙，只是 IP 地址不同。

目标地址：192.168.68.250 该设备为防火墙  不能 ping 通

目标地址：192.168.64.194 该设备为防火墙  可以 ping 通	


```shell
[root@nginx ~]# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 06:dd:18:6a:fc:10 brd ff:ff:ff:ff:ff:ff
    inet 192.168.69.94/24 brd 192.168.69.255 scope global dynamic ens5
       valid_lft 2386sec preferred_lft 2386sec
    inet6 fe80::4dd:18ff:fe6a:fc10/64 scope link 
       valid_lft forever preferred_lft forever
[root@nginx ~]# 
[root@nginx ~]# 
[root@nginx ~]# ping -c 5 192.168.64.194
PING 192.168.64.194 (192.168.64.194) 56(84) bytes of data.
64 bytes from 192.168.64.194: icmp_seq=1 ttl=58 time=2.39 ms
64 bytes from 192.168.64.194: icmp_seq=2 ttl=58 time=2.30 ms
64 bytes from 192.168.64.194: icmp_seq=3 ttl=58 time=2.26 ms
64 bytes from 192.168.64.194: icmp_seq=4 ttl=58 time=2.27 ms
64 bytes from 192.168.64.194: icmp_seq=5 ttl=58 time=2.41 ms

--- 192.168.64.194 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4005ms
rtt min/avg/max/mdev = 2.267/2.331/2.410/0.060 ms
[root@nginx ~]# 
[root@nginx ~]# ping -c 5 192.168.68.250
PING 192.168.68.250 (192.168.68.250) 56(84) bytes of data.

--- 192.168.68.250 ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 3999ms
```

### 排查

检查 192.168.68.250 的安全组、ACL、路由表，均未发现异常。与 192.168.64.194 进行对比，二者网络配置一致。

接着排查 192.168.68.250 防火墙的安全策略，也未发现问题。之后对比两台防火墙的路由表，发现 192.168.68.250 的防火墙路由表中，与问题相关联的路由规则如下：

```shell
    目标              下一个跃点        跃点数    标记      接口
   0.0.0.0/0          192.168.68.1         10       A S     ethernet1/1
   192.168.68.0/24      192.168.68.250       0        A C     ethernet1/1
   192.168.68.250/32    0.0.0.0            0        A H     -
   192.168.69.0/24      192.168.66.1         10       A S     ethernet1/2
```

因此，源地址为 192.168.69.94 ，目的地址为 192.168.68.250 的 ping 请求，防火墙会从网络接口 eth1 收到包，而根据路由表的配置，输出接口为 eth2。

通常情况下，由于默认启用反向路径过滤(rp_filter)功能，输入设备与输出设备不同时，反向路径过滤检查将会失败，该包将被丢弃。

![rp_filte.png](https://i.loli.net/2020/08/04/2tzKY9kcNIMDOnU.png)

为验证是该问题引起的，在 192.168.64.194 防火墙（正常的那台）上路由增加了一条 192.168.69.0/24 192.168.63.1 10 A S ethernet1/2 ，则从 192.168.69.94 ping 不通 192.168.64.194，反向测试成功。

注：若防火墙在使用，切勿直接增加路由。

### 解决

 - 法一：修改防火墙路由表，使其从同一个网络接口输出。
 
 移除 “   192.168.69.0/24      192.168.66.1         10       A S     ethernet1/2” 的路由，使数据包从 eth1 出去。
  
 - 法二：关闭实例的"源或目标检查"，禁用 rp filter
 
 a. 源/目标检查属性禁用后，实例会处理并未明确指定至该实例的网络通信。
 
 b. 将实例的 sysctl 内核参数 "net.ipv4.conf.all.rp_filter" , "net.ipv4.conf.default.rp_filter" , "net.ipv4.conf.eth2.rp_filter" 修改为0或者2。
 

```plain
rp_filter (Reverse Path Filtering)参数定义了网卡对接收到的数据包进行反向路由验证的规则。
0为关闭反向路由校验；1为开启严格的反向路由校验；2为开启松散的反向路由校验；
对每个进来的数据包，只校验其源地址是否可达，即反向路由是否能通（通过任意网口），如果反向路径不通，则直接丢弃该数据包。

当进行源地址检查时，rp_filter 的值为 net.ipv4.conf.all.rp_filter 和 net.ipv4.conf.eth0.rp_filter 中的最大值。

若只关闭一个接口的 rp filter，应将 net.ipv4.conf.all.rp_filter 设为 0，并开启其他接口的 rp filter，再调整目标接口的 rp filter 为0。

假设实例上有两个接口 eth0，eth1，若关闭 eth0 的 rp filter  ，操作如下：
sysctl -w net.ipv4.conf.all.rp_filter=0
sysctl -w net.ipv4.conf.eth0.rp_filter=0
sysctl -w net.ipv4.conf.eth1.rp_filter=1
```

## REF

[被抛弃的tcp_recycle ](https://mp.weixin.qq.com/s/uwykopNnkcRL5JXTVufyBw)

[net.ipv4.tcp_timestamps参数设置](https://www.jianshu.com/p/a66ecd12927e)

[网站响应时快时慢的真相？只有 1% 的人知道 ](https://mp.weixin.qq.com/s/zoiZhS1wJTm28tBRSzZREg)

[一场由tcp_timestamps 引发的无解追击案](https://blog.51cto.com/fuyuan2016/1795998)

[TCP timestamp](https://perthcharles.github.io/2015/08/27/timestamp-intro/)

[net.ipv4.tcp_timestamps引发的tcp syn无响应案](https://blog.csdn.net/pyxllq/article/details/80351827)

[The TCP Timestamp Option](https://cloudshark.io/articles/tcp-timestamp-option/)

[ip-sysctl.txt](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)

[理解 net.ipv4.conf.all.rp_filter](https://zhuanlan.zhihu.com/p/129784373)

[Linux内核参数之rp_filter](https://www.jianshu.com/p/717e6cd9d2bb)
