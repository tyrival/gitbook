## 使用采集管道解析

将Filebeat模块与Logstash一起使用时，可以使用Filebeat提供的采集管道来解析数据。您需要将管道加载到Elasticsearch并配置Logstash以使用它们。

**加载采集管道：**

在安装了Filebeat的系统上，运行 `setup` 命令并指定 `--pipelines` 选项以加载特定模块的接收管道。例如，以下命令加载系统和nginx模块的采集管道：

```shell
filebeat setup --pipelines --modules nginx,system
```

此安装步骤需要连接到Elasticsearch，因为Filebeat需要将采集管道加载到Elasticsearch中。如有必要，可以在运行命令之前临时禁用已配置的输出并启用Elasticsearch输出。

**配置Logstash使用管道：**

在安装Logstash的系统上，创建一个Logstash管道配置，该配置从Logstash输入（如Beats或Kafka）读取，并将事件发送到Elasticsearch输出。将Elasticsearch输出中的管道选项设置为 `%{[@metadata][pipeline]}` 以使用先前加载的采集管道。

这是一个配置示例，它从Beats输入读取数据，并使用Filebeat采集管道来解析模块收集的数据：

```yaml
input {
  beats {
    port => 5044
  }
}

output {
  if [@metadata][pipeline] {
    elasticsearch {
      hosts => "https://061ab24010a2482e9d64729fdb0fd93a.us-east-1.aws.found.io:9243"
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
      pipeline => "%{[@metadata][pipeline]}" ①
      user => "elastic"
      password => "secret"
    }
  } else {
    elasticsearch {
      hosts => "https://061ab24010a2482e9d64729fdb0fd93a.us-east-1.aws.found.io:9243"
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
      user => "elastic"
      password => "secret"
    }
  }
}
```

![1](../source/images/common/1.png) 将 `pipeline` 选项设置为 `%{[@metadata][pipeline]}`。此设置将Logstash配置为根据事件中传递的元数据选择正确的采集管道。

有关设置和运行模块的更多信息，请参阅 [Filebeat模块文档](https://www.elastic.co/guide/en/beats/filebeat/6.7/filebeat-modules-overview.html)。

有关完整示例，请参阅 [示例：设置Filebeat模块以使用Kafka和Logstash](../09-Working-with-Filebeat-Modules/Example:-Set-up-Filebeat-modules-to-work-with-Kafka-and-Logstash.md)。