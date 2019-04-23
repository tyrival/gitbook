## 获取集群设置
集群获取设置API允许检索集群设置。
```sh
GET /_cluster/settings
```

或
```sh
GET /_cluster/settings?include_defaults=true
```

在上面的第二个示例中，参数`include_defaults`确保返回未明确设置的设置。 默认情况下，`include_defaults`设置为`false`。
