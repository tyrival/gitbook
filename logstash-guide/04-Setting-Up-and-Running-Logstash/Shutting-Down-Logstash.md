## 关闭

如果您将Logstash作为服务运行，请使用以下命令之一来停止它：

- systemd

```shell
systemctl stop logstash
```

- upstart

```shell
initctl stop logstash
```

- sysv

```shell
/etc/init.d/logstash stop
```

如果您直接在POSIX系统的控制台中运行Logstash，则可以发送SIGTERM到Logstash进程来停止它。例如：

```shell
kill -TERM {logstash_pid}
```

或者，在控制台中输入 `Ctrl-C`。

### 受控关机期间会发生什么？

当尝试关闭正在运行的Logstash实例时，Logstash会在安全关闭之前执行几个步骤。依次是：

- 停止所有输入、过滤和输出插件
- 处理所有还未处理的事件
- 终止Logstash进程

以下情况会影响关闭过程：

- 输入插件正在以慢速接收数据。
- 一个缓慢的过滤器（如正在执行 `sleep(10000)` 的Ruby过滤器），或正在执行一项非常繁重的查询任务的Elasticsearch过滤器。
- 一个输出插件已断开连接，正在等待重新连接，以输出等待处理的事件。

这些情况使关闭过程的时间消耗和完成时间变得不可预测。

Logstash有一个停顿检测机制，可以在关机期间分析管道和插件的行为。此机制周期性地生成信息，信息内容包括内部队列的待处理事件数量，以及管道工作者线程列表。

如需启用强制终止Logstash功能，请在启动Logstash时使用 `—pipeline.unsafe_shutdown` 参数。

> **警告：**
> 任何不安全关闭，包括Logstash进程的强制终止或Logstash进程的崩溃，可能导致数据丢失（除非已启用Logstash持久队列），所以请尽可能安全地关闭Logstash。

### 失速检测示例

在此示例中，过滤器的缓慢执行会阻止管道彻底关闭。而由于Logstash以 `—pipeline.unsafe_shutdown` 参数启动，关闭会导致丢失20个事件。

```shell
bin/logstash -e 'input { generator { } } filter { ruby { code => "sleep 10000" } }
  output { stdout { codec => dots } }' -w 1 --pipeline.unsafe_shutdown
Pipeline main started
^CSIGINT received. Shutting down the agent. {:level=>:warn}
stopping pipeline {:id=>"main", :level=>:warn}
Received shutdown signal, but pipeline is still waiting for in-flight events
to be processed. Sending another ^C will force quit Logstash, but this may cause
data loss. {:level=>:warn}
{"inflight_count"=>125, "stalling_thread_info"=>{["LogStash::Filters::Ruby",
{"code"=>"sleep 10000"}]=>[{"thread_id"=>19, "name"=>"[main]>worker0",
"current_call"=>"(ruby filter code):1:in `sleep'"}]}} {:level=>:warn}
The shutdown process appears to be stalled due to busy or blocked plugins.
Check the logs for more information. {:level=>:error}
{"inflight_count"=>125, "stalling_thread_info"=>{["LogStash::Filters::Ruby",
{"code"=>"sleep 10000"}]=>[{"thread_id"=>19, "name"=>"[main]>worker0",
"current_call"=>"(ruby filter code):1:in `sleep'"}]}} {:level=>:warn}
{"inflight_count"=>125, "stalling_thread_info"=>{["LogStash::Filters::Ruby",
{"code"=>"sleep 10000"}]=>[{"thread_id"=>19, "name"=>"[main]>worker0",
"current_call"=>"(ruby filter code):1:in `sleep'"}]}} {:level=>:warn}
Forcefully quitting logstash.. {:level=>:fatal}
```

当 `--pipeline.unsafe_shutdown` 未启用时，Logstash会继续运行，并周期性地生成报告。