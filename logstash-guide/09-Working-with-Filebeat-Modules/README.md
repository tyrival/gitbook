# 第9章-Filebeat

Filebeat与预构建的 [模块](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-modules.html) 打包在一起，其中包含收集，解析，丰富和可视化各种日志文件格式的数据所需的配置。 每个Filebeat模块由一个或多个包含摄取节点管道的文件集，Elasticsearch模板，Filebeat输入配置和Kibana仪表板组成。

您可以将Filebeat模块与Logstash一起使用，但您需要进行一些额外的设置。 最简单的方法是 [使用采集管道解析](../09-Working-with-Filebeat-Modules/Use-ingest-pipelines-for-parsing.md)。 如果摄取管道不符合您的要求，您可以 [使用Logstash管道解析](../09-Working-with-Filebeat-Modules/Use-Logstash-pipelines-for-parsing.md) 而不是摄取管道。

这两种方法都允许您使用Filebeat配置索引匹配和仪表板，只要您维护索引和仪表板所需的字段结构即可。

