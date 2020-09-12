---
layout: post
title: ansible 多用户先后连接同一目标主机时的文件权限问题
categories: linux
description: ansible 多用户先后连接同一目标主机时的文件权限问题
keywords: linux, ansible
---


昨晚上线运行 playbook 时，始终连不上目标主机，playbook 中使用的用户为 ops， 报错信息如下：

```shell
192.168.122.22 | UNREACHABLE!: Authentication or permission failure. In some cases, you may have been able to authenticate 
and did not have permissions on the target directory. Consider changing the remote tmp path in ansible.cfg to a path rooted 
in "/tmp". Failed command was: ( umask 77 && mkdir -p "` echo /tmp/.ansible/tmp/ansible-tmp-1599834488.69-59423497030671 `" && echo 
ansible-tmp-1599834488.69-59423497030671="` echo /tmp/.ansible/tmp/ansible-tmp-1599834488.69-59423497030671 `" ), exited with result 1
```

## 分析

尝试切换到 /etc/ansible/hosts ，使用相同的 inventory 文件往同一个目标组下发，运行正常。

而二者区别就是 hosts 目录下有自定义的 ansible.cfg 配置文件，而 playbook 所使用的 inventory 文件所在目录无 ansible.cfg，根据 ansible 的配置文件读取顺序（ANSIBLE_CONFIG -> 当前目录 ansible.cfg --> 家目录 ansible.cfg --> /etc/ansible/ansible.cfg），playbook 执行时所使用的配置文件为 `/etc/ansible/ansible.cfg`。于是怀疑问题出在了 ansible 配置文件，对比两个配置文件，后者多了一个 `remote_tmp     = /tmp/.ansible/tmp` 的配置，其值正是报错信息中的路径。

`remote_tmp` 定义了模块传送到目标主机后所存放的目标路径，模块执行完后，会被清理掉。

登录目标主机查看目录权限，属主和属组均为 sre，文件权限为 700，而 ansible 创建目录时指定的 umask 正是 077，说明之前 sre 用户之前在目标主机上执行过任务。
此后 ops 用户执行时，由于没有写权限，因此报 `did not have permissions on the target directory`，也就无法在目标路径创建临时目录，最终导致主机状态 `UNREACHABLE!`

```shell
$ls -lrth /tmp/.ansible/tmp/ -d --time-style=long
drwx------ 3 sre sre 4.0K 2020-09-09 22:12 /tmp/.ansible/tmp/
```

## 解决

 - 注释 `/etc/ansible/ansible.cfg` 中的 `remote_tmp` 配置
 - 修改 `remote_tmp` 配置的值为 `/tmp/.ansible-${USER}/tmp`
 

## 参考

[authentication-or-permission-failure-did-not-have-permissions-on-the-remote-dir](https://stackoverflow.com/questions/35176548/authentication-or-permission-failure-did-not-have-permissions-on-the-remote-dir)

[]()

[remote-tmp](https://docs.ansible.com/ansible/2.3/intro_configuration.html#remote-tmp)






