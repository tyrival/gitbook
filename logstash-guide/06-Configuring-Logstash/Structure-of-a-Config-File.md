## 构建配置文件

对于要添加到事件处理管道的每种插件类型，Logstash配置文件都有一个独立的配置代码块。例如：

```yaml
# 部分配置
input {
  ...
}

filter {
  ...
}

output {
  ...
}
```

每个部分都包含一个或多个插件的配置选项。如果指定多个过滤器，则会按照它们在配置文件中的顺序使用它们。

### 插件配置

插件的配置包括插件名称，后面紧跟该插件的设置块。例如，以下为配置2个文件输入：

```yaml
input {
  file {
    path => "/var/log/messages"
    type => "syslog"
  }

  file {
    path => "/var/log/apache/access.log"
    type => "apache"
  }
}
```

在此示例中，为每个文件输入配置了两个项目：路径和类型。

插件的配置项因插件类型而异。有关各插件的信息，请参阅 [第17章-输入插件Input](../17-Input-plugins/README.md)，[第18章-输出插件Output](../18-Output-plugins/README.md)，[第19章-过滤器Filter ](../19-Filter-plugins/README.md)和 [第20章-编解码器Codec](../20-Coder-plugins/README.md)。

### 值类型

插件可能要求设置的值为特定类型，例如布尔值、列表、hash。包含以下类型。

#### Array

数组类型大多不建议使用类似 `string` 的标准类型，插件定义 `:list => true` 属性，为了更好地进行类型检查。仍然需要处理不需要类型检查的hash或混合类型的列表。

例：

```yaml
users => [{id => 1，name => bob}，{id => 2，name => jane}]
```

#### Lists

List中的各个元素可以进行类型检查，但List本身不能被检查类型。插件作者可以在声明参数时指定 `:list => true` 来启用列表检查。

例：

```yaml
path => [ "/var/log/messages", "/var/log/*.log" ]
uris => [ "http://elastic.co", "http://example.net" ]
```

此示例配置 `path`为2个 `string` 组成的列表，还将 `uris` 参数配置为URI列表，如果提供的URI全部无效，则失败。

#### Boolean

布尔值必须为 `true` 或 `false`。请注意，`true` 和 `false` 关键字不包含在引号中。

例：

```yaml
ssl_enable => true
```

#### Bytes

字节字段是表示有效字节单位的一个字符串。这是在插件选项中声明特定大小的便捷形式。支持SI（k M G T P E Z Y）和二进制（Ki Mi Gi Ti Pi Ei Zi Yi）单位。二进制单位基数为1024，SI单位基数为1000。此字段不区分大小写，并接受值和单位之间的空格。如果未指定单位，则整数字符串表示字节数。

例：

```yaml
my_bytes => "1113"   # 1113 bytes
my_bytes => "10MiB"  # 10485760 bytes
my_bytes => "100kib" # 102400 bytes
my_bytes => "180 mb" # 180000000 bytes
```

#### Codec

编解码器是用于呈现数据的Logstash编解码器的名称。编解码器可用于输入和输出。

输入编解码器提供在输入数据之前解码数据的便捷方式。输出编解码器提供了在数据离开输出之前进行编码的便捷方式。输入或输出编解码器使用户无需在Logstash管道中使用单独的过滤器。

可以在  [第20章-编解码器Codec](../20-Coder-plugins/README.md) 页面找到可用的编解码器。

例：

```yaml
codec =>"json"
```

#### Hash

哈希是以 `"field1" => "value1"` 格式指定的键值对的集合。请注意，多个键值条目使用空格分隔，而不是逗号。

例：

```yaml
match => {
  "field1" => "value1"
  "field2" => "value2"
  ...
}
```

#### Number

数字必须是有效的数值（浮点或整数）。

例：

```yaml
port => 33
```

#### Password

密码是一个具有值但不被打印或作为日志输出的字符串。

例：

```yaml
my_password =>"密码"
```

#### URI

URI可以是完整的URL，如 *http://elastic.co/* ，也可以是像 *[foobar](foobar)* 这样的简单标识符。如果URI包含密码，例如 *http://user:pass@example.net*，则不会记录或打印URI的密码部分。

例：

```yaml
my_uri => "http://foo:bar@example.net"
```

#### Path

路径是表示有效操作系统路径的字符串。

例：

```yaml
my_path => "/tmp/logstash"
```

#### String

字符串必须是单个字符序列。请注意，字符串值需用引号包括起来，单引号或双引号都可以。

#### Escape Sequences

默认情况下，不启用转义序列。如果您希望在带引号的字符串中使用转义序列，则需要在logstash.yml中设置 `config.support_escapes: true`。如果为 `true`，则引用的字符串（单引号或双引号）将进行转义：

| 文本 | 转义结果          |
| ---- | ----------------- |
| \r   | 回车符 (ASCII 13) |
| \n   | 换行符 (ASCII 10) |
| \t   | 制表符 (ASCII 9)  |
| \\\  | 反斜杠 (ASCII 92) |
| \\"  | 双引号 (ASCII 34) |
| \\'  | 单引号 (ASCII 39) |

例：

```yaml
name => "Hello world"
name => 'It\'s a beautiful day'
```

### Comments

注释与perl、ruby和python中的注释相同。注释以#字符开头，不需要位于一行的开头。

例：

```yaml
# 这是注释
input { # 注释可以用在行末
  # ...
}
```

