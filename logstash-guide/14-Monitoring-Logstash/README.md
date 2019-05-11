# 第14章-监控

运行Logstash时，它会自动捕获和监视Logstash的运行状况和性能度量。

Logstash收集的度量包括：

- Logstash节点信息，如管道设置，操作系统信息和JVM信息。
- 插件信息，包括已安装插件的列表。
- 节点统计信息，如JVM统计信息，进程统计信息，事件相关统计信息和管道运行时统计信息。
- 热线程。

您可以使用Logstash提供的基本 [监控API](../15-Monitoring-APIs/README.md) 来检索这些性能度量。默认情况下这些API可用，无需任何额外配置。

或者，您可以配置 [X-Pack监视](../06-Configuring-Logstash/X-Pack-monitoring.md) 以将数据发送到监控集群。

> **注意：**
> 监控是基本许可下的X-Pack功能，因此可以免费使用。

您可以使用X-Pack中的 [监控UI](../14-Monitoring-Logstash/Monitoring-UI.md) 来查看度量，并深入了解Logstash的运行方式。

X-Pack中的 [管道视图UI](../14-Monitoring-Logstash/Pipeline-Viewer-UI.md) 提供了对复杂管道配置的行为和性能的可视化。它显示了整个流水线拓扑、数据流和分支逻辑的图形表示，覆盖了视图中每个插件的重要度量，如每秒事件数。

本文档重点介绍Logstash中的X-Pack监视基础结构和设置。有关监视Elastic堆栈（包括Elasticsearch和Kibana）的介绍，请参阅 [监视Elastic stack](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/xpack-monitoring.html)。
