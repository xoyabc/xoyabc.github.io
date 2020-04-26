---
layout: post
title: 更改 ansible 默认并行进程数
categories: linux
description: 更改 ansible 默认并行进程数，避免同时升级时造成服务中断
keywords: linux, ansible
---

线上部分服务的机器数量有限，不能同时变更，否则会造成服务中断。

ansible 中默认的并发数为 5 ，若机器数量小于 5 台的服务在升级期间，服务状态会异常，进而造成业务使用异常。

为此，有以下需求：

 - 变更涉及的设备，并发数设为 1
 - 其他设备，并发数仍设为 5
 

而 ansible 是按照以下顺序读取配置文件的，使用第一个寻找到的配置文件的配置，其他的则忽略

 - **ANSIBLE_CONFIG** (设置好的环境变量)
 - **ansible.cfg** (位于当前目录)
 - **~/.ansible.cfg** (位于用户家目录)
 - **/etc/ansible/ansible.cfg**
 
因此，可以指定不同的配置文件来设置不同的并发数。
 
## 更改默认并发数

控制并发数的参数为 `forks`, 该选项设置与主机通信时的默认并行进程数，位于 `defaults` 配置区块。因此，只需修改该值即可。
 
在变更设备的 inventory 所在目录，新建 ansible.cfg ，贴入以下内容：

```shell
[defaults]
forks          = 1
```

如果想保留原有的其他配置，可复制一份 ansible.cfg ，之后修改 `forks` 为 5 即可。

> cat /etc/ansible/ansible.cfg |grep -Ev '^#' |sed '/^\s*$/d' |sed -r '/forks/s/(.*)=(.*)/\1= 1/g' > /path/to/yout/inventory_dir/ansible.cfg


## 修改前后执行时间对比

测试命令： `time ansible -i upgrade_hosts cdts  -m shell -a "uname -n"`

- 修改前

```shell
real    0m1.732s
user    0m1.200s
sys     0m0.360s
```

 - 修改后
 
```shell
real    0m4.129s
user    0m1.248s
sys     0m0.364s
```


## REF

[the-configuration-file](https://docs.ansible.com/ansible/2.7/reference_appendices/config.html?#the-configuration-file)

[Ansible的配置文件](https://ansible-tran.readthedocs.io/en/latest/docs/intro_configuration.html)




