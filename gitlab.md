---
title: gitlab
date: 2088-08-08 08:08:08
tags:
---
# Gitlab


``` bash
#docker 启动
#!/bin/bash
export gitlab_ip=10.28.50.130
docker run -idt \
  --hostname gitlab.corp.fruitday.com \
  --publish $gitlab_ip:443:443 --publish $gitlab_ip:80:80 --publish $gitlab_ip:22:22 \
  --name gitlab-ce \
  --restart always \
  --volume /data/docker/gitlab/config:/etc/gitlab \
  --volume /data/docker/gitlab/logs:/var/log/gitlab \
  --volume /data/docker/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:10.0.3-ce.0

#备份
/usr/bin/gitlab-rake gitlab:backup:create
#默认备份目录
/var/opt/gitlab/backups

#复制备份及配置文件至目标服务器
scp /var/opt/gitlab/backups/xxx_gitlab_backup.tar  /var/opt/gitlab/backups/
scp /etc/gitlab/gitlab.rb xx.xx.xx.xx:/etc/gitlab/gitlab.rb
scp /etc/gitlab/gitlab-secrets.json xx.xx.xx.xx:/etc/gitlab/gitlab-secrets.json

#恢复
gitlab-ctl reconfigure
gitlab-rake gitlab:backup:restore BACKUP=1533614595_2018_08_07_9.2.5



##BUG
#1.页面显示Appearance无法保存, 使用psql登录postgresql删除对应记录解决
cat /var/opt/gitlab/gitlab-rails/etc/database.yml 
cat /etc/passwd
su - gitlab-psql 
psql -h /var/opt/gitlab/postgresql -d gitlabhq_production

#2.SSH服务无法使用dss秘钥登录，修改对应sshd配置文件，并重载服务
#查找进程及配置文件
ps -ef|grep sshd
#重载配置文件
kill -USR1 $PID

```
