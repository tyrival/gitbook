## file

- 插件版本：v4.1.9
- 发布于：2018-12-19
- [更新日志](https://github.com/logstash-plugins/logstash-input-file/blob/v4.1.9/CHANGELOG.md)

对于其他版本，请参阅 [插件版本文档](https://www.elastic.co/guide/en/logstash-versioned-plugins/current/input-file-index.html)。

### 帮助

有关插件的问题，请在 [论坛](https://discuss.elastic.co) 中创建一个主题。对于bug或功能请求，请在 [Github](https://github.com/logstash-plugins/logstash-input-file) 创建issue。有关Elastic支持插件的列表，请参阅 [Elastic支持列表](https://www.elastic.co/support/matrix#matrix_logstash_plugins)。

### 说明

文件中的事件流，通常以类似于 `tail -0F` 的方式向后拼接它们，但可选择从头开始读取它们。

通常，日志记录会在每行写入的末尾添加换行符。默认情况下，假定每个事件都是一行，并且换行符之前的文本被视为一行。如果您想将多个日志行连接到一个事件中，您将需要使用多行编解码器。插件在发现新文件和处理每个发现的文件之间循环。发现的文件具有生命周期，它们从"watched"或"ignored"状态开始，生命周期的状态还包括是："active"，"closed"和"unwatched"

默认情况下，使用4095个文件的窗口来限制正在使用的文件句柄数。处理有许多阶段：

- 检查自上次以来，"closed"或"ignored"文件的大小是否已更改，如果是，则将其置于"watched"状态。
- 选择足够的"watched"文件来填充窗口中的可用空间，这些文件变为"active"。
- 打开并读取活动文件，默认情况下，每个文件从最后的已知位置读取到当前内容（EOF）的末尾。

在某些情况下，能够控制读取哪些文件、读取顺序，以及是完全读取还是带状/条带读取是非常有用的。完整读取是文件A，然后是文件B，然后是文件C，依此类推。带状或条带是读取文件A的一部分，然后是文件B然后是文件C，依此循环到文件A，直到读取所有文件。通过更改 [`file_chunk_count`](#filechunkcount) 和 [`file_chunk_size`](#filechunksize) 来指定带状读取。如果您希望所有文件中的某些事件尽早出现在Kibana中，则绑定和排序可能很有用。

该插件有两种操作模式，Tail模式和Read模式。

#### Tail模式

在此模式下，插件旨在跟踪更改的文件，并将新内容附加到文件。在此模式下，文件被视为无限的内容流，EOF没有特殊意义。插件总是假设会有更多内容。文件轮换时，如果检测到较小或零大小，则当前位置重置为零，并继续流式传输。遇到分隔符，则将累积的字符作为一行发出。

#### Read模式
在这种模式下，插件会将每个文件视为内容完整，即有限的行数据流，此时EOF非常重要。不需要最后一个分隔符，因为EOF意味着累积的字符可以作为一行发出。此外，这里的EOF意味着文件可以关闭，并置于"unwatched"状态——这会自动释放活动窗口中的空间。此模式还可以处理压缩文件，因为它们是内容完整的。读取模式还允许在完全处理文件后执行操作。

过去尝试用无限数据流模拟读模式，但效果并不理想，专用的读模式是一种改进。

### 追踪监视文件中的当前位置

该插件跟踪每个文件中的当前位置，并将其记录在不同的sincedb文件中。这样就可以在重启Logstash后，让它从停止的位置开始，而不会错过Logstash停止时添加到文件中的行。

默认情况下，sincedb文件在Logstash的数据目录中，文件名基于正在监视的文件名模式（即 `path` 选项）。因此，更改文件名模式将导致其使用新的sincedb文件，并且将丢失现有的当前位置状态。如果您需要更改文件名模式，则可以使用 `sincedb_path` 选项明确指向sincedb路径。

每个输入必须使用不同的 `sincedb_path`，反之会导致问题。每个输入的读取检查点必须存储在不同的路径中，以便信息不会覆盖。

通过标识符跟踪文件。该标识符由inode，主设备号和次设备号组成。在Windows中，从kernel32 API调用中获取不同的标识符。

Sincedb记录现在可以过期，这意味着在一段时间后，不会记住旧文件的读取位置。文件系统可能需要为新内容重用inode。理想情况下，我们不会使用旧内容的读取位置，但我们没有可靠的方法来检测发生了inode重用。这与Read模式更相关，因其在sincedb中跟踪了大量文件。但请记住，如果记录已过期，将再次读取先前看到的文件。

Sincedb文件是包含四列（< v5.0.0），五列或六列的文本文件：

1. inode编号（或同类编号）。
2. 文件系统的主要设备号（或同类内容）。
3. 文件系统的次设备号（或同类内容）。
4. 文件中的当前字节偏移量。
5. 最后一个活动时间戳（浮点数）
6. 此记录匹配的最后一条已知路径（对于转换为新格式的旧sincedb记录，这是空白的）。

在非Windows系统上，您可以获取文件的inode编号，例如 `ls -li`。

### 从网络远程读取卷

File输入未在远程文件系统（如NFS，Samba，s3fs-fuse等）上进行全面测试，但偶尔会测试NFS。远程FS客户端给出的文件大小，用于控制在规定时间内读取的数据量，从而避免读入到已分配但尚未填充的内存中。由于我们在标识符中使用主设备和次设备来跟踪文件的"最后读取"位置，并且重新挂载设备时，主次设备可能发生变更，会导致与sincedb记录不匹配。Read模式可能不适用于远程文件系统，因为客户端发现时的文件大小，可能与远端的文件大小不同，这是远程客户端复制过程中的延迟导致的。

### Tail模式的文件轮换

无论文件是通过重命名还是复制操作轮换，都可以通过此输入检测和处理文件轮换。要支持在轮换发生后一段时间写入轮换文件的程序，请在文件名匹配中包含原始文件名和轮换文件名（例如/var/log/syslog和/var/log/syslog.1）用来监视（`path` 选项）。对于重命名，inode将从 `/var/log/syslog` 移动到 `/var/log/syslog.1`，并且"state"也在内部变更，旧内容将不会被重读，但在重命名的文件中的新内容将被读取。如果将复制/截断的内容复制到新文件路径（如果发现），将被视为新文件并从头开始读取。因此，复制的文件路径不应该是要监视的文件名匹配（ `path` 选项）。截断将被检测到到并将"最后读取"位置更新为零。

### 文件输入配置配置项

此插件支持以下配置选项以及稍后描述的 [通用配置项](#通用配置项)。

> **注意：**
> 持续时间设置可以文本形式指定，例如"250 ms"，此字符串将转换为十进制秒。支持相当一部分的自然语言和缩写持续时间，详见 [时间字符串](#时间字符串)。

| 设置                                                  | 输入类型                                                     | 必须 |
| ----------------------------------------------------- | ------------------------------------------------------------ | ---- |
| [`close_older`](#closeolder)                         | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串) | 否   |
| [`delimiter`](#delimiter)                             | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`discover_interval`](#discoverinterval)             | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`exclude`](#exclude)                                 | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`file_chunk_count`](#filechunkcount)               | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`file_chunk_size`](#filechunksize)                 | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`file_completed_action`](#filecompletedaction)     | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为`["delete", "log", "log_and_delete"]` | 否   |
| [`file_completed_log_path`](#filecompletedlogpath) | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [sortby)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为`["last_modified", "path"]` | 否   |
| [`file_sort_direction`](#filesortdirection)         | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为`["asc", "desc"]` | 否   |
| [`ignore_older`](#ignoreolder)                       | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串) | 否   |
| [`max_open_files`](#maxopenfiles)                   | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) | 否   |
| [`mode`](#mode)                                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为`["tail", "read"]` | 否   |
| [`path`](#path)                                       | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 是   |
| [`sincedb_clean_after`](#sincedbcleanafter)         | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串) | 否   |
| [`sincedb_path`](#sincedbpath)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`sincedb_write_interval`](#sincedbwriteinterval)   | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串) | 否   |
| [`start_position`](#startposition)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为`["beginning", "end"]` | 否   |
| [`stat_interval`](#statinterval)                     | [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串) | 否   |

另请参阅 [通用配置项](#通用配置项) 以获取所有输入插件支持的选项列表。

##### close_older

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串)
- 默认值为 `"1 hour"`

文件输入将在读取的文件（数字类型默认单位为秒），并经过指定时间后关闭，当文件被tail或read时，处理方式不同。如果tail，并且传入数据后经过一定时间，还未收到新数据，则可以关闭文件（允许打开其他文件），但是在检测到新数据时将排队等待重新打开。如果read，文件将在最后一个字节被读取，并经过指定时间后，则关闭。当将插件升级到4.1.0+，如果是read而不是不tail，并且不切换为读取模式，则保留此设置以实现向后兼容性。

##### delimiter

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 默认值为 `"\n"`

设置新行分隔符，默认为 "\n"。请注意，在读取压缩文件时，不使用此设置，而是使用标准的Windows或Unix行结尾。

##### discover_interval

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为15

使用 `path` 选项中的文件名匹配规则来发现新文件的频率。该值是 `stat_interval`  的倍数，例如 `stat_interval` 是 "500 ms"，则每15 X 500毫秒=7.5秒发现新文件。在实践中，这将是最好的做法，因为需要考虑阅读新内容所花费的时间。

##### exclude

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 此设置没有默认值。

排除（与文件名匹配的规则，而不是完整路径）。文件名匹配在这里也是有效的。例如，如果你有

```ruby
path => "/var/log/*"
```

在Tail模式下，您可能希望排除gzip压缩文件：

```ruby
exclude => "*.gz"
```

##### file_chunk_count

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `4611686018427387903`

与 `file_chunk_size` 结合使用时，此选项设置从每个文件中读取多少块（带或条带）后，移动到下一个活动文件。例如，`file_chunk_count` 为32，`file_chunk_size` 为32KB，每次将处理每个活动文件的1MB。由于默认值非常大，因此在移动到下一个活动文件之前，文件会被有效地读取到EOF。

##### file_chunk_size

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 默认值为 `32768`（32KB）

以block或chunk的形式从磁盘读取文件内容，并从中提取数据行。请参阅 [`file_chunk_count`](#filechunkcount) 以查看默认情况下更改此设置的原因和时机。

##### file_completed_action

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为 `delete, log, log_and_delete`
- 默认为 `delete`。

在读取模式下，完成文件时应执行什么操作。如果指定了删除，则将删除该文件。如果指定了log，则将文件的完整路径记录到file_completed_log_path设置中指定的文件中。如果指定了log_and_delete，则会执行上述两个操作。

##### file_completed_log_path

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

将完整读取的文件路径附加到哪个文件。当 `file_completed_action` 为 log 或 log_and_delete 时，指定文件的路径。重要提示：此文件会不停向后附加，所以它可能会变得非常大，您需要负责进行文件轮换。

##### file_sort_by

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为 `last_modified, path`
- 默认值为 `last_modified`。

指定 "watched" 的文件用于排序的属性。可以按修改日期或完整路径对文件进行排序。以前，已发现并 "watched" 的文件的处理顺序取决于操作系统。

##### file_sort_direction

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为 `asc, desc`
- 默认为asc。

在排序 "watched" 的文件时，选择升序或降序。如果最早的数据首先是重要的，那么 `last_modified` + `asc` 的默认值是好的。如果最新的数据首先更重要，那么选择 `last_modified` + `desc`。如果对文件完整路径使用特定命名约定，那 `path` + `asc` 可能有助于控制文件处理的顺序。

##### ignore_older

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串)
- 此设置没有默认值。

当文件输入发现文件的最后修改时间（如果指定了数字，则为秒）在指定时间之前，将忽略该文件。在发现之后，如果修改了被忽略的文件，则不再忽略它，并且读取新数据。默认情况下，禁用此选项。注意这里的默认单位是秒。

##### max_open_files

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number)
- 此设置没有默认值。

设置消费输入时使用的最大文件句柄数。如果需要处理的文件多于此数字，请使用close_older关闭某些文件。这不应该设置为操作系统的执行最大值，因为其他Logstash插件和操作系统进程也需要用到文件句柄。内部设置默认值4095。

##### mode

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)，可选项为 `tail, read`
- 默认值为 `tail`。

文件输入的运行模式为tail文件或完整read文件。Read模式现在支持gzip文件处理。如果指定 "read"，则忽略以下其他设置：

1. `start_position`（文件始终从头开始读取）
2. `close_older`（达到EOF时文件自动关闭）

如果指定 "read"，则注意以下设置：

1. `ignore_older`（不处理旧文件）
2. `file_completed_action`（处理文件时应采取的操作）
3. `file_completed_log_path`（应该将完成的文件路径记录到哪个文件）

##### path

- 这是必需的设置。
- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 此设置没有默认值。

要用作输入的文件路径。您可以在此处使用文件名匹配，例如 `/var/log/*.log`。如果使用 `/var/log/**/*.log` 进行匹配，将对所有 `/var/log` 下的 `* .log` 文件进行的递归搜索。路径必须是绝对的，不能是相对的。

您还可以配置多个路径。请参阅 [Logstash配置文件](../06-Configuring-Logstash/Structure-of-a-Config-File.md)。

##### sincedb_clean_after

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串)
- 此设置的默认值为 "2 weeks"。
- 如果指定了数字，则将其解释为天数，并且可以是十进制数，例如0.5是12小时。

sincedb记录最后一次活动的时间戳。如果在过去N天内未在跟踪文件中检测到任何更改，则其sincedb跟踪记录将过期，并且不会保留。此选项有助于防止inode回收问题。 Filebeat有一个 [inode回收的FAQ](https://www.elastic.co/guide/en/beats/filebeat/6.7/faq.html#inode-reuse-issue)。

##### sincedb_path

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

将写入磁盘的sincedb数据库文件的路径（跟踪受监视日志文件的当前位置）。默认情况下会将sincedb文件写入 `<path.data>/plugins/inputs/file` 。注意：它必须是文件路径而不是目录路径

##### sincedb_write_interval

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串)
- 默认值为 `"15 seconds"`

将监视文件的日志写入sincedb的频率（以秒为单位）。

##### start_position

- 价值可以是以下任何一种： `beginning`, `end`
- 默认值为 `"end"`

选择Logstash最初开始读取文件的位置：开头或结尾。默认将文件视为实时流，因此从结尾开始。如果您要导入旧数据，请将其设置为开头。

此选项限于新文件且之前未发现的"第一次联系"情况，即Logstash读取的sincedb文件中没有当前此文件的记录。如果之前已经发现了文件，则此选项无效，并且将使用sincedb文件中记录的位置。

##### stat_interval

- 值类型是 [number](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Number) 或 [时间字符串](#时间字符串)
- 默认值为 `"1 second"`

我们查看统计文件是否已被修改的频率（以秒为单位）。调高此参数将减少调用次数，但会增加检测新日志行的时间。

> **注意：**
>
> 循环发现新文件并检查它们是否已经增长/缩小。在循环之前，此循环将休眠 `stat_interval` 秒。但是，如果文件增长，则会读取新内容并使数据行进行排队。在所有增长的文件中读取和排队可能需要一些时间，尤其是在管道拥挤的情况下。因此整个循环时间是 `stat_interval` 和文件读取时间的组合。

### 通用配置项

所有输入插件都支持以下配置选项：

| 设置                              | 输入类型                                                     | 必须 |
| --------------------------------- | ------------------------------------------------------------ | ---- |
| [`add_field`](#addfield)         | [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash) | 否   |
| [`codec`](#codec)                 | [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Codec) | 否   |
| [`enable_metric`](#enablemetric) | [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean) | 否   |
| [`id`](#id)                       | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |
| [`tags`](#tags)                   | [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array) | 否   |
| [`type`](#type)                   | [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String) | 否   |

#### 详情

##### add_field

- 值类型是 [hash](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Hash)
- 默认值为 `{}`

向事件添加字段

##### codec

- 值类型是 [codec](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Codec)
- 默认值是 `"plain"`

用于输入数据的编解码器。输入编解码器是一种在输入之前解码数据的便捷方法，无需在Logstash管道中使用单独的过滤器。

##### enable_metric

- 值类型是 [boolean](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Boolean)
- 默认值为 `true`

默认情况下，禁用或启用此特定插件实例的指标记录，我们会记录所有的可用指标，但您可以禁用特定插件的指标收集。

##### id

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

为插件配置添加唯一 `ID`。如果未指定ID，Logstash将生成一个ID。强烈建议在配置中设置此ID。当您有两个或更多相同类型的插件时，这尤其有用，例如，如果您有2个beat输入，添加命名ID将有助于使用API监视Logstash。

```json
input {
  beats {
    id => "my_plugin_id"
  }
}
```

##### tags

- 值类型是 [array](../06-Configuring-Logstash/Structure-of-a-Config-File.md#Array)
- 此设置没有默认值。

为您的活动添加任意数量的任意标签。

这有助于后续处理。

##### type

- 值类型是 [string](../06-Configuring-Logstash/Structure-of-a-Config-File.md#String)
- 此设置没有默认值。

将 `type` 字段添加到此输入处理的所有事件。

类型主要用于过滤器激活。

类型存储为事件本身的一部分，因此您也可以使用该类型在Kibana中搜索它。

如果您尝试在已有事件的事件上设置类型（例如，当您将事件从发运人发送到索引器时），则新输入将不会覆盖现有类型。发运人设置的类型即使在发送到另一个Logstash服务器时，仍会保留。

### 时间字符串

格式为 `number` 或 `string`，它们之间的空格是可选的。 所以"45s"和"45 s"都是有效的。

> **提醒：**
>
> 使用最合适的持续时间，例如"3 days"而不是"72 hours"。

##### Weeks

支持的值： `w` `week` `weeks`， "2 w", "1 week", "4 weeks"

##### Days

支持的值： `d` `day` `days`， "2 d", "1 day", "2.5 days"

##### Hours

支持的值： `h` `hour` `hours`，例如 "4 h", "1 hour", "0.5 hours"

##### Minutes

支持的值： `m` `min` `minute` `minutes`，例如 "45 m", "35 min", "1 minute", "6 minutes"

##### Seconds

支持的值： `s` `sec` `second` `seconds`，例如 "45 s", "15 sec", "1 second", "2.5 seconds"

##### Milliseconds

支持的值： `ms` `msec` `msecs`，例如 "500 ms", "750 msec", "50 msecs"

> 注意
> 不支持 `milli` `millis` `milliseconds`

##### Microseconds

支持的值： `us` `usec` `usecs`，例如 "600 us", "800 usec", "900 usecs"

> 注意
> 不支持 `micro` `micros` `microseconds`
