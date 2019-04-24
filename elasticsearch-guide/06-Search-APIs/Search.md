## 搜索
搜索API允许您执行搜索查询并返回匹配的结果。 可以使用简单的[查询字符串作为参数](../06-Search-APIs/URI-Search.md)，或使用[请求体](../06-Search-APIs/Request-Body-Search.md)来提供查询。

### 多索引
所有搜索API都可以跨多个索引应用，并支持[多索引语法](../04-API-Conventions/Multiple-Indices.md)。 例如，我们可以搜索twitter索引中的所有文档：

```sh
GET /twitter/_search?q=user:kimchy
```

我们还可以跨多个索引搜索具有特定标记的所有文档（例如，当每个用户有一个索引时）：

```sh
GET /kimchy,elasticsearch/_search?q=tag:wow
```

或者我们可以使用`_all`搜索所有可用的索引：
```sh
GET /_all/_search?q=tag:wow
```
