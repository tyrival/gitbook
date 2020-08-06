## 分页

可以使用`from`和`size`参数实现查询结果的分页。`from`参数定义要获取的第一个结果的偏移量。`size`参数允许您配置要返回的最大命中数。

`from`和`size`可以设置为请求参数，也可以在搜索主体中设置。`from`默认值为`0`，`size`默认为`10`。

```
GET /_search
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

请注意，`from + size`不能超过`index.max_result_window`索引设置，默认为10,000。有关更有效的深度滚动方法，请参阅[滚动](../Request-Body-Search/Scroll.md)或[Search-After](../Request-Body-Search/Search-After.md) API。
