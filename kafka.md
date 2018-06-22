---
title: kafka
date: 2088-08-08 08:08:08
tags:
---

# kafka-manager ui安装
``` bash

./kafka-topics.sh --list -zookeeper localhost:2181
1.安装sbt
curl https://bintray.com/sbt/rpm/rpm > bintray-sbt-rpm.repo
mv bintray-sbt-rpm.repo /etc/yum.repos.d/
yum install sbt
2.下载编译
git clone https://github.com/yahoo/kafka-manager.git
cd kafka-manager
#写入ali私服
cat > .sbt <<EOF
[repositories]
local
aliyun: http://maven.aliyun.com/nexus/content/groups/public
typesafe: http://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
EOF
sbt clean dist
3.命令执行完成后，在 target/universal 目录中会生产一个zip压缩包kafka-manager-1.3.3.7.zip。将压缩包拷贝到要部署的目录下解压。
4.修改配置文件
#conf/application.conf
kafka-manager.zkhosts="localhost:2181"
5.启动
bin/kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port=8080
```

# 常用命令
``` bash
#列出topics
./kafka-topics.sh --list -zookeeper localhost:2181
#查看topic 详情
./kafka-topics.sh --describe --zookeeper localhost:2181 --topic brande.nginx.access
#模拟生成消费者
/kafka-console-consumer.sh --zookeeper localhost:2181 --topic brande.nginx.access --from-beginning

```
