---
title: keepalived
date: 2088-08-08 08:08:08
tags:
---

# keepalived
``` bash
#注意事项
virtual_router_id 局域网中不可重复
systemctl stop keepalived可能会出现问题，需要ps查看实际情况，kil来进行停止进程

#nginx master配置
vrrp_script chk {
  script "/usr/bin/netstat -ntlp|grep docker|grep 80"
  interval 2
  weight -20
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 66
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk
    }
    virtual_ipaddress {
        10.28.50.119
    }
}

```
