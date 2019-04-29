## 集中管理管道

要配置 [集中管道管理](../07-Managing-Logstash/Centralized-Pipeline-Management.md)：

1. 确认您使用的是包含管道管理功能的许可证。

   有关更多信息，请参阅 https://www.elastic.co/subscriptions 和 [许可证管理](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/license-management.html)。

2. 在 `logstash.yml` 文件中指定 [配置管理设置](#配置管理设置)。至少设置：
   - `xpack.management.enabled: true` 表示启用集中配置管理。
   - `xpack.management.elasticsearch.hosts` 指定将存储Logstash管道配置和元数据的Elasticsearch实例。
   - `xpack.management.pipeline.id` 用于注册要集中管理的管道。

3. 重新启动Logstash。

4. 如果您的Elasticsearch集群受基本身份验证保护，请将 `logstash_admin` 角色分配给将使用集中管道管理的所有用户。请参阅 [X-Pack安全性](../06-Configuring-Logstash/X-Pack-security.md)。

> **注意：**
> 在未配置和启用X-Pack安全性时，集中管理被禁用。

> **重要：**
> 在将Logstash配置为使用集中管道管理之后，您将无法再指定本地管道配置。这意味着启用此功能时，`pipelines.yml` 文件和 `path.config` 和 `config.string` 等设置处于非活动状态。

### 配置管理设置
您可以在 `logstash.yml` 中，设置 `xpack.management` 以启用 [集中管道管理](../07-Managing-Logstash/Centralized-Pipeline-Management.md)。有关配置Logstash的更多信息，请参阅 [logstash.yml](../04-Setting-Up-and-Running-Logstash/logstash.yml.md)。

以下示例假设Elasticsearch和Kibana安装在localhost上，并且启用了基本AUTH，但没有SSL的基本设置。如果您使用的是SSL，则需要指定其他SSL设置。

```yaml
xpack.management.enabled: true
xpack.management.elasticsearch.hosts: "http://localhost:9200/"
xpack.management.elasticsearch.username: logstash_admin_user
xpack.management.elasticsearch.password: t0p.s3cr3t
xpack.management.logstash.poll_interval: 5s
xpack.management.pipeline.id: ["apache", "cloudwatch_logs"]
```

`xpack.management.enabled`

    设置为 `true` 以启用Logstash的X-Pack集中配置管理。
	
`xpack.management.logstash.poll_interval`

	Logstash实例多长时间轮询Elasticsearch的管道更改。默认值为5秒。
	
`xpack.management.pipeline.id`

	指定管道ID列表，以逗号分隔，注册集中管道管理。更改此设置后，您需要重新启动Logstash使更改生效。
	
`xpack.management.elasticsearch.hosts`

	将存储Logstash管道配置和元数据的Elasticsearch实例。这可能与Logstash配置的输出部分中指定的Elasticsearch实例相同，也可能是其他实例。默认为 `http://localhost:9200`。
	
`xpack.management.elasticsearch.username和xpack.management.elasticsearch.password`

	如果您的Elasticsearch集群受基本身份验证保护，则这些设置提供Logstash实例用于进行身份验证以访问配置数据的用户名和密码。您在此处指定的用户名应具有 `logstash_admin` 角色，该角色提供对 `.logstash- *` 索引的访问以管理配置。
	
`xpack.management.elasticsearch.ssl.certificate_authority`

	可选设置，使您可以为Elasticsearch实例的证书颁发机构指定 `.pem` 文件的路径。
	
`xpack.management.elasticsearch.ssl.truststore.path`

	可选设置，提供Java密钥库（JKS）的路径以验证服务器的证书。
	
`xpack.management.elasticsearch.ssl.truststore.password`

	可选设置，为信任库提供密码。
	
`xpack.management.elasticsearch.ssl.keystore.path`

	可选设置，提供Java密钥库（JKS）的路径以验证客户端的证书。
	
`xpack.management.elasticsearch.ssl.keystore.password`

	可选设置，为密钥库提供密码。

