## 查询语法介绍

Elasticsearch提供了一种JSON样式的特定语言，可用于执行查询。这被称为 [查询DSL](../../11-Query-DSL/README.md)。查询语言非常全面，乍一看可能复杂的令人生畏，但实际上可以从一些基本示例开始学习。

回到上一个例子，我们执行了这个查询：

```sh
GET /bank/_search
{
  "query": { "match_all": {} }
}
```

解析上面的内容，`query`部分告诉我们查询定义是什么，`match_all`部分只是我们想要运行的查询类型。 `match_all`表示只搜索指定索引中的所有文档。

除了`query`参数，我们还可以传递其他参数来影响搜索结果。在上面的部分示例中，我们传递了`sort`，这里我们传入`size`：

```sh
GET /bank/_search
{
  "query": { "match_all": {} },
  "size": 1
}
```

请注意，如果未指定大小，则默认为10。

此示例执行`match_all`并返回文档10到19：

```sh
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
```

`from`参数（从0开始）指定从哪个文档索引开始，`size`参数指定从`from`参数开始返回的文档数。在实现搜索结果的分页时，此功能非常有用。请注意，如果未指定`from`，则默认为0。

此示例执行`match_all`并按帐户余额对结果进行降序排序，并返回前10个（默认大小）文档。

```sh
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
```
