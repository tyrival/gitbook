## 任务管理
>**警告**
>任务管理API是新功能，仍应被视为测试版功能。 今后的API可能以不向后兼容的方式更改

### 当前任务信息
任务管理API允许检索有关当前在集群中的一个或多个节点上执行的任务的信息。

```sh
GET _tasks ①
GET _tasks?nodes=nodeId1,nodeId2 ②
GET _tasks?nodes=nodeId1,nodeId2&actions=cluster:* ③
```

① 检索当前在集群中的所有节点上运行的所有任务。

② 检索节点nodeId1和nodeId2上运行的所有任务。有关如何选择单个节点的详细信息，请参阅[指定节点](../10-Cluster-APIs/README.md#指定节点)。

③ 检索在节点nodeId1和nodeId2上运行的所有与集群相关的任务。

结果将类似于以下内容：
```json
{
  "nodes" : {
    "oTUltX4IQMOUUVeiohTt8A" : {
      "name" : "H5dfFeA",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1:9300",
      "tasks" : {
        "oTUltX4IQMOUUVeiohTt8A:124" : {
          "node" : "oTUltX4IQMOUUVeiohTt8A",
          "id" : 124,
          "type" : "direct",
          "action" : "cluster:monitor/tasks/lists[n]",
          "start_time_in_millis" : 1458585884904,
          "running_time_in_nanos" : 47402,
          "cancellable" : false,
          "parent_task_id" : "oTUltX4IQMOUUVeiohTt8A:123"
        },
        "oTUltX4IQMOUUVeiohTt8A:123" : {
          "node" : "oTUltX4IQMOUUVeiohTt8A",
          "id" : 123,
          "type" : "transport",
          "action" : "cluster:monitor/tasks/lists",
          "start_time_in_millis" : 1458585884904,
          "running_time_in_nanos" : 236042,
          "cancellable" : false
        }
      }
    }
  }
}
```
还可以检索特定任务的信息。以下示例检索有关任务`oTUltX4IQMOUUVeiohTt8A:124`

```sh
GET _tasks/oTUltX4IQMOUUVeiohTt8A:124
```

如果未找到任务，则API返回404。

要检索特定任务的所有子项：

```sh
GET _tasks?parent_task_id=oTUltX4IQMOUUVeiohTt8A:123
```

如果未找到父级，则API不会返回404。

您还可以使用`detailed`请求参数来获取有关正在运行的任务的更多信息。这对于从一个任务告知另一个任务很有用，但执行成本更高。例如，使用详细的请求参数获取所有搜索：

```sh
GET _tasks?actions=*search&detailed
```

结果可能如下所示：
```json
{
  "nodes" : {
    "oTUltX4IQMOUUVeiohTt8A" : {
      "name" : "H5dfFeA",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1:9300",
      "tasks" : {
        "oTUltX4IQMOUUVeiohTt8A:464" : {
          "node" : "oTUltX4IQMOUUVeiohTt8A",
          "id" : 464,
          "type" : "transport",
          "action" : "indices:data/read/search",
          "description" : "indices[test], types[test], search_type[QUERY_THEN_FETCH], source[{\"query\":...}]",
          "start_time_in_millis" : 1483478610008,
          "running_time_in_nanos" : 13991383,
          "cancellable" : true
        }
      }
    }
  }
}
```
新的`description`字段包含人类易读文本，其标识任务正在执行的特定请求，例如识别由搜索任务执行的搜索请求，如上面的示例。其他类型的任务有不同的描述，如[_reinde](../05-Document-APIs/Reindex-API.md)有搜索和目的地，或[_bulk](../05-Document-APIs/Bulk-API.md)只有请求数和目的地索引。许多请求只有空描述，因为有关请求的更详细信息不容易获得或在识别请求时特别有用。

>**重要**
>具有详细信息的`_tasks`请求也可能返回`status`。这是任务内部状态的报告。因此，其格式因任务而异。虽然我们尝试保持特定任务的状态在版本之间保持一致，但这并不总是可行的，因为我们有时会更改实现。在这种情况下，我们可能会从特定请求的状态中删除字段，因此您对`status`执行的任何解析都可能会在次要版本中中断。

任务API还可用于等待特定任务的完成。以下调用将阻止10秒或直到id为`oTUltX4IQMOUUVeiohTt8A:12345`的任务完成。

```sh
GET _tasks/oTUltX4IQMOUUVeiohTt8A:12345?wait_for_completion=true&timeout=10s
```

您还可以等待某些操作类型的所有任务完成。此命令将等待所有`reindex`任务完成：

```sh
GET _tasks?actions=*reindex&wait_for_completion=true&timeout=10s
```

还可以使用list tasks命令的_cat版本列出任务，该命令接受与标准列表任务命令相同的参数。

```sh
GET _cat/tasks
GET _cat/tasks?detailed
```

### 任务取消
如果要将长时间运行的任务取消，则可以使用取消任务API取消它。以下示例取消任务`oTUltX4IQMOUUVeiohTt8A:12345`:

```sh
POST _tasks/oTUltX4IQMOUUVeiohTt8A:12345/_cancel
```

任务取消命令支持与list tasks命令相同的任务选择参数，因此可以同时取消多个任务。例如，以下命令将取消在节点`nodeId1`和`nodeId2`上运行的所有reindex任务。

```sh
POST _tasks/_cancel?nodes=nodeId1,nodeId2&actions=*reindex
```

### 任务分组
任务API命令返回的任务列表可以按节点（默认）分组，也可以使用`group_by`参数按父任务分组。以下命令将分组更改为父任务：

```sh
GET _tasks?group_by=parents
```

可以通过将`group_by`参数指定为`none`来禁用分组：

```sh
GET _tasks?group_by=none
```

### 识别运行中任务
当在HTTP请求标头上提供时，`X-Opaque-Id`标头将作为响应中的标头，以及任务信息中的标头字段返回。这允许跟踪某些调用，或将某些任务与启动它们的客户端关联：

```sh
curl -i -H "X-Opaque-Id: 123456" "http://localhost:9200/_tasks?group_by=parents"
```

结果将类似于以下内容：

```sh
HTTP/1.1 200 OK
X-Opaque-Id: 123456 ①
content-type: application/json; charset=UTF-8
content-length: 831

{
  "tasks" : {
    "u5lcZHqcQhu-rUoFaqDphA:45" : {
      "node" : "u5lcZHqcQhu-rUoFaqDphA",
      "id" : 45,
      "type" : "transport",
      "action" : "cluster:monitor/tasks/lists",
      "start_time_in_millis" : 1513823752749,
      "running_time_in_nanos" : 293139,
      "cancellable" : false,
      "headers" : {
        "X-Opaque-Id" : "123456" ②
      },
      "children" : [
        {
          "node" : "u5lcZHqcQhu-rUoFaqDphA",
          "id" : 46,
          "type" : "direct",
          "action" : "cluster:monitor/tasks/lists[n]",
          "start_time_in_millis" : 1513823752750,
          "running_time_in_nanos" : 92133,
          "cancellable" : false,
          "parent_task_id" : "u5lcZHqcQhu-rUoFaqDphA:45",
          "headers" : {
            "X-Opaque-Id" : "123456" ③
          }
        }
      ]
    }
  }
}
```

① id作为响应头的一部分

② REST请求启动的任务的id

③ REST请求启动的任务的子任务
