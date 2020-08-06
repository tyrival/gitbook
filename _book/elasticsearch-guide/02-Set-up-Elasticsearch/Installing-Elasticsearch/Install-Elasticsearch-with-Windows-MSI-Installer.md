## Windows MSI安装

> **警告**
>
> 此功能处于测试阶段，可能会发生变化。设计和代码不如官方GA功能成熟，并且按原样提供，不提供任何保证。测试版功能不受官方GA功能支持SLA的约束。

可以使用`.msi`包在Windows上安装Elasticsearch。这可以将Elasticsearch安装为Windows服务，也可以使用附带的`elasticsearch.exe`可执行文件手动运行。

> **提醒**
>
> 以前，Elasticsearch使用 [.zip](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-.zip-on-Windows.md) 存档在Windows安装。如果您愿意，可以继续使用`.zip`方法。

此软件包可在Elastic许可下免费使用。它包含开源和免费商业功能以及付费商业功能。开始 [为期30天的试用](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/license-management.html)，试用所有付费商业功能。有关Elastic许可级别的信息，请参阅 [订阅](https://www.elastic.co/cn/subscriptions) 页面。

可以在 [Elasticsearch下载](https://www.elastic.co/cn/downloads/elasticsearch) 页面上找到最新的稳定版Elasticsearch。其他版本可在 [历史版本](https://www.elastic.co/downloads/past-releases) 页面上找到。

> **注意**
>
> Elasticsearch需要Java 8或更高版本。使用 [官方Oracle发行版](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 或 [OpenJDK](http://openjdk.java.net) 等开源发行版。

### 下载.msi包

从 https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.7.1.msi 下载Elasticsearch v6.7.1的`.msi`包

### 图形界面（GUI）安装

双击下载的`.msi`包以启动GUI向导，该向导将指导您完成安装过程。您可以通过单击`?`按钮显示一个帮助信息面板，面板中包含每个输入的附加信息：

![](../../source/images/ch-02/msi_installer_help.png)

在第一屏选择安装目录。此外，选择将放置数据，日志和配置的目录，或 [使用默认位置](#命令行选项)：

![](../../source/images/ch-02/msi_installer_locations.png)

然后选择Elasticsearch是作为服务安装还是根据需要手动启动。作为服务安装时，您还可以配置Windows帐户以运行服务，无论是在安装后启动服务或随Windows启动：

![](../../source/images/ch-02/msi_installer_service.png)

> **重要**
>
> 选择要运行服务的Windows帐户时，请确保所选帐户具有足够的权限来访问所选的安装和其他部署目录。还要确保帐户能够运行Windows服务。

常见配置项在“配置”章节中，除了内存和网络设置外，还允许设置集群名称，节点名称和角色：

![](../../source/images/ch-02/msi_installer_configuration.png)

安装过程中可以下载和安装的常见插件列表，可选择配置HTTPS代理以下载这些插件。

> **提醒**
>
> 确保安装计算机可以访问Internet，并且所有企业防火墙都配置为允许从`artifacts.elastic.co`下载：

![](../../source/images/ch-02/msi_installer_selected_plugins.png)

从版本6.3.0开始，默认情况下 [捆绑](https://www.elastic.co/cn/products/x-pack/open) X-Pack。除安全配置和内置用户配置外，最后一步还允许选择要安装的许可证类型：

![](../../source/images/ch-02/msi_installer_xpack.png)

> **注意**
>
> X-Pack包括一个试用版或基本许可证。试用许可证有效期为30天，之后您可以获得其中一个 [可用订阅](https://www.elastic.co/cn/subscriptions)。基本许可是免费且永久的。有关哪些许可证可用的功能的详细信息，请参阅可用的订阅。

单击安装按钮后，安装将开始：

![](../../source/images/ch-02/msi_installer_installing.png)

并提示何时成功安装：

![](../../source/images/ch-02/msi_installer_success.png)

### 命令行安装

`.msi`还可以使用命令行安装Elasticsearch。如果使用与GUI安装相同的默认设置，可以先导航到下载目录，然后运行：

```sh
msiexec.exe /i elasticsearch-6.7.1.msi /qn
```

默认情况下，`msiexec.exe`不会等待安装过程完成，因为它在Windows子系统中运行。要等待进程完成并确保相应地设置`％ERRORLEVEL％`，建议使用`start/wait`创建进程并等待它退出

```sh
start /wait msiexec.exe /i elasticsearch-6.7.1.msi /qn
```

与其他MSI安装包一样，可以在`％TEMP％`目录中找到安装过程的日志文件，其中随机生成的名称遵循格式`MSI <random>.LOG`。可以使用`/l`命令行参数提供日志文件的路径

```sh
start /wait msiexec.exe /i elasticsearch-6.7.1.msi /qn /l install.log
```

可以使用查看支持的Windows Installer命令行参数

```sh
msiexec.exe /help
```

或查看 [Windows Installer SDK命令行选项](https://docs.microsoft.com/windows/desktop/Msi/command-line-options)。

### 命令行选项

GUI中公开的所有设置也可以作为命令行参数（在Windows Installer文档中称为属性）提供，可以传递给`msiexec.exe`：
| 参数                               | 说明 |
| ---------------------------------- | ------------------------------------------------------------ |
| `INSTALLDIR`                       | 安装目录。路径中的最终文件夹名必须是Elasticsearch的版本。默认为`%ProgramW6432%\Elastic\Elasticsearch\6.7.1`. |
| `DATADIRECTORY`                    | 存储数据的目录。默认为`%ALLUSERSPROFILE%\Elastic\Elasticsearch\data` |
| `CONFIGDIRECTORY`                  | 存储配置的目录。默认为`%ALLUSERSPROFILE%\Elastic\Elasticsearch\config` |
| `LOGSDIRECTORY`                    | 存储日志的目录。默认为`%ALLUSERSPROFILE%\Elastic\Elasticsearch\logs` |
| `PLACEWRITABLELOCATIONSINSAMEPATH` | 是否应在安装目录下创建数据，配置和日志目录。默认`false` |
| `INSTALLASSERVICE`                 | 是否已安装Elasticsearch并将其配置为Windows服务。默认为`true` |
| `STARTAFTERINSTALL`                | 安装完成后是否启动Windows服务。默认为`true` |
| `STARTWHENWINDOWSSTARTS`           | Windows服务是否在Windows启动时启动。默认为`true` |
| `USELOCALSYSTEM`                   | Windows服务是否在LocalSystem帐户下运行。默认为`true` |
| `USENETWORKSERVICE`                | Windows服务是否在NetworkService帐户下运行。默认为`false` |
| `USEEXISTINGUSER`                  | Windows服务是否在指定的现有帐户下运行。默认为`false` |
| `USER`                             | 运行Windows服务的帐户的用户名。默认为`""` |
| `PASSWORD`                         | 运行Windows服务的帐户的密码。默认为`""` |
| `CLUSTERNAME`                      | 集群的名称。默认为`elasticsearch` |
| `NODENAME`                         | 节点的名称。默认为`%COMPUTERNAME%`  |
| `MASTERNODE`                       | Elasticsearch是否配置为主节点。默认为`true` |
| `DATANODE`                         | Elasticsearch是否配置为数据节点。默认为`true` |
| `INGESTNODE`                       | Elasticsearch是否配置为摄取节点。默认为`true` |
| `SELECTEDMEMORY`                   | 为Elasticsearch分配给JVM堆的内存量。除非目标计算机总共少于4GB，否则默认为`2048`，在这种情况下，它默认为总内存的50％。 |
| `LOCKMEMORY`                       | 是否应该使用`bootstrap.memory_lock`来尝试将进程地址空间锁定到RAM中。默认为`false` |
| `UNICASTNODES`                     | 表单`host:port`或`host`中以逗号分隔的主机列表，用于单播发现。默认为 `""` |
| `MINIMUMMASTERNODES`               | 为了形成集群而必须可见的符合主节点的最小节点数。默认为`""` |
| `NETWORKHOST`                      | 用于将节点绑定到该节点并将其发布（通告）到集群中其他节点的主机名或IP地址。默认为`""` |
| `HTTPPORT`                         | 用于通过HTTP公开Elasticsearch API的端口。默认为`9200` |
| `TRANSPORTPORT`                    | 用于集群内节点之间内部通信的端口。默认为`9300` |
| `PLUGINS`                          | 以逗号分隔的插件列表，作为安装的一部分进行下载和安装。默认为`""` |
| `HTTPSPROXYHOST`                   | 用于通过HTTPS下载插件的代理主机。默认为`""` |
| `HTTPSPROXYPORT`                   | 用于通过HTTPS下载插件的代理端口。默认为`443` |
| `HTTPPROXYHOST`                    | 用于通过HTTP下载插件的代理主机。默认为`""` |
| `HTTPPROXYPORT`                    | 用于通过HTTP下载插件的代理端口。默认为`80` |
| `XPACKLICENSE`                     | 要安装的许可证类型，`Basic`或`Trial`。默认为`Basic` |
| `XPACKSECURITYENABLED`             | 使用`Trial`许可证进行安装时，是否启用了安全功能。默认为`true` |
| `BOOTSTRAPPASSWORD`                | 使用`Trial`许可证进行安装并启用安全功能时，将使用用于引导集群的密码作为密钥库中的`bootstrap.password`设置保留。默认为随机值。 |
| `SKIPSETTINGPASSWORDS`             | 安装`Trial`许可证并启用安全功能时，安装是否应跳过设置内置用户。默认为`false` |
| `ELASTICUSERPASSWORD`              | 使用`Trial`许可证安装并启用安全功能时，用于内置用户`elastic`的密码。默认为`""` |
| `KIBANAUSERPASSWORD`               | 使用`Trial`许可证进行安装并启用安全功能时，将使用用于内置用户`kibana`的密码。默认为`""` |
| `LOGSTASHSYSTEMUSERPASSWORD`       | 使用`Trial`许可证进行安装并启用安全功能时，将使用用于内置用户`logstash_system`的密码。默认为`""` |

要传递值，只需使用格式`<PROPERTYNAME>="<VALUE>"`将参数和值附加到安装命令即可。例如，要将不同的安装目录用于默认目录：

```sh
start /wait msiexec.exe /i elasticsearch-6.7.1.msi /qn INSTALLDIR="C:\Custom Install Directory{version}"
```

有关与包含引号的值相关的其他规则，请参阅 [Windows Installer SDK命令行选项](https://docs.microsoft.com/zh-cn/windows/desktop/Msi/command-line-options)。

### 启用自动创建X-Pack索引

Elastic Stack功能尝试在Elasticsearch中自动创建多个索引。默认情况下，Elasticsearch配置为允许自动创建索引，不需要其他步骤。但是，如果在Elasticsearch中禁用了自动索引创建，则必须在`elasticsearch.yml`中配置`action.auto_create_index`，以允许X-Pack创建以下索引：

```yaml
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

> **重要**
>
> 如果您使用的是 [Logstash](https://www.elastic.co/cn/products/logstash) 或 [Beat](https://www.elastic.co/products/beats)s，则很可能在`action.auto_create_index`设置中需要其他索引名称，具体值取决于您的本地配置。如果您不确定环境的正确值，可以考虑将值设置为`*`，这将允许自动创建所有索引。

### 命令行运行

安装后，如果未作为服务安装并配置为在安装完成时启动，可以从命令行启动Elasticsearch，如下所示：

```sh
.\bin\elasticsearch.exe
```

命令行终端输出如下内容：

![](../../source/images/ch-02/elasticsearch_exe.png)

默认情况下，Elasticsearch在前台运行，除了`LOGSDIRECTORY`中的`<cluster name>.log`文件外，还将其日志打印到S`TDOUT`，可以通过按`Ctrl-C`来停止。

### 命令行配置

默认情况下，Elasticsearch从`%ES_PATH_CONF%\elasticsearch.yml`文件加载其配置。[配置](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md) 中介绍了此配置文件的格式。

也可以在命令行中使用`-E`语法指定可在配置文件中指定的任何设置，如下所示：

```sh
.\bin\elasticsearch.exe -E cluster.name=my_cluster -E node.name=node_1
```

> **注意**
>
> 包含空格的值必须用引号括起来。例如`-E path.logs="C:\My Logs\logs"`

> **提醒**
>
> 通常，应将任何集群通用的设置（如`cluster.name`）添加到`elasticsearch.yml`配置文件中，而可以在命令行上指定任何特定于节点的设置（如`node.name`）。

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

### Windows上安装Elasticsearch服务

Elasticsearch可以作为服务安装，以便在后台运行，也可以在启动时自动启动，无需任何用户交互。使用以下命令行选项安装时可以实现此目的

- `INSTALLASSERVICE=true`
- `STARTAFTERINSTALL=true`
- `STARTWHENWINDOWSSTARTS=true`

安装后，Elasticsearch将显示在服务控制面板中：

![](../../source/images/ch-02/msi_installer_installed_service.png)

可以从控制面板中停止和重新启动，也可以从命令行使用以下命令重新启动和重新启动：

使用命令提示符：

```sh
sc.exe stop Elasticsearch
sc.exe start Elasticsearch
```

使用PowerShell：

```powershell
Get-Service Elasticsearch | Stop-Service
Get-Service Elasticsearch | Start-Service
```

可以对`jvm.options`和`elasticsearch.yml`配置文件进行更改，以便在安装后配置服务。大多数更改（如JVM设置）都需要重新启动服务才能生效。

### 使用图形界面（GUI）升级

`.msi`程序包支持将已安装的Elasticsearch版本升级到较新版本。通过GUI的升级过程处理升级所有已安装的插件，以及保留数据和配置。

下载并双击较新版本的`.msi`包将启动GUI向导。第一步将列出先前安装的只读属性：

![](../../source/images/ch-02/msi_installer_upgrade_notice.png)

下一步允许更改某些配置选项：

![](../../source/images/ch-02/msi_installer_upgrade_configuration.png)

最后，设置插件步骤允许升级或删除当前安装的插件，以及下载和安装当前未安装的插件：

![](../../source/images/ch-02/msi_installer_upgrade_plugins.png)

### 命令行升级

`.msi`还可以使用命令行升级Elasticsearch。

> **重要**
>
> 命令行升级需要传递与第一次安装时相同的命令行属性；Windows Installer不记得这些属性。
>
> 例如，如果最初使用命令行选项`PLUGINS="ingest-geoip"`和`LOCKMEMORY="true"`安装，则在从命令行执行升级时必须传递这些相同的值。
>
> 例外情况是INSTALLDIR参数（如果最初指定的话），它必须是当前安装的不同目录。如果设置INSTALLDIR，则路径中的最终目录必须是Elasticsearch的版本，例如
>
> ```
> C:\Program Files\Elastic\Elasticsearch\6.7.1
> ```

假设使用所有默认值安装Elasticsearch，最简单的升级是首先导航到下载目录，然后运行：

```sh
start /wait msiexec.exe /i elasticsearch-6.7.1.msi /qn
```

与安装过程类似，可以使用/ l命令行参数传递升级过程的日志文件的路径

```sh
start /wait msiexec.exe /i elasticsearch-6.7.1.msi /qn /l upgrade.log
```

### 使用添加/删除程序以卸载
`.msi`包将安装时添加的所有目录和文件进行卸载。

> **警告**
>
> 卸载将删除安装时创建的所有内容，但数据，配置或日志目录除外。建议您在升级之前复制数据目录，或考虑使用快照API。

MSI安装程序包不提供卸载GUI。通过按Windows键并键入添加或删除程序以打开系统设置，可以卸载已安装的程序。

打开后，在已安装的应用程序列表中找到Elasticsearch安装，单击并选择`Uninstall`：

![](../../source/images/ch-02/msi_installer_uninstall.png)

这将启动卸载过程。

### 命令行卸载

也可以通过导航到包含`.msi`包的目录，并运行命令行从命令行执行卸载：

```sh
start /wait msiexec.exe /x elasticsearch-6.7.1.msi /qn
```

与安装过程类似，可以使用`/l`命令行参数传递卸载过程的日志文件的路径

```sh
start /wait msiexec.exe /x elasticsearch-6.7.1.msi /qn /l uninstall.log
```

### 下一步

您现在已经设置了测试Elasticsearch环境。在开始认真开发或使用Elasticsearch投入生产之前，您必须进行一些额外的设置：

- 了解如何 [配置Elasticsearch](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md)。
- 配置 [重要的Elasticsearch配置](../../02-Set-up-Elasticsearch/Important-Elasticsearch-configuration.md)。
- 配置 [重要的系统设置](../../02-Set-up-Elasticsearch/Important-System-Configuration.md)。
