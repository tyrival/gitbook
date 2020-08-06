## 配置文件

Logstash有两种类型的配置文件：管道配置文件，用于定义Logstash处理管道；以及设置文件，用于指定控制Logstash启动和执行的选项。

### 管道配置文件
在定义Logstash处理管道的各个阶段时，可以创建管道配置文件。在deb和rpm上，将管道配置文件放在`/etc/logstash/conf.d`目录中。 Logstash尝试仅加载`/etc/logstash/conf.d`目录中具有`.conf`扩展名的文件，并忽略所有其他文件。

有关详细信息，请参阅 [第6章 配置项](../06-Configuring-Logstash/README.md)。

### 设置文件
设置文件已在Logstash安装时定义，包括以下设置文件：

#### logstash.yml
包含Logstash配置项。您可以在此文件中设置参数，从而代替在命令行传递参数。您在命令行中传递的任何参数都会覆盖`logstash.yml`文件中的相应设置。有关详细信息，请参阅 [logstash.yml](../04-Setting-Up-and-Running-Logstash/logstash.yml.md)。

#### pipelines.yml
包含在单个Logstash实例中运行多个管道的框架和说明。有关详细信息，请参阅 [多管道](../06-Configuring-Logstash/Multiple-Pipelines.md)。

#### jvm.options
包含JVM配置项。此文件可设置堆空间的初始值和最大值，以及为Logstash设置所在地区。每个项目需要在单独的行上指定。此文件中的所有其他设置均为专家使用的设置，通常不建议修改。

#### log4j2.properties
包含`log4j 2`库的默认设置。有关详细信息，请参阅 [日志](../04-Setting-Up-and-Running-Logstash/Logging.md)。

#### startup.options（Linux）
包含 `/usr/share/logstash/bin` 中 `system-install` 脚本使用的选项，以便为系统构建适当的启动脚本。安装Logstash软件包时，系统安装脚本将在安装结束后执行，并使用 `startup.options` 中的配置来设置用户、组、服务名称和服务描述等。默认情况下，Logstash服务安装在用户 `logstash` 下。 `startup.options` 文件使您可以更轻松地安装Logstash服务的多个实例。您可以复制文件并更改特定设置的值。请注意，启动时不会读取 `startup.options` 文件。如果要更改Logstash启动脚本（例如，要更改Logstash用户或从其他配置路径读取），则必须重新运行 `system-install` 脚本（以root用户身份）以启用新设置。
