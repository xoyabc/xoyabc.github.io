---
layout: post
title: nginx “Too Many Open Files” 错误及解决
categories: nginx
description: nginx “Too Many Open Files” 错误及解决
keywords: nginx
---

近期，由于疫情，大家都开始远程办公，公司业务量急剧上涨，在线用户数是平时的五倍。

昨日就遇到了入会超时的问题，查看 nginx 错误日志发现很多“failed (24: Too many open files)”的错误。

```shell
2020/02/11 08:59:57 [alert] 20486#20486: *7643535 socket() failed (24: Too many open files) while connecting to upstream, client: 192.168.1.1, server: aaa.xxx.com,
2020/02/11 08:59:57 [crit] 20486#20486: accept4() failed (24: Too many open files)
```

## 排查步骤

- 1，ulimit 查看系统打开文件数

100 万，已足够大。

```shell
$ulimit -n
1000000
```

- 2，查看内核层打开文件数

100 万，已足够大。

```shell
$sysctl -a |grep fs.file-max
fs.file-max = 1000000
```

```
fs.file-max
file-max是设置 系统所有进程一共可以打开的文件数量。是系统级别的。

ulimit
设置当前shell以及由它启动的进程的资源限制。

对服务器来说，file-max, ulimit都需要设置，否则就可能出现文件描述符用尽的问题，为了让机器在重启之后仍然有效，强烈建立作以下配置，以确保file-max, ulimit的值正确无误：

1. 修改/etc/sysctl.conf, 加入             
fs.file-max = 6553560

2.系统默认的ulimit对文件打开数量的限制是1024，修改/etc/security/limits.conf并加入以下配置，永久生效
* soft nofile 65535 
* hard nofile 65535
```

后谷歌得知，nginx中也有限制文件打开数。

打开文件数 = worker_connections * worker_processes，其大小受 worker_rlimit_nofile 值限制。

而此前 nginx worker_processes 值为1，由于 worker_connections 默认值为 512，于是打开文件数仅为 512，而请求量较大，最终导致打开文件
数超过 nginx 处理上限，报`Too many open files`错误，用户入会接口响应超时，入会失败。


nginx 官方文档各参数含义

```
worker_rlimit_nofile  无默认值。打开文件数上限，最大值并不是65535，可以设为10 万
Default:	—
Context:	main

Changes the limit on the maximum number of open files (RLIMIT_NOFILE) for worker processes. Used to increase the limit without 
restarting the main process.

Syntax:	worker_connections number;
Default:	worker_connections 512;
Context:	events

需要注意的是，这个数值包括了与客户端及代理服务器之间的连接，因此计算文件描述符占用数量时无需再乘以2。另一个就是连接数不能超过最大打开
文件数，而这个值是通过 worker_rlimit_nofile 设置的。
Sets the maximum number of simultaneous connections that can be opened by a worker process.

It should be kept in mind that this number includes all connections (e.g. connections with proxied servers, among others), not
only connections with clients. Another consideration is that the actual number of simultaneous connections cannot exceed the 
current limit on the maximum number of open files, which can be changed by worker_rlimit_nofile.


Syntax:	worker_processes number | auto;
Default:	worker_processes 1;
Context:	main

Defines the number of worker processes.
The optimal value depends on many factors including (but not limited to) the number of CPU cores, the number of hard disk drives 
that store data, and load pattern. When one is in doubt, setting it to the number of available CPU cores would be a good start 
(the value “auto” will try to autodetect it).

The auto parameter is supported starting from versions 1.3.8 and 1.2.5.

Syntax:	worker_cpu_affinity cpumask ...;
worker_cpu_affinity auto [cpumask];
Default:	—
Context:	main
Binds worker processes to the sets of CPUs. Each CPU set is represented by a bitmask of allowed CPUs. There should be a separate 
set defined for each of the worker processes. By default, worker processes are not bound to any specific CPUs.
```

## 优化

- 1，修改 worker_processes 值为8

- 2，worker_rlimit_nofile 设为 100000

- 3，worker_connections 设为 12500; (100000/8)

- 4，增加 worker_cpu_affinity 配置充分利用多核
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;

最终配置见下：

```
cat /usr/local/openresty/nginx/conf/nginx.conf

......
worker_processes  8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
worker_rlimit_nofile 100000;
events {
  use epoll;
  worker_connections  12500;
}
......

```


## 其他

系统的最大文件打开数不一定是 65535。之所以设置最大文件打开数，一是为了避免内存耗尽，因为每个文件描述符都会使用内核内存。二是为了防止程序文件描述符泄露，消耗系统资源。

2011年，默认的打开文件数上限由1024调为4096。一些程序使用的文件描述符数量远大于默认值。例如，MongoDB 推荐提高上限到 64,000


## REF


[largest-allowed-maximum-number-of-open-files-in-linux](https://unix.stackexchange.com/questions/334187/largest-allowed-maximum-number-of-open-files-in-linux)








