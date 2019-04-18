## 使用noexec不挂载JNA缓存目录

> **注意**
>
> 这仅适用于Linux。

Elasticsearch使用Java Native Access（JNA）库来执行一些与平台相关的本机代码。在Linux上，支持此库的本机代码在运行时从JNA存档中提取。默认情况下，此代码将解压缩到Elasticsearch临时目录，该目录默认为`/tmp`的子目录。或者，可以使用JVM标志`-Djna.tmpdir=<path>`设置此位置。由于本机库作为可执行文件映射到JVM虚拟地址空间，因此不能使用`noexec`挂载提取此代码的位置的基础挂载点，因为这会阻止JVM进程将此代码映射为可执行文件。在某些强化的Linux安装中，这是`/tmp`的默认挂载选项。使用`noexec`底层安装的一个指示是，在启动时，JNA将无法使用`java.lang.UnsatisfiedLinkerError`异常加载，并且 `failed to map segment from shared object`。请注意，异常消息可能因JVM版本而异。此外，依赖于通过JNA执行本机代码的Elasticsearch组件将失败，并显示消息表明它是`because JNA is not available`。如果您看到此类错误消息，则必须重新安装用于JNA的临时目录，以便不使用`noexec`挂载。
