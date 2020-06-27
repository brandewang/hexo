---
title: MYSQL
date: 2088-08-08 08:08:08
tags:
---


# install

``` bash
#安装yum仓库
rpm -Uvh https://repo.mysql.com//mysql80-community-release-el7-3.noarch.rpm
#编辑yum仓库文件，选择需要的mysql版本
/etc/yum.repos.d/mysql-community.repo
#显示所有版本
yum list mysql-community-server --showduplicates
#安装指定版本mysql
#mysql-community-server.x86_64	5.7.30-1.el7
yum install mysql-community-server-5.7.30-1.el7.x86_64

#初始化启动mysqld
systemctl start mysqld

#查看日志,获取初始化mysql默认密码
vi /var/log/mysqld.log

#登陆后修改默认root密码
alter user root@localhost identified by 'somepassword'
flush privileges


```

# optimize

``` bash
cat > my.cnf << EOF
[mysqld]
#data
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
pid-file=/var/run/mysqld/mysqld.pid

symbolic-links=0

#log
log-error=/var/log/mysqld.log

#language
log_timestamps = SYSTEM
character-set-server=utf8
collation-server=utf8_general_ci
lower_case_table_names=1

#connect
open_files_limit=65535
back_log=600
max_connections=3000
max_connect_errors=6000
interactive_timeout=604800
wait_timeout=604800

#innodb
default-storage-engine=INNODB
innodb_buffer_pool_size=12G  #推荐设置为系统内存的80%

#slowquery
long_query_time=3
slow-query-log=on

EOF
```


