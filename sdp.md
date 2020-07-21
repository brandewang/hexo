---
title: service desk
date: 2088-08-08 08:08:08
tags:
---

# readme

``` bash
Linux版本64位：
https://download.manageengine.com/products/service-desk/91677414/ManageEngine_ServiceDesk_Plus_64bit.bin
Linux版本32位：
https://download.manageengine.com/products/service-desk/91677414/ManageEngine_ServiceDesk_Plus.bin
安装手册:
https://help.servicedeskplus.com/introduction/installation-linux.html
优化:
https://help.servicedeskplus.com/general-features/performance-guide.html

#centos8
ln -s /usr/lib64/libtinfo.so.6.1 /usr/lib64/libtinfo.so.5

#Postgresql  解决时区调用32位so的包
yum install glib.i686 
ln -s /tmp/.s.PGSQL.65432 /tmp/.s.PGSQL.5432
./psql -U sdpadmin  -d servicedesk
sdp@123

#/opt/sdp/ServiceDesk/pgsql/ext_conf/00framework_ext.conf
log_timezone = 'PRC'
timezone = 'PRC'

#report
RHEL / Centos:
sudo yum install fontconfig dejavu-sans-fonts dejavu-serif-fonts

#restore
bin/restoreData.sh -c /opt/sdp/ServiceDesk/backup/backup_postgres_11110_fullbackup_05_20_2020_07_50/backup_postgres_11110_fullbackup_05_20_2020_07_50_part_1.data

```
