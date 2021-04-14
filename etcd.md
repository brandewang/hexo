---
title: etcd
date: 2088-08-08 08:08:08
tags:
---

- 官网: etcd.io
- git: https://github.com/etcd-io/etcd

# 常用命令
```
#查看集群状态
etcdctl --cacert=/opt/etcd/ssl/ca.pem --cert=/opt/etcd/ssl/server.pem --key=/opt/etcd/ssl/server-key.pem endpoint status --cluster -w table
etcdctl --cacert=/opt/etcd/ssl/ca.pem --cert=/opt/etcd/ssl/server.pem --key=/opt/etcd/ssl/server-key.pem endpoint health --cluster -w table

#备份
etcdctl --cacert=/opt/etcd/ssl/ca.pem --cert=/opt/etcd/ssl/server.pem --key=/opt/etcd/ssl/server-key.pem snapshot save snap.db

#恢复
1. 先暂停kube-apiserver和etcd
systemctl stop kube-apiserver
systemctl stop etcd
mv /var/lib/etcd/default.etcd /var/lib/etcd/default.etcd.bak
2. 在每个节点上恢复
ETCDCTL_API=3 etcdctl snapshot restore snap.db \
--name etcd-1 \ --initial-cluster="etcd-1=https://192.168.31.71:2380,etcd-
2=https://192.168.31.72:2380,etcd-3=https://192.168.31.73:2380" \ --initial-cluster-token=etcd-cluster \ --initial-advertise-peer-urls=https://192.168.31.71:2380 \ --data-dir=/var/lib/etcd/default.etcd
3. 启动kube-apiserver和etcd
systemctl start kube-apiserver systemctl start etcd
```

