## X-Pack设置

X-Pack的设置项在 `elasticsearch.yml` ， `kibana.yml`，以及 `logstash.yml` 配置文件。

| X-Pack功能        | Elasticsearch设置                                            | Kibana设置                                                   | Logstash设置                                                 |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| APM UI            | No                                                           | [Yes](https://www.elastic.co/guide/en/kibana/6.7/apm-settings-kb.html) | No                                                           |
| Development Tools | No                                                           | [Yes](https://www.elastic.co/guide/en/kibana/6.7/dev-settings-kb.html) | No                                                           |
| Graph             | No                                                           | [Yes](https://www.elastic.co/guide/en/kibana/6.7/graph-settings-kb.html) | No                                                           |
| Machine learning  | [Yes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ml-settings.html) | [Yes](https://www.elastic.co/guide/en/kibana/6.7/ml-settings-kb.html) | No                                                           |
| Management        | No                                                           | No                                                           | [Yes](../06-Configuring-Logstash/Configuring-Centralized-Pipeline-Management.md) |
| Monitoring        | [Yes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/monitoring-settings.html) | [Yes](https://www.elastic.co/guide/en/kibana/6.7/monitoring-settings-kb.html) | [Yes](../06-Configuring-Logstash/X-Pack-monitoring.md#监控设置) |
| Reporting         | No                                                           | [Yes](https://www.elastic.co/guide/en/kibana/6.7/reporting-settings-kb.html) | No                                                           |
| Security          | [Yes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/security-settings.html) | [Yes](https://www.elastic.co/guide/en/kibana/6.7/security-settings-kb.html) | No                                                           |
| — Auditing        | [Yes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/auditing-settings.html) | No                                                           | No                                                           |
| Watcher           | [Yes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/notification-settings.html) | No                                                           | No                                                           |

`elasticsearch.yml` 中还有 [X-Pack 协议设置](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/license-settings.html)。

更多Logstash配置项请查询 [logstash.yml](../04-Setting-Up-and-Running-Logstash/logstash.yml.md)

