## mutate

- 插件版本：v3.4.0
- 发布于：2019-02-04
- [更新日志](https://github.com/logstash-plugins/logstash-filter-mutate/blob/v3.4.0/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/filter-mutate-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-filter-mutate) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

mutate过滤器允许您在字段上执行常规修改。 您可以重命名，删除，替换和修改事件中的字段。

#### 处理顺序

配置文件中的修改按以下顺序执行：

- coerce
- rename
- update
- replace
- convert
- gsub
- uppercase
- capitalize
- lowercase
- strip
- remove
- split
- join
- merge
- copy

您可以通过定义多个mutate块来控制顺序。

例：

```ruby
filter {
    mutate {
        split => ["hostname", "."]
        add_field => { "shortHostname" => "%{hostname[0]}" }
    }

    mutate {
        rename => ["shortHostname", "hostname" ]
    }
}
```

### Mutate过滤器配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 配置项                     | 输入类型                                                   | 必须 |
| --------------------------- | ------------------------------------------------------------ | -------- |
| [`convert`](#convert)       | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`copy`](#copy)             | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`gsub`](#gsub)             | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否       |
| [`join`](#join)             | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`lowercase`](#lowercase)   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否       |
| [`merge`](#merge)           | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`coerce`](#coerce)         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`rename`](#rename)         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`replace`](#replace)       | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`split`](#split)           | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`strip`](#strip)           | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否       |
| [`update`](#update)         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`uppercase`](#uppercase)   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否       |
| [`capitalize`](#capitalize) | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否       |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### convert

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

将字段的值转换为其他类型，例如将字符串转换为整数。如果字段值是数组，则将转换所有元素。如果该字段是哈希，则不会采取任何操作。

有效的转换目标及其对不同输入的预期行为是：

- integer：
  - 字符串被解析；支持逗号分隔符（例如，字符串 `"1,000"` 产生一个值为1000的整数）；当字符串有小数部分时，它们会被截断。
  - 浮点数和小数被截断（例如，`3.99` 变为 `3`，`-2.7` 变为 `-2`）
  - boolean true和boolean false分别转换为 `1` 和 `0`

- integer_eu：
  - 与整数相同，但字符串值支持点分隔符和逗号小数（例如，`"1.000"` 生成一个值为1000的整数）

- float：
  - 整数转换为浮点数
  - 字符串被解析；支持逗号分隔符和点小数（例如，`"1,000.5"` 生成一个值为 1000.5 的整数）
  - boolean true和boolean false分别转换为 `1.0` 和 `0.0`
- float_eu：
  - 与float相同，除了字符串值支持点分隔符和逗号小数（例如，`"1.000,5"` 产生一个值为一千零一半的整数）

- string：
  - 所有值都使用UTF-8进行字符串化和编码

- boolean：
  - 整数0转换为布尔值 `false`
  - 整数1转换为布尔值 `true`
  - float 0.0转换为布尔值 `false`
  - float 1.0转换为布尔值 `true`
  - 字符串 `"true"`，`"t"`，`"yes"`，`"y"`，`"1" `和 `"1.0" `转换为布尔值 `true`
  - 字符串 `"false"`，`"f"`，`"no"`，`"n"`，`"0" `和 `"0.0"` 转换为布尔值 `false`
  - 空字符串转换为布尔值 `false`
  - 所有其他值在没有转换的情况下直接传递并记录警告消息
  - 对于数组，使用上面的规则分别处理每个值

此插件可以转换同一文档中的多个字段，请参阅下面的示例。

例：

```ruby
filter {
  mutate {
    convert => {
      "fieldname" => "integer"
      "booleanfield" => "boolean"
    }
  }
}
```

##### copy

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

将现有字段复制到另一个字段。将覆盖现有目标字段。

例：

```ruby
filter {
  mutate {
     copy => { "source_field" => "dest_field" }
  }
}
```

##### gsub

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 此设置没有默认值。

将正则表达式与字段值匹配，并将所有匹配替换为替换字符串。仅支持字符串或字符串数组的字段。对于其他类型的领域，将不采取任何行动。

此配置采用每个字段/替换包含3个元素的数组。

请注意转义配置文件中的任何反斜杠。

例：

```ruby
filter {
  mutate {
    gsub => [
      # replace all forward slashes with underscore
      "fieldname", "/", "_",
      # replace backslashes, question marks, hashes, and minuses
      # with a dot "."
      "fieldname2", "[\\?#-]", "."
    ]
  }
}
```

##### join

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

使用分隔符加入数组。在非数组字段上什么都不做。

例：

```ruby
filter {
  mutate {
    join => { "fieldname" => "," }
  }
}
```

##### lowercase

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 此设置没有默认值。

将字符串转换为其小写等效项。

例：

```ruby
filter {
  mutate {
    lowercase => [ "fieldname" ]
  }
}
```

##### merge

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

合并两个数组或散列字段。字符串字段将自动转换为数组，因此：

```
`array` + `string` will work
`string` + `string` will result in an 2 entry array in `dest_field`
`array` and `hash` will not work
```

例：

```ruby
filter {
  mutate {
     merge => { "dest_field" => "added_field" }
  }
}
```

##### coerce

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

设置存在但仍为null的字段的默认值

例：

```ruby
filter {
  mutate {
    # Sets the default value of the 'field1' field to 'default_value'
    coerce => { "field1" => "default_value" }
  }
}
```

##### rename

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

重命名一个或多个字段。

例：

```ruby
filter {
  mutate {
    # Renames the 'HOSTORIP' field to 'client_ip'
    rename => { "HOSTORIP" => "client_ip" }
  }
}
```

##### replace

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

用新值替换字段的值。新值可以包含 `%{foo}` 字符串，以帮助您从事件的其他部分构建新值。

例：

```ruby
filter {
  mutate {
    replace => { "message" => "%{source_host}: My new message" }
  }
}
```

##### split

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

使用分隔符将字段拆分为数组。仅适用于字符串字段。

例：

```ruby
filter {
  mutate {
     split => { "fieldname" => "," }
  }
}
```

##### strip

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 此设置没有默认值。

从字段中取出空格。注意：这仅适用于前导和尾随空格。

例：

```ruby
filter {
  mutate {
     strip => ["field1", "field2"]
  }
}
```

##### update

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

使用新值更新现有字段。 如果该字段不存在，则不执行任何操作。

例：

```ruby
filter {
  mutate {
    update => { "sample" => "My new message" }
  }
}
```

##### uppercase

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 此设置没有默认值。

将字符串转换为大写的等效字符串。

例：

```ruby
filter {
  mutate {
    uppercase => [ "fieldname" ]
  }
}
```

##### capitalize

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 此设置没有默认值。

将字符串转换为大写等效字符串。

例：

```ruby
filter {
  mutate {
    capitalize => [ "fieldname" ]
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
  mutate {
    add_field => { "foo_%{somefield}" => "Hello world, from %{host}" }
  }
}
```

```sh
# You can also add multiple fields at once:
filter {
  mutate {
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
  mutate {
    add_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also add multiple tags at once:
filter {
  mutate {
    add_tag => [ "foo_%{somefield}", "taggedy_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将添加标记 `foo_hello`（第二个示例当然会添加 `taggedy_tag` 标记）。

##### enable_metric

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

默认情况下，禁用或启用此插件实例的指标记录，我们会记录所有可用的度量标准，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

为插件配置添加唯一 `ID`。如果未指定ID，Logstash将生成一个ID。强烈建议在配置中设置此ID。当您有两个或更多相同类型的插件时，这尤其有用，例如，如果您有2个http过滤器。在这种情况下添加命名ID将有助于在使用监视API时监视Logstash。

```sh
filter {
  mutate {
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
  mutate {
    remove_field => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple fields at once:
filter {
  mutate {
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
  mutate {
    remove_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple tags at once:
filter {
  mutate {
    remove_tag => [ "foo_%{somefield}", "sad_unwanted_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将删除标记 foo_hello（如果存在）。第二个例子也会删除一个不需要的标签。

