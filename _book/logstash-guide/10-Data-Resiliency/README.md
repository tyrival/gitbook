# 第10章-数据弹性

当数据流经事件处理管道时，可能无法向其预定义的目的地输出事件。 例如，数据可能包含错误的数据类型，或者Logstash可能异常终止。

为防止数据丢失，并确保事件不断地通过管道，Logstash提供以下数据弹性功能。

- [持久化队列](../10-Data-Resiliency/Persistent-Queues.md) 将事件以内部队列的形式存储在磁盘上，从而防止数据丢失。
- [死信队列](../10-Data-Resiliency/Dead-Letter-Queues.md) 为Logstash无法处理的事件提供磁盘存储。 您可以使用 `dead_letter_queue` 输入插件重新处理死信队列中的事件。

默认情况下禁用这些数据弹性功能。 要启用这些功能，您必须在 [logstash.yml](../04-Setting-Up-and-Running-Logstash/logstash.yml.md) 中显式声明。
