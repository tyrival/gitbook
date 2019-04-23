## 节点信息
集群节点信息API允许检索一个或多个（或所有）集群节点信息。

```sh
GET /_nodes
GET /_nodes/nodeId1,nodeId2
```

第一个命令检索集群中所有节点的信息。第二个命令选择性地检索仅nodeId1和nodeId2的节点信息。[这里](../10-Cluster-APIs/README.md)解释了所有节点选项。

默认情况下，它只返回节点的所有属性和核心设置：

`build_hash`
    此版本中最后一个git提交的短哈希。

`host`
    节点的主机名。

`ip`
    节点的IP地址。

`name`
    节点的名称。

`total_indexing_buffer`
    在必须将最近编制索引的文档写入磁盘之前，允许使用总堆。此大小是此节点上所有分片的共享池，由[索引缓冲区设置](../14-Modules/Indices/Indexing-Buffer.md)控制。

`total_indexing_buffer_in_bytes`
    与`total_indexing_buffer`相同，但以字节表示。

`transport_address`
    接受传输HTTP连接的主机和端口。

`version`
    在此节点上运行的Elasticsearch版本。

它还允许仅获取有关`settings`, `os`, `process`, `jvm`, `thread_pool`, `transport`, `http`, `plugins`, `ingest`和`indices`的信息：

```sh
# return just process
GET /_nodes/process

# same as above
GET /_nodes/_all/process

# return just jvm and process of only nodeId1 and nodeId2
GET /_nodes/nodeId1,nodeId2/jvm,process

# same as above
GET /_nodes/nodeId1,nodeId2/info/jvm,process

# return all the information of only nodeId1 and nodeId2
GET /_nodes/nodeId1,nodeId2/_all
```

`_all`标志可以设置为返回所有信息——或者您可以简单地省略它。

#### 操作系统信息
可以设置`os`标志以检索与操作系统有关的信息：

`os.refresh_interval_in_millis`
    刷新OS统计信息的时间间隔
    
`os.name`
    操作系统的名称（例如：Linux，Windows，Mac OS X）
    
`os.arch`
    JVM体系结构的名称（例如：amd64，x86）
    
`os.version`
    操作系统的版本
    
`os.available_processors`
    Java虚拟机可用的处理器数
    
`os.allocated_processors`
    实际用于计算线程池大小的处理器数。此编号可以使用节点的处理器设置进行设置，并且默认为OS报告的处理器数。在这两种情况下，这个数字永远不会大于32。

#### 进程信息
可以设置`process`标志以检索与当前运行进程有关的信息：

`process.refresh_interval_in_millis`
    刷新进程统计信息的时间间隔
    
`process.id`
    进程标识符（PID）
    
`process.mlockall`
    指示进程地址空间是否已成功锁定在内存中

#### 插件信息
`plugins` - 如果设置，结果将包含每个节点的已安装插件和模块的详细信息：

```sh
GET /_nodes/plugins
```

结果看起来类似于：

```json
{
  "_nodes": ...
  "cluster_name": "elasticsearch",
  "nodes": {
    "USpTGYaBSIKbgSUJR2Z9lg": {
      "name": "node-0",
      "transport_address": "192.168.17:9300",
      "host": "node-0.elastic.co",
      "ip": "192.168.17",
      "version": "{version}",
      "build_flavor": "{build_flavor}",
      "build_type": "zip",
      "build_hash": "587409e",
      "roles": [
        "master",
        "data",
        "ingest"
      ],
      "attributes": {},
      "plugins": [
        {
          "name": "analysis-icu",
          "version": "{version}",
          "description": "The ICU Analysis plugin integrates Lucene ICU module into elasticsearch, adding ICU relates analysis components.",
          "classname": "org.elasticsearch.plugin.analysis.icu.AnalysisICUPlugin",
          "has_native_controller": false
        }
      ],
      "modules": [
        {
          "name": "lang-painless",
          "version": "{version}",
          "description": "An easy, safe and fast scripting language for Elasticsearch",
          "classname": "org.elasticsearch.painless.PainlessPlugin",
          "has_native_controller": false
        }
      ]
    }
  }
}
```

每个插件和模块都有以下信息：

- `name`：插件名称
- `version`：该插件的Elasticsearch版本
- `description`：插件目的的简短描述
- `classname`：插件入口点的完全限定类名
- `has_native_controller`：插件是否具有本机控制器进程

#### 摄取信息
`ingest` - 如果设置，结果将包含有关每个节点的可用处理器的详细信息：

```sh
GET /_nodes/ingest
```

结果看起来类似于：

```json
{
  "_nodes": ...
  "cluster_name": "elasticsearch",
  "nodes": {
    "USpTGYaBSIKbgSUJR2Z9lg": {
      "name": "node-0",
      "transport_address": "192.168.17:9300",
      "host": "node-0.elastic.co",
      "ip": "192.168.17",
      "version": "{version}",
      "build_flavor": "{build_flavor}",
      "build_type": "zip",
      "build_hash": "587409e",
      "roles": [],
      "attributes": {},
      "ingest": {
        "processors": [
          {
            "type": "date"
          },
          {
            "type": "uppercase"
          },
          {
            "type": "set"
          },
          {
            "type": "lowercase"
          },
          {
            "type": "gsub"
          },
          {
            "type": "convert"
          },
          {
            "type": "remove"
          },
          {
            "type": "fail"
          },
          {
            "type": "foreach"
          },
          {
            "type": "split"
          },
          {
            "type": "trim"
          },
          {
            "type": "rename"
          },
          {
            "type": "join"
          },
          {
            "type": "append"
          }
        ]
      }
    }
  }
}
```

每个摄取处理器都有以下信息：

- `type`：处理器类型
