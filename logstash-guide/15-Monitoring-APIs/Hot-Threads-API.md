## 热线程API

热线程API获取Logstash的当前热线程。 热线程是一个Java线程，它具有较高的CPU使用率并且执行时间超过正常时间。

```js
curl -XGET 'localhost:9600/_node/hot_threads?pretty'
```

输出是一个JSON文档，其中包含Logstash顶级热线程的细分。

响应示例：

```sh
{
  "hot_threads" : {
    "time" : "2017-06-06T18:25:28-07:00",
    "busiest_threads" : 3,
    "threads" : [ {
      "name" : "Ruby-0-Thread-7",
      "percent_of_cpu_time" : 0.0,
      "state" : "timed_waiting",
      "path" : "/path/to/logstash-6.7.1/vendor/bundle/jruby/1.9/gems/puma-2.16.0-java/lib/puma/thread_pool.rb:187",
      "traces" : [ "java.lang.Object.wait(Native Method)", "org.jruby.RubyThread.sleep(RubyThread.java:1002)", "org.jruby.RubyKernel.sleep(RubyKernel.java:803)" ]
    }, {
      "name" : "[test2]>worker3",
      "percent_of_cpu_time" : 0.85,
      "state" : "waiting",
      "traces" : [ "sun.misc.Unsafe.park(Native Method)", "java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)", "java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:836)" ]
    }, {
      "name" : "[test2]>worker2",
      "percent_of_cpu_time" : 0.85,
      "state" : "runnable",
      "traces" : [ "org.jruby.RubyClass.allocate(RubyClass.java:225)", "org.jruby.RubyClass.newInstance(RubyClass.java:856)", "org.jruby.RubyClass$INVOKER$i$newInstance.call(RubyClass$INVOKER$i$newInstance.gen)" ]
    } ]
  }
}
```

可用参数包括：

| 参数                  | 说明                                                     |
| --------------------- | -------------------------------------------------------- |
| `thread`              | 返回的热线程数量，默认为10                               |
| `human`               | 如果为true，则返回纯文本而不是JSON格式。 默认值为false。 |
| `ignore_idle_threads` | 如果为true，则不返回空闲线程。 默认值为true。            |

Logstash通用的监控API请查看 [通用选项](../16-Working-with-plugins/README.md)。

您可以使用 `?human` 参数来返回一个具有可读性格式的文档。

```sh
curl -XGET 'localhost:9600/_node/hot_threads?human=true'
```

具有可读性的响应示例：

```sh
 ::: {}
 Hot threads at 2017-06-06T18:31:17-07:00, busiestThreads=3:
 ================================================================================
 0.0 % of cpu usage, state: timed_waiting, thread name: 'Ruby-0-Thread-7'
 /path/to/logstash-6.7.1/vendor/bundle/jruby/1.9/gems/puma-2.16.0-java/lib/puma/thread_pool.rb:187
         java.lang.Object.wait(Native Method)
         org.jruby.RubyThread.sleep(RubyThread.java:1002)
         org.jruby.RubyKernel.sleep(RubyKernel.java:803)
 --------------------------------------------------------------------------------
 0.0 % of cpu usage, state: waiting, thread name: 'defaultEventExecutorGroup-5-4'
         sun.misc.Unsafe.park(Native Method)
         java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
         java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
 --------------------------------------------------------------------------------
 0.05 % of cpu usage, state: timed_waiting, thread name: '[test]-pipeline-manager'
         java.lang.Object.wait(Native Method)
         java.lang.Thread.join(Thread.java:1253)
         org.jruby.internal.runtime.NativeThread.join(NativeThread.java:75)
```
