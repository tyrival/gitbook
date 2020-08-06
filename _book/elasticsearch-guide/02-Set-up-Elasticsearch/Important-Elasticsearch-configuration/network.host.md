## 网络主机

默认情况下，Elasticsearch仅绑定到环回地址 - 例如`127.0.0.1`和`[::1]`。这足以在服务器上运行单个开发节点。

> **提醒**
>
> 实际上，可以从单个节点上的同一`$ES_HOME`位置启动多个节点。这对于测试Elasticsearch形成集群的能力非常有用，但它不推荐用于生产环境。

为了在其他服务器上形成包含节点的集群，您的节点将需要绑定到非环回地址。虽然有许多 [网络设置](../../14-Modules/Network-Settings.md)，但通常您需要配置的是`network.host`：

```yaml
network.host: 192.168.1.10
```

`network.host`还可以接受一些特殊值，例如`_local_`，`_ site_`，`_global_`和修饰符，如`:ip4`和`:ip6`，其详细信息可在 [`network.host`的特殊值](../../14-Modules/Network-Settings.md#network_host的特殊值)中找到。

> **重要**
>
> 只要为`network.host`提供自定义设置，Elasticsearch就会假定您从开发模式转移到生产模式，并将许多系统启动检查从警告升级到异常。有关更多信息，请参阅 [开发模式和生产模式](../../02-Set-up-Elasticsearch/Important-System-Configuration.md#开发模式和生产模式)。

