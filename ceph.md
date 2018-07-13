---
title: ceph
date: 2088-08-08 08:08:08
tags:
---
# install ceph 
``` bash
#配置yum源
##rpm-jewel##ceph版本,字母开头j为10
##el7##操作系统版本

cat >> /etc/yum.repos.d/ceph.repo <EOF
[Ceph]
name=Ceph packages for $basearch
baseurl=http://download.ceph.com/rpm-jewel/el7/$basearch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1

[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://download.ceph.com/rpm-jewel/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1

[ceph-source]
name=Ceph source packages
baseurl=http://download.ceph.com/rpm-jewel/el7/SRPMS
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1
EOF

#创建临时配置文件路径
mkdir ~/ceph-cluster && cd ~/ceph-cluster
#安装ceph-deploy
yum install -y ceph-deploy
#生成配置文件并添加mon节点，mon节点需为奇数，该配置文件当使用ceph-deploy install node时推送覆盖节点本身当配置文件
ceph-deploy new node1 node2 node3
#设置pool的默认复制数量，这里设置为2。
echo 'osd pool default size = 2' > ~/ceph-cluster/ceph.conf
#在节点安装ceph软件包并推送配置文件,如果是mon节点则会进一步安装mon服务
ceph-deploy install node1 node2 node3
```


# command
```
#创建一个osd
ceph-deploy osd prepare brande-ceph02:/var/local/osd0 brande-ceph03:/var/local/osd1
ceph-deploy osd activate brande-ceph02:/var/local/osd0 brande-ceph03:/var/local/osd1
#查看osd元数据
ceph osd metadata
#删除一个osd
查看ceph pg dump，确认osd不被单独占用
ceph osd crush remove osd.2
ceph osd rm osd.2
#删除osd相关权限
ceph auth delete osd.2
```
