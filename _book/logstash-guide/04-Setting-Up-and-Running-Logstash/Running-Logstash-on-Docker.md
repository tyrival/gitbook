## Docker运行

Logstash的Docker镜像可从Elastic Docker注册中心获得。基础镜像是 [centos:7](https://hub.docker.com/_/centos/)。

已发布的Docker镜像和版本列表，请访问 [www.docker.elastic.co](https://www.docker.elastic.co)。 源代码在 [GitHub](https://github.com/elastic/logstash-docker/tree/6.7)。

这些图像可以在Elastic许可下免费使用。 包含开源、免费商业功能和付费商业功能。 [30天试用](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/license-management.html)，可试用所有付费商业功能。有关弹性许可级别的信息，请参阅 [订阅](https://www.elastic.co/subscriptions)。

### 拉取镜像

获取Logstash的镜像非常简单，只需向Elastic Docker注册中心使用`docker pull`命令。

```bash
docker pull docker.elastic.co/logstash/logstash:6.7.0
```

您可以同样的方式下载其他Docker镜像，这些镜像仅包含Apache 2.0许可下的可用功能。 更多镜像，请访问 [www.docker.elastic.co](https://www.docker.elastic.co)。