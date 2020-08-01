---
title: MySQL命令操作
date: 2020-04-21 15:52:01
tags:
- mysql
categories: mysql
---

## MySQL常用命令操作

<!--more-->

```sh
mysql端口3306          oracle端口1521         sqlserver端口1433

[root@localhost ~]# ls /var/lib/mysql/          #yum安装数据库存放的目录

[root@localhost ~]# mysql_secure_installation   #安全加固脚本

MariaDB [(none)]> \! hostname                   #执行系统命令\!

MariaDB [mysql]> prompt \u@[\D \r:\m\s]         #提示符

MariaDB [(none)]> show character set;           #查看所有字符集

MariaDB [(none)]> show create database test;    #查看test数据库字符集

MariaDB [mysql]> create database db3 character set utf8mb4;    #创建数据库指定字符集

[root@localhost ~]# mysql -uroot -pqwe123 --prompt="\u@[\R:\m]"

[root@localhost ~]# vim /etc/my.cnf.d/mysql-clients.cnf     #永久保存提示符

[root@localhost ~]# vim /etc/my.cnf.d/server.cnf 加入skip-networking=1       #关闭3306端口

[root@localhost ~]# mysqladmin -uroot -p123456 ping         #测mysql是否正常运行
mysqld is alive

/*注释内容*/                                     #多行注释
--注释内容                                       #单行注释   

MariaDB [(none)]> select user();                #查看用户名

MariaDB [(none)]> select version();             #查看版本号

MariaDB [(none)]> \h                            #查看命令,help

MariaDB [(none)]> status                        #显示服务器状态信息

MariaDB [(none)]> show databases;               #列出数据库

MariaDB [(none)]> use mysql                     #切换数据库mysql

MariaDB [mysql]> show tables;                   #列出表的列表

MariaDB [mysql]> desc user;                     #查看user表的定义

MariaDB [mysql]> show columns from user;        #查看user表的定义

MariaDB [mysql]> select user,host,password from user;  #从user表中挑出user,host,password字

MariaDB [mysql]> create table student (                #创建表
    -> stuid smallint unsigned auto_increment primary key,
    -> name char(10) not null,
    -> gender enum('m','f') default 'm',
    -> age tinyint unsigned,mobile char(11));
Query OK, 0 rows affected (0.00 sec)

MariaDB [mysql]> show create table student;                 #查看student表的详细定义

MariaDB [mysql]> drop table student;                        #删除表

MariaDB [mysql]> delete from mysql.student;                 #删除student表的内容

MariaDB [hellodb]> delete from teachers where name='yangyang';  #删除字段记录

MariaDB [mysql]> show table status like 'student'\G         #查看表的状态

MariaDB [mysql]> show table status from mysql\G             #查看库所有表的状态

MariaDB [mysql]> create database db1;                       #创建数据库

MariaDB [mysql]> drop database db1;                         #删除数据库

MariaDB [mysql]> show databases;                            #列出数据库

MariaDB [db2]> select user,host,password from mysql.user;   #查找mysql库的user表user,host,password字段

创建user表，参考mysql.user中user,host,password三个字段
MariaDB [db2]> create table user select user,host,password from mysql.user;

MariaDB [db2]> create table user2 like mysql.user;          #like克隆表结构

insert添加记录
MariaDB [mysql]> insert into student(stuid,name,gender,age,mobile)values(1,'alice','f',20,'13116527893')；  
MariaDB [mysql]> insert into student(age,name)values(22,'bob')；

添加多条记录
MariaDB [mysql]> insert into student(name,age,mobile)values('xiaoming',18,'15116789319'),('xiaohong',14,'19087654288')；

参考mysql.student表的结构建表
MariaDB [db2]> create table student like mysql.student;

修改表的字符集
MariaDB [db2]> alter table student character set utf8mb4;

[root@localhost ~]# vim /etc/my.cnf         #建议加入字符集设置，避免建库、表、记录
character-set-server=utf8mb4

[root@localhost ~]# mysql --default-character-set=utf8mb4   #客户端指定utf8mb4字符集连接数据库

MariaDB [mysql]> insert hello select * from student;    #将student表导入到hello表

修改表update，第二条记录
MariaDB [mysql]> update student set name='aaa',mobile=10000 where stuid=2;

[root@localhost mysql]# mysql -uroot -pqwe123 -U        #安全选项U，避免修改表时忘记加where判断，或者
/etc/my.cnf.d/mysql-clients.cnf加入safe-updates

MariaDB [mysql]> delete from student where stuid=3;     #删除表的第三行记录

MariaDB [mysql]> insert student(name,age,stuid)value('dd',23,3);    #指定序号插入此条记录

MariaDB [mysql]> select * from student;                 #显示表的内容

MariaDB [mysql]> show tables;                           #显示表的列表  

MariaDB [mysql]> delete from hello;                     #清空表的内容
MariaDB [mysql]> truncate table db2.student;            #清空表的内容，速度更快

[root@localhost ~]# mysql --default-character-set=utf8mb4 < hellodb_innodb.sql      #导入数据库

MariaDB [hellodb]> select stuid,name,age from students;                   #挑选字段显示

MariaDB [mysql]> select host as 主机,user 用户,password 密码 from user;     #添加别名，as可省略


where条件判断
MariaDB [hellodb]> select * from students where age<20;         #查找age小于20的记录

MariaDB [mysql]> select * from students where gender = 'f';     #查找gender=f的记录

MariaDB [hellodb]> select * from students where name like 's%'; #查找name是以s开头的记录

MariaDB [mysql]> select * from students where name like '%s%';  #查找name中包含s的记录

MariaDB [hellodb]> select * from students where classid is null;        #查询classid的值为null的记录

MariaDB [hellodb]> select * from students where classid is not null;    #查询classid值不为null的记录

MariaDB [hellodb]> select * from students where age >=20 and age<=30;   #查询age大于等于20小于等于30的记录

MariaDB [mysql]> select * from students where age between 20 and 30;    #查询age大于等于20小于等于30的记录

MariaDB [hellodb]> select * from users where name='admin' and password=''or'1'='1';         #sql注入

MariaDB [hellodb]> select 1+2;                  #做运算

MariaDB [mysql]> select * from students where age in (18,20,22);        #查询age=18、20、22

MariaDB [hellodb]> select * from students where name rlike '^s.*';      #rlike正则表达式

MariaDB [mysql]> select distinct gender from students;                  #distinct去重gender

avg()平均值函数
MariaDB [hellodb]> select avg(age) from students;                       #统计age的平均值
MariaDB [hellodb]> select avg(age) from students where gender='M';      #统计gender=M的age平均值

group by 分类
MariaDB [hellodb]> select gender,avg(age) from students group by gender;  #根据gender分类统计age平均值
MariaDB [hellodb]> select gender,max(age) from students group by gender;  #对gender分类后然后分别取age的最大值
MariaDB [hellodb]> select gender,count(*) from students group by gender;  #对gender分类后，然后分别取个数
MariaDB [hellodb]> select gender 性别,count(*) 数量 from students group by gender;  #加别名

MariaDB [hellodb]> select gender,count(*) from students group by gender having gender = 'm';   
#group by分组之后使用having

MariaDB [hellodb]> select gender,count(*) from students where gender = 'm' group by gender;    
#group by分组之前使用where

MariaDB [hellodb]> select classid,gender,avg(age) from students group by classid,gender;      
 #对classid、gender分类，然后分别>显示classid,gender,avg(age)记录

MariaDB [hellodb]> select age from students order by age;       #order by 排序
MariaDB [hellodb]> select age from students order by age desc;  #desc倒叙

MariaDB [hellodb]> select classid,gender,avg(age) from students group by classid,gender order by avg(age) desc;   
  #对avg(age)倒叙排列

MariaDB [hellodb]> select classid,age from students order by classid,age;      
 #多列排序，对classid排序，然后再对age排序

MariaDB [hellodb]> select classid,age from students order by classid desc,age;  
#多列排序，对classid倒叙，然后再对age正序（正序asc可省略）

MariaDB [hellodb]> select * from students order by age desc limit 3;    #排序之后，取前三

MariaDB [hellodb]> select * from students order by age desc limit 2,3;  #排序之后，跳过前两个之后，取后续的三个

MariaDB [hellodb]> select classid,avg(age) from students group by classid having avg(age)>30;       
#显示平均年龄大于30的分组


MariaDB [hellodb]> select stuid,name,age from students
    -> union        #union纵向合并，union支持自动去重
    -> select tid,name,age from teachers;

MariaDB [hellodb]> select * from students cross join teachers;      #cross join横向合并

MariaDB [hellodb]> select * from students inner join teachers on students.teacherid=teachers.tid;    
   #横向合并inner join（内连接） 取交集 on判断相等

MariaDB [hellodb]> select stuid,s.name,tid,t.name from students s inner join teachers t on s.teacherid=t.tid;   
#挑字段显示

MariaDB [hellodb]> select stuid,s.name,tid,t.name from students s,teachers t where s.teacherid=t.tid;           
#挑字段显示，老的语法

MariaDB [mysql]> select stuid,s.name,tid,t.name from students s inner join teachers t on s.teacherid=t.tid and stuid >3;
   #内连接之后再过滤

ariaDB [hellodb]> select * from students left outer join teachers on students.teacherid=teachers.tid;       #（left out join）
左外连接：左边的表内容全要，右边的表和左边表的相同部分

MariaDB [hellodb]> select * from students right outer join teachers on students.teacherid=teachers.tid;     #（right outer join）
右外连接：右边的表内容全要，左边的表和右边表的相同部分

MariaDB [hellodb]> select * from students left outer join teachers on students.teacherid=teachers.tid and teachers.tid is null;
     #左外连接，并且排除俩表相同的内容


MariaDB [hellodb]> select * from students left outer join teachers on students.teacherid=teachers.tid
    -> union
    -> select * from students right outer join teachers on students.teacherid=teachers.tid;
    #完全外连接：students和teachers表的内容全要

MariaDB [hellodb]> select stuid,name,age from students where age > (select avg(age) from students);     #子查询

MariaDB [hellodb]> select * from (select s.stuid,s.name s_name,s.teacherid,t.tid,t.name t_name from students s left outer join 
teachers t on s.teacherid=t.tid union select s.stuid,s.name,s.teacherid,t.tid,t.name from students s right outer join teachers
 t on s.teacherid=t.tid) as a where a.teacherid is null or a.tid is null;      #利用子查询，排除掉俩表的交集部分

MariaDB [hellodb]> select st.stuid,st.name,sc.score,co.course from students as st inner join scores sc on st.stuid=sc.stuid inner join 
courses co on sc.courseid=co.courseid;       #三表查询

MariaDB [hellodb]> select emp.name emp_name,leader.name leader_name from employee emp inner join employee as leader on emp.leaderid=leader.id;
  #自连接

MariaDB [hellodb]> create view view_students as select stuid,name from students;        
#创建虚拟表，视图==查询语句的别名，利用视图可以隐藏敏感信息

MariaDB [hellodb]> show table status like 'view_students'\G     #查看表的状态可以区分表和视图

MariaDB [hellodb]> drop view view_students;     #删除view_students视图表

https://dev.mysql.com/doc/refman/5.7/en/func-op-summary-ref.html    #mysql系统函数介绍
自定义函数保存在mysql.proc表中

mysql用户和权限管理

MariaDB [mysql]> create user hua@'192.168.2.100' identified by '123456';    #创建账户，允许hua用户在192.168.2.100机器上远程连接

[root@centos6 ~]# mysql -uhua -p123456 -h192.168.2.6        #远程连接

MariaDB [mysql]> drop user ''@localhost;                    #初始化删除匿名用户
MariaDB [mysql]> drop user ''@localhost.localdomain;        #初始化删除匿名用户

MariaDB [mysql]> select password('123456')  #password函数加密123456

MariaDB [mysql]> set password for hua@'192.168.2.100'=password('77777');    #修改密码

MariaDB [mysql]> update mysql.user set password=password('123456') where user='root';
   #用改表的方式给root设置密码（flush privileges刷新权限）  

免密登录，用户破解账户密码[root@localhost ~]# vim /etc/my.cnf    
[mysqld]
skip-networking
skip-grant-tables    
[root@localhost ~]# systemctl restart mariadb
MariaDB [(none)]> update mysql.user set password=password('77777') where user='root';

mysql> grant all on wordpress.* to wpuser@'192.168.2.%' identified by '123456';   #创建wpuser用户并设置权限
mysql> show grants for wpuser@'192.168.2.%';    #查看wpuser用户权限
MariaDB [(none)]> grant all on *.* to root@'192.168.2.%';       #设置管理账号
MariaDB [(none)]> show grants for root@'192.168.2.%';
Navicat for MySQL                               #图形化管理数据库工具
MariaDB [(none)]> revoke delete on wordpress.* from wpuser@'192.168.2.%';     #收回delete权限
```