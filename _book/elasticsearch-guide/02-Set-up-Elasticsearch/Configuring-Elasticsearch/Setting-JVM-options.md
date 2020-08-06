## JVM设置项

您应该很少需要更改Java虚拟机（JVM）选项。如果有过，最可能的更改是 [设置堆大小](../../02-Set-up-Elasticsearch/Important-Elasticsearch-configuration/Setting-the-heap-size.md)。本文档的其余部分详细说明了如何设置JVM选项。

设置JVM选项（包括系统属性和JVM标志）的首选方法是通过`jvm.options`配置文件。此文件的默认位置是`config/jvm.options`（从tar或zip发行版安装时）和`/etc/elasticsearch/jvm.options`（从Debian或RPM软件包安装时）。

此文件包含遵循特殊语法的，以行分隔的JVM参数列表：

- 仅包含空格的行被忽略
- 以`#`开头的行被视为注释并被忽略

```text
# this is a comment
```

- 以`-`开头的行被视为JVM选项，该选项独立于JVM的版本而应用

```text
-Xmx2g
```

- `数字:`开头的行被视为JVM选项，仅当JVM的版本与数字匹配时才适用

```text
8:-Xmx2g
```

- `数字-:`开头的行被视为JVM选项，仅当JVM的版本大于或等于数字时才适用

```text
8-:-Xmx2g
```

- `数字-数字:`开头的行被视为JVM选项，仅当JVM的版本属于这两个数字的范围时才适用

```text
8-9:-Xmx2g
```

- 所有其他行都被拒绝了

您可以将自定义JVM标志添加到此文件，并将此配置检查到版本控制系统中。

设置Java虚拟机选项的另一种机制是通过`ES_JAVA_OPTS`环境变量。例如：

```sh
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djava.io.tmpdir=/path/to/temp/dir"
./bin/elasticsearch
```

使用RPM或Debian软件包时，可以在 [系统配置文件](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#系统配置文件) 中指定`ES_JAVA_OPTS`。

JVM具有监控`JAVA_TOOL_OPTIONS`环境变量的内置机制。我们故意在包装脚本中忽略此环境变量。这样做的主要原因是在某些操作系统（例如，Ubuntu）上，默认情况下通过此环境变量安装了代理，我们不希望干扰Elasticsearch。

此外，一些其他Java程序支持`JAVA_OPTS`环境变量。这不是JVM中内置的机制，而是生态系统中的约定。但是，我们不支持此环境变量，而是支持通过`jvm.options`文件或环境变量`ES_JAVA_OPTS`设置JVM选项，如上所述。