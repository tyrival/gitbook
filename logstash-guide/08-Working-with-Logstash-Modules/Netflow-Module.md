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

`--setup` 选项在Elasticsearch中创建 `netflow- *` 索引匹配，并导入Kibana看板和可视化。运行 `--setup` 是一次性设置步骤。后续运行需省略该参数，以避免覆盖现有的Kibana仪表板。

此处的命令假定您已在本地主机上运行Elasticsearch和Kibana。如果不是，则需要指定其他连接选项。请参阅 [配置模块](#配置模块)。

2. 探索数据：

   a. 打开浏览器并导航到 http://localhost:5601。如果启用了安全性，则需要指定在设置安全性时使用的Kibana用户名和密码。

   b. 打开 **Netflow: Network Overview Dashboard**。

   c. 有关数据探索的其他详细信息，请参阅 [探索数据](#探索数据)。

### 探索数据

一旦Logstash Netflow模块开始处理事件，您就可以立即开始使用Kibana看板来探索和可视化您的网络流数据。

您可以按默认样式使用仪表板，也可以根据现有用例和业务需求对其进行定制。

#### 看板示例

在 **Overview** 看板上，您可以查看基本流量数据的摘要，以及设置过滤器，然后再深入了解数据。

Netflow概述仪表板
例如，在"对话伙伴"仪表板上，您可以在任何对话中查看客户端和服务器的源和目标地址。

Netflow对话伙伴仪表板
在流量分析仪表板上，您可以通过查看以字节为单位的流量来识别高容量对话。

Netflow流量分析仪表板
然后，您可以转到"地理位置"仪表板，在该仪表板中可以显示热图上目的地和来源的位置。

Netflow地理位置仪表板

### 配置模块

您可以通过指定logstash.yml设置文件中的设置或覆盖命令行中的设置来进一步优化Logstash Netflow模块的行为。

例如，logstash.yml文件中的以下配置将Logstash设置为侦听端口9996以获取网络流量数据：

模块：
   -  name：netflow

    var.input.udp.port：9996
要在命令行中指定相同的设置，请使用：

bin / logstash --modules netflow -M netflow.var.input.udp.port = 9996
有关配置模块的更多信息，请参阅使用Logstash模块。

配置Optionsedit
Netflow模块提供以下设置以配置模块的行为。这些设置包括特定于Netflow的选项以及所有Logstash模块支持的常用选项。

在命令行覆盖设置时，请记住在设置前添加模块名称，例如netflow.var.input.udp.port而不是var.input.udp.port。

如果未指定配置设置，Logstash将使用默认值。