---
layout: post
title: 备份SecureCRT中的Button Bar 
categories: work
description: 备份SecureCRT中的Button Bar 
keywords: SecureCRT
---

系统或网络运维工作中，为提高工作效率，经常会使用Button Bar功能保存一些常用命令。

但如果重装系统或软件后，每次都需要重新一个个添加Button，非常浪费时间。为此，谷歌找了下保存方法，以下为7.2版本对应的操作步骤。

## 1.进入配置文件所在文件夹

依次打开 Options-->Global Options-->General-->Configuration Paths，找到配置文件所在文件夹，这里为“C:\Users\lxh\AppData\Roaming\VanDyke\Config”

![1.jpg](https://i.loli.net/2018/05/19/5afff1bdb3a2e.jpg)

## 2.备份ButtonBar.ini文件

打开上一步找到的配置文件夹"C:\Users\lxh\AppData\Roaming\VanDyke\Config"，拷贝其下的ButtonBarV3.ini即可。

![2.jpg](https://i.loli.net/2018/05/19/5afff1bdb5da7.jpg)

## 参考：

[Ability to save button bar configurations](https://forums.vandyke.com/archive/index.php/t-2604.html)
