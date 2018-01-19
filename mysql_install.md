---
title: MYSQL_INSTALL
date: 2018-01-19 09:50:38
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
log_timestamps = SYSTEM
character-set-server=utf8
collation-server=utf8_general_ci

skip-external-locking
skip-name-resolve

user=mysql
port=3306
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
tmpdir=/usr/local/mysql/temp
# server_id = .....
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

lower_case_table_names=1

default-storage-engine=INNODB

innodb_purge_threads=1
innodb_file_per_table=1
innodb_buffer_pool_size=2G
innodb_log_buffer_size=32M
innodb_log_file_size=512M
innodb_log_files_in_group=3
innodb_flush_log_at_trx_commit=2
innodb_flush_method=O_DIRECT
#thread_concurrency=32
long_query_time=2
slow-query-log=on
slow-query-log-file=/usr/local/mysql/logs/mysql-slow.log

[mysqldump]
quick
max_allowed_packet=32M

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

EOF

#初始化数据库
cd /usr/local/mysql
mkdir /usr/local/mysql/logs&& mkdir /usr/local/mysql/temp
chown -R mysql:mysql .

* 注意：MySQL 5.7.6之前的版本执行这个脚本初始化系统数据库
./bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql --

datadir=/usr/local/mysql/data
* 5.7.6之后版本初始系统数据库脚本（本文使用此方式初始化）
./bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --

datadir=/usr/local/mysql/data
chown -R root .
chown -R mysql data temp 
chown mysql .

./bin/mysql_ssl_rsa_setup


#配置mysql服务
cp support-files/mysql.server /etc/init.d/mysqld

#设置数据库密码
/usr/local/mysql/bin/mysql -e "grant all privileges on *.* to 'root'@'localhost' 

identified by 'fruit@123' with grant option;"

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


