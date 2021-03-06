---
title: linux
date: 2088-08-08 08:08:08
tags:
---

# 调优
```
#内核参数优化
cat >>/etc/sysctl.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1  #将桥接流量传递到iptables的链
net.bridge.bridge-nf-call-iptables = 1 #将桥接流量传递到iptables的链
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
#kernel.shmmax = 4294967285   # 单位字节，可用最大共享内存段，4G 大小一般为物理内存-1byte
#kernel.shmmni = 4096     # 最大共享内存段数量，为默认值不必修改
#kernel.shmall = 1048575  # 单位页, 可用最大共享内存分配页, 不小于 kernel.shmmax / getconf PAGE_SIZE
net.core.somaxconn = 10240
net.core.netdev_max_backlog = 250000
#net.core.rmem_default = 8388608  被net.ipv4.tcp_rmem[1] 覆盖
#net.core.wmem_default = 8388608  被net.ipv4.tcp_wmem[1] 覆盖
#net.core.rmem_max = 16777216     被net.ipv4.tcp_rmem[2] 覆盖
#net.core.wmem_max = 16777216     被net.ipv4.tcp_wmem[2] 覆盖
net.ipv4.tcp_rmem = 16384 1048576 12582912
net.ipv4.tcp_wmem = 16384 1048576 12582912
net.ipv4.tcp_mem = 1541646 2055528 3083292
net.ipv4.tcp_max_syn_backlog = 10240
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_syncookies = 1
#net.ipv4.tcp_tw_recycle = 0     内核4.12后已弃用，用nat访问的客户端，容易引起服务端包混乱
net.ipv4.tcp_tw_resue = 1
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_moderate_rcvbuf = 1
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_retries1 = 2
net.ipv4.tcp_retries2 = 3
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 2
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
vm.swappiness = 0
fs.file-max = 204800
EOF
sysctl -p

#IO调度优化
Anticipatory I/O scheduler 适用于大多数环境,特别是写入较多的环境(比如文件服务器)Web,App等应用,但不太合适数据库应用，
Deadline I/O scheduler 通常与Anticipatory相当,但更简洁小巧,更适合于数据库应用
CFQ I/O scheduler 为所有进程分配等量的带宽,适合于桌面多任务及多媒体应用，适用于有大量进程的多用户系统，默认IO调度器
NOOP I/O scheduler NOOP对于闪存设备,RAM,嵌入式系统是最好的选择.其应用环境主要有以下两种：一是物理设备包含自己的I/O调度程序，比如SCSI的TCQ；二是寻道时间可以忽略不计的设备，比如SSD等。

#临时设置
cat /sys/block/sda/queue/scheduler
echo deadline > /sys/block/sda/queue/scheduler

#永久设置
/etc/default/grub
GRUB_CMDLINE_LINUX="elevator=deadline"

grub-mkconfig -o /boot/grub/grub.cfg
```

# shell
```
#登录shell
1:/etc/profile
2:/etc/profile.d/*.sh
3:$HOME/.bash_profile 会加载$HOME/.bashrc和/etc/bashrc
4:$HOME/.bash_login
5:$HOME/.profile

#交互式非登录shell
1.$HOME/.bashrc
2./etc/bashrc  会加载/etc/profile.d/*.sh


#bash
#打印调试信息
set -x

#遇错退出
set -e

#sed
#\1代表第一个括号匹配
echo 'abcabcabc' | sed 's/\(ab\)c/\1/'

```

# nfs

```
开机自动挂载
vi /etc/fstab
10.55.5.12:/volume1/nfs-bak01 /opt/bak  nfs     rw,soft,intr,timeo=30,retry=3,_netdev   0 0
```

# network

```
#centos8
#网卡启停
systemctl restart network
nmcli connection reload 
nmcli connection down eth0 
nmcli connection up eth0 
```


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

## kill信号

``` bash
#强制关闭
kill -9 $PID

#重载配置
kill -HUP $PID

#重读配置文件
kill -USR1 $PID

#重载程序
kill -USR2 $PID
```


## 日志切割

``` bash
#尽量使用logrotate,避免重复造轮

compress				#通过gzip压缩转储以后的日志
nocompress				#不压缩
copytruncate			#用于还在打开中的日志文件，把当前日志备份并截断
nocopytruncate			#备份日志文件但是不截断
create mode owner group #转储文件，使用指定的文件模式创建新的日志文件
nocreate                #不建立新的日志文件
delaycompress           #和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
nodelaycompress         #覆盖 delaycompress 选项，转储同时压缩。
errors address          #专储时的错误信息发送到指定的Email 地址
ifempty                 #即使是空文件也转储，这个是 logrotate 的缺省选项。
notifempty              #如果是空文件的话，不转储
mail address            #把转储的日志文件发送到指定的E-mail 地址
nomail                  #转储时不发送日志文件
olddir directory        #转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
noolddir                #转储后的日志文件和当前日志文件放在同一个目录下
prerotate/endscript     #在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
daily                   #指定转储周期为每天
weekly                  #指定转储周期为每周
monthly                 #指定转储周期为每月
rotate count            #指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
tabooext [+] list		#让logrotate不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave, v, 和 ~
size size               #当日志文件到达指定的大小时才转储，bytes(缺省)及KB(sizek)或MB(sizem)
missingok				#在日志轮循期间，任何错误将被忽略，例如“文件无法找到”之类的错误。

#vim /etc/logrotate.d/nginx
/opt/logs/nginx/*.log {
	daily
	rotate 5
	missingok
	notifempty
	sharedscripts
	postrotate
	    if [ -f /opt/run/nginx.pid ]; then
	        kill -USR1 `cat /opt/run/nginx.pid`
	    fi
	endscript
}

#vim /etc/logrotated.d/services
/opt/logs/*/*.log {
    daily
    rotate 5
    missingok
    notifempty
    sharedscripts
    postrotate
		cat /opt/run/*.pid|xargs -I {} kill -USR1 {}
    endscript
}


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


#iperf
#10.55.3.52 启动监听程序
iperf -s
#10.55.3.51 运行测试命令 间隔1秒反馈 连续检测5秒
iperf -c 10.55.3.52 -i 1 -t 5
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


# iptables
``` bash
#清空规则
iptables -F
iptables -X
iptables -Z
iptables -L -n

iptables -F -t nat
iptables -X -t nat
iptables -Z -t nat
iptables -L -n -t nat

```
# gre tunnel
|vpc|外网地址|内网地址|隧道互联地址|
|--|--|--|--|
|杭州|121.196.208.175|10.55.5.100|192.168.1.1|
|北京|60.205.187.39|10.55.6.100|192.168.1.1|
``` bash
#杭州
modprobe  ip_gre
ip tunnel add tun1-bj mode gre remote 60.205.187.39 local 121.196.208.175
ip link set tun1-bj up
ipaddr add 192.168.1.1 peer 192.168.1.2 dev tun1-bj
route add -net 10.55.6.0/24 tun1-bj

#北京
modprobe  ip_gre
ip tunnel add tun1-hz mode gre remote 121.196.208.175 local 60.205.187.39
ip link set tun1-hz up
ipaddr add 192.168.1.2 peer 192.168.1.1 dev tun1-hz
route add -net 10.55.5.0/24 tun1-hz

```

# cfssl

```
# ca-csr.json
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "Shanghai",
      "ST": "Shanghai",
      "O": "GIHTG",
      "OU": "ops"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}

# ca-config.json
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "server": {
        "expiry": "876000h",
        "usages": [
          "signing",
          "key encipherment",
          "server auth"
        ]
      },
      "client": {
        "expiry": "876000h",
        "usages": [
          "signing",
          "key encipherment",
          "client auth"
        ]
      }
    }
  }
}

# kub-apiserver-csr.json
{
  "CN": "kube-apiserver",
  "hosts": [
  "127.0.0.1",
    ],
  "key": {
    "algo": "rsa",
    "size": 2048
  }
}


#初始化ca证书
cfssl gencert -initca ca-csr.json |cfssljson -bare ca -
#CA签发证书
gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server kube-apiserver-csr.json |cfssljson -bare kube-apiserver
#续签ca证书, ca-csr.json和原始csr保持不变
cfssl gencert -renewca -ca ca.pem -ca-key ca-key.pem
```

# openssl

```
#自签证书
openssl genrsa -out dashboard.key 2048
openssl req -days 36000 -new -out dashboard.csr -key dashboard.key -subj '/CN=k8s-dashboard.corp.gihtg.com'
openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=whoami.gihtg.com"

#验证证书和私钥是否匹配
diff -eq <(openssl x509 -pubkey -noout -in cert.crt) <(openssl rsa -pubout -in cert.key)

#验证该证书是否ca签发
cert=/etc/kubernetes/pki/apiserver.crt
cacert=/etc/kubernetes/pki/ca.crt
expr $(openssl x509 -in ${cacert} -hash -noout) = $(openssl x509 -in ${cert} -issuer_hash -noout)
### 0 为 false， 1 为 true

#openssl 查看证书细节
- 打印证书的过期时间
openssl x509 -in signed.crt -noout -dates
- 打印出证书的内容：
openssl x509 -in cert.pem -noout -text
- 打印出证书的系列号
openssl x509 -in cert.pem -noout -serial
- 打印出证书的拥有者名字
openssl x509 -in cert.pem -noout -subject
- 以RFC2253规定的格式打印出证书的拥有者名字
openssl x509 -in cert.pem -noout -subject -nameopt RFC2253
- 在支持UTF8的终端一行过打印出证书的拥有者名字
openssl x509 -in cert.pem -noout -subject -nameopt oneline -nameopt -escmsb
- 打印出证书的MD5特征参数
openssl x509 -in cert.pem -noout -fingerprint
- 打印出证书的SHA特征参数
openssl x509 -sha1 -in cert.pem -noout -fingerprint
- 把PEM格式的证书转化成DER格式
openssl x509 -in cert.pem -inform PEM -out cert.der -outform DER
- 把一个证书转化成CSR
openssl x509 -x509toreq -in cert.pem -out req.pem -signkey key.pem
- 给一个CSR进行处理，颁发字签名证书，增加CA扩展项
openssl x509 -req -in careq.pem -extfile openssl.cnf -extensions v3_ca -signkey key.pem -out cacert.pem
- 给一个CSR签名，增加用户证书扩展项
openssl x509 -req -in req.pem -extfile openssl.cnf -extensions v3_usr -CA cacert.pem -CAkey key.pem -CAcreateserial
- 查看csr文件细节：
openssl req -in my.csr -noout -text
```
