---
layout: post
title: 安卓手机预装包直播无法观看问题
categories: linux
description: 安卓手机预装包直播无法观看问题
keywords: linux
---

## 问题描述

用户报障使用预装包打开直播间失败，在后台监控系统根据客户 ID 过滤对应时间段的埋点日志：

```
客户端IP：153.153.153.207
节点IP：124.198.198.143
省份：江苏
运营商：联通
URL：https://pull.test.com/live/6666666.flv 
```

## 排查

1，绑定host，使用 VLC 观看测试正常

> 124.198.198.143 pull.test.com

2，观察第三方博睿拨测，查看可用性正常

3，观察江苏地区整体带宽及状态码正常

以上三点可大概率排除服务节点问题，此问题很大可能为个例。后沟通了解到客户在应用市场下载的最新版播放正常，进一步佐证节点服务正常，但也不排除可能为服务端兼容性问题，为进一步排查，让用户提供异常的抓包。

4，使用 wireshark 分析抓包

 - 使用以下命令过滤出 pull.test.com 的 HTTPS 包

> tls.handshake.extensions_server_name == "pull.test.com"

![pre-installed-app-play-failed-1.png](https://s2.loli.net/2022/12/04/6rh31te8YfyqIMg.png)

 - 追踪第一个 50 号包的 TCP 流

![pre-installed-app-play-failed-2.png](https://s2.loli.net/2022/12/04/kRXBjUSwmINe8Hd.png)

 - 数据流详细信息

三次握手正常，说明网络没问题。客户端在发出 client hello 后服务端就抛出异常，之后服务端主动关闭连接。异常错误为：`Description: Protocol Version (70)`。

![pre-installed-app-play-failed-3.png](https://s2.loli.net/2022/12/04/YOEWeT5ALo74ju6.png)

查阅 RFC 文档 [rfc5246#section-7.2.2](https://www.rfc-editor.org/rfc/rfc5246#section-7.2.2)，此错误描述为：

```
The protocol version the client has attempted to negotiate is recognized but not supported.
(For example, old protocol versions might be avoided for security reasons.)  This message is
always fatal.
```

大意为客户端协商的 TLS 版本服务端不支持，比如处于安全原因，应避免使用旧协议版本。

 - 查看客户端请求时发出的 TLS 版本为 1.0

![pre-installed-app-play-failed-4.png](https://s2.loli.net/2022/12/04/Lq2fhmk6HDScFso.png)

综上，基本可以确认是服务端不支持 TLS 1.0。

5，验证服务端是否支持 TLS 1.0

使用 curl 指定协议版本测试，可以看到对端不支持的错误提示：

```shell
curl -vo /dev/null https://pull.test.com/live/6666666.flv  --resolve pull.test.com:443:124.198.198.143  --tlsv1.0
* Added pull.test.com:443:124.198.198.143 to DNS cache
* About to connect() to pull.test.com port 443 (#0)
*   Trying 124.198.198.143...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Connected to pull.test.com (124.198.198.143) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* NSS error -12190 (SSL_ERROR_PROTOCOL_VERSION_ALERT)
* Peer reports incompatible or unsupported protocol version.
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Closing connection 0
curl: (35) Peer reports incompatible or unsupported protocol version.
```

6，微信群联系第三方 CDN 侧，确认其是否支持 TLS 1.0

第三方反馈出于安全考虑，没有支持 tls1.0 和 tls 1.1 ，针对这个配置在走紧急配置流程，不过需要进行审批和配置下发等操作，时间上大约需要 2 个小时左右。

7. 用户使用最新版的包播放并抓包

可以看到使用的协议为 TLS 1.2

![pre-installed-app-play-failed-5.png](https://s2.loli.net/2022/12/04/hvoeVTQrc5YFkb4.png)

## 解决

让第三方 CDN 开启 tls1.0 和 tls 1.1 支持

## REF

[TLS 1.3 Alert Protocol](https://github.com/halfrost/Halfrost-Field/blob/master/contents/Protocol/TLS_1.3_Alert_Protocol.md)

[The TLS Protocol V1.0 - RFC 2246 中文版](http://kipway.com/kipway_TSL.html)





