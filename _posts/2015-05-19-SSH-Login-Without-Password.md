---
layout: post
title: CentOS 6.5中配置ssh免密码登录 
categories: linux
description: CentOS 6.5中配置ssh免密码登录 
keywords: linux
---

现有两台机器A与B，A：10.240.210.131 用户为root；B：10.240.210.233 用户为root,要实现在A机器上通过ssh连接到B时不用输入密码。

## 原理

使用一种被称为"公私钥"认证的方式来进行ssh登录. "公私钥"认证方式简单的解释为：

```

1. 在客户端上创建一对公私钥 （公钥文件：~/.ssh/id_rsa.pub； 私钥文件：~/.ssh/id_rsa）

2. 把公钥放到服务器上（~/.ssh/authorized_keys）, 自己保留好私钥

3. 当ssh登录时,ssh程序会发送私钥去和服务器上的公钥做匹配.如果匹配成功就可以登录了
```

## 配置过程

1.在A机器执行ssh-keygen -t rsa生成一对rsa公私钥，生成的密钥对会存放在~/.ssh目录下。
```shell
# 执行命令时如有提示一路回车即可。
root@test .ssh$ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
bf:fa:14:0f:48:a7:65:70:18:ed:60:bf:00:2f:b5:7e root@10.130
The key's randomart image is:
+--[ RSA 2048]----+
|        o+.      |
|      . =o.      |
|       =.=+      |
|      ..+*o      |
|       oS.o.     |
|        ..E+     |
|         .o .    |
|         . .     |
|        .oo      |
+-----------------+
```
2.在B机器上root的用户目录下创建~/.ssh目录。
```shell
ssh root@10.240.210.233
mkdir -p .ssh
```
3.将10.240.210.131上用户“root”的公钥拷贝到root@10.240.210.233上，来实现无密码ssh。
```shell
$ cat .ssh/id_rsa.pub | ssh aliceB@hostB 'cat >> .ssh/authorized_keys'
```

附注：

上述的创建目录并复制的操作（步骤2与3）也可以通过一个 ssh-copy-id 命令一步完成。

**必须要使用参数i指定公钥的路径。不然传送的公钥会出现错误，内容被替换为错误信息The agent has no identities**

```shell
root@test .ssh$ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.240.210.233
```

## 疑难解答

即使在密钥认证生效后，你可能仍然需要输入SSH密码。如果遇到这种情况，请检查系统日志（如/var/log/secure）以查看是否出现下面的异常。

> Authentication refused: bad ownership or modes for file /home/aliceB/.ssh/authorized_keys

在这种情况下，密钥认证的失败是由于`~/.ssh/authorized_keys`文件的权限或拥有者不正确。一般情况，如果这个文件对除了
你之外的所有用户都可读，就会出现这个错误。可用下面的方式改变文件的权限以修正错误。

> $ chmod 700 ~/.ssh/authorized_keys









