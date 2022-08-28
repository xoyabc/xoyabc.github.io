---
layout: post
title: github markdown 文档图片异常显示问题
categories: linux
description: github的 md 文档，嵌入的图片外链无法正常显示，直接访问图片链接正常
keywords: python, pexpect
---

github的 md 文档，嵌入的图片外链无法正常显示，直接访问图片链接正常。

github 预览图片显示碎裂，效果见下：

[![vWvhQK.png](https://s1.ax1x.com/2022/08/29/vWvhQK.png)](https://imgse.com/i/vWvhQK)


## 分析

github 使用 Camo 代理显示图片时，而 camo 只支持 image 类型的 `Content-Type`，详见：[the list of types supported by Camo](https://github.com/atmos/camo/blob/master/mime-types.json)

使用 `curl` 检查图片链接返回的  `Content-Type`

```bash
# curl -i https://img1.test.com/cn/1.server-manager.jpg
HTTP/1.1 302 Moved Temporarily
Date: Tue, 12 Jul 2022 19:26:05 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Server: nginx
Location: https://img1.test.com/cn/1.server-manager.jpg
X-Trace: 302-1657653965433-0-0-0-0-0
Strict-Transport-Security: max-age=3600
X-Via: 1.1 hx172:1 (Cdn Cache Server V2.0), 1.1 am20:10 (Cdn Cache Server V2.0)
X-Ws-Request-Id: 62cdcacd_PSmgmamMIA2dr149_2157-48853
```

返回的 `Content-Type` 为 text/html，不在 camo 的支持范围内，导致无法显示。


## 解决

由于 img1.test.com 走了 CDN，联系第三方 CDN 修改返回的 `Content-Type` header 头为 `image/jpeg`。修改后查看 md 文档，图片可以正常显示。

## REF

[about-anonymized-urls](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-anonymized-urls)

