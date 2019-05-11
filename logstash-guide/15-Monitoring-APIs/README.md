# 第15章-API

Logstash提供以下监视API以检索有关Logstash的运行时度量：

- [节点信息API](../15-Monitoring-APIs/Node-Info-API.md)
- [插件信息API](../15-Monitoring-APIs/Plugins-Info-API.md)
- [节点状态API](../15-Monitoring-APIs/Node-Stats-API.md)
- [热线程API](../15-Monitoring-APIs/Hot-Threads-API.md)

您可以使用root资源来检索有关Logstash实例的常规信息，包括主机和版本。

```sh
curl -XGET 'localhost:9600/?pretty'
```

响应示例：

```json
{
   "host": "skywalker",
   "version": "6.7.1",
   "http_address": "127.0.0.1:9600"
}
```

> **注意：**
> 默认情况下，监视API尝试绑定到 `tcp:9600`。如果此端口已被另一个Logstash实例使用，则需要使用指定的 `--http.port` 参数启动Logstash以绑定到其他端口。有关更多信息，请参阅 [命令行参数](../04-Setting-Up-and-Running-Logstash/Running-Logstash-from-the-Command-Line.md)。

### 通用选项

以下选项可应用于所有Logstash监控API。

#### 漂亮的结果

当对任何请求附加 `?pretty=true` 时，返回的JSON将被格式化（仅用于调试！）。

#### 可读的输出

> **注意：**
> 对于Logstash 6.7.1，仅 [热线程API](../15-Monitoring-APIs/Hot-Threads-API.md) 支持 `human` 选项。指定 `human=true` 时，结果将以纯文本而不是JSON格式返回。默认值为false。

统计以适合于人（例如 `"exists_time": "1h"`或 `"size": "1kb"`）和计算机（例如 `"exists_time_in_millis": 3600000` 或 `"size_in_bytes": 1024`）解读的格式返回。可以通过向查询字符串添加 `?human=false` 来关闭人可读的值。当统计结果被监控工具消费时，这是有意义的，而不是用于人类消费。 `human` 标识符的默认值为 `false`。
