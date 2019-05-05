## .zip或.tar.gz安装

Elasticsearch以`.zip`和`.tar.gz`包的形式提供。这些包可用于在任何系统上安装Elasticsearch，是试用Elasticsearch时最简单的包格式。

此软件包可在Elastic许可下免费使用。它包含开源和免费商业功能以及付费商业功能。开始 [为期30天的试用](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/license-management.html) ，试用所有付费商业功能。有关Elastic许可级别的信息，请参阅 [订阅](https://www.elastic.co/cn/subscriptions) 页面。

可以在 [Elasticsearch下载](https://www.elastic.co/cn/downloads/elasticsearch) 页面上找到最新的稳定版Elasticsearch。其他版本可在 [历史版本](https://www.elastic.co/downloads/past-releases) 页面上找到。

> **注意**
>
> Elasticsearch需要Java 8或更高版本。使用 [官方Oracle发行版](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 [OpenJDK](http://openjdk.java.net) 等开源发行版。

### 下载并安装.zip包

可以按如下方式下载和安装Elasticsearch v6.7.1的`.zip`存档：

```sh
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.7.1.zip
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.7.1.zip.sha512
shasum -a 512 -c elasticsearch-6.7.1.zip.sha512 ①
unzip elasticsearch-6.7.1.zip
cd elasticsearch-6.7.1/ ②
```


① 比较下载的`.zip`的SHA和发布的校验码，该校验和应输出`elasticsearch-{version}.zip: OK`

② 该目录称为`$ES_HOME`

或者，您可以下载以下软件包，其中仅包含Apache 2.0许可下提供的功能：

https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-6.7.1.zip

### 下载并安装.tar.gz包

可以下载和安装Elasticsearch v6.7.1的`.tar.gz`存档，如下所示：

```sh
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.7.1.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.7.1.tar.gz.sha512
shasum -a 512 -c elasticsearch-6.7.1.tar.gz.sha512 ①
tar -xzf elasticsearch-6.7.1.tar.gz
cd elasticsearch-6.7.1/ ②
```


① 比较下载的`.tar.gz`的SHA和发布的校验码，该校验和应输出`elasticsearch-{version}.zip: OK`

② 该目录称为`$ES_HOME`

或者，您可以下载以下软件包，其中仅包含Apache 2.0许可代码：

https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-6.7.1.tar.gz

### 启用X-Pack自动创建索引

X-Pack将尝试在Elasticsearch中自动创建多个索引。默认情况下，Elasticsearch配置为允许自动创建索引，不需要其他步骤。但是，如果在Elasticsearch中禁用了自动索引创建，则必须在`elasticsearch.yml`中配置 [`action.auto_create_index`](../../05-Document-APIs/Index-API.md)，以允许X-Pack创建以下索引：

```yaml
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

> **重要**
>
> 如果您使用的是 [Logstash](https://www.elastic.co/products/logstash) 或 [Beats](https://www.elastic.co/products/beats)，则很可能在`action.auto_create_index`设置中需要其他索引名称，具体值取决于您的本地配置。如果您不确定环境的正确值，可以考虑将值设置为`*`，这将允许自动创建所有索引。

### 命令行运行

可以从命令行启动Elasticsearch，如下所示：

```sh
./bin/elasticsearch
```

默认情况下，Elasticsearch在前台运行，将其日志打印到标准输出（`stdout`），并可以通过按`Ctrl-C`来停止。

> **注意**
>
> 与Elasticsearch一起打包的所有脚本都需要一个支持数组的Bash版本，并假设Bash在`/bin/bash`中可用。因此，Bash应该直接或通过符号链接在此路径上可用。

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

可以使用命令行上的`-q`或`--quiet`选项禁止日志打印到`stdout`。

### 作为守护进程运行

要将Elasticsearch作为守护程序运行，请在命令行中指定`-d`，并使用`-p`选项将进程ID记录在文件中：

```sh
./bin/elasticsearch -d -p pid
```

可以在`$ES_HOME/logs/`目录中找到日志消息。

要关闭Elasticsearch，请终止`pid`文件中记录的进程ID：

```sh
pkill -F pid
```

> **注意**
>
> [RPM](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-RPM.md)和[Debian](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-Debian-Package.md)软件包中提供启动脚本，用于启动和停止Elasticsearch进程。

### .zip和.tar.gz的目录结构

`.zip`和`.tar.gz`包完全是独立的。默认情况下，所有文件和目录都包含在`$ES_HOME`中——解压缩归档时创建的目录。

这非常方便，因为您不必创建任何目录来开始使用Elasticsearch，卸载Elasticsearch就像删除`$ES_HOME`目录一样简单。但是，建议更改config目录，数据目录和logs目录的默认位置，以便以后不删除重要数据。

| 文件夹      | 描述                                                         | 默认位置             | 设置项                                                       |
| ----------- | ------------------------------------------------------------ | -------------------- | ------------------------------------------------------------ |
| **home**    | Elasticsearch根目录或`$ES_HOME`                              | 安装包解压缩到的目录 |                                                              |
| **bin**     | 二进制脚本，包含启动节点的`elasticsearch`和安装插件的`elasticsearch-plugin`命令 | `$ES_HOME/bin`       |                                                              |
| **conf**    | 配置文件，包括`elasticsearch.yml`                            | `$ES_HOME/config`    | [`ES_PATH_CONF`](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md#配置文件路径) |
| **data**    | 此节点的数据文件，包含索引、分片，可以容纳多个位置           | `$ES_HOME/data`      | `path.data`                                                  |
| **logs**    | 日志文件夹                                                   | `$ES_HOME/logs`      | `path.logs`                                                  |
| **plugins** | 插件文件夹，每个插件都包含在其中的子文件夹中                 | `$ES_HOME/plugins`   |                                                              |
| **repo**    | 共享文件系统存储库位置。 可以容纳多个位置。 文件系统存储库可以放在此处指定的任何目录的任何子目录中。 | 未配置               | `path.repo`                                                  |
| **script**  | 脚本文件夹                                                   | `$ES_HOME/scripts`   | `path.scripts`                                               |

### 下一步

您现在已经设置了测试Elasticsearch环境。在开始认真开发或使用Elasticsearch投入生产之前，您必须进行一些额外的设置：

- 了解如何 [配置Elasticsearch](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md)。
- 配置 [重要的Elasticsearch配置](../../02-Set-up-Elasticsearch/Important-Elasticsearch-configuration.md)。
- 配置 [重要的系统设置](../../02-Set-up-Elasticsearch/Important-System-Configuration.md)。
