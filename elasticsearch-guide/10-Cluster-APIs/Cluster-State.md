## 集群状态
集群状态API允许访问表示整个集群状态的元数据。这包括诸如此类的信息
- 集群的节点集合
- 所有集群级别设置
- 有关集群中索引的信息，包括其映射和设置
- 集群中所有分片的位置

响应是集群状态的内部呈现，其格式可能会因版本而异。如果可能，您应该使用其他更稳定的[集群API](../10-Cluster-APIs/README.md)从集群状态获取任何信息。
```sh
GET /_cluster/state
```

响应提供了集群本身的状态，可以对其进行过滤以仅检索感兴趣的部分，如下所述。

除元数据部分外，集群的`cluster_uuid`也作为顶级响应的一部分返回(在6.4.0中添加)。

>**注意**
>当集群仍在形成时，`cluster_uuid`可能是`_na_`以及集群状态的版本为`-1`。

默认情况下，集群状态请求将路由到主节点，以确保返回最新的集群状态。出于调试目的，您可以通过将`local=true`添加到查询字符串，来检索特定节点的本地集群状态。

### 响应过滤器
集群状态包含有关集群中所有索引的信息，包括它们的映射，以及模板和其他元数据。这意味着它有时可能非常大。为了避免处理所有这些信息，您只能请求所需的集群状态部分：
```sh
GET /_cluster/state/{metrics}
GET /_cluster/state/{metrics}/{indices}
```

`{metrics}`是以逗号分隔的以下选项列表。

`version`
    显示集群状态版本。
    
`master_node`
    显示响应的选定`master_node`部分
    
`nodes`
    显示响应的节点部分
    
`routing_table`
    显示响应的`routing_table`部分。如果提供以逗号分隔的索引列表，则返回的输出将仅包含这些索引的路由表。
    
`metadata`
    显示响应的元数据部分。如果提供以逗号分隔的索引列表，则返回的输出将仅包含这些索引的元数据。
        
`blocks`
    显示响应的块部分。
    
`_all`
    显示所有度量。

以下示例仅返回`foo`和`bar`索引的`metadata`和`routing_table`数据：
```sh
GET /_cluster/state/metadata,routing_table/foo,bar
```

下一个示例返回`foo`和`bar`索引的所有内容：
```sh
GET /_cluster/state/_all/foo,bar
```

最后，此示例仅返回`blocks`元数据：
```sh
GET /_cluster/state/blocks
```
