---
title: mysql主从复制
date: 2020-02-21 09:31:27
tags:
- mysql
- 主从
categories: mysql
---

## MySQL主从复制

<!--more-->

```sh
主：192.168.2.6
1、修改文件
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
server_id=7
binlog_format=row
log_bin=/data/mariadb-bin

[root@localhost ~]# systemctl restart mariadb

2、导出二进制位置信息
[root@localhost ~]# mysql -e 'show master logs' > pos.log
[root@localhost ~]# cat pos.log 
mariadb-bin.000009      245

3、创建复制用户
MariaDB [(none)]> grant replication slave on *.* to repluser@'192.168.2.100' identified by '123456';

从：192.168.2.100
1、修改配置文件
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
server_id=6
read_only

[root@localhost ~]# systemctl restart mariadb

2、参考帮助，配置同步信息
MariaDB [(none)]> help change master to

MariaDB [(none)]> CHANGE MASTER TO   
    -> MASTER_HOST='192.168.2.6',  
    ->  MASTER_USER='repluser',   
    ->  MASTER_PASSWORD='123456',   
    ->  MASTER_PORT=3306,   
    ->  MASTER_LOG_FILE='mariadb-bin.000009',   
    ->  MASTER_LOG_POS=245;

3、查看配置的信息
[root@localhost ~]# ll /var/lib/mysql/
[root@localhost ~]# cat /var/lib/mysql/master.info

4、MariaDB [(none)]> show processlist;    #查看线程
MariaDB [(none)]> show slave status\G

启动线程
MariaDB [(none)]> start slave;
MariaDB [(none)]> show slave status\G


==========================================================================

基于一台旧服务的基础上，实现主从复制：

主：192.168.2.6
1、修改配置
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
server_id=7
binlog_format=row
log_bin=/data/mariadb-bin

2、查看二进制日志文件列表，及大小，供从服务器备份
MariaDB [(none)]> show master logs;

3、完全备份
[root@localhost ~]# mysqldump -A --single-transaction -F --master-data=1 > /data/backup/all.sql

4、复制到从服务器
[root@localhost ~]# scp /data/backup/all.sql 192.168.2.100:

5、为了测试效果，在主服务器加入新的记录
MariaDB [hellodb]> insert teachers (name,age) values('bobo',22);
MariaDB [hellodb]> insert teachers (name,age) values('huahua',22);


从：192.168.2.100
1、修改配置
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
server_id=17
read_only

2、清空数据库，重启服务
[root@localhost ~]# rm -rf /var/lib/mysql/*
[root@localhost ~]# systemctl restart mariadb

3、打开主服务器的备份文件
修改此行CHANGE MASTER TO MASTER_LOG_FILE='mariadb-bin.000011', MASTER_LOG_POS=245;

[root@localhost ~]# vim all.sql
CHANGE MASTER TO
MASTER_HOST='192.168.2.6',
MASTER_USER='repluser',
MASTER_PASSWORD='123456',
MASTER_PORT=3306, 
MASTER_LOG_FILE='mariadb-bin.000011', MASTER_LOG_POS=245;

[root@localhost ~]# systemctl restart mariadb

4、导入文件
[root@localhost ~]# mysql < all.sql

5、开启线程
MariaDB [hellodb]> start slave;

6、查看线程开启情况
MariaDB [hellodb]> show slave status\G
 Slave_IO_Running: Yes
            Slave_SQL_Running: Yes



=====================================================================

从服务器调整为主服务器
1主2从，当主服务器故障，需要提升一台从服务器作为新的主服务器

1、判断哪个从节点数据备份的最多最新
MariaDB [(none)]> show slave status\G

2、停止主从复制，提升新的主
MariaDB [(none)]> stop slave;
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
log_bin
server_id=16
binlog_format=row
[root@localhost ~]# systemctl restart mariadb

3、从节点
[root@localhost ~]# systemctl stop mariadb
[root@localhost ~]# rm -rf /var/lib/mysql/*

4、主节点完全备份，并复制到从节点
[root@localhost ~]# mysqldump -A --single-transaction --master-data=1 > all.sql
[root@localhost ~]# scp all.sql 192.168.2.26:


5、从节点导入主节点备份的数据
[root@localhost ~]# systemctl start mariadb
[root@localhost ~]# vim all.sql
CHANGE MASTER TO 
MASTER_HOST='192.168.2.16',
  MASTER_USER='repluser',
  MASTER_PASSWORD='123456',
  MASTER_PORT=3306, MASTER_LOG_FILE='mariadb-bin.000003', MASTER_LOG_POS=245;
[root@localhost ~]# mysql < all.sql
MariaDB [(none)]> start slave;
```