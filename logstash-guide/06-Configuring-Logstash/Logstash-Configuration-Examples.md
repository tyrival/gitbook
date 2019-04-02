## 配置示例

下面的示例说明了如何配置Logstash对事件进行过滤，处理Apache日志和syslog消息，以及如何使用条件来控制过滤器或输出处理的事件。

> **TIP：**
> 如果构建grok模式时需要帮助，请尝试 [Grok Debugger](https://www.elastic.co/guide/en/kibana/6.7/xpack-grokdebugger.html}。 Grok Debugger是基本许可证下的X-Pack功能，因此可以免费使用。

#### 配置过滤器

过滤器是一种在线处理机制，可以根据需求灵活地对数据进行切片和切块。 我们来看一些过滤器。 以下配置文件设置了 `grok` 和 `data` 过滤器。

```yaml
input { stdin { } }

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

使用此配置运行Logstash：

```shell
bin/logstash -f logstash-filter.conf
```

现在输入如下信息，这些内容会被stdin输入进行处理：

```shell
127.0.0.1 - - [11/Dec/2013:00:01:45 -0800] "GET /xampp/status.php HTTP/1.1" 200 3891 "http://cadenza/xampp/navi.php" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0"
```

可以看到一些信息返回给stdout，如下：

```yaml
{
        "message" => "127.0.0.1 - - [11/Dec/2013:00:01:45 -0800] \"GET /xampp/status.php HTTP/1.1\" 200 3891 \"http://cadenza/xampp/navi.php\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0\"",
     "@timestamp" => "2013-12-11T08:01:45.000Z",
       "@version" => "1",
           "host" => "cadenza",
       "clientip" => "127.0.0.1",
          "ident" => "-",
           "auth" => "-",
      "timestamp" => "11/Dec/2013:00:01:45 -0800",
           "verb" => "GET",
        "request" => "/xampp/status.php",
    "httpversion" => "1.1",
       "response" => "200",
          "bytes" => "3891",
       "referrer" => "\"http://cadenza/xampp/navi.php\"",
          "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0\""
}
```

正如您所看到的，Logstash（在 `grok` 过滤器的帮助下）能够解析日志行（恰好采用Apache“组合日志”格式）并将其分解为许多不同的离散信息码。这对查询和分析我们的日志数据非常有用。例如，您将能够轻松地运行HTTP响应代码，IP地址，引用等报告。 Logstash包含了很多grok模式，因此当您需要解析常见的日志格式，很可能已经有人为您完成了工作。有关更多信息，请参阅GitHub上的 [Logstash grok模式](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns) 列表。

此示例中使用的另一个过滤器是 `data` 过滤器。此过滤器解析时间戳并将其用作事件的时间戳（无论您何时摄取日志数据）。您可看到即使Logstash之后在某个时间点摄取事件，此示例中的 `@timestamp` 字段仍旧为2013年12月11日。这在回填日志时这很方便，它使您能够告诉Logstash“将此值用作此事件的时间戳”。

#### 处理 Apache 日志

让我们做一些更实际的事情：处理apache2的访问日志文件！我们将从localhost上的文件中读取输入，并根据我们的需要使用 [条件表达式](../06-Configuring-Logstash/Accessing-Event-Data-and-Fields-in-the-Configuration.md#条件表达式) 处理事件。首先，使用以下内容创建一个名为logstash-apache.conf的文件（您可以更改日志的文件路径以满足您的需要）：

```yaml
input {
  file {
    path => "/tmp/access_log"
    start_position => "beginning"
  }
}

filter {
  if [path] =~ "access" {
    mutate { replace => { "type" => "apache_access" } }
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
  }
  stdout { codec => rubydebug }
}
```

然后，使用以下日志条目（示例中为“/ tmp / access_log”）创建您在上面配置的输入文件（或使用您自己的Web服务器中的一些）：

```shell
71.141.244.242 - kurt [18/May/2011:01:48:10 -0700] "GET /admin HTTP/1.1" 301 566 "-" "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.9.2.3) Gecko/20100401 Firefox/3.6.3"
134.39.72.245 - - [18/May/2011:12:40:18 -0700] "GET /favicon.ico HTTP/1.1" 200 1189 "-" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; InfoPath.2; .NET4.0C; .NET4.0E)"
98.83.179.51 - - [18/May/2011:19:35:08 -0700] "GET /css/main.css HTTP/1.1" 200 1837 "http://www.safesand.com/information.htm" "Mozilla/5.0 (Windows NT 6.0; WOW64; rv:2.0.1) Gecko/20100101 Firefox/4.0.1"
```

现在，运行Logstash时使用 `-f` 指定此配置文件：

```shell
bin/logstash -f logstash-apache.conf
```

现在您应该在Elasticsearch中看到您的apache日志数据！ Logstash打开并读取指定的输入文件，处理它遇到的每个事件。 记录到此文件的任何日志行都将被捕获，由Logstash作为事件处理，并存储在Elasticsearch中。 另外还有一个额外的好处，它们被设置为 “apache_access” 的字段 “type” 并被隐藏（这是由输入配置中的类型 ⇒ “apache_access” 行完成的）。

在此配置中，Logstash仅监控apache access_log，但通过更改上述配置中的一行，可以轻松地同时查看access_log和error_log（实际上，任何匹配* log的文件）：

```yaml
input {
  file {
    path => "/tmp/*_log"
...
```

当你重新启动Logstash时，它将处理错误和访问日志。 但是，如果您检查数据（可能使用elasticsearch-kopf），您将看到access_log被分解为离散字段，但error_log不是。 那是因为我们使用了一个grok过滤器来匹配标准的组合apache日志格式，并自动将数据拆分成单独的字段。 如果我们可以根据其格式控制线的解析方式，那不是很好吗？ 好吧，我们可以......

请注意，Logstash不会重新处理已在access_log文件中看到的事件。 从文件中读取时，Logstash会保存其位置，并仅在添加新行时处理它们。 

### 使用条件表达式

可以使用条件表达式对过滤器和输出的事件进行控制。例如，可以根据事件所在的文件对其进行标注（access_log，error_log，以及其它一些以log结尾的文件）。

```yaml
input {
  file {
    path => "/tmp/*_log"
  }
}

filter {
  if [path] =~ "access" {
    mutate { replace => { type => "apache_access" } }
    grok {
      match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    date {
      match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
  } else if [path] =~ "error" {
    mutate { replace => { type => "apache_error" } }
  } else {
    mutate { replace => { type => "random_logs" } }
  }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

此示例使用 `type` 字段标记所有事件，但实际上不解析 `error` 或 `random` 文件。 有很多类型的错误日志，它们应该如何标记取决于您正在使用的日志。

同样，您可以使用条件将事件定向到特定输出。 例如，您可以：

- 状态为5xx的apache事件的将发送警报到nagios
- 状态为4xx将记录到Elasticsearch
- 所有状态代码将通过statsd

要告诉nagios哪些http事件状态为5xx，首先需要检查类型字段的值。 如果是apache，那么您可以检查状态字段是否包含5xx错误。 如果包含，将其发送给nagios，反之则检查状态字段是否包含4xx错误。 如果包含，请将其发送给Elasticsearch。 最后，无论状态字段包含什么，都将所有apache状态代码发送到statsd：

```yaml
output {
  if [type] == "apache" {
    if [status] =~ /^5\d\d/ {
      nagios { ...  }
    } else if [status] =~ /^4\d\d/ {
      elasticsearch { ... }
    }
    statsd { increment => "apache.%{status}" }
  }
}
```

###处理Syslog消息
Syslog是Logstash最常见的用例之一，它对符合RFC3164的日志行处理得非常好。 Syslog实际上是UNIX网络日志记录的标准，它将消息从客户端计算机发送到本地文件，或通过rsyslog发送到集中式日志服务器。 此示例中，无需运行syslog实例，我们将从命令行中伪造它，以便了解发生的情况。

首先，让我们为Logstash + syslog创建一个简单的配置文件，名为 `logstash-syslog.conf`。

```yaml
input {
  tcp {
    port => 5000
    type => syslog
  }
  udp {
    port => 5000
    type => syslog
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}

output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

使用新的配置运行Logstash：

```shell
bin/logstash -f logstash-syslog.conf
```

通常，客户端计算机将连接Logstash实例所在的5000端口，并向其发送消息。此例中，我们只是telnet到Logstash并输入一个日志行（类似于之前在STDIN中输入日志行的方式）。然后打开另一个shell窗口与Logstash syslog交互，并输入以下命令：

```shell
telnet localhost 5000
```

将以下行复制并粘贴为样本。 （可以任意尝试一些自定义样本，但请记住，如果您的数据不正确， `grok` 过滤器可能无法解析）。

```shell
Dec 23 12:11:43 louis postfix/smtpd[31499]: connect from unknown[95.75.93.154]
Dec 23 14:42:56 louis named[16000]: client 199.48.164.7#64817: query (cache) 'amsterdamboothuren.com/MX/IN' denied
Dec 23 14:30:01 louis CRON[619]: (www-data) CMD (php /usr/share/cacti/site/poller.php >/dev/null 2>/var/log/cacti/poller-error.log)
Dec 22 18:28:06 louis rsyslogd: [origin software="rsyslogd" swVersion="4.2.0" x-pid="2253" x-info="http://www.rsyslog.com"] rsyslogd was HUPed, type 'lightweight'.
```

现在可以看到Logstash在原来的shell中输出了处理和解析的信息。

```shell
{
                 "message" => "Dec 23 14:30:01 louis CRON[619]: (www-data) CMD (php /usr/share/cacti/site/poller.php >/dev/null 2>/var/log/cacti/poller-error.log)",
              "@timestamp" => "2013-12-23T22:30:01.000Z",
                "@version" => "1",
                    "type" => "syslog",
                    "host" => "0:0:0:0:0:0:0:1:52617",
        "syslog_timestamp" => "Dec 23 14:30:01",
         "syslog_hostname" => "louis",
          "syslog_program" => "CRON",
              "syslog_pid" => "619",
          "syslog_message" => "(www-data) CMD (php /usr/share/cacti/site/poller.php >/dev/null 2>/var/log/cacti/poller-error.log)",
             "received_at" => "2013-12-23 22:49:22 UTC",
           "received_from" => "0:0:0:0:0:0:0:1:52617",
    "syslog_severity_code" => 5,
    "syslog_facility_code" => 1,
         "syslog_facility" => "user-level",
         "syslog_severity" => "notice"
}
```

