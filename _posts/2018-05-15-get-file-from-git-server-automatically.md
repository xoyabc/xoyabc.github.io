---
layout: post
title: 使用pexpect模块自动拉取git仓库代码
categories: linux
description: 从github拉取代码时，经常需要输入密码，可使用pexpect模块免交互拉取
keywords: python, pexpect
---

从github拉取代码时，经常需要输入密码，可使用pexpect模块设置密码后免交互拉取。

**目录**

<!-- vim-markdown-toc GFM -->

* [Pexpect的用途](#pexpect的用途)
* [使用pexpect模块自动拉取git仓库代码](#使用pexpect模块自动拉取git仓库代码)

<!-- vim-markdown-toc -->

## Pexpect的用途

主要用于：

 - 自动化交互应用程序，如ssh, ftp, passwd, telnet等
 - 软件包的自动安装脚本
 - 软件自动化测试

## 使用pexpect模块自动拉取git仓库代码

使用git拉取代码时，每次输入`git pull`命令后都需要手动输入一遍密码，如果动作较频繁，非常耗费时间。

![1.jpg](https://i.loli.net/2018/05/15/5afaf3329b2b1.jpg)

可使用pexpect模块，免交互执行拉取动作。

```
#/usr/bin/python
# -*- coding: utf-8 -*-
from pexpect import *
import sys

my_secret_password='my_password'

child = spawn('git pull')
child.expect('Password: ')
child.sendline(my_secret_password)
#print child.before   # The before property will contain all text up to the expected string pattern. 
print child.after     # The after string will contain the text that was matched by the expected pattern. 
child.interact()      # Give control of the child to the user.
```

脚本输出：

`print child.after`

![1.jpg](https://i.loli.net/2018/05/15/5afaf6340dbcf.jpg)

`print child.before`

![2.jpg](https://i.loli.net/2018/05/15/5afaf6923b517.jpg)

通过以上两图对比可知，`child.before`里面存放的为匹配字符串之前的所有内容，`child.after`里面存放的为匹配到的字符串。

因此，`print child.after`中有**Password: **而`print child.before`没有。

