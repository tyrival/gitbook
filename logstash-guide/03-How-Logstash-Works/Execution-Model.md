## 运行模型

Logstash事件处理管道协调输入、过滤器和输出的执行。

Logstash管道中的每个输入都在其自己的线程中运行。将事件写入位于内存（默认）或磁盘上的中央队列。每个管道工作线程从该队列中获取一批事件，使其通过配置的过滤器，然后将过滤后的事件传递给所有输出插件。事件批的大小和管道工作线程的数量是可配置的（请参阅 [性能调试和查看](https://github.com/tyrival/logstash-guide/blob/master/chapters/13-Performance-Tuning.md#性能调试和查看)）。

默认情况下，Logstash在管道阶段（输入→若干过滤器→输出）之间使用内存中的有界队列来缓冲事件。如果Logstash不安全地终止，则存储在内存中的事件都将丢失。为防止数据丢失，您可以启用Logstash将正在进行的事件持久化保存到磁盘。 有关更多信息，请参阅 [持久化队列](https://github.com/tyrival/logstash-guide/blob/master/chapters/10-Data-Resiliency.md#持久化队列)。
