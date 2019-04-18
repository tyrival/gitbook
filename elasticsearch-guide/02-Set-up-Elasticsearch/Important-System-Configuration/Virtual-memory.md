## 虚拟内存

默认情况下，Elasticsearch使用 [mmapfs](../../15-Index-Modules/Store.md#`mmapfs`) 目录来存储其索引。 操作系统的mmap计数的默认限制可能太低，这可能导致内存不足异常。

在Linux上，您可以通过以`root`身份运行以下命令来增加限制：

```sh
sysctl -w vm.max_map_count=262144
```

要永久设置此值，请更新`/etc/sysctl.conf`中的`vm.max_map_count`设置。 要在重启后进行验证，请运行`sysctl vm.max_map_count`。

RPM和Debian软件包将自动配置此设置。 无需进一步配置。