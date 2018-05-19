---
layout: post
title: 使用find命令查找多个文件名
categories: linux
description: 使用find命令同时查找多个文件名
keywords: shell, 常用命令
---

使用find命令同时查找多个文件名，适用于查找两种或以上扩展名文件。

## 查找前目录下的所有.gz和.log结尾的文件

### 当前目录文件列表
```bash
[root@test test1]# ll
总用量 0
-rw-r--r-- 1 root root 0 4月  24 10:58 1
-rw-r--r-- 1 root root 0 4月  26 12:28 1.gz
-rw-r--r-- 1 root root 0 4月  26 12:28 1.gz.log
-rw-r--r-- 1 root root 0 4月  24 10:58 2
-rwxr-xr-x 1 root root 0 4月  26 12:53 c
```

### 命令如下：
```bash
# 使用-o
[root@test test1]# find . -type f \( -name "*.gz" -o -name "*.log" \)
./1.gz.log
./1.gz
[root@test test1]# find . -type f -iname "*.log" -o -iname "*.gz"            
./1.gz.log
./1.gz
# 使用默认的正则方式
[root@test test1]# find . -type f -regex '.*\(\.gz\|\.log\)'
./1.gz.log
./1.gz
# 采用posix-extended正则
[root@test test1]#  find . -type f -regextype posix-extended -regex '.*.(log|gz)'
./1.gz.log
./1.gz
```

