## http

- 插件版本：v1.0.1
- 发布于：2019-02-06
- [更新日志](https://github.com/logstash-plugins/logstash-filter-http/blob/v1.0.1/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/filter-http-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-filter-http) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

HTTP筛选器提供与外部Web服务/ REST API的集成。

### HTTP过滤器配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                                                         | 输入类型                                                     | 必须 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [`body`](#body) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)，[array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)，[hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`body_format`](#bodyformat) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`headers`](#headers) | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`query`](#query) | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`target_body`](#targetbody) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`target_headers`](#targetheaders) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`url`](#url) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 是   |
| [`verb`](#verb) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

有多个配置项与Http连接相关：

| 设置                                                      | 输入类型                                                     | 必须 |
| --------------------------------------------------------- | ------------------------------------------------------------ | ---- |
| [`automatic_retries`](#automaticretries)                 | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`cacert`](#cacert)                                       | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`client_cert`](#clientcert)                             | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`client_key`](#clientkey)                               | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`connect_timeout`](#connecttimeout)                     | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`cookies`](#cookies)                                     | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`follow_redirects`](#followredirects)                   | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`keepalive`](#keepalive)                                 | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`keystore`](#keystore)                                   | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`keystore_password`](#keystorepassword)                 | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`keystore_type`](#keystoretype)                         | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`password`](#password)                                   | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`pool_max`](#poolmax)                                   | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`pool_max_per_route`](#poolmaxperroute)               | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`proxy`](#proxy)                                         | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`request_timeout`](#requesttimeout)                     | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`retry_non_idempotent`](#retrynonidempotent)           | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`socket_timeout`](#sockettimeout)                       | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`truststore`](#truststore)                               | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`truststore_password`](#truststorepassword)             | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`truststore_type`](#truststoretype)                     | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`user`](#user)                                           | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`validate_after_inactivity`](#validateafterinactivity) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### body
- 值类型可以是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)，[array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)，[hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 没有默认值

要发送的HTTP请求的body。

##### body_format
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，值可以是 `"json"` 或 `"text"`
- 默认值为 `"text"`

如果设置为 `"json"`，则主体将序列化为JSON。否则它按原样发送。

##### headers
- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 没有默认值

要在请求中发送的HTTP头。标题的名称及其值都可以引用事件字段中的值。

##### query
- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 没有默认值

定义要在HTTP请求中发送的查询字符串参数（键值对）。

##### target_body
- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 没有默认值

定义用于放置HTTP响应正文的目标字段。如果省略此设置，则数据将存储在"body"字段中。

##### target_headers
- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 没有默认值

定义用于放置HTTP响应标头的目标字段。如果省略此设置，则数据将存储在"header"字段中。

##### url
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 没有默认值

发送请求的URL。可以从事件字段中获取值。

##### verb
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)，值包含 `"GET"`, `"HEAD"`, `"PATCH"`, `"DELETE"`, `"POST"`
- 默认值为 `"GET"`

用于HTTP请求的动词。

### HTTP过滤器连接项
##### automatic_retries

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `1`

客户端应重试失败的URL多少次。如果启用了keepalive，我们强烈建议不要将此值设置为零。有些服务器错误地结束了Keepalive，需要重试！注意：如果仅设置 `retry_non_idempotent`，则将重试GET，HEAD，PUT，DELETE，OPTIONS和TRACE请求。

##### cacert
- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果需要使用自定义X.509 CA（.pem证书），请在此处指定路径

##### client_cert
- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果您想使用客户端证书（注意，大多数人不想这样），请在此处设置x509证书的路径

##### client_key
- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果您使用的是客户端证书，请在此处指定加密密钥的路径

##### connect_timeout
- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `10`

等待建立连接的超时（以秒为单位）。默认值是10秒

##### cookies
- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

启用cookie支持。启用此功能后，客户端将像通常的Web浏览器一样在请求中保留cookie。默认情况下启用

##### follow_redirects
- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

应该遵循重定向吗？默认为 `true`

##### keepalive
- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

启用此选项可启用HTTP keepalive支持。我们强烈建议将 `automatic_retries` 设置为至少一个，以修复与已损坏的keepalive实现的交互。

##### keystore
- 值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。

如果需要使用自定义密钥库（`.jks`），请在此处指定。这不适用于.pem键！

##### keystore_password
- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

在此处指定密钥库密码。注意，使用keytool创建的大多数.jks文件都需要密码！

##### keystore_type
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"JKS"`

在此处指定密钥库类型。` JKS` 或 `PKCS12` 之一。默认是 `JKS`

##### password
- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

密码与HTTP身份验证的用户名一起使用。

##### pool_max
- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `50`

最大并发连接数。默认为50

##### pool_max_per_route
- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `25`

与单个主机的最大并发连接数。默认为 `25`

##### proxy
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

如果您想使用HTTP代理。这支持多种配置语法：

1. 代理主机的形式： `http://proxy.org:1234`
2. 代理主机格式为： `{host => "proxy.org", port => 80, scheme => 'http', user => 'username@host', password => 'password'}`
3. 表单中的代理主机：`{url => 'http://proxy.org:1234', user => 'username@host', password => 'password'}`

##### request_timeout
- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `60`

整个请求的超时（以秒为单位）。

##### retry_non_idempotent
- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

如果启用了`automatic_retries`，则会导致重试非幂等HTTP谓词（例如POST）。

##### socket_timeout
- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `10`

等待套接字上的数据的超时（以秒为单位）。默认值是 `10s`

##### truststore
值类型是 [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
此设置没有默认值。
如果需要使用自定义信任库（`.jks`），请在此处指定。这不适用于.pem证书！

##### truststore_password
- 值类型是 [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。

在此处指定信任库密码。注意，使用keytool创建的大多数.jks文件都需要密码！

##### truststore_type
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为 `"JKS"`

在此处指定信任库类型。` JKS` 或 `PKCS12` 之一。默认是 `JKS`

##### USER
- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

用于所有请求的HTTP身份验证的用户名。请注意，您也可以设置此每个URL。如果设置此项，则还必须设置 `password` 选项。

##### validate_after_inactivity
- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为 `200`

在检查过时连接以确定是否需要keepalive请求之前等待多长时间。如果定期发生连接错误，请考虑将此值设置为低于默认值，可能设置为0。

此客户端基于Apache Commons。以下是 [Apache Commons文档](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/impl/conn/PoolingHttpClientConnectionManager.html#setValidateAfterInactivity(int)) 描述此选项的方式："定义不活动的时间段（以毫秒为单位），之后必须在租用给使用者之前重新验证持久连接。传递给此方法的非正值会禁用连接验证。此检查有助于检测已经变得陈旧（半封闭）但在游泳池中保持不活动状态的连接。"

### 通用配置项

所有过滤器插件都支持以下配置选项：

| 设置                                | 输入类型                                                     | 必须 |
| ----------------------------------- | ------------------------------------------------------------ | ---- |
| [`add_field`](#addfield)           | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`add_tag`](#addtag)               | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`enable_metric`](#enablemetric)   | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`id`](#id)                         | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`periodic_flush`](#periodicflush) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`remove_field`](#removefield)     | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`remove_tag`](#removetag)         | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |

##### add_field
- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为 `{}`

如果此过滤器配置成功，则向此事件添加任意字段。字段名称可以是动态的，并使用 `％{field}` 包含事件的部分内容。

例：

```sh
filter {
  http {
    add_field => { "foo_%{somefield}" => "Hello world, from %{host}" }
  }
}
```

```sh
# You can also add multiple fields at once:
filter {
  http {
    add_field => {
      "foo_%{somefield}" => "Hello world, from %{host}"
      "new_field" => "new_static_value"
    }
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将添加字段 `foo_hello` （如果存在），上面的值和 `%{host}` 部分替换为事件中的该值。第二个例子还会添加一个硬编码字段。

##### add_tag
- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

如果此过滤器成功，则向事件添加任意标记。标签可以是动态的，并使用 `%{field}` 语法包含事件的一部分。

例：

```sh
filter {
  http {
    add_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also add multiple tags at once:
filter {
  http {
    add_tag => [ "foo_%{somefield}", "taggedy_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将添加标记 `foo_hello`（第二个示例当然会添加 `taggedy_tag` 标记）。

##### enable_metric
- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

默认情况下，禁用或启用此插件实例的指标记录，我们会记录所有可用的度量标准，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

为插件配置添加唯一 `ID`。如果未指定ID，Logstash将生成一个ID。强烈建议在配置中设置此ID。当您有两个或更多相同类型的插件时，这尤其有用，例如，如果您有2个http过滤器。在这种情况下添加命名ID将有助于在使用监视API时监视Logstash。

```sh
filter {
  http {
    id => "ABC"
  }
}
```

##### periodic_flush
- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `false`

定期调用过滤器刷新方法。可选的。

##### remove_field
- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

如果此过滤器成功，请从此事件中删除任意字段。例：

```sh
filter {
  http {
    remove_field => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple fields at once:
filter {
  http {
    remove_field => [ "foo_%{somefield}", "my_extraneous_field" ]
  }
}
```

##### remove_tag

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 默认值为 `[]`

如果此过滤器配置成功，则从事件中删除任意tag。tag以是动态的，并使用 `%{field}` 语法包含事件的一部分。

例：

```sh
filter {
  http {
    remove_tag => [ "foo_%{somefield}" ]
  }
}
```

```sh
# You can also remove multiple tags at once:
filter {
  http {
    remove_tag => [ "foo_%{somefield}", "sad_unwanted_tag"]
  }
}
```

如果事件具有字段 `"somefield" == "hello"` ，则此过滤器成功时将删除标记 foo_hello（如果存在）。第二个例子也会删除一个不需要的标签。
