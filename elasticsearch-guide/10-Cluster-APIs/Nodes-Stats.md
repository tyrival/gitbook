## 节点统计信息
### 节点统计
集群节点统计信息API允许检索一个或多个（或所有）集群节点统计信息。
```sh
GET /_nodes/stats
GET /_nodes/nodeId1,nodeId2/stats
```

第一个命令检索集群中所有节点的统计信息。第二个命令选择性地检索仅`nodeId1`和`nodeId2`的节点统计信息。[这里](../10-Cluster-APIs/README.md)解释了所有节点选择性选项。

默认情况下，返回所有统计信息。你可以通过组合`indices`，`os`，`process`，`jvm`，`transport`，`http`，`fs`，`breaker`和`thread_pool`中的任何一个来限制它。例如：

| 参数                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| `indices`            | 包括大小，文档数量，索引和删除时间，搜索时间，字段缓存大小，合并和刷新的统计信息 |
| `fs`                 | 文件系统信息，数据路径，可用磁盘空间，读/写统计信息（请参阅[FS信息](#FS信息)） |
| `http`               | HTTP连接信息                                                 |
| `jvm`                | JVM统计信息，内存池信息，垃圾回收，缓冲池，加载/卸载类的数量 |
| `os`                 | 操作系统统计信息，平均负载，内存，交换（请参阅[操作系统统计信息](#操作系统统计信息)） |
| `process`            | 进程统计信息，内存消耗，cpu使用情况，打开文件描述符（请参阅[进程统计信息](#进程统计信息)） |
| `thread_pool`        | 每个线程池的统计信息，包括当前大小，队列和被拒绝的任务       |
| `transport`          | 集群通信中发送和接收字节的统计信息                   |
| `breaker`            | 现场数据断路器的统计数据                                 |
| `discovery`          | 发现的统计数据                                           |
| `ingest`             | 摄取预处理的统计信息                                     |
| `adaptive_selection` | [自适应副本选择](../06-Search-APIs/README.md#自适应副本选择)的统计信息请参阅[自适应选择统计](#自适应选择统计)             |

```sh
# return just indices
GET /_nodes/stats/indices

# return just os and process
GET /_nodes/stats/os,process

# return just process for node with IP address 10.0.0.1
GET /_nodes/10.0.0.1/stats/process
```

可以通过`/_nodes/stats/_all or /_nodes/stats?metric=_all`显式请求所有统计信息。

#### FS信息
可以设置fs标志以检索与文件系统有关的信息：

`fs.timestamp`
    上次刷新文件存储统计信息

`fs.total.total_in_bytes`
    所有文件存储的总大小（以字节为单位）

`fs.total.free_in_bytes`
    所有文件存储中未分配的字节总数

`fs.total.available_in_bytes`
    所有文件存储上此Java虚拟机可用的总字节数

`fs.data`
    所有文件存储的列表

`fs.data.path`
    文件存储的路径

`fs.data.mount`
    文件存储的挂载点（例如：/dev/sda2）

`fs.data.type`
    文件存储的类型（例如：ext4）

`fs.data.total_in_bytes`
    文件存储的总大小（以字节为单位）

`fs.data.free_in_bytes`
    文件存储中未分配的字节总数

`fs.data.available_in_bytes`
    此Java文件存储上此Java虚拟机可用的总字节数

`fs.data.spins`（仅限Linux）
    指示文件存储是否由旋转存储支持。 null表示我们无法确定它，true表示设备可能旋转，false表示它不能（例如：固态磁盘）。

`fs.io_stats.devices`（仅限Linux）
    支持Elasticsearch数据路径的每个设备的磁盘指标数组。定期探测这些磁盘指标，并计算最后一个探测和当前探测之间的平均值。

`fs.io_stats.devices.device_name`（仅限Linux）
    Linux设备名称。

`fs.io_stats.devices.operations`（仅限Linux）
    自启动Elasticsearch以来完成的设备读写操作总数。

`fs.io_stats.devices.read_operations`（仅限Linux）
    自启动Elasticsearch以来完成的设备读取操作总数。
`fs.io_stats.devices.write_operations`（仅限Linux）
    自启动Elasticsearch以来完成的设备写入操作总数。

`fs.io_stats.devices.read_kilobytes`（仅限Linux）
    自启动Elasticsearch以来为设备读取的总字节数。

`fs.io_stats.devices.write_kilobytes`（仅限Linux）
    自启动Elasticsearch以来为设备写入的总字节数。

`fs.io_stats.operations`（仅限Linux）
    自Elasticsearch启动以来Elasticsearch使用的所有设备的读写操作总数。

`fs.io_stats.read_operations`（仅限Linux）
    自Elasticsearch启动以来Elasticsearch使用的所有设备的读取操作总数。

`fs.io_stats.write_operations`（仅限Linux）
    自Elasticsearch启动以来Elasticsearch使用的所有设备的写入操作总数。

`fs.io_stats.read_kilobytes`（仅限Linux）
    自启动Elasticsearch以来Elasticsearch使用的所有设备读取的总字节数。

`fs.io_stats.write_kilobytes`（仅限Linux）
    自Elasticsearch启动以来Elasticsearch使用的所有设备上写入的千字节总数。

#### 操作系统统计信息
可以设置os标志以检索与操作系统有关的统计信息：

`os.timestamp`
    上次刷新操作系统统计信息

`os.cpu.percent`
    最近整个系统的CPU使用率，如果不支持，则为-1

`os.cpu.load_average.1m`
    系统上的一分钟负载平均值（如果一分钟负载平均值不可用，则不存在字段）

`os.cpu.load_average.5m`
    系统平均负载为5分钟（如果没有平均5分钟负载，则不存在字段）

`os.cpu.load_average.15m`
    系统上的平均负载为15分钟（如果没有十五分钟的负载平均值，则不存在字段）

`os.mem.total_in_bytes`
    物理内存总量，以字节为单位

`os.mem.free_in_bytes`
    可用物理内存量，以字节为单位

`os.mem.free_percent`
    可用内存的百分比

`os.mem.used_in_bytes`
    已用物理内存量，以字节为单位

`os.mem.used_percent`
    已用内存的百分比

`os.swap.total_in_bytes`
    交换空间总量，以字节为单位

`os.swap.free_in_bytes`
    可用交换空间量（以字节为单位）

`os.swap.used_in_bytes`
    已用交换空间量（以字节为单位）

`os.cgroup.cpuacct.control_group`（仅限Linux）
    Elasticsearch进程所属的cpuacct控制组

`os.cgroup.cpuacct.usage_nanos`（仅限Linux）
    与Elasticsearch进程在同一cgroup中的所有任务消耗的总CPU时间（以纳秒为单位）

`os.cgroup.cpu.control_group`（仅限Linux）
    Elasticsearch进程所属的cpu控制组

`os.cgroup.cpu.cfs_period_micros`（仅限Linux）
    与Elasticsearch进程在同一cgroup中的所有任务的定期时间（以微秒为单位）应该重新分配对CPU资源的访问权限。

`os.cgroup.cpu.cfs_quota_micros`（仅限Linux）
    与Elasticsearch进程相同的cgroup中的所有任务可以在一个时间段内运行的总时间（以微秒为单位）`os.cgroup.cpu.cfs_period_micros`

`os.cgroup.cpu.stat.number_of_elapsed_periods`（仅限Linux）
    已过去的报告周期数（由`os.cgroup.cpu.cfs_period_micros`指定）

`os.cgroup.cpu.stat.number_of_times_throttled`（仅限Linux）
    与Elasticsearch进程相同的cgroup中的所有任务都受到限制的次数。

`os.cgroup.cpu.stat.time_throttled_nanos`（仅限Linux）
    与Elasticsearch进程相同的cgroup中的所有任务都受到限制的总时间（以纳秒为单位）。

`os.cgroup.memory.control_group`（仅限Linux）
Elasticsearch进程所属的内存控制组

`os.cgroup.memory.limit_in_bytes`（仅限Linux）
        允许与Elasticsearch进程在同一cgroup中的所有任务的最大用户内存量（包括文件缓存）。此值可能太大而无法存储在long中，因此以字符串形式返回，以便返回的值可以与底层操作系统接口返回的值完全匹配。任何太大而无法解析为long的值几乎肯定意味着没有为cgroup设置限制。

`os.cgroup.memory.usage_in_bytes`（仅限Linux）
    由与Elasticsearch进程相同的cgroup中的所有任务按cgroup中的进程的总当前内存使用量（以字节为单位）。此值存储为字符串，以便与`os.cgroup.memory.limit_in_bytes`保持一致。

>**注意**
>要使cgroup统计信息可见，必须将cgroup编译到内核中，必须配置cpu和cpuacct cgroup子系统，并且必须可以从`/sys/fs/cgroup/cpu and /sys/fs/cgroup/cpuacct`中读取统计信息。

#### 进程统计信息
可以设置进程标志以检索与当前运行进程有关的统计信息：

`process.timestamp`
上次刷新流程统计信息

`process.open_file_descriptors`
    与当前进程关联的已打开文件描述符数，如果不支持，则为-1

`process.max_file_descriptors`
    系统允许的最大文件描述符数，如果不支持，则为-1

`process.cpu.percent`
    CPU使用率百分比，如果在计算统计数据时未知，则为-1

`process.cpu.total_in_millis`
    运行Java虚拟机的进程使用的CPU时间（以毫秒为单位），如果不支持，则为-1

`process.mem.total_virtual_in_bytes`
    保证可供正在运行的进程使用的虚拟内存的大小（以字节为单位）

### 索引统计
您可以获取有关`node`, `indices`,或`shards` `级别的索引统计信息。

```sh
# Fielddata summarised by node
GET /_nodes/stats/indices/fielddata?fields=field1,field2

# Fielddata summarised by node and index
GET /_nodes/stats/indices/fielddata?level=indices&fields=field1,field2

# Fielddata summarised by node, index, and shard
GET /_nodes/stats/indices/fielddata?level=shards&fields=field1,field2

# You can use wildcards for field names
GET /_nodes/stats/indices/fielddata?fields=field*
```

支持的指标是：
- `completion`
- `docs`
- `fielddata`
- `flush`
- `get`
- `indexing`
- `merge`
- `query_cache`
- `recovery`
- `refresh`
- `request_cache`
- `search`
- `segments`
- `store`
- `translog`
- `warmer`

### 搜索组
您可以获取有关在此节点上执行的搜索的搜索组的统计信息。
```sh
# All groups with all stats
GET /_nodes/stats?groups=_all

# Some groups from just the indices stats
GET /_nodes/stats/indices?groups=foo,bar
```

### 摄取统计信息
可以设置摄取标志以检索与摄取有关的统计信息：

`ingest.total.count`
    在此节点的生命周期内摄取的文档总数

`ingest.total.time_in_millis`
    在此节点的生命周期内，在摄取预处理文档上花费的总时间

`ingest.total.current`
    目前正在摄取的文件总数。

`ingest.total.failed`
    在此节点的生存期内，总数摄取预处理操作失败

除了这些总体摄取统计数据之外，还在每个管道的基础上提供这些统计数据。

#### 自适应选择统计
可以设置adaptive_selection标志以检索涉及自适应副本选择的统计数据。这些统计信息由节点键入。对于每个节点：

`adaptive_selection.outgoing_searches`
    从这些统计信息的节点到键控节点的未完成搜索请求的数量。

`avg_queue_size`
    密钥节点上搜索请求的指数加权移动平均队列大小。

`avg_service_time_ns`
    密钥节点上搜索请求的指数加权移动平均服务时间。

`avg_response_time_ns`
    密钥节点上搜索请求的指数加权移动平均响应时间。

`rank`
    该节点的等级;用于路由搜索请求时的分片选择。

