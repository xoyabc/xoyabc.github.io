---
layout: post
title: LVS+keepalived+nginx集群部署
categories: linux
description: LVS+keepalived+nginx集群部署
keywords: linux, nginx
---

### 主机信息

| 主机      | IP-LAN       | IP-WAN        |
|---------|--------------|---------------|
| LVS01   | 172.17.35.54 | 119.253.82.16 |
| LVS02   | 172.17.35.55 | 119.253.82.17 |
| VIP     | 172.17.35.56 | 119.253.82.6  |
| nginx01 | 172.17.35.57 |               |
| nginx02 | 172.17.35.58 |               |


### 更新源

四台机器均需要执行

```shell
cat > /etc/apt/sources.list <EOF
deb http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty-updates main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty-proposed main restricted universe multiverse 
deb-src http://mirrors.163.com/ubuntu/ trusty-backports main restricted universe multiverse
EOF

apt-get update
```

### 安装 keepalived

分别在两台 LVS 机器上执行

```shell
apt-get install -y ipvsadm keepalived
```

### 安装 nginx

分别在两台 nginx 机器上执行

```shell
apt-get install -y nginx
```

### 添加 LVS 配置脚本

分别在两台 nginx 机器上执行

> vim /usr/local/sbin/lvs_dr_rs.sh

贴入以下内容，注意更改内外网的 VIP

```
#!/bin/bash
#########################################################################
# File Name: realserver.sh
# Author: LookBack
# Email: admin#05hd.com
# Version:
# Created Time: 2015年05月29日 星期五 18时48分39秒
#########################################################################

# Script to start LVS DR real server.   
# description: LVS DR real server   
#   
#.  /etc/rc.d/init.d/functions
VIP=119.253.82.6
VIP1=172.17.35.56
host=`/bin/hostname`
case "$1" in  
start)   
       # Start LVS-DR real server on this machine.   
        /sbin/ifconfig lo down   
        /sbin/ifconfig lo up   
        echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore   
        echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce   
        echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore   
        echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
        /sbin/ifconfig lo:0 $VIP broadcast $VIP netmask 255.255.255.255 up  
        /sbin/ifconfig lo:1 $VIP1 broadcast $VIP1 netmask 255.255.255.255 up  
        /sbin/route add -host $VIP dev lo:0
        /sbin/route add -host $VIP1 dev lo:1
;;  
stop)
        # Stop LVS-DR real server loopback device(s).  
        /sbin/ifconfig lo:0 down   
        /sbin/ifconfig lo:1 down   
        echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore   
        echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce   
        echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore   
        echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
;;  
status)
        # Status of LVS-DR real server.  
        islothere=`/sbin/ifconfig lo:0 | grep $VIP`   
        isrothere=`netstat -rn | grep "lo:0" | grep $VIP`   
        if [ ! "$islothere" -o ! "isrothere" ];then   
            # Either the route or the lo:0 device   
            # not found.   
            echo "LVS-DR real server Stopped."   
        else   
            echo "LVS-DR real server Running."   
        fi   
;;   
*)   
            # Invalid entry.   
            echo "$0: Usage: $0 {start|status|stop}"   
            exit 1   
;;   
esac
```

添加执行权限

> chmod a+x /usr/local/sbin/lvs_dr_rs.sh
> bash /usr/local/sbin/lvs_dr_rs.sh start

加入开机自启

> echo 'bash /usr/local/sbin/lvs_dr_rs.sh start' >> /etc/rc.local


### LVS01 机器配置

virtual_router_id 注意事项：
 - 虚拟路由器的标识，对应备用节点的此值必须相同，以指明各个节点属于同一VRRP组
 - 本机两个 vrrp_instance 组的此值不能相同，否则会会引发交换机广播风暴。
 - virtual_router_id 默认使用IP最后一位。

vim /etc/keepalived/keepalived.conf

贴入一下内容，注意检查 **virtual_router_id**

```
 global_defs {
   router_id D-Medial-1
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 56  #本机两个vrrp_instance组的此值不能相同，但对应备用节点的此值必须相同，以指明各个节点属于同一VRRP组
    nopreempt
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.17.35.56 dev eth0 label eth0:0
    }
}
virtual_server 172.17.35.56 80 {
    delay_loop 5
    lb_algo wrr
    lb_kind DR
#    persistence_timeout 10
    protocol TCP
    real_server 172.17.35.54 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 5
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
    real_server 172.17.35.55 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 5
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}
vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 6  #本机两个vrrp_instance组的此值不能相同，但对应备用节点的此值必须相同，以指明各个节点属于同一VRRP组
    nopreempt
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        119.253.82.6 dev eth1 label eth1:0
    }
}
virtual_server 119.253.82.6 80 {
    delay_loop 5
    lb_algo wrr
    lb_kind DR
#    persistence_timeout 10
    protocol TCP
    real_server 119.253.82.16 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 5
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
    real_server 119.253.82.17 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 5
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}
```


### LVS02 机器配置

从主节点拷贝 keepalived.conf ，之后修改
 - state MASTER 改为 BACKUP
 - 内外网 vrrp_instance 中的 priority 100 改为 90
 - router_id D-Medial-1 改为 router_id D-Medial-2

### 两台 nginx 开启转发功能

> echo 1 > /proc/sys/net/ipv4/ip_forward

加入开机自启

> echo 'echo 1 > /proc/sys/net/ipv4/ip_forward' >> /etc/rc.local


### 启动 keepalived

先启动 LVS01，等 30 秒后启动 LVS02，避免同时占用 VIP

> service keepalived start

### 加入开机启动

> update-rc.d keepalived defaults 90

### 验证

关闭其中一台 nginx 后，分别绑定 172.17.35.56 及 119.253.82.6 测试到 80 端口的连通性

> for i in $(seq 1 1 20);do nc -zv -w 2 172.17.35.56 80;done
> for i in $(seq 1 1 20);do nc -zv -w 2 119.253.82.6 80;done

主要关注：

- 负载均衡是否正常
- 后端 Real Server（nginx）出问题 LVS 是否自动剔除
- LVS 机器其中一台宕机后，VIP 是否漂移到另一台LVS继续提供服务


### 问题记录

#### 更新源报错

```
W: GPG error: http://mirrors.aliyun.com wheezy-updates InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 8B48AD6246925553 NO_PUBKEY 7638D0442B90D010

W: GPG error: http://mirrors.aliyun.com wheezy/updates InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 9D6D8F6BC857C906 NO_PUBKEY 8B48AD6246925553

W: GPG error: http://mirrors.aliyun.com wheezy Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 8B48AD6246925553 NO_PUBKEY 7638D0442B90D010 NO_PUBKEY 6FB2A1C265FFB764
```

解决：

```shell
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8B48AD6246925553

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 7638D0442B90D010

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 9D6D8F6BC857C906

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6FB2A1C265FFB764
```

### 注意事项

keepalived 配置时需要注意以下几项：

 - virtual_router_id 取值在0-255之间，用来区分多个 instance 的 VRRP 组播
 - 配置时需要查看下本网段的 virtual_router_id ，不能与其他机器重复

