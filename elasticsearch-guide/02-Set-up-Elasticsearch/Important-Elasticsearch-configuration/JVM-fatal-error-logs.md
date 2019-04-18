## JVM事故错误日志

默认情况下，Elasticsearch将JVM配置为将致命错误日志写入默认日志目录（[RPM](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-RPM.md) 和 [Debian](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-Debian-Package.md) 为`/var/log/elasticsearch`，[tar和zip](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-.zip-or-.tar.gz.md) 为Elasticsearch安装根目录下的`logs`目录）。这些是JVM在遇到致命错误（例如，分段错误）时生成的日志。 如果此路径不适合接收日志，则应将 [jvm.options](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Setting-JVM-options.md) 中的条目`-XX:ErrorFile=...`修改为备用路径。

