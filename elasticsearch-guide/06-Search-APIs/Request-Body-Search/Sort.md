## 排序

您可以在指定字段上添加一个或多个排序，包括升序或降序。排序是在每个字段级别定义的，可使用内置的字段`_score`进行按分数排序，以及字段`_doc`按索引排序。

假设有以下索引映射：

```bash
PUT /my_index
{
    "mappings": {
        "_doc": {
            "properties": {
                "post_date": { "type": "date" },
                "user": {
                    "type": "keyword"
                },
                "name": {
                    "type": "keyword"
                },
                "age": { "type": "integer" }
            }
        }
    }
}
```

```bash
GET /my_index/_search
{
    "sort" : [
        { "post_date" : {"order" : "asc"}},
        "user",
        { "name" : "desc" },
        { "age" : "desc" },
        "_score"
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

> **注意**
>
> `_doc`是性能最好的排序，除此之外没什么其他使用场景。因此，如果您不关心文档的返回顺序，那么您应该按`_doc`排序。这在[滚动](../Request-Body-Search/Scroll.md)时尤其有用。

### 排序值

每个文档的排序值也作为响应的一部分返回。

### 排序顺序

`order`选项有如下可选值：

| 值     | 说明 |
| ------ | ---- |
| `asc`  | 升序 |
| `desc` | 降序 |

`_score`的默认为倒序，其他属性默认为顺序。

### 排序模式选项

Elasticsearch支持数组或多值字段进行排序。 `mode`选项控制选择哪个数组值进行排序。 `mode`选项可以具有以下值：

| 值       | 说明                                       |
| -------- | ------------------------------------------ |
| `min`    | 取最小值                                   |
| `max`    | 取最大值                                   |
| `sum`    | 所有值的和，仅适用于基于数字的数组字段     |
| `avg`    | 所有值的平均值，仅适用于基于数字的数组字段 |
| `median` | 所有值的中位数，仅适用于基于数字的数组字段 |

升序排序中的默认排序模式为`min`  - 取最小值。降序的默认排序模式是`max`  -取最大值。

### 排序模式示例

在下面的示例中，每个文档的价格字段有多个价格。在这种情况下，结果将根据每个文档的平均价格进行升序排序。

```bash
PUT /my_index/_doc/1?refresh
{
   "product": "chocolate",
   "price": [20, 4]
}

POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```

### 嵌套对象排序

Elasticsearch还支持按一个或多个嵌套对象内的字段进行排序。`nested`字段的嵌套排序选项，具有以下属性：

###### `path`

​	定义要排序的嵌套对象。实际的排序字段必须是此嵌套对象的直接属性，而不是嵌套属性。按嵌套字段排序时，此字段必填。

###### `filter`
​	嵌套路径的内部对象应匹配的过滤器，以便通过排序考虑其字段值。常见的情况是在嵌套过滤器或查询中重复查询/过滤。默认情况下，没有`nested_filter`处于活动状态。

###### `max_children`

​	选择排序值时每个根文档要考虑的最大子项数。默认为无限制。

###### `nested`

​	与顶级`nested`相同，但适用于当前嵌套对象中的另一个嵌套路径。

> **警告**
> 不推荐使用Elasticseach 6.1之前的嵌套排序选项`nested_path`和`nested_filter`，请使用上面的选项。

### 嵌套排序示例

在下面的示例中，`offer`是`nested`类型的字段。需要指定嵌套路径；否则，Elasticsearch不知道需要捕获哪些嵌套排序值。

```bash
POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
       {
          "offer.price" : {
             "mode" :  "avg",
             "order" : "asc",
             "nested": {
                "path": "offer",
                "filter": {
                   "term" : { "offer.color" : "blue" }
                }
             }
          }
       }
    ]
}
```

在下面的示例中，`parent`字段和`child`字段的类型是`nested`。需要在每个级别指定`nested_path`；否则，Elasticsearch不知道需要捕获哪些嵌套级别排序值。

```bash
POST /_search
{
   "query": {
      "nested": {
         "path": "parent",
         "query": {
            "bool": {
                "must": {"range": {"parent.age": {"gte": 21}}},
                "filter": {
                    "nested": {
                        "path": "parent.child",
                        "query": {"match": {"parent.child.name": "matt"}}
                    }
                }
            }
         }
      }
   },
   "sort" : [
      {
         "parent.child.age" : {
            "mode" :  "min",
            "order" : "asc",
            "nested": {
               "path": "parent",
               "filter": {
                  "range": {"parent.age": {"gte": 21}}
               },
               "nested": {
                  "path": "parent.child",
                  "filter": {
                     "match": {"parent.child.name": "matt"}
                  }
               }
            }
         }
      }
   ]
}
```

通过脚本排序和按地理距离排序时，也支持嵌套排序。

### `missing`值

`missing`参数指定如何处理缺少排序字段的文档：`missing`值可以设置为`_last`，`_first`或自定义值（将用于缺失文档的排序值）。默认为`_last`。

例如：

```bash
GET /_search
{
    "sort" : [
        { "price" : {"missing" : "_last"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}
```

> **注意**
> 如果嵌套的内部对象与`nested_filter`不匹配，则使用缺失值。

### 忽略未映射字段

默认情况下，如果没有与字段关联的映射，搜索请求将失败。 `unmapped_type`选项允许您忽略没有映射字段的文档，并且不按其排序。此参数的值用于确定排序值。以下是如何使用它的示例：

```bash
GET /_search
{
    "sort" : [
        { "price" : {"unmapped_type" : "long"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}
```

如果查询到的文档没有`price`映射路径，那么只要这类文档存在一个`long`类型的映射，Elasticsearch将继续处理它们。

### 地理距离排序

允许按`_geo_distance`排序。下面是一个示例，假设`pin.location`是`geo_point`类型的字段：

```bash
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : [-70, 40],
                "order" : "asc",
                "unit" : "km",
                "mode" : "min",
                "distance_type" : "arc",
                "ignore_unmapped": true
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

###### `distance_type`

​	如何计算距离。可以是`arc`（默认）或`plane`（更快，但在长距离和靠近极点时不准确）。
###### `mode`
​	如果某个字段有多个地理位置该怎么办。默认情况下，按升序排序时考虑最短距离，按降序排序时考虑最长距离。支持的值是`min`，`max`，`median`和`avg`。

###### `unit`

​	计算排序值时使用的单位。默认值为`m`（米）。

###### `ignore_unmapped`

​	指示是否应将未映射的字段视为缺失值。将其设置为`true`等效于在字段排序中指定`unmapped_type`。默认值为`false`（未映射的字段会导致搜索失败）。

> **注意**
>
> 地理距离排序不支持可配置的缺失值：当文档没有用于距离计算的字段的值时，距离将始终被视为等于无穷大。

提供坐标时支持以下格式：

#### Lat Lon作为属性

```bash
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : {
                    "lat" : 40,
                    "lon" : -70
                },
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

#### Lat Lon作为字符串

`lat, lon`格式

```bash
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : "40,-70",
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

#### Geohash

```bash
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : "drm3btev3e86",
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

#### Lat Lon作为数组

格式为`[lat, lon]`，注意，这里lon/lat的顺序符合[GeoJSON](https://geojson.org)

```bash
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : [-70, 40],
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

### 多参考点

例如，可以将多个地理点作为包含任何`geo_point`格式的数组传递

```bash
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : [[-70, 40], [-71, 42]],
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

文档的最终距，离将是文档中包含的所有点，与排序请求中给出的所有点的`min`/`max`/`avg`（通过`mode`定义）距离。

### 基于脚本的排序

允许基于自定义脚本进行排序，这是一个示例：

```bash
GET /_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    },
    "sort" : {
        "_script" : {
            "type" : "number",
            "script" : {
                "lang": "painless",
                "source": "doc['field_name'].value * params.factor",
                "params" : {
                    "factor" : 1.1
                }
            },
            "order" : "asc"
        }
    }
}
```

### 跟踪分数

在字段上排序时，不计算分数。通过将`track_scores`设置为`true`，仍将计算和跟踪分数。

```bash
GET /_search
{
    "track_scores": true,
    "sort" : [
        { "post_date" : {"order" : "desc"} },
        { "name" : "desc" },
        { "age" : "desc" }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

### 内存考虑因素

排序时，相关的排序字段值将加载到内存中。这意味着每个分片应该有足够的内存来包含它们。对于基于字符串的类型，不应分析/标记化排序的字段。对于数字类型，如果可能，建议将类型显式设置为较窄的类型（如`short`，`integer`和`float`）。
