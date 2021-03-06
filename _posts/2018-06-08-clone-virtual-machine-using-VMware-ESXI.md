---
layout: post
title: ESXI5.5中克隆虚拟机
categories: linux
description: 使用ESXI导入模板时，由于本机到宿主机网络传输较慢，可在ESXI中克隆机器，提升机器的部署效率。
keywords: ESXI, linux
---

使用VMware ESXI 5.5导入ova模板时，由于本机到宿主机网络传输较慢，可在ESXI中克隆机器，提升机器的部署效率。
本文所使用的系统为ubuntu 14.04，被克隆的虚拟机为28.210，克隆后的虚拟机为28.213。

> (注：28.210虚拟机对应的数据存储为28.200目录，是由于第一次给虚拟机起名是28.200，后来重命名了。)

## 1. 新建一个未加磁盘的虚拟机

创建虚拟机28.213，注意要选择“不创建磁盘”。

![ESXI虚拟机克隆-2.jpg](https://i.loli.net/2018/06/08/5b195b2ec87a4.jpg)
![ESXI虚拟机克隆-3.jpg](https://i.loli.net/2018/06/08/5b195b2eca09e.jpg)
![ESXI虚拟机克隆-4.jpg](https://i.loli.net/2018/06/08/5b195b2ed4cee.jpg)
![ESXI虚拟机克隆-5.jpg](https://i.loli.net/2018/06/08/5b195b2ed1db0.jpg)
![ESXI虚拟机克隆-6.jpg](https://i.loli.net/2018/06/08/5b195b2ecfc80.jpg)
![ESXI虚拟机克隆-7.jpg](https://i.loli.net/2018/06/08/5b195b2ed343f.jpg)
![ESXI虚拟机克隆-8.jpg](https://i.loli.net/2018/06/08/5b195b2ecb6c3.jpg)
![ESXI虚拟机克隆-9.jpg](https://i.loli.net/2018/06/08/5b195b2ece5e0.jpg)

## 2. 复制vmdk文件到新建虚机的数据目录

依次点击`摘要`-->`datastore`，进入数据存储浏览器。

![ESXI虚拟机克隆-10.jpg](https://i.loli.net/2018/06/08/5b195c9add3b0.jpg)

复制28.200中的vmdk文件到28.213目录。进入28.200目录-->选中28.200.vmdk右键复制-->切换到28.213目录-->右键粘贴即可。

![ESXI虚拟机克隆-11.jpg](https://i.loli.net/2018/06/08/5b195c9ae6236.jpg)
![ESXI虚拟机克隆-12.jpg](https://i.loli.net/2018/06/08/5b195c9ae30ee.jpg)

## 3. 给新建虚机28.213添加磁盘

在28.213处，右键选择“编辑设置”，依次点击`添加`-->`磁盘`，给虚拟机添加磁盘。注意要选择**使用现有虚拟磁盘**

![ESXI虚拟机克隆-19.jpg](https://raw.githubusercontent.com/xoyabc/xoyabc.github.io/master/images/blog/ESXI-clone-19.jpg)
![ESXI虚拟机克隆-13.jpg](https://i.loli.net/2018/06/08/5b195c9ae479c.jpg)
![ESXI虚拟机克隆-14.jpg](https://i.loli.net/2018/06/08/5b195c9ae035e.jpg)
![ESXI虚拟机克隆-15.jpg](https://i.loli.net/2018/06/08/5b195c9ae79bc.jpg)
![ESXI虚拟机克隆-16.jpg](https://i.loli.net/2018/06/08/5b195c9ada218.jpg)
![ESXI虚拟机克隆-17.jpg](https://i.loli.net/2018/06/08/5b195c9ae1bfa.jpg)
![ESXI虚拟机克隆-18.jpg](https://i.loli.net/2018/06/08/5b195c9adba0a.jpg)

## 4. 开机并更改IP主机名

打开虚拟机电源，进入系统后更改IP、主机名信息。

## 5. 遇到的问题

 - Q
 
修改完IP执行`/etc/init.d/networking restart`报错
```bash
root@ubuntu:~# /etc/init.d/networking restart
stop: Job failed while stopping
start: Job is already running: networking
```
 - A
 
 1)停掉并启用网卡
 使用`ifdown eth0 && ifup eth0`关闭并启动eth0网卡即可生效。
 
 2)14.04移除了`networking服务`的重启功能，可使用13.10中的`/etc/init/networking.conf`及`/etc/init.d/networking`文件覆盖即可。

具体步骤可参考：[restore_networking](https://github.com/metral/restore_networking)

 ```
There is a bug (feature?) in 14.04 where “service networking start|stop|restart” does nothing. 

This command/script worked in 13.10/13.04/12.04, however it doesn’t work in 14.04. To revert 

back to the 13.10 networking scripts go to:

https://github.com/metral/restore_networking

This will replace the 14.04 scripts with the one used in 13.10.
 ```
 
 
## 参考

- [assigning-a-static-ip-to-ubuntu-server-14-04-lts](https://askubuntu.com/questions/470237/assigning-a-static-ip-to-ubuntu-server-14-04-lts)
- [how-to-set-a-static-ip-to-ubuntu-14-04-server](https://www.simpleprecision.com/how-to-set-a-static-ip-to-ubuntu-14-04-server/)

全文完~~
