## 反序列化数据

本节中描述的插件对于将数据反序列化为Logstash事件非常有用。

[avro codec](../20-Coder-plugins/avro.md)

将序列化的Avro记录读取为Logstash事件。此插件反序列化单个Avro记录，它不适用于阅读Avro文件。 Avro文件具有必须在输入时处理的唯一格式。

以下配置反序列化来自Kafka的输入：

```json
input {
  kafka {
    codec => {
      avro => {
        schema_uri => "/tmp/schema.avsc"
      }
    }
  }
}
...
```

[csv filter](../19-Filter-plugins/csv.md)

将逗号分隔的值数据解析为单个字段。默认情况下，过滤器会自动生成字段名称（column1，column2等），或者您可以指定名称列表。您还可以更改列分隔符。

以下配置将CSV数据解析为 `columns` 字段中指定的字段名称：

```json
filter {
  csv {
    separator => ","
    columns => [ "Transaction Number", "Date", "Description", "Amount Debit", "Amount Credit", "Balance" ]
  }
}
```

[fluent codec](../20-Coder-plugins/fluent.md)

读取Fluentd `msgpack`架构。

以下配置解码从 `fluent-logger-ruby` 收到的日志：

```json
input {
  tcp {
    codec => fluent
    port => 4000
  }
}
```

[json codec](../20-Coder-plugins/json.md)

解码（通过输入）和编码（通过输出）JSON格式的内容，在JSON数组中为每个元素创建一个事件。

以下配置解码文件中的JSON格式内容：

```json
input {
  file {
    path => "/path/to/myfile.json"
    codec =>"json"
}
```

[protobuf filter](../20-Coder-plugins/protobuf.md)

读取protobuf编码的消息并将其转换为Logstash事件。需要将protobuf定义编译为Ruby文件。您可以使用 [ruby-protoc编译器](https://github.com/codekitchen/ruby-protocol-buffers) 编译它们。

以下配置解码来自Kafka流的事件：

```json
input
  kafka {
    zk_connect => "127.0.0.1"
    topic_id => "your_topic_goes_here"
    codec => protobuf {
      class_name => "Animal::Unicorn"
      include_path => ['/path/to/protobuf/definitions/UnicornProtobuf.pb.rb']
    }
  }
}
```

[xml filter](../19-Filter-plugins/xml.md)

将XML解析为字段。

以下配置解析存储在 `message` 字段中的整个XML文档：

```json
filter {
  xml {
    source => "message"
  }
}
```