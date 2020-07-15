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

两台防火墙为主备工作模式，内部网络结构均为 NAT 模式，具体为 DNAT。外部流量进入防火墙时从 eth1 网口流入，之后会将数据包中的源地址转换为防火墙 eth2 地址，目标地址转换为 nginx，即"源地址--防火

墙 eth1" -->"防火墙-->nginx"


### 问题

在一台 server 上使用 `nc -zv -w 5 IP 80` 命令分别测试到两台防火墙的 80 端口连通性，一个通，一个不通，通的为备机。

### 排查

 - 安全组限制
 无限制
 
 - 防火墙配置
 配置无异常，安全策略无限制
 
 - 抓包分析
还是要使大招。同时在防火墙和后端 nginx server 上抓包。防火墙上的包文件名为 firewall.pcap，nginx 上的包文件为 nginx.pcap。

wireshark 打开 firewall.pcap ,使用 `ip.addr == server IP` 过滤包，之后展开 IP 层详情信息，找到 `Identification` 的值

![firewall-1.png](https://i.loli.net/2020/07/16/TQVcyNqHz85LfPm.png)











