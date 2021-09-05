---
layout: post
title: top 和 ps 中 priority 值不同的原因
categories: python
description: top 和 ps 中 priority 值不同的原因
keywords: linux
---

## 现象

分别使用 `top` 及 `ps` 查看 `nginx`（PID 为19910）进程的优先级。

1，`top` 中值为 `20`

```shell
top -b -n 1 -p 19910                              
top - 09:57:13 up 435 days, 19 min,  2 users,  load average: 0.00, 0.00, 0.00
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   8175360 total,  4617480 used,  3557880 free,   167444 buffers
KiB Swap:  8384508 total,    10408 used,  8374100 free.  3955108 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
19910 root      20   0   85948   2944   1784 S   0.0  0.0   0:00.00 nginx
```

2，`ps -eo pri` 中值为 `19`

```shell
ps -eo pid,ppid,ni,pri,psr,pcpu,stat,cmd |head -n1;ps -eo pid,ppid,ni,pri,psr,pcpu,stat,cmd |grep nginx
  PID  PPID  NI PRI PSR %CPU STAT CMD
19910     1   0  19   0  0.0 Ss   nginx: master process /usr/sbin/nginx
19911 19910   0  19   2  0.0 S    nginx: worker process
19912 19910   0  19   1  0.0 S    nginx: worker process
19914 19910   0  19   1  0.0 S    nginx: worker process
19915 19910   0  19   0  0.0 S    nginx: worker process
```

3，`ps -elf` 中值为 `80`

```shell
ps -elf |head -n1;ps -elf |grep [n]ginx
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
5 S root     19910     1  0  80   0 - 34452 sigsus Jul29 ?        00:00:00 nginx: master process /usr/sbin/nginx
5 S www-data 25732 19910  0  80   0 - 34485 ep_pol Aug31 ?        00:00:22 nginx: worker process
5 S www-data 25733 19910  0  80   0 - 34555 ep_pol Aug31 ?        00:00:23 nginx: worker process
5 S www-data 25734 19910  0  80   0 - 34452 ep_pol Aug31 ?        00:00:24 nginx: worker process
5 S www-data 25735 19910  0  80   0 - 34452 ep_pol Aug31 ?        00:00:23 nginx: worker process
```


## `ps` 中的 `priority`

ps 中的 `priority` 数据源来自 `/proc/stat`，共有如下几种 priority，其中 priority 为原始值，其他值均由其计算而来。值越小，优先级越高。

| option   | calculation    | rt range     | nrt range  | notes                    |
|----------|----------------|--------------|------------|--------------------------|
| pri      | 39-priority    | 41 to 139    | 0 to 39    |                          |
| priority | priority       | -100 to -2   | -39 to 0   | raw value                |
| opri     | 60+priority    | -40 to 58    | 60 99      |                          |
| pri_api  | -1 - priority  | 1 to 99      | -40 to -1  | correct for RT           |
| pri_bar  | priority + 1   | -99 to -1    | 1 to 40    |                          |
| pri_baz  | priority + 100 | 1 to 99      | 100 to 139 | internal kernel priority |
| pri_foo  | priority-20    |  -120 to -21 | -20 to 19  | correct for NRT          |


 - Real-time Tasks
 
   - 实时任务是有时间限制的任务，有期限，必须在期限前完成。
   - 无 nice 值。
   - 任务的调度优先级永远高于所有的非实时任务，
 
 - Non-Real-time Tasks
 
   - 非实时任务是无时间限制的任务，无期限，无需在期限前完成。
   - 有 nice 值。
   - 现代计算机系统中已不再使用。
 
 使用以下 ps 命令可将 nginx 进程的各个 priority 的值打印出来。
 
 ```shell
 ps -o pid,nice,pri,priority,opri,pri_api,pri_bar,pri_baz,pri_foo 19910
  PID  NI PRI PRI PRI API BAR BAZ FOO
19910   0  19  20  80 -21  21 120   0
 ```

## 分析

 - top

top 打印的值为原始值 `priority` 20，为 `/proc/[pid]/stat` 中的第 18 列。

```shell
awk '{print $18}' /proc/19910/stat  
20
```

```shell
// legal as UNIX "PRI"
// "priority"         (was -20..20, now -100..39)
static int pr_priority(char *restrict const outbuf, const proc_t *restrict const pp){    /* -20..20 */
    return snprintf(outbuf, COLWID, "%ld", pp->priority);
}
```

 - ps -eo pri
 
`pri` 值为 39 减去 `priority`。

> pri = 39 - priority = 39 - 20 = 19

```shell
// not legal as UNIX "PRI"
// "pri"               (was 20..60, now    0..139)
static int pr_pri(char *restrict const outbuf, const proc_t *restrict const pp){         /* 20..60 */
    return snprintf(outbuf, COLWID, "%ld", 39 - pp->priority);
```


 - `ps -elf`

`-l` 选项默认取的是 `opri` 的值，其值为 `priority` 的值加上 60。

> opri = priority + 60 = 20 + 60 = 80

```shell
// legal as UNIX "PRI"
// "intpri" and "opri" (was 39..79, now  -40..99)
static int pr_opri(char *restrict const outbuf, const proc_t *restrict const pp){        /* 39..79 */
    return snprintf(outbuf, COLWID, "%ld", 60 + pp->priority);
}
```


