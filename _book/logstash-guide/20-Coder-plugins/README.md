# 第20章-编解码器Codec

编解码器修改事件数据的表现形式。 编解码器本质上是流过滤器，可以作为输入或输出的一部分。

下面列举一些输出插件，Elastic支持的插件列表，请查阅 [支持矩阵](https://www.elastic.co/cn/support/matrix)。

| 插件                                         | 描述                                                 | Github                                                       |
| -------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| [avro](../20-Coder-plugins/avro.md)             | 将序列化Avro记录读取为Logstash事件                   | [logstash-codec-avro](https://github.com/logstash-plugins/logstash-codec-avro) |
| [cef](../20-Coder-plugins/cef.md)               | 读取ArcSight通用事件格式（CEF）                      | [logstash-codec-cef](https://github.com/logstash-plugins/logstash-codec-cef) |
| [cloudfront](../20-Coder-plugins/cloudfront.md) | 读取AWS CloudFront报告                               | [logstash-codec-cloudfront](https://github.com/logstash-plugins/logstash-codec-cloudfront) |
| [cloudtrail](../20-Coder-plugins/cloudtrail.md) | 读取AWS CloudTrail日志文件                           | [logstash-codec-cloudtrail](https://github.com/logstash-plugins/logstash-codec-cloudtrail) |
| [collectd](../20-Coder-plugins/collectd.md)     | 使用UDP从 `collectd` 二进制协议读取事件              | [logstash-codec-collectd](https://github.com/logstash-plugins/logstash-codec-collectd) |
| [dots](../20-Coder-plugins/dots.md)             | 每个事件向 `stdout` 发送1个点，以进行性能跟踪        | [logstash-codec-dots](https://github.com/logstash-plugins/logstash-codec-dots) |
| [edn](../20-Coder-plugins/edn.md)               | 读取EDN格式的数据                                    | [logstash-codec-edn](https://github.com/logstash-plugins/logstash-codec-edn) |
| [edn_lines](../20-Coder-plugins/edn_lines.md)   | 读取换行符分隔的EDN格式数据                          | [logstash-codec-edn_lines](https://github.com/logstash-plugins/logstash-codec-edn_lines) |
| [es_bulk](../20-Coder-plugins/es_bulk.md)       | 将Elasticsearch批量格式带上元数据读入不同的事件      | [logstash-codec-es_bulk](https://github.com/logstash-plugins/logstash-codec-es_bulk) |
| [fluent](../20-Coder-plugins/fluent.md)         | 读取 `fluentd` `msgpack` 模式                        | [logstash-codec-fluent](https://github.com/logstash-plugins/logstash-codec-fluent) |
| [graphite](../20-Coder-plugins/graphite.md)     | 读取 `graphite` 格式的行                             | [logstash-codec-graphite](https://github.com/logstash-plugins/logstash-codec-graphite) |
| [gzip_lines](../20-Coder-plugins/gzip_lines.md) | 读取 `gzip` 编码的内容                               | [logstash-codec-gzip_lines](https://github.com/logstash-plugins/logstash-codec-gzip_lines) |
| [**json**](../20-Coder-plugins/json.md)             | 读取JSON格式的内容，JSON数组中的每个元素创建一个事件 | [logstash-codec-json](https://github.com/logstash-plugins/logstash-codec-json) |
| [**json_lines**](../20-Coder-plugins/json_lines.md) | 读取换行分隔的JSON                                   | [logstash-codec-json_lines](https://github.com/logstash-plugins/logstash-codec-json_lines) |
| [line](../20-Coder-plugins/line.md)             | 按行读取文本数据                                     | [logstash-codec-line](https://github.com/logstash-plugins/logstash-codec-line) |
| [msgpack](../20-Coder-plugins/msgpack.md)       | 读取MessagePack编码内容                              | [logstash-codec-msgpack](https://github.com/logstash-plugins/logstash-codec-msgpack) |
| [multiline](../20-Coder-plugins/multiline.md)   | 合并多行消息到一个事件                               | [logstash-codec-multiline](https://github.com/logstash-plugins/logstash-codec-multiline) |
| [netflow](../20-Coder-plugins/netflow.md)       | 读取Netflow v5和v9数据                               | [logstash-codec-netflow](https://github.com/logstash-plugins/logstash-codec-netflow) |
| [nmap](../20-Coder-plugins/nmap.md)             | 将Nmap数据读入XML格式                                | [logstash-codec-nmap](https://github.com/logstash-plugins/logstash-codec-nmap) |
| [plain](../20-Coder-plugins/plain.md)           | 连续读取事件为纯文本，事件间不分隔                   | [logstash-codec-plain](https://github.com/logstash-plugins/logstash-codec-plain) |
| [protobuf](../20-Coder-plugins/protobuf.md)     | 读取protobuf消息并转化为Logstash事件                 | [logstash-codec-protobuf](https://github.com/logstash-plugins/logstash-codec-protobuf) |
| [rubydebug](../20-Coder-plugins/rubydebug.md)   | 将Ruby Awesome Print库应用于Logstash事件             | [logstash-codec-rubydebug](https://github.com/logstash-plugins/logstash-codec-rubydebug) |

