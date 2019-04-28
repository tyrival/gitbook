## 系统设置

配置系统设置的位置取决于您用于安装Elasticsearch的软件包，以及您使用的操作系统。

使用`.zip`或`.tar.gz`软件包时，可以配置系统设置：

- 临时配置用 [`ulimit`](#ulimit)，或
- 永久配置用 [`/etc/security/limits.conf`](#etcsecuritylimitsconf)。

使用RPM或Debian软件包时，大多数系统设置都在 [系统配置文件](#系统配置文件) 中。但是，使用systemd的系统在 [systemd配置](#Systemd配置)。

### `ulimit`

在Linux系统上，`ulimit`可用于临时更改资源限制。在切换到运行Elasticsearch的用户之前，通常需要先成为为`root`。例如，要将打开的文件句柄数（`ulimit -n`）设置为65,536，可以执行以下操作：

```sh
sudo su  ①
ulimit -n 65535 ②
su elasticsearch ③
```


![](../../source/images/common/1.png) 成为`root`。

![](../../source/images/common/2.png) 更改打开文件的最大数量。

![](../../source/images/common/3.png) 成为`elasticsearch`用户以启动Elasticsearch。

新限制仅在当前会话期间应用。

您可以使用`ulimit -a`查询所有当前应用的限制。

### `/etc/security/limits.conf`

在Linux系统上，可以通过编辑`/etc/security/limits.conf`文件为指定用户设置持久限制。要将`elasticsearch`用户的最大打开文件数设置为65,536，请将以下行添加到`limits.conf`文件中：

```sh
elasticsearch  -  nofile  65535
```

此更改仅在`elasticsearch`用户下次打开新会话时生效。

> **注意**
>
> **Ubuntu和`limits.conf*`**
> Ubuntu忽略`init.d`启动的进程的`limits.conf`文件。要启用`limits.conf`文件，请编辑`/etc/pam.d/su`并取消注释以下行：
>
> ```
> # session required pam_limits.so
> ```

### 系统配置文件

使用RPM或Debian软件包时，可以在系统配置文件中指定系统设置和环境变量，该文件位于：

| 类型   | 路径                           |
| ------ | ------------------------------ |
| RPM    | `/etc/sysconfig/elasticsearch` |
| Debian | `/etc/default/elasticsearch`   |

但是，对于使用systemd的系统，需要通过 [systemd](#Systemd配置) 指定系统限制。

### Systemd配置

在使用 [systemd](https://en.wikipedia.org/wiki/Systemd) 的系统上使用RPM或Debian软件包时，必须通过systemd指定系统限制。

systemd服务文件（`/usr/lib/systemd/system/elasticsearch.service`）包含默认应用的限制。

要覆盖它们，请添加一个名为`/etc/systemd/system/elasticsearch.service.d/override.conf`的文件（或者，您可以运行`sudo systemctl edit elasticsearch`，它会在默认编辑器中自动打开文件）。 在此文件中进行更改设置，例如：

```sh
[Service]
LimitMEMLOCK=infinity
```

完成后，运行以下命令重新加载单位：

```sh
sudo systemctl daemon-reload
```
