## 提取字段和调整数据

本节中描述的插件对于提取字段和将非结构化数据解析为字段非常有用。

[dissect filter](../19-Filter-plugins/dissect.md)

使用分隔符将非结构化事件数据提取到字段中。剖析过滤器不使用正则表达式，速度非常快。但是，如果数据结构因线而异，则grok滤波器更合适。

例如，假设您有一个包含以下消息的日志：

```sh
Apr 26 12:20:02 localhost systemd[1]: Starting system activity accounting tool...
```

以下配置剖析了消息：

```sh
filter {
  dissect {
    mapping => { "message" => "%{ts} %{+ts} %{+ts} %{src} %{prog}[%{pid}]: %{msg}" }
  }
}
```

应用dissect过滤器后，事件将被分解到以下字段中：

```sh
{
  "msg"        => "Starting system activity accounting tool...",
  "@timestamp" => 2017-04-26T19:33:39.257Z,
  "src"        => "localhost",
  "@version"   => "1",
  "host"       => "localhost.localdomain",
  "pid"        => "1",
  "message"    => "Apr 26 12:20:02 localhost systemd[1]: Starting system activity accounting tool...",
  "type"       => "stdin",
  "prog"       => "systemd",
  "ts"         => "Apr 26 12:20:02"
}
```

[kv filter](../19-Filter-plugins/kv.md)

解析键值对。

例如，假设您有一条包含以下键值对的日志消息：

```sh
ip=1.2.3.4 error=REFUSED
```

以下配置将键值对解析为字段：

```sh
filter {
  kv { }
}
```

应用过滤器后，示例中的事件将包含以下字段：

- `ip: 1.2.3.4`
- `error: REFUSED`

[grok filter](../19-Filter-plugins/grok.md)

将非结构化事件数据解析为字段。此工具非常适合系统日志，Apache和其他Web服务器日志，MySQL日志，以及通常为人类而不是计算机使用而编写的任何日志格式。 Grok的工作原理是将文本模式组合成与日志匹配的内容。

例如，假设您有一个包含以下消息的HTTP请求日志：

```sh
55.3.244.1 GET /index.html 15824 0.043
```

以下配置将消息解析为字段：

```sh
filter {
  grok {
    match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
  }
}
```

应用过滤器后，示例中的事件将包含以下字段：

- `client: 55.3.244.1`
- `method: GET`
- `request: /index.html`
- `bytes: 15824`
- `duration: 0.043`
