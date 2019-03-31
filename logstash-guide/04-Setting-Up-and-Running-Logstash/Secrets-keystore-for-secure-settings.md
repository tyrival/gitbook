## 密钥库和安全设置

配置Logstash时，可能需要配置一些敏感信息，例如密码。您可以使用Logstash密钥库来安全地存储这些配置信息，而不是依赖文件系统权限来保护它们。

将密钥的键值添加到密钥库后，可以在配置敏感信息时，使用密钥键代替密钥值。

引用密钥的语法与 [使用环境变量](../06-Configuring-Logstash/Using-Environment-Variables-in-the-Configuration.md) 的语法相同：
```yaml
${KEY}
```
其中KEY是密钥的键名。

例如，假设密钥库包含一个键为`ES_PWD`的密钥，其值为`yourelasticsearchpassword`：

- 在配置文件中，使用：`output { elasticsearch {...password => "${ES_PWD}" } } }`
- 在 `logstash.yml` 中，使用：`xpack.management.elasticsearch.password: ${ES_PWD}`

请注意，Logstash密钥库与Elasticsearch密钥库不同，虽然Elasticsearch密钥库允许您按名称存储`elasticsearch.yml`值，但Logstash密钥库允许您指定可在Logstash配置中引用的任意名称。

> **注意：**
> 目前不支持从 `pipelines.yml` 或命令行 (-e) 引用密钥库数据。

> **注意：**
> 如需在 [管道集中管理](../07-Managing-Logstash/Centralized-Pipeline-Management.md) 引用密钥库数据，要求每个Logstash部署都具有密钥库的本地副本。

当Logstash解析设置（`logstash.yml`）或配置（`/etc/logstash/conf.d/*.conf`）时，它会在解析环境变量之前解析密钥库中的密钥。

### 密钥库密码
您可以在环境变量`LOGSTASH_KEYSTORE_PASS`中设置Logstash密钥库的访问密码。如果在设置此变量后创建Logstash密钥库，则密钥库将受密码保护，这说明运行Logstash实例必须能够访问环境变量，同时还需要为所有操作密钥库（添加，列出，删除等）的用户正确设置此环境变量。

用户可以自由选择是否使用密钥库密码，即使未设置密码，数据也会被加密，但是，强烈建议为密钥库设置密码，并对所有包含环境变量值的文件进行权限限制。如果您选择不设置密码，则可以跳过本节的后续部分。

例如设置密码，然后创建密钥库：

```sh
set +o history
export LOGSTASH_KEYSTORE_PASS=mypassword
set -o history
bin/logstash-keystore create
```

此设置要求运行Logstash的用户定义环境变量 `LOGSTASH_KEYSTORE_PASS=mypassword` 。如果未定义该环境变量，则Logstash无法访问密钥库。

从RPM或DEB程序包安装运行Logstash时，环境变量储存于 `/etc/sysconfig/logstash` 中。

> **注意：**
> 您可能需要创建 `/etc/sysconfig/logstash`。此文件的拥有者为root用户，其中包含600个权限。`/etc/sysconfig/logstash` 的格式为 `ENVIRONMENT_VARIABLE=VALUE`，每行设置条目。

对于其他发行版，例如Docker或ZIP，请参阅运行时环境的文档（Windows，Docker等），以了解如何为运行Logstash的用户设置环境变量。确保只有该用户可以访问环境变量（以及密码）。

### 密钥库位置
密钥库与 `logstash.yml` 文件同样位于Logstash的 `path.settings` 目录中。对密钥库执行任何操作时，建议为keystore命令设置 `path.settings`。例如，要在RPM / DEB安装上创建密钥库：

```sh
set +o history
export LOGSTASH_KEYSTORE_PASS=mypassword
set -o history
sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash create
```

有关默认目录位置的详细信息，请参阅 [目录结构](../04-Setting-Up-and-Running-Logstash/Logstash-Directory-Layout.md)。

> **注意：**
> 如果 `path.settings` 未指向与 `logstash.yml` 相同的目录，您将看到警告。

### 创建密钥库

使用 `create` 命令创建密钥库：

```shell
bin/logstash-keystore create
```

密钥库创建在 `path.settings` 配置的目录下。

> **注意：**
>
> 建议创建密钥库时设置一个 [密钥库密码](../04-Setting-Up-and-Running-Logstash/Secrets-keystore-for-secure-settings.md#密钥库密码)

### 增加密钥

要存储敏感值，例如Elasticsearch的身份验证凭据，请使用 `add` 命令：

```shell
bin/logstash-keystore add ES_PWD
```

出现提示后，输入此键对应的值。

### 列出密钥

要列出所有密钥库中的密钥，如下：

```shell
bin/logstash-keystore list
```

### 移除密钥

要移除密钥库的密钥，如下：

```shell
bin/logstash-keystore remove ES_PWD
```

