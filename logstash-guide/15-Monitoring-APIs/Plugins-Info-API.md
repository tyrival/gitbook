## 插件信息API

插件信息API获取当前安装的所有Logstash插件的信息。 此API返回运行 `bin/logstash-plugin list --verbose` 命令的输出。

```js
curl -XGET 'localhost:9600/_node/plugins?pretty'
```

有关可应用于所有Logstash监控API的选项列表，请参阅 [通用选项](../16-Working-with-plugins/README.md)。

输出是JSON文档。

响应示例：

```js
{
  "total": 93,
  "plugins": [
    {
      "name": "logstash-codec-cef",
      "version": "4.1.2"
    },
    {
      "name": "logstash-codec-collectd",
      "version": "3.0.3"
    },
    {
      "name": "logstash-codec-dots",
      "version": "3.0.2"
    },
    {
      "name": "logstash-codec-edn",
      "version": "3.0.2"
    },
    .
    .
    .
  ]
```

