# 第17章-输入插件Input

输入插件允许Logstash读取特定的源。

下面列举一些输入插件，Elastic支持的插件列表，请查阅 [支持矩阵](https://www.elastic.co/cn/support/matrix)。

| 插件                                                         | 说明                                                    | Github                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ |
| [azure_event_hubs](../17-Input-plugins/azure_event_hubs.md)  | 从Azure Event Hub接受事件                               | [azure_event_hubs](https://github.com/logstash-plugins/azure_event_hubs) |
| [**beats**](../17-Input-plugins/beats.md)                    | 从Elastic Beats框架接受事件                             | [logstash-input-beats](https://github.com/logstash-plugins/logstash-input-beats) |
| [cloudwatch](../17-Input-plugins/cloudwatch.md)              | 从Amazon Web Services CloudWatch API拉取事件            | [logstash-input-cloudwatch](https://github.com/logstash-plugins/logstash-input-cloudwatch) |
| [couchdb_changes](../17-Input-plugins/couchdb_changes.md)    | 来自CouchDB的 `_changes` URI的流事件                    | [logstash-input-couchdb_changes](https://github.com/logstash-plugins/logstash-input-couchdb_changes) |
| [**dead_letter_queue**](../17-Input-plugins/dead_letter_queue.md) | 从Logstash死信队列读取事件                              | [logstash-input-dead_letter_queue](https://github.com/logstash-plugins/logstash-input-dead_letter_queue) |
| [**elasticsearch**](../17-Input-plugins/elasticsearch.md)    | 从Elasticsearch集群读取查询结果                         | [logstash-input-elasticsearch](https://github.com/logstash-plugins/logstash-input-elasticsearch) |
| [exec](../17-Input-plugins/exec.md)                          | 捕获shell命令的输出作为事件                             | [logstash-input-exec](https://github.com/logstash-plugins/logstash-input-exec) |
| [**file**](../17-Input-plugins/file.md)                      | 来自文件的流事件                                        | [logstash-input-file](https://github.com/logstash-plugins/logstash-input-file) |
| [ganglia](../17-Input-plugins/ganglia.md)                    | 通过UDP读取Ganglia数据包                                | [logstash-input-ganglia](https://github.com/logstash-plugins/logstash-input-ganglia) |
| [gelf](../17-Input-plugins/gelf.md)                          | 从Graylog2读取GELF-format消息作为事件                   | [logstash-input-gelf](https://github.com/logstash-plugins/logstash-input-gelf) |
| [generator](../17-Input-plugins/generator.md)                | 生成随机日志事件用于测试                                | [logstash-input-generator](https://github.com/logstash-plugins/logstash-input-generator) |
| [github](../17-Input-plugins/github.md)                      | 从Github webhook读取事件                                | [logstash-input-github](https://github.com/logstash-plugins/logstash-input-github) |
| [google_pubsub](../17-Input-plugins/google_pubsub.md)        | 从Google Cloud PubSub服务消费事件                       | [logstash-input-google_pubsub](https://github.com/logstash-plugins/logstash-input-google_pubsub) |
| [graphite](../17-Input-plugins/graphite.md)                  | 从 `graphite` 工具读取性能数据                          | [logstash-input-graphite](https://github.com/logstash-plugins/logstash-input-graphite) |
| [heartbeat](../17-Input-plugins/heartbeat.md)                | 生成心跳事件用于测试                                    | [logstash-input-heartbeat](https://github.com/logstash-plugins/logstash-input-heartbeat) |
| [**http**](../17-Input-plugins/http.md)                      | 通过HTTP或HTTPS接收事件                                 | [logstash-input-http](https://github.com/logstash-plugins/logstash-input-http) |
| [**http_poller**](../17-Input-plugins/http_poller.md)            | 将一个HTTP API输出解码为事件                            | [logstash-input-http_poller](https://github.com/logstash-plugins/logstash-input-http_poller) |
| [imap](../17-Input-plugins/imap.md)                          | 从IMAP服务器读取邮件                                    | [logstash-input-imap](https://github.com/logstash-plugins/logstash-input-imap) |
| [irc](../17-Input-plugins/irc.md)                            | 从IRC服务器读取事件                                     | [logstash-input-irc](https://github.com/logstash-plugins/logstash-input-irc) |
| [**jdbc**](../17-Input-plugins/jdbc.md)                      | 从JDBC数据创建事件                                      | [logstash-input-jdbc](https://github.com/logstash-plugins/logstash-input-jdbc) |
| [jms](../17-Input-plugins/jms.md)                            | 从Jms中间件读取事件                                     | [logstash-input-jms](https://github.com/logstash-plugins/logstash-input-jms) |
| [jmx](../17-Input-plugins/jmx.md)                            | 通过JMX从远端Java应用获取性能数据                       | [logstash-input-jmx](https://github.com/logstash-plugins/logstash-input-jmx) |
| [**kafka**](../17-Input-plugins/kafka.md)                    | 从Kafka读取一个主题                                     | [logstash-input-kafka](https://github.com/logstash-plugins/logstash-input-kafka) |
| [kinesis](../17-Input-plugins/kinesis.md)                    | 从一个AWS Kinesis流读取事件                             | [logstash-input-kinesis](https://github.com/logstash-plugins/logstash-input-kinesis) |
| [log4j](../17-Input-plugins/log4j.md)                        | 通过一个TCP socket从Log4j `SocketAppender` 对象读取事件 | [logstash-input-log4j](https://github.com/logstash-plugins/logstash-input-log4j) |
| [lumberjack](../17-Input-plugins/lumberjack.md)              | 使用Lumberjack协议读取事件                              | [logstash-input-lumberjack](https://github.com/logstash-plugins/logstash-input-lumberjack) |
| [meetup](../17-Input-plugins/meetup.md)                      | 捕获命令行工具的输出作为一个事件                        | [logstash-input-meetup](https://github.com/logstash-plugins/logstash-input-meetup) |
| [pipe](../17-Input-plugins/pipe.md)                          | 从一个长期运行的命令管道获取流事件                      | [logstash-input-pipe](https://github.com/logstash-plugins/logstash-input-pipe) |
| [puppet_facter](../17-Input-plugins/puppet_facter.md)        | 从Puppet服务器接收facter                                | [logstash-input-puppet_facter](https://github.com/logstash-plugins/logstash-input-puppet_facter) |
| [rabbitmq](../17-Input-plugins/rabbitmq.md)                  | 从一个RabbitMQ exchange拉取事件                         | [logstash-input-rabbitmq](https://github.com/logstash-plugins/logstash-input-rabbitmq) |
| [redis](../17-Input-plugins/redis.md)                        | 从一个Redis实例读取事件                                 | [logstash-input-redis](https://github.com/logstash-plugins/logstash-input-redis) |
| [relp](../17-Input-plugins/relp.md)                          | 通过一个TCP socket接收RELP事件                          | [logstash-input-relp](https://github.com/logstash-plugins/logstash-input-relp) |
| [rss](../17-Input-plugins/rss.md)                            | 捕获命令行工具的输出作为一个事件                        | [logstash-input-rss](https://github.com/logstash-plugins/logstash-input-rss) |
| [s3](../17-Input-plugins/s3.md)                              | 从一个S3 bucket获取流事件                               | [logstash-input-s3](https://github.com/logstash-plugins/logstash-input-s3) |
| [salesforce](../17-Input-plugins/salesforce.md)              | 基于Saleforce SOQL查询创建事件                          | [logstash-input-salesforce](https://github.com/logstash-plugins/logstash-input-salesforce) |
| [snmp](../17-Input-plugins/snmp.md)                          | 使用SNMP协议对接网络设备                                | [logstash-input-snmp](https://github.com/logstash-plugins/logstash-input-snmp) |
| [snmptrap](../17-Input-plugins/snmptrap.md)                  | 基于SNMP Trap消息创建事件                               | [logstash-input-snmptrap](https://github.com/logstash-plugins/logstash-input-snmptrap) |
| [sqlite](../17-Input-plugins/sqlite.md)                      | 基于SQLite数据库的行创建事件                            | [logstash-input-sqlite](https://github.com/logstash-plugins/logstash-input-sqlite) |
| [sqs](../17-Input-plugins/sqs.md)                            | 从Amazon WebServices Simple Queue Service队列拉取事件   | [logstash-input-sqs](https://github.com/logstash-plugins/logstash-input-sqs) |
| [stdin](../17-Input-plugins/stdin.md)                        | 从标准输入读取事件                                      | [logstash-input-stdin](https://github.com/logstash-plugins/logstash-input-stdin) |
| [stomp](../17-Input-plugins/stomp.md)                        | 通过STOMP协议接收并创建消息                             | [logstash-input-stomp](https://github.com/logstash-plugins/logstash-input-stomp) |
| [syslog](../17-Input-plugins/syslog.md)                      | 从syslog消息读取事件                                    | [logstash-input-syslog](https://github.com/logstash-plugins/logstash-input-syslog) |
| [tcp](../17-Input-plugins/tcp.md)                            | 从一个TCP socket读取事件                                | [logstash-input-tcp](https://github.com/logstash-plugins/logstash-input-tcp) |
| [twitter](../17-Input-plugins/twitter.md)                    | 从Twitter Streaming API读取事件                         | [logstash-input-twitter](https://github.com/logstash-plugins/logstash-input-twitter) |
| [udp](../17-Input-plugins/udp.md)                            | 通过UDP读取事件                                         | [logstash-input-udp](https://github.com/logstash-plugins/logstash-input-udp) |
| [unix](../17-Input-plugins/unix.md)                          | 通过一个Unix socket读取事件                             | [logstash-input-unix](https://github.com/logstash-plugins/logstash-input-unix) |
| [varnishlog](../17-Input-plugins/varnishlog.md)              | 从 `varnish` 缓存共享内存日志读取                       | [logstash-input-varnishlog](https://github.com/logstash-plugins/logstash-input-varnishlog) |
| [websocket](../17-Input-plugins/websocket.md)                | 从一个websocket读取事件                                 | [logstash-input-websocket](https://github.com/logstash-plugins/logstash-input-websocket) |
| [wmi](../17-Input-plugins/wmi.md)                            | 基于一个WMI查询结果创建事件                             | [logstash-input-wmi](https://github.com/logstash-plugins/logstash-input-wmi) |
| [xmpp](../17-Input-plugins/xmpp.md)                          | 通过XMPP/Jabber协议接收事件                             | [logstash-input-xmpp](https://github.com/logstash-plugins/logstash-input-xmpp) |

