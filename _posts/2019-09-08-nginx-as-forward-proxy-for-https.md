---
layout: post
title: nginx添加第三方模块支持代理 HTTPS 请求
categories: linux
description: nginx使用ngx_http_proxy_connect_module模块，使其能处理 CONNECT 请求
keywords: linux, CONNECT
---


最近新上线一个服务，访问时走的https协议，由于有访问外网的需求，需要走代理访问，但测试时发现接口返回到目标server超时，猜测未走代理访问所致，便使用 curl 模拟测试是否是代理问题。

命令及返回结果如下：

```bash

curl -v -l -H "Content-type:application/json" -X POST -d '{post data}' https://service_dommain/urlserver/long2short/create -x '192.168.1.1:10101'

* Hostname was NOT found in DNS cache
*   Trying 192.168.1.1...
* Connected to 192.168.1.1 (192.168.1.1) port 10101 (#0)
* Establish HTTP proxy tunnel to service_domain:443
> CONNECT service_domain:443 HTTP/1.1
> Host: service_domain:443
> User-Agent: curl/7.35.0
> Proxy-Connection: Keep-Alive
> Content-type:application/json
> 
< HTTP/1.1 400 Bad Request
< Server: openresty/1.13.6.2
< Date: Sun, 08 Sep 2019 14:27:41 GMT
< Content-Type: text/html
< Content-Length: 179
< Connection: close
< 
* Received HTTP code 400 from proxy after CONNECT
* Connection #0 to host 192.168.1.1 left intact
curl: (56) Received HTTP code 400 from proxy after CONNECT

```

结果又发现 `curl: (56) Received HTTP code 400 from proxy after CONNECT` 的报错，之后经谷歌查询，原因为 nginx 作为代理使用时，不支持处理 CONNECT 方法的请求。

请求过程如下：

1，与代理服务器 192.168.1.1 的10101端口建连(tcp)

2，由于请求地址协议为HTTPS，curl 发送 CONNECT 方法请求来建立隧道，以便进行后续的SSL通信

3，nginx 作为代理时，不支持处理 CONNECT 方法的请求，于是返回 400 错误码。

不过可以安装第三方模块，使 nginx 支持处理 HTTPS 请求，下面详细说下安装及配置方法。

## 使用第三方模块

参考： https://github.com/chobits/ngx_http_proxy_connect_module#install

### 安装
以 openresty 1.13.6.2 为例。

```
# 克隆项目到本地
cd /opt/
git clone https://github.com/chobits/ngx_http_proxy_connect_module.git

# 进入之前安装时解压的源码目录
cd openresty-1.13.6.2/
./configure --add-module=/opt/ngx_http_proxy_connect_module
patch -p 1 < /opt/ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_1014.patch
make -j 4
make -j 4 install 
```

### 配置 vhost

server {
    listen 10101 default_server;
    # dns resolver used by forward proxying
    resolver                       8.8.8.8;
    resolver_timeout 3s;
	# forward proxy for CONNECT request
    proxy_connect;
    root /data/www;
    index index.html;

    # forward proxy for non-CONNECT request
    location / {
            proxy_pass http://$http_host$request_uri;
			proxy_set_header Host $http_host;
    }
}

### 测试

```bash
curl -v -l -H "Content-type:application/json" -X POST -d '{post_data}' https://service_domain/urlserver/long2short/create -x '192.168.1.1:10101'                                                                                             
* Hostname was NOT found in DNS cache
*   Trying 192.168.1.1...
* Connected to 192.168.1.1 (192.168.1.1) port 10101 (#0) # 连接代理服务器
* Establish HTTP proxy tunnel to service_domain:443     
> CONNECT service_domain:443 HTTP/1.1      # 发送 CONNECT 请求，代理服务器与源服务器(service_domain的源IP)建立隧道
> Host: service_domain:443
> User-Agent: curl/7.35.0
> Proxy-Connection: Keep-Alive
> Content-type:application/json
> 
< HTTP/1.1 200 Connection Established  #nginx代理服务器返回200状态码，表示连接成功建立
< Proxy-agent: nginx
< 
* Proxy replied OK to CONNECT request
* successfully set certificate verify locations:
*   CAfile: none
  CApath: /etc/ssl/certs
* SSLv3, TLS handshake, Client hello (1):
* SSLv3, TLS handshake, Server hello (2):
* SSLv3, TLS handshake, CERT (11):
* SSLv3, TLS handshake, Server key exchange (12):
* SSLv3, TLS handshake, Server finished (14):
* SSLv3, TLS handshake, Client key exchange (16):
* SSLv3, TLS change cipher, Client hello (1):
* SSLv3, TLS handshake, Finished (20):
* SSLv3, TLS change cipher, Client hello (1):
* SSLv3, TLS handshake, Finished (20):
* SSL connection using ECDHE-RSA-AES256-GCM-SHA384
* Server certificate:
...
...
*        subjectAltName: service_domain matched
*        issuer: C=US; O=DigiCert Inc; OU=www.digicert.com; CN=GeoTrust RSA CA 2018
*        SSL certificate verify ok.
> POST /urlserver/long2short/create HTTP/1.1
> User-Agent: curl/7.35.0
> Host: service_domain
> Accept: */*
> Content-type:application/json
> Content-Length: 81
> 
* upload completely sent off: 81 out of 81 bytes
< HTTP/1.1 200 OK
< Date: Sun, 08 Sep 2019 15:41:34 GMT
* Server beegoServer:1.4.3 is not blacklisted
< Server: beegoServer:1.4.3
< Content-Length: 184
< Content-Type: application/json
< Via: 1.1 service_domain
< 
* Connection #0 to host 192.168.1.1 left intact
{"status":0,"result":{"shortUrl":"https://dwz.cn/6666666"},"content_type":"application/json"}
```

## HTTP 代理

HTTP代理通常使用 CONNECT 方法，也可以使用 POST, GET, PUT 和 DELETE 方法，下面分别说明

### CONNECT 方法

- RFC7231 中的 CONNECT 方法

CONNECT 请求使代理服务器建立一条到目标服务器的隧道，成功建立后，请求会**保持原样**，并转发给目标服务器，当通信双方断开连接时，隧道也随之关闭。隧道通常通过一个或多个代理，建立一条端到端的虚拟线路，届时便可以使用SSL进行通信。

CONNECT 请求是专为请求代理服务器设计的。**源服务器**收到 CONNECT 请求时，会返回一个 2xx 的状态码，表示连接成功建立。然而，大部分**源服务器**并不支持。（注：这里的**源服务器**应为代理服务器）

第二段中，RFC文档中原文如下：

```palin
  CONNECT is intended only for use in requests to a proxy.  An origin
   server that receives a CONNECT request for itself MAY respond with a
   2xx (Successful) status code to indicate that a connection is
   established.  However, most origin servers do not implement CONNECT.
```

- RFC7540 中的 CONNECT 方法

在HTTP/1.x中，伪方法 CONNECT 用于将 HTTP 连接转换成到远程主机的隧道。CONNECT 主要用被 HTTP 代理用来建立与源服务器(目标服务器)的TLS会话，以便请求https资源。

在 HTTP/2 中，不再有专门的 CONNECT 方法。通过将 ":method"伪头部字段设为 "CONNECT"，来在一个HTTP/2流上建立一个隧道

```plain
伪头部字段是http2内置的几个特殊的以”:”开始的key，用于替代HTTP/1.x中请求行/响应行中的信息，比如请求方法，响应状态码等。
# HTTP/1.1 及 HTTP/2 请求对照
GET /resource HTTP/1.1           HEADERS
Host: example.org          ==>     + END_STREAM
Accept: image/jpeg                 + END_HEADERS
                                        :method = GET
                                        :scheme = https
                                        :path = /resource
                                        host = example.org
                                        accept = image/jpeg
# HTTP/1.1 及 HTTP/2 响应对照
HTTP/1.1 304 Not Modified        HEADERS
ETag: "xyzzy"              ==>     + END_STREAM
Expires: Thu, 23 Jan ...           + END_HEADERS
                                     :status = 304
                                     etag = "xyzzy"
                                     expires = Thu, 23 Jan ...
```

### 非 CONNECT 方法

通过 POST, GET, PUT 和 DELETE 方法实现 HTTP 代理，因此大多数代理防火墙均可以实现。

过程如下：

1，客户端发起请求待代理

2，代理请求源服务器

3，代理服务器收到源服务器返回的响应

4，代理服务器将响应发送给客户端

### 两种方法的请求日志

测试机IP：192.168.1.88

代理服务器IP：198.222.241.219


- CONNECT 

请求链接为 `https://service_domain/urlserver/long2short/create`（https）

> curl -v -l -H "Content-type:application/json" -X POST -d '{post_data}' https://service_domain/urlserver/long2short/create -x '192.168.1.1:10101' -A 'UA_CONNECT'

- POST

请求链接为 `http://service_domain/urlserver/long2short/create`（http）

> curl -v -l -H "Content-type:application/json" -X POST -d '{post_data}' http://service_domain/urlserver/long2short/create -x '192.168.1.1:10101' -A 'UA_WITHOUT_CONNECT'

![nginx-forward-1.jpg](https://i.loli.net/2019/09/09/oFrEA5ksuGcbwH3.jpg)

CONNECT 时，代理服务器日志中 request 字段记录的是使用 CONNECT 与源服务器建立连接的URL，即 `访问域名:443` ，源服务器日志的 x_forwarded_for 字段中，记录的只有代理服务器IP  `198.222.241.219`

POST 时，代理服务器日志中 request 字段记录的是完整请求URL，源服务器日志的 x_forwarded_for 字段中，记录的是测试服务器IP和我代理服务器IP `192.168.1.88, 198.222.241.219`

## REF

[nginx-as-forward-proxy-for-https](https://superuser.com/questions/604352/nginx-as-forward-proxy-for-https)

[ngx_http_proxy_connect_module](https://github.com/chobits/ngx_http_proxy_connect_module) 

[HTTP_CONNECT_tunneling](https://en.wikipedia.org/wiki/HTTP_tunnel#HTTP_CONNECT_tunneling)

[the rfc7231 CONNECT method request](https://tools.ietf.org/html/rfc7231#section-4.3.6)

[the rfc7540 CONNECT method request](https://httpwg.org/specs/rfc7540.html#CONNECT)

[HTTP2伪头部字段](https://blog.csdn.net/yangguosb/article/details/80650692)

[rfc7540中文翻译](https://github.com/abbshr/rfc7540-translation-zh_cn/blob/master/8-zh-cn.md)
