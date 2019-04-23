## 远程集群信息
集群远程信息API允许检索所有已配置的远程集群信息。
```sh
GET /_remote/info
```

此命令返回由配置的远程集群别名键入的连接和端点信息。

`seeds`
    配置的远程集群的初始种子传输地址。

`http_addresses`
    已发布的所有已连接远程节点的http地址。

`connected`
    如果至少有一个到远程集群的连接，则为True。

`num_nodes_connected`
    远程集群中已连接节点的数量。

`max_connections_per_cluster`
    为远程集群维护的最大连接数。

`initial_connect_timeout`
    远程集群连接的初始连接超时。

`skip_unavailable`
    是否通过跨集群搜索请求搜索远程集群但是没有其节点可用，是否跳过远程集群。
