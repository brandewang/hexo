---
title: ElasticStack
date: 2088-08-08 08:08:08
tags:
---

- url: elastic.co

启动先决条件:
- 调整进程最大打开文件数数量
```
#临时设置
ulimit -n 65535
#永久设置
vi /etc/security/limits.conf
* hard nofile 65535
* soft nofile 65535
```
- 调整进程最大虚拟内存区域数量
```
#临时设置
sysctl -w vm.max_map_count=262144
#永久设置
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
sysctl -p
```

elasticsearch二进制部署
```
mkdir -p /opt/elk
cd /opt/elk
tar zxvf elasticsearch-7.9.3-linux-x86_64.tar.gz
mv elasticsearch-7.9.3 elasticsearch
useradd es
chown -R es:es elasticsearch

#elasticsearch.yml
cluster.name: elk-cluster    # 集群名称
node.name: node-1           # 集群节点名称
#path.data: /path/to/data   # 数据目录
#path.logs: /path/to/logs   # 日志目录
network.host: 0.0.0.0       # 监听地址
http.port: 9200             # 监听端口
#transport.tcp.port: 9300   # 内部节点之间通信端口
discovery.seed_hosts: ["10.55.3.71", "10.55.3.72", "10.55.3.73"] # 集群节点列表
cluster.initial_master_nodes: ['node-1']  # 首次启动指定的master节点

#elasticsearch.service
[Unit]
Description=elasticsearch
[Service]
User=es
LimitNOFILE=65535
ExecStart=/opt/elk/elasticsearch/bin/elasticsearch
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
[Install]
WantedBy=multi-user.target

#查看集群节点: curl -XGET 'http://127.0.0.1:9200/_cat/nodes?pretty'
#查询集群状态: curl -i XGET 'http://127.0.0.1:9200/_cluster/health?pretty'
```



Filebeat
``` bash
#filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /opt/logs/*/*.log
output.logstash:
  hosts: ["10.55.3.43:5044"]


#docker
docker run -idt \
  --name=filebeat \
  --user=root \
  --volume="/opt/logs:/opt/logs:ro" \
  --volume="/opt/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" \
  docker.elastic.co/beats/filebeat:7.9.1

```

Logstash
``` bash
#logstash.conf
input {
  beats {
    port => 5044
    client_inactivity_timeout => 3000
  }
}

filter {
  grok {
    match => ["message", "%{TIMESTAMP_ISO8601:logdate}\s+(?<content>.*)"]
  }
  ruby {
    code => "
      path = event.get('log')['file']['path']
      puts format('path = %<path>s', path: path)
      if (!path.nil?) && (!path.empty?)
        event.set('app', path.split('/')[-2])
      end
      hostname = event.get('host')['name']
      if (!hostname.nil?) && (!hostname.empty?)
        event.set('hostname', hostname)
      end
    "
  }
  date {
    match => ["logdate", "ISO8601"]
  }
}

output {
  elasticsearch {
    action => "index"
    index => "prod.java-%{+YYYY.MM.dd}"
    hosts => ["10.55.3.46:9200", "10.55.3.47:9200", "10.55.3.48:9200"]
  }
}

```

ElasticSearch
``` bash
#/etc/sysctl.conf
vm.max_map_count = 262144
sysctl -w vm.swappiness = 10


#elasticsearch.yml
node.name: node01
path.data: /opt/elasticsearch/data
path.logs: /opt/elasticsearch/logs
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
node.master: true
node.data: true

#Cluster
cluster.name: es-cluster01
discovery.zen.minimum_master_nodes: 2
discovery.zen.ping.unicast.hosts: ["brande-es01", "brande-es02", "brande-es03"]
#only initial master node
cluster.initial_master_nodes: ["node01"]

# Allow CORS
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-credentials: true


#swarm 部署
#manager节点运行以下操作

#创建overlay网络es, 在docker-compose.yml中外部调用，subnet网段需要小于ingress网段，不然会以ingress作为发现地址。
docker network create --driver overlay --subnet 10.0.6.0/24 es

#docker stack deploy es --compose-file=docker-compose.yml

#docker-compose.yml
version: '3'
services:
  elasticsearch1:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.1
    deploy:
      replicas: 1
      placement:
        constraints: [node.hostname == brande-es01]
    environment:
      - cluster.name=es
      - node.name=es1
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - discovery.zen.minimum_master_nodes=2
      - discovery.zen.fd.ping_timeout=120s
      - discovery.zen.fd.ping_retries=6
      - discovery.zen.fd.ping_interval=30s
      - cluster.initial_master_nodes=es1,es2,es3
      - "discovery.zen.ping.unicast.hosts=es_elasticsearch1,es_elasticsearch2,es_elasticsearch3"
      - "ES_JAVA_OPTS=-Xms2G -Xmx2G"
    volumes:
      - /opt/elasticsearch/data:/usr/share/elasticsearch/data
      - /opt/elasticsearch/logs:/usr/share/elasticsearch/logs
    ports:
      - 9200:9200
    networks:
      - es

  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.1
    deploy:
      replicas: 1
      placement:
        constraints: [node.hostname == brande-es02]
    environment:
      - cluster.name=es
      - node.name=es-2
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - discovery.zen.minimum_master_nodes=2
      - discovery.zen.fd.ping_timeout=120s
      - discovery.zen.fd.ping_retries=6
      - discovery.zen.fd.ping_interval=30s
      - cluster.initial_master_nodes=es1,es2,es3
      - "discovery.zen.ping.unicast.hosts=es_elasticsearch1,es_elasticsearch2,es_elasticsearch3"
      - "ES_JAVA_OPTS=-Xms2G -Xmx2G"
    volumes:
      - /opt/elasticsearch/data:/usr/share/elasticsearch/data
      - /opt/elasticsearch/logs:/usr/share/elasticsearch/logs
    ports:
      - 9201:9200
    networks:
      - es

  elasticsearch3:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.1
    deploy:
      replicas: 1
      placement:
        constraints: [node.hostname == brande-es03]
    environment:
      - cluster.name=es
      - node.name=es-3
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - discovery.zen.minimum_master_nodes=2
      - discovery.zen.fd.ping_timeout=120s
      - discovery.zen.fd.ping_retries=6
      - discovery.zen.fd.ping_interval=30s
      - cluster.initial_master_nodes=es1,es2,es3
      - "discovery.zen.ping.unicast.hosts=es_elasticsearch1,es_elasticsearch2,es_elasticsearch3"
      - "ES_JAVA_OPTS=-Xms2G -Xmx2G"
    volumes:
      - /opt/elasticsearch/data:/usr/share/elasticsearch/data
      - /opt/elasticsearch/logs:/usr/share/elasticsearch/logs
    ports:
      - 9202:9200
    networks:
      - es

networks:
  es:
    external:
      name: es
```


# UI
elasticsearch-head
``` bash
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install --registry=https://registry.npm.taobao.org
npm run start
curl http://localhost:9100/


```

cerebro
``` bash
#github
https://github.com/lmenezes/cerebro

#docker
docker run -idt --name cerebro -p 9000:9000 lmenezes/cerebro
```

ElasticHD
```
docker run -p 9800:9800 -d --name elastichd  containerize/elastichd
```

