## 转换摄取节点通道

在实现 [摄取](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ingest.html) 管道以解析数据之后，您可能要使用Logstash中更丰富的转换功能。例如，如果要执行以下操作，则可能需要使用Logstash而不是摄取管道：

- 从多个输入摄取。Logstash可以本地从许多其他来源（如TCP，UDP，syslog和关系数据库）中摄取数据。
- 使用多个输出。摄取节点仅支持Elasticsearch作为输出，但您可能希望使用多个输出。例如，您可能希望将传入数据储存到S3以及在Elasticsearch中对其进行索引。
- 利用Logstash中更丰富的转换功能，例如外部查找。
- 在摄取数据时使用持久队列功能来处理峰值（来自Beats和其他来源）。

为了便于您迁移配置，Logstash提供了一个摄取管道转换工具。转换工具将摄取管道定义作为输入，并在可能的情况下创建等效的Logstash配置作为输出。

有关工具限制的完整列表，请参阅 [限制](#限制)。

### 运行工具

您可在Logstash安装的 `bin` 目录中找到转换工具。请参阅 [目录结构](../04-Setting-Up-and-Running-Logstash/Logstash-Directory-Layout.md) 以查找系统上 `bin` 的位置。

要运行转换工具，请使用以下命令：

```shell
bin/ingest-convert.sh --input INPUT_FILE_URI --output OUTPUT_FILE_URI [--append-stdio]
```

此处：

- `INPUT_FILE_URI` 是一个文件URI，它指向一个定义摄取节点管道的JSON文件的完整路径。
- `OUTPUT_FILE_URI` 是将由该工具生成的Logstash DSL文件的文件URI。
- `--append-stdio` 是一个可选参数，它将stdin和stdout部分添加到配置中，而不是添加默认的Elasticsearch输出。

此命令需要文件URI，因此请确保使用正斜杠并指定文件的完整路径。

例如：

```shell
bin/ingest-convert.sh --input file:///tmp/ingest/apache.json --output file:///tmp/ingest/apache.conf
```

### 限制

- 不支持无痛脚本转换。
- 转换仅可使用 [支持的处理器](#支持的处理器) 的子集。对于不支持的处理器，该工具会生成警告并继续尽可能进行转换。

### 支持的处理器

此工具目前支持以下摄取节点处理器进行转换：

- Append
- Convert
- Date
- GeoIP
- Grok
- Gsub
- Json
- Lowercase
- Rename
- Set