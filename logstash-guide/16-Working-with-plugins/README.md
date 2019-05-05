# 第16章-插件

Logstash拥有丰富的输入、过滤器、编解码器和输出插件。插件可作为独立的gems包托管在RubyGems.org上。通过 `bin/logstash-plugin` 脚本访问的插件管理器，来管理插件的生命周期。您可以使用下面命令行接口（CLI）来安装、删除和升级插件。

### 代理配置

大多数插件管理器命令都需要访问互联网上的 [RubyGems.org](https://rubygems.org)。如果您的服务器位于防火墙后面，则可以设置这些环境变量，从而配置Logstash使用您的代理。

```shell
export http_proxy=http://localhost:3128
export https_proxy=http://localhost:3128
```

### 插件清单

Logstash发行包捆绑了常见的插件，因此您可以直接使用它们。部署中当前可用的插件包括：

```shell
bin/logstash-plugin list ①
bin/logstash-plugin list --verbose ②
bin/logstash-plugin list '*namefragment*' ③
bin/logstash-plugin list --group output ④
```


① 将列出所有已安装的插件

② 将列出已安装的插件及版本信息

③ 将列出包含namefragment的所有已安装插件

④ 将列出特定组的所有已安装插件（输入，过滤器，编解码器，输出）

### 安装插件

通常情况下，您可以通过访问互联网来安装插件。此时，您可以在公共存储库（RubyGems.org）上查询插件，并在Logstash中进行安装。

```shell
bin/logstash-plugin install logstash-output-kafka
```

成功安装插件后，您可以在配置文件中开始使用它。

#### 高级：从本地安装插件

在某些情况下，您希望安装尚未发布或未托管在RubyGems.org上的插件。 Logstash为您提供了从本地安装插件的方式，该插件打包为ruby gem。使用文件位置：

```shell
bin/logstash-plugin install /path/to/logstash-output-kafka-1.0.0.gem
```

#### 高级：使用 `--path.plugins`

使用Logstash `--path.plugins` 参数，您可以加载文件系统上的插件源代码。通常，这是在迭代自定义插件，并希望在创建ruby gem之前测试它的开发人员使用的。

路径需要位于特定的目录层次结构中：`PATH/logstash/TYPE/NAME.rb`，其中TYPE是输入过滤器、输出或编解码器，NAME是插件的名称。

```shell
＃假设代码位于 /opt/shared/lib/logstash/inputs/my-custom-plugin-code.rb
bin/logstash --path.plugins /opt/shared/lib
```

### 更新插件

插件有自己的发布周期，通常独立于Logstash的核心发布周期发布。 使用update子命令可以获得最新版本的插件。

```shell
bin/logstash-plugin update ①
bin/logstash-plugin update logstash-output-kafka ②
```


① 将更新所有已安装的插件

② 将仅更新此插件

#### 删除插件

如果您需要从Logstash安装中删除插件：

```shell
bin/logstash-plugin remove logstash-output-kafka
```

### 代理支持

前面的部分依赖于Logstash能够与RubyGems.org进行通信。 在某些环境中，转发代理用于处理HTTP请求。 通过设置 `HTTP_PROXY` 环境变量，可以通过代理安装和更新Logstash插件：

```shell
export HTTP_PROXY=http://127.0.0.1:3128

bin/logstash-plugin install logstash-output-kafka
```

一旦设置，安装、更新等插件命令可以通过此代理使用。
