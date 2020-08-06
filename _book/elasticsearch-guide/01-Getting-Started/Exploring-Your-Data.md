## 探索数据

### 示例数据集

现在我们已经了解了基础知识，让我们尝试更真实的数据集。我准备了一份关于客户银行账户信息的虚构JSON文档样本。每个文档都有以下架构：

```json
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```

这些数据是使用www.json-generator.com/生成的，因此请忽略数据的实际值和语义，因为这些都是随机生成的。

### 加载示例数据集
您可以从 [此处](https://github.com/elastic/elasticsearch/raw/master/docs/src/test/resources/accounts.json)下载样本数据集（accounts.json）。将它解压缩到我们当前的目录，然后将它加载到我们的集群中，如下所示：

```sh
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
curl "localhost:9200/_cat/indices?v"
```

响应：

```txt
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank  l7sSYV2cQXmu6_4rJWVIww   5   1       1000            0    128.6kb        128.6kb
```

这意味着我们只是成功地将1000个文档批量索引到银行索引中（在`_doc`类型下）。
