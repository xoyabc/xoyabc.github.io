---
layout: post
title: 更换 Apache/Nginx/HAProxy 证书
categories: http
description: 分别阐述如何更换 Apache/Nginx/HAProxy 证书
keywords: linux, http
---


总结一下 Apache/Nginx/HAProxy 更换证书的操作步骤及注意事项。

## 证书信任链 与 证书链

证书信任链一般由 根证书-->中间证书-->根证书 三段构成，只有当整个证书信任链上的各个证书都有效时，浏览器才会认定证书是有效的。

证书链文件是指 用户证书、中间证书和根证书 组成的证书文件，是证书部署和验证证书是否可信任环节中最重要的组成部分。

![certificate_changer_01.jpg](https://i.loli.net/2021/07/22/k7Q9WzVKxsSt53E.jpg)


 - 根证书（CA）

受信任CA证书颁发机构给自己颁发的证书,是信任链的起始点。浏览器判断根证书是否受信任主要是通过检索根证书是否在浏览器的根证书库可信任列表里。

 - 中间证书（二级CA）
根证书颁发机构CA对中间证书颁发机构的公钥进行数字签名得到的证书。中间证书的作用主要是为了保护根证书。因为如果直接采用根证书签发证书，一旦发生根证书泄露，将造成极大的安全问题。
中间证书可以不止一个，目前我们经常看到有两级中间证书的，原则上中间证书层数越多，根证书越安全。但是一般情况下最多也不超过2级。

- 用户证书
由中间证书CA签发给用户的证书。用户证书由中间证书证明可信。用户证书是浏览器上实际体现和使用的证书。

## 证书格式

证书一般包含证书(扩展名为 crt)及私钥（扩展名为 key）

### crt 证书

这里以包含证书链的完整证书为例，一般为三段，从上至下依次为：
用户证书(crt)--中间证书(chain)–-根证书(root)（根证书可无）

crt 文件

```plain
# 第一段：用户证书
-----BEGIN CERTIFICATE-----
2WI5dtWBq5eyd8GfslHp
-----END CERTIFICATE-----
# 第二段：中间证书
-----BEGIN CERTIFICATE-----
dGhpcyBDZXJ0aWZpY2F0ZSBjb25z
-----END CERTIFICATE-----
# 第三段：根证书
-----BEGIN CERTIFICATE-----
iGxvDl7I90VUwHwYDVR0jBBgwFoA
-----END CERTIFICATE-----
```

### 私钥

key 文件

```plain
-----BEGIN PRIVATE KEY-----
pQ8tXI7cdSMjQJlWdtqpKxMGZD5X
-----END PRIVATE KEY-----
```

## 更换证书

更换前备份，更换后平滑重启即可。

### Apache 更换证书

证书配置：

```shell
SSLCertificateFile "/etc/apache2/ssl/test.cer"
SSLCertificateKeyFile "/etc/apache2/ssl/test.key"
SSLCertificateChainFile "/etc/apache2/ssl/ca.crt"
```

test.cer 内容为 crt 文件前两段，即 用户证书+中间证书

test.key 即为私钥

ca.crt 内容为 crt 文件第三段，即 根证书


### nginx 更换证书

证书配置：
```shell
ssl_certificate /usr/local/openresty/nginx/conf/ssl/test.cer;
ssl_certificate_key /usr/local/openresty/nginx/conf/ssl/test.key;
```

test.cer 内容为 crt 文件全部内容

test.key 即为私钥

### haproxy 更换证书

需要将证书及密钥文件以顺序拼接在一样来创建pem 文件。

pem文件本质上只是将证书、密钥及证书认证中心证书（可有可无）拼接成一个文件。我们只是简单地将证书及密钥文件并以这个顺序拼接在一样来创建 xip.io.pem 文件。这是 HAProxy 读取SSL证书首选的方式

> sudo cat /etc/ssl/xip.io/xip.io.crt /etc/ssl/xip.io/xip.io.key | sudo tee /etc/ssl/xip.io/xip.io.pem

- **特别注意**

> 中间证书必须要有，否则在部分设备(已知的有小米手机)上打开网站会提示“证书不可信”的安全提示。

![certificate-1.png](https://i.loli.net/2021/07/27/rqFsI3bc8DpZm2l.png)


证书配置：

```shell
crt-base /data/ssl
bind *:443 ssl crt test.pem
```

pem内容为 用户证书(crt) + 私钥（key）+ 中间证书(chain)，按顺序拼接在一起


## REF

 - [什么是证书链？证书链的详细介绍](https://www.anxinssl.com/9801.html)
 
 - [https之证书验证](https://blog.csdn.net/u012852986/article/details/78873387) 
 
 - [linux下使用openssl生成https的crt和key证书](https://www.cnblogs.com/caidingyu/p/11904277.html)

