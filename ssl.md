---
title: ssl
date: 2088-08-08 08:08:08
tags:
---


# letencrypt
#安装amce配置工具certbot
```
yum install certbot
```

#配置nginx验证路径，并将公网DNS指向该nginx服务器
```
    server {
        listen 80;
        server_name test.domain.com;
        access_log off;
        location ^~ /.well-known/acme-challenge/ {
            root /opt/nginx/etc/ssl/certbot/;
            #default_type "text/plain";
            #try_files $uri =404;
        }
	  }
```

#自动生成letencrypt颁发的证书
certbot certonly --agree-tos -d test.domain.com --webroot -w /opt/nginx/etc/ssl/certbot/ --dry-run

#忽略时间限制，强制更新证书
certbot renew --cert-name test.domain.com --force-renewal

#设置crontab，使证书每天自动保持续签尝试，并使nginx重新加载配置，保持证书最新
crontab -e
0 1 * * *  /usr/bin/certbot renew --renew-hook '/usr/bin/nginx -s reload'
