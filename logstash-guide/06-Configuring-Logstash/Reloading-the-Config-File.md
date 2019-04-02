##重载配置文件

从Logstash 2.3开始，您可以设置Logstash自动检测配置更改及重新加载。

要启用配置自动重载，请使用 `—config.reload.automatic`（或 `-r`）命令行选项启动Logstash。例如：

```shell
bin/logstash –f apache.config --config.reload.automatic
```

> **注意：**
> 使用 `-e` 标志以从命令行载入指定配置时，` —config.reload.automatic` 选项不可用。

默认情况下，Logstash每3秒检查一次配置更改。如需更改此间隔，请使用 `--config.reload.interval <interval>` 选项，其中 `<interval>` 指定Logstash定期检查配置更改的间隔时间。

如果Logstash已在未启用自动重载的情况下运行，则可以强制Logstash重新加载配置文件，并通过向运行Logstash的进程发送SIGHUP（信号挂断）来重新启动管道。例如：

```shell
kill -SIGHUP 14175
```

其中14175是运行Logstash的进程的ID。

### 自动重载配置工作原理

当Logstash检测到配置的更改时，它会通过停止所有输入来停止当前管道，并尝试使用更新的配置创建新管道。验证新配置的语法后，Logstash会验证是否可以初始化所有输入和输出（例如，所有必需的端口都已打开）。如果检查成功，Logstash将使用新管道替换现有管道。如果检查失败，则旧管道继续运行，并将错误传播到控制台。

在自动配置重新加载期间，JVM不会重新启动。管道的创建和替换都发生在同一个进程中。

[`grok`](../19-Filter-plugins/grok.md) 过滤器的文件更改也会重新加载，但仅当配置文件中的更改触发重新加载（或重新启动管道）时才会一同被重新加载。
