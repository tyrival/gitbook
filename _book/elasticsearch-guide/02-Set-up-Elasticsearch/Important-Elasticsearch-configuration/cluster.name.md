## 集群名称

节点与所有其他节点的`cluster.name`名称相同时，才会加入同一集群。 默认名称为`elasticsearch`，但您应将其更改为适当的名称，该名称一般用于描述集群的用途。

```yaml
cluster.name: logging-prod
```

确保不同的集群要用不同的集群名称，否则最终会导致节点加入错误的集群。
