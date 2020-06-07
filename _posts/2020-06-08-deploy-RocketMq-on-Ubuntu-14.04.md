---
layout: post
title: Ubuntu 14.04 下部署 RocketMq
categories: python
description: Ubuntu 14.04 下部署 RocketMq
keywords: linux
---

## 安装 jdk 1.8

RocketMq 依赖 java 环境，因此需要先安装 java

### 下载包

前往 [javase-jdk8-downloads](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) 页面下载对应的 tar 包，下载时

需要登录 Oracle 账号。也可以从度盘下载。

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









