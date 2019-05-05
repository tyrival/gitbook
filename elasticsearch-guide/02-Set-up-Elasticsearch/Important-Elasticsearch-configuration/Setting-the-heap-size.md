## 设置堆大小

默认情况下，Elasticsearch告诉JVM堆最小值和最大值为1GB。迁移到生产环境时，配置堆大小以确保Elasticsearch有足够的可用堆是很重要的。

Elasticsearch在 [jvm.options](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Setting-JVM-options.md) 中指定`Xms`（最小堆大小）和`Xmx`（最大堆大小）。

这些设置的值取决于服务器上可用的RAM量。根据经验来说，建议按照如下规则：

- 将最小堆大小（`Xms`）和最大堆大小（`Xmx`）设置为相等。

- Elasticsearch可用的堆越多，它可用于缓存的内存就越多。但请注意，过多的堆可能会使您陷入长时间的GC暂停。

- 将`Xmx`设置为不超过物理RAM的50％，以确保有足够的物理RAM留给内核文件系统缓存。

- `Xmx`不要超过JVM用于压缩对象指针（压缩oops）的截止值；精确的截止值在32GB左右变化。您可以通过查看日志来验证您是否超过限制，如下所示：

  ```
  heap size [1.9gb], compressed ordinary object pointers [true]
  ```

- 更好的是，尽量保持低于零基础压缩oops的阈值；确切的截止值有所不同，但大多数系统上26GB是个安全值，但在某些系统上可能高达30GB。您可以通过使用JVM选项`-XX:+UnlockDiagnosticVMOptions -XX:+PrintCompressedOopsMode`启动Elasticsearch，并查找如下信息来验证您是否在限制之内：

  ```
  heap address: 0x000000011be00000, size: 27648 MB, zero based Compressed Oops
  ```

- 以上表示启用了零基础压缩oops，反之可能为如下消息

  ```
  heap address: 0x0000000118400000, size: 28672 MB, Compressed Oops with base: 0x00000001183ff000
  ```

以下是如何通过jvm.options文件设置堆大小的示例：

```txt
-Xms2g ①
-Xmx2g ②
```


① 将最小堆大小设置为2g。

② 将最大堆大小设置为2g。

也可以通过环境变量设置堆大小。这可以通过注释 [`jvm.options`](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Setting-JVM-options.md) 文件中的`Xms`和`Xmx`设置，并通过`ES_JAVA_OPTS`设置这些值来完成：

```sh
ES_JAVA_OPTS="-Xms2g -Xmx2g" ./bin/elasticsearch ①
ES_JAVA_OPTS="-Xms4000m -Xmx4000m" ./bin/elasticsearch ②
```


① 将最小和最大堆大小设置为2 GB。

② 将最小和最大堆大小设置为4000 MB。

> **注意**
>
> 为 [Windows服务](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-.zip-on-Windows.md#Windows安装Elasticsearch服务) 配置堆不同于上面。最初为Windows服务填充的值可以如上配置，但在安装服务则不同。有关其他详细信息，请参阅 [Windows服务文档](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-.zip-on-Windows.md#Windows安装Elasticsearch服务) 。
