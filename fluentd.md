---
title: fluentd
date: 2088-08-08 08:08:08
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

# tail读取nginx_access日志,传输到elasticsearch
``` bash
cat > /etc/td-agent/config.d/pro.nginx.conf << EOF
<label @ELASTIC>
  #<filter **>
  #  @type grep
  #  <exclude>
  #    key action
  #    pattern ^logout$
  #  </exclude>
  #  @type record_transformer
  #  <record>
  #     hostname "#{Socket.gethostname}"
  #  </record>
  #</filter>
  <match **>
    @type elasticsearch
    include_tag_key true
    hosts 10.28.201.81:9200,10.28.201.82:9200,10.28.201.83:9200
#覆盖index_name设置，格式#{logstash_prefix}-#{formated_date}
    logstash_format true
    logstash_prefix ${tag}
#elasticsearch插件默认使用utc时间,设置为false则可使用本地正确时间
    utc_index false
    <buffer>
      @type file
      path /data/td-agent/prod.nginx.buffer
      chunk_limit_size 8MB
      total_limit_size 512MB
      flush_mode interval
      flush_interval 5s
      overflow_action block
      retry_type exponential_backoff
      retry_forever true
    </buffer>
  </match>
</label>

<source>
  @type tail
  path /data/logs/nginx/access*.log
  exclude_path ["/data/logs/nginx/access-test.log"]
  pos_file /data/td-agent/prod.nginx.pos
  tag prod.nginx.access
  rotate_wait 20
  <parse>
    @type regexp
    expression /^(?<request_time>[^\t]*)\t(?<upstream_response_time>[^\t]*)\t(?<remote_addr>[^\t]*)\t(?<request_length>[^\t]*)\t(?<upstream_addr>[^\t]*)\t(?<time_local>[^\t]*)\t(?<host>[^\t]*)\t(?<request>[^\t]*)\t(?<status>[^\t]*)\t(?<bytes_sent>[^\t]*)\t(?<http_referer>[^\t]*)\t(?<http_user_agent>[^\t]*)\t(?<gzip_ratio>[^\t]*)\t(?<http_x_forwarded_for>[^\t]*)\t(?<server_addr>[^\t]*)\t(?<server_port>[^\t]*)$/
    time_key time_local
    time_format %d/%b/%Y:%H:%M:%S %z
  </parse>
  @label @ELASTIC
</source>
```

# EFK+kafka
``` bash
#输入
<label @ELASTIC>
  <match **>
    @type kafka
    brokers 10.28.201.91:9092,10.28.201.92:9092,10.28.201.93:9093
    default_topic brande.nginx.access
    output_data_type json
    output_include_tag true
    output_include_time true
  </match>
</label>
<source>
  @type tail
  path /var/log/nginx/access*.log
  exclude_path ["/var/log/nginx/access-test.log"]
  pos_file /data/td-agent/prod.nginx.pos
  tag brande.nginx.access
  rotate_wait 20
  <parse>
    @type regexp
    expression /^(?<request_time>[^\t]*)\t(?<upstream_response_time>[^\t]*)\t(?<remote_addr>[^\t]*)\t(?<request_length>[^\t]*)\t(?<upstream_addr>[^\t]*)\t(?<time_local>[^\t]*)\t(?<host>[^\t]*)\t(?<request>[^\t]*)\t(?<request_body>[^\t]*)\t(?<status>[^\t]*)\t(?<bytes_sent>[^\t]*)\t(?<http_referer>[^\t]*)\t(?<http_user_agent>[^\t]*)\t(?<gzip_ratio>[^\t]*)\t(?<http_x_forwarded_for>[^\t]*)\t(?<server_addr>[^\t]*)\t(?<server_port>[^\t]*)$/
    time_key time_local
    time_format %d/%b/%Y:%H:%M:%S %z
  </parse>
  @label @ELASTIC
</source>

#输出
<source>
  @type kafka
  format json
  brokers 10.28.201.91:9092,10.28.201.92:9092,10.28.201.93:9092
  topics brande.nginx.access
  offset_zookeeper    10.28.201.91:2181,10.28.201.92:2181,10.28.201.93:2181
#在zookeeper中记录topics的偏移
  #offset_zk_root_node <offset path in zookeeper> default => '/fluent-plugin-kafka'
  @label @ELASTIC
</source>

<label @ELASTIC>
  <match  **>
    @type elasticsearch
    include_tag_key true
    hosts 10.28.201.81:9200,10.28.201.82:9200,10.28.201.83:9200
    logstash_format true
    logstash_prefix ${tag}
    logstash_dateformat %Y.%m.%d-%H
    utc_index false
  </match>
</label>

```


