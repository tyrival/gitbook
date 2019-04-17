## Debian包安装

可以从我们的 [网站](#手动下载并安装Debian软件包) 或 [APT存储库](#从APT库安装) 下载Elasticsearch的Debian软件包。它可用于在任何基于Debian的系统（如Debian和Ubuntu）上安装Elasticsearch。

此软件包可在Elastic许可下免费使用。它包含开源和免费商业功能以及付费商业功能。开始 [为期30天的试用](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/license-management.html) ，试用所有付费商业功能。有关Elastic许可级别的信息，请参阅 [订阅](https://www.elastic.co/cn/subscriptions) 页面。

可以在 [Elasticsearch下载](https://www.elastic.co/cn/downloads/elasticsearch) 页面上找到最新的稳定版Elasticsearch。其他版本可在 [历史版本](https://www.elastic.co/downloads/past-releases) 页面上找到。

> **注意**
>
> Elasticsearch需要Java 8或更高版本。使用 [官方Oracle发行版](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 [OpenJDK](http://openjdk.java.net) 等开源发行版。

### 导入PGP Key

我们使用带有指纹的Elasticsearch签名密钥（PGP密钥 [D88E42B4](https://pgp.mit.edu/pks/lookup?op=vindex&search=0xD27D666CD88E42B4)，可从https://pgp.mit.edu获得）对所有软件包进行签名：

```
4609 5ACC 8548 582C 1A26 99A9 D27D 666C D88E 42B4
```

下载并安装公共签名密钥：

```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

### 从APT库安装

在继续之前，您可能需要在Debian上安装`apt-transport-https`软件包：

```sh
sudo apt-get install apt-transport-https
```

将存储库位置保存到`/etc/apt/sources.list.d/elastic-6.x.list`：

```sh
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
```

> **注意**
>
> 这些说明不使用`add-apt-repository`有以下几个原因：
>
> 1. `add-apt-repository`将条目添加到系统`/etc/apt/sources.list`文件中，而不是空白的储存库文件`/etc/apt/sources.list.d`。
> 2. `add-apt-repository`许多发行版上不是默认安装的，需要许多非默认依赖项。
> 3. 较旧版本的`add-apt-repository`总是添加`deb-src`条目，这会导致错误，因为我们不提供源包。如果添加了`deb-src`条目，则在删除`deb-src`行之前，您将看到如下错误：
>
> ```
> Unable to find expected entry 'main/source/Sources' in Release file
> (Wrong sources.list entry or malformed file)
> ```

您可以使用以下命令安装Elasticsearch Debian软件包：

```sh
sudo apt-get update && sudo apt-get install elasticsearch
```

> **警告**
>
> 如果同一Elasticsearch存储库存在两个条目，则在`apt-get update`期间会出现如下错误：
>
> ```text
> Duplicate sources.list entry https://artifacts.elastic.co/packages/6.x/apt/ ...`
> ```
>
> 检查重复条目的`/etc/apt/sources.list.d/elasticsearch-6.x.list`，或在`/etc/apt/sources.list.d/`和`/etc/apt/`中的文件中找到重复的条目`sources.list`文件。

> **注意**
>
> 在systemd-based的发行版中，安装脚本将尝试设置内核参数（例如，`vm.max_map_count`）；您可以通过屏蔽systemd-sysctl.service单元来跳过此步骤。

> **注意**
>
> 还提供了另一个包，其中仅包含Apache 2.0许可下提供的功能。要安装它，请使用以下源列表：
>
> ```sh
> echo "deb https://artifacts.elastic.co/packages/oss-6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
> ```

### 手动下载并安装Debian软件包

可以从网站下载Elasticsearch v6.7.1的Debian软件包，安装如下：

```sh
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.7.1.deb
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.7.1.deb.sha512
shasum -a 512 -c elasticsearch-6.7.1.deb.sha512 ①
sudo dpkg -i elasticsearch-6.7.1.deb
```


![](../../source/images/common/1.png) 比较下载的Debian软件包的SHA和发布的校验码，校验应输出 `elasticsearch-{version}.deb: OK`

或者，您可以下载以下软件包，其中仅包含Apache 2.0许可下提供的功能：

https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-6.7.1.deb

### 启用X-Pack自动创建索引

X-Pack将尝试在Elasticsearch中自动创建多个索引。默认情况下，Elasticsearch配置为允许自动创建索引，不需要其他步骤。但是，如果在Elasticsearch中禁用了自动索引创建，则必须在`elasticsearch.yml`中配置 [`action.auto_create_index`](../../05-Document-APIs/Index-API.md)，以允许X-Pack创建以下索引：

```yaml
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

> **重要**
>
> 如果您使用的是 [Logstash](https://www.elastic.co/products/logstash) 或 [Beats](https://www.elastic.co/products/beats)，则很可能在`action.auto_create_index`设置中需要其他索引名称，具体值取决于您的本地配置。如果您不确定环境的正确值，可以考虑将值设置为`*`，这将允许自动创建所有索引。

### SysV  init与systemd

安装后Elasticsearch不会自动启动。如何启动和停止Elasticsearch取决于您的系统是使用SysV `init`还是`systemd`（较新的发行版使用）。您可以通过运行此命令来判断正在使用哪个：

```sh
ps -p 1
```

### SysV init运行

使用`update-rc.d`命令将Elasticsearch配置为在系统启动时自动启动：

```sh
sudo update-rc.d elasticsearch defaults 95 10
```

可以使用`service`命令启动和停止Elasticsearch：

```sh
sudo -i service elasticsearch start
sudo -i service elasticsearch stop
```

如果Elasticsearch因任何原因无法启动，它将打印STDOUT失败的原因。日志文件可以在`/var/log/elasticsearch/`中找到。

### 使用systemd运行
要将Elasticsearch配置为在系统启动时自动启动，请运行以下命令：

```sh
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

Elasticsearch可以按如下方式启动和停止：

```sh
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

这些命令不提示Elasticsearch是否成功启动。相反，此信息将写入位于`/var/log/elasticsearch/`中的日志文件中。

默认情况下，Elasticsearch服务不会在`systemd`日志中记录信息。要启用`journalctl`日志记录，必须从`elasticsearch.service`文件中的`ExecStart`命令行中删除`--quiet`选项。

启用`systemd`日志记录后，可以使用`journalctl`命令获取日志记录信息：

后面紧跟：

```sh
sudo journalctl -f
```

列出elasticsearch服务的journal：

```sh
sudo journalctl --unit elasticsearch
```

要从给定时间开始列出elasticsearch服务的journalctl：

```sh
sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"
```

查看`man journalctl`或 https://www.freedesktop.org/software/systemd/man/journalctl.html 以获取更多命令行选项。

### 检查运行情况

您可以通过向`localhost`上的端口`9200`发送HTTP请求，来测试您的Elasticsearch节点是否正在运行：

```sh
GET /
```

应该会给你一个像这样的响应：

```json
{
  "name" : "Cp8oag6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "6.7.1",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "f27399d",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.0",
    "minimum_wire_compatibility_version" : "1.2.3",
    "minimum_index_compatibility_version" : "1.2.3"
  },
  "tagline" : "You Know, for Search"
}
```

### 配置

Elasticsearch默认使用`/etc/elasticsearch`进行运行配置。此目录的所有权和此目录中的所有文件都设置为 `root:elasticsearch` ，并且目录设置了`setgid`标志，以便在`/etc/elasticsearch`下创建的所有文件和子目录也使用此所有权创建（例如，如果使用 [密钥库工具](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Secure-settings.md) 创建密钥库）。需对此进行维护，以便Elasticsearch进程可以通过组权限读取此目录下的文件。

默认情况下，Elasticsearch从`/etc/elasticsearch/elasticsearch.yml`文件加载配置。[配置章节](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md) 中介绍了此配置文件的格式。

Debian软件包还有一个系统配置文件（`/etc/default/elasticsearch`），它允许您设置以下参数：

| 参数                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| `JAVA_HOME`          | 设置要使用的自定义Java路径。                                 |
| `MAX_OPEN_FILES`     | 最大打开文件数，默认为`65535`。                              |
| `MAX_LOCKED_MEMORY`  | 最大锁定内存大小。如果您在elasticsearch.yml中使用`bootstrap.memory_lock`选项，则设置为`unlimited`。 |
| `MAX_MAP_COUNT`      | 进程可能具有的最大内存映射区域数。如果使用`mmapfs`作为索引存储类型，请确保将其设置为较高的值。有关更多信息，请查看有关`max_map_count`的 [linux内核文档](https://github.com/torvalds/linux/blob/master/Documentation/sysctl/vm.txt)。这是在启动Elasticsearch之前通过`sysctl`设置的。默认为`262144`。 |
| `ES_PATH_CONF`       | 配置文件目录（需要包含`elasticsearch.yml`，`jvm.options`和`log4j2.properties`文件）；默认为`/etc/elasticsearch`。 |
| `ES_JAVA_OPTS`       | 您可能想要应用的任何其他JVM系统属性。                        |
| `RESTART_ON_UPGRADE` | 在程序包升级时配置重新启动，默认为`false`。这意味着您必须在手动安装软件包后重新启动Elasticsearch实例。这样做的原因是为了确保群集中的升级不会导致连续的分片重新分配，从而导致高网络流量并缩短群集的响应时间。 |

> 注意
> 使用`systemd`的发行版要求通过`systemd`而不是通过`/etc/sysconfig/elasticsearch`文件配置系统资源限制。有关更多信息，请参阅 [Systemd配置](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#Systemd配置)。

### Debian包目录结构

Debian软件包将配置文件，日志和数据目录放在基于Debian的系统的适当位置：

| 文件夹      | 描述                                                         | 默认位置                           | 设置项                                                       |
| ----------- | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------------------------------ |
| **home**    | Elasticsearch根目录或`$ES_HOME`                              | `/usr/share/elasticsearch`         |                                                              |
| **bin**     | 二进制脚本，包含启动节点的`elasticsearch`和安装插件的`elasticsearch-plugin`命令 | `/usr/share/elasticsearch/bin`     |                                                              |
| **conf**    | 配置文件，包括`elasticsearch.yml`                            | `/etc/elasticsearch`               | [`ES_PATH_CONF`](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md#配置文件路径) |
| **conf**    | 环境变量，包含堆大小，文件描述符等                           | `/etc/default/elasticsearch`       |                                                              |
| **data**    | 此节点的数据文件，包含索引、分片，可以容纳多个位置           | `/var/lib/elasticsearch`           | `path.data`                                                  |
| **logs**    | 日志文件夹                                                   | `/var/log/elasticsearch`           | `path.logs`                                                  |
| **plugins** | 插件文件夹，每个插件都包含在其中的子文件夹中                 | `/usr/share/elasticsearch/plugins` |                                                              |
| **repo**    | 共享文件系统存储库位置。 可以容纳多个位置。 文件系统存储库可以放在此处指定的任何目录的任何子目录中。 | 未配置                             | `path.repo`                                                  |

### 下一步

您现在已经设置了测试Elasticsearch环境。在开始认真开发或使用Elasticsearch投入生产之前，您必须进行一些额外的设置：

- 了解如何 [配置Elasticsearch](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md)。
- 配置 [重要的Elasticsearch配置](../../02-Set-up-Elasticsearch/Important-Elasticsearch-configuration.md)。
- 配置 [重要的系统设置](../../02-Set-up-Elasticsearch/Important-System-Configuration.md)。
