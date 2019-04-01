## 日志

记录Logstash操作的内部日志，位于 ` LOGSTASH_HOME/logs`中（DEB或RPM系统为 `/var/log/logstash`）。默认日志记录级别为 `INFO`。 Logstash的日志框架基于 [Log4j2框架](http://logging.apache.org/log4j/2.x/)，其大部分功能直接暴露给用户。

您可对特定子系统、模块或插件的日志记录进行配置。

当您需要调试问题，特别是插件问题时，可以考虑将日志记录级别增加到 `DEBUG`，以获取更详细的消息。例如，如果您正在调试Elasticsearch Output的问题，则可以仅为该组件增加日志级别。这种方法可以减少过度记录的噪音，从而更专注于问题。

您可以使用 `log4j2.properties` 文件或Logstash API配置日志记录。

- **`log4j2.properties` 文件**。修改 `log4j2.properties` 文件后，必须重启Logstash使更改生效。更改将在后续重新启动时保留。
- **日志API**。通过Logging API进行的更改立即生效，无需重新启动。重新启动Logstash后，更改将不会保留。

### Log4j2配置

Logstash默认附带一个开箱即用的 `log4j2.properties` 文件。您可以修改此文件以更改轮换策略、类型和其他 [log4j2配置](https://logging.apache.org/log4j/2.x/manual/configuration.html#Loggers)。

对此文件所做的任何更改，需重新启动Logstash才能生效，修改内容在重启后不会丢失。

以下是 `outputs.elasticsearch` 的示例：

```properties
logger.elasticsearchoutput.name = logstash.outputs.elasticsearch
logger.elasticsearchoutput.level = debug
```

### 日志API

对于临时修改日志记录，修改 `log4j2.properties` 并重启Logstash会导致不必要的停机时间，此时，可以通过日志记录API动态更新日志记录级别。这些设置立即生效，无需重启。

> **注意：**
> 默认情况下，日志API使用 `tcp:9600` 。如果此端口已被另一个Logstash实例使用，则在启动Logstash时，需要使用 `—http.port` 参数绑定到其他端口。有关更多信息，请参阅 [命令行运行](../04-Setting-Up-and-Running-Logstash/Running-Logstash-from-the-Command-Line.md)。

#### 日志配置列表
通过向 `_node / logging` 发送 `GET`请求，可以查看可用的日志记录子系统列表

```shell
curl -XGET 'localhost:9600/_node/logging?pretty'
```

响应示例：

```shell
{
...
  "loggers" : {
    "logstash.agent" : "INFO",
    "logstash.api.service" : "INFO",
    "logstash.basepipeline" : "INFO",
    "logstash.codecs.plain" : "INFO",
    "logstash.codecs.rubydebug" : "INFO",
    "logstash.filters.grok" : "INFO",
    "logstash.inputs.beats" : "INFO",
    "logstash.instrument.periodicpoller.jvm" : "INFO",
    "logstash.instrument.periodicpoller.os" : "INFO",
    "logstash.instrument.periodicpoller.persistentqueue" : "INFO",
    "logstash.outputs.stdout" : "INFO",
    "logstash.pipeline" : "INFO",
    "logstash.plugins.registry" : "INFO",
    "logstash.runner" : "INFO",
    "logstash.shutdownwatcher" : "INFO",
    "org.logstash.Event" : "INFO",
    "slowlog.logstash.codecs.plain" : "TRACE",
    "slowlog.logstash.codecs.rubydebug" : "TRACE",
    "slowlog.logstash.filters.grok" : "TRACE",
    "slowlog.logstash.inputs.beats" : "TRACE",
    "slowlog.logstash.outputs.stdout" : "TRACE"
  }
}
```

#### 修改日志级别

在子系统、模块或插件的名称前增加 `logger.`。

以下是 `outputs.elasticsearch` 的示例：

```shell
curl -XPUT 'localhost:9600/_node/logging?pretty' -H 'Content-Type: application/json' -d'
{
    "logger.logstash.outputs.elasticsearch" : "DEBUG"
}
'
```

当此设置生效时，Logstash配置中指定的所有Elasticsearch将输出DEBUG级别的日志。 请注意，此新设置是暂时的，无法在重新启动后继续运行。

> **注意：**
> 如果要在重启后保留日志记录更改，请将其添加到 `log4j2.properties`。

#### 重置动态日志级别

当日志记录级别可能已通过日志API动态更改时，如需重置，可发送 `PUT` 请求到 `_node/logging/reset`。 所有日志记录级别都将恢复为 `log4j2.properties` 中指定的值。

```shell
curl -XPUT 'localhost:9600/_node/logging/reset?pretty'
```

### 日志文件存储位置

可以通过 `--path.logs` 指定日志文件存储路径。

### 慢速日志slowlog

当特定事件通过管道时花费时间异常，Logstash的slowlog可以对这种情况进行记录。如同普通的应用程序日志，slowlog可以在 `--path.logs` 目录中找到。使用以下选项在 `logstash.yml` 中配置slowlog：

```yaml
slowlog.threshold.warn（默认值: -1）
slowlog.threshold.info（默认值: -1）
slowlog.threshold.debug（默认值: -1）
slowlog.threshold.trace（默认值: -1）
```

Slowlog默认禁用。默认阈值为 `-1nanos` ，表示无限阈值，不会使用slowlog。

#### 启用slowlog

`slowlog.threshold` 字段使用时间值格式，可以支持较大的触发间隔。您可以使用以下时间单位指定范围：nanos（纳秒）、微秒（微秒）、ms（毫秒）、s（秒）、m（分钟）、h（小时）、d（天）。

当您提高日志级别时，slowlog会变得更加灵敏，并记录更多事件。

例：

```yaml
slowlog.threshold.warn: 2s
slowlog.threshold.info: 1s
slowlog.threshold.debug: 500ms
slowlog.threshold.trace: 100ms
```

以上示例中：

- 如果日志级别设置为 `warn`，则日志显示处理时间超过2秒的事件。
- 如果日志级别设置为 `info`，则日志显示要处理的时间超过1秒的事件。
- 如果日志级别设置为 `trace`，则日志显示要处理的时间超过100毫秒的事件。
- 如果日志级别设置为 `debug`，则日志显示要处理的时间超过500毫秒的事件。

日志包含了导致速度缓慢的完整事件和过滤器配置。