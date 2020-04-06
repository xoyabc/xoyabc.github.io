---
layout: post
title: 使用 rocketmq-console 监控RocketMQ消费组下线及消息积压
categories: linux
description: 使用 rocketmq-console 监控RocketMQ消费组下线及消息积压
keywords: linux, RocketMQ
---

主要介绍使用 rocketmq-console 监控消费者下线，主要包括安装、编译及配置，之后通过编写 openfalcon 自定义脚本实现日志告警。

## 安装

### MVN

```shell
apt-get update
apt-get install -y maven
```

配置文件中加入阿里镜像源

> vim /etc/maven/settings.xml 

在 <mirrors> </mirrors> 节点下加入以下配置：

```plain
      <mirror>
                  <id>alimaven</id>
                  <mirrorOf>central</mirrorOf>
                  <name>aliyun maven</name>
                  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      </mirror>
      <mirror>
                 <id>ui</id>
                 <mirrorOf>central</mirrorOf>
                 <name>Human Readable Name for this Mirror.</name>
                 <url>http://uk.maven.org/maven2/</url>
      </mirror>
      <mirror>
                 <id>jboss-public-repository-group</id>
                 <mirrorOf>central</mirrorOf>
                 <name>JBoss Public Repository Group</name>
                 <url>http://repository.jboss.org/nexus/content/groups/public</url>
      </mirror>
```

### rocketmq-console

#### 下载包

```shell
git clone -b release-rocketmq-console-1.0.0 https://github.com/apache/rocketmq-externals.git
```

#### 修改配置

- rocketmq-console/src/main/resources/static/view/pages/consumer.html

取消 `Monitor Config` 注释

```shell
将
                            <!--<button name="client" ng-click="monitor(consumerGroup.group)"-->
                            <!--class="btn btn-sm btn-primary" type="button">Monitor Config-->
                            <!--</button>-->
                            
改为

                           <button name="client" ng-click="monitor(consumerGroup.group)"
                                   class="btn btn-sm btn-primary" type="button">Monitor Config
                           </button>         
```

 + rocketmq-console/src/main/java/org/apache/rocketmq/console/task/MonitorTask.java
   
   * 将 `@Scheduled(cron = "* * * * * ?")` 改为 `@Scheduled(cron = "0 */1 * * * ?")`
   * 文件头前面加入 `import org.springframework.scheduling.annotation.Scheduled;`，即每分钟执行一次。
   
 - rocketmq-console/src/main/resources/logback.xml
 
 替换${user.home}为目标目录  /var/log/app/，并创建对应目录，`mkdir -p /var/log/app`，
 
 ```shell
     <file>/var/log/app/rocketmq-console.log</file>
     <fileNamePattern>/var/log/app/rocketmq-console-%d{yyyy-MM-dd}.%i.log
 ```
 
 ## 编译
 
 ```shell
 cd rocketmq-externals-rocketmq-console-1.0.0/rocketmq-console/
 mvn clean package -Dmaven.test.skip=true
 ```
 
 编译完成后会在 rocketmq-externals-rocketmq-console-1.0.0/rocketmq-console/target/ 路径下生成一个 jar 包，文件名为 rocketmq-console-ng-1.0.0.jar，如果机器环境和系统版本一致，其他配有 java 环境的机器就无需再次编译，使用此包即可。
 
 ## 启动
 
 **10.233.233.11:9876;10.233.233.12:9876;10.233.233.13:9876** 需要替换为实际的 nameserver 集群地址
 
 > java -jar  /path_to_your_rocketmq_console_jar/rocketmq-console-ng-1.0.0.jar --server.port=12581 --rocketmq.config.namesrvAddr='10.233.233.11:9876;10.233.233.12:9876;10.233.233.13:9876'

启动完成后的日志文件为 /var/log/app/rocketmq-console.log

## 配置

登录 rocketmq-console 后台，http://10.233.233.11:12581/#/consumer，在 MONITOR CONFIG 处配置实际使用的 consumer group，将 minCount 及 maxDiffTotal 均设为1（也可根据实际情况配置，我们监控是通过日志，所以统一设为1）

```plain
这里 `10.233.233.11` 为运行 `rocketmq-console` 的机器IP
```

## 监控

- 消费组下线

日志中会打印 `not online` ，监控关键字即可。

```plain
 [2019-06-04 09:54:07.439]  WARN examineConsumerConnectionInfo exception, test_consumer_group
org.apache.rocketmq.client.exception.MQBrokerException: CODE: 206  DESC: the consumer group[test_consumer_group] not online
```

> 注意有换行

- 消息积压

```plain
 [2019-06-04 09:50:08.401]  INFO op=look consumeInfo {"group":"test_consumer_group","count":0,"consumeTps":0,"diffTotal":0}
```

   "count":0      表示消费者数量
   
   "diffTotal":0  表示消息积压数量

监控 `diffTotal` 值即可，其阈值可通过观察实际集群的数据设置。附上取值命令：

## REF

[RocketMQ 添加监控和系统告警通知](https://www.jianshu.com/p/dc85351b7d5e)

