## Source过滤

允许控制每次命中返回`_source`字段的方式。

默认情况下，操作将返回`_source`字段的内容，除非您已使用`stored_fields`参数或已禁用`_source`字段。

您可以使用`_source`参数关闭`_source`检索：

要禁用`_source`检索设置为`false`：

```bash
GET /_search
{
    "_source": false,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

`_source`还接受一个或多个通配符模式来控制应该返回_source的哪些部分：

例如：

```bash
GET /_search
{
    "_source": "obj.*",
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

或

```bash
GET /_search
{
    "_source": [ "obj1.*", "obj2.*" ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

最后，你可以指定`includes`和`excludes`匹配来实现完全控制：

```bash
GET /_search
{
    "_source": {
        "includes": [ "obj1.*", "obj2.*" ],
        "excludes": [ "*.description" ]
    },
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

