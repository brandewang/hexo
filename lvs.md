---
title: LVS
date: 2088-08-08 08:08:08
tags:
---

# LVS DR模式配置
``` bash
Director node:(eth0 192.168.0.8  vip eth0:0 192.168.0.38)
Real server1:(eth0 192.168.0.18 vip lo:0 192.168.0.38)
Real server2:(eth0 192.168.0.28 vip lo:0 192.168.0.38)

# director
yum install -y ipvsadm

cat > /home/www/publish-shell/lvs_dr.sh <EOF
echo 1 > /proc/sys/net/ipv4/ip_forward
ipv=/sbin/ipvsadm
vip=192.168.0.38
rs1=192.168.0.18
rs2=192.168.0.28
ifconfig eth0:0 down
ifconfig eth0:0 $vip broadcast $vip netmask 255.255.255.255 up
route add -host $vip dev eth0:0
$ipv -C
$ipv -A -t $vip:80 -s wrr
$ipv -a -t $vip:80 -r $rs1:80 -g -w 3
$ipv -a -t $vip:80 -r $rs2:80 -g -w 1
EOF
bash /home/www/publish-shell/lvs_dr.sh

# rs
cat > /home/www/publish-shell/lvs_dr_rs.sh <EOF
#! /bin/bash
vip=192.168.0.38
ifconfig lo:0 $vip broadcast $vip netmask 255.255.255.255 up
route add -host $vip lo:0
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
EOF
bash /home/www/publish-shell/lvs_dr_rs.sh

#ipvsadm查看当前状态
ipvsadm -l
#ipvsadm规则清空
ipvsadm -C

# keepalive配置
keepalive通过检测ip地址存活状态来管理ipvsadm规则
#安装
yum install -y ipvsadm keepalived
#打开ip地址转发
echo 1 > /proc/sys/net/ipv4/ip_forward
#修改keepalived配置信息
#backup节点需要调整两个值
#state MASTER -> state BACKUP
#priority 100 -> priority 90

cat > /etc/keepalived/keepalived.conf << EOF
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.28.20.55
    }
}

virtual_server 10.28.20.55 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    persistence_timeout 0
    protocol TCP

    real_server 10.28.20.48 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }

    real_server 10.28.20.49 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}
EOF

```



