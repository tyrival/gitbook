## 管道间通信（测试功能）

使用Logstash的多管道功能时，您可能希望在一个Logstash实例连接多个管道，并通过配置使这些管道的运行和逻辑相互隔离。管道的输入/输出支持本文档后面讨论的许多高级模式。

如果需要在Logstash实例之间设置通信，请使用 [实例间通信](../06-Configuring-Logstash/Logstash-to-Logstash-Communication.md) 或队列中间件（如Kafka或Redis）。

### 配置概述

同一个Logstash实例中运行的两个管道可以通过管道输入和输出进行连接。这些管道输入使用客户端-服务器的模式，每个输入注册一个虚拟地址，管道输出可以进行连接到此地址。

1. 创建一个下游管道，用来侦听虚拟地址上的事件。
2. 创建一个生成事件的上游管道，通过管道输出将它们发送到一个或多个虚拟地址。

以下是此配置的简单示例：

```yaml
# config/pipelines.yml
- pipeline.id: upstream
  config.string: input { stdin {} } output { pipeline { send_to => [myVirtualAddress] } }
- pipeline.id: downstream
  config.string: input { pipeline { address => myVirtualAddress } }
```

#### 工作原理
管道输入作为虚拟服务器，侦听本地进程中一个虚拟地址。只有在本地同一个Logstash实例上运行的管道输出，才能将事件发送到此地址。管道输出可以将事件发送到虚拟地址列表。如果下游管道被阻塞或不可用，则将阻止管道输出。

通过管道发送事件时，将完全复制其数据。对下游管道中的事件的修改，不会影响任何上游管道中的该事件。

管道插件可能是管道间通信的最有效方式，但它仍然会产生性能消耗。 Logstash必须在Java堆上为每个下游管道完全复制每个事件，使用此功能可能会影响Logstash的堆内存利用率。

#### 交付保证

标准配置中，管道输入/输出至少具有一次交付保证。如果地址被屏蔽或不可用，将阻止输出。

默认情况下，管道输出上的 `ensure_delivery` 选项设置为 `true`。如果将 `ensure_delivery` 标志更改为 `false`，则下游管道不可用时，发送的消息会被丢弃。如果希望禁用下游管道时，使上游管道不至于等待，请使用 `ensure_delivery => false`。

无论 `ensure_delivery` 标志的值如何，阻塞状态的下游管道（不同于下游管道的不可用状态）都会阻塞正在发送事件的管道输出。

这些交付保证还会报告此功能的关闭行为。当管道重新加载时，所有更改将会在用户请求后立刻生效，即使是删除正在接收上游管道事件的下游管道。当然，这将导致上游管道阻塞。您必须还原下游管道，才能正常关闭Logstash。您也可以进行强行停止，此时除非正在使用持久队列，否则未处理完的事件可能会丢失，。

#### 避免循环

连接管道时，请保持数据在一个方向上流动。循环的数据或将管道接入一个循环可能会导致问题。Logstash在关闭之前，会等待每个管道的工作完成，而管道循环会阻止Logstash的正常关闭。

### 建筑模式

您可以使用管道输入和输出来更好地组织代码，简化控制流，并使复杂的配置相互隔离。有无数方法可以连接管道。这里介绍的提供了一些方法。

- [分配器模式](#分配器模式)
- [输出隔离器模式](#输出隔离器模式)
- [分叉路径模式](#分叉路径模式)
- [集合模式](#集合模式)

> **注意：**
> 这些示例使用 `config.string` 来说明流程。您还可以通过配置文件设置管道到管道的通信。

#### 分配器模式
在单个输入接收多种类型的数据来源的情况下，您可以使用分配器模式，每种类型都有自己复杂的处理规则集。使用分配器模式，管道根据类型将数据路由到其他管道。每种类型都路由到一个管道，而这个管道只有处理该类型数据的逻辑。通过这种方式，可以隔离各种类型的处理逻辑。

作为示例，在许多组织中，单个beats输入可用于从各种源接收流量，每个源具有其自己的处理逻辑。处理这种类型数据的常用方法是使用一些 `if` 条件分离流量，并以不同方式处理每种类型。当配置冗长而复杂时，这种方法很容易变得混乱。

这是一个示例分配器模式配置。

```yaml
# config/pipelines.yml
- pipeline.id: beats-server
  config.string: |
    input { beats { port => 5044 } }
    output {
        if [type] == apache {
          pipeline { send_to => weblogs }
        } else if [type] == system {
          pipeline { send_to => syslog }
        } else {
          pipeline { send_to => fallback }
        }
    }
- pipeline.id: weblog-processing
  config.string: |
    input { pipeline { address => weblogs } }
    filter {
       # Weblog filter statements here...
    }
    output {
      elasticsearch { hosts => [es_cluster_a_host] }
    }
- pipeline.id: syslog-processing
  config.string: |
    input { pipeline { address => syslog } }
    filter {
       # Syslog filter statements here...
    }
    output {
      elasticsearch { hosts => [es_cluster_b_host] }
    }
- pipeline.id: fallback-processing
    config.string: |
    input { pipeline { address => fallback } }
    output { elasticsearch { hosts => [es_cluster_b_host] } }
```

请注意，由于每个管道仅适用于单个特定任务，因此数据流是如此简单。

#### 输出隔离器模式

如果多个输出中的一个遇到临时故障，您可以使用输出隔离器模式来防止Logstash被阻塞。 默认情况下，Logstash在任何一个输出下线时都将阻塞，所以这种方式对于保证至少传送一次的数据非常重要。

例如，服务器可能配置成将日志数据同时发送给Elasticsearch和HTTP服务。 由于一些常规原因，HTTP服务可能经常不可用。 在这种场景下，只要HTTP端点关闭，数据就会暂停发送给Elasticsearch。

使用输出隔离器模式和持久化队列后，即使一个输出关闭，我们也可以继续发送数据到Elasticsearch。

以下是使用输出隔离器模式的此方案的示例。

```yaml
# config/pipelines.yml
- pipeline.id: intake
  queue.type: persisted
  config.string: |
    input { beats { port => 5044 } }
    output { pipeline { send_to => [es, http] } }
- pipeline.id: buffered-es
  queue.type: persisted
  config.string: |
    input { pipeline { address => es } }
    output { elasticsearch { } }
- pipeline.id: buffered-http
  queue.type: persisted
  config.string: |
    input { pipeline { address => http } }
    output { http { } }
```

在此体系结构中，每个阶段都有自己的队列，以及自己的调优方案和设置。请注意，此方法使用的磁盘空间最多为三倍，并且产生的序列化/反序列化开销是单个管道的三倍。

####分叉路径模式

当必须根据不同的规则集多次处理单个事件时，可以使用分叉路径模式。在管道输入和输出可用之前，通常通过使用 `clone` 过滤器和 `if/else` 规则来解决此需求。

让我们假设一个用例，我们在自己的系统中接收数据并进行索引，并将编辑后的数据发布到搭配的S3存储桶。 我们可能会使用上面描述的输出隔离器模式来解耦我们对各个系统的写入。分叉路径模式的显着特征是下游管道中存在附加规则。

以下是分叉路径配置的示例。

```yaml
# config/pipelines.yml
- pipeline.id: intake
  queue.type: persisted
  config.string: |
    input { beats { port => 5044 } }
    output { pipeline { send_to => ["internal-es", "partner-s3"] } }
- pipeline.id: buffered-es
  queue.type: persisted
  config.string: |
    input { pipeline { address => "internal-es" } }
    # Index the full event
    output { elasticsearch { } }
- pipeline.id: partner
  queue.type: persisted
  config.string: |
    input { pipeline { address => "partner-s3" } }
    filter {
      # Remove the sensitive data
      mutate { remove_field => 'sensitive-data' }
    }
    output { s3 { } } # Output to partner's bucket
```

####集合模式

如果要为许多不同的管道定义一个通用的输出和预输出过滤器集合，则可以使用集合模式。 这种模式与分销商模式相反，许多管道流入单个管道，在那里它们共享输出和处理。这种模式以减少隔离为代价简化了配置，因为所有数据都是通过单个管道发送的。

这是收集器模式的示例。

```yaml
# config/pipelines.yml
- pipeline.id: beats
  config.string: |
    input { beats { port => 5044 } }
    output { pipeline { send_to => [commonOut] } }
- pipeline.id: kafka
  config.string: |
    input { kafka { ... } }
    output { pipeline { send_to => [commonOut] } }
- pipeline.id: partner
  # This common pipeline enforces the same logic whether data comes from Kafka or Beats
  config.string: |
    input { pipeline { address => commonOut } }
    filter {
      # Always remove sensitive data from all input sources
      mutate { remove_field => 'sensitive-data' }
    }
    output { elasticsearch { } }
```

