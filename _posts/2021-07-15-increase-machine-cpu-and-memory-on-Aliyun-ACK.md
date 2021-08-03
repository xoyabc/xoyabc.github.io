---
layout: post
title: 阿里云 ACK 容器化环境节点升配
categories: python
description: 阿里云 ACK 容器化环境给 sts 方式部署的 pod 所在节点升配
keywords: docker, k8s
---

  由于成本因素，近期将部分 AWS 客户切到了阿里云，但阿里云会议节点原来的机器配置较低，需要升配。原有实例规格为 ecs.c6.4xlarge，升级后为 ecs.c6.6xlarge
  
  会议节点的机器比较特殊：
  
  - 需要绑定公网 IP
  - 公网 IP 有白名单并关联的有公网域名
  - 部署方式为 StatefulSet

StatefulSet 部署的 pod 有编号（默认从0开始，依次递增），为保持 pod 编号与新开节点 IP 最后一位为递增顺序，即第一个 pod（编号为0）对应第一台节点，因此需要逐个替换。

经实际操作验证，ACK 节点一次扩容多台时都分布在一个可用区。为保证落在不同可用区，扩容数量选择 1 个，共计 6 台机器，所以总计需要执行 6 次扩容。

替换总共分为以下几步：

- 新增 ecs.c6.6xlarge 规格的节点并置为不可调度
- 旧节点置为不可调度，保证 pod 删除后不会被控制器重新创建
- 删除旧 pod ，为后续移除节点做准备
- 旧节点解绑弹性公网 IP
- 新节点绑定原有弹性IP
- 新节点解除不可调度


更换顺序：

| 更换顺序   | pod    | 旧节点                       |
|--------|--------|---------------------------|
| 1      | meeting-0 | cn-beijing.192.168..241.108  |
| 2      | meeting-1 | cn-beijing.192.168..240.42   |
| 3      | meeting-2 | cn-beijing.192.168..240.52   |
| 4      | meeting-3 | cn-beijing.192.168..241.118  |
| 5      | meeting-4 | cn-beijing.192.168..240.39   |
| 6      | meeting-5 | cn-beijing.192.168..241.100  |

## 新增节点

### 节点扩容

1，节点池中的实例规格改为 ecs.c6.6xlarge

点击 节点管理-->节点池-->meeting-->编辑
![prod-ack-increase-node-edit.png](https://i.loli.net/2021/08/02/hIupwO9BbEXA3YH.png)

实例规格处搜 `ecs.c6.6x`，点击 `+` 号，添加到已选规格中。已选规格处点击 `-` 号，移除原有的 `ecs.c6.4x`

![prod-ack-increase-node-add-6x.png](https://i.loli.net/2021/08/02/VxvENYd3kFUonJ9.png)


2，点击 节点管理-->节点池-->meeting-->扩容

![prod-ack-increase-node-2.png](https://i.loli.net/2021/08/02/qa6bXGUhdP4JNLS.png)

3，扩容的节点数量选择 1，点击提交

![prod-ack-increase-node-1.png](https://i.loli.net/2021/08/02/rIKBAMPFueTlnQ6.png)

可运行以下命令，观察新加入的节点是否为 ready

> kubectl get node -l k8s-meeting=true -w

4，扩容节点打 ROLE 标签

> kubectl label node cn-beijing.192.168..240.88 node-role.kubernetes.io/meeting=node

```plain
cn-beijing.192.168..240.88 需要替换为新开节点的 IP
```

5，新开节点置为不可调度

> kubectl cordon cn-beijing.192.168..240.88

```plain
cn-beijing.192.168..240.88 需要替换为新开节点的 IP
```


### 旧节点置为不可调度

```shell
kubectl cordon cn-beijing.192.168..241.108
```

```plain
cn-beijing.192.168..241.108 为节点名称，实际执行时注意替换
```

### 删除旧 pod

> kubectl delete pod meeting-0

```plain
meeting-0 为 pod 名称，实际执行时注意替换
```

### 旧节点解绑弹性公网 IP

控制台切换到 `云服务器 ECS`，搜索节点 IP 解绑即可。

### 新节点绑定原有弹性公网 IP

控制台切换到 `云服务器 ECS`,搜索新开节点 IP 绑定旧节点解绑的弹性 IP

```plain
pod 中的 业务 docker 容器启动依赖公网 IP，因此在 pod 调度到新开节点前需要提前绑定弹性 IP
```

### 新节点解除不可调度

执行以下命令

> kubectl uncordon cn-beijing.192.168..240.88

```plain
cn-beijing.192.168..240.88 需要替换为新开节点的 IP
```

之后 Scheduler 会将 pod 调度到新开节点

## 更改 sts 的资源限制

待 6 个节点全部更换完成后，更改 StatefulSet 中的资源限制。

执行以下命令更改配置

> kubectl edit sts meeting

 - limits 处 cpu 值改为 "22"
 - limits 处 memory 值改为 46Gi

更改后的配置

```yaml
resources:
  limits:
    cpu: "22"
    memory: 46Gi
```


## 移除旧节点

### 更改为按量付费

旧节点之前为包月计费，需要改为按量计费后才会退换剩余的包月费用。

控制台切换到 `云服务器 ECS`，搜索旧节点，改为按量付费。

![prod-ack-increase-node-change-cost.png](https://i.loli.net/2021/08/02/UXzkj9LJ6tZfEoR.png)

### 移除节点

节点管理-->节点池-->meeting-->详情

![prod-ack-increase-node-detail.png](https://i.loli.net/2021/08/02/rmBd6sSkpziTRJ5.png)

切换到`节点管理`选项，选中节点，点击`移除节点`

![prod-ack-increase-node-remove.png](https://i.loli.net/2021/08/02/WyocGKZhbux8aS3.png)

勾选 `自动排空节点（drain）` 及`同时释放 ECS`，点击确定

![prod-ack-increase-node-remove-confirm.png](https://i.loli.net/2021/08/02/KfsAJYe4o2hBOw1.png)

