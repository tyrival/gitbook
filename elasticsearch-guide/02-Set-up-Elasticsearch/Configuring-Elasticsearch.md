## 配置

Elasticsearch具有良好的默认值，只需要很少的配置。可以使用 [更新集群设置API](../10-Cluster-APIs/Cluster-Update-Settings.md) 在正在运行的集群上更改大多数设置。

配置文件应包含指定节点的设置（例如`node.name`和路径），或为了加入集群而需要的节点设置，例如`cluster.name`和`network.host`。

### 配置文件路径

Elasticsearch有三个配置文件：

- `elasticsearch.yml`用于配置Elasticsearch
- `jvm.options`用于配置Elasticsearch JVM
- `log4j2.properties`用于配置Elasticsearch日志

这些文件位于config目录中，其默认位置取决于安装是来自存档分发（`tar.gz`或`zip`）还是包分发（Debian或RPM软件包）。

对于归档分发，config目录位置默认为`$ES_HOME/config`。可以通过`ES_PATH_CONF`环境变量更改config目录的位置，如下所示：

```sh
ES_PATH_CONF=/path/to/my/config ./bin/elasticsearch
```

或者，您可以通过命令行，或通过shell配置文件`export`环境变量`ES_PATH_CONF` 。

对于发行包，config目录位置默认为`/etc/elasticsearch`。 config目录的位置也可以通过`ES_PATH_CONF`环境变量进行更改，但请注意，在shell中设置它是不够的。相反，此变量来自`/etc/default/elasticsearch`（对于Debian软件包）和`/etc/sysconfig/elasticsearch`（对于RPM软件包）。您需要相应地编辑其中一个文件中的`ES_PATH_CONF=/etc/elasticsearch`条目以更改config目录位置。

### 配置文件格式

配置格式为 [YAML](https://yaml.org)。以下是更改数据路径和日志目录的示例：

```yaml
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```

设置也可以按如下方式平铺：

```yaml
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

### 环境变量引用

在配置文件中用`$ {…}`表示法引用的环境变量，将替换为环境变量的值，例如：

```yaml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

### 提示设置

> **注意**
>
> 不推荐使用提示设置。请对敏感属性值使用 [安全配置](../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Secure-settings.md)。并非所有设置都可以转换为使用 [安全配置](../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Secure-settings.md)。

对于您不希望存储在配置文件中的设置，可以使用值`${prompt.text}`或`${prompt.secret}`，并在前台启动Elasticsearch。 `${prompt.secret}`已禁用回显，因此输入的值不会显示在您的终端中；`${prompt.text}`将允许您在键入时查看该值。例如：

```yaml
node:
  name: ${prompt.text}
```

启动Elasticsearch时，系统会提示您输入实际值，如下所示：

```sh
Enter value for [node.name]:
```

> **注意**
>
> 如果在设置中使用`${prompt.text}`或`${prompt.secret}`，并且该过程作为服务运行或在后台运行，则Elasticsearch将无法启动。
