## 管道聚合
管道聚合处理其他聚合而不是文档集的输出，将信息添加到输出树。有许多不同类型的管道聚合，每个都计算来自其他聚合的不同信息，但这些类型可以分为两个系列：

**父聚合**
    一系列管道聚合，与其父聚合的输出一起提供，并且能够计算要添加到现有存储桶的新存储桶或新聚合。

**兄弟聚合**
    与兄弟聚合的输出一起提供的管道聚合，并且能够计算与兄弟聚合处于同一级别的新聚合。
    
管道聚合可以通过使用`buckets_path`参数指示所需指标的路径，来引用执行计算所需的聚合。可以在下面的[`bucketets_path`语法](#bucketspath语法)部分中找到定义这些路径的语法。

管道聚合不能具有子聚合，但根据类型，它可以引用`buckets_path`中的另一个管道，从而允许链接管道聚合。例如，您可以将两个导数链接在一起以计算二阶导数（即导数的导数）。

>**注意**
>由于管道聚合仅添加到输出，因此在链接管道聚合时，每个管道聚合的输出将包含在最终输出中。

### buckets_path语法
大多数管道聚合需要另一个聚合作为输入。输入聚合是通过`buckets_path`参数定义的，该参数遵循特定格式：

```
AGG_SEPARATOR       =  '>' ;
METRIC_SEPARATOR    =  '.' ;
AGG_NAME            =  <the name of the aggregation> ;
METRIC              =  <the name of the metric (in case of multi-value metrics aggregation)> ;
PATH                =  <AGG_NAME> [ <AGG_SEPARATOR>, <AGG_NAME> ]* [ <METRIC_SEPARATOR>, <METRIC> ] ;
```

例如，路径`"my_bucket>my_stats.avg"`将路径指向`"my_stats"`指标中的`avg`值，该指标包含在`"my_bucket"`存储桶聚合中。

路径是相对于管道聚合的位置，它们不是绝对路径，并且路径不能回到“聚合树”。例如，此移动的平均值嵌入在`date_histogram`中，并引用一个"sibling"指标`"the_sum"`：

```sh
POST /_search
{
    "aggs": {
        "my_date_histo":{
            "date_histogram":{
                "field":"timestamp",
                "interval":"day"
            },
            "aggs":{
                "the_sum":{
                    "sum":{ "field": "lemmings" } ①
                },
                "the_movavg":{
                    "moving_avg":{ "buckets_path": "the_sum" } ②
                }
            }
        }
    }
}
```

① 该指标称为`"the_sum"`

② `buckets_path`通过相对路径`"the_sum"`引用指标

`buckets_path`也用于兄弟管道聚合，其中聚合是“一系列”桶的“下一个”，而不是嵌入它们“内部”。例如，`max_bucket`聚合使用`buckets_path`指定嵌入在兄弟聚合中的度量：

```sh
POST /_search
{
    "aggs" : {
        "sales_per_month" : {
            "date_histogram" : {
                "field" : "date",
                "interval" : "month"
            },
            "aggs": {
                "sales": {
                    "sum": {
                        "field": "price"
                    }
                }
            }
        },
        "max_monthly_sales": {
            "max_bucket": {
                "buckets_path": "sales_per_month>sales" ①
            }
        }
    }
}
```

① `buckets_path`指示我们想要`sales_per_month`日期直方图中销售聚合的最大值的`max_bucket`聚合。

### 特别路径
`buckets_path`可以使用特殊的`"_count"`路径，而不是路径到指标。这指示管道聚合使用文档计数作为其输入。例如，可以根据每个存储桶的文档计数计算移动平均值，而不是特定指标：

```sh
POST /_search
{
    "aggs": {
        "my_date_histo": {
            "date_histogram": {
                "field":"timestamp",
                "interval":"day"
            },
            "aggs": {
                "the_movavg": {
                    "moving_avg": { "buckets_path": "_count" } ①
                }
            }
        }
    }
}
```

① 通过使用`_count`而不是度量名称，我们可以计算直方图中文档计数的移动平均值

`buckets_path`还可以使用`"_bucket_count"`和多桶聚合的路径来使用该聚合，在管道聚合中返回的桶数而不是指标。例如，这里可以使用`bucket_selector`来过滤掉不包含内部术语聚合的桶的桶：

```sh
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "histo": {
      "date_histogram": {
        "field": "date",
        "interval": "day"
      },
      "aggs": {
        "categories": {
          "terms": {
            "field": "category"
          }
        },
        "min_bucket_selector": {
          "bucket_selector": {
            "buckets_path": {
              "count": "categories._bucket_count" ①
            },
            "script": {
              "source": "params.count != 0"
            }
          }
        }
      }
    }
  }
}
```

① 通过使用`_bucket_count`而不是指标名称，我们可以过滤掉那些不包含用于`categories`聚合的桶的组合桶

### 处理聚合名中的点
支持替代语法，以处理名称中具有点的聚合或指标，例如第`99.9`[百分位](../07-Aggregations/Metrics-Aggregations/Percentiles-Aggregation.md)。该指标可称为：

```
"buckets_path": "my_percentile[99.9]"
```

### 处理数据间隙
现实世界中的数据通常有噪音，有时会包含间隙——数据不存在的地方。这可能有各种原因，最常见的是：

- 落入存储桶的文档不包含必填字段
- 一个或多个存储桶中没有与查询匹配的文档
- 指标计算无法生成值，可能是因为依赖的桶缺少值。某些管道聚合必须满足一定的要求（例如，导数无法计算第一个值的指标，因为没有先前的值，HoltWinters移动平均值需要“预热”数据才能开始计算等）

当遇到间隙或丢失数据时，间隙策略是一种向管道聚合通知所需行为的机制。所有管道聚合都接受`gap_policy`参数。目前有两种差距政策可供选择：

**skip**
    此选项将丢失的数据视为存储桶不存在。它将跳过存储桶并继续使用下一个可用值进行计算。

**insert_zeros**
    此选项将使用零（`0`）替换缺失值，并且管道聚合计算将正常进行。
