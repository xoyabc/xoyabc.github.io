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

## HTTP & TCP & DNS

[TCP三次握手，四次断开](https://www.chen-hao.com.cn/posts/34933/)

[MTU的奥秘](https://dabing1022.github.io/2017/10/08/MTU%E7%9A%84%E5%A5%A5%E7%A7%98/)

[一起学习 DNS](https://dabing1022.github.io/2017/02/04/%E4%BA%86%E8%A7%A3%E8%AE%A4%E8%AF%86DNS/)

[聊一聊HTTP的Range, Content-Range](https://dabing1022.github.io/2016/12/24/%E8%81%8A%E4%B8%80%E8%81%8AHTTP%E7%9A%84Range,%20Content-Range/)

[DNS劫持和HTTP劫持有什么区别](http://bigsec.com/bigsec-news/wechat-16824-yunyingshangjiechi)







