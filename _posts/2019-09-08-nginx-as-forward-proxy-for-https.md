---
layout: post
title: nginx添加第三方模块支持处理 CONNECT 请求
categories: linux
description: nginx使用ngx_http_proxy_connect_module模块，使其能处理CONNECT请求
keywords: linux, CONNECT
---


## 背景

最近新上线一个服务，由于有访问外网的需求，且访问时走的https协议。实际测试中发现报错为到目标server超时，猜测未走代理访问所致，便使用 curl 模拟测试，结果又发现有 `curl: (56) Received HTTP code 400 from proxy after CONNECT` 的报错，之后经谷歌查询，该报错原因为 nginx默认的正向代理不支持处理 CONNECT 方法的请求。
