## prune

- 插件版本：v3.0.3
- 发布于：2017-11-07
- [更新日志](https://github.com/logstash-plugins/logstash-filter-prune/blob/v3.0.3/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/filter-prune-index.html)。

### 安装

对于默认情况下未捆绑的插件，可以通过运行 `bin/logstash-plugin install logstash-filter-prune` 轻松安装。 有关更多详细信息，请参阅 [使用插件](../16-Working-with-plugins/README.md)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-filter-prune) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

剪枝过滤器用于根据白名单或字段名称黑名单或其值删除事件中的字段（名称和值也可以是正则表达式）。

这可以是例如如果你有一个json或kv过滤器，它可以创建许多字段，这些字段的名称你不一定知道事先的名字，你只想保留它们的子集。

用法帮助：要指定确切的字段名称或值，请使用正则表达式语法^ some_name_or_value $。用法示例：输入数据{“msg”：“hello world”，“msg_short”：“hw”}

```sh
filter {
    prune {
    	whitelist_names => [ "msg" ]
    }
}
```
允许`"msg"`和`"msg_short"`通过。

而：

```sh
filter {
    prune {
    	whitelist_names => ["^msg$"]
    }
}
```
只允许`"msg"`通过。

Logstash将事件的`tags`存储为需要修剪的字段。请记住`whitelist_names => [ "^tags$" ]`以在修剪后维护`tags`或使用`blacklist_values => [ "^tag_name$" ]`来消除特定`tag`。\

> **注意**
>
> 此过滤器目前仅支持顶级字段上的操作，基于名称或值的子字段的白名单和黑名单不起作用。

### Prune过滤器配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 配置项                                 | 输入类型                                                     | 必须 |
| -------------------------------------- | ------------------------------------------------------------ | ---- |
| [`blacklist_names`](#blacklistnames)   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`blacklist_values`](#blacklistvalues) | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`interpolate`](#interpolate)          | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`whitelist_names`](#whitelistnames)   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`whitelist_values`](#whitelistvalues) | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### `blacklist_names`

- 值类型是[array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为`["%{[^}]+}"]`

排除名称与指定regexp匹配的字段，默认情况下排除未解析的`%{field}`字符串。

```sh
filter {
  prune {
    blacklist_names => [ "method", "(referrer|status)", "${some}_field" ]
  }
}
```

##### `blacklist_values`

- 值类型是[hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为`{}`

如果指定的字段的值与提供的正则表达式之一匹配，则排除它们。如果字段值是数组，则每个数组项与正则表达式匹配，并且将排除匹配的数组项。

```sh
filter {
  prune {
    blacklist_values => [ "uripath", "/index.php",
                          "method", "(HEAD|OPTIONS)",
                          "status", "^[^2]" ]
  }
}
```

##### `interpolate`

- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为false

触发是否应为动态值插入配置字段和值（解析`%{some_field}`时）。可能会增加一些性能开销。默认为false。

##### `whitelist_names`

- 值类型是[array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为[]
仅当字段的名称与指定的正则表达式匹配时才包含字段，默认为空列表，表示包含所有内容。

```sh
filter {
  prune {
    whitelist_names => [ "method", "(referrer|status)", "${some}_field" ]
  }
}
```

##### `whitelist_values`

- 值类型是[hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为{}

仅当指定字段的值与提供的正则表达式之一匹配时才包括它们。如果字段值是数组，则每个数组项与正则表达式匹配，并且仅包含匹配的数组项。

```sh
filter {
  prune {
    whitelist_values => [ "uripath", "/index.php",
                          "method", "(GET|POST)",
                          "status", "^[^2]" ]
  }
}
```

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
  prune {
    add_field => { "foo_%{somefield}" => "Hello world, from %{host}" }
  }
}
```

```sh
# You can also add multiple fields at once:
filter {
  prune {
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
  prune {
    add_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also add multiple tags at once:
filter {
  prune {
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
  prune {
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
  prune {
    remove_field => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple fields at once:
filter {
  prune {
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
  prune {
    remove_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple tags at once:
filter {
  prune {
    remove_tag => [ "foo_%{somefield}", "sad_unwanted_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将删除标记 foo_hello（如果存在）。第二个例子也会删除一个不需要的标签。
