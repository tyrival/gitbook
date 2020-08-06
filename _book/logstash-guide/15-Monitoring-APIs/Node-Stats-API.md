## 节点状态API

节点状态API可以获取Logstash的运行状态。

```sh
curl -XGET 'localhost:9600/_node/stats/<types>'
```

其中 `<types>` 是可选项，并且可以通过参数指定想要返回的类型。

默认情况下会返回所有状态，您可以通过参数组合，对返回值进行限制，多个参数用逗号分隔：

| 参数                   | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| [jvm](#JVM状态)        | 获取JVM状态，包括线程、内存、GC，以及运行时间等状态          |
| [process](#处理状态)   | 获取处理状态，包括文件描述、内存消耗、CPU使用情况等状态      |
| [events](#事件状态)    | 获取Logstash实例的事件相关统计信息（无论创建或销毁了多少管道） |
| [pipelines](#管道状态) | 获取Logstash每个管道的运行状态                               |
| [reloads](#重载状态)   | 获取重载配置成功或失败的运行状态                             |
| [os](#操作系统状态)    | 获取Logstash运行在容器中的运行状态                           |

Logstash通用的监控API请查看 [通用选项](../16-Working-with-plugins/README.md)。

### JVM状态

以下请求返回包含JVM统计信息的JSON文档：

```sh
curl -XGET 'localhost:9600/_node/stats/jvm?pretty'
```

响应示例：

```json
{
  "jvm" : {
    "threads" : {
      "count" : 49,
      "peak_count" : 50
    },
    "mem" : {
      "heap_used_percent" : 14,
      "heap_committed_in_bytes" : 309866496,
      "heap_max_in_bytes" : 1037959168,
      "heap_used_in_bytes" : 151686096,
      "non_heap_used_in_bytes" : 122486176,
      "non_heap_committed_in_bytes" : 133222400,
      "pools" : {
        "survivor" : {
          "peak_used_in_bytes" : 8912896,
          "used_in_bytes" : 288776,
          "peak_max_in_bytes" : 35782656,
          "max_in_bytes" : 35782656,
          "committed_in_bytes" : 8912896
        },
        "old" : {
          "peak_used_in_bytes" : 148656848,
          "used_in_bytes" : 148656848,
          "peak_max_in_bytes" : 715849728,
          "max_in_bytes" : 715849728,
          "committed_in_bytes" : 229322752
        },
        "young" : {
          "peak_used_in_bytes" : 71630848,
          "used_in_bytes" : 2740472,
          "peak_max_in_bytes" : 286326784,
          "max_in_bytes" : 286326784,
          "committed_in_bytes" : 71630848
        }
      }
    },
    "gc" : {
      "collectors" : {
        "old" : {
          "collection_time_in_millis" : 607,
          "collection_count" : 12
        },
        "young" : {
          "collection_time_in_millis" : 4904,
          "collection_count" : 1033
        }
      }
    },
    "uptime_in_millis" : 1809643
  }
}
```

### 处理状态

以下请求返回包含处理统计信息的JSON文档：

```sh
curl -XGET 'localhost:9600/_node/stats/process?pretty'
```

响应示例：

```json
{
  "process" : {
    "open_file_descriptors" : 184,
    "peak_open_file_descriptors" : 185,
    "max_file_descriptors" : 10240,
    "mem" : {
      "total_virtual_in_bytes" : 5486125056
    },
    "cpu" : {
      "total_in_millis" : 657136,
      "percent" : 2,
      "load_average" : {
        "1m" : 2.38134765625
      }
    }
  }
}
```

### 事件状态

以下请求返回包含Logstash实例事件相关的统计信息的JSON文档：

```sh
curl -XGET 'localhost:9600/_node/stats/events?pretty'
```

响应示例：

```json
{
  "events" : {
    "in" : 293658,
    "filtered" : 293658,
    "out" : 293658,
    "duration_in_millis" : 2324391,
    "queue_push_duration_in_millis" : 343816
  }
}
```

### 管道状态

以下请求返回包含管道统计信息的JSON文档，包含：

- 每个管道输入、过滤或输出的事件数
- 每个已配置的过滤器或输出阶段的统计信息
- 有关配置重新加载成功和失败的信息（启用 [重载配置文件](../06-Configuring-Logstash/Reloading-the-Config-File.md) 时）
- 有关持久队列的信息（启用 [持久化队列](../10-Data-Resiliency/Persistent-Queues.md) 时）

```sh
curl -XGET 'localhost:9600/_node/stats/pipelines?pretty'
```

响应示例：

```json
{
  "pipelines" : {
    "test" : {
      "events" : {
        "duration_in_millis" : 365495,
        "in" : 216485,
        "filtered" : 216485,
        "out" : 216485,
        "queue_push_duration_in_millis" : 342466
      },
      "plugins" : {
        "inputs" : [ {
          "id" : "35131f351e2dc5ed13ee04265a8a5a1f95292165-1",
          "events" : {
            "out" : 216485,
            "queue_push_duration_in_millis" : 342466
          },
          "name" : "beats"
        } ],
        "filters" : [ {
          "id" : "35131f351e2dc5ed13ee04265a8a5a1f95292165-2",
          "events" : {
            "duration_in_millis" : 55969,
            "in" : 216485,
            "out" : 216485
          },
          "failures" : 216485,
          "patterns_per_field" : {
            "message" : 1
          },
          "name" : "grok"
        }, {
          "id" : "35131f351e2dc5ed13ee04265a8a5a1f95292165-3",
          "events" : {
            "duration_in_millis" : 3326,
            "in" : 216485,
            "out" : 216485
          },
          "name" : "geoip"
        } ],
        "outputs" : [ {
          "id" : "35131f351e2dc5ed13ee04265a8a5a1f95292165-4",
          "events" : {
            "duration_in_millis" : 278557,
            "in" : 216485,
            "out" : 216485
          },
          "name" : "elasticsearch"
        } ]
      },
      "reloads" : {
        "last_error" : null,
        "successes" : 0,
        "last_success_timestamp" : null,
        "last_failure_timestamp" : null,
        "failures" : 0
      },
      "queue" : {
        "type" : "memory"
      }
    },
    "test2" : {
      "events" : {
        "duration_in_millis" : 2222229,
        "in" : 87247,
        "filtered" : 87247,
        "out" : 87247,
        "queue_push_duration_in_millis" : 1532
      },
      "plugins" : {
        "inputs" : [ {
          "id" : "d7ea8941c0fc48ac58f89c84a9da482107472b82-1",
          "events" : {
            "out" : 87247,
            "queue_push_duration_in_millis" : 1532
          },
          "name" : "twitter"
        } ],
        "filters" : [ ],
        "outputs" : [ {
          "id" : "d7ea8941c0fc48ac58f89c84a9da482107472b82-2",
          "events" : {
            "duration_in_millis" : 139545,
            "in" : 87247,
            "out" : 87247
          },
          "name" : "elasticsearch"
        } ]
      },
      "reloads" : {
        "last_error" : null,
        "successes" : 0,
        "last_success_timestamp" : null,
        "last_failure_timestamp" : null,
        "failures" : 0
      },
      "queue" : {
        "type" : "memory"
      }
    }
  }
}
```

您可以通过指定管道ID来查看特定管道的状态，下面的示例中，管道ID为 `test`：

```sh
curl -XGET 'localhost:9600/_node/stats/pipelines/test?pretty'
```

响应示例：

```json
{
    "test" : {
      "events" : {
        "duration_in_millis" : 365495,
        "in" : 216485,
        "filtered" : 216485,
        "out" : 216485,
        "queue_push_duration_in_millis" : 342466
      },
      "plugins" : {
        "inputs" : [ {
          "id" : "35131f351e2dc5ed13ee04265a8a5a1f95292165-1",
          "events" : {
            "out" : 216485,
            "queue_push_duration_in_millis" : 342466
          },
          "name" : "beats"
        } ],
        "filters" : [ {
          "id" : "35131f351e2dc5ed13ee04265a8a5a1f95292165-2",
          "events" : {
            "duration_in_millis" : 55969,
            "in" : 216485,
            "out" : 216485
          },
          "failures" : 216485,
          "patterns_per_field" : {
            "message" : 1
          },
          "name" : "grok"
        }, {
          "id" : "35131f351e2dc5ed13ee04265a8a5a1f95292165-3",
          "events" : {
            "duration_in_millis" : 3326,
            "in" : 216485,
            "out" : 216485
          },
          "name" : "geoip"
        } ],
        "outputs" : [ {
          "id" : "35131f351e2dc5ed13ee04265a8a5a1f95292165-4",
          "events" : {
            "duration_in_millis" : 278557,
            "in" : 216485,
            "out" : 216485
          },
          "name" : "elasticsearch"
        } ]
      },
      "reloads" : {
        "last_error" : null,
        "successes" : 0,
        "last_success_timestamp" : null,
        "last_failure_timestamp" : null,
        "failures" : 0
      },
      "queue" : {
        "type" : "memory"
      }
    }
  }
}
```

### 重载状态

以下请求返回包含重载成败状态信息的JSON文档，包含：

```sh
curl -XGET 'localhost:9600/_node/stats/reloads?pretty'
```

响应示例：

```json
{
  "reloads": {
    "successes": 0,
    "failures": 0
  }
}
```

### 操作系统状态

当Logstash在容器中运行时，以下请求将返回包含cgroup信息的JSON文档，以便更准确地查看CPU负载，包括是否对容器进行了限制。

```sh
curl -XGET 'localhost:9600/_node/stats/os?pretty'
```

响应示例：

```json
{
  "os" : {
    "cgroup" : {
      "cpuacct" : {
        "control_group" : "/elastic1",
        "usage_nanos" : 378477588075
                },
      "cpu" : {
        "control_group" : "/elastic1",
        "cfs_period_micros" : 1000000,
        "cfs_quota_micros" : 800000,
        "stat" : {
          "number_of_elapsed_periods" : 4157,
          "number_of_times_throttled" : 460,
          "time_throttled_nanos" : 581617440755
        }
      }
    }
  }
```
