---
title: nfs
date: 2088-08-08 08:08:08
tags:
---

#nfs
``` bash
#server
yum install -y nfs-utils rpcbind
systemctl enable rpcbind nfs-server
systemctl start rpcbind nfs-server

vim /etc/exports
/opt/nfs/backup 10.55.5.0/24(rw)

#client
mount -t nfs -o rw,soft,intr,timeo=30,retry=3 10.55.5.11:/opt/nfs/backup /opt/bak
```
