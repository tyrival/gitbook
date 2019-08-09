## URI搜索
可以通过定义请求参数来使用URI进行搜索。此搜索模式的参数并未包含所有的请求配置项，但它可以方便快速地使用"curl"进行测试。 示例如下：

```
GET twitter/_search?q=user:kimchy
```

响应示例如下：

```json
{
    "timed_out": false,
    "took": 62,
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
                    "date" : "2009-11-15T14:12:12",
                    "message" : "trying out Elasticsearch",
                    "likes": 0
                }
            }
        ]
    }
}
```

### 参数
URI中的参数如下：

| 参数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `q`                            | 查询字符串（映射到`query_string`查询，有关详细信息，请参阅[query_string查询](../11-Query-DSL/Full-text-queries/Query-String-Query.md)）。 |
| `df`                           | 在查询中未定义字段前缀时使用的默认字段。                     |
| `analyzer`                     | 分析查询字符串时要使用的分析器名称。                         |
| `analyze_wildcard`             | 是否应分析通配符和前缀查询。默认为`false`。                    |
| `batched_reduce_size`          | 协调节点上一次减少的分片结果数。如果请求中潜在的分片数量很大，则应将此值用作保护机制，以减少每个搜索请求的内存开销。 |
| `default_operator`             | 要使用的默认运算符，可以是`AND`或`OR`。默认为`OR`。                  |
| `lenient`                      | 如果设置为`true`将忽略基于格式的失败（如向数字字段提供文本）。默认为`false`。 |
| `explain`                      | 对于每个命中的解释，包含如何计算命中得分。                   |
| `_source`                      | 设置为`false`以禁用`_source`字段的检索。您还可以使用`_source_includes`和`_source_excludes`检索部分文档（有关详细信息，请参阅[请求体](../06-Search-APIs/Request-Body-Search/Source-filtering.md)文档） |
| `stored_fields`                | 每个匹配的文档的返回的字段选择，逗号分隔。不指定任何值将导致不返回任何字段。 |
| `sort`                         | 排序执行。可以是`fieldName`或`fieldName:asc` / `fieldName:desc`的形式。 `fieldName`可以是文档中的实际字段，也可以是特殊的`_score`名称，表示基于分数的排序。可以有几个`sort`参数（顺序很重要）。 |
| `track_scores`                 | 排序时，设置为`true`以便保持跟踪分数，并将其作为每个匹配的一部分返回。 |
| `track_total_hits`             | 设置为`false`以禁用跟踪匹配总数。 （有关详细信息，请参阅[索引排序](../15-Index-Modules/Index-Sorting.md)）。默认为true。 |
| `timeout`                      | 搜索超时，将搜索请求限制在指定的时间值内执行，并使用在到期时累积的点击数进行保释。默认为无超时。 |
| `terminate_after`              | 在达到查询执行将提前终止时，为每个分片收集的最大文档数。如果设置，响应将有一个boolean字段`terminate_early`以指示查询执行是否实际终止了。默认为no terminate_after。 |
| `from`                         | 从命中的索引开始返回。默认为`0`。                              |
| `size`                         | 要返回的命中数量。默认为`10`。                                 |
| `search_type`                  | 要执行的搜索操作的类型。可以是`dfs_query_then_fetch`或`query_then_fetch`。默认为`query_then_fetch`。有关可以执行的不同搜索类型的更多详细信息，请参阅[查询类型](../06-Search-APIs/Request-Body-Search/Search-Type.md)。 |
| `allow_partial_search_results` | 如果请求将产生部分结果，则设置为`false`以返回整体故障。默认为`true`，这将在超时或部分失败的情况下允许部分结果。可以使用集群级别设置·来控制此默认值。 |

