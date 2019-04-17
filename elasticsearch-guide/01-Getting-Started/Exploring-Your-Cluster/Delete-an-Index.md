## 删除一个索引

现在让我们删除刚刚创建的索引，然后再次列出所有索引：

```sh
DELETE /customer?pretty
GET /_cat/indices?v
```

响应：

```txt
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

这意味着索引已成功删除，现在我们回到了我们在集群中，没有任何内容。

在我们继续之前，让我们再仔细看看到目前为止我们学到的一些API命令：

```sh
PUT /customer
PUT /customer/_doc/1
{
  "name": "John Doe"
}
GET /customer/_doc/1
DELETE /customer
```

如果我们仔细研究上述命令，我们实际上可以看到在Elasticsearch中访问数据的模式。 该模式可归纳如下：

```sh
<HTTP Verb> /<Index>/<Type>/<ID>
```

这种REST访问模式在所有API命令中都非常普遍，如果您能够记住它，将在掌握Elasticsearch方面有一个良好的开端。
