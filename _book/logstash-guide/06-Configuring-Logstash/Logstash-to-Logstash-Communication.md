## 实例间通信

您可以通过将Lumberjack输出连接到Beats输入，来设置两个Logstash实例之间的通信。例如，如果数据路径跨越网络或防火墙边界，则可能需要使用此配置。反之，如果不是迫切需要Logstash-to-Logstash通信，那么请不要实现它。

如果要在一个Logstash实例中查找多个管道的连接信息，请参阅 [管道间通信](../06-Configuring-Logstash/Pipeline-to-Pipeline-Communication.md)。

### 配置概述

使用Lumberjack协议连接两台Logstash机器。

- 生成受信任的SSL证书（Lumberjack协议要求）。
- 将SSL证书复制到上游Logstash计算机。
- 将SSL证书和密钥复制到下游Logstash计算机。
- 设置上游Logstash机器以使用Lumberjack输出发送数据。
- 设置下游Logstash机器以通过Beats输入侦听传入的Lumberjack连接。
- 测试一下。

#### 生成自签名SSL证书和key

某些操作系统可以使用 `openssl req` 命令生成自签名证书和密钥，另一些则可能需要手工安装openssl命令行程序。

运行以下命令：

```shell
openssl req -x509 -batch -nodes -newkey rsa:2048 -keyout lumberjack.key -out lumberjack.cert -subj /CN=localhost
```

此处：

- `lumberjack.key` 是要创建的SSL密钥的名称
- `lumberjack.cert` 是要创建的SSL证书的名称
- `localhost` 是上游Logstash计算机的名称

此命令生成类似于以下内容的输出：

生成2048位RSA私钥

```shell
Generating a 2048 bit RSA private key
.................................+++
....................+++
writing new private key to 'lumberjack.key'
```

#### 复制SSL证书和key

- 将SSL证书复制到上游Logstash计算机。
- 将SSL证书和密钥复制到下游Logstash计算机。

#### 启动上游Logstash实例

启动Logstash并生成测试事件：

```shell
bin/logstash -e 'input { generator { count => 5 } } output { lumberjack { codec => json hosts => "mydownstreamhost" ssl_certificate => "lumberjack.cert" port => 5000 } }'
```

此示例命令使用提供的SSL证书向 `下游服务器:5000`发送五个事件。

#### 启动下游Logstash实例
启动Logstash的下游实例：

```shell
bin/logstash -e 'input { beats { codec => json port => 5000 ssl => true ssl_certificate => "lumberjack.cert" ssl_key => "lumberjack.key"} }'
```

此示例命令设置侦听端口5000作为Beats输入。

### 验证通信

观察下游Logstash获取传入事件。您应该看到五个类似于如下的递增事件：

```shell
{
  "@timestamp" => 2018-02-07T12:16:39.415Z,
  "sequence"   => 0
  "tags"       => [
    [0] "beats_input_codec_json_applied"
  ],
  "message"    => "Hello world",
  "@version"   => "1",
  "host"       => "ls1.semicomplete.com"
}
```

如果您看到具有一致字段和格式的所有五个事件，依次递增，那么您的配置是正确的。
