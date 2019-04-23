## 节点分配说明API
集群分配说明API的目的是为集群中的分片分配提供解释。对于未分配的分片，API提供了分片未分配原因的说明。对于已分配的分片，API提供了有关分片在其当前节点上保留，但未移动或重新平衡到另一个节点的原因的说明。在尝试诊断碎片未分配的原因，或者为什么碎片会继续保留在当前节点上时，此API非常有用。

### 解释API请求
要解释分片的分配，首先应该存在一个索引：

```sh
PUT /myindex
```

然后可以解释该索引的分片分配：

```sh
GET /_cluster/allocation/explain
{
  "index": "myindex",
  "shard": 0,
  "primary": true
}
```

指定要解释的分片的`index`和`shard`的ID，以及指示是否解释给定分片ID，或其副本分片之一的主分片的`primary`标志。这三个参数是必需的。

您还可以指定可选的`current_node`请求参数，以仅解释当前位于`current_node`上的分片。 `current_node`可以指定为节点标识或节点名称。

```sh
GET /_cluster/allocation/explain
{
  "index": "myindex",
  "shard": 0,
  "primary": false,
  "current_node": "nodeA"  ①
}
```

![](../../elasticsearch-guide/source/images/common/1.png) 分片0当前具有副本的节点

您还可以让Elasticsearch发送body为空的请求，来解释它找到的第一个未分配的分片：

```sh
GET /_cluster/allocation/explain
```

### 说明API响应
本节包括各种方案下的集群分配说明API响应输出的示例。

未分配的分片的API响应：
```json
{
  "index" : "idx",
  "shard" : 0,
  "primary" : true,
  "current_state" : "unassigned",                 ①
  "unassigned_info" : {
    "reason" : "INDEX_CREATED",                   ②
    "at" : "2017-01-04T18:08:16.600Z",
    "last_allocation_status" : "no"
  },
  "can_allocate" : "no",                          ③
  "allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes",
  "node_allocation_decisions" : [
    {
      "node_id" : "8qt2rY-pT6KNZB3-hGfLnw",
      "node_name" : "node-0",
      "transport_address" : "127.0.0.1:9401",
      "node_attributes" : {},
      "node_decision" : "no",                     ④
      "weight_ranking" : 1,
      "deciders" : [
        {
          "decider" : "filter",                   ⑤
          "decision" : "NO",
          "explanation" : "node does not match index setting [index.routing.allocation.include] filters [_name:\"non_existent_node\"]"  ⑥
        }
      ]
    }
  ]
}
```

![](../../elasticsearch-guide/source/images/common/1.png) 碎片的当前状态
![](../../elasticsearch-guide/source/images/common/2.png) 碎片最初未分配的原因
![](../../elasticsearch-guide/source/images/common/3.png) 是否分配分片
![](../../elasticsearch-guide/source/images/common/4.png) 是否将分片分配给特定节点
![](../../elasticsearch-guide/source/images/common/5.png) 导致无法决定节点的决策者
![](../../elasticsearch-guide/source/images/common/6.png) 解释为什么决策者返回一个没有决定，有一个有用的提示指向导致决定的设置

通过将`include_disk_info`参数设置为`true`，可以返回集群信息服务收集的磁盘使用情况和分片大小的信息：

```sh
GET /_cluster/allocation/explain?include_disk_info=true
```

此外，如果您希望将所有决策都包含在最终决策中，则`include_yes_decisions`参数将返回每个节点的所有决策：

```sh
GET /_cluster/allocation/explain?include_yes_decisions=true
```

`include_yes_decisions`的默认值为`false`，仅包含响应中的`no`决策。这通常是您想要的，因为没有决定表明为什么分片未分配或无法移动，并且包括所有决策包括是的，这为API的响应输出添加了大量冗余。

先前已分配给集群中节点的未分配主分片的API响应输出：

```json
{
  "index" : "idx",
  "shard" : 0,
  "primary" : true,
  "current_state" : "unassigned",
  "unassigned_info" : {
    "reason" : "NODE_LEFT",
    "at" : "2017-01-04T18:03:28.464Z",
    "details" : "node_left[OIWe8UhhThCK0V5XfmdrmQ]",
    "last_allocation_status" : "no_valid_shard_copy"
  },
  "can_allocate" : "no_valid_shard_copy",
  "allocate_explanation" : "cannot allocate because a previous copy of the primary shard existed but can no longer be found on the nodes in the cluster"
}
```

由于延迟分配而未分配的副本的API响应输出：
```json
{
  "index" : "idx",
  "shard" : 0,
  "primary" : false,
  "current_state" : "unassigned",
  "unassigned_info" : {
    "reason" : "NODE_LEFT",
    "at" : "2017-01-04T18:53:59.498Z",
    "details" : "node_left[G92ZwuuaRY-9n8_tc-IzEg]",
    "last_allocation_status" : "no_attempt"
  },
  "can_allocate" : "allocation_delayed",
  "allocate_explanation" : "cannot allocate because the cluster is still waiting 59.8s for the departed node holding a replica to rejoin, despite being allowed to allocate the shard to at least one other node",
  "configured_delay" : "1m",                      ①
  "configured_delay_in_millis" : 60000,
  "remaining_delay" : "59.8s",                    ②
  "remaining_delay_in_millis" : 59824,
  "node_allocation_decisions" : [
    {
      "node_id" : "pmnHu_ooQWCPEFobZGbpWw",
      "node_name" : "node_t2",
      "transport_address" : "127.0.0.1:9402",
      "node_decision" : "yes"
    },
    {
      "node_id" : "3sULLVJrRneSg0EfBB-2Ew",
      "node_name" : "node_t0",
      "transport_address" : "127.0.0.1:9400",
      "node_decision" : "no",
      "store" : {                                 ③
        "matching_size" : "4.2kb",
        "matching_size_in_bytes" : 4325
      },
      "deciders" : [
        {
          "decider" : "same_shard",
          "decision" : "NO",
          "explanation" : "the shard cannot be allocated to the same node on which a copy of the shard already exists [[idx][0], node[3sULLVJrRneSg0EfBB-2Ew], [P], s[STARTED], a[id=eV9P8BN1QPqRc3B4PLx6cg]]"
        }
      ]
    }
  ]
}
```

![](../../elasticsearch-guide/source/images/common/1.png) 在分配由于节点将其保留离开集群而不存在的副本分片之前配置的延迟
![](../../elasticsearch-guide/source/images/common/2.png) 分配副本分片之前的剩余延迟
![](../../elasticsearch-guide/source/images/common/3.png) 有关节点上找到的分片数据的信息

分配的分片的API响应输出，不允许保留在其当前节点上并且需要移动：
```json
{
  "index" : "idx",
  "shard" : 0,
  "primary" : true,
  "current_state" : "started",
  "current_node" : {
    "id" : "8lWJeJ7tSoui0bxrwuNhTA",
    "name" : "node_t1",
    "transport_address" : "127.0.0.1:9401"
  },
  "can_remain_on_current_node" : "no",            ①
  "can_remain_decisions" : [                      ②
    {
      "decider" : "filter",
      "decision" : "NO",
      "explanation" : "node does not match index setting [index.routing.allocation.include] filters [_name:\"non_existent_node\"]"
    }
  ],
  "can_move_to_other_node" : "no",                ③
  "move_explanation" : "cannot move shard to another node, even though it is not allowed to remain on its current node",
  "node_allocation_decisions" : [
    {
      "node_id" : "_P8olZS8Twax9u6ioN-GGA",
      "node_name" : "node_t0",
      "transport_address" : "127.0.0.1:9400",
      "node_decision" : "no",
      "weight_ranking" : 1,
      "deciders" : [
        {
          "decider" : "filter",
          "decision" : "NO",
          "explanation" : "node does not match index setting [index.routing.allocation.include] filters [_name:\"non_existent_node\"]"
        }
      ]
    }
  ]
}
```

![](../../elasticsearch-guide/source/images/common/1.png) 是否允许分片保留在其当前节点上
![](../../elasticsearch-guide/source/images/common/2.png) 决策者决定不允许将碎片保留在其当前节点上
![](../../elasticsearch-guide/source/images/common/3.png) 是否允许将分片分配给另一个节点

以下是将分片移动到另一个节点的输出，显示其仍保留在其当前节点上，因为集群找不到更好的平衡办法：
```json
{
  "index" : "idx",
  "shard" : 0,
  "primary" : true,
  "current_state" : "started",
  "current_node" : {
    "id" : "wLzJm4N4RymDkBYxwWoJsg",
    "name" : "node_t0",
    "transport_address" : "127.0.0.1:9400",
    "weight_ranking" : 1
  },
  "can_remain_on_current_node" : "yes",
  "can_rebalance_cluster" : "yes",                ①
  "can_rebalance_to_other_node" : "no",           ②
  "rebalance_explanation" : "cannot rebalance as no target node exists that can both allocate this shard and improve the cluster balance",
  "node_allocation_decisions" : [
    {
      "node_id" : "oE3EGFc8QN-Tdi5FFEprIA",
      "node_name" : "node_t1",
      "transport_address" : "127.0.0.1:9401",
      "node_decision" : "worse_balance",          ③
      "weight_ranking" : 1
    }
  ]
}
```

![](../../elasticsearch-guide/source/images/common/1.png) 是否允许在集群上进行重新平衡
![](../../elasticsearch-guide/source/images/common/2.png) 是否可以将分片重新平衡到另一个节点
![](../../elasticsearch-guide/source/images/common/3.png) 分片无法重新平衡到节点的原因，在这种情况下表明它没有提供比当前节点更好的平衡
