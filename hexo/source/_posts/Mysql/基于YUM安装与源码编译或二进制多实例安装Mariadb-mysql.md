---
title: 基于YUM安装与源码编译或二进制多实例安装Mariadb.mysql
date: 2020-03-10 15:12:34
tags:
- mysql
- mariadb
categories: mysql
---

## 基于YUM

- 1 安装

```
yum install mariadb
```

<!--more-->

- 2 创建多实例对应的目录结构

```
mkdir /mysql/{3306,3307,3308}/{data,etc,socket,log,bin,pid} -pv
chown -R mysql.mysql /mysql 
```

- 3 创建多实例的数据库文件

```
 mysql_install_db --datadir=/mysql/3306/data/ --user=mysql
 mysql_install_db --datadir=/mysql/3307/data/ --user=mysql
 mysql_install_db --datadir=/mysql/3308/data/ --user=mysql
````
* 4 创建对应配置文件 
```

cp /etc/my.cnf /mysql/3306/etc
vim /mysql/3306/etc/my.cnf
[mysqld]
port=3306 加一行
datadir=/mysql/3306/data
socket=/mysql/3306/socket/mysql.sock
[mysqld_safe]
log-error=/mysql/3306/log/mariadb.log
pid-file=/mysql/3306/pid/mariadb.pid

```
cp /mysql/3306/etc/my.cnf  /mysql/3307/etc/my.cnf   
/mysql/3307/etc/my.cnf 修改
cp /mysql/3306/etc/my.cnf  /mysql/3308/etc/my.cnf   
/mysql/3308/etc/my.cnf 修改

* 5 准备各实例的启动脚本
vi /mysql/{3306,3307,3308}/bin/mysqld 
cat /mysq/3306/bin/mysqld 
#!/bin/bash
port=3306
mysql_user="root"
mysql_pwd="centos"
cmd_path="/usr/bin"
mysql_basedir="/mysql"
mysql_sock="${mysql_basedir}/${port}/socket/mysql.sock"

function_start_mysql()
{
    if [ ! -e "$mysql_sock" ];then
      printf "Starting MySQL...\n"
      ${cmd_path}/mysqld_safe --defaults-file=${mysql_basedir}/${port}/etc/my.cnf  &> /dev/null  &
    else
      printf "MySQL is running...\n"
      exit
    fi
}


function_stop_mysql()
{
    if [ ! -e "$mysql_sock" ];then
       printf "MySQL is stopped...\n"
       exit
    else
       printf "Stoping MySQL...\n"
       ${cmd_path}/mysqladmin -u ${mysql_user} -p${mysql_pwd} -S ${mysql_sock} shutdown
   fi
}


function_restart_mysql()
{
    printf "Restarting MySQL...\n"
    function_stop_mysql
    sleep 2
    function_start_mysql
}

case $1 in
start)
    function_start_mysql
;;
stop)
    function_stop_mysql
;;
restart)
    function_restart_mysql
;;
*)
    printf "Usage: ${mysql_basedir}/${port}/bin/mysqld {start|stop|restart}\n"
esac

* 改权限
chmod +x /mysql/{3306,3307,3308}/bin/mysqld 

* 6 启动和关闭实例
/mysql/{3306,3307,3308}/bin/mysqld start 
/mysql/{3306,3307,3308}/bin/mysqld stop  

* 7 测试连接
mysql -S /mysql/{3306,3307,3308}/socket/mysql.sock

* 8 安全加固
mysqladmin  -S /mysql/{3306,3307,3308}/socket/mysql.sock   password 'centos'
vi /mysql/{3306,3307,3308}/bin/mysqld  加上对应centos口令

## 二进制的安装

1.先创建mysql用户及mysql组，并制定家目录为/data/mysql

```
groupadd -r -g 306 mysql #指定属组gid为306
  useradd -r -g 306 -u 306 -d /data/mysql mysql #指定属主uid为306，家目录为/data/mysql
```

2.准备数据目录(mysq用户家目录),并修正权限

```
mkdir /data/mysql -p ;chown mysql:mysql /data/mysql
```

3.去官网下载mariadb 二进制tar包（链接是CentOS7X86_64的10.4.13稳定版）
4.解压tar包指/usr/local目录下,递归改属主为root、属组为mysql，并在/usr/local目录下创建一个名为mysql的软链接指向解压好的mariadb目录

```
tar xzf mariadb-10.4.13-linux-x86_64.tar.gz -c /usr/local
cd /usr/local
ln -sv mariadb-10.4.13-linux-x86_64 mysql
chown -R root:mysql /usr/local/mysql/
```

5.创建配置文件,并修改

```
mkdir /etc/mysql/
cp /etc/my.cnf /etc/mysql/my.cnf
sed -ri '/datadir=/s@(.*=).*@\1/data/mysql@' /etc/mysql/my.cnf #修改配置文件，指定数据库储存路径
sed -ri '/datadir/a\innodb_file_per-table=on\nskip_name_resolve=on' /etc/mysql/my.cnf #设置每个表独立文件 和 禁用主机名解析
```

6.创建数据库文件

```
/usr/local/mysql/scripts/mysql_install_db --datadir=/data/mysql --user=mysql
```

7.创建服务脚本并启动服务

```
cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
```

8.增加PATH环境变量路径,并生效。

```
echo 'PATH=/usr/local/mysql/bin:$PATH' >/etc/profile.d/mysql.sh
. /etc/profile.d/mysql.sh
```

9 启动数据库

```
systemctl start mysqld
```

10.运行安全初始化脚本，设置root口令、禁用匿名登陆、禁用远程主机登陆、删除test数据库，并立即生效(根据提示操作)。

```
ln -s /var/lib/mysql/mysql.sock /tmp
/usr/local/mysql/bin/mysql_secure_installation
```

至此，二进制安装mariadb数据库就完成了~源码编译安装MySQL

- 准备编译环境

×××：
 https://dev.mysql.com/downloads/mysql/

- 创建`mysql`组与用户

```shell
/usr/sbin/groupadd -g 366 -r mysql
/usr/sbin/useradd -c "MySQL" -u 366 -g mysql -s /sbin/nologin -r -d /data/mysql mysql
```

- 准备数据库目录

```shell
mkdir /data/mysql -pv
 chown mysql.mysql /data/mysql
```

- 准备源码文件

```shell
tar xf mariadb-10.2.23.tar.gz
cd mariadb-10.2.23
```

- 预编译

```shell
cmake . \
-DCMAKE_INSTALL_PREFIX=/app/mysql \
-DMYSQL_DATADIR=/data/mysql/ \
-DSYSCONFDIR=/etc/mysql \
-DMYSQL_USER=mysql \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITHOUT_MROONGA_STORAGE_ENGINE=1 \
-DWITH_DEBUG=0 \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_ZLIB=system \
-DWITH_LIBWRAP=0 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_UNIX_ADDR=/data/mysql/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
```

- 编译安装

```shell
make -j 8
make install
```

- 导出环境变量

```shell
echo 'PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
source /etc/profile.d/mysql.sh
```

- 初始化数据库

```shell
cd /usr/local/mysql
scripts/mysql_install_db --user=mysql --datadir=/data/mysql
```

- 准备配置文件

```shell
cp support-files/my-innodb-heavy-4G.cnf /etc/my.cnf.d/my.cnf
vim /etc/my.cnf.d/my.cnf   #[mysqld] 配置段中中加入如下内容
datadir = /data/mysql
innodb_file_per_table = on
skip_name_resolve = on
```

- 准备服务脚本

```shell
cp support-files/mysql.server /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
```

- 启动服务

```shell
systemctl start mysqld
```

- 安全初始化

```shell
mysql_secure_installation
```