## Docker配置

Logstash区分两种类型的配置：[设置和管道配置](../04-Setting-Up-and-Running-Logstash/Logstash-Configuration-Files.md)。

### 管道配置
管道配置文件必须位于Logstash可以访问的位置。默认情况下，Logstash容器将在 `/usr/share/logstash/pipeline/` 中查找管道配置文件。

在此示例中，将配置文件从宿主机映射进容器中，从而为Logstash提供配置文件：

```shell
docker run --rm -it -v ~/pipeline/:/usr/share/logstash/pipeline/ docker.elastic.co/logstash/logstash:6.7.0
```

然后，Logstash将宿主机目录 `~/pipeline/` 中的所有文件解析为管道配置。

如果文件夹中无任何配置，Logstash将以最小配置运行，它将监听来自 [beats输入插件](../17-Input-plugins/beats.md) 的消息，并将接受到的信息输出给 `stdout`。启动日志将输入类似如下内容：

```shell
Sending Logstash logs to /usr/share/logstash/logs which is now configured via log4j2.properties.
[2016-10-26T05:11:34,992][INFO ][logstash.inputs.beats    ] Beats inputs: Starting input listener {:address=>"0.0.0.0:5044"}
[2016-10-26T05:11:35,068][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>500}
[2016-10-26T05:11:35,078][INFO ][org.logstash.beats.Server] Starting server on port: 5044
[2016-10-26T05:11:35,078][INFO ][logstash.pipeline        ] Pipeline main started
[2016-10-26T05:11:35,105][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
```

这是镜像的默认配置，定义在 `/usr/share/logstash/pipeline/logstash.conf` 中。你可以通过以上输出，来确保Logstash选取了正确的管道配置，以及 `logstash.conf` 或整个 `pipeline` 目录已经被正确替换。

### Settings

镜像提供了几种修改设置的方法。传统方法是提供自定义 `logstash.yml` 文件，但也可以使用环境变量来定义设置。

#### 从宿主机挂载

设置文件也可以通过宿主机向容器内挂载。 Logstash的设置文件储存路径为 `/usr/share/logstash config/`。

可以映射整个目录：

```shell
docker run --rm -it -v ~/settings/:/usr/share/logstash/config/ docker.elastic.co/logstash/logstash:6.7.0
```

也可以映射单个文件：

```shell
docker run --rm -it -v ~/settings/logstash.yml:/usr/share/logstash/config/logstash.yml docker.elastic.co/logstash/logstash:6.7.0
```

> **注意：**
> 挂载的配置文件在宿主机和容器中，权限和所有权完全相同。需要对其设置权限，使文件可读，并且最好使容器的 `logstash` 用户（UID 1000）不可写。

#### 自定义镜像

从宿主机挂载并不是唯一的选择。如果你希望采用不侵入容器的方法，可以使用类似如下的 `Dockerfile`，以自定义的配置生成镜像：

```dockerfile
FROM docker.elastic.co/logstash/logstash:6.7.0
RUN rm -f /usr/share/logstash/pipeline/logstash.conf
ADD pipeline/ /usr/share/logstash/pipeline/
ADD config/ /usr/share/logstash/config/
```

请确保在自定义映像中替换或删除原有 `logstash.conf` 文件，以便不使用基础镜像的样例配置。

#### 配置环境变量

在Docker中，可以通过环境变量配置Logstash。容器启动时，辅助进程会检查系统中是否有可映射到Logstash设置的环境变量。当容器启动时，环境变量中对应的值将注入到 `logstash.yml` 中。

为了与容器编排系统兼容，环境变量以全部大写写入，使用下划线为单词分隔符

以下是一些示例翻译：

#### 表1. Docker环境变量示例

| 环境变量                   | Logstash设置项             |
| -------------------------- | -------------------------- |
| `PIPELINE_WORKERS`         | `pipeline.workers`         |
| `LOG_LEVEL`                | `log.level`                |
| `XPACK_MONITORING_ENABLED` | `xpack.monitoring.enabled` |


此方法基本可配置 [logstash.yml](../04-Setting-Up-and-Running-Logstash/logstash.yml.md) 中列出的任何设置项。

> **注意：**
> 通过环境变量定义设置项会导致 `logstash.yml` 修改。如果 `logstash.yml` 是从宿主机挂载的，则此行为可能不合适。因此，不建议将宿主机挂载技术与环境变量技术相结合，建议选择其中一种方法。

### Docker默认值

使用Docker镜像时，以下设置项的默认值与其他使用场景不同：

| 设置项                                 | 默认值                      |
| -------------------------------------- | --------------------------- |
| `http.host`                            | `0.0.0.0`                   |
| `xpack.monitoring.elasticsearch.hosts` | `http://elasticsearch:9200` |

> **注意：**
> `xpack.monitoring.elasticsearch.hosts` 设置项未在 `-oss` 映像中定义。

这些设置项在 `logstash.yml` 中设为默认值。可以使用自定义 `logstash.yml` 或环境变量进行覆盖。

> **重要：**
> 如果将 `logstash.yml` 替换为自定义版本，请确保将上述默认值复制到自定义文件中（如果要保留它们）。 如果没有，它们将被新文件忽略，从而无法使用上述默认值。

### 日志配置

在Docker中，Logstash日志默认转为标准输出。 要更改此行为，请使用上述任何技术替换 `/usr/share/logstash/config/log4j2.properties` 中的文件。
