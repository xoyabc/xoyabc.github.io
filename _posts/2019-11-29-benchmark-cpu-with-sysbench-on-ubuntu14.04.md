---
layout: post
title: 在 ubuntu14.04 上使用 sysbench 测试CPU性能
categories: python
description: 使用sysbench测试CPU性能
keywords: linux, cpu
---

sysbench 是一个开源的性能测试工具，可用来测试数据库及系统（CPU、内存、IO）性能。

本文主要使用 sysbench 测试CPU性能，并对比 AWS，阿里云，腾讯云三家的数据，为后续上云选型提供数据支撑。

## 安装

版本为 0.4.12，注意新版本部分参数有变化。

```shell
apt-get install sysbench

$dpkg -al |grep sysbench

ii  sysbench                           0.4.12-1.1                                 amd64        Cross-platform and multi-threaded benchmark tool
```

## 常用参数

 - --cpu-max-prime 素数生成数量的上限

```
- 若设置为3，则表示2、3、5（这样要计算1-5共5次）
- 若设置为10，则表示2、3、5、7、11、13、17、19、23、29（这样要计算1-29共29次）
- 默认值为10000
```

- --num-threads 线程数

```
- 若设置为1，则sysbench仅启动1个线程进行素数的计算
- 若设置为2，则sysbench会启动2个线程，同时分别进行素数的计算
- 默认值为1
```

- --max-time 运行时长，单位秒

```
- 若设置为5，则sysbench会在5秒内循环往复进行素数计算，
  从输出结果可以看到在5秒内完成了几次，
  比如配合--cpu-max-prime=3，则表示第一轮算得3个素数，
  如果时间还有剩就再进行一轮素数计算，直到时间耗尽。
  每完成一轮就叫一个event
- 默认值为10
- 相同时间，比较的是谁完成的event多
```

- --max-requests:请求上限次数

```
- 若设置为100，则表示当完成100次event后，即使时间还有剩，也停止运行
- 默认值为0，则表示不限event次数
- 相同event次数，比较的是谁用时更少
```

- 输出结果中的关键字说明

```
event: 完成了几轮的素数计算

stddev(标准差): 在相同时间内，多个线程分别完成的素数计算次数是否稳定，如果数值越低，则表示多个线程的结果越接近(即越稳定)。该参数对于单线程无意义。
```

## 测试

将素数上限和线程数设为一致，通过调整时间及请求数来比较性能：

- 相同时间，比较event
- 相同event，比较时间
- 时间和event都相同，比较stddev(标准差)

AWS、阿里云、腾讯云机器配置

| 厂商 | CPU型号 | 频率(GHz)	| 核数 |
| :-----------: | :-----------: | :------------: |  :------------: |
| AWS | Intel(R) Xeon(R) Platinum 8124M CPU | 3 | 8 |
| 阿里云 | Intel(R) Xeon(R) Platinum 8163 CPU |	2.5	| 4 |
| 腾讯云 | Intel(R) Xeon(R) Gold 61xx CPU | 2.5	| 4 |

### 相同时间，比较event数

三台实例素数上限均为 200000，线程数均为4，设置执行时间上限为10s，比较 event 值，越大性能越好

> sysbench --num-threads=4 --max-time=10 --cpu-max-prime=200000 --test=cpu run

| 厂商  | event | stddev |
|:---:|:-----:|:------:|
| AWS | 726   | 0\.01  |
| 阿里云 | 580   | 0\.01  |
| 腾讯云 | 620   | 0\.01  |

结论：AWS最好，腾讯次之，阿里最差

详细数据：

```shell
# AWS

sysbench 0.4.12:  multi-threaded system evaluation benchmark


Running the test with following options:

Number of threads: 4


Doing CPU performance benchmark


Threads started!

Time limit exceeded, exiting...

(last message repeated 3 times)

Done.


Maximum prime number checked in CPU test: 200000



Test execution summary:

    total time:                          10.0280s

    total number of events:              726

    total time taken by event execution: 40.0692

    per-request statistics:

         min:                                 52.96ms

         avg:                                 55.19ms

         max:                                 62.72ms

         approx.  95 percentile:              55.49ms


Threads fairness:

    events (avg/stddev):           181.5000/0.50

    execution time (avg/stddev):   10.0173/0.01


# ALIYUN

sysbench 0.4.12:  multi-threaded system evaluation benchmark


Running the test with following options:

Number of threads: 4


Doing CPU performance benchmark


Threads started!

Time limit exceeded, exiting...

(last message repeated 3 times)

Done.


Maximum prime number checked in CPU test: 200000



Test execution summary:

    total time:                          10.0563s

    total number of events:              580

    total time taken by event execution: 40.1296

    per-request statistics:

         min:                                 66.54ms

         avg:                                 69.19ms

         max:                                 87.79ms

         approx.  95 percentile:              78.54ms


Threads fairness:

    events (avg/stddev):           145.0000/8.09

    execution time (avg/stddev):   10.0324/0.01



# TXYUN

sysbench 0.4.12:  multi-threaded system evaluation benchmark


Running the test with following options:

Number of threads: 4


Doing CPU performance benchmark


Threads started!

Time limit exceeded, exiting...

(last message repeated 3 times)

Done.


Maximum prime number checked in CPU test: 200000



Test execution summary:

    total time:                          10.0575s

    total number of events:              620

    total time taken by event execution: 40.1807

    per-request statistics:

         min:                                 64.36ms

         avg:                                 64.81ms

         max:                                 80.02ms

         approx.  95 percentile:              65.04ms


Threads fairness:

    events (avg/stddev):           155.0000/0.00

    execution time (avg/stddev):   10.0452/0.01
```

### 相同event，比较时间

三台实例素数上限均为 20000，线程数均为4，event 均为 100000，比较时间，越少性能越好。

> sysbench --num-threads=4 --max-requests=100000 --cpu-max-prime=20000 --test=cpu run

| 厂商  | time\(s\) | stddev |
|:---:|:---------:|:------:|
| AWS | 58\.4855  | 0      |
| 阿里云 | 75\.4926  | 0      |
| 腾讯云 | 67\.3867  | 0\.01  |

结论：AWS最好，腾讯次之，阿里最差

详细数据：

```shell
# AWS

sysbench 0.4.12:  multi-threaded system evaluation benchmark


Running the test with following options:

Number of threads: 4


Doing CPU performance benchmark


Threads started!

Done.


Maximum prime number checked in CPU test: 20000



Test execution summary:

    total time:                          58.4855s

    total number of events:              100000

    total time taken by event execution: 233.9123

    per-request statistics:

         min:                                  2.16ms

         avg:                                  2.34ms

         max:                                 11.61ms

         approx.  95 percentile:               2.36ms


Threads fairness:

    events (avg/stddev):           25000.0000/37.22

    execution time (avg/stddev):   58.4781/0.00


 # ALIYUN

sysbench 0.4.12:  multi-threaded system evaluation benchmark


Running the test with following options:

Number of threads: 4


Doing CPU performance benchmark


Threads started!

Done.


Maximum prime number checked in CPU test: 20000



Test execution summary:

    total time:                          75.4926s

    total number of events:              100000

    total time taken by event execution: 301.9016

    per-request statistics:

         min:                                  2.73ms

         avg:                                  3.02ms

         max:                                 18.81ms

         approx.  95 percentile:               4.37ms


Threads fairness:

    events (avg/stddev):           25000.0000/290.20

    execution time (avg/stddev):   75.4754/0.00


# TXYUN

sysbench 0.4.12:  multi-threaded system evaluation benchmark


Running the test with following options:

Number of threads: 4


Doing CPU performance benchmark


Threads started!

Done.


Maximum prime number checked in CPU test: 20000



Test execution summary:

    total time:                          67.3867s

    total number of events:              100000

    total time taken by event execution: 269.4948

    per-request statistics:

         min:                                  2.64ms

         avg:                                  2.69ms

         max:                                 17.21ms

         approx.  95 percentile:               2.74ms


Threads fairness:

    events (avg/stddev):           25000.0000/53.98

    execution time (avg/stddev):   67.3737/0.01
```

## REF

[linux sysbench (一): CPU性能测试详解](https://www.jianshu.com/p/be8dfa2ec10b)

[How to Benchmark Your System (CPU, File IO, MySQL) with Sysbench](https://www.howtoforge.com/how-to-benchmark-your-system-cpu-file-io-mysql-with-sysbench)

[sysbench](https://github.com/akopytov/sysbench)




