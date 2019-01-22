---
title: Glusterfs
date: 2088-08-08 08:08:08
tags:
---
#  Glusterfs
``` bash
#安装
1.配置epel
yum install http://mirrors.163.com/centos/7.3.1611/extras/x86_64/Packages/epel-release-7-9.noarch.rpm
2.配置glusterfs yum repo
cat >>  /etc/yum.repos.d/gluster.repo <EOF
[gluster]
name=gluster
baseurl=https://buildlogs.centos.org/centos/7/storage/x86_64/gluster-3.8/
gpgcheck=0
enabled=1
EOF
3.安装启动相关服务(手动解决依赖问题)
yum install -y glusterfs-server samba rpcbind
systemctl start glusted.serivce
systemctl enable glusterd.service
systemctl start rpcbind
systemctl enable rpcbind
systemctl status rpcbind
#如果已禁用ipv6,请关闭相应service中的配置
vi /usr/lib/systemd/system/rpcbind.socket
4.hosts文件配置
vi /etc/hosts
#peer vol管理
1.加入伙伴节点
#任意节点操作
gluster peer probe node2
gluster peer probe node3
2.查看伙伴状态
gluster peer status
3.创建卷
#分布卷
gluster volume create vol1 \
> node1:/glusterfs/sdb/vol1 \
> node2:/glusterfs/sdb/vol1 \
> node3:/glusterfs/sdb/vo1
#分布条带复制卷
gluster volume create vol1 str 2 repl 2 \
> node1:/glusterfs/sdb/vol1 \
> node2:/glusterfs/sdb/vol1 \
> node3:/glusterfs/sdb/vol1 \
> node4:/glusterfs/sdb/vol1
4.查看卷状态
gluster volume status
gluster volume info vol1
5./启/停/删除卷
gluster volume start vol1
gluster volume stop vol1
gluster volume delete vol1
6.扩展收缩卷 (扩展或收缩卷时，也要按照卷的类型，加入或减少的brick个数必须满足相应的要求)
$gluster volume add-brick mamm-volume [strip|repli <count>] brick1...
$gluster volume remove-brick mamm-volume [repl <count>] brick1...
7.迁移卷(替换)
volume replace-brick <VOLNAME> <SOURCE-BRICK> <NEW-BRICK> {commit force}

#客户端链接
1.安装
yum install glusterfs glusterfs-fuse attr -y
2.glusterfs方式挂载(默认支持高可用)
mount -t glusterfs node2:vol1 /mnt/vol1
3.nfs方式挂载
#打开卷nfs
gluster volume set vol01 nfs.disable off
mount -t nfs 10.28.20.67:/vol01 /vol01_nfs/

#如出现gluster卷不可用,umount对应路径即可
```
