---
layout: post
title: p7b格式证书转换为crt
categories: work
description: p7b格式证书转换为crt
keywords: crt, certificate 
---

近期在配置https工单时，遇到两起客户提供的证书为p7b格式的文件，需要手动将其以base64编码后逐个导出，之后再拼接证书内容并改扩展名为crt，过程较繁琐。

为提升效率，总结了p7b转换crt的方法，步骤如下:

以1.a.com.p7b证书为例，转换为1.a.com.crt
 
1. 运行fold命令转换格式
```
fold -w 64 1.a.com.p7b > temp.p7b
```
2. 使用OPENSSL将p7b转换为crt
```
openssl pkcs7 -print_certs -in temp.p7b |grep -Ev '^\s*$|subject|issuer' > 1.a.com.crt
```
对应脚本

```bash
#!/bin/bash
p7b_file="$1"
p7b_filename=$(echo ${p7b_file} |sed -r 's#(.*).p7b#\1#g')
usage ()
{
    echo "Usage:sh $0 p7b_file"
    exit 0
}
[ $# -ne 1 ] && usage
fold -w 64 ${p7b_file} > temp.p7b
openssl pkcs7 -print_certs -in temp.p7b |grep -Ev '^\s*$|subject|issuer' > ${p7b_filename}.crt
```

* 使用方法
`sh p7b_to_crt.sh p7b文件`

* 实际用例
`sh p7b_to_crt.sh owner1a_520wdy_com.p7b`

    生成的crt在当前目录下，与p7b文件同名。

![1.jpg](https://i.loli.net/2018/05/19/5affe37cd1984.jpg)
