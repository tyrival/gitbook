## http

- 插件版本：v3.3.0
- 发布于：2019-02-04
- [更新日志](https://github.com/logstash-plugins/logstash-input-http/blob/v3.3.0/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/input-http-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-input-http) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

使用此输入，您可以通过http（https）接收单行或多行事件。应用程序可以向其发送HTTP请求，Logstash会将其转换为事件以供后续处理。用户可以传递纯文本，JSON或任何格式化数据，并在此输入中使用相应的编解码器。对于 Content-Type `application/json` ，使用 `json` 编解码器，但对于所有其他数据格式，使用 `plain` 编解码器。

此输入还可用于接收与其他服务和应用程序集成的webhook请求。通过利用Logstash中提供的庞大插件生态系统，您可以直接从应用程序触发可操作的事件。

### 阻止行为

HTTP协议不能很好地处理长时间运行的请求。当Logstash积压时，此插件将返回429(busy)错误，否则它将超时请求。

如果遇到429错误，客户端应该睡眠，使用一些随机抖动以指数方式退回，然后重试其请求。

如果Logstash队列被阻止并且有可用的HTTP输入线程，则此插件将阻止。这将导致大多数HTTP客户端超时。在这种情况下，仍会处理已发送的事件。这不是最好的做法，将在以后的版本中进行更改。将来，如果队列繁忙，此插件将始终返回429，并且在繁忙的队列中不会超时。

### 安全

此插件支持标准HTTP基本身份验证头以标识请求者。您可以在将数据发送到此输入时，传递用户名、密码组合

您还可以通过多种方式设置SSL并通过https安全地发送数据，例如验证客户端的证书。

### 编解码器设置

这个插件有两个编解码器配置选项： `codec` 和 `additional_codecs`。

`additional_codecs` 中的值优先于 `codec` 选项中指定的值。也就是说，仅当在 `additional_codecs` 中找不到请求的content-type的编解码器时，才应用默认 `codec`。

### Http输入配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                                                         | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`additional_codecs`](#additional_codecs)                    | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`cipher_suites`](#cipher_suites)                            | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`host`](#host)                                              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`keystore`](#keystore)                                      | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path) | 否   |
| [`keystore_password`](#keystore_password)                    | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password) | 否   |
| [`password`](#password)                                      | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password) | 否   |
| [`port`](#port)                                              | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`max_pending_requests`](#max_pending_requests)              | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`response_headers`](#response_headers)                      | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`ssl`](#ssl)                                                | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`ssl_certificate`](#ssl_certificate)                        | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path) | 否   |
| [`ssl_certificate_authorities`](#ssl_certificate_authorities) | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`ssl_handshake_timeout`](#ssl_handshake_timeout)            | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`ssl_key`](#ssl_key)                                        | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path) | 否   |
| [`ssl_key_passphrase`](#ssl_key_passphrase)                  | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password) | 否   |
| [`ssl_verify_mode`](#ssl_verify_mode)                        | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项有`["none", "peer", "force_peer"]` | 否   |
| [`threads`](#threads)                                        | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`tls_max_version`](#tls_max_version)                        | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`tls_min_version`](#tls_min_version)                        | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`user`](#user)                                              | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`verify_mode`](#verify_mode)                                | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项有`["none", "peer", "force_peer"]` | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### additional_codecs

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash)
- 默认值为 `{"application/json"=>"json"}`

为特定内容类型应用特定编解码器。只有在选中此列表并且找不到请求的内容类型的编解码器，才会应用默认编解码器

##### cipher_suites

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 默认值是 `java.lang.String[TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256, TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384, TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256, TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256]@459cfcca`

要使用的密码套件列表，按优先级列出。

##### host

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 默认值为 `"0.0.0.0"`

主机或ip绑定

##### keystore

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path)
- 此设置没有默认值。
- 不推荐使用此选项

用于验证客户端证书的JKS密钥库

注意：不推荐使用此选项，它将在Logstash的下一个主要版本中删除。请改用 `ssl_certificate和ssl_key`。

##### keystore_password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password)
- 此设置没有默认值。
- 不推荐使用此选项

设置信任库密码

注意：不推荐使用此选项，它将在Logstash的下一个主要版本中删除。请改用`ssl_certificate` 和 `ssl_key`。

##### password

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password)
- 此设置没有默认值。

基本授权密码

##### port

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为`8080`

要绑定的TCP端口

##### max_content_length

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为104857600

HTTP请求的最大内容（以字节为单位）。默认为100mb。

##### max_pending_requests

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为200

在由工作者线程处理之前，存储在临时队列中的最大传入请求数。如果请求到达并且队列已满，则将立即返回429响应。此队列用于处理微突发事件并提高整体吞吐量，因此应该非常小心地更改，因为它可能导致内存压力和影响性能。如果您需要处理传入请求中的周期性或不可预见的峰值，请考虑为logstash管道启用持久队列。

##### response_headers

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash)
- 默认值为 `{"Content-Type"=>"text/plain"}`

指定一组自定义响应头

##### remote_host_target_field

- 值类型是字符串
- 默认值为 `"host"`

为http请求的客户端主机指定目标字段

##### request_headers_target_field

- 值类型是字符串
- 默认值为 `"headers"`

为http请求的客户端主机指定目标字段

##### ssl

- 值类型是布尔值
- 默认值为 `false`

默认情况下，事件以纯文本形式发送。您可以通过将 `ssl` 设置为true，并配置 `ssl_certificate` 和 `ssl_key` 选项来启用加密。

##### ssl_certificate

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path)
- 此设置没有默认值。

要使用的SSL证书。

##### ssl_certificate_authorities

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 默认值为 `[]`

根据这些权限验证客户端证书。您可以定义多个文件或路径。将读取所有证书并将其添加到信任库。您需要将 `ssl_verify_mode` 配置为 `peer` 或 `force_peer` 以启用验证。

##### ssl_handshake_timeout

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为10000

不完整的ssl握手超时的时间（以毫秒为单位）

##### ssl_key

- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Path)
- 此设置没有默认值。

要使用的SSL密钥。注意：此密钥必须采用PKCS8格式，您可以使用 [OpenSSL](https://www.openssl.org/docs/man1.1.0/man1/pkcs8.html) 进行转换以获取更多信息。

##### ssl_key_passphrase

- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Password)
- 此设置没有默认值。

要使用的SSL密钥密码。

##### ssl_verify_mode

- 值类型为 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项有`none, peer, force_peer`
- 默认值为 `"none"`

默认情况下，服务器不进行任何客户端验证。

`peer` 会让服务器要求客户端提供证书。如果客户端提供证书，则将对其进行验证。

`force_peer` 将使服务器要求客户端提供证书。如果客户端未提供证书，则将关闭连接。

此选项需要与 `ssl_certificate_authorities` 和已定义的CA列表一起使用。

##### threads

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值是处理器数

用于接受连接和处理请求的线程数

##### tls_max_version

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `1.2`

允许加密连接的最大TLS版本。该值必须是以下值之一：对于TLS 1.0为1.0，对于TLS 1.1为1.1，对于TLS 1.2为1.2

##### tls_min_version

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `1`

允许加密连接的最小TLS版本。该值必须是以下值之一：对于TLS 1.0为1.0，对于TLS 1.1为1.1，对于TLS 1.2为1.2

##### USER

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

基本授权的用户名

##### verify_mode

- 值类型为 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项有  `none`, `peer`, `force_peer`
- 默认值为 `"none"`
- 不推荐使用此选项

设置客户端证书验证方法。有效方法：none，peer，force_peer

注意：不推荐使用此选项，它将在Logstash的下一个主要版本中删除。请改用 `ssl_verify_mode`。

### 通用配置项

所有输入插件都支持以下配置选项：

| 设置                              | 输入类型                                                     | 必须 |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| [`add_field`](#add_field)         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`codec`](#codec)                 | [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Codec) | 否   |
| [`enable_metric`](#enable_metric) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`id`](#id)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`tags`](#tags)                   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`type`](#type)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |

#### 详情

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