---
title: MYSQL_COMMAND
date: 2018-03-23 09:50:38
tags:
---

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

## 查询数据库大小
``` bash
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
