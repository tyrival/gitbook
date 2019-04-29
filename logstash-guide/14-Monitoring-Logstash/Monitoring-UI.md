## 监控UI

运行Logstash 5.2或更高版本时，您可以使用 [X-Pack监视功能](https://www.elastic.co/cn/products/stack/monitoring) 深入了解有关Logstash部署的指标。在Overview仪表板中，您可以查看Logstash接收和发送的所有事件，以及有关内存使用情况和正常运行时间的信息：

![overviewstats](../source/images/ch-14/overviewstats.png)

然后，您可以深入查看有关特定节点的统计信息：

![nodestats](../source/images/ch-14/nodestats.png)

> **注意：**
> Logstash节点基于其持久性UUID被视为唯一节点，该节点在节点启动时写入 [`path.data`](../04-Setting-Up-and-Running-Logstash/logstash.yml.md) 目录。

在使用监视UI之前，请 [配置Logstash监控](../04-Setting-Up-and-Running-Logstash/Installing-X-Pack.md)。

有关使用监控UI的信息，请参阅 [Kibana中的X-Pack监控](https://www.elastic.co/guide/en/kibana/6.7/xpack-monitoring.html)。
