## 事件API

本节面向插件开发人员和Logstash Ruby过滤器的用户。下面介绍用户在自定义插件和Ruby过滤器中，访问Logstash基于事件的数据的方式，这是最近的升级功能（从5.0版开始）。请注意，[配置中引用事件数据和字段](../06-Configuring-Logstash/Accessing-Event-Data-and-Fields-in-the-Configuration.md) 不受此更改的影响，并将继续使用现有语法。

#### 事件对象
事件，是在Logstash内部封装数据流的主要对象，并为插件开发人员提供API，从而与事件的内容进行交互。通常，此API用于插件和Ruby过滤器中进行数据检索和转换。事件对象包含发送到Logstash的原始数据，以及Logstash过滤阶段创建的所有其他字段。

在5.0中，我们在纯Java中重新实现了事件类及其支持类。由于事件是数据处理中的关键组件，因此Java中的重写可提高性能，并在将数据存储在磁盘上时提供高效的序列化。在大多数情况下，此更改旨在保持向后兼容性并对用户透明。基于此，我们已经更新并发布了Logstash生态系统中的大多数插件，以遵守新的API更改。但是，如果您要维护自定义插件或使用Ruby过滤器，则此更改将对您产生影响。本指南的目的是说明新API，并提供迁移到新更改的示例。

#### 事件API

在5.0版之前，开发人员可以通过直接使用Ruby哈希语法来访问和操作事件数据。例如，`event[field] = foo` 。虽然这很强大，但我们的目标是抽象内部实现细节并提供定义良好的getter和setter API。

##### Get API

Getter是事件中对基于字段的数据的只读访问。

**语法**： `event.get(field)`

**返回**：此字段的值，如果该字段不存在，则返回nil。返回值可以是字符串，数字或时间戳标量值。

`field` 是发送到Logstash或在转换过程之后创建的结构化字段。 `field`也可以是嵌套的字段引用，例如 `[field][bar]` 。

例子：

```ruby
event.get("foo" ) # => "baz"
event.get("[foo]") # => "zab"
event.get("[foo][bar]") # => 1
event.get("[foo][bar]") # => 1.0
event.get("[foo][bar]") # =>  [1, 2, 3]
event.get("[foo][bar]") # => {"a" => 1, "b" => 2}
event.get("[foo][bar]") # =>  {"a" => 1, "b" => 2, "c" => [1, 2]}
```

访问 @metadata

```ruby
event.get("[@metadata][foo]") # => "baz"
```

##### Set API

此API可用于改变事件中的数据。

**语法**： `event.set(field, value)`

**返回**：修改后的事件可用于可链接的调用。

例子：

```ruby
event.set("foo", "baz")
event.set("[foo]", "zab")
event.set("[foo][bar]", 1)
event.set("[foo][bar]", 1.0)
event.set("[foo][bar]", [1, 2, 3])
event.set("[foo][bar]", {"a" => 1, "b" => 2})
event.set("[foo][bar]", {"a" => 1, "b" => 2, "c" => [1, 2]})
event.set("[@metadata][foo]", "baz")
```

不支持在事件中设置后对集合进行变换这种做法。

```ruby
h = {"a" => 1, "b" => 2, "c" => [1, 2]}
event.set("[foo][bar]", h)

h["c"] = [3, 4]
event.get("[foo][bar][c]") # => undefined

Suggested way of mutating collections:

h = {"a" => 1, "b" => 2, "c" => [1, 2]}
event.set("[foo][bar]", h)

h["c"] = [3, 4]
event.set("[foo][bar]", h)

# 或者,
event.set("[foo][bar][c]", [3, 4])
```

#### Ruby过滤器

[Ruby过滤器](../19-Filter-plugins/ruby.md) 可用于执行任何ruby代码，并使用上述API操纵事件数据。例如，使用新API：

```ruby
filter {
  ruby {
    code => 'event.set("lowercase_field", event.get("message").downcase)'
  }
}
```

此过滤器将小写 `message` 字段，并将其设置为名为 `lowercase_field` 的新字段
