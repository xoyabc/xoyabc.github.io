---
layout: post
title: php使用curl扩展请求HTTPS链接报sslv3 alert 错误
categories: linux
description: php使用curl扩展请求HTTPS链接报sslv3 alert 错误
keywords: php, curl
---

php使用curl扩展请求HTTPS链接报sslv3 alert 错误

## 报错信息
使用php的curl请求https链接时报`"error:14094410:SSL routines:ssl3_read_bytes:sslv3 alert handshake failure"` 错误

## 原因分析

使用php的curl扩展时，curl_setop的CURLOPT_SSLVERSION取值为3，对应协议为ssl v3，因为之前的POODLE 病毒爆发，许多网站禁用了sslv3（nginx默认是禁用的，ssl_protocols 默认值为TLSv1 TLSv1.1 TLSv1.2;）

## 解决办法

将
```
curl_setopt($ch, CURLOPT_SSLVERSION, 3);   
```

改为

```
curl_setopt($ch, CURLOPT_SSLVERSION, 4); 

```

## CURLOPT_SSLVERSION 取值及含义

- CURL_SSLVERSION_TLSv1_2 需要php版本>=5.5.19
- TLS 1.1 and TLS 1.2 are supported since OpenSSL 1.0.1

```
CURL_SSLVERSION_DEFAULT (0)
CURL_SSLVERSION_TLSv1 (1),
CURL_SSLVERSION_SSLv2 (2), 
CURL_SSLVERSION_SSLv3 (3),
CURL_SSLVERSION_TLSv1_0 (4),
CURL_SSLVERSION_TLSv1_1 (5),
CURL_SSLVERSION_TLSv1_2 (6).
```

## REF

[function.curl-setopt](https://php.net/manual/en/function.curl-setopt.php)

[tls-1-2-not-working-in-curl](https://stackoverflow.com/questions/30145089/tls-1-2-not-working-in-curl)

[php-35-error14094410ssl-routinesssl3-read-bytessslv3-alert-handshake-failur](https://stackoverflow.com/questions/23568539/php-35-error14094410ssl-routinesssl3-read-bytessslv3-alert-handshake-failur)

[when-was-tls-1-2-support-added-to-openssl](https://stackoverflow.com/questions/48178052/when-was-tls-1-2-support-added-to-openssl)

[how-to-disable-sslv3-in-linux](https://bobcares.com/blog/how-to-disable-sslv3-in-linux/)
