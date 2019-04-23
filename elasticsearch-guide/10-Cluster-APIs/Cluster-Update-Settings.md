## 更新集群设置
使用此API可以查看和更改集群设置。

要查看集群设置：
```sh
GET /_cluster/settings
```

默认情况下，此API调用仅返回已明确定义的设置，但也可以包含默认设置。

对设置的更新可以是持久的，这意味着它们可以在重新启动时应用，也可以在瞬态时应用，在这些情以下是持久更新的示例：
```sh
PUT /_cluster/settings
{
    "persistent" : {
        "indices.recovery.max_bytes_per_sec" : "50mb"
    }
}
```

此更新是暂时的：
```sh
PUT /_cluster/settings?flat_settings=true
{
    "transient" : {
        "indices.recovery.max_bytes_per_sec" : "20mb"
    }
}
```

对更新的响应将返回更改的设置，如对瞬态示例的响应：
```json
{
    ...
    "persistent" : { },
    "transient" : {
        "indices.recovery.max_bytes_per_sec" : "20mb"
    }
}
```

您可以通过指定空值来重置持久或瞬态设置。如果重置瞬态设置，则应用定义的这些值中的第一个：
- 持久的设定
- 配置文件中的设置
- 默认值。

此示例重置设置：
```sh
PUT /_cluster/settings
{
    "transient" : {
        "indices.recovery.max_bytes_per_sec" : null
    }
}
```

响应不包括已重置的设置：
```json
{
    ...
    "persistent" : {},
    "transient" : {}
}
```

您还可以使用通配符重置设置。 例如，要重置所有动态`indices.recovery`设置：
```sh
PUT /_cluster/settings
{
    "transient" : {
        "indices.recovery.*" : null
    }
}
```

### 优先顺序
集群设置的优先顺序是：
- 瞬态集群设置
- 持久集群设置
- `elasticsearch.yml`配置文件中的设置。
最好使用`settings` API设置所有集群设置，并仅将`elasticsearch.yml`文件用于本地配置。 这样，您可以确保所有节点上的设置都相同。 另一方面，如果您使用配置文件不小心在不同节点上定义了不同的设置，会很难发现问题。

您可以在[模块](../14-Modules/README.md)中找到可以动态更新的设置列表。
