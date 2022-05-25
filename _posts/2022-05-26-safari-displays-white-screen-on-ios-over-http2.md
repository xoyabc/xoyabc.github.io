---
layout: post
title: HTTP2 非法头部导致苹果手机访问白屏
categories: http
description: HTTP2 非法头部导致苹果手机访问白屏
keywords: http, ios
---

## 故障现象

用户反馈苹果手机浏览器打开网页失败，页面显示报错“无法解析响应”

URL：
> https://s.test.com/protocols/format/66bd0c66a96d56a8b33b11b664451337.html?_t=1632403876589

[![XAWrCj.png](https://s1.ax1x.com/2022/05/26/XAWrCj.png)](https://imgtu.com/i/XAWrCj)

## 排查

### 绑定源站测试

由于该域名经过 CDN 加速，首先绑定源站测试，测试结果显示正常

[![Xinmi8.png](https://s1.ax1x.com/2022/05/24/Xinmi8.png)](https://imgtu.com/i/Xinmi8)

## 绑定 CDN 节点测试
电脑绑定 CDN 节点的 host，同时开启 fiddler 抓包，手机连接电脑发射的热点，打开报障的 URL，可以复现到现象。
从抓包数据看，请求协议为 HTTP2。

使用 curl 模拟请求，其中 114.230.205.224 为 CDN IP

```
curl -Lvo /dev/null --http2 "https://s.test.com/protocols/format/66bd0c66a96d56a8b33b11b664451337.html?_t=163163163163" --resolve s.test.com:443:114.230.205.224
```

[![XAW6vq.md.png](https://s1.ax1x.com/2022/05/26/XAW6vq.md.png)](https://imgtu.com/i/XAW6vq)

可以看到 `Invalid HTTP header field` 的报错，具体报错见下：
```bash
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
* http2 error: Invalid HTTP header field was received: frame type: 1, stream: 1, name: [proxy-connection], value: [keep-alive]
* HTTP/2 stream 0 was not closed cleanly: PROTOCOL_ERROR (err 1)
* stopped the pause stream!
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
* Connection #0 to host s.test.com left intact
curl: (92) HTTP/2 stream 0 was not closed cleanly: PROTOCOL_ERROR (err 1)
* Closing connection 0
```

猜测为 `proxy-connection` 头 与 HTTP2 兼容有问题，后查看 RFC 文档 [8.1.2.2. Connection-Specific Header Fields
](https://httpwg.org/specs/rfc7540.html#rfc.section.8.1.2.2) 得知，HTTP2 不使用 `Connection` header 头，任何带有特定连接的 header 头都会被视为格式错误，例如 Keep-Alive, Proxy-Connection, Transfer-Encoding, and Upgrade。当中间层将 HTTP/1.x 消息转换为 HTTP2 时，需要移除这些头。

```
HTTP/2 does not use the Connection header field to indicate connection-specific header fields; in this protocol, connection-specific metadata is conveyed by other means. An endpoint MUST NOT generate an HTTP/2 message containing connection-specific header fields; any message containing connection-specific header fields MUST be treated as malformed
......
This means that an intermediary transforming an HTTP/1.x message to HTTP/2 will need to remove any header fields nominated by the Connection header field, along with the Connection header field itself. Such intermediaries SHOULD also remove other connection-specific header fields, such as Keep-Alive, Proxy-Connection, Transfer-Encoding, and Upgrade, even if they are not nominated by the Connection header field.
```

## 解决

1，联系 CDN 厂商，当响应 HTTP/2 消息时，移除掉 Proxy-Connection、Keep-Alive、Transfer-Encoding、Upgrade 响应头。

2，联系 CDN 厂商修改配置，不缓存 Proxy-Connection、Keep-Alive、Transfer-Encoding、Upgrade 响应头。

## REF

[http2.0非法头部导致iphone访问白屏](https://cloud.tencent.com/developer/article/1754005)
