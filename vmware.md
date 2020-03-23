---
title: vmware
date: 2088-08-08 08:08:08
tags:
---

#install
``` bash
#当前版本
vsphere6.7u3b
vc6.7u3b
PS:essential版本缺少大部分集群功能也不包含vMotion以及vsphere distributed switch


#配置时区
tzselect
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#配置AD认证
VC的FQDN需要与在AD注册的计算机名保持一致
```



