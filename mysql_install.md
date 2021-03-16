---
title: MYSQL_INSTALL
date: 2088-08-08 08:08:08
tags:
---


# install dependencies
yum install -y cmake make gcc gcc-c++ bison ncurses ncurses-devel
cd /usr/local/src &&wget 

https://sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
tar -zxvf boost_1_59_0.tar.gz -C /usr/local/

#创建mysql系统用户
groupadd mysql
useradd -r -g mysql -s /bin/nologin mysql

#install mysql (mysql5.5后使用cmake编译)
cd /usr/local/src &&\
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.20.tar.gz &&\
tar zxvf mysql-5.7.20.tar.gz  &&\ 
cd /usr/local/src/mysql-5.7.20 &&\
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DWITH_BOOST=/usr/local/boost_1_59_0 \
-DSYSCONFDIR=/etc \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DENABLED_LOCAL_INFILE=1 \
-DEXTRA_CHARSETS=all


-DCMAKE_INSTALL_PREFIX：安装路径
-DMYSQL_DATADIR：数据存放目录
-DWITH_BOOST：boost源码路径
-DSYSCONFDIR：my.cnf配置文件目录
-DDEFAULT_CHARSET：数据库默认字符编码
-DDEFAULT_COLLATION：默认排序规则
-DENABLED_LOCAL_INFILE：允许从本文件导入数据
-DEXTRA_CHARSETS：安装所有字符集

----------------------------------------------------------------------------------

make -j `grep processor /proc/cpuinfo | wc -l` 
-j参数表示根据CPU核数指定编译时的线程数，可以加快编译速度。默认为1个线程编译，经测试

单核CPU，1G的内存，编译完需要将近1个小时。
make install

#配置文件参数优化
cat > /etc/my.cnf <<EOF
[client]
port=3306
socket=/usr/local/mysql/mysql.sock
[mysqld]
#sql_mode 设置
#宽松模式 not null 自动赋空
sql_mode='NO_ENGINE_SUBSTITUTION'
#严格模式
#sql_mode='ONLY_FULL_GROUP_BY,NO_AUTO_VALUE_ON_ZERO,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,
ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,PIPES_AS_CONCAT,ANSI_QUOTES'

#字符集设
log_timestamps = SYSTEM
character-set-server=utf8
collation-server=utf8_general_ci

skip-external-locking
skip-name-resolve

user=mysql
port=3306

#系统及日志时区设置
system_time_zone=CST
time_zone=SYSTEM
log_timestamps=SYSTEM


#数据及日志存放位置
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
tmpdir=/usr/local/mysql/temp
socket=/usr/local/mysql/mysql.sock
log-error=/usr/local/mysql/logs/mysql_error.log
pid-file=/usr/local/mysql/mysql.pid
open_files_limit=65535
back_log=600
max_connections=500
max_connect_errors=6000
interactive_timeout=604800
wait_timeout=604800
#open_tables=600
#table_cache = 650
#opened_tables = 630

#binary log 设置
#binlog-format=ROW
log-bin=binlog/mysql-bin
log-bin-index=binlog/mysql-bin.index
max_binlog_size=1g
binlog_cache_size=4m
#设置为1每次I/O写入磁盘，推荐为1，比较安全
sync_binlog=1


---
innodb_flush_log_at_trx_commit：0,1,2
n = 0（高效，但不安全--无论服务器宕机或者mysql宕机都会丢数据）
每隔一秒，把事务日志缓存区的数据写到日志文件中，以及把日志文件的数据刷新到磁盘上
n = 1 （低效，非常安全--都不会丢数据）
每个事务提交时候，把事务日志从缓存区写到日志文件中，并且，刷新日志文件的数据到磁盘上，优化使用此模式保证数据安全性
n = 2（高效，但不安全--服务器宕机会丢数据）
每个事务提交的时候，把事务日志数据从缓存区写到日志文件中，每隔一秒，刷新一次日志文件，但不一定刷新到磁盘上，而是取决于操作系统的调度；
如何保证事务安全
- innodb_flush_log_at_trx_commit&&sync_binlog 都设为1
- 事务要和binlog保证一致性---才不会导致主从不一致
---


max_allowed_packet=32M
sort_buffer_size=4M
join_buffer_size=4M
thread_cache_size=300
query_cache_type=1
query_cache_size=256M
query_cache_limit=2M
query_cache_min_res_unit=16k

tmp_table_size=256M
max_heap_table_size=256M

key_buffer_size=256M
read_buffer_size=1M
read_rnd_buffer_size=16M
bulk_insert_buffer_size=64M
#设置表明不区分大小写
lower_case_table_names=1
#默认数据库引擎
default-storage-engine=INNODB

innodb_purge_threads=1
innodb_file_per_table=1
innodb_buffer_pool_size=2G
innodb_log_buffer_size=32M
innodb_log_file_size=512M
innodb_log_files_in_group=3
innodb_flush_log_at_trx_commit=1
innodb_flush_method=O_DIRECT
#thread_concurrency=32

#慢查询设置
long_query_time=2
slow-query-log=on
slow-query-log-file=/usr/local/mysql/logs/mysql-slow.log

#slave 从库设置
#logs-slave-updates=1 开启后从库可写入bin log 来自主库的复制
#read-only=1 普通用户只读
#master-info-file=master.info
#relay-log=relay/relay-bin
#relay-log-index=relay/relay-bin.index
#relay-log-info-file=relay/relay-log.info
#当slave从库宕机后，假如relay-log损坏了,放弃原有的relay-log,重新从master上获取日志,保证了relay-log的完整性。默认情况下该功能是关闭的,建议开启。
#relay_log_recovery=1
#当设置为1 每次I/O都写入磁盘，设置为0由操作系统决定，建议默认不设置
#sync_relay_log=
#sync_relay_log_info=


[mysqldump]
quick
max_allowed_packet=32M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

EOF

#初始化数据库
```bash
cd /usr/local/mysql
mkdir /usr/local/mysql/logs&& mkdir /usr/local/mysql/temp && mkdir /usr/local/mysql/data
chown -R mysql:mysql .

* 注意：MySQL 5.7.6之前的版本执行这个脚本初始化系统数据库
./bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
* 5.7.6之后版本初始系统数据库脚本（本文使用此方式初始化）
./bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
chown -R root .
chown -R mysql data temp 
chown mysql .

./bin/mysql_ssl_rsa_setup


#配置mysql服务
cp support-files/mysql.server /etc/init.d/mysqld

#设置数据库密码
/usr/local/mysql/bin/mysql -e "
grant all privileges on *.* to 'root'@'localhost' identified by 'fruit@123' with grant option;"

#配置mysql环境变量
shell> vim /etc/profile
shell> export PATH=/usr/local/mysql/bin:$PATH
shell> source /etc/profile


————————————————————————
如果中途编译失败了，需要删除cmake生成的预编译配置参数的缓存文件和make编译后生成的文

件，再重新编译。
shell> cd /opt/mysql-server
shell> rm -f CMakeCache.txt
shell> make clean

```
