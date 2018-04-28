---
title: elastic
date: 2018-04-29 10:32
tags:
---
# install elasticsearch cluster
``` bash
#导入elastic gpg-key
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
#添加repo
cat  > /etc/yum.repos.d/elasticsearch-6.repo <<EOF
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
#通过yum 安装 elasticsearch
yum install elasticsearch-6.2.4-1.noarch
#配置elastic环境变量
cat > /etc/profile.d/elastic_env.sh <<EOF
PATH=$PATH:/usr/share/elasticsearch/bin
export PATH
#配置memlock参数
cat > /etc/security/limits.d/10-elastic.conf <<EOF
* soft memlock unlimited
* hard memlock unlimited
#elasticsearch 配置文件
cat > /etc/elasticsearch/elasticsearch.yml <<EOF
cluster.name: {{ cluster_name }}
node.name: {{ ansible_hostname }}
node.master: true
node.data: true
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: {{ ansible_default_ipv4.address }}
http.port: 9200
transport.tcp.port: 9300
#扩展集群时使用最初集群配置即可
discovery.zen.ping.unicast.hosts: ["10.28.20.44", "10.28.20.45", "10.28.20.46"]
discovery.zen.minimum_master_nodes: 1
bootstrap.memory_lock: true
bootstrap.system_call_filter: false
http.cors.enabled: true
http.cors.allow-origin: "*"
EOF
```



# install elasticsearch-head
``` bash
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
curl http://localhost:9100/
```

# backup cluster data
``` bash
# 创建nfs共享存储  all_squash所有用户为匿名访问,anouid&gid=0 设置anonymous匿名用户访问id为0(root)
# 使所有用户拥有root读写权限，由于yum安装时es进程由elasticsearch启动，另集群用户id不一致.
cat > /etc/exports <<EOF
/tmp/nfs  10.28.20.* (rw,all_squash,anonuid=0,anongid=0)
EOF
# 在所有节点挂载nfs共享存储
mount -t nfs 10.28.20.44:/tmp/nfs /tmp/myback/
# 修改/etc/elasticsearch/elasticsearch.yml文件，添加path.repo配置,路径与nfs挂载本地路径一致
echo 'path.repo: ["/tmp/myback"]' >> 修改/etc/elasticsearch/elasticsearch.yml
# 通过http api接口创建仓库,其中myback为创建的仓库名
curl -XPUT  -H 'Content-type: application/json' -d '{ "type":"fs", "settings": {"location": "/tmp/myback"}}' http://10.28.20.46:9200/_snapshot/myback
# 通过http api接口备份索引数据,其中weather为索引名称，可同时备份多个，用逗号隔开。
# snapshot-brande为备份名
curl -XPUT  -H 'Content-type: application/json' -d '{ "indices": "weather"}' http://10.28.20.44:9200/_snapshot/myback/snapshot-brande
#查看备份内容,会显示对应的备份数据
ll /tmp/myback/
```

# restore cluster data
``` bash
# 迁移数据
将nfs共享存储中的备份数据挂载至目标服务器
# 创建仓库
与backup中创建方式一致
# 使用 http api进行恢复，如索引已存在，先关闭索引，恢复数据库再开启
curl -XPOST  http://10.28.20.46:9200/weather/_close
curl -XPOST  http://10.28.20.46:9200/_snapshot/myback/snapshot-brande/_restore
curl -XPOST  http://10.28.20.46:9200/weather/_open
#查看恢复状态
curl -XPOST  http://10.28.20.46:9200/_snapshot/myback/snapshot-brande/_status
```
