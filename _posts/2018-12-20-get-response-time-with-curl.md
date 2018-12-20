---
layout: post
title: 使用curl获取网络请求中各个环节的响应时间
categories: linux
description: 使用curl获取网络请求中各个环节的响应时间
keywords: linux
---

在处理网站打开慢 case 时，可以使用 curl 命令分别获取各个环节的用时，从而定位问题。

## 参数含义

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
 
 建连时间，包括DNS解析时间
 
 - time_starttransfer
 
从建立TCP连接，到WEB服务器返回第一个字节数据所用的时间，包括建连、包括DNS解析时间。
 
 - time_total
 
 请求所用总时间。从client发出请求，到WEB服务器发送所有响应数据的时间
 
 - time_namelookup
 
 DNS解析时间，从请求开始到DNS解析完毕所用时间

其他参数含义：

- -s

静默输出

- -o

将输出保留在文件中，这里为保存到/dev/null，即空设备，不占用磁盘空间。


## 实例分析

近日，公司有同事反馈通过代理访问外网特别慢，遂在代理服务器上直接访问外网，查看各个响应时间，发现 DNS 解析耗时较长。

### 访问百度
![curl-get-total-time.png](https://raw.githubusercontent.com/xoyabc/xoyabc.github.io/master/images/blog/curl-get-total-time.png)

### 下载大文件

```shell
root@test:~$curl -ksvo /dev/null https://ys.louxiaohui.com/mysql-bin.000015 -w 'time_connect: %{time_connect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\ntime_namelookup: %{time_namelookup}\n' 
* Hostname was NOT found in DNS cache 
*   Trying 95.169.20.135... 
* Connected to ys.louxiaohui.com (95.169.20.135) port 443 (#0) 
* successfully set certificate verify locations: 
*   CAfile: none 
  CApath: /etc/ssl/certs 
* SSLv3, TLS handshake, Client hello (1): 
} [data not shown] 
* SSLv3, TLS handshake, Server hello (2): 
{ [data not shown] 
* SSLv3, TLS handshake, CERT (11): 
{ [data not shown] 
* SSLv3, TLS handshake, Server key exchange (12): 
{ [data not shown] 
* SSLv3, TLS handshake, Server finished (14): 
{ [data not shown] 
* SSLv3, TLS handshake, Client key exchange (16): 
} [data not shown] 
* SSLv3, TLS change cipher, Client hello (1): 
} [data not shown] 
* SSLv3, TLS handshake, Finished (20): 
} [data not shown] 
* SSLv3, TLS change cipher, Client hello (1): 
{ [data not shown] 
* SSLv3, TLS handshake, Finished (20): 
{ [data not shown] 
* SSL connection using ECDHE-RSA-AES128-GCM-SHA256 
* Server certificate: 
*        subject: CN=ys.louxiaohui.com 
*        start date: 2018-11-28 15:34:14 GMT 
*        expire date: 2019-02-26 15:34:14 GMT 
*        issuer: C=US; O=Let's Encrypt; CN=Let's Encrypt Authority X3 
*        SSL certificate verify ok. 
> GET /mysql-bin.000015 HTTP/1.1 
> User-Agent: curl/7.35.0 
> Host: ys.louxiaohui.com 
> Accept: */* 
>  
< HTTP/1.1 200 OK 
* Server nginx is not blacklisted 
< Server: nginx 
< Date: Thu, 20 Dec 2018 16:51:00 GMT 
< Content-Type: application/octet-stream 
< Content-Length: 57663862 
< Last-Modified: Thu, 20 Dec 2018 16:42:12 GMT 
< Connection: keep-alive 
< ETag: "5c1bc664-36fe176" 
< Strict-Transport-Security: max-age=15768000 
< Accept-Ranges: bytes 
<  
{ [data not shown] 
* Connection #0 to host ys.louxiaohui.com left intact 
time_connect: 4.670 
time_starttransfer: 5.154 
time_total: 10.213 
time_namelookup: 4.513
```

从访问百度的结果中可得知：

 - DNS解析用时 5.514秒
 
 - 建连用时为 0.002秒，5.516（time_connect）- （time_namelookup）5.514 = 0.002
 
 - 首字节传输用时 0.004秒， 5.520（time_starttransfer）- 5.516（time_connect） = 0.004

 - 请求所用总时间 5.514

### 解决办法

更改DNS地址即可

由于所在运营商为北京联通，根据之前的测试结果，使用查询时间较短的DNS： 202.106.0.20，202.106.195.68

以下为之前测试的北京联通各个DNS的查询时长：

| DNS             | 平均查询时长 |
| --------------- | ------------ |
| 123.125.81.6    | 1            |
| 202.106.0.20    | 2            |
| 202.106.195.68  | 3.67         |
| 123.123.123.123 | 4.33         |
| 117.50.22.22    | 5            |
| 218.30.118.6    | 9.67         |
| 114.114.114.114 | 13.33        |
| 114.114.115.115 | 13.67        |
| 180.76.76.76    | 16.67        |


## REF:

[curl命令测试网络请求中DNS解析、响应时间](https://blog.csdn.net/dreamer2020/article/details/78152576)

[curl 查看一个web站点的响应时间(rt)](https://blog.csdn.net/caoshuming_500/article/details/14044697)

[curl获取响应时间及常用方法](https://blog.csdn.net/jackyzhousales/article/details/82799494)

[全国 DNS 服务器 IP 地址汇总](https://ip.cn/dns.html)

