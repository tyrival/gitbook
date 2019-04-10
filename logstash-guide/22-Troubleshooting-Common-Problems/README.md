# 第22章-常见问题

我们正在添加更多疑难解答提示，可以不定期的来查看。如果您有需要添加的内容，请：

- 在 https://github.com/elastic/logstash/issues 上创建一个issue
- 在 https://github.com/elastic/logstash 上创建一个请求，提出您想要的修改

另请参阅 [Logstash论坛](https://discuss.elastic.co/c/logstash)。

### 安装和设置

#### 无法访问临时目录

某些版本的JRuby运行和某些插件的库（例如，TCP输入中的Netty网络库）将可执行文件复制到临时目录。当 `/tmp` 挂载 `noexec` 时，这种情况会导致后续失败。

**错误样例**

```shell
[2018-03-25T12:23:01,149][ERROR][org.logstash.Logstash ]
java.lang.IllegalStateException: org.jruby.exceptions.RaiseException:
(LoadError) Could not load FFI Provider: (NotImplementedError) FFI not
available: java.lang.UnsatisfiedLinkError: /tmp/jffi5534463206038012403.so:
/tmp/jffi5534463206038012403.so: failed to map segment from shared object:
Operation not permitted
```

**可能的解决方案**

- 使用 `exec` 设置挂载 `/tmp`。
- 使用 `jvm.options` 中的 `-Djava.io.tmpdir` 设置备用目录。

### 数据采集

#### 错误响应代码429

`429` 消息表示应用程序正忙于处理其他请求。例如，Elasticsearch发送 `429` 代码以通知Logstash（或其他索引器）批量失败，因为采集队列已满。 Logstash将重试发送文档。

**可能的方案**

检查Elasticsearch以查看是否需要注意。

- https://www.elastic.co/guide/en/elasticsearch/reference/6.7/cluster-stats.html
- https://www.elastic.co/guide/en/elasticsearch/reference/6.7/es-monitoring.html

**错误样例**

```shell
[2018-08-21T20:05:36,111][INFO ][logstash.outputs.elasticsearch] retrying
failed action with response code: 429
({"type"=>"es_rejected_execution_exception", "reason"=>"rejected execution of
org.elasticsearch.transport.TransportService$7@85be457 on
EsThreadPoolExecutor[bulk, queue capacity = 200,
org.elasticsearch.common.util.concurrent.EsThreadPoolExecutor@538c9d8a[Running,
pool size = 16, active threads = 16, queued tasks = 200, completed tasks =
685]]"})
```

### 常规性能优化

有关一般性能调整技巧和指南，请参阅 [第13章-性能调优](../13-Performance-Tuning/README.md)

### 常见的Kafka问题和解决方案

#### Kafka会话超时问题（输入端）

**症状**

吞吐量问题和重复事件处理的Logstash日志警告：

```shell
[2017-10-18T03:37:59,302][WARN][org.apache.kafka.clients.consumer.internals.ConsumerCoordinator]
Auto offset commit failed for group clap_tx1: Commit cannot be completed since
the group has already rebalanced and assigned the partitions to another member.
```

后续调用 `poll()` 之间的时间比配置的 `session.timeout.ms` 长，这通常意味着轮询循环花费了太多时间处理消息。您可以通过增加会话超时，或通过 `max.poll.records` 减少 `poll()` 中返回的批量的最大大小来解决此问题。

```shell
[INFO][org.apache.kafka.clients.consumer.internals.ConsumerCoordinator] Revoking
previously assigned partitions [] for group log-ronline-node09
`[2018-01-29T14:54:06,485][INFO]`[org.apache.kafka.clients.consumer.internals.ConsumerCoordinator]
Setting newly assigned partitions [elk-pmbr-9] for group log-pmbr
```

**背景**

Kafka跟踪消费者组中的各个消费者（例如，许多Logstash实例），并尝试为每个消费者提供他们正在消费的主题中的一个或多个特定数据分区。为了实现这一目标，Kafka会跟踪消费者（Logstash Kafka输入线程）是否在其分配的分区上取得进展，并重新分配在设定的时间范围内没有取得进展的分区。

当Logstash向Kafka请求事件的速度超过他们的处理能力时，会触发分区的重新分配。重新分配分区需要时间，并且可能导致事件的重复处理和明显的吞吐量问题。

**可能的解决方案**

- 在一个请求中减少Logstash从Kafka Broker轮询的每个请求的记录数
- 减少Kafka输入线程的数量
- 增加Kafka Consumer配置中的相关超时

**细节**

`max_poll_records` 选项设置一个请求中要提取的记录数。如果超过默认值500，请尝试减少它。

`consumer_threads` 选项设置输入线程的数量。如果该值超过 `logstash.yml` 文件中配置的管道工作者数，则应该减少它。如果该值大于4，且客户端具有时间/资源，请尝试将其减少到4或更少。尝试从值1开始，然后逐渐递增以找到最佳性能。

`session_timeout_ms` 选项设置相关的超时。将其设置为一个值，以确保可以在时间限制内安全地处理`max_poll_records` 中的事件数。

```shell
EXAMPLE
Pipeline throughput is `10k/s` and `max_poll_records` is set to 1k =>. The value
must be at least 100ms if `consumer_threads` is set to `1`. If it is set to a
higher value `n`, then the minimum session timeout increases proportionally to
`n * 100ms`.
```

在实践中，必须将值设置为远高于理论值，因为管道中的输出和过滤器的行为遵循分布。该值也应高于您期望输出停止的最长时间。默认设置为 `10s == 10000ms`。如果由于负载或类似的影响（例如Elasticsearch输出）而导致输出可能会出现周期性问题，那么可将此值增加至 `60s`。

从性能角度来看，降低 `max_poll_records` 值比增加超时值更可取。如果客户端的问题是由定期停止的输出引起的，那么增加超时是唯一的选择。检查日志以获取停止输出的证据，例如 `ES output logging status 429`。

### 大量的偏移提交（Kafka输入侧）

**症状**

Logstash的Kafka输入导致偏移主题的提交数量远远超过预期。通常涉及冗余的偏移提交，其中重复提交相同的偏移量。

**解决方案**

对于Kafka Broker版本0.10.2.1到1.0.x：问题是由Kafka中的错误引起的。 https://issues.apache.org/jira/browse/KAFKA-6362客户的最佳选择是将Kafka升级到1.1或更高版本。

对于旧版本的Kafka，或者如果版本升级没有完全解决问题：可能因为 `poll_timeout_ms` 的值相对于Kafka接收事件的速率（或者是Kafka在两次事件爆发之间闲置时间）太小。增加 `poll_timeout_ms` 的值会按比例减少此方案中的偏移提交数。例如，将其提高10倍将导致偏移提交减少10倍。

### Kafka输入中的编解码器错误（仅限插件版本6.3.4之前）

**症状**

Logstash的Kafka输入从配置的编解码器中随机记录错误，并（或）错误地读取事件（部分读取，在多个事件之间混合数据等）。

```shell
Log example:  [2018-02-05T13:51:25,773][FATAL][logstash.runner          ] An
unexpected error occurred! {:error=>#<TypeError: can't convert nil into String>,
:backtrace=>["org/jruby/RubyArray.java:1892:in `join'",
"org/jruby/RubyArray.java:1898:in `join'",
"/usr/share/logstash/logstash-core/lib/logstash/util/buftok.rb:87:in `extract'",
"/usr/share/logstash/vendor/bundle/jruby/1.9/gems/logstash-codec-line-3.0.8/lib/logstash/codecs/line.rb:38:in
`decode'",
"/usr/share/logstash/vendor/bundle/jruby/1.9/gems/logstash-input-kafka-5.1.11/lib/logstash/inputs/kafka.rb:241:in
`thread_runner'",
"file:/usr/share/logstash/vendor/jruby/lib/jruby.jar!/jruby/java/java_ext/java.lang.rb:12:in
`each'",
"/usr/share/logstash/vendor/bundle/jruby/1.9/gems/logstash-input-kafka-5.1.11/lib/logstash/inputs/kafka.rb:240:in
`thread_runner'"]}
```

**背景**

在多个线程上运行时（`consumer_threads` 设置为> 1），Kafka Input插件处理编解码器实例的方式存在bug。 https://github.com/logstash-plugins/logstash-input-kafka/issues/210

**解决方案**

- 将Kafka Input插件升级到v.6.3.4或更高版本。
- 如果（且仅当）无法升级，请将 `consumer_threads` 设置为1。
