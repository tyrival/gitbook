# 第6章 配置

要配置Logstash，需要创建一个配置文件，指定要使用的插件和每个插件的设置。 您可以在配置中引用事件的字段，并在满足特定条件时，使用条件来处理事件。 运行logstash时，可使用 `-f` 指定配置文件。

让我们创建一个简单的配置文件，并使用它来运行Logstash。 创建名为"logstash-simple.conf"的文件，并将其保存到Logstash根目录中。

```yaml
input { stdin { } }
output {
  elasticsearch { hosts => ["localhost:9200"] }
  stdout { codec => rubydebug }
}
```

然后，运行Logstash并使用 `-f` 参数指定配置文件。

```shell
bin/logstash -f logstash-simple.conf
```

Logstash读取到指定的配置文件，并输出到Elasticsearch和stdout。 在我们继续讨论一些更复杂的 [配置示例](../06-Configuring-Logstash/Logstash-Configuration-Examples.md) 之前，让我们仔细看看配置文件中的内容。

