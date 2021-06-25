#  Metrics aggregations

The aggregations in this family compute metrics based on values extracted in one way or another from the documents that are being aggregated. The values are typically extracted from the fields of the document (using the field data), but can also be generated using scripts.

此系列中的聚合基于从正在聚合的文档中以一种或另一种方式提取的值来计算度量。这些值通常从文档的字段中提取（使用字段数据），但也可以使用脚本生成。

Numeric metrics aggregations are a special type of metrics aggregation which output numeric values. Some aggregations output a single numeric metric (e.g. `avg`) and are called `single-value numeric metrics aggregation`, others generate multiple metrics (e.g. `stats`) and are called `multi-value numeric metrics aggregation`. The distinction between single-value and multi-value numeric metrics aggregations plays a role when these aggregations serve as direct sub-aggregations of some bucket aggregations (some bucket aggregations enable you to sort the returned buckets based on the numeric metrics in each bucket).

数字度量聚合是一种特殊类型的度量聚合，它输出数值。一些聚合输出单个数字度量（例如`avg`）并称为`single-value numeric metrics aggregation`，其他聚合生成多个度量（例如`stats`）并称为`multi-value numeric metrics aggregation`。当这些聚合用作某些存储桶聚合的直接子聚合时（某些存储桶聚合使您能够根据每个存储桶中的数字度量对返回的存储桶进行排序），单值和多值数字度量聚合之间的区别会起作用。

# Avg aggregation

A `single-value` metrics aggregation that computes the average of numeric values that are extracted from the aggregated documents. These values can be extracted either from specific numeric fields in the documents.

一种`single-value`度量聚合，用于计算从聚合文档中提取的数值的平均值。这些值可以从文档中的特定数字字段中提取。

Assuming the data consists of documents representing exams grades (between 0 and 100) of students we can average their scores with:

假设数据由代表学生考试成绩（0 到 100 之间）的文件组成，我们可以用以下方法平均他们的分数：

```console
POST /exams/_search?size=0
{
  "aggs": {
    "avg_grade": { "avg": { "field": "grade" } }
  }
}
```

The above aggregation computes the average grade over all documents. The aggregation type is `avg` and the `field` setting defines the numeric field of the documents the average will be computed on. The above will return the following:

上述聚合计算所有文档的平均等级。聚合类型是`avg`，并且`field`设置定义了将计算平均值的文档的数字字段。以上将返回以下内容：

```console-result
{
  ...
  "aggregations": {
    "avg_grade": {
      "value": 75.0
    }
  }
}
```

The name of the aggregation (`avg_grade` above) also serves as the key by which the aggregation result can be retrieved from the returned response.

聚合的名称（`avg_grade`如上）也用作可以从返回的响应中检索聚合结果的键。

### Script

Let’s say the exam was exceedingly difficult, and you need to apply a grade correction. Average a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) to get a corrected average:

假设考试非常困难，您需要进行成绩更正。平均[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)以获得更正的平均值：

```console
POST /exams/_search?size=0
{
  "runtime_mappings": {
    "grade.corrected": {
      "type": "double",
      "script": {
        "source": "emit(Math.min(100, doc['grade'].value * params.correction))",
        "params": {
          "correction": 1.2
        }
      }
    }
  },
  "aggs": {
    "avg_corrected_grade": {
      "avg": {
        "field": "grade.corrected"
      }
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
POST /exams/_search?size=0
{
  "aggs": {
    "grade_avg": {
      "avg": {
        "field": "grade",
        "missing": 10     (1)
      }
    }
  }
}
```

1.  Documents without a value in the `grade` field will fall into the same bucket as documents that have the value `10`.

    该`grade`字段中没有值的文档将与具有该值的文档落入同一个桶中`10`。

### Histogram fields

When average is computed on [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html), the result of the aggregation is the weighted average of all elements in the `values` array taking into consideration the number in the same position in the `counts` array.

当对[直方图字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)计算平均值时，聚合的结果是`values`数组中所有元素的加权平均值，同时考虑了数组中相同位置的数字`counts`。

For example, for the following index that stores pre-aggregated histograms with latency metrics for different networks:

例如，对于以下存储具有不同网络延迟指标的预聚合直方图的索引：

```console
PUT metrics_index/_doc/1
{
  "network.name" : "net-1",
  "latency_histo" : {
      "values" : [0.1, 0.2, 0.3, 0.4, 0.5], (1)
      "counts" : [3, 7, 23, 12, 6] (2)
   }
}

PUT metrics_index/_doc/2
{
  "network.name" : "net-2",
  "latency_histo" : {
      "values" :  [0.1, 0.2, 0.3, 0.4, 0.5], (1)
      "counts" : [8, 17, 8, 7, 6] (2)
   }
}

POST /metrics_index/_search?size=0
{
  "aggs": {
    "avg_latency":
      { "avg": { "field": "latency_histo" }
    }
  }
}
```

For each histogram field the `avg` aggregation adds each number in the `values` array <1> multiplied by its associated count in the `counts` array <2>. Eventually, it will compute the average over those values for all histograms and return the following result:

对于每个直方图字段，`avg`聚合将`values`数组 <1>中的每个数字乘以`counts`数组 <2>中的关联计数。最终，它将计算所有直方图的这些值的平均值并返回以下结果：

```console-result
{
  ...
  "aggregations": {
    "avg_latency": {
      "value": 0.29690721649
    }
  }
}
```

#  Boxplot aggregation

A `boxplot` metrics aggregation that computes boxplot of numeric values extracted from the aggregated documents. These values can be generated from specific numeric or [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html) in the documents.

一种`boxplot`度量聚合，用于计算从聚合文档中提取的数值的箱线图。这些值可以从文档中的特定数字或[直方图字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)生成。

The `boxplot` aggregation returns essential information for making a [box plot](https://en.wikipedia.org/wiki/Box_plot): minimum, maximum, median, first quartile (25th percentile) and third quartile (75th percentile) values.

该`boxplot`用于制造聚集回报的基本信息[箱线图](https://en.wikipedia.org/wiki/Box_plot)的最小值，最大值，中位数，第一个四分位数（第25个百分点）和第三个四分位数（第75百分位）值。

###  Syntax

A `boxplot` aggregation looks like this in isolation:

单独的`boxplot`聚合看起来像这样：

```js
{
  "boxplot": {
    "field": "load_time"
  }
}
```

Let’s look at a boxplot representing load time:

让我们看一个表示加载时间的箱线图：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_boxplot": {
      "boxplot": {
        "field": "load_time" (1)
      }
    }
  }
}
```

1.  The field `load_time` must be a numeric field

    该字段`load_time`必须是数字字段

The response will look like this:

响应将如下所示：

```console-result
{
  ...

 "aggregations": {
    "load_time_boxplot": {
      "min": 0.0,
      "max": 990.0,
      "q1": 165.0,
      "q2": 445.0,
      "q3": 725.0,
      "lower": 0.0,
      "upper": 990.0
    }
  }
}
```

In this case, the lower and upper whisker values are equal to the min and max. In general, these values are the 1.5 * IQR range, which is to say the nearest values to `q1 - (1.5 * IQR)` and `q3 + (1.5 * IQR)`. Since this is an approximation, the given values may not actually be observed values from the data, but should be within a reasonable error bound of them. While the Boxplot aggregation doesn’t directly return outlier points, you can check if `lower > min` or `upper < max` to see if outliers exist on either side, and then query for them directly.

在这种情况下，下胡须值和上胡须值等于最小值和最大值。通常，这些值是 1.5 * IQR 范围，也就是说最接近`q1 - (1.5 * IQR)`和 的值`q3 + (1.5 * IQR)`。由于这是一个近似值，给定的值实际上可能不是从数据中观察到的值，但应该在它们的合理误差范围内。虽然 Boxplot 聚合不直接返回异常点，但您可以检查`lower > min`或`upper < max`查看任一侧是否存在异常值，然后直接查询它们。

###  Script

If you need to create a boxplot for values that aren’t indexed exactly you should create a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) and get the boxplot of that. For example, if your load times are in milliseconds but you want values calculated in seconds, use a runtime field to convert them:

如果您需要为未完全索引的值创建箱线图，您应该创建一个[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)并获取它的箱线图。例如，如果您的加载时间以毫秒为单位，但您希望以秒为单位计算值，请使用运行时字段来转换它们：

```console
GET latency/_search
{
  "size": 0,
  "runtime_mappings": {
    "load_time.seconds": {
      "type": "long",
      "script": {
        "source": "emit(doc['load_time'].value / params.timeUnit)",
        "params": {
          "timeUnit": 1000
        }
      }
    }
  },
  "aggs": {
    "load_time_boxplot": {
      "boxplot": { "field": "load_time.seconds" }
    }
  }
}
```

### Boxplot values are (usually) approximate

The algorithm used by the `boxplot` metric is called TDigest (introduced by Ted Dunning in [Computing Accurate Quantiles using T-Digests](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf)).

该`boxplot`指标使用的算法称为 TDigest（由 Ted Dunning 在[Computing Accurate Quantiles using T-](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf) Digests 中引入 ）。

> WARNING: Boxplot as other percentile aggregations are also [non-deterministic](https://en.wikipedia.org/wiki/Nondeterministic_algorithm). This means you can get slightly different results using the same data.
>
> Boxplot 作为其他百分位聚合也是 [非确定性的](https://en.wikipedia.org/wiki/Nondeterministic_algorithm)。这意味着您可以使用相同的数据获得略有不同的结果。

###  Compression

Approximate algorithms must balance memory utilization with estimation accuracy. This balance can be controlled using a `compression` parameter:

近似算法必须平衡内存利用率和估计精度。可以使用`compression`参数控制此平衡：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_boxplot": {
      "boxplot": {
        "field": "load_time",
        "compression": 200    (1)
      }
    }
  }
}
```

1.  Compression controls memory usage and approximation error

   压缩控制内存使用和近似误差

The TDigest algorithm uses a number of "nodes" to approximate percentiles — the more nodes available, the higher the accuracy (and large memory footprint) proportional to the volume of data. The `compression` parameter limits the maximum number of nodes to `20 * compression`.

TDigest 算法使用多个“节点”来近似百分位数——可用节点越多，与数据量成正比的准确性（和大内存占用）就越高。该`compression`参数将最大节点数限制为`20 * compression`。

Therefore, by increasing the compression value, you can increase the accuracy of your percentiles at the cost of more memory. Larger compression values also make the algorithm slower since the underlying tree data structure grows in size, resulting in more expensive operations. The default compression value is `100`.

因此，通过增加压缩值，您可以以更多内存为代价来提高百分位数的准确性。较大的压缩值也会使算法变慢，因为底层树数据结构的大小会增加，从而导致操作成本更高。默认压缩值为 `100`.

A "node" uses roughly 32 bytes of memory, so under worst-case scenarios (large amount of data which arrives sorted and in-order) the default settings will produce a TDigest roughly 64KB in size. In practice data tends to be more random and the TDigest will use less memory.

“节点”使用大约 32 字节的内存，因此在最坏的情况下（大量数据按顺序到达），默认设置将产生大约 64KB 大小的 TDigest。在实践中，数据往往更随机，TDigest 将使用更少的内存。

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "grade_boxplot": {
      "boxplot": {
        "field": "grade",
        "missing": 10     (1)
      }
    }
  }
}
```

1.  Documents without a value in the `grade` field will fall into the same bucket as documents that have the value `10`.

   该`grade`字段中没有值的文档将与具有该值的文档落入同一个桶中`10`。

# Cardinality aggregation

A `single-value` metrics aggregation that calculates an approximate count of distinct values.

`single-value`度量聚集，计算不同的值的近似计数。

Assume you are indexing store sales and would like to count the unique number of sold products that match a query:

假设您正在索引商店销售额并希望计算与查询匹配的已售产品的唯一数量：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "type_count": {
      "cardinality": {
        "field": "type"
      }
    }
  }
}
```

Response:

```console-result
{
  ...
  "aggregations": {
    "type_count": {
      "value": 3
    }
  }
}
```

###  Precision control

This aggregation also supports the `precision_threshold` option:

此聚合还支持以下`precision_threshold`选项：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "type_count": {
      "cardinality": {
        "field": "type",
        "precision_threshold": 100 (1)
      }
    }
  }
}
```

1. The `precision_threshold` options allows to trade memory for accuracy, and defines a unique count below which counts are expected to be close to accurate. Above this value, counts might become a bit more fuzzy. The maximum supported value is 40000, thresholds above this number will have the same effect as a threshold of 40000. The default value is `3000`.

   这些`precision_threshold`选项允许用内存来换取准确性，并定义一个唯一计数，低于该计数预计将接近准确。高于此值，计数可能会变得更加模糊。支持的最大值为 40000，高于此数字的阈值将与阈值 40000 具有相同的效果。默认值为`3000`。

###  Counts are approximate

Computing exact counts requires loading values into a hash set and returning its size. This doesn’t scale when working on high-cardinality sets and/or large values as the required memory usage and the need to communicate those per-shard sets between nodes would utilize too many resources of the cluster.

计算精确计数需要将值加载到散列集并返回其大小。在处理高基数集和/或大值时，这不会扩展，因为所需的内存使用量以及在节点之间通信这些每个分片集的需要会使用集群的太多资源。

This `cardinality` aggregation is based on the [HyperLogLog++](https://static.googleusercontent.com/media/research.google.com/fr//pubs/archive/40671.pdf) algorithm, which counts based on the hashes of the values with some interesting properties:

此`cardinality`聚合基于 [HyperLogLog++](https://static.googleusercontent.com/media/research.google.com/fr//pubs/archive/40671.pdf) 算法，该算法基于具有一些有趣属性的值的哈希值进行计数：

- configurable precision, which decides on how to trade memory for accuracy,

  可配置的精度，决定如何用内存来换取精度，

- excellent accuracy on low-cardinality sets,

  在低基数集上具有出色的准确性，

- fixed memory usage: no matter if there are tens or billions of unique values, memory usage only depends on the configured precision.

  固定内存使用：无论是数百还是数十亿个唯一值，内存使用仅取决于配置的精度。

For a precision threshold of `c`, the implementation that we are using requires about `c * 8` bytes.

对于精度阈值`c`，我们正在使用的实现需要大约`c * 8`字节。

The following chart shows how the error varies before and after the threshold:

下图显示了阈值前后误差的变化情况：

![cardinality error](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/cardinality_error.png)

For all 3 thresholds, counts have been accurate up to the configured threshold. Although not guaranteed, this is likely to be the case. Accuracy in practice depends on the dataset in question. In general, most datasets show consistently good accuracy. Also note that even with a threshold as low as 100, the error remains very low (1-6% as seen in the above graph) even when counting millions of items.

对于所有 3 个阈值，计数一直准确到配置的阈值。虽然不能保证，但很可能就是这种情况。实践中的准确性取决于所讨论的数据集。一般来说，大多数数据集始终显示出良好的准确性。另请注意，即使阈值低至 100，即使在计算数百万个项目时，错误仍然非常低（如上图所示为 1-6%）。

The HyperLogLog++ algorithm depends on the leading zeros of hashed values, the exact distributions of hashes in a dataset can affect the accuracy of the cardinality.

HyperLogLog++ 算法取决于散列值的前导零，数据集中散列的确切分布会影响基数的准确性。

Please also note that even with a threshold as low as 100, the error remains very low, even when counting millions of items.

还请注意，即使阈值低至 100，错误仍然非常低，即使在计算数百万个项目时也是如此。

###  Pre-computed hashes

On string fields that have a high cardinality, it might be faster to store the hash of your field values in your index and then run the cardinality aggregation on this field. This can either be done by providing hash values from client-side or by letting Elasticsearch compute hash values for you by using the [`mapper-murmur3`](https://www.elastic.co/guide/en/elasticsearch/plugins/master/mapper-murmur3.html) plugin.

在具有高基数的字符串字段上，将字段值的散列存储在索引中然后在该字段上运行基数聚合可能会更快。这可以通过从客户端提供哈希值或让 Elasticsearch 使用[`mapper-murmur3`](https://www.elastic.co/guide/en/elasticsearch/plugins/master/mapper-murmur3.html)插件为您计算哈希值来完成 。

> NOTE: Pre-computing hashes is usually only useful on very large and/or high-cardinality fields as it saves CPU and memory. However, on numeric fields, hashing is very fast and storing the original values requires as much or less memory than storing the hashes. This is also true on low-cardinality string fields, especially given that those have an optimization in order to make sure that hashes are computed at most once per unique value per segment.
>
> 预计算散列通常只对非常大和/或高基数的字段有用，因为它可以节省 CPU 和内存。但是，在数字字段上，散列非常快，并且存储原始值所需的内存与存储散列所需的内存一样多或少。对于低基数字符串字段也是如此，特别是考虑到那些具有优化以确保每个段的每个唯一值最多计算一次哈希值的情况。

###  Script

If you need the cardinality of the combination of two fields, create a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) combining them and aggregate it.

如果您需要两个字段组合的基数，请创建一个组合它们的[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)并将其聚合。

```console
POST /sales/_search?size=0
{
  "runtime_mappings": {
    "type_and_promoted": {
      "type": "keyword",
      "script": "emit(doc['type'].value + ' ' + doc['promoted'].value)"
    }
  },
  "aggs": {
    "type_promoted_count": {
      "cardinality": {
        "field": "type_and_promoted"
      }
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
POST /sales/_search?size=0
{
  "aggs": {
    "tag_cardinality": {
      "cardinality": {
        "field": "tag",
        "missing": "N/A" (1)
      }
    }
  }
}
```

1. Documents without a value in the `tag` field will fall into the same bucket as documents that have the value `N/A`.

   该`tag`字段中没有值的文档将与具有该值的文档落入同一个桶中`N/A`。

#  Extended stats aggregation

A `multi-value` metrics aggregation that computes stats over numeric values extracted from the aggregated documents.

一种`multi-value`度量聚合，它计算从聚合文档中提取的数值的统计信息。

The `extended_stats` aggregations is an extended version of the [`stats`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-stats-aggregation.html) aggregation, where additional metrics are added such as `sum_of_squares`, `variance`, `std_deviation` and `std_deviation_bounds`.

所述`extended_stats`聚合是的扩展版本[`stats`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-stats-aggregation.html)聚集，其中附加的度量被加到如`sum_of_squares`，`variance`，`std_deviation`和`std_deviation_bounds`。

Assuming the data consists of documents representing exams grades (between 0 and 100) of students

假设数据包含代表学生考试成绩（0 到 100 之间）的文件

```console
GET /exams/_search
{
  "size": 0,
  "aggs": {
    "grades_stats": { "extended_stats": { "field": "grade" } }
  }
}
```

The above aggregation computes the grades statistics over all documents. The aggregation type is `extended_stats` and the `field` setting defines the numeric field of the documents the stats will be computed on. The above will return the following:

上述聚合计算所有文档的成绩统计。聚合类型是`extended_stats`，并且`field`设置定义了将在其上计算统计信息的文档的数字字段。以上将返回以下内容：

The `std_deviation` and `variance` are calculated as population metrics so they are always the same as `std_deviation_population` and `variance_population` respectively.

在`std_deviation`和`variance`所以他们总是一样的计算公式为人口指标`std_deviation_population`和`variance_population`分别。

```console-result
{
  ...

  "aggregations": {
    "grades_stats": {
      "count": 2,
      "min": 50.0,
      "max": 100.0,
      "avg": 75.0,
      "sum": 150.0,
      "sum_of_squares": 12500.0,
      "variance": 625.0,
      "variance_population": 625.0,
      "variance_sampling": 1250.0,
      "std_deviation": 25.0,
      "std_deviation_population": 25.0,
      "std_deviation_sampling": 35.35533905932738,
      "std_deviation_bounds": {
        "upper": 125.0,
        "lower": 25.0,
        "upper_population": 125.0,
        "lower_population": 25.0,
        "upper_sampling": 145.71067811865476,
        "lower_sampling": 4.289321881345245
      }
    }
  }
}
```

The name of the aggregation (`grades_stats` above) also serves as the key by which the aggregation result can be retrieved from the returned response.

聚合的名称（`grades_stats`如上）也用作可以从返回的响应中检索聚合结果的键。

###  Standard Deviation Bounds

By default, the `extended_stats` metric will return an object called `std_deviation_bounds`, which provides an interval of plus/minus two standard deviations from the mean. This can be a useful way to visualize variance of your data. If you want a different boundary, for example three standard deviations, you can set `sigma` in the request:

默认情况下，该`extended_stats`指标将返回一个名为`std_deviation_bounds` 的对象，该对象提供与平均值正负两个标准差的区间。这可能是一种可视化数据方差的有用方法。如果您想要不同的边界，例如三个标准偏差，您可以`sigma`在请求中设置：

```console
GET /exams/_search
{
  "size": 0,
  "aggs": {
    "grades_stats": {
      "extended_stats": {
        "field": "grade",
        "sigma": 3          (1)
      }
    }
  }
}
```

1. `sigma` controls how many standard deviations +/- from the mean should be displayed

   `sigma` 控制应该显示多少标准偏差 +/-

`sigma` can be any non-negative double, meaning you can request non-integer values such as `1.5`. A value of `0` is valid, but will simply return the average for both `upper` and `lower` bounds.

`sigma`可以是任何非负双精度值，这意味着您可以请求非整数值，例如`1.5`. 的值`0`是有效的，但只会返回`upper`和`lower`边界的平均值。

The `upper` and `lower` bounds are calculated as population metrics so they are always the same as `upper_population` and `lower_population` respectively.

在`upper`和`lower`所以他们总是一样的界限被计算为人口指标`upper_population`和 `lower_population`分别。

> NOTE:  Standard Deviation and Bounds require normality
>
> The standard deviation and its bounds are displayed by default, but they are not always applicable to all data-sets. Your data must be normally distributed for the metrics to make sense. The statistics behind standard deviations assumes normally distributed data, so if your data is skewed heavily left or right, the value returned will be misleading.
>
> ### 标准偏差和界限需要正态性
>
> 默认情况下显示标准偏差及其界限，但它们并不总是适用于所有数据集。您的数据必须正态分布才能使指标有意义。标准差背后的统计数据假设数据呈正态分布，因此如果您的数据严重向左或向右倾斜，则返回的值将具有误导性。

###  Script

If you need to aggregate on a value that isn’t indexed, use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html). Say the we found out that the grades we’ve been working on were for an exam that was above the level of the students and we want to "correct" it:

如果您需要对未编入索引的值进行聚合，请使用[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)。假设我们发现我们一直在努力的成绩是针对高于学生水平的考试，我们想“纠正”它：

```console
GET /exams/_search
{
  "size": 0,
  "runtime_mappings": {
    "grade.corrected": {
      "type": "double",
      "script": {
        "source": "emit(Math.min(100, doc['grade'].value * params.correction))",
        "params": {
          "correction": 1.2
        }
      }
    }
  },
  "aggs": {
    "grades_stats": {
      "extended_stats": { "field": "grade.corrected" }
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
GET /exams/_search
{
  "size": 0,
  "aggs": {
    "grades_stats": {
      "extended_stats": {
        "field": "grade",
        "missing": 0        (1)
      }
    }
  }
}
```

1.  Documents without a value in the `grade` field will fall into the same bucket as documents that have the value `0`.

    该`grade`字段中没有值的文档将与具有该值的文档落入同一个桶中`0`。

#  Geo-bounds aggregation

A metric aggregation that computes the bounding box containing all geo values for a field.

计算包含字段的所有地理值的边界框的度量聚合。

Example:

```console
PUT /museums
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

POST /museums/_bulk?refresh
{"index":{"_id":1}}
{"location": "52.374081,4.912350", "name": "NEMO Science Museum"}
{"index":{"_id":2}}
{"location": "52.369219,4.901618", "name": "Museum Het Rembrandthuis"}
{"index":{"_id":3}}
{"location": "52.371667,4.914722", "name": "Nederlands Scheepvaartmuseum"}
{"index":{"_id":4}}
{"location": "51.222900,4.405200", "name": "Letterenhuis"}
{"index":{"_id":5}}
{"location": "48.861111,2.336389", "name": "Musée du Louvre"}
{"index":{"_id":6}}
{"location": "48.860000,2.327000", "name": "Musée d'Orsay"}

POST /museums/_search?size=0
{
  "query": {
    "match": { "name": "musée" }
  },
  "aggs": {
    "viewport": {
      "geo_bounds": {
        "field": "location",    (1)
        "wrap_longitude": true  (2)
      }
    }
  }
}
```

1. The `geo_bounds` aggregation specifies the field to use to obtain the bounds.

   该`geo_bounds`聚集指定要使用获得的界限领域。

2. `wrap_longitude` is an optional parameter which specifies whether the bounding box should be allowed to overlap the international date line. The default value is `true`.

   `wrap_longitude`是一个可选参数，它指定是否允许边界框与国际日期变更线重叠。默认值为`true`。

The above aggregation demonstrates how one would compute the bounding box of the location field for all documents with a business type of shop.

上面的聚合演示了如何为具有商店业务类型的所有文档计算位置字段的边界框。

The response for the above aggregation:

```console-result
{
  ...
  "aggregations": {
    "viewport": {
      "bounds": {
        "top_left": {
          "lat": 48.86111099738628,
          "lon": 2.3269999679178
        },
        "bottom_right": {
          "lat": 48.85999997612089,
          "lon": 2.3363889567553997
        }
      }
    }
  }
}
```

#### Geo Bounds Aggregation on `geo_shape` fields

The Geo Bounds Aggregation is also supported on `geo_shape` fields.

`geo_shape`字段也支持地理边界聚合。

If [`wrap_longitude`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-geobounds-aggregation.html#geo-bounds-wrap-longitude) is set to `true` (the default), the bounding box can overlap the international date line and return a bounds where the `top_left` longitude is larger than the `top_right` longitude.

如果[`wrap_longitude`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-geobounds-aggregation.html#geo-bounds-wrap-longitude)设置为`true` （默认），则边界框可以与国际日期变更线重叠并返回`top_left`经度大于经度的边界`top_right` 。

For example, the upper right longitude will typically be greater than the lower left longitude of a geographic bounding box. However, when the area crosses the 180° meridian, the value of the lower left longitude will be greater than the value of the upper right longitude. See [Geographic bounding box](http://docs.opengeospatial.org/is/12-063r5/12-063r5.html#30) on the Open Geospatial Consortium website for more information.

例如，地理边界框的右上经度通常大于左下经度。但是，当该区域穿过 180° 子午线时，左下经度的值将大于右上经度的值。有关详细信息，请参阅 开放地理空间联盟网站上的[地理边界框](http://docs.opengeospatial.org/is/12-063r5/12-063r5.html#30)。

Example:

```console
PUT /places
{
  "mappings": {
    "properties": {
      "geometry": {
        "type": "geo_shape"
      }
    }
  }
}

POST /places/_bulk?refresh
{"index":{"_id":1}}
{"name": "NEMO Science Museum", "geometry": "POINT(4.912350 52.374081)" }
{"index":{"_id":2}}
{"name": "Sportpark De Weeren", "geometry": { "type": "Polygon", "coordinates": [ [ [ 4.965305328369141, 52.39347642069457 ], [ 4.966979026794433, 52.391721758934835 ], [ 4.969425201416015, 52.39238958618537 ], [ 4.967944622039794, 52.39420969150824 ], [ 4.965305328369141, 52.39347642069457 ] ] ] } }

POST /places/_search?size=0
{
  "aggs": {
    "viewport": {
      "geo_bounds": {
        "field": "geometry"
      }
    }
  }
}
```

```console-result
{
  ...
  "aggregations": {
    "viewport": {
      "bounds": {
        "top_left": {
          "lat": 52.39420966710895,
          "lon": 4.912349972873926
        },
        "bottom_right": {
          "lat": 52.374080987647176,
          "lon": 4.969425117596984
        }
      }
    }
  }
}
```

#  Geo-centroid aggregation

A metric aggregation that computes the weighted [centroid](https://en.wikipedia.org/wiki/Centroid) from all coordinate values for geo fields.

从地理字段的所有坐标值计算加权[质心的](https://en.wikipedia.org/wiki/Centroid)度量聚合。

Example:

```console
PUT /museums
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

POST /museums/_bulk?refresh
{"index":{"_id":1}}
{"location": "52.374081,4.912350", "city": "Amsterdam", "name": "NEMO Science Museum"}
{"index":{"_id":2}}
{"location": "52.369219,4.901618", "city": "Amsterdam", "name": "Museum Het Rembrandthuis"}
{"index":{"_id":3}}
{"location": "52.371667,4.914722", "city": "Amsterdam", "name": "Nederlands Scheepvaartmuseum"}
{"index":{"_id":4}}
{"location": "51.222900,4.405200", "city": "Antwerp", "name": "Letterenhuis"}
{"index":{"_id":5}}
{"location": "48.861111,2.336389", "city": "Paris", "name": "Musée du Louvre"}
{"index":{"_id":6}}
{"location": "48.860000,2.327000", "city": "Paris", "name": "Musée d'Orsay"}

POST /museums/_search?size=0
{
  "aggs": {
    "centroid": {
      "geo_centroid": {
        "field": "location" (1)
      }
    }
  }
}
```

1. The `geo_centroid` aggregation specifies the field to use for computing the centroid. (NOTE: field must be a [Geo-point](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) type)

    该`geo_centroid`集合指定现场使用用于计算质心。（注意：字段必须是[地理点](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)类型）

The above aggregation demonstrates how one would compute the centroid of the location field for all documents with a crime type of burglary.

上面的聚合演示了如何计算所有犯罪类型为入室盗窃的文档的位置字段的质心。

The response for the above aggregation:

```console-result
{
  ...
  "aggregations": {
    "centroid": {
      "location": {
        "lat": 51.00982965203002,
        "lon": 3.9662131341174245
      },
      "count": 6
    }
  }
}
```

The `geo_centroid` aggregation is more interesting when combined as a sub-aggregation to other bucket aggregations.

`geo_centroid`当组合为其他桶聚合的子聚合时，聚合会更有趣。

Example:

```console
POST /museums/_search?size=0
{
  "aggs": {
    "cities": {
      "terms": { "field": "city.keyword" },
      "aggs": {
        "centroid": {
          "geo_centroid": { "field": "location" }
        }
      }
    }
  }
}
```

The above example uses `geo_centroid` as a sub-aggregation to a [terms](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) bucket aggregation for finding the central location for museums in each city.

上面的示例`geo_centroid`用作[术语](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html)桶聚合的子聚合， 用于查找每个城市博物馆的中心位置。

The response for the above aggregation:

```console-result
{
  ...
  "aggregations": {
    "cities": {
      "sum_other_doc_count": 0,
      "doc_count_error_upper_bound": 0,
      "buckets": [
        {
          "key": "Amsterdam",
          "doc_count": 3,
          "centroid": {
            "location": {
              "lat": 52.371655656024814,
              "lon": 4.909563297405839
            },
            "count": 3
          }
        },
        {
          "key": "Paris",
          "doc_count": 2,
          "centroid": {
            "location": {
              "lat": 48.86055548675358,
              "lon": 2.3316944623366
            },
            "count": 2
          }
        },
        {
          "key": "Antwerp",
          "doc_count": 1,
          "centroid": {
            "location": {
              "lat": 51.22289997059852,
              "lon": 4.40519998781383
            },
            "count": 1
          }
        }
      ]
    }
  }
}
```

####  Geo Centroid Aggregation on `geo_shape` fields

The centroid metric for geo-shapes is more nuanced than for points. The centroid of a specific aggregation bucket containing shapes is the centroid of the highest-dimensionality shape type in the bucket. For example, if a bucket contains shapes comprising of polygons and lines, then the lines do not contribute to the centroid metric. Each type of shape’s centroid is calculated differently. Envelopes and circles ingested via the [Circle](https://www.elastic.co/guide/en/elasticsearch/reference/master/ingest-circle-processor.html) are treated as polygons.

地理形状的质心度量比点更微妙。包含形状的特定聚合桶的质心是该桶中维数最高的形状类型的质心。例如，如果一个桶包含由多边形和线组成的形状，那么这些线对质心度量没有贡献。每种形状的质心计算方式不同。通过[Circle](https://www.elastic.co/guide/en/elasticsearch/reference/master/ingest-circle-processor.html)摄取的信封和圆圈被视为多边形。

| Geometry Type      | Centroid Calculation                                         |
| ------------------ | ------------------------------------------------------------ |
| [Multi]Point       | equally weighted average of all the coordinates<br />所有坐标的加权平均值 |
| [Multi]LineString  | a weighted average of all the centroids of each segment, where the weight of each segment is its length in degrees<br />每个段的所有质心的加权平均值，其中每个段的权重是它的长度（以度为单位） |
| [Multi]Polygon     | a weighted average of all the centroids of all the triangles of a polygon where the triangles are formed by every two consecutive vertices and the starting-point. holes have negative weights. weights represent the area of the triangle in deg^2 calculated<br />多边形所有三角形的所有质心的加权平均值，其中三角形由每两个连续顶点和起点形成。孔具有负权重。权重表示计算出的 deg^2 中三角形的面积 |
| GeometryCollection | The centroid of all the underlying geometries with the highest dimension. If Polygons and Lines and/or Points, then lines and/or points are ignored. If Lines and Points, then points are ignored |

具有最高维度的所有基础几何的质心。如果是多边形和线和/或点，则忽略线和/或点。如果是 Lines 和 Points，则忽略点Example:

```console
PUT /places
{
  "mappings": {
    "properties": {
      "geometry": {
        "type": "geo_shape"
      }
    }
  }
}

POST /places/_bulk?refresh
{"index":{"_id":1}}
{"name": "NEMO Science Museum", "geometry": "POINT(4.912350 52.374081)" }
{"index":{"_id":2}}
{"name": "Sportpark De Weeren", "geometry": { "type": "Polygon", "coordinates": [ [ [ 4.965305328369141, 52.39347642069457 ], [ 4.966979026794433, 52.391721758934835 ], [ 4.969425201416015, 52.39238958618537 ], [ 4.967944622039794, 52.39420969150824 ], [ 4.965305328369141, 52.39347642069457 ] ] ] } }

POST /places/_search?size=0
{
  "aggs": {
    "centroid": {
      "geo_centroid": {
        "field": "geometry"
      }
    }
  }
}
```

```console-result
{
  ...
  "aggregations": {
    "centroid": {
      "location": {
        "lat": 52.39296147599816,
        "lon": 4.967404240742326
      },
      "count": 2
    }
  }
}
```

> WARNING:  Using `geo_centroid` as a sub-aggregation of `geohash_grid`
>
> The [`geohash_grid`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-geohashgrid-aggregation.html) aggregation places documents, not individual geo-points, into buckets. If a document’s `geo_point` field contains [multiple values](https://www.elastic.co/guide/en/elasticsearch/reference/master/array.html), the document could be assigned to multiple buckets, even if one or more of its geo-points are outside the bucket boundaries.
>
> If a `geocentroid` sub-aggregation is also used, each centroid is calculated using all geo-points in a bucket, including those outside the bucket boundaries. This can result in centroids outside of bucket boundaries.
>
> ###  使用`geo_centroid`作为一个子聚集`geohash_grid`
>
> 该[`geohash_grid`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-geohashgrid-aggregation.html) 聚集的地方文件，而不是单个地理点，成桶。如果文档的`geo_point`字段包含[多个值](https://www.elastic.co/guide/en/elasticsearch/reference/master/array.html)，则可以将文档分配给多个存储桶，即使其一个或多个地理点在存储桶边界之外。
>
> 如果`geocentroid`还使用子聚合，则使用桶中的所有地理点计算每个质心，包括桶边界之外的地理点。这可能导致质心超出桶边界。

# Geo-Line Aggregation

The `geo_line` aggregation aggregates all `geo_point` values within a bucket into a LineString ordered by the chosen `sort` field. This `sort` can be a date field, for example. The bucket returned is a valid [GeoJSON Feature](https://tools.ietf.org/html/rfc7946#section-3.2) representing the line geometry.

`geo_line`聚合聚合所有`geo_point`的铲斗内的值成线段形式有序由所选择的`sort`场。`sort`例如，这可以是日期字段。返回的存储桶是表示线几何的有效 [GeoJSON 特征](https://tools.ietf.org/html/rfc7946#section-3.2)。

```console
PUT test
{
    "mappings": {
        "dynamic": "strict",
        "_source": {
            "enabled": false
        },
        "properties": {
            "my_location": {
                "type": "geo_point"
            },
            "group": {
                "type": "keyword"
            },
            "@timestamp": {
                "type": "date"
            }
        }
    }
}

POST /test/_bulk?refresh
{"index": {}}
{"my_location": {"lat":37.3450570, "lon": -122.0499820}, "@timestamp": "2013-09-06T16:00:36"}
{"index": {}}
{"my_location": {"lat": 37.3451320, "lon": -122.0499820}, "@timestamp": "2013-09-06T16:00:37Z"}
{"index": {}}
{"my_location": {"lat": 37.349283, "lon": -122.0505010}, "@timestamp": "2013-09-06T16:00:37Z"}

POST /test/_search?filter_path=aggregations
{
  "aggs": {
    "line": {
      "geo_line": {
        "point": {"field": "my_location"},
        "sort": {"field": "@timestamp"}
      }
    }
  }
}
```

Which returns:

```js
{
  "aggregations": {
    "line": {
      "type" : "Feature",
      "geometry" : {
        "type" : "LineString",
        "coordinates" : [
          [
            -122.049982,
            37.345057
          ],
          [
            -122.050501,
            37.349283
          ],
          [
            -122.049982,
            37.345132
          ]
        ]
      },
      "properties" : {
        "complete" : true
      }
    }
  }
}
```

### Options

- **`point`**

  (Required)

This option specifies the name of the `geo_point` field

此选项指定`geo_point`字段的名称

Example usage configuring `my_location` as the point field:

配置`my_location`为点字段的示例用法：

```js
"point": {
  "field": "my_location"
}
```

- **`sort`**

  (Required)

This option specifies the name of the numeric field to use as the sort key for ordering the points

此选项指定用作排序点的排序键的数字字段的名称

Example usage configuring `@timestamp` as the sort key:

配置`@timestamp`为排序键的示例用法：

```js
"point": {
  "field": "@timestamp"
}
```

- **`include_sort`**

  (Optional, boolean, default: `false`)

This option includes, when true, an additional array of the sort values in the feature properties.

当为真时，此选项包括要素属性中排序值的附加数组。

- **`sort_order`**

  (Optional, string, default: `"ASC"`)

This option accepts one of two values: "ASC", "DESC".

此选项接受以下两个值之一：“ASC”、“DESC”。

The line is sorted in ascending order by the sort key when set to "ASC", and in descending with "DESC".

当设置为“ASC”时，该行按排序键升序排序，并按“DESC”降序排序。

- **`size`**

  (Optional, integer, default: `10000`)

The maximum length of the line represented in the aggregation. Valid sizes are between one and 10000.

聚合中表示的行的最大长度。有效大小介于 1 到 10000 之间。

#  Matrix stats aggregation

The `matrix_stats` aggregation is a numeric aggregation that computes the following statistics over a set of document fields:

该`matrix_stats`聚合是指将计算在一组文档领域的下列统计数据的数字汇总：

| `count`       | Number of per field samples included in the calculation.<br />计算中包含的每场样本数。 |
| ------------- | ------------------------------------------------------------ |
| `mean`        | The average value for each field.<br />每个字段的平均值。    |
| `variance`    | Per field Measurement for how spread out the samples are from the mean.<br />每场测量样本与平均值的分布情况。 |
| `skewness`    | Per field measurement quantifying the asymmetric distribution around the mean.<br />每场测量量化围绕平均值的不对称分布。 |
| `kurtosis`    | Per field measurement quantifying the shape of the distribution.<br />每场测量量化分布的形状。 |
| `covariance`  | A matrix that quantitatively describes how changes in one field are associated with another.<br />定量描述一个领域的变化如何与另一个相关联的矩阵。 |
| `correlation` | The covariance matrix scaled to a range of -1 to 1, inclusive. Describes the relationship between field distributions.<br />协方差矩阵缩放到 -1 到 1（含）的范围。描述字段分布之间的关系。 |

> IMPORTANT: Unlike other metric aggregations, the `matrix_stats` aggregation does not support scripting.
>
> 与其他指标聚合不同，该`matrix_stats`聚合不支持脚本。

The following example demonstrates the use of matrix stats to describe the relationship between income and poverty.

下面的例子演示了使用矩阵统计来描述收入和贫困之间的关系。

```console
GET /_search
{
  "aggs": {
    "statistics": {
      "matrix_stats": {
        "fields": [ "poverty", "income" ]
      }
    }
  }
}
```

The aggregation type is `matrix_stats` and the `fields` setting defines the set of fields (as an array) for computing the statistics. The above request returns the following response:

聚合类型是`matrix_stats`并且`fields`设置定义了用于计算统计信息的字段集（作为数组）。上述请求返回以下响应：

```console-result
{
  ...
  "aggregations": {
    "statistics": {
      "doc_count": 50,
      "fields": [ {
          "name": "income",
          "count": 50,
          "mean": 51985.1,
          "variance": 7.383377037755103E7,
          "skewness": 0.5595114003506483,
          "kurtosis": 2.5692365287787124,
          "covariance": {
            "income": 7.383377037755103E7,
            "poverty": -21093.65836734694
          },
          "correlation": {
            "income": 1.0,
            "poverty": -0.8352655256272504
          }
        }, {
          "name": "poverty",
          "count": 50,
          "mean": 12.732000000000001,
          "variance": 8.637730612244896,
          "skewness": 0.4516049811903419,
          "kurtosis": 2.8615929677997767,
          "covariance": {
            "income": -21093.65836734694,
            "poverty": 8.637730612244896
          },
          "correlation": {
            "income": -0.8352655256272504,
            "poverty": 1.0
          }
        } ]
    }
  }
}
```

The `doc_count` field indicates the number of documents involved in the computation of the statistics.

该`doc_count`字段表示参与统计计算的文档数量。

###  Multi Value Fields

The `matrix_stats` aggregation treats each document field as an independent sample. The `mode` parameter controls what array value the aggregation will use for array or multi-valued fields. This parameter can take one of the following:

`matrix_stats`聚合将每个文档字段作为一个独立的样品。该`mode`参数控制聚合将用于数组或多值字段的数组值。此参数可以采用以下其中一项：

| `avg`    | (default) Use the average of all values.<br />（默认）使用所有值的平均值。 |
| -------- | ------------------------------------------------------------ |
| `min`    | Pick the lowest value.<br />选择最低值。                     |
| `max`    | Pick the highest value.<br /> 选择最高值。                   |
| `sum`    | Use the sum of all values.<br />使用所有值的总和。           |
| `median` | Use the median of all values.<br />使用所有值的中位数。      |

###  Missing Values

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value. This is done by adding a set of fieldname : value mappings to specify default values per field.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。这是通过添加一组 fieldname : value 映射来指定每个字段的默认值来完成的。

```console
GET /_search
{
  "aggs": {
    "matrixstats": {
      "matrix_stats": {
        "fields": [ "poverty", "income" ],
        "missing": { "income": 50000 }      (1)
      }
    }
  }
}
```

1. Documents without a value in the `income` field will have the default value `50000`.

   `income`字段中没有值的文档将具有默认值`50000`。

# Max aggregation

A `single-value` metrics aggregation that keeps track and returns the maximum value among the numeric values extracted from the aggregated documents.

一种`single-value`度量聚合，它跟踪并返回从聚合文档中提取的数值中的最大值。

> NOTE: The `min` and `max` aggregation operate on the `double` representation of the data. As a consequence, the result may be approximate when running on longs whose absolute value is greater than `2^53`.
>
> `min`和`max`在聚合操作`double`的数据的表示。因此，在绝对值大于 `2^53`的 long 上运行时，结果可能是近似的。

Computing the max price value across all documents

计算所有文档的最大价格值

```console
POST /sales/_search?size=0
{
  "aggs": {
    "max_price": { "max": { "field": "price" } }
  }
}
```

Response:

```console-result
{
  ...
  "aggregations": {
      "max_price": {
          "value": 200.0
      }
  }
}
```

As can be seen, the name of the aggregation (`max_price` above) also serves as the key by which the aggregation result can be retrieved from the returned response.

可以看出，聚合的名称（`max_price`如上）也作为可以从返回的响应中检索聚合结果的键。

###  Script

If you need to get the `max` of something more complex than a single field, run an aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html).

如果您需要获取`max`比单个字段更复杂的内容，请在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。

```console
POST /sales/_search
{
  "size": 0,
  "runtime_mappings": {
    "price.adjusted": {
      "type": "double",
      "script": """
        double price = doc['price'].value;
        if (doc['promoted'].value) {
          price *= 0.8;
        }
        emit(price);
      """
    }
  },
  "aggs": {
    "max_price": {
      "max": { "field": "price.adjusted" }
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
POST /sales/_search
{
  "aggs" : {
      "grade_max" : {
          "max" : {
              "field" : "grade",
              "missing": 10       (1)
          }
      }
  }
}
```

1. Documents without a value in the `grade` field will fall into the same bucket as documents that have the value `10`.

    该`grade`字段中没有值的文档将与具有该值的文档落入同一个桶中`10`。

###  Histogram fields

When `max` is computed on [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html), the result of the aggregation is the maximum of all elements in the `values` array. Note, that the `counts` array of the histogram is ignored.

当`max`在[histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)上计算时，聚合的结果是`values`数组中所有元素的最大值。请注意，`counts`直方图数组被忽略。

For example, for the following index that stores pre-aggregated histograms with latency metrics for different networks:

例如，对于以下存储具有不同网络延迟指标的预聚合直方图的索引：

```console
PUT metrics_index/_doc/1
{
  "network.name" : "net-1",
  "latency_histo" : {
      "values" : [0.1, 0.2, 0.3, 0.4, 0.5], 
      "counts" : [3, 7, 23, 12, 6] 
   }
}

PUT metrics_index/_doc/2
{
  "network.name" : "net-2",
  "latency_histo" : {
      "values" :  [0.1, 0.2, 0.3, 0.4, 0.5], 
      "counts" : [8, 17, 8, 7, 6] 
   }
}

POST /metrics_index/_search?size=0
{
  "aggs" : {
    "min_latency" : { "min" : { "field" : "latency_histo" } }
  }
}
```

The `max` aggregation will return the maximum value of all histogram fields:

该`max`聚合将返回所有直方图领域中的最大值：

```console-result
{
  ...
  "aggregations": {
    "min_latency": {
      "value": 0.5
    }
  }
}
```

# Median absolute deviation aggregation

This `single-value` aggregation approximates the [median absolute deviation](https://en.wikipedia.org/wiki/Median_absolute_deviation) of its search results.

此`single-value`聚合近似于 其搜索结果的[中值绝对偏差](https://en.wikipedia.org/wiki/Median_absolute_deviation)。

Median absolute deviation is a measure of variability. It is a robust statistic, meaning that it is useful for describing data that may have outliers, or may not be normally distributed. For such data it can be more descriptive than standard deviation.

中值绝对偏差是可变性的度量。这是一个稳健的统计量，这意味着它可用于描述可能具有异常值或可能不呈正态分布的数据。对于此类数据，它可以比标准偏差更具描述性。

It is calculated as the median of each data point’s deviation from the median of the entire sample. That is, for a random variable X, the median absolute deviation is median(|median(X) - Xi|).

它计算为每个数据点与整个样本中位数的偏差的中位数。也就是说，对于随机变量 X，中值绝对偏差为中值(|中值(X) - X i |)。

###  Example

Assume our data represents product reviews on a one to five star scale. Such reviews are usually summarized as a mean, which is easily understandable but doesn’t describe the reviews' variability. Estimating the median absolute deviation can provide insight into how much reviews vary from one another.

假设我们的数据代表一到五星级的产品评论。这样的评论通常被总结为一个平均值，这很容易理解，但没有描述评论的可变性。估计中值绝对偏差可以深入了解评论之间的差异。

In this example we have a product which has an average rating of 3 stars. Let’s look at its ratings' median absolute deviation to determine how much they vary

在这个例子中，我们有一个平均评分为 3 星的产品。让我们看看它的评级的中值绝对偏差，以确定它们的差异有多大

```console
GET reviews/_search
{
  "size": 0,
  "aggs": {
    "review_average": {
      "avg": {
        "field": "rating"
      }
    },
    "review_variability": {
      "median_absolute_deviation": {
        "field": "rating"  (1)
      }
    }
  }
}
```

1. `rating` must be a numeric field

   `rating` 必须是数字字段

The resulting median absolute deviation of `2` tells us that there is a fair amount of variability in the ratings. Reviewers must have diverse opinions about this product.

由此产生的中值绝对偏差`2`告诉我们，评级存在相当大的可变性。评论者必须对这个产品有不同的看法。

```console-result
{
  ...
  "aggregations": {
    "review_average": {
      "value": 3.0
    },
    "review_variability": {
      "value": 2.0
    }
  }
}
```

###  Approximation

The naive implementation of calculating median absolute deviation stores the entire sample in memory, so this aggregation instead calculates an approximation. It uses the [TDigest data structure](https://github.com/tdunning/t-digest) to approximate the sample median and the median of deviations from the sample median. For more about the approximation characteristics of TDigests, see [Percentiles are (usually) approximate](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-approximation).

计算中值绝对偏差的简单实现将整个样本存储在内存中，因此此聚合改为计算近似值。它使用[TDigest 数据结构](https://github.com/tdunning/t-digest) 来近似样本中位数和与样本中位数的偏差中位数。有关 TDigests 的近似特性的更多信息，请参阅 Percentiles [are (usually)](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-approximation) roach 。

The tradeoff between resource usage and accuracy of a TDigest’s quantile approximation, and therefore the accuracy of this aggregation’s approximation of median absolute deviation, is controlled by the `compression` parameter. A higher `compression` setting provides a more accurate approximation at the cost of higher memory usage. For more about the characteristics of the TDigest `compression` parameter see [Compression](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-compression).

资源使用和 TDigest 分位数近似的准确性之间的权衡，因此该聚合的中值绝对偏差近似的准确性由`compression`参数控制。更高的`compression`设置以更高的内存使用为代价提供更准确的近似值。有关 TDigest`compression`参数特性的更多信息， 请参阅 [压缩](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-compression)。

```console
GET reviews/_search
{
  "size": 0,
  "aggs": {
    "review_variability": {
      "median_absolute_deviation": {
        "field": "rating",
        "compression": 100
      }
    }
  }
}
```

The default `compression` value for this aggregation is `1000`. At this compression level this aggregation is usually within 5% of the exact result, but observed performance will depend on the sample data.

`compression`此聚合的默认值为`1000`。在此压缩级别，此聚合通常在精确结果的 5% 以内，但观察到的性能将取决于样本数据。

###  Script

In the example above, product reviews are on a scale of one to five. If you want to modify them to a scale of one to ten, use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html).

在上面的示例中，产品评论的等级为 1 到 5。如果要将它们修改为 1 到 10 的比例，请使用[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)。

```console
GET reviews/_search?filter_path=aggregations
{
  "size": 0,
  "runtime_mappings": {
    "rating.out_of_ten": {
      "type": "long",
      "script": {
        "source": "emit(doc['rating'].value * params.scaleFactor)",
        "params": {
          "scaleFactor": 2
        }
      }
    }
  },
  "aggs": {
    "review_average": {
      "avg": {
        "field": "rating.out_of_ten"
      }
    },
    "review_variability": {
      "median_absolute_deviation": {
        "field": "rating.out_of_ten"
      }
    }
  }
}
```

Which will result in:

这将导致：

```console-result
{
  "aggregations": {
    "review_average": {
      "value": 6.0
    },
    "review_variability": {
      "value": 4.0
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

Let’s be optimistic and assume some reviewers loved the product so much that they forgot to give it a rating. We’ll assign them five stars

让我们保持乐观，假设一些评论者非常喜欢该产品，以至于忘记给它评分。我们会给他们五颗星

```console
GET reviews/_search
{
  "size": 0,
  "aggs": {
    "review_variability": {
      "median_absolute_deviation": {
        "field": "rating",
        "missing": 5
      }
    }
  }
}
```

#  Min aggregation

A `single-value` metrics aggregation that keeps track and returns the minimum value among numeric values extracted from the aggregated documents.

一种`single-value`度量聚合，它跟踪并返回从聚合文档中提取的数值中的最小值。

> NOTE: The `min` and `max` aggregation operate on the `double` representation of the data. As a consequence, the result may be approximate when running on longs whose absolute value is greater than `2^53`.
>
> `min`和`max`在聚合操作`double`的数据的表示。因此，在绝对值大于 `2^53`的 long 上运行时，结果可能是近似的。

Computing the min price value across all documents:

计算所有文档的最低价格值：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "min_price": { "min": { "field": "price" } }
  }
}
```

Response:

```console-result
{
  ...

  "aggregations": {
    "min_price": {
      "value": 10.0
    }
  }
}
```

As can be seen, the name of the aggregation (`min_price` above) also serves as the key by which the aggregation result can be retrieved from the returned response.

可以看出，聚合的名称（`min_price`如上）也作为可以从返回的响应中检索聚合结果的键。

###  Script

If you need to get the `min` of something more complex than a single field, run the aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html).

如果您需要获取`min`比单个字段更复杂的内容，请在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。

```console
POST /sales/_search
{
  "size": 0,
  "runtime_mappings": {
    "price.adjusted": {
      "type": "double",
      "script": """
        double price = doc['price'].value;
        if (doc['promoted'].value) {
          price *= 0.8;
        }
        emit(price);
      """
    }
  },
  "aggs": {
    "min_price": {
      "min": { "field": "price.adjusted" }
    }
  }
}
```

### Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
POST /sales/_search
{
  "aggs": {
    "grade_min": {
      "min": {
        "field": "grade",
        "missing": 10 (1)
      }
    }
  }
}
```

1.  Documents without a value in the `grade` field will fall into the same bucket as documents that have the value `10`.

    该`grade`字段中没有值的文档将与具有该值的文档落入同一个桶中`10`。

###  Histogram fields

When `min` is computed on [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html), the result of the aggregation is the minimum of all elements in the `values` array. Note, that the `counts` array of the histogram is ignored.

当`min`在[histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)上计算时，聚合的结果是`values`数组中所有元素的最小值。请注意，`counts`直方图数组被忽略。

For example, for the following index that stores pre-aggregated histograms with latency metrics for different networks:

例如，对于以下存储具有不同网络延迟指标的预聚合直方图的索引：

```console
PUT metrics_index/_doc/1
{
  "network.name" : "net-1",
  "latency_histo" : {
      "values" : [0.1, 0.2, 0.3, 0.4, 0.5], 
      "counts" : [3, 7, 23, 12, 6] 
   }
}

PUT metrics_index/_doc/2
{
  "network.name" : "net-2",
  "latency_histo" : {
      "values" :  [0.1, 0.2, 0.3, 0.4, 0.5], 
      "counts" : [8, 17, 8, 7, 6] 
   }
}

POST /metrics_index/_search?size=0
{
  "aggs" : {
    "min_latency" : { "min" : { "field" : "latency_histo" } }
  }
}
```

The `min` aggregation will return the minimum value of all histogram fields:

该`min`聚合将返回所有直方图领域的最小值：

```console-result
{
  ...
  "aggregations": {
    "min_latency": {
      "value": 0.1
    }
  }
}
```

#  Percentile ranks aggregation

A `multi-value` metrics aggregation that calculates one or more percentile ranks over numeric values extracted from the aggregated documents. These values can be extracted from specific numeric or [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html) in the documents.

一种`multi-value`度量聚合，它根据从聚合文档中提取的数值计算一个或多个百分位等级。这些值可以从文档中的特定数字或[直方图字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)中提取。

> NOTE: Please see [Percentiles are (usually) approximate](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-approximation) and [Compression](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-compression) for advice regarding approximation and memory use of the percentile ranks aggregation
>
> 请参阅[百分位数是（通常）近似值](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-approximation) 和[压缩](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html#search-aggregations-metrics-percentile-aggregation-compression)以获取有关百分位数等级聚合的近似值和内存使用的建议

Percentile rank show the percentage of observed values which are below certain value. For example, if a value is greater than or equal to 95% of the observed values it is said to be at the 95th percentile rank.

百分位排名显示低于特定值的观察值的百分比。例如，如果某个值大于或等于观测值的 95%，则称其处于第 95 个百分位等级。

Assume your data consists of website load times. You may have a service agreement that 95% of page loads complete within 500ms and 99% of page loads complete within 600ms.

假设您的数据包含网站加载时间。您可能有一项服务协议，即 95% 的页面加载在 500 毫秒内完成，99% 的页面加载在 600 毫秒内完成。

Let’s look at a range of percentiles representing load time:

让我们看看代表加载时间的百分位数范围：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_ranks": {
      "percentile_ranks": {
        "field": "load_time",   (1)
        "values": [ 500, 600 ]
      }
    }
  }
}
```

1. The field `load_time` must be a numeric field

   该字段`load_time`必须是数字字段

The response will look like this:

```console-result
{
  ...

 "aggregations": {
    "load_time_ranks": {
      "values": {
        "500.0": 90.01,
        "600.0": 100.0
      }
    }
  }
}
```

From this information you can determine you are hitting the 99% load time target but not quite hitting the 95% load time target

从这些信息中，您可以确定您达到了 99% 的加载时间目标，但还没有完全达到 95% 的加载时间目标

###  Keyed Response

By default the `keyed` flag is set to `true` associates a unique string key with each bucket and returns the ranges as a hash rather than an array. Setting the `keyed` flag to `false` will disable this behavior:

默认情况下，该`keyed`标志设置为`true`将唯一的字符串键与每个存储桶相关联，并将范围作为散列而不是数组返回。将`keyed`标志设置为`false`将禁用此行为：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_ranks": {
      "percentile_ranks": {
        "field": "load_time",
        "values": [ 500, 600 ],
        "keyed": false
      }
    }
  }
}
```

Response:

```console-result
{
  ...

  "aggregations": {
    "load_time_ranks": {
      "values": [
        {
          "key": 500.0,
          "value": 90.01
        },
        {
          "key": 600.0,
          "value": 100.0
        }
      ]
    }
  }
}
```

###  Script

If you need to run the aggregation against values that aren’t indexed, use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html). For example, if our load times are in milliseconds but we want percentiles calculated in seconds:

如果您需要针对未编入索引的值运行聚合，请使用[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)。例如，如果我们的加载时间以毫秒为单位，但我们希望以秒为单位计算百分位数：

```console
GET latency/_search
{
  "size": 0,
  "runtime_mappings": {
    "load_time.seconds": {
      "type": "long",
      "script": {
        "source": "emit(doc['load_time'].value / params.timeUnit)",
        "params": {
          "timeUnit": 1000
        }
      }
    }
  },
  "aggs": {
    "load_time_ranks": {
      "percentile_ranks": {
        "values": [ 500, 600 ],
        "field": "load_time.seconds"
      }
    }
  }
}
```

###  HDR Histogram

> NOTE: This setting exposes the internal implementation of HDR Histogram and the syntax may change in the future.
>
> 此设置公开了 HDR Histogram 的内部实现，并且将来可能会更改语法。

[HDR Histogram](https://github.com/HdrHistogram/HdrHistogram) (High Dynamic Range Histogram) is an alternative implementation that can be useful when calculating percentile ranks for latency measurements as it can be faster than the t-digest implementation with the trade-off of a larger memory footprint. This implementation maintains a fixed worse-case percentage error (specified as a number of significant digits). This means that if data is recorded with values from 1 microsecond up to 1 hour (3,600,000,000 microseconds) in a histogram set to 3 significant digits, it will maintain a value resolution of 1 microsecond for values up to 1 millisecond and 3.6 seconds (or better) for the maximum tracked value (1 hour).

[HDR 直方图](https://github.com/HdrHistogram/HdrHistogram)（高动态范围直方图）是一种替代实现，在计算延迟测量的百分位等级时非常有用，因为它可以比 t-digest 实现更快，但需要在更大的内存占用上进行权衡。此实现保持固定的最坏情况百分比错误（指定为有效数字的数量）。这意味着，如果在设置为 3 位有效数字的直方图中记录了从 1 微秒到 1 小时（3,600,000,000 微秒）的值，对于高达 1 毫秒和 3.6 秒（或更好）的值，它将保持 1 微秒的值分辨率) 为最大跟踪值（1 小时）。

The HDR Histogram can be used by specifying the `hdr` object in the request:

可以通过`hdr`在请求中指定对象来使用 HDR Histogram ：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_ranks": {
      "percentile_ranks": {
        "field": "load_time",
        "values": [ 500, 600 ],
        "hdr": {                                  (1)
          "number_of_significant_value_digits": 3 (2)
        }
      }
    }
  }
}
```

1. `hdr` object indicates that HDR Histogram should be used to calculate the percentiles and specific settings for this algorithm can be specified inside the object

   `hdr` object 表示应该使用 HDR Histogram 来计算百分位数，并且可以在对象内部指定此算法的特定设置

2. `number_of_significant_value_digits` specifies the resolution of values for the histogram in number of significant digits

   `number_of_significant_value_digits` 以有效位数指定直方图值的分辨率

The HDRHistogram only supports positive values and will error if it is passed a negative value. It is also not a good idea to use the HDRHistogram if the range of values is unknown as this could lead to high memory usage.

HDRHistogram 仅支持正值，如果传递负值则会出错。如果值的范围未知，则使用 HDRHistogram 也不是一个好主意，因为这可能会导致高内存使用率。

### Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_ranks": {
      "percentile_ranks": {
        "field": "load_time",
        "values": [ 500, 600 ],
        "missing": 10           (1)
      }
    }
  }
}
```

1. Documents without a value in the `load_time` field will fall into the same bucket as documents that have the value `10`.

   该`load_time`字段中没有值的文档将与具有该值的文档落入同一个桶中`10`。

#  Percentiles aggregation

A `multi-value` metrics aggregation that calculates one or more percentiles over numeric values extracted from the aggregated documents. These values can be extracted from specific numeric or [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html) in the documents.

一种`multi-value`度量聚合，它计算从聚合文档中提取的数值的一个或多个百分位数。这些值可以从文档中的特定数字或[直方图字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)中提取。

Percentiles show the point at which a certain percentage of observed values occur. For example, the 95th percentile is the value which is greater than 95% of the observed values.

百分位数显示出现一定百分比的观察值的点。例如，第 95 个百分位数是大于观测值 95% 的值。

Percentiles are often used to find outliers. In normal distributions, the 0.13th and 99.87th percentiles represents three standard deviations from the mean. Any data which falls outside three standard deviations is often considered an anomaly.

百分位数通常用于查找异常值。在正态分布中，第 0.13 和第 99.87 个百分位数表示与平均值的三个标准差。任何超出三个标准差的数据通常都被视为异常。

When a range of percentiles are retrieved, they can be used to estimate the data distribution and determine if the data is skewed, bimodal, etc.

当检索到一系列百分位数时，它们可用于估计数据分布并确定数据是否偏斜、双峰等。

Assume your data consists of website load times. The average and median load times are not overly useful to an administrator. The max may be interesting, but it can be easily skewed by a single slow response.

假设您的数据包含网站加载时间。平均和中值加载时间对管理员来说并不过分有用。最大值可能很有趣，但它很容易被单个缓慢的响应所扭曲。

Let’s look at a range of percentiles representing load time:

让我们看看代表加载时间的百分位数范围：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_outlier": {
      "percentiles": {
        "field": "load_time" (1)
      }
    }
  }
}
```

1. The field `load_time` must be a numeric field

    该字段`load_time`必须是数字字段

By default, the `percentile` metric will generate a range of percentiles: `[ 1, 5, 25, 50, 75, 95, 99 ]`. The response will look like this:

默认情况下，该`percentile`指标将生成一个百分位数范围：`[ 1, 5, 25, 50, 75, 95, 99 ]`. 响应将如下所示：

```console-result
{
  ...

 "aggregations": {
    "load_time_outlier": {
      "values": {
        "1.0": 5.0,
        "5.0": 25.0,
        "25.0": 165.0,
        "50.0": 445.0,
        "75.0": 725.0,
        "95.0": 945.0,
        "99.0": 985.0
      }
    }
  }
}
```

As you can see, the aggregation will return a calculated value for each percentile in the default range. If we assume response times are in milliseconds, it is immediately obvious that the webpage normally loads in 10-725ms, but occasionally spikes to 945-985ms.

如您所见，聚合将为默认范围内的每个百分位返回一个计算值。如果我们假设响应时间以毫秒为单位，很明显网页通常在 10-725 毫秒内加载，但偶尔会达到 945-985 毫秒。

Often, administrators are only interested in outliers — the extreme percentiles. We can specify just the percents we are interested in (requested percentiles must be a value between 0-100 inclusive):

通常，管理员只对异常值感兴趣——极端的百分位数。我们可以只指定我们感兴趣的百分比（请求的百分比必须是 0-100 之间的值）：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_outlier": {
      "percentiles": {
        "field": "load_time",
        "percents": [ 95, 99, 99.9 ] (1)
      }
    }
  }
}
```

1.  Use the `percents` parameter to specify particular percentiles to calculate

   使用`percents`参数指定要计算的特定百分位数

###  Keyed Response

By default the `keyed` flag is set to `true` which associates a unique string key with each bucket and returns the ranges as a hash rather than an array. Setting the `keyed` flag to `false` will disable this behavior:

默认情况下，该`keyed`标志设置为`true`将唯一的字符串键与每个存储桶相关联，并将范围作为散列而不是数组返回。将`keyed`标志设置为`false`将禁用此行为：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_outlier": {
      "percentiles": {
        "field": "load_time",
        "keyed": false
      }
    }
  }
}
```

Response:

```console-result
{
  ...

  "aggregations": {
    "load_time_outlier": {
      "values": [
        {
          "key": 1.0,
          "value": 5.0
        },
        {
          "key": 5.0,
          "value": 25.0
        },
        {
          "key": 25.0,
          "value": 165.0
        },
        {
          "key": 50.0,
          "value": 445.0
        },
        {
          "key": 75.0,
          "value": 725.0
        },
        {
          "key": 95.0,
          "value": 945.0
        },
        {
          "key": 99.0,
          "value": 985.0
        }
      ]
    }
  }
}
```

###  Script

If you need to run the aggregation against values that aren’t indexed, use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html). For example, if our load times are in milliseconds but you want percentiles calculated in seconds:

如果您需要针对未编入索引的值运行聚合，请使用[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)。例如，如果我们的加载时间以毫秒为单位，但您希望以秒为单位计算百分位数：

```console
GET latency/_search
{
  "size": 0,
  "runtime_mappings": {
    "load_time.seconds": {
      "type": "long",
      "script": {
        "source": "emit(doc['load_time'].value / params.timeUnit)",
        "params": {
          "timeUnit": 1000
        }
      }
    }
  },
  "aggs": {
    "load_time_outlier": {
      "percentiles": {
        "field": "load_time.seconds"
      }
    }
  }
}
```

###  Percentiles are (usually) approximate

There are many different algorithms to calculate percentiles. The naive implementation simply stores all the values in a sorted array. To find the 50th percentile, you simply find the value that is at `my_array[count(my_array) * 0.5]`.

有许多不同的算法来计算百分位数。天真的实现只是将所有值存储在一个排序数组中。要找到第 50 个百分位数，您只需找到位于 的值`my_array[count(my_array) * 0.5]`。

Clearly, the naive implementation does not scale — the sorted array grows linearly with the number of values in your dataset. To calculate percentiles across potentially billions of values in an Elasticsearch cluster, *approximate* percentiles are calculated.

显然，简单的实现无法扩展——排序后的数组随着数据集中值的数量线性增长。为了计算 Elasticsearch 集群中潜在的数十亿个值的 百分位数，需要计算*近似*百分位数。

The algorithm used by the `percentile` metric is called TDigest (introduced by Ted Dunning in [Computing Accurate Quantiles using T-Digests](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf)).

该`percentile`指标使用的算法称为 TDigest（由 Ted Dunning 在[Computing Accurate Quantiles using T-](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf) Digests 中引入 ）。

When using this metric, there are a few guidelines to keep in mind:

使用此指标时，需要牢记一些准则：

- Accuracy is proportional to `q(1-q)`. This means that extreme percentiles (e.g. 99%) are more accurate than less extreme percentiles, such as the median

  精度与 成正比`q(1-q)`。这意味着极端百分位数（例如 99%）比不太极端的百分位数（例如中位数）更准确

- For small sets of values, percentiles are highly accurate (and potentially 100% accurate if the data is small enough).

  对于一小组值，百分位数是高度准确的（如果数据足够小，则可能 100% 准确）。

- As the quantity of values in a bucket grows, the algorithm begins to approximate the percentiles. It is effectively trading accuracy for memory savings. The exact level of inaccuracy is difficult to generalize, since it depends on your data distribution and volume of data being aggregated

  随着桶中值的数量增加，算法开始近似百分位数。它有效地为节省内存而交易准确性。准确程度难以概括，因为这取决于您的数据分布和聚合数据量

The following chart shows the relative error on a uniform distribution depending on the number of collected values and the requested percentile:

下图显示了均匀分布的相对误差，具体取决于收集值的数量和请求的百分位数：

![percentiles error](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/percentiles_error.png)

It shows how precision is better for extreme percentiles. The reason why error diminishes for large number of values is that the law of large numbers makes the distribution of values more and more uniform and the t-digest tree can do a better job at summarizing it. It would not be the case on more skewed distributions.

它显示了极端百分位数的精度如何更好。大量值的误差减少的原因是大数定律使值的分布越来越均匀，t-digest树可以更好地总结它。对于更偏斜的分布，情况就不是这样了。

> WARNING: Percentile aggregations are also [non-deterministic](https://en.wikipedia.org/wiki/Nondeterministic_algorithm). This means you can get slightly different results using the same data.
>
> 百分位聚合也是 [非确定性的](https://en.wikipedia.org/wiki/Nondeterministic_algorithm)。这意味着您可以使用相同的数据获得略有不同的结果。

###  Compression

Approximate algorithms must balance memory utilization with estimation accuracy. This balance can be controlled using a `compression` parameter:

近似算法必须平衡内存利用率和估计精度。可以使用`compression`参数控制此平衡：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_outlier": {
      "percentiles": {
        "field": "load_time",
        "tdigest": {
          "compression": 200    (1)
        }
      }
    }
  }
}
```

1.  Compression controls memory usage and approximation error

   压缩控制内存使用和近似误差

The TDigest algorithm uses a number of "nodes" to approximate percentiles — the more nodes available, the higher the accuracy (and large memory footprint) proportional to the volume of data. The `compression` parameter limits the maximum number of nodes to `20 * compression`.

TDigest 算法使用多个“节点”来近似百分位数——可用节点越多，与数据量成正比的准确性（和大内存占用）就越高。该`compression`参数将最大节点数限制为`20 * compression`。

Therefore, by increasing the compression value, you can increase the accuracy of your percentiles at the cost of more memory. Larger compression values also make the algorithm slower since the underlying tree data structure grows in size, resulting in more expensive operations. The default compression value is `100`.

因此，通过增加压缩值，您可以以更多内存为代价来提高百分位数的准确性。较大的压缩值也会使算法变慢，因为底层树数据结构的大小会增加，从而导致操作成本更高。默认压缩值为 `100`.

A "node" uses roughly 32 bytes of memory, so under worst-case scenarios (large amount of data which arrives sorted and in-order) the default settings will produce a TDigest roughly 64KB in size. In practice data tends to be more random and the TDigest will use less memory.

“节点”使用大约 32 字节的内存，因此在最坏的情况下（大量数据按顺序到达），默认设置将产生大约 64KB 大小的 TDigest。在实践中，数据往往更随机，TDigest 将使用更少的内存。

###  HDR Histogram

> NOTE: This setting exposes the internal implementation of HDR Histogram and the syntax may change in the future.
>
> 此设置公开了 HDR Histogram 的内部实现，并且将来可能会更改语法。

[HDR Histogram](https://github.com/HdrHistogram/HdrHistogram) (High Dynamic Range Histogram) is an alternative implementation that can be useful when calculating percentiles for latency measurements as it can be faster than the t-digest implementation with the trade-off of a larger memory footprint. This implementation maintains a fixed worse-case percentage error (specified as a number of significant digits). This means that if data is recorded with values from 1 microsecond up to 1 hour (3,600,000,000 microseconds) in a histogram set to 3 significant digits, it will maintain a value resolution of 1 microsecond for values up to 1 millisecond and 3.6 seconds (or better) for the maximum tracked value (1 hour).

[HDR 直方图](https://github.com/HdrHistogram/HdrHistogram)（高动态范围直方图）是一种替代实现，在计算延迟测量的百分位数时非常有用，因为它可以比 t-digest 实现更快，但需要权衡更大的内存占用。此实现保持固定的最坏情况百分比错误（指定为有效数字的数量）。这意味着，如果在设置为 3 位有效数字的直方图中记录了从 1 微秒到 1 小时（3,600,000,000 微秒）的值，对于高达 1 毫秒和 3.6 秒（或更好）的值，它将保持 1 微秒的值分辨率) 为最大跟踪值（1 小时）。

The HDR Histogram can be used by specifying the `method` parameter in the request:可以通过`method`在请求中指定参数来使用 HDR Histogram ：

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "load_time_outlier": {
      "percentiles": {
        "field": "load_time",
        "percents": [ 95, 99, 99.9 ],
        "hdr": {                                  (1)
          "number_of_significant_value_digits": 3 (2)
        }
      }
    }
  }
}
```

1. `hdr` object indicates that HDR Histogram should be used to calculate the percentiles and specific settings for this algorithm can be specified inside the object

   `hdr` object 表示应该使用 HDR Histogram 来计算百分位数，并且可以在对象内部指定此算法的特定设置

2. `number_of_significant_value_digits` specifies the resolution of values for the histogram in number of significant digits

3. `number_of_significant_value_digits` 以有效位数指定直方图值的分辨率

The HDRHistogram only supports positive values and will error if it is passed a negative value. It is also not a good idea to use the HDRHistogram if the range of values is unknown as this could lead to high memory usage.

HDRHistogram 仅支持正值，如果传递负值则会出错。如果值的范围未知，则使用 HDRHistogram 也不是一个好主意，因为这可能会导致高内存使用率。

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
GET latency/_search
{
  "size": 0,
  "aggs": {
    "grade_percentiles": {
      "percentiles": {
        "field": "grade",
        "missing": 10       (1)
      }
    }
  }
}
```

1. Documents without a value in the `grade` field will fall into the same bucket as documents that have the value `10`.

   该`grade`字段中没有值的文档将与具有该值的文档落入同一个桶中`10`。

#  Rate aggregation

A `rate` metrics aggregation can be used only inside a `date_histogram` and calculates a rate of documents or a field in each `date_histogram` bucket. The field values can be generated extracted from specific numeric or [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html) in the documents.

`rate`度量聚集只能内的使用`date_histogram`，并计算文档的速率或在每一个场 `date_histogram`桶。可以从文档中的特定数字或[直方图字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)中提取字段值 。

###  Syntax

A `rate` aggregation looks like this in isolation:

单独的`rate`聚合看起来像这样：

```js
{
  "rate": {
    "unit": "month",
    "field": "requests"
  }
}
```

The following request will group all sales records into monthly bucket and than convert the number of sales transaction in each bucket into per annual sales rate.

以下请求将所有销售记录分组到每月桶中，然后将每个桶中的销售交易数量转换为每年的销售率。

```console
GET sales/_search
{
  "size": 0,
  "aggs": {
    "by_date": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"  (1)
      },
      "aggs": {
        "my_rate": {
          "rate": {
            "unit": "year"  (2)
          }
        }
      }
    }
  }
}
```

1.  Histogram is grouped by month.

    直方图按月份分组。

2.  But the rate is converted into annual rate.

   但该费率转换为年费率。

The response will return the annual rate of transaction in each bucket. Since there are 12 months per year, the annual rate will be automatically calculated by multiplying monthly rate by 12.

响应将返回每个存储桶中的年交易率。由于每年有 12 个月，因此将通过将月率乘以 12 来自动计算年率。

```console-result
{
  ...
  "aggregations" : {
    "by_date" : {
      "buckets" : [
        {
          "key_as_string" : "2015/01/01 00:00:00",
          "key" : 1420070400000,
          "doc_count" : 3,
          "my_rate" : {
            "value" : 36.0
          }
        },
        {
          "key_as_string" : "2015/02/01 00:00:00",
          "key" : 1422748800000,
          "doc_count" : 2,
          "my_rate" : {
            "value" : 24.0
          }
        },
        {
          "key_as_string" : "2015/03/01 00:00:00",
          "key" : 1425168000000,
          "doc_count" : 2,
          "my_rate" : {
            "value" : 24.0
          }
        }
      ]
    }
  }
}
```

Instead of counting the number of documents, it is also possible to calculate a sum of all values of the fields in the documents in each bucket or the number of values in each bucket. The following request will group all sales records into monthly bucket and than calculate the total monthly sales and convert them into average daily sales.

除了计算文档数量，还可以计算每个桶中文档中字段的所有值的总和或每个桶中值的数量。以下请求将所有销售记录分组到每月桶中，然后计算每月总销售额并将其转换为平均每日销售额。

```console
GET sales/_search
{
  "size": 0,
  "aggs": {
    "by_date": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"  (1)
      },
      "aggs": {
        "avg_price": {
          "rate": {
            "field": "price", (2)
            "unit": "day"  (3)
          }
        }
      }
    }
  }
}
```

1. Histogram is grouped by month.

   直方图按月份分组。

2. Calculate sum of all sale prices

   计算所有销售价格的总和

3. Convert to average daily sales

    转换为平均每日销售额

The response will contain the average daily sale prices for each month.

响应将包含每个月的平均每日销售价格。

```console-result
{
  ...
  "aggregations" : {
    "by_date" : {
      "buckets" : [
        {
          "key_as_string" : "2015/01/01 00:00:00",
          "key" : 1420070400000,
          "doc_count" : 3,
          "avg_price" : {
            "value" : 17.741935483870968
          }
        },
        {
          "key_as_string" : "2015/02/01 00:00:00",
          "key" : 1422748800000,
          "doc_count" : 2,
          "avg_price" : {
            "value" : 2.142857142857143
          }
        },
        {
          "key_as_string" : "2015/03/01 00:00:00",
          "key" : 1425168000000,
          "doc_count" : 2,
          "avg_price" : {
            "value" : 12.096774193548388
          }
        }
      ]
    }
  }
}
```

By adding the `mode` parameter with the value `value_count`, we can change the calculation from `sum` to the number of values of the field:

通过添加`mode`带有 value的参数`value_count`，我们可以将计算从 更改`sum`为字段的值数量：

```console
GET sales/_search
{
  "size": 0,
  "aggs": {
    "by_date": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"  (1)
      },
      "aggs": {
        "avg_number_of_sales_per_year": {
          "rate": {
            "field": "price", (2)
            "unit": "year",  (3)
            "mode": "value_count" (4)
          }
        }
      }
    }
  }
}
```

1.  Histogram is grouped by month.

   直方图按月份分组。

2.  Calculate number of all sale prices

   计算所有销售价格的数量

3.  Convert to annual counts

    转换为年度计数

4. Changing the mode to value count

    将模式更改为值计数

The response will contain the average daily sale prices for each month.

响应将包含每个月的平均每日销售价格。

```console-result
{
  ...
  "aggregations" : {
    "by_date" : {
      "buckets" : [
        {
          "key_as_string" : "2015/01/01 00:00:00",
          "key" : 1420070400000,
          "doc_count" : 3,
          "avg_number_of_sales_per_year" : {
            "value" : 36.0
          }
        },
        {
          "key_as_string" : "2015/02/01 00:00:00",
          "key" : 1422748800000,
          "doc_count" : 2,
          "avg_number_of_sales_per_year" : {
            "value" : 24.0
          }
        },
        {
          "key_as_string" : "2015/03/01 00:00:00",
          "key" : 1425168000000,
          "doc_count" : 2,
          "avg_number_of_sales_per_year" : {
            "value" : 24.0
          }
        }
      ]
    }
  }
}
```

By default `sum` mode is used.

默认情况`sum`下使用模式。

**`"mode": "sum"`**

calculate the sum of all values field

计算所有值字段的总和

**`"mode": "value_count"`**

use the number of values in the field

使用字段中的值数量

###  Relationship between bucket sizes and rate

The `rate` aggregation supports all rate that can be used [calendar_intervals parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-datehistogram-aggregation.html#calendar_intervals) of `date_histogram` aggregation. The specified rate should compatible with the `date_histogram` aggregation interval, i.e. it should be possible to convert the bucket size into the rate. By default the interval of the `date_histogram` is used.

该`rate`集合支持，可用于所有率[calendar_intervals参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-datehistogram-aggregation.html#calendar_intervals)的`date_histogram` 聚集。指定的速率应该与`date_histogram`聚合间隔兼容，即应该可以将桶大小转换为速率。默认情况下使用的间隔`date_histogram`。

- **`"rate": "second"`**

  compatible with all intervals

  兼容所有区间

- **`"rate": "minute"`**

  compatible with all intervals

  兼容所有区间

- **`"rate": "hour"`**

  compatible with all intervals

  兼容所有区间

- **`"rate": "day"`**

  compatible with all intervals

  兼容所有区间

- **`"rate": "week"`**

  compatible with all intervals

  兼容所有区间

- **`"rate": "month"`**

  compatible with only with `month`, `quarter` and `year` calendar intervals

  仅与`month`,`quarter`和`year`日历间隔 兼容

- **`"rate": "quarter"`**

  compatible with only with `month`, `quarter` and `year` calendar intervals

  仅与`month`,`quarter`和`year`日历间隔 兼容

- **`"rate": "year"`**

  compatible with only with `month`, `quarter` and `year` calendar intervals

  仅与`month`,`quarter`和`year`日历间隔 兼容

There is also an additional limitations if the date histogram is not a direct parent of the rate histogram. In this case both rate interval and histogram interval have to be in the same group: [`second`, ` minute`, `hour`, `day`, `week`] or [`month`, `quarter`, `year`]. For example, if the date histogram is `month` based, only rate intervals of `month`, `quarter` or `year` are supported. If the date histogram is `day` based, only `second`, ` minute`, `hour`, `day`, and `week` rate intervals are supported.

如果日期直方图不是汇率直方图的直接父项，则还有一个额外的限制。在这种情况下，速率间隔和直方图间隔必须在同一组中：[ `second`, `minute`, `hour`, `day`, `week`] 或 [ `month`, `quarter`, `year`]。例如，如果日期直方图`month`基于，只有评论的时间间隔`month`，`quarter`或`year`支持。如果基于日期直方图`day`，则仅支持 `second`、 `分钟` `hour`、`day`、 和`week`速率间隔。

###  Script

If you need to run the aggregation against values that aren’t indexed, run the aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html). For example, if we need to adjust our prices before calculating rates:

如果您需要针对未编入索引的值运行聚合，请在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。例如，如果我们需要在计算费率之前调整价格：

```console
GET sales/_search
{
  "size": 0,
  "runtime_mappings": {
    "price.adjusted": {
      "type": "double",
      "script": {
        "source": "emit(doc['price'].value * params.adjustment)",
        "params": {
          "adjustment": 0.9
        }
      }
    }
  },
  "aggs": {
    "by_date": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "avg_price": {
          "rate": {
            "field": "price.adjusted"
          }
        }
      }
    }
  }
}
```

```console-result
{
  ...
  "aggregations" : {
    "by_date" : {
      "buckets" : [
        {
          "key_as_string" : "2015/01/01 00:00:00",
          "key" : 1420070400000,
          "doc_count" : 3,
          "avg_price" : {
            "value" : 495.0
          }
        },
        {
          "key_as_string" : "2015/02/01 00:00:00",
          "key" : 1422748800000,
          "doc_count" : 2,
          "avg_price" : {
            "value" : 54.0
          }
        },
        {
          "key_as_string" : "2015/03/01 00:00:00",
          "key" : 1425168000000,
          "doc_count" : 2,
          "avg_price" : {
            "value" : 337.5
          }
        }
      ]
    }
  }
}
```

#  Scripted metric aggregation

A metric aggregation that executes using scripts to provide a metric output.

使用脚本执行以提供指标输出的指标聚合。

> WARNING: Using scripts can result in slower search speeds. See [Scripts, caching, and search speed](https://www.elastic.co/guide/en/elasticsearch/reference/master/scripts-and-search-speed.html).
>
> 使用脚本会导致搜索速度变慢。请参阅 [脚本、缓存和搜索速度](https://www.elastic.co/guide/en/elasticsearch/reference/master/scripts-and-search-speed.html)。

Example:

```console
POST ledger/_search?size=0
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "profit": {
      "scripted_metric": {
        "init_script": "state.transactions = []", (1)
        "map_script": "state.transactions.add(doc.type.value == 'sale' ? doc.amount.value : -1 * doc.amount.value)",
        "combine_script": "double profit = 0; for (t in state.transactions) { profit += t } return profit",
        "reduce_script": "double profit = 0; for (a in states) { profit += a } return profit"
      }
    }
  }
}
```

1. `init_script` is an optional parameter, all other scripts are required.

   `init_script` 是可选参数，所有其他脚本都是必需的。

The above aggregation demonstrates how one would use the script aggregation compute the total profit from sale and cost transactions.

上述聚合演示了如何使用脚本聚合计算销售和成本交易的总利润。

The response for the above aggregation:

上述聚合的响应：

```console-result
{
  "took": 218,
  ...
  "aggregations": {
    "profit": {
      "value": 240.0
    }
  }
}
```

The above example can also be specified using stored scripts as follows:

也可以使用存储的脚本指定上述示例，如下所示：

```console
POST ledger/_search?size=0
{
  "aggs": {
    "profit": {
      "scripted_metric": {
        "init_script": {
          "id": "my_init_script"
        },
        "map_script": {
          "id": "my_map_script"
        },
        "combine_script": {
          "id": "my_combine_script"
        },
        "params": {
          "field": "amount"           (1)
        },
        "reduce_script": {
          "id": "my_reduce_script"
        }
      }
    }
  }
}
```

1. script parameters for `init`, `map` and `combine` scripts must be specified in a global `params` object so that it can be shared between the scripts.

    脚本参数`init`，`map`而且`combine`脚本必须在全球被指定的`params`对象，以便它可以在脚本之间共享。

For more details on specifying scripts see [script documentation](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html).

有关指定脚本的更多详细信息，请参阅[脚本文档](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html)。

###  Allowed return types

Whilst any valid script object can be used within a single script, the scripts must return or store in the `state` object only the following types:

虽然可以在单个脚本中使用任何有效的脚本对象，但脚本必须仅返回或在`state`对象中存储以下类型：

- primitive types

  原始类型

- String

  字符串

- Map (containing only keys and values of the types listed here)

  映射（仅包含此处列出的类型的键和值）

- Array (containing elements of only the types listed here)

  数组（仅包含此处列出的类型的元素）

###  Scope of scripts

The scripted metric aggregation uses scripts at 4 stages of its execution:

脚本化指标聚合在其执行的 4 个阶段使用脚本：

**init_script**

Executed prior to any collection of documents. Allows the aggregation to set up any initial state.

在任何文档集合之前执行。允许聚合设置任何初始状态。

In the above example, the `init_script` creates an array `transactions` in the `state` object.

在上面的例子中，在对象中`init_script`创建了一个数组。`transactions``state`

**map_script**

Executed once per document collected. This is a required script. If no combine_script is specified, the resulting state needs to be stored in the `state` object.

每个收集的文档执行一次。这是必需的脚本。如果未指定 combine_script ，则需要将结果状态存储在`state`对象中。

In the above example, the `map_script` checks the value of the type field. If the value is *sale* the value of the amount field is added to the transactions array. If the value of the type field is not *sale* the negated value of the amount field is added to transactions.

在上面的例子中，`map_script`检查 type 字段的值。如果值为*sale*，则将金额字段的值添加到交易数组中。如果类型字段的值不是*销售*，则将金额字段的否定值添加到交易中。

**combine_script**

Executed once on each shard after document collection is complete. This is a required script. Allows the aggregation to consolidate the state returned from each shard.

文档收集完成后在每个分片上执行一次。这是必需的脚本。允许聚合合并从每个分片返回的状态。

In the above example, the `combine_script` iterates through all the stored transactions, summing the values in the `profit` variable and finally returns `profit`.

在上面的例子中，`combine_script`遍历所有存储的事务，对`profit`变量中的值求和，最后返回`profit`。

**reduce_script**

Executed once on the coordinating node after all shards have returned their results. This is a required script. The script is provided with access to a variable `states` which is an array of the result of the combine_script on each shard.

在所有分片返回结果后，在协调节点上执行一次。这是必需的脚本。该脚本可以访问一个变量`states`，该变量是每个分片上 combine_script 结果的数组。

In the above example, the `reduce_script` iterates through the `profit` returned by each shard summing the values before returning the final combined profit which will be returned in the response of the aggregation.

在上面的示例中，在返回将在聚合响应中返回的最终组合利润之前，`reduce_script`迭代`profit`每个分片返回的值对值求和。

### Worked example

Imagine a situation where you index the following documents into an index with 2 shards:

想象一下您将以下文档索引到具有 2 个分片的索引中的情况：

```console
PUT /transactions/_bulk?refresh
{"index":{"_id":1}}
{"type": "sale","amount": 80}
{"index":{"_id":2}}
{"type": "cost","amount": 10}
{"index":{"_id":3}}
{"type": "cost","amount": 30}
{"index":{"_id":4}}
{"type": "sale","amount": 130}
```

Lets say that documents 1 and 3 end up on shard A and documents 2 and 4 end up on shard B. The following is a breakdown of what the aggregation result is at each stage of the example above.

假设文档 1 和 3 最终在分片 A 上，文档 2 和 4 最终在分片 B 上。以下是对上面示例的每个阶段的聚合结果的细分。

#### Before init_script

`state` is initialized as a new empty object.

`state` 被初始化为一个新的空对象。

```js
"state" : {}
```

####  After init_script

This is run once on each shard before any document collection is performed, and so we will have a copy on each shard:

这在执行任何文档收集之前在每个分片上运行一次，因此我们将在每个分片上都有一个副本：

**Shard A**

```js
"state" : {
    "transactions" : []
}
```

**Shard B**

```js
"state" : {
    "transactions" : []
}
```

####  After map_script

Each shard collects its documents and runs the map_script on each document that is collected:

每个分片收集其文档并在收集的每个文档上运行 map_script：

**Shard A**

```js
"state" : {
    "transactions" : [ 80, -30 ]
}
```

**Shard B**

```js
"state" : {
    "transactions" : [ -10, 130 ]
}
```

#### After combine_script

The combine_script is executed on each shard after document collection is complete and reduces all the transactions down to a single profit figure for each shard (by summing the values in the transactions array) which is passed back to the coordinating node:

在文档收集完成后，在每个分片上执行 combine_script 并将所有交易减少到每个分片的单个利润数字（通过对交易数组中的值求和），并将其传递回协调节点：

**Shard A**

50

**Shard B**

120

####  After reduce_script

The reduce_script receives a `states` array containing the result of the combine script for each shard:

reduce_script 接收一个`states`包含每个分片的组合脚本结果的数组：

```js
"states" : [
    50,
    120
]
```

It reduces the responses for the shards down to a final overall profit figure (by summing the values) and returns this as the result of the aggregation to produce the response:

它将分片的响应减少到最终的整体利润数字（通过对值求和），并将其作为聚合结果返回以生成响应：

```js
{
  ...

  "aggregations": {
    "profit": {
      "value": 170
    }
  }
}
```

###  Other parameters

**params**

Optional. An object whose contents will be passed as variables to the `init_script`, `map_script` and `combine_script`. This can be useful to allow the user to control the behavior of the aggregation and for storing state between the scripts. If this is not specified, the default is the equivalent of providing:

可选的。一个对象，其内容将作为变量传递给 `init_script`,`map_script`和`combine_script`。这对于允许用户控制聚合的行为和在脚本之间存储状态很有用。如果未指定，则默认值相当于提供：

```js
"params" : {}
```

###  Empty buckets

If a parent bucket of the scripted metric aggregation does not collect any documents an empty aggregation response will be returned from the shard with a `null` value. In this case the `reduce_script`'s `states` variable will contain `null` as a response from that shard. `reduce_script`'s should therefore expect and deal with `null` responses from shards.

如果脚本化指标聚合的父存储桶未收集任何文档，将从分片返回一个带有`null`值的空聚合响应。在这种情况下，`reduce_script`的`states`变量将包含`null`作为来自该分片的响应。 `reduce_script`因此， 's 应该期望并处理`null`来自分片的响应。

#  Stats aggregation

A `multi-value` metrics aggregation that computes stats over numeric values extracted from the aggregated documents.

一种`multi-value`度量聚合，它计算从聚合文档中提取的数值的统计信息。

The stats that are returned consist of: `min`, `max`, `sum`, `count` and `avg`.

返回的统计数据包括：`min`，`max`，`sum`，`count`和`avg`。

Assuming the data consists of documents representing exams grades (between 0 and 100) of students

假设数据包含代表学生考试成绩（0 到 100 之间）的文件

```console
POST /exams/_search?size=0
{
  "aggs": {
    "grades_stats": { "stats": { "field": "grade" } }
  }
}
```

The above aggregation computes the grades statistics over all documents. The aggregation type is `stats` and the `field` setting defines the numeric field of the documents the stats will be computed on. The above will return the following:

上述聚合计算所有文档的成绩统计。聚合类型是`stats`，并且`field`设置定义了将在其上计算统计信息的文档的数字字段。以上将返回以下内容：

```console-result
{
  ...

  "aggregations": {
    "grades_stats": {
      "count": 2,
      "min": 50.0,
      "max": 100.0,
      "avg": 75.0,
      "sum": 150.0
    }
  }
}
```

The name of the aggregation (`grades_stats` above) also serves as the key by which the aggregation result can be retrieved from the returned response.

聚合的名称（`grades_stats`如上）也用作可以从返回的响应中检索聚合结果的键。

###  Script

If you need to get the `stats` for something more complex than a single field, run the aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html).

如果您需要获取`stats`比单个字段更复杂的内容，请在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。

```console
POST /exams/_search
{
  "size": 0,
  "runtime_mappings": {
    "grade.weighted": {
      "type": "double",
      "script": """
        emit(doc['grade'].value * doc['weight'].value)
      """
    }
  },
  "aggs": {
    "grades_stats": {
      "stats": {
        "field": "grade.weighted"
      }
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
POST /exams/_search?size=0
{
  "aggs": {
    "grades_stats": {
      "stats": {
        "field": "grade",
        "missing": 0      (1)
      }
    }
  }
}
```

1.  Documents without a value in the `grade` field will fall into the same bucket as documents that have the value `0`.

    该`grade`字段中没有值的文档将与具有该值的文档落入同一个桶中`0`。

# String stats aggregation

A `multi-value` metrics aggregation that computes statistics over string values extracted from the aggregated documents. These values can be retrieved either from specific `keyword` fields.

一种`multi-value`度量聚合，用于计算从聚合文档中提取的字符串值的统计信息。可以从特定`keyword`字段中检索这些值。

The string stats aggregation returns the following results:

字符串 stats 聚合返回以下结果：

- `count` - The number of non-empty fields counted.

  `count` - 计算的非空字段数。

- `min_length` - The length of the shortest term.

  `min_length` - 最短期限的长度。

- `max_length` - The length of the longest term.

  `max_length` - 最长期限的长度。

- `avg_length` - The average length computed over all terms.

  `avg_length` - 计算所有术语的平均长度。

- `entropy` - The [Shannon Entropy](https://en.wikipedia.org/wiki/Entropy_(information_theory)) value computed over all terms collected by the aggregation. Shannon entropy quantifies the amount of information contained in the field. It is a very useful metric for measuring a wide range of properties of a data set, such as diversity, similarity, randomness etc.

  `entropy`-对聚合收集的所有项计算的[香农熵](https://en.wikipedia.org/wiki/Entropy_(information_theory))值。香农熵量化了字段中包含的信息量。它是一个非常有用的度量标准，可用于测量数据集的各种属性，例如多样性、相似性、随机性等。

For example:

```console
POST /my-index-000001/_search?size=0
{
  "aggs": {
    "message_stats": { "string_stats": { "field": "message.keyword" } }
  }
}
```

The above aggregation computes the string statistics for the `message` field in all documents. The aggregation type is `string_stats` and the `field` parameter defines the field of the documents the stats will be computed on. The above will return the following:

上述聚合计算`message`所有文档中字段的字符串统计信息。聚合类型是`string_stats`，`field`参数定义将计算统计信息的文档的字段。以上将返回以下内容：

```console-result
{
  ...

  "aggregations": {
    "message_stats": {
      "count": 5,
      "min_length": 24,
      "max_length": 30,
      "avg_length": 28.8,
      "entropy": 3.94617750050791
    }
  }
}
```

The name of the aggregation (`message_stats` above) also serves as the key by which the aggregation result can be retrieved from the returned response.

聚合的名称（`message_stats`如上）也用作可以从返回的响应中检索聚合结果的键。

###  Character distribution

The computation of the Shannon Entropy value is based on the probability of each character appearing in all terms collected by the aggregation. To view the probability distribution for all characters, we can add the `show_distribution` (default: `false`) parameter.

香农熵值的计算基于每个字符出现在聚合收集的所有术语中的概率。要查看所有字符的概率分布，我们可以添加`show_distribution`(default: `false`) 参数。

```console
POST /my-index-000001/_search?size=0
{
  "aggs": {
    "message_stats": {
      "string_stats": {
        "field": "message.keyword",
        "show_distribution": true  (1)
      }
    }
  }
}
```

1. Set the `show_distribution` parameter to `true`, so that probability distribution for all characters is returned in the results.

    将`show_distribution`参数设置为`true`，以便在结果中返回所有字符的概率分布。

```console-result
{
  ...

  "aggregations": {
    "message_stats": {
      "count": 5,
      "min_length": 24,
      "max_length": 30,
      "avg_length": 28.8,
      "entropy": 3.94617750050791,
      "distribution": {
        " ": 0.1527777777777778,
        "e": 0.14583333333333334,
        "s": 0.09722222222222222,
        "m": 0.08333333333333333,
        "t": 0.0763888888888889,
        "h": 0.0625,
        "a": 0.041666666666666664,
        "i": 0.041666666666666664,
        "r": 0.041666666666666664,
        "g": 0.034722222222222224,
        "n": 0.034722222222222224,
        "o": 0.034722222222222224,
        "u": 0.034722222222222224,
        "b": 0.027777777777777776,
        "w": 0.027777777777777776,
        "c": 0.013888888888888888,
        "E": 0.006944444444444444,
        "l": 0.006944444444444444,
        "1": 0.006944444444444444,
        "2": 0.006944444444444444,
        "3": 0.006944444444444444,
        "4": 0.006944444444444444,
        "y": 0.006944444444444444
      }
    }
  }
}
```

The `distribution` object shows the probability of each character appearing in all terms. The characters are sorted by descending probability.

该`distribution`对象显示每个字符出现在所有术语中的概率。字符按降序排列。

###  Script

If you need to get the `string_stats` for something more complex than a single field, run the aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html).

如果您需要获取`string_stats`比单个字段更复杂的内容，请在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。

```console
POST /my-index-000001/_search
{
  "size": 0,
  "runtime_mappings": {
    "message_and_context": {
      "type": "keyword",
      "script": """
        emit(doc['message.keyword'].value + ' ' + doc['context.keyword'].value)
      """
    }
  },
  "aggs": {
    "message_stats": {
      "string_stats": { "field": "message_and_context" }
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
POST /my-index-000001/_search?size=0
{
  "aggs": {
    "message_stats": {
      "string_stats": {
        "field": "message.keyword",
        "missing": "[empty message]" (1)
      }
    }
  }
}
```

Documents without a value in the `message` field will be treated as documents that have the value `[empty message]`.

`message`字段中没有值的文档将被视为具有值的文档`[empty message]`。

#  Sum aggregation

A `single-value` metrics aggregation that sums up numeric values that are extracted from the aggregated documents. These values can be extracted either from specific numeric or [histogram](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html) fields.

对`single-value`从聚合文档中提取的数值求和的指标聚合。这些值可以从特定的数字或[直方图](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)字段中提取。

Assuming the data consists of documents representing sales records we can sum the sale price of all hats with:

假设数据由代表销售记录的文档组成，我们可以将所有帽子的销售价格与：

```console
POST /sales/_search?size=0
{
  "query": {
    "constant_score": {
      "filter": {
        "match": { "type": "hat" }
      }
    }
  },
  "aggs": {
    "hat_prices": { "sum": { "field": "price" } }
  }
}
```

Resulting in:

```console-result
{
  ...
  "aggregations": {
    "hat_prices": {
      "value": 450.0
    }
  }
}
```

The name of the aggregation (`hat_prices` above) also serves as the key by which the aggregation result can be retrieved from the returned response.

聚合的名称（`hat_prices`如上）也用作可以从返回的响应中检索聚合结果的键。

### Script

If you need to get the `sum` for something more complex than a single field, run the aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html).

如果您需要获取`sum`比单个字段更复杂的内容，请在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。

```console
POST /sales/_search?size=0
{
  "runtime_mappings": {
    "price.weighted": {
      "type": "double",
      "script": """
        double price = doc['price'].value;
        if (doc['promoted'].value) {
          price *= 0.8;
        }
        emit(price);
      """
    }
  },
  "query": {
    "constant_score": {
      "filter": {
        "match": { "type": "hat" }
      }
    }
  },
  "aggs": {
    "hat_prices": {
      "sum": {
        "field": "price.weighted"
      }
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default documents missing the value will be ignored but it is also possible to treat them as if they had a value. For example, this treats all hat sales without a price as being `100`.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，缺少值的文档将被忽略，但也可以将它们视为具有值。例如，这将所有没有价格的帽子销售视为`100`。

```console
POST /sales/_search?size=0
{
  "query": {
    "constant_score": {
      "filter": {
        "match": { "type": "hat" }
      }
    }
  },
  "aggs": {
    "hat_prices": {
      "sum": {
        "field": "price",
        "missing": 100 
      }
    }
  }
}
```

###  Histogram fields

When sum is computed on [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html), the result of the aggregation is the sum of all elements in the `values` array multiplied by the number in the same position in the `counts` array.

当 sum 在[histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)上计算时，聚合的结果是`values` 数组中所有元素的总和乘以数组中相同位置的数字`counts`。

For example, for the following index that stores pre-aggregated histograms with latency metrics for different networks:

例如，对于以下存储具有不同网络延迟指标的预聚合直方图的索引：

```console
PUT metrics_index/_doc/1
{
  "network.name" : "net-1",
  "latency_histo" : {
      "values" : [0.1, 0.2, 0.3, 0.4, 0.5], 
      "counts" : [3, 7, 23, 12, 6] 
   }
}

PUT metrics_index/_doc/2
{
  "network.name" : "net-2",
  "latency_histo" : {
      "values" :  [0.1, 0.2, 0.3, 0.4, 0.5], 
      "counts" : [8, 17, 8, 7, 6] 
   }
}

POST /metrics_index/_search?size=0
{
  "aggs" : {
    "total_latency" : { "sum" : { "field" : "latency_histo" } }
  }
}
```

For each histogram field the `sum` aggregation will multiply each number in the `values` array <1> multiplied by its associated count in the `counts` array <2>. Eventually, it will add all values for all histograms and return the following result:

对于每个直方图字段，`sum`聚合会将`values`数组 <1>中的每个数字乘以`counts`数组 <2>中的关联计数。最终，它会将所有直方图的所有值相加并返回以下结果：

```console-result
{
  ...
  "aggregations": {
    "total_latency": {
      "value": 28.8
    }
  }
}
```

# T-test aggregation

A `t_test` metrics aggregation that performs a statistical hypothesis test in which the test statistic follows a Student’s t-distribution under the null hypothesis on numeric values extracted from the aggregated documents. In practice, this will tell you if the difference between two population means are statistically significant and did not occur by chance alone.

`t_test`度量聚集，它进行其中检验统计如下从聚集文档中提取数值在零假设下一个学生t分布的统计假设检验。在实践中，这将告诉您两个总体均值之间的差异是否具有统计显着性并且并非偶然发生。

###  Syntax

A `t_test` aggregation looks like this in isolation:

单独的`t_test`聚合看起来像这样：

```js
{
  "t_test": {
    "a": "value_before",
    "b": "value_after",
    "type": "paired"
  }
}
```

Assuming that we have a record of node start up times before and after upgrade, let’s look at a t-test to see if upgrade affected the node start up time in a meaningful way.

假设我们有升级前后节点启动时间的记录，让我们看一下 t 检验，看看升级是否以有意义的方式影响了节点启动时间。

```console
GET node_upgrade/_search
{
  "size": 0,
  "aggs": {
    "startup_time_ttest": {
      "t_test": {
        "a": { "field": "startup_time_before" },  (1)
        "b": { "field": "startup_time_after" },   (2)
        "type": "paired"                          (3)
      }
    }
  }
}
```

1. The field `startup_time_before` must be a numeric field.

    该字段`startup_time_before`必须是数字字段。

2. The field `startup_time_after` must be a numeric field.

   该字段`startup_time_after`必须是数字字段。

3.  Since we have data from the same nodes, we are using paired t-test.

    由于我们有来自相同节点的数据，我们使用配对 t 检验。

The response will return the p-value or probability value for the test. It is the probability of obtaining results at least as extreme as the result processed by the aggregation, assuming that the null hypothesis is correct (which means there is no difference between population means). Smaller p-value means the null hypothesis is more likely to be incorrect and population means are indeed different.

响应将返回测试的 p 值或概率值。它是获得至少与聚合处理的结果一样极端的结果的概率，假设原假设是正确的（这意味着总体均值之间没有差异）。较小的 p 值意味着零假设更有可能是不正确的，并且总体均值确实不同。

```console-result
{
  ...

 "aggregations": {
    "startup_time_ttest": {
      "value": 0.1914368843365979 (1)
    }
  }
}
```

1. The p-value.

    p 值。

###  T-Test Types

The `t_test` aggregation supports unpaired and paired two-sample t-tests. The type of the test can be specified using the `type` parameter:

`t_test`聚合支持未配对和配对的双样本t检验。可以使用`type`参数指定测试的类型：

- **`"type": "paired"`**

  performs paired t-test

  执行配对 t 检验

- **`"type": "homoscedastic"`**

  performs two-sample equal variance test

  执行两样本等方差检验

- **`"type": "heteroscedastic"`**

  performs two-sample unequal variance test (this is default)

  执行两样本不等方差检验（这是默认值）

###  Filters

It is also possible to run unpaired t-test on different sets of records using filters. For example, if we want to test the difference of startup times before upgrade between two different groups of nodes, we use the same field `startup_time_before` by separate groups of nodes using terms filters on the group name field:

也可以使用过滤器对不同的记录集运行未配对的 t 检验。例如，如果我们想测试两个不同节点组之间升级前启动时间的差异，我们使用`startup_time_before`组名称字段上的术语过滤器，通过不同的节点组使用相同的字段：

```console
GET node_upgrade/_search
{
  "size": 0,
  "aggs": {
    "startup_time_ttest": {
      "t_test": {
        "a": {
          "field": "startup_time_before",         (1) 
          "filter": {
            "term": {
              "group": "A"                        (2)
            }
          }
        },
        "b": {
          "field": "startup_time_before",         (3)
          "filter": {
            "term": {
              "group": "B"                        (4)
            }
          }
        },
        "type": "heteroscedastic"                 (5)
      }
    }
  }
}
```

1. The field `startup_time_before` must be a numeric field.

   该字段`startup_time_before`必须是数字字段。

2.  Any query that separates two groups can be used here.

   可以在此处使用任何分隔两个组的查询。

3.  We are using the same field

   我们正在使用相同的字段

4. but we are using different filters.

   但我们使用了不同的过滤器。

5.  Since we have data from different nodes, we cannot use paired t-test.

    由于我们有来自不同节点的数据，我们不能使用配对 t 检验。

```console-result
{
  ...

 "aggregations": {
    "startup_time_ttest": {
      "value": 0.2981858007281437 (1)
    }
  }
}
```

1.  The p-value.

   p 值。

Populations don’t have to be in the same index. If data sets are located in different indices, the term filter on the [`_index`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-index-field.html) field can be used to select populations.

人口不必在同一索引中。如果数据集位于不同的索引中，[`_index`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-index-field.html)则可以使用字段上的术语过滤器来选择总体。

###  Script

If you need to run the `t_test` on values that aren’t represented cleanly by a field you should, run the aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html). For example, if you want to adjust out load times for the before values:

如果您需要运行`t_test`未由字段清晰表示的on 值，请在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。例如，如果要调整之前值的加载时间：

```console
GET node_upgrade/_search
{
  "size": 0,
  "runtime_mappings": {
    "startup_time_before.adjusted": {
      "type": "long",
      "script": {
        "source": "emit(doc['startup_time_before'].value - params.adjustment)",
        "params": {
          "adjustment": 10
        }
      }
    }
  },
  "aggs": {
    "startup_time_ttest": {
      "t_test": {
        "a": {
          "field": "startup_time_before.adjusted"
        },
        "b": {
          "field": "startup_time_after"
        },
        "type": "paired"
      }
    }
  }
}
```

#  Top hits aggregation

A `top_hits` metric aggregator keeps track of the most relevant document being aggregated. This aggregator is intended to be used as a sub aggregator, so that the top matching documents can be aggregated per bucket.

一个`top_hits`指标聚合不断被聚合跟踪最相关的文档。此聚合器旨在用作子聚合器，以便可以按桶聚合顶部匹配的文档。

> TIP: We do not recommend using `top_hits` as a top-level aggregation. If you want to group search hits, use the [`collapse`](https://www.elastic.co/guide/en/elasticsearch/reference/master/collapse-search-results.html) parameter instead.
>
> 我们不建议将其`top_hits`用作顶级聚合。如果您想对搜索匹配进行分组，请改用[`collapse`](https://www.elastic.co/guide/en/elasticsearch/reference/master/collapse-search-results.html) 参数。

The `top_hits` aggregator can effectively be used to group result sets by certain fields via a bucket aggregator. One or more bucket aggregators determines by which properties a result set get sliced into.

该`top_hits`聚合器可以有效地通过某些字段经由铲斗聚合器用于将结果集。一个或多个存储桶聚合器确定将结果集划分为哪些属性。

###  Options

- `from` - The offset from the first result you want to fetch.

  `from` - 与您要获取的第一个结果的偏移量。

- `size` - The maximum number of top matching hits to return per bucket. By default the top three matching hits are returned.

  `size`- 每个存储桶返回的最大匹配命中数。默认情况下，返回前三个匹配的匹配项。

- `sort` - How the top matching hits should be sorted. By default the hits are sorted by the score of the main query.

  `sort`- 应如何对最匹配的命中进行排序。默认情况下，命中按主查询的分数排序。

###  Supported per hit features

The top_hits aggregation returns regular search hits, because of this many per hit features can be supported:

top_hits 聚合返回常规搜索命中，因为可以支持许多 per hit 功能：

- [Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html)
- [Explain](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-explain)
- [Named queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html#named-queries)
- [Search fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#search-fields-param)
- [Source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#source-filtering)
- [Stored fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#stored-fields)
- [Script fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#script-fields)
- [Doc value fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#docvalue-fields)
- [Include versions](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-version)
- [Include Sequence Numbers and Primary Terms](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-seq-no-primary-term)

> IMPORTANT: If you **only** need `docvalue_fields`, `size`, and `sort` then [Top metrics](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-top-metrics.html) might be a more efficient choice than the Top Hits Aggregation.
>
> 如果你**只**需要`docvalue_fields`，`size`以及`sort`再 [前指标](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-top-metrics.html)可能是一个更有效的选择，而不是在排名靠前的聚集。

`top_hits` does not support the [`rescore`](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#rescore) parameter. Query rescoring applies only to search hits, not aggregation results. To change the scores used by aggregations, use a [`function_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html) or [`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html) query.

`top_hits`不支持该[`rescore`](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#rescore)参数。查询重新评分仅适用于搜索命中，而不适用于聚合结果。要更改聚合使用的分数，请使用[`function_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html)或 [`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html)查询。

###  Example

In the following example we group the sales by type and per type we show the last sale. For each sale only the date and price fields are being included in the source.

在下面的示例中，我们按类型对销售进行分组，并按类型显示最后一次销售。对于每次销售，来源中仅包含日期和价格字段。

```console
POST /sales/_search?size=0
{
  "aggs": {
    "top_tags": {
      "terms": {
        "field": "type",
        "size": 3
      },
      "aggs": {
        "top_sales_hits": {
          "top_hits": {
            "sort": [
              {
                "date": {
                  "order": "desc"
                }
              }
            ],
            "_source": {
              "includes": [ "date", "price" ]
            },
            "size": 1
          }
        }
      }
    }
  }
}
```

Possible response:

```console-result
{
  ...
  "aggregations": {
    "top_tags": {
       "doc_count_error_upper_bound": 0,
       "sum_other_doc_count": 0,
       "buckets": [
          {
             "key": "hat",
             "doc_count": 3,
             "top_sales_hits": {
                "hits": {
                   "total" : {
                       "value": 3,
                       "relation": "eq"
                   },
                   "max_score": null,
                   "hits": [
                      {
                         "_index": "sales",
                         "_id": "AVnNBmauCQpcRyxw6ChK",
                         "_source": {
                            "date": "2015/03/01 00:00:00",
                            "price": 200
                         },
                         "sort": [
                            1425168000000
                         ],
                         "_score": null
                      }
                   ]
                }
             }
          },
          {
             "key": "t-shirt",
             "doc_count": 3,
             "top_sales_hits": {
                "hits": {
                   "total" : {
                       "value": 3,
                       "relation": "eq"
                   },
                   "max_score": null,
                   "hits": [
                      {
                         "_index": "sales",
                         "_id": "AVnNBmauCQpcRyxw6ChL",
                         "_source": {
                            "date": "2015/03/01 00:00:00",
                            "price": 175
                         },
                         "sort": [
                            1425168000000
                         ],
                         "_score": null
                      }
                   ]
                }
             }
          },
          {
             "key": "bag",
             "doc_count": 1,
             "top_sales_hits": {
                "hits": {
                   "total" : {
                       "value": 1,
                       "relation": "eq"
                   },
                   "max_score": null,
                   "hits": [
                      {
                         "_index": "sales",
                         "_id": "AVnNBmatCQpcRyxw6ChH",
                         "_source": {
                            "date": "2015/01/01 00:00:00",
                            "price": 150
                         },
                         "sort": [
                            1420070400000
                         ],
                         "_score": null
                      }
                   ]
                }
             }
          }
       ]
    }
  }
}
```

###  Field collapse example

Field collapsing or result grouping is a feature that logically groups a result set into groups and per group returns top documents. The ordering of the groups is determined by the relevancy of the first document in a group. In Elasticsearch this can be implemented via a bucket aggregator that wraps a `top_hits` aggregator as sub-aggregator.

字段折叠或结果分组是一种功能，可将结果集按逻辑分组，并按组返回顶级文档。组的顺序由组中第一个文档的相关性决定。在 Elasticsearch 中，这可以通过将聚合器包装`top_hits`为子聚合器的存储桶聚合器来实现。

In the example below we search across crawled webpages. For each webpage we store the body and the domain the webpage belong to. By defining a `terms` aggregator on the `domain` field we group the result set of webpages by domain. The `top_hits` aggregator is then defined as sub-aggregator, so that the top matching hits are collected per bucket.

在下面的示例中，我们搜索已抓取的网页。对于每个网页，我们存储网页所属的正文和域。通过`terms`在`domain`字段上定义聚合器，我们按域对网页的结果集进行分组。`top_hits`然后将 聚合器定义为子聚合器，以便每个桶收集顶部匹配的命中。

Also a `max` aggregator is defined which is used by the `terms` aggregator’s order feature to return the buckets by relevancy order of the most relevant document in a bucket.

还`max`定义了一个聚合器，`terms`聚合器的订单功能使用它来按存储桶中最相关文档的相关性顺序返回存储桶。

```console
POST /sales/_search
{
  "query": {
    "match": {
      "body": "elections"
    }
  },
  "aggs": {
    "top_sites": {
      "terms": {
        "field": "domain",
        "order": {
          "top_hit": "desc"
        }
      },
      "aggs": {
        "top_tags_hits": {
          "top_hits": {}
        },
        "top_hit" : {
          "max": {
            "script": {
              "source": "_score"
            }
          }
        }
      }
    }
  }
}
```

At the moment the `max` (or `min`) aggregator is needed to make sure the buckets from the `terms` aggregator are ordered according to the score of the most relevant webpage per domain. Unfortunately the `top_hits` aggregator can’t be used in the `order` option of the `terms` aggregator yet.

目前需要`max`（或`min`）聚合器来确保`terms`聚合器中的存储桶根据每个域最相关网页的分数进行排序。不幸的是，`top_hits`聚合器还不能用于聚合器的`order`选项中`terms`。

###  top_hits support in a nested or reverse_nested aggregator

If the `top_hits` aggregator is wrapped in a `nested` or `reverse_nested` aggregator then nested hits are being returned. Nested hits are in a sense hidden mini documents that are part of regular document where in the mapping a nested field type has been configured. The `top_hits` aggregator has the ability to un-hide these documents if it is wrapped in a `nested` or `reverse_nested` aggregator. Read more about nested in the [nested type mapping](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html).

如果`top_hits`聚合器包含在 a`nested`或`reverse_nested`聚合器中，则将返回嵌套命中。嵌套命中在某种意义上是隐藏的迷你文档，它们是常规文档的一部分，其中在映射中配置了嵌套字段类型。如果`top_hits`聚合器包含在`nested` 或`reverse_nested`聚合器中，则聚合器能够取消隐藏这些文档。阅读有关嵌套在[嵌套类型映射](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html)中的更多信息。

If nested type has been configured a single document is actually indexed as multiple Lucene documents and they share the same id. In order to determine the identity of a nested hit there is more needed than just the id, so that is why nested hits also include their nested identity. The nested identity is kept under the `_nested` field in the search hit and includes the array field and the offset in the array field the nested hit belongs to. The offset is zero based.

如果配置了嵌套类型，则单个文档实际上被索引为多个 Lucene 文档，并且它们共享相同的 id。为了确定嵌套命中的身份，需要的不仅仅是 id，这就是嵌套命中还包括其嵌套身份的原因。嵌套标识保留在`_nested`搜索命中中的字段下，并包括数组字段和嵌套命中所属的数组字段中的偏移量。偏移量是基于零的。

Let’s see how it works with a real sample. Considering the following mapping:

让我们看看它是如何处理真实样本的。考虑以下映射：

```console
PUT /sales
{
  "mappings": {
    "properties": {
      "tags": { "type": "keyword" },
      "comments": {                           (1)
        "type": "nested",
        "properties": {
          "username": { "type": "keyword" },
          "comment": { "type": "text" }
        }
      }
    }
  }
}
```

1. The `comments` is an array that holds nested documents under the `product` object.

   `comments`是保持下嵌套文档的阵列`product`对象。

And some documents:

```console
PUT /sales/_doc/1?refresh
{
  "tags": [ "car", "auto" ],
  "comments": [
    { "username": "baddriver007", "comment": "This car could have better brakes" },
    { "username": "dr_who", "comment": "Where's the autopilot? Can't find it" },
    { "username": "ilovemotorbikes", "comment": "This car has two extra wheels" }
  ]
}
```

It’s now possible to execute the following `top_hits` aggregation (wrapped in a `nested` aggregation):

现在可以执行以下`top_hits`聚合（包装在`nested`聚合中）：

```console
POST /sales/_search
{
  "query": {
    "term": { "tags": "car" }
  },
  "aggs": {
    "by_sale": {
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "by_user": {
          "terms": {
            "field": "comments.username",
            "size": 1
          },
          "aggs": {
            "by_nested": {
              "top_hits": {}
            }
          }
        }
      }
    }
  }
}
```

Top hits response snippet with a nested hit, which resides in the first slot of array field `comments`:

带有嵌套命中的热门命中响应片段，位于数组字段的第一个槽中`comments`：

```console-result
{
  ...
  "aggregations": {
    "by_sale": {
      "by_user": {
        "buckets": [
          {
            "key": "baddriver007",
            "doc_count": 1,
            "by_nested": {
              "hits": {
                "total" : {
                   "value": 1,
                   "relation": "eq"
                },
                "max_score": 0.3616575,
                "hits": [
                  {
                    "_index": "sales",
                    "_id": "1",
                    "_nested": {
                      "field": "comments",  (1)
                      "offset": 0 (2)
                    },
                    "_score": 0.3616575,
                    "_source": {
                      "comment": "This car could have better brakes", 
                      "username": "baddriver007"
                    }
                  }
                ]
              }
            }
          }
          ...
        ]
      }
    }
  }
}
```

1.  Name of the array field containing the nested hit

    包含嵌套命中的数组字段的名称

2. Position if the nested hit in the containing array

    如果嵌套命中在包含数组中的位置

3. Source of the nested hit

   嵌套命中的来源

If `_source` is requested then just the part of the source of the nested object is returned, not the entire source of the document. Also stored fields on the **nested** inner object level are accessible via `top_hits` aggregator residing in a `nested` or `reverse_nested` aggregator.

如果`_source`被请求，则只返回嵌套对象源的一部分，而不是文档的整个源。**嵌套**内部对象级别上存储的字段也可以通过`top_hits`驻留在`nested`或`reverse_nested`聚合器中的聚合器访问。

Only nested hits will have a `_nested` field in the hit, non nested (regular) hits will not have a `_nested` field.

只有嵌套命中才会`_nested`在命中中有字段，非嵌套（常规）命中不会有`_nested`字段。

The information in `_nested` can also be used to parse the original source somewhere else if `_source` isn’t enabled.

如果`_source`未启用，`_nested`中的信息也可用于解析其他地方的原始源。

If there are multiple levels of nested object types defined in mappings then the `_nested` information can also be hierarchical in order to express the identity of nested hits that are two layers deep or more.

如果在映射中定义了多个级别的嵌套对象类型，那么`_nested`信息也可以是分层的，以便表达两层或更深的嵌套命中的身份。

In the example below a nested hit resides in the first slot of the field `nested_grand_child_field` which then resides in the second slow of the `nested_child_field` field:

在下面的示例中，嵌套命中位于字段的第一个槽中`nested_grand_child_field`，然后位于该字段的第二个慢速中`nested_child_field`：

```js
...
"hits": {
 "total" : {
     "value": 2565,
     "relation": "eq"
 },
 "max_score": 1,
 "hits": [
   {
     "_index": "a",
     "_id": "1",
     "_score": 1,
     "_nested" : {
       "field" : "nested_child_field",
       "offset" : 1,
       "_nested" : {
         "field" : "nested_grand_child_field",
         "offset" : 0
       }
     }
     "_source": ...
   },
   ...
 ]
}
...
```

#  Top metrics aggregation

The `top_metrics` aggregation selects metrics from the document with the largest or smallest "sort" value. For example, this gets the value of the `m` field on the document with the largest value of `s`:

在`top_metrics`从与最大或最小的“排序”有价票证聚合选择指标。例如，这将获取`m`文档上具有最大值的字段的值`s`：

```console
POST /test/_bulk?refresh
{"index": {}}
{"s": 1, "m": 3.1415}
{"index": {}}
{"s": 2, "m": 1.0}
{"index": {}}
{"s": 3, "m": 2.71828}
POST /test/_search?filter_path=aggregations
{
  "aggs": {
    "tm": {
      "top_metrics": {
        "metrics": {"field": "m"},
        "sort": {"s": "desc"}
      }
    }
  }
}
```

Which returns:

```js
{
  "aggregations": {
    "tm": {
      "top": [ {"sort": [3], "metrics": {"m": 2.718280076980591 } } ]
    }
  }
}
```

`top_metrics` is fairly similar to [`top_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-top-hits-aggregation.html) in spirit but because it is more limited it is able to do its job using less memory and is often faster.

`top_metrics`与[`top_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-top-hits-aggregation.html) 在精神上非常相似，但因为它更有限，它能够使用更少的内存来完成它的工作，并且通常更快。

### `sort`

The `sort` field in the metric request functions exactly the same as the `sort` field in the [search](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html) request except:

`sort`指标请求中的字段功能`sort`与[搜索](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html)请求中的字段 完全相同，除了：

- It can’t be used on [binary](https://www.elastic.co/guide/en/elasticsearch/reference/master/binary.html), [flattened](https://www.elastic.co/guide/en/elasticsearch/reference/master/flattened.html), [ip](https://www.elastic.co/guide/en/elasticsearch/reference/master/ip.html), [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html), or [text](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) fields.

  它不能用于[binary](https://www.elastic.co/guide/en/elasticsearch/reference/master/binary.html)、[flattened](https://www.elastic.co/guide/en/elasticsearch/reference/master/flattened.html)、[ip](https://www.elastic.co/guide/en/elasticsearch/reference/master/ip.html)、 [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html)或[text](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html)字段。

- It only supports a single sort value so which document wins ties is not specified.

  它仅支持单个排序值，因此未指定哪个文档获胜。

The metrics that the aggregation returns is the first hit that would be returned by the search request. So,

聚合返回的指标是搜索请求将返回的第一个命中。所以，

- **`"sort": {"s": "desc"}`**

  gets metrics from the document with the highest `s`

  从具有最高的文档中获取指标 `s`

- **`"sort": {"s": "asc"}`**

  gets the metrics from the document with the lowest `s`

  从文档中获取最低的指标 `s`

- **`"sort": {"_geo_distance": {"location": "35.7796, -78.6382"}}`**

  gets metrics from the documents with `location` **closest** to `35.7796, -78.6382`

  获得指标的文件与`location` **最接近**于`35.7796, -78.6382`

- **`"sort": "_score"`**

  gets metrics from the document with the highest score

  从得分最高的文档中获取指标

### `metrics`

`metrics` selects the fields of the "top" document to return. You can request a single metric with something like `"metrics": {"field": "m"}` or multiple metrics by requesting a list of metrics like `"metrics": [{"field": "m"}, {"field": "i"}`.

`metrics`选择要返回的“顶级”文档的字段。您可以`"metrics": {"field": "m"}`通过请求指标列表来请求具有类似指标或多个指标的单个指标`"metrics": [{"field": "m"}, {"field": "i"}`。

`metrics.field` supports the following field types:

`metrics.field` 支持以下字段类型：

- [`boolean`](https://www.elastic.co/guide/en/elasticsearch/reference/master/boolean.html)
- [`ip`](https://www.elastic.co/guide/en/elasticsearch/reference/master/ip.html)
- [keywords](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html)
- [numbers](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)

Except for keywords, [runtime fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) for corresponding types are also supported. `metrics.field` doesn’t support fields with [array values](https://www.elastic.co/guide/en/elasticsearch/reference/master/array.html). A `top_metric` aggregation on array values may return inconsistent results.

除了关键字，还支持对应类型的[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)。`metrics.field`不支持带有[数组值的](https://www.elastic.co/guide/en/elasticsearch/reference/master/array.html)字段。`top_metric`数组值的 聚合可能会返回不一致的结果。

The following example runs a `top_metrics` aggregation on several field types.

以下示例`top_metrics`对多种字段类型运行聚合。

```console
PUT /test
{
  "mappings": {
    "properties": {
      "d": {"type": "date"}
    }
  }
}
POST /test/_bulk?refresh
{"index": {}}
{"s": 1, "m": 3.1415, "i": 1, "d": "2020-01-01T00:12:12Z", "t": "cat"}
{"index": {}}
{"s": 2, "m": 1.0, "i": 6, "d": "2020-01-02T00:12:12Z", "t": "dog"}
{"index": {}}
{"s": 3, "m": 2.71828, "i": -12, "d": "2019-12-31T00:12:12Z", "t": "chicken"}
POST /test/_search?filter_path=aggregations
{
  "aggs": {
    "tm": {
      "top_metrics": {
        "metrics": [
          {"field": "m"},
          {"field": "i"},
          {"field": "d"},
          {"field": "t.keyword"}
        ],
        "sort": {"s": "desc"}
      }
    }
  }
}
```

Which returns:

```js
{
  "aggregations": {
    "tm": {
      "top": [ {
        "sort": [3],
        "metrics": {
          "m": 2.718280076980591,
          "i": -12,
          "d": "2019-12-31T00:12:12.000Z",
          "t.keyword": "chicken"
        }
      } ]
    }
  }
}
```

### `size`

`top_metrics` can return the top few document’s worth of metrics using the size parameter:

`top_metrics`可以使用 size 参数返回前几个文档的度量值：

```console
POST /test/_bulk?refresh
{"index": {}}
{"s": 1, "m": 3.1415}
{"index": {}}
{"s": 2, "m": 1.0}
{"index": {}}
{"s": 3, "m": 2.71828}
POST /test/_search?filter_path=aggregations
{
  "aggs": {
    "tm": {
      "top_metrics": {
        "metrics": {"field": "m"},
        "sort": {"s": "desc"},
        "size": 3
      }
    }
  }
}
```

Which returns:

```js
{
  "aggregations": {
    "tm": {
      "top": [
        {"sort": [3], "metrics": {"m": 2.718280076980591 } },
        {"sort": [2], "metrics": {"m": 1.0 } },
        {"sort": [1], "metrics": {"m": 3.1414999961853027 } }
      ]
    }
  }
}
```

The default `size` is 1. The maximum default size is `10` because the aggregation’s working storage is "dense", meaning we allocate `size` slots for every bucket. `10` is a **very** conservative default maximum and you can raise it if you need to by changing the `top_metrics_max_size` index setting. But know that large sizes can take a fair bit of memory, especially if they are inside of an aggregation which makes many buckes like a large [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-top-metrics.html#search-aggregations-metrics-top-metrics-example-terms). If you till want to raise it, use something like:

默认`size`值为 1。最大默认大小是`10`因为聚合的工作存储是“密集的”，这意味着我们`size`为每个存储桶分配插槽。`10` 是一个**非常**保守的默认最大值，如果需要，您可以通过更改`top_metrics_max_size`索引设置来提高它。但是要知道大尺寸会占用相当多的内存，特别是如果它们位于聚合内部，这会像大[术语聚合](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-top-metrics.html#search-aggregations-metrics-top-metrics-example-terms)一样产生很多费用 。如果您想提高它，请使用以下内容：

```console
PUT /test/_settings
{
  "top_metrics_max_size": 100
}
```

> NOTE: If `size` is more than `1` the `top_metrics` aggregation can’t be the **target** of a sort.



如果`size`超过`1`该`top_metrics`集合不能成为**目标**排序的。

### Examples

####  Use with terms

This aggregation should be quite useful inside of [`terms`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) aggregation, to, say, find the last value reported by each server.

这种聚合在[`terms`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) 聚合内部应该非常有用，例如，找到每个服务器报告的最后一个值。

```console
PUT /node
{
  "mappings": {
    "properties": {
      "ip": {"type": "ip"},
      "date": {"type": "date"}
    }
  }
}
POST /node/_bulk?refresh
{"index": {}}
{"ip": "192.168.0.1", "date": "2020-01-01T01:01:01", "m": 1}
{"index": {}}
{"ip": "192.168.0.1", "date": "2020-01-01T02:01:01", "m": 2}
{"index": {}}
{"ip": "192.168.0.2", "date": "2020-01-01T02:01:01", "m": 3}
POST /node/_search?filter_path=aggregations
{
  "aggs": {
    "ip": {
      "terms": {
        "field": "ip"
      },
      "aggs": {
        "tm": {
          "top_metrics": {
            "metrics": {"field": "m"},
            "sort": {"date": "desc"}
          }
        }
      }
    }
  }
}
```

Which returns:

```js
{
  "aggregations": {
    "ip": {
      "buckets": [
        {
          "key": "192.168.0.1",
          "doc_count": 2,
          "tm": {
            "top": [ {"sort": ["2020-01-01T02:01:01.000Z"], "metrics": {"m": 2 } } ]
          }
        },
        {
          "key": "192.168.0.2",
          "doc_count": 1,
          "tm": {
            "top": [ {"sort": ["2020-01-01T02:01:01.000Z"], "metrics": {"m": 3 } } ]
          }
        }
      ],
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0
    }
  }
}
```

Unlike `top_hits`, you can sort buckets by the results of this metric:

与 不同`top_hits`，您可以根据此指标的结果对存储桶进行排序：

```console
POST /node/_search?filter_path=aggregations
{
  "aggs": {
    "ip": {
      "terms": {
        "field": "ip",
        "order": {"tm.m": "desc"}
      },
      "aggs": {
        "tm": {
          "top_metrics": {
            "metrics": {"field": "m"},
            "sort": {"date": "desc"}
          }
        }
      }
    }
  }
}
```

Which returns:

```js
{
  "aggregations": {
    "ip": {
      "buckets": [
        {
          "key": "192.168.0.2",
          "doc_count": 1,
          "tm": {
            "top": [ {"sort": ["2020-01-01T02:01:01.000Z"], "metrics": {"m": 3 } } ]
          }
        },
        {
          "key": "192.168.0.1",
          "doc_count": 2,
          "tm": {
            "top": [ {"sort": ["2020-01-01T02:01:01.000Z"], "metrics": {"m": 2 } } ]
          }
        }
      ],
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0
    }
  }
}
```

####  Mixed sort types

Sorting `top_metrics` by a field that has different types across different indices producs somewhat surprising results: floating point fields are always sorted independently of whole numbered fields.

`top_metrics`按跨不同索引具有不同类型的字段排序会产生一些令人惊讶的结果：浮点字段总是独立于整数字段进行排序。

```console
POST /test/_bulk?refresh
{"index": {"_index": "test1"}}
{"s": 1, "m": 3.1415}
{"index": {"_index": "test1"}}
{"s": 2, "m": 1}
{"index": {"_index": "test2"}}
{"s": 3.1, "m": 2.71828}
POST /test*/_search?filter_path=aggregations
{
  "aggs": {
    "tm": {
      "top_metrics": {
        "metrics": {"field": "m"},
        "sort": {"s": "asc"}
      }
    }
  }
}
```

Which returns:

```js
{
  "aggregations": {
    "tm": {
      "top": [ {"sort": [3.0999999046325684], "metrics": {"m": 2.718280076980591 } } ]
    }
  }
}
```

While this is better than an error it **probably** isn’t what you were going for. While it does lose some precision, you can explicitly cast the whole number fields to floating points with something like:

虽然这比错误要好，但它**可能**不是您想要的。虽然它确实失去了一些精度，但您可以使用以下内容将整数字段显式转换为浮点数：

```console
POST /test*/_search?filter_path=aggregations
{
  "aggs": {
    "tm": {
      "top_metrics": {
        "metrics": {"field": "m"},
        "sort": {"s": {"order": "asc", "numeric_type": "double"}}
      }
    }
  }
}
```

Which returns the much more expected:

哪个返回更多预期：

```js
{
  "aggregations": {
    "tm": {
      "top": [ {"sort": [1.0], "metrics": {"m": 3.1414999961853027 } } ]
    }
  }
}
```

#  Value count aggregation

A `single-value` metrics aggregation that counts the number of values that are extracted from the aggregated documents. These values can be extracted either from specific fields in the documents, or be generated by a provided script. Typically, this aggregator will be used in conjunction with other single-value aggregations. For example, when computing the `avg` one might be interested in the number of values the average is computed over.

一种`single-value`度量聚合，用于计算从聚合文档中提取的值的数量。这些值可以从文档中的特定字段中提取，也可以由提供的脚本生成。通常，此聚合器将与其他单值聚合结合使用。例如，在计算时，`avg` 一个人可能对计算平均值的值的数量感兴趣。

`value_count` does not de-duplicate values, so even if a field has duplicates each value will be counted individually.

`value_count` 不会去重复值，所以即使一个字段有重复，每个值也会被单独计算。

```console
POST /sales/_search?size=0
{
  "aggs" : {
    "types_count" : { "value_count" : { "field" : "type" } }
  }
}
```

Response:

```console-result
{
  ...
  "aggregations": {
    "types_count": {
      "value": 7
    }
  }
}
```

The name of the aggregation (`types_count` above) also serves as the key by which the aggregation result can be retrieved from the returned response.

聚合的名称（`types_count`如上）也用作可以从返回的响应中检索聚合结果的键。

###  Script

If you need to count something more complex than the values in a single field you should run the aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html).

如果您需要计算比单个字段中的值更复杂的值，您应该在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。

```console
POST /sales/_search
{
  "size": 0,
  "runtime_mappings": {
    "tags": {
      "type": "keyword",
      "script": """
        emit(doc['type'].value);
        if (doc['promoted'].value) {
          emit('hot');
        }
      """
    }
  },
  "aggs": {
    "tags_count": {
      "value_count": {
        "field": "tags"
      }
    }
  }
}
```

###  Histogram fields

When the `value_count` aggregation is computed on [histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html), the result of the aggregation is the sum of all numbers in the `counts` array of the histogram.

当`value_count`在[histogram fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/histogram.html)上计算聚合时，聚合的结果是`counts`直方图数组中所有数字的总和。

For example, for the following index that stores pre-aggregated histograms with latency metrics for different networks:

例如，对于以下存储具有不同网络延迟指标的预聚合直方图的索引：

```console
PUT metrics_index/_doc/1
{
  "network.name" : "net-1",
  "latency_histo" : {
      "values" : [0.1, 0.2, 0.3, 0.4, 0.5],
      "counts" : [3, 7, 23, 12, 6] 
   }
}

PUT metrics_index/_doc/2
{
  "network.name" : "net-2",
  "latency_histo" : {
      "values" :  [0.1, 0.2, 0.3, 0.4, 0.5],
      "counts" : [8, 17, 8, 7, 6] 
   }
}

POST /metrics_index/_search?size=0
{
  "aggs": {
    "total_requests": {
      "value_count": { "field": "latency_histo" }
    }
  }
}
```

For each histogram field the `value_count` aggregation will sum all numbers in the `counts` array <1>. Eventually, it will add all values for all histograms and return the following result:

对于每个直方图字段，`value_count`聚合将对`counts`数组 <1>中的所有数字求和。最终，它会将所有直方图的所有值相加并返回以下结果：

```console-result
{
  ...
  "aggregations": {
    "total_requests": {
      "value": 97
    }
  }
}
```

#  Weighted avg aggregation

A `single-value` metrics aggregation that computes the weighted average of numeric values that are extracted from the aggregated documents. These values can be extracted either from specific numeric fields in the documents.

一种`single-value`度量聚合，用于计算从聚合文档中提取的数值的加权平均值。这些值可以从文档中的特定数字字段中提取。

When calculating a regular average, each datapoint has an equal "weight" … it contributes equally to the final value. Weighted averages, on the other hand, weight each datapoint differently. The amount that each datapoint contributes to the final value is extracted from the document.

在计算常规平均值时，每个数据点都有相同的“权重”……它对最终值的贡献相同。另一方面，加权平均值对每个数据点的权重不同。每个数据点对最终值的贡献是从文档中提取的。

As a formula, a weighted average is the `∑(value * weight) / ∑(weight)`

作为一个公式，加权平均是 `∑(value * weight) / ∑(weight)`

A regular average can be thought of as a weighted average where every value has an implicit weight of `1`.

可以将常规平均值视为加权平均值，其中每个值的隐含权重为`1`。

**Table 46. `weighted_avg` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `value`        | The configuration for the field or script that provides the values<br />提供值的字段或脚本的配置 | Required |               |
| `weight`       | The configuration for the field or script that provides the weights<br />提供权重的字段或脚本的配置 | Required |               |
| `format`       | The numeric response formatter<br />数字响应格式化程序       | Optional |               |

The `value` and `weight` objects have per-field specific configuration:

在`value`和`weight`对象有每场具体配置：



**Table 47. `value` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `field`        | The field that values should be extracted from<br />应从中提取值的字段 | Required |               |
| `missing`      | A value to use if the field is missing entirely<br />如果字段完全丢失，则使用的值 | Optional |               |



**Table 48. `weight` Parameters**

| Parameter Name | Description                                                  | Required | Default Value |
| -------------- | ------------------------------------------------------------ | -------- | ------------- |
| `field`        | The field that weights should be extracted from<br />应从中提取权重的字段 | Required |               |
| `missing`      | A weight to use if the field is missing entirely<br />如果该字段完全丢失，则使用权重 | Optional |               |

###  Examples

If our documents have a `"grade"` field that holds a 0-100 numeric score, and a `"weight"` field which holds an arbitrary numeric weight, we can calculate the weighted average using:

如果我们的文档有一个`"grade"`包含 0-100 数字分数的`"weight"`字段和一个包含任意数字权重的字段，我们可以使用以下方法计算加权平均值：

```console
POST /exams/_search
{
  "size": 0,
  "aggs": {
    "weighted_grade": {
      "weighted_avg": {
        "value": {
          "field": "grade"
        },
        "weight": {
          "field": "weight"
        }
      }
    }
  }
}
```

Which yields a response like:

```console-result
{
  ...
  "aggregations": {
    "weighted_grade": {
      "value": 70.0
    }
  }
}
```

While multiple values-per-field are allowed, only one weight is allowed. If the aggregation encounters a document that has more than one weight (e.g. the weight field is a multi-valued field) it will abort the search. If you have this situation, you should build a [Runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-weight-avg-aggregation.html#search-aggregations-metrics-weight-avg-aggregation-runtime-field) to combine those values into a single weight.

虽然每个字段允许有多个值，但只允许一个权重。如果聚合遇到具有多个权重的文档（例如，权重字段是多值字段），它将中止搜索。如果您遇到这种情况，您应该构建一个[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-weight-avg-aggregation.html#search-aggregations-metrics-weight-avg-aggregation-runtime-field) 以将这些值组合成一个权重。

This single weight will be applied independently to each value extracted from the `value` field.

这个单一的权重将独立应用于从`value`字段中提取的每个值。

This example show how a single document with multiple values will be averaged with a single weight:

此示例显示了如何使用单个权重对具有多个值的单个文档进行平均：

```console
POST /exams/_doc?refresh
{
  "grade": [1, 2, 3],
  "weight": 2
}

POST /exams/_search
{
  "size": 0,
  "aggs": {
    "weighted_grade": {
      "weighted_avg": {
        "value": {
          "field": "grade"
        },
        "weight": {
          "field": "weight"
        }
      }
    }
  }
}
```

The three values (`1`, `2`, and `3`) will be included as independent values, all with the weight of `2`:

三个值 ( `1`、`2`和`3`) 将作为独立值包含在内，所有值的权重均为`2`：

```console-result
{
  ...
  "aggregations": {
    "weighted_grade": {
      "value": 2.0
    }
  }
}
```

The aggregation returns `2.0` as the result, which matches what we would expect when calculating by hand: `((1*2) + (2*2) + (3*2)) / (2+2+2) == 2`

聚合`2.0`作为结果返回，这符合我们手动计算时的预期： `((1*2) + (2*2) + (3*2)) / (2+2+2) == 2`

###  Runtime field

If you have to sum or weigh values that don’t quite line up with the indexed values, run the aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html).

如果您必须对与索引值不太一致的值求和或加权，请在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合。

```console
POST /exams/_doc?refresh
{
  "grade": 100,
  "weight": [2, 3]
}
POST /exams/_doc?refresh
{
  "grade": 80,
  "weight": 3
}

POST /exams/_search?filter_path=aggregations
{
  "size": 0,
  "runtime_mappings": {
    "weight.combined": {
      "type": "double",
      "script": """
        double s = 0;
        for (double w : doc['weight']) {
          s += w;
        }
        emit(s);
      """
    }
  },
  "aggs": {
    "weighted_grade": {
      "weighted_avg": {
        "value": {
          "script": "doc.grade.value + 1"
        },
        "weight": {
          "field": "weight.combined"
        }
      }
    }
  }
}
```

Which should look like:

应该是这样的：

```console-result
{
  "aggregations": {
    "weighted_grade": {
      "value": 93.5
    }
  }
}
```

###  Missing values

The `missing` parameter defines how documents that are missing a value should be treated. The default behavior is different for `value` and `weight`:

该`missing`参数定义应如何处理缺少值的文档。默认行为是不同的`value`和`weight`：

By default, if the `value` field is missing the document is ignored and the aggregation moves on to the next document. If the `weight` field is missing, it is assumed to have a weight of `1` (like a normal average).

默认情况下，如果该`value`字段丢失，则文档将被忽略并且聚合会移动到下一个文档。如果该`weight`字段丢失，则假定其权重为`1`（如正常平均值）。

Both of these defaults can be overridden with the `missing` parameter:

这两个默认值都可以用`missing`参数覆盖：

```console
POST /exams/_search
{
  "size": 0,
  "aggs": {
    "weighted_grade": {
      "weighted_avg": {
        "value": {
          "field": "grade",
          "missing": 2
        },
        "weight": {
          "field": "weight",
          "missing": 3
        }
      }
    }
  }
}
```

