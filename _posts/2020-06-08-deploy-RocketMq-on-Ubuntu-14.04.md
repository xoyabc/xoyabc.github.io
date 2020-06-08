---
layout: post
title: Ubuntu 14.04 下部署 RocketMq
categories: python
description: Ubuntu 14.04 下部署 RocketMq
keywords: linux
---

Ubuntu 14.04 下部署 RocketMq，三主模式。


## 安装 jdk 1.8

RocketMq 依赖 java 环境，因此需要先安装 java

### 下载包

前往 [javase-jdk8-downloads](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) 页面下载对应的 tar 包，下载时需要登录 Oracle 账号。也可以从度盘下载。

### 安装

以 `jdk-8u211-linux-x64.tar.gz` 为例说明。

```shell
# 解压包
tar zxf jdk-8u211-linux-x64.tar.gz -C /usr/java

# 配置环境变量
cat > /etc/profile.d/java.sh << EOF
export JAVA_HOME=/usr/java/jdk1.8.0_211
export JRE_HOME=\${JAVA_HOME}/jre  
export CLASSPATH=.:\${JAVA_HOME}/lib:\${JRE_HOME}/lib  
export PATH=\${JAVA_HOME}/bin:\$PATH
EOF

# 使环境变量生效
chmod a+x /etc/profile.d/java.sh
source /etc/profile.d/java.sh
```

最后可使用 `java -version` 检查版本号

## 安装 RocketMq

### 下载包

```shell
cd /opt
wget https://archive.apache.org/dist/rocketmq/4.5.1/rocketmq-all-4.5.1-bin-release.zip
apt-get install unzip -y --force-yes
unzip rocketmq-all-4.5.1-bin-release.zip -d /usr/local/
```

### JVM 启动参数调优

启动参数含义

 - -Xmx：应用程序能够使用的最大内存数
 - -Xms：用来设置程序初始化的时候内存栈的大小，增加这个值会提高程序的启动性能
 - -Xmn：新生代大小


根据依据机器配置，调整参数值，推荐值见下：

| 内存大小 | 启动脚本          | 对应值                                                                                                            |
|------|---------------|----------------------------------------------------------------------------------------------------------------|
| 8G   | runbroker\.sh | JAVA\_OPT="$\{JAVA\_OPT\} \-server \-Xms4g \-Xmx4g \-Xmn2g"                                                    |
| 8G   | runserver\.sh | JAVA\_OPT="$\{JAVA\_OPT\} \-server \-Xms2g \-Xmx2g \-Xmn1g \-XX:MetaspaceSize=128m \-XX:MaxMetaspaceSize=320m" |
| 16G  | runbroker\.sh | JAVA\_OPT="$\{JAVA\_OPT\} \-server \-Xms8g \-Xmx8g \-Xmn4g"                                                    |
| 16G  | runserver\.sh | JAVA\_OPT="$\{JAVA\_OPT\} \-server \-Xms4g \-Xmx4g \-Xmn2g \-XX:MetaspaceSize=128m \-XX:MaxMetaspaceSize=320m" |
| 32G  | runbroker\.sh | JAVA\_OPT="$\{JAVA\_OPT\} \-server \-Xms16g \-Xmx16g \-Xmn8g"                                                  |
| 32G  | runserver\.sh | JAVA\_OPT="$\{JAVA\_OPT\} \-server \-Xms8g \-Xmx8g \-Xmn4g \-XX:MetaspaceSize=128m \-XX:MaxMetaspaceSize=320m" |


修改启动参数值，以本机内存为 8G 示例：

```shell
# runserver.sh
sed -ri '/JAVA_OPT.*-server/s/-Xms.*-Xmx.*-Xmn[0-9]+(m|g)/-Xms2g -Xmx2g -Xmn1g/g' /usr/local/rocketmq-all-4.5.1-bin-release/bin/runserver.sh

# runbroker.sh
sed -ri '/JAVA_OPT.*-server/s/-Xms.*-Xmx.*-Xmn[0-9]+(m|g)/-Xms4g -Xmx4g -Xmn2g/g' /usr/local/rocketmq-all-4.5.1-bin-release/bin/runbroker.sh
```

### 创建数据目录及修改日志配置文件

```shell
mkdir /usr/local/rocketmq-all-4.5.1-bin-release/data

mkdir /usr/local/rocketmq-all-4.5.1-bin-release/logs

mkdir -p /data/rocketmq /data/rocketmq/data /data/rocketmq/commitlog /data/rocketmq/consumequeue /data/rocketmq/index /data/rocketmq/checkpoint /data/rocketmq/abort

cd /usr/local/rocketmq-all-4.5.1-bin-release/conf && sed -i 's#${user.home}#/usr/local/rocketmq-all-4.5.1-bin-release#g' *.xml
```

### 创建 broker 启动文件

在三台机器的 3m-noslave 目录下分别创建相应的 broker 启动文件

```shell
mkdir -p /usr/local/rocketmq-all-4.5.1-bin-release/conf/3m-noslave && cd /usr/local/rocketmq-all-4.5.1-bin-release/conf/3m-noslave
```

定义集群名，此处为 rocketmq-cluster-test-xoyabc

第一台，vim  broker-a.properties 

注意要根据实际情况修改以下几项的值。

 - 修改 namesrvAddr 为实际的集群IP
 
 - 修改 brokerClusterName
 
 - 修改 brokerIP1

```plain
namesrvAddr=192.168.1.92:9876;192.168.1.93:9876;192.168.1.94:9876
brokerClusterName=rocketmq-cluster-test-xoyabc
brokerName=broker-1
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
maxMessageSize=67108864
brokerIP1=192.168.1.94
#存储路径
storePathRootDir=/data/rocketmq/data
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/abort
 
# 加入优化参数,解决MQ Broker 写入信息时 Broker busy 异常问题
sendMessageThreadPoolNums=16
useReentrantLockWhenPutMessage=true
waitTimeMillsInSendQueue=600
osPageCacheBusyTimeOutMills=5000
```

第二台，vim broker-b.properties

 - 复制 broker-a.properties
 
 - 修改 brokerName 为broker-2
 
 - 修改 brokerIP1 为第二台机器IP

第三台，vim broker-c.properties

 - 复制 broker-a.properties
 
 - 修改 brokerName 为broker-3
 
 - 修改 brokerIP1 为第三台机器IP

### 检查配置文件

```shell
cat /usr/local/rocketmq-all-4.5.1-bin-release/conf/3m-noslave/*.properties  |grep -E 'namesrvAddr|brokerClusterName|brokerName|brokerIP1'
```

### 启动NameServer

3台均需要执行

```shell
cd /usr/local/rocketmq-all-4.5.1-bin-release/bin

nohup sh mqnamesrv &
```

### 启动broker

在第一台启动：

```shell
cd /usr/local/rocketmq-all-4.5.1-bin-release/bin
bash -x os.sh
nohup ./mqbroker -c /usr/local/rocketmq-all-4.5.1-bin-release/conf/3m-noslave/broker-a.properties &
```

在第二台启动：

```shell
cd /usr/local/rocketmq-all-4.5.1-bin-release/bin
bash -x os.sh
nohup ./mqbroker -c /usr/local/rocketmq-all-4.5.1-bin-release/conf/3m-noslave/broker-b.properties  &
```

在第三台启动：

```shell
cd /usr/local/rocketmq-all-4.5.1-bin-release/bin
bash -x os.sh
nohup ./mqbroker -c /usr/local/rocketmq-all-4.5.1-bin-release/conf/3m-noslave/broker-c.properties &
```

附注：

```shell

# 停止 broker 及 NameServer

sh bin/mqshutdown broker

The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

sh bin/mqshutdown namesrv

The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```

### 检查端口

```shell
ss -natlp|grep -E "9876|10911"
9876: Namesrv
10911:broker
```

### 初始化topic

需要修改集群名及节点IP，在第一台 server 上初始化即可。

```shell
cd  /usr/local/rocketmq-all-4.5.1-bin-release
sh bin/mqadmin updateTopic -c rocketmq-cluster-test-xoyabc -n 192.168.1.94:9876 -t test_rpc_topic -w 1 -r 1
sh bin/mqadmin updateTopic -c rocketmq-cluster-test-xoyabc -n 192.168.1.94:9876 -t test_audio_topic -w 1 -r 1

```

查看topic是否创建成功

```shell
$ sh bin/mqadmin topicList -n 192.168.1.92:9876
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0



RMQ_SYS_TRANS_HALF_TOPIC
rocketmq-cluster-test-xoyabc
broker-2
BenchmarkTest
OFFSET_MOVED_EVENT
broker-3
broker-1
test_rpc_topic
TBW102
broker-3995
SELF_TEST_TOPIC
test_audio_topic
%RETRY%convertserver
%RETRY%benchmark_consumer_37
```


## 问题记录

### 多网卡时，自动识别IP可能会错误，需要手动指定本机IP

> echo "brokerIP1=你的IP" > broker.properties

## REF

[ rocketMq 排坑：如何设置 rocketMq broker 的 ip 地址](https://my.oschina.net/u/3476125/blog/897429)

[quick-start](https://rocketmq.apache.org/docs/quick-start/)

