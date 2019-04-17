## Docker安装

Elasticsearch有Docker镜像。镜像使用 [centos:7](https://hub.docker.com/_/centos/) 作为基本镜像。

有关所有已发布的Docker镜像和标记的列表，请访问 www.docker.elastic.co。源文件在 [Github](https://github.com/elastic/elasticsearch/tree/6.7/distribution/docker) 中。

此软件包可在Elastic许可下免费使用。它包含开源和免费商业功能以及付费商业功能。开始 [为期30天的试用](https://www.elastic.co/guide/en/elastic-stack-overview/6.7/license-management.html)，试用所有付费商业功能。有关Elastic许可级别的信息，请参阅 [订阅](https://www.elastic.co/cn/subscriptions) 页面。

### 拉取镜像

获取Elasticsearch镜像仅需要发送`docker pull`命令到Elastic Docker中心。

```sh
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.7.1
```

或者，您可以下载仅包含Apache 2.0许可下可用功能的其他Docker映像。要下载图像，请访问 www.docker.elastic.co。

### 命令行运行

#### 开发模式

使用以下命令可以快速启动Elasticsearch以进行开发或测试：

```sh
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.7.1
```

#### 生产模式

> **重要**
>
> `vm.max_map_count`内核设置需要设置为至少`262144`才能用于生产环境。根据您的平台：
>
> - Linux
>
>   `vm.max_map_count`设置在`/etc/sysctl.conf`中：
>
>   ```sh
>   $ grep vm.max_map_count /etc/sysctl.conf
>   vm.max_map_count=262144
>   ```
>
>   要在实时系统上应用该设置，请执行以下操作： `sysctl -w vm.max_map_count=262144`
>
> - Docker for MacOS
>
>   必须在xhyve虚拟机中设置`vm.max_map_count`：
>
>   ```sh
>   $ screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
>   ```
>
>   只需按Enter键并配置`sysctl`，就像对待Linux一样：
>
>   ```sh
>   sysctl -w vm.max_map_count=262144
>   ```
>
> - Windows和macOS的Docker Toolbox
>
>   必须通过docker-machine设置`vm.max_map_count`设置：
>
>   ```txt
>   docker-machine ssh
>   sudo sysctl -w vm.max_map_count=262144
>   ```

以下示例显示包含两个Elasticsearch节点的集群。要打开群集，请使用 [docker-compose.yml](#`docker-compose.yml`：) 并输入：

```sh
docker-compose up
```

> **注意**
>
> Linux上的Docker未预装`docker-compose`。可以在 [Docker Compose网页](https://docs.docker.com/compose/install/#install-using-pip) 上找到安装它的说明。

节点`elasticsearch`侦听`localhost:9200`，`elasticsearch2`通过Docker网络与`elasticsearch`通讯。

此示例还使用名为`esdata1`和`esdata2`的Docker命名卷，如果尚未存在，将创建它们。

##### `docker-compose.yml`：

```yaml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.7.1
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.7.1
    container_name: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local

networks:
  esnet:
```

要停止群集，请键入`docker-compose down`。数据卷将持久化存在，因此可以使用`docker-compose up`使用相同的数据再次启动集群。要销毁群集和数据卷，只需键入`docker-compose down -v`即可。

#### 检查集群的状态

```txt
curl http://127.0.0.1:9200/_cat/health
1472225929 15:38:49 docker-cluster green 2 2 4 2 0 0 0 0 - 100.0%
```

日志消息转到控制台，由配置的Docker日志记录驱动程序处理。默认情况下，您可以使用docker日志访问日志。

### Docker配置Elasticsearch

Elasticsearch从`/usr/share/elasticsearch/config/`中的文件加载配置。配置项记录在 [配置](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md) 和 [JVM配置](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch/Setting-JVM-options.md)。

该镜像提供了几种配置Elasticsearch设置的方法，传统方法是提供自定义文件，即`elasticsearch.yml`。也可以使用环境变量来设置选项：

##### A. 通过Docker环境变量传入参数

例如，要使用`docker run`定义集群名称，可以传递 `-e "cluster.name=mynewclustername"`。双引号是必需的。

##### B. 绑定配置

创建自定义配置文件并将其挂载到映像的相应文件上。例如，可以使用以下参数来执行`docker run`绑定挂载`custom_elasticsearch.yml`：

```sh
-v full_path_to/custom_elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```

> **重要**
>
> 容器使用**uid:gid 1000:1000**作为用户`elasticsearch`运行Elasticsearch。绑定挂载的主机目录和文件（例如上面的`custom_elasticsearch.yml`）需要此用户可以访问。对于 [数据和日志目录](../../02-Set-up-Elasticsearch/Important-Elasticsearch-configuration/path.data-and-path.logs.md)，例如`/usr/share/elasticsearch/data`，还需要写访问权限。另见下面的注释1。

##### C. 定制镜像

在某些环境中，自定义镜像可能更有意义，自定义`Dockerfile`很简单：

```sh
FROM docker.elastic.co/elasticsearch/elasticsearch:6.7.1
COPY --chown=elasticsearch:elasticsearch elasticsearch.yml /usr/share/elasticsearch/config/
```

然后，您可以使用以下内容构建镜像：

```sh
docker build --tag=elasticsearch-custom .
docker run -ti -v /usr/share/elasticsearch/data elasticsearch-custom
```

某些插件需要其他安全权限。您必须在运行Docker映像时附加`tty`，并在提示是否接受时选择yes，或者单独检查安全权限，以及您是否愿意将`—batch`标志添加到plugin install命令来明确接受它们。有关详细信息，请参阅 [插件管理文档](https://www.elastic.co/guide/en/elasticsearch/plugins/6.7/_other_command_line_parameters.html)。

##### D. 覆盖镜像的默认CMD

通过覆盖镜像的默认命令，可以将命令行选项传递给Elasticsearch进程。例如：

```sh
docker run <various parameters> bin/elasticsearch -Ecluster.name=mynewclustername
```

### 配置Elasticsearch Docker镜像的SSL/TLS

请参阅 [Docker容器间通讯加密](../../02-Set-up-Elasticsearch/Configuring-security/Encrypting-communications-in-an-Elasticsearch-Docker-Container.md)。

### 生产环境的说明和默认值

我们收集了许多生产用途的最佳实践。下面提到的任何Docker参数都假定使用`docker run`。

1. 默认情况下，Elasticsearch使用uid:gid `1000:1000`作为用户`elasticsearch`在容器内运行。

> **警告**
>
> 一个例外是 [Openshift](https://docs.openshift.com/container-platform/3.6/creating_images/guidelines.html#openshift-specific-guidelines)，它使用任意分配的用户ID运行容器。 Openshift将呈现持久性卷，gid设置为`0`，无需任何调整即可工作。

如果要挂载本地目录或文件，请确保此用户可以读取，而 [数据和日志目录](../../02-Set-up-Elasticsearch/Important-Elasticsearch-configuration/path.data-and-path.logs.md) 还需要写访问权限。一个好的策略是为本地目录授予对gid `1000`或`0`的组访问权限。例如，要准备一个本地目录来挂载存储数据：

```sh
mkdir esdatadir
chmod g+rwx esdatadir
chgrp 1000 esdatadir
```

作为最后的手段，您还可以强制容器通过环境变量`TAKE_FILE_OWNERSHIP`，来改变 [数据和日志目录](../../02-Set-up-Elasticsearch/Important-Elasticsearch-configuration/path.data-and-path.logs.md) 的挂载所有权。在这种情况下，它们将由uid:gid `1000:0` 根据需要提供对Elasticsearch进程的读/写访问权限。

2. 确保为elasticsearch容器提供[nofile](../../02-Set-up-Elasticsearch/Important-System-Configuration/Configuring-system-settings.md)和[nproc](../../02-Set-up-Elasticsearch/Bootstrap-Checks/Maximum-number-of-threads-checks.md)的增加的ulimits非常重要。验证Docker守护程序的 [init system](https://github.com/moby/moby/tree/ea4d1243953e6b652082305a9c3cda8656edab26/contrib/init) 是否已将这些设置为可接受的值，如果需要，可以在守护程序中调整它们，或者为每个容器覆盖它们，例如使用`docker run`：

```sh
--ulimit nofile=65535:65535
```

> **注意**
>
> 检查上述ulimits的Docker守护程序默认值的一种方法是运行：
>
> ```sh
> docker run --rm centos:7 /bin/bash -c 'ulimit -Hn && ulimit -Sn && ulimit -Hu && ulimit -Su'
> ```

3. 需要禁用swapping以提高性能和节点稳定性。这可以通过 [Elasticsearch文档](../../02-Set-up-Elasticsearch/Important-System-Configuration/Disable-swapping.md) 中提到的方法来实现。如果你选择`bootstrap.memory_lock: true`方法，除了通过 [配置方法](#Docker配置Elasticsearch) 定义它之外，你还需要`memlock: true` 进行控制，在 [Docker守护进程](https://docs.docker.com/engine/reference/commandline/dockerd/#default-ulimits) 中定义或为容器指定设置。这在上面的 [docker-compose.yml](#`docker-compose.yml`：) 进行了展示。如果使用`docker run`：

```
-e "bootstrap.memory_lock=true" --ulimit memlock=-1:-1
```

4. 该镜像 [暴露](https://docs.docker.com/engine/reference/builder/#/expose) TCP端口9200和9300。对于群集，建议使用`--publish-all`随机化已发布的端口，除非您为每个主机固定一个容器。

5. 使用`ES_JAVA_OPTS`环境变量来设置堆大小。例如，`docker run`使用`-e ES_JAVA_OPTS="-Xms16g -Xmx16g"`来配置为16G。

6. 部署指定版本的Elasticsearch映像。例如，`docker.elastic.co/elasticsearch/elasticsearch:6.7.1`。

7. 始终挂载`/usr/share/elasticsearch/data`卷，如 [生产模式](#生产模式) 中所示，原因如下：

   a. 容器被关闭时，elasticsearch节点的数据不会丢失

   b. Elasticsearch对I/O敏感，Docker存储驱动程序不适合快速I/O.

   c. 允许使用高级 [Docker卷插件](https://docs.docker.com/engine/extend/legacy_plugins/)

8. 如果您使用的是devicemapper存储驱动程序，请确保未使用默认的`loop-lvm`模式。配置使用docker-engine代替 [direct-lvm](https://docs.docker.com/storage/storagedriver/device-mapper-driver/)。
9. 请考虑使用其他 [日志驱动程序](https://docs.docker.com/config/containers/logging/configure/) 集中采集日志。另请注意，默认的json-file日志记录驱动程序不适合生产环境。

### 下一步

您现在已经设置了测试Elasticsearch环境。在开始认真开发或使用Elasticsearch投入生产之前，您必须进行一些额外的设置：

- 了解如何 [配置Elasticsearch](../../02-Set-up-Elasticsearch/Configuring-Elasticsearch.md)。
- 配置 [重要的Elasticsearch配置](../../02-Set-up-Elasticsearch/Important-Elasticsearch-configuration.md)。
- 配置 [重要的系统设置](../../02-Set-up-Elasticsearch/Important-System-Configuration.md)。