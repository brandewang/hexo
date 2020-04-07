---
title: linux
date: 2088-08-08 08:08:08
tags:
---

# cpu
``` 
同一个进程同一时间段只能在一个cpu中运行，如果进程数小于cpu数，那么未使用的cpu将会空闲。
同一个线程同一时间段只能在一个cpu内核中运行，如果线程数小于cpu内核数，那么将有多余的内核空闲。
```

# 持久化systemd-journal日志
``` bash
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/jounal
systemctl restart systemd-journald
```

## 生成随机密码
``` bash
openssl rand -base64 10
openssl rand -base64 10|tr A-Z a-z
openssl rand -base64 32|tr A-Z a-z|cut -c 1-10
```

## 时间同步
``` bash
chronyc -a makestep
ntpdate 10.28.50.10
```

## 环境变量

``` bash
#mysql env
export PATH=/usr/local/mysql/bin:$PATH

#jdk env
export JAVA_HOME=/usr/java/jdk1.8.0_151
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

```

## 安装vim8.0

``` bash
# vim8.0通过filetype plugin 判断python文件解决制表符问题
# python文件编写优先使用空格
# 卸载老的vim
yum remove vim-* -y
# 下载第三方yum源
wget -P /etc/yum.repos.d/  https://copr.fedorainfracloud.org/coprs/lbiaggi/vim80-ligatures/repo/epel-7/lbiaggi-vim80-ligatures-epel-7.repo
# install vim
yum  install vim-enhanced sudo -y
# 验证vim版本
rpm -qa |grep vim
```


## 常用命令

``` bash
#历史操作格式化
cat >> /etc/profile <<EOF
export HISTTIMEFORMAT="[%F %T] [`whoami`] "
EOF

#丢包率
lostpk=`timeout 5  ping -q -s 500 202.96.209.133 -W 1000 -c 100 -A |grep transmitted|awk '{print $6}'`
#总耗时
rrt=`timeout 5  ping -q -s 500 202.96.209.133 -W 1000 -c 100 -A |grep transmitted|awk '{print $10}'`

#bash调试模式并加入时间戳，调试默认通道为stderr,加入IFS可以保留stdin的头尾空格
bash -x  test.sh 2>&1 |while IFS= read -r line; do echo "$(date +'%H:%M:%S') $line"; done

#获取TCP连接情况
netstat -n |awk '/^tcp/ {++S[$NF]} END {for(a in S) print a,S[a]}'
```


## 压力测试

``` bash
##stress
#内存压测
stress --vm 1 --vm-bytes 190M --vm-hang 0
stress --vm 2 --vm-bytes 300M --vm-keep #产生两个子进程 每个分配300M内存
#CPU压测
stress -c 2 # 消耗2个cpu资源

##fio
#Linux系统查看默认队列深度
lsscsi -l
#测试场景：
#100%随机，100%读， 4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=randread -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=rand_100read_4k

#100%随机，100%写， 4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=randwrite -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=rand_100write_4k

#100%顺序，100%读 ，4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=read -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=sqe_100read_4k

#100%顺序，100%写 ，4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=sqe_100write_4k

#100%随机，70%读，30%写 4K
fio -filename=/dev/emcpowerb -direct=1 -iodepth 1 -thread -rw=randrw -rwmixread=70 -ioengine=psync -bs=4k -size=1000G -numjobs=50 -runtime=180 -group_reporting -name=randrw_70read_4k
```

## ssh 

``` bash
#解决dsa类型的key认证
##加入校验类型
/etc/ssh/sshd_config
PubkeyAcceptedKeyTypes=+ssh-dss
##允许所有加密规则
/etc/sysconfig/sshd
CRYPTO_POLICY=

```
