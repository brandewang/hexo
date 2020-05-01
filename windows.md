---
title: windows
date: 2088-08-08 08:08:08
tags:
---

## 时间服务
``` bash
#验证pdc
w32tm /monitor /domain:book.com 

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
```

