## Elastic Cloud

Logstash提供了两个简化使用 [Elastic Cloud](https://cloud.elastic.co) 模块的设置。 Elastic Cloud中的Elasticsearch和Kibana主机名可能很难在Logstash配置或命令行中设置，此时可以使用Cloud ID。

> **注意：**
> Cloud ID仅在启用Logstash模块时才使用，否则指定Cloud ID无效。Cloud ID适用于通过模块发送的数据，通过X-Pack监控发送的运行时度量，以及Logstash的X-Pack集中管理功能使用的端点，除非在 `logstash.yml` 中指定了对X-Pack设置的显式覆盖。

### Cloud ID

在Elastic Cloud Web控制台中找到的Cloud ID，由Logstash用于构建Elasticsearch和Kibana主机设置。它是一个base64编码的文本值，大约120个字符，由大小写字母和数字组成。如果您有多个Cloud ID，则可以添加一个标签，这个标签在内部会被忽略，但可以帮助您区分它们。要添加标签，您应该在Cloud ID前面加上标签和分隔符，格式为 `"<label>:<cloud-id>"`

`cloud.id`将覆盖这些设置：

```yaml
var.elasticsearch.hosts
var.kibana.host
```

### Cloud Auth

这是可选的。按照以下格式 `"<username>:<password>"` 构建值，其中，`username` 和 `password` 是在Cloud UI中创建集群时，给出的Cloud账号密码。由于您的Cloud密码是可更改的，如果您在Cloud UI中更改了，请记住在此处也要同步更改。

指定 `cloud.auth` 时将覆盖这些设置：

```yaml
var.elasticsearch.username
var.elasticsearch.password
var.kibana.username
var.kibana.password
```

例：

可以在 [logstash.yml](../04-Setting-Up-and-Running-Logstash/logstash.yml.md) 中指定这些设置。它们应与您之前添加的任何模块配置分开。

```yaml
# example with a label
cloud.id: "staging:dXMtZWFzdC0xLmF3cy5mb3VuZC5pbyRub3RhcmVhbCRpZGVudGlmaWVy"
cloud.auth: "elastic:YOUR_PASSWORD"
```

```yaml
# example without a label
cloud.id: "dXMtZWFzdC0xLmF3cy5mb3VuZC5pbyRub3RhcmVhbCRpZGVudGlmaWVy"
cloud.auth: "elastic:YOUR_PASSWORD"
```

这些设置同样可以通过命令行进行指定，如下：

```yaml
bin/logstash --modules netflow -M "netflow.var.input.udp.port=3555" --cloud.id <cloud-id> --cloud.auth <cloud.auth>
```

