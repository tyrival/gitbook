## 节点信息API

可以通过如下API检索节点信息：

```sh
curl -XGET 'localhost:9600/_node/<types>'
```

其中 `<types>` 是可选的，用于指定您想获取信息的节点。

您可以通过组合以下类型，来限制返回的信息，多种类型以逗号分隔：

| 类型                  | 说明                                 |
| --------------------- | ------------------------------------ |
| [pipeline](#管道信息) | 获取指定管道的信息以及每个管道的设置 |
| [os](#操作系统信息)   | 获取节点级别的操作系统信息           |
| [jvm](#JVM信息)       | 获取节点级别的JVM信息，包含线程信息  |

更多可用的Logstash监控API的参数，请查看 [通用选项](../15-Monitoring-APIs/README.md#通用选项)。

### 管道信息

以下请求返回显示管道信息的JSON文档，例如工作者数量，批处理大小和批处理延迟：

```sh
curl -XGET 'localhost:9600/_node/pipelines?pretty'
```

如果要查看有关管道的其他信息，例如每个已配置输入、过滤器或输出阶段的统计信息，请参阅 [节点状态API](../15-Monitoring-APIs/Node-Stats-API.md) 下的 [管道状态](../15-Monitoring-APIs/Node-Stats-API.md#管道状态) 部分。

响应示例：

```json
{
  "pipelines" : {
    "test" : {
      "workers" : 1,
      "batch_size" : 1,
      "batch_delay" : 5,
      "config_reload_automatic" : false,
      "config_reload_interval" : 3
    },
    "test2" : {
      "workers" : 8,
      "batch_size" : 125,
      "batch_delay" : 5,
      "config_reload_automatic" : false,
      "config_reload_interval" : 3
    }
  }
}
```

您可以通过包含管道ID来查看特定管道的信息。在以下示例中，管道的ID是 `test`：

```sh
curl -XGET 'localhost:9600/_node/pipelines/test?pretty'
```

响应示例：

```json
{
  "pipelines" : {
    "test" : {
      "workers" : 1,
      "batch_size" : 1,
      "batch_delay" : 5,
      "config_reload_automatic" : false,
      "config_reload_interval" : 3
    }
  }
}
```

如果指定了无效的管道ID，则请求将返回404 Not Found错误。

### 操作系统信息

以下请求返回一个JSON文档，该文档显示操作系统名称，体系结构，版本和可用处理器：

```sh
curl -XGET 'localhost:9600/_node/os?pretty'
```

响应示例：

```json
{
  "os": {
    "name": "Mac OS X",
    "arch": "x86_64",
    "version": "10.12.4",
    "available_processors": 8
  }
}
```

### JVM信息

以下请求返回一个JSON文档，该文档显示节点级JVM统计信息，例如JVM进程ID，版本，VM信息，内存使用情况以及有关垃圾收集器的信息：

```sh
curl -XGET 'localhost:9600/_node/jvm?pretty'
```

响应示例：

```json
{
  "jvm": {
    "pid": 59616,
    "version": "1.8.0_65",
    "vm_name": "Java HotSpot(TM) 64-Bit Server VM",
    "vm_version": "1.8.0_65",
    "vm_vendor": "Oracle Corporation",
    "start_time_in_millis": 1484251185878,
    "mem": {
      "heap_init_in_bytes": 268435456,
      "heap_max_in_bytes": 1037959168,
      "non_heap_init_in_bytes": 2555904,
      "non_heap_max_in_bytes": 0
    },
    "gc_collectors": [
      "ParNew",
      "ConcurrentMarkSweep"
    ]
  }
}
```
