---
layout: post
title: 在 ubuntu14.04 上使用 sysbench 测试CPU性能
categories: python
description: 使用sysbench测试CPU性能
keywords: linux, cpu
---

使用sysbench测试CPU性能，分别对比AWS，阿里云，腾讯云的数据，为后续选型作参考。

## 安装

```shell
apt-get install sysbench

$dpkg -al |grep sysbench

ii  sysbench                           0.4.12-1.1                                 amd64        Cross-platform and multi-threaded benchmark tool
```
## 常用参数

 - --cpu-max-prime 素数生成数量的上限

```
- 若设置为3，则表示2、3、5（这样要计算1-5共5次）
- 若设置为10，则表示2、3、5、7、11、13、17、19、23、29（这样要计算1-29共29次）
- 默认值为10000
```



