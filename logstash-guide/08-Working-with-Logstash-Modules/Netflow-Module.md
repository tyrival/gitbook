## Netflow模块

Logstash Netflow模块简化了网络流数据收集，并使其规范化和可视化。通过一个命令实现网络流数据解析，将事件索引到Elasticsearch，并安装一套Kibana仪表板，以便您进行数据探索。

Logstash模块支持Netflow 5和9。

### Flow Data是什么？

Netflow是一种来自网络设备的流数据记录，它遍历了设备的连接信息，包括源IP地址和端口，目标IP地址和端口，服务类型，VLAN以及可以编码为帧和协议头的其他信息。利用Netflow数据，网络运营商可以不仅监控穿过其网络的数据量，还可以了解流量来自何处，流向何处以及它所属的服务或应用程序。

#### 环境需求

以下假设您已经安装了Elastic Stack（Logstash，Elasticsearch和Kibana）5.6或更高版本。您需要的产品可 [在此下载](https://www.elastic.co/downloads/) 和轻松安装。

### 入门

1. 通过在Logstash安装目录中运行以下命令来启动Logstash Netflow模块：

```shell
bin/logstash --modules netflow --setup -M netflow.var.input.udp.port=NNNN
```

其中 `NNNN` 是Logstash将侦听网络流量数据的UDP端口。如果未指定端口，则Logstash将默认侦听端口2055。

`--modules netflow` 选项会启动Netflow-aware的Logstash管道以供摄取。

`--setup` 选项在Elasticsearch中创建 `netflow- *` 索引匹配，并导入Kibana仪表板和可视化。运行 `--setup` 是一次性设置步骤。后续运行需省略该参数，以避免覆盖现有的Kibana仪表板。

此处的命令假定您已在本地主机上运行Elasticsearch和Kibana。如果不是，则需要指定其他连接选项。请参阅 [配置模块](#配置模块)。

2. 探索数据：

   a. 打开浏览器并导航到 http://localhost:5601。如果启用了安全性，则需要指定在设置安全性时使用的Kibana用户名和密码。

   b. 打开 **Netflow: Network Overview Dashboard**。

   c. 有关数据探索的其他详细信息，请参阅 [探索数据](#探索数据)。

### 探索数据

一旦Logstash Netflow模块开始处理事件，您就可以立即开始使用Kibana仪表板来探索和可视化您的网络流数据。

您可以按默认样式使用仪表板，也可以根据现有用例和业务需求对其进行定制。

#### 仪表板示例

在 **Overview** 仪表板上，您可以查看基本流量数据的摘要，以及设置过滤器，然后再深入了解数据。

![netflow-overview](../source/images/ch-08/netflow-overview.png)

例如，在 **Conversation Partners** 仪表板上，您可以在任何通讯中查看客户端和服务器的源地址和目标地址。

![netflow-conversation-partners](../source/images/ch-08/netflow-conversation-partners.png)
在 **Traffic Analysis** 仪表板上，您查看以字节为单位的流量，从而来识别高流量通讯。

![netflow-traffic-analysis](../source/images/ch-08/netflow-traffic-analysis.png)

然后，您可以转到 **Geo Location** 仪表板，在该仪表板中可以显示热图上目的地和来源的位置。

![netflow-geo-location](../source/images/ch-08/netflow-geo-location.png)

### 配置模块

您可以通过设置 `logstash.yml` ，或者使用命令行覆盖其设置，来进一步优化Logstash Netflow模块。

例如，`logstash.yml` 中，如下配置将Logstash设置为侦听端口9996，以获取网络流量数据：

```yaml
modules:
  - name: netflow
    var.input.udp.port: 9996
```

要在命令行中指定相同的设置，请使用：

```shell
bin/logstash --modules netflow -M netflow.var.input.udp.port=9996
```

有关配置模块的更多信息，请参阅 [第8章-模块](../08-Working-with-Logstash-Modules/README.md)。

#### 配置项

Netflow模块提供以下设置项来进行配置。这些设置包括Netflow特有的选项，以及所有Logstash模块支持的通用选项。

在命令行覆盖设置时，请记住在设置前添加模块名称，例如 `netflow.var.input.udp.port` 而不是 `var.input.udp.port`。

如果未指定配置设置，Logstash将使用默认值。

##### Netflow 配置项

`var.input.udp.port`

- 值类型是 [Number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)

- 默认值为2055。

  设置Logstash侦听网络流量数据的UDP端口。 虽然2055是此设置的默认设置，但某些设备使用的范围为9995到9998，其中9996是最常用的替代方案。

`var.input.udp.workers`

- 值类型是 [Number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)

- 默认值为2。

  处理数据包的线程数。

`var.input.udp.receive_buffer_bytes`

- 值类型是 [Number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)

- 默认值为212992。

  套接字以字节为单位接收缓冲区大小 如果receive_buffer_bytes大于允许值，操作系统将使用最大允许值。 如果需要增加此最大允许值，请查看操作系统文档。

`var.input.udp.queue_size`

- 值类型是 [Number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)

- 默认值为2000。

  这是在数据包开始丢弃之前可以在内存中保留的未处理UDP数据包的数量。

##### 通用配置项

以下是所有模块都支持的通用配置项。

`var.elasticsearch.hosts`

- 值类型是 [URI](../06-Configuring-Logstash/Structure-of-a-Config-File.md#URI)

- 默认值为 `"localhost:9200"`

  设置Elasticsearch集群的主机。对于每个主机，您必须指定主机名和端口。例如，"myhost:9200"。如果给定一个 [Array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)，Logstash将在hosts参数中指定的主机上加载平衡请求。从主机列表中排除 [专用主节点](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/modules-node.html) 非常重要，以防止Logstash向主节点发送批量请求。因此，此参数应仅引用Elasticsearch中的数据或客户端节点

  这里的URL中出现的任何特殊字符必须是URL转义！这意味着 `＃` 应该以 `％23` 的形式输入。

`var.elasticsearch.username`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值是"elastic"

  向Elasticsearch集群进行身份验证的用户名。

`var.elasticsearch.password`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值为"changeme"

  向Elasticsearch集群进行身份验证的密码。

`var.elasticsearch.ssl.enabled`

- 值类型是 [Boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)

- 此设置没有默认值。

  启用与Elasticsearch集群的SSL / TLS安全通信。保留此未指定将使用在 `hosts` 中列出的URL中指定的任何方案。如果未指定显式协议，则将使用纯HTTP。如果此处明确禁用了SSL，则在主机中提供HTTPS URL时，插件将拒绝启动。

`var.elasticsearch.ssl.verification_mode`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值是"strict"

  与Elasticsearch通信时的主机名验证设置。设置为 `disable` 以关闭主机名验证。禁用此功能存在严重的安全问题。

`var.elasticsearch.ssl.certificate_authority`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 此设置没有默认值

  与Elasticsearch通信时用于验证SSL证书的X.509证书的路径。

`var.elasticsearch.ssl.certificate`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 此设置没有默认值

  与Elasticsearch通信时用于客户端身份验证的X.509证书的路径。

`var.elasticsearch.ssl.key`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 此设置没有默认值

  与Elasticsearch通信时用于客户端身份验证的证书密钥的路径。

`var.kibana.host`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值为 "localhost:5601"

  设置用于导入仪表板和可视化的Kibana实例的主机名和端口。例如： "myhost:5601"。

`var.kibana.scheme`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值为"http"

  设置用于到达Kibana实例的协议。选项包括："http"或"https"。默认值为"http"。

`var.kibana.username`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值是"elastic"

  用于对安全的Kibana实例进行身份验证的用户名。

`var.kibana.password`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值为"changeme"

  用于向安全Kibana实例进行身份验证的密码。

`var.kibana.ssl.enabled`

- 值类型是 [Boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值为false

  启用与Kibana实例的SSL/TLS安全通信。

`var.kibana.ssl.verification_mode`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 默认值是"strict"

  与Kibana通信时的主机名验证设置。设置为禁用以关闭主机名验证。禁用此功能存在严重的安全问题。

`var.kibana.ssl.certificate_authority`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 此设置没有默认值

  与Kibana通信时用于验证SSL证书的X.509证书的路径。

`var.kibana.ssl.certificate`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 此设置没有默认值

  与Kibana通信时用于客户端身份验证的X.509证书的路径。

`var.kibana.ssl.key`

- 值类型是 [String](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)

- 此设置没有默认值

  与Kibana通信时用于客户端身份验证的证书密钥的路径。
