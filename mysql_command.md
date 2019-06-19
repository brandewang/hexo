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

#5.7+ version
systemctl stop mysqld
systemctl set-environment MYSQLD_OPTS=”–skip-grant-tables”
systemctl start mysqld
mysql -u root
mysql> UPDATE mysql.user SET authentication_string = PASSWORD(‘MyNewPassword’) WHERE User = ‘root’ AND Host = ‘localhost’; 
mysql> FLUSH PRIVILEGES; 
mysql> quit
systemctl stop mysqld
systemctl unset-environment MYSQLD_OPTS
systemctl start mysqld
echo 'validate_password_policy=0' >> /etc/my.cnf
ALTER USER testuser IDENTIFIED BY '123456';
```

## 权限设置
``` bash
grant all privileges on *.* to 'root'@'localhost' identified by 'fruit@123' with grant option;
flush privileges;
```

## 日志清理
``` bash
#什么时候会清理过期日志，每次进行LOG flush 时会自动删除过期日志，触发log flush:
1. 重启MYSQL
2. bin-log文件大小打到参数max_binlog_size限制
3. 手动执行清理命令

#自动清理
#设置超过7天的日志自动删除
vim my.conf
expire_logs_days =7 

#手动清理
#如果没有主从,可以通过下面的命令重置数据库日志，清除之前的日志文件
reset master
#如果存在复制关系，可以通过PURGE来清理：
mysql -uroot -p
#清除某个bin-log
purge master logs to 'mysql-bin.010'
#清除某个时间之前的日志
purge master logs before '2016-02-28 13:00:00'
#清除3天前的日志
purge master logs before date_sub(now(), inverval 3 day); 
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

## 查询实例大小&数据库大小&表大小
``` bash
select concat(round(sum(DATA_LENGTH/1024/1024),2), 'MB') as data from information_schema.TABLES;

select TABLE_SCHEMA, concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,
concat(truncate(sum(index_length)/1024/1024,2),'MB') as index_size
from information_schema.tables
group by TABLE_SCHEMA
order by data_length desc;

select table_schema,table_name,engine,table_rows,data_length+index_length length,DATA_FREE from information_schema.tables where engine='InnoDB' and DATA_FREE!=0;
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
mysqldump --routines --events --lock-all-tables --databases $(cat /root/dbs-to-dump.sql) > /root/full-data-dump.sql
#不导出gtid(用于主从同步)可加入该参数 --set-gtid-purged=off
#锁定全表 --lock-all-tables (允许中断时使用 锁住方式类似 flush tables with read lock 的全局锁)
#使用事务保证一致性 --single-transaction (不允许中断时使用)

  
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

## mysql binlog
``` bash
#mysql中查看binlog
1.获取binlog文件列表
mysql> show binary logs;
2.查看当前正在写入的binlog文件
mysql> show master status;
3.查看指定binlog文件的内容语法：
mysql> SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]
mysql> SHOW BINLOG EVENTS IN 'mysql-bin.000005' \G
mysql> SHOW BINLOG EVENTS IN 'mysql-bin.000005' FROM 194 LIMIT 2 \G;

#mysqlbinlog 使用
a、提取指定的binlog日志  
# mysqlbinlog /opt/data/APP01bin.000001  
# mysqlbinlog /opt/data/APP01bin.000001|grep insert  
/*!40019 SET @@session.max_insert_delayed_threads=0*/;  
insert into tb values(2,'jack')  
  
b、提取指定position位置的binlog日志  
# mysqlbinlog --start-position="120" --stop-position="332" /opt/data/APP01bin.000001  
  
c、提取指定position位置的binlog日志并输出到压缩文件  
# mysqlbinlog --start-position="120" --stop-position="332" /opt/data/APP01bin.000001 |gzip >extra_01.sql.gz  
  
d、提取指定position位置的binlog日志导入数据库  
# mysqlbinlog --start-position="120" --stop-position="332" /opt/data/APP01bin.000001 | mysql -uroot -p  
  
e、提取指定开始时间的binlog并输出到日志文件  
# mysqlbinlog --start-datetime="2014-12-15 20:15:23" /opt/data/APP01bin.000002 --result-file=extra02.sql  
  
f、提取指定位置的多个binlog日志文件  
# mysqlbinlog --start-position="120" --stop-position="332" /opt/data/APP01bin.000001 /opt/data/APP01bin.000002|more  
  
g、提取指定数据库binlog并转换字符集到UTF8  
# mysqlbinlog --database=test --set-charset=utf8 /opt/data/APP01bin.000001 /opt/data/APP01bin.000002 >test.sql  
  
h、远程提取日志，指定结束时间   
# mysqlbinlog -urobin -p -h192.168.1.116 -P3306 --stop-datetime="2014-12-15 20:30:23" --read-from-remote-server mysql-bin.000033 |more  
  
i、远程提取使用row格式的binlog日志并输出到本地文件  
# mysqlbinlog -urobin -p -P3606 -h192.168.1.177 --read-from-remote-server -vv inst3606bin.000005 >row.sql  


# RDS日志提取
#--to-last-log 到最后一个binlog日志
#--result-file output文件
mysqlbinlog  --read-from-remote-server --host=127.0.0.1 --port=3306  --user root --password   --to-last-log  --result-file=/tmp/xx mysql-bin.000001

```

## SQL对象收集
``` bash
#1.1查看所有视图
SHOW FULL TABLES IN dms_sample WHERE TABLE_TYPE LIKE 'VIEW';     
 
#1.2查看索引
SELECT DISTINCT
  TABLE_NAME,
  INDEX_NAME
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = 'dms_sample';
 
#1.3查看外键约束
USE INFORMATION_SCHEMA;
SELECT TABLE_NAME,
       COLUMN_NAME,
       CONSTRAINT_NAME,
       REFERENCED_TABLE_NAME,
       REFERENCED_COLUMN_NAME
FROM KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = "dms_sample"
      AND REFERENCED_COLUMN_NAME IS NOT NULL;
 
#1.4查看所有用户
select host, user from mysql.user;
 
#1.5查看所有触发器
use dms_sample;
show triggers;
 
#1.6查看所有存储过程
select name from mysql.proc where db = 'dms_sample' and type = 'PROCEDURE';
```


### SQL
``` bash
#同一张表互换两列数据
update product as a, product as b set a.original_price=b.price, a.price=b.original_price where a.id=b.id;

```
