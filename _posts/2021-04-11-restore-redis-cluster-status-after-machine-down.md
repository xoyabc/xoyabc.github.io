---
layout: post
title: redis 集群机器宕机后主从关系恢复
categories: linux
description: redis 集群机器宕机后，恢复主从关系，使得三个主节点分别落到三台机器上
keywords: linux, redis
---

redis 集群总共有三台机器，一主两从，共计启动 9 个实例。
redis 集群机器宕机后，可能会出现两个 master 节点落到一台机器上的情况，若此机器宕机，在其他节点尚未选举出新的主节点之前，集群只有一个主节点，存在单点风险。

## redis 集群信息

| IP             | 端口                |
|----------------|-------------------|
| 192.168.100.71  | 7000(主) 7002 7003 |
| 192.168.100.72  | 7000(主) 7001 7003 |
| 192.168.100.73  | 7000(主) 7001 7002 |


## 初始化并重新创建集群

### 禁止业务机器往集群写入

由于业务机器上的服务会连接 redis 写入数据，集群初始化会失败，需要禁止写入，可以使用 iptables，只允许集群内的机器访问 7000-70003 端口，拒绝其他机器连接。

```shell
iptables -A INPUT -p tcp  -s 192.168.100.71 -m multiport --dports 7000:7003 -j ACCEPT  
iptables -A INPUT -p tcp  -s 192.168.100.72 -m multiport --dports 7000:7003 -j ACCEPT  
iptables -A INPUT -p tcp  -s 192.168.100.73 -m multiport --dports 7000:7003 -j ACCEPT  
iptables -A INPUT -p tcp  -m multiport --dports 7000:7003 -j DROP
```

### 删除本地备份文件

> rm -f /usr/local/redis/cluster/nodes-7000.conf /usr/local/redis/cluster/nodes-7001.conf /usr/local/redis/cluster/nodes-7002.conf  /usr/local/redis/cluster/nodes-7003.conf  
rm -f /usr/local/redis/cluster/dump.rdb 

### 停止 redis

> ps -ef |grep -E '7000|7001|7002|7003' |grep -v grep |awk '{print $2}' |xargs kill -9

### 启动 redis

 - 192.168.100.71

```shell
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7000/redis7000.conf  
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7002/redis7002.conf
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7003/redis7003.conf
```

 - 192.168.100.72

```shell
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7000/redis7000.conf
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7001/redis7001.conf
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7003/redis7003.conf
```

 - 192.168.100.73

```shell
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7000/redis7000.conf
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7001/redis7001.conf
/usr/local/redis-cluster/bin/redis-server /usr/local/redis-cluster/cluster/7002/redis7002.conf
```

### 重置集群

执行 cluster reset 重置，清理旧的集群信息。

 - 192.168.100.71

```shell
redis-cli -h 192.168.100.71 -p 7000 -c cluster reset
redis-cli -h 192.168.100.71 -p 7002 -c cluster reset
redis-cli -h 192.168.100.71 -p 7003 -c cluster reset
```

 - 192.168.100.72

```shell
redis-cli -h 192.168.100.72 -p 7000 -c cluster reset
redis-cli -h 192.168.100.72 -p 7001 -c cluster reset
redis-cli -h 192.168.100.72 -p 7003 -c cluster reset
```

 - 192.168.100.73

```shell
redis-cli -h 192.168.100.73 -p 7000 -c cluster reset
redis-cli -h 192.168.100.73 -p 7001 -c cluster reset
redis-cli -h 192.168.100.73 -p 7002 -c cluster reset
```

### 清空数据


 - 192.168.100.71

```shell
redis-cli -h 192.168.100.71 -p 7000 flushdb
redis-cli -h 192.168.100.71 -p 7002 flushdb
redis-cli -h 192.168.100.71 -p 7003 flushdb
```

 - 192.168.100.72

```shell
redis-cli -h 192.168.100.72 -p 7000 flushdb
redis-cli -h 192.168.100.72 -p 7001 flushdb
redis-cli -h 192.168.100.72 -p 7003 flushdb
```

 - 192.168.100.73

```shell
redis-cli -h 192.168.100.73 -p 7000 flushdb
redis-cli -h 192.168.100.73 -p 7001 flushdb
redis-cli -h 192.168.100.73 -p 7002 flushdb
```

### 创建集群

 - 创建

登录第一台机器 192.168.100.71，执行以下命令创建即可。

```shell
cd /data/pkg/redis-3.0.4/src
./redis-trib.rb create 192.168.100.71:7000 192.168.100.72:7000 192.168.100.73:7000


>>> Creating cluster
/usr/local/lib/ruby/gems/2.5.0/gems/redis-3.3.0/lib/redis/client.rb:459: warning: constant ::Fixnum is deprecated
>>> Performing hash slots allocation on 3 nodes...
Using 3 masters:
192.168.100.71:7000
192.168.100.72:7000
192.168.100.73:7000
M: c966ac76a40be0c58a8295f0ce4fac800a89ffc0 192.168.100.71:7000
   slots:0-5460 (5461 slots) master
M: 8a3b3c98d2d9feb75227b3054da00ed5abb6a 192.168.100.72:7000
   slots:5461-10922 (5462 slots) master
M: a6396903ffc958711481836ceff121ddd2ff752d 192.168.100.73:7000
   slots:10923-16383 (5461 slots) master
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join..
>>> Performing Cluster Check (using node 192.168.100.71:7000)
M: c966ac76a40be0c58a8295f0ce4fac800a89ffc0 192.168.100.71:7000
   slots:0-5460 (5461 slots) master
   0 additional replica(s)
M: a6396903ffc958711481836ceff121ddd2ff752d 192.168.100.73:7000
   slots:10923-16383 (5461 slots) master
   0 additional replica(s)
M: 8a3b3c98d2d9feb75227b3054da00ed5abb6a 192.168.100.72:7000
   slots:5461-10922 (5462 slots) master
   0 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

 - 指定主节点

在第一台机器上继续操作，手动指定主节点。

```shell
./redis-trib.rb add-node --slave --master-id c966ac76a40be0c58a8295f0ce4fac800a89ffc0 192.168.100.72:7001 192.168.100.71:7000
./redis-trib.rb add-node --slave --master-id c966ac76a40be0c58a8295f0ce4fac800a89ffc0 192.168.100.73:7001 192.168.100.71:7000
 
./redis-trib.rb add-node --slave --master-id 8a3b3c98d2d9feb75227b3054da00ed5abb6a113 192.168.100.71:7002 192.168.100.72:7000
./redis-trib.rb add-node --slave --master-id 8a3b3c98d2d9feb75227b3054da00ed5abb6a113 192.168.100.73:7002 192.168.100.72:7000
 
./redis-trib.rb add-node --slave --master-id a6396903ffc958711481836ceff121ddd2ff752d 192.168.100.71:7003 192.168.100.73:7000
./redis-trib.rb add-node --slave --master-id a6396903ffc958711481836ceff121ddd2ff752d 192.168.100.72:7003 192.168.100.73:7000
```

### 查看集群状态

```shell
redis-cli -h 192.168.100.71 -p 7000 cluster info
redis-cli -h 192.168.100.71 -p 7000 cluster nodes
```

### 删除新增的防火墙规则

```shell
# 显示规则和相对应的编号
iptables -L -n --line-number

# 找到新增规则对应的编号，删除
iptables -D INPUT 规则编号
```

## REF

[Redis集群节点主从关系调整 ](https://blog.51cto.com/u_14661718/2468022)
