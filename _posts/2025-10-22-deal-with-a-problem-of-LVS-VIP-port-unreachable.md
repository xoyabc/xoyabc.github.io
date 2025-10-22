---
layout: post
title: 记一次 LVS VIP 端口不通问题
categories: k8s
description: LVS部署后，测试到VIP+端口不通
keywords: Linux, lvs
---

## 问题
为验证国产CPU架构服务器对业务的适配，像往常一样，部署了 LVS+nginx ，之后使用 telnet 验证 VIP+443 端口，发现：
 - telnet ipv4 VIP + 443，不通
```bash
# telnet 111.222.333.154 443
Trying 111.222.333.154...
telnet: connect to address 111.222.333.154: Connection refused
``` 
 - telnet ipv6 VIP + 443，通
```bash
# telnet 2409:8888:8888:8888::b 443
Trying 2409:8888:8888:8888::b...
Connected to 2409:8888:8888:8888::b.
Escape character is '^]'.
```

服务器信息如下：

ipv4 VIP：111.222.333.154 

ipv6 VIP：2409:8888:8888:8888::b

LVS 主机1 IP：111.222.333.146

LVS 主机2 IP：111.222.333.147

RS 主机1 IP： 111.222.333.148

RS 主机2 IP： 111.222.333.149



## 排查

1，以下经常出问题的几点，排查后均未发现异常
 - ping ipv4 vip 检测连通性
 - RS 主机未配置 arp 抑制
 - RS 主机未添加 ipv4 vip 本机路由
 - 执行 ipvsadm -Ln 检查 lvs 转发配置
 - 同节点其他主机抢占 ipv4 vip
 - RS 及 LVS 主机端口网络策略


2，根据数据包请求链路逐层排查确认
	
>请求链路：1、客户端-->2、lvs VIP-->3、RS 主机 lo环回口-->4、根据四元组（源 IP、源端口、目标 IP、目标端口）查找后端应用socket结构体

基于第一步中 ping 的结果分析，1到3部均正常，怀疑为应用程序的 ipv4 端口未监听。为验证猜想，登录后端其中一台主机 111.222.333.148，执行 `netstat -ntpl |grep 443` 查看端口监听情况。
	
	
	
```bash
# netstat -ntpl |grep 443
tcp        0      0 127.0.0.1:443           0.0.0.0:*               LISTEN      3409342/nginx: mast
tcp        0      0 111.222.333.148:443      0.0.0.0:*               LISTEN      3409342/nginx: mast
tcp6       0      0 :::443                  :::*                    LISTEN      3409342/nginx: mast
```

从执行结果可以看出：

- Ipv4

nginx 只监听了本机外网地址 111.222.333.148，未监听 VIP 111.222.333.154。

- Ipv6

":::" 表示监听了所有可用的 IPv6 地址。

原因基本可以确认，由于 Nginx 未监听 ipv4 VIP+端口，找不到数据包对应的 socket，即该数据包不属于任何一个已建立的连接，因此该数据包会被丢弃，导致端口不通。

- 为什么没有监听到 ipv4 VIP+端口？

由于研发定制开发的 Nginx，会在启动时读取所有网口的IP，之后只监听读取到的这些IP。而 Nginx 先于RS主机部署，此时RS 主机 lo 环回口还未配置 ipv4 VIP，导致 Nginx 最终未监听。
	
## 解决

重启两台 RS 主机上的 Nginx pod，观察 ipv4 VIP 监听正常，测试 ipv4 VIP+443 端口恢复正常。

## 后续优化

- [x] 先部署 lvs，后容器化部署 Nginx
- [ ] Nginx 启动时，默认监听所有 ipv4 地址
	

## REF
[Linux 网络收包流程](https://www.cnblogs.com/edisonfish/p/17578159.html)
	

