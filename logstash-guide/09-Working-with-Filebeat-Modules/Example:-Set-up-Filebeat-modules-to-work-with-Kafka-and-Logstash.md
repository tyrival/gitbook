## 示例：安装Filebeat并连接Kafka和Logstash

本节介绍当你在Filebeat和Logstash之间使用Kafka时，安装 [Filebeat](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-modules-overview.html) 的方法。此示例的主要目标是展示如何从Filebeat加载摄取管道并将其与Logstash一起使用。

本节中的示例显示了topic名称为硬编码的简单配置。有关配置选项的完整列表，请参阅有关配置 [Kafka输入插件](../17-Input-plugins/kafka.md) 文档。另请参阅在Filebeat Reference中的 [配置Kafka输出](https://www.elastic.co/guide/en/beats/filebeat/6.7/kafka-output.html)。

### 设置并运行Filebeat

1. 如果尚未设置Filebeat索引模板和示例Kibana仪表板，请运行Filebeat `setup` 命令立即执行此操作：

```shell
filebeat -e setup
```

`-e` 标志是可选的，并将输出发送到标准错误而不是syslog。

此步骤为一次性设置，需要连接到Elasticsearch和Kibana，因为Filebeat需要在Elasticsearch中创建索引模板并将示例仪表板加载到Kibana中。有关配置与Elasticsearch的连接的详细信息，请参阅Filebeat模块 [快速入门](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-modules-quickstart.html)。

加载模板和仪表板后，您将看到成功加载的消息`INFO {kib} dashboards successfully loaded. Loaded dashboards`。

2. 运行 `modules enable` 命令以启用要运行的模块。例如：

```shell
filebeat modules enable system
```

您可以通过编辑Filebeat `modules.d` 目录下的配置文件来进一步配置模块。例如，如果日志文件不在模块预期的位置，则可以设置 `var.paths` 选项。

运行 `setup` 命令，并指定 `--pipelines`和 `--modules`选项，以加载您已启用的模块的接收管道。此步骤还需要连接到Elasticsearch。如果要使用Logstash管道而不是摄取节点来解析数据，请跳过此步骤。

```shell
filebeat setup --pipelines --modules system
```

4. 配置Filebeat以将日志行发送到Kafka。为此，在 `filebeat.yml` 配置文件中，通过注释掉它来禁用Elasticsearch输出，并启用Kafka输出。例如：

```yaml
#output.elasticsearch:
  #hosts: ["localhost:9200"]
output.kafka:
  hosts: ["kafka:9092"]
  topic: "filebeat"
  codec.json:
    pretty: false
```

5. 启动Filebeat。例如：

```shell
filebeat -e
```

Filebeat将尝试向Logstash发送消息并继续，直到Logstash可用于接收它们。

> **注意：**
> 根据您安装Filebeat的方式，当您尝试运行Filebeat模块时，可能会看到与文件所有权或权限相关的错误。如果遇到与文件所有权或权限相关的错误，请参阅Beats Platform Reference中的 [配置文件所有权和权限](https://www.elastic.co/guide/en/beats/libbeat/6.7/config-file-permissions.html)。

### 创建并启动Logstash管道

1. 在安装Logstash的系统上，创建一个Logstash管道配置，该配置从Kafka输入读取并将事件发送到Elasticsearch输出：

```yaml
input {
  kafka {
    bootstrap_servers => "myhost:9092"
    topics => ["filebeat"]
    codec => json
  }
}

output {
  if [@metadata][pipeline] {
    elasticsearch {
      hosts => "https://myEShost:9200"
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
      pipeline => "%{[@metadata][pipeline]}" 
      user => "elastic"
      password => "secret"
    }
  } else {
    elasticsearch {
      hosts => "https://myEShost:9200"
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
      user => "elastic"
      password => "secret"
    }
  }
}
```


将 `pipeline` 选项设置为 `％{[@ metadata] [pipeline]}`。此设置将Logstash配置为根据事件中传递的元数据选择正确的摄取管道。

如果要使用Logstash管道而不是接收节点来解析数据，请参阅 [使用Logstash管道解析](../09-Working-with-Filebeat-Modules/Use-Logstash-pipelines-for-parsing.md) 下的示例中的 `filter` 和 `output` 设置进行解析。

2. 启动Logstash，传入刚刚定义的管道配置文件。例如：

```shell
bin/logstash -f mypipeline.conf
```

Logstash应该启动管道并开始从Kafka输入接收事件。

### 可视化数据

要显示Kibana中的数据，请通过将浏览器指向端口5601来启动Kibana Web界面。例如 http://127.0.0.1:5601。单击 **Dashboard**，然后查看Filebeat仪表板。
