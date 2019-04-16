## 搜索API

现在让我们从一些简单的搜索开始吧。运行搜索有两种基本方法：一种是通过 [REST请求URI](../../06-Search-APIs/URI-Search.md) 发送搜索参数，另一种是通过 [REST请求体](../../06-Search-APIs/Request-Body-Search.md) 发送搜索参数。请求体方法允许您以更易读的JSON格式定义搜索。我们将尝试一个请求URI方法的示例，但是对于本教程的其余部分，我们将专门使用请求体方法。

可以从`_search`端点访问用于搜索的REST API。此示例返回银行索引中的所有文档：

```js
GET /bank/_search?q=*&sort=account_number:asc&pretty
```

作为CURLVIEW在CONSOLE中复制
让我们首先剖析搜索电话。我们在银行索引中搜索（`_search`端点），`q=*`参数表示Elasticsearch匹配索引中的所有文档。`sort=account_number:asc`参数表示使用文档的`account_number`字段对结果进行升序排序。`pretty`参数告诉Elasticsearch返回格式化的JSON结果。

响应（部分显示）：

```js
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```

响应内容中，我们看到以下部分：

- `took` - Elasticsearch执行搜索的时间（以毫秒为单位）
- `timed_out`  - 告诉我们搜索是否超时
- `_shards`  - 告诉我们搜索了多少个分片，以及搜索成功/失败分片的计数
- `hits` - 搜索结果
- `hits.total`  - 符合我们搜索条件的文档总数
- `hits.hits`  - 搜索结果的实际数组（默认为前10个文档）
- `hits.sort`  - 结果的排序键（如果按分数排序则丢失）
- `hits._score`和`max_score`  - 暂时忽略这些字段

以下是使用请求体的方法，功能上述完全相同：

```js
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

这里的不同之处在于，我们不是在URI中传递`q=*` ，而是向`_search` API提供JSON样式的查询请求体。我们将在下一节讨论这个JSON查询。

重要的是要理解，一旦您获得了搜索结果，Elasticsearch就完全完成了请求，并且不会在结果中维护任何类型的服务器端资源或打开游标。这与SQL等许多其他平台形成鲜明对比，其他平台您可能最初预先获得查询结果的部分子集，然后如果要获取（或翻页）其余的，则必须连续使用某种有状态服务器端游标，返回服务器的结果。