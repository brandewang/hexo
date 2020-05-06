---
title: windows
date: 2088-08-08 08:08:08
tags:
---

## 时间服务

``` bash
#初始化组件
w32tm /unregister
w32tm /register

#验证pdc
w32tm /monitor /domain:gihg.com 

#查看同步源
w32tm /query /source
#查看同步状态
w32tm /query /status
#查看时间同步配置
w32tm /query /configuration
#查看时区
w32tm /tz
#手动同步
w32tm /resync
#服务启停
net start/stop w32time
#重新发现时间源
w32tm /resync /rediscover

#查看ntp服务器连接状态
w32tm /stripchart /computer:ntp1.aliyun.com
#手动配置ntpclient
w32tm /config /manualpeerlist:ntp1.aliyun.com /syncfromflags:manual /reliable:yes /update
w32tm /config /manualpeerlist:"ntp1.aliyun.com,0x1 ntp2.aliyun.com,0x1 ntp3.aliyun.com,0x1 ntp4.aliyun.com,0x1" /syncfromflags:manual /reliable:yes  /update
#手动配置nt5ds
w32tm /config /syncfromflags:domhier /reliable:yes /update

```

## AD服务

``` bash
#检查域控错误信息
dcdiag /s:host.domain.com
#ad同步检查
repadmin /showrepl
repadmin /syncall
```
