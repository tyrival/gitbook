## 搜索请求体
搜索请求可以在其请求体内使用[查询DSL](../11-Query-DSL/README.md)来执行查询。 这是一个例子：

```sh
GET /twitter/_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

示例响应如下：

```json
{
    "took": 1,
    "timed_out": false,
    "_shards":{
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
    },
    "hits":{
        "total" : 1,
        "max_score": 1.3862944,
        "hits" : [
            {
                "_index" : "twitter",
                "_type" : "_doc",
                "_id" : "0",
                "_score": 1.3862944,
                "_source" : {
                    "user" : "kimchy",
                    "message": "trying out Elasticsearch",
                    "date" : "2009-11-15T14:12:12",
                    "likes" : 0
                }
            }
        ]
    }
}
```

### 参数

| 参数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `timeout`                      | 搜索超时，将搜索请求限制在指定的时间值内执行，并使用在到期时累积的点击数进行保释。使用[取消搜索](../06-Search-APIs/README.md#取消搜索)机制在达到超时后取消搜索请求。默认为无超时。见[时间单位](../04-API-Conventions/Common-options.md#时间单位)。 |
| `from`                         | 检索命中结果的起始偏移量。默认为0。                            |
| `size`                         | 要返回的命中结果数量。默认为10.如果您不关心某些匹配但仅关于匹配和/或聚合的数量，则将值设置为0将有助于提高性能。 |
| `search_type`                  | 要执行的搜索操作的类型。可以是`dfs_query_then_fetch`或`query_then_fetch`。默认为`query_then_fetch`。请参阅[搜索类型](../06-Search-APIs/Request-Body-Search/Search-Type.md)以获取更多信息 |
| `request_cache`                | 设置为`true`或`false`以启用或禁用大小为0的请求的搜索结果缓存，即聚合和建议（未返回顶级匹配）。请参阅[分片请求缓存](../14-Modules/Indices/Shard-request-cache.md)。 |
| `allow_partial_search_results` | 如果请求将产生部分结果，则设置为`false`以返回整体故障。默认为`true`，这将在超时或部分失败的情况下允许部分结果。可以使用集群级别设置`search.default_allow_partial_results`来控制此默认值。 |
| `terminate_after`              | 在达到查询执行将提前终止时，为每个分片收集的最大文档数。如果设置，响应将有一个boolean字段`terminate_early`以指示查询执行是否实际终止了。默认为no terminate_after。 |
| `batched_reduce_size`          | 应在协调节点上一次减少的分片结果数。如果请求中潜在的分片数量很大，则应将此值用作保护机制，以减少每个搜索请求的内存开销。 |

除此之外，`search_type`，`request_cache`和`allow_partial_search_results`设置作为必须的查询字符串参数传递。搜索请求的其余部分应该在正文中传递。正文内容也可以作为名为`source`的REST参数传递。

HTTP GET和HTTP POST都可用于使用body执行搜索。由于并非所有客户端都支持带请求体的GET，因此也允许POST。

### 快速检查匹配的文档

> **注意**
>
> `terminate_after`始终在`post_filter`之后应用，并在分片上收集到足够的命中时停止查询以及聚合执行。虽然聚合的文档计数可能不会反映响应中的`hits.total`，因为聚合是在后过滤之前应用的。

如果我们只想知道是否有任何与特定查询匹配的文档，我们可以将大小设置为0，以表示我们对搜索结果不感兴趣。此外，我们可以将`terminate_after`设置为1，以指示只要找到第一个匹配的文档（每个分片），就可以终止查询执行。

```sh
GET /_search?q=message:number&size=0&terminate_after=1
```

响应将不包含任何匹配，因为`size`设置为0. `hits.total`将等于0，表示没有匹配的文档，而大于0意味着当查询提前终止时，至少存在与查询匹配的文档数量。此外，如果查询提前终止，则`terminate_early`标志将在响应中设置为`true`。

```json
{
  "took": 3,
  "timed_out": false,
  "terminated_early": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.0,
    "hits": []
  }
}
```

响应中的`took`时间包含此请求处理所需的毫秒数，在节点收到查询后快速开始，直到完成所有与搜索相关的工作，并且在将上述JSON返回给客户端之前。这意味着它包括在线程池中等待的时间，在整个集群中执行分布式搜索以及收集所有结果。
