## X-Pack监控

要监视Logstash节点：

1. 确定监控数据的发送位置，通常是生产群集。有关典型监视架构的示例，请参阅 [监视工作原理](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/how-monitoring-works.html)。

> **重要：**
> 要将Logstash可视化为Elastic Stack的一部分（如步骤6所示），请将性能指标发送到生产群集。专用监视集群接收到后，将显示监视集群下的Logstash性能。

2. 确定生产群集上的 `xpack.monitoring.collection.enabled` 是否设置为 `true`。如果该设置为 `false`，则性能监控数据集在Elasticsearch中不可用，并从所有来源中被忽略。
3. 配置Logstash节点，在 `logstash.yml中` 设置 `xpack.monitoring.elasticsearch.hosts` 使其发送性能指标。如果启用了X-Pack安全性，则还需要指定 [内置 `logstash_system` 用户](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/built-in-users.html) 的凭据。有关这些设置的详细信息，请参阅 [监控设置](#监控设置)。

```yaml
xpack.monitoring.elasticsearch.hosts: ["http://es-prod-node-1:9200", "http://es-prod-node-2:9200"]	#1
xpack.monitoring.elasticsearch.username: "logstash_system"	#2
xpack.monitoring.elasticsearch.password: "changeme"
```

​	![img](https://www.elastic.co/guide/en/logstash/6.7/images/icons/callouts/1.png) 如果在生产群集上启用了SSL/TLS，则必须通过HTTPS进行连接。从v5.2.1开始，您可以以数组形式指定多个Elasticsearch主机，或以字符串形式指定单个主机。如果指定了多个URL，则Logstash可以将请求在这些生产节点上进行负载均衡。

​	[![img](https://www.elastic.co/guide/en/logstash/6.7/images/icons/callouts/2.png)](https://www.elastic.co/guide/en/logstash/6.7/configuring-logstash.html#CO3-2) 如果在生产群集上禁用了X-Pack安全性，则可以省略这些 `username` 和 `password` 设置。

4. 如果在生产Elasticsearch集群上启用了SSL / TLS，请指定将用于验证集群中节点标识的可信CA证书。

要将CA证书添加到Logstash节点的可信证书，您可以使用certificate_authority设置指定PEM编码证书的位置：

xpack.monitoring.elasticsearch.ssl.certificate_authority：/path/to/ca.crt
或者，您可以使用信任库（包含证书的Java密钥库文件）配置受信任证书：

xpack.monitoring.elasticsearch.ssl.truststore.path：/ path / to / file
xpack.monitoring.elasticsearch.ssl.truststore.password：密码
此外，您可以选择使用密钥库（包含证书的Java密钥库文件）设置客户端证书：

xpack.monitoring.elasticsearch.ssl.keystore.path：/ path / to / file
xpack.monitoring.elasticsearch.ssl.keystore.password：密码
将sniffing设置为true以启用elasticsearch群集的其他节点的发现。默认为false。

xpack.monitoring.elasticsearch.sniffing：false
重新启动Logstash节点。
要验证X-Pack监视配置，请将Web浏览器指向Kibana主机，然后从侧面导航中选择“监视”。 Logstash部分中应显示Logstash节点报告的度量标准。启用安全性后，要查看监视仪表板，您必须以具有kibana_user和monitoring_user角色的用户身份登录Kibana。

### 监控设置

Upgradingedit后重新启用Logstash监控
从旧版本的X-Pack升级时，出于安全原因，会禁用内置的logstash_system用户。要恢复监视，请更改密码并重新启用logstash_system用户。

监控Logstashedit中的设置
您可以在logstash.yml中设置以下xpack.monitoring设置，以控制如何从Logstash节点收集监视数据。但是，默认情况下在大多数情况下效果最佳。有关配置Logstash的更多信息，请参阅logstash.yml。

一般监测设定
xpack.monitoring.enabled
默认情况下禁用监控。设置为true以启用X-Pack监视。
xpack.monitoring.elasticsearch.hosts
要将Logstash指标发送到的Elasticsearch实例。这可能与Logstash配置的输出部分中指定的Elasticsearch实例相同，也可能是其他实例。这不是专用监控集群的URL。即使您使用的是专用监控集群，也必须通过生产集群路由Logstash指标。您可以将单个主机指定为字符串，也可以将多个主机指定为数组。默认为http：// localhost：9200。
xpack.monitoring.elasticsearch.username和xpack.monitoring.elasticsearch.password
如果您的Elasticsearch受基本身份验证保护，则这些设置会提供Logstash实例用于对传送监视数据进行身份验证的用户名和密码。
监控收集设置
xpack.monitoring.collection.interval
控制在Logstash端收集和发送数据样本的频率。默认为10秒。如果修改收集时间间隔，请将kibana.yml中的xpack.monitoring.min_interval_seconds选项设置为相同的值。
X-Pack监控TLS / SSL Settingsedit
您可以配置以下传输层安全性（TLS）或安全套接字层（SSL）设置