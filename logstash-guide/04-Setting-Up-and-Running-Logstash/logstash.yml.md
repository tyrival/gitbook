## logstash.yml

配置文件 `logstash.yml` 的选项可以控制Logstash执行。例如，您可以对管道、配置文件的位置、日志等进行配置。当Logstash运行时，`logstash.yml` 文件中的大多数设置也可用作 [命令行运行](04-Setting-Up-and-Running-Logstash/Running-Logstash-from-the-Command-Line.md) 参数。在命令行中设置的参数，都会覆盖 `logstash.yml` 文件中的相应配置项。

`logstash.yml` 文件是用YAML编写的，其中的配置项目取决于Logstash（请参阅 [目录结构](04-Setting-Up-and-Running-Logstash/Logstash-Directory-Layout.md)），文件结构可以使用分层模式或平铺模式。 例如，要使用分层模式来设置管道批处理大小和批处理延迟，请指定：
```yaml
pipeline:
  batch:
    size: 125
    delay: 50
```
同样的配置项，如果采用平铺模式，如下：
```yaml
pipeline.batch.size: 125
pipeline.batch.delay: 50
```
`logstash.yml` 也支持使用基本的环境变量作为值
```yaml
pipeline:
  batch:
    size: ${BATCH_SIZE}
    delay: ${BATCH_DELAY:50}
node:
  name: "node_${LS_NODE_NAME}"
path:
   queue: "/tmp/${QUEUE_DIR:queue}"
```
需要注意的是，`${VAR_NAME:default_value}` 表达式同样支持，在上面的示例中设置默认批处理延迟 `50` 和的 `path.queue` 默认值为 `/tmp/queue`。

可以在 `logstash.yml` 中指定模块。 模块定义格式如下：
```yaml
modules:
  - name: MODULE_NAME1
    var.PLUGIN_TYPE1.PLUGIN_NAME1.KEY1: VALUE
    var.PLUGIN_TYPE1.PLUGIN_NAME1.KEY2: VALUE
    var.PLUGIN_TYPE2.PLUGIN_NAME2.KEY1: VALUE
    var.PLUGIN_TYPE3.PLUGIN_NAME3.KEY1: VALUE
  - name: MODULE_NAME2
    var.PLUGIN_TYPE1.PLUGIN_NAME1.KEY1: VALUE
    var.PLUGIN_TYPE1.PLUGIN_NAME1.KEY2: VALUE
```
如果命令行中使用了 [命令行参数](../04-Setting-Up-and-Running-Logstash/Running-Logstash-from-the-Command-Line.md#命令行参数) `--module`，则上面定义的所有模块都会被忽略。

`logstash.yaml` 文件包括如下配置项，如果你使用了X-Pack，可以查看 [设置X-Pack](../04-Setting-Up-and-Running-Logstash/Setting-Up-X-Pack.md)

| 配置项                        | 说明                                                         | 默认值                                                       |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `node.name`                   | 节点名称。                                                   | 服务器hostname                                               |
| `path.data`                   | Logstash及其插件用于持久化数据的位置。                       | `LOGSTASH_HOME/data`                                         |
| `pipeline.id`                 | 管道ID。                                                     | `main`                                                       |
| `pipeline.java_execution`     | 管道的Java执行引擎。                                         | `false`                                                      |
| `pipeline.workers`            | 管道的工作者数量，即执行过滤器和输出阶段的并发数，如果你发现事件正在备份，或者CPU运行不饱和，可以考虑提高此参数的值，以实现对服务器性能的利用率。 | 服务器的CPU核心数                                            |
| `pipeline.batch.size`         | 管道的最大批容量，当单个管道工作者接收到的事件达到此数量时，则立刻将这些事件打包为一个批，进行过滤和输出处理。如果给此参数赋予较大的数值，可以提高效率，但会导致更高的内存开销，从而需要在 `jvm.options` 配置文件中配置更高的堆内存空间，更多信息请查看 [jvm.options配置文件](../04-Setting-Up-and-Running-Logstash/Logstash-Configuration-Files.md)。 | `125`                                                        |
| `pipeline.batch.delay`        | 为管道声明一个时间延迟的值，当创建管道在接收到一个事件后，接下来未接受到事件的空闲时间达到这个值，则将管道内缓存的事件打包为一个批，传递给管道工作者进行处理，即使这个批还未达到最大批容量。 | `50`                                                         |
| `pipeline.unsafe_shutdown`    | 默认情况下，当Logstash接收到关闭命令时，如果还有未处理的事件，会拒绝执行关闭操作，直到所有事件都进行了输出。当此选项设置为`true`时，当Logstash接收到关闭命令后，会无视内存中存在的未处理事件而强行关闭，从而会导致数据丢失。 | `false`                                                      |
| `path.config`                 | Logstash的主管道配置文件位置，当你指定一个目录或含通配符的路径时，Logstash会按照字母顺序读取该目录下的配置文件。 | 视平台而定，查看 [目录结构](../04-Setting-Up-and-Running-Logstash/Logstash-Directory-Layout.md) |
| `config.string`               | 主管道配置字符串，语法与配置文件语法相同。                   | None                                                         |
| `config.test_and_exit`        | 当设置为`true`时，会校验配置文件内容的合法性并退出（需要注意的是，此设置不会检查grok正则匹配的正确性）。Logstash可以从目录中读取多个配置文件，如果将此设置与`log.level：debug`结合使用，Logstash将记录组合的配置文件，使用它来自的源文件注释每个配置块。 | `false`                                                      |
| `config.reload.automatic`     | 当设置为`true`时，会定期检查配置项，无论配置项是否变更，都会重新加载。可以手工执行SIGHUP命令触发此操作。 | `false`                                                      |
| `config.reload.interval`      | Logstash定期检查配置文件变更的间隔时间。                     | `3s`                                                         |
| `config.debug`                | 当设置为 `true`时，将完整的配置编译信息打印到日志消息中。你必须同时设置 `log.level: debug`。警告：日志消息将包含以纯文本传递给插件的配置信息，这可能导致明文密码出现在您的日志中！ | `false`                                                      |
| `config.support_escapes`      | 当设置为`true`时，带引号的字符串将处理以下转义序列：`\n`转为文字换行符（ASCII 10）。 `\r`成为文字回车（ASCII 13）。`\t`转为文字标签（ASCII 9）。`\\`转为字面反斜杠`\`。 `\"`转为文字双引号。`\'`转为文字引号。 | `false`                                                      |
| `modules`                     | 配置时，`modules`必须位于此表上面所述的嵌套YAML结构中。      | None                                                         |
| `queue.type`                  | 用于事件缓冲的内部队列模型。`memory`表示基于内存存储的队列，`persistent`表示基于磁盘存储的ACKed队列（[持久化队列](../10-Data-Resiliency/Persistent-Queues.md)）。 | `memory`                                                     |
| `path.queue`                  | 当启用持久化队列时(`queue.type: persisted`)，此配置项声明队列的持久哈 u存储路径。 | `path.data/queue`                                            |
| `queue.page_capacity`         | 当启用持久化队列时(`queue.type: persisted`)，声明页数据文件的容量，队列数据（只能向尾部附加新数据）会按照页文件的最大容量，被分割存储到一个或多个页文件中。 | 64mb                                                         |
| `queue.max_events`            | 启用持久队列时(`queue.type: persisted`)，队列中未读事件的最大数量。 | 0 (不限制)                                                   |
| `queue.max_bytes`             | 队列的总容量，以字节数表示。 需要确保磁盘的容量大于此处指定的值。 如果同时指定了`queue.max_events`和`queue.max_bytes`，则Logstash将使用先达到的标准。 | 1024mb (1g)                                                  |
| `queue.checkpoint.acks`       | 启用持久队列时（`queue.type：persisted`），强制检查点之前的最大ACK事件数。 `queue.checkpoint.acks：0`表示无限制。 | 1024                                                         |
| `queue.checkpoint.writes`     | 启用持久队列时（`queue.type：persisted`），强制检查点之前写入事件的最大数量。 queue.checkpoint.writes：0`表示无限制。 | 1024                                                         |
| `queue.checkpoint.retry`      | 启用后，每当检查点写入失败，Logstash将针对每个检查点重试一次。 后续错误将不会重试。 这是仅在具有非标准行为（如SANs）的文件系统上检查点写入失败的解决方法，除特殊情况外，不建议启用。 | `false`                                                      |
| `queue.drain`                 | 启用后，Logstash将等待直到持久队列耗尽，才能关闭。           | `false`                                                      |
| `dead_letter_queue.enable`    | 用于指示Logstash启用插件支持的DLQ功能的标志。                | `false`                                                      |
| `dead_letter_queue.max_bytes` | 每个死信队列的最大值。 如果条目进入后，导致死信队列容量超过此最大值，则会删除该条目。 | `1024mb`                                                     |
| `path.dead_letter_queue`      | 死信队列存储数据文件的目录路径。                             | `path.data/dead_letter_queue`                                |
| `http.host`                   | 服务器IP                                                     | `"127.0.0.1"`                                                |
| `http.port`                   | The bind port for the metrics REST endpoint.                 | `9600`                                                       |
| `log.level`                   | 日志级别，可选项包括:`fatal`、`error`、`warn`、`info`、`debug`、`trace` | `info`                                                       |
| `log.format`                  | 日志格式。`json` 表示JSON格式， `plain` 表示采用 `Object#.inspect`. | `plain`                                                      |
| `path.logs`                   | The directory where Logstash will write its log to.          | `LOGSTASH_HOME/logs`                                         |
| `path.plugins`                | 插件的存储位置，可以指定多个路径。建议按照如下规则储存插件：`PATH/logstash/TYPE/NAME.rb`，其中 `TYPE`包括 `inputs`、`filters`、`outputs`、或`codecs`，`NAME` 表示插件名称 | 视平台而定，查看 [目录结构](../04-Setting-Up-and-Running-Logstash/Logstash-Directory-Layout.md) |

