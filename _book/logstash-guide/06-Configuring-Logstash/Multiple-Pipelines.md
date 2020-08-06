## 多管道

如果想在同一进程中运行多个管道，Logstash提供通过 `pipelines.yml` 配置文件来进行实现。 此文件必须放在 `path.settings` 文件夹中，并遵循以下结构：

```yaml
- pipeline.id: my-pipeline_1
  path.config: "/etc/path/to/p1.config"
  pipeline.workers: 3
- pipeline.id: my-other-pipeline
  path.config: "/etc/different/path/p2.cfg"
  queue.type: persisted
```

此文件以YAML格式定义了一个字典列表，其中每个字典描述一个管道，并通过键值对设置该管道。该示例展示了通过ID和配置路径描述的两个不同管道。其中第一个管道设置 `pipeline.workers` 的值为3，而另一个管道启用了持久化队列功能。未在 `pipelines.yml` 中显式设置的值将使用 [`logstash.yml`](../04-Setting-Up-and-Running-Logstash/logstash.yml.md) 中指定的默认值。

当您无参启动的Logstash时，它将读取 `pipelines.yml` 并实例化其中指定的所有管道。另一方面，当您使用 `-e`或 `-f` 时，Logstash会忽略 `pipelines.yml` 并记录有关它的警告。

### 参考意见

如果当前配置的多个事件流不使用相同的输入、过滤器和输出，并且使用标签和条件相互分离，则使用多个管道尤其有用。

在单个实例中具有多个管道，还允许这些事件流具有不同的性能和持久性参数（例如，设置不同的管道工作者和持久队列）。这种分离意味着一个管道发生输出阻塞时，不会增加另一个管道的压力。

也就是说，考虑管道之间的资源竞争，并为各管道调整默认值，这一点非常重要。例如，考虑减少每个管道使用的管道工作者的数量，因为默认情况下，每个管道将在每个CPU核心中建立一个工作者。

持久化队列和死信队列是按管道隔离的，其位置由 `pipeline.id` 值声明。
