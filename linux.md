---
title: linux
date: 2088-08-08 08:08:08
tags:
---
# 持久化systemd-journal日志
``` bash
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/jounal
systemctl restart systemd-journald
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
```
