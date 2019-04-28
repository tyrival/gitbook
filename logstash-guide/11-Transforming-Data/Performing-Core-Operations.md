## 核心操作

本节中描述的插件对核心操作很有用，例如变异和删除事件。

[date filter](../19-Filter-plugins/date.md)

解析字段中的日期，以用作事件的Logstash时间戳。

以下配置解析名为 `logdate` 的字段，以设置Logstash时间戳：

```sh
filter {
  date {
    match => [ "logdate", "MMM dd yyyy HH:mm:ss" ]
  }
}
```

[drop filter](../19-Filter-plugins/drop.md)

丢弃事件。该过滤器通常与条件组合使用。

以下配置删除调试级别日志消息：

```sh
filter {
  if [loglevel] == "debug" {
    drop { }
  }
}
```

[fingerprint filter](../19-Filter-plugins/fingerprint.md)

通过应用一致的哈希值使用指纹字段。

以下配置指纹 `IP`，`@timestamp`和 `message` 字段，并将哈希值添加到名为 `generated_id` 的元数据字段中：

```sh
filter {
  fingerprint {
    source => ["IP", "@timestamp", "message"]
    method => "SHA1"
    key => "0123"
    target => "[@metadata][generated_id]"
  }
}
```

[mutate filter](../19-Filter-plugins/mutate.md)

在字段上执行常规操作。您可以重命名、删除、替换和修改事件中的字段。

以下配置将 `HOSTORIP` 字段重命名为 `client_ip`：

```sh
filter {
  mutate {
    rename => { "HOSTORIP" => "client_ip" }
  }
}
```

以下配置从指定字段中删除前导和尾随空格：

```sh
filter {
  mutate {
    strip => ["field1", "field2"]
  }
}
```

[ruby filter](../19-Filter-plugins/ruby.md)

执行Ruby代码。

以下配置执行的Ruby代码取消90％事件：

```sh
filter {
  ruby {
    code => "event.cancel if rand <= 0.90"
  }
}
```
