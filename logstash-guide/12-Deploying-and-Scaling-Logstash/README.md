# 第12章-部署和集群扩展

Elastic Stack有大量应用场景，从操作日志和指标分析到企业和应用程序搜索。确保您的数据可扩展，持久且安全地传输到Elasticsearch非常重要，尤其是对于重要任务的环境。

本文档的目标是说明Logstash最常见的架构，以及如何在部署增长时进行有效扩展。重点将放在运营日志、指标和安全分析用例上，因为它们往往需要更大规模的部署。此处提供的部署和扩展建议可能会根据您自己的要求而有所不同。

### 入门

对于初次使用的用户，如果您只想基于日志文件来掌握Elastic Stack的强大功能，我们建议您尝试使用 [Filebeat模块](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-modules-overview.html)。 Filebeat模块使您能够在几分钟内快速收集，解析和索引流行的日志类型，并查看预定义的Kibana仪表板。 [Metricbeat模块](https://www.elastic.co/guide/en/beats/metricbeat/6.7/metricbeat-modules.html) 提供类似的功能，并具有指标数据。在这种情况下，Beats会将数据直接发送到Elasticsearch，其中 [Ingest Nodes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ingest.html) 将处理和索引您的数据。

![deploy1](../source/images/ch-12/deploy1.png)

#### 介绍

将Logstash集成到您的架构中的主要好处是什么？

- 根据采集峰值进行扩展 -  Logstash具有基于磁盘的自适应缓冲系统，可吸收传入的吞吐量，从而减轻背压
- 从各种数据源（如数据库，S3或消息队列）中摄取
- 将数据发送到多个目标，如S3，HDFS或写入文件
- 使用条件数据流逻辑构建更复杂的处理流水线

### 采集扩展

Beats和Logstash使采集变得很精彩。它们共同提供了可扩展且具有弹性的全面解决方案。值得期待的有：

- 水平可扩展性，高可用性和可变负载处理
- 消息持久性，至少一次交付保证
- 具有身份验证和有线加密的端到端安全传输

#### Beats和Logstash

Beats遍历数千个边缘主机服务器，收集，并将日志发送到Logstash。 Logstash充当数据统一和丰富的集中式流媒体引擎。 [Beats](../17-Input-plugins/beats.md) 公开了一个安全的，基于确认的端点，以便将数据发送到Logstash。

![deploy2](../source/images/ch-12/deploy2.png)

> **注意：**
> 强烈建议启用持久化队列，以上架构已经假定它们为启用状态。我们建议您查看 [持久化队列](../10-Data-Resiliency/Persistent-Queues.md) 文档以了解功能优势，以及有关弹性的更多详细信息。

#### 扩展

Logstash是可水平扩展的，可以形成运行相同管道的节点组。 Logstash的自适应缓冲功能通过可变吞吐量负载，可以实现流畅的流式传输。如果Logstash层成为摄取瓶颈，只需添加更多节点即可向外扩展。以下是一些常规建议：

- Beats应该在一组Logstash节点之间进行 [负载均衡](https://www.elastic.co/guide/en/beats/filebeat/6.7/load-balancing.html)。
- 建议至少使用两个Logstash节点以实现高可用性。
- 通常每个Logstash节点只部署一个Beats输入，但每个Logstash节点也可以部署多个Beats输入，以便使不同数据源的端点相互独立。

#### 弹性

在此摄取流程中使用 [Filebeat](https://www.elastic.co/products/beats/filebeat) 或 [Winlogbeat](https://www.elastic.co/products/beats/winlogbeat) 进行日志收集时，可以保证**至少一次处理**。从Filebeat或Winlogbeat到Logstash，以及从Logstash到Elasticsearch的通信协议都是同步的，并且支持确认，其他Beats则尚未支持确认。

Logstash持久队列提供跨节点故障的保护。对于Logstash中的磁盘级弹性，确保磁盘冗余非常重要。对于内部部署，建议您配置RAID。在云或容器化环境中运行时，建议您使用具有反映数据SLA的复制策略的永久磁盘。

> **注意：**
> 确保 `queue.checkpoint.writes: 1`设置，以保证至少一次处理。有关更多详细信息，请参阅 [持久化队列的持久性](../10-Data-Resiliency/Persistent-Queues.md#控制持久性)。

#### 处理

Logstash通常会使用 [grok](../19-Filter-plugins/grok.md) 或 [dissect](../19-Filter-plugins/dissect.md) 提取字段，增加 [地理信息](../19-Filter-plugins/geoip.md)，并可以使用 [文件](../19-Filter-plugins/translate.md)，[数据库](../19-Filter-plugins/jdbc_streaming.md)或[Elasticsearch](../19-Filter-plugins/elasticsearch.md)查找数据集进一步丰富事件。请注意，处理的复杂度会影响整体吞吐量和CPU利用率。请务必查看其他可用的[过滤器插件](../19-Filter-plugins/README.md)。

#### 安全传输

整个传递链中都提供企业级安全性。

- 对于从 [Beats到Logstash](https://www.elastic.co/guide/en/beats/filebeat/6.7/configuring-ssl-logstash.html) 以及从 [Logstash到Elasticsearch](../06-Configuring-Logstash/X-Pack-security.md) 的传输，建议使用通讯加密。
- 与Elasticsearch通信时有很多安全选项，包括基本身份验证，TLS，PKI，LDAP，AD和其他自定义域。要启用Elasticsearch安全性，请参阅 [X-Pack文档](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/xpack-security.html)。

#### 监控

运行Logstash 5.2或更高版本时，[监控UI](https://www.elastic.co/products/stack/monitoring)提供对部署指标的详细展示，有助于在扩展时观察性能并缓解瓶颈。监控是基本许可下的X-Pack功能，因此可以免费使用。要开始使用，请参阅 [监控Logstash](../14-Monitoring-Logstash/README.md)。

如果选择外部监控，则有[监控API](../15-Monitoring-APIs/README.md)可获取实时性能快照。

### 增加其他流行源

用户可能有其他收集日志记录数据的机制，并且很容易将它们集成，并集中到Elastic stack中。让我们来看几个场景：

![deploy3](../source/images/ch-12/deploy3.png)

#### TCP，UDP和HTTP协议

TCP，UDP和HTTP协议是将数据提供给Logstash的常用方法。 Logstash可以使用相应的[TCP](../17-Input-plugins/tcp.md)，[UDP](../17-Input-plugins/udp.md)和[HTTP](../17-Input-plugins/http.md)输入插件发布监听器。下面列举的数据源通常通过这三种协议之一摄取。

> **注意：**
> TCP协议不支持应用程序级别的确认，因此连接问题可能导致数据丢失。

对于高可用性方案，应添加第三方硬件或软件负载平衡器（如HAProxy）以扇形输出到一组Logstash节点的流量。

#### 网络和安全数据

虽然Beats可能已经满足您的数据提取场景，但网络和安全数据集有多种形式。让我们来谈谈其他几个采集点：

- Network wire data - 使用 [Packetbeat](https://www.elastic.co/cn/products/beats/packetbeat) 收集和分析网络流量。
- Netflow v5/v9/v10  -  Logstash使用 [Netflow编解码器](../20-Coder-plugins/netflow.md)了解Netflow/IPFIX出口的数据。
- Nmap  -  Logstash接受并使用Nmap编解码器解析Nmap XML数据。
- SNMP trap -  Logstash具有本机 [SNMP trap输入](../17-Input-plugins/snmptrap.md)。
- CEF  -  Logstash接受并使用 [CEF编解码器](../20-Coder-plugins/cef.md) 从Arcsight SmartConnectors等系统解析CEF数据。有关详细信息，请参阅此 [博客系列](https://www.elastic.co/blog/integrating-elasticsearch-with-arcsight-siem-part-1)。

#### 集中式Syslog服务器

现有的syslog服务器技术（如rsyslog和syslog-ng）通常将syslog发送到Logstash TCP或UDP端点以进行采集、处理和持久化。如果数据格式符合RFC3164，则可以将其直接提供给Logstash [syslog输入](../17-Input-plugins/syslog.md)。

#### 基础设施和应用的数据以及IoT

可以使用 [Metricbeat](https://www.elastic.co/cn/products/beats/metricbeat) 收集基础设施和应用程序性能指标，但应用程序也可以将Webhooks发送到Logstash HTTP输入，或者使用 [http_poller输入插件](../17-Input-plugins/http_poller.md) 从HTTP端点轮询性能指标。

对于使用log4j2进行日志记录的应用程序，建议使用SocketAppender将JSON发送到Logstash TCP输入。或者，log4j2也可以使用Filebeat监听文件以进行收集。建议不要使用log4j1 SocketAppender。

像Rasberry Pis，智能手机和互联车辆这样的物联网设备通常通过其中一种协议发送遥测数据。

### 集成消息队列

如果您正在利用消息队列技术作为现有基础架构的一部分，那么将这些数据放入Elastic Stack很容易。对于使用Redis或RabbitMQ等外部排队层的现有用户来说，只是为了使用Logstash进行数据缓冲，建议使用Logstash持久队列而不是外部排队层。通过消除采集架构中不必要的复杂层，这将有助于整体轻松管理。

对于想要集成现有Kafka部署数据或需要短暂存储的用户，Kafka可以作为Beats持久存储的数据中心，提供给Logstash节点可以使用。

![deploy4](../source/images/ch-12/deploy4.png)

其他TCP，UDP和HTTP源可以使用Logstash持久保存到Kafka，以代替负载均衡器实现高可用性。Logstash节点组可以使用Kafka输入的主题进行消费，以进一步转换和丰富传输中的数据。

#### 弹性和恢复

当Logstash从Kafka消费时，应启用持久队列，并添加传输弹性，以减少Logstash节点故障期间重新处理的需求。在此上下文中，建议使用默认的持久队列磁盘分配大小 `queue.max_bytes: 1GB`。

如果将Kafka配置为长时间保留数据，则可以在灾难恢复和协调的情况下从Kafka重新处理数据。

#### 集成其他消息队列

虽然不需要额外的排队层，但Logstash可以使用各种其他消息队列技术，如 [RabbitMQ](../17-Input-plugins/rabbitmq.md) 和 [Redis](../17-Input-plugins/redis.md)。它还支持从[Pub/Sub](../17-Input-plugins/google_pubsub.md)，[Kinesis](../17-Input-plugins/kinesis.md) 和 [SQS](../17-Input-plugins/sqs.md) 等托管排队服务中提取。
