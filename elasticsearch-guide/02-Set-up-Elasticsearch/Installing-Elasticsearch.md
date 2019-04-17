## 安装

### 主机

Elasticsearch可以在您自己的硬件上运行，也可以使用我们在 [Elastic Cloud](https://www.elastic.co/cloud/) 上托管的Elasticsearch Service，该服务可在AWS和GCP上使用。您可以 [免费试用托管服务](https://www.elastic.co/cloud/elasticsearch-service/signup)。

### 自主安装

Elasticsearch提供以下包格式：

| 格式           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| `zip`/`tar.gz` | `zip`和`tar.gz`包适合在任何系统上安装，是大多数系统上Elasticsearch入门的最简单选择。<br/>[.zip或.tar.gz安装](../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-.zip-or-.tar.gz.md) 或 [.zip在Windows安装](../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-.zip-on-Windows.md) |
| `deb`          | `deb`包适用于Debian，Ubuntu和其他基于Debian的系统。 Debian软件包可以从Elasticsearch网站或我们的Debian存储库下载。<br/>[Debian包安装](../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-Debian-Package.md) |
| `rpm`          | rpm软件包适合安装在Red Hat，Centos，SLES，OpenSuSE和其他基于RPM的系统上。 RPM可以从Elasticsearch网站或我们的RPM存储库下载。使用RPM安装Elasticsearch<br />[RPM安装](../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-RPM.md) |
| `msi`          | [beta] - 此功能处于测试版状态，可能会发生变化。设计和代码不如官方GA功能成熟，并且按原样提供，不提供任何保证。测试版功能不受官方GA功能支持SLA的约束。<br/>`msi`软件包适合安装在具有.NET 4.5的Windows 64位系统上，是在Windows上开始使用Elasticsearch的最简单选择。 MSI可以从Elasticsearch网站下载。<br />[Windows MSI安装](../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-Windows-MSI-Installer.md) |
| `docker`       | 镜像可用于将Elasticsearch作为Docker容器运行。它们可以从Elastic Docker Registry下载。<br/>[Docker安装](../02-Set-up-Elasticsearch/Installing-Elasticsearch/Install-Elasticsearch-with-Docker.md) |

### 配置管理工具

我们还提供以下配置管理工具来帮助进行大型部署：

| 工具    | 地址                                                         |
| ------- | ------------------------------------------------------------ |
| Puppet  | [puppet-elasticsearch](https://github.com/elastic/puppet-elasticsearch) |
| Chef    | [cookbook-elasticsearch](https://github.com/elastic/cookbook-elasticsearch) |
| Ansible | [ansible-elasticsearch](https://github.com/elastic/ansible-elasticsearch) |

