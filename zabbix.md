---
title: zabbix
date: 2088-08-08 08:08:08
tags:
---
# Zabbix

## IPMI

``` bash
# 安装ipmitool
yum install -y ipmitool
# 获取目标传感器数据, 这里10.28.200.10为(idrac8 ipmi), 需在web页面idrac设置->网络设置->ipmi 中勾选lan
ipmitool -I lanplus -H 10.28.200.10 -U root -P fruit@123 sdr list
ipmitool -I lanplus -H 10.28.200.10 -U root -P fruit@123 sdr get 'Temp'

# 设置zabbix item
# 如Sensor ID(通过ipmitool获取)不重复,则使用以下格式
Type:IPMI agent
IPMI sensor: Temp
# 如Sensor ID重复,则设置key:value格式,完整的name可从"grep 'Added sensor' /var/log/zabbix/zabbix_server.log"中获取
Type:IPMI agent
IPMI sensor: name:(3.2).Temp
```
