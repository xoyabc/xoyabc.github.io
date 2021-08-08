---
layout: post
title: 记一次业务机器文件句柄占用过高问题
categories: linux
description: 记一次业务机器文件句柄占用过高问题
keywords: linux
---


### 现象

监控显示某业务机器系统文件句柄使用量大于 15000

![fd-used-too-high.png](https://i.loli.net/2021/08/08/p6Un14HRtbEPCdz.png)

### 排查

登录业务机器，执行以下命令找出打开文件数最多的进程

> lsof -n |awk '{num[$2]++}END{for(i in num) print num[i],i}' |sort -nr |head 

确认进程号为 `11342`，执行 `lsof -p 11342` 查看句柄占用信息，发现有大量 `CLOSE_WAIT` 状态的 TCP 连接

```shell
COMMAND     PID  USER   FD   TYPE            DEVICE       SIZE/OFF                NODE NAME
appserver  11342 root  224u  IPv4            2940079        0t0       TCP  2zqw0uq5rrsZ:9702->192.168.121.44:https (CLOSE_WAIT)
appserver  11342 root  225u  IPv4            2934562        0t0       TCP  2zqw0uq5rrsZ:55478->192.168.120.181:https (CLOSE_WAIT)
appserver  11342 root  226u  IPv4            2946272        0t0       TCP  2zqw0uq5rrsZ:9856->192.168.121.44:https (CLOSE_WAIT)
appserver  11342 root  227u  IPv4            2943998        0t0       TCP  2zqw0uq5rrsZ:55622->192.168.120.181:https (CLOSE_WAIT)
appserver  11342 root  228u  IPv4            2947267        0t0       TCP  2zqw0uq5rrsZ:9868->192.168.121.44:https (CLOSE_WAIT)
appserver  11342 root  229u  IPv4            3016704        0t0       TCP  2zqw0uq5rrsZ:11326->192.168.121.44:https (CLOSE_WAIT)
appserver  11342 root  230u  IPv4            3025554        0t0       TCP  2zqw0uq5rrsZ:57236->192.168.120.181:https (CLOSE_WAIT)
```

我们先看一下 TCP 四次挥手过程

![tcp-4-way-handshake.png](https://i.loli.net/2021/08/08/OwtzZYJhlom3n7y.png)

可以看到，`COLSE_WAIT` 为四次挥手中被动关闭方其中的一个状态，表示被动关闭方等待关闭。一般被动关闭方为服务器。

```plain
当客户端 close() 一个 SOCKET 后发送 FIN 报文给服务器，服务器毫无疑问地将会回应一个 ACK 报文给对客户端，此时 TCP 连接则进入到
CLOSE_WAIT 状态。接下来呢，服务器需要检查自己是否还有数据要发送给客户端，如果没有的话，那服务器也就可以 close() 这个 SOCKET 
并发送 FIN 报文给客户端，即关闭自己到客户端方向的连接。有数据的话则看应用程序的策略，继续发送或丢弃。

简单地说，当服务器处于 CLOSE_WAIT 状态下，需要完成的事情是等待应用程序去关闭连接。
```
  
而服务器有大量 CLOSE_WAIT 状态的 SOCKET ，说明应用程序没有及时关闭连接，也就是没有发送 FIN 包给客户端。


### 解决

联系研发排查，确认为程序未关闭连接，导致句柄泄露，会在下个版本中修复。

### REF

[关于close_wait状态的理解](https://blog.csdn.net/qq_39382769/article/details/90703382)

[TCP三次握手/四次挥手 及 状态变迁图](https://blog.csdn.net/pmt123456/article/details/56677578)

[TCP/IP详解--连接状态变迁图CLOSE WAIT](https://blog.csdn.net/hfhhgfv/article/details/84064230)

