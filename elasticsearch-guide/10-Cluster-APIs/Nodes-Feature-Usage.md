## 节点特征用法
集群节点使用API​​允许检索有关每个节点的功能使用的信息。
```sh
GET _nodes/usage
GET _nodes/nodeId1,nodeId2/usage
```

第一个命令检索集群中所有节点的使用情况。第二个命令选择性地仅检索`nodeId1`和`nodeId2`的节点使用情况。[这里](../10-Cluster-APIs/README.md)解释了所有节点选项。

#### REST操作用法信息
响应中的`rest_actions`字段包含REST操作类名的映射，其中包含在节点上调用操作的次数：

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "cluster_name": "my_cluster",
  "nodes": {
    "pQHNt5rXTTWNvUgOrdynKg": {
      "timestamp": 1492553961812, ①
      "since": 1492553906606, ②
      "rest_actions": {
        "org.elasticsearch.rest.action.admin.cluster.RestNodesUsageAction": 1,
        "org.elasticsearch.rest.action.admin.indices.RestCreateIndexAction": 1,
        "org.elasticsearch.rest.action.document.RestGetAction": 1,
        "org.elasticsearch.rest.action.search.RestSearchAction": 19, 
        "org.elasticsearch.rest.action.admin.cluster.RestNodesInfoAction": 36
      }
    }
  }
}
```

![](../../elasticsearch-guide/source/images/common/1.png) 执行此节点使用请求的时间戳。
![](../../elasticsearch-guide/source/images/common/2.png) 用于开始使用情况信息记录的时间戳。这相当于节点启动的时间。
![](../../elasticsearch-guide/source/images/common/3.png) 此节点已调用搜索操作19次。
