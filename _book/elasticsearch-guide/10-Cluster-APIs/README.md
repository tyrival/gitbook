# 集群API
### 指定节点
某些集群级API可以在节点上运行，这些节点可以使用节点过滤器指定。例如，[任务管理](../10-Cluster-APIs/Task-Management-API.md)，[节点统计信息](../10-Cluster-APIs/Nodes-Stats.md)和[节点信息](../10-Cluster-APIs/Nodes-Info.md)API都可以反馈过滤后的节点集，而不是所有节点的结果。

节点过滤器是一个以逗号分隔的过滤器列表，每个过滤器都会从所选子集中添加或删除节点。过滤器可以是以下之一：

- `_all`，将所有节点添加到子集。
- `_local`，将本地节点添加到子集。
- `_master`，将当前选举的主节点添加到子集。
- 节点ID或名称，将此节点添加到子集。
- IP地址或主机名，用于将所有匹配的节点添加到子集。
- 匹配，使用`*`通配符，将名称、地址或主机名匹配的所有节点添加到子集中。
- `master:true`，`data:true`，`ingest:true`或`coordinating_only:true`，分别将所有主节点的子节点，所有数据节点，所有摄取节点和所有仅协调节点添加到子集。
- `master:false`，`data:false`，`ingest:false`或`coordinating_only:false`，分别从子集中删除所有主节点的节点，所有数据节点，所有摄取节点和所有仅协调节点。
- 使用`*`通配符形式为`attrname:attrvalue`，它使用自定义节点属性向所有节点添加其名称和值匹配的子集。通过在配置文件中设置`node.attr.attrname:attrvalue`属性来匹配自定义节点属性。

>**注意**
>节点过滤器按照给定的顺序运行，如果使用从集合中删除节点的过滤器，这一点很重要。例如`_all`，`master:false`表示除了主节点之外的所有节点，但是`master:false`，`_ all`表示与`_all`相同，因为`_all`过滤器在`master:false`过滤器之后运行。

>**注意**
>如果没有给出过滤器，则默认为选择所有节点。但是，如果给出任何过滤器，则它们以空的选定子集开始运行。这意味着从所选子集中删除节点的过滤器（例如`master:false`）仅在它们位于其他过滤器之后才有用。单独使用时，`master:false`选择无节点。

以下是使用节点过滤器和[节点信息API](../10-Cluster-APIs/Nodes-Info.md)的一些示例。

```sh
# If no filters are given, the default is to select all nodes
GET /_nodes
# Explicitly select all nodes
GET /_nodes/_all
# Select just the local node
GET /_nodes/_local
# Select the elected master node
GET /_nodes/_master
# Select nodes by name, which can include wildcards
GET /_nodes/node_name_goes_here
GET /_nodes/node_name_goes_*
# Select nodes by address, which can include wildcards
GET /_nodes/10.0.0.3,10.0.0.4
GET /_nodes/10.0.0.*
# Select nodes by role
GET /_nodes/_all,master:false
GET /_nodes/data:true,ingest:true
GET /_nodes/coordinating_only:true
# Select nodes by custom attribute (e.g. with something like `node.attr.rack: 2` in the configuration file)
GET /_nodes/rack:2
GET /_nodes/ra*:2
GET /_nodes/ra*:2*
```
