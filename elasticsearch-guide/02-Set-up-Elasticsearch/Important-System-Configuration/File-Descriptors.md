## 文件描述符

> **注意**
>
> 这仅适用于Linux和macOS，如果在Windows上运行Elasticsearch，则可以安全地忽略它。在Windows上，JVM使用仅受可用资源限制的 [API](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx)。

Elasticsearch使用大量文件描述符或文件句柄。文件描述符被用尽可能导致灾难性的结果，最有可能导致数据丢失。确保运行Elasticsearch的用户打开文件描述符数量限制增加到65,536或更高。

对于`.zip`和`.tar.gz`包，在启动Elasticsearch之前将 [`ulimit -n 65535`](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#`ulimit`) 设置为root，或者在 [`/etc/security/limits.conf`](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#`/etc/security/limits_conf`) 中将`nofile`设置为`65535`。

在macOS上，您还必须将JVM选项`-XX:-MaxFDLimit`传递给Elasticsearch，以便它使用更高的文件描述符限制。

RPM和Debian软件包已将文件描述符的最大数量默认为65535，无需进一步配置。

您可以使用 [节点状态API](../../10-Cluster-APIs/Nodes-Stats.md) 检查为每个节点配置的`max_file_descriptor`，其中包括：

```js
GET _nodes/stats/process?filter_path=**.max_file_descriptors
```
