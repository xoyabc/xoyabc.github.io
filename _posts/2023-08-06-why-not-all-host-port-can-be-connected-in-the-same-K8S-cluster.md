---
layout: post
title: Service 是 NodePort 类型，为什么集群中部分节点的端口不通？
categories: k8s
description: Service 是 NodePort 类型，为什么集群中部分节点的端口不通？
keywords: k8s, linux
---

近日测试反馈，测试环境有一个服务的端口，集群中只有部分主机通。该服务是通过 K8S 下 NodePort 类型的 service 对外暴露的，理论上通过集群中任一主机都可以访问。
