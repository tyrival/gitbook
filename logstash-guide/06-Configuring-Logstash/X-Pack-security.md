## X-Pack安全性

Logstash Elasticsearch插件（[输出插件Output](../18-Output-plugins/README.md)，[输入插件Input](../17-Input-plugins/README.md)，[过滤器Filter](../19-Filter-plugins/README.md) 和 [监控](../14-Monitoring-Logstash/README.md)）支持HTTP上的身份验证和加密。

要将Logstash集群进行安全设置，您需要为Logstash配置身份验证凭据。如果身份验证失败，Logstash会抛出异常并停止处理管道。

如果在群集上启用了加密，则还需要在Logstash配置中启用TLS / SSL。

如果要使用X-Pack来监控Logstash实例，并将监控数据存储在安全的Elasticsearch集群中，则必须使用具有相应权限的用户的用户名和密码配置Logstash。

除了为Logstash配置身份验证凭据之外，还需要授予用户访问Logstash索引的权限。

#### 配置基础验证

Logstash要能管理索引模板，创建索引，以及在它创建的索引中写入和删除文档。

为Logstash设置身份验证凭据：

1. 使用Kibana中的 **Management > Roles** 页面或 `role` API创建 `logstash_writer` 角色。对于**群集**权限，请添加 `manage_index_templates` 和 `monitor`。对于**索引**权限，添加 `write`，`delete` 和 `create_index`。

   如果您计划使用 [索引生命周期管理](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/getting-started-index-lifecycle-management.html)，还可以为集群添加 `manage_ilm`，为索引添加 `manage` 和 `manage_ilm`。

```shell
POST _xpack/security/role/logstash_writer
{
  "cluster": ["manage_index_templates", "monitor", "manage_ilm"], ①
  "indices": [
    {
      "names": [ "logstash-*" ], ②
      "privileges": ["write","delete","create_index","manage","manage_ilm"]  ③
    }
  ]
}
```

![1](../source/images/common/1.png) 如果启用了 [索引生命周期管理](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/getting-started-index-lifecycle-management.html)，则群集需要 `manage_ilm` 权限。

![2](../source/images/common/2.png) 如果使用自定义Logstash索引匹配，请自定义匹配而不是默认 `logstash-*` 匹配。

![3](../source/images/common/3.png) 如果启用了 [索引生命周期管理](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/getting-started-index-lifecycle-management.html)，则该角色需要 `manage` 和 `manage_ilm` 权限才能加载索引生命周期策略，创建滚动别名以及创建和管理滚动索引。

2. 创建 `logstash_internal` 用户并为其分配 `logstash_writer` 角色。您可以从Kibana中的 **Management > Users** 页面或通过 `user` API创建用户：

```shell
POST _xpack/security/user/logstash_internal
{
  "password" : "x-pack-test-password",
  "roles" : [ "logstash_writer"],
  "full_name" : "Internal Logstash User"
}
```

3. 配置Logstash，从而为刚创建的 `logstash_internal` 用户进行身份验证。您可以为 `Logstash .conf` 文件中的每个Elasticsearch插件单独配置凭据。例如：

```yaml
input {
  elasticsearch {
    ...
    user => logstash_internal
    password => x-pack-test-password
  }
}
filter {
  elasticsearch {
    ...
    user => logstash_internal
    password => x-pack-test-password
  }
}
output {
  elasticsearch {
    ...
    user => logstash_internal
    password => x-pack-test-password
  }
}
```

#### 授权用户访问Logstash索引

要访问Logstash创建的索引，用户需要 `read`和 `view_index_metadata` 权限：

1. 创建 `logstash_reader` 角色，该角色具有Logstash索引的 `read` 和 `view_index_metadata` 特权。您可以从Kibana中的 **Management > Roles** 页面或通过 `role` API创建角色：

```shell
POST _xpack/security/role/logstash_reader
{
  "indices": [
    {
      "names": [ "logstash-*" ], ①
      "privileges": ["read","view_index_metadata"]
    }
  ]
}
```


![1](../source/images/common/1.png) 如果使用自定义Logstash索引匹配，请指定该匹配而不是默认的 `logstash- *` 匹配。

2. 为Logstash用户分配 `logstash_reader` 角色。如果Logstash用户需使用 [集中管道管理](../07-Managing-Logstash/Centralized-Pipeline-Management.md)，还要分配 `logstash_admin` 角色。您可以从Kibana中的 **Management > Users** 页面或通过 `user` API创建和管理用户：

```shell
POST _xpack/security/user/logstash_user
{
  "password" : "x-pack-test-password",
  "roles" : [ "logstash_reader", "logstash_admin"], ①
  "full_name" : "Kibana User for Logstash"
}
```


![1](../source/images/common/1.png) `logstash_admin` 是一个内置角色，可以访问 `.logstash- *` 索引以管理配置。

#### 配置Elasticsearch 输出使用PKI验证

`elasticsearch` 输出支持PKI身份验证。要使用X.509客户端证书进行身份验证，请在Logstash的 `.conf` 文件中配置 `keystore` 和 `keystore_password` 选项：

```yaml
output {
  elasticsearch {
    ...
    keystore => /path/to/keystore.jks
    keystore_password => realpassword
    truststore =>  /path/to/truststore.jks ①
    truststore_password =>  realpassword
  }
}
```


![1](../source/images/common/1.png) 如果使用单独的信任库，则还需要信任库路径和密码。

#### 配置Logstash使用TLS加密

如果在Elasticsearch集群上启用了TLS加密，则需要在Logstash的 `.conf` 文件中配置 `ssl` 和 `cacert` 选项：

```yaml
output {
  elasticsearch {
    ...
    ssl => true
    cacert => '/path/to/cert.pem' ①
  }
}
```


![1](../source/images/common/1.png) 包含证书颁发机构证书的本地 `.pem` 文件的路径。

### 配置监控凭据

如果您打算发送Logstash [监控](../14-Monitoring-Logstash/README.md) 数据到一个安全集群，则需要配置Logstash用来发送监控数据时用来验证用户名和密码。

X-Pack安全预置了 [`logstash_system` 内置用户](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/built-in-users.html) 。此用户具有监视功能所需的最低权限，不应用于任何其他目的—— 它特别不适用于Logstash管道。

默认情况下，`logstash_system` 用户没有密码。在设置密码之前，该用户不会被启用。通过更改密码API可设置该用户的密码：

```shell
PUT _xpack/security/user/logstash_system/_password
{
  "password": "t0p.s3cr3t"
}
```

然后在 `logstash.yml` 中配置用户和密码：

```yaml
xpack.monitoring.elasticsearch.username: logstash_system
xpack.monitoring.elasticsearch.password: t0p.s3cr3t
```

如果您最初安装了旧版本的X-Pack，然后进行了升级，出于安全原因，`logstash_system` 用户可能已默认为 `disabled`。您可以通过 `user` API启用用户：

```shell
PUT _xpack/security/user/logstash_system/_enable
```

#### 为集中管道管理配置凭据

如果您计划使用Logstash [集中管道管理](../07-Managing-Logstash/Centralized-Pipeline-Management.md)，则需要配置Logstash用于管理配置的用户名和密码。

您可以在 `logstash.yml` 配置文件中配置用户和密码：

```yaml
xpack.management.elasticsearch.username:logstash_admin_user ①
xpack.management.elasticsearch.password：t0p.s3cr3t
```


![1](../source/images/common/1.png) 您在此处指定的用户必须具有内置的 `logstash_admin` 角色以及您之前创建的 `logstash_writer` 角色。
