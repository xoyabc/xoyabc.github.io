---
layout: post
title: 使用 rocketmq-console 监控RocketMQ消费组及消息积压
categories: linux
description: 使用 rocketmq-console 监控RocketMQ消费组及消息积压
keywords: linux, RocketMQ
---

主要介绍使用 rocketmq-console 监控消费者下线，包括编译及配置过程。

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






