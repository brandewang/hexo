---
title: ntp
date: 2088-08-08 08:08:08
tags:
---

# ntp docker
``` bash
#!/bin/bash
docker run -idt --name=ntp\
           --restart=always\
           --detach=true\
           --publish=123:123/udp\
           --cap-add=SYS_TIME\
           cturra/ntp
```
