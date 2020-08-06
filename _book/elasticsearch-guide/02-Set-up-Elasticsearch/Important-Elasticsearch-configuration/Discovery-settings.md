## 发现设置

Elasticsearch使用名为“Zen Discovery”的自定义发现，实现进行集群的节点连接和主选举。在投入生产之前，应该配置两个重要的发现设置。

#### `discovery.zen.ping.unicast.hosts`

开箱即用，没有任何网络配置，Elasticsearch将绑定到可用的环回地址，并将扫描端口9300到9305，以尝试连接到在同一服务器上运行的其他节点，自动构建集群，无需进行任何配置。

当需要在其他服务器上形成具有节点的集群时，您必须提供集群中可能是实时且可联系的其他节点的种子列表。这可以指定如下：

```yaml
discovery.zen.ping.unicast.hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 ①
   - seeds.mydomain.com ②
```


① 端口将默认为`transport.profiles.default.port`，如果未指定，将使用`transport.port`。

② 解析为多个IP地址的主机名将尝试所有已解析的地址。

#### `discovery.zen.minimum_master_nodes`

为防止数据丢失，必须配置`discovery.zen.minimum_master_nodes`设置，以便每个合格主节点都知道为了形成集群必须的最小主节点数。

如果没有此设置，集群遭受网络故障时，可能会拆分为两个独立的集群——脑裂，这将导致数据丢失。在 [使用`minium_master_nodes`避免脑裂](../../14-Modules/Node.md#使用miniummasternodes避免脑裂) 提供了更详细的解释。

为避免脑裂，应将此设置为合法数量的合格节点：

```
(master_eligible_nodes / 2) + 1
```

换句话说，如果有三个符合主节点的节点，则最小主节点应设置为`(3 / 2) + 1`或`2`：

```yaml
discovery.zen.minimum_master_nodes: 2
```
