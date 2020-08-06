## GC日志

默认情况下，Elasticsearch启用GC日志。这在 [jvm.options](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Setting-JVM-options.md) 中配置，默认与Elasticsearch日志相同的位置。 默认配置每64MB滚动一次日志，最多可占用2GB的磁盘空间。