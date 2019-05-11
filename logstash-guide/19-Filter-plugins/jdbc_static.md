## jdbc_static

- 插件版本 v1.0.6
- 发布于：2018-10-29
- [更新日志](https://github.com/logstash-plugins/logstash-filter-jdbc_static/blob/v1.0.6/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/filter-jdbc_static-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-filter-jdbc_static) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

此过滤器使用从远程数据库预加载的数据来丰富事件数据。

此过滤器最适合使用静态或不经常更改的参考数据（如环境，用户和产品）来丰富事件数据。

此过滤器的工作方式是从远程数据库获取数据，在本地内存中的 [Apache Derby](https://db.apache.org/derby/manuals/#docs_10.14) 数据库中缓存数据，并通过查找本地数据库中缓存的数据来丰富事件。您可以设置过滤器，以加载远程数据一次（对于静态数据），或者可以安排远程加载定期运行（对于需要刷新的数据）。

要定义过滤器，请指定三个主要部分：local_db_objects，loaders和lookups。

**local_db_objects**

​	定义用于构建本地数据库结构的列，类型和索引。列名和类型应与外部数据库匹配。根据需要定义许多这些对象以构建本地数据库结构。

**loaders**

​	查询外部数据库以获取将在本地缓存的数据集。根据需要定义尽可能多的加载器以获取远程数据。每个加载器都应该填充 `local_db_objects` 定义的表。确保SQL语句中加载的列名和数据类型，与 `local_db_objects` 下定义的列匹配。每个加载器都有独立的远程数据库连接。

**lookups**

​	在本地数据库上执行查询以丰富事件数据。根据需要定义尽可能多的查询，以便通过一次查询所有表来丰富事件数据。理想情况下，SQL语句应该只返回一行。任何行都将转换为Hash对象，并存储目标字段数组中。

​	以下config示例从远程数据库中提取数据，将其缓存在本地数据库中，并通过查询本地数据库中缓存的数据来丰富事件。

```sh
filter {
  jdbc_static {
    loaders => [ ①
      {
        id => "remote-servers"
        query => "select ip, descr from ref.local_ips order by ip"
        local_table => "servers"
      },
      {
        id => "remote-users"
        query => "select firstname, lastname, userid from ref.local_users order by userid"
        local_table => "users"
      }
    ]
    local_db_objects => [ ②
      {
        name => "servers"
        index_columns => ["ip"]
        columns => [
          ["ip", "varchar(15)"],
          ["descr", "varchar(255)"]
        ]
      },
      {
        name => "users"
        index_columns => ["userid"]
        columns => [
          ["firstname", "varchar(255)"],
          ["lastname", "varchar(255)"],
          ["userid", "int"]
        ]
      }
    ]
    local_lookups => [ ③
      {
        id => "local-servers"
        query => "select descr as description from servers WHERE ip = :ip"
        parameters => {ip => "[from_ip]"}
        target => "server"
      },
      {
        id => "local-users"
        query => "select firstname, lastname from users WHERE userid = :id"
        parameters => {id => "[loggedin_userid]"}
        target => "user" ④
      }
    ]
    # using add_field here to add & rename values to the event root
    add_field => { server_name => "%{[server][0][description]}" }
    add_field => { user_firstname => "%{[user][0][firstname]}" } ⑤
    add_field => { user_lastname => "%{[user][0][lastname]}" } ⑥
    remove_field => ["server", "user"]
    staging_directory => "/tmp/logstash/jdbc_static/import_data"
    loader_schedule => "* */2 * * *" # run loaders every 2 hours
    jdbc_user => "logstash"
    jdbc_password => "example"
    jdbc_driver_class => "org.postgresql.Driver"
    jdbc_driver_library => "/tmp/logstash/vendor/postgresql-42.1.4.jar"
    jdbc_connection_string => "jdbc:postgresql://remotedb:5432/ls_test_2"
  }
}
```

① 查询外部数据库以获取将在本地缓存的数据集。

② 定义用于构建本地数据库结构的列，类型和索引。 列名和类型应与外部数据库匹配。 表定义的顺序应与加载器查询的顺序相匹配。 请参阅  [加载列和local_db_object顺序依赖项](#加载列和localdbobject顺序依赖项)。

③ 在本地数据库上执行查询查询以丰富事件数据。

④ 指定查询结果中要存储的事件字段。 如果查找返回多个列，则数据将作为JSON对象存储在字段中。

⑤ ⑥　 从JSON对象获取数据，并将其存储在顶级事件字段中，以便在Kibana中进行分析。

这是一个完整的例子：

```sh
input {
  generator {
    lines => [
      '{"from_ip": "10.2.3.20", "app": "foobar", "amount": 32.95}',
      '{"from_ip": "10.2.3.30", "app": "barfoo", "amount": 82.95}',
      '{"from_ip": "10.2.3.40", "app": "bazfoo", "amount": 22.95}'
    ]
    count => 200
  }
}

filter {
  json {
    source => "message"
  }

  jdbc_static {
    loaders => [
      {
        id => "servers"
        query => "select ip, descr from ref.local_ips order by ip"
        local_table => "servers"
      }
    ]
    local_db_objects => [
      {
        name => "servers"
        index_columns => ["ip"]
        columns => [
          ["ip", "varchar(15)"],
          ["descr", "varchar(255)"]
        ]
      }
    ]
    local_lookups => [
      {
        query => "select descr as description from servers WHERE ip = :ip"
        parameters => {ip => "[from_ip]"}
        target => "server"
      }
    ]
    staging_directory => "/tmp/logstash/jdbc_static/import_data"
    loader_schedule => "*/30 * * * *"
    jdbc_user => "logstash"
    jdbc_password => "logstash??"
    jdbc_driver_class => "org.postgresql.Driver"
    jdbc_driver_library => "/Users/guy/tmp/logstash-6.0.0/vendor/postgresql-42.1.4.jar"
    jdbc_connection_string => "jdbc:postgresql://localhost:5432/ls_test_2"
  }
}

output {
  stdout {
    codec => rubydebug {metadata => true}
  }
}
```

假设加载程序从Postgres数据库中获取以下数据：

```shell
select * from ref.local_ips order by ip;
    ip     |         descr
-----------+-----------------------
 10.2.3.10 | Authentication Server
 10.2.3.20 | Payments Server
 10.2.3.30 | Events Server
 10.2.3.40 | Payroll Server
 10.2.3.50 | Uploads Server
```

基于IP的值，服务器的描述丰富了事件数据：

```shell
{
           "app" => "bazfoo",
      "sequence" => 0,
        "server" => [
        [0] {
            "description" => "Payroll Server"
        }
    ],
        "amount" => 22.95,
    "@timestamp" => 2017-11-30T18:08:15.694Z,
      "@version" => "1",
          "host" => "Elastics-MacBook-Pro.local",
       "message" => "{\"from_ip\": \"10.2.3.40\", \"app\": \"bazfoo\", \"amount\": 22.95}",
       "from_ip" => "10.2.3.40"
}
```

### 使用此插件对接多个管道

> **重要：**
>
> Logstash使用一个内存中的Apache Derby实例，作为整个JVM的查找数据库引擎。由于每个插件实例在共享Derby引擎中使用唯一数据库，因此与尝试创建和填充相同表的插件不存在冲突。无论插件是在单个管道中定义，还是在多个管道中定义，都是如此。但是，在设置过滤器后，您应该查看查询结果，并查看日志以验证操作是否正确。

### 加载列和local_db_object顺序依赖项

> **重要：**
>
> 出于加载程序性能原因，加载机制使用内置的Derby文件导入功能导入CSV文件，从而将远程数据添加到本地数据库。检索到的列按原样写入CSV文件，文件导入过程需要与 `local_db_object` 设置中指定的列的顺序一一对应。请确保此顺序。

### Jdbc_static过滤器配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                                                | 输入类型                                                     | 必须 |
| --------------------------------------------------- | ------------------------------------------------------------ | ---- |
| [`jdbc_connection_string`](#jdbcconnectionstring) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 是   |
| [`jdbc_driver_class`](#jdbcdriverclass)           | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 是   |
| [`jdbc_driver_library`](#jdbcdriverlibrary)       | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`jdbc_password`](#jdbcpassword)                   | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`jdbc_user`](#jdbcuser)                           | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`tag_on_failure`](#tagonfailure)                 | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`tag_on_default_use`](#tagondefaultuse)         | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`staging_directory`](#stagingdirectory)           | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`loader_schedule`](#loaderschedule)               | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`loaders`](#loaders)                               | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`local_db_objects`](#localdbobjects)             | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`local_lookups`](#locallookups)                   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### jdbc_connection_string

- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

JDBC连接字符串。

##### jdbc_driver_class
- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

要加载的JDBC驱动程序类，例如 "org.apache.derby.jdbc.ClientDriver"。

> **注意：**
>
> 根据 [issue 43](https://github.com/logstash-plugins/logstash-input-jdbc/issues/43) ，如果您使用的是Oracle JDBC驱动程序（ojdbc6.jar），则正确的 `jdbc_driver_class` 是 `"Java::oracle.jdbc.driver.OracleDriver"`。

##### jdbc_driver_library
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

JDBC驱动程序库到第三方驱动程序库的路径。如果需要多个库，请在一个字符串中使用逗号分隔的路径。

如果未提供驱动程序类，则插件会在Logstash Java类路径中查找它。

##### jdbc_password
- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

JDBC密码。

##### jdbc_user
- 这是必需的设置。
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

JDBC用户。

##### tag_on_default_use
- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `["_jdbcstaticdefaultsused"]`

如果未找到记录并使用默认值，则将值附加到 `tag` 字段。

##### tag_on_failure
- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `["_jdbcstaticfailure"]`

如果发生SQL错误，则将值附加到 `tag` 字段。

##### staging_directory
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值派生自 Ruby临时目录+ plugin_name +"import_data"
- 例如 `"/tmp/logstash/jdbc_static/import_data"`

使用的目录将数据分段用于批量加载，应该有足够的磁盘空间，来处理您希望用于丰富事件的数据。由于Apache Derby中的开放错误，此插件的早期版本无法很好地处理超过数千行的加载数据集。此设置引入了加载大型记录集的另一种方法。收到每一行后，它会假脱机到文件，然后使用系统导入表系统调用导入该文件。

如果发生SQL错误，则将值附加到标记字段。

##### loader_schedule
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

您可以根据特定计划定期运行远程加载。此调度语法由 [rufus-scheduler](https://github.com/jmettraux/rufus-scheduler) 提供支持。语法类似于cron，具有特定于Rufus的一些扩展（例如，时区支持）。有关此语法的更多信息，请参阅 [解析cronlines和时间字符串](#https://github.com/jmettraux/rufus-scheduler#parsing-cronlines-and-time-strings)。

例子：

| 语法                        | 说明                                   |
| --------------------------- | -------------------------------------- |
| `*/30 * * * *`              | 每年1-3月的每天上午5点，每分钟执行一次 |
| `* 5 * 1-3 *`               | 每年1-3月的每天上午5点，每分钟执行一次 |
| `0 * * * *`                 | 每天每小时的第0分钟执行                |
| `0 6 * * * America/Chicago` | UTC/GMT -5时区，每天上午6点执行        |

使用如下Logstash内置shell命令调试：

```shell
bin/logstash -i irb
irb(main):001:0> require 'rufus-scheduler'
=> true
irb(main):002:0> Rufus::Scheduler.parse('*/10 * * * *')
=> #<Rufus::Scheduler::CronLine:0x230f8709 @timezone=nil, @weekdays=nil, @days=nil, @seconds=[0], @minutes=[0, 10, 20, 30, 40, 50], @hours=nil, @months=nil, @monthdays=nil, @original="*/10 * * * *">
irb(main):003:0> exit
```

上面调用返回的对象，`Rufus::Scheduler::CronLine` 的一个实例显示了执行的秒数，分钟数等。

##### loaders

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

该数组应包含一个或多个哈希。每个哈希都根据下表进行验证。

| 设置                   | Input type              | Required |
| ---------------------- | ----------------------- | -------- |
| id                     | string                  | No       |
| table                  | string                  | Yes      |
| query                  | string                  | Yes      |
| max_rows               | number                  | No       |
| jdbc_connection_string | string                  | No       |
| jdbc_driver_class      | string                  | No       |
| jdbc_driver_library    | a valid filesystem path | No       |
| jdbc_password          | password                | No       |
| jdbc_user              | string                  | No       |

**loaders字段描述：**

​	**id**

​	可选标识符。这用于标识生成错误消息和日志行的加载程序。

​	**table**

​	Loader将填充的本地查找数据库中的目标表。

​	**query**

​	为获取远程记录而执行的SQL语句。使用SQL别名和强制类型转换，来确保记录的列和数据类型与 `local_db_objects` 中定义的本地数据库中的表结构相匹配。

​	**max_rows**

​	此设置的默认值为100万。由于查找数据库是内存中的，因此它将占用JVM堆空间。如果查询返回数百万行，则应增加给予Logstash的JVM内存或限制返回的行数，可能是对事件数据中最常见的行数。

​	**jdbc_connection_string**

​	如果未在加载程序中设置，则此设置默认为插件级别 `jdbc_connection_string` 设置。

​	**jdbc_driver_class**

​	如果未在加载程序中设置，则此设置默认为插件级 `jdbc_driver_class` 设置。

​	**jdbc_driver_library**

​	如果未在加载程序中设置，则此设置默认为插件级别 `jdbc_driver_library` 设置。

​	**jdbc_password**

​	如果未在加载程序中设置，则此设置默认为插件级别 `jdbc_password` 设置。

​	**jdbc_user**

​	如果未在加载程序中设置，则此设置默认为插件级别 `jdbc_user` 设置。

##### local_db_objects

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

该数组应包含一个或多个哈希。每个Hash代表本地查找数据库的表模式。每个哈希都根据下表进行验证。

| Setting           | Input type | Required |
| ----------------- | ---------- | -------- |
| name              | string     | Yes      |
| columns           | array      | Yes      |
| index_columns     | number     | No       |
| preserve_existing | boolean    | No       |

**Local_db_objects字段描述：**

​	**name**

​	要在数据库中创建的表的名称。

​	**columns**

​	列声明数组。每个列声明都是两个元素的数组，例如 `["ip", "varchar(15)"]`。第一个元素是列名字符串。第二个元素是一个 [Apache Derby SQL类型](https://db.apache.org/derby/docs/10.14/ref/crefsqlj31068.html) 的字符串。在构建本地查找表时检查字符串内容，而不是在验证设置时检查。因此，任何拼写错误的SQL类型字符串都会导致错误。

​	**index_columns**

​	一串字符串。必须在列设置中定义每个字符串。索引名称将在内部生成。不支持唯一或排序的索引。

​	**preserve_existing**

​	此设置为 `true` 时，检查表是否已存在于本地查找数据库中。如果您在同一个Logstash实例中运行多个管道，并且多个管道正在使用此插件，那么您必须阅读页面顶部的重要多管道通知。

**local_lookups**

- 值类型是数组
- 默认值为 `[]`

该数组应包含一个或多个哈希。每个Hash代表一个查找丰富。每个哈希都根据下表进行验证。

| Setting            | Input type | Required |
| ------------------ | ---------- | -------- |
| id                 | string     | No       |
| query              | string     | Yes      |
| parameters         | hash       | Yes      |
| target             | string     | No       |
| default_hash       | hash       | No       |
| tag_on_failure     | string     | No       |
| tag_on_default_use | string     | No       |

**Local_lookups字段描述：**

​	**id**

​	可选标识符。这用于标识生成错误消息和日志行的查找。如果省略此设置，则使用默认ID。

​	**query**

​	执行SQL SELECT语句以实现查找。要使用参数，请使用命名参数语法，例如 `"SELECT * FROM MYTABLE WHERE ID = :id"`。

​	**parameters**

​	键/值哈希或字典。键（LHS）是在SQL语句 `SELECT * FROM sensors WHERE reference = :p1` 中替换的文本。值（RHS）是事件中的字段名称。插件从事件中读取该键的值，并将该值替换为语句，例如， `parameters => { "p1" => "ref" }`。引用是自动的——您不需要在语句中加引号。如果需要添加前缀/后缀或将两个事件字段值连接在一起以构建替换值，则仅在RHS上使用字段插值语法。例如，假设一条具有id和位置的IOT消息，并且您有一个具有 `id-loc_id` 列的传感器表。在这种情况下，您的参数哈希将如下所示： `parameters => { "p1" => "%{[id]}-%{[loc_id]}" }`。

​	**target**

​	将接收查找数据的字段的可选名称。如果省略此设置，则使用id设置（或默认ID）。查找的数据（转换为Hashes的结果数组）永远不会添加到事件的根目录中。如果要执行此操作，则应使用 `add_field` 设置。这意味着您可以完全控制字段/值如何放在事件的根目录中，例如，add_field => { user_firstname => "%{[user][0][firstname]}" } —— 其中 `[user]` 是目标字段，`[0]` 是数组中的第一个结果，`[firstname]` 是结果哈希中的键。

​	**default_hash**

​	当查找未返回任何结果时，将放在目标字段数组中的可选哈希。如果您需要确保配置的其他部分中的后续引用实际引用某些内容，请使用此设置。

​	**tag_on_failure**

​	一个可选字符串，它覆盖插件级设置。这在定义多个查找时很有用。

​	**tag_on_default_use**

​	一个可选字符串，它覆盖插件级设置。这在定义多个查找时很有用。

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
  jdbc_static {
    add_field => { "foo_%{somefield}" => "Hello world, from %{host}" }
  }
}
```

```sh
# You can also add multiple fields at once:
filter {
  jdbc_static {
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
  jdbc_static {
    add_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also add multiple tags at once:
filter {
  jdbc_static {
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
  jdbc_static {
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
  jdbc_static {
    remove_field => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple fields at once:
filter {
  jdbc_static {
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
  jdbc_static {
    remove_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple tags at once:
filter {
  jdbc_static {
    remove_tag => [ "foo_%{somefield}", "sad_unwanted_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将删除标记 foo_hello（如果存在）。第二个例子也会删除一个不需要的标签。

