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
```
