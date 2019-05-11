# 第18章-输出插件Output

输出插件发送数据到一个特定目的地，输出是管道的最终阶段。

下面列举一些输出插件，Elastic支持的插件列表，请查阅 [支持矩阵](https://www.elastic.co/cn/support/matrix)。

| 插件                                                         | 说明                                                       | Github                                                       |
| ------------------------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------ |
| [boundary](../18-Output-plugins/boundary.md)                 | 基于Logstash事件发送注解到Boundary                         | [logstash-output-boundary](https://github.com/logstash-plugins/logstash-output-boundary) |
| [circonus](../18-Output-plugins/circonus.md)                 | 基于Logstash事件发送注解到Circonus                         | [logstash-output-circonus](https://github.com/logstash-plugins/logstash-output-circonus) |
| [cloudwatch](../18-Output-plugins/cloudwatch.md)             | 聚合并发送度量数据到AWS CloudWatch                         | [logstash-output-cloudwatch](https://github.com/logstash-plugins/logstash-output-cloudwatch) |
| [csv](../18-Output-plugins/csv.md)                           | 以限定的格式将事件写到磁盘上                               | [logstash-output-csv](https://github.com/logstash-plugins/logstash-output-csv) |
| [datadog](../18-Output-plugins/datadog.md)                   | 基于Logstash事件发送注解到DataDogHQ                        | [logstash-output-datadog](https://github.com/logstash-plugins/logstash-output-datadog) |
| [datadog_metrics](../18-Output-plugins/datadog_metrics.md)   | 基于Logstash事件发送度量到DataDogHQ                        | [logstash-output-datadog_metrics](https://github.com/logstash-plugins/logstash-output-datadog_metrics) |
| [elastic_app_search](../18-Output-plugins/elastic_app_search.md) | 发送事件到Elastic App Search的解决方案                     | [logstash-output-elastic_app_search](https://github.com/logstash-plugins/logstash-output-elastic_app_search) |
| [**elasticsearch**](../18-Output-plugins/elasticsearch.md)   | 储存日志到Elasticsearch                                    | [logstash-output-elasticsearch](https://github.com/logstash-plugins/logstash-output-elasticsearch) |
| [email](../18-Output-plugins/email.md)                       | 当接收到输出内容时，发送数据到指定的email                  | [logstash-output-email](https://github.com/logstash-plugins/logstash-output-email) |
| [exec](../18-Output-plugins/exec.md)                         | 为匹配的事件之行一个command命令                            | [logstash-output-exec](https://github.com/logstash-plugins/logstash-output-exec) |
| [file](../18-Output-plugins/file.md)                         | 将事件写入磁盘上的文件                                     | [logstash-output-file](https://github.com/logstash-plugins/logstash-output-file) |
| [ganglia](../18-Output-plugins/ganglia.md)                   | 将度量写入Ganglia的 `gmond`                                | [logstash-output-ganglia](https://github.com/logstash-plugins/logstash-output-ganglia) |
| [gelf](../18-Output-plugins/gelf.md)                         | 为Graylog2生成GELF格式的输出                               | [logstash-output-gelf](https://github.com/logstash-plugins/logstash-output-gelf) |
| [google_bigquery](../18-Output-plugins/google_bigquery.md)   | 将事件写入Google BigQuery                                  | [logstash-output-google_bigquery](https://github.com/logstash-plugins/logstash-output-google_bigquery) |
| [google_pubsub](../18-Output-plugins/google_pubsub.md)       | 上传日志事件到Google Cloud PubSub                          | [logstash-output-google_pubsub](https://github.com/logstash-plugins/logstash-output-google_pubsub) |
| [graphite](../18-Output-plugins/graphite.md)                 | 将度量写入Graphite                                         | [logstash-output-graphite](https://github.com/logstash-plugins/logstash-output-graphite) |
| [graphtastic](../18-Output-plugins/graphtastic.md)           | 发送度量数据到窗口                                         | [logstash-output-graphtastic](https://github.com/logstash-plugins/logstash-output-graphtastic) |
| [**http**](../18-Output-plugins/http.md)                         | 发送事件到通用的HTTP或HTTPS端点                            | [logstash-output-http](https://github.com/logstash-plugins/logstash-output-http) |
| [influxdb](../18-Output-plugins/influxdb.md)                 | 将度量写入InfluxDB                                         | [logstash-output-influxdb](https://github.com/logstash-plugins/logstash-output-influxdb) |
| [irc](../18-Output-plugins/irc.md)                           | 将事件写入IRC                                              | [logstash-output-irc](https://github.com/logstash-plugins/logstash-output-irc) |
| [juggernaut](../18-Output-plugins/juggernaut.md)             | 推送消息到Juggernaut websockets服务                        | [logstash-output-juggernaut](https://github.com/logstash-plugins/logstash-output-juggernaut) |
| [kafka](../18-Output-plugins/kafka.md)                       | 将事件写入一个Kafka主题                                    | [logstash-output-kafka](https://github.com/logstash-plugins/logstash-output-kafka) |
| [librato](../18-Output-plugins/librato.md)                   | 基于Logstash事件发送度量、竹节和警报到Librato              | [logstash-output-librato](https://github.com/logstash-plugins/logstash-output-librato) |
| [loggly](../18-Output-plugins/loggly.md)                     | 发送日志到Loggly                                           | [logstash-output-loggly](https://github.com/logstash-plugins/logstash-output-loggly) |
| [lumberjack](../18-Output-plugins/lumberjack.md)             | 通过 `lumberjack` 协议发送事件                             | [logstash-output-lumberjack](https://github.com/logstash-plugins/logstash-output-lumberjack) |
| [metriccatcher](../18-Output-plugins/metriccatcher.md)       | 将度量写入MetricCatcher                                    | [logstash-output-metriccatcher](https://github.com/logstash-plugins/logstash-output-metriccatcher) |
| [mongodb](../18-Output-plugins/mongodb.md)                   | 将事件写入MongoDB                                          | [logstash-output-mongodb](https://github.com/logstash-plugins/logstash-output-mongodb) |
| [nagios](../18-Output-plugins/nagios.md)                     | 发送被动检查结果到Nagios                                   | [logstash-output-nagios](https://github.com/logstash-plugins/logstash-output-nagios) |
| [nagios_nsca](../18-Output-plugins/nagios_nsca.md)           | 使用NSCA协议发送被动检查结果到Nagios                       | [logstash-output-nagios_nsca](https://github.com/logstash-plugins/logstash-output-nagios_nsca) |
| [opentsdb](../18-Output-plugins/opentsdb.md)                 | 发送度量到OpenTSDB                                         | [logstash-output-opentsdb](https://github.com/logstash-plugins/logstash-output-opentsdb) |
| [pagerduty](../18-Output-plugins/pagerduty.md)               | 根据预配置的服务和升级策略发送通知                         | [logstash-output-pagerduty](https://github.com/logstash-plugins/logstash-output-pagerduty) |
| [pipe](../18-Output-plugins/pipe.md)                         | 输出管道事件到另一个程序的标准输入                         | [logstash-output-pipe](https://github.com/logstash-plugins/logstash-output-pipe) |
| [rabbitmq](../18-Output-plugins/rabbitmq.md)                 | 推送事件到一个RabbitMQ exchange                            | [logstash-output-rabbitmq](https://github.com/logstash-plugins/logstash-output-rabbitmq) |
| [redis](../18-Output-plugins/redis.md)                       | 使用 `RPUSH` 命令发送事件到一个Redis队列                   | [logstash-output-redis](https://github.com/logstash-plugins/logstash-output-redis) |
| [redmine](../18-Output-plugins/redmine.md)                   | 使用Redmine API创建tickets                                 | [logstash-output-redmine](https://github.com/logstash-plugins/logstash-output-redmine) |
| [riak](../18-Output-plugins/riak.md)                         | 将事件写入Riak分布式键值对存储                             | [logstash-output-riak](https://github.com/logstash-plugins/logstash-output-riak) |
| [riemann](../18-Output-plugins/riemann.md)                   | 发送度量到Riemann                                          | [logstash-output-riemann](https://github.com/logstash-plugins/logstash-output-riemann) |
| [s3](../18-Output-plugins/s3.md)                             | 发送Logstash事件到Amazon Simple Storage Service            | [logstash-output-s3](https://github.com/logstash-plugins/logstash-output-s3) |
| [sns](../18-Output-plugins/sns.md)                           | 发送事件到Amazon的 Simple Notification Service             | [logstash-output-sns](https://github.com/logstash-plugins/logstash-output-sns) |
| [solr_http](../18-Output-plugins/solr_http.md)               | 在solr储存并索引日志                                       | [logstash-output-solr_http](https://github.com/logstash-plugins/logstash-output-solr_http) |
| [sqs](../18-Output-plugins/sqs.md)                           | 推送事件到一个Amazon Web Services Simple Queue Service队列 | [logstash-output-sqs](https://github.com/logstash-plugins/logstash-output-sqs) |
| [statsd](../18-Output-plugins/statsd.md)                     | 使用 `statsd` 网络守护进程发送度量                         | [logstash-output-statsd](https://github.com/logstash-plugins/logstash-output-statsd) |
| [stdout](../18-Output-plugins/stdout.md)                     | 打印 事件到标准输出                                        | [logstash-output-stdout](https://github.com/logstash-plugins/logstash-output-stdout) |
| [stomp](../18-Output-plugins/stomp.md)                       | 使用STOMP协议写入事件                                      | [logstash-output-stomp](https://github.com/logstash-plugins/logstash-output-stomp) |
| [syslog](../18-Output-plugins/syslog.md)                     | 发送事件到一个 `syslog` 服务器                             | [logstash-output-syslog](https://github.com/logstash-plugins/logstash-output-syslog) |
| [tcp](../18-Output-plugins/tcp.md)                           | 通过一个TCP socket写入事件                                 | [logstash-output-tcp](https://github.com/logstash-plugins/logstash-output-tcp) |
| [timber](../18-Output-plugins/timber.md)                     | 发送事件到Timber.io日志服务                                | [logstash-output-timber](https://github.com/logstash-plugins/logstash-output-timber) |
| [udp](../18-Output-plugins/udp.md)                           | 通过UDP发送事件                                            | [logstash-output-udp](https://github.com/logstash-plugins/logstash-output-udp) |
| [webhdfs](../18-Output-plugins/webhdfs.md)                   | 通过 `webhdfs` REST API发送Logstash事件到HDFS              | [logstash-output-webhdfs](https://github.com/logstash-plugins/logstash-output-webhdfs) |
| [websocket](../18-Output-plugins/websocket.md)               | 发布消息到一个websocket                                    | [logstash-output-websocket](https://github.com/logstash-plugins/logstash-output-websocket) |
| [xmpp](../18-Output-plugins/xmpp.md)                         | 通过XMPP发送事件                                           | [logstash-output-xmpp](https://github.com/logstash-plugins/logstash-output-xmpp) |
| [zabbix](../18-Output-plugins/zabbix.md)                     | 发送事件到一个Zabbix服务器                                 | [logstash-output-zabbix](https://github.com/logstash-plugins/logstash-output-zabbix) |

