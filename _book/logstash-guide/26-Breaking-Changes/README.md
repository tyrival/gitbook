# 第26章-重大变更

我们努力维护次要版本（例如6.x到6.y）之间的向后兼容性，以便您可以在不更改任何配置文件的情况下进行升级。通常仅在主要版本（例如5.x到6.y）之间引入重大更改。有时，我们被迫在特定的主要版本中破坏兼容性以确保操作的正确性。

本节介绍迁移到Logstash 6.0.0及更高版本时需要注意的更改。

> **注意：**
> 不建议直接在非连续主要版本（1.x到6.x）之间迁移。

有关重大更改的说明，请参阅以下主题：

- [更早的PQ版本到6.3.0的重大变更](#更早的PQ版本到6.3.0的重大变更)
- [6.0重大变更](#6.0重大变更)

另请参见 [发行说明](https://www.elastic.co/guide/en/logstash/6.7/releasenotes.html)。

### 更早的PQ版本到6.3.0的重大变更
如果要从Logstash 6.2.x或更早期版本（包括5.x）升级并启用持久队列，我们强烈建议您在升级之前排空或删除持久队列。有关信息和说明，请参阅 [启用持久队列升级](../05-Uprading-Logstash/Upgrading-with-the-Persistent-Queue-Enabled.md)。

我们正在努力解决数据不兼容问题，以便将来升级不需要其他步骤。

### 6.0重大变更
以下是6.0的重大变化。

#### Logstash核心变更

这些变更可能会影响任何Logstash实例，并且与插件无关，但前提是您正使用受影响的功能。

#### 应用程序设置

- `config.reload.interval` 已更改为使用时间值字符串，如 `5m`，`10s` 等。以前，用户必须自己将其转换为毫秒时间值。

#### RPM/Deb包变更

- 对于 `rpm` 和 `deb` 发布工件，与。`.conf` 类型的配置文件必须位于conf.d文件夹中，否则将不会加载文件。

#### 命令行（CLI）功能

- `-e` 和 `-f` CLI参数现在是互斥的。这也适用于相应的长格式选项 `config.string` 和 `path.config`。这意味着通过 `-e` 提供的任何配置，将不再附加到通过 `-f` 提供的配置中。
- 使用 `-f` 或 `config.path` 提供的配置，不会自动附加 `stdin` 输入和 `stdout` 输出。

#### 插件变更
##### Elasticsearch输出变更

- 默认 `document_type` 已从 `logs` 更改为 `doc`，以便与Beats保持一致。此外，建议用户Elasticsearch 6.0弃用doctypes，7.0将删除它们。有关详细信息，请参阅 [删除映射类型](https://www.elastic.co/guide/en/elasticsearch/reference/master/removal-of-types.html)。
- `flush_size` 和 `idle_flush_time` 选项现已过时。
- 请注意，[_all](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/mapping-all-field.html) 字段在6.0中已弃用。新的映射模板更新体现了这一点。如果您使用的是自定义映射模板，则可能需要对其进行更新。

##### Kafka输入变更

- 将Kafka客户端支持升级到v0.11.0.0，该版本仅支持Kafka代理v0.10.x或更高版本。
  - 有关Kafka与Logstash兼容性的信息，请参阅 [Kafka输入插件文档](../17-Input-plugins/kafka.md)。

- 装饰字段现在嵌套在 `@metadata` 下，以避免与Beats映射冲突。
  - 有关更多详细信息，请参阅 [Kafka输入插件文档](../17-Input-plugins/kafka.md) 中的元数据字段部分。

- `ssl` 选项现已过时。

##### Kafka输出变更

- 将Kafka客户端支持升级到v0.11.0.0，该版本仅支持Kafka代理v0.10.x或更高版本。
  - 有关Kafka与Logstash兼容性的信息，请参阅 [Kafka输出插件文档](../18-Output-plugins/kafka.md)。

- `block_on_buffer_full`，`ssl` 和 `timeout_ms` 选项现已过时。

##### Beats输入变更

- 当 [Multiline编解码器插件](../20-Coder-plugins/multiline.md) 与Beats输入插件一起使用时，Logstash将不再启动。
  - 建议使用Filebeat中的多行支持作为替代。有关详细信息，请参阅 [Filebeat配置选项](https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html)。

- 现在，选项 `congestion_threshold` 和 `target_field_for_codec` 已过时。

##### 与Logstash捆绑的插件

根据使用情况数据，6.0默认包中删除了以下插件。您仍然可以手动安装这些插件：

- logstash-codec-oldlogstashjson
- logstash-input-couchdb_changes
- logstash-input-irc
- logstash-input-log4j
- logstash-input-lumberjack
- logstash-filter-uuid
- logstash-output-xmpp
- logstash-output-irc
- logstash-output-statsd
