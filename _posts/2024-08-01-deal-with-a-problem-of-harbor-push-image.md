---
layout: post
title: 记一次 harbor 推送镜像失败问题
categories: python
description: 公网 harbor 往内网测试环境 harbor 推送镜像失败问题排查
keywords: Linux, docker, harbor
---

## 问题
为实现统一管理，镜像需要交由 A 部门集中构建，之后再推送到测试环境 harbor 仓库。
在A 部门 harbor 新建目标仓库：系统管理——仓库管理——新建目标，菜单中新建目标并做测试连接，发现连接失败。

![harbor-push-connect.png](https://s2.loli.net/2024/08/01/YBxFZuls3DqeAaP.jpg)

## 排查

测试环境由于只有内网地址，需要通过配置 NAT 公网映射打通网络。
数据走向：39.134.174.82（A 部门harbor 公网地址）-->39.156.3.216:10443（测试环境 harbor 公网映射地址）-->测试环境 nginx 反向代理：10.193.40.16:10443-->测试环境 harbor：10.193.40.13:8043

分别在 A 部门 harbor 所在机器进行 telnet、curl 测试及防火墙侧抓包排查，均无异常。

 - telnet 测试时抓包
   ![harbor-push-tcpdump.png](https://s2.loli.net/2024/08/01/WeitT8UIXYwVq4u.jpg)
 - curl 测试，返回正常
   ![harbor-push-curl.png](https://s2.loli.net/2024/08/01/NyjS4XmIacUfYFW.jpg)
 - 做 NAT 映射的防火墙抓包，收到了 A 部门harbor 的请求，建连成功
   ![harbor-push-firewall.png](https://s2.loli.net/2024/08/01/gRBP8Sd1NxizfQn.jpg)








