# 第4章 配置和运行

阅读本章前，请先阅读 [第2章 快速入门](../02-Getting-Started-with-Logstash/README.md) 来了解基本配置项。

## 目录结构

本节介绍解压缩Logstash安装包时创建的默认目录结构。

### .zip和.tar.gz的目录结构

`.zip`和`.tar.gz`包完全是独立的。 默认情况下，所有文件和目录都包含在主目录中，即解压缩时创建的目录。

这使您很方便的不必创建任何目录来使用Logstash，且卸载Logstash就像删除主目录一样简单。 但是，建议更改数据和日志的默认存储位置，避免在操作Logstash目录时影响重要数据。

| 类型     | 说明                                                         | 默认位置                            | 配置项          |
| -------- | ------------------------------------------------------------ | ----------------------------------- | --------------- |
| home     | Logstash安装的默认位置                                       | `{extract.path}` ，即解压后的根目录 |                 |
| bin      | 二进制码脚本，包括启动Logstash命令 `logstash` 和安装插件命令 `logstash-plugin` | `{extract.path}/bin`                |                 |
| settings | 配置文件，包括 `logstash.yml` 和 `jvm.options`               | `{extract.path}/config`             | `path.settings` |
| logs     | 日志文件                                                     | `{extract.path}/logs`               | `path.logs`     |
| plugins  | 本地的非Ruby-Gem插件文件，都保存在其子目录下。建议仅在开发环境使用。 | `{extract.path}/plugins`            | `path.plugins`  |
| data     | 当数据需要持久化时，Logstash及其插件使用的数据文件           | `{extract.path}/data`               | `path.data`     |

### Debian和RMP包的目录结构

Debian和RPM包将各类文件放在系统中的适当位置。

| 类型     | 说明                                                         | 默认位置                      | 配置项                        |
| -------- | ------------------------------------------------------------ | ----------------------------- | ----------------------------- |
| home     | Logstash安装的默认位置                                       | `/usr/share/logstash`         |                               |
| bin      | 二进制码脚本，包括启动Logstash命令 `logstash` 和安装插件命令 `logstash-plugin` | `/usr/share/logstash/bin`     |                               |
| settings | 配置文件，包括 `logstash.yml` 和 `jvm.options`               | `/etc/logstash`               | `path.settings`               |
| conf     | Logstash管道配置文件                                         | `/etc/logstash/conf.d/*.conf` | `/etc/logstash/pipelines.yml` |
| logs     | 日志文件                                                     | `/var/log/logstash`           | `path.logs`                   |
| plugins  | 本地的非Ruby-Gem插件文件，都保存在其子目录下。建议仅在开发环境使用。 | `/usr/share/logstash/plugins` | `path.plugins`                |
| data     | 当数据需要持久化时，Logstash及其插件使用的数据文件           | `/var/lib/logstash`           | `path.data`                   |

### Docker镜像目录结构

Docker镜像是使用 `.tar.gz` 包创建的。

| 类型     | 说明                                                         | 默认位置                       | 配置项          |
| -------- | ------------------------------------------------------------ | ------------------------------ | --------------- |
| home     | Logstash安装的默认位置                                       | `/usr/share/logstash`          |                 |
| bin      | 二进制码脚本，包括启动Logstash命令 `logstash` 和安装插件命令 `logstash-plugin` | `/usr/share/logstash/bin`      |                 |
| settings | 配置文件，包括 `logstash.yml` 和 `jvm.options`               | `/usr/share/logstash/config`   | `path.settings` |
| conf     | Logstash管道配置文件                                         | `/usr/share/logstash/pipeline` | `path.config`   |
| plugins  | 本地的非Ruby-Gem插件文件，都保存在其子目录下。建议仅在开发环境使用。 | `/usr/share/logstash/plugins`  | `path.plugins`  |
| data     | 当数据需要持久化时，Logstash及其插件使用的数据文件           | `/usr/share/logstash/data`     | `path.data`     |

Logstash的Docker容器默认不生成日志，而使用标准日志输出。



## 配置文件





## logstash.yml





## 密钥库





## 命令行运行





## 系统服务





## Docker





## 日志





## 关闭





