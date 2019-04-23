## Delete API

Delete API允许根据id从指定索引删除指定类型的JSON文档。以下示例从名为twitter的索引中删除JSON文档，该索引名为`_doc`，id为`1`：

```sh
DELETE / twitter / _doc / 1
```
上述删除操作的结果是：

```json
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 2,
    "_primary_term": 1,
    "_seq_no": 5,
    "result": "deleted"
}
```

### 乐观并发控制

删除操作可以是有条件的，只有在为文档的最后一次修改通过指定`if_seq_no`和`if_primary_term`，来分配了序列号和主要术语时才能执行。如果检测到不匹配，则操作将导致`VersionConflictException`和状态代码409.有关详细信息，请参阅[乐观并发控制](../05-Document-APIs/Optimistic-concurrency-control.md)。

### 版本控制
索引的每个文档都是版本化的。删除文档时，可以指定`version`，以确保我们尝试删除的相关文档实际上已被删除，并且在此操作期间没有更改。对文档执行的每个写操作（包括删除）都会导致其版本递增。删除文档的版本号在删除后仍可短时间使用，以便控制并发操作。已删除文档的版本保持可用的时间长度由`index.gc_deletes`索引设置确定，默认为60秒。

### 路由
使用控制路由的能力进行索引时，为了删除文档，还应提供路由值。例如：

```sh
DELETE /twitter/_doc/1?routing=kimchy
```

以上将删除id为`1`的推文，但将根据用户进行路由。请注意，在没有正确路由的情况下发出删除将导致文档不被删除。

当`_routing`映射根据`require`设置且未指定路由值时，delete API将抛出`RoutingMissingException`并拒绝该请求。

### 自动创建索引
如果使用[外部版本控制](../05-Document-APIs/Index-API.md)，则删除操作会自动创建索引（如果之前尚未创建索引）（请参阅创建[索引API`](../08-Indices-APIs/Create-Index.md)以手动创建索引），并且还会自动为特定类型创建动态类型映射（如果它之前尚未创建（请查看[put mapping API](../08-Indices-APIs/Put-Mapping.md)以手动创建类型映射）。

### 分发
删除操作将hash为特定的分片ID。然后它被重定向到该id组中的主分片，并复制（如果需要）到该id组内的分片副本。

### 等待活跃分片
进行删除请求时，可以将`wait_for_active_shards`参数设置为在开始处理删除请求之前，至少要求一定数量的分片副本处于活动状态。有关更多详细信息和用法示例，请参见[此处](../05-Document-APIs/Index-API.md#等待活跃分片)。

### 刷新
控制何时此请求所做的更改对搜索可见。看[?refresh](../05-Document-APIs/refresh.md)。

### 超时
执行删除操作时，分配用于执行删除操作的主分片可能不可用。原因可能是主分片当前正在从存储恢复或正在进行重定位。默认情况下，删除操作将在主分片上等待最多1分钟，然后失败并响应错误。 `timeout`参数可用于显式指定等待的时间。以下是将其设置为5分钟的示例：

```sh
DELETE /twitter/_doc/1?timeout=5m
```
