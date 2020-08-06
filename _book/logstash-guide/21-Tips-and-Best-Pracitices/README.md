# 第21章-技巧和最佳实践

我们正在添加更多提示和最佳做法，请稍后再回来查看。如果您有需要添加的内容，请：

- 在 https://github.com/elastic/logstash/issues 上创建一个issue
- 在 https://github.com/elastic/logstash 上创建一个请求，提出您想要的修改

另请参阅 [Logstash论坛](https://discuss.elastic.co/c/logstash)。

### 命令行

#### Windows上的Shell命令

命令行示例通常显示单引号。在Windows系统上，用双引号替换单引号。

**例**

Linux上的命令为：

```shell
bin/logstash -e 'input { stdin { } } output { stdout {} }'
```


Windows系统上使用此格式：

```shell
bin\logstash -e "input { stdin { } } output { stdout {} }"
```

### 管道
#### 管道管理

您可以使用Kibana中的本地管道配置，或 [配置集中管理管道](../06-Configuring-Logstash/Configuring-Centralized-Pipeline-Management.md) 来管理Logstash实例中的管道。

将Logstash配置为使用集中管道管理后，您将无法再指定本地管道配置。启用集中管道管理时，`pipelines.yml` 文件和诸如 `path.config` 和 `config.string` 之类的设置处于非活动状态。

### Kafka

#### Kafka设置

##### 主题的分区

"每个主题我应该使用多少个分区？"

至少是Logstash节点的数量乘以每个节点的消费者线程。

最好是使用上述数字的倍数。增加现有主题的分区数量非常复杂，分区的开销非常低，建议是分区数量的5到10倍，只要整个分区计数不超过2000。

超过1000个分区的过度分区是错误的。尽量不要超过1000个分区。

##### 消费者线程

"我应该配置多少个消费者线程？"

较低的值往往更有效，并且具有更少的内存开销。尝试值1然后不断上升。该值通常应低于管道工作者的数量。大于4的值很少会导致性能提升。

#### Kafka输入和持久队列（PQ）编辑

##### Kafka偏移量提交

"只有在事件被安全地保存到PQ之后，Kafka Input才会提交偏移吗？"

"Kafa Input是否仅为已完全通过管道的事件提交偏移量？"

不，我们不能保证。偏移量定期提交给Kafka。如果对PQ的写入速度很慢或被阻止，则可以提交未安全保存到PQ的事件的偏移量。
