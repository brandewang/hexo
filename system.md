---
title: system
date: 2088-08-08 08:08:08
tags:
---
# 持久化systemd-journal日志
``` bash
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/jounal
systemctl restart systemd-journald
```
