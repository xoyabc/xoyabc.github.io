---
layout: post
title: 如何获取MySQL当前版本号？
categories: translate
description: 总结了几种常见的获取MySQL版本号的方法
keywords: mysql
---

几种常见的获取MySQL版本号的方法。

## 法一

使用 `mysql -uroot -p` 连接到MySQL，然后使用以下命令：

```shell
SELECT VERSION();
```

示例：

```shell
[root@host ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.40-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> SELECT VERSION();
+------------+
| VERSION()  |
+------------+
| 5.6.40-log |
+------------+
1 row in set (0.00 sec)
```

## 法二

使用 `mysql -uroot -p` 连接到MySQL，然后使用以下命令：

```
show variables like '%version%';
```

示例：

```shell
[root@host ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 119
Server version: 5.6.40-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show variables like '%version%';
+-------------------------+------------------------------+
| Variable_name           | Value                        |
+-------------------------+------------------------------+
| innodb_version          | 5.6.40                       |
| protocol_version        | 10                           |
| slave_type_conversions  |                              |
| version                 | 5.6.40-log                   |  # 版本号
| version_comment         | MySQL Community Server (GPL) |  # 社区版 
| version_compile_machine | i686                         |  # 主机的硬件架构名称
| version_compile_os      | linux-glibc2.12              |
+-------------------------+------------------------------+
7 rows in set (0.00 sec)
```

## 法三

如果你懒的话，这个就是最快的方法了（ :smile: 懒人必备），Centos / RHEL ，Ubuntu 均适用。

使用 `mysql --version` 或 `mysql -V`

示例：

```shell
[root@host ~]# mysql --version
mysql  Ver 14.14 Distrib 5.6.40, for linux-glibc2.12 (i686) using  EditLine wrapper
```

译者注：

```plain
Q：这里查到的版本号为 5.6.40，而法一及法二中查到的版本号为 5.6.40-log，为什么不一致呢？

A：此方法实际查到的是客户端的版本号，因客户端与服务端的版本一般一致，可用于快速查看版本。
```

## 法四

使用 `mysqladmin version -u USER -p PASSWD`

示例：

```shell
[root@host ~]# mysqladmin -uroot -p version
Enter password: 
mysqladmin  Ver 8.42 Distrib 5.6.40, for linux-glibc2.12 on i686
Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Server version          5.6.40-log                    # 版本号
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /tmp/mysql.sock
Uptime:                 1 hour 23 min 47 sec

Threads: 3  Questions: 1305  Slow queries: 3  Opens: 80  Flush tables: 1  Open tables: 73  Queries per second avg: 0.259
```

## 法五

使用 `mysql -uroot -p` 连接到MySQL，然后使用以下命令：

```shell
select @@version;
```

示例：

```shell
[root@host ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 168
Server version: 5.6.40-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> select @@version;
+------------+
| @@version  |
+------------+
| 5.6.40-log | # 版本号
+------------+
1 row in set (0.00 sec)
```

## 法六

登录 MySQL 后，在顶部处即可查看版本信息。

```shell
mysql -uroot -p
```

示例：

```shell
[root@host ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 191
Server version: 5.6.40-log MySQL Community Server (GPL)   # 版本号

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

## 法七

使用 `mysql -uroot -p` 连接到MySQL，然后使用以下命令：

```shell
STATUS
```

示例：

```shell

[root@host ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 191
Server version: 5.6.40-log MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> STATUS;
--------------
mysql  Ver 14.14 Distrib 5.6.40, for linux-glibc2.12 (i686) using  EditLine wrapper

Connection id:          191
Current database:
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         5.6.40-log MySQL Community Server (GPL)  # 版本号
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    utf8mb4
Conn.  characterset:    utf8mb4
UNIX socket:            /tmp/mysql.sock
Uptime:                 1 hour 49 min 20 sec

Threads: 2  Questions: 1622  Slow queries: 4  Opens: 80  Flush tables: 1  Open tables: 73  Queries per second avg: 0.247
--------------

```

## 法八

使用包管理器命令查询

- Fedora / RHEL / Red Hat / CentOS 

```shell
rpm -qa | grep mysql
```

或

```shell
yum info mysql-server
```

示例：

```shell
[root@host ~]# yum info mysql-server
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
epel/metalink                                                                         |  17 kB     00:00     
 * base: mirror.hostduplex.com
 * elrepo-kernel: repos.lax-noc.com
 * epel: d2lzkl7pfhq30w.cloudfront.net
 * extras: repos-lax.psychz.net
 * updates: mirror.fileplanet.com
base                                                                                  | 3.7 kB     00:00     
elrepo-kernel                                                                         | 2.9 kB     00:00     
epel                                                                                  | 4.7 kB     00:00     
extras                                                                                | 3.3 kB     00:00     
nginx                                                                                 | 2.9 kB     00:00     
updates                                                                               | 3.4 kB     00:00     
可安装的软件包
Name        : mysql-server
Arch        : i686
Version     : 5.1.73
Release     : 8.el6_8
Size        : 8.8 M
Repo        : base
Summary     : The MySQL server and related files
URL         : http://www.mysql.com
License     : GPLv2 with exceptions
Description : MySQL is a multi-user, multi-threaded SQL database server. MySQL is a
            : client/server implementation consisting of a server daemon (mysqld)
            : and many different client programs and libraries. This package contains
            : the MySQL server and some accompanying files and directories.
```

- Debian / Ubuntu 

```shell
[root@ubuntu ~]# dpkg --list | egrep 'mysql-(server|client)'
ii  mysql-client-5.5                   5.5.54-0ubuntu0.14.04.1            amd64        MySQL database client binaries
ii  mysql-client-core-5.5              5.5.54-0ubuntu0.14.04.1            amd64        MySQL database core client binaries
ii  mysql-server                       5.5.54-0ubuntu0.14.04.1            all          MySQL database server (metapackage depending on the latest version)
ii  mysql-server-5.5                   5.5.54-0ubuntu0.14.04.1            amd64        MySQL database server binaries and system database setup
ii  mysql-server-core-5.5              5.5.54-0ubuntu0.14.04.1            amd64        MySQL database server binaries
```

注：此方法不适用于编译安装的 MySQL


## 总结

| 命令 | 是否需要登录 MySQL | 是否适用于 Fedora / RHEL / Red Hat / CentOS / Debian / Ubuntu | 备注 | 
| :-----------: | :-----------: | :------------: | :------------: |
| SELECT VERSION(); | 是 | 是 | 可结合 `-e` 命令，不登陆 MySQL 获取版本号 |
| show variables like '%version%'; | 是 | 是 | 可结合 `-e` 命令，不登陆 MySQL 获取版本号 |
| mysql --version | 否 | 是 | 实际查到的是客户端的版本号 |
| mysqladmin version -u USER -p PASSWD | 否 | 是 |  |
| select @@version; | 是 | 是 | 可结合 `-e` 命令，不登陆 MySQL 获取版本号 |
| mysql -uroot -p | 是 | 是 | 可结合 `-e` 命令，不登陆 MySQL 获取版本号 |
| STATUS | 是 | 是 | 可结合 `-e` 命令，不登陆 MySQL 获取版本号 |
| rpm 或 dpkg | 否 | 否 | 其中 rpm 用于 Fedora / RHEL / Red Hat / CentOS LINUX发行版，dpkg 用于 Debian / Ubuntu LINUX发行版 |

## REF

[how-to-retrieve-the-current-version-of-a-mysql-database](https://stackoverflow.com/questions/8987679/how-to-retrieve-the-current-version-of-a-mysql-database)

[tell-version-mysql-unix-linux-command](https://www.cyberciti.biz/faq/tell-version-mysql-unix-linux-command/)
