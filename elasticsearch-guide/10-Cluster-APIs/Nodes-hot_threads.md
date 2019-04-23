## 节点hot_threads
此API会生成集群中每个选定节点上的热线程细分。 它的端点是`/_nodes/hot_threads`和 `/_nodes/{nodes}/hot_threads`：

```sh
GET /_nodes/hot_threads
GET /_nodes/nodeId1,nodeId2/hot_threads
```

第一个命令获取集群中所有节点的热线程。 第二个命令只获取nodeId1和nodeId2的热线程。 可以使用[节点过滤器](../10-Cluster-APIs/README.md#指定节点)选择节点。

输出是纯文本，每个节点的热线程都有细分。 允许的参数是：

| 参数                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| `threads`             | 要提供的热线程数，默认为3。                                  |
| `interval`            | 进行第二次线程采样的间隔。 默认为500毫秒                     |
| `type`                | 要采样的类型，默认为cpu，但支持wait和block来查看处于等待或阻塞状态的热线程。 |
| `ignore_idle_threads` | 如果为真，则过滤掉已知的空闲线程（例如，在套接字选择中等待，或从空队列中获取任务）。 默认为true。 |


