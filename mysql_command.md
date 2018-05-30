---
title: MYSQL_COMMAND
date: 2088-08-08 08:08:08
tags:
---

## MySQL修改root密码
``` bash
#1.用SET PASSWORD命令
MySQL -u root
　　mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
#2.用mysqladmin
mysqladmin -u root password "newpass"
#如果root已经设置过密码，采用如下方法
mysqladmin -u root password oldpass "newpass"
#3.用UPDATE直接编辑user表
mysql -u root
　mysql> use mysql;
　mysql> UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
　mysql> FLUSH PRIVILEGES;
#4.在丢失root密码的时候，可以这样
mysqld_safe --skip-grant-tables&
　　mysql -u root mysql
　　mysql> UPDATE user SET password=PASSWORD("new password") WHERE user='root';
　　mysql> FLUSH PRIVILEGES;
```

## 权限设置
``` bash
grant all privileges on *.* to 'root'@'localhost' identified by 'fruit@123' with grant option;
flush privileges;
```

## 临时设置外键失效
``` bash
SET FOREIGN_KEY_CHECKS = 0; 
delete from JBPM4_EXECUTION;  #执行删除操作
SET FOREIGN_KEY_CHECKS = 1;  # 操作结束后恢复外键 
```

## 相关FOREIGN_KEY查询
``` bash
SELECT TABLE_NAME,COLUMN_NAME,CONSTRAINT_NAME, REFERENCED_TABLE_NAME,REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE 
WHERE REFERENCED_TABLE_NAME = 'table_name';
```
 
## timeout
``` bash
connect_timeout：在获取连接阶段（authenticate）起作用
interactive_timeout和wait_timeout：在连接空闲阶段（sleep）起作用
net_read_timeout和net_write_timeout：则是在连接繁忙阶段（query）起作用。
```

## 查询实例大小&数据库大小
``` bash
select concat(round(sum(DATA_LENGTH/1024/1024),2), 'MB') as data from information_schema.TABLES;

select TABLE_SCHEMA, concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,
concat(truncate(sum(index_length)/1024/1024,2),'MB') as index_size
from information_schema.tables
group by TABLE_SCHEMA
order by data_length desc;
```

## 死锁处理
``` bash
#查询当前数据库正在运行的线程,判断是否有执行时间过长的查询
select * from information_schema.processlist where command not like 'sleep' and time > 5 \G
#查询当前事务 ，根据trx_state状态，查看对应lock wait的表
select * from information_schema.innodb_trx where trx_state like "%lock%"\G
#查询当前锁定的事务,根据lock_table得到相关被锁定表的事务ID(lock_trx_id)
select * from information_shcema.innodb_locks\G
#Kill 相关事务ID对应的线程ID
#RDS需调用存储过程 
CALL mysql.rds_kill('ID');
```
## 设置表名不区分大小写
lower_case_table_names = 1

## 数据库readonly设置
``` bash
#普通用户设置只读
set global read_only=1;
#普通用户取消只读
set global read_only=0;
#全局表锁   保证数据不会发生变更 mysqldump前可设置
flush tables with read lock;
#解除全局锁
unlock tables;
```

## mysqldump
``` bash
#--all-databases  导出所有数据库
#--master-data=1  该选项将binlog的位置和文件名追加到输出文件中
#--master-data=2  将位置和文件名添加注释，实际导入时需要手动输入master binlog 位置
#--single-transaction  在导出数据之前提交一个BEGIN,BEGIN不会阻塞任何应用程序且能保证数据库一致性

mysqldump -h $hostname -u$user -p$password --master-data=1 --all-databases --single-transaction > mysql.sql

#dump 数据库实例的所有信息(除去mysql,sys,information_schema 和performance_schema数据库)
mysql -uroot -pfruit@123 -BNe "SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME NOT IN ('mysql', 'performance_schema', 'information_schema', 'sys')" | tr 'n' ' ' > /root/dbs-to-dump.sql
#dump 数据库
mysqldump --routines --events --single-transaction --databases $(cat /root/dbs-to-dump.sql) > /root/full-data-dump.sql
#获取用户和权限信息. 该操作会备份所有用户的权限.
wget percona.com/get/pt-show-grants
yum install perl-DBD-MySQL
#确保/var/lib/mysql/mysql.sock存在，可以使用ln -s创建软链,或使用--host=127.0.0.1 --port=3306
ln -s /usr/local/mysql/mysqld.sock /var/lib/mysql/mysql.sock
perl pt-show-grants --user=root --password=root --flush > /root/grants.sql

#导入
mysql -uroot < /root/grants.sql
mysql -e "SET GLOBAL max_allowed_packet=1024*1024*1024";
mysql -uroot -p --max-allowed-packet=1G < /root/full-data-dump.sql;

```

## 查看数据库碎片& 整理碎片释放空间
``` bash
#Data_free 表示可回收的闲置空间
show table status like table_name;

#整理碎片 整理期间会锁表
OPTIMIZE table table_name;

#innoDB存储引擎 
innoDB引擎的表分为独享表空间和同享表空间的表，我们可以通过show variables like ‘innodb_file_per_table’;来查看是否开启独享表空间。
当开启独享表空间是无法对表进行optimize操作，会卡住很长时间并返回 Tables does not support optimize, doing recreate + analyze instead.
#可以改用这条语句优化 相当于重建表 也会出现锁表
ALTER TABLE yourdatabasename.yourtablename ENGINE='InnoDB';

```

### SQL
``` bash
#同一张表互换两列数据
update product as a, product as b set a.original_price=b.price, a.price=b.original_price where a.id=b.id;

```
