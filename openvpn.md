---
title: openvpn
date: 2088-08-08 08:08:08
tags:
---

# install

``` bash
#安装yum仓库
yum install epel-release

#安装openvpn
yum install openvpn

#从github下载easyrsa
cd /etc/openvpn
wget https://github.com/OpenVPN/easy-rsa/archive/v3.0.6.zip
unzip v3.0.6.zip
mv easy-rsa-3.0.6 easy-rsa

#配置easyrsa vars
cd /etc/openvpn/easy-rsa/easyrsa3
cp vars.example vars

vim vars

set_var EASYRSA_REQ_COUNTRY     "CN"
set_var EASYRSA_REQ_PROVINCE    "SHANGHAI"
set_var EASYRSA_REQ_CITY        "SHANGHAI"
set_var EASYRSA_REQ_ORG         "GIHTG"
set_var EASYRSA_REQ_EMAIL       "ops@gihtg.com"
set_var EASYRSA_REQ_OU          "OU_IT"
#set_var EASYRSA_KEY_SIZE       2048
#set_var EASYRSA_ALGO           rsa


#创建服务端证书及key
1.初始化
cd /etc/openvpn/easy-rsa/easyrsa3/
./easyrsa init-pki

2.创建根证书
./easyrsa build-ca

3.创建服务器端证书
./easyrsa gen-req server nopass

4.签约服务器端证书
./easyrsa sign server server

5.创建Diffie-Hellman，确保key穿越不安全网络的命令
./easyrsa gen-dh

6.创建客户端key及生成证书
./easyrsa gen-req brande
./easyrsa sign client brande

7.吊销客户端证书
./easyrsa revoke brande
./easyrsa gen-crl


#openvpn server 配置
vim /etc/openvpn/server/server.conf
local 0.0.0.0     #监听地址
port 1194     #监听端口
proto tcp     #监听协议
dev tun     #采用路由隧道模式
ca /etc/openvpn/server/ca.crt      #ca证书路径
cert /etc/openvpn/server/server.crt       #服务器证书
key /etc/openvpn/server/server.key  #服务器秘钥
dh /etc/openvpn/server/dh.pem     #密钥交换协议文件
crl-verify /etc/openvpn/easy-rsa/easyrsa3/pki/crl.pem   #证书吊销列表
server 10.8.0.0 255.255.255.0    #给客户端分配地址池,不能和内网地址冲突
route 10.8.1.0 255.255.255.0     #添加路由，用于手动分配地址段
route 10.8.2.0 255.255.255.0
ifconfig-pool-persist ipp.txt
;push "redirect-gateway def1 bypass-dhcp"      #修改客户端默认网关
push "route 10.55.2.0 255.255.255.0"    #手动添加路由
push "route 10.55.3.0 255.255.255.0"
push "route 10.55.4.0 255.255.255.0"
push "route 10.55.5.0 255.255.255.0"
push "dhcp-option DNS 10.55.5.1"        #dhcp分配dns
client-config-dir ccd   #配置自定义用户信息
client-to-client       #允许客户端之间互相通信
;duplicate-cn   #允许同一个用户同时登陆
keepalive 10 120       #存活时间，10秒ping一次,120 如未收到响应则视为断线
comp-lzo      #传输数据压缩
max-clients 100     #最多允许 100 客户端连接
user openvpn       #用户
group openvpn      #用户组
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
log         /var/log/openvpn/openvpn.log
verb 3
#用户名密码登陆
script-security 3
auth-user-pass-verify /etc/openvpn/server/checkpsw.sh via-env    #指定用户认证脚本
username-as-common-name
verify-client-cert none  #跳过客户端证书验证

#iptables
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.8.1.0/24 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 10.8.2.0/24 -j MASQUERADE

#systemd 配置
WorkingDirectory=/etc/openvpn/server
ExecStart=/usr/sbin/openvpn  --config server.conf

#checkpsw.sh
#!/bin/sh
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman
#
# This script will authenticate OpenVPN users against
# a plain text file. The passfile should simply contain
# one row per user with the username first followed by
# one or more space(s) or tab(s) and then the password.

PASSFILE="/etc/openvpn/server/psw-file"
LOG_FILE="/etc/openvpn/server/openvpn-password.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`

###########################################################

if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
  exit 1
fi

CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`

if [ "${CORRECT_PASSWORD}" = "" ]; then
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${CORRECT_PASSWORD}" ]; then
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1

#psw-file
brande  gihtg@123


#openvpn 客户端配置
client
dev tun	#与服务器保持一致
proto tcp
remote 180.166.191.167 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
;cert brande.crt #密钥模式
;key brande.key  #密钥模式
comp-lzo
verb 3
#用户名密码模式登陆
auth-user-pass
```


# openvpn_web

``` bash
#openvpn  配置
vim /etc/openvpn/server/server.conf
verify-client-cert none
username-as-common-name
plugin /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn

script-security 2
client-connect /etc/openvpn/server/connect.py
client-disconnect /etc/openvpn/server/disconnect.py

#connect.py
#!/usr/bin/python

import os
import time
import sqlite3

username = os.environ['common_name']
trusted_ip = os.environ['trusted_ip']
trusted_port = os.environ['trusted_port']
local = os.environ['ifconfig_local']
remote = os.environ['ifconfig_pool_remote_ip']
timeunix= os.environ['time_unix']

logintime = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))

conn = sqlite3.connect("/etc/openvpn/openvpn.db")
cursor = conn.cursor()
query = "insert into t_logs(username, timeunix, trusted_ip, trusted_port, local, remote, logintime) values('%s','%s', '%s', '%s', '%s', '%s', '%s')" %  (username, timeunix, trusted_ip, trusted_port, local, remote, logintime)
cursor.execute(query)
conn.commit()
conn.close()

#disconnect.py
#!/usr/bin/python

import os
import time
import sqlite3

username = os.environ['common_name']
trusted_ip = os.environ['trusted_ip']
received = os.environ['bytes_received']
sent = os.environ['bytes_sent']

logouttime = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))

conn = sqlite3.connect("/etc/openvpn/openvpn.db")
cursor = conn.cursor()
query = "update t_logs set logouttime='%s', received='%s', sent= '%s' where username = '%s' and trusted_ip = '%s'" %  (logouttime, received, sent, username, trusted_ip)
cursor.execute(query)
conn.commit()
conn.close()


#pam for openvpn
vim /etc/pam.d/openvpn
auth        required    pam_sqlite3.so db=/etc/openvpn/openvpn.db table=t_user user=username passwd=password expire=expire crypt=1
account     required    pam_sqlite3.so db=/etc/openvpn/openvpn.db table=t_user user=username passwd=password expire=expire crypt=1

#install pam_sqlite3.so
git clone https://gitee.com/lang13002/pam_sqlite3.git
cd pam_sqlite3
make && make install

#download openvpn_web
git clone https://gitee.com/lang13002/openvpn_web.git

#create schema from sql
chmod 777 /etc/openvpn/openvpn.db
sqlite3 /etc/openvpn/openvpn.db
sqlite> .read openvpn_web/model/openvpn.sql

#启动服务
python myapp.py
```
