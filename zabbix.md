---
title: zabbix
date: 2088-08-08 08:08:08
tags:
---

## url
``` bash
#template,module & more
https://share.zabbix.com/
#doc
https://www.zabbix.com/documentation/4.0/zh/manual
#download
https://www.zabbix.com/cn/download_agents
```

## install

``` bash
# mysql-community 5.7.29
rpm -Uvh https://repo.mysql.com//mysql80-community-release-el7-3.noarch.rpm
##编辑repo文件安装5.7版本

yum install mysql-community-server
create database zabbix character set utf8 collate utf8_bin;
##数据库优化 /etc/my.cnf 


# zabbix-server-mysql 4.0.19 (由于需要调用各种环境或工具建议不要使用容器)
rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
yum -y install zabbix-server-mysql
#初始化表结构 
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix


## zabbix_server.conf 相关参数
DBName=zabbix
DBUser=zabbix
DBPassword=

# zabbix-web-nginx-mysql 4.0.19(docker)
docker run --name zabbix-web \
	-e ZBX_SERVER_HOST="10.28.50.8" \
	-e DB_SERVER_HOST="10.28.50.8" \
	-e MYSQL_USER="zabbix" \
	-e MYSQL_PASSWORD="zabbix" \
	-e PHP_TZ="Asia/Shanghai" \
	-p 80:80 -p 443:443 \
	-d zabbix/zabbix-web-nginx-mysql:alpine-4.0.19


# zabbix-proxy-mysql 4.0.14(docker)
## 每个proxy需单独配置数据库
## zabbix-proxy.conf 相关参数
ProxyLocalBuffer=0                           #当数据发送到Server，还要在本地保留多少小时.不保留
ProxyOfflineBuffer=3                         #当数据没有发送到Server，在本地保留多少小时，3小时。
HeartbeatFrequency=60                        #心跳检测代理在Server的可用性
ConfigFrequency=60                         #代理多久从Server获取一次配置变化，默认3600秒.
DataSenderFrequency=1                        #这个是proxy端向server端发送数据的时间，单位是秒

PS:ConfigFrequency会直接影响proxy从server拉取配置的间隔,在未拉取最新配置时,agentd会发生 no active checks on server

docker run --name zabbix-proxy \
	-e DB_SERVER_HOST="10.28.50.9" \
	-e MYSQL_USER="zabbix_proxy" \
	-e MYSQL_PASSWORD="zabbix_proxy" \
	-e ZBX_HOSTNAME="zabbix-proxy01" \
	-e ZBX_SERVER_HOST="10.28.50.8" \
	-e ZBX_CONFIGFREQUENCY="60" \
	-p 10051:10051 \
	-d zabbix/zabbix-proxy-mysql:alpine-4.0.14

# zabbix-agent 4.0.14(由于需要调用各种环境或工具建议不要使用容器)
rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
yum install zabbix-agent
## zabbix_agentd.conf 相关参数
PS:连接zabbix-proxy的客户端请设置Server及ServerActive为代理IP
Server=
ServerActive=
HostnameItem=system.hostname
HostMetadataItem=system.uname
Timeout=6


# zabbix使用配置顺序
1.配置media(告警脚本)
2.配置用户(配置完成后先登陆一次,否则影响action triggers的触发(BUG))
3.配置模版
4.配置主机组
5.配置action auto registration(主机自动注册)
6.配置action triggers(告警脚本)
7.配置low-level discovery(自定义发现脚本)
8.配置dashboard自定义(数值统计)
9.配置必要宏macros(如snmp_community)
10.配置map(分组服务器概况,网络设备概况及拓扑及端口流量)
11.配置screen组合graph及map
12.配置grafana可视化界面

```

## optimize

``` bash
#server配置优化
1.参照web中zabbix-server图形监控项目及触发器调整到最优
#调整zabbix_server进程数量
StartPollers=80
StartPingers=10
StartPollersUnreachable=80
StartIPMIPollers=10
StartTrappers=20
StartDBSyncers=6
#调整内存大小
VMwareCacheSize=64M
CacheSize=32M
HistoryCacheSize=256M
TrendCacheSize=64M
HistoryIndexCacheSize = 32M
ValueCacheSize=64M

#架构优化
1.item设置为主动模式,缓解zabbix-server压力
2.加入proxy代理(主动模式),缓解zabbix-server压力

#数据库优化
1.my.cnf参数优化
2.分库分表
#调整数据库innodb
innodb_file_per_table = 1
innodb_buffer_pool_size=<large> (~75% of total RAM)
innodb_buffer_pool_instances = 4 (MySQL 5.6 - 8)
innodb_flush_log_at_trx_commit = 2
innodb_flush_method = O_DIRECT
innodb_log_file_size = 256M


#数据库空间计算
1.历史数据 (每条历史数据需要占用大概50个字节(Bytes))
假如历史数据你要保留90天，有3000个监控项，监控间隔60秒，(即每秒处理数据量=3000/60=50个)
3000/60 *3600 *24 *90 *50字节=18GB

2.趋势数据 (趋势数据每条记录数据大约占用128字节(Bytes))
假如有3000个监控项（即会产生3000条/h趋势数据），想保留1年的趋势数据
3000 *24 *365 *128字节=3GB

3.事件数据(事件大概占用130字节(Bytes))
假如，平均1秒钟产生一条事件，想要保存事件数据1年
3600 *24 *365 *130字节 = 3.8GB

##数据库硬盘空间大小
数据库硬盘空间  = 配置文件大小 + 历史数据大小 + 趋势记录大小 + 事件记录大小
关于配置文件大小(Zabbix配置)，很小，基本可以忽略不记。

```

## Discovery
``` bash
web->configuration->Discovery
主要通过配置ip range和check响应方式发现主机

1.snmp发现设备(idrac,ilo,switch,route)
通过配置community团体名称来区分,如idrac8,ilo2,cisco.以此来区分check类型,在action中应用不同的check类型来link不同的模版

2.zabbix_agent
主要用来发现系统

```

## Media
``` bash
Name: wechat
Type: script
Script name: wechat.py
Script parameters: {ALERT.SENDTO}
				   {ALERT.SUBJECT}
				   {ALERT.MESSAGE}
```

## Action
``` bash
#Trigger
step 1-10 600s
step 11-0 1h

##Default subject
{TRIGGER.STATUS}: {TRIGGER.NAME}
##Default message
异常主机名: {HOST.NAME}
异常主机地址: {HOST.IP}
异常级别: {TRIGGER.SEVERITY}
异常时间: {EVENT.DATE} {EVENT.TIME} 

异常内容:
now value: {{HOST.HOST}:{ITEM.KEY}.last()}
MIN for 10 minutes: {{HOST.HOST}:{ITEM.KEY}.min(900)}

事件ID: {EVENT.ID}

##Recovery subject
{TRIGGER.STATUS}: {TRIGGER.NAME}

##Recovery message
主机名: {HOST.NAME}
主机地址: {HOST.IP}
异常时间: {EVENT.DATE} {EVENT.TIME} 
恢复时间: {EVENT.RECOVERY.DATE} {EVENT.RECOVERY.TIME}

Latest value: {{HOST.HOST}:{ITEM.KEY}.last()}
MAX for 10 minutes: {{HOST.HOST}:{ITEM.KEY}.max(600)}
MIN for 10 minutes: {{HOST.HOST}:{ITEM.KEY}.min(600)}

事件ID: {EVENT.ID}
```


## IPMI

``` bash
# 安装ipmitool
yum install -y ipmitool
# 获取目标传感器数据, 这里10.28.200.10为(idrac8 ipmi), 需在web页面idrac设置->网络设置->ipmi 中勾选lan
ipmitool -I lanplus -H 10.28.200.10 -U root -P fruit@123 sdr list
ipmitool -I lanplus -H 10.28.200.10 -U root -P fruit@123 sdr get 'Temp'

# 设置zabbix item
# 如Sensor ID(通过ipmitool获取)不重复,则使用以下格式
Type:IPMI agent
IPMI sensor: Temp
# 如Sensor ID重复,则设置key:value格式,完整的name可从"grep 'Added sensor' /var/log/zabbix/zabbix_server.log"中获取
Type:IPMI agent
IPMI sensor: name:(3.2).Temp
```

