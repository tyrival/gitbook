## 删除文档

删除文档非常简单。 此示例显示如何删除ID为2的以前的客户：

```sh
DELETE /customer/_doc/2?pretty
```

查看 [Delete By Query API](../../05-Document-APIs/Delete-By-Query-API.md) 以删除与特定查询匹配的所有文档。 值得注意的是，删除整个索引比使用 [Delete By Query API](../../05-Document-APIs/Delete-By-Query-API.md) 删除所有文档会更有效率。

