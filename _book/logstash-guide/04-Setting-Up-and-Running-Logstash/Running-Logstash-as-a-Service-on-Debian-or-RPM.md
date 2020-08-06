## 作为Debian或RPM系统服务运行

Logstash安装后不会自动启动。 如何启动和停止Logstash取决于您的系统是使用systemd、upstart还是SysV。

以下是一些常见的操作系统和版本，以及它们使用的启动方式。 此列表只提供参考信息，并非完全覆盖所有情况。 

| Linux发行版              | 系统服务            |
| ------------------------ | ------------------- |
| Ubuntu 16.04 及更新      | [systemd](#systemd) |
| Ubuntu 12.04 到 15.10    | [upstart](#upstart) |
| Debian 8 "jessie" 及更新 | [systemd](#systemd) |
| Debian 7 "wheezy" 及更旧 | [sysv](#sysv)       |
| CentOS (和RHEL) 7 及更新 | [systemd](#systemd) |
| CentOS (和RHEL) 6        | [upstart](#upstart) |

### Systemd

类似Debian Jessie，Ubuntu 15.10+和许多SUSE衍生产品等的发行版，使用systemd和 `systemctl` 命令来启动和停止服务。 Logstash将systemd单元文件放在 `/etc/systemd/system` 中，用于deb和rpm的系统服务。 安装软件包后，您可以使用以下命令启动Logstash：

```shell
sudo systemctl start logstash.service
```

### Upstart

upstart使用如下命令：

```shell
sudo initctl start logstash
```

upstart系统自动生成的配置文件为 `/etc/init/logstash.conf`

### SysV

SysV使用如下命令：

```shell
sudo /etc/init.d/logstash start
```

SysV系统自动生成的配置文件为 `/etc/init.d/logstash`