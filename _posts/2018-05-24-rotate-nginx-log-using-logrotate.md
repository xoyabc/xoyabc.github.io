---
layout: post
title: 使用logrotate切割nginx日志
categories: nginx
description: nginx默认不会对日志进行切割，为防止单个日志文件过大，需对其进行切割。
keywords: linux, nginx
---

nginx默认不会对日志进行切割，为防止单个日志文件过大，需对其进行切割。本文的所有操作均是在`CentOS release 6.8 (Final)`中执行。

## 1. logrotate介绍

logrotate程序是一个日志文件管理工具。用于分割日志文件，删除旧的日志文件，并创建新的日志文件，起到**转储**作用。可以节省磁盘空间。

Linux系统默认安装logrotate工具，它默认的配置文件在：
```shell
/etc/logrotate.conf
/etc/logrotate.d/
```

logrotate.conf 才主要的配置文件，logrotate.conf中的`include /etc/logrotate.d`引入了/etc/logrotate.d目录下的配置文件，类似于nginx中的include。
logrotate.d 是一个目录，该目录里的所有文件都会被主动的读入/etc/logrotate.conf中执行。另外，如果 /etc/logrotate.d/ 里面的文件中没有设定一些细节，则
会以/etc/logrotate.conf这个文件的设定来作为默认值。

logrotate是基于CRON来运行的，其脚本是/etc/cron.daily/logrotate，默认每天执行一次。实际运行时，Logrotate会调用配置文件/etc/logrotate.conf。可以在
/etc/logrotate.d目录里放置自定义好的配置文件，用来覆盖Logrotate的缺省值。

```shell
#!/bin/sh

/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```

## 2. 使用logrotate切割nginx日志文件

### 2.1 目录及日志文件说明

| nginx日志目录  |nginx日志文件 |
| :--------:     | :--------: |
| /var/log/nginx | access.log  error.log |


### 2.2 切割配置文件

```Bash
# cat /etc/logrotate.d/nginx 
/var/log/nginx/*.log {
        daily
        missingok
        rotate 52
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                if [ -f /var/run/nginx.pid ]; then
                        kill -USR1 `cat /var/run/nginx.pid`
                fi
        endscript
}
```

相关参数说明：

 - daily
 
   指定转储周期为每天
 - missingok
 
   如果日志丢失，不报错继续滚动下一个日志
 - rotate 52
 
   保留多少个日志文件(轮转几次).默认保留四个  
   
 - compress
 
   是否通过gzip压缩转储以后的日志文件，如xxx.log-20180524.gz ；如果不需要压缩，注释掉就行
   
 - delaycompress
 
   和compress 一起使用时，转储的日志文件到下一次转储时才压缩。
   ```
   如在2018年5月23号对access.log执行切割，会生成access.log-20180523文件，到24号才会把此文件压缩为access.log-20180523.gz
   ```
   
 - notifempty
 
   当日志文件为空时，不进行轮转
   
 - create 640 nginx adm
 
   指定新建的日志文件权限以及所属的用户和组
   
 - sharedscripts
 
   在所有日志都轮转后统一执行一次脚本。如果没有配置这个，那么每个日志轮转后都会执行postrotate与endscript中间定义的指令。
   该参数适用于对多个日志文件同时进行切割，比如这里的`/var/log/nginx/*.log`匹配到的就是`error.log、access.log`2个日志文件。
   
 - postrotate与endscript 
 
   在logrotate转储之后需要执行的指令，例如重新启动 (kill -HUP) 某个服务！这两个关键字必须独立成行

### 2.3 日志切割后

见下图：

   ![nginx_log_rotate.jpg](https://raw.githubusercontent.com/xoyabc/xoyabc.github.io/master/images/blog/nginx_log_rotate.jpg)


