---
title: fluentd
date: 2018-05-02 14:20
tags:
---
# 生命周期
1. input(entry嵌入) -> filter -> output
2. input -> Label(filter&output)


# 获取systemd日志

``` bash
# 持久化systemd-journal日志
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/jounal
systemctl restart systemd-journald
# 对td-agent添加附属组systemd-journal以便访问/var/log/journal下的日志
usermod -a -G systemd-journal td-agent 

#需安装input-plugin插件 fluent-plugin-systemd
td-agent-gem install fluent-plugin-systemd -v 0.3.1

#添加配置文件
tail -n 1 /etc/td-agent/td-agent.conf 
include config.d/*.conf

cat > /etc/td-agent/config.d/journal.conf << EOF
<source>
  @type systemd
  tag sshd
  path /var/log/journal
  filters [{ "_SYSTEMD_UNIT": "sshd.service" }]
  read_from_head true
  <storage>
    @type local
    persistent false
    path /var/log/td-agent/sshd.pos
  </storage>
  @label @journal
</source>

<label @journal>
  <filter **>
    @type systemd_entry
    field_map {"MESSAGE": "log", "_PID": ["process", "pid"], "_CMDLINE": "process", "_COMM": "cmd"}
    field_map_strict false
    fields_lowercase true
    fields_strip_underscores true
  </filter>
  <match sshd>
    @type stdout
  </match>
</label>
EOF
```
