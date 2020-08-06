## grok

- 插件版本：v4.0.4
- 发布于：2018-10-19
- [更新日志](https://github.com/logstash-plugins/logstash-filter-grok/blob/v4.0.4/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/filter-grok-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-filter-grok) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

解析任意文本并对其进行构造。

Grok是一种很好的方法，可以将非结构化日志数据解析为结构化和可查询的内容。

此工具非常适用于syslog日志、apache和其他Web服务器日志、mysql日志，以及通常为人类而非计算机使用的任何日志格式。

Logstash默认提供约120种模式。你可以在这里找到它们：https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns。您可以轻松添加自己的。（参见 `patterns_dir` 设置）

如果您需要帮助构建模式以匹配您的日志，您会发现 http://grokdebug.herokuapp.com/ 和 http://grokconstructor.appspot.com/ 应用程序非常有用！

#### Grok和Dissect怎样选择？

[dissect过滤器插件](../19-Filter-plugins/dissect.md) 是使用分隔符，将非结构化事件数据提取到字段中的另一种方法。

Dissect与Grok的不同之处在于它不使用正则表达式并且速度更快。当数据可靠地重复时，解析很有效。当文本结构因行而异时，Grok是更好的选择。

当线路的一部分可靠地重复时，您可以同时使用Dissect和Grok作为混合用例，但整条线路不行。 Dissect过滤器可以解构重复行的部分。 Grok过滤器可以处理剩余的字段值，具有更多的正则表达式可预测性。

### Grok Basics

Grok的工作原理是将文本模式组合成与日志匹配的内容。

grok模式的语法是 `%{SYNTAX:SEMANTIC}`

`SYNTAX` 是与您文本匹配规则的名称。例如，`3.44` 将与 `NUMBER` 模式匹配，`55.3.244.1` 将与 `IP` 模式匹配。语法表示你如何匹配。

`SEMANTIC` 是您为匹配的文本提供的标识符。例如，`3.44` 可能是事件的 `duration` 时间，因此您可以将其称为持续时间。此外，字符串 `55.3.244.1` 可以标识发出请求的 `client`。

对于上面的示例，您的grok过滤器看起来像这样：

```ruby
%{NUMBER:duration} %{IP:client}
```

您可以选择将数据类型转换添加到grok匹配。默认情况下，所有语义都保存为字符串。如果您希望转换语义的数据类型，例如将字符串更改为整数，则使用目标数据类型将其后缀。例如 `%{NUMBER:num:int}` ，它将 `num` 语义从字符串转换为整数。目前唯一支持的转换是 `int` 和 `float`。

**示例**：通过语法和语义的概念，我们可以从示例日志中提取有用的字段，如虚构的http请求日志：

```ruby
55.3.244.1 GET /index.html 15824 0.043
```

这种匹配规则可能是：

```ruby
%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}
```

一个更现实的例子，让我们从文件中读取这些日志：

```ruby
input {
  file {
    path => "/var/log/http.log"
  }
}
filter {
  grok {
    match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
  }
}
```

在grok过滤器之后，该事件将包含一些额外的字段：

- `client: 55.3.244.1`
- `method: GET`
- `request: /index.html`
- `bytes: 15824`
- `duration: 0.043`

### 正则表达式

Grok基于正则表达式，因此任何正则表达式在grok中都是有效的。正则表达式库是Oniguruma，您可以在 [Oniguruma站点](https://github.com/kkos/oniguruma/blob/master/doc/RE) 上看到支持的regexp语法。

### 自定义匹配规则

有时logstash没有您需要的规则。为此，您有几个选择。

首先，您可以使用Oniguruma语法进行命名捕获，它可以匹配一段文本并将其保存为字段：

```ruby
(?<field_name>the pattern here)
```

例如，后缀日志的 `queue id` 为10或11个字符的十六进制值。我可以像这样轻松捕获：

```ruby
(?<queue_id>[0-9A-F]{10,11})
```

或者，您可以创建自定义模式文件。

- 创建一个名为 `patterns` 的目录，其中包含一个名为 `extra` 的文件（文件名无关紧要，但为自己命名有意义）
- 在该文件中，将您需要的规则写为规则名称，空格，然后是该模式的正则表达式。

例如，如上所述执行postfix队列id示例：

```ruby
# contents of ./patterns/postfix:
POSTFIX_QUEUEID [0-9A-F]{10,11}
```

然后使用此插件中的 `patterns_dir`，告诉logstash您的自定义模式目录所在的位置。这是一个包含示例日志的完整示例：

```ruby
Jan  1 06:25:43 mailserver14 postfix/cleanup[21403]: BEF25A72965: message-id=<20130101142543.5828399CCAF@mailserver14.example.com>
```

```ruby
filter {
  grok {
    patterns_dir => ["./patterns"]
    match => { "message" => "%{SYSLOGBASE} %{POSTFIX_QUEUEID:queue_id}: %{GREEDYDATA:syslog_message}" }
  }
}
```

以上将匹配并产生以下字段：

- `timestamp: Jan 1 06:25:43`
- `logsource: mailserver14`
- `program: postfix/cleanup`
- `pid: 21403`
- `queue_id: BEF25A72965`
- `syslog_message: message-id=<20130101142543.5828399CCAF@mailserver14.example.com>`

 `timestamp`, `logsource`, `program`, 和 `pid` 字段来自 `SYSLOGBASE` 规则，该规则本身由其他规则定义。

另一种选择是使用 `pattern_definitions` 在过滤器中定义内联模式。这主要是为了方便，并允许用户定义可以在该过滤器中使用的模式。`pattern_definitions` 中新定义的模式在特定的 `grok` 过滤器之外将不可用。

### Grok过滤器配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 配置项                     | 输入类型                                                   | 必须 |
| --------------------------------------------- | ------------------------------------------------------------ | -------- |
| [`break_on_match`](#breakonmatch)           | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | No       |
| [`keep_empty_captures`](#keepemptycaptures) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | No       |
| [`match`](#match)                             | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | No       |
| [`named_captures_only`](#namedcapturesonly) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | No       |
| [`overwrite`](#overwrite)                     | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | No       |
| [`pattern_definitions`](#patterndefinitions) | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | No       |
| [`patterns_dir`](#patternsdir)               | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | No       |
| [`patterns_files_glob`](#patternsfilesglob) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | No       |
| [`tag_on_failure`](#tagonfailure)           | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | No       |
| [`tag_on_timeout`](#tagontimeout)           | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | No       |
| [`timeout_millis`](#timeoutmillis)           | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | No       |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### break_on_match

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

第一次匹配后中断。 grok的第一次成功匹配将导致过滤器完成。如果你想让grok尝试所有匹配（也许你正在解析不同的东西），那么将其设置为false。

##### keep_empty_captures

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

如果为 `true`，则将空捕获保留为事件字段。

##### match

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为 `{}`

一个hash，用于定义查看位置的映射以及使用哪些模式。

例如，以下内容将匹配给定模式的消息字段中的现有值，如果找到匹配项，则会将字段持续时间添加到具有捕获值的事件中：

```ruby
   filter {
     grok {
       match => {
         "message" => "Duration: %{NUMBER:duration}"
       }
     }
   }
```

如果需要将多个规则与单个字段匹配，则值可以是规则数组：

```ruby
filter {
  grok {
    match => {
      "message" => [
        "Duration: %{NUMBER:duration}",
        "Speed: %{NUMBER:speed}"
      ]
    }
  }
}
```

##### named_captures_only

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

如果为 `true`，则仅存储来自grok的名为capture的存储。

##### overwrite

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

要覆盖的字段。

这允许您覆盖已存在的字段中的值。

例如，如果 `message` 字段中有syslog行，则可以使用部分匹配覆盖 `message` 字段，如下所示：

```ruby
filter {
  grok {
    match => { "message" => "%{SYSLOGBASE} %{DATA:message}" }
    overwrite => [ "message" ]
  }
}
```

在这种情况下，像5月29日16:37:11悲伤记录器的行：hello world将被解析，hello world将覆盖原始消息。

##### pattern_definitions

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为 `{}`

匹配规则名称和匹配规则元组的散列，用于定义当前过滤器使用的自定义规则。匹配现有名称的规则将覆盖预先存在的定义。可以将其视为可用于grok定义的内联模式

##### patterns_dir

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

Logstash默认带有一堆模式，因此除非您要添加其他模式，否则您不一定需要自己定义。您可以使用此设置指向多个模式目录。请注意，Grok将读取与patterns_files_glob匹配的目录中的所有文件，并假设它是一个规则文件（包括任何代字号备份文件）。

```ruby
patterns_dir => ["/opt/logstash/patterns", "/opt/logstash/extra_patterns"]
```

规则文件是纯文本格式：

```ruby
NAME PATTERN
```

例如：

```ruby
NUMBER \d+
```

创建管道时会加载模式。

##### patterns_files_glob

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"*"`

Glob模式，用于选择patterns_dir指定的目录中的模式文件

##### tag_on_failure

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `["_grokparsefailure"]`

如果没有成功匹配，则将值附加到标记字段

##### tag_on_timeout

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"_groktimeout"`

如果grok regexp超时，则标记为应用。

##### timeout_millis

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为30000

在这段时间后尝试终止正则表达式。如果应用了多个规则，这适用于每个规则。超时永远不会提前，但可能延迟一些。实际超时是基于250ms量化的近似值。设置为0可禁用超时。

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
  grok {
    add_field => { "foo_%{somefield}" => "Hello world, from %{host}" }
  }
}
```

```sh
# You can also add multiple fields at once:
filter {
  grok {
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
  grok {
    add_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also add multiple tags at once:
filter {
  grok {
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
  grok {
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
  grok {
    remove_field => [ "foo_%{somefield}" ]
  }
}
```

```json
# You can also remove multiple fields at once:
filter {
  grok {
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
  grok {
    remove_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple tags at once:
filter {
  grok {
    remove_tag => [ "foo_%{somefield}", "sad_unwanted_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将删除标记 foo_hello（如果存在）。第二个例子也会删除一个不需要的标签。
