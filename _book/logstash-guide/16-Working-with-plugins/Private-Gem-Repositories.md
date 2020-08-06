## 私有Gem仓库

Logstash插件管理器连接到Ruby gems存储库，以安装和更新Logstash插件。默认情况下，此存储库为 http://rubygems.org。

某些场景下无法使用默认存储库，如以下示例所示：

- 防火墙阻止访问默认存储库。
- 您正在本地开发自己的插件。
- 网络物理隔离的要求。

使用自定义gem存储库时，请确保使插件依赖项可用。

几个开源项目使您可以运行自己的插件服务器，其中包括：

- [Geminabox](https://github.com/geminabox/geminabox)
- [Gemirro](https://github.com/PierreRambaud/gemirro)
- [Gemfury](https://gemfury.com)
- [Artifactory](https://jfrog.com/open-source/)

### 编辑Gemfile

gemfile是一个配置文件，用于指定插件管理所需的信息。每个gem文件都有一个 `source` 行，用于指定插件内容的位置。

默认情况下，gemfile的 `source` 行如下：

```shell
＃这是一个Logstash生成的Gemfile。
＃如果手动修改此文件，则所有注释和格式都将丢失。
source "https://rubygems.org"
```

要更改源，请编辑source行以包含首选源，如以下示例所示：

```shell
＃这是一个Logstash生成的Gemfile。
＃如果手动修改此文件，则所有注释和格式都将丢失。
source "https://my.private.repository"
```

保存新版本的gemfile后，通常使用 [插件管理命令](../16-Working-with-plugins/README.md)。

以下链接包含有关设置一些常用存储库的更多资料：

- Geminabox
- [Artifactory](https://www.jfrog.com/confluence/display/RTF/RubyGems+Repositories)
- 运行 [rubygems镜像](https://guides.rubygems.org/run-your-own-gem-server/)