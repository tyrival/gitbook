## 缓存目录

默认情况下，Elasticsearch启动脚本在系统临时目录下创建的专用临时目录。

在某些Linux发行版上，系统将清除`/tmp`中的文件和目录（如果它们最近未被访问过）。如果长时间不使用需要临时目录的功能，则可能导致专用临时目录在Elasticsearch运行时被删除。如果随后使用需要临时目录的功能，则会导致问题。

如果使用`.deb`或`.rpm`软件包安装Elasticsearch，并在`systemd`下运行它，那么Elasticsearch使用的专用临时目录将从定期清理中排除。

但是，如果您打算在Linux上运行`.tar.gz`发行版很长一段时间，那么您应该考虑为Elasticsearch创建一个专用的临时目录，该临时目录不在会被清除旧文件和目录的路径下。此目录应具有权限集，以便只有运行Elasticsearch的用户才能访问它。然后在启动Elasticsearch之前将`$ES_TMPDIR`环境变量设置为指向它。