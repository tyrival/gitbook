## 查询

请求体中的query元素允许使用[第11章 查询DSL](../../11-Query-DSL/README.md)定义一个查询

```bash
GET /_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

