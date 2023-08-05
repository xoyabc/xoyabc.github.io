---
layout: post
title: Service 是 NodePort 类型，为什么集群中部分节点的端口不通？
categories: k8s
description: Service 是 NodePort 类型，为什么集群中部分节点的端口不通？
keywords: k8s, linux
---

## 问题

近日测试反馈，测试环境有一个服务，通过  NodePort 对外暴露的端口 8081 ，集群中只有部分主机通。该服务是通过 K8S 下 NodePort 类型的 service 对外暴露的，理论上通过集群中任一主机都可以访问。

实际测试 40.5 可以通， 40.4 不通。
![svc-nodeport-telnet.png](https://s2.loli.net/2023/08/06/pLqYf9WP1SubElj.png)

## 排查

通过 service 对外暴露端口，从端口过来的报文数据，都需要经过反向代理 kube-proxy 流入后端 pod 的 targetPort ，从而到达 pod 上的容器内。部分主机端口不通，说明没有正常转发到目标 pod。

- 1，查看防火墙规则，没有针对 8081 端口做限制
- 2，查看端口不通的节点状态正常
- 3，进一步分析 pod，发现通的四个 IP 40.5-40.8 刚好是 pod 所在的主机，猜测可能和 service 的转发策略有关

![svc-nodeport-get-pod.png](https://s2.loli.net/2023/08/06/53rbfOqAElQRnHY.png)

运行 `kubectl get svc service-name -n rtmp -o yaml` 查看 service 配置如下：

```shell
apiVersion: v1
kind: Service
metadata:
annotations:
prometheus.io/app-metrics-project: rtmp
labels:
name: srs-service-name
namespace: rtmp
spec:
clusterIP: 10.99.99.99
clusterIPs:
- 10.99.99.99
externalTrafficPolicy: Local
ipFamilies:
- IPv4
ipFamilyPolicy: SingleStack
ports:
- name: srs-service-name
nodePort: 8081
port: 1935
protocol: TCP
targetPort: 1935
sessionAffinity: None
type: NodePort
status:
loadBalancer: {}
```

可以看到 externalTrafficPolicy 的值为 Local，externaltrafficpolicy 用于把集群外部的服务引入到集群内部来，之后在集群内部直接使用，而值为 Local 表示流量只发给本机的 Pod。

## 解决

更改部署的实例数，确保集群中每个节点上都有部署 Pod。

## REF

[K8s中的external-traffic-policy是什么？](https://blog.csdn.net/agonie201218/article/details/122215040)



