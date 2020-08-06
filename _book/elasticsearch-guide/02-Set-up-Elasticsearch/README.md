# 第2章 设置

本节包含设置Elasticsearch并使其运行的信息，包括：

- 下载
- 安装
- 开始
- 配置

### 支持平台

官方支持的操作系统和JVM可在此处获得：[支持矩阵](https://www.elastic.co/cn/support/matrix)。 Elasticsearch在列出的平台上进行了测试，但它也可能在其他平台上运行。

### Java（JVM）版本

Elasticsearch是使用Java构建的，并且至少需要 [Java8](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 才能运行。 仅支持Oracle Java和OpenJDK。 应在所有Elasticsearch节点和客户端上使用相同的JVM版本。

我们建议在Java 8发行版系列中安装**Java 1.8.0_131或更高版本**。 我们建议使用受 [支持](https://www.elastic.co/cn/support/matrix) 的 [Java LTS版](https://www.oracle.com/technetwork/java/java-se-support-roadmap.html)。 如果使用错误的Java版本，Elasticsearch将拒绝启动。

可以通过设置`JAVA_HOME`环境变量来配置Elasticsearch将使用的Java版本。