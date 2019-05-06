---
title: nginx
date: 2088-08-08 08:08:08
tags:
---

# Nginx Install
``` bash
cd /usr/local/src
wget https://github.com/brandewang/dockerfiles/raw/master/nginx/nginx-1.12.2.tar.gz
wget https://github.com/brandewang/dockerfiles/raw/master/nginx/nginx_upstream_check_module.tar.gz
tar zxvf nginx-1.12.2.tar.gz
tar zxvf nginx_upstream_check_module.tar.gz
cd /usr/local/src/nginx-1.12.2 &&\
	patch -p1 < ../nginx_upstream_check_module/check_1.12.1+.patch &&\
	mkdir -p /var/lib/nginx/tmp/client_body &&\
	./configure	--prefix=/usr/local/nginx --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/data/logs/nginx/error.log --http-log-path=/data/logs/nginx/access.log --pid-path=/var/run/nginx.pid --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --pid-path=/var/run/nginx.pid --lock-path=/var/lock/subsys/nginx --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_gzip_static_module --with-http_stub_status_module --with-mail --with-mail_ssl_module --with-openssl-opt=enable-tlsext --with-http_perl_module --with-stream --with-pcre --add-module=/usr/local/src/nginx_upstream_check_module &&\
	make && make install &&\
	ln -s /usr/local/nginx/sbin/nginx /usr/bin/nginx &&\
	mkdir /etc/nginx/conf.d && mkdir /etc/nginx/iplist

#set vim syntax for nginx
cd  /usr/share/vim/vim74/syntax &&\
	wget --timeout=20 -O nginx.vim https://vim.sourceforge.io/scripts/download_script.php?src_id=19394 &&\
	echo 'au BufRead,BufNewFile /etc/nginx/*,/usr/local/nginx/conf/*  set ft=nginx' >> /usr/share/vim/vim74/filetype.vim
```

# Nginx Config
``` bash
##### Main Section #####
user nobody ;
worker_processes  auto;

error_log  /data/logs/nginx/error.log notice;
pid        /var/run/nginx.pid;

#docker & supervisor --- disable daemon
#daemon off;


##### Events Section #####

events {
    worker_connections  65535;
}

##### Http Section #####

http {

    include         /etc/nginx/mime.types;
    default_type  application/octet-stream;

##### Log Definition #####

    log_format  main  '$request_time\t$upstream_response_time\t$remote_addr\t$request_length\t$upstream_addr\t$time_local\t$host\t$request\t$status\t$bytes_sent\t$http_referer\t$http_user_agent\t$gzip_ratio\t$http_x_forwarded_for\t$server_addr\t$server_port';

    access_log /data/logs/nginx/access.log main;

  ## Network Connection Configuration ##

    keepalive_timeout  10;
    tcp_nodelay        on;
    tcp_nopush         on;
    sendfile           on;

    gzip on;
    gzip_types  text/css text/javascript application/x-javascript text/xml application/xml;
    gzip_proxied any;
    gzip_http_version 1.0;
    gzip_vary on;
    gzip_min_length 1k;

  ## Memory and Disk IO resource ##

    server_names_hash_bucket_size 128;
    client_max_body_size 50m;
    client_body_buffer_size 1m;
    client_body_temp_path /dev/shm;
    client_header_buffer_size 64k;
    large_client_header_buffers 4 64k;


  ## Proxy Tunning ##

    proxy_buffering  on;
    proxy_buffers 400 256k;
    proxy_buffer_size 64k;
    proxy_temp_path /dev/shm;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_intercept_errors on;
    charset utf-8;
    proxy_connect_timeout 1800s;
    proxy_read_timeout 1800s;
    proxy_send_timeout 1800s;
    fastcgi_connect_timeout 1800;
    fastcgi_send_timeout 1800;
    fastcgi_read_timeout 1800;

    fastcgi_intercept_errors on;
    proxy_ignore_client_abort on;

  ## Security Tunning ##

    server_tokens off;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    server {
        listen 80;
        server_name _;
        access_log off;
        location / {
            access_log /data/logs/nginx/access-503.log main;
            return 503;
        }
        location /wpad.dat{
            return 404;
        }
    }
    server {
        listen 1111;
        server_name _;
        access_log off;
        location = /nginx_status {
            stub_status on;
        }

        location = /upstream_status {
            check_status;
        }
    }

    server {
#        limit_req   zone=one  burst=120;
        listen 8080;
        server_name _;
#        access_log off;
        location / {
            root   html;
            index  index.html index.htm;
        }
        location  /test {
            default_type    application/json;
            return 502 '{"status":502,"msg":"服务正在升级，请稍后再试……"}';
        }
    }
##### white iplist #####
#    geo $white {
#        default 1;
#        172.28.0.0/16 0;
#        192.168.0.0/16 0;
#    }
#
#    map $white $limit {
#        1 $binary_remote_addr;
#        0 "";
#    }
#
#    limit_req_zone  $limit  zone=one:10m  rate=1r/s;

    include /etc/nginx/conf.d/*.conf;
##### inclue #####
#    upstream app {
#        server 192.168.1.1:8080 weight=5;
#        server 192.168.1.2:8080 weight=5;
#        ip_hash;
#        check interval=1000 rise=1 fall=1 timeout=1500 type=http default_down=false;
#    }
#
#    server {
#        listen 80;
#        server_name app.domain.com;
#        include /etc/nginx/ip_list/corp-ip;
#        access_log /data/logs/nginx/access-corp.log main;
#        location / {
#            proxy_pass http://app;
#        }
#    }
###SSL
#    server {
#        listen 80;
#        server_name app.domain.com
#        location / {
#            rewrite ^(.*)$  https://$host$1 permanent;
#        }
#    }
#
#    server {
#        listen 443 ssl;
#        server_name  auth.fruitday.com;
#        ssl on;
#        ssl_certificate /etc/nginx/ssl/ca.crt;
#        ssl_certificate_key /etc/nginx/ssl/ca.key;
#        location / {
#            proxy_pass http://app;
#        }
#    }
}

##### TCP stream #####
#stream {
#    server {
#        listen 6080;
#        proxy_pass webvirtmgr-6080;
#    }
#
#    upstream webvirtmgr-6080 {
#        server 10.28.50.127:6080 weight=5 max_fails=3 fail_timeout=30s;
#    }
#}
```
