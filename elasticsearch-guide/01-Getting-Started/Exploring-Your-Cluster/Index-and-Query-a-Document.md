## 索引并查询一个文档

现在让我们在客户索引中加入一些内容。我们将一个简单的customer文档编入customer索引中，ID为1，如下所示：

```sh
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
```

响应：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

从上面可以看出，在customer索引中成功创建了一个新的customer文档。该文档还具有我们在索引时指定的内部标识1。值得注意的是，Elasticsearch不需要在文档编入索引之前显式创建索引。在前面的示例中，如果客户索引事先尚未存在，则Elasticsearch将自动创建customer索引。

我们现在检索刚刚编入索引的文档：

```sh
GET /customer/_doc/1?pretty
```

响应：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 25,
  "_primary_term" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```

除了增加了个`found`字段之外，没有任何异常，声明我们找到了一个具有请求ID 1的文档和另一个字段`_source`，它返回我们从上一步索引的完整JSON文档。
