## 安全配置

某些设置是敏感的，依靠文件系统权限来保护其值是不够的。对于此场景，Elasticsearch提供了一个密钥库和`elasticsearch-keystore`工具来管理密钥库中的设置。

> **注意**
>
> 此处的所有命令都应使用运行Elasticsearch的用户来运行。

> **注意**
>
> 只有一些设置可以从密钥库中读取。请参阅每个设置的文档，以查看它是否作为密钥库的一部分受支持。

> **注意**
>
> 只有在重新启动Elasticsearch之后，对密钥库的所有修改才会生效。

> **注意**
>
> elasticsearch密钥库目前仅提供模糊处理。将来，将增加密码保护。

这些设置与`elasticsearch.yml`中的常规设置一样，需要在集群中的每个节点上指定。目前，所有安全设置都是特定于节点的设置，每个节点上的值必须相同。

### 创建密钥库

要创建`elasticsearch.keystore`，请使用create命令：

```sh
bin/elasticsearch-keystore create
```

文件`elasticsearch.keystore`将与`elasticsearch.yml`一起创建。

### 列出密钥库的设置

使用`list`命令可以获得密钥库中的设置列表：

```sh
bin/elasticsearch-keystore list
```

### 添加字符串设置

可以使用`add`命令添加敏感字符串设置，例如云插件的身份验证凭据：

```sh
bin/elasticsearch-keystore add the.setting.name.to.set
```

该工具将提示设置的值。要通过stdin传递值，请使用`--stdin`标志：

```sh
cat /file/containing/setting/value | bin/elasticsearch-keystore add --stdin the.setting.name.to.set
```

### 添加文件设置

您可以使用`add-file`命令添加敏感文件，例如云插件的身份验证密钥文件。确保在设置名称后包含文件路径作为参数。

```sh
bin/elasticsearch-keystore add-file the.setting.name.to.set /path/example-file.json
```

### 删除设置

要从密钥库中删除设置，请使用`remove`命令：

```sh
bin/elasticsearch-keystore remove the.setting.name.to.remove
```

### 重载安全设置

与`elasticsearch.yml`中的设置值一样，对密钥库内容的更改不会自动应用于正在运行的elasticsearch节点。重新读取设置需要重新启动节点。但是，某些安全设置被标记为可重新加载。可以重新读取这些设置并将其应用于正在运行的节点上。

所有安全设置的值（可重新加载或不可重载）在所有集群节点上必须相同。在进行所需的安全设置更改后，使用`bin/elasticsearch-keystore add`命令，调用：

```sh
POST _nodes/reload_secure_settings
```

此API将在每个集群节点上解密并重新读取整个密钥库，但仅应用可重新加载的安全设置。对其他设置的更改将在下次重新启动后生效。一旦调用返回，重载就已完成，这意味着所有依赖于这些设置的内部数据结构都已更改。一切看起来好像设置从一开始就具有新值。

更改多个可重新加载的安全设置时，请在集群的所有节点上修改完设置，然后发出`reload_secure_settings`调用，而不是在每个节点的修改后单独重新加载。
