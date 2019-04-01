## 安装包升级

使用从Elastic官网下载的Logstash二进制文件升级。

- 关闭Logstash管道，停止所有将事件发送到Logstash的输入源。
- 下载与您的主机环境匹配的 [Logstash安装文件](https://www.elastic.co/cn/downloads/logstash)。
- 解压缩安装文件到Logstash目录中。
- 使用 `logstash --config.test_and_exit -f <configuration-file>` 命令测试配置文件。 某些Logstash插件的配置选项在6.x版本中已更改。
- 更新配置文件后重新启动Logstash管道。

