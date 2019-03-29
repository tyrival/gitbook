# 第3章 工作原理

Logstash事件处理管道有三个阶段：输入→过滤器→输出。输入生成事件，过滤器修改它们，输出将它们发送到目的地。输入和输出支持编解码器，使你能够在数据进入或退出管道时对数据进行编码或解码，而无需使用单独的过滤器。

### 输入插件
您使用输入插件将数据导入Logstash。常用的输入插件包括：

- **file**：从文件系统上读取文件，类似于UNIX命令 `tail -0F`

- **syslog**：在514端口上侦听syslog消息，并根据RFC3164格式进行解析

- **redis**：使用redis通道和redis列表，从redis服务器读取数据。 Redis通常用作集中式Logstash中的“代理”，使Logstash事件在远程Logstash“发运者”中形成队列。

- **beats**：处理Beats发送的事件。

有关输入插件的详细信息，请参阅 [第17章 输入插件](https://github.com/tyrival/logstash-guide/blob/master/chapters/17-Input-plugins.md)。

### 过滤器
过滤器是Logstash管道中的中间处理设备。您可以按需组合过滤器，以便在事件满足特定条件时对其执行操作。常见过滤器包括：

- **grok**：解析和构造文本。 目前为止，Grok是Logstash中将非结构化日志数据解析为结构化和可查询内容的最佳方式。它内置的120种模式能够很大程度上解决你的需求！
- **mutate**：对事件的字段执行常规转换。可对事件中的字段进行重命名、删除、替换和修改。
- **drop**：完全删除事件，例如调试事件。
- **clone**：制作事件的副本，可能添加或删除字段。
- **geoip**：添加有关IP地址的地理位置的信息（在Kibana中可以显示惊艳的图表效果！）

有关过滤器的详细信息，请参阅 [第19章 过滤器Filter](https://github.com/tyrival/logstash-guide/blob/master/chapters/19-FIlter-plugins.md)。

### 输出插件
输出插件是Logstash管道的最后阶段。同一事件可以走多个输出插件，但是当所有输出插件处理完成，事件就会彻底结束执行过程。常用的输出插件包括：

- **elasticsearch**：将事件数据发送到Elasticsearch。当你打算以高效、方便且易于查询的格式保存数据， Elasticsearch是您的最佳选择……对，我们护犊子 :)
- **file**：将事件数据写入磁盘上的文件。
- **graphite**：将事件数据发送到graphite——一种流行的用于存储和绘制指标的开源工具。 http://graphite.readthedocs.io/en/latest/
- **statsd**：将事件数据发送到statsd，这是一种“监听统计信息，例如计数器和定时器，通过UDP将聚合发送给一个或多个可插入后端服务”的服务。如果您已经在使用statsd，这可能对您有用！

有关输出插件的更多信息，请参阅 [第18章 输出插件Output plugins](https://github.com/tyrival/logstash-guide/blob/master/chapters/18-Output-plugins.md)。

### 编解码器

编解码器基本上是流过滤器，可以作为输入插件或输出插件的一部分。使用编解码器可以轻松地将消息传输与序列化过程分开。流行的编解码器包括json，msgpack和plain（text）。

- **json**：以JSON格式编码或解码数据。
- **multiline**：将多行文本事件（如java异常和堆栈跟踪消息）合并到一个事件中。

有关编解码器的更多信息，请参阅 [第20章 编解码器Codec](https://github.com/tyrival/logstash-guide/blob/master/chapters/20-Coder-plugins.md)。



## 运行模型

Logstash事件处理管道协调输入、过滤器和输出的执行。

Logstash管道中的每个输入都在其自己的线程中运行。将事件写入位于内存（默认）或磁盘上的中央队列。每个管道工作线程从该队列中获取一批事件，使其通过配置的过滤器，然后将过滤后的事件传递给所有输出插件。事件批的大小和管道工作线程的数量是可配置的（请参阅 [性能调试和查看](https://github.com/tyrival/logstash-guide/blob/master/chapters/13-Performance-Tuning.md#性能调试和查看)）。

默认情况下，Logstash在管道阶段（输入→若干过滤器→输出）之间使用内存中的有界队列来缓冲事件。如果Logstash不安全地终止，则存储在内存中的事件都将丢失。为防止数据丢失，您可以启用Logstash将正在进行的事件持久化保存到磁盘。 有关更多信息，请参阅 [持久化队列](https://github.com/tyrival/logstash-guide/blob/master/chapters/10-Data-Resiliency.md#持久化队列)。
