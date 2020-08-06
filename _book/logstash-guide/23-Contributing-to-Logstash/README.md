# 第23章-插件开发

在1.5版之前，Logstash每个版本都包含了所有插件。这使得使用任何插件变得容易，但它使插件开发变得复杂 - 如果插件需要修补，则需要新版本的Logstash。从版本1.5开始，所有插件都独立于Logstash核心。现在，您可以更轻松地将自己的输入，编解码器，过滤器或输出插件添加到Logstash！

### 添加插件

由于现在可以独立于Logstash核心开发和部署插件，因此有一些文档可指导您完成编码和部署自己的插件的过程：

- [生成插件](../16-Working-with-plugins/Generating-Plugins.md)
- [开发输入插件](../23-Contributing-to-Logstash/How-to-write-a-Logstash-input-plugin.md)
- [开发编解码器插件](../23-Contributing-to-Logstash/How-to-write-a-Logstash-codec-plugin.md)
- [开发过滤器插件](../23-Contributing-to-Logstash/How-to-write-a-Logstash-filter-plugin.md)
- [开发输出插件](../23-Contributing-to-Logstash/How-to-write-a-Logstash-output-plugin.md)
- [开发插件补丁](../23-Contributing-to-Logstash/Contributing-a-Patch-to-a-Logstash-Plugin.md)
- [插件社区维护指南](../23-Contributing-to-Logstash/Logstash-Plugins-Community-Maintainer-Guide.md)
- [提交插件](../23-Contributing-to-Logstash/Submitting-your-plugin-to-RubyGems)

#### 关闭插件API

从Logstash 2.0开始，我们改变了输入插件关闭的方式，以提高关机可靠性。插件关闭有三种方法：`stop`，`stop?` 和 `close`。

- 从插件线程外部调用 `stop` 方法。此方法表示插件停止。
- 当已调用 `stop` 方法关闭插件后，`stop?` 方法返回 `true`。
- 在插件的 `run` 方法和插件的线程都退出之后，`close` 方法执行最终的书记和清理。`close` 方法是以前版本的Logstash中的 `teardown` 方法。

 `shutdown`、`finished`、`finished?`、`running?`、 `terminating?`  是冗余的，不再出现在插件Base类中。

可以使用插件关闭API的 [示例代码](https://github.com/logstash-plugins/logstash-input-example/blob/master/lib/logstash/inputs/example.rb)。

### 扩展Logstash核心

我们也欢迎Logstash核心功能集的贡献和错误修复。

请仔细阅读我们的 [贡献指南](https://github.com/elastic/logstash/blob/master/CONTRIBUTING.md) 和 [Logstash自述文件](https://github.com/elastic/logstash/blob/master/README.md)。
