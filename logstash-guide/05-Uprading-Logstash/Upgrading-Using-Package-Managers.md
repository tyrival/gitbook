## 包管理器升级

使用包管理器升级Logstash的过程如下：

- 关闭Logstash管道，停止所有将事件发送到Logstash的输入源。
- 使用“软件包库”部分中的说明，更新软件包库的链接，使其指向6.x存储库而不是先前版本。
- 根据您的操作系统，运行 `apt-get upgrade logstash` 或 `yum update logstash` 命令。
- 使用 `logstash --config.test_and_exit -f <configuration-file>` 命令测试配置文件。某些Logstash插件的配置项在6.x版本中已更改。
- 更新配置文件后重启Logstash管道。
