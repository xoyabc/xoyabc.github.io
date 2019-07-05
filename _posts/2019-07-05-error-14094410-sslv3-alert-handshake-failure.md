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

将 `3`(ssl v3.0) 改为 `4`(tls v1.0)

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

## 检测网站是否支持 sslv3

使用以下命令，注意需要将 `example.com` 替换为你要检测的网站

```
openssl s_client -connect example.com:443 -ssl3
```

- 不支持

结果中无 `Certificate chain` 或 `Server certificate`

```

# openssl s_client -connect ccfbits.org:443 -ssl3                   
CONNECTED(00000003)
3072198380:error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number:s3_pkt.c:339:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 5 bytes and written 7 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : 0000
    Session-ID: 
    Session-ID-ctx: 
    Master-Key: 
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1562344090
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---

```

- 支持

结果中有 `Certificate chain` 及 `Server certificate`

```

# openssl s_client -connect www.nyt99.com:443 -ssl3                   
CONNECTED(00000003)
depth=0 C = cn, ST = fujian, O = "xiamen chinasource internet service co.,ltd.", OU = vhost, CN = vhost, emailAddress = service@message.zzy.cn
verify error:num=20:unable to get local issuer certificate
verify return:1
depth=0 C = cn, ST = fujian, O = "xiamen chinasource internet service co.,ltd.", OU = vhost, CN = vhost, emailAddress = service@message.zzy.cn
verify error:num=27:certificate not trusted
verify return:1
depth=0 C = cn, ST = fujian, O = "xiamen chinasource internet service co.,ltd.", OU = vhost, CN = vhost, emailAddress = service@message.zzy.cn
verify error:num=21:unable to verify the first certificate
verify return:1
---
Certificate chain
 0 s:/C=cn/ST=fujian/O=xiamen chinasource internet service co.,ltd./OU=vhost/CN=vhost/emailAddress=service@message.zzy.cn
   i:/C=cn/ST=fujian/L=xiamen/O=xiamen chinasource internet service co.,ltd./OU=vhost/CN=vhost/emailAddress=service@message.zzy.cn
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEBDCCA22gAwIBAgIBATANBgkqhkiG9w0BAQQFADCBrTELMAkGA1UEBhMCY24x
DzANBgNVBAgTBmZ1amlhbjEPMA0GA1UEBxMGeGlhbWVuMTUwMwYDVQQKEyx4aWFt
ZW4gY2hpbmFzb3VyY2UgaW50ZXJuZXQgc2VydmljZSBjby4sbHRkLjEOMAwGA1UE
CxMFdmhvc3QxDjAMBgNVBAMTBXZob3N0MSUwIwYJKoZIhvcNAQkBFhZzZXJ2aWNl
QG1lc3NhZ2Uuenp5LmNuMB4XDTEwMDYyMTA4MDUyMFoXDTIwMDYxODA4MDUyMFow
gZwxCzAJBgNVBAYTAmNuMQ8wDQYDVQQIEwZmdWppYW4xNTAzBgNVBAoTLHhpYW1l
biBjaGluYXNvdXJjZSBpbnRlcm5ldCBzZXJ2aWNlIGNvLixsdGQuMQ4wDAYDVQQL
EwV2aG9zdDEOMAwGA1UEAxMFdmhvc3QxJTAjBgkqhkiG9w0BCQEWFnNlcnZpY2VA
bWVzc2FnZS56enkuY24wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBALbWyCO6
ydDHPJlwgoP2xiMsK/J8FqwcL7LH3H93zw8XOqO0FK7frQSb7li3NAXCranWRHOO
CYVk9nbIq+HhJvv7mgyLFxoJDeOk9BuOuB077byIZe3QhjwzlHyWWaVPribdyWbx
9vz40hun0MnwSp3fCce0P4Tj7HzCn+LWQ3MbAgMBAAGjggFBMIIBPTAJBgNVHRME
AjAAMCwGCWCGSAGG+EIBDQQfFh1PcGVuU1NMIEdlbmVyYXRlZCBDZXJ0aWZpY2F0
ZTAdBgNVHQ4EFgQUmG9CVMBPB8lvX/5LA2DJCxovMaYwgeIGA1UdIwSB2jCB14AU
+kLtae7Gnbsnbufm1uZ3jIqb+x6hgbOkgbAwga0xCzAJBgNVBAYTAmNuMQ8wDQYD
VQQIEwZmdWppYW4xDzANBgNVBAcTBnhpYW1lbjE1MDMGA1UEChMseGlhbWVuIGNo
aW5hc291cmNlIGludGVybmV0IHNlcnZpY2UgY28uLGx0ZC4xDjAMBgNVBAsTBXZo
b3N0MQ4wDAYDVQQDEwV2aG9zdDElMCMGCSqGSIb3DQEJARYWc2VydmljZUBtZXNz
YWdlLnp6eS5jboIJAON9ga1mXq26MA0GCSqGSIb3DQEBBAUAA4GBAG65XKRMFw56
tcdJ76WAVF3ZqikGZmJdg5b7FzusQKGNhk97Q5CF3nxHJxQz4MWfU9O5ZG/39xmI
hbpeXPOyTpF9sJpdOPymbV8zlNrMx5gwXyMtNzM8QnZ2Qk0W2jd0NctsonytT/cN
9RAJ9yDa9K1cAlUElsZECIXhKahy9fm3
-----END CERTIFICATE-----
subject=/C=cn/ST=fujian/O=xiamen chinasource internet service co.,ltd./OU=vhost/CN=vhost/emailAddress=service@message.zzy.cn
issuer=/C=cn/ST=fujian/L=xiamen/O=xiamen chinasource internet service co.,ltd./OU=vhost/CN=vhost/emailAddress=service@message.zzy.cn
---
No client certificate CA names sent
Server Temp Key: ECDH, prime256v1, 256 bits
---
SSL handshake has read 1425 bytes and written 274 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-SHA
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : SSLv3
    Cipher    : ECDHE-RSA-AES256-SHA
    Session-ID: A13CFBAAAD31B8C02E63CEA9886967D2E4114C0AD9DD1A563BF7FE4945E3C193
    Session-ID-ctx: 
    Master-Key: F15CAF0DEB2E0B122BD6A54AD5A266BD21C5BAFF24AAFDE7F0A65D83CB91FDE0B2824F977E4AFFE0E66FAEC063D77FEB
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1562344543
    Timeout   : 7200 (sec)
    Verify return code: 21 (unable to verify the first certificate)

```

## REF

[function.curl-setopt](https://php.net/manual/en/function.curl-setopt.php)

[tls-1-2-not-working-in-curl](https://stackoverflow.com/questions/30145089/tls-1-2-not-working-in-curl)

[php-35-error14094410ssl-routinesssl3-read-bytessslv3-alert-handshake-failur](https://stackoverflow.com/questions/23568539/php-35-error14094410ssl-routinesssl3-read-bytessslv3-alert-handshake-failur)

[when-was-tls-1-2-support-added-to-openssl](https://stackoverflow.com/questions/48178052/when-was-tls-1-2-support-added-to-openssl)

[how-to-disable-sslv3-in-linux](https://bobcares.com/blog/how-to-disable-sslv3-in-linux/)
