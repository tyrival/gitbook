## dead_letter_queue

- 插件版本：v1.1.4
- 发布于：2018-04-06
- [更新日志](https://github.com/logstash-plugins/logstash-input-dead_letter_queue/blob/v1.1.4/CHANGELOG.md)

其他版本请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/input-dead_letter_queue-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-input-dead_letter_queue) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

Logstash输入从Logstash的死信队列中读取事件。

```sh
input {
  dead_letter_queue {
    path => "/var/logstash/data/dead_letter_queue"
    start_timestamp => "2017-04-04T23:40:37"
  }
}
```

有关处理死信队列中事件的更多信息，请参阅 [死信队列](../10-Data-Resiliency/Dead-Letter-Queues.md)。

### 死信队列输入配置项

此插件支持以下配置选项，以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                              | 输入类型                                                     | 必须 |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| [`add_field`](#addfield)         | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`codec`](#codec)                 | 有效的文件系统路径                                           | 否   |
| [`enable_metric`](#enablemetric) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`id`](#id)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`tags`](#tags)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`type`](#type)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

 ##### commit_offsets
- 值类型是布尔值
- 默认值为 `true`

指定此输入在处理事件时是否提交偏移量。通常，如果要在死信队列中的事件上多次迭代，但不想保存状态，则指定 `false`。您在死信队列中探索事件的时候可以使用此方法。

#####path
- 这是必需的设置。
- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

由Logstash实例创建的死信队列目录的路径。这是读取"死”事件的路径，通常使用 `path.dead_letter_queue` 在原始Logstash实例中配置。

##### pipeline_id
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"main”`

要读取其事件的管道的ID。

##### sincedb_path
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。
  将写入磁盘的sincedb数据库文件的路径（跟踪死信队列的当前位置）。默认情况下会将sincedb文件写入 `<path.data>/plugins/inputs/dead_letter_queue`。

> 注意
> 该值必须是文件路径，而不是目录路径。

##### start_timestamp

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

ISO8601格式的时间戳，从您希望开始处理事件时开始。例如， `2017-04-04T23:40:37`。

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
