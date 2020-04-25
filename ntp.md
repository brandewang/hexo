---
title: ntp
date: 2088-08-08 08:08:08
tags:
---

# ntp docker
``` bash
#!/bin/bash
docker run --name=ntp \
 --restart=always \
 --detach=true \
 --publish=123:123/udp \
 --cap-add=SYS_TIME \
 --env=NTP_SERVERS='ntp1.aliyun.com,ntp2.aliyun.com,ntp3.aliyun.com,ntp4.aliyun.com' \
 cturra/ntp


查看时间同步源：
chronyc sources -v

查看时间同步源状态：
chronyc sourcestats -v

#校准时间服务器
chronyc tracking

#同步 先校准
chronyc -a makestep
ntpdate 10.28.50.10

```
