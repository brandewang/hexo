---
title: prometheus
date: 2088-08-08 08:08:08
tags:
---

- 官网: prometheus.io
- git: https://github.com/prometheus/prometheus

prometheus服务配置
```
mkdir -p /opt/monitor/prometheus

#prometheus.service
[Unit]
Description=prometheus

[Service]
ExecStart=/opt/monitor/prometheus/prometheus --config.file=/opt/monitor/prometheus/prometheus.yml --storage.tsdb.path=/opt/monitor/prometheus/data
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
Type=simple

[Install]
WantedBy=multi-user.target

#prometheus.yml 全局配置
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 127.0.0.1:9093

rule_files:
  - "rules/*.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
```

altermanager 服务配置
- git: https://github.com/prometheus/alertmanager
```
mkdir /opt/monitor/alertmanager

#alertmanager.service
[Unit]
Description=alertmanager

[Service]
ExecStart=/opt/monitor/alertmanager/alertmanager --config.file=/opt/monitor/alertmanager/alertmanager.yml
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target

#alertmanager.yml 全局配置参考
global:
  resolve_timeout: 5m #当alertmanager持续多长时间未接受到告警后，标记告警状态为resolved (已解决)
  #邮箱服务器
  smtp_smarthost: "smtphz.qiye.163.com:465"
  smtp_from: "ops@gihtg.com"
  smtp_auth_username: "ops@gihtg.com"
  smtp_auth_password: ""
  smtp_require_tls: false

# 告警模版
templates:
- "template/*.tmpl"


route:
  group_by: ['instance'] #对告警进行分组，将标签相同对告警合并为一个通知
  group_wait: 10s #触发告警后,等待10s,再发送通知,设置这个时间是为了尽可能多的将告警合并为一个通知。
  group_interval: 10s #相同Group之间发送告警通知的时间间隔
  repeat_interval: 10m #之前已经发送的告警,如果还没恢复,即一直处于FIRING状态,等10m后再次发送。
  receiver: 'admin'
  routes:
  - receiver: 'database-admin'
    group_wait: 10s
    match_re:
      service: mysql|cassandra
  - receiver: 'frontend-admin'
    group_by: [product, environment]
    match:
      team: frontend
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
- name: 'admin'
  email_configs:
  - to: 'brande.wang@gihtg.com'
    #html: '{{ template "test.html" . }} # 邮箱模版
    #headers: { Subject: "prometheus 告警测试邮件" } 邮件标题
    send_resolved: true
- name: 'database-admin'
  email_configs:
  - to: 'brande.wang@gihtg.com'
- name: 'frontend-admin'
  email_configs:
  - to: 'brande.wang@gihtg.com'
inhibit_rules:
  - source_match:
      severity: 'critical' #指定告警级别
    target_match:
      severity: 'warning'  #指定意志告警级别
    equal: ['alertname', 'instance'] #设置匹配相同的标签
```


# Linux监控 node_exporter
- git:https://github.com/prometheus/node_exporter
- grafana dashboard import id: 9276

#二进制部署
```
mkdir -p /opt/monitor/node_exporter

#node_exporter.service
[Unit]
Description=node_exporter

[Service]
ExecStart=/opt/monitor/node_exporter/node_exporter --collector.systemd --collector.systemd.unit-include=(sshd|docker).service
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```


# Docker容器监控 cAdvisor
- git:https://github.com/google/cadvisor/
- grafana dashboard import id: 193

#二进制部署
```
mkdir -p /opt/monitor/cadvisor

#cadvisor.service
[Unit]
Description=cAdvisor

[Service]
ExecStart=/opt/monitor/cadvisor/cadvisor --port=8080
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```


#docker部署
```
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest


在Ret Hat,CentOS, Fedora 等发行版上需要传递如下参数，因为 SELinux 加强了安全策略：
--privileged=true
启动后访问：http://127.0.0.1:8080查看页面，/metric查看指标
```

#MYSQL监控 mysqld_exporte
- git: https://github.com/prometheus/mysqld_exporter
- grafana dashboard import id: 7362

#二进制部署
```
mkdir -p /opt/monitor/mysqld_exporter

#mysql user create
CREATE USER 'exporter'@'localhost' IDENTIFIED BY '123456' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';

#/opt/monitor/mysqld_exporter/my.cnf
[client]
user=exporter
password=123456

#mysqld_exporter.service
[Unit]
Description=mysqld_exporter

[Service]
ExecStart=/opt/monitor/mysqld_exporter/mysqld_exporter --config.my-cnf=/opt/monitor/mysqld_exporter/my.cnf
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process

[Install]
WantedBy=multi-user.target
```

# Prometheus 自动发现
- file_sd_configs
- consul_sd_configs
- k8s_sd_configs

```
#file_sd_configs 基于文件的自动发现
#prometheus.yml
  - job_name: 'file_sd'
    file_sd_configs:
    - files: ['/opt/monitor/prometheus/sd_config/*.yml']
      refresh_interval: 5s
#sd_config/node.yml
- targets: ['10.55.5.25:9100']
  labels:
    team: 'team1'
    service: 'mysql'
- targets: ['10.55.3.100:9100']
  labels:
    team: 'team2'
    service: 'mongo' 

#consul_sd_configs 基于consul的自动发现
#prometheus.yml
  - job_name: 'node_exporter'
    consul_sd_configs:
    - server: '10.55.5.25:8500'
      services: ['Linux']

#consul注册服务
curl -X PUT -d '{"id":"gihtg-prometheus","name":"Linux","address":"10.55.3.100","port":9100,"tags":["service"],"checks":[{"http": "http://10.55.3.100:9100","interval":"5s"}]}' http://10.55.5.25:8500/v1/agent/service/register
#consul注销服务
curl -X PUT http://10.55.5.25:8500/v1/agent/service/deregister/gihtg-spring-config


#k8s_sd_configs 基于k8s的自动发现
#新增prometheus rbac权限
#prometheus-rbac.yml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - services
  - endpoints
  - pods
  - nodes/proxy
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - "extensions"
  resources:
    - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-system

#将sa的token写入token.k8s
echo $(kubectl describe secrets $(kubectl get secrets -n kube-system |grep prometheus |cut -f1 -d ' ') -n kube-system |grep -E '^token' |cut -f2 -d':'|tr -d '\t'|tr -d ' ') > token.k8s

#prometheus.yml
  - job_name: 'kubernetes-cadvisor'
    scheme: https
    tls_config:
      insecure_skip_verify: true
    bearer_token_file: /opt/monitor/prometheus/token.k8s
    kubernetes_sd_configs:
    - role: node
      api_server: https://10.55.3.88:8443
      bearer_token_file: /opt/monitor/prometheus/token.k8s
      tls_config:
        insecure_skip_verify: true
```


