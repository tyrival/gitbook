## 使用Logstash管道解析

本节中的示例显示如何构建Logstash管道配置，以替换Filebeat模块提供的接收管道。管道获取Filebeat模块收集的数据，将其解析为Filebeat索引所期望的字段，并将字段发送到Elasticsearch，以便可以可视化Filebeat提供的预构建仪表板中的数据。

这种方法比使用现有的提取管道解析数据更耗时，但它可以让您更好地控制数据的处理方式。通过编写自己的管道配置，您可以在提取字段后执行其他处理，例如删除字段，或者可以将负载从Elasticsearch提取节点移动到Logstash节点。

在决定使用Logstash配置替换摄取管道之前，请阅读 [使用采集管道解析](../09-Working-with-Filebeat-Modules/Use-ingest-pipelines-for-parsing.md)。

以下是一些示例，说明如何实现Logstash配置以替换摄取管道：

- [Apache 2日志](#Apache 2日志)
- [MySQL日志](#MySQL日志)
- [Nginx日志](#Nginx日志)
- [System日志](#System日志)

> **提醒：**
> Logstash提供了 [转换摄取节点通道](06-Configuring-Logstash/Converting-Ingest-Node-Pipelines.md)，可帮助您将摄取管道定义迁移到Logstash配置。该工具目前不支持所有可用于摄取节点的处理器，但它是一个很好的起点。

### Apache 2 Logs
此示例中的Logstash管道配置，显示了如何发送和解析 [Apache2 Filebeat模块](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-module-apache2.html) 收集的访问和错误日志。

```json
input {
  beats {
    port => 5044
    host => "0.0.0.0"
  }
}
filter {
  if [fileset][module] == "apache2" {
    if [fileset][name] == "access" {
      grok {
        match => { "message" => ["%{IPORHOST:[apache2][access][remote_ip]} - %{DATA:[apache2][access][user_name]} \[%{HTTPDATE:[apache2][access][time]}\] \"%{WORD:[apache2][access][method]} %{DATA:[apache2][access][url]} HTTP/%{NUMBER:[apache2][access][http_version]}\" %{NUMBER:[apache2][access][response_code]} %{NUMBER:[apache2][access][body_sent][bytes]}( \"%{DATA:[apache2][access][referrer]}\")?( \"%{DATA:[apache2][access][agent]}\")?",
          "%{IPORHOST:[apache2][access][remote_ip]} - %{DATA:[apache2][access][user_name]} \\[%{HTTPDATE:[apache2][access][time]}\\] \"-\" %{NUMBER:[apache2][access][response_code]} -" ] }
        remove_field => "message"
      }
      mutate {
        add_field => { "read_timestamp" => "%{@timestamp}" }
      }
      date {
        match => [ "[apache2][access][time]", "dd/MMM/YYYY:H:m:s Z" ]
        remove_field => "[apache2][access][time]"
      }
      useragent {
        source => "[apache2][access][agent]"
        target => "[apache2][access][user_agent]"
        remove_field => "[apache2][access][agent]"
      }
      geoip {
        source => "[apache2][access][remote_ip]"
        target => "[apache2][access][geoip]"
      }
    }
    else if [fileset][name] == "error" {
      grok {
        match => { "message" => ["\[%{APACHE_TIME:[apache2][error][timestamp]}\] \[%{LOGLEVEL:[apache2][error][level]}\]( \[client %{IPORHOST:[apache2][error][client]}\])? %{GREEDYDATA:[apache2][error][message]}",
          "\[%{APACHE_TIME:[apache2][error][timestamp]}\] \[%{DATA:[apache2][error][module]}:%{LOGLEVEL:[apache2][error][level]}\] \[pid %{NUMBER:[apache2][error][pid]}(:tid %{NUMBER:[apache2][error][tid]})?\]( \[client %{IPORHOST:[apache2][error][client]}\])? %{GREEDYDATA:[apache2][error][message1]}" ] }
        pattern_definitions => {
          "APACHE_TIME" => "%{DAY} %{MONTH} %{MONTHDAY} %{TIME} %{YEAR}"
        }
        remove_field => "message"
      }
      mutate {
        rename => { "[apache2][error][message1]" => "[apache2][error][message]" }
      }
      date {
        match => [ "[apache2][error][timestamp]", "EEE MMM dd H:m:s YYYY", "EEE MMM dd H:m:s.SSSSSS YYYY" ]
        remove_field => "[apache2][error][timestamp]"
      }
    }
  }
}
output {
  elasticsearch {
    hosts => localhost
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

### MySQL日志

此示例中的Logstash管道配置，显示了如何发送和解析 [mysql Filebeat模块](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-module-mysql.html) 收集的错误和slowlog日志。

```json
input {
  beats {
    port => 5044
    host => "0.0.0.0"
  }
}
filter {
  if [fileset][module] == "mysql" {
    if [fileset][name] == "error" {
      grok {
        match => { "message" => ["%{LOCALDATETIME:[mysql][error][timestamp]} (\[%{DATA:[mysql][error][level]}\] )?%{GREEDYDATA:[mysql][error][message]}",
          "%{TIMESTAMP_ISO8601:[mysql][error][timestamp]} %{NUMBER:[mysql][error][thread_id]} \[%{DATA:[mysql][error][level]}\] %{GREEDYDATA:[mysql][error][message1]}",
          "%{GREEDYDATA:[mysql][error][message2]}"] }
        pattern_definitions => {
          "LOCALDATETIME" => "[0-9]+ %{TIME}"
        }
        remove_field => "message"
      }
      mutate {
        rename => { "[mysql][error][message1]" => "[mysql][error][message]" }
      }
      mutate {
        rename => { "[mysql][error][message2]" => "[mysql][error][message]" }
      }
      date {
        match => [ "[mysql][error][timestamp]", "ISO8601", "YYMMdd H:m:s" ]
        remove_field => "[mysql][error][time]"
      }
    }
    else if [fileset][name] == "slowlog" {
      grok {
        match => { "message" => ["^# User@Host: %{USER:[mysql][slowlog][user]}(\[[^\]]+\])? @ %{HOSTNAME:[mysql][slowlog][host]} \[(IP:[mysql][slowlog][ip])?\](\s*Id:\s* %{NUMBER:[mysql][slowlog][id]})?\n# Query_time: %{NUMBER:[mysql][slowlog][query_time][sec]}\s* Lock_time: %{NUMBER:[mysql][slowlog][lock_time][sec]}\s* Rows_sent: %{NUMBER:[mysql][slowlog][rows_sent]}\s* Rows_examined: %{NUMBER:[mysql][slowlog][rows_examined]}\n(SET timestamp=%{NUMBER:[mysql][slowlog][timestamp]};\n)?%{GREEDYMULTILINE:[mysql][slowlog][query]}"] }
        pattern_definitions => {
          "GREEDYMULTILINE" => "(.|\n)*"
        }
        remove_field => "message"
      }
      date {
        match => [ "[mysql][slowlog][timestamp]", "UNIX" ]
      }
      mutate {
        gsub => ["[mysql][slowlog][query]", "\n# Time: [0-9]+ [0-9][0-9]:[0-9][0-9]:[0-9][0-9](\\.[0-9]+)?$", ""]
      }
    }
  }
}
output {
  elasticsearch {
    hosts => localhost
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

### Nginx日志

此示例中的Logstash管道配置显示了如何发送和解析 [nginx Filebeat模块](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-module-nginx.html) 收集的访问和错误日志。

```json
input {
  beats {
    port => 5044
    host => "0.0.0.0"
  }
}
filter {
  if [fileset][module] == "nginx" {
    if [fileset][name] == "access" {
      grok {
        match => { "message" => ["%{IPORHOST:[nginx][access][remote_ip]} - %{DATA:[nginx][access][user_name]} \[%{HTTPDATE:[nginx][access][time]}\] \"%{WORD:[nginx][access][method]} %{DATA:[nginx][access][url]} HTTP/%{NUMBER:[nginx][access][http_version]}\" %{NUMBER:[nginx][access][response_code]} %{NUMBER:[nginx][access][body_sent][bytes]} \"%{DATA:[nginx][access][referrer]}\" \"%{DATA:[nginx][access][agent]}\""] }
        remove_field => "message"
      }
      mutate {
        add_field => { "read_timestamp" => "%{@timestamp}" }
      }
      date {
        match => [ "[nginx][access][time]", "dd/MMM/YYYY:H:m:s Z" ]
        remove_field => "[nginx][access][time]"
      }
      useragent {
        source => "[nginx][access][agent]"
        target => "[nginx][access][user_agent]"
        remove_field => "[nginx][access][agent]"
      }
      geoip {
        source => "[nginx][access][remote_ip]"
        target => "[nginx][access][geoip]"
      }
    }
    else if [fileset][name] == "error" {
      grok {
        match => { "message" => ["%{DATA:[nginx][error][time]} \[%{DATA:[nginx][error][level]}\] %{NUMBER:[nginx][error][pid]}#%{NUMBER:[nginx][error][tid]}: (\*%{NUMBER:[nginx][error][connection_id]} )?%{GREEDYDATA:[nginx][error][message]}"] }
        remove_field => "message"
      }
      mutate {
        rename => { "@timestamp" => "read_timestamp" }
      }
      date {
        match => [ "[nginx][error][time]", "YYYY/MM/dd H:m:s" ]
        remove_field => "[nginx][error][time]"
      }
    }
  }
}
output {
  elasticsearch {
    hosts => localhost
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

### 系统日志

此示例中的Logstash管道配置显示了如何发送和解析由 [系统Filebeat模块](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-module-system.html) 收集的系统日志。

```json
input {
  beats {
    port => 5044
    host => "0.0.0.0"
  }
}
filter {
  if [fileset][module] == "system" {
    if [fileset][name] == "auth" {
      grok {
        match => { "message" => ["%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} sshd(?:\[%{POSINT:[system][auth][pid]}\])?: %{DATA:[system][auth][ssh][event]} %{DATA:[system][auth][ssh][method]} for (invalid user )?%{DATA:[system][auth][user]} from %{IPORHOST:[system][auth][ssh][ip]} port %{NUMBER:[system][auth][ssh][port]} ssh2(: %{GREEDYDATA:[system][auth][ssh][signature]})?",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} sshd(?:\[%{POSINT:[system][auth][pid]}\])?: %{DATA:[system][auth][ssh][event]} user %{DATA:[system][auth][user]} from %{IPORHOST:[system][auth][ssh][ip]}",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} sshd(?:\[%{POSINT:[system][auth][pid]}\])?: Did not receive identification string from %{IPORHOST:[system][auth][ssh][dropped_ip]}",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} sudo(?:\[%{POSINT:[system][auth][pid]}\])?: \s*%{DATA:[system][auth][user]} :( %{DATA:[system][auth][sudo][error]} ;)? TTY=%{DATA:[system][auth][sudo][tty]} ; PWD=%{DATA:[system][auth][sudo][pwd]} ; USER=%{DATA:[system][auth][sudo][user]} ; COMMAND=%{GREEDYDATA:[system][auth][sudo][command]}",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} groupadd(?:\[%{POSINT:[system][auth][pid]}\])?: new group: name=%{DATA:system.auth.groupadd.name}, GID=%{NUMBER:system.auth.groupadd.gid}",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} useradd(?:\[%{POSINT:[system][auth][pid]}\])?: new user: name=%{DATA:[system][auth][user][add][name]}, UID=%{NUMBER:[system][auth][user][add][uid]}, GID=%{NUMBER:[system][auth][user][add][gid]}, home=%{DATA:[system][auth][user][add][home]}, shell=%{DATA:[system][auth][user][add][shell]}$",
                  "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} %{DATA:[system][auth][program]}(?:\[%{POSINT:[system][auth][pid]}\])?: %{GREEDYMULTILINE:[system][auth][message]}"] }
        pattern_definitions => {
          "GREEDYMULTILINE"=> "(.|\n)*"
        }
        remove_field => "message"
      }
      date {
        match => [ "[system][auth][timestamp]", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
      }
      geoip {
        source => "[system][auth][ssh][ip]"
        target => "[system][auth][ssh][geoip]"
      }
    }
    else if [fileset][name] == "syslog" {
      grok {
        match => { "message" => ["%{SYSLOGTIMESTAMP:[system][syslog][timestamp]} %{SYSLOGHOST:[system][syslog][hostname]} %{DATA:[system][syslog][program]}(?:\[%{POSINT:[system][syslog][pid]}\])?: %{GREEDYMULTILINE:[system][syslog][message]}"] }
        pattern_definitions => { "GREEDYMULTILINE" => "(.|\n)*" }
        remove_field => "message"
      }
      date {
        match => [ "[system][syslog][timestamp]", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
      }
    }
  }
}
output {
  elasticsearch {
    hosts => localhost
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

