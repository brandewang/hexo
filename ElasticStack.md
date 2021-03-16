---
title: ElasticStack
date: 2088-08-08 08:08:08
tags:
---

- url: elastic.co

# ElasticSearch

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

# Kibana
```
mkdir -p /opt/elk
cd /opt/elk
tar zxvf kibana-7.11.1-linux-x86_64.tar.gz
mv kibana-7.11.1-linux-x86_64 kibana

#config/kibana.yml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://10.55.3.71:9200"]
i18n.locale: "zh-CN"

#kibana.service
[Unit]
Description=kibana

[Service]
ExecStart=/opt/elk/kibana/bin/kibana --allow-root
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

# Filebeat
部署
``` 
#filebeat.yml
filebeat.inputs:
- type: log  #日志类型
  enabled: true  #是否启用
  paths:
    - /opt/logs/nginx/access.log #日志路径可同配
  fields_under_root: true  #是否将自定义fields直接加入到顶层
  fields:  #配置环境，项目，服务名，用于规划索引名称
    env: prd
    project: ops
    service: nginx01
  tags: ["proxy", "nginx", "access_log"]  #加入日志类型，用于匹配同日志类型字段

- type: log     #多类型输入配置
  enabled: true
  paths:
    - /opt/logs/helloworld01/*.log
  fileds_under_root: true
  fields:
    env: prd
    project: ms
    service: helloworld01
  tags: ["java", "springboot", "app_log"]
#java堆栈日志多行匹配
	multiline.pattern: '^\s'
	multiline.negate: false
	multiline.match: after

- type: log
  enabled: true
  paths:
    - /opt/logs/helloworld02/*.log
  fields_under_root: true
  fields:
    env: prd
    project: ms
    service: helloworld02
  tags: ["java", "springboot", "app_log"]

#推送到Logstash
output.logstash:
  hosts: ["10.55.3.70:5044"]
#推送到ES
setup.ilm.enabled: false
setup.template.name: "microservice-product"
setup.template.pattern: "microservice-product-*"
output.elasticsearch:
  hosts: ["10.55.3.71:9200"]
  index: "microservice-product-%{+yyy.MM.dd}"

常见的Java堆栈日志:
Exception in thread "main" java.lang.NullPointerException
  at com.example.myproject.Book.getTitle(Book.java:16)
	at com.example.myproject.Author.BookTitles(Author.java:25)
	at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
	
#filebeat.service
[Unit]
Description=FileBeat

[Service]
ExecStart=/opt/elk/filebeat/filebeat -c /opt/elk/filebeat/filebeat.yml
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target


#docker
docker run -idt \
  --name=filebeat \
  --user=root \
  --volume="/opt/logs:/opt/logs:ro" \
  --volume="/opt/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" \
  docker.elastic.co/beats/filebeat:7.9.1

```

# Logstash
二进制部署
``` 
yum install java-1.8.0-openjdk -y
cd /opt/elk
tar zxvf logstash-7.11.1.tar.gz
mv logstash-7.11.1.tar.gz logstash

#logstash.service
[Unit]
Description=logstach

[Service]
ExecStart=/opt/elk/logstash/bin/logstash
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target

#logstash.conf
pipeline: # 管道配置
  batch:
    size: 125
    delay: 5
path.config: /opt/elk/logstash/conf.d #logstash -e模式下不允许使用该参数
#定期检查配置是否修改，并重新加载管道。也可以使用SIGHUP信号手动触发
#config.reload.automatic: false
#config.reload.interval: 3s
#http.enabled: true
http.host: 0.0.0.0
http.port: 9600-9700
log.level: info
path.logs: /opt/elk/logstash/logs

#debug
/opt/elk/logstash/bin/logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
```

input
```
#文件
input{
  file {
    path => "/var/log/test/*.log"
    exclude => "error.log"
    tags => "nginx"
    tags => "web"
    type => "access"
    add_field => {
        "project" => "microservice"
        "app" => "product"
    }
  }
}

#beats
input {
  beats {
	  host => "0.0.0.0"
		port => 5044
	}
}
```

filter
```
#JSON
JSON插件: 接收一个json数据,将其展开为Logstash事件中的数据结构,放到事件顶层。
常用字段:
- source 指定要解析的字段，一般是原始消息message字段
- target 将解析的结果放到指定字段, 如果不指定，默认在事件顶层

模拟数据:'{"remote_addr": "192.168.1.11","url":"/index","status":"200"}'
filter {
  json {
	  source => "message"
	}
}

将Nginx访问日志格式改为JSON:
log_format json  '{"@timestamp":"$time_iso8601", "remote_addr": "$remote_addr", "body_bytes_sent": "$body_bytes_sent",
"request_time": "$request_time", "status": "$status", "request_uri": "$request_uri", "request_method": "$request_method",
"http_referrer": "$http_referer", "http_x_forwarded_for": "$http_x_forwarded_for", "http_user_agent": "$http_user_agent"}';


#KV
KV插件: 接收一个键值数据，按照指定分隔符解析为Logstash事件中的数据结构，放到事件顶层.

模拟数据:'www.github.com?id=1&name=brande&age=32'
filter {
  kv {
    field_split => "&?"
  }
}

#Grok
Grok插件: 如果采集的日志格式是非结构化的，可以些正则表达式提取，grok是正则表达式支持的实现。
常用字段:
- match 正则匹配模式
- patterns_dir 自定义正则模式文件
Logstash内置的正则匹配模式，在安装目录下可以看到，路径：
vendor/bundle/jruby/2.5.0/gems/logstash-patterns-core-4.1.2/patterns/grok-patterns

正则匹配模式语法格式: %{SYNTAX:SEMANTIC}
- SYNTAX 模式名称，模式文件中的第一列
- SEMANTIC 匹配文件的字段名
例如: %{IP:client}

模拟数据: 192.168.1.10 GET /index.html 15824 0.043

示例: 正则匹配HTTP请求日志

filter {
  grok {
	  match => {
		  "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"
		}
	}
}

多模式匹配,写多个正则表达式,只要满足其中一条就能匹配成功
filter {
  grok {
	  match => [
		  "message" , "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" %{CID:cid},
		  "message" , "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" %{CID:cid} %{TAG:tag}
		]
	}
}

#GeoIP
GeoIP插件: 根据Maxmind GeoLite2数据库中的数据添加有关IP地址位置信息
常用字段:
- source 指定要解析的IP字段,结果保存到geoip字段
- database GeoLite2数据库文件的路径
- fields 保留极细的指定字段
下载地址: https://www.maxmind.com/en/accounts/436070/geoip/downloads(需要登录)

示例1:
filter{
  grok {
	  patterns_dir => "/opt/patterns"
		match => {
		  "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" 
		}
	}
	geoip {
	  source => "client"
		database => "/opt/GeoLite2-City.mmdb"
	}
}

示例2: 保留解析的指定字段
filter{
  grok {
	  patterns_dir => "/opt/patterns"
		match => {
		  "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"
		}
	}
	geoip {
	  source => "client"
		database => "/opt/GeoLite2-City.mmdb"
		target => "geoip"
		fields => ["city_name", "country_code2", "country_name", "region_name"]
	}
}
```

Output
```
#elasticsearch
output {
  elastichsearch {
	  hosts => ["192.168.3.71:9200", "192.168.3.72:9200", "192.168.3.73:9200"]
		index => "microservice-product-%{+YYYY.MM.dd}"
	}
}
```

# UI

elasticsearch-head
``` 
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install --registry=https://registry.npm.taobao.org
npm run start
curl http://localhost:9100/
```

cerebro
```
#github
https://github.com/lmenezes/cerebro

#docker
docker run -idt --name cerebro -p 9000:9000 lmenezes/cerebro
```

ElasticHD
```
docker run -p 9800:9800 -d --name elastichd  containerize/elastichd
```

# 日志收集方案
- 应用日志收集
- k8s容器应用日志收集
- K8S标准输出日志收集

应用日志收集
```
#filebeat
#nginx日志收集
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /opt/logs/nginx/access.log
  fields_under_root: true
  fields:
    env: dev
    project: ops
    service: nginx01
  tags: ["proxy", "nginx", "access_log"]
output.logstash:
  hosts: ["10.55.3.70:5044"]

#springboot日志收集
filebeat.inputs
- type: log
  enabled: true
  paths:
    - /opt/logs/helloworld01/*.log
  fields_under_root: true
  fields:
    env: prd
    project: ms
    service: helloworld01
  tags: ["java", "springboot", "app_log"]
  multiline.pattern: '^\s'
  multiline.negate: false
  multiline.match: after
output.logstash:
  hosts: ["10.55.3.70:5044"]

#logstash
#fileds获取字段用于生成索引
#tags获取字段用于匹配同类型过滤方案
input{
  beats {
    host => "0.0.0.0"
    port => 5044
  }
}
filter {
  if [env] in ["prd", "test", "dev"] and [project] and [service] {
    mutate {
      add_field => {
          "[@metadata][target_index]" => "%{[env]}-%{[project]}-%{[service]}-%{+YYYY.MM.dd}"
      }
    }
  } else {
    mutate {
      add_field => {
          "[@metadata][target_index]" => "unknown-%{+YYYY.MM.dd}"
      }
    }
  }
  if "springboot" in [tags] and "app_log" in [tags] {
    grok {
      match => {
        "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"
      }
    }
  }
  if "nginx" in [tags] and "access_log" in [tags] {
    json {
      source => "message"
    }
  }
}
output{
  elasticsearch {
    hosts => ["10.55.3.71:9200", "10.55.3.72:9200", "10.55.3.73:9200"]
    index => "%{[@metadata][target_index]}"
  }
}
```

# 扩容方案
- filebeat > logstash > elasticsearch
- filebeat > redis单机 > logstash > elasticsearch
- filebeat > redis主从(sentinel + keepalived) > logstash(多台) > elasticsearch
- filebeat > kafka集群 > logstash(多台) > elasticsearch

ruby示例
```
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

# ElasticSearch Swarm
``` 
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

