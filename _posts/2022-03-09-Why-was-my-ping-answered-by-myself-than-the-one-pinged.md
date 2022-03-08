---
layout: post
title: 为什么 ping 目标主机对方却收不到包?
categories: python
description: 同网段内ping另一主机，目标主机收不到包，实际却是本机在回包。
keywords: linux
---

## 问题

直播业务中转集群节点间设备会互相拉取 hls ts 切片及接收转码集群推送的转码切片流，原来走的是公网，为节省成本，需要走专线，配置上专线IP后：

 - 中转-->转码集群，相互访问正常
 - 中转 A-->中转 B，节点间互拉异常

在 A 主机上，使用 curl 请求 B 主机，分别查看 A/B 主机的 nginx 访问日志，发现请求打到了 A 主机，curl 命令及日志见下：

```shell
# 请求
curl -svo /dev/null http://11.174.20.83/cdn-mon/monitor.html -H "Host: cdn-mon.jcloud.com"  -A lxhlxhlxh

# 日志
其中 11.174.20.81 为客户端IP，11.174.20.83 为服务端IP
2022-03-09T02:37:12+08:00   :1646764632.579   :-   :11.174.20.81   :11.174.20.83   :GET   :http   :cdn-mon.jcloud.com   :/cdn-mon/monitor.html   :/cdn-mon/monitor.html   :HTTP/1.1   :200   :lxhlxhlxh   :END
```





