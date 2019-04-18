## 禁用swapping

大多数操作系统尝试使用尽可能多的内存来存储文件系统缓存，并急切地交换（swap）掉未使用的应用程序内存。这可能导致部分JVM堆甚至其可执行页面被换出到磁盘。

交换对性能，节点稳定性非常不利，应该不惜一切代价避免。它可能导致垃圾收集持续数分钟而不是毫秒，并且可能导致节点响应缓慢甚至断开与群集的连接。在弹性分布式系统中，让操作系统终止节点更有效。

有三种禁用交换的方法。首选是完全禁用交换。如果这不是一个选项，是否选择最小化虚拟内存锁定取决于您的环境。

### 禁用所有交换文件

通常Elasticsearch是唯一运行的服务，其内存使用量由JVM选项控制。应该没有必要启用交换。

在Linux系统上，您可以通过运行以下命令暂时禁用交换：

```sh
sudo swapoff -a
```

要永久禁用它，您需要编辑`/etc/fstab`文件并注释掉包含单词`swap`的所有行。

在Windows上，可以在`System Properties → Advanced → Performance → Advanced → Virtual memory`禁用分页文件来实现同样的效果。

### 配置`swappiness`

Linux系统上可用的另一个选项，是确保将sysctl值`vm.swappiness`设置为`1`。这会降低内核交换的倾向，并且在正常情况下不进行交换，同时仍允许整个系统在紧急情况下交换。

### 启用`bootstrap.memory_lock`

另一种选择是在Linux/Unix系统上使用 [mlockall](http://pubs.opengroup.org/onlinepubs/007908799/xsh/mlockall.html)，或在Windows上使用 [VirtualLock](https://docs.microsoft.com/windows/desktop/api/memoryapi/nf-memoryapi-virtuallock)，以尝试将进程地址空间锁定到RAM中，从而防止任何Elasticsearch内存被换出。这可以通过将此行添加到`config/elasticsearch.yml`文件来完成：

```yaml
bootstrap.memory_lock: true
```

> **警告**
>
> 如果尝试分配的内存超过可用内存，`mlockall`可能会导致JVM或shell会话退出！

启动Elasticsearch后，您可以通过检查此请求的输出中的`mlockall`值，来查看是否已成功应用此设置：

```js
GET _nodes?filter_path=**.mlockall
```

如果您看到`mlockall`为`false`，则表示`mlockall`请求失败。您还将在日志中看到包含`Unable to lock JVM Memory`字样的更多信息的行。

在Linux/Unix系统上，最可能的原因是运行Elasticsearch的用户没有锁定内存的权限。这可以授予如下：

**`.zip`和`.tar.gz`**

​	在启动Elasticsearch之前将root设置为 [`ulimit -l unlimited`](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#`ulimit`)，或者在 [`/etc/security/limits.conf`](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#`/etc/security/limits_conf`) 中将`memlock`设置为`unlimited`。

**RPM和Debian**

​	在 [系统配置文件](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md) 中将`MAX_LOCKED_MEMORY`设置为`unlimited`（或者对于使用systemd的系统，请参见下文）。

**使用`systemd`的系统**

​	在 [systemd配置](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#Systemd配置) 中将`LimitMEMLOCK`设置为无穷大。

`mlockall`失败的另一个可能原因是 [使用`noexec`挂载JNA临时目录（通常是`/tmp`的子目录）](../../02-Set-up-Elasticsearch/Important-System-Configuration/JNA-temporary-directory-not-mounted-with-noexec.md)。这可以通过使用`ES_JAVA_OPTS`环境变量为JNA指定新的临时目录来解决：

```sh
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djna.tmpdir=<path>"
./bin/elasticsearch
```

或者在jvm.options配置文件中设置此JVM标志。