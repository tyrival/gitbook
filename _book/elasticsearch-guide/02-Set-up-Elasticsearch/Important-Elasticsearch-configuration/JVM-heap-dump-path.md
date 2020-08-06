## JVM堆转存路径

默认情况下，Elasticsearch配置JVM为将内存异常转存到默认数据目录（[RPM](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-RPM.md) 和 [Debian](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-Debian-Package.md) 为`/var/lib/elasticsearch`，[tar和zip](../../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-.zip-or-.tar.gz.md) 为Elasticsearch安装根目录下的`data`目录）。如果此路径不适合接收堆转存，则应修改 [`jvm.options`](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Setting-JVM-options.md) 中的`-XX:HeapDumpPath=...`，如果指定了目录，JVM将根据正在运行的实例的PID生成堆转储的文件名。如果指定了固定文件名而不是目录，则当JVM在内存不足异常上执行堆转储时，该文件不能存在，否则堆转储将失败。

