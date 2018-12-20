---
layout: post
title: 使用curl获取网络请求中各个环节的响应时间
categories: linux
description: 使用curl获取网络请求中各个环节的响应时间
keywords: linux
---

在处理网站打开慢 case 时，可以使用 curl 命令分别获取各个环节的用时，从而定位问题。

下面以访问百度为例说明。

```shell
[root@host ~]# curl -so /dev/null http://www.baidu.com -w 'time_connect: %{time_connect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\ntime_namelookup: %{time_namelookup}\n' 
time_connect: 0.010
time_starttransfer: 0.186
time_total: 0.186
time_namelookup: 0.002
```

各个指标含义：

 - time_connect
 
 建连时间
 
 - time_starttransfer
 
 服务器返回首字节所用时间
 
 - time_total
 
 请求所用总时间
 
 - time_namelookup
 
 DNS解析
