---
layout: post
title: LVS下因主备都是LocalNode导致的包转发死循环问题
categories: linux
description: LVS下因主备都是LocalNode导致的包转发死循环问题
keywords: linux, lvs
---

## 问题现象

telnet LVS VIP会间歇性未响应,提示`NO route to host`。

![lvs_localnode_1.png](https://i.loli.net/2018/07/31/5b6083d295616.png)

## 背景

localnode指的是 realserver 和 director在同一台设备上，这样做是为了节省机器资源。 

## 问题原因

LVS主备都是RS和Director部署在同一台设备上，而且主备都启用了ipvsadm设置的转发规则，那么就会有50% 机会 master director 把包转给 
backup，backup又有50%机会把包转给 master，master 收到包后，又有50%机会把包转给 backup，然后陷入死循环。这种情况只会出现在VIP:PORT组合中。

## 解决办法

Director A 配置 iptables ，除了 Director B以外的包，都标记 mark 为3
Director B 配置 iptables ，除了 Director A以外的包，都标记 mark 为4

在入口处做标识，区分出是正常请求，还是来自于Director的转发请求。如果是正常请求(客户端进来的包，DR-A上标记为3，DR-B上标记为4)，则进入
转发模块，根据调度策略，转给本设备或另一台设备处理；如果是转发请求（来自于另一设备的分流请求），则不需要再进入本设备转发模块了，直接交
由RealServer处理。

### 参考配置
``` bash
# http://www.austintek.com/LVS/LVS-HOWTO/HOWTO/LVS-HOWTO.localnode.html#two_box_lvs_active_active
1. On node1 create an iptables rule of the form:
-t mangle -I PREROUTING -d $VIP -p tcp -m tcp --dport $VPORT -m mac ! --mac-source $MAC_NODE2 -j MARK --set-mark 0x6
2. where $MAC_NODE2 is node2's MAC address as seen by node1. Do a similar trick on node2:
-t mangle -I PREROUTING -d $VIP -p tcp -m tcp --dport $VPORT -m mac ! --mac-source $MAC_NODE1 -j MARK --set-mark 0x7
where $MAC_NODE1 is node1's MAC address as seen by node2.
 
3. Change your keepalived.conf so that it uses fwmarks.
node1: virtual_server fwmark 6 { node2: virtual_server fwmark 7 {
```

### 示例

本例中设备IP、VIP、网卡MAC信息如下：
```
DR1/RS1
eth0 00:0C:29:DC:D2:6B 172.17.85.54
eth1 00:0C:29:DC:D2:75 119.253.282.16
DR2/RS2
eth0 00:0C:29:57:88:CE 172.17.85.55
eth1 00:0C:29:57:88:D8 119.253.282.17
VIP
eth0 172.17.85.56
eth1 119.253.282.6
```

分别在两台RS上更改配置。

1. DR1/RS1-172.17.85.54

 - Director1 的内网VIP 172.17.85.56配置 iptables ，除了 Director2 以外的包，都设置 mask 为3。
 ```bash
 iptables  -t mangle -I PREROUTING -d 172.17.85.56 -p tcp -m tcp --dport 80 -m mac ! --mac-source 00:0C:29:57:88:CE -j MARK --set-mark 0x3
 ```
 
  - Director1 的外网VIP 119.253.282.6配置 iptables ，除了 Director2 以外的包，都设置 mask 为5。
  ```bash
  iptables  -t mangle -I PREROUTING -d 119.253.282.6 -p tcp -m tcp --dport 80 -m mac ! --mac-source 00:0C:29:57:88:D8 -j MARK --set-mark 0x5
  ```
  
  - keepalived.conf
  ```bash
  1)
  virtual_server 172.17.85.56 80 {
  改为
  virtual_server fwmark 3 {
  2)
  virtual_server 119.253.282.6 80 {
  改为
  virtual_server fwmark 5 {
  ```
2. DR2/RS2-172.17.85.55

 - Director2 的内网VIP 172.17.85.56配置 iptables ，除了 Director1 以外的包，都设置 mask 为4。
 ```bash
 iptables  -t mangle -I PREROUTING -d 172.17.85.56 -p tcp -m tcp --dport 80 -m mac ! --mac-source 00:0C:29:DC:D2:6B -j MARK --set-mark 0x4
 ```
 
 - Director2 的外网VIP 119.253.282.6配置 iptables ，除了 Director1以外的包，都设置 mask 为6。
 ```bash
 iptables  -t mangle -I PREROUTING -d 119.253.282.6 -p tcp -m tcp --dport 80 -m mac ! --mac-source 00:0C:29:DC:D2:75 -j MARK --set-mark 0x6
 ```
 
 - keepalived.conf
 ```bash
 1)
 virtual_server 172.17.85.56 80 {
 改为
 virtual_server fwmark 4 {
 2)
 virtual_server 119.253.282.6 80 {
 改为
 virtual_server fwmark 6 {
 ```
 
 分别在两台设备上平滑加载keepalived配置,`/etc/init.d/keepalived reload`,之后测试正常。
 
 ![lvs_localnode_2.png](https://i.loli.net/2018/07/31/5b60837fe635c.png)
 
  
## 参考文章

[LVS DR模式的一些问题 ](http://linbo.github.io/2017/08/20/lvs-dr)

[两台机器 Centos7 LVS+Keepalived 实现负载/高可用之 DR 集成部署(LocalNode/Two Box)及原理详解](http://jiangyu86.cn/2018/03/04/cluster/lvs_localnode_dr/)
