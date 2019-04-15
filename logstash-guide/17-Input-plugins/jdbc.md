## jdbc

- 插件版本：v4.3.13
- 发布于：2018-09-04
- [更新日志](https://github.com/logstash-plugins/logstash-input-jdbc/blob/v4.3.13/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/input-jdbc-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-input-jdbc) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

此插件可以从JDBC数据库中提取数据到Logstash中。您可以使用cron语法定期抽取（请参阅 `schedule` 设置），或运行查询将数据加载到Logstash中。结果集中的每一行都成为一个事件，将列转换为事件中的字段。

### 驱动

此插件未集成JDBC驱动库。必须配置 `jdbc_driver_library` 选项，并将所需的jdbc驱动库显式载入插件。

### 定时任务

设置此插件的输入根据特定时间表定期运行。此调度语法由 [rufus-scheduler](https://github.com/jmettraux/rufus-scheduler) 提供支持。语法类似于cron，具有特定于Rufus的一些扩展（例如，时区支持）。

例子：

| 语法                        | 说明                                   |
| --------------------------- | -------------------------------------- |
| `* 5 * 1-3 *`               | 每年1-3月的每天上午5点，每分钟执行一次 |
| `0 * * * *`                 | 每天每小时的第0分钟执行                |
| `0 6 * * * America/Chicago` | UTC/GMT -5时区，每天上午6点执行        |

可以在此处找到描述此语法的 [更多文档](https://github.com/jmettraux/rufus-scheduler#parsing-cronlines-and-time-strings)。

### 状态

该插件将以元数据的格式持久化 `sql_last_value` 参数，存储在的 `last_run_metadata_path` 的文件中。执行查询时，将使用 `sql_last_value` 的当前值覆盖此文件中的参数。下次管道启动时，将通过读取文件来更新此值。如果 `clean_run` 设置为 `true`，则将忽略此值，并且`sql_last_value` 将设置为1970年1月1日，如果 `use_column_value` 为true，则为0，就好像没有执行任何查询一样。

### 大数据结果集

许多JDBC驱动程序使用 `fetch_size` 参数，来限制从游标到客户端缓存中一次预取的结果数，然后从结果集中检索更多结果。这是使用 `jdbc_fetch_size` 参数进行配置的。此插件默认情况下未设置提取大小，因此将使用指定驱动程序的默认大小。

### 用法

以下是设置插件以从MySQL数据库获取数据的示例。首先，我们将适当的JDBC驱动程序库放在当前路径中（这可以放在文件系统的任何位置）。在这个例子中，我们使用 user:mysql 连接到mydb数据库，并希望在songs表中输入与特定艺术家匹配的所有行。以下示例演示了可能的Logstash配置。此示例中的schedule选项将使插件每分钟执行此输入语句。

```ruby
input {
  jdbc {
    jdbc_driver_library => "mysql-connector-java-5.1.36-bin.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
    jdbc_user => "mysql"
    parameters => { "favorite_artist" => "Beethoven" }
    schedule => "* * * * *"
    statement => "SELECT * from songs where artist = :favorite_artist"
  }
}
```

### 配置SQL

此输入需要sql语句。这可以通过字符串形式传入，也可以从文件（`statement_filepath`）读取。当SQL语句很大或很难在配置中提供时，通常使用文件选项。 文件选项仅支持一个SQL语句。该插件只接受其中一种形式。它无法同时从文件和语句配置参数中读取语句。

### 配置多个SQL

当需要从不同数据库表或视图中提取数据时，配置多个SQL语句非常有用。可以为每个语句定义单独的Logstash配置文件，也可以在单个配置文件中定义多个语句。在单个Logstash配置文件中使用多个语句时，必须将每个语句定义为单独的jdbc输入（包括jdbc驱动程序，连接字符串和其他必需参数）。

请注意，如果任何语句使用 `sql_last_value` 参数（例如，仅用于采集自上次运行以来的增量更改数据），则每个输入应定义其自己的 `last_run_metadata_path` 参数。如果不这样做所有输入都会将其状态存储到相同（默认）元数据文件中，会覆盖彼此的 `sql_last_value`，而导致意外的结果。

### 预定义参数

某些参数是内置的，可以在查询中使用。 如下：

`sql_last_value` ：用于计算要查询的行的值。 在运行首次查询之前，其被设置为1970年1月1日星期四，如果 `use_column_value` 为true，并且设置了 `tracking_column`，则设置为0。 在后续查询运行后，它会相应更新。

例：

```ruby
input {
  jdbc {
    statement => "SELECT id, mycolumn1, mycolumn2 FROM my_table WHERE id > :sql_last_value"
    use_column_value => true
    tracking_column => "id"
    # ... other configuration bits
  }
}
```

### JDBC输入配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                                                         | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`clean_run`](#clean_run)                                    | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`columns_charset`](#columns_charset)                        | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`connection_retry_attempts`](#connection_retry_attempts)    | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`connection_retry_attempts_wait_time`](#connection_retry_attempts_wait_time) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`jdbc_connection_string`](#jdbc_connection_string)          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 是   |
| [`jdbc_default_timezone`](#jdbc_default_timezone)            | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`jdbc_driver_class`](#jdbc_driver_class)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 是   |
| [`jdbc_driver_library`](#jdbc_driver_library)                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`jdbc_fetch_size`](#jdbc_fetch_size)                        | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`jdbc_page_size`](#jdbc_page_size)                          | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`jdbc_paging_enabled`](#jdbc_paging_enabled)                | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`jdbc_password`](#jdbc_password)                            | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password) | 否   |
| [`jdbc_password_filepath`](#jdbc_password_filepath)          | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path) | 否   |
| [`jdbc_pool_timeout`](#jdbc_pool_timeout)                    | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`jdbc_user`](#jdbc_user)                                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`jdbc_validate_connection`](#jdbc_validate_connection)      | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`jdbc_validation_timeout`]#jdbc_validation_timeout)         | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`last_run_metadata_path`](#last_run_metadata_path)          | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`lowercase_column_names`](#lowercase_column_names)          | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`parameters`](#parameters)                                  | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`record_last_run`](#record_last_run)                        | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`schedule`](#schedule)                                      | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`sequel_opts`](#sequel_opts)                                | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`sql_log_level`](#sql_log_level)                            | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，值包含 `["fatal", "error", "warn", "info", "debug"]` | 否   |
| [`statement`](#statement)                                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`statement_filepath`](#statement_filepath)                  | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path) | 否   |
| [`tracking_column`](#tracking_column)                        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`tracking_column_type`](#tracking_column_type)              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，值包含`["numeric", "timestamp"]` | 否   |
| [`use_column_value`](#use_column_value)                      | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### clean_run

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为false

是否应保留先前的运行状态

##### columns_charset

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash)
- 默认值为 `{}`

特定列的字符编码。此选项将覆盖指定列的：`charset` 选项。

例：

```ruby
input {
  jdbc {
    ...
    columns_charset => { "column0" => "ISO-8859-1" }
    ...
  }
}
```

这只会将具有ISO-8859-1的column0转换为原始编码。

##### connection_retry_attempts

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `1`

尝试连接数据库的最大次数

##### connection_retry_attempts_wait_time

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为0.5

连接尝试之间休眠的秒数

##### jdbc_connection_string

- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

JDBC连接字符串

##### jdbc_default_timezone

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

时区转换。SQL不允许时间戳字段中的时区数据。此插件将自动将SQL时间戳字段转换为Logstash时间戳，采用ISO8601格式的相对UTC时间。

使用此设置将手动分配指定的时区偏移量，而不是使用本地计算机的时区设置。例如，您必须使用规范时区， **America/Denver**。

##### jdbc_driver_class

- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

如果您使用的是Oracle JDBC，则需要加载JDBC驱动程序类，例如"org.apache.derby.jdbc.ClientDriver" NB https://github.com/logstash-plugins/logstash-input-jdbc/issues/43 驱动（ojdbc6.jar）正确的 `jdbc_driver_class` 是 `"Java::oracle.jdbc.driver.OracleDriver"`

##### jdbc_driver_library

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

试图将JDBC逻辑抽象为mixin，以便在其他插件中进行重用（输入/输出）当有人包含此模块时调用此方法将这些方法添加到给定的基础。 JDBC驱动程序库到第三方驱动程序库的路径。如果需要多个库，您可以用逗号分隔它们。

如果没有提供，Plugin将在Logstash Java类路径中查找驱动程序类。

##### jdbc_fetch_size

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 此设置没有默认值。

JDBC获取大小。如果没有提供，将使用相应的驱动程序默认值

##### jdbc_page_size

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `100000`

JDBC页面大小

##### jdbc_paging_enabled

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `false`

JDBC启用分页

这将导致sql语句被分解为多个查询。每个查询将使用限制和偏移来共同检索完整的结果集。限制大小使用 `jdbc_page_size` 设置。

请注意，查询顺序无法保证。

##### jdbc_password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password)
- 此设置没有默认值。

JDBC密码

##### jdbc_password_filepath

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path)
- 此设置没有默认值。

JDBC密码文件名

##### jdbc_pool_timeout

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `5`

连接池配置。在引发PoolTimeoutError之前等待获取连接的秒数（默认值为5）

##### jdbc_user

- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

JDBC用户

##### jdbc_validate_connection

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `false`

连接池配置。使用前验证连接。

##### jdbc_validation_timeout

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `3600`

连接池配置。验证连接的频率（以秒为单位）

##### last_run_metadata_path

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 默认值为 `"$HOME/.logstash_jdbc_last_run"`

上次运行时文件的路径

##### lowercase_column_names

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `true`

是否强制标识符字段的小写

##### parameters

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash)
- 默认值为 `{}`

查询参数的哈希值，例如 `{ "target_id" => "321" }`

##### record_last_run

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `true`

是否在 [`last_run_metadata_path`](#last_run_metadata_path) 中保存状态

##### schedule

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

以Cron格式定期运行语句的时间表，例如："* * * * *"（每分钟执行查询）

默认情况下没有计划。如果没有给出计划，那么该语句只运行一次。

##### sequel_opts

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash)
- 默认值为 `{}`

一般/供应商特定的续集配置选项。

可选连接池配置示例 max_connections  — 连接池的最大连接数

特定于供应商的选项的示例可以在本文档页面中找到：https://github.com/jeremyevans/sequel/blob/master/doc/opening_databases.rdoc

##### sql_log_level

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，值包含 `["fatal", "error", "warn", "info", "debug"]`
- 默认值为 `"info"`

记录SQL查询的日志级别，接受的值是常见的致命，错误，警告，信息和调试。默认值为info。

##### statement

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

如果未定义，即使编解码器未使用，Logstash会提醒要执行的声明。

要使用参数，请使用命名参数语法。例如：

```ruby
"SELECT * FROM MYTABLE WHERE id = :target_id"
```

此处，":target_id" 是一个命名参数。您可以使用 `parameters` 配置命名参数。

##### statement_filepath

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path)
- 此设置没有默认值。

包含要执行的语句的文件路径

##### tracking_column

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

如果 `use_column_value` 设置为 `true`，则要跟踪其值的列

##### tracking_column_type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，值包含 `numeric`, `timestamp`
- 默认值为 `"numeric"`

跟踪列的类型。目前只有 "numeric" 和 "timestamp"

##### use_column_value

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `false`

设置为 `true` 时，使用定义的[` tracking_column`](#tracking_column) 值作为 `:sql_last_value`。设置为 `false` 时，`:sql_last_value` 反映上次执行查询的时间。

### 通用配置项

所有输入插件都支持以下配置选项：

| 设置                              | 输入类型                                                     | 必须 |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| [`add_field`](#add_field)         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`codec`](#codec)                 | [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Codec) | 否   |
| [`enable_metric`](#enable_metric) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`id`](#id)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`tags`](#tags)                   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`type`](#type)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |

#### 详情

##### add_field

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash)
- 默认值为 `{}`

向事件添加字段

##### codec

- 值类型是 [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Codec)
- 默认值是 `"plain"`

用于输入数据的编解码器。输入编解码器是一种在输入之前解码数据的便捷方法，无需在Logstash管道中使用单独的过滤器。

##### enable_metric

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `true`

默认情况下，禁用或启用此特定插件实例的指标记录，我们会记录所有的可用指标，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
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

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 此设置没有默认值。

为您的活动添加任意数量的任意标签。

这有助于后续处理。

##### type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

将 `type` 字段添加到此输入处理的所有事件。

类型主要用于过滤器激活。

类型存储为事件本身的一部分，因此您也可以使用该类型在Kibana中搜索它。

如果您尝试在已有事件的事件上设置类型（例如，当您将事件从发运人发送到索引器时），则新输入将不会覆盖现有类型。发运人设置的类型即使在发送到另一个Logstash服务器时，仍会保留。