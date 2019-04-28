## elasticsearch

- 插件版本：v9.4.0
- 发布于：2019-02-06
- [更新日志](https://github.com/logstash-plugins/logstash-output-elasticsearch/blob/v9.4.0/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/output-elasticsearch-index.html)。

### 帮助
有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-output-elasticsearch) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

> **注意：**
>
> **兼容性说明**
>
> 从Elasticsearch 5.3开始，有一个名为 `http.content_type.required` 的 [HTTP设置](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/modules-http.html)。如果此选项设置为 `true`，并且您使用的是Logstash 2.4到5.2，则需要将Elasticsearch输出插件更新到6.2.5或更高版本。

如果您计划使用Kibana Web界面，请使用Elasticsearch输出插件，将您的日志数据导入Elasticsearch。

> **提醒：**
>
> 您可以在自己的硬件上运行Elasticsearch，也可以在Elastic Cloud上使用我们 [托管的Elasticsearch Service](https://www.elastic.co/cloud/elasticsearch-service)。 AWS和GCP均提供Elasticsearch服务。[免费试用Elasticsearch服务](https://www.elastic.co/cloud/elasticsearch-service/signup)。

此输出仅支持HTTP协议。从Logstash 2.0开始，HTTP是与Elasticsearch交互的首选协议。出于多种原因，我们强烈建议在节点上使用HTTP协议。 HTTP只是稍微慢一点，但更容易管理和使用。使用HTTP协议时，可以升级Elasticsearch版本，而无需在锁定步骤中升级Logstash。

您可以在 https://www.elastic.co/products/elasticsearch 上了解有关Elasticsearch的更多信息

### Elasticsearch 5.x 模板管理

Logstash 5.0的索引模板已更改为对接Elasticsearch 5.0的映射更改。最重要的是，字符串多字段的子字段已从 `.raw` 更改为 `.keyword` 以匹配ES默认行为。

##### 用户安装 ES 5.x 和 LS 5.x

此更改不会影响您，您将继续使用ES默认值。

##### 用户使用 ES 5.x 从 LS 2.x 升级到 LS 5.x

如果 `logstash` 模板已存在，LS将不会强制升级模板。这意味着您仍将使用 `.raw` 来处理来自2.x的子字段。如果选择使用新模板，则必须在安装新模板后重新索引数据。

### 重试策略

重试策略在8.1.1版本中发生了重大变化。此插件使用Elasticsearch批量API来优化其导入Elasticsearch。这些请求可能会遇到部分或全部故障。批量API将批量请求发送到HTTP端点。HTTP请求错误代码的处理方式与单个文档的错误代码不同。

对批量API的HTTP请求预计会返回200响应代码。所有其他响应代码将无限期重试。

以下文档错误处理如下：

- 如果启用，则将400和404错误发送到死信队列（DLQ）。如果未启用DLQ，将发出日志消息，并且将删除该事件。有关详细信息，请参阅 [死信队列策略](#死信队列策略)。
- 409错误（冲突）被记录为警告并被删除。

请注意，不再重试409异常。如果遇到409异常，请设置更高的 `retry_on_conflict` 值。 Elasticsearch比这个插件更有效地重试这些异常。

### 死信队列策略

Elasticsearch的映射（404）错误可能导致数据丢失。不幸的是，如果没有人为干预，并且没有查看导致映射不匹配的字段，则无法处理映射错误。如果启用了死信队列，则导致映射错误的原始事件将存储在文件中，以后再处理。通常，可以删除违规字段并将其重新编入索引到Elasticsearch。如果未启用DLQ，并且发生映射错误，则会将问题记录为警告，并且事件将被删除。有关在死信队列中处理事件的更多信息，请参阅 [死信队列](../10-Data-Resiliency/Dead-Letter-Queues.md)。

### 索引生命周期管理（ILM）

> **注意：**
>
> 索引生命周期管理功能需要插件版本9.3.1或更高版本。

> **注意：**
>
> 此功能需要6.6.0或更高版本的Elasticsearch实例，并且至少具有基本许可证

Logstash可以使用 [索引生命周期管理](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/index-lifecycle-management.html) 来自动化索引的管理。

索引生命周期管理的使用由 `ilm_enabled` 控制。`ilm_enabled` 标志可以设置为 `true`，`false`（默认）或 `auto`。

将 `ilm_enabled` 设置为 `auto`，将自动检测Elasticsearch实例是否为7.0.0及更高版本，以及是否启用了索引生命周期管理，如果是，则进行使用。

将 `ilm_enabled` 设置为 `true`，将启用索引生命周期管理功能（如果它在Elasticsearch集群上可用）。如果 `ilm_enabled` 设置为 `true`，但ILM在Elasticsearch集群不可用，则插件将无法启动。

启用ILM支持将覆盖索引设置并调整Logstash模板，写入模板的必要设置以支持索引生命周期管理，包括要使用的索引策略和滚动别名。

Logstash将为要写入的索引创建滚动别名，包括实际索引将如何命名的规则，除非已指定已存在的ILM策略，否则还将创建默认策略。默认策略配置为在索引大小达到50G字节时，或者在30天之后（以先发生者为准）滚动索引。

默认滚动别名为 `logstash`，其滚动索引的默认模式为 `{now/d}-00001`，它将在索引滚动时使用日期命名索引，后面加上流水号。请注意，匹配规则必须以短划线和将递增的数字结尾。

有关命名的更多详细信息，请参阅 [Rollover API文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/indices-rollover-index.html#_using_date_math_with_the_rollover_api)。

可以修改滚动别名，ilm模式和策略。

请参阅下面的配置示例：

```ruby
output {
  elasticsearch {
    ilm_enabled => true
    ilm_rollover_alias => "custom"
    ilm_pattern => "000001"
    ilm_policy => "custom_policy"
  }
}
```

> **注意：**
>
> 自定义ILM策略必须已存在于Elasticsearch集群上，才能使用。

> **注意：**
>
> 如果修改了滚动别名或规则，则需要覆盖索引模板，因为设置 `index.lifecycle.name` 和 `index.lifecycle.rollover_alias` 会自动写入模板

> **注意：**
>
> 如果在输出定义中提供了index属性，则它将被滚动别名覆盖。

### 批量大小

此插件尝试将批量事件作为单个请求发送。但是，如果请求超过20MB，我们会将其分解为多个批处理请求。如果单个文档超过20MB，它将作为单个请求发送。

### DNS缓存

此插件使用JVM查找DNS条目，并受到 [networkaddress.cache.ttl](https://docs.oracle.com/javase/7/docs/technotes/guides/net/properties.html)（jVM的全局设置）值的约束。

例如，要将DNS TTL设置为1秒，您可以将 `LS_JAVA_OPTS` 环境变量设置为 `-Dnetworkaddress.cache.ttl = 1`。

请记住，启用keepalive的连接，在keepalive生效时不会重新评估其DNS值。

### HTTP压缩

此插件支持请求和响应的压缩。默认情况下启用响应压缩，对于Elasticsearch 5.0及更高版本，用户不必在Elasticsearch中设置任何配置，以便发送回压缩响应。对于5.0之前的版本，在Elasticsearch中必须将 `http.compression` 设置为 `true`，才能在使用此插件时利用响应压缩

对于请求压缩，无论Elasticsearch版本如何，用户都必须在其Logstash配置文件中启用 `http_compression` 设置。

### Elasticsearch输出配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                                                         | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`action`](#action)                                          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`bulk_path`](#bulkpath)                                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`cacert`](#cacert)                                          | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`custom_headers`](#customheaders)                          | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`doc_as_upsert`](#docasupsert)                            | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`document_id`](#documentid)                                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`document_type`](#documenttype)                            | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`failure_type_logging_whitelist`](#failuretypeloggingwhitelist) | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`healthcheck_path`](#healthcheckpath)                      | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`hosts`](#hosts)                                            | [uri](../06-Configuring-Logstash/Structure-of-a-Config-File.md#uri) | 否   |
| [`http_compression`](#httpcompression)                      | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`ilm_enabled`](#ilmenabled)                                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，可选项为`["true", "false", "auto"]` | 否   |
| [`ilm_pattern`](#ilmpattern)                                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`ilm_policy`](#ilmpolicy)                                  | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`ilm_rollover_alias`](#ilmrolloveralias)                  | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`index`](#index)                                            | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`keystore`](#keystore)                                      | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`keystore_password`](#keystorepassword)                    | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`manage_template`](#managetemplate)                        | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`parameters`](#parameters)                                  | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`parent`](#parent)                                          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`password`](#password)                                      | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`path`](#path)                                              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`pipeline`](#pipeline)                                      | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`pool_max`](#poolmax)                                      | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`pool_max_per_route`](#poolmaxperroute)                  | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`proxy`](#proxy)                                            | [uri](../06-Configuring-Logstash/Structure-of-a-Config-File.md#uri) | 否   |
| [`resurrect_delay`](#resurrectdelay)                        | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`retry_initial_interval`](#retryinitialinterval)          | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`retry_max_interval`](#retrymaxinterval)                  | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`retry_on_conflict`](#retryonconflict)                    | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`routing`](#routing)                                        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`script`](#script)                                          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`script_lang`](#scriptlang)                                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`script_type`](#scripttype)                                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，可选项为`["inline", "indexed", "file"]` | 否   |
| [`script_var_name`](#scriptvarname)                        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`scripted_upsert`](#scriptedupsert)                        | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`sniffing`](#sniffing)                                      | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`sniffing_delay`](#sniffingdelay)                          | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`sniffing_path`](#sniffingpath)                            | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`ssl`](#ssl)                                                | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`ssl_certificate_verification`](#sslcertificateverification) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`template`](#template)                                      | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`template_name`](#templatename)                            | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`template_overwrite`](#templateoverwrite)                  | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`timeout`](#timeout)                                        | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`truststore`](#truststore)                                  | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`truststore_password`](#truststorepassword)                | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`upsert`](#upsert)                                          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`user`](#user)                                              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`validate_after_inactivity`](#validateafterinactivity)    | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`version`](#version)                                        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`version_type`](#versiontype)                              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，可选项为`["internal", "external", "external_gt", "external_gte", "force"]` | 否   |

另请参阅  [通用配置项](#通用配置项) 以获取所有输出插件支持的选项列表。

##### action

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为"索引"

此配置与协议无关（即非http，非java特定），表示要执行的Elasticsearch操作。方法是：

- index：索引文档（来自Logstash的事件）。
- delete：按id删除文档（此操作需要id）
- create：索引文档，如果索引中已存在该id的文档，则会失败。
- update：按ID更新文档。更新有一个特殊情况，如果更新文档时文档不存在，则进行新增。请参阅 `upsert` 选项。注意：这在Elasticsearch 1.x中不起作用也不受支持。请升级到ES 2.x或更高版本，以在Logstash中使用此功能！
- sprintf样式字符串，用于根据事件内容更改操作。值 `%{[foo]}` 将使用foo字段进行操作

有关操作的更多详细信息，请查看 [Elasticsearch批量API文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

##### bulk_path

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

用于执行\_bulk请求的HTTP路径，默认为路径参数和"_bulk"的串联

##### cacert

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

用于验证服务器证书的.cer或.pem文件

##### doc_as_upsert

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

为更新模式启用 `doc_as_upsert`。如果Elasticsearch中不存在 `document_id`，请使用source创建新文档

##### document_id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

索引的文档ID。用于覆盖Elasticsearch中具有相同ID的现有条目。

##### document_type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。
- 不推荐使用此选项

注意：由于 [删除了Elasticsearch 6.0中的类型](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/removal-of-types.html)，因此不推荐使用此选项。它将在Logstash的下一个主要版本中删除。这会将文档类型设置为将事件写入。通常，您应该尝试仅将类似事件写入相同类型。字符串扩展 `%{foo}` 在这里起作用。如果您没有为此选项设置值：

- 对于弹性搜索集群6.x及以上：将使用doc的值;
- 对于弹性搜索集群5.x及以下：将使用事件的类型字段，如果该字段不存在，则将使用doc的值。

##### failure_type_logging_whitelist

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

在白名单中设置您不想记录的Elasticsearch错误。一个比较有用的场景是您想要跳过所有409错误 —— `document_already_exists_exception` 时。

##### custom_headers

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

将一组键值对注入每个请求的头信息中，并发送给elasticsearch节点。头信息将用于任何类型的请求（_bulk请求，模板安装，运行状况检查和嗅探）。这些自定义标头将被 `http_compression` 等设置覆盖。

##### healthcheck_path

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

HTTP路径，当后端被标记为发送时发送HEAD请求，请求在后台发送，以查看它是否在再次有资格为请求提供服务之前再次返回。如果您有自定义防火墙规则，则可能需要更改此设置

##### hosts
- 值类型是 [uri](../06-Configuring-Logstash/Structure-of-a-Config-File.md#uri)
- 默认值为 `[//127.0.0.1]`

设置远程实例的主机。如果给定一个数组，它将在 `hosts` 参数中指定的主机上进行请求的负载均衡。请记住，http协议使用 [http](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html#modules-http) 地址（例如，9200，而不是9300）。 `"127.0.0.1"` `["127.0.0.1:9200","127.0.0.2:9200"]` `["http://127.0.0.1"]``["https://127.0.0.1:9200"]` `["https://127.0.0.1:9200/mypath"]` （如果在子路径上使用代理）从主机列表中排除 [专用主节点](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html) 非常重要，以防止Logstash向主节点发送批量请求。因此，此参数应仅引用Elasticsearch中的数据或客户端节点。

这里的URL中出现的任何特殊字符必须是URL转义！这意味着 `＃` 应该以 `％23` 的形式输入。

##### http_compression

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

对请求启用gzip压缩。请注意，默认情况下，Elasticsearch v5.0及更高版本的响应压缩处于启用状态

##### ilm_enabled

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，值包含：`true, false, auto`
- 默认值为 `false`

默认设置 `false` 将禁用索引生命周期管理功能。

如果Elasticsearch集群支持，则将此标志设置为 `true` 将启用索引生命周期管理功能。这是在7.0.0及更高版本的Elasticsearch上启用索引生命周期管理所必需的。

如果Elasticsearch集群在启用ILM功能的情况下，运行Elasticsearch 7.0.0或更高版本，则将此标志设置为 `auto` 将自动启用索引生命周期管理功能，否则将禁用它。

> **注意：**
>
> 此功能要求在Elasticsearch集群版本6.6.0或更高版本上安装基本许可证

##### ilm_pattern

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `{now/d}-000001`

索引生命周期管理的生成索引的规则。规则中指定的值将在创建时附加到别名，并在ILM创建新索引时自动递增。

指定ilm模式时可以使用Date Math，有关详细信息，请参阅 [Rollover API文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/indices-rollover-index.html#_using_date_math_with_the_rollover_api)

> **注意：**
>
> 更新模式将需要重写索引模板

> **注意：**
>
> 模式必须以短划线和数字结束，当索引滚动时，该数字将自动递增。

##### ilm_policy

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值是 `logstash`
  修改此设置以使用自定义索引生命周期管理策略，而不是默认值。如果未设置此值，则默认策略将自动安装到Elasticsearch中

> 注意
> 如果指定了此设置，则策略必须已存在于Elasticsearch集群中。

##### ilm_rollover_alias

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值是 `logstash`

翻转别名是将使用索引生命周期管理管理的索引将被写入的别名。

> **注意：**
>
> 如果同时指定了 `index` 和 `ilm_rollover_alias`，则 `ilm_rollover_alias` 优先。

> **注意：**
>
> 更新滚动别名将需要重写索引模板

##### index

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"logstash-%{+YYYY.MM.dd}"`

写入事件的索引，可以使用 `%{foo}` 动态语法。默认值将按日对索引进行分区，以便您可以更轻松地删除旧数据或仅搜索特定日期范围。索引不能包含大写字符。对于每周索引，建议使用ISO 8601格式，例如，` logstash-%{+xxxx.ww}`。 LS使用Joda从事件时间戳格式化索引模式。 Joda格式在 [这里](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html) 定义。

##### keystore

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

用于向服务器提供证书的密钥库。它可以是.jks或.p12

##### keystore_password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

设置密钥库密码

##### manage_template

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

从Logstash 1.3开始，如果名称为 `template_name` 的模板尚不存在，则在Logstash启动期间将模板应用于Elasticsearch。默认情况下，此默认模板的内容是 `logstash-%{+YYYY.MM.dd}` ，它始终根据模式 `logstash- *` 匹配索引。如果您需要支持其他索引名称，或者想要更改模板中的映射，可以通过将 `template` 设置自定义模板的路径。

将 `manage_template` 设置为false将禁用此功能。如果您需要对模板创建进行更多控制（例如，根据字段名称动态创建索引），则应将 `manage_template` 设置为false，并使用REST API手动应用模板。

##### parameters

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

传递一组键值对作为URL查询字符串。此查询字符串将添加到hosts配置中列出的每个主机。如果主机列表包含已具有查询字符串的URL，则将附加此处指定的URL。

##### parent

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `nil`

对于子文档，关联父项的ID。这可以使用 `%{foo}` 动态语法。

##### password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

用于向安全Elasticsearch集群进行身份验证的密码

##### path

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

Elasticsearch服务器所在的HTTP路径。如果必须将Elasticsearch运行于代理之后，并且需要代理对Elasticsearch HTTP API进行重映射，请使用此选项。请注意，如果您在hosts字段中使用路径作为URL的组件，则可能也不会设置此字段。这会在启动时引发错误

##### pipeline

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `nil`

设置您希望为事件执行的摄取管道。您还可以在此处使用事件相关配置，例 `pipeline => "%{INGEST_PIPELINE}"`

##### pool_max

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `1000`

当输出尝试有效地重用连接时有一个最大值，此设置决定输出创建的最大打开连接数。将此设置得太低可能意味着经常关闭/打开不良的连接。

##### pool_max_per_route

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `100`

当输出尝试有效地重用连接时，我们每个端点都有一个最大值。此设置决定输出创建的每个端点的最大打开连接数。将此设置得太低可能意味着经常关闭/打开不良的连接。

##### proxy

- 值类型是 [uri](../06-Configuring-Logstash/Structure-of-a-Config-File.md#uri)
- 此设置没有默认值。

设置转发HTTP代理的地址。这用于接受散列作为参数，但现在只接受URI类型的参数以防止泄漏凭据。

##### resurrect_delay

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `5`

尝试复活的频率间隔。复活是指检查标记下来的后端端点，以查看它们是否已恢复生命的过程

##### retry_initial_interval

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `2`

设置批量重试之间的初始间隔（秒）。每次重试时加倍，直到 `retry_max_interval`

##### retry_max_interval

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `64`

设置批量重试之间的最大间隔（秒）。

##### retry_on_conflict

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `1`

Elasticsearch应在内部重试更新/上传文档的次数有关详细信息，请参阅 [部分更新](https://www.elastic.co/guide/en/elasticsearch/guide/current/partial-updates.html)

##### routing

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

要应用于所有已处理事件的路由覆盖。这可以使用 `%{foo}` 语法动态化。

##### script

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `""`

为脚本化更新模式设置脚本名称

例：

```ruby
output {
  elasticsearch {
    script => "ctx._source.message = params.event.get('message')"
  }
}
```

##### script_lang

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值是 `"painless"`

设置使用的脚本的语言。如果未设置，则默认为ES 5.0中的 "painless"。在Elasticsearch 6及更高版本上使用索引（存储）脚本时，必须将此参数设置为 `""`（空字符串）。

##### script_type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，可以是：`inline`, `indexed`, `file`
- 默认值为 `["inline"]`

定义"script"变量内联引用的脚本类型："script"包含内联脚本的索引："script"包含在elasticsearch文件中直接编入索引的脚本名称："script"包含存储在elasticsearch的config目录中的脚本名称

##### script_var_name

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"event"`

设置传递给脚本的变量名称（脚本化更新）

##### scripted_upsert

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

如果启用，脚本负责创建不存在的文档（脚本更新）

##### sniffing

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

此设置要求Elasticsearch提供所有群集节点的列表，并将它们添加到主机列表中。对于Elasticsearch 1.x和2.x，任何具有` http.enabled`（默认情况下为on）的节点都将添加到主机列表中，仅包括主节点！对于Elasticsearch 5.x和6.x，任何具有 `http.enabled`（默认情况下为on）的节点都将添加到主机列表中，仅排除主节点。

##### sniffing_delay

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `5`

嗅闻尝试之间等待的时间，以秒为单位

##### sniffing_path

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

用于嗅探请求的HTTP路径默认值是通过连接路径值和"_nodes / http"来计算的，如果设置了`sniffing_path`，它将被用作绝对路径，这里不使用完整URL，只有路径，例如路径。"/sniff/\_nodes/http"

##### ssl

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 此设置没有默认值。

启用与Elasticsearch集群的SSL/TLS安全通信。保留此未指定将使用在主机中列出的URL中指定的任何方案。如果未指定显式协议，则将使用纯HTTP。如果在此处明确禁用SSL，则如果在主机中提供HTTPS URL，则插件将拒绝启动

##### ssl_certificate_verification

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

用于验证服务器证书的选项。禁用此功能会严重影响安全性。有关禁用证书验证的更多信息，请阅读 https://www.cs.utexas.edu/~shmat/shmat_ccs12.pdf

##### template

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果您愿意，可以在此处设置自己模板的路径。如果未设置，将使用包含的模板。

##### template_name

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"logstash"`

此配置项定义如何在Elasticsearch中命名模板。请注意，如果您已使用模板管理功能，并随后更改此功能，则需要手动修剪旧模板，例如

```shell
curl -XDELETE <http：// localhost：9200 / _template / OldTemplateName？pretty>
```

其中 `OldTemplateName` 是以前的设置。

##### template_overwrite

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

template_overwrite选项将始终使用模板指示的模板，或包含的模板覆盖Elasticsearch中指示的模板。默认情况下，此选项设置为false。如果您始终希望了解Logstash提供的模板，则此选项对您非常有用。同样，如果你有自己的模板文件由puppet管理，并且你希望能够定期更新它，这个选项也可以帮助那里。

请注意，如果您使用自己的自定义版本的Logstash模板（logstash），将此设置为true将使Logstash覆盖"logstash"模板（即删除所有自定义设置）

##### timeout

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `60`

设置发送Elasticsearch的网络操作和请求的超时（以秒为单位）。如果发生超时，将重试该请求。

##### truststore

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

用于验证服务器证书的信任库。它可以是.jks或.p12。使用 `:truststore` 或 `:cacert`。

##### truststore_password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

设置信任库密码

##### upsert

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `""`

为更新模式设置upsert内容。如果 `document_id` 不存在，则使用此参数创建一个新文档作为json字符串

##### USER

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

用于对安全的Elasticsearch集群进行身份验证的用户名

##### validate_after_inactivity

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `10000`

在使用keepalive在连接执行请求之前，检查连接是否过时之前要等待多长时间。您可能希望将此值设置得更低，如果您经常收到连接错误引用Apache commons文档（此客户端基于Apache Commmons）：定义不活动的时间段（以毫秒为单位），之后必须重新验证持久连接，然后再将其租给消费者。传递给此方法的非正值会禁用连接验证。此检查有助于检测已变为陈旧（半关闭）的连接，同时在池中保持不活动状态。有关详细信息，请参阅 [这些文档](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/impl/conn/PoolingHttpClientConnectionManager.html#setValidateAfterInactivity(int))

##### version

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

用于索引的版本。使用 `%{my_version}` 等sprintf语法，在此处使用字段值。请参阅https://www.elastic.co/blog/elasticsearch-versioning-support。

##### version_type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，值包含： `internal`, `external`, `external_gt`, `external_gte`, `force`
- 此设置没有默认值。

用于索引的version_type。请参阅https://www.elastic.co/blog/elasticsearch-versioning-support。另见[https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index\_.html#_version_types](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#_version_types)

### 通用配置项

所有输出插件都支持以下配置选项：

| 设置                              | 输入类型                                                     | 必须 |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| [`enable_metric`](#enablemetric) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`id`](#id)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

##### enable_metric

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

禁用或启用此特定插件实例的度量标准记录。 默认情况下，我们会记录所有可用的指标，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

为插件配置添加唯一ID。 如果未指定ID，Logstash将生成一个ID。 强烈建议在配置中设置此ID。 当您有两个或更多相同类型的插件时，这尤其有用。 例如，如果您有2个elasticsearch输出。 在这种情况下添加命名ID将有助于在使用监视API时监视Logstash。

```json
output {
  elasticsearch {
    id => "my_plugin_id"
  }
}
```
