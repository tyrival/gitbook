## 日志配置

Elasticsearch使用 [Log4j 2](https://logging.apache.org/log4j/2.x/) 进行日志记录。 可以使用log4j2.properties文件配置Log4j 2。Elasticsearch暴露三个属性`${sys:es.logs.base_path}`，`${sys:es.logs.cluster_name}`和 `${sys:es.logs.node_name}`（如果通过`node.name`显式设置节点名称） ），可以在配置文件中引用以确定日志文件的位置。 属性`${sys:es.logs.base_path}`将解析为日志目录，`${sys:es.logs.cluster_name}`将解析为集群名称（在默认配置中用作日志文件名的前缀），以及 `${sys:es.logs.node_name}`将解析为节点名称（如果明确设置了节点名称）。

例如，如果您的日志目录（`path.logs`）是`/var/log/elasticsearch`并且您的集群名`为production`，那么`${sys:es.logs.base_path}`将解析为`/var/log/elasticsearch`和`$ {${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log`将解析为`/var/log/elasticsearch/production.log`。

```properties
appender.rolling.type = RollingFile  ①
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log ②
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c{1.}] [%node_name]%marker %.-10000m%n
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}-%i.log.gz ③
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy ④
appender.rolling.policies.time.interval = 1 ⑤
appender.rolling.policies.time.modulate = true ⑥
appender.rolling.policies.size.type = SizeBasedTriggeringPolicy ⑦
appender.rolling.policies.size.size = 256MB ⑧
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.fileIndex = nomax
appender.rolling.strategy.action.type = Delete ⑨
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path}
appender.rolling.strategy.action.condition.type = IfFileName ⑩
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-* ⑪
appender.rolling.strategy.action.condition.nested_condition.type = IfAccumulatedFileSize ⑫
appender.rolling.strategy.action.condition.nested_condition.exceeds = 2GB ⑬
```

① 配置附加类型为`RollingFile`

② 登录`/var/log/elasticsearch/production.log`

③ 将日志滚动到 `/var/log/elasticsearch/production-yyyy-MM-dd-i.log`；日志将在每个卷上压缩，`i`将递增

④ 使用基于时间的滚动策略

⑤ 每天滚动日志

⑥ 在每天的时间边界上对齐卷（而不是每隔二十四小时滚动）

⑦ 使用基于大小的滚动策略

⑧ 256 MB后滚动日志

⑨ 滚动日志时使用删除操作

⑩ 仅删除与文件模式匹配的日志

⑪ 该模式仅删除主日志

⑫ 仅在我们累积了太多压缩日志时才删除

⑬ 压缩日志的大小条件为2GB

> **注意**
>
> Log4j的配置解析会被无用的空格混淆；如果您在此页面上复制并粘贴任何Log4j设置，或者输入任何Log4j配置，请务必删除每一行配置两端的空格。

注意，您可以在`appender.rolling.filePattern`中使用`.zip`替换`.gz`，以使用zip格式压缩滚动日志。如果删除`.gz`扩展名，则日志将不会在滚动时进行压缩。

如果要在指定的时间段内保留日志文件，可以使用具有删除操作的翻转策略。

```properties
appender.rolling.strategy.type = DefaultRolloverStrategy ①
appender.rolling.strategy.action.type = Delete ②
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path} ③
appender.rolling.strategy.action.condition.type = IfFileName ④
appender.rolling.strategy.action.condition.glob = ${sys:es.logs.cluster_name}-* ⑤
appender.rolling.strategy.action.condition.nested_condition.type = IfLastModified ⑥
appender.rolling.strategy.action.condition.nested_condition.age = 7D ⑦
```


① 配置`DefaultRolloverStrategy`

② 配置`delete`操作以处理翻转

③ Elasticsearch日志的基本路径

④ 处理滚动时应用的条件

⑤ 删除与glob `${sys:es.logs.cluster_name}-*`匹配的基本路径中的文件；这是日志文件滚动到的glob；这只需要删除已滚动的Elasticsearch日志，但也不需要删除弃用和慢速日志

⑥ 应用于与glob匹配的文件的嵌套条件

⑦ 保留日志七天

可以加载多个配置文件（在这种情况下，它们将被合并），只要它们被命名为`log4j2.properties`并且将Elasticsearch配置目录作为祖先目录。这对于暴露其他记录器的插件很有用。日志记录部分包含java包及其相应的日志级别。 appender部分包含日志的目标。有关如何自定义日志记录和所有支持的appender的详细信息，请参阅 [Log4j文档](http://logging.apache.org/log4j/2.x/manual/configuration.html)。

### 配置日志级别

有四种配置日志记录级别的方法，每种方法都有适合使用的情况。

1. 通过命令行：`-E <name of logging hierarchy>=<level>`（例如，`-E logger.org.elasticsearch.transport=trace`）。当您临时调试单个节点上的问题时（例如，启动问题或开发期间），这是最合适的。
2. 通过`elasticsearch.yml`: `<name of logging hierarchy>: <level>`（例如，`logger.org.elasticsearch.transport: trace`）。当您临时调试问题但未通过命令行启动Elasticsearch（例如，通过服务），或者您希望更长期地调整日志记录级别时，这是最合适的。
3. 通过 [集群设置](../../14-Modules/Cluster/Miscellaneous-cluster-settings.md#Logger)：

```sh
PUT /_cluster/settings
{
  "transient": {
    "<name of logging hierarchy>": "<level>"
  }
}
```

例如：

```sh
PUT /_cluster/settings
{
  "transient": {
    "logger.org.elasticsearch.transport": "trace"
  }
}
```

当您需要动态调整正在运行的集群上的日志记录级别时，这是最合适的。

4. 通过`log4j2.properties`：

```properties
logger.<unique_identifier>.name = <name of logging hierarchy>
logger.<unique_identifier>.level = <level>
```

例如：

```properties
logger.transport.name = org.elasticsearch.transport
logger.transport.level = trace
```

当您需要对日志记录器进行细粒度控制时（例如，您希望将记录器发送到另一个文件，或以不同方式管理记录器；这是一种罕见的用例），这是最合适的。

### 弃用的日志

除常规日志记录外，Elasticsearch还允许您启用已弃用操作的日志记录。例如，如果您将来需要迁移某些功能，这可以让您尽早决定。默认情况下，将在WARN级别启用弃用日志记录，该级别是将发出所有弃用日志消息的级别。

```properties
logger.deprecation.level = warn
```

这将在日志目录中创建每日滚动弃用日志文件。定期检查此文件，尤其是当您打算升级到新的主要版本时。

默认日志记录配置已将弃用日志的卷策略设置为在1 GB后滚动和压缩，并最多保留五个日志文件（四个滚动日志和活动日志）。

您可以通过将弃用日志级别设置为`error`来在`config/log4j2.properties`文件中禁用它。
