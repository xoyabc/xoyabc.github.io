---
layout: wiki
title: Linux/Unix
categories: Linux
description: 类 Unix 系统下的一些常用命令和用法。
keywords: Linux
---

类 Unix 系统下的一些常用命令和用法。

## 实用命令

### fuser

查看文件被谁占用。

```sh
fuser -u .linux.md.swp
```

### id

查看当前用户、组 id。

### lsof

查看打开的文件列表。

> An  open  file  may  be  a  regular  file,  a directory, a block special file, a character special file, an executing text reference, a library, a stream or a network file (Internet socket, NFS file or UNIX domain socket.)  A specific file or all the files in a file system may be selected by path.

#### 查看网络相关的文件占用

```sh
lsof -i
```

#### 查看端口占用

```sh
lsof -i tcp:5037
```

#### 查看某个文件被谁占用

```sh
lsof .linux.md.swp
```

#### 查看某个用户占用的文件信息

```sh
lsof -u mazhuang
```

`-u` 后面可以跟 uid 或 login name。

#### 查看某个程序占用的文件信息

```sh
lsof -c Vim
```

注意程序名区分大小写。

## 安全知识

### ubunbu下使用 `fail2ban`自动封禁攻击IP
可自动读取`var/log/auth.log`下的攻击者IP，使用iptables进行封禁

[pam-servicesshd-ignoring-max-retries](https://serverfault.com/questions/588297/pam-servicesshd-ignoring-max-retries)


> While the other answers are correct in elimiating the error message you got, consider that this error message may just be a symptom of another underlying problem.

> You get these messages because there are many failing login attempts via ssh on your system. There may be someone trying to brute-force into your box (was the case when I got the same messages on my system). Read your var/log/auth.log for research...

> If this is the case, you shoud consider installing a tool like 'fail2ban' (sudo apt-get install fail2ban on Ubuntu). It automatically reads the log files of your system, searches for multiple failed login attempts and blocks the malicious clients for a configurable time via iptables...


