## beats

- 插件版本：v5.1.8
- 发布于：2018-11-30
- [更新日志](https://github.com/logstash-plugins/logstash-input-beats/blob/v5.1.8/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/input-beats-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-input-beats) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

此输入插件使Logstash能够从 [Elastic Beats](https://www.elastic.co/products/beats) 框架接收事件。

以下示例显示如何配置Logstash侦听端口5044，以获取传入的Beats连接，并索引到Elasticsearch。

```ruby
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => "localhost:9200"
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}" ①
    document_type => "%{[@metadata][type]}" ②
  }
}
```


![1](../source/images/common/1.png) 指定要将事件写入的索引。有关此设置的更多信息，请参阅 [Beats版本索引](#Beats版本索引)。

![2](../source/images/common/2.png) 从Logstash 6.0开始，由于Logstash 6.0 [删除了类型](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/removal-of-types.html)，因此不推荐使用 `document_type` 选项。它将在Logstash的下一个主要版本中彻底删除。如果您运行的是Logstash 6.0或更高版本，请不要在配置中设置 `document_type`，因为Logstash默认情况下将类型设置为 `doc`。

> **重要：**
> 如果要发运跨多行的事件，则需要使用Filebeat中提供的选项来 [配置处理多行事件](https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html)，然后再将事件数据发送到Logstash。您不能使用 [Multiline编解码器插件](../20-Coder-plugins/multiline.md) 来处理多行事件。这样做会导致无法启动Logstash。

### Beats版本索引

如果希望使未来的架构变更对Elasticsearch现有索引和映射的影响最小化，请配置Elasticsearch输出，以写入版本索引。您为 `index` 设置特定的匹配模式来控制索引名称：

```yaml
index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
```

`%{[@metadata][beat]}`

​	将索引名称的第一部分设置为 `beat` 元数据字段的值，例如 `filebeat`。

`%{[@metadata][version]}`

​	将名称的第二部分设置为Beat版本，例如 `6.7.1`。

`%{+YYYY.MM.dd}`

​	根据Logstash `@timestamp`字段将名称的第三部分设置为日期。

此配置会每日索引名称，如 `filebeat-6.7.1-2019-04-04`。

### Beats输入配置项

此插件支持以下配置选项，以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                                                         | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`add_hostname`](#add_hostname)                              | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`cipher_suites`](#cipher_suites) | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`client_inactivity_timeout`](#client_inactivity_timeout) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`host`](#host) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`include_codec_tag`](#include_codec_tag) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`port`](#port) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 是   |
| [`ssl`](#ssl) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`ssl_certificate`](#ssl_certificate) | 有效的文件系统路径                                           | 否   |
| [`ssl_certificate_authorities`](#ssl_certificate_authorities) | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`ssl_handshake_timeout`](#ssl_handshake_timeout) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`ssl_key`](#ssl_key) | 有效的文件系统路径                                           | 否   |
| [`ssl_key_passphrase`](#ssl_key_passphrase) | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password) | 否   |
| [`ssl_verify_mode`](#ssl_verify_mode) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选值为`["none", "peer", "force_peer"]` | 否   |
| [`ssl_peer_metadata`](#ssl_peer_metadata) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`tls_max_version`](#tls_max_version) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`tls_min_version`](#tls_min_version) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### add_hostname

> **注意：**
>
> **在5.1.4中新增。**
>
> 添加了字段以允许用户控制 `host` 字段是否自动添加到事件。

> **警告：**
>
> **在5.1.4中弃用。**
>
> 在此插件的未来版本中，将删除此设置，并且不会将hosts字段添加到事件中。

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `true`

标记以确定是否使用主机名字段中节拍提供的值将主机字段添加到事件。

##### cipher_suites

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 默认值是 `java.lang.String[TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256]@459cfcca`

要使用的密码套件列表，按优先级列出。

##### client_inactivity_timeout

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `60`

在X秒不活动后关闭空闲客户端。

##### host

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 默认值为 `"0.0.0.0"`

要监听的IP地址。

##### include_codec_tag

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为true

##### port

- 这是必需的设置。
- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 此设置没有默认值。

要监听的端口。

##### ssl

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `false`
- 默认情况下，事件以纯文本形式发送。您可以通过将 `ssl` 设置为true，并配置 `ssl_certificate` 和 `ssl_key` 选项来启用加密。

##### ssl_certificate

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path)
- 此设置没有默认值。

要使用的SSL证书。

##### ssl_certificate_authorities

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 默认值为 `[]`
- 根据这些权限验证客户端证书。您可以定义多个文件或路径。将读取所有证书并将其添加到信任库。您需要将 `ssl_verify_mode` 配置为 `peer` 或 `force_peer` 以启用验证。

##### ssl_handshake_timeout

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `10000`

ssl握手超时时间（以毫秒为单位）

##### ssl_key

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path)
- 此设置没有默认值。

要使用的SSL密钥。注意：此密钥必须采用PKCS8格式，您可以使用 [OpenSSL](https://www.openssl.org/docs/man1.1.0/man1/pkcs8.html) 进行转换以获取更多信息。

##### ssl_key_passphrase

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password)
- 此设置没有默认值。

要使用的SSL密钥密码。

##### ssl_verify_mode

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选值为`["none", "peer", "force_peer"]`
- 默认值为 `"none"`

默认情况下，服务器不进行任何客户端验证。

`peer` 会让服务器要求客户端提供证书。如果客户端提供证书，则将对其进行验证。

`force_peer` 将使服务器要求客户端提供证书。如果客户端未提供证书，则将关闭连接。

此选项需要与 `ssl_certificate_authorities` 和已定义的CA列表一起使用。

##### ssl_peer_metadata

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `false`

允许在事件的元数据中存储客户端证书信息。

此选项仅在 `ssl_verify_mode` 设置为 `peer` 或 `force_peer` 时有效。

##### tls_max_version

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `1.2`

允许加密连接的最大TLS版本。必须是以下值之一：对于TLS 1.0为1.0，对于TLS 1.1为1.1，对于TLS 1.2为1.2

##### tls_min_version

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `1`

允许加密连接的最小TLS版本。该值必须是以下值之一：对于TLS 1.0为1.0，对于TLS 1.1为1.1，对于TLS 1.2为1.2

### 通用配置项

所有输入插件都支持以下配置选项：

| 设置                                                         | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`add_field`](#add_field) | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`codec`](#codec) | [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Codec) | 否   |
| [`enable_metric`](#enable_metric) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`id`](#id) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`tags`](#tags) | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`type`](#type) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |

### 详情

##### add_field

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash)
- 默认值为 `{}`

向事件添加字段

##### codec

- 值类型是 [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Codec)
- 默认值是 `"plain"`

用于输入数据的编解码器。输入编解码器是一种在输入之前解码数据的便捷方法，无需在Logstash管道中使用单独的过滤器。

##### enable_metric

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `true`

默认情况下，禁用或启用此特定插件实例的指标记录，我们会记录所有的可用指标，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

为插件配置添加唯一 `ID`。如果未指定ID，Logstash将生成一个ID。强烈建议在配置中设置此ID。当您有两个或更多相同类型的插件时，这尤其有用，例如，如果您有2个beat输入，添加命名ID将有助于使用API监视Logstash。

```json
input {
  beats {
    id => "my_plugin_id"
  }
}
```

##### tags

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 此设置没有默认值。

为您的活动添加任意数量的任意标签。

这有助于后续处理。

##### type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

将 `type` 字段添加到此输入处理的所有事件。

类型主要用于过滤器激活。

类型存储为事件本身的一部分，因此您也可以使用该类型在Kibana中搜索它。

如果您尝试在已有事件的事件上设置类型（例如，当您将事件从发运人发送到索引器时），则新输入将不会覆盖现有类型。发运人设置的类型即使在发送到另一个Logstash服务器时，仍会保留。

> **注意：**
> Beats发运人自动在事件上设置类型字段。您无法在Logstash配置中覆盖此设置。如果在Logstash中指定了[type](#type) 选项，则会将其忽略。
