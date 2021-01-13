---
layout: post
title: 记一次执行 ansible 命令导致 load 值高问题
categories: python
description: 记一次执行 ansible 命令导致 load 值高问题
keywords: linux
---


今早收到大量机器负载高的告警，登录服务器查看后发现 load 值高，但 CPU 使用率正常。

## 问题分析

平均负载是指单位时间内，系统处于运行队列的平均长度，运行队列的进程包括可运行状态和不可中断状态进程，也就是平均活跃进程数，它和 CPU 使用率并没有直接关系。

 - 可运行状态的进程，是指正在使用 CPU 或者正在等待 CPU 的进程，也就是我们常用 ps 命令看到的，处于 R 状态（Running 或 Runnable）的进程。
 
 - 不可中断状态的进程则是正处于内核态关键流程中的进程，并且这些流程是不可打断的，比如最常见的是等待硬件设备的 I/O 响应，也就是我们在 ps 命令中看到的 D 状态（Uninterruptible Sleep，也称为 Disk Sleep）的进程。

当 CPU 使用率很低，但 load 很高时， 我们可以推断大量线程处于 I/O 操作状态，系统的性能瓶颈出现在磁盘或者网络 I/O。

找出处于运行队列的进程

```shell
$ ps -eTo stat,pid,tid,ppid,comm  --no-header | sed -e 's/^ \*//' | perl -nE 'chomp;say if (m!^\S*[RD]+\s*!)'
D+    6351  6351  6350 python
D    18742 18742 18741 updatedb.mlocat

$ ps -ef |grep 6351
sre   6351  6350  0 04:31 pts/2    00:00:00 /usr/bin/python /home/ubuntu/.ansible/tmp/ansible-tmp-1607200299.1-63852466099206/AnsiballZ_setup.py
root     25973 25898  0 09:31 pts/3    00:00:00 grep --color=auto 6351
```

使用 `strace` 跟踪

`strace /usr/bin/python /home/sre/.ansible/tmp/ansible-tmp-1607219505.47-170658201224102/AnsiballZ_setup.py && sleep 0`

```plain
fstat(7, {st_mode=S_IFIFO|0600, st_size=0, ...}) = 0
munmap(0x7f5b09f23000, 4096)            = 0
select(8, [5 7], [], [5 7], {1, 0})     = 1 (in [5], left {0, 999432})
read(5, "DEVLINKS=/dev/disk/by-id/nvme-Am"..., 9000) = 855
select(8, [5 7], [], [5 7], {1, 0})     = 2 (in [5 7], left {0, 999998})
--- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=2809, si_status=0, si_utime=11, si_stime=11} ---
read(5, "", 9000)                       = 0
read(7, "", 9000)                       = 0
wait4(2809, [{WIFEXITED(s) && WEXITSTATUS(s) == 0}], WNOHANG, NULL) = 2809
close(5)                                = 0
close(7)                                = 0
chdir("/root")                          = 0
statfs("/Core", 
```

卡在了 `statfs("/Core"` 处，而 **/Core**为 NFS 挂载点， 而此时 NFS 挂载超时，执行 `ls /Core` 同样卡住不动。将对应的 ansible python 进程 kill 后，load 值就降下来了。

## REF

[找到Linux虚机Load高的"元凶"](https://blog.csdn.net/xiaoyida11/article/details/103717647)

[基础篇：到底应该怎么理解“平均负载”？](https://time.geekbang.org/column/article/69618)

[记一次容器内执行ansible命令卡住](https://blog.csdn.net/qq_27565769/article/details/110107474)
