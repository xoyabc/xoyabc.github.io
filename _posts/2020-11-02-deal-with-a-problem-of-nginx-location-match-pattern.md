---
layout: post
title: 记一次 nginx location 正则匹配时遇到的坑
categories: python
description: 记一次 nginx location 正则匹配时遇到的坑
keywords: linux, nginx
---

近期公司某服务新版本上线，要将原本转发给 php 的一个接口转发给 go 服务，由于正则匹配不精确，造成部分接口访问异常。

## 需求

将 https://www.test.com/rest/getUserInfo/3323135 的请求转发到后端 go 服务

## 实现方式

nginx location 正则匹配为：`/rest/getUserInfo/*`，此时包含以下 URI 的接口均会匹配到：

```shell
    /rest/getUserInfo/
    /rest/getUserInfo
    /rest/getUserInfoByEmail
    /rest/getUserInfoBySecurityPass/3323135
```

实际测试后，发现 /rest/getUserInfoByEmail 接口访问反而异常。

## 问题分析

由于 ` * ` 是代表 0 个、1 个或多个，就会匹配到带 "/" 及不带 "/"，而 `/rest/getUserInfoByEmail` 恰好满足不带 "/"，因此此接口也转发给了 go 服务。

实际上，该接口仍未在 go 服务中实现，进而造成访问失败。

## 解决

若接口明确带有 "/"，正则匹配时以斜线结尾不要带 "*"，修改为以下任意一种即可

```shell
/rest/getUserInfo/
/rest/getUserInfo/$
```
