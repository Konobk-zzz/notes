# Pipeline aggregations

Pipeline aggregations work on the outputs produced from other aggregations rather than from document sets, adding information to the output tree. There are many different types of pipeline aggregation, each computing different information from other aggregations, but these types can be broken down into two families:

管道聚合处理从其他聚合而不是文档集产生的输出，将信息添加到输出树。有许多不同类型的管道聚合，每种聚合都计算来自其他聚合的不同信息，但这些类型可以分为两大类：

- ***Parent\***

  A family of pipeline aggregations that is provided with the output of its parent aggregation and is able to compute new buckets or new aggregations to add to existing buckets.

  一系列管道聚合，随其父聚合的输出一起提供，并且能够计算新存储桶或新聚合以添加到现有存储桶。

- ***Sibling\***

  Pipeline aggregations that are provided with the output of a sibling aggregation and are able to compute a new aggregation which will be at the same level as the sibling aggregation.

  管道聚合随同级聚合的输出一起提供，并且能够计算与同级聚合处于同一级别的新聚合。

Pipeline aggregations can reference the aggregations they need to perform their computation by using the `buckets_path` parameter to indicate the paths to the required metrics. The syntax for defining these paths can be found in the [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) section below.

管道聚合可以通过使用`buckets_path` 参数指示所需指标的路径来引用它们执行计算所需的聚合。定义这些路径的语法可以在下面的[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)部分找到 。

Pipeline aggregations cannot have sub-aggregations but depending on the type it can reference another pipeline in the `buckets_path` allowing pipeline aggregations to be chained. For example, you can chain together two derivatives to calculate the second derivative (i.e. a derivative of a derivative).

管道聚合不能有子聚合，但根据类型，它可以在`buckets_path` 允许链接管道聚合时引用另一个管道。例如，您可以将两个导数链接在一起以计算二阶导数（即导数的导数）。

> NOTE: Because pipeline aggregations only add to the output, when chaining pipeline aggregations the output of each pipeline aggregation will be included in the final output.
>
> 由于管道聚合仅添加到输出，因此在链接管道聚合时，每个管道聚合的输出将包含在最终输出中。

### `buckets_path` Syntax

Most pipeline aggregations require another aggregation as their input. The input aggregation is defined via the `buckets_path` parameter, which follows a specific format:

大多数管道聚合需要另一个聚合作为它们的输入。输入聚合是通过`buckets_path` 参数定义的，它遵循特定的格式：

```ebnf
AGG_SEPARATOR       =  `>` ;
METRIC_SEPARATOR    =  `.` ;
AGG_NAME            =  <the name of the aggregation> ;
METRIC              =  <the name of the metric (in case of multi-value metrics aggregation)> ;
MULTIBUCKET_KEY     =  `[<KEY_NAME>]`
PATH                =  <AGG_NAME><MULTIBUCKET_KEY>? (<AGG_SEPARATOR>, <AGG_NAME> )* ( <METRIC_SEPARATOR>, <METRIC> ) ;
```

For example, the path `"my_bucket>my_stats.avg"` will path to the `avg` value in the `"my_stats"` metric, which is contained in the `"my_bucket"` bucket aggregation.

例如，路径`"my_bucket>my_stats.avg"`将指向度量值中的`avg`值`"my_stats"`，该值包含在`"my_bucket"`存储桶聚合中。

Paths are relative from the position of the pipeline aggregation; they are not absolute paths, and the path cannot go back "up" the aggregation tree. For example, this derivative is embedded inside a date_histogram and refers to a "sibling" metric `"the_sum"`:

路径相对于管道聚合的位置；它们不是绝对路径，并且路径不能回到聚合树的“上层”。例如，此导数嵌入在 date_histogram 中并引用“兄弟”度量`"the_sum"`：

```console
POST /_search
{
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "lemmings" }              (1)
        },
        "the_deriv": {
          "derivative": { "buckets_path": "the_sum" } (2)
        }
      }
    }
  }
}
```

1. The metric is called `"the_sum"`

   该指标称为 `"the_sum"`

2. The `buckets_path` refers to the metric via a relative path `"the_sum"`

   在`buckets_path`通过相对路径是指该度量`"the_sum"`

`buckets_path` is also used for Sibling pipeline aggregations, where the aggregation is "next" to a series of buckets instead of embedded "inside" them. For example, the `max_bucket` aggregation uses the `buckets_path` to specify a metric embedded inside a sibling aggregation:

`buckets_path`也用于兄弟管道聚合，其中聚合位于一系列存储桶的“旁边”，而不是嵌入“内部”。例如，`max_bucket`聚合使用`buckets_path`来指定嵌入在同级聚合中的度量：

```console
POST /_search
{
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
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
        "buckets_path": "sales_per_month>sales" (1)
      }
    }
  }
}
```

1. `buckets_path` instructs this max_bucket aggregation that we want the maximum value of the `sales` aggregation in the `sales_per_month` date histogram.

   `buckets_path`指示这个 max_bucket 聚合我们想要日期直方图中`sales`聚合 的最大值`sales_per_month`。

If a Sibling pipeline agg references a multi-bucket aggregation, such as a `terms` agg, it also has the option to select specific keys from the multi-bucket. For example, a `bucket_script` could select two specific buckets (via their bucket keys) to perform the calculation:

如果兄弟管道 agg 引用了多桶聚合，例如`terms`agg，它还可以选择从多桶中选择特定键。例如，a`bucket_script`可以选择两个特定的桶（通过它们的桶键）来执行计算：

```console
POST /_search
{
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sale_type": {
          "terms": {
            "field": "type"
          },
          "aggs": {
            "sales": {
              "sum": {
                "field": "price"
              }
            }
          }
        },
        "hat_vs_bag_ratio": {
          "bucket_script": {
            "buckets_path": {
              "hats": "sale_type['hat']>sales",   (1)
              "bags": "sale_type['bag']>sales"    (2)
            },
            "script": "params.hats / params.bags"
          }
        }
      }
    }
  }
}
```

1. `buckets_path` selects the hats and bags buckets (via `['hat']`/`['bag']``) to use in the script specifically, instead of fetching all the buckets from `sale_type` aggregation

   `buckets_path`选择帽子和袋子桶（通过`['hat']`/ `['bag']``）专门在脚本中使用，而不是从`sale_type`聚合中获取所有桶

###  Special Paths

Instead of pathing to a metric, `buckets_path` can use a special `"_count"` path. This instructs the pipeline aggregation to use the document count as its input. For example, a derivative can be calculated on the document count of each bucket, instead of a specific metric:

`buckets_path`可以使用特殊`"_count"`路径而不是指向度量标准。这指示管道聚合使用文档计数作为其输入。例如，可以根据每个桶的文档计数计算导数，而不是特定的度量：

```console
POST /_search
{
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      },
      "aggs": {
        "the_deriv": {
          "derivative": { "buckets_path": "_count" } (1)
        }
      }
    }
  }
}
```

1. By using `_count` instead of a metric name, we can calculate the derivative of document counts in the histogram

   通过使用`_count`而不是度量名称，我们可以计算直方图中文档计数的导数

The `buckets_path` can also use `"_bucket_count"` and path to a multi-bucket aggregation to use the number of buckets returned by that aggregation in the pipeline aggregation instead of a metric. For example, a `bucket_selector` can be used here to filter out buckets which contain no buckets for an inner terms aggregation:

该`buckets_path`还可以使用`"_bucket_count"`和路径多桶聚集使用由管道中的聚集，而不是度量聚合返回桶的数目。例如，`bucket_selector`这里可以使用a过滤掉不包含内部术语聚合的桶的桶：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
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
              "count": "categories._bucket_count" (1)
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

1.  By using `_bucket_count` instead of a metric name, we can filter out `histo` buckets where they contain no buckets for the `categories` aggregation

   通过使用`_bucket_count`而不是指标名称，我们可以过滤掉`histo`不包含用于`categories`聚合的存储桶的存储桶

### Dealing with dots in agg names

An alternate syntax is supported to cope with aggregations or metrics which have dots in the name, such as the `99.9`th [percentile](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html). This metric may be referred to as:

支持另一种语法来处理名称中带有点的聚合或度量，例如`99.9`th [percentile](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html)。该指标可称为：

```console
"buckets_path": "my_percentile[99.9]"
```

### Dealing with gaps in the data

Data in the real world is often noisy and sometimes contains **gaps** — places where data simply doesn’t exist. This can occur for a variety of reasons, the most common being:

现实世界中的数据通常是嘈杂的，有时还包含**差距** ——数据根本不存在的地方。发生这种情况的原因有多种，最常见的是：

- Documents falling into a bucket do not contain a required field

  落入存储桶的文档不包含必填字段

- There are no documents matching the query for one or more buckets

  没有与一个或多个存储桶的查询匹配的文档

- The metric being calculated is unable to generate a value, likely because another dependent bucket is missing a value. Some pipeline aggregations have specific requirements that must be met (e.g. a derivative cannot calculate a metric for the first value because there is no previous value, HoltWinters moving average need "warmup" data to begin calculating, etc)

  正在计算的指标无法生成值，可能是因为另一个相关存储桶缺少值。某些管道聚合具有必须满足的特定要求（例如，导数无法计算第一个值的度量，因为没有先前的值，HoltWinters 移动平均需要“预热”数据才能开始计算等）

Gap policies are a mechanism to inform the pipeline aggregation about the desired behavior when "gappy" or missing data is encountered. All pipeline aggregations accept the `gap_policy` parameter. There are currently two gap policies to choose from:

间隙策略是一种机制，用于在遇到“间隙”或丢失数据时通知管道聚合所需的行为。所有管道聚合都接受该`gap_policy`参数。目前有两种差距政策可供选择：

- ***skip\***

  This option treats missing data as if the bucket does not exist. It will skip the bucket and continue calculating using the next available value.

  此选项将丢失的数据视为存储桶不存在。它将跳过存储桶并使用下一个可用值继续计算。

- ***insert_zeros\***

  This option will replace missing values with a zero (`0`) and pipeline aggregation computation will proceed as normal.

  此选项将用零 ( `0`)替换缺失值，并且管道聚合计算将照常进行。

- ***keep_values\***

  This option is similar to skip, except if the metric provides a non-null, non-NaN value this value is used, otherwise the empty bucket is skipped.

  此选项类似于跳过，除非指标提供非空、非 NaN 值，否则将使用此值，否则将跳过空存储桶。

#  Average bucket aggregation

A sibling pipeline aggregation which calculates the mean value of a specified metric in a sibling aggregation. The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.

同级管道聚合，用于计算同级聚合中指定指标的平均值。指定的指标必须是数字，同级聚合必须是多桶聚合。

###  Syntax

```js
"avg_bucket": {
  "buckets_path": "sales_per_month>sales",
  "gap_policy": "skip",
  "format": "#,##0.00;(#,##0.00)"
}
```

###  Parameters

**`buckets_path`**

(Required, string) Path to the buckets to average. For syntax, see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax).

（必需，字符串）要平均的桶的路径。有关语法，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)。

**`gap_policy`**

(Optional, string) Policy to apply when gaps are found in the data. For valid values, see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy). Defaults to `skip`.

（可选，字符串）在数据中发现差距时应用的策略。有关有效值，请参阅 [处理数据中的间隙](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)。默认为`skip`.

**`format`**

(Optional, string) [DecimalFormat pattern](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DecimalFormat.html) for the output value. If specified, the formatted value is returned in the aggregation’s `value_as_string` property.

（可选，字符串） 输出值的[DecimalFormat 模式](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/text/DecimalFormat.html)。如果指定，则在聚合的`value_as_string`属性中返回格式化的值 。

###  Response body

**`value`**

(float) Mean average value for the metric specified in `buckets_path`.

(float) 中指定的度量的平均平均值`buckets_path`。

**`value_as_string`**

(string) Formatted output value for the aggregation. This property is only provided if a `format` is specified in the request.

（字符串）聚合的格式化输出值。只有`format`在请求中指定了a 时才提供此属性。

###  Example

The following `avg_monthly_sales` aggregation uses `avg_bucket` to calculate average sales per month:

以下`avg_monthly_sales`聚合用于`avg_bucket`计算每月的平均销售额：

```console
POST _search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    },
    "avg_monthly_sales": {
// tag::avg-bucket-agg-syntax[]               (1)
      "avg_bucket": {
        "buckets_path": "sales_per_month>sales",
        "gap_policy": "skip",
        "format": "#,##0.00;(#,##0.00)"
      }
// end::avg-bucket-agg-syntax[]               (2)
    }
  }
}
```

1.  Start of the `avg_bucket` configuration. Comment is not part of the example.

    开始`avg_bucket`配置。评论不是示例的一部分。

2.  End of the `avg_bucket` configuration. Comment is not part of the example.

   `avg_bucket`配置结束。评论不是示例的一部分。

The request returns the following response:

```console-result
{
  "took": 11,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "sales_per_month": {
      "buckets": [
        {
          "key_as_string": "2015/01/01 00:00:00",
          "key": 1420070400000,
          "doc_count": 3,
          "sales": {
            "value": 550.0
          }
        },
        {
          "key_as_string": "2015/02/01 00:00:00",
          "key": 1422748800000,
          "doc_count": 2,
          "sales": {
            "value": 60.0
          }
        },
        {
          "key_as_string": "2015/03/01 00:00:00",
          "key": 1425168000000,
          "doc_count": 2,
          "sales": {
            "value": 375.0
          }
        }
      ]
    },
    "avg_monthly_sales": {
      "value": 328.33333333333333,
      "value_as_string": "328.33"
    }
  }
}
```

#  Bucket script aggregation

A parent pipeline aggregation which executes a script which can perform per bucket computations on specified metrics in the parent multi-bucket aggregation. The specified metric must be numeric and the script must return a numeric value.

执行脚本的父管道聚合，该脚本可以对父多桶聚合中的指定指标执行每个桶计算。指定的度量必须是数字，并且脚本必须返回一个数字值。

###  Syntax

A `bucket_script` aggregation looks like this in isolation:

单独的`bucket_script`聚合看起来像这样：

```console
{
  "bucket_script": {
    "buckets_path": {
      "my_var1": "the_sum",                     (1)
      "my_var2": "the_value_count"
    },
    "script": "params.my_var1 / params.my_var2"
  }
}
```

1.  Here, `my_var1` is the name of the variable for this buckets path to use in the script, `the_sum` is the path to the metrics to use for that variable.

    此处，`my_var1`是要在脚本中使用的此存储桶路径的变量名称，`the_sum`是要用于该变量的指标的路径。

**Table 49. `bucket_script` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `script`       | The script to run for this aggregation. The script can be inline, file or indexed. (see [Scripting](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html) for more details)<br />为此聚合运行的脚本。脚本可以是内联的、文件的或索引的。（ 有关更多详细信息，请参阅[脚本](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html)） | Required |               |
| `buckets_path` | A map of script variables and their associated path to the buckets we wish to use for the variable (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />脚本变量的映射及其到我们希望用于变量的存储桶的关联路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

The following snippet calculates the ratio percentage of t-shirt sales compared to total sales each month:

以下代码段计算了 T 恤销售额与每月总销售额的比率百分比：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_sales": {
          "sum": {
            "field": "price"
          }
        },
        "t-shirts": {
          "filter": {
            "term": {
              "type": "t-shirt"
            }
          },
          "aggs": {
            "sales": {
              "sum": {
                "field": "price"
              }
            }
          }
        },
        "t-shirt-percentage": {
          "bucket_script": {
            "buckets_path": {
              "tShirtSales": "t-shirts>sales",
              "totalSales": "total_sales"
            },
            "script": "params.tShirtSales / params.totalSales * 100"
          }
        }
      }
    }
  }
}
```

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "total_sales": {
                   "value": 550.0
               },
               "t-shirts": {
                   "doc_count": 1,
                   "sales": {
                       "value": 200.0
                   }
               },
               "t-shirt-percentage": {
                   "value": 36.36363636363637
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "total_sales": {
                   "value": 60.0
               },
               "t-shirts": {
                   "doc_count": 1,
                   "sales": {
                       "value": 10.0
                   }
               },
               "t-shirt-percentage": {
                   "value": 16.666666666666664
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "total_sales": {
                   "value": 375.0
               },
               "t-shirts": {
                   "doc_count": 1,
                   "sales": {
                       "value": 175.0
                   }
               },
               "t-shirt-percentage": {
                   "value": 46.666666666666664
               }
            }
         ]
      }
   }
}
```

# Bucket count K-S test correlation aggregation

> WARNING: This functionality is experimental and may be changed or removed completely in a future release. Elastic will take a best effort approach to fix any issues, but experimental features are not subject to the support SLA of official GA features.
>
> 此功能是实验性的，可能会在未来版本中完全更改或删除。Elastic 将尽最大努力解决任何问题，但实验性功能不受官方 GA 功能的支持 SLA 的约束。

A sibling pipeline aggregation which executes a two sample Kolmogorov–Smirnov test (referred to as a "K-S test" from now on) against a provided distribution, and the distribution implied by the documents counts in the configured sibling aggregation. Specifically, for some metric, assuming that the percentile intervals of the metric are known beforehand or have been computed by an aggregation, then one would use range aggregation for the sibling to compute the p-value of the distribution difference between the metric and the restriction of that metric to a subset of the documents. A natural use case is if the sibling aggregation range aggregation nested in a terms aggregation, in which case one compares the overall distribution of metric to its restriction to each term.

一个兄弟管道聚合，它针对提供的分布执行两个样本 Kolmogorov–Smirnov 测试（从现在开始称为“KS 测试”），并且文档隐含的分布在配置的兄弟聚合中计数。具体来说，对于某些度量，假设度量的百分位区间是事先已知的或已通过聚合计算，则可以使用兄弟的范围聚合来计算度量与限制之间分布差异的 p 值将该度量转换为文档的子集。一个自然的用例是兄弟聚合范围聚合嵌套在术语聚合中，在这种情况下，将度量的整体分布与其对每个术语的限制进行比较。

###  Parameters

**`buckets_path`**

(Required, string) Path to the buckets that contain one set of values to correlate. Must be a `_count` path For syntax, see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax).

（必需，字符串）包含一组要关联的值的存储桶的路径。必须是`_count`路径 有关语法，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)。

**`alternative`**

(Optional, list) A list of string values indicating which K-S test alternative to calculate. The valid values are: "greater", "less", "two_sided". This parameter is key for determining the K-S statistic used when calculating the K-S test. Default value is all possible alternative hypotheses.

（可选，列表）字符串值列表，指示要计算的 KS 测试替代项。有效值为：“greater”、“less”、“two_side”。此参数是确定计算 KS 检验时使用的 KS 统计量的关键。默认值是所有可能的替代假设。

**`fractions`**

(Optional, list) A list of doubles indicating the distribution of the samples with which to compare to the `buckets_path` results. In typical usage this is the overall proportion of documents in each bucket, which is compared with the actual document proportions in each bucket from the sibling aggregation counts. The default is to assume that overall documents are uniformly distributed on these buckets, which they would be if one used equal percentiles of a metric to define the bucket end points.

（可选，列表）表示要与`buckets_path`结果进行比较的样本分布的双精度列表 。在典型用法中，这是每个桶中文档的总体比例，将其与来自同级聚合计数的每个桶中的实际文档比例进行比较。默认情况是假设整个文档均匀分布在这些存储桶上，如果使用度量的相等百分位数来定义存储桶端点，则会出现这种情况。

**`sampling_method`**

(Optional, string) Indicates the sampling methodology when calculating the K-S test. Note, this is sampling of the returned values. This determines the cumulative distribution function (CDF) points used comparing the two samples. Default is `upper_tail`, which emphasizes the upper end of the CDF points. Valid options are: `upper_tail`, `uniform`, and `lower_tail`.

（可选，字符串）表示计算 KS 检验时的抽样方法。请注意，这是对返回值的采样。这决定了用于比较两个样本的累积分布函数 (CDF) 点。默认为`upper_tail`，它强调 CDF 点的上端。有效的选项为：`upper_tail`，`uniform`，和`lower_tail`。

### Syntax

A `bucket_count_ks_test` aggregation looks like this in isolation:

单独的`bucket_count_ks_test`聚合看起来像这样：

```console
{
  "bucket_count_ks_test": {
    "buckets_path": "range_values>_count", (1)
    "alternative": ["less", "greater", "two_sided"], (2)
    "sampling_method": "upper_tail" (3)
  }
}
```

1.  The buckets containing the values to test against.

    包含要测试的值的存储桶。

2.  The alternatives to calculate.

   要计算的替代方案。

3.  The sampling method for the K-S statistic.

   KS 统计量的抽样方法。

###  Example

The following snippet runs the `bucket_count_ks_test` on the individual terms in the field `version` against a uniform distribution. The uniform distribution reflects the `latency` percentile buckets. Not shown is the pre-calculation of the `latency` indicator values, which was done utilizing the [percentiles](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html) aggregation.

以下代码段针对均匀分布`bucket_count_ks_test`在该字段`version`中的各个术语上运行。均匀分布反映了`latency`百分位桶。未显示`latency`指标值的预计算，这是利用[百分位数](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html)聚合完成的 。

This example is only using the deciles of `latency`.

此示例仅使用 的十分位数`latency`。

```console
POST correlate_latency/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "buckets": {
      "terms": { (1)
        "field": "version",
        "size": 2
      },
      "aggs": {
        "latency_ranges": {
          "range": { (2)
            "field": "latency",
            "ranges": [
              { "to": 0 },
              { "from": 0, "to": 105 },
              { "from": 105, "to": 225 },
              { "from": 225, "to": 445 },
              { "from": 445, "to": 665 },
              { "from": 665, "to": 885 },
              { "from": 885, "to": 1115 },
              { "from": 1115, "to": 1335 },
              { "from": 1335, "to": 1555 },
              { "from": 1555, "to": 1775 },
              { "from": 1775 }
            ]
          }
        },
        "ks_test": { (3)
          "bucket_count_ks_test": {
            "buckets_path": "latency_ranges>_count",
            "alternative": ["less", "greater", "two_sided"]
          }
        }
      }
    }
  }
}
```

1.  The term buckets containing a range aggregation and the bucket correlation aggregation. Both are utilized to calculate the correlation of the term values with the latency.

   术语桶包含范围聚合和桶相关聚合。两者都用于计算术语值与延迟的相关性。

2.  The range aggregation on the latency field. The ranges were created referencing the percentiles of the latency field.

    延迟字段上的范围聚合。范围是参考延迟字段的百分位数创建的。

3. The bucket count K-S test aggregation that tests if the bucket counts comes from the same distribution as `fractions`; where `fractions` is a uniform distribution.

    桶计数 KS 测试聚合，用于测试桶计数是否来自与 相同的分布`fractions`；其中`fractions`是均匀分布。

And the following may be the response:

```console-result
{
  "aggregations" : {
    "buckets" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "1.0",
          "doc_count" : 100,
          "latency_ranges" : {
            "buckets" : [
              {
                "key" : "*-0.0",
                "to" : 0.0,
                "doc_count" : 0
              },
              {
                "key" : "0.0-105.0",
                "from" : 0.0,
                "to" : 105.0,
                "doc_count" : 1
              },
              {
                "key" : "105.0-225.0",
                "from" : 105.0,
                "to" : 225.0,
                "doc_count" : 9
              },
              {
                "key" : "225.0-445.0",
                "from" : 225.0,
                "to" : 445.0,
                "doc_count" : 0
              },
              {
                "key" : "445.0-665.0",
                "from" : 445.0,
                "to" : 665.0,
                "doc_count" : 0
              },
              {
                "key" : "665.0-885.0",
                "from" : 665.0,
                "to" : 885.0,
                "doc_count" : 0
              },
              {
                "key" : "885.0-1115.0",
                "from" : 885.0,
                "to" : 1115.0,
                "doc_count" : 10
              },
              {
                "key" : "1115.0-1335.0",
                "from" : 1115.0,
                "to" : 1335.0,
                "doc_count" : 20
              },
              {
                "key" : "1335.0-1555.0",
                "from" : 1335.0,
                "to" : 1555.0,
                "doc_count" : 20
              },
              {
                "key" : "1555.0-1775.0",
                "from" : 1555.0,
                "to" : 1775.0,
                "doc_count" : 20
              },
              {
                "key" : "1775.0-*",
                "from" : 1775.0,
                "doc_count" : 20
              }
            ]
          },
          "ks_test" : {
            "less" : 2.248673241788478E-4,
            "greater" : 1.0,
            "two_sided" : 5.791639181800257E-4
          }
        },
        {
          "key" : "2.0",
          "doc_count" : 100,
          "latency_ranges" : {
            "buckets" : [
              {
                "key" : "*-0.0",
                "to" : 0.0,
                "doc_count" : 0
              },
              {
                "key" : "0.0-105.0",
                "from" : 0.0,
                "to" : 105.0,
                "doc_count" : 19
              },
              {
                "key" : "105.0-225.0",
                "from" : 105.0,
                "to" : 225.0,
                "doc_count" : 11
              },
              {
                "key" : "225.0-445.0",
                "from" : 225.0,
                "to" : 445.0,
                "doc_count" : 20
              },
              {
                "key" : "445.0-665.0",
                "from" : 445.0,
                "to" : 665.0,
                "doc_count" : 20
              },
              {
                "key" : "665.0-885.0",
                "from" : 665.0,
                "to" : 885.0,
                "doc_count" : 20
              },
              {
                "key" : "885.0-1115.0",
                "from" : 885.0,
                "to" : 1115.0,
                "doc_count" : 10
              },
              {
                "key" : "1115.0-1335.0",
                "from" : 1115.0,
                "to" : 1335.0,
                "doc_count" : 0
              },
              {
                "key" : "1335.0-1555.0",
                "from" : 1335.0,
                "to" : 1555.0,
                "doc_count" : 0
              },
              {
                "key" : "1555.0-1775.0",
                "from" : 1555.0,
                "to" : 1775.0,
                "doc_count" : 0
              },
              {
                "key" : "1775.0-*",
                "from" : 1775.0,
                "doc_count" : 0
              }
            ]
          },
          "ks_test" : {
            "less" : 0.9642895789647244,
            "greater" : 4.58718174664754E-9,
            "two_sided" : 5.916656831139733E-9
          }
        }
      ]
    }
  }
}
```

#  Bucket correlation aggregation

> WARNING: This functionality is experimental and may be changed or removed completely in a future release. Elastic will take a best effort approach to fix any issues, but experimental features are not subject to the support SLA of official GA features.
>
> 此功能是实验性的，可能会在未来版本中完全更改或删除。Elastic 将尽最大努力解决任何问题，但实验性功能不受官方 GA 功能的支持 SLA 的约束。

A sibling pipeline aggregation which executes a correlation function on the configured sibling multi-bucket aggregation.

同级管道聚合，它在配置的同级多桶聚合上执行相关函数。

###  Parameters

**`buckets_path`**

(Required, string) Path to the buckets that contain one set of values to correlate. For syntax, see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax).

（必需，字符串）包含一组要关联的值的存储桶的路径。有关语法，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)。

**`function`**

(Required, object) The correlation function to execute.

（必需，对象）要执行的关联函数。

​	**Properties of function**

​	**`count_correlation`**

​	(Required*, object) The configuration to calculate a count correlation. This function is designed for determining the correlation of a term value and a given metric. Consequently, it needs to meet the following requirements.

（必需*，对象）用于计算计数相关性的配置。此函数旨在确定术语值与给定度量的相关性。因此，它需要满足以下要求。

- The `buckets_path` must point to a `_count` metric.

  在`buckets_path`必须指向一个`_count`指标。

- The total count of all the `bucket_path` count values must be less than or equal to `indicator.doc_count`.

  所有`bucket_path`计数值的总计数必须小于或等于`indicator.doc_count`。

- When utilizing this function, an initial calculation to gather the required `indicator` values is required.

  使用此功能时，需要进行初始计算以收集所需`indicator`值。

​		**Properties of count_correlation**

​		**`indicator`**

​		(Required, object) The indicator with which to correlate the configured `bucket_path` values.

​		（必需，对象）与配置`bucket_path`值相关联的指标。

​	**`expectations`**

​	(Required, array) An array of numbers with which to correlate the configured `bucket_path` values. The length of this value must always equal the number of buckets returned by the `bucket_path`.

​	（必需，数组）与配置`bucket_path`值相关联的数字数组。此值的长度必须始终等于 返回的桶数`bucket_path`。

​	**`fractions`**

​	(Optional, array) An array of fractions to use when averaging and calculating variance. This should be used if the pre-calculated data and the `buckets_path` have known gaps. The length of `fractions`, if provided, must equal `expectations`.

​	（可选，数组）在平均和计算方差时使用的分数数组。如果预先计算的数据和`buckets_path`已知的差距，则应使用此方法 。的长度（`fractions`如果提供）必须等于`expectations`。

​	**`doc_count`**

​	(Required, integer) The total number of documents that initially created the `expectations`. It’s required to be greater than or equal to the sum of all values in the `buckets_path` as this is the originating superset of data to which the term values are correlated.

​	（必需，整数）最初创建`expectations`. 它必须大于或等于 中所有值的总和，`buckets_path`因为这是与术语值相关的原始数据超集。

###  Syntax

A `bucket_correlation` aggregation looks like this in isolation:

单独的`bucket_correlation`聚合看起来像这样：

```js
{
  "bucket_correlation": {
    "buckets_path": "range_values>_count", (1)
    "function": {
      "count_correlation": { (2)
        "expectations": [...],
        "doc_count": 10000
      }
    }
  }
}
```

1. The buckets containing the values to correlate against.

   包含要关联的值的桶。

2. The correlation function definition.

    相关函数定义。

###  Example

The following snippet correlates the individual terms in the field `version` with the `latency` metric. Not shown is the pre-calculation of the `latency` indicator values, which was done utilizing the [percentiles](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html) aggregation.

以下代码段将字段中的各个术语`version`与`latency`指标相关联。未显示`latency`指标值的预计算，这是利用[百分位数](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html)聚合完成的 。

This example is only using the 10s percentiles.

此示例仅使用 10s 百分位数。

```console
POST correlate_latency/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "buckets": {
      "terms": { (1)
        "field": "version",
        "size": 2
      },
      "aggs": {
        "latency_ranges": {
          "range": { (2)
            "field": "latency",
            "ranges": [
              { "to": 0.0 },
              { "from": 0, "to": 105 },
              { "from": 105, "to": 225 },
              { "from": 225, "to": 445 },
              { "from": 445, "to": 665 },
              { "from": 665, "to": 885 },
              { "from": 885, "to": 1115 },
              { "from": 1115, "to": 1335 },
              { "from": 1335, "to": 1555 },
              { "from": 1555, "to": 1775 },
              { "from": 1775 }
            ]
          }
        },
        "bucket_correlation": { (3)
          "bucket_correlation": {
            "buckets_path": "latency_ranges>_count",
            "function": {
              "count_correlation": {
                "indicator": {
                   "expectations": [0, 52.5, 165, 335, 555, 775, 1000, 1225, 1445, 1665, 1775],
                   "doc_count": 200
                }
              }
            }
          }
        }
      }
    }
  }
}
```

1. The term buckets containing a range aggregation and the bucket correlation aggregation. Both are utilized to calculate the correlation of the term values with the latency.

   术语桶包含范围聚合和桶相关聚合。两者都用于计算术语值与延迟的相关性。

2. The range aggregation on the latency field. The ranges were created referencing the percentiles of the latency field.

   延迟字段上的范围聚合。范围是参考延迟字段的百分位数创建的。

3. The bucket correlation aggregation that calculates the correlation of the number of term values within each range and the previously calculated indicator values.

   桶相关性聚合，用于计算每个范围内的术语值数量与之前计算的指标值的相关性。

And the following may be the response:

```console-result
{
  "aggregations" : {
    "buckets" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "1.0",
          "doc_count" : 100,
          "latency_ranges" : {
            "buckets" : [
              {
                "key" : "*-0.0",
                "to" : 0.0,
                "doc_count" : 0
              },
              {
                "key" : "0.0-105.0",
                "from" : 0.0,
                "to" : 105.0,
                "doc_count" : 1
              },
              {
                "key" : "105.0-225.0",
                "from" : 105.0,
                "to" : 225.0,
                "doc_count" : 9
              },
              {
                "key" : "225.0-445.0",
                "from" : 225.0,
                "to" : 445.0,
                "doc_count" : 0
              },
              {
                "key" : "445.0-665.0",
                "from" : 445.0,
                "to" : 665.0,
                "doc_count" : 0
              },
              {
                "key" : "665.0-885.0",
                "from" : 665.0,
                "to" : 885.0,
                "doc_count" : 0
              },
              {
                "key" : "885.0-1115.0",
                "from" : 885.0,
                "to" : 1115.0,
                "doc_count" : 10
              },
              {
                "key" : "1115.0-1335.0",
                "from" : 1115.0,
                "to" : 1335.0,
                "doc_count" : 20
              },
              {
                "key" : "1335.0-1555.0",
                "from" : 1335.0,
                "to" : 1555.0,
                "doc_count" : 20
              },
              {
                "key" : "1555.0-1775.0",
                "from" : 1555.0,
                "to" : 1775.0,
                "doc_count" : 20
              },
              {
                "key" : "1775.0-*",
                "from" : 1775.0,
                "doc_count" : 20
              }
            ]
          },
          "bucket_correlation" : {
            "value" : 0.8402398981360937
          }
        },
        {
          "key" : "2.0",
          "doc_count" : 100,
          "latency_ranges" : {
            "buckets" : [
              {
                "key" : "*-0.0",
                "to" : 0.0,
                "doc_count" : 0
              },
              {
                "key" : "0.0-105.0",
                "from" : 0.0,
                "to" : 105.0,
                "doc_count" : 19
              },
              {
                "key" : "105.0-225.0",
                "from" : 105.0,
                "to" : 225.0,
                "doc_count" : 11
              },
              {
                "key" : "225.0-445.0",
                "from" : 225.0,
                "to" : 445.0,
                "doc_count" : 20
              },
              {
                "key" : "445.0-665.0",
                "from" : 445.0,
                "to" : 665.0,
                "doc_count" : 20
              },
              {
                "key" : "665.0-885.0",
                "from" : 665.0,
                "to" : 885.0,
                "doc_count" : 20
              },
              {
                "key" : "885.0-1115.0",
                "from" : 885.0,
                "to" : 1115.0,
                "doc_count" : 10
              },
              {
                "key" : "1115.0-1335.0",
                "from" : 1115.0,
                "to" : 1335.0,
                "doc_count" : 0
              },
              {
                "key" : "1335.0-1555.0",
                "from" : 1335.0,
                "to" : 1555.0,
                "doc_count" : 0
              },
              {
                "key" : "1555.0-1775.0",
                "from" : 1555.0,
                "to" : 1775.0,
                "doc_count" : 0
              },
              {
                "key" : "1775.0-*",
                "from" : 1775.0,
                "doc_count" : 0
              }
            ]
          },
          "bucket_correlation" : {
            "value" : -0.5759855613334943
          }
        }
      ]
    }
  }
}
```

#  Bucket selector aggregation

A parent pipeline aggregation which executes a script which determines whether the current bucket will be retained in the parent multi-bucket aggregation. The specified metric must be numeric and the script must return a boolean value. If the script language is `expression` then a numeric return value is permitted. In this case 0.0 will be evaluated as `false` and all other values will evaluate to true.

执行脚本的父管道聚合，该脚本确定当前存储桶是否将保留在父多存储桶聚合中。指定的指标必须是数字，并且脚本必须返回一个布尔值。如果脚本语言是`expression`数字返回值是允许的。在这种情况下，0.0 将被评估为`false` ，所有其他值将评估为真。

> NOTE: The bucket_selector aggregation, like all pipeline aggregations, executes after all other sibling aggregations. This means that using the bucket_selector aggregation to filter the returned buckets in the response does not save on execution time running the aggregations.
>
> 与所有管道聚合一样，bucket_selector 聚合在所有其他同级聚合之后执行。这意味着使用 bucket_selector 聚合来过滤响应中返回的存储桶不会节省运行聚合的执行时间。

###  Syntax

A `bucket_selector` aggregation looks like this in isolation:

单独的`bucket_selector`聚合看起来像这样：

```console
{
  "bucket_selector": {
    "buckets_path": {
      "my_var1": "the_sum",                     (1)
      "my_var2": "the_value_count"
    },
    "script": "params.my_var1 > params.my_var2"
  }
}
```

1.  Here, `my_var1` is the name of the variable for this buckets path to use in the script, `the_sum` is the path to the metrics to use for that variable.

   此处，`my_var1`是要在脚本中使用的此存储桶路径的变量名称，`the_sum`是要用于该变量的指标的路径。

**Table 50. `bucket_selector` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `script`       | The script to run for this aggregation. The script can be inline, file or indexed. (see [Scripting](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html) for more details)<br />为此聚合运行的脚本。脚本可以是内联的、文件的或索引的。（ 有关更多详细信息，请参阅[脚本](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html)） | Required |               |
| `buckets_path` | A map of script variables and their associated path to the buckets we wish to use for the variable (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />脚本变量的映射及其到我们希望用于变量的存储桶的关联路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |

The following snippet only retains buckets where the total sales for the month is more than 200:

以下代码段仅保留当月总销售额超过 200 的存储桶：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_bucket_filter": {
          "bucket_selector": {
            "buckets_path": {
              "totalSales": "total_sales"
            },
            "script": "params.totalSales > 200"
          }
        }
      }
    }
  }
}
```

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "total_sales": {
                   "value": 550.0
               }
            },(1)
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "total_sales": {
                   "value": 375.0
               },
            }
         ]
      }
   }
}
```

1. Bucket for `2015/02/01 00:00:00` has been removed as its total sales was less than 200

   Bucket for`2015/02/01 00:00:00`已被移除，因为其总销售额低于 200

#  Bucket sort aggregation

A parent pipeline aggregation which sorts the buckets of its parent multi-bucket aggregation. Zero or more sort fields may be specified together with the corresponding sort order. Each bucket may be sorted based on its `_key`, `_count` or its sub-aggregations. In addition, parameters `from` and `size` may be set in order to truncate the result buckets.

父管道聚合对其父多桶聚合的桶进行排序。零个或多个排序字段可以与相应的排序顺序一起指定。每个桶可根据其进行排序`_key`，`_count`或者它的子聚集。此外，可以设置参数`from`和`size`以截断结果桶。

> NOTE: The `bucket_sort` aggregation, like all pipeline aggregations, is executed after all other non-pipeline aggregations. This means the sorting only applies to whatever buckets are already returned from the parent aggregation. For example, if the parent aggregation is `terms` and its `size` is set to `10`, the `bucket_sort` will only sort over those 10 returned term buckets.
>
> 在`bucket_sort`聚集，像所有管线的聚合，是其他所有非管道聚合后执行。这意味着排序仅适用于已从父聚合返回的任何桶。例如，如果父聚合是`terms`并且其`size`设置为`10`，`bucket_sort`则只会对这 10 个返回的术语桶进行排序。

###  Syntax

A `bucket_sort` aggregation looks like this in isolation:

单独的`bucket_sort`聚合看起来像这样：

```console
{
  "bucket_sort": {
    "sort": [
      { "sort_field_1": { "order": "asc" } },   (1)
      { "sort_field_2": { "order": "desc" } },
      "sort_field_3"
    ],
    "from": 1,
    "size": 3
  }
}
```

1. Here, `sort_field_1` is the bucket path to the variable to be used as the primary sort and its order is ascending.

   这里，`sort_field_1`是要用作主要排序的变量的桶路径，其顺序是升序。

**Table 51. `bucket_sort` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `sort`         | The list of fields to sort on. See [`sort`](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html) for more details.<br />要排序的字段列表。有关[`sort`](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html)更多详细信息，请参阅。 | Optional |               |
| `from`         | Buckets in positions prior to the set value will be truncated.<br />位于设置值之前位置的存储桶将被截断。 | Optional | `0`           |
| `size`         | The number of buckets to return. Defaults to all buckets of the parent aggregation.<br />要返回的桶数。默认为父聚合的所有桶。 | Optional |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |

The following snippet returns the buckets corresponding to the 3 months with the highest total sales in descending order:
以下代码段按降序返回与总销售额最高的 3 个月对应的桶：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "total_sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_bucket_sort": {
          "bucket_sort": {
            "sort": [
              { "total_sales": { "order": "desc" } } (1)
            ],
            "size": 3                                (2)
          }
        }
      }
    }
  }
}
```

1. `sort` is set to use the values of `total_sales` in descending order

   `sort`设置为`total_sales`按降序使用的值

2. `size` is set to `3` meaning only the top 3 months in `total_sales` will be returned

   `size`设置为`3`意味着仅`total_sales`返回前 3 个月

And the following may be the response:

```console-result
{
   "took": 82,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "total_sales": {
                   "value": 550.0
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "total_sales": {
                   "value": 375.0
               },
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "total_sales": {
                   "value": 60.0
               },
            }
         ]
      }
   }
}
```

###  Truncating without sorting

It is also possible to use this aggregation in order to truncate the result buckets without doing any sorting. To do so, just use the `from` and/or `size` parameters without specifying `sort`.

也可以使用此聚合来截断结果桶而不进行任何排序。为此，只需使用`from`和/或`size`参数而不指定`sort`.

The following example simply truncates the result so that only the second bucket is returned:

以下示例只是截断了结果，以便仅返回第二个存储桶：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "bucket_truncate": {
          "bucket_sort": {
            "from": 1,
            "size": 1
          }
        }
      }
    }
  }
}
```

Response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2
            }
         ]
      }
   }
}
```

#  Cumulative cardinality aggregation

A parent pipeline aggregation which calculates the Cumulative Cardinality in a parent histogram (or date_histogram) aggregation. The specified metric must be a cardinality aggregation and the enclosing histogram must have `min_doc_count` set to `0` (default for `histogram` aggregations).

父管道聚合，用于计算父直方图（或 date_histogram）聚合中的累积基数。指定的指标必须是基数聚合，并且封闭的直方图必须`min_doc_count`设置为`0`（`histogram`聚合的默认值）。

The `cumulative_cardinality` agg is useful for finding "total new items", like the number of new visitors to your website each day. A regular cardinality aggregation will tell you how many unique visitors came each day, but doesn’t differentiate between "new" or "repeat" visitors. The Cumulative Cardinality aggregation can be used to determine how many of each day’s unique visitors are "new".

该`cumulative_cardinality`AGG是每天寻找“总的新项目”，像新访问者数量到您的网站是有用的。常规基数聚合会告诉您每天有多少独立访问者，但不区分“新”访问者或“重复”访问者。累积基数聚合可用于确定每天的唯一访问者中有多少是“新的”。

###  Syntax

A `cumulative_cardinality` aggregation looks like this in isolation:

单独的`cumulative_cardinality`聚合看起来像这样：

```console
{
  "cumulative_cardinality": {
    "buckets_path": "my_cardinality_agg"
  }
}
```

**Table 52. `cumulative_cardinality` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the cardinality aggregation we wish to find the cumulative cardinality for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望找到累积基数的基数聚合的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

The following snippet calculates the cumulative cardinality of the total daily `users`:

以下代码段计算总 daily 的累积基数`users`：

```console
GET /user_hits/_search
{
  "size": 0,
  "aggs": {
    "users_per_day": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      },
      "aggs": {
        "distinct_users": {
          "cardinality": {
            "field": "user_id"
          }
        },
        "total_new_users": {
          "cumulative_cardinality": {
            "buckets_path": "distinct_users" (1)
          }
        }
      }
    }
  }
}
```

1. `buckets_path` instructs this aggregation to use the output of the `distinct_users` aggregation for the cumulative cardinality

   `buckets_path`指示此聚合将聚合的输出`distinct_users`用于累积基数

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "users_per_day": {
         "buckets": [
            {
               "key_as_string": "2019-01-01T00:00:00.000Z",
               "key": 1546300800000,
               "doc_count": 2,
               "distinct_users": {
                  "value": 2
               },
               "total_new_users": {
                  "value": 2
               }
            },
            {
               "key_as_string": "2019-01-02T00:00:00.000Z",
               "key": 1546387200000,
               "doc_count": 2,
               "distinct_users": {
                  "value": 2
               },
               "total_new_users": {
                  "value": 3
               }
            },
            {
               "key_as_string": "2019-01-03T00:00:00.000Z",
               "key": 1546473600000,
               "doc_count": 3,
               "distinct_users": {
                  "value": 3
               },
               "total_new_users": {
                  "value": 4
               }
            }
         ]
      }
   }
}
```

Note how the second day, `2019-01-02`, has two distinct users but the `total_new_users` metric generated by the cumulative pipeline agg only increments to three. This means that only one of the two users that day were new, the other had already been seen in the previous day. This happens again on the third day, where only one of three users is completely new.

请注意，第二天`2019-01-02`有两个不同的用户，但`total_new_users`累积管道 agg 生成的指标仅增加到三个。这意味着当天的两个用户中只有一个是新用户，另一个在前一天已经见过。这在第三天再次发生，三个用户中只有一个是全新的。

###  Incremental cumulative cardinality

The `cumulative_cardinality` agg will show you the total, distinct count since the beginning of the time period being queried. Sometimes, however, it is useful to see the "incremental" count. Meaning, how many new users are added each day, rather than the total cumulative count.

该`cumulative_cardinality`AGG会告诉你的总，重复计数，因为时间周期的开始被质疑。但是，有时查看“增量”计数很有用。意思是，每天添加多少新用户，而不是累计总数。

This can be accomplished by adding a `derivative` aggregation to our query:

这可以通过向`derivative`我们的查询添加聚合来实现：

```console
GET /user_hits/_search
{
  "size": 0,
  "aggs": {
    "users_per_day": {
      "date_histogram": {
        "field": "timestamp",
        "calendar_interval": "day"
      },
      "aggs": {
        "distinct_users": {
          "cardinality": {
            "field": "user_id"
          }
        },
        "total_new_users": {
          "cumulative_cardinality": {
            "buckets_path": "distinct_users"
          }
        },
        "incremental_new_users": {
          "derivative": {
            "buckets_path": "total_new_users"
          }
        }
      }
    }
  }
}
```

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "users_per_day": {
         "buckets": [
            {
               "key_as_string": "2019-01-01T00:00:00.000Z",
               "key": 1546300800000,
               "doc_count": 2,
               "distinct_users": {
                  "value": 2
               },
               "total_new_users": {
                  "value": 2
               }
            },
            {
               "key_as_string": "2019-01-02T00:00:00.000Z",
               "key": 1546387200000,
               "doc_count": 2,
               "distinct_users": {
                  "value": 2
               },
               "total_new_users": {
                  "value": 3
               },
               "incremental_new_users": {
                  "value": 1.0
               }
            },
            {
               "key_as_string": "2019-01-03T00:00:00.000Z",
               "key": 1546473600000,
               "doc_count": 3,
               "distinct_users": {
                  "value": 3
               },
               "total_new_users": {
                  "value": 4
               },
               "incremental_new_users": {
                  "value": 1.0
               }
            }
         ]
      }
   }
}
```

#  Cumulative sum aggregation

A parent pipeline aggregation which calculates the cumulative sum of a specified metric in a parent histogram (or date_histogram) aggregation. The specified metric must be numeric and the enclosing histogram must have `min_doc_count` set to `0` (default for `histogram` aggregations).

父管道聚合，用于计算父直方图（或 date_histogram）聚合中指定指标的累积总和。指定的指标必须是数字，并且封闭的直方图必须`min_doc_count`设置为`0`（`histogram`聚合的默认值）。

###  Syntax

A `cumulative_sum` aggregation looks like this in isolation:

单独的`cumulative_sum`聚合看起来像这样：

```console
{
  "cumulative_sum": {
    "buckets_path": "the_sum"
  }
}
```

**Table 53. `cumulative_sum` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the buckets we wish to find the cumulative sum for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望找到累积总和的桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

The following snippet calculates the cumulative sum of the total monthly `sales`:

以下代码段计算每月总金额的累计总和`sales`：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "cumulative_sales": {
          "cumulative_sum": {
            "buckets_path": "sales" (1)
          }
        }
      }
    }
  }
}
```

1. `buckets_path` instructs this cumulative sum aggregation to use the output of the `sales` aggregation for the cumulative sum

   `buckets_path`这个指示累积和聚集，使用的输出`sales`聚集的累计总和

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               },
               "cumulative_sales": {
                  "value": 550.0
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               },
               "cumulative_sales": {
                  "value": 610.0
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               },
               "cumulative_sales": {
                  "value": 985.0
               }
            }
         ]
      }
   }
}
```

# Derivative aggregation

A parent pipeline aggregation which calculates the derivative of a specified metric in a parent histogram (or date_histogram) aggregation. The specified metric must be numeric and the enclosing histogram must have `min_doc_count` set to `0` (default for `histogram` aggregations).

父管道聚合，用于计算父直方图（或 date_histogram）聚合中指定指标的导数。指定的指标必须是数字，并且封闭的直方图必须`min_doc_count`设置为`0`（`histogram`聚合的默认值）。

###  Syntax

A `derivative` aggregation looks like this in isolation:

单独的`derivative`聚合看起来像这样：

```console
"derivative": {
  "buckets_path": "the_sum"
}
```

**Table 54. `derivative` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the buckets we wish to find the derivative for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望为其找到导数的桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

###  First Order Derivative

The following snippet calculates the derivative of the total monthly `sales`:

以下代码段计算总月度的导数`sales`：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_deriv": {
          "derivative": {
            "buckets_path": "sales" (1)
          }
        }
      }
    }
  }
}
```

1. `buckets_path` instructs this derivative aggregation to use the output of the `sales` aggregation for the derivative

   `buckets_path`指示该衍生物聚合使用的输出`sales`聚集的衍生物

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               } (1)
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               },
               "sales_deriv": {
                  "value": -490.0 (2)
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2, (3)
               "sales": {
                  "value": 375.0
               },
               "sales_deriv": {
                  "value": 315.0
               }
            }
         ]
      }
   }
}
```

1. No derivative for the first bucket since we need at least 2 data points to calculate the derivative

    第一个桶没有导数，因为我们需要至少 2 个数据点来计算导数

2.  Derivative value units are implicitly defined by the `sales` aggregation and the parent histogram so in this case the units would be $/month assuming the `price` field has units of $.

   衍生值单位由`sales`聚合和父直方图隐式定义，因此在这种情况下，假设`price`字段单位为 $ ，单位将为 $/月。

3.  The number of documents in the bucket are represented by the `doc_count`

    桶中的文档数由 `doc_count`

###  Second Order Derivative

A second order derivative can be calculated by chaining the derivative pipeline aggregation onto the result of another derivative pipeline aggregation as in the following example which will calculate both the first and the second order derivative of the total monthly sales:

二阶导数可以通过将导数管道聚合链接到另一个导数管道聚合的结果上来计算，如下例将计算每月总销售额的一阶和二阶导数：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_deriv": {
          "derivative": {
            "buckets_path": "sales"
          }
        },
        "sales_2nd_deriv": {
          "derivative": {
            "buckets_path": "sales_deriv" (1)
          }
        }
      }
    }
  }
}
```

1. `buckets_path` for the second derivative points to the name of the first derivative

   `buckets_path` 因为二阶导数指向一阶导数的名称

And the following may be the response:

以下可能是响应：

```console-result
{
   "took": 50,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               } 
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               },
               "sales_deriv": {
                  "value": -490.0
               } (1)
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               },
               "sales_deriv": {
                  "value": 315.0
               },
               "sales_2nd_deriv": {
                  "value": 805.0
               }
            }
         ]
      }
   }
}
```

1. No second derivative for the first two buckets since we need at least 2 data points from the first derivative to calculate the second derivative

   前两个桶没有二阶导数，因为我们需要来自一阶导数的至少 2 个数据点来计算二阶导数

###  Units

The derivative aggregation allows the units of the derivative values to be specified. This returns an extra field in the response `normalized_value` which reports the derivative value in the desired x-axis units. In the below example we calculate the derivative of the total sales per month but ask for the derivative of the sales as in the units of sales per day:

导数聚合允许指定导数值的单位。这会在响应中返回一个额外的字段， `normalized_value`以所需的 x 轴单位报告导数值。在下面的示例中，我们计算每月总销售额的导数，但要求以每天的销售额为单位的销售额的导数：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "sales_deriv": {
          "derivative": {
            "buckets_path": "sales",
            "unit": "day" (1)
          }
        }
      }
    }
  }
}
```

1. `unit` specifies what unit to use for the x-axis of the derivative calculation

   `unit` 指定用于导数计算的 x 轴的单位

And the following may be the response:

```console-result
{
   "took": 50,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               } (1)
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               },
               "sales_deriv": {
                  "value": -490.0, (1)
                  "normalized_value": -15.806451612903226 (2)
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               },
               "sales_deriv": {
                  "value": 315.0,
                  "normalized_value": 11.25
               }
            }
         ]
      }
   }
}
```

1. `value` is reported in the original units of *per month*

   `value`*以每月*的原始单位报告

2. `normalized_value` is reported in the desired units of *per day*

   `normalized_value`*以每天*所需的单位报告

#  Extended stats bucket aggregation

A sibling pipeline aggregation which calculates a variety of stats across all bucket of a specified metric in a sibling aggregation. The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.

同级管道聚合，它计算同级聚合中指定指标的所有存储桶的各种统计信息。指定的指标必须是数字，同级聚合必须是多桶聚合。

This aggregation provides a few more statistics (sum of squares, standard deviation, etc) compared to the `stats_bucket` aggregation.

与`stats_bucket`聚合相比，此聚合提供了更多的统计信息（平方和、标准偏差等）。

###  Syntax

A `extended_stats_bucket` aggregation looks like this in isolation:

单独的`extended_stats_bucket`聚合看起来像这样：

```console
{
  "extended_stats_bucket": {
    "buckets_path": "the_sum"
  }
}
```

**Table 55. `extended_stats_bucket` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the buckets we wish to calculate stats for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望为其计算统计数据的存储桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |
| `sigma`        | The number of standard deviations above/below the mean to display<br />要显示的高于/低于均值的标准差数 | Optional | 2             |

The following snippet calculates the extended stats for monthly `sales` bucket:

以下代码段计算每月`sales`存储桶的扩展统计信息：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    },
    "stats_monthly_sales": {
      "extended_stats_bucket": {
        "buckets_path": "sales_per_month>sales" (1)
      }
    }
  }
}
```

1. `bucket_paths` instructs this `extended_stats_bucket` aggregation that we want the calculate stats for the `sales` aggregation in the `sales_per_month` date histogram.

   `bucket_paths`指示此`extended_stats_bucket`聚合我们希望`sales`在`sales_per_month`日期直方图中计算聚合的 统计数据。

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               }
            }
         ]
      },
      "stats_monthly_sales": {
         "count": 3,
         "min": 60.0,
         "max": 550.0,
         "avg": 328.3333333333333,
         "sum": 985.0,
         "sum_of_squares": 446725.0,
         "variance": 41105.55555555556,
         "variance_population": 41105.55555555556,
         "variance_sampling": 61658.33333333334,
         "std_deviation": 202.74505063146563,
         "std_deviation_population": 202.74505063146563,
         "std_deviation_sampling": 248.3109609609156,
         "std_deviation_bounds": {
           "upper": 733.8234345962646,
           "lower": -77.15676792959795,
           "upper_population" : 733.8234345962646,
           "lower_population" : -77.15676792959795,
           "upper_sampling" : 824.9552552551645,
           "lower_sampling" : -168.28858858849787
         }
      }
   }
}
```

# Inference bucket aggregation

A parent pipeline aggregation which loads a pre-trained model and performs inference on the collated result fields from the parent bucket aggregation.

一个父管道聚合，它加载一个预先训练的模型并对来自父存储桶聚合的整理结果字段执行推理。

To use the inference bucket aggregation, you need to have the same security privileges that are required for using the [get trained models API](https://www.elastic.co/guide/en/elasticsearch/reference/master/get-trained-models.html).

要使用推理桶聚合，您需要具有使用[gettrained models API](https://www.elastic.co/guide/en/elasticsearch/reference/master/get-trained-models.html)所需的相同安全权限 。

###  Syntax

A `inference` aggregation looks like this in isolation:

单独的`inference`聚合看起来像这样：

```console
{
  "inference": {
    "model_id": "a_model_for_inference", (1)
    "inference_config": { (2)
      "regression_config": {
        "num_top_feature_importance_values": 2
      }
    },
    "buckets_path": {
      "avg_cost": "avg_agg", (3)
      "max_cost": "max_agg"
    }
  }
}
```

1. The unique identifier or alias for the trained model.

    训练模型的唯一标识符或别名。

2. The optional inference config which overrides the model’s default settings

   覆盖模型默认设置的可选推理配置

3. Map the value of `avg_agg` to the model’s input field `avg_cost`

   将`avg_agg` 的值映射到模型的输入字段`avg_cost`

**Table 56. `inference` Parameters**

| Parameter Name     | Description                                                  | Required | Default Value |
| ------------------ | ------------------------------------------------------------ | -------- | ------------- |
| `model_id`         | The ID or alias for the trained model.<br />训练模型的 ID 或别名。 | Required | -             |
| `inference_config` | Contains the inference type and its options. There are two types: [`regression`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-inference-bucket-aggregation.html#inference-agg-regression-opt) and [`classification`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-inference-bucket-aggregation.html#inference-agg-classification-opt)<br />包含推理类型及其选项。有两种类型：[`regression`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-inference-bucket-aggregation.html#inference-agg-regression-opt)和[`classification`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-inference-bucket-aggregation.html#inference-agg-classification-opt) | Optional | -             |
| `buckets_path`     | Defines the paths to the input aggregations and maps the aggregation names to the field names expected by the model. See [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details<br />定义输入聚合的路径并将聚合名称映射到模型所需的字段名称。有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) | Required | -             |

###  Configuration options for inference models

The `inference_config` setting is optional and usually isn’t required as the pre-trained models come equipped with sensible defaults. In the context of aggregations some options can be overridden for each of the two types of model.

该`inference_config`设置是可选的，通常不需要，因为预先训练的模型配备了合理的默认值。在聚合的上下文中，可以为两种类型的模型中的每一种覆盖一些选项。

#####  Configuration options for regression models

**`num_top_feature_importance_values`**

(Optional, integer) Specifies the maximum number of [feature importance](https://www.elastic.co/guide/en/machine-learning/master/ml-feature-importance.html) values per document. By default, it is zero and no feature importance calculation occurs.

（可选，整数）指定每个文档的最大 [特征重要性](https://www.elastic.co/guide/en/machine-learning/master/ml-feature-importance.html)值数。默认为零，不进行特征重要性计算。

#####  Configuration options for classification models

**`num_top_classes`**

(Optional, integer) Specifies the number of top class predictions to return. Defaults to 0.

（可选，整数）指定要返回的顶级预测的数量。默认为 0。

**`num_top_feature_importance_values`**

(Optional, integer) Specifies the maximum number of [feature importance](https://www.elastic.co/guide/en/machine-learning/master/ml-feature-importance.html) values per document. By default, it is zero and no feature importance calculation occurs.

（可选，整数）指定每个文档的最大 [特征重要性](https://www.elastic.co/guide/en/machine-learning/master/ml-feature-importance.html)值数。默认为零，不进行特征重要性计算。

**`prediction_field_type`**

(Optional, string) Specifies the type of the predicted field to write. Acceptable values are: `string`, `number`, `boolean`. When `boolean` is provided `1.0` is transformed to `true` and `0.0` to `false`.

（可选，字符串）指定要写入的预测字段的类型。可接受的值为：`string`、`number`、`boolean`。当`boolean`提供 `1.0`转化为`true`与`0.0`对`false`。

###  Example

The following snippet aggregates a web log by `client_ip` and extracts a number of features via metric and bucket sub-aggregations as input to the inference aggregation configured with a model trained to identify suspicious client IPs:

以下代码片段聚合了一个网络日志，`client_ip`并通过指标和存储桶子聚合提取了许多特征，作为推理聚合的输入，该聚合配置了一个经过训练以识别可疑客户端 IP 的模型：

```console
GET kibana_sample_data_logs/_search
{
  "size": 0,
  "aggs": {
    "client_ip": { (1)
      "composite": {
        "sources": [
          {
            "client_ip": {
              "terms": {
                "field": "clientip"
              }
            }
          }
        ]
      },
      "aggs": { (2)
        "url_dc": {
          "cardinality": {
            "field": "url.keyword"
          }
        },
        "bytes_sum": {
          "sum": {
            "field": "bytes"
          }
        },
        "geo_src_dc": {
          "cardinality": {
            "field": "geo.src"
          }
        },
        "geo_dest_dc": {
          "cardinality": {
            "field": "geo.dest"
          }
        },
        "responses_total": {
          "value_count": {
            "field": "timestamp"
          }
        },
        "success": {
          "filter": {
            "term": {
              "response": "200"
            }
          }
        },
        "error404": {
          "filter": {
            "term": {
              "response": "404"
            }
          }
        },
        "error503": {
          "filter": {
            "term": {
              "response": "503"
            }
          }
        },
        "malicious_client_ip": { (3)
          "inference": {
            "model_id": "malicious_clients_model",
            "buckets_path": {
              "response_count": "responses_total",
              "url_dc": "url_dc",
              "bytes_sum": "bytes_sum",
              "geo_src_dc": "geo_src_dc",
              "geo_dest_dc": "geo_dest_dc",
              "success": "success._count",
              "error404": "error404._count",
              "error503": "error503._count"
            }
          }
        }
      }
    }
  }
}
```

1.  A composite bucket aggregation that aggregates the data by `client_ip`.

    按 `client_ip` 聚合数据的复合桶聚合。

2.  A series of metrics and bucket sub-aggregations.

    一系列指标和桶子聚合。

3.  Inference bucket aggregation that specifies the trained model and maps the aggregation names to the model’s input fields.

    推理桶聚合，指定训练模型并将聚合名称映射到模型的输入字段。

# Max bucket aggregation

A sibling pipeline aggregation which identifies the bucket(s) with the maximum value of a specified metric in a sibling aggregation and outputs both the value and the key(s) of the bucket(s). The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.

同级管道聚合，它标识具有同级聚合中指定度量的最大值的桶，并输出桶的值和键。指定的指标必须是数字，同级聚合必须是多桶聚合。

###  Syntax

A `max_bucket` aggregation looks like this in isolation:

单独的`max_bucket`聚合看起来像这样：

```js
{
  "max_bucket": {
    "buckets_path": "the_sum"
  }
}
```

**Table 57. `max_bucket` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the buckets we wish to find the maximum for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望找到最大值的桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

The following snippet calculates the maximum of the total monthly `sales`:

以下代码段计算每月总金额的最大值`sales`：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
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
        "buckets_path": "sales_per_month>sales" (1)
      }
    }
  }
}
```

1. `buckets_path` instructs this max_bucket aggregation that we want the maximum value of the `sales` aggregation in the `sales_per_month` date histogram.

   `buckets_path`指示这个 max_bucket 聚合我们想要日期直方图中`sales`聚合 的最大值`sales_per_month`。

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               }
            }
         ]
      },
      "max_monthly_sales": {
          "keys": ["2015/01/01 00:00:00"], (1)
          "value": 550.0
      }
   }
}
```

1. `keys` is an array of strings since the maximum value may be present in multiple buckets

   `keys` 是一个字符串数组，因为最大值可能存在于多个存储桶中

#  Min bucket aggregation

A sibling pipeline aggregation which identifies the bucket(s) with the minimum value of a specified metric in a sibling aggregation and outputs both the value and the key(s) of the bucket(s). The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.

同级管道聚合，它标识具有同级聚合中指定度量的最小值的桶，并输出桶的值和键。指定的指标必须是数字，同级聚合必须是多桶聚合。

###  Syntax

A `min_bucket` aggregation looks like this in isolation:

单独的`min_bucket`聚合看起来像这样：

```js
{
  "min_bucket": {
    "buckets_path": "the_sum"
  }
}
```

**Table 58. `min_bucket` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the buckets we wish to find the minimum for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望找到最小值的桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

The following snippet calculates the minimum of the total monthly `sales`:
以下代码段计算每月总金额的最小值`sales`：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    },
    "min_monthly_sales": {
      "min_bucket": {
        "buckets_path": "sales_per_month>sales" (1)
      }
    }
  }
}
```

1. `buckets_path` instructs this min_bucket aggregation that we want the minimum value of the `sales` aggregation in the `sales_per_month` date histogram.
   `buckets_path`指示此 min_bucket 聚合我们想要日期直方图中`sales`聚合 的最小值`sales_per_month`。

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               }
            }
         ]
      },
      "min_monthly_sales": {
          "keys": ["2015/02/01 00:00:00"], (1)
          "value": 60.0
      }
   }
}
```

1. `keys` is an array of strings since the minimum value may be present in multiple buckets
   `keys` 是一个字符串数组，因为最小值可能存在于多个存储桶中

#  Moving function aggregation

Given an ordered series of data, the Moving Function aggregation will slide a window across the data and allow the user to specify a custom script that is executed on each window of data. For convenience, a number of common functions are predefined such as min/max, moving averages, etc.

给定一系列有序数据，移动函数聚合将在数据上滑动一个窗口，并允许用户指定在每个数据窗口上执行的自定义脚本。为方便起见，预定义了许多常用函数，例如最小值/最大值、移动平均值等。

###  Syntax

A `moving_fn` aggregation looks like this in isolation:

单独的`moving_fn`聚合看起来像这样：

```console
{
  "moving_fn": {
    "buckets_path": "the_sum",
    "window": 10,
    "script": "MovingFunctions.min(values)"
  }
}
```

**Table 59. `moving_fn` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | Path to the metric of interest (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details<br />感兴趣的度量的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) | Required |               |
| `window`       | The size of window to "slide" across the histogram.<br />在直方图上“滑动”的窗口大小。 | Required |               |
| `script`       | The script that should be executed on each window of data<br />应该在每个数据窗口上执行的脚本 | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data. See [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy).<br />在数据中发现差距时应用的策略。请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)。 | Optional | `skip`        |
| `shift`        | [Shift](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-movfn-aggregation.html#shift-parameter) of window position.<br />窗口位置[偏移](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-movfn-aggregation.html#shift-parameter)。 | Optional | 0             |

`moving_fn` aggregations must be embedded inside of a `histogram` or `date_histogram` aggregation. They can be embedded like any other metric aggregation:

`moving_fn`聚合必须嵌入在 a`histogram`或`date_histogram`聚合中。它们可以像任何其他指标聚合一样嵌入：

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {                  (1)
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }   (2)
        },
        "the_movfn": {
          "moving_fn": {
            "buckets_path": "the_sum",  (3)
            "window": 10,
            "script": "MovingFunctions.unweightedAvg(values)"
          }
        }
      }
    }
  }
}
```

1. A `date_histogram` named "my_date_histo" is constructed on the "timestamp" field, with one-day intervals

   `date_histogram`在“时间戳”字段上构造了一个名为“my_date_histo”的名称，间隔为一天

2.  A `sum` metric is used to calculate the sum of a field. This could be any numeric metric (sum, min, max, etc)

   `sum`度量用于计算一个字段的总和。这可以是任何数字度量（总和、最小值、最大值等）

3.  Finally, we specify a `moving_fn` aggregation which uses "the_sum" metric as its input.

    最后，我们指定一个`moving_fn`使用“the_sum”度量作为输入的聚合。

Moving averages are built by first specifying a `histogram` or `date_histogram` over a field. You can then optionally add numeric metrics, such as a `sum`, inside of that histogram. Finally, the `moving_fn` is embedded inside the histogram. The `buckets_path` parameter is then used to "point" at one of the sibling metrics inside of the histogram (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for a description of the syntax for `buckets_path`.

移动平均线是通过首先指定一个`histogram`或`date_histogram`一个字段来构建的。然后，您可以选择`sum`在该直方图中添加数字度量，例如。最后，将`moving_fn`嵌入到直方图中。所述`buckets_path`然后参数是在直方图内的兄弟度量之一用来“点”（见 [`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)的语法的描述`buckets_path`。

An example response from the above aggregation may look like:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "my_date_histo": {
         "buckets": [
             {
                 "key_as_string": "2015/01/01 00:00:00",
                 "key": 1420070400000,
                 "doc_count": 3,
                 "the_sum": {
                    "value": 550.0
                 },
                 "the_movfn": {
                    "value": null
                 }
             },
             {
                 "key_as_string": "2015/02/01 00:00:00",
                 "key": 1422748800000,
                 "doc_count": 2,
                 "the_sum": {
                    "value": 60.0
                 },
                 "the_movfn": {
                    "value": 550.0
                 }
             },
             {
                 "key_as_string": "2015/03/01 00:00:00",
                 "key": 1425168000000,
                 "doc_count": 2,
                 "the_sum": {
                    "value": 375.0
                 },
                 "the_movfn": {
                    "value": 305.0
                 }
             }
         ]
      }
   }
}
```

###  Custom user scripting

The Moving Function aggregation allows the user to specify any arbitrary script to define custom logic. The script is invoked each time a new window of data is collected. These values are provided to the script in the `values` variable. The script should then perform some kind of calculation and emit a single `double` as the result. Emitting `null` is not permitted, although `NaN` and +/- `Inf` are allowed.

移动函数聚合允许用户指定任意脚本来定义自定义逻辑。每次收集新的数据窗口时都会调用该脚本。这些值在`values`变量中提供给脚本。然后脚本应该执行某种计算并发出单个`double`作为结果。`null`尽管允许`NaN`和 +/- ，但不允许发射`Inf`。

For example, this script will simply return the first value from the window, or `NaN` if no values are available:

例如，此脚本将简单地从窗口返回第一个值，或者`NaN`如果没有可用值：

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "return values.length > 0 ? values[0] : Double.NaN"
          }
        }
      }
    }
  }
}
```

###  shift parameter

By default (with `shift = 0`), the window that is offered for calculation is the last `n` values excluding the current bucket. Increasing `shift` by 1 moves starting window position by `1` to the right.

默认情况下（使用`shift = 0`），提供用于计算的窗口是`n`不包括当前存储桶的最后一个值。增加`shift`1 将起始窗口位置`1`向右移动。

- To include current bucket to the window, use `shift = 1`.

  要将当前存储桶包含到窗口中，请使用`shift = 1`.

- For center alignment (`n / 2` values before and after the current bucket), use `shift = window / 2`.

  对于中心对齐（`n / 2`当前存储桶之前和之后的值），请使用`shift = window / 2`.

- For right alignment (`n` values after the current bucket), use `shift = window`.

  对于右对齐（`n`当前存储桶之后的值），请使用`shift = window`.

If either of window edges moves outside the borders of data series, the window shrinks to include available values only.

如果任一窗口边缘移动到数据系列的边界之外，则窗口会缩小以仅包含可用值。

###  Pre-built Functions

For convenience, a number of functions have been prebuilt and are available inside the `moving_fn` script context:

为方便起见，已预先构建了许多函数并可在`moving_fn`脚本上下文中使用：

- `max()`
- `min()`
- `sum()`
- `stdDev()`
- `unweightedAvg()`
- `linearWeightedAvg()`
- `ewma()`
- `holt()`
- `holtWinters()`

The functions are available from the `MovingFunctions` namespace. E.g. `MovingFunctions.max()`

这些函数可从`MovingFunctions`命名空间中获得。例如`MovingFunctions.max()`

#### max Function

This function accepts a collection of doubles and returns the maximum value in that window. `null` and `NaN` values are ignored; the maximum is only calculated over the real values. If the window is empty, or all values are `null`/`NaN`, `NaN` is returned as the result.

此函数接受双精度集合并返回该窗口中的最大值。`null`和`NaN`值被忽略；最大值仅根据实际值计算。如果窗口为空，或者所有值都是`null`/ `NaN`，`NaN`则作为结果返回。

**Table 60. `max(double[] values)` Parameters**

| Parameter Name | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `values`       | The window of values to find the maximum<br />查找最大值的值窗口 |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_moving_max": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "MovingFunctions.max(values)"
          }
        }
      }
    }
  }
}
```

####  min Function

This function accepts a collection of doubles and returns the minimum value in that window. `null` and `NaN` values are ignored; the minimum is only calculated over the real values. If the window is empty, or all values are `null`/`NaN`, `NaN` is returned as the result.
此函数接受双精度集合并返回该窗口中的最小值。 `null`和`NaN`值被忽略；最小值仅根据实际值计算。如果窗口为空，或者所有值都是`null`/ `NaN`，`NaN`则作为结果返回。

**Table 61. `min(double[] values)` Parameters**

| Parameter Name | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `values`       | The window of values to find the minimum<br />寻找最小值的值窗口 |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_moving_min": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "MovingFunctions.min(values)"
          }
        }
      }
    }
  }
}
```

####  sum Function

This function accepts a collection of doubles and returns the sum of the values in that window. `null` and `NaN` values are ignored; the sum is only calculated over the real values. If the window is empty, or all values are `null`/`NaN`, `0.0` is returned as the result.

此函数接受双精度集合并返回该窗口中值的总和。 `null`和`NaN`值被忽略；总和仅根据实际值计算。如果窗口为空，或者所有值都是`null`/ `NaN`，`0.0`则作为结果返回。

**Table 62. `sum(double[] values)` Parameters**

| Parameter Name | Description                                                 |
| -------------- | ----------------------------------------------------------- |
| `values`       | The window of values to find the sum of<br />求和的值的窗口 |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_moving_sum": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "MovingFunctions.sum(values)"
          }
        }
      }
    }
  }
}
```

####  stdDev Function

This function accepts a collection of doubles and average, then returns the standard deviation of the values in that window. `null` and `NaN` values are ignored; the sum is only calculated over the real values. If the window is empty, or all values are `null`/`NaN`, `0.0` is returned as the result.

此函数接受双精度和平均值的集合，然后返回该窗口中值的标准偏差。 `null`和`NaN`值被忽略；总和仅根据实际值计算。如果窗口为空，或者所有值都是 `null`/ `NaN`，`0.0`则作为结果返回。

**Table 63. `stdDev(double[] values)` Parameters**

| Parameter Name | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `values`       | The window of values to find the standard deviation of<br />用于查找标准偏差的值的窗口 |
| `avg`          | The average of the window<br />窗口平均值                    |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_moving_sum": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "MovingFunctions.stdDev(values, MovingFunctions.unweightedAvg(values))"
          }
        }
      }
    }
  }
}
```

The `avg` parameter must be provided to the standard deviation function because different styles of averages can be computed on the window (simple, linearly weighted, etc). The various moving averages that are detailed below can be used to calculate the average for the standard deviation function.

该`avg`参数必须提供给标准偏差函数，因为可以在窗口上计算不同风格的平均值（简单、线性加权等）。下面详述的各种移动平均值可用于计算标准偏差函数的平均值。

####  unweightedAvg Function

The `unweightedAvg` function calculates the sum of all values in the window, then divides by the size of the window. It is effectively a simple arithmetic mean of the window. The simple moving average does not perform any time-dependent weighting, which means the values from a `simple` moving average tend to "lag" behind the real data.

该`unweightedAvg`函数计算窗口中所有值的总和，然后除以窗口的大小。它实际上是窗口的简单算术平均值。简单移动平均线不执行任何与时间相关的加权，这意味着`simple`移动平均线的值往往“滞后”于实际数据。

`null` and `NaN` values are ignored; the average is only calculated over the real values. If the window is empty, or all values are `null`/`NaN`, `NaN` is returned as the result. This means that the count used in the average calculation is count of non-`null`,non-`NaN` values.

`null`和`NaN`值被忽略；平均值仅根据实际值计算。如果窗口为空，或者所有值都是 `null`/ `NaN`，`NaN`则作为结果返回。这意味着在平均计算中使用的计数是非`null`、非`NaN` 值的计数。

**Table 64. `unweightedAvg(double[] values)` Parameters**

| Parameter Name | Description                                                 |
| -------------- | ----------------------------------------------------------- |
| `values`       | The window of values to find the sum of<br />求和的值的窗口 |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "MovingFunctions.unweightedAvg(values)"
          }
        }
      }
    }
  }
}
```

###  linearWeightedAvg Function

The `linearWeightedAvg` function assigns a linear weighting to points in the series, such that "older" datapoints (e.g. those at the beginning of the window) contribute a linearly less amount to the total average. The linear weighting helps reduce the "lag" behind the data’s mean, since older points have less influence.

该`linearWeightedAvg`函数为系列中的点分配线性权重，以便“较旧”的数据点（例如窗口开头的数据点）对总平均值的贡献线性较小。线性加权有助于减少数据均值后面的“滞后”，因为较旧的点影响较小。

If the window is empty, or all values are `null`/`NaN`, `NaN` is returned as the result.

如果窗口为空，或者所有值都是`null`/ `NaN`，`NaN`则作为结果返回。

**Table 65. `linearWeightedAvg(double[] values)` Parameters**

| Parameter Name | Description                                                 |
| -------------- | ----------------------------------------------------------- |
| `values`       | The window of values to find the sum of<br />求和的值的窗口 |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "MovingFunctions.linearWeightedAvg(values)"
          }
        }
      }
    }
  }
}
```

###  ewma Function

The `ewma` function (aka "single-exponential") is similar to the `linearMovAvg` function, except older data-points become exponentially less important, rather than linearly less important. The speed at which the importance decays can be controlled with an `alpha` setting. Small values make the weight decay slowly, which provides greater smoothing and takes into account a larger portion of the window. Larger values make the weight decay quickly, which reduces the impact of older values on the moving average. This tends to make the moving average track the data more closely but with less smoothing.

`ewma`函数（又名“单指数”）类似于该`linearMovAvg`函数，不同之处在于旧数据点的重要性呈指数下降，而不是线性下降。重要性衰减的速度可以通过`alpha` 设置来控制。较小的值会使权重衰减缓慢，从而提供更大的平滑度并考虑更大的窗口部分。较大的值会使权重快速衰减，从而减少旧值对移动平均线的影响。这往往会使移动平均线更紧密地跟踪数据，但平滑度较低。

`null` and `NaN` values are ignored; the average is only calculated over the real values. If the window is empty, or all values are `null`/`NaN`, `NaN` is returned as the result. This means that the count used in the average calculation is count of non-`null`,non-`NaN` values.

`null`和`NaN`值被忽略；平均值仅根据实际值计算。如果窗口为空，或者所有值都是 `null`/ `NaN`，`NaN`则作为结果返回。这意味着在平均计算中使用的计数是非`null`、非`NaN` 值的计数。

**Table 66. `ewma(double[] values, double alpha)` Parameters**

| Parameter Name | Description                                                 |
| -------------- | ----------------------------------------------------------- |
| `values`       | The window of values to find the sum of<br />求和的值的窗口 |
| `alpha`        | Exponential decay<br />指数衰减                             |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "MovingFunctions.ewma(values, 0.3)"
          }
        }
      }
    }
  }
}
```

###  holt Function

The `holt` function (aka "double exponential") incorporates a second exponential term which tracks the data’s trend. Single exponential does not perform well when the data has an underlying linear trend. The double exponential model calculates two values internally: a "level" and a "trend".

`holt`函数（又名“双指数”）包含跟踪数据趋势的第二个指数项。当数据具有潜在的线性趋势时，单指数表现不佳。双指数模型在内部计算两个值：“水平”和“趋势”。

The level calculation is similar to `ewma`, and is an exponentially weighted view of the data. The difference is that the previously smoothed value is used instead of the raw value, which allows it to stay close to the original series. The trend calculation looks at the difference between the current and last value (e.g. the slope, or trend, of the smoothed data). The trend value is also exponentially weighted.

水平计算类似于`ewma`，是数据的指数加权视图。不同之处在于使用了先前平滑的值而不是原始值，这使其与原始系列保持接近。趋势计算着眼于当前值和上一个值之间的差异（例如平滑数据的斜率或趋势）。趋势值也是指数加权的。

Values are produced by multiplying the level and trend components.

值是通过将水平和趋势分量相乘而产生的。

`null` and `NaN` values are ignored; the average is only calculated over the real values. If the window is empty, or all values are `null`/`NaN`, `NaN` is returned as the result. This means that the count used in the average calculation is count of non-`null`,non-`NaN` values.

`null`和`NaN`值被忽略；平均值仅根据实际值计算。如果窗口为空，或者所有值都是 `null`/ `NaN`，`NaN`则作为结果返回。这意味着在平均计算中使用的计数是非`null`、非`NaN` 值的计数。

**Table 67. `holt(double[] values, double alpha)` Parameters**

| Parameter Name | Description                                                 |
| -------------- | ----------------------------------------------------------- |
| `values`       | The window of values to find the sum of<br />求和的值的窗口 |
| `alpha`        | Level decay value<br />电平衰减值                           |
| `beta`         | Trend decay value<br />趋势衰减值                           |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "MovingFunctions.holt(values, 0.3, 0.1)"
          }
        }
      }
    }
  }
}
```

In practice, the `alpha` value behaves very similarly in `holtMovAvg` as `ewmaMovAvg`: small values produce more smoothing and more lag, while larger values produce closer tracking and less lag. The value of `beta` is often difficult to see. Small values emphasize long-term trends (such as a constant linear trend in the whole series), while larger values emphasize short-term trends.

在实践中，`alpha`值的行为非常类似`holtMovAvg`如`ewmaMovAvg`：小的值产生更平滑和更滞后，而较大的值产生更接近跟踪和较少的滞后。的价值`beta`往往很难看到。较小的值强调长期趋势（例如整个系列中的恒定线性趋势），而较大的值强调短期趋势。

###  holtWinters Function

The `holtWinters` function (aka "triple exponential") incorporates a third exponential term which tracks the seasonal aspect of your data. This aggregation therefore smooths based on three components: "level", "trend" and "seasonality".

该`holtWinters`函数（又名“三重指数”）包含跟踪数据季节性方面的第三个指数项。因此，这种聚合基于三个组成部分进行平滑处理：“水平”、“趋势”和“季节性”。

The level and trend calculation is identical to `holt` The seasonal calculation looks at the difference between the current point, and the point one period earlier.

水平和趋势计算与`holt`季节性计算查看当前点和前一周期点之间的差异。

Holt-Winters requires a little more handholding than the other moving averages. You need to specify the "periodicity" of your data: e.g. if your data has cyclic trends every 7 days, you would set `period = 7`. Similarly if there was a monthly trend, you would set it to `30`. There is currently no periodicity detection, although that is planned for future enhancements.

Holt-Winters 比其他移动平均线需要更多的控制。您需要指定数据的“周期性”：例如，如果您的数据每 7 天有一次循环趋势，您将设置`period = 7`. 同样，如果有月度趋势，您可以将其设置为`30`。目前没有周期性检测，但计划用于未来的增强。

`null` and `NaN` values are ignored; the average is only calculated over the real values. If the window is empty, or all values are `null`/`NaN`, `NaN` is returned as the result. This means that the count used in the average calculation is count of non-`null`,non-`NaN` values.

`null`和`NaN`值被忽略；平均值仅根据实际值计算。如果窗口为空，或者所有值都是 `null`/ `NaN`，`NaN`则作为结果返回。这意味着在平均计算中使用的计数是非`null`、非`NaN` 值的计数。

**Table 68. `holtWinters(double[] values, double alpha)` Parameters**

| Parameter Name   | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| `values`         | The window of values to find the sum of<br />求和的值的窗口  |
| `alpha`          | Level decay value<br />电平衰减值                            |
| `beta`           | Trend decay value<br />趋势衰减值                            |
| `gamma`          | Seasonality decay value<br />季节性衰减值                    |
| `period`         | The periodicity of the data<br />数据的周期性                |
| `multiplicative` | True if you wish to use multiplicative holt-winters, false to use additive<br />如果您希望使用乘法 holt-winter，则为 true，如果您希望使用加法，则为 false |

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_sum": {
          "sum": { "field": "price" }
        },
        "the_movavg": {
          "moving_fn": {
            "buckets_path": "the_sum",
            "window": 10,
            "script": "if (values.length > 5*2) {MovingFunctions.holtWinters(values, 0.3, 0.1, 0.1, 5, false)}"
          }
        }
      }
    }
  }
}
```

> WARNING: Multiplicative Holt-Winters works by dividing each data point by the seasonal value. This is problematic if any of your data is zero, or if there are gaps in the data (since this results in a divid-by-zero). To combat this, the `mult` Holt-Winters pads all values by a very small amount (1*10-10) so that all values are non-zero. This affects the result, but only minimally. If your data is non-zero, or you prefer to see `NaN` when zero’s are encountered, you can disable this behavior with `pad: false`
>
> 乘法 Holt-Winters 的工作原理是将每个数据点除以季节性值。如果您的任何数据为零，或者数据中存在差距（因为这会导致被零除），这就会出现问题。为了解决这个问题， `mult`Holt-Winters 将所有值填充了一个非常小的数量 (1*10 -10 )，以便所有值都不为零。这会影响结果，但影响很小。如果您的数据不为零，或者您希望看到`NaN`何时遇到零，您可以禁用此行为`pad: false`

####  "Cold Start"

Unfortunately, due to the nature of Holt-Winters, it requires two periods of data to "bootstrap" the algorithm. This means that your `window` must always be **at least** twice the size of your period. An exception will be thrown if it isn’t. It also means that Holt-Winters will not emit a value for the first `2 * period` buckets; the current algorithm does not backcast.

不幸的是，由于 Holt-Winters 的性质，它需要两个周期的数据来“引导”算法。这意味着您`window`必须始终**至少**是您经期的两倍。如果不是，则会抛出异常。这也意味着 Holt-Winters 不会为第一个`2 * period`桶发出值；当前算法不会回溯。

You’ll notice in the above example we have an `if ()` statement checking the size of values. This is checking to make sure we have two periods worth of data (`5 * 2`, where 5 is the period specified in the `holtWintersMovAvg` function) before calling the holt-winters function.

你会注意到在上面的例子中我们有一个`if ()`检查值大小的语句。这是为了确保在调用 holt-winters 函数之前我们有两个周期的数据（`5 * 2`，其中 5 是`holtWintersMovAvg`函数中指定的周期）。

# Moving percentiles aggregation

Given an ordered series of [percentiles](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html), the Moving Percentile aggregation will slide a window across those percentiles and allow the user to compute the cumulative percentile.

给定一系列有序的[百分位数](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html)，移动百分位数聚合将在这些百分位数上滑动一个窗口，并允许用户计算累积百分位数。

This is conceptually very similar to the [Moving Function](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-movfn-aggregation.html) pipeline aggregation, except it works on the percentiles sketches instead of the actual buckets values.

这在概念上与[移动函数](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-movfn-aggregation.html)管道聚合非常相似，不同之处在于它适用于百分位数草图而不是实际的桶值。

###  Syntax

A `moving_percentiles` aggregation looks like this in isolation:

单独的`moving_percentiles`聚合看起来像这样：

```console
{
  "moving_percentiles": {
    "buckets_path": "the_percentile",
    "window": 10
  }
}
```

**Table 69. `moving_percentiles` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | Path to the percentile of interest (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details<br /> 兴趣百分位数的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) | Required |               |
| `window`       | The size of window to "slide" across the histogram.<br />在直方图上“滑动”的窗口大小。 | Required |               |
| `shift`        | [Shift](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-movfn-aggregation.html#shift-parameter) of window position.<br />窗口位置[偏移](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-movfn-aggregation.html#shift-parameter)。 | Optional | 0             |

`moving_percentiles` aggregations must be embedded inside of a `histogram` or `date_histogram` aggregation. They can be embedded like any other metric aggregation:

`moving_percentiles`聚合必须嵌入在 a`histogram`或`date_histogram`聚合中。它们可以像任何其他指标聚合一样嵌入：

```console
POST /_search
{
  "size": 0,
  "aggs": {
    "my_date_histo": {                          (1)
        "date_histogram": {
        "field": "date",
        "calendar_interval": "1M"
      },
      "aggs": {
        "the_percentile": {                     (2)
            "percentiles": {
            "field": "price",
            "percents": [ 1.0, 99.0 ]
          }
        },
        "the_movperc": {
          "moving_percentiles": {
            "buckets_path": "the_percentile",   (3)
            "window": 10
          }
        }
      }
    }
  }
}
```

1.  A `date_histogram` named "my_date_histo" is constructed on the "timestamp" field, with one-day intervals

   `date_histogram`在“时间戳”字段上构造了一个名为“my_date_histo”的名称，间隔为一天

2.  A `percentile` metric is used to calculate the percentiles of a field.

   `percentile`度量用于计算字段的百分。

3.  Finally, we specify a `moving_percentiles` aggregation which uses "the_percentile" sketch as its input.

    最后，我们指定一个`moving_percentiles`使用“the_percentile”草图作为输入的聚合。

Moving percentiles are built by first specifying a `histogram` or `date_histogram` over a field. You then add a percentile metric inside of that histogram. Finally, the `moving_percentiles` is embedded inside the histogram. The `buckets_path` parameter is then used to "point" at the percentiles aggregation inside of the histogram (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for a description of the syntax for `buckets_path`).

移动百分位数是通过首先指定一个`histogram`或`date_histogram`一个字段来构建的。然后在该直方图中添加一个百分位指标。最后，将`moving_percentiles`嵌入到直方图中。`buckets_path`然后，该参数用于“指向”直方图中的百分位数聚合（有关 的[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)说明，请参阅 [语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)`buckets_path`）。

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "my_date_histo": {
         "buckets": [
             {
                 "key_as_string": "2015/01/01 00:00:00",
                 "key": 1420070400000,
                 "doc_count": 3,
                 "the_percentile": {
                     "values": {
                       "1.0": 150.0,
                       "99.0": 200.0
                     }
                 }
             },
             {
                 "key_as_string": "2015/02/01 00:00:00",
                 "key": 1422748800000,
                 "doc_count": 2,
                 "the_percentile": {
                     "values": {
                       "1.0": 10.0,
                       "99.0": 50.0
                     }
                 },
                 "the_movperc": {
                   "values": {
                     "1.0": 150.0,
                     "99.0": 200.0
                   }
                 }
             },
             {
                 "key_as_string": "2015/03/01 00:00:00",
                 "key": 1425168000000,
                 "doc_count": 2,
                 "the_percentile": {
                    "values": {
                      "1.0": 175.0,
                      "99.0": 200.0
                    }
                 },
                 "the_movperc": {
                    "values": {
                      "1.0": 10.0,
                      "99.0": 200.0
                    }
                 }
             }
         ]
      }
   }
}
```

The output format of the `moving_percentiles` aggregation is inherited from the format of the referenced [`percentiles`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html) aggregation.

`moving_percentiles`聚合的输出格式继承自引用[`percentiles`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html)聚合的格式 。

Moving percentiles pipeline aggregations always run with `skip` gap policy.

移动百分位数管道聚合始终使用`skip`间隙策略运行。

###  shift parameter

By default (with `shift = 0`), the window that is offered for calculation is the last `n` values excluding the current bucket. Increasing `shift` by 1 moves starting window position by `1` to the right.

默认情况下（使用`shift = 0`），提供用于计算的窗口是`n`不包括当前存储桶的最后一个值。增加`shift`1 将起始窗口位置`1`向右移动。

- To include current bucket to the window, use `shift = 1`.

  要将当前存储桶包含到窗口中，请使用`shift = 1`.

- For center alignment (`n / 2` values before and after the current bucket), use `shift = window / 2`.

  对于中心对齐（`n / 2`当前存储桶之前和之后的值），请使用`shift = window / 2`.

- For right alignment (`n` values after the current bucket), use `shift = window`.

  对于右对齐（`n`当前存储桶之后的值），请使用`shift = window`.

If either of window edges moves outside the borders of data series, the window shrinks to include available values only.

如果任一窗口边缘移动到数据系列的边界之外，则窗口会缩小以仅包含可用值。

# Normalize aggregation

A parent pipeline aggregation which calculates the specific normalized/rescaled value for a specific bucket value. Values that cannot be normalized, will be skipped using the [skip gap policy](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy).

计算特定桶值的特定标准化/重新缩放值的父管道聚合。无法标准化的值将使用[跳过间隙策略跳过](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)。

###  Syntax

A `normalize` aggregation looks like this in isolation:

单独的`normalize`聚合看起来像这样：

```console
{
  "normalize": {
    "buckets_path": "normalized",
    "method": "percent_of_sum"
  }
}
```

**Table 70. `normalize_pipeline` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the buckets we wish to normalize (see [`buckets_path` syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望标准化的存储桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `method`       | The specific [method](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-normalize-aggregation.html#normalize_pipeline-method) to apply<br />具体申请[方法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-normalize-aggregation.html#normalize_pipeline-method) | Required |               |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

### Methods

The Normalize Aggregation supports multiple methods to transform the bucket values. Each method definition will use the following original set of bucket values as examples: `[5, 5, 10, 50, 10, 20]`.
Normalize Aggregation 支持多种方法来转换桶值。每个方法定义都将使用以下原始存储桶值集作为示例：`[5, 5, 10, 50, 10, 20]`.

- ***rescale_0_1\***

  This method rescales the data such that the minimum number is zero, and the maximum number is 1, with the rest normalized linearly in-between.

  此方法重新调整数据，使得最小数为零，最大数为 1，其余部分线性归一化。

  ```
  x' = (x - min_x) / (max_x - min_x)
  [0, 0, .1111, 1, .1111, .3333]
  ```

- ***rescale_0_100\***

  This method rescales the data such that the minimum number is zero, and the maximum number is 100, with the rest normalized linearly in-between.

  此方法重新调整数据，使得最小数为零，最大数为 100，其余数在两者之间线性归一化。

  ```
  x' = 100 * (x - min_x) / (max_x - min_x)
  [0, 0, 11.11, 100, 11.11, 33.33]
  ```

- ***percent_of_sum\***

  This method normalizes each value so that it represents a percentage of the total sum it attributes to.

  此方法对每个值进行归一化，使其代表其归属于的总和的百分比。

  ```
  x' = x / sum_x
  [5%, 5%, 10%, 50%, 10%, 20%]
  ```

- ***mean\***

  This method normalizes such that each value is normalized by how much it differs from the average.

  此方法进行标准化，以便通过每个值与平均值的差异来标准化每个值。

  ```
  x' = (x - mean_x) / (max_x - min_x)
  [4.63, 4.63, 9.63, 49.63, 9.63, 9.63, 19.63]
  ```

- ***zscore\***

  This method normalizes such that each value represents how far it is from the mean relative to the standard deviation

  此方法标准化，使得每个值表示相对于标准偏差与平均值的距离

  ```
  x' = (x - mean_x) / stdev_x
  [-0.68, -0.68, -0.39, 1.94, -0.39, 0.19]
  ```

- ***softmax\***

  This method normalizes such that each value is exponentiated and relative to the sum of the exponents of the original values.

  此方法标准化，使得每个值都被取幂并相对于原始值的指数总和。

  ```
  x' = e^x / sum_e_x
  [2.862E-20, 2.862E-20, 4.248E-18, 0.999, 9.357E-14, 4.248E-18]
  ```

###  Example

The following snippet calculates the percent of total sales for each month:

以下代码段计算每个月的总销售额百分比：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        },
        "percent_of_total_sales": {
          "normalize": {
            "buckets_path": "sales",          (1)
            "method": "percent_of_sum",       (2)
            "format": "00.00%"                (3)
          }
        }
      }
    }
  }
}
```

1. `buckets_path` instructs this normalize aggregation to use the output of the `sales` aggregation for rescaling

   `buckets_path`指示此规范化聚合使用聚合的输出`sales`进行重新缩放

2. `method` sets which rescaling to apply. In this case, `percent_of_sum` will calculate the sales value as a percent of all sales in the parent bucket

   `method`设置要应用的重新缩放。在这种情况下，`percent_of_sum`将计算销售额占父存储桶中所有销售额的百分比

3. `format` influences how to format the metric as a string using Java’s `DecimalFormat` pattern. In this case, multiplying by 100 and adding a *%*

   `format`影响如何使用 Java 的`DecimalFormat`模式将指标格式化为字符串。在这种情况下，乘以 100 并添加*%*

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               },
               "percent_of_total_sales": {
                  "value": 0.5583756345177665,
                  "value_as_string": "55.84%"
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               },
               "percent_of_total_sales": {
                  "value": 0.06091370558375635,
                  "value_as_string": "06.09%"
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               },
               "percent_of_total_sales": {
                  "value": 0.38071065989847713,
                  "value_as_string": "38.07%"
               }
            }
         ]
      }
   }
}
```

#  Percentiles bucket aggregation

A sibling pipeline aggregation which calculates percentiles across all bucket of a specified metric in a sibling aggregation. The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.

同级管道聚合，它计算同级聚合中指定指标的所有存储桶的百分位数。指定的指标必须是数字，同级聚合必须是多桶聚合。

### Syntax

A `percentiles_bucket` aggregation looks like this in isolation:

单独的`percentiles_bucket`聚合看起来像这样：

```console
{
  "percentiles_bucket": {
    "buckets_path": "the_sum"
  }
}
```

**Table 71. `percentiles_bucket` Parameters**

| Parameter Name | Description                                                  | Required | Default Value                  |
| -------------- | ------------------------------------------------------------ | -------- | ------------------------------ |
| `buckets_path` | The path to the buckets we wish to find the percentiles for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望为其找到百分位数的桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |                                |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`                         |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`                         |
| `percents`     | The list of percentiles to calculate<br />要计算的百分位数列表 | Optional | `[ 1, 5, 25, 50, 75, 95, 99 ]` |
| `keyed`        | Flag which returns the range as an hash instead of an array of key-value pairs<br />将范围作为散列而不是键值对数组返回的标志 | Optional | `true`                         |

The following snippet calculates the percentiles for the total monthly `sales` buckets:

以下代码段计算每月总`sales`存储桶的百分位数：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    },
    "percentiles_monthly_sales": {
      "percentiles_bucket": {
        "buckets_path": "sales_per_month>sales", (1)
        "percents": [ 25.0, 50.0, 75.0 ]         (2)
      }
    }
  }
}
```

1. `buckets_path` instructs this percentiles_bucket aggregation that we want to calculate percentiles for the `sales` aggregation in the `sales_per_month` date histogram.

   `buckets_path`指示这个 percentiles_bucket 聚合我们要计算日期直方图中`sales`聚合的百分位数`sales_per_month`。

2. `percents` specifies which percentiles we wish to calculate, in this case, the 25th, 50th and 75th percentiles.

   `percents` 指定我们希望计算的百分位数，在本例中为第 25、第 50 和第 75 个百分位数。

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               }
            }
         ]
      },
      "percentiles_monthly_sales": {
        "values" : {
            "25.0": 375.0,
            "50.0": 375.0,
            "75.0": 550.0
         }
      }
   }
}
```

###  Percentiles_bucket implementation

The Percentile Bucket returns the nearest input data point that is not greater than the requested percentile; it does not interpolate between data points.

Percentile Bucket 返回不大于请求的百分位的最近输入数据点；它不会在数据点之间进行插值。

The percentiles are calculated exactly and is not an approximation (unlike the Percentiles Metric). This means the implementation maintains an in-memory, sorted list of your data to compute the percentiles, before discarding the data. You may run into memory pressure issues if you attempt to calculate percentiles over many millions of data-points in a single `percentiles_bucket`.

百分位数是精确计算的，不是近似值（与百分位数指标不同）。这意味着在丢弃数据之前，实现会在内存中维护数据的排序列表以计算百分位数。如果您尝试在单个 .csv 文件中计算数百万个数据点的百分位数，您可能会遇到内存压力问题`percentiles_bucket`。

# Serial differencing aggregation

Serial differencing is a technique where values in a time series are subtracted from itself at different time lags or periods. For example, the datapoint f(x) = f(xt) - f(xt-n), where n is the period being used.

序列差分是一种技术，其中时间序列中的值在不同的时间滞后或周期从自身中减去。例如，数据点 f(x) = f(x t ) - f(x t-n )，其中 n 是使用的周期。

A period of 1 is equivalent to a derivative with no time normalization: it is simply the change from one point to the next. Single periods are useful for removing constant, linear trends.

周期为 1 相当于没有时间归一化的导数：它只是从一个点到下一个点的变化。单个周期可用于消除恒定的线性趋势。

Single periods are also useful for transforming data into a stationary series. In this example, the Dow Jones is plotted over ~250 days. The raw data is not stationary, which would make it difficult to use with some techniques.

单周期也可用于将数据转换为平稳序列。在这个例子中，道琼斯指数绘制了大约 250 天。原始数据不是固定的，这将使其难以与某些技术一起使用。

By calculating the first-difference, we de-trend the data (e.g. remove a constant, linear trend). We can see that the data becomes a stationary series (e.g. the first difference is randomly distributed around zero, and doesn’t seem to exhibit any pattern/behavior). The transformation reveals that the dataset is following a random-walk; the value is the previous value +/- a random amount. This insight allows selection of further tools for analysis.

通过计算一阶差分，我们对数据进行去趋势化（例如，移除恒定的线性趋势）。我们可以看到数据变成了一个平稳序列（例如，第一个差异随机分布在零附近，并且似乎没有表现出任何模式/行为）。转换表明数据集遵循随机游走；该值是前一个值 +/- 随机数量。这种洞察力允许选择进一步的分析工具。

**Figure 3. Dow Jones plotted and made stationary with first-differencing**
![dow](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/pipeline_serialdiff/dow.png)

Larger periods can be used to remove seasonal / cyclic behavior. In this example, a population of lemmings was synthetically generated with a sine wave + constant linear trend + random noise. The sine wave has a period of 30 days.

更大的周期可用于消除季节性/周期性行为。在这个例子中，一群旅鼠是用正弦波 + 恒定线性趋势 + 随机噪声综合生成的。正弦波的周期为 30 天。

The first-difference removes the constant trend, leaving just a sine wave. The 30th-difference is then applied to the first-difference to remove the cyclic behavior, leaving a stationary series which is amenable to other analysis.

一阶差分消除了恒定趋势，只留下一个正弦波。然后将第 30 次差值应用于第一个差值以消除循环行为，留下一个适合其他分析的平稳序列。

**Figure 4. Lemmings data plotted made stationary with 1st and 30th difference**

![lemmings](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/pipeline_serialdiff/lemmings.png)

###  Syntax

A `serial_diff` aggregation looks like this in isolation:

单独的`serial_diff`聚合看起来像这样：

```console
{
  "serial_diff": {
    "buckets_path": "the_sum",
    "lag": 7
  }
}
```

**Table 72. `serial_diff` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | Path to the metric of interest (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details<br />感兴趣的度量的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) | Required |               |
| `lag`          | The historical bucket to subtract from the current value. E.g. a lag of 7 will subtract the current value from the value 7 buckets ago. Must be a positive, non-zero integer<br />要从当前值中减去的历史存储桶。例如，滞后 7 将从 7 个桶之前的值中减去当前值。必须是非零的正整数 | Optional | `1`           |
| `gap_policy`   | Determines what should happen when a gap in the data is encountered.<br />确定遇到数据间隙时应该发生的情况。 | Optional | `insert_zero` |
| `format`       | Format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

`serial_diff` aggregations must be embedded inside of a `histogram` or `date_histogram` aggregation:
`serial_diff`聚合必须嵌入在 a`histogram`或`date_histogram`聚合中：

```console
POST /_search
{
   "size": 0,
   "aggs": {
      "my_date_histo": {                  (1)
         "date_histogram": {
            "field": "timestamp",
            "calendar_interval": "day"
         },
         "aggs": {
            "the_sum": {
               "sum": {
                  "field": "lemmings"     (2)
               }
            },
            "thirtieth_difference": {
               "serial_diff": {                (3)
                  "buckets_path": "the_sum",
                  "lag" : 30
               }
            }
         }
      }
   }
}
```

1.  A `date_histogram` named "my_date_histo" is constructed on the "timestamp" field, with one-day intervals

   `date_histogram`在“时间戳”字段上构造了一个名为“my_date_histo”的名称，间隔为一天

2.  A `sum` metric is used to calculate the sum of a field. This could be any metric (sum, min, max, etc)

   `sum`度量用于计算一个字段的总和。这可以是任何指标（总和、最小值、最大值等）

3.  Finally, we specify a `serial_diff` aggregation which uses "the_sum" metric as its input.

    最后，我们指定一个`serial_diff`使用“the_sum”度量作为输入的聚合。

Serial differences are built by first specifying a `histogram` or `date_histogram` over a field. You can then optionally add normal metrics, such as a `sum`, inside of that histogram. Finally, the `serial_diff` is embedded inside the histogram. The `buckets_path` parameter is then used to "point" at one of the sibling metrics inside of the histogram (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for a description of the syntax for `buckets_path`.

序列差异是通过首先指定一个`histogram`或`date_histogram`一个字段来构建的。然后，您可以选择`sum`在该直方图中添加正常指标，例如。最后，将`serial_diff`嵌入到直方图中。所述`buckets_path`然后参数是在直方图内的兄弟度量之一用来“点”（见 [`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)的语法的描述`buckets_path`。

#  Stats bucket aggregation

A sibling pipeline aggregation which calculates a variety of stats across all bucket of a specified metric in a sibling aggregation. The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.

同级管道聚合，它计算同级聚合中指定指标的所有存储桶的各种统计信息。指定的指标必须是数字，同级聚合必须是多桶聚合。

###  Syntax

A `stats_bucket` aggregation looks like this in isolation:

单独的`stats_bucket`聚合看起来像这样：

```console
{
  "stats_bucket": {
    "buckets_path": "the_sum"
  }
}
```

**Table 73. `stats_bucket` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the buckets we wish to calculate stats for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望为其计算统计数据的存储桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

The following snippet calculates the stats for monthly `sales`:

以下代码段计算每月的统计数据`sales`：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    },
    "stats_monthly_sales": {
      "stats_bucket": {
        "buckets_path": "sales_per_month>sales" (1)
      }
    }
  }
}
```

1. `bucket_paths` instructs this `stats_bucket` aggregation that we want the calculate stats for the `sales` aggregation in the `sales_per_month` date histogram.

   `bucket_paths`指示此`stats_bucket`聚合我们希望`sales`在`sales_per_month`日期直方图中计算聚合的 统计数据。

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               }
            }
         ]
      },
      "stats_monthly_sales": {
         "count": 3,
         "min": 60.0,
         "max": 550.0,
         "avg": 328.3333333333333,
         "sum": 985.0
      }
   }
}
```

# Sum bucket aggregation

A sibling pipeline aggregation which calculates the sum across all buckets of a specified metric in a sibling aggregation. The specified metric must be numeric and the sibling aggregation must be a multi-bucket aggregation.

同级管道聚合，它计算同级聚合中指定指标的所有桶的总和。指定的指标必须是数字，同级聚合必须是多桶聚合。

###  Syntax

A `sum_bucket` aggregation looks like this in isolation:

单独的`sum_bucket`聚合看起来像这样：

```console
{
  "sum_bucket": {
    "buckets_path": "the_sum"
  }
}
```

**Table 74. `sum_bucket` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `buckets_path` | The path to the buckets we wish to find the sum for (see [`buckets_path` Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax) for more details)<br />我们希望找到总和的桶的路径（有关更多详细信息，请参阅[`buckets_path`语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#buckets-path-syntax)） | Required |               |
| `gap_policy`   | The policy to apply when gaps are found in the data (see [Dealing with gaps in the data](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy) for more details)<br />在数据中发现差距时应用的政策（有关更多详细信息，请参阅[处理数据中的差距](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html#gap-policy)） | Optional | `skip`        |
| `format`       | format to apply to the output value of this aggregation<br />应用于此聚合的输出值的格式 | Optional | `null`        |

The following snippet calculates the sum of all the total monthly `sales` buckets:

以下代码段计算所有每月总`sales`存储桶的总和：

```console
POST /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    },
    "sum_monthly_sales": {
      "sum_bucket": {
        "buckets_path": "sales_per_month>sales" (1)
      }
    }
  }
}
```

1. `buckets_path` instructs this sum_bucket aggregation that we want the sum of the `sales` aggregation in the `sales_per_month` date histogram.

   `buckets_path`指示这个 sum_bucket 聚合我们想要日期直方图中的`sales`聚合 总和`sales_per_month`。

And the following may be the response:

```console-result
{
   "took": 11,
   "timed_out": false,
   "_shards": ...,
   "hits": ...,
   "aggregations": {
      "sales_per_month": {
         "buckets": [
            {
               "key_as_string": "2015/01/01 00:00:00",
               "key": 1420070400000,
               "doc_count": 3,
               "sales": {
                  "value": 550.0
               }
            },
            {
               "key_as_string": "2015/02/01 00:00:00",
               "key": 1422748800000,
               "doc_count": 2,
               "sales": {
                  "value": 60.0
               }
            },
            {
               "key_as_string": "2015/03/01 00:00:00",
               "key": 1425168000000,
               "doc_count": 2,
               "sales": {
                  "value": 375.0
               }
            }
         ]
      },
      "sum_monthly_sales": {
          "value": 985.0
      }
   }
}
```