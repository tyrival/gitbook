## 批处理

除了能够索引，更新和删除单个文档之外，Elasticsearch还提供了使用 [复合操作API](../../05-Document-APIs/Bulk-API.md) 批量执行上述任何操作的功能。此功能非常重要，因为它提供了一种非常有效的机制，可以尽可能快地进行多个操作，并尽可能少地进行网络开销。

作为一个简单的示例，以下调用在一个批量操作中索引两个文档（ID 1  -  John Doe和ID 2  -  Jane Doe）：

```sh
POST /customer/_doc/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```

此示例更新第一个文档（ID为1），然后在一个批量操作中删除第二个文档（ID为2）：

```sh
POST /customer/_doc/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```

请注意，对于删除操作，之后没有相应的源文档，因为删除只需要删除文档的ID。

复合操作API不会因其中一个操作失败而失败。如果单个操作因任何原因失败，它将继续处理其后的其余操作。批量API返回时，它将为每个操作提供一个状态（按照发送的顺序），以便您可以检查特定操作是否失败。

