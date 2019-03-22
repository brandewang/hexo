---
title: redis
date: 2088-08-08 08:08:08
tags:
---

# Redis

``` bash
#选择数据库
redis-cli -n number
redis-cli select number
#获取键类型
redis-cli type key
#查找键
redis-cli keys name*
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
