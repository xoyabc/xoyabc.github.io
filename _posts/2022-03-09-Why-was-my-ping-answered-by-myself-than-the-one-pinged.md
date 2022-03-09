---
layout: post
title: 为什么 ping 目标主机对方却收不到包?
categories: python
description: 同网段内ping另一主机，目标主机收不到包，实际却是本机在回包。
keywords: linux
---

## 问题

直播业务中转集群节点间设备会互相拉取 hls ts 切片及接收外部转码集群推送的转码切片流，原来走的是公网，为节省成本，需要走专线，配置上专线IP后：

 - 中转-->转码集群，相互访问正常
 - 中转 A-->中转 B，节点间互拉异常

在 A 主机上，使用 curl 请求 B 主机，分别查看 A B 主机的 nginx 访问日志，发现请求打到了 A 主机，没有打到 B 主机，与预期不符。curl 命令及日志见下：

```shell
# 指定特殊 UA 头方便过滤日志
curl -svo /dev/null http://10.174.20.83/monitor.html -H "Host: mon.cloud.com"  -A lxhlxhlxh

# 主机 A 上的日志，主机 B 过滤不到日志
# 10.174.20.81 为客户端IP，10.174.20.83 为服务端IP
2022-03-09T02:37:12+08:00   :1646764632.579   :-   :10.174.20.81   :10.174.20.83   :GET   :http   :mon.cloud.com   :/monitor.html   :/monitor.html   :HTTP/1.1   :200   :lxhlxhlxh   :END
```

## 排查

当向 10.174.20.83 发起请求时，首先会查本机的路由表。

| Destination | Gateway       | Genmask       | Use Iface |
|-------------|---------------|---------------|-----------|
| 10.174.20.0 | 0.0.0.0       | 255.255.255.0 | lo        |
| 0.0.0.0     | 100.88.88.254 | 0.0.0.0       | eth0      |

（Gateway是0.0.0.0或者*表示目标是本主机所属的网络，不需要路由）

注意：上面的 lo 是假设的，为了方便下面的说明，实际在 Linux 中执行 `route -n` 是看不到 lo 接口的，因为它不对外。

目标IP 10.174.20.83，本机 lo 网卡 IP 为 10.174.20.81。分别拿 IP 及掩码进行 `&` 运算，得到网络位。

 - lo 网卡 10.174.20.81 & 255.255.255.0 = 10.174.20.0

 - 目标 IP 地址 10.174.20.83 & 255.255.255.0 = 10.174.20.0

结果表明目标 IP 和主机在同一网段内，根据 IP 最长匹配原则，之后会走 lo 那条路由，不会走最后的默认网关 100.88.88.254。而且 lo 环回网卡离内核进，在路由表中为第一条，即使在有多条路由匹配的情况下，数据包也会优先走 lo 环回接口。

环回接口有一个特点，就是**接收到的数据包又会发回给本机，也就是说回环网卡是自己和自己玩**，因此如果走的是环回接口发送数据包，永远也发不出去，因此我们不能让数据包走环回接口，所以需要将掩码设置成 255.255.255.255

为验证数据包是发给了本机，在 10.174.20.81 上 ping 10.174.20.83，同时在两台主机开启抓包。

 - 81 ping 83

```shell
# 使用 -s 指定数据包大小
# ping -s 1000 -c 3 -i 0.5 10.174.20.83       
PING 10.174.20.83 (10.174.20.83) 1000(1028) bytes of data.
1008 bytes from 10.174.20.83: icmp_seq=1 ttl=64 time=0.059 ms
1008 bytes from 10.174.20.83: icmp_seq=2 ttl=64 time=0.050 ms
1008 bytes from 10.174.20.83: icmp_seq=3 ttl=64 time=0.050 ms

--- 10.174.20.83 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 0.050/0.053/0.059/0.004 ms
```

 - 81 上抓包

可以看到回包

```shell
# tcpdump -i lo:6 -s0 -nv '((icmp) and (dst host 10.174.20.81))' 
tcpdump: listening on lo:6, link-type EN10MB (Ethernet), capture size 65535 bytes
23:27:13.156396 IP (tos 0x0, ttl 64, id 27424, offset 0, flags [none], proto ICMP (1), length 1028)
    10.174.20.83 > 10.174.20.81: ICMP echo reply, id 63337, seq 1, length 1008
23:27:13.657089 IP (tos 0x0, ttl 64, id 27861, offset 0, flags [none], proto ICMP (1), length 1028)
    10.174.20.83 > 10.174.20.81: ICMP echo reply, id 63337, seq 2, length 1008
23:27:14.160123 IP (tos 0x0, ttl 64, id 28266, offset 0, flags [none], proto ICMP (1), length 1028)
    10.174.20.83 > 10.174.20.81: ICMP echo reply, id 63337, seq 3, length 1008
```

 - 83 上抓包

无数据

```shell
# tcpdump -i eth0 -s0 -nvvv '((icmp) and (dst host 10.174.20.81))'   
tcpdump: WARNING: eth0: no IPv4 address assigned
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
```

## 解决

1，掩码改为32位

> sed -i 's#255.255.255.0#255.255.255.255#g' /etc/sysconfig/network-scripts/ifcfg-l0:6

重启网卡后再次 ping 10.174.20.83，发现直接不通了，后查看路由表，发现没有 10.174.20.0/24 段的路由，增加以下策略路由后恢复。

> ip route add 10.174.20.0/24 via 100.88.88.254 dev eth0 src 10.174.20.81

同时需要将路由写到配置文件中，避免机器重启后丢失。

> echo '10.174.20.0/24 via  100.88.88.254  src 10.174.20.81' >> /etc/sysconfig/network-scripts/route-eth0


## REF

[LVS中VIP配置在环回接口上，子网掩码为什么是4个255？](https://blog.csdn.net/SmallCatBaby/article/details/89876508)
