---
title: mysql半同步复制原理
date: 2020-01-21 09:39:34
tags:
- mysql
- 原理
categories: mysql
---

从MySQL5.5开始，MySQL以插件的形式支持半同步复制。如何理解半同步呢？首先我们来看看异步，全同步的概念

 

**异步复制（Asynchronous replication）**

MySQL默认的复制即是异步的，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主如果crash掉了，此时主上已经提交的事务可能并没有传到从上，如果此时，强行将从提升为主，可能导致新主上的数据不完整。

 <!--more-->

**全同步复制（Fully synchronous replication）**

指当主库执行完一个事务，所有的从库都执行了该事务才返回给客户端。因为需要等待所有从库执行完该事务才能返回，所以全同步复制的性能必然会收到严重的影响。

 

**半同步复制（Semisynchronous replication）**

介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到relay log中才返回给客户端。相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，这个延迟最少是一个TCP/IP往返的时间。所以，半同步复制最好在低延时的网络中使用。

 

下面来看看半同步复制的原理图：

 ![img](https://images2015.cnblogs.com/blog/576154/201608/576154-20160804163916122-156935432.jpg)

 

 

**半同步复制的潜在问题**

客户端事务在存储引擎层提交后，在得到从库确认的过程中，主库宕机了，此时，可能的情况有两种

 

**事务还没发送到从库上**

此时，客户端会收到事务提交失败的信息，客户端会重新提交该事务到新的主上，当宕机的主库重新启动后，以从库的身份重新加入到该主从结构中，会发现，该事务在从库中被提交了两次，一次是之前作为主的时候，一次是被新主同步过来的。

 

**事务已经发送到从库上**

此时，从库已经收到并应用了该事务，但是客户端仍然会收到事务提交失败的信息，重新提交该事务到新的主上。

 

**无数据丢失的半同步复制**

针对上述潜在问题，MySQL 5.7引入了一种新的半同步方案：Loss-Less半同步复制。

 

针对上面这个图，“Waiting Slave dump”被调整到“Storage Commit”之前。

 

当然，之前的半同步方案同样支持，MySQL 5.7.2引入了一个新的参数进行控制-rpl_semi_sync_master_wait_point

rpl_semi_sync_master_wait_point有两种取值

 

**AFTER_SYNC**

这个即新的半同步方案，Waiting Slave dump在Storage Commit之前。

 

**AFTER_COMMIT**

老的半同步方案，如图所示。

 

**半同步复制的安装部署**

要想使用半同步复制，必须满足以下几个条件：

1. MySQL 5.5及以上版本

2. 变量have_dynamic_loading为YES

3. 异步复制已经存在

 

**首先加载插件**

因用户需执行INSTALL PLUGIN, SET GLOBAL, STOP SLAVE和START SLAVE操作，所以用户需有SUPER权限。

主：

mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';

从：

mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

 

**查看插件是否加载成功**

有两种方式

\1. 

mysql> show plugins;

```
rpl_semi_sync_master       | ACTIVE   | REPLICATION        | semisync_master.so | GPL  
```

\2. 

mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS  WHERE PLUGIN_NAME LIKE '%semi%';

```
+----------------------+---------------+
| PLUGIN_NAME          | PLUGIN_STATUS |
+----------------------+---------------+
| rpl_semi_sync_master | ACTIVE        |
+----------------------+---------------+
1 row in set (0.00 sec)
```

 

**启动半同步复制**

在安装完插件后，半同步复制默认是关闭的，这时需设置参数来开启半同步

主：

mysql> SET GLOBAL rpl_semi_sync_master_enabled = 1;

从：

mysql> SET GLOBAL rpl_semi_sync_slave_enabled = 1;

 

以上的启动方式是在命令行操作，也可写在配置文件中。

主：

```
plugin-load=rpl_semi_sync_master=semisync_master.so
rpl_semi_sync_master_enabled=1
```

从：

```
plugin-load=rpl_semi_sync_slave=semisync_slave.so
rpl_semi_sync_slave_enabled=1
```

在有的高可用架构下，master和slave需同时启动，以便在切换后能继续使用半同步复制

```
plugin-load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
rpl-semi-sync-master-enabled = 1
rpl-semi-sync-slave-enabled = 1
```

 

**重启从上的IO线程**

mysql> STOP SLAVE IO_THREAD;

mysql> START SLAVE IO_THREAD;

如果没有重启，则默认还是异步复制，重启后，slave会在master上注册为半同步复制的slave角色。

这时候，主的error.log中会打印如下信息：

```
2016-08-05T10:03:40.104327Z 5 [Note] While initializing dump thread for slave with UUID <ce9aaf22-5af6-11e6-850b-000c2988bad2>, found a zombie dump thread with the same UUID. Master is killing the zombie dump thread(4).
2016-08-05T10:03:40.111175Z 4 [Note] Stop asynchronous binlog_dump to slave (server_id: 2)
2016-08-05T10:03:40.119037Z 5 [Note] Start binlog_dump to master_thread_id(5) slave_server(2), pos(mysql-bin.000003, 621)
2016-08-05T10:03:40.119099Z 5 [Note] Start semi-sync binlog_dump to slave (server_id: 2), pos(mysql-bin.000003, 621)
```

 

**查看半同步是否在运行**

主：

mysql> show status like 'Rpl_semi_sync_master_status';

```
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| Rpl_semi_sync_master_status | ON    |
+-----------------------------+-------+
1 row in set (0.00 sec)
```

从：

mysql> show status like 'Rpl_semi_sync_slave_status';

```
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Rpl_semi_sync_slave_status | ON    |
+----------------------------+-------+
1 row in set (0.20 sec)
```

这两个变量常用来监控主从是否运行在半同步复制模式下。

至此，MySQL半同步复制搭建完毕~

 

**事实上，半同步复制并不是严格意义上的半同步复制**

当半同步复制发生超时时（由rpl_semi_sync_master_timeout参数控制，单位是毫秒，默认为10000，即10s），会暂时关闭半同步复制，转而使用异步复制。当master dump线程发送完一个事务的所有事件之后，如果在rpl_semi_sync_master_timeout内，收到了从库的响应，则主从又重新恢复为半同步复制。

 

下面来测试一下

![img](https://images2015.cnblogs.com/blog/576154/201608/576154-20160805151130434-530685964.png)

 

该验证分为三个阶段

 

1. 在Slave执行stop slave之前，主的insert操作很快就能返回。

 

2. 在Slave执行stop slave后，主的insert操作需要10.01s才返回，而这与rpl_semi_sync_master_timeout参数的时间相吻合。

这时，查看两个状态的值，均为“OFF”了。

同时，主的error.log中打印如下信息：

```
2016-08-05T11:51:49.855452Z 6 [Warning] Timeout waiting for reply of binlog (file: mysql-bin.000003, pos: 1447), semi-sync up to file mysql-bin.000003, position 1196.
2016-08-05T11:51:49.855742Z 6 [Note] Semi-sync replication switched OFF.
```

 

3. 在Slave执行start slave后，主的insert操作很快就能返回，此时，两个状态的值也变为“ON”了。

同时，主的error.log中会打印如下信息：

```
2016-08-05T11:52:40.477098Z 7 [Note] Start binlog_dump to master_thread_id(7) slave_server(2), pos(mysql-bin.000003, 1196)
2016-08-05T11:52:40.477168Z 7 [Note] Start semi-sync binlog_dump to slave (server_id: 2), pos(mysql-bin.000003, 1196)
2016-08-05T11:52:40.523475Z 0 [Note] Semi-sync replication switched ON at (mysql-bin.000003, 1447)
```

 

**其它变量**

**环境变量**

[![复制代码](mysql半同步复制原理/copycode.gif)](javascript:void(0);)

```
mysql> show variables like '%Rpl%';
+-------------------------------------------+------------+
| Variable_name                             | Value      |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled              | ON         |
| rpl_semi_sync_master_timeout              | 10000      |
| rpl_semi_sync_master_trace_level          | 32         |
| rpl_semi_sync_master_wait_for_slave_count | 1          |
| rpl_semi_sync_master_wait_no_slave        | ON         |
| rpl_semi_sync_master_wait_point           | AFTER_SYNC |
| rpl_stop_slave_timeout                    | 31536000   |
+-------------------------------------------+------------+
7 rows in set (0.30 sec)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

rpl_semi_sync_master_wait_for_slave_count

MySQL 5.7.3引入的，该变量设置主需要等待多少个slave应答，才能返回给客户端，默认为1。

 

rpl_semi_sync_master_wait_no_slave

**ON**

默认值，当状态变量Rpl_semi_sync_master_clients中的值小于rpl_semi_sync_master_wait_for_slave_count时，Rpl_semi_sync_master_status依旧显示为ON。

**OFF**

当状态变量Rpl_semi_sync_master_clients中的值于rpl_semi_sync_master_wait_for_slave_count时，Rpl_semi_sync_master_status立即显示为OFF，即异步复制。

说得直白一点，如果我的架构是1主2从，2个从都采用了半同步复制，且设置的是rpl_semi_sync_master_wait_for_slave_count=2，如果其中一个挂掉了，对于rpl_semi_sync_master_wait_no_slave设置为ON的情况，此时显示的仍然是半同步复制，如果rpl_semi_sync_master_wait_no_slave设置为OFF，则会立刻变成异步复制。

 

**状态变量**

[![复制代码](mysql半同步复制原理/copycode.gif)](javascript:void(0);)

```
mysql> show status like '%Rpl_semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 6     |
| Rpl_semi_sync_master_no_times              | 1     |
| Rpl_semi_sync_master_no_tx                 | 1     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 1120  |
| Rpl_semi_sync_master_tx_wait_time          | 4483  |
| Rpl_semi_sync_master_tx_waits              | 4     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 4     |
+--------------------------------------------+-------+
14 rows in set (0.00 sec)
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

上述状态变量中，比较重要的有以下几个

Rpl_semi_sync_master_clients

当前半同步复制从的个数，如果是一主多从的架构，并不包含异步复制从的个数。

 

Rpl_semi_sync_master_no_tx

The number of commits that were not acknowledged successfully by a slave.

具体到上面的测试中，指的是insert into test.test values(2)这个事务。

 

Rpl_semi_sync_master_yes_tx

The number of commits that were acknowledged successfully by a slave.

具体到上面的测试中，指的是以下四个事务

create database test;

create table test.test(id int);

insert into test.test values(1);

insert into test.test values(3);

 

**总结**

1. 在一主多从的架构中，如果要开启半同步复制，并不要求所有的从都是半同步复制。

2. MySQL 5.7极大的提升了半同步复制的性能。

​    5.6版本的半同步复制，dump thread 承担了两份不同且又十分频繁的任务：传送binlog 给slave ，还需要等待slave反馈信息，而且这两个任务是串行的，dump thread 必须等待 slave 返回之后才会传送下一个 events 事务。dump thread 已然成为整个半同步提高性能的瓶颈。在高并发业务场景下，这样的机制会影响数据库整体的TPS 。

​    5.7版本的半同步复制中，独立出一个 ack collector thread ，专门用于接收slave 的反馈信息。这样master 上有两个线程独立工作，可以同时发送binlog 到slave ，和接收slave的反馈。

 

**参考**

1. MariaDB原理与实现

2. <http://dev.mysql.com/doc/refman/5.7/en/replication-semisync.html>

3. <http://sanwen8.cn/p/105GRDe.html>

4. 知数堂《MySQL 5.7 Replication新特性》分享