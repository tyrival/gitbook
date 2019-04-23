## 挂起集群任务
挂起的集群任务API返回尚未执行的任何集群级更改（例如，创建索引，更新映射，分配或失败分片）的列表。

>**注意**
>此API返回所有挂起的更新集群状态的列表。 这些任务与[任务管理API](../10-Cluster-APIs/Task-Management-API.md)报告的任务不同，后者包括用户发起的定期任务和任务，例如节点统计信息，搜索查询或创建索引请求。 但是，如果用户启动的任务（如create index命令）导致集群状态更新，则任务api和挂起的集群任务API都可能会报告此任务的活动。
```sh
GET /_cluster/pending_tasks
```

通常，这将返回一个空列表，因为集群级别的更改通常很快。 但是，如果有排队的任务，输出将如下所示：
```json
{
   "tasks": [
      {
         "insert_order": 101,
         "priority": "URGENT",
         "source": "create-index [foo_9], cause [api]",
         "time_in_queue_millis": 86,
         "time_in_queue": "86ms"
      },
      {
         "insert_order": 46,
         "priority": "HIGH",
         "source": "shard-started ([foo_2][1], node[tMTocMvQQgGCkj7QDHl3OA], [P], s[INITIALIZING]), reason [after recovery from shard_store]",
         "time_in_queue_millis": 842,
         "time_in_queue": "842ms"
      },
      {
         "insert_order": 45,
         "priority": "HIGH",
         "source": "shard-started ([foo_2][0], node[tMTocMvQQgGCkj7QDHl3OA], [P], s[INITIALIZING]), reason [after recovery from shard_store]",
         "time_in_queue_millis": 858,
         "time_in_queue": "858ms"
      }
  ]
}
```
