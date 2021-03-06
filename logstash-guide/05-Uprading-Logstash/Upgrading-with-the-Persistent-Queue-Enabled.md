## 启用持久化队列时的升级

仅当您从启用了持久队列的Logstash 6.2.x或更早版本升级时，以下内容才适用。

我们遗憾地说，由于Logstash 6.2.x及更早版本中的几个序列化问题，在启用持久队列的情况下升级Logstash时，用户将不得不采取一些额外的步骤。虽然我们努力在主要版本中保持向后兼容性，但这些错误要求我们在版本6.3.0中打破兼容性以确保操作的正确性。有关此问题的更多技术细节，请查看我们的跟踪github问题 [#9494](https://github.com/elastic/logstash/issues/9494)。

### 排空持久性队列

如果您正在使用持久性队列，我们强烈建议您在升级之前将其排空或删除。

排空队列：

- 在logstash.yml文件中，设置 `queue.drain:true`。
- 重启Logstash使此设置生效。
- 关闭Logstash（使用CTRL + C或SIGTERM），并等待队列清空。

队列为空时：

- 完成升级。
- 重启Logstash。

我们正在努力解决数据不兼容问题，以便将来升级不需要这些步骤。