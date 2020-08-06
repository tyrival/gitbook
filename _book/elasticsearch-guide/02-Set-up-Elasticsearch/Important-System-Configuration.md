## 重要系统配置

理想情况下，Elasticsearch应该在服务器上单独运行，并使用它可用的所有资源。 为此，您需要配置操作系统以允许运行Elasticsearch的用户访问，这比默认情况下允许的资源更多的资源。

在投入生产之前，必须考虑以下设置：

- [禁用swapping](../02-Set-up-Elasticsearch/Important-System-Configuration/Disable-swapping.md)
- [增加文件描述符](../02-Set-up-Elasticsearch/Important-System-Configuration/File-Descriptors.md)
- [确保足够的虚拟内存](../02-Set-up-Elasticsearch/Important-System-Configuration/Virtual-memory.md)
- [确保足够的线程](../02-Set-up-Elasticsearch/Important-System-Configuration/Number-of-threads.md)
- [JVM DNS缓存设置](../02-Set-up-Elasticsearch/Important-System-Configuration/DNS-cache-settings.md)
- [使用`noexec`不挂载临时目录](../02-Set-up-Elasticsearch/Important-System-Configuration/JNA-temporary-directory-not-mounted-with-noexec.md)

### 开发模式和生产模式

默认情况下，Elasticsearch假定您正在开发模式下工作。 如果未正确配置上述任何设置，则会向日志文件写入警告，但您将能够启动并运行Elasticsearch节点。

一旦配置了`network.host`之类的网络设置，Elasticsearch就会假定您正在转向生产，并将上述警告升级为异常。 这些异常将阻止您的Elasticsearch节点启动。 这是一项重要的安全措施，可确保您不会因服务器配置错误而丢失数据。

