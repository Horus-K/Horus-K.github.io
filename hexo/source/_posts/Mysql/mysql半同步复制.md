---
title: mysql半同步复制
date: 2020-01-21 09:37:31
tags:
- mysql
- 半同步
categories: mysql
---

## MySQL半同步复制

<!--more-->

```bash
1、搭建主从复制
master:主
[root@master ~]# yum -y install mariadb-server
[root@master ~]# vim /etc/my.cnf
[mysqld]
server_id=6
binlog_format=row
log_bin
skip_name_resolve
[root@master ~]# systemctl start mariadb
MariaDB [(none)]> grant replication slave on *.* to repluser@'192.168.2.%' identified by '123456';      #创建复制账号

slave:从
[root@slave ~]# yum -y install mariadb-server
[root@slave ~]# vim /etc/my.cnf
[mysqld]
read-only
server_id=16
skip_name_resolve
binlog_format=row
[root@slave ~]# systemctl start mariadb
MariaDB [(none)]> CHANGE MASTER TO
    -> MASTER_HOST='192.168.2.6',
    ->   MASTER_USER='repluser',
    ->   MASTER_PASSWORD='123456',
    ->   MASTER_PORT=3306,
    ->   MASTER_LOG_FILE='mariadb-bin.000001',
    ->   MASTER_LOG_POS=245;
Query OK, 0 rows affected (0.02 sec)
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G


2、测试主从同步是否正常
master:
MariaDB [(none)]> create database db1;

slave:
MariaDB [(none)]> show databases;

3、安装插件
master:主
MariaDB [(none)]>  INSTALL PLUGIN rpl_semi_sync_master SONAME
    -> 'semisync_master.so';
Query OK, 0 rows affected (0.01 sec)

slave:从
MariaDB [(none)]>  INSTALL PLUGIN rpl_semi_sync_slave SONAME
    -> 'semisync_slave.so';
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> show plugins;       #查看插件


4、修改配置文件，启用插件
master:
[root@master ~]# vim /etc/my.cnf
[mysqld]
rpl_semi_sync_master_enabled
[root@master ~] systemctl restart mariadb

slave:
[root@slave ~]# vim /etc/my.cnf
[mysqld]
rpl_semi_sync_slave_enabled
[root@slave ~]# systemctl restart mariadb


5、测试
slave:
MariaDB [(none)]> stop slave;

master:
MariaDB [(none)]> create database db12;
Query OK, 1 row affected (6.00 sec)

slave:
MariaDB [(none)]> start slave;
Query OK, 0 rows affected (0.00 sec)
```