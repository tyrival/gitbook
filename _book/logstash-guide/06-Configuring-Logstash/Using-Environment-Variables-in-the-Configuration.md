## 配置中引用环境变量

### 概览

- 通过使用 `${var}` ，可以在Logstash插件的配置中引用环境变量。
- Logstash启动时，每个环境变量的引用会被其值替换。
- 替换是区分大小写的。
- 对未定义变量的引用会引发Logstash配置错误。
- 可为 `${var:default value}` 提供默认值。 如果未定义环境变量，Logstash将使用默认值。
- 可以在任何类型的插件选项中引用环境变量，类型包括：string、number、boolean、array或hash。
- 环境变量是不可变的。 如果环境变量更新，则必须重新启动Logstash以使用新值。

### 示例
以下示例展示如何用环境变量来设置一些常用配置选项的值。

#### 设置TCP端口
这是一个使用环境变量来设置TCP端口的示例：

```yaml
input {
  tcp {
    port => "${TCP_PORT}"
  }
}
```

现在，让我们设置 `TCP_PORT` ：

```shell
export TCP_PORT=12345
```

Logstash启动时使用的是如下配置：

```yaml
input {
  tcp {
    port => 12345
  }
}
```

如果 `TCP_PORT` 环境变量未设置，Logstash将返回一个配置错误。

此时可以通过指定一个默认值来解决问题：

```yaml
input {
  tcp {
    port => "${TCP_PORT:54321}"
  }
}
```

现在，如果变量未定义，Logstash使用默认值，而不是返回配置错误：

```yaml
input {
  tcp {
    port => 54321
  }
}
```

如果环境变量已定义，Logstash将使用环境变量指定的值，而不是默认值。

#### 设置 Tag 值

下面是一个使用环境变量设置tag值的示例：

```yaml
filter {
  mutate {
    add_tag => [ "tag1", "${ENV_TAG}" ]
  }
}
```

让我们设置 `ENV_TAG` 的值：

```shell
export ENV_TAG="tag2"
```

当Logstash启动时，将使用如下配置：

```yaml
filter {
  mutate {
    add_tag => [ "tag1", "tag2" ]
  }
}
```

### 设置文件路径

下面是一个使用环境变量设置日志文件的路径的示例：

```yaml
filter {
  mutate {
    add_field => {
      "my_path" => "${HOME}/file.log"
    }
  }
}
```

让我们设置 `HOME` 的值：

```shell
export HOME="/path"
```

当Logstash启动时，将使用如下配置：

```yaml
filter {
  mutate {
    add_field => {
      "my_path" => "/path/file.log"
    }
  }
}
```

