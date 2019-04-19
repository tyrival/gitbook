## Get API

Get API允许根据其id从索引中获取类型化的JSON文档。以下示例从名为twitter的索引中获取一个JSON文档，该索引名为`_doc`，id值为0：

```sh
GET twitter/_doc/0
```

上述get操作的结果是：

```json
{
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "0",
    "_version" : 1,
    "_seq_no" : 10,
    "_primary_term" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

上述结果中包含我们希望获取的文档的`index`，`_type`，`_id`和`_version`，以及实际`_source`（如果可以找到的话，如响应中的`found`字段所示）。

API还允许使用`HEAD`检查文档是否存在，例如：

```sh
HEAD twitter/_doc/0
```

### 实时

默认情况下，get API是实时的，并且不受索引刷新率的影响（当数据对搜索可见时）。如果文档已更新但尚未刷新，则get API将立刻发出刷新指令以使文档可见。这也将使上次刷新后其他文档发生变化。如需禁用实时GET，可以将`realtime`参数设置为`false`。

### Source过滤器

默认情况下，除非已使用`stored_fields`参数或禁用了`_source`字段，否则get操作将返回`_source`字段的内容。您可以使用`_source`参数关闭`_source`检索：

```sh
GET twitter/_doc/0?_source=false
```

如果您只需要完整`_source`中的一个或两个字段，则可以使用`_source_includes`和`_source_excludes`参数来包含或过滤掉您需要的部分。部分检索对于大型文档尤其有用，可以节省网络开销。这两个参数都使用逗号分隔的字段列表或通配符表达式。例：

```sh
GET twitter/_doc/0?_source_includes=*.id&_source_excludes=entities
```

如果您只想指定包含，则可以使用较短的表示法：

```sh
GET twitter/_doc/0?_source=*.id,retweeted
```

### 存储字段

Get操作允许指定一组存储的字段，这些字段将通过传递`stored_fields`参数返回。如果未存储请求的字段，则将忽略它们。例如，考虑以下映射：

```sh
PUT twitter
{
   "mappings": {
      "_doc": {
         "properties": {
            "counter": {
               "type": "integer",
               "store": false
            },
            "tags": {
               "type": "keyword",
               "store": true
            }
         }
      }
   }
}
```

现在我们可以添加一个文档：

```sh
PUT twitter/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

然后尝试检索它：

```sh
GET twitter/_doc/1?stored_fields=tags,counter
```

上述get操作的结果是：

```json
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "1",
   "_version": 1,
   "_seq_no" : 22,
   "_primary_term" : 1,
   "found": true,
   "fields": {
      "tags": [
         "red"
      ]
   }
}
```

从文档本身获取的字段值始终作为数组返回。由于未存储`counter`字段，因此在尝试获取`stored_field`时，get请求会忽略它。

也可以检索像`_routing`字段这样的元数据字段：

```sh
PUT twitter/_doc/2?routing=user1
{
    "counter" : 1,
    "tags" : ["white"]
}
```

```sh
GET twitter/_doc/2?routing=user1&stored_fields=tags,counter
```

上述get操作的结果是：

```sh
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "2",
   "_version": 1,
   "_seq_no" : 13,
   "_primary_term" : 1,
   "_routing": "user1",
   "found": true,
   "fields": {
      "tags": [
         "white"
      ]
   }
}
```

此外，只能通过`stored_field`选项返回叶子字段。因此无法返回对象字段，此类请求将失败。

### 直接获取`_source`

使用`/{index}/{type}/{id}/_source`端点只获取文档的`_source`字段，而不包含任何其他内容。例如：

```sh
GET twitter/_doc/1/_source
```

您还可以使用相同的源过滤参数来控制返回`_source`的哪些部分：

```sh
GET twitter/_doc/1/_source?_source_includes=*.id&_source_excludes=entities
```

注意，`_source`端点还有一个HEAD变体，可以有效地测试文档源的存在。如果在 [映射](../12-Mapping/Meta-Fields/_source-field.md) 中禁用了现有文档，则该文档将没有`_source`。

```sh
HEAD twitter/_doc/1/_source
```

### 路由

索引使用控制路由的功能时，为了获取文档，还应提供路由值。例如：

```sh
GET twitter/_doc/2?routing=user1
```

以上将获得id为`2`的推文，但将根据用户进行路由。请注意，在没有正确路由的情况下发出get，将导致无法获取文档。

### 首选项

控制哪个分片副本执行get请求的`preference`。默认情况下，操作在分片副本之间随机化。

`preference`可以设置为：

`_primary`

​	该操作将仅在主分片上执行。

`_local`

​	如果可能，操作将优选在本地分配的分片上执行。

**自定义（字符串）值**

​	将使用自定义值来保证相同的分片将用于相同的自定义值。当在不同的刷新状态下命中不同的分片时，这可以帮助“跳跃值”。示例值可以是web session ID或用户名。

### 刷新

可以将`refresh`参数设置为`true`，以便在get操作之前刷新相关的分片并使其可搜索。将其设置为`true`应该仔细考虑并验证，确定这不会导致系统负载过重（并减慢索引速度）之后进行。

### 分发

Get操作被hash为特定的分片ID。然后它被重定向到该分片ID中的一个副本并返回结果。副本是主分片及其在该分片ID组中的副本。这意味着我们拥有的副本越多，我们将拥有更好的GET缩放。

### 版本控制支持

仅当`version`参数的当前版本等于指定版本时，才可以使用此参数来检索文档。除了始终检索文档的版本类型`FORCE`之外，所有版本类型的此行为都相同。请注意，不推荐使用`FORCE`版本类型。

在内部，Elasticsearch已将旧文档标记为已删除，并添加了一个全新的文档之后，旧版本的文档不会立即消失，但您将无法访问它。当您继续索引更多数据时，Elasticsearch会在后台清除已删除的文档。
