---
layout: post
title: 在ESXI5.5中为ubuntu14.04系统的虚拟机安装VMware Tools
categories: linux
description: 在ESXI5.5中为ubuntu14.04系统的虚拟机安装VMware Tools
keywords: python, pexpect
---

统计ESXI中的虚拟机IP时，因未安装Vmware Tools，在虚拟机摘要选项卡处，查看不到虚拟机的IP地址。

![install vmware tools in esxi 5.5-1.png](https://i.loli.net/2018/09/22/5ba653e3db5d1.png)

## 影响

 - 通过摘要查看IP
 - 通过`vim-cmd  vmsvc/get.summary  10  |grep  ipAddress`命令查看虚拟机IP。

## 解决

### 更新源

```shell
apt-get  update
```

### 安装open-vm-tools

```shell
apt-get  install  open-vm-tools
```

之后在摘要处可查看虚拟机IP。

![install vmware tools in esxi 5.5-2.png](https://i.loli.net/2018/09/22/5ba653e3bc3a3.png)

## REF

[Installing VMware tools on an Ubuntu guest](https://help.ubuntu.com/community/VMware/Tools)

[Install VMware tools on Ubuntu 18.04 Bionic Beaver Linux](https://linuxconfig.org/install-vmware-tools-on-ubuntu-18-04-bionic-beaver-linux)
