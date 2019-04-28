## http

- 插件版本：v5。2。4
- 发布于：2019-01-30
- [更新日志](https://github.com/logstash-plugins/logstash-output-http/blob/v5.2.4/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/output-http-index.html)。

### 帮助
有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-output-http) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

此输出允许您将事件发送到通用HTTP（S）端点。

此输出将并行执行最多pool_max请求以提高性能。 优化此插件性能时请考虑这一点。

另外，请注意，当使用并行执行时，不保证事件的严格排序！

请注意，此gem尚不支持编解码器。 请使用*format*选项。

### Http输出配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 配置项                                                       | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`automatic_retries`](#automaticretries) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`cacert`](#cacert) | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)                                      | 否   |
| [`client_cert`](#clientcert) | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)                                      | 否   |
| [`client_key`](#clientkey) | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)                                      | 否   |
| [`connect_timeout`](#connecttimeout) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`content_type`](#contenttype) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`cookies`](#cookies) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`follow_redirects`](#followredirects) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`format`](#format) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string), 值包括`["json", "json_batch", "form", "message"]` | 否   |
| [`headers`](#headers) | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`http_compression`](#httpcompression) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`http_method`](#htt_method) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string), 值包括`["put", "post", "patch", "delete", "get", "head"]` | 是   |
| [`ignorable_codes`](ignorablecodes) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`keepalive`](#keepalive) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`keystore`](#keystore) | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)                                      | 否   |
| [`keystore_password`](#keystorepassword) | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`keystore_type`](#keystoretype) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`mapping`](#mapping) | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`message`](#message) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`pool_max`](#poolmax) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`pool_max_per_route`](#poolmaxperroute) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`proxy`](#proxy) | <<,>>                                                        | 否   |
| [`request_timeout`](#requesttimeout) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`retry_failed`](#retryfailed) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`retry_non_idempotent`](#etrynonidempotent) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`retryable_codes`](#retryablecodes) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`socket_timeout`](#sockettimeout) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`truststore`](#truststore) | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)                                      | 否   |
| [`truststore_password`](#truststorepassword) | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`truststore_type`](#truststoretype) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`url`](#url) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 是   |
| [`validate_after_inactivity`](#validateafterinactivity) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |

另请参阅  [通用配置项](#通用配置项) 以获取所有输出插件支持的选项列表。

##### `automatic_retries`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`1`

客户端应重试失败的URL多少次。如果启用了keepalive，我们强烈建议不要将此值设置为零。有些服务器错误地结束了Keepalive，需要重试！注意：如果仅设置 `retry_non_idempotent`，则将重试GET，HEAD，PUT，DELETE，OPTIONS和TRACE请求。

##### `cacert`

- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果需要使用自定义X.509 CA（.pem证书），请在此处指定路径

##### `client_cert`

- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果您想使用客户端证书（注意，大多数人不想这样），请在此处设置x509证书的路径

##### `client_key`

- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果您使用的是客户端证书，请在此处指定加密密钥的路径

##### `connect_timeout`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`10`

等待建立连接的超时（以秒为单位）。默认值是10秒

##### `content_type`

- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

内容类型

如果未指定，则默认为以下内容：

- 如果格式是"json"，"application/json"
- 如果格式是"json_batch"，"application/json"。所有Logstash事件批将连接成一个数组并在一个请求中发送。
- 如果格式是"form"，"application/x-www-form-urlencoded"

##### `cookies`

- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为`true`

启用cookie支持。启用此功能后，客户端将像通常的Web浏览器一样将Cookie保留在请求中。默认情况下启用

##### `follow_redirects`

- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为`true`

应该遵循重定向吗？默认为true

##### `format`

- [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string), 值包括`"json", "json_batch", "form", "message"`
- 默认值为`json`

设置http正文的格式。

如果是json_batch，则此输出接收的每批事件将被放入单个JSON数组中，并在一个请求中发送。这对于高吞吐量方案（例如在Logstash实例之间发送数据）特别有用。

如果是form，则body将映射（或整个事件）转换为查询参数字符串，例如，`foo=bar&baz=fizz...`

如果是消息，那么正文将是根据消息格式化事件的结果

否则，事件将以json的形式发送。

##### `headers`

- 值类型是[hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

使用格式的自定义标题是`headers => ["X-My-Header", "%{host}"]`

##### `http_compression`

- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为`false`

启用请求压缩支持。启用此功能后，插件将使用gzip压缩http请求。

##### `http_method`

- 这是必需的设置。
- [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string), 值包括`"put", "post", "patch", "delete", "get", "head"`
- 此设置没有默认值。

HTTP动词。 "put"，"post"，"patch"，"delete"，"get"，"head"之一

##### `ignorable_codes`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 此设置没有默认值。

如果你想考虑一些non-2xx代码成功，请在这里美剧它们。返回这些代码的回复将被视为成功

##### `keepalive`

- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为true

启用此选项可启用HTTP keepalive支持。我们强烈建议将`automatic_retries`至少设置为1，以修复与已损坏的keepalive实现的交互。

##### `keystore`

- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果需要使用自定义密钥库（`.jks`），请在此处指定。这不适用于.pem键！

##### `keystore_password`

- 值类型是密码
- 此设置没有默认值。

在此处指定密钥库密码。注意，使用keytool创建的大多数.jks文件都需要密码！

##### `keystore_type`

- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为`"JKS"`

在此处指定密钥库类型。 `JKS`或`PKCS12`之一。默认是`JKS`

##### `mapping`

- 值类型是[hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。

这使您可以选择发送的事件的结构和部分。

例如：

```sh
mapping => {"foo" => "%{host}"
           "bar" => "%{type}"}
```
           

##### `message`

- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

##### `pool_max`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`50`

最大并发连接数。默认为`50`

##### `pool_max_per_route`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`25`

与单个主机的最大并发连接数。默认为`25`

##### `proxy`

- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

如果您想使用HTTP代理。这支持多种配置语法：

1. 代理主机的形式： `http://proxy.org:1234`
2. 代理主机格式为： `{host => "proxy.org", port => 80, scheme => 'http', user => 'username@host', password => 'password'}`
3. 表单中的代理主机：`{url => 'http://proxy.org:1234', user => 'username@host', password => 'password'}`

##### `request_timeout`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为60

该模块可以根据[Manticore]（https://github.com/cheald/manticore）轻松添加完全配置的HTTP客户端到logstash。有关其用法的示例，请参阅 https://github.com/logstash-plugins/logstash-input-http_poller 获取整个请求的超时（以秒为单位）信息

##### `retry_failed`

- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为`true`

如果您不希望此输出重试失败的请求，请将此项设置为`false`

##### `retry_non_idempotent`

- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为`false`

如果启用了`automatic_retries`，则会导致重试非幂等HTTP谓词（例如POST）。

##### `retryable_codes`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`[429,500,502,503,504]`

如果遇到响应代码，此插件将重试这些请求

##### `socket_timeout`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`10`

等待套接字上的数据的超时（以秒为单位）。默认值是`10s`

##### `truststore`

- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果需要使用自定义信任库（`.jks`），请在此处指定。这不适用于.pem证书！

##### `truststore_password`

- 值类型是[password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

在此处指定信任库密码。注意，使用keytool创建的大多数.jks文件都需要密码！

##### `truststore_type`

- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为`JKS`

在此处指定信任库类型。 `JKS`或`PKCS12`之一。默认是`JKS`

##### `url`

- 这是必需的设置。
- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

要使用的URL

##### `validate_after_inactivity`

- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为200

在检查过时连接以确定是否需要keepalive请求之前等待多长时间。如果定期发生连接错误，请考虑将此值设置为低于默认值，可能设置为0。

此客户端基于Apache Commons。以下是 [Apache Commons文档](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/impl/conn/PoolingHttpClientConnectionManager.html#setValidateAfterInactivity(int)) 描述此选项的方式："定义不活动的时间段（以毫秒为单位），之后必须在租用给使用者之前重新验证持久连接。传递给此方法的非正值会禁用连接验证。此检查有助于检测已经变得陈旧（半封闭）但在游泳池中保持不活动状态的连接。"

### 通用配置项

所有输出插件都支持以下配置选项：

| 设置                              | 输入类型                                                     | 必须 |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| [`codec`](#codec)                 | [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#codec) | 否   |
| [`enable_metric`](#enablemetric)  | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`id`](#id)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

##### codec

- 值类型是 [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#codec)
- 默认值为 `plain`

用于输出数据的编解码器。 输出编解码器是在数据离开输出之前进行编码的一种简单途径，无需在Logstash管道中使用单独的过滤器。

##### enable_metric

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

禁用或启用此特定插件实例的度量标准记录。 默认情况下，我们会记录所有可用的指标，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

为插件配置添加唯一ID。 如果未指定ID，Logstash将生成一个ID。 强烈建议在配置中设置此ID。 当您有两个或更多相同类型的插件时，这尤其有用。 例如，如果您有2个elasticsearch输出。 在这种情况下添加命名ID将有助于在使用监视API时监视Logstash。

```sh
output {
  http {
    id => "my_plugin_id"
  }
}
```
