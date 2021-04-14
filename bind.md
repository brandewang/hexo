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
-----master------
vim /etc/named.conf
#添加局域网监听地址，添加允许查询地址段，设置转发即使存在zone . 也不使用
listen-on port 53 { 127.0.0.1;10.28.50.6 };
allow-query     { localhost;10.28.50.0/24;any; };
forward only;
forwarders { 202.96.209.133; };
#设置允许复制的slave的ip，不允许则为none
allow-transfer  { 10.28.20.61; };

#检查配置文件
named-checkconf


#确认include此文件
include "/etc/named.rfc1912.zones";

#修改zone配置文件，添加新的解析域，指向数据文件/var/named/db.cephcookbook.com
vim /etc/named.rfc1912.zones

zone "cephcookbook.com" IN {
    type master;
    file "db.cephcookbook.com";
    allow-update { none; };
    allow-transfer { 10.28.20.61;};
};

#vim /var/named/db.cephcookbook.com
$TTL 600; 10 minutes
@ IN SOA cephcookbook.com. root.cephcookbook.com. (
	20180704 ; serial
	3H ; refresh
	15M ; retry
	1W ; expire
	1D ) ; minimum
@ IN NS cephcookbook.com.
@ IN A 10.28.50.6
#设置slave NS记录
@ IN NS slave
slave IN A 10.28.20.61
#这里的10为MX的权重
@ IN MX 10 mail


* IN A 10.28.201.61
brande IN A 10.28.201.62
angel IN CNAME brande
mail IN A 10.28.201.63

-----slave------
#为slave 写入zone信息,重启服务后将自动生成 file, file类型需设置text, 否则可能会造成乱码
vim /etc/named.rfc1912.zones
zone "cephcookbook.com" IN {
        type slave;
        file "slaves/db.cephcookbook.com";
        masterfile-format text;
        masters { 10.28.50.6;};
};

```

docker
```
mkdir -p /opt/bind && docker run --name='bind' -d -p 53:53/udp -p 53:53/tcp -p 10000:10000/tcp   -e WEBMIN_ENABLED=true   -v /opt/bind:/data   sameersbn/bind:latest

options {
        dnssec-enable no;
        dnssec-validation no;
        recursion yes;
        auth-nxdomain no;    # conform to RFC1035
        allow-transfer {
             any;
                };
        allow-query {
                any;
                };
        forwarders {
                202.96.209.133;
                202.96.209.5;
                8.8.8.8;
                114.114.114.114;
                };
}
```
