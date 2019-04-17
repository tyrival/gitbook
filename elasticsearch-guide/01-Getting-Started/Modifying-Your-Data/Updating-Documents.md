## 更新文档

除了能够索引和替换文档，我们还可以更新文档。请注意，Elasticsearch实际上并没有在原有地点覆盖更新。每当我们进行更新时，Elasticsearch都会删除旧文档，然后对新文档创建索引。

此示例显示如何通过将name字段更改为“Jane Doe”，来更新以前的文档（ID为1）：

```sh
POST /customer/_doc/1/_update?pretty
{
  "doc": { "name": "Jane Doe" }
}
```

此示例显示如何通过将name字段更改为“Jane Doe”来更新我们以前的文档（ID为1），同时为其添加age字段：

```sh
POST /customer/_doc/1/_update?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
```

也可以使用简单脚本执行更新。此示例使用脚本将age增加5：

```sh
POST /customer/_doc/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```

在上面的示例中，`ctx._source`指的是即将更新的当前源文档。

Elasticsearch提供了在给定查询条件（如SQL UPDATE-WHERE语句）的情况下，更新多个文档的功能。请参阅 [查询并更新API](../../05-Document-APIs/Update-By-Query-API.md)。
