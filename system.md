---
title: system
date: 2088-08-08 08:08:08
tags:
---
# 持久化systemd-journal日志
``` bash
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/jounal
systemctl restart systemd-journald
```

## 常用命令

``` bash
#丢包率
lostpk=`timeout 5  ping -q -s 500 202.96.209.133 -W 1000 -c 100 -A |grep transmitted|awk '{print $6}'`
#总耗时
rrt=`timeout 5  ping -q -s 500 202.96.209.133 -W 1000 -c 100 -A |grep transmitted|awk '{print $10}'`
