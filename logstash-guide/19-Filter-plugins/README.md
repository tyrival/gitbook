# 第19章-过滤器Filter

过滤器插件对事件执行中间处理。 过滤器通常根据事件的特征有条件地应用。

下面列举一些输出插件，Elastic支持的插件列表，请查阅 [支持矩阵](https://www.elastic.co/cn/support/matrix)。

| 插件                                                         | 描述                                                         | Github                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [aggregate](../19-Filter-plugins/aggregate.md)               | 使一个任务发起的多个事件进行聚合                             | [logstash-filter-aggregate](https://github.com/logstash-plugins/logstash-filter-aggregate) |
| [alter](../19-Filter-plugins/alter.md)                       | 对 `mutate` 过滤器不处理的字段进行常规改造                   | [logstash-filter-alter](https://github.com/logstash-plugins/logstash-filter-alter) |
| [cidr](../19-Filter-plugins/cidr.md)                         | 根据网络地址块列表检查IP                                     | [logstash-filter-cidr](https://github.com/logstash-plugins/logstash-filter-cidr) |
| [cipher](../19-Filter-plugins/cipher.md)                     | 为事件增加或移除一个cipher                                   | [logstash-filter-cipher](https://github.com/logstash-plugins/logstash-filter-cipher) |
| [clone](../19-Filter-plugins/clone.md)                       | 复制事件                                                     | [logstash-filter-clone](https://github.com/logstash-plugins/logstash-filter-clone) |
| [csv](../19-Filter-plugins/csv.md)                           | 通过逗号分隔符将值解析为不同的字段                           | [logstash-filter-csv](https://github.com/logstash-plugins/logstash-filter-csv) |
| [date](../19-Filter-plugins/date.md)                         | 从事件的字段解析日期，成为Logstash时间戳                     | [logstash-filter-date](https://github.com/logstash-plugins/logstash-filter-date) |
| [de_dot](../19-Filter-plugins/de_dot.md)                     | 将字段中的点进行移除                                         | [logstash-filter-de_dot](https://github.com/logstash-plugins/logstash-filter-de_dot) |
| [dissect](../19-Filter-plugins/dissect.md)                   | 使用分隔符将非结构化的事件数据提取到字段中                   | [logstash-filter-dissect](https://github.com/logstash-plugins/logstash-filter-dissect) |
| [dns](../19-Filter-plugins/dns.md)                           | 执行标准或反向DNS检索                                        | [logstash-filter-dns](https://github.com/logstash-plugins/logstash-filter-dns) |
| [drop](../19-Filter-plugins/drop.md)                         | 放弃所有事件                                                 | [logstash-filter-drop](https://github.com/logstash-plugins/logstash-filter-drop) |
| [elapsed](../19-Filter-plugins/elapsed.md)                   | 计算一对事件之间的间隔时间                                   | [logstash-filter-elapsed](https://github.com/logstash-plugins/logstash-filter-elapsed) |
| [elasticsearch](../19-Filter-plugins/elasticsearch.md)       | 将Elasticsearch中之前的日志事件字段复制到当前事件            | [logstash-filter-elasticsearch](https://github.com/logstash-plugins/logstash-filter-elasticsearch) |
| [environment](../19-Filter-plugins/environment.md)           | 将环境变量作为元数据储存到子字段                             | [logstash-filter-environment](https://github.com/logstash-plugins/logstash-filter-environment) |
| [extractnumbers](../19-Filter-plugins/extractnumbers.md)     | 从字符串提取数字                                             | [logstash-filter-extractnumbers](https://github.com/logstash-plugins/logstash-filter-extractnumbers) |
| [fingerprint](../19-Filter-plugins/fingerprint.md)           | 通过使用一致哈希值来Fingerprints字段                         | [logstash-filter-fingerprint](https://github.com/logstash-plugins/logstash-filter-fingerprint) |
| [geoip](../19-Filter-plugins/geoip.md)                       | 增加IP对应的地理信息                                         | [logstash-filter-geoip](https://github.com/logstash-plugins/logstash-filter-geoip) |
| [grok](../19-Filter-plugins/grok.md)                         | 解析非结构化的事件数据到字段                                 | [logstash-filter-grok](https://github.com/logstash-plugins/logstash-filter-grok) |
| [**http**](../19-Filter-plugins/http.md)                     | 提供与外部web service/REST API集成的功能                     | [logstash-filter-http](https://github.com/logstash-plugins/logstash-filter-http) |
| [i18n](../19-Filter-plugins/i18n.md)                         | 从字段中移除指定字符                                         | [logstash-filter-i18n](https://github.com/logstash-plugins/logstash-filter-i18n) |
| [**jdbc_static**](../19-Filter-plugins/jdbc_static.md)       | 使用从远端数据库中预加载的数据丰富事件                       | [logstash-filter-jdbc_static](https://github.com/logstash-plugins/logstash-filter-jdbc_static) |
| [**jdbc_streaming**](../19-Filter-plugins/jdbc_streaming.md) | 使用您的数据库数据丰富事件                                   | [logstash-filter-jdbc_streaming](https://github.com/logstash-plugins/logstash-filter-jdbc_streaming) |
| [json](../19-Filter-plugins/json.md)                         | 解析JSON事件                                                 | [logstash-filter-json](https://github.com/logstash-plugins/logstash-filter-json) |
| [json_encode](../19-Filter-plugins/json_encode.md)           | 序列化JSON的字段                                             | [logstash-filter-json_encode](https://github.com/logstash-plugins/logstash-filter-json_encode) |
| [kv](../19-Filter-plugins/kv.md)                             | 解析键值对                                                   | [logstash-filter-kv](https://github.com/logstash-plugins/logstash-filter-kv) |
| [memcached](../19-Filter-plugins/memcached.md)               | 与外部Memcache中的数据集成                                   | [logstash-filter-memcached](https://github.com/logstash-plugins/logstash-filter-memcached) |
| [metricize](../19-Filter-plugins/metricize.md)               | 采用包含多个指标的复杂事件，并将这些指标拆分为多个事件，每个事件都包含一个指标 | [logstash-filter-metricize](https://github.com/logstash-plugins/logstash-filter-metricize) |
| [metrics](../19-Filter-plugins/metrics.md)                   | 聚合指标                                                     | [logstash-filter-metrics](https://github.com/logstash-plugins/logstash-filter-metrics) |
| [**mutate**](../19-Filter-plugins/mutate.md)                 | 修改字段                                                     | [logstash-filter-mutate](https://github.com/logstash-plugins/logstash-filter-mutate) |
| [**prune**](../19-Filter-plugins/prune.md)                       | 基于字段的白名单或黑名单修剪事件数据                         | [logstash-filter-prune](https://github.com/logstash-plugins/logstash-filter-prune) |
| [range](../19-Filter-plugins/range.md)                       | 根据给定的容量或长度限制校验指定字段                         | [logstash-filter-range](https://github.com/logstash-plugins/logstash-filter-range) |
| [ruby](../19-Filter-plugins/ruby.md)                         | 执行任意ruby代码                                             | [logstash-filter-ruby](https://github.com/logstash-plugins/logstash-filter-ruby) |
| [sleep](../19-Filter-plugins/sleep.md)                       | 休眠指定的时间                                               | [logstash-filter-sleep](https://github.com/logstash-plugins/logstash-filter-sleep) |
| [split](../19-Filter-plugins/split.md)                       | 分割多行消息为不同的事件                                     | [logstash-filter-split](https://github.com/logstash-plugins/logstash-filter-split) |
| [syslog_pri](../19-Filter-plugins/syslog_pri.md)             | 解析 `syslog` 消息的 `PRI`（priority）字段                   | [logstash-filter-syslog_pri](https://github.com/logstash-plugins/logstash-filter-syslog_pri) |
| [**throttle**](../19-Filter-plugins/throttle.md)                 | 限制事件的数量                                               | [logstash-filter-throttle](https://github.com/logstash-plugins/logstash-filter-throttle) |
| [tld](../19-Filter-plugins/tld.md)                           | 替换默认消息字段为任意您在配置中指定的内容                   | [logstash-filter-tld](https://github.com/logstash-plugins/logstash-filter-tld) |
| [translate](../19-Filter-plugins/translate.md)               | 基于一个hash或YAML文件替换字段内容                           | [logstash-filter-translate](https://github.com/logstash-plugins/logstash-filter-translate) |
| [truncate](../19-Filter-plugins/truncate.md)                 | 当字段长度超过指定长度时，截断字段                           | [logstash-filter-truncate](https://github.com/logstash-plugins/logstash-filter-truncate) |
| [urldecode](../19-Filter-plugins/urldecode.md)               | 使URL编码的字段进行解码                                      | [logstash-filter-urldecode](https://github.com/logstash-plugins/logstash-filter-urldecode) |
| [useragent](../19-Filter-plugins/useragent.md)               | 将客户端字符串解析到字段中                                   | [logstash-filter-useragent](https://github.com/logstash-plugins/logstash-filter-useragent) |
| [**uuid**](../19-Filter-plugins/uuid.md)                     | 为事件增加一个UUID                                           | [logstash-filter-uuid](https://github.com/logstash-plugins/logstash-filter-uuid) |
| [xml](../19-Filter-plugins/xml.md)                           | 解析XML到字段                                                | [logstash-filter-xml](https://github.com/logstash-plugins/logstash-filter-xml) |

