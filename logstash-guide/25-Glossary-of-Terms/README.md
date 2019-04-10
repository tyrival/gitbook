# 第25章-术语

**@metadata - 元数据**

用于存储您不希望包含在输出 [事件](#event - 事件) 的特殊字段。例如，`@metadata` 字段对于创建用于 [条件](#conditional - 条件) 语句的临时字段很有用。

**codec plugin - 编解码器插件**

Logstash [插件](#plugin - 插件)，用于更改 [事件](#event - 事件) 的数据表示形式。编解码器本质上是流数据过滤器，可以作为输入或输出的一部分。使用编解码器可以将消息传输与序列化过程分开。流行的编解码器包括json，msgpack和 plain (text)。

**conditional - 条件**

基于语句（也称为条件）是真还是假来执行某些操作的控制流。 Logstash支持 `if`，`else if` 和 `else` 语句。您可以使用条件语句来应用过滤器，并根据您指定的条件将事件发送到特定输出。

**event - 事件**

单个信息单元，包含时间戳和附加数据。事件通过输入到达，随后被解析，加时间戳并通过Logstash [管道](#pipeline - 管道) 传递。

**field - 字段**

[事件](#event - 事件) 的属性。例如，apache访问日志中的每个事件都有属性，例如状态代码（200,404），请求路径（"/"，"index.html"），HTTP方法（GET，POST），客户端IP地址等等。 Logstash使用术语"字段"来引用这些属性。

**field reference - 字段引用**

对事件 [字段](#field - 字段) 的引用。此引用可能出现在Logstash配置文件的输出块或过滤器块中。字段引用通常包含在方括号（`[]`）括号中，例如`[fieldname]`。如果您指的是顶级字段，则可以省略 `[]` 并只使用字段名称。要引用嵌套字段，请指定该字段的完整路径：`[顶级字段] [嵌套字段]`。

**filter - 过滤插件**

Logstash [插件](#plugin - 插件)，用于对 [事件](#event - 事件) 执行中间处理。通常，过滤器在通过输入摄取事件数据之后，根据配置规则改变、丰富，并且（或者）修改事件数据。过滤器通常根据事件的特征有条件地应用。流行的过滤插件包括grok，mutate，drop，clone和geoip。过滤阶段是可选的。

**gem**

一个独立的代码包，托管在 [RubyGems.org](https://rubygems.org) 上。 Logstash [插件](#plugin - 插件) 打包为Ruby Gems。您可以使用Logstash [插件管理器](#plugin manager - 插件管理器) 来管理Logstash gem。

**hot thread - 热线程**

具有高CPU使用率，且执行时间超常的Java线程。

**input plugin - 输入插件**

一个Logstash [插件](#plugin - 插件)，用于读取特定源的 [事件](#event - 事件) 数据。输入插件是Logstash事件处理 [管道](#pipeline - 管道) 中的第一个阶段。流行的输入插件包括file，syslog，redis和beats。

**indexer - 索引器**

一个Logstash实例，其任务是与Elasticsearch集群连接以索引 [事件](#event - 事件) 数据。

**message broker - 消息中间件**

消息中间件也称为消息缓冲区或消息队列，是外部软件（如Redis，Kafka或RabbitMQ），它将来自Logstash [发运者](#shipper - 发运者) 实例的消息存储为中间存储，等待Logstash索引器实例处理。

**output plugin - 输出插件**

Logstash [插件](#plugin - 插件)，用于将 [事件](#event - 事件) 数据写入特定目标。输出是事件 [管道](#pipeline - 管道) 的最后阶段。流行的输出插件包括elasticsearch，file，graphite和statsd。

**pipeline - 管道**

用于通过Logstash工作流描述 [事件](#event - 事件) 流的术语。管道通常由一系列输入、过滤和输出阶段组成。[输入](#input plugin - 输入插件) 阶段从源获取数据并生成事件，[过滤](#filter - 过滤插件) 阶段（可选）修改事件数据，[输出](#output plugin - 输出插件) 阶段将数据写入目标。输入和输出支持 [编解码器](#codec plugin - 编解码器插件)，使您能够在数据进入或退出管道时，对数据进行编码或解码，而无需使用单独的过滤器。

**plugin - 插件**

一个独立的软件包，用于实现Logstash事件处理 [管道](#pipeline - 管道) 中的一个阶段。 可用插件列表包括 [输入插件](#input plugin - 输入插件)，[输出插件](#output plugin - 输出插件)，[编解码器插件](#codec plugin - 编解码器插件) 和 [过滤器插件](#filter - 过滤插件)。这些插件以Ruby gems的形式实现，并托管在RubyGems.org上。 您可以通过配置插件来定义事件处理 [管道](#pipeline - 管道) 的各个阶段。

**plugin manager - 插件管理器**

通过 `bin/logstash-plugin` 脚本访问，插件管理器使您可以管理Logstash [插件](#plugin - 插件)的生命周期。 您可以使用插件管理器命令行界面（CLI）安装、删除和升级插件。

**shipper - 发运者**

Logstash的一个实例，它将事件发送到另一个Logstash实例或其他应用程序。

**worker - 工作者**

Logstash使用的过滤器线程模型，每个工作者按顺序接收 [事件](#event - 事件)，并应用所有过滤器，然后将事件发送到输出队列。 这允许跨CPU的可扩展性，因为许多过滤器是CPU密集型的。