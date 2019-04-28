## http_poller

- 插件版本：v4.0.5
- 发布于：2018-04-06
- [更新日志](https://github.com/logstash-plugins/logstash-input-http_poller/blob/v4.0.5/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/input-http_poller-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-input-http_poller) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

这个Logstash输入插件允许您调用HTTP API，将其输出数据解码为事件，并按配置的渠道将其发送出去。 这个插件背后的想法来自于需要读取springboot指标端点，而不是配置jmx来监视我的java应用程序内存、GC等。

### 示例

从URL列表中读取，并使用编解码器解码响应内容。 配置应如下所示：

```sh
input {
  http_poller {
    urls => {
      test1 => "http://localhost:9200"
      test2 => {
        # Supports all options supported by ruby's Manticore HTTP client
        method => get
        user => "AzureDiamond"
        password => "hunter2"
        url => "http://localhost:9200/_cluster/health"
        headers => {
          Accept => "application/json"
        }
     }
    }
    request_timeout => 60
    # Supports "cron", "every", "at" and "in" schedules by rufus scheduler
    schedule => { cron => "* * * * * UTC"}
    codec => "json"
    # A hash of request metadata info (timing, response headers, etc.) will be sent here
    metadata_target => "http_poller_metadata"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

将HTTP轮询器与自定义CA或自签名证书一起使用。

如果您有自签名证书，则需要将服务器证书转换为有效的＃`.jks`或`.p12`文件。简单来说就是运行以下代码，用服务器的URL替换占位符`MYURL`和`MYPORT`。

```sh
openssl s_client -showcerts -connect MYURL:MYPORT </dev/null 2>/dev/null|openssl x509 -outform PEM > downloaded_cert.pem; keytool -import -alias test -file downloaded_cert.pem -keystore downloaded_truststore.jks
```

上面的代码片段将创建两个文件：`downloaded_cert.pem`和`downloaded_truststore.jks`。在此过程中，系统将提示您为`jks`文件设置密码。要配置logstash，请使用如下所示的配置。

```sh
 http_poller {
   urls => {
     myurl => "https://myhostname:1234"
   }
   truststore => "/path/to/downloaded_truststore.jks"
   truststore_password => "mypassword"
   schedule => { cron => "* * * * * UTC"}
 }
```

### Http_poller输入配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

| 设置                                                    | 输入类型                                                     | 必须 |
| ------------------------------------------------------- | ------------------------------------------------------------ | ---- |
| [`user`](#user)                                         | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`password`](#password)                                 | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`automatic_retries`](#automaticretries)                | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`cacert`](#http_pollercacert)                          | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`client_cert`](#clientcert)                            | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`client_key`](#clientkey)                              | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`connect_timeout`](#connecttimeout)                    | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`cookies`](#cookies)                                   | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`follow_redirects`](#follow_redirects)                 | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`keepalive`](#keepalive)                               | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`keystore`](#keystore)                                 | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`keystore_password`](#keystorepassword)                | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`keystore_type`](#keystorepassword)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`metadata_target`](#metadatatarget)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`pool_max`](#poolmax)                                  | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`pool_max_per_route`](#poolmaxperroute)                | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`proxy`](#proxy)                                       | <<,>>                                                        | 否   |
| [`request_timeout`](#requesttimeout)                    | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`retry_non_idempotent`](#retrynonidempotent)           | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`schedule`](#schedule)                                 | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`socket_timeout`](#sockettimeout)                      | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |
| [`target`](#target)                                     | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`truststore`](#truststore)                             | [path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path) | 否   |
| [`truststore_password`](#truststorepassword)            | [password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password) | 否   |
| [`truststore_type`](#truststoretype)                    | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`urls`](#urls)                                         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 是   |
| [`validate_after_inactivity`](#validateafterinactivity) | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### USER
- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。
用于所有请求的HTTP身份验证的用户名。请注意，您也可以为每个URL单独设置。如果设置此项，则还必须设置`password`选项。

##### password
- 值类型是[password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。
密码与HTTP身份验证的`user`一起使用。

##### automatic_retries
- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为1
客户端访问URL失败重试的次数。如果启用了keepalive，我们强烈建议不要将此值设置为0。有些服务器错误地结束了Keepalive时，需要重试！注意：如果仅设置`retry_non_idempotent`，则将重试GET，HEAD，PUT，DELETE，OPTIONS和TRACE请求。

##### cacert
- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。
如果需要使用自定义X.509 CA（.pem证书），请在此处指定路径

##### client_cert
- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。
如果您想使用客户端证书（注意，大多数人不想这样），请在此处设置x509证书的路径

##### client_key
- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。
如果您使用的是客户端证书，请在此处指定加密密钥的路径

##### connect_timeout
- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为10
等待建立连接的超时（以秒为单位）。默认值是`10s`

##### cookies
- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为true
启用cookie支持。启用此功能后，客户端将像通常的Web浏览器一样将Cookie保留在请求中。默认情况下启用

##### follow_redirects
- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为true
应该遵循重定向吗？默认为`true`

##### keepalive
- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为true
启用此选项可启用HTTP keepalive支持。我们强烈建议将`automatic_retries`至少设置为1，以修复与损坏的keepalive的交互。

##### keystore
- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。
如果需要使用自定义密钥库（`.jks`），请在此处指定。这不适用于.pem键！

##### keystore_password
- 值类型是[password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。
在此处指定密钥库密码。注意，使用keytool创建的大多数.jks文件都需要密码！

##### keystore_type
- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为`"JKS"`
在此处指定密钥库类型。 `JKS`或`PKCS12`之一。默认是`JKS`

##### metadata_target
- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为`"@metadata"`
如果您想使用请求/响应元数据。将此值设置为您要存储元数据嵌套哈希的字段的名称。

##### pool_max
- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`50`
最大并发连接数。默认为`50`

##### pool_max_per_route
- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`25`
与单个主机的最大并发连接数。默认为`25`

##### proxy
- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。
如果您想使用HTTP代理。这支持多种配置语法：
1. 代理格式：`http://proxy.org:1234`
2. 代理格式：`{host => "proxy.org", port => 80, scheme => 'http', user => 'username@host', password => 'password'}`
3. 代理格式：`{url => 'http://proxy.org:1234', user => 'username@host', password => 'password'}`

##### request_timeout
- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`60`
整个请求的超时（以秒为单位）。

##### retry_non_idempotent
- 值类型是[boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为`false`
如果启用了`automatic_retries`，则会导致重试非幂等HTTP谓词（例如POST）。

##### schedule
- 值类型是[hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。
何时定期从网址轮询格式：带+键的哈希："cron" | "every" | "in" | "at" + value: string Examples: a) { "every" ⇒ "1h" } b) { "cron" ⇒ "* * * * * UTC" }。更多格式请参阅：rufus/scheduler格式

##### socket_timeout
- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`10`
等待套接字上的数据的超时（以秒为单位）。默认值是`10s`

##### target
- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。
定义用于放置接收数据的目标字段。如果省略此设置，则数据将存储在事件的根（顶级）。

##### truststore
- 值类型是[path](../06-Configuring-Logstash/Structure-of-a-Config-File.md#path)
- 此设置没有默认值。
如果需要使用自定义信任库（`.jks`），请在此处指定。这不适用于.pem证书！

##### truststore_passwordedit
- 值类型是[password](../06-Configuring-Logstash/Structure-of-a-Config-File.md#password)
- 此设置没有默认值。
在此处指定信任库密码。注意，使用keytool创建的大多数.jks文件都需要密码！

##### truststore_type
- 值类型是[string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 默认值为`JKS`
在此处指定信任库类型。 `JKS`或`PKCS12`之一。默认是`JKS`

##### urls
- 这是必需的设置。
- 值类型是[hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 此设置没有默认值。
这种格式的网址哈希：`"name" => "url"`。 名称和网址将在outputed事件中传递

##### validate_after_inactivity
- 值类型是[number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#number)
- 默认值为`200`
在使用keepalive在连接上执行请求之前，检查连接是否过时之前要等待多长时间。 ＃如果定期发生连接错误，您可能希望将此值设置为较低，可能为0.引用Apache commons文档（此客户端基于Apache Commmons）：定义连接在保持不活动多长时间（以毫秒为单位）之后，必须重新验证持久连接，然后才提供给消费者。 非正值会禁用连接验证。 此检查有助于检测已变为陈旧（半关闭）的连接，同时在池中保持不活动状态。 有关详细信息，请参阅 [这些文档](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/impl/conn/PoolingHttpClientConnectionManager.html#setValidateAfterInactivity(int)

### 通用配置项

所有输入插件都支持以下配置选项：

| 设置                              | 输入类型                                                     | 必须 |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| [`add_field`](#addfield)         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash) | 否   |
| [`codec`](#codec)                 | [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#codec) | 否   |
| [`enable_metric`](#enablemetric) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean) | 否   |
| [`id`](#id)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |
| [`tags`](#tags)                   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array) | 否   |
| [`type`](#type)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string) | 否   |

#### 详情

##### add_field

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#hash)
- 默认值为 `{}`

向事件添加字段

##### codec

- 值类型是 [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#codec)
- 默认值是 `"plain"`

用于输入数据的编解码器。输入编解码器是一种在输入之前解码数据的便捷方法，无需在Logstash管道中使用单独的过滤器。

##### enable_metric

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#boolean)
- 默认值为 `true`

默认情况下，禁用或启用此特定插件实例的指标记录，我们会记录所有的可用指标，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
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

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#array)
- 此设置没有默认值。

为您的活动添加任意数量的任意标签。

这有助于后续处理。

##### type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#string)
- 此设置没有默认值。

将 `type` 字段添加到此输入处理的所有事件。

类型主要用于过滤器激活。

类型存储为事件本身的一部分，因此您也可以使用该类型在Kibana中搜索它。

如果您尝试在已有事件的事件上设置类型（例如，当您将事件从发运人发送到索引器时），则新输入将不会覆盖现有类型。发运人设置的类型即使在发送到另一个Logstash服务器时，仍会保留。
