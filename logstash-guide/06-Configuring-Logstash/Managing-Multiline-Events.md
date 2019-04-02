## 多行事件管理

一些用例生成跨多行文本的事件。为了正确处理这些多行事件，Logstash首先要知道如何判断是否属于多行事件。

多行事件处理很复杂，依赖于正确的事件排序，保证有序日志处理的最佳方法是在管道中尽早处理。

[multiline](../20-Coder-plugins/multiline.md) 编解码器是用于处理Logstash管道中的多行事件的首选工具。多行编解码器使用一组简单的规则合并来自单个输入的行。

> **重要：**
> 如果您使用的是支持多个主机的Logstash输入插件，例如 [beats](../17-Input-plugins/beats.md) 输入插件，则不应使用 [multiline](../20-Coder-plugins/multiline.md) 编解码器来处理多行事件，这样做可能会导致流混合和损坏的事件数据。在这种情况下，您需要在事件数据发送到Logstash之前处理多行事件。

配置多行编解码器的重要事项如下：

- `pattern` 选项指定一个正则表达式，数据行与指定正则表达式匹配，从而判断为前一行的延续或新多行事件的开始。您可以使用 [grok](../19-Filter-plugins/grok.md) 正则表达式模板进行配置。
- `what` 选项有两个可选值：`previous` 或 `next`。`previous` 表示数据行与 `pattern` 选项中的值匹配时，拼接到上一行。`next` 表示数据行与 `pattern` 选项中的值匹配时，拼接到下一行。`negate` 选项表示当数据行与 `pattern` 正则表达式不匹配时，应用多行编解码器。

有关配置选项的更多信息，请参阅 [multiline](../20-Coder-plugins/multiline.md) 编解码器的完整文档。

### 多行编解码器配置示例
本节中的示例包含以下示例：

- 将Java堆栈跟踪组合到单个事件中
- 将C型线延续组合成单一的事件
- 根据时间戳组合多行事件

#### Java栈跟踪

Java堆栈跟踪由多行组成，起始行后面的每一行以空格开头，如下例所示：

```shell
Exception in thread "main" java.lang.NullPointerException
        at com.example.myproject.Book.getTitle(Book.java:16)
        at com.example.myproject.Author.getBookTitles(Author.java:25)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
```

要将这些行合并到Logstash中的单个事件中，请对多行编解码器使用以下配置：

```yaml
input {
  stdin {
    codec => multiline {
      pattern => "^\s"
      what => "previous"
    }
  }
}
```

此配置将空格开头的任何行合并到上一行。

#### 线延续

一些编程语言在行尾使用 `\` 字符来表示该行继续，如下例所示：

```shell
printf ("%10.10ld  \t %10.10ld \t %s\
  %f", w, x, y, z );
```

要将这些行合并到Logstash中的单个事件中，请对多行编解码器使用以下配置：

```yaml
input {
  stdin {
    codec => multiline {
      pattern => "\\$"
      what => "next"
    }
  }
}
```

此配置将以 `\` 字符结尾的任何行与其下一行合并。

#### 时间戳

服务的日志（如Elasticsearch）通常以时间戳开头，接着是特定的活动信息，如在这个例子中：

```shell
[2015-08-24 11:49:14,389][INFO ][env] [Letha] using [1] data paths, mounts [[/
(/dev/disk1)]], net usable_space [34.5gb], net total_space [118.9gb], types [hfs]
```

要将这些行合并到Logstash中的单个事件中，请对多行编解码器使用以下配置：

```yaml
input {
  file {
    path => "/var/log/someapp.log"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601} "
      negate => true
      what => previous
    }
  }
}
```

此配置使用 `negate` 选项指定任何不以时间戳开头的行都属于上一行。