## 线程数

Elasticsearch为不同类型的操作使用许多线程池。 重要的是它能够在需要时创建新线程。 确保Elasticsearch用户可以创建的线程数至少为4096。

这可以通过在启动Elasticsearch之前以root身份设置 [`ulimit -n 4096`](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#ulimit) ，或者在 [`/etc/security/limits.conf`](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md#etcsecuritylimitsconf) 中将`nproc`设置为`4096`来完成。

软件包在`systemd`下作为服务运行时，将自动配置Elasticsearch进程的线程数。 无需其他配置。

