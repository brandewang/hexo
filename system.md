---
title: system
date: 2018-05-02 10:00
tags:
---
# 持久化systemd-journal日志
``` bash
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/jounal
systemctl restart systemd-journald
```
