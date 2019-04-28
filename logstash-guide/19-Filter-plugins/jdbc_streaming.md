## jdbc_streaming

- 插件版本：v1.0.5
- 发布于：2019-02-04
- [更新日志](https://github.com/logstash-plugins/logstash-filter-jdbc_streaming/blob/v1.0.5/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/filter-jdbc_streaming-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-filter-jdbc_streaming) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

此过滤器执行SQL查询并将结果集存储在指定为目标的字段中。它会将结果本地缓存到LRU缓存中，并且到期

例如，您可以根据事件中的id加载一行

```ruby
filter {
  jdbc_streaming {
    jdbc_driver_library => "/path/to/mysql-connector-java-5.1.34-bin.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydatabase"
    jdbc_user => "me"
    jdbc_password => "secret"
    statement => "select * from WORLD.COUNTRY WHERE Code = :code"
    parameters => { "code" => "country_code"}
    target => "country_details"
  }
}
```

### Jdbc_streaming过滤器配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 配置项                     | 输入类型                                                   | 必须 |
| ------------------------------------------------------- | ------------------------------------------------------------ | -------- |
| [`cache_expiration`](#cacheexpiration)                 | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否       |
| [`cache_size`](#cachesize)                             | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否       |
| [`default_hash`](#defaulthash)                         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`jdbc_connection_string`](#jdbcconnectionstring)     | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | Yes      |
| [`jdbc_driver_class`](#jdbcdriverclass)               | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 是       |
| [`jdbc_driver_library`](#jdbcdriverlibrary)           | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否       |
| [`jdbc_password`](#jdbcpassword)                       | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否       |
| [`jdbc_user`](#jdbcuser)                               | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否       |
| [`jdbc_validate_connection`](#jdbcvalidateconnection) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否       |
| [`jdbc_validation_timeout`](#jdbcvalidationtimeout)   | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否       |
| [`parameters`](#jdbcstreaming-parameters)              | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否       |
| [`statement`](#statement)                               | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 是       |
| [`tag_on_default_use`](#tagondefaultuse)             | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否       |
| [`tag_on_failure`](#tagonfailure)                     | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否       |
| [`target`](#target)                                     | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 是       |
| [`use_cache`](#usecache)                               | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否       |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### cache_expiration

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `5.0`

任何条目应保留在缓存中的最小秒数，默认为5秒数值，您可以使用小数，例如 `{ "cache_expiration" => 0.25 }`。如果存在瞬态jdbc错误，则缓存将存储给定的空结果参数设置和绕过jbdc查找，这将default_hash合并到事件中，直到缓存条目到期，然后将再次尝试jdbc查找相同的参数相反，虽然缓存包含有效结果任何可能导致jdbc错误的外部问题，对于cache_expiration期间不会被注意到。

##### cache_size

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `500`

存储最大缓存条目数，默认为500个条目，最近最少使用的条目将被逐出

##### default_hash

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为 `{}`

定义在查找无法返回匹配行时使用的默认对象。确保此对象的键名与语句中的列匹配

##### jdbc_connection_string

- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

JDBC连接字符串

##### jdbc_driver_class

- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

要加载的JDBC驱动程序类，例如 "oracle.jdbc.OracleDriver" 或 "org.apache.derby.jdbc.ClientDriver"

##### jdbc_driver_library

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

试图将JDBC逻辑抽象为mixin，以便在其他插件中进行重用（输入/输出）当有人包含此模块时调用此方法将这些方法添加到给定的基础。 JDBC驱动程序库到第三方驱动程序库的路径。

##### jdbc_password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

JDBC密码

##### jdbc_user

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

JDBC用户

##### jdbc_validate_connection

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

连接池配置。使用前验证连接。

##### jdbc_validation_timeout

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `3600`

连接池配置。验证连接的频率（以秒为单位）

##### parameters

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为 `{}`

查询参数的哈希值，例如 `{ "id" => "id_field" }`

##### statement

- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

要执行的声明。 要使用参数，请使用命名参数语法，例如"SELECT * FROM MYTABLE WHERE ID =:id"

##### tag_on_default_use

- 值类型是[array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `["_jdbcstreamingdefaultsused"]`

如果未找到记录并使用默认值，则将值附加到 `tag` 字段

##### tag_on_failure

- 值类型是[array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `["_jdbcstreamingfailure"]`

如果发生sql错误，请将值附加到 `tag` 字段

##### target

- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

定义目标字段以存储提取的结果如果存在，则覆盖字段

##### use_cache

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为true

启用或禁用缓存，布尔值为true或false，默认为true

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
  jdbc_streaming {
    add_field => { "foo_%{somefield}" => "Hello world, from %{host}" }
  }
}
```

```sh
# You can also add multiple fields at once:
filter {
  jdbc_streaming {
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
  jdbc_streaming {
    add_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also add multiple tags at once:
filter {
  jdbc_streaming {
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
  jdbc_streaming {
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
  jdbc_streaming {
    remove_field => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple fields at once:
filter {
  jdbc_streaming {
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
  jdbc_streaming {
    remove_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple tags at once:
filter {
  jdbc_streaming {
    remove_tag => [ "foo_%{somefield}", "sad_unwanted_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将删除标记 foo_hello（如果存在）。第二个例子也会删除一个不需要的标签。
