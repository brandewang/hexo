---
title: itop
date: 2088-08-08 08:08:08
tags:
---

# itop
``` bash
#运行环境配置信息
web/conf/production/config-itop.php
参数 app_root_url 应用跟目录跳转连接，需要配置正确，否则会引起页面跳转错误
PS:修改后需要清空相关二进制文件,否则部分连接地址依旧使用老的app_root_url
查找 grep -r 'app_root_url'(旧) web/data/cache-production/apc-emul/


app_root_url: http://itop.fruitday.com/

#nginx 反向代理重定向根到web
server {
    listen 80;
    server_name itop.fruitday.com;
    access_log /data/logs/nginx/access-corp.log main;
    location / {
        proxy_pass http://itop/;
    }
    location = / {
        rewrite ^  /web permanent;
    }
}


#安装
##mysql(5.7.29)
1.mysql参数配置
2.itop数据库创建
3.itop用户权限创建

##php(7.2 yum安装便于添加扩展)
yum install epel-release unzip
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum install -y php72w-common.x86_64 php72w-fpm.x86_64 php72w-mysql.x86_64 php72w-cli.x86_64 php72w-soap.x86_64 php72w-ldap.x86_64 php72w-gd.x86_64  php72w-mbstring.x86_64 php72w-xml.x86_64
yum install -y graphviz
yum install -y httpd mod_php72w.x86_64

#php.ini
memory_limit = 256M
date.timezone = RPC

#nginx(暂定)

#itop(2.6.3)
curl -O https://nchc.dl.sourceforge.net/project/itop/itop/2.6.3/iTop-2.6.3-5092.zip

```
