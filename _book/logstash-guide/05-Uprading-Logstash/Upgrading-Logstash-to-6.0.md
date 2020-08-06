## 升级Logstash到6.0

升级Logstash之前，请阅读 [第26章-重大变更](../26-Breaking-Changes/README.md)。

如要在Elastic Stack中安装Logstash和其他组件，请参阅 [Elastic Stack安装和升级文档](https://www.elastic.co/guide/en/elastic-stack/6.7/index.html)。

### Logstash 6.0.0 `document_type`写入Elasticsearch 6.x时的相关问题

我们在此警示用户，Logstash 6.0.0写入Elasticsearch 6.0+群集时可能导致错误的行为。当Logstash尝试建立事件索引时，如果遇到多个`type`值，Logstash会发生索引错误。这些错误类似于如下示例消息：

```shell
[2017-11-21T14:26:01,991][WARN ][logstash.outputs.elasticsearch] Could not index
event to Elasticsearch.{:status=>400,  :response=>{"error"=>{"reason"=>"Rejecting
mapping update to [myindex] as the final mapping would have more than 1 type:
[type1, type2]"}}}}
```

Logstash从以下来源接收数据时，用户可能会遇到此错误：

- 多种类型的节拍
- Filebeat的实例聚合了不同类型的多个文件
- 多个Logstash输入指定不同的 `type` 值

要在Logstash 6.0.0中解决此问题，请将设置 `document_type => doc` 添加到Elasticsearch输出配置。我们将在新版本的Logstash中发布补丁以尽快解决此问题。

Logstash历史版本中一直使用 `type` 字段的值来默认设置Elasticsearch类型。 Elasticsearch 6.0单个索引 [不再支持多个类型](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/removal-of-types.html)。这就是此新功能将仅可用于即将发布修复补丁的Elasticsearch 6.0+集群。

请继续向下阅读，以获取Logstash和Elasticsearch 6.0文档类型的更多信息。

### 处理Elasticsearch 6.0+的文档类型

从Elasticsearch 6.0开始，文档类型 [即将推出](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/removal-of-types.html)，每个索引只支持一种映射类型。对于Logstash用户，这意味着在文档内部将从使用文档类型转为使用 `type` 字段。效果相同，但用法略有不同。这可能意味着需要重新配置现有的Kibana仪表板以使用 `type` 字段而不是文档类型。

如果您在Logstash中使用默认映射模板，则需要升级映射模板。为此，在迁移Elasticsearch到6.0版之后，必须使用6.x模板覆盖现有模板。这需要为所有已配置的Elasticsearch输出都指定以下设置： `template_overwrite => true`。

### 何时升级

全新安装开始时，应从Elastic Stack中选择相同版本。

Elasticsearch 6.0不需要Logstash 6.0。 Elasticsearch 6.0集群将通过默认的HTTP通信层，从Logstash 5.x实例接收数据。这为确定何时相对于Elasticsearch升级升级Logstash提供了一些灵活性。您可能不方便将它们一起升级，所以只需首先升级Elasticsearch，而不需要同时完成它们。

您应该及时升级以获得Logstash 6.0带来的性能改进，这样做的方式对您的应用环境最有意义。

### 何时不升级

如果您需要的任何Logstash插件与Logstash 6.0不兼容，那么您应该等到它准备好再升级。

虽然我们努力确保兼容性，但Logstash 6.0并不完全向后兼容。如Elastic Stack升级指南中所述，在Elasticsearch升级到6.0之前，不应升级Logstash 6.0。这是实用的，因为一些Logstash 6.0插件可能会尝试使用只存在Elasticsearch早期版本中，而不存在的Elasticsearch 6.0功能。