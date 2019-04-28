## elasticsearch

- 插件版本：v4.3.0
- 发布于：2019-02-04
- [更新日志](https://github.com/logstash-plugins/logstash-input-elasticsearch/blob/v4.3.0/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/input-elasticsearch-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-input-elasticsearch) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

> **注意：**
> **兼容性说明**
> 从Elasticsearch 5.3开始，有个名为 `http.content_type.required` 的 [HTTP设置](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/modules-http.html)。如果此选项设置为 `true`，并且您使用的是Logstash 2.4到5.2，则需要将Elasticsearch输入插件更新到4.0.2或更高版本。

从Elasticsearch集群中读取检索结果。这对于重现测试日志、重建索引等非常有用。您可以使用cron语法定期计划采集（请参阅 `schedule` 设置），或运行一次查询以将数据加载到Logstash中。

例：

```ruby
input {
  # Read all documents from Elasticsearch matching the given query
  elasticsearch {
    hosts => "localhost"
    query => '{ "query": { "match": { "statuscode": 200 } }, "sort": [ "_doc" ] }'
  }
}
```

这将使用以下格式创建Elasticsearch查询：

```json
curl 'http://localhost:9200/logstash-*/_search?&scroll=1m&size=1000' -d '{
  "query": {
    "match": {
      "statuscode": 200
    }
  },
  "sort": [ "_doc" ]
}'
```

### 定时任务

可以通过定时任务来执行此插件的输入。此调度语法由 [rufus-scheduler](https://github.com/jmettraux/rufus-scheduler) 提供支持。语法类似于cron，具有指定于Rufus的一些扩展（例如，时区支持）。

例子：

| 语法                        | 说明                                   |
| --------------------------- | -------------------------------------- |
| `* 5 * 1-3 *`               | 每年1-3月的每天上午5点，每分钟执行一次 |
| `0 * * * *`                 | 每天每小时的第0分钟执行                |
| `0 6 * * * America/Chicago` | UTC/GMT -5时区，每天上午6点执行        |

可以在此处找到描述此语法的 [更多文档](https://github.com/jmettraux/rufus-scheduler#parsing-cronlines-and-time-strings)。

### Elasticsearch输入配置选项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                                | 输入类型                                                     | 必须 |
| ----------------------------------- | ------------------------------------------------------------ | ---- |
| [`ca_file`](#cafile)               | 有效的文件系统路径                                           | 否   |
| [`docinfo`](#docinfo)               | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`docinfo_fields`](#docinfofields) | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`docinfo_target`](#docinfotarget) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`hosts`](#hosts)                   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`index`](#index)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`password`](#password)             | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`query`](#query)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`schedule`](#schedule)             | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`scroll`](#scroll)                 | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`size`](#size)                     | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`slices`](#slices)                 | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`ssl`](#ssl)                       | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`user`](#user)                     | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### ca_file

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

PEM编码格式的SSL证书颁发机构文件，必要时还必须包含任何链证书。

##### docinfo

- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

如果设置，请在事件中包含Elasticsearch文档信息，例如索引，类型和ID。

关于元数据，可能需要注意的是，如果您正在采集文档，意图重新索引它们（或只是更新它们），那么elasticsearch输出中的 `action` 选项想要知道如何处理这些事情。可以动态添加到元数据字段。

例

```ruby
input {
  elasticsearch {
    hosts => "es.production.mysite.org"
    index => "mydata-2018.09.*"
    query => '{ "query": { "query_string": { "query": "*" } } }'
    size => 500
    scroll => "5m"
    docinfo => true
  }
}
output {
  elasticsearch {
    index => "copy-of-production.%{[@metadata][_index]}"
    document_type => "%{[@metadata][_type]}"
    document_id => "%{[@metadata][_id]}"
  }
}
```

> **注意：**
> 从Logstash 6.0开始，由于删除了Logstash 6.0中的类型，因此不推荐使用 `document_type` 选项。它将在Logstash的 [下一个主要版本中删除](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/removal-of-types.html)。

##### docinfo_fields

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `["_index", "_type", "_id"]`

如果通过启用 `docinfo` 选项请求文档元数据存储，则此选项会列出要在当前事件中保存的元数据字段。有关详细信息，请参阅 [Elasticsearch元数据文档](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/_document_metadata.html)。

##### docinfo_target

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"@metadata"`

如果通过启用 `docinfo` 选项请求文档元数据存储，则此选项将用于存储元数据字段的字段命名为子字段。

##### hosts

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 此设置没有默认值。

用于查询的一个或多个Elasticsearch主机的列表。每个主机可以是IP，HOST，IP:PORT 或 HOST:PORT。端口默认为9200。

##### index

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"logstash- *"`

要搜索的索引或别名。有关如何引用多个索引的更多信息，请参阅Elasticsearch文档中的 [多索引文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-index.html)。

##### password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

在向Elasticsearch服务器进行身份验证时，与 `user` 选项中的用户名一起使用的密码。如果设置为空字符串，则将禁用身份验证。

##### query

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `'{ "sort": [ "_doc" ] }'`

要执行的查询。有关更多信息，请阅读Elasticsearch查询 [DSL文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)。

##### schedule

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

以Cron格式定期运行语句的时间表，例如："* * * * *"（每分钟执行查询，分钟）

默认情况下没有计划。如果没有给出计划，那么该语句只运行一次。

##### scroll

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"1m"`

此参数控制滚动请求的保持活动时间（以秒为单位）并启动滚动过程。每次往返（即在前一个滚动请求之间，到下一个滚动请求之间）应用超时。

##### size

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `1000`

这允许您设置每个滚动返回的最大命中数。

##### slices

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 没有默认值。
- 合理的值范围从2到大约8。

在某些情况下，可以通过使用 [Sliced Scroll API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll) 同时使用多个不同的查询切片，来提高整体吞吐量，尤其是在管道花费大量时间等待Elasticsearch提供结果的情况下。

如果设置，则 `slices` 参数告诉插件将工作分成多少个切片，并将从切片中并行生成事件，直到所有切片都完成滚动。

> **注意：**
> Elasticsearch手册指出，当滚动查询使用的索片多于索引中的分片时，查询和Elasticsearch集群可能会产生负面的性能影响。

如果未设置 `slices` 参数，则插件不会将切片指令注入查询。

##### ssl

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为false

如果启用，则在与Elasticsearch服务器通信时将使用SSL（即将使用HTTPS而不是普通HTTP）。

##### user

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

在向Elasticsearch服务器进行身份验证时，与密码选项中的密码一起使用的用户名。如果设置为空字符串，则将禁用身份验证。

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

```json
input {
  beats {
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
