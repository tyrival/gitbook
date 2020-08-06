## 执行查询

现在我们已经看到了一些基本的搜索参数，让我们再深入研究一下Query DSL。我们先来看一下返回的文档字段。默认情况下，完整的JSON文档作为完整搜索结果的一部分返回。这被称为源（搜索命中的`_source`字段）。如果我们不希望返回整个源文档，我们可以只请求返回源中的几个字段。

此示例显示如何从搜索中返回两个字段，`account_number`和`balance`（在`_source`内部）：

```sh
GET /bank/_search
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
```

请注意，上面的示例只是简化了`_source`字段。它仍将只返回一个名为`_source`的字段，但在其中，只包含字段`account_number`和`balance`。

如果您使用过SQL，则上述内容在概念上与SQL `SELECT FROM`字段列表有些相似。

现在让我们转到查询部分。以前，我们已经看到`match_all`查询如何用于匹配所有文档。现在让我们介绍一种称为 [匹配查询](../../11-Query-DSL/Full-text-queries/Match-Query.md) 的新查询，它可以被认为是基本的搜索查询（即针对特定字段或字段集进行的搜索）。

此示例返回编号为20的帐户：

```sh
GET /bank/_search
{
  "query": { "match": { "account_number": 20 } }
}
```

此示例返回地址中包含术语“mill”的所有帐户：

```sh
GET /bank/_search
{
  "query": { "match": { "address": "mill" } }
}
```

此示例返回地址中包含术语“mill”或“lane”的所有帐户：

```sh
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```

此示例是 `match` (`match_phrase`) 的变体，它返回地址中包含短语“mill lane”的所有帐户：

```sh
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

现在让我们介绍一下`bool`查询。 [bool查询](../../11-Query-DSL/Compound-queries/Bool-Query.md) 允许我们使用布尔逻辑，将较小的查询组成更大的查询。

此示例组合两个`match`查询，并返回地址中包含“mill”和“lane”的所有帐户：

```sh
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，`bool must`子句指定必须为true才能将文档视为匹配的所有查询。

相反，此示例组合两个`match`查询，并返回地址中包含“mill”或“lane”的所有帐户：

```sh
GET /bank/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，`bool should`子句指定了一个查询列表，其中任何一个查询必须为true才能使文档被视为匹配。

此示例组合两个`match`查询，并返回地址中既不包含“mill”也不包含“lane”的所有帐户：

```sh
GET /bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，`bool must_not`子句指定了一个查询列表，对于文档而言，这些查询都不能为真匹配。

我们可以在bool查询中同时组合`must`，`should`和`must_not`子句。此外，我们可以在任何这些bool子句中组合bool查询，来模仿任何复杂的多级布尔逻辑。

此示例返回任何40岁但未居住在ID（aho）的人的所有帐户：

```sh
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```
