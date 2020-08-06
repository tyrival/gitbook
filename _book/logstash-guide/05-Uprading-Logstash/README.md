# 第5章 升级

> **重要：**
>
> 升级Logstash之前：
>
> - 请参阅 [第26章-重大变更](../26-Breaking-Changes/README.md) 文档。
> - 阅读 [发行说明](https://www.elastic.co/guide/en/logstash/6.7/releasenotes.html)。
> - 在生产环境集群升级之前，需在开发环境中进行升级测试。
>
> 升级Logstash时：
>
> - 如果使用X-Pack监视，则必须在Logstash升级时重新使用数据目录。 否则，Logstash节点将分配一个新的持久性UUID，并成为监控数据中的新节点。

如要升级其他产品，请阅读 [Elastic Stack安装和升级指南](https://www.elastic.co/guide/en/elastic-stack/6.7/index.html)。 想要一个适合您的堆栈的升级列表吗？ 试试我们的 [交互式升级指南](https://www.elastic.co/cn/products/upgrade_guide)。

升级Logstash的信息，请参阅以下主题：

- [包管理器升级](../05-Uprading-Logstash/Upgrading-Using-Package-Managers.md)
- [安装包升级](../05-Uprading-Logstash/Upgrading-Using-a-Direct-Download.md)
- [升级到6.0](../05-Uprading-Logstash/Upgrading-Logstash-to-6.0.md)
- [启用持久化队列时的升级](../05-Uprading-Logstash/Upgrading-with-the-Persistent-Queue-Enabled.md)

