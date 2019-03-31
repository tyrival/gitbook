## 命令行运行

要从命令行运行Logstash，请使用以下命令：

```shell
bin/logstash [options]
```

其中 `options` 是 [命令行参数](#命令行参数)。 bin目录的位置因平台而异。请参阅 [目录结构](../04-Setting-Up-and-Running-Logstash/Logstash-Directory-Layout.md) 来查找 `bin\logstash `的位置。

下面以自定义配置文件 `mypipeline.conf` 来运行Logstash：

```shell
bin/logstash -f mypipeline.conf
```

在命令行中使用的参数会覆盖 `logstash.yml` 中设置的默认值，但  `logstash.yml` 文件本身不会被修改。下次Logstash运行的参数仍旧以该文件为模板。

在试用Logstash时，命令行参数很有用，但是，在生产环境中，建议使用 [logstash.yml](../04-Setting-Up-and-Running-Logstash/logstash.yml.md) 来控制Logstash执行。使用该设置文件可以更方便地指定多个选项，并且它为您提供了一个可版本化的文件，您可以使用该文件保证Logstash运行参数的一致性。

### 命令行参数

Logstash具有以下参数，您可以使用 `—help` 参数来显示此信息。

**`--node.name NAME`**
指定此Logstash实例的名称。如果没有给出值，则默认为当前主机名。
**`-f, --path.config CONFIG_PATH`**
从特定文件或目录加载Logstash配置。如果给出了目录，则该目录中的所有文件将按字母排序，然后拼接成为一个配置文件再解析。此参数不可在一个命令中多次使用。如果多次使用，Logstash将使用最后一次出现（例如，`-f foo -f bar` 与 `-f bar` 相同）。

您可以指定通配符（[通配符支持](../06-Configuring-Logstash/Glob-Pattern-Support.md)），任何匹配的文件将按上述顺序加载。例如，您可以使用通配符按名称加载特定文件：

```shell
bin/logstash --debug -f '/tmp/{one,two,three}'
```

使用此命令，Logstash将连接三个配置文件 `/tmp/one`，`/tmp/two` 和 `/tmp/three`，并将它们拼接为一个配置文件进行解析。

**`-e, --config.string CONFIG_STRING`**
    使用给定的字符串作为配置信息。语法与配置文件内容相同。如果未指定输入，则以下内容用作缺省输入：`input { stdin { type => stdin } }` ；如果未指定输出，则以下内容用作缺省输出：`output { stdout { codec => rubydebug } }`。如果您希望同时使用这两个默认值，请使用空字符串作为 `-e` 标志。默认值为nil。
**`--java-execution`**
    使用Java执行引擎而不是默认的Ruby执行引擎。
**`--modules`**
启动已命名的模块。与 `-M` 选项一起使用，可为指定模块的默认变量赋值。如果在命令行中使用了 `--modules`，则将忽略 `logstash.yml` 中所有模块的设置。该标志与 `-f` 和 `-e` 标志互斥。只能指定 `-f`，`-e` 或 `--modules` 中的一个。可以指定多个模块，以逗号分隔，或者多次调用 `--modules` 标志来指定多个模块。
**`-M, --modules.variable`**
    指定模块的可配置选项值。设置Logstash变量值的格式为 `-M "MODULE_NAME.var.PLUGIN_TYPE.PLUGIN_NAME.KEY_NAME=value"`。对于其他设置，则是 `-M "MODULE_NAME.KEY_NAME.SUB_KEYNAME=value"`。`-M` 标志可以根据需要多次使用。如果未指定 `-M` 选项，则将使用默认值。`-M`标志仅与 `--modules` 标志一起使用。如果没有 `--modules` 标志，它将被忽略。
**`--pipeline.id ID`**
    设置管道的ID，默认为main。
**`-w， --pipeline.workers COUNT`**
    设置要运行的管道工作者数量。此选项用于指定管道执行过滤器和输出阶段的并发数量。如果发现事件正在备份，或者CPU未饱和，请考虑增加此数量以更好地利用主机性能。默认值是主机CPU核心的数量。
**`-b, --pipeline.batch.size SIZE`**
    管道的最大批容量，当单个管道工作者接收到的事件达到此数量时，则立刻将这些事件打包为一个批，进行过滤和输出处理。默认值为125。如果给此参数赋予较大的数值，可以提高效率，但会导致更高的内存开销，从而需要在 `jvm.options` 配置文件中配置更高的堆内存空间，更多信息请查看 [配置文件](../04-Setting-Up-and-Running-Logstash/Logstash-Configuration-Files.md)。
**`-u, --pipeline.batch.delay DELAY_IN_MS`**
    为管道声明一个时间延迟的值，当创建管道在接收到一个事件后，接下来未接受到事件的空闲时间达到这个值，则将管道内缓存的事件打包为一个批，传递给管道工作者进行处理，即使这个批还未达到最大批容量。默认50毫秒。
**`--pipeline.unsafe_shutdown`**
    默认情况下，当Logstash接收到关闭命令时，如果还有未处理的事件，会拒绝执行关闭操作，直到所有事件都进行了输出。当此选项设置为`true`时，当Logstash接收到关闭命令后，会无视内存中存在的未处理事件而强行关闭，从而会导致数据丢失。
**`--path.data PATH`**
    此配置指向一个可写目录。Logstash存储数据的位置。插件也可以访问此路径。默认值是Logstash根目录下的 `data` 目录。
**`-p, --path.plugins PATH`**
    自定义插件的存放路径。可以多次使用该参数以指定多个路径。插件应该位于特定的目录层次结构中： `PATH/logstash/TYPE/NAME.rb`，其中TYPE是输入`input`、过滤器`filter`、输出`output`、或者编解码器`codec`, NAME是插件的名称。
**`-l, --path.logs PATH`**
    Logstash内部日志所在的目录。
**`--log.level LEVEL`**
    设置Logstash的日志级别。可选值包括：
    - fatal: 记录应用程序中止后通常会出现的非常严重的错误消息
    - error: 记录错误
    - warn: 记录警告
    - info: 详细日志信息（这是默认值）
    - debug: 日志调试信息（适用于开发人员）
    - trace: 记录调试信息以外的更细粒度的消息
**`--config.debug`**
    将编译的完整配置信息显示为调试日志（必须同时启用 `--log.level=debug`）。警告：日志消息将包含作为纯文本传递给插件配置的任何密码选项，有可能导致明文密码出现在您的日志中！
**`-i, --interactive SHELL`**
    交互模式。可选参数值为"irb"和"pry"。
**`-V, --version`**
    显示Logstash及其相关的版本。
**`-t, --config.test_and_exit`**
    检查配置的语法是否有效，然后退出。请注意，使用此参数不会检查grok模式的正确性。 Logstash可以从目录中读取多个配置文件。如果将此标志与 `--log.level=debug` 组合使用，Logstash将记录组合​​后的配置文件，并使用来自源文件的注释来标记每个配置块。
**`-r, --config.reload.automatic`**
    监听配置更改并在配置更改时重新加载。注意：使用SIGHUP手动重新加载配置。默认值为false。
**`--config.reload.interval RELOAD_INTERVAL`**
    轮询配置以确定是否变更的频率。默认值为3秒。
**`--http.host HTTP_HOST`**
    Web API的主机地址。默认值为"127.0.0.1"。
**`--http.port HTTP_PORT`**
    Web API端口。默认值为9600-9700。此设置在端口9600-9700的范围内选择第一个可用端口号。
**`--log.format FORMAT`**
    指定Logstash是否应该以JSON格式（每行一个事件）或纯文本（使用Ruby的Object#inspect）生成日志。默认为"plain"。
**`--path.settings SETTINGS_DIR`**
    设置包含 [logstash.yml](../04-Setting-Up-and-Running-Logstash/logstash.yml.md) 的目录以及log4j日志配置。这也可以通过 `LS_SETTINGS_DIR` 环境变量设置。默认值是Logstash 根目录下的 `config` 目录。
**`-h, --help`**
    帮助
