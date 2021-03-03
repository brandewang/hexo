---
title: consul
date: 2088-08-08 08:08:08
tags:
---

# 注册
```
curl -X PUT -d '{"id": "cdh-node2","name": "cdh-node2","address": "10.23.215.217","port": 9100,"tags": ["dev"],"checks": [{"http": "http://10.23.215.217:9100/","interval": "5s"}]}' http://10.23.215.236:8500/v1/agent/service/register
```

# 删除 指定ID
```
curl -X PUT http://10.23.215.236:8500/v1/agent/service/deregister/cdh-node2
```
