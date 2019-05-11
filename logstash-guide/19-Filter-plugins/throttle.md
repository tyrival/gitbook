## throttle

- 插件版本：v4.0.4
- 发布于：2017-11-07
- [更新日志](https://github.com/logstash-plugins/logstash-filter-throttle/blob/v4.0.4/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/filter-throttle-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-filter-throttle) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

Throttle过滤器用于限制事件的数量。 过滤器配置有下限"before_count"和上限"after_count"以及时间段。 通过过滤器的事件将根据其键和事件时间戳进行计数。 只要计数小于"before_count"或大于"after_count"，事件就会被"限制"，这意味着过滤器将被视为成功，并且将添加（或删除）任何标记或字段。

该插件是线程安全的，可以正确跟踪过去的事件。

例如，如果您想要限制事件在2次出现后只收到一个事件，并且在10分钟内得到的事件不超过3次，那么您可使用配置：

```sh
period => 600
max_age => 1200
before_count => 3
after_count => 5
```

结果如下：

```
event 1 - throttled (successful filter, period start)
event 2 - throttled (successful filter)
event 3 - not throttled
event 4 - not throttled
event 5 - not throttled
event 6 - throttled (successful filter)
event 7 - throttled (successful filter)
event x - throttled (successful filter)
period end
event 1 - throttled (successful filter, period start)
event 2 - throttled (successful filter)
event 3 - not throttled
event 4 - not throttled
event 5 - not throttled
event 6 - throttled (successful filter)
...
```

另一个示例是，当你希望每小时只收到1个事件，可以使用如下配置：

```sh
period => 3600
max_age => 7200
before_count => -1
after_count => 1
```

结果如下：

```
event 1 - not throttled (period start)
event 2 - throttled (successful filter)
event 3 - throttled (successful filter)
event 4 - throttled (successful filter)
event x - throttled (successful filter)
period end
event 1 - not throttled (period start)
event 2 - throttled (successful filter)
event 3 - throttled (successful filter)
event 4 - throttled (successful filter)
...
```

一个常见的用例是，在使用多个字段作为key时，使用throttle过滤器对小于3和大于5的事件进行限制，然后使用drop过滤器来删除限制事件。 此配置可能显示为：

```sh
filter {
  throttle {
    before_count => 3
    after_count => 5
    period => 3600
    max_age => 7200
    key => "%{host}%{message}"
    add_tag => "throttled"
  }
  if "throttled" in [tags] {
    drop { }
  }
}
```

另一种情况是存储所有事件，但仅发送非限制事件的电子邮件，以便在发生系统错误时，op的收件箱不会充满电子邮件。 此配置可能显示为：

```sh
filter {
  throttle {
    before_count => 3
    after_count => 5
    period => 3600
    max_age => 7200
    key => "%{message}"
    add_tag => "throttled"
  }
}
output {
  if "throttled" not in [tags] {
    email {
      from => "logstash@mycompany.com"
      subject => "Production System Alert"
      to => "ops@mycompany.com"
      via => "sendmail"
      body => "Alert on %{host} from path %{path}:\n\n%{message}"
      options => { "location" => "/usr/sbin/sendmail" }
    }
  }
  elasticsearch_http {
    host => "localhost"
    port => "19200"
  }
}
```

收到事件时，事件key存储在key_cache中，并引用了时间槽缓存。事件根据时间戳分配到时间槽（动态创建）。时间槽计数器递增。当接收到下一个事件（相同的key）时，在相同的“周期”内，它被分配到相同的时间槽。时间槽计数器再次递增。

如果超过max_age，则时间槽到期。数量限制根据最新事件时间戳和max_age配置选项计算。

### Throttle过滤器配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 配置项                                                       | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`after_count`](#aftercount) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`before_count`](#beforecount) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`key`](#key) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 是   |
| [`max_age`](#maxage) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`max_counters`](#maxcounters) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`period`](#period) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### `after_count`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`-1`

大于此计数的事件将受到限制。将此值设置为-1（默认值）将导致不会根据上限限制事件。

##### `before_count`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`-1`

小于此计数的事件将受到限制。将此值设置为-1（默认值）将导致不会根据下限限制事件。

##### `key`

- 这是必需的设置。
- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

用于识别事件的关键。具有相同密钥的事件会被组合在一起。允许字段替换，因此您可以组合多个字段。

##### `max_age`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`3600`

时间槽的最大年龄。较高的值可以更好地跟踪异步事件流，但需要更多内存。根据经验，您应该将此值设置为至少两倍的周期。或者将此值设置为周期+具有相同键的无序事件之间的最大时间间隔。如果同时处理无序事件，则低于指定时间段的值会产生意外结果。

##### `max_counters`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`100000`

在减少时间槽的最大年龄之前，要存储的最大计数器数。将此值设置为-1将取消限制计数器的数量上限。此配置值应仅用作内存控制机制，如果达到该值，则可能导致早期计数器到期。建议保留默认值并确保选择key，以限制所需的计数器数量（即不要使用UUID作为密钥）。

##### `period`

- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为`"60"`

第一次出现事件之后的秒数，直到创建新时间槽。根据每个唯一key和每个时间槽跟踪此时段。该值允许字段替换，允许您指定某些类型的事件在特定时间段内节流。

### 通用配置项

所有过滤器插件都支持以下配置选项：

| 设置                                | 输入类型                                                     | 必须 |
| ----------------------------------- | ------------------------------------------------------------ | ---- |
| [`add_field`](#addfield)           | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`add_tag`](#addtag)               | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`enable_metric`](#enablemetric)   | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`id`](#id)                         | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`periodic_flush`](#periodicflush) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`remove_field`](#removefield)     | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`remove_tag`](#removetag)         | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |

##### add_field
- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为 `{}`

如果此过滤器配置成功，则向此事件添加任意字段。字段名称可以是动态的，并使用 `％{field}` 包含事件的部分内容。

例：

```sh
filter {
  throttle {
    add_field => { "foo_%{somefield}" => "Hello world, from %{host}" }
  }
}
```

```sh
# You can also add multiple fields at once:
filter {
  throttle {
    add_field => {
      "foo_%{somefield}" => "Hello world, from %{host}"
      "new_field" => "new_static_value"
    }
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将添加字段 `foo_hello` （如果存在），上面的值和 `%{host}` 部分替换为事件中的该值。第二个例子还会添加一个硬编码字段。

##### add_tag
- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

如果此过滤器成功，则向事件添加任意标记。标签可以是动态的，并使用 `%{field}` 语法包含事件的一部分。

例：

```sh
filter {
  throttle {
    add_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also add multiple tags at once:
filter {
  throttle {
    add_tag => [ "foo_%{somefield}", "taggedy_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将添加标记 `foo_hello`（第二个示例当然会添加 `taggedy_tag` 标记）。

##### enable_metric
- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

默认情况下，禁用或启用此插件实例的度量记录，我们会记录所有可用的度量标准，但您可以禁用特定插件的度量收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

为插件配置添加唯一 `ID`。如果未指定ID，Logstash将生成一个ID。强烈建议在配置中设置此ID。当您有两个或更多相同类型的插件时，这尤其有用，例如，如果您有2个http过滤器。在这种情况下添加命名ID将有助于在使用监视API时监视Logstash。

```sh
filter {
  throttle {
    id => "ABC"
  }
}
```

##### periodic_flush
- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

定期调用过滤器刷新方法。可选的。

##### remove_field
- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

如果此过滤器成功，请从此事件中删除任意字段。例：

```sh
filter {
  throttle {
    remove_field => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple fields at once:
filter {
  throttle {
    remove_field => [ "foo_%{somefield}", "my_extraneous_field" ]
  }
}
```

##### remove_tag

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

如果此过滤器配置成功，则从事件中删除任意tag。tag以是动态的，并使用 `%{field}` 语法包含事件的一部分。

例：

```sh
filter {
  throttle {
    remove_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple tags at once:
filter {
  throttle {
    remove_tag => [ "foo_%{somefield}", "sad_unwanted_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将删除标记 foo_hello（如果存在）。第二个例子也会删除一个不需要的标签。
