---
title: grafana
date: 2088-08-08 08:08:08
tags:
---

- url:https://grafana.com/dashboards?search=zabbix

服务配置
```
mkdir /opt/monitor/grafana
#grafana.service
[Unit]
Description=grafana

[Service]
ExecStart=/opt/monitor/grafana/bin/grafana-server -homepath=/opt/monitor/grafana
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

docker部署
``` bash
#!/bin/bash
docker run -d \
-p 3000:3000 \
--name=grafana \
-e "GF_INSTALL_PLUGINS=alexanderzobnin-zabbix-app" \
-v /opt/grafana/etc:/etc/grafana -v /opt/grafana/var:/var/lib/grafana \
grafana/grafana:6.7.3

```

## 修改Home
``` bash
#root权限进入
docker exec -it --user root grafana /bin/bash
#备份home.json
cp -a /usr/share/grafana/public/dashboards/home.json{,.bak}
#根据需求编写home.json
```
