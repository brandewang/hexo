---
title: redis
date: 2088-08-08 08:08:08
tags:
---

# Redis

## 安装
``` bash
cd /opt
wget https://github.com/antirez/redis/archive/3.2.12.tar.gz
tar zxvf 3.2.12.tar.gz
cd redis-3.2.12
make && make install
mkdir /opt/redis6379
cp redis.conf /opt/redis6379/redis6379.conf
#redis6379.conf
daemonize yes
logfile /opt/redis6379/redis6379.log
dir /opt/redis6379
#启动master
redis-server /opt/redis6379/redis6379.conf
#启动slave
redis-server /opt/redis6380/redis.conf --slaveof 127.0.0.1 6379
redis-server /opt/redis6381/redis.conf --slaveof 127.0.0.1 6379
#sentinel哨兵监控主从状态  1或3台 配置文件一致
mkdir /opt/redis-sentinel
cp sentinel.conf /opt/redis-sentinel/sentinel.conf
#sentinel.conf
dir /opt/redis-sentinel/ #工作目录
port 26379 #运行端口
sentinel monitor mymaster 127.0.0.1 6379 1 #监控一个名为mymaster的主实例,最后为选举需要同意的人数
#启动sentinel
nohup redis-sentinel /opt/redis-sentinel/sentinel.conf  > /opt/redis-sentinel/sentinel.log 2>&1 &
```

## redis-cluster
``` bash
#reids安装
cd /opt
wget https://github.com/antirez/redis/archive/3.2.12.tar.gz
tar zxvf 3.2.12.tar.gz
cd redis-3.2.12
make && make install
mkdir -p /usr/local/redis-cluster/{redis01,redis02,redis03,redis04,redis05,redis06,redis07}
cat > /usr/local/redis-cluster/start_all.sh <<EOF
#!/bin/bash
cd /usr/local/redis-cluster/redis01 && redis-server redis.conf
cd /usr/local/redis-cluster/redis02 && redis-server redis.conf
cd /usr/local/redis-cluster/redis03 && redis-server redis.conf
cd /usr/local/redis-cluster/redis04 && redis-server redis.conf
cd /usr/local/redis-cluster/redis05 && redis-server redis.conf
cd /usr/local/redis-cluster/redis06 && redis-server redis.conf
EOF
cp /opt/redis-3.2.12/src/redis-trib.rb /usr/local/redis-cluster/
cp /opt/redis-3.2.12/redis.conf /usr/local/redis-cluster/redis01/redis.conf
#分别修改各节点redis.conf,注意port如果在同一主机则不能相同
daemonize yes
port 7001
cluster-config-file nodes.conf
cluster-node-timeout 15000
appendonly yes
#ruby安装
yum install -y ruby rubygems -y
cd /opt
wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz
tar zxvf ruby-2.3.1.tar.gz
cd ruby-2.3.1/ && ./configure --prefix=/usr/local/ruby-2.3.1
make && make install
rm -f /usr/bin/ruby
ln -s /usr/local/ruby-2.3.1/bin/ruby /usr/bin/ruby
gem install redis -v 3.2  #确认版本兼容 v4可能会有问题

#启动redis各节点
cd /usr/local/redis-cluster && bash start_all.sh
#创建集群 slave为1
./redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006
./redis-trib.rb check 127.0.0.1:7001 #确认集群状态
#添加集群节点
##复制配置文件 创建节点工作目录
cd /usr/local/redis-cluster && cp -r redis01 redis07
sed -i "s/7001/7007/g" redis07/redis.conf
#删除多余文件,以防发生意外  nodes.conf保存集群角色信息, rdb及aof为持久化文件
cd redis07 && rm -f nodes.conf dump.rdb appendonly.aof
#启动新节点
cd /usr/local/redis-cluster/redis07 && redis-server redis.conf
#将新节点加入集群 127.0.0.1:7001集群中任意节点都可以
./redis-trib.rb add-node 127.0.0.1:7007 127.0.0.1:7001
./redis-trib.rb check 127.0.0.1:7001 #确认集群状态
./redis-trib.rb reshard 127.0.0.1:7001 #重新分配集群分片 1.分片数量 2.接受节点 3.all或选择节点
#集群添加一个slave节点 127.0.0.1:7001集群中任意节点都可以
./redis-trib.rb add-node --slave --master-id d053c59ed2c9bdacbd31d3cd2777e4eca8e86512 127.0.0.1:7008 127.0.0.1:7001

#删除集群节点,如果是slave节点可以直接删除,master节点需将切片reshard移动到其他master节点后才能删除
/redis-trib.rb del-node 127.0.0.1:7001 d053c59ed2c9bdacbd31d3cd2777e4eca8e86512
```

## 常用命令
``` bash
#选择数据库
redis-cli -n number
redis-cli select number
#获取键类型
redis-cli type key
#查找键
redis-cli keys name*
#查看TTL
redis-cli ttl key
#删除键
redis-cli del key
redis-cli keys name*|xargs redis-cli del
# 获取所有db的大键
host=tms-redis-prod.nyegwm.ng.0001.cnn1.cache.amazonaws.com.cn
dbs=`redis-cli -h $host info keyspace| sed -r 's/db([0-9]+):.*/\1/g'|grep -v space|awk 'BEGIN{ ORS=" "}1'`
for db in $dbs
do
  echo "---------------BEGIN DB:$db-----------------"
  redis-cli -h $host -n $db --bigkeys
  echo -e "----------------END-------------------------\n\n"
done


```
