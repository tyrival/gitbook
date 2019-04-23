## 集群健康度
集群运行状况API允许获得有关集群运行状况的非常简单的状态。 例如，在具有5个分片和一个副本的单个索引的安静单节点集群上，这：
```sh
GET _cluster/health
```

返回：
```json
{
  "cluster_name" : "testcluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 5,
  "active_shards" : 5,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 5,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 50.0
}
```

API也可以针对一个或多个索引执行，以获得指定的索引运行状况：
```sh
GET /_cluster/health/test1,test2
```

集群运行状况为：`green`, `yellow`, `red`。 在分片级别，`red`状态表示特定分片未在集群中分配，`yellow`表示分配了主分片但副本不分配，`green`表示分配了所有分片。 索引级别状态由最差的分片状态控制。 集群状态由最差的索引状态控制。

API的主要优点之一是能够等集群达到某个高水位健康级别。 例如，以下将等待50秒以使集群达到黄色级别（如果它在50秒之前达到绿色或黄色状态，它将在该点返回）：
```sh
GET /_cluster/health?wait_for_status=yellow&timeout=50s
```

### 请求参数
集群运行状况API接受以下请求参数：

`level`
值为`cluster`、`indices`或`shards`。控制返回的健康信息级别。默认为`cluster`。

`wait_for_status`
	值为`green`、`yellow`或`red`。将等待（直到提供超时），直到集群的状态变为指定状态或更好，即`green`>`yellow`>`red`。默认情况下，不会等待。

`wait_for_no_relocating_shards`
    一个布尔值，控制是否等待（直到超时提供）集群没有分片重定位。默认为`false`，这意味着它不会等待重新定位分片。

`wait_for_no_initializing_shards`
        一个布尔值，控制是否等待（直到提供超时）集群没有分片初始化。默认为`false`，这意味着它不会等待初始化分片。

`wait_for_active_shards`
一个控制等待多少活动分片的数字，`all`表示等待集群中的所有分片都处于活动状态，或者`0`表示不等待。默认为`0`。

`wait_for_nodes`
    请求等待，直到指定的`N`个节点可用。它还接受`>=N`，`<=N`，`>N`和`<N`。也可以使用`ge(N)`，`le(N)`，`gt(N)`和`lt(N)`代替符号。

`wait_for_events`
    值为`immediate`, `urgent`, `high`, `normal`, `low`, `languid`。等待，直到当前排队的事件具有给定优先级的事件。

`timeout`
    基于时间的参数控制，在提供wait_for_XXX之一时等待多长时间。默认为`30s`。

`master_timeout`
    基于时间的参数，用于控制尚未发现或断开主服务器的等待时间。如果未提供，则使用与`timeout`相同的值。

`local`
    如果为true则返回本地节点信息，并且不提供主节点的状态。默认值`false`。

以下是在分片级别获取集群运行状况的示例：
```sh
GET /_cluster/health/twitter?level=shards
```
