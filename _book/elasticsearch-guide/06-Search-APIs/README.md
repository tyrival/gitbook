# 搜索API
大多数搜索API都是[多索引](../06-Search-APIs/Search.md)的，但[说明API](../06-Search-APIs/Explain-API.md)端点除外。

### 路由
执行搜索时，它将广播到所有索引/索引分片（副本之间的循环）。 可以通过提供路由参数来控制将搜索哪些分片。 例如，在索引推文时，路由值可以是用户名：

```sh
POST /twitter/_doc?routing=kimchy
{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

在这种情况下，如果我们只想搜索特定用户的推文，我们可以将其指定为路由，从而导致搜索只接触相关的分片：

```sh
POST /twitter/_search?routing=kimchy
{
    "query": {
        "bool" : {
            "must" : {
                "query_string" : {
                    "query" : "some query string here"
                }
            },
            "filter" : {
                "term" : { "user" : "kimchy" }
            }
        }
    }
}
```

路由参数可以是多值，为逗号分隔的字符串。这将导致命中路由值匹配的相关分片。

## 自适应副本选择
作为以循环方式发送到数据副本的请求的替代方法，您可以启用自适应副本选择。这允许协调节点根据如下标准，将请求发送到被认为“最佳”的副本：

- 协调节点与包含数据副本的节点之间传递请求的响应时间
- 搜索请求在包含数据的节点上的执行时间
- 包含数据的节点上的搜索线程池的队列大小

可以通过将动态集群设置`cluster.routing.use_adaptive_replica_selection`从`false`更改为`true`来启用此功能：

```sh
PUT /_cluster/settings
{
    "transient": {
        "cluster.routing.use_adaptive_replica_selection": true
    }
}
```

## 统计分组
搜索可以与统计组相关联，统计组维护每个组的统计聚合。稍后可以使用[索引统计信息](../08-Indices-APIs/Indices-Stats.md) API专门检索它。例如，这是一个搜索正文请求，它将请求与两个不同的组相关联：

```sh
POST /_search
{
    "query" : {
        "match_all" : {}
    },
    "stats" : ["group1", "group2"]
}
```

## 全局搜索超时
作为[搜索请求体](../06-Search-APIs/Request-Body-Search.md)的一部分，单个搜索可能会超时。由于搜索请求可能有许多来源，因此Elasticsearch具有全局搜索超时的动态集群级设置，该设置适用于未在请求正文中设置超时的所有搜索请求。这些请求将在指定时间后[取消搜索](#取消搜索)。因此，关于超时响应的相同警告适用。

设置键为`search.default_search_timeout`，可以使用[更新集群设置](../10-Cluster-APIs/Cluster-Update-Settings.md)端点进行设置。默认值为无全局超时。将此值设置为`-1`会将全局搜索超时重置为无超时。

## 取消搜索
可以使用标准[任务取消](../10-Cluster-APIs/Task-Management-API.md#任务取消)机制取消搜索。默认情况下，正在运行的搜索仅检查它是否在片段的边界上被取消，因此大片段延迟可能会使请求延迟。通过将动态集群级别设置`search.low_level_cancellation`设置为`true`，可以提高搜索取消响应效率。但是，它带来了更频繁的取消检查的额外开销，这在大型快速运行的搜索查询中是明显的。更改此设置仅影响更改后开始的搜索。

## 并发和平行搜索
默认情况下，Elasticsearch不会根据请求命中的分片数拒绝任何搜索请求。虽然Elasticsearch将优化协调节点上的搜索执行，但大量分片会对CPU和内存产生重大影响。以这样的方式组织数据通常更好，即更少的大分片。如果您要配置软限制，可以更新群集设置`action.search.shard_count.limit`，以拒绝搜索过多分片的搜索请求。

请求参数`max_concurrent_shard_requests`可用于控制搜索API为请求执行的最大并发分片请求数。此参数应用于保护单个请求不会使群集过载（例如，默认请求将命中群集中的所有索引，如果每个节点的分片数量很高，则可能导致碎片请求被拒绝）。此默认值基于群集中的数据节点数，但最多为`256`个。
