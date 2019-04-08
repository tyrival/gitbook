## X-Pack监控概述

本节介绍Logstash，包括对其内部部件的高级解释。 Logstash的X-Pack监控包含：

- [采集](#采集)
- [输出](#输出)

这些部分是在启用Logstash的X-Pack监控时创建的，它们位于专用监视默认Logstash管道之外。此配置意味着所有数据和处理对普通Logstash处理的影响最小。作为单独管道中存在的第二个好处，可以重用现有的Logstash功能（例如elasticsearch输出）以从其重试策略中受益。

> **注意：**
> 由Logstash的X-Pack监视使用的elasticsearch输出，仅通过 `logstash.yml` 进行配置。它未使用Logstash配置中的任何内容，这些配置也可以使用自己独立的elasticsearch输出。

与Logstash的X-Pack监视配置为一起使用的Elasticsearch集群应该是生产集群。此配置使Elasticsearch生产集群能够将元数据（例如，其集群UUID）添加到Logstash监控数据，然后将其路由到监控集群。有关典型监视体系结构的详细信息，请参阅 [监控工作原理](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/how-monitoring-works.html)。

### 采集

采集，顾名思义是收集东西。在Logstash的X-Pack监控中，收集器只是输入，与普通Logstash配置提供 [输入](../03-How-Logstash-Works/README.md) 的方式相同。

与Elasticsearch的X-Pack监控一样，每个收集器都可以创建零个或多个监控文档。当前实现时，每个Logstash节点都运行两种类型的采集器：一个用于节点统计，一个用于管道统计。

| 采集器     | 数据类型         | 描述                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| Node Stats | `logstash_stats` | 收集正在运行的节点的详细信息，例如内存利用率和CPU使用率（例如，`GET /_stats`）。这将在启用了X-Pack监视的每个Logstash节点上运行。一个常见的失败情况是Logstash目录被复制，其中包含 `path.data` 目录（默认为 `./data`），它会复制Logstash节点的持久UUID。结果，通常看起来一个或多个Logstash节点未能收集监视数据，而实际上它们都是错误地报告为相同的Logstash节点。仅在升级Logstash时重新使用 `path.data` 目录，以便升级后的节点替换以前的版本。 |
| 管道状态   | `logstash_state` | 收集有关节点运行管道的详细信息，该管道为监控管道UI提供支持。 |

收集器运行收集的时间间隔默认为10秒（`10s`）。单个收集器的故障不会影响任何其他收集器。每个收集器作为普通的Logstash输入，在其隔离的监视管道中创建单独的Logstash事件。 Logstash输出然后发送数据。

可以动态配置收集时间间隔，也可以禁用数据收集。有关收集器的配置选项的详细信息，请参阅 [采集监控设置](../06-Configuring-Logstash/X-Pack-monitoring.md#采集监控设置)。

> **警告：**
> 与Elasticsearch和Kibana的X-Pack监视不同，Logstash上没有 `xpack.monitoring.collection.enabled` 设置。您必须使用 `xpack.monitoring.enabled` 设置来启用和禁用数据收集。

如果Kibana的监控图表中存在间隙，通常是因为收集器发生故障或监控集群未收到数据（例如，它正在重新启动）。如果收集器发生故障，则尝试执行收集的节点上应存在已记录的错误。

### 输出

与所有Logstash管道一样，专用监视管道的目的是将事件发送到输出阶段。对于Logstash的X-Pack监视，输出始终是 `elasticsearch` 输出。但是，与普通的Logstash管道不同，输出是通过在 `logstash.yml` 设置文件中的 `xpack.monitoring.elasticsearch.*` 配置的。

除了其独特的配置方式之外，此 `elasticsearch` 输出的行为与所有 `elasticsearch` 输出相同，包括当输出存在问题时暂停数据收集的能力。

> **重要：**
> 所有Logstash节点共享相同的设置至关重要。否则，监控数据可能以不同的方式路由，或路由到不同的位置。

#### 默认配置

如果Logstash节点未显式定义X-Pack监视输出设置，则使用以下默认配置：

```yaml
xpack.monitoring.elasticsearch.hosts: [ "http://localhost:9200" ]
```

Logstash的X-Pack监视生成的所有数据，都使用 `.monitoring-logstash` 模板在监视集群中编制索引，该模板由Elasticsearch中的 [导出](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/es-monitoring-exporters.html) 程序管理。

如果您正在使用启用了X-Pack安全性的群集，则需要执行额外步骤才能正确配置Logstash。有关更多信息，请参阅 [X-Pack监控](../06-Configuring-Logstash/X-Pack-monitoring.md)。

> **重要：**
> 在讨论与 `elasticsearch` 输出相关的安全性时，必须记住所有用户都在生产集群上进行管理，生产集群在 `xpack.monitoring.elasticsearch.hosts` 中设置。当您从开发环境迁移到具有专用监视集群的生产环境时，这一点尤为重要。

有关输出的配置选项的详细信息，请参阅 [采集监控设置](../06-Configuring-Logstash/X-Pack-monitoring.md#采集监控设置)。
