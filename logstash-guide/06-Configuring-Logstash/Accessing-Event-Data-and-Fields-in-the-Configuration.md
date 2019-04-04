## 配置中引用事件数据和字段

Logstash代理是一个包括3个阶段的处理管道：输入→过滤器→输出。输入阶段生成事件，过滤器修改它们，输出阶段将它们发送到目的地。

所有活动都有属性。例如，apache访问日志将包含状态代码（200，404），请求路径（"/"，"index.html"），HTTP类型（GET，POST），客户端IP地址等。Logstash可调用这些属性"字段"。

Logstash中的一些配置选项需要存在字段才能运行。事件在输入时生成，故在输入块中没有可以解析的字段，因为它们还不存在！

由于它们依赖于事件和字段，因此以下配置选项仅适用于过滤器和输出块。

> **重要：**
> 下面描述的字段引用，sprintf格式和条件不能用于输入块。

### 字段引用

能够通过名称引用字段通常很有用。为此，您可以使用Logstash的 [字段引用语法](../00-Others/Field-References-Deep-Dive.md)。

访问字段的基本语法是 `[fieldname]`。如果您引用的是顶级字段，则可以省略 `[]` 并简单地使用 `fieldname`。如需引用嵌套字段，请指定该字段的完整路径：`[顶级字段][嵌套字段]`。

例如，以下事件有五个顶级字段（agent, ip, request, response, ua）和三个嵌套字段（status, bytes, os）。

```json
{
  "agent": "Mozilla/5.0 (compatible; MSIE 9.0)",
  "ip": "192.168.24.44",
  "request": "/index.html",
  "response": {
    "status": 200,
    "bytes": 52353
  },
  "ua": {
    "os": "Windows 7"
  }
}
```

要引用 `os` 字段，请指定 `[ua][os]`。要引用顶级字段（如 `request`），只需指定字段名称即可。

更多详细信息，请参阅 [字段引用详解](../00-Others/Field-References-Deep-Dive.md)。

### sprintf 格式

字段引用格式也可用于Logstash调用sprintf格式的内容。此格式使您可以从其他字符串中引用字段值。例如，statsd输出具有增量设置，使您可以按状态代码保留apache日志的计数：

```yaml
output {
  statsd {
    increment => "apache.%{[response][status]}"
  }
}
```

同样，您可以将 `@timestamp` 字段中的时间戳转换为字符串。无需在花括号内指定字段名称，而是使用 `+FORMAT` 语法，其中 `FORMAT` 是 [时间格式](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html)。

例如，如果要根据事件的日期、小时、以及 `type` 字段使用通过文件输出插件输出到日志：

```yaml
output {
  file {
    path => "/var/log/%{type}.%{+yyyy.MM.dd.HH}"
  }
}
```

### 条件表达式

如果只想在特定条件下过滤或输出事件，可以使用条件表达式。

Logstash中的条件表达式的语法和功能与编程语言中的条件表达式类似。条件语支持 `if`、`else if` 和 `else` 语句并且可以嵌套。

条件语法如下：

```js
if EXPRESSION {
  ...
} else if EXPRESSION {
  ...
} else {
  ...
}
```

什么是表达式？比较测试，布尔逻辑等等！

您可以使用以下比较运算符：

- 等式类：`==`, `!=`, `<`, `>`, `<=`, `>=`
- 正则类：`=~`, `!~`（检查右边是否匹配左边的字符串值）
- 包含类：`in`, `not in`

支持的布尔运算符是：

- `and`, `or`, `nand`, `xor`

支持的一元运算符是：

- `!`

表达式可能很长很复杂，它可以包含其他表达式，您可以使用 `!` 来取消表达式，并且可以使用括号 `(...)` 对它们进行分组。

例如，如果字段 `action` 有一个值为 `login`，则以下条件使用mutate过滤器移除字段 `secret`：

```yaml
filter {
  if [action] == "login" {
    mutate { remove_field => "secret" }
  }
}
```

在单个条件中指定多个表达式：

```yaml
output {
  # Send production errors to pagerduty
  if [loglevel] == "ERROR" and [deployment] == "production" {
    pagerduty {
    ...
    }
  }
}
```

您可以使用 `in` 运算符来测试字段是否包含特定字符串、键或元素（对列表类型）：

```yaml
filter {
  if [foo] in [foobar] {
    mutate { add_tag => "field in field" }
  }
  if [foo] in "foo" {
    mutate { add_tag => "field in string" }
  }
  if "hello" in [greeting] {
    mutate { add_tag => "string in field" }
  }
  if [foo] in ["hello", "world", "foo"] {
    mutate { add_tag => "field in list" }
  }
  if [missing] in [alsomissing] {
    mutate { add_tag => "shouldnotexist" }
  }
  if !("foo" in ["hello", "world"]) {
    mutate { add_tag => "shouldexist" }
  }
}
```

可以相同的方式使用 `not in` 条件。例如，可以使用 `not in` 仅将通过 `grok` 的事件路由到Elasticsearch：

```yaml
output {
  if "_grokparsefailure" not in [tags] {
    elasticsearch { ... }
  }
}
```

可以检查特定字段是否存在，但目前无法区分不存在的字段与完全错误的字段。表达式 `if [foo]` 在以下情况下返回 `false`：

- `[foo]` 在活动中不存在，
- `[foo]` 存在于事件中，但为 `false`
- `[foo]` 存在于事件中，但为null

有关更复杂的示例，请参阅 [使用条件表达式](../06-Configuring-Logstash/Logstash-Configuration-Examples.md#使用条件表达式)。

### @metadata 字段

在Logstash 1.5及更高版本中，存在一个名为 `@metadata` 的特殊字段。 `@metadata` 的内容在输出时不会成为事件的一部分，这使得它非常适合用于条件、字段引用、及sprintf格式的扩展和构建事件字段。

以下配置文件将从STDIN生成事件。 输入任何文本都将成为事件中的 `message` 字段。 过滤器块中的 `mutate` 事件将添加一些字段，一些字段嵌套在 `@metadata` 字段中。

```yaml
input { stdin { } }

filter {
  mutate { add_field => { "show" => "This data will be in the output" } }
  mutate { add_field => { "[@metadata][test]" => "Hello" } }
  mutate { add_field => { "[@metadata][no_show]" => "This data will not be in the output" } }
}

output {
  if [@metadata][test] == "Hello" {
    stdout { codec => rubydebug }
  }
}
```

接下来看看得到了什么结果：

```shell
$ bin/logstash -f ../test.conf
Pipeline main started
asdf
{
    "@timestamp" => 2016-06-30T02:42:51.496Z,
      "@version" => "1",
          "host" => "example.com",
          "show" => "This data will be in the output",
       "message" => "asdf"
}
```

输入的 "asdf" 成为 `message` 字段内容，条件成功解析了嵌套在 `@metadata` 字段中的 `test` 字段的内容。 但是输出没有显示名为 `@metadata` 的字段或其内容。

如果添加了配置 `metadata => true`，`rubydebug` 编解码器允许您显示 `@metadata` 字段的内容：

```yaml
stdout { codec => rubydebug { metadata => true } }
```

接下来看看输出内容有什么变化：

```shell
$ bin/logstash -f ../test.conf
Pipeline main started
asdf
{
    "@timestamp" => 2016-06-30T02:46:48.565Z,
     "@metadata" => {
           "test" => "Hello",
        "no_show" => "This data will not be in the output"
    },
      "@version" => "1",
          "host" => "example.com",
          "show" => "This data will be in the output",
       "message" => "asdf"
}
```

现在可以看到 `@metadata` 字段及其子属性。

> **重要：**
>
> 仅 `rubydebug` 编解码器可以用来显示 `@metadata` 字段的内容。

当你需要使用临时字段，但不希望它在最终输出中，就可以使用 `@metadata` 字段。

这个新字段最常见的使用场景之一，可能是使用 `date` 过滤器，并使其具有临时时间戳。

以下简化过的配置文件使用Apache和Nginx服务器通用的时间戳格式。 以前，在使用它覆盖 `@timestamp` 字段后，您必须自己删除 `@timestamp` 字段。 现在有了 `@metadata` 字段，则不再需要如此操作：

```yaml
input { stdin { } }

filter {
  grok { match => [ "message", "%{HTTPDATE:[@metadata][timestamp]}" ] }
  date { match => [ "[@metadata][timestamp]", "dd/MMM/yyyy:HH:mm:ss Z" ] }
}

output {
  stdout { codec => rubydebug }
}
```

请注意，此配置将提取的日期放入 `grok` 过滤器的 `[@metadata] [timestamp]` 字段中。 让我们为这个配置提供一个示例日期字符串，看看会出现什么：

```shell
$ bin/logstash -f ../test.conf
Pipeline main started
02/Mar/2014:15:36:43 +0100
{
    "@timestamp" => 2014-03-02T14:36:43.000Z,
      "@version" => "1",
          "host" => "example.com",
       "message" => "02/Mar/2014:15:36:43 +0100"
}
```

就是这样！ 输出中没有额外的字段，配置文件也更信息，因为您不必在日期过滤器中转换后删除 "timestamp" 字段。

另一个用例是 CouchDB Changes 输入插件（参见 https://github.com/logstash-plugins/logstash-input-couchdb_changes）。 此插件自动将 CouchDB 文档字段元数据捕获到输入插件本身的 `@metadata` 字段中。当事件通过Elasticsearch索引时，Elasticsearch 输出插件允许您指定 `action`（删除、更新、插入等）和 `document_id`，如下所示：

```yaml
output {
  elasticsearch {
    action => "%{[@metadata][action]}"
    document_id => "%{[@metadata][_id]}"
    hosts => ["example.com"]
    index => "index_name"
    protocol => "http"
  }
}
```

