## 创建一个索引

现在让我们创建一个名为“customer”的索引，然后再次列出所有索引：

```js
PUT /customer?pretty
GET /_cat/indices?v
```

第一个命令使用PUT动词创建名为“customer”的索引。我们只是简单地追加`pretty`到尾部，告诉它格式化打印JSON响应信息（如果存在JSON信息的话）。

响应：

```txt
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0       260b           260b
```

第二个命令的结果告诉我们，我们现在有一个名为customer的索引，它有5个主分片和1个副本（默认值），并且它包含0个文档。

您可能还注意到customer索引标记了黄色运行状况。回想一下我们之前的讨论，黄色表示某些副本尚未（尚未）分配。此索引发生这种情况的原因是因为默认情况下Elasticsearch为此索引创建了一个副本。由于我们目前只有一个节点在运行，因此在另一个节点加入集群的稍后时间点之前，尚无法分配一个副本（为了高可用性）。将该副本分配到第二个节点后，此索引的运行状况将变为绿色。