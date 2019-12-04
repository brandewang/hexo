---
title: haproxy
date: 2088-08-08 08:08:08
tags:
---

``` bash
#/etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    stats socket /var/lib/haproxy/stats

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen  admin_stats
    bind 0.0.0.0:10080
    mode http
    log 127.0.0.1 local0 err
    stats refresh 30s
    stats uri /status
    stats realm welcome login\ Haproxy
    stats auth admin:123456
    stats hide-version
    stats admin if TRUE

listen nginx
    bind 0.0.0.0:8080
    mode http
    option httplog
    balance roundrobin
    server 10.28.50.6 10.28.50.6:8080 check inter 2000 fall 2 rise 2 weight 1
    server 10.28.50.7 10.28.50.7:8080 check inter 2000 fall 2 rise 2 weight 1

#/etc/rsyslog.conf
$ModLoad imudp
$UDPServerRun 514
local2.*    /var/log/haproxy.log
```
