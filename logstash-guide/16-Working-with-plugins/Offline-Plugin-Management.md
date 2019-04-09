## 离线插件管理

Logstash [插件管理器](../16-Working-with-plugins/README.md) 提供脱机插件管理功能，您可以在无法访问Internet的系统上，使用插件包安装Logstash插件。

此过程需要Logstash服务器访问公共或 [私有Gem仓库](../16-Working-with-plugins/Private-Gem-Repositories.md) 下载并打包所需的所有文件和依赖项，从而脱机安装插件。

### 离线构建插件

脱机插件包是一个压缩文件，其中包含脱机Logstash安装所需的所有插件，以及这些插件的依赖项。

要构建脱机插件包：

- 确保要打包的所有插件都安装在服务器上，并且服务器可以访问Internet。
- 运行 `bin/logstash-plugin prepare-offline-pack` 子命令来打包插件和依赖项：

```shell
bin/logstash-plugin prepare-offline-pack --output OUTPUT [PLUGINS] --overwrite
```

其中：

- `OUTPUT` 指定将写入压缩插件包的zip文件。默认文件是 `/LOGSTASH_HOME/logstash-offline-plugins-6.7.1.zip`。如果您使用的是5.2.x和5.3.0，则此位置应为zip文件，其内容将被覆盖。
- `[PLUGINS]` 指定要包含在包中的一个或多个插件。
- `--overwrite` 指定是否要覆盖该位置的现有文件

例子：

```sh
bin/logstash-plugin prepare-offline-pack logstash-input-beats ①
bin/logstash-plugin prepare-offline-pack logstash-filter-* ②
bin/logstash-plugin prepare-offline-pack logstash-filter-* logstash-input-beats ③
```


![1](../source/images/common/1.png) 打包Beats输入插件和任何依赖项。

![2](../source/images/common/2.png) 使用通配符打包所有过滤器插件和任何依赖项。

![3](../source/images/common/3.png) 打包所有过滤器插件，Beats输入插件和任何依赖项。

> **注意：**
> 下载指定插件的所有依赖项可能需要一些时间，具体取决于列出的插件。

### 离线安装插件

要安装脱机插件包：

1. 将压缩包移动到要安装插件的计算机上。
2. 运行 `bin/logstash-plugin install` 子命令并传入脱机插件包的文件URI。

Windows示例：

```sh
bin/logstash-plugin install file:///c:/path/to/logstash-offline-plugins-6.7.1.zip
```

Linux示例：

```sh
bin/logstash-plugin install file:///path/to/logstash-offline-plugins-6.7.1.zip
```

此命令需要文件URI，因此请确保使用正斜杠并指定包的完整路径。

### 离线更新插件

要脱机更新插件，请更新服务器上的插件，然后使用您生成和安装插件包时相同的方式：

1. 在服务器上，运行 `bin/logstash-plugin update` 子命令以更新插件。请参阅 [更新插件](../16-Working-with-plugins/README.md#更新插件)。
2. 创建新版本的插件包。请参阅 [离线构建插件](#离线构建插件)。
3. 安装新版本的插件包。请参阅 [离线安装插件](#离线安装插件)。