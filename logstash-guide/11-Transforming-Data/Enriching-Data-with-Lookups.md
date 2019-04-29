## 丰富数据

这些插件可以帮助你通过增加额外信息来丰富数据，比如GeoIP和客户端信息

### 插件

[**dns filter**](../19-Filter-plugins/dns.md)

dns过滤器插件执行标准或反向DNS查找。

以下配置对 `source_host` 字段中的地址执行反向查找，并将其替换为域名：

```sh
filter {
  dns {
    reverse => [ "source_host" ]
    action => "replace"
  }
}
```

[**elasticsearch filter**](../19-Filter-plugins/elasticsearch.md)

elasticsearch过滤器将Elasticsearch中以前日志事件的字段复制到当前事件。

以下配置显示了如何使用此过滤器的完整示例。每当Logstash收到"end"事件时，它都会使用此Elasticsearch过滤器根据某个操作标识符查找匹配的"start"事件。然后，它将"start"事件中的 `@timestamp` 字段复制到"end"事件的新字段中。最后，使用日期过滤器和ruby过滤器的组合，示例中的代码计算两个事件之间的持续时间（以小时为单位）。

```yaml
  if [type] =="end"{
     elasticsearch {
        hosts => ["es-server"]
        query =>"type：start AND operation：％{[opid]}"
        fields => {"@ timestamp"=>"已开始"}
     }
     日期{
        match => ["[开始]"，"ISO8601"]
        target =>"[开始]"
     }
     红宝石{
        code =>'event.set（"duration_hrs"，（event.get（"@ timestamp"） -  event.get（"started"））/ 3600）rescue nil'
    }
  }
```
[**geoip filter**](../19-Filter-plugins/geoip.md)

geoip过滤器添加有关IP地址位置的地理信息。例如：

```sh
filter {
  geoip {
    source => "clientip"
  }
}
```

应用geoip过滤器后，将使用geoip字段丰富事件。例如：

```sh
filter {
  geoip {
    source => "clientip"
  }
}
```

[**http filter**](../19-Filter-plugins/http.md)

http过滤器与外部Webservice/REST API集成，并支持对任何HTTP服务或端点进行数据丰富。此插件适合许多丰富数据的场景，例如社交API，情感API，安全提要API和业务服务API。

[**jdbc_static filter**](../19-Filter-plugins/jdbc_static.md)

jdbc_static过滤器使用从远程数据库预加载的数据来丰富事件。

以下示例从远程数据库中提取数据，将其缓存在本地数据库中，并使用本地数据库中缓存的数据来丰富事件。

```yaml
filter {
  jdbc_static {
    loaders => [ ①
      {
        id => "remote-servers"
        query => "select ip, descr from ref.local_ips order by ip"
        local_table => "servers"
      },
      {
        id => "remote-users"
        query => "select firstname, lastname, userid from ref.local_users order by userid"
        local_table => "users"
      }
    ]
    local_db_objects => [ ②
      {
        name => "servers"
        index_columns => ["ip"]
        columns => [
          ["ip", "varchar(15)"],
          ["descr", "varchar(255)"]
        ]
      },
      {
        name => "users"
        index_columns => ["userid"]
        columns => [
          ["firstname", "varchar(255)"],
          ["lastname", "varchar(255)"],
          ["userid", "int"]
        ]
      }
    ]
    local_lookups => [ ③
      {
        id => "local-servers"
        query => "select descr as description from servers WHERE ip = :ip"
        parameters => {ip => "[from_ip]"}
        target => "server"
      },
      {
        id => "local-users"
        query => "select firstname, lastname from users WHERE userid = :id"
        parameters => {id => "[loggedin_userid]"}
        target => "user" ④
      }
    ]
    # using add_field here to add & rename values to the event root
    add_field => { server_name => "%{[server][0][description]}" }
    add_field => { user_firstname => "%{[user][0][firstname]}" } ⑤
    add_field => { user_lastname => "%{[user][0][lastname]}" } ⑥
    remove_field => ["server", "user"]
    jdbc_user => "logstash"
    jdbc_password => "example"
    jdbc_driver_class => "org.postgresql.Driver"
    jdbc_driver_library => "/tmp/logstash/vendor/postgresql-42.1.4.jar"
    jdbc_connection_string => "jdbc:postgresql://remotedb:5432/ls_test_2"
  }
}
```

![1](../source/images/common/1.png) 查询外部数据库，获取将缓存在本地的数据集。

![2](../source/images/common/2.png) 定义用于构建本地数据库结构的列，类型和索引。列名和类型应与外部数据库匹配。

![3](../source/images/common/3.png) 在本地数据库上执行查询查询以丰富事件。

![4](../source/images/common/4.png) 指定将存储查找数据的事件字段。如果查找返回多个列，则数据将作为JSON对象存储在字段中。

![5](../source/images/common/5.png) ![6](../source/images/common/6.png) 从JSON对象获取数据并将其存储在顶级事件字段中，以便在Kibana中进行分析。

[**jdbc_streaming filter**](../19-Filter-plugins/jdbc_streaming.md)

jdbc_streaming过滤器使用数据库数据丰富事件。

以下示例执行SQL查询并将结果集存储在名为country_details的字段中：

```sh
filter {
  jdbc_streaming {
    jdbc_driver_library => "/path/to/mysql-connector-java-5.1.34-bin.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydatabase"
    jdbc_user => "me"
    jdbc_password => "secret"
    statement => "select * from WORLD.COUNTRY WHERE Code = :code"
    parameters => { "code" => "country_code"}
    target => "country_details"
  }
}
```

[**memcached filter**](../19-Filter-plugins/memcached.md)

memcached过滤器可以对Memcached对象缓存系统进行键/值查找丰富。它支持读取（GET）和写入（SET）操作。它是安全分析用例的显着补充。

[**translate filter**](../19-Filter-plugins/translate.md)

translate过滤器根据散列或文件中指定的替换值替换字段内容。目前支持以下文件类型：YAML，JSON和CSV。

以下示例获取response_code字段的值，将其转换为基于字典中指定的值的描述，然后从事件中删除response_code字段：

```sh
filter {
  translate {
    field => "response_code"
    destination => "http_response"
    dictionary => {
      "200" => "OK"
      "403" => "Forbidden"
      "404" => "Not Found"
      "408" => "Request Timeout"
    }
    remove_field => "response_code"
  }
}
```

[**useragent filter**](../19-Filter-plugins/useragent.md)

useragent过滤器将客户端字符串解析为字段。

以下示例获取代理字段中的用户代理字符串，将其解析为用户代理字段，并将用户代理字段添加到名为user_agent的新字段中。它还会删除原始 `agent` 字段：

```sh
filter {
  useragent {
    source => "agent"
    target => "user_agent"
    remove_field => "agent"
  }
}
```

应用过滤器后，将使用客户端字段丰富事件。例如：

```json
        "user_agent": {
          "os": "Mac OS X 10.12",
          "major": "50",
          "minor": "0",
          "os_minor": "12",
          "os_major": "10",
          "name": "Firefox",
          "os_name": "Mac OS X",
          "device": "Other"
        }
```
