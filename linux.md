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

## 内核升级

``` bash
1，载入公钥

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

2，安装ELRepo
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm


3，添加repository 后, 列出可以使用的kernel包版本
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available

4、安装需要的kernel版本，这里安装 kernel-lt
yum --enablerepo=elrepo-kernel install kernel-lt

内核版本介绍：
lt:longterm的缩写：长期维护版；
ml:mainline的缩写：最新稳定版；

5，查看内核的启动顺序
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg

6，设置启动顺序
默认启动的顺序是从0开始，新内核是从头插入（目前位置在0，而4.4.4的是在1），所以需要选择0。
grub2-set-default 0
#查看内核启动项
grub2-editenv list

7，卸载老版本kernel内核工具
rpm -qa|grep kernel|grep 3.10
rpm -qa|grep kernel|grep 3.10|xargs yum remove -y
备注：有一个正在运行的kernel3.10卸载不了，因为正在运行中，重启之后可卸载。

8，安装新版的工具包
yum --enablerepo=elrepo-kernel install -y kernel-lt-tools
检查：
rpm -qa|grep kernel

9，重启并检查版本
reboot
uname -a
```
