---
title: bind
date: 2088-08-08 08:08:08
tags:
---
# install bind
``` bash
# install
yum install bind*
# 修改bind配置文件
vim /etc/named.conf
#添加局域网监听地址，添加允许查询地址段，设置转发即使存在zone . 也不使用
listen-on port 53 { 127.0.0.1;10.28.50.6 };
allow-query     { localhost;10.28.50.0/24; };
forward only;
    forwarders {
    202.96.209.133;
 };
#确认include此文件
include "/etc/named.rfc1912.zones";

#修改zone配置文件，添加新的解析域，指向数据文件/var/named/db.cephcookbook.com
vim /etc/named.rfc1912.zones

zone "cephcookbook.com" IN {
    type master;
    file "db.cephcookbook.com";
    allow-update { none; };
};

#vim /var/named/db.cephcookbook.com
$TTL 86400
@ IN SOA cephcookbook.com. root.cephcookbook.com. (
20180704 ; serial
28800 ; refresh
14400 ; retry
3600000 ; expire
86400 ) ; minimum
@ IN NS cephcookbook.com.
@ IN A 10.28.50.6


* IN A 10.28.201.61
brande IN A 10.28.201.62
angel IN CNAME brande
```


