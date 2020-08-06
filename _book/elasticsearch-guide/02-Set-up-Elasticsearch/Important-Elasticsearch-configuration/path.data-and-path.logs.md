## 路径设置

如果您使用`.zip`或`.tar.gz`存档，则`data`和`logs`目录是`$ES_HOME`的子文件夹。 如果这些重要文件夹保留在其默认位置，则在将Elasticsearch升级到新版本时，存在删除它们的高风险。

在生产使用中，您几乎肯定会想要更改数据和日志文件夹的位置：

```yaml
path:
  logs: /var/log/elasticsearch
  data: /var/data/elasticsearch
```

RPM和Debian发行版已经使用`data`和`logs`的自定义路径。

path.data设置可以设置为多个路径，在这种情况下，所有路径都将用于存储数据（尽管属于单个分片的文件将全部存储在同一数据路径中）：

```yaml
path:
  data:
    - /mnt/elasticsearch_1
    - /mnt/elasticsearch_2
    - /mnt/elasticsearch_3
```

