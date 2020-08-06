## 节点名称

默认情况下，Elasticsearch将使用随机生成的UUID的前七个字符作为节点ID。 请注意，节点ID是持久的，并且在节点重新启动时不会更改，因此默认节点名称也不会更改。

最好是配置一个更有意义的名称，它还具有在重新启动节点后保持不变的优点：

```yaml
node.name: prod-data-2
```

`node.name`也可以设置为服务器的HOSTNAME，如下所示：

```yaml
node.name: ${HOSTNAME}
```

