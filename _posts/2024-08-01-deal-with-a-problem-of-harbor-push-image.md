---
layout: post
title: 记一次 harbor 推送镜像失败问题
categories: k8s
description: 公网 harbor 往内网测试环境 harbor 推送镜像失败问题排查
keywords: Linux, docker, harbor
---

## 问题
为实现统一管理，镜像需要交由 A 部门集中构建，之后再推送到测试环境 harbor 仓库。
在A 部门 harbor 新建目标仓库：系统管理-->仓库管理-->新建目标，菜单中新建目标并做测试连接，发现连接失败。

![harbor-push-connect.png](https://s2.loli.net/2024/08/01/YBxFZuls3DqeAaP.jpg)

## 排查

测试环境由于只有内网地址，需要通过配置 NAT 公网映射打通网络。

访问走向：39.134.174.82（A 部门harbor 公网地址）-->39.156.3.216:10443（测试环境 harbor 公网映射地址）-->测试环境 nginx 反向代理：10.193.40.16:10443-->测试环境 harbor：10.193.40.13:8043

1，分别在 A 部门 harbor 所在机器进行 telnet、curl 测试及防火墙侧抓包排查，均无异常。

 - telnet 测试时抓包，回包正常
   ![harbor-push-tcpdump.png](https://s2.loli.net/2024/08/01/WeitT8UIXYwVq4u.jpg)
 - curl 测试，返回正常
   ![harbor-push-curl.png](https://s2.loli.net/2024/08/01/NyjS4XmIacUfYFW.jpg)
 - 做 NAT 映射的防火墙抓包，收到了 A 部门harbor 的请求，建连成功
   ![harbor-push-firewall.png](https://s2.loli.net/2024/08/01/gRBP8Sd1NxizfQn.jpg)

2，怀疑中间的 ngnix 的反向代理有问题，又新申请了公网直接映射到测试环境 harbor 的网络策略。
   
   测试后问题依然存在。
   变更后访问走向：39.134.174.82（A 部门harbor 公网地址）-->39.156.3.216:8043（测试环境 harbor 公网映射地址）-->测试环境 harbor：10.193.40.13:8043

3，A 部门确认 harbor 所在机器 /etc/docker/daemon.json 配置已经添加 39.156.3.216:10443 为受信任地址
```bash
#  cat /etc/docker/daemon.json 
{
  "insecure-registries":["39.156.3.216:8043"],
}
```

4，协调 A 部门人员使用 docker login 登录测试
   
   显示超时报错
   ![harbor-push-docker-login.png](https://s2.loli.net/2024/08/01/XVuYbxprFP7JGTi.jpg)
   
   报错的第二个链接 https://10.193.40.13:8043 为内网地址，该地址为 ./harbor/common/config/core/env 配置文件的 EXT_ENDPOINT 参数值，EXT_ENDPOINT 表示 harbor 对外提供的访问地址。
   原因基本已经确认，由于返回的内网地址，公网到内网地址不通，通过将 EXT_ENDPOINT 改成域名 + host绑定公网地址可解决此问题。

## 解决

EXT_ENDPOINT 改成域名并绑定 host

### A 部门所在 harbor 机器修改

1， 修改 /etc/hosts 文件，新增域名解析：

39.156.3.216 harbor.test.com

2，修改 /etc/docker/daemon.json 配置文件，并重启docker：

insecure-registries 部分添加 "harbor.test.com:10443" 配置
![harbor-push-daemon-json-A-dev.png](https://s2.loli.net/2024/08/01/tkJLmMYBDhbOTuK.jpg)
重启 docker ：systemctl restart docker

3，再次新建目标测试连接，正常

![harbor-push-connect-succeed.png](https://s2.loli.net/2024/08/01/HWy9sJhFdKqvmCL.jpg)


### 测试环境内网机器修改

由于测试环境内网机器也用到 harbor 推送镜像，同样也需要修改 docker 配置及 host 文件。

1，修改harbor安装目录下~/harbor/common/config/core/env 文件，并重启harbor

EXT_ENDPOINT 改为 https://harbor.test.com:10443
![harbor-push-harbor-env.png](https://s2.loli.net/2024/08/01/9UA8FcrVvSDbThy.jpg)

重启 harbor：
```bash
docker-compose stop
docker-compose up -d
```
2，修改 /etc/hosts 文件，添加域名解析：

10.193.40.16 harbor.test.com

注意这里为内网 nginx 代理地址

3，修改 /etc/docker/daemon.json 配置文件，并重启docker：

insecure-registries 部分添加 "harbor.test.com:10443" 配置
![harbor-push-daemon-json-A-dev.png](https://s2.loli.net/2024/08/01/tkJLmMYBDhbOTuK.jpg)
重启 docker ：systemctl restart docker

## REF

[Harbor介绍](https://t.goodrain.com/d/8204-dockerharborrainbond)

[harbor登录时报错error parsing HTTP 404 response body: invalid character](https://www.cnblogs.com/muzlei/p/17748915.html)



