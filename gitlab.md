---
title: gitlab
date: 2088-08-08 08:08:08
tags:
---


``` bash
#docker 启动
#!/bin/bash
ip addr add 10.55.5.22/24 dev eth0
export gitlab_ip=10.55.5.22
docker run -idt \
  --hostname gitlab.corp.gihtg.com \
  --publish $gitlab_ip:443:443 --publish $gitlab_ip:80:80 --publish $gitlab_ip:22:22 \
  --name gitlab \
  --restart always \
  --volume /opt/gitlab/etc:/etc/gitlab \
  --volume /opt/gitlab/logs:/var/log/gitlab \
  --volume /opt/gitlab/var:/var/opt/gitlab \
  gitlab/gitlab-ce:12.8.9-ce.0


#configuration
grep -v '^#\|^$' etc/gitlab.rb

gitlab_rails['time_zone'] ='Asia/Shanghai'   # 设置时区
gitlab_rails['ldap_enabled'] = true
gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
  main: # 'main' is the GitLab 'provider ID' of this LDAP server
    label: 'gihtg'           # 显示在登录页面上的名称
    host: '10.55.5.1'      # LDAP服务地址
    port: 389               # LDAP服务端口，如果LDAP基于SSL在端口通常为636
    uid: 'sAMAccountName'   # LDAP中用户名对应的属性，通常为'sAMAccountName'
    bind_dn: 'CN=gitlab,OU=OU_application_account,OU=Users,OU=GIHTG,DC=gihtg,DC=com' # 同步用户信息的账户格式为'domain\username'
    password: 'gihtg@123'     # 同步用户信息的账户密码
    encryption: 'plain'     # 'start_tls' or 'simple_tls' or 'plain'
    verify_certificates: false  # 如果使用SSL，则设为true
    active_directory: true     # 如果是 Active Directory LDAP server 则设为true
    allow_username_or_email_login: false  # 是否允许email登录
    lowercase_usernames: false            # 是否将用户名转为小写
    block_auto_created_users: false       # 是否自动创建用户
    base: 'OU=GIHTG,DC=gihtg,DC=com' # 搜索LDAP用户是的BaseDN
    user_filter: '(&(objectCategory=person)(objectClass=user)(memberOf=CN=gitlab-user,OU=OU_application_account,OU=Users,OU=GIHTG,DC=gihtg,DC=com))'
EOS


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
vim /etc/sshd/sshd_config
PubkeyAcceptedKeyTypes=+ssh-dss
#查找进程及配置文件
ps -ef|grep sshd
#重载配置文件
kill -USR1 $PID

```
