---
layout: post
title: 机房搬迁总结
categories: python
description: 遇到的坑及解决办法
keywords: linux
---

  由于公司原有机房合同到期，因此需要搬迁。新机房和旧机房是在一栋楼，所以搬迁也就是从2楼搬到3楼。
  
  今天，搬迁工作终于接近尾声。从9月5号开始做准备，至今已经接近2个月。搬迁中虽然遇到过一些小问题，但好在都能在客户使用前得以解决。
  
  在此也要感谢网络组，dba同学的鼎力支持。
  
  此文主要记录搬迁中遇到的问题及解决办法。
  
## 信息统计
  
  共计69台机器，包括宿主机和物理机，其中90%机器系统均为ESXI 5.x，虚拟机居多，实际虚机有450台。由于CMDB里有统计不全或统计错误的问题，期间编写了获取ESXI 下虚机信息的脚本，参见 [get_esxi_host_and_vm_info](https://github.com/xoyabc/scripts/tree/master/get_esxi_host_and_vm_info)
  
  统计的信息内容如下：
  
  - 确认哪些机器需要搬迁，
  - 统计机器的SN号和2楼所在机柜
  - 宿主机下所有虚拟机的IP地址，所属服务，开机自启设置(基于现有cmdb及脚本)
  - 物理机网口、宿主机网口(vmnic0-11)所属vlan
  - 统计有心跳线的设备及对端设备IP
  - 心跳线所在网口及对端设备心跳线网口(heartbeat健康检查采用心跳线直连，避免广播风暴影响)
  - 确认各个机器所属的环境(一共有6套环境)
  - 确认机柜数量&机柜电压&电流上限
  - 3楼单机柜位置可容纳机器数量（10个）
  
## 迁移方案

### 重要设备提前迁移
  
  涉及设备为核心代理，公共数据库等重要设备，需要提前迁移，规避宿主机起不来的风险
    
  刚开始采用的方案是导出导入模板的方案，导入导出相当于复制了2遍，非常耗时，后来采用 Vcenter 克隆的方案
    
### ESXI中的物理适配器(vmnic0-11)与物理网口对应关系
     
   由于各个vminic所在的 vlan 不同，在 ESXI 中只能看到vmnic网卡及vlan的对应关系，但是搬迁时是需要拔掉网线的，因此需要确认各个物理适配器(vmnic0-11)与机器网口的对应关系。待搬迁到3楼时，根据网络组提供的交换机端口，找到对应的物理网口，连接网线。
     
   默认情况下机器左侧第一个网口是vmnic0，第二个是vmnic1,依次类推即可。但是部分机器有后来加的网卡，这里又有三种情况：
     
   | 外加网卡分布  | ESXI 中适配器vmnic名称 | vmnic 适配器数字顺序 |
   | :---: | :---: | :---: | 
   | 一排2个网口 | 左侧的网口对应 vminic5,右侧的网口对应vminic4 | 从右往左递增 |
   | 两排2个网口 | 第一排右侧的网口对应 vminic4，第二排左侧的网口对应 vminic7 | 自上而下，从右往左递增 |
   | 两排4个网口 | 第一排右侧的网口对应 vminic4，第二排左侧的网口对应 vminic11 | 自上而下，从右往左递增 |
     
## 搬迁前所做动作
 
### 设备贴IP标签

  1，之前设备存在没贴标签和标签上IP错误的情况，根据CMDB信息及脚本统计的设备SN号，确认设备IP，之后在设备前面贴标签，以便搬迁时快速找到对应设备
  
  2，在设备背部网口旁贴标签，以便搬迁到3楼后快速找到网口，插入交换机网线。
    
   以下为之前整理的设备背部网口标签说明
    
   | 标签内容  | 说明 | 搬迁后动作 |
   | :-----------: | :-----------: | :------------: |
   | 宿-管 | 宿主机管理网口 | 入到vlan200所在交换机口 |
   | 无 | 未插网线 | 依然不插网线 |
   | 未贴标签的网口 | 之前插的有网线，搬迁至3楼后均插入到trunk口 |待宿主机开机后，根据之前统计的vmnic0-7口与vlan的对应关系表，进入配置-->网络-->vmnic0-7-->属性-->编辑，修改vmnic的vlan id，和之前的一样 |

### 验证将普通口改为 trunk 口的可行性
    
   搬迁前基本上所有宿主机的 vmnic 网口所属vlan均为access模式，即指定了 vlan id。由于每个 vmnic下的vlan id 均不同，网络组配置起来就比较麻烦，
   
   将所有端口改为trunk 模式后，网络组可以使用 `int range gi` 批量配置，提高效率，降低出错率。待开机后，运维人员只需参照之前统计好的网口及vlan id对应表，在 vmnic 的属性处，指定 vlan id 即可。
   
### 无 fstab文件的虚机迁移方案
   
   部分虚机无 fstab 文件及 分区表，重启后无法进入到 grub。后来经测试`迁移前挂起--迁移后打开电源`可以不用重启虚机及服务，决定采用此方案。
   
   采用此方案时，需要注意一下几点：
   
   1）关机虚拟机的开机自启
   
  若宿主机开启虚拟机随宿主机一起启动，即便迁移前挂起，虚拟机还是会重启。需要取消虚拟机的开机自启设置，挂起才会生效。
  
  关闭方法：依次点击`配置-->虚拟机启动/关机-->属性-->取消允许虚拟机与系统一起自动启动和停止前的勾`
  ![close_vm_auto_start.png](https://i.loli.net/2018/11/01/5bd9dc7c7f087.png)
  ![close_vm_auto_start-2.png](https://i.loli.net/2018/11/01/5bd9dceb603d8.png)
  
  2）有VIP的主备虚拟机挂起及打开顺序
  
  VIP在哪台哪台就是主。
  
  挂起：先从后主
  
  打开：先主后从
  
### 系统配置检查
  
  路由表检查：是否将路由写入到/etc/network/interfaces
  
  磁盘挂载检查：nfs，mfs对应挂载项是否写入到/etc/fatab
  
### 配置文件备份
  
  1) 基础
  
  IP `ip a`，路由 `route -n`，deb包`dpkg -al`
  
  /etc/network/interfaces，/etc/hosts，/etc/hostname，/etc/crontab，/etc/fstab
  
  2) 应用配置
  
  apache，openresty，nginx 及自研软件

### 合并宿主机网口
  
  部分宿主机占用网口较多， 最多的一个从vmnic0 到 vmnic11，且vmnic2 -vmnic11所在的vlan 都是同一个。因交换机端口数有限，如搬迁到3楼后还占用12个口，会造成其他机器无口可用，因此需要合并网口。
  
  经测试，可以在线修改，但还是建议晚上修改。修改时需更改虚拟机的网络标签，将vmnic3 - vmnic11 改为vmnic1 或 vmnic2即可。完成后宿主机就只占用了vmnic0 - vmnic3，共4个网口，相比之前减少了8个。
  
### 规划3楼机柜图
  
  主要依据宿主机所属环境，遵循主备机器各放一个机柜的原则。规划完后即可统计出需要的网线长度。我们的机器是从5U处开始摆放的，一般上面五台用1.5米，下面5台2米即可。
  
### 打标签
  
  提前给电源线，网线，设备前置后置面板的IP标签打好，搬迁时贴到对应设备上。此处的网口需提前向网络组申请，待网络组提供后即可打签。
  
  以宿主机IP 192.168.100.10(网络适配器 vmnic0-3，双电源)为例，标签内容及数量如下：
  
  | 标签内容 | 数量 | 
  | :-----------: | :-----------: | 
  | 电源线-192.168.100.10 | 2 |
  | 192.168.100.10 | 2 |
  | 192.168.100.10-vmnic0-Gi/3 | 2 | 
  | 192.168.100.10-vmnic1-Gi/4 | 2 | 
  | 192.168.100.10-vmnic2-Gi/5 | 2 | 
  | 192.168.100.10-vmnic3-Gi/6 | 2 | 
  
  说明：Gi/3，Gi/4，Gi/5，Gi/6为连接的交换机端口。
  
### 关闭磁盘自检(fsck)

```shell
# Method 1
cp /etc/fstab /etc/fstab.bak && sed -ir '/^\//s/.$/0/g' /etc/fstab
sed -ir '/^UUID/s/.$/0/g' /etc/fstab
diff /etc/fstab /etc/fstab.bak  
# Method 2
sed -i.bak '/.*\/.*/s/\(2\)\s*$/0/' /etc/fstab
# Method 3 & 4
sed -ri.bak '/.*\/.*/s/(2)\s*$/0/' /etc/fstab
sed -ri.bak 's#^(.*)/(.*)(2)(\s*)$#\1/\20\4#g' /etc/fstab
# 使用扩展正则(--regexp-extended)
# -i参数后使用".bak"，替换前先备份原文件，备份文件名为原文件名.bak
# /.*\/.*/ 匹配有挂载点的行，防止修改其他注释行
```
 
## 搬迁中需做动作
   
   - 网线贴签
   - 再次确认是否关闭宿主机上虚拟机的开机自启
   - 联系dba备份数据库
   - 已经处于关机状态的虚拟机重命名，后面加一个off
   - 挂起顺序：先从后主  打开顺序：先主后从
 
## 搬迁过程中遇到的问题及解决办法

### 系统启动时卡在grub处

现象：

![system_block_on_booting.png](https://i.loli.net/2018/11/08/5be317258a9df.png)

解决办法：

`vim /boot/grub/grub.cfg`, 搜索`timeout`，将 `terminal_output gfxterm` 下的 -1或-2（下方标红处）改为 30

![system_block_on_booting-2.png](https://i.loli.net/2018/11/08/5be319f2da796.png)

### heartbeat启动失败

现象：

```shell
heartbeat[4912]: 2018/10/10_23:21:47 ERROR: Current node [cm-proxy34-171-dexin] not in configuration!
heartbeat[4912]: 2018/10/10_23:21:47 info: By default, cluster nodes are named by `uname -n` and must be declared with a 'node' directive in the ha.cf file.
heartbeat[4912]: 2018/10/10_23:21:47 info: See also: http://linux-ha.org/wiki/Ha.cf#node_directive
heartbeat[4912]: 2018/10/10_23:21:47 WARN: Logging daemon is disabled --enabling logging daemon is recommended
heartbeat[4912]: 2018/10/10_23:21:47 ERROR: Configuration error, heartbeat not started.
heartbeat[4912]: 2018/10/10_23:21:47 debug: Exiting from pid 4912 [rc=6]
```
因主机名与ha.cf配置文件中的node名称不一致导致无法启动

解决办法：

使用hostname将主机名改为和 `/etc/ha.d/ha.cf` 中的一致即可。也可先将主机名改为与node名称一致，待启动heartbeat后，再使用 `hostname` 命令改回原来的主机名即可。

### ESXI 重启后找不到存储器

NFS 有一台ESXI宿主机启动后，第二块存储器始终添加不上。

后连上显示器，按`F2`重置ESXI配置，重新启动系统，重新配置IP，重启网络，再用VpxClient连接后，存储器连接即恢复正常。猜测原因可能是ESXI分区表乱了，初始化配置后恢复。
  
这次搬迁算是全程参与了，期间也遇到了不少坑，熬了不少的夜，但是收获也是不少，希望机器搬迁到3楼后更稳定~
 
## 后记
  
  前天晚上(`2018/11/06`) 最后一批机器已经搬到楼上了，中间又遇到了存储器添加不上的问题，所幸后来解决了(处理方法参见本文中的`搬迁过程中遇到的问题及解决办法`)，搬迁之旅终于告一段落...
