## Index API

> **重要**
>
> 请看 [移除映射类型](../12-Mapping/Removal-of-mapping-types.md)。

Index API在指定索引中添加或更新JSON文档，使其可搜索。以下示例将JSON文档插入到类型为`_doc`，id为1的“twitter”索引中：

```js
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上述索引操作的结果是：

```js
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,w
    "result" : "created"
}
```

`_shards`标头提供有关索引操作的复制过程的信息：

`total`

​	表示执行索引操作的分片副本（主分片和副本分片）的数量。

`successful`

​	表示索引操作成功的分片副本数。

`failed`

​	索引操作失败的情况下，包含与复制相关的错误的数组。

在`successful`至少为1的情况下，索引操作是成功的。

> **注意**
>
> 索引操作成功返回时，可能无法全部启动副本分片（默认情况下只需要主分片，但可以 [更改](#等待活跃分片) 此行为）。在这种情况下，`total`将等于`number_of_replicas`设置的总分片数，并且`success`将等于已启动的分片数（主分片加上副本分片）。如果没有失败，则`failed`将为0。

### 自动索引创建

索引操作会在索引不存在时自动创建，并应用已配置的 [索引模板](../08-Indices-APIs/Index-Templates.md)。索引操作还会为指定的类型创建动态类型映射（如果尚不存在）。默认情况下，如果需要，新字段和对象将自动添加到指定类型的映射定义中。有关映射定义的更多信息，请查看 [映射](../12-Mapping/README.md)；有关手动更新类型映射的信息，请查看 [Put映射API](../08-Indices-APIs/Put-Mapping.md)。

自动索引创建由`action.auto_create_index`设置控制。此设置默认为`true`，表示始终自动创建索引。将此设置的值更改为由逗号分隔的匹配规则列表，则可仅使匹配规则索引的自动创建。它也可以通过在列表中用`+`或`-`前缀来明确允许和禁止。最后，将此设置更改为`false`，则可以完全禁用它。

```js
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*" ①
    }
}

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "false" ②
    }
}

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true" ③
    }
}
```


①  只允许自动创建名为`twitter`，`index10`的索引，不允许匹配`index1*`的索引，允许匹配`ind*`的索引。匹配规则按照给定的顺序进行匹配。

②  完全禁用索引的自动创建。

③  允许使用任何名称自动创建索引。这是默认值。

### 类型操作

索引操作使用`op_type`允许“缺席”，强制`create`操作。使用`create`时，如果索引中已存在该id的文档，则索引操作将失败。

以下是使用`op_type`参数的示例：

```js
PUT twitter/_doc/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

`create`的另一个方式是使用以下uri：

```js
PUT twitter/_doc/1/_create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

### 自动生成ID

可以在不指定id的情况下执行索引操作。在这种情况下，将自动生成id。此外，`op_type`将自动设置为`create`。这是一个例子（注意使用**POST**而不是**PUT**）：

```js
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上述索引操作的结果是：

```js
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "W0tpsmIBdwcYyG50zbta",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result": "created"
}
```

### 乐观并发控制

索引操作可以是有条件的，只有在为文档的最后一次修改分配了`if_seq_no`和`if_primary_term`参数，指定了序列号和主要术语时，才能执行索引操作。如果检测到不匹配，则操作将导致`VersionConflictException`和状态码409。有关详细信息，请参阅 [乐观并发控制](../05-Document-APIs/Optimistic-concurrency-control.md)。

### 路由

默认情况下，通过使用文档id的哈希值来控制分片放置或`routing`。 为了更明确地控制，可以使用`routing`参数在每个操作的基础上直接指定馈入路由器使用的散列函数的值。 例如：

```js
POST twitter/_doc?routing=kimchy
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

“_doc”文档根据提供的`routing`参数路由到分片：“kimchy”。

在设置显式映射时，可以选择使用`_routing`字段来指示索引操作从文档本身中提取路由值。 这个额外的文档解析过程会产生很小的开销。 当定义了`_routing`映射并将其设置为必需，如果未提供或提取路由值，则索引操作将失败。

### 分发

索引操作根据其路由指向主分片（请参阅上面的“路由”部分），并在包含此分片的实际节点上执行。 主分片完成操作后，如果需要，更新将分发到适用的副本。

### 等待活跃分片

为了提高对系统写入的弹性，可以将索引操作配置为在继续操作之前，等待一定数量的活动分片副本。如果活动分片副本数量达不到此数量，则写入操作必须等待并重试，直到必需的分片副本已启动或发生超时。默认情况下，写操作仅等待主分片在继续之前处于活动状态（即`wait_for_active_shards=1`）。可以通过设置`index.write.wait_for_active_shards`在索引设置中动态覆盖此默认值。要更改每个操作的此行为，可以使用`wait_for_active_shards`请求参数。

有效值是`all`或任何正整数，直到索引中每个分片的已配置副本总数（即`number_of_replicas+1`）。指定负值或大于分片副本数的数字将引发错误。

例如，假设我们有一个包含三个节点A，B和C的集群，我们创建一个索引`index`，副本数设置为3（产生4个分片副本，比节点多一个副本）。如果我们尝试索引操作，默认情况下，操作只会确保每个分片的主副本在继续之前可用。这意味着即使B和C发生故障，并且A托管了主要分片副本，索引操作仍将仅使用一个数据副本。如果在请求为3（并且所有3个节点都已启动）上设置了`wait_for_active_shards`，则索引操作将需要3个活动分片副本才能继续执行，因为集群中有3个活动节点，每个都有一个副本，因此应该满足该要求碎片的副本。但是，如果我们将`wait_for_active_shards`设置为`all`（或者4，它们是相同的），则索引操作将不会继续，因为我们没有在索引中激活每个分片的所有4个副本。除非在集群中启动新节点以托管分片的第四个副本，否则操作将超时。

需要重点注意的是，在写入操作中，此设置大大降低了所需分片副本不被写入的可能性，但它并未完全消除这种可能性，因为此检查在写入操作开始之前发生。一旦写入操作正在进行，某个分片副本仍然可能失效，但仍可在主要副本上成功。写操作响应的`_shards`部分显示复制成功/失败的分片副本数。

```js
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    }
}
```

### 刷新

控制请求所做的更改结果何时可被搜索到。请参阅 [刷新](../05-Document-APIs/refresh.md)。

### 空操作更新

使用Index API更新文档时，即使文档未更改，也始终会创建新版本的文档。如果不希望这样做，请使用`_update` API，并将`detect_noop`设置为true。此选项在Index API上不可用，因为Index API不会获取旧源，也无法将其与新源进行比较。

关于何时不接受空操作更新，没有一条硬性规定。它是许多因素的组合，例如您的数据源发送实际空操作更新的频率，以及Elasticsearch在接收更新的分片上运行的每秒查询数。

### 超时

执行索引操作时，执行索引操作的主分片可能失效。原因可能是主分片当前正从网关恢复或正在进行重定位。默认情况下，索引操作将在主分片上等待最多1分钟，然后失败并响应错误。 `timeout`参数可用于显式指定等待的时间。以下是将其设置为5分钟的示例：

```js
PUT twitter/_doc/1?timeout=5m
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

### 版本

每个索引文档都有一个版本号。默认情况下，使用从1开始的内部版本控制，并在每次更新（包括删除）时递增。可选地，版本号可以设置为外部值（例如在数据库中维护）。要启用此功能，应将`version_type`设置为`external`。提供的值必须是大于等于0且小于9.2e+18的Long类型。

使用外部版本类型时，系统会检查传递给索引请求的版本号是否大于当前存储文档的版本号。如果为true，则将索引文档并使用新版本号。如果提供的值小于或等于存储文档的版本号，则会发生版本冲突，索引操作将失败。例如：

```js
PUT twitter/_doc/1?version=2&version_type=external
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
```

**注意：**版本控制是完全实时的，不受搜索操作的近实时方面的影响。如果未提供任何版本，则执行该操作而不进行任何版本检查。

由于提供的版本2高于当前文档版本1，则上述操作将成功。如果文档已更新且其版本设置为2或更高，则索引命令将失败并导致冲突（409 http状态代码）。

一个好的副作用是，当版本号切换为来源于数据库时，不需要严格维护异步索引操作的执行顺序。如果使用外部版本控制，使用数据库中的数据更新Elasticsearch索引的用例会被简化，因为索引操作时无论发生什么故障，都只会使用最新的版本号。

#### 版本类型

除了上述的`external`版本类型，Elasticsearch还支持用于特定场景其他类型。以下是不同版本类型及其概述。

`internal`

​	仅在给定版本与存储文档的版本相同时才对文档索引。 （在6.7.0中弃用。请改用`if_seq_no`和`if_primary_term`。有关更多详细信息，请参阅 [乐观并发控制](../05-Document-APIs/Optimistic-concurrency-control.md)）。

`external`或`external_gt`

​	如果给定版本严格高于存储文档的版本或者没有现有文档，则仅索引文档。给定版本将用作新版本，并将与新文档一起存储。提供的版本必须是非负Long型。

`external_gte`

​	仅在给定版本等于或高于存储文档的版本时索引文档。如果没有现有文档，操作也将成功。给定版本将用作新版本，并将与新文档一起存储。提供的版本必须是非负Long型。

> **注意**：`external_gte`版本类型适用于特殊用例，应小心使用。如果使用不当，可能会导致数据丢失。还有另一个选项`force`，它被弃用，因为它可能导致主分片和副本分片出现偏差。
