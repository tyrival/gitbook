## kafka

- 插件版本：v8.3.1
- 发布于：2018-12-19
- [更新日志](https://github.com/logstash-plugins/logstash-input-kafka/blob/v8.3.1/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/input-kafka-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-input-kafka) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

此输入将读取Kafka主题中的事件。

此插件使用Kafka Client 2.1.0。有关兼容性，请参阅官方 [Kafka兼容性参考](https://cwiki.apache.org/confluence/display/KAFKA/Compatibility+Matrix)。如果链接的兼容性wiki不是最新的，请联系Kafka支持/社区以确认兼容性。

如果您需要此插件中尚未提供的功能（包括客户端版本升级），请提交有关您所需内容的详细信息。

此输入支持通过以下方式连接到Kafka：

- SSL（需要插件版本3.0.0或更高版本）
- Kerberos SASL（需要插件版本5.1.0或更高版本）

默认情况下，安全性已禁用，可以根据需要打开。

Logstash Kafka消费者处理组管理，并使用默认的偏移管理策略消费Kafka主题。

Logstash实例默认形成一个逻辑组来订阅Kafka主题，每个Logstash Kafka消费者可以运行多个线程，来提高读取吞吐量。或者，您可以使用相同的 `group_id` 运行多个Logstash实例，以在物理机之间分配负载。主题中的消息将分发到具有相同 `group_id` 的所有Logstash实例。

理想情况下，您应该拥有与分区数量一样多的线程以实现完美平衡——线程多于分区意味着某些线程将处于空闲状态

有关更多信息，请参阅 http://kafka.apache.org/documentation.html#theconsumer

Kafka消费者配置：http://kafka.apache.org/documentation.html#consumerconfigs

### 元数据字段

来自Kafka代理的以下元数据添加在 `[@metadata]` 字段下：

- `[@metadata] [kafka] [topic]`：消息被消费的原始Kafka主题。
- `[@metadata] [kafka] [consumer_group]`：消费者群体
- `[@metadata] [kafka] [partition]`：此消息的分区信息。
- `[@metadata] [kafka] [offset]`：此消息的原始记录偏移量。
- `[@metadata] [kafka] [key]`：记录密钥，如果有的话。
- `[@metadata] [kafka] [timestamp]`：记录中的时间戳。根据您的代理配置，这可以是创建记录时（默认），也可以是中间件接收时。有关属性 log.message.timestamp.type 的更多信息，请访问 https://kafka.apache.org/10/documentation.html#brokerconfigs

请注意，`@metadata` 字段不是事件输出的一部分。如果您需要将这些信息插入到原始事件中，则必须使用 `mutate` 过滤器手动将所需字段复制到 `event` 中。

### Kafka输入配置选项

此插件支持这些配置选项以及稍后描述的 [通用配置项](#通用配置项)。

> **注意：**
>
> 其中一些选项映射到Kafka选项。有关更多详细信息，请参阅 https://kafka.apache.org/documentation。

| 设置                                                         | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`auto_commit_interval_ms`](#autocommitintervalms)        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`auto_offset_reset`](#autooffsetreset)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`bootstrap_servers`](#bootstrapservers)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`check_crcs`](#checkcrcs)                                  | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`client_id`](#clientid)                                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`connections_max_idle_ms`](#connectionsmaxidlems)        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`consumer_threads`](#consumerthreads)                      | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`decorate_events`](#decorateevents)                        | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`enable_auto_commit`](#enableautocommit)                  | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`exclude_internal_topics`](#excludeinternaltopics)        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`fetch_max_bytes`](#fetchmaxbytes)                        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`fetch_max_wait_ms`](#fetchmaxwaitms)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`fetch_min_bytes`](#fetchminbytes)                        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`group_id`](#groupid)                                      | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`heartbeat_interval_ms`](#heartbeatintervalms)            | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`jaas_path`](#jaaspath)                                    | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`kerberos_config`](#kerberosconfig)                        | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`key_deserializer_class`](#keydeserializerclass)          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`max_partition_fetch_bytes`](#maxpartitionfetchbytes)    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`max_poll_interval_ms`](#maxpollintervalms)              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`max_poll_records`](#maxpollrecords)                      | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`metadata_max_age_ms`](#metadatamaxagems)                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`partition_assignment_strategy`](#partitionassignmentstrategy) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`poll_timeout_ms`](#polltimeoutms)                        | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`receive_buffer_bytes`](#receivebufferbytes)              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`reconnect_backoff_ms`](#reconnectbackoffms)              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`request_timeout_ms`](#requesttimeoutms)                  | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`retry_backoff_ms`](#retrybackoffms)                      | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`sasl_kerberos_service_name`](#saslkerberosservicename)  | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`sasl_mechanism`](#saslmechanism)                          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`security_protocol`](#securityprotocol)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，可选项有`["PLAINTEXT", "SSL", "SASL_PLAINTEXT", "SASL_SSL"]` | 否   |
| [`send_buffer_bytes`](#sendbufferbytes)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`session_timeout_ms`](#sessiontimeoutms)                  | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`ssl_endpoint_identification_algorithm`](#sslendpointidentificationalgorithm) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`ssl_key_password`](#sslkeypassword)                      | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`ssl_keystore_location`](#sslkeystorelocation)            | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`ssl_keystore_password`](#sslkeystorepassword)            | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`ssl_keystore_type`](#sslkeystoretype)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`ssl_truststore_location`](#ssltruststorelocation)        | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`ssl_truststore_password`](#ssltruststorepassword)        | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`ssl_truststore_type`](#ssltruststoretype)                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`topics`](#topics)                                          | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`topics_pattern`](#topicspattern)                          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`value_deserializer_class`](#valuedeserializerclass)      | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### auto_commit_interval_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"5000"`

消费者抵消提交给Kafka的频率（以毫秒为单位）。

##### auto_offset_reset

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

当Kafka中没有初始偏移量或偏移量超出范围时该怎么办：

- earliest：自动将偏移重置为最早的偏移量
- latest：自动将偏移重置为最新偏移
- none：如果没有找到消费者组的先前偏移量，则向消费者抛出异常
- anything else：向消费者抛出异常。

##### bootstrap_servers

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"localhost:9092"`

用于建立与群集的初始连接的Kafka实例的URL列表。此列表应采用 `host1:port1, host2:port2` 的形式。这些URL仅用于初始连接以发现完整的集群成员资格（可能会动态更改），因此此列表不需要包含完整的服务器集（由于部分服务器可能不可用，你可能需要填写多个服务器地址）。

##### check_crcs

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

自动检查消耗的记录的CRC32。这可确保消息不会发生线上或磁盘损坏。此检查会增加一些开销，因此在寻求极端性能的情况下可能会被禁用。

##### client_id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"logstash"`

发出请求时传递给服务器的id字符串。这样做的目的是通过包含应用程序名，来追踪超出 ip/port 的请求源。

##### connections_max_idle_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

在此配置指定的毫秒数后关闭空闲连接。

##### consumer_threads

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为1

理想情况下，您应该拥有与分区数量一样多的线程以实现完美平衡 - 线程多于分区意味着某些线程将处于空闲状态

##### decorate_events

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

此选项可以向事件添加主题、消息大小等Kafka元数据。这会将名为 `kafka` 的字段添加到包含以下属性的logstash事件中：

- `topic`：此消息与之关联的主题
- `consumer_group`：用于读取此事件的使用者组
- `partition`：与此消息关联的分区
- `offset`：与此消息关联的分区的偏移量
- `key`：包含消息密钥的ByteBuffer

##### enable_auto_commit

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"true"`

如果为true，则定期向Kafka提交消费者已经返回的消息的偏移量。当进程失败时，将使用此已提交的偏移量作为消耗开始的位置。

##### exclude_internal_topics

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

是否应将内部主题（如偏移量）的记录暴露给消费者。如果设置为true，则从内部主题接收记录的唯一方法是订阅它。

##### fetch_max_bytes

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

服务器应为获取请求返回的最大数据量。这不是绝对最大值，如果获取的第一个非空分区中的第一条消息大于此值，则仍将返回消息以确保消费者可以取得进展。

##### fetch_max_wait_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

如果没有足够的数据立即满足 `fetch_min_bytes`，则在回答获取请求之前服务器将阻塞的最长时间。这应该小于或等于 `poll_timeout_ms` 中使用的超时

##### fetch_min_bytes

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

服务器应为获取请求返回的最小数据量。如果数据不足，请求将在回答请求之前等待那么多数据累积。

##### group_id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"logstash"`

此使用者所属的组的标识符。消费者群体是一个恰好由多个处理器组成的逻辑订阅者。主题中的消息将分发到具有相同 `group_id` 的所有Logstash实例

##### heartbeat_interval_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

心跳与消费者协调员之间的预期时间。心跳用于确保消费者的会话保持活动状态，并在新消费者加入或离开群组时促进重新平衡。该值必须设置为低于 `session.timeout.ms`，但通常应设置为不高于该值的1/3。它可以调整得更低，以控制正常重新平衡的预期时间。

##### jaas_path

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

Java身份验证和授权服务（JAAS）API为Kafka提供用户身份验证和授权服务。此设置提供JAAS文件的路径。 Kafka客户端的示例JAAS文件：

```java
KafkaClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useTicketCache=true
  renewTicket=true
  serviceName="kafka";
  };
```

请注意，在配置文件中指定 `jaas_path` 和 `kerberos_config` 会将这些添加到全局JVM系统属性中。这意味着如果您有多个Kafka输入，则所有这些输入将共享相同的 `jaas_path` 和 `kerberos_config`。如果不希望这样，则必须在不同的JVM实例上运行单独的Logstash实例。

##### kerberos_config

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

kerberos配置文件的可选路径。这是krb5.conf样式，详见https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/krb5_conf.html

##### key_deserializer_class

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"org.apache.kafka.common.serialization.StringDeserializer"`

用于反序列化记录密钥的Java类

##### max_partition_fetch_bytes

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

服务器将返回的每个分区的最大数据量。用于请求的最大总内存为 `#partitions * max.partition.fetch.bytes`。此大小必须至少与服务器允许的最大消息数一样大，否则生产者可以发送大于消费者可以获取的消息。如果发生这种情况，消费者可能会遇到尝试在某个分区上获取大量消息的问题。

##### max_poll_interval_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

使用消费者组管理时 poll() 调用之间的最大延迟。这为消费者在获取更多记录之前可以闲置的时间量设置了上限。如果在此超时到期之前未调用 poll()，则认为使用者失败，并且该组将重新平衡以便将分区重新分配给另一个成员。配置 `request_timeout_ms` 的值必须始终大于 `max_poll_interval_ms`

##### max_poll_records

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

一次调用 poll() 时返回的最大记录数。

##### metadata_max_age_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

即使没有看到任何分区更改以主动发现任何新的代理或分区，也要强制刷新元数据的时间段（以毫秒为单位）

##### partition_assignment_strategy

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

客户端用于使用者实例之间，分配分区所有权的分区分配策略的类名。映射到Kafka  `partition.assignment.strategy` 设置，默认为 `org.apache.kafka.clients.consumer.RangeAssignor`。

##### poll_timeout_ms

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `100`

kafka消费者将等待从主题接收新消息的时间

##### receive_buffer_bytes

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

读取数据时要使用的TCP接收缓冲区（SO_RCVBUF）的大小。

##### reconnect_backoff_m

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

尝试重新连接到给定主机之前等待的时间。这避免了在紧密循环中重复连接到主机。此退避适用于消费者向代理发送的所有请求。

##### request_timeout_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

配置控制客户端等待请求响应的最长时间。如果在超时之前没有收到响应，则客户端将在必要时重新发送请求，或者如果重试耗尽则请求失败。

##### retry_backoff_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

对指定主题分区的提取请求之后，重试前的等待时间。这避免了在紧密循环中重复取出和失败。

##### sasl_kerberos_service_name

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

Kafka代理运行的Kerberos主体名称。这可以在Kafka的JAAS配置或Kafka的配置中定义。

##### sasl_mechanism

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"GSSAPI"`

用于客户端连接的 [SASL机制](http://kafka.apache.org/documentation.html#security_sasl)。这可以是安全提供者可用的任何机制。 GSSAPI是默认机制。

##### security_protocol

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，值包含：`PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL`
- 默认值为 `PLAINTEXT`

要使用的安全协议，可以是PLAINTEXT，SSL，SASL_PLAINTEXT，SASL_SSL

##### send_buffer_bytes

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

发送数据时使用的TCP发送缓冲区（SO_SNDBUF）的大小

##### session_timeout_ms

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

超时之后，如果未调用 `poll_timeout_ms`，则将使用者标记为dead，并为 `group_id` 标识的组触发重新平衡操作

##### ssl_endpoint_identification_algorithm

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"https"`

端点识别算法默认为 `"https"`。设置为空字符串 `""` 以禁用端点验证

##### ssl_key_password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

密钥库文件中私钥的密码。

##### ssl_keystore_location

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果需要客户端身份验证，则此设置存储密钥库路径。

##### ssl_keystore_password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

如果需要客户端身份验证，则此设置存储密钥库密码

##### ssl_keystore_type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

密钥库类型。

##### ssl_truststore_location

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

用于验证Kafka代理证书的JKS信任库路径。

##### ssl_truststore_password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

信任库密码

##### ssl_truststore_type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

信任库类型。

##### topics

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `["logstash"]`

要订阅的主题列表，默认为["logstash"]。

##### topics_pattern

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

要订阅的主题正则表达式模式。使用此配置时，将忽略主题配置。

##### value_deserializer_class

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"org.apache.kafka.common.serialization.StringDeserializer"`

Java类用于反序列化记录的值

### 通用配置项

所有输入插件都支持以下配置选项：

| 设置                              | 输入类型                                                     | 必须 |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| [`add_field`](#addfield)         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`codec`](#codec)                 | [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#codec) | 否   |
| [`enable_metric`](#enablemetric) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`id`](#id)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`tags`](#tags)                   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`type`](#type)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

#### 详情

##### add_field

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为 `{}`

向事件添加字段

##### codec

- 值类型是 [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#codec)
- 默认值是 `"plain"`

用于输入数据的编解码器。输入编解码器是一种在输入之前解码数据的便捷方法，无需在Logstash管道中使用单独的过滤器。

##### enable_metric

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

默认情况下，禁用或启用此特定插件实例的指标记录，我们会记录所有的可用指标，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

为插件配置添加唯一 `ID`。如果未指定ID，Logstash将生成一个ID。强烈建议在配置中设置此ID。当您有两个或更多相同类型的插件时，这尤其有用，例如，如果您有2个beat输入，添加命名ID将有助于使用API监视Logstash。

```sh
input {
  kafka {
    id => "my_plugin_id"
  }
}
```

##### tags

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 此设置没有默认值。

为您的活动添加任意数量的任意标签。

这有助于后续处理。

##### type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

将 `type` 字段添加到此输入处理的所有事件。

类型主要用于过滤器激活。

类型存储为事件本身的一部分，因此您也可以使用该类型在Kibana中搜索它。

如果您尝试在已有事件的事件上设置类型（例如，当您将事件从发运人发送到索引器时），则新输入将不会覆盖现有类型。发运人设置的类型即使在发送到另一个Logstash服务器时，仍会保留。
