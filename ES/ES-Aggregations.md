#  Aggregations

An aggregation summarizes your data as metrics, statistics, or other analytics. Aggregations help you answer questions like:

聚合将您的数据汇总为指标、统计数据或其他分析。聚合可帮助您回答以下问题：

- What’s the average load time for my website?
- Who are my most valuable customers based on transaction volume?
- What would be considered a large file on my network?
- How many products are in each product category?

- 我的网站的平均加载时间是多少？
- 根据交易量，谁是我最有价值的客户？
- 什么会被视为我的网络上的大文件？
- 每个产品类别中有多少产品？

Elasticsearch organizes aggregations into three categories:

Elasticsearch 将聚合组织为三类：

- [Metric](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics.html) aggregations that calculate metrics, such as a sum or average, from field values.
- [Bucket](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket.html) aggregations that group documents into buckets, also called bins, based on field values, ranges, or other criteria.
- [Pipeline](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html) aggregations that take input from other aggregations instead of documents or fields.
- [度量](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics.html)聚集该计算度量，诸如总和或平均值，从字段值。
- 基于字段值、范围或其他条件将文档分组到桶中的[桶](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket.html)聚合，也称为桶。
- 从其他聚合而不是文档或字段获取输入的[管道](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html)聚合。

## Run an aggregation

You can run aggregations as part of a [search](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-your-data.html) by specifying the [search API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)'s `aggs` parameter. The following search runs a [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) on `my-field`:

您可以通过指定[搜索 API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)的参数将聚合作为[搜索](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-your-data.html)的一部分运行。以下搜索运行 [方面聚集](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html)在 ：`aggs``my-field`

```console
GET /my-index-000001/_search
{
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      }
    }
  }
}
```

Aggregation results are in the response’s `aggregations` object:

聚合结果在响应的`aggregations`对象中：

```console-result
{
  "took": 78,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 5,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [...]
  },
  "aggregations": {
    "my-agg-name": {                          (1) 
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": []
    }
  }
}
```

1.  Results for the `my-agg-name` aggregation.

   `my-agg-name`聚合的结果。

##  Change an aggregation’s scope

Use the `query` parameter to limit the documents on which an aggregation runs:

使用`query`参数来限制聚合运行的文档：

```console
GET /my-index-000001/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1d/d",
        "lt": "now/d"
      }
    }
  },
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      }
    }
  }
}
```

##  Return only aggregation results

By default, searches containing an aggregation return both search hits and aggregation results. To return only aggregation results, set `size` to `0`:

默认情况下，包含聚合的搜索会返回搜索命中和聚合结果。要仅返回聚合结果，请设置`size`为`0`：

```console
GET /my-index-000001/_search
{
  "size": 0,
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      }
    }
  }
}
```

##  Run multiple aggregations

You can specify multiple aggregations in the same request:

您可以在同一个请求中指定多个聚合：

```console
GET /my-index-000001/_search
{
  "aggs": {
    "my-first-agg-name": {
      "terms": {
        "field": "my-field"
      }
    },
    "my-second-agg-name": {
      "avg": {
        "field": "my-other-field"
      }
    }
  }
}
```

##  Run sub-aggregations

Bucket aggregations support bucket or metric sub-aggregations. For example, a terms aggregation with an [avg](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-avg-aggregation.html) sub-aggregation calculates an average value for each bucket of documents. There is no level or depth limit for nesting sub-aggregations.

桶聚合支持桶或指标子聚合。例如，带有[avg](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-avg-aggregation.html) 子聚合的术语聚合计算每个文档桶的平均值。嵌套子聚合没有级别或深度限制。

```console
GET /my-index-000001/_search
{
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      },
      "aggs": {
        "my-sub-agg-name": {
          "avg": {
            "field": "my-other-field"
          }
        }
      }
    }
  }
}
```

The response nests sub-aggregation results under their parent aggregation:

响应在其父聚合下嵌套子聚合结果：

```console-result
{
  ...
  "aggregations": {
    "my-agg-name": {                           (1)
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "foo",
          "doc_count": 5,
          "my-sub-agg-name": {                 (2)
            "value": 75.0
          }
        }
      ]
    }
  }
}
```

1.  Results for the parent aggregation, `my-agg-name`.

    父聚合的结果，`my-agg-name`。

2. Results for `my-agg-name`'s sub-aggregation, `my-sub-agg-name`.

    结果`my-agg-name`的子聚集`my-sub-agg-name`。

##  Add custom metadata

Use the `meta` object to associate custom metadata with an aggregation:

使用`meta`对象将自定义元数据与聚合相关联：

```console
GET /my-index-000001/_search
{
  "aggs": {
    "my-agg-name": {
      "terms": {
        "field": "my-field"
      },
      "meta": {
        "my-metadata-field": "foo"
      }
    }
  }
}
```

The response returns the `meta` object in place:

响应返回`meta`对象：

```console-result
{
  ...
  "aggregations": {
    "my-agg-name": {
      "meta": {
        "my-metadata-field": "foo"
      },
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": []
    }
  }
}
```

##  Return the aggregation type

By default, aggregation results include the aggregation’s name but not its type. To return the aggregation type, use the `typed_keys` query parameter.

默认情况下，聚合结果包括聚合的名称，但不包括其类型。要返回聚合类型，请使用`typed_keys`查询参数。

```console
GET /my-index-000001/_search?typed_keys
{
  "aggs": {
    "my-agg-name": {
      "histogram": {
        "field": "my-field",
        "interval": 1000
      }
    }
  }
}
```

The response returns the aggregation type as a prefix to the aggregation’s name.

响应返回聚合类型作为聚合名称的前缀。

> IMPORTANT: Some aggregations return a different aggregation type from the type in the request. For example, the terms, [significant terms](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-significantterms-aggregation.html), and [percentiles](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html) aggregations return different aggregations types depending on the data type of the aggregated field.
>
> 某些聚合返回与请求中的类型不同的聚合类型。例如，术语、 [重要术语](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-significantterms-aggregation.html)和[百分位数](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-percentile-aggregation.html) 聚合根据聚合字段的数据类型返回不同的聚合类型。

```console-result
{
  ...
  "aggregations": {
    "histogram#my-agg-name": {                 (1)
      "buckets": []
    }
  }
}
```

1. The aggregation type, `histogram`, followed by a `#` separator and the aggregation’s name, `my-agg-name`.

   聚合类型 ，`histogram`后跟`#`分隔符和聚合名称`my-agg-name`。

##  Use scripts in an aggregation

When a field doesn’t exactly match the aggregation you need, you should aggregate on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html):

当一个字段与您需要的聚合不完全匹配时，您应该在[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上聚合：

```console
GET /my-index-000001/_search?size=0
{
  "runtime_mappings": {
    "message.length": {
      "type": "long",
      "script": "emit(doc['message.keyword'].value.length())"
    }
  },
  "aggs": {
    "message_length": {
      "histogram": {
        "interval": 10,
        "field": "message.length"
      }
    }
  }
}
```

Scripts calculate field values dynamically, which adds a little overhead to the aggregation. In addition to the time spent calculating, some aggregations like [`terms`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) and [`filters`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-filters-aggregation.html) can’t use some of their optimizations with runtime fields. In total, performance costs for using a runtime field varies from aggregation to aggregation.

脚本动态计算字段值，这给聚合增加了一点开销。除了计算花费的时间之外，一些聚合喜欢[`terms`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) 并且[`filters`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-filters-aggregation.html)不能将它们的一些优化用于运行时字段。总的来说，使用运行时字段的性能成本因聚合而异。

##  Aggregation caches

For faster responses, Elasticsearch caches the results of frequently run aggregations in the [shard request cache](https://www.elastic.co/guide/en/elasticsearch/reference/master/shard-request-cache.html). To get cached results, use the same [`preference` string](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-shard-routing.html#shard-and-node-preference) for each search. If you don’t need search hits, [set `size` to `0`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations.html#return-only-agg-results) to avoid filling the cache.

为了更快的响应，Elasticsearch 将频繁运行的聚合的结果缓存在分片[请求缓存中](https://www.elastic.co/guide/en/elasticsearch/reference/master/shard-request-cache.html)。要获得缓存的结果，请对每个搜索使用相同的[`preference`字符串](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-shard-routing.html#shard-and-node-preference)。如果你不需要搜索命中，[设置`size`以`0`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations.html#return-only-agg-results)避免填充缓存。

Elasticsearch routes searches with the same preference string to the same shards. If the shards' data doesn’t change between searches, the shards return cached aggregation results.

Elasticsearch 将具有相同首选项字符串的搜索路由到相同的分片。如果分片的数据在搜索之间没有变化，分片会返回缓存的聚合结果。

## Limits for `long` values

When running aggregations, Elasticsearch uses [`double`](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) values to hold and represent numeric data. As a result, aggregations on [`long`](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) numbers greater than `253` are approximate.

在运行聚合时，Elasticsearch 使用[`double`](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)值来保存和表示数字数据。因此，对[`long`](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)大于数字的聚合是近似的。`253`

#  Bucket aggregations

Bucket aggregations don’t calculate metrics over fields like the metrics aggregations do, but instead, they create buckets of documents. Each bucket is associated with a criterion (depending on the aggregation type) which determines whether or not a document in the current context "falls" into it. In other words, the buckets effectively define document sets. In addition to the buckets themselves, the `bucket` aggregations also compute and return the number of documents that "fell into" each bucket.

桶聚合不像度量聚合那样计算字段上的度量，而是创建文档桶。每个桶都与一个标准（取决于聚合类型）相关联，该标准确定当前上下文中的文档是否“落入”其中。换句话说，桶有效地定义了文档集。除了桶本身，`bucket`聚合还计算并返回“落入”每个桶的文档数量。

Bucket aggregations, as opposed to `metrics` aggregations, can hold sub-aggregations. These sub-aggregations will be aggregated for the buckets created by their "parent" bucket aggregation.

与`metrics`聚合相反，桶聚合可以保存子聚合。这些子聚合将为其“父”存储桶聚合创建的存储桶聚合。

There are different bucket aggregators, each with a different "bucketing" strategy. Some define a single bucket, some define fixed number of multiple buckets, and others dynamically create the buckets during the aggregation process.

有不同的bucket聚合器，每个都有不同的“bucketing”策略。一些定义单个存储桶，一些定义固定数量的多个存储桶，还有一些在聚合过程中动态创建存储桶。

> NOTE: The maximum number of buckets allowed in a single response is limited by a dynamic cluster setting named [`search.max_buckets`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#search-settings-max-buckets). It defaults to 65,536. Requests that try to return more than the limit will fail with an exception.
>
> 单个响应中允许的最大存储桶数受名为 的动态集群设置的限制 [`search.max_buckets`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#search-settings-max-buckets)。默认为 65,536。尝试返回超过限制的请求将失败并抛出异常。

#  Adjacency matrix aggregation

A bucket aggregation returning a form of [adjacency matrix](https://en.wikipedia.org/wiki/Adjacency_matrix). The request provides a collection of named filter expressions, similar to the `filters` aggregation request. Each bucket in the response represents a non-empty cell in the matrix of intersecting filters.

一个桶聚合返回一种[邻接矩阵](https://en.wikipedia.org/wiki/Adjacency_matrix)的形式。该请求提供了一组命名过滤器表达式，类似于`filters`聚合请求。响应中的每个桶代表交叉过滤器矩阵中的一个非空单元格。

Given filters named `A`, `B` and `C` the response would return buckets with the following names:

由于过滤器命名`A`，`B`并`C`响应将具有以下名称返回桶：

|       | A    | B    | C    |
| ----- | ---- | ---- | ---- |
| **A** | A    | A&B  | A&C  |
| **B** |      | B    | B&C  |
| **C** |      |      | C    |

The intersecting buckets e.g `A&C` are labelled using a combination of the two filter names with a default separator of `&`. Note that the response does not also include a `C&A` bucket as this would be the same set of documents as `A&C`. The matrix is said to be *symmetric* so we only return half of it. To do this we sort the filter name strings and always use the lowest of a pair as the value to the left of the separator.

相交的桶例如`A&C`使用两个过滤器名称的组合和默认分隔符标记`&`。请注意，响应也不包含`C&A`存储桶，因为这与`A&C`. 矩阵被认为是*对称的，*所以我们只返回它的一半。为此，我们对过滤器名称字符串进行排序，并始终使用一对中最低的作为分隔符左侧的值。

### Example

The following `interactions` aggregation uses `adjacency_matrix` to determine which groups of individuals exchanged emails.

以下`interactions`聚合用于`adjacency_matrix`确定哪些个人组交换了电子邮件。

```console
PUT emails/_bulk?refresh
{ "index" : { "_id" : 1 } }
{ "accounts" : ["hillary", "sidney"]}
{ "index" : { "_id" : 2 } }
{ "accounts" : ["hillary", "donald"]}
{ "index" : { "_id" : 3 } }
{ "accounts" : ["vladimir", "donald"]}

GET emails/_search
{
  "size": 0,
  "aggs" : {
    "interactions" : {
      "adjacency_matrix" : {
        "filters" : {
          "grpA" : { "terms" : { "accounts" : ["hillary", "sidney"] }},
          "grpB" : { "terms" : { "accounts" : ["donald", "mitt"] }},
          "grpC" : { "terms" : { "accounts" : ["vladimir", "nigel"] }}
        }
      }
    }
  }
}
```

The response contains buckets with document counts for each filter and combination of filters. Buckets with no matching documents are excluded from the response.

响应包含带有每个过滤器和过滤器组合的文档计数的桶。没有匹配文档的存储桶将从响应中排除。

```console-result
{
  "took": 9,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "interactions": {
      "buckets": [
        {
          "key":"grpA",
          "doc_count": 2
        },
        {
          "key":"grpA&grpB",
          "doc_count": 1
        },
        {
          "key":"grpB",
          "doc_count": 2
        },
        {
          "key":"grpB&grpC",
          "doc_count": 1
        },
        {
          "key":"grpC",
          "doc_count": 1
        }
      ]
    }
  }
}
```

###  Parameters

**`filters`**

(Required, object) Filters used to create buckets.

（必需，对象）用于创建存储桶的过滤器。

> Properties of filters
>
>  `filters`的属性
>
> **`<filter>`**
>
> (Required, [Query DSL object](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html)) Query used to filter documents. The key is the filter name.
>
> （必需，[查询 DSL 对象](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html)）用于过滤文档的查询。关键是过滤器名称。
>
> At least one filter is required. The total number of filters cannot exceed the [`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) setting. See [Filter limits](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-adjacency-matrix-aggregation.html#adjacency-matrix-agg-filter-limits).
>
> 至少需要一个过滤器。过滤器总数不能超过 [`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) 设置。请参阅[过滤器限制](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-adjacency-matrix-aggregation.html#adjacency-matrix-agg-filter-limits)。

**`separator`**

(Optional, string) Separator used to concatenate filter names. Defaults to `&`.

（可选，字符串）用于连接过滤器名称的分隔符。默认为`&`.

###  Response body

**`key`**

(string) Filters for the bucket. If the bucket uses multiple filters, filter names are concatenated using a `separator`.

（字符串）存储桶的过滤器。如果存储桶使用多个过滤器，过滤器名称将使用`separator`.

**`document_count`**

(integer) Number of documents matching the bucket’s filters.

（整数）与存储桶过滤器匹配的文档数。

###  Usage

On its own this aggregation can provide all of the data required to create an undirected weighted graph. However, when used with child aggregations such as a `date_histogram` the results can provide the additional levels of data required to perform [dynamic network analysis](https://en.wikipedia.org/wiki/Dynamic_network_analysis) where examining interactions *over time* becomes important.

这种聚合本身可以提供创建无向加权图所需的所有数据。但是，当与子聚合（例如 a）一起使用时`date_histogram`，结果可以提供执行[动态网络分析](https://en.wikipedia.org/wiki/Dynamic_network_analysis)所需的额外数据级别， 其中*随着时间的推移*检查交互变得很重要。

### Filter limits

For N filters the matrix of buckets produced can be N²/2 which can be costly. The circuit breaker settings prevent results producing too many buckets and to avoid excessive disk seeks the `indices.query.bool.max_clause_count` setting is used to limit the number of filters.

对于 N 个过滤器，生成的桶矩阵可能是 N²/2，这可能是昂贵的。断路器设置可防止结果产生过多的存储桶并避免过多的磁盘搜索，该`indices.query.bool.max_clause_count`设置用于限制过滤器的数量。

#  Auto-interval date histogram aggregation

A multi-bucket aggregation similar to the [Date histogram](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-datehistogram-aggregation.html) except instead of providing an interval to use as the width of each bucket, a target number of buckets is provided indicating the number of buckets needed and the interval of the buckets is automatically chosen to best achieve that target. The number of buckets returned will always be less than or equal to this target number.

类似于[日期直方图](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-datehistogram-aggregation.html)的多桶聚合，除了不提供用作每个桶的宽度的间隔，而是提供了指示所需桶数的目标桶数，并自动选择桶的间隔以最好地实现那个目标。返回的桶数将始终小于或等于此目标数。

The buckets field is optional, and will default to 10 buckets if not specified.

buckets 字段是可选的，如果未指定，则默认为 10 个桶。

Requesting a target of 10 buckets.

请求 10 个存储桶的目标。

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sales_over_time": {
      "auto_date_histogram": {
        "field": "date",
        "buckets": 10
      }
    }
  }
}
```

###  Keys

Internally, a date is represented as a 64 bit number representing a timestamp in milliseconds-since-the-epoch. These timestamps are returned as the bucket `key`s. The `key_as_string` is the same timestamp converted to a formatted date string using the format specified with the `format` parameter:

在内部，日期表示为 64 位数字，表示以毫秒为单位的时间戳。这些时间戳作为桶`key`s返回 。这`key_as_string`是使用`format`参数指定的格式转换为格式化日期字符串的相同时间戳：

> TIP: If no `format` is specified, then it will use the first date [format](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html) specified in the field mapping.
>
> 如果没有`format`指定，那么它将使用字段映射中指定的第一个日期 [格式](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)。

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sales_over_time": {
      "auto_date_histogram": {
        "field": "date",
        "buckets": 5,
        "format": "yyyy-MM-dd" (1)
      }
    }
  }
}
```

1. Supports expressive date [format pattern](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html#date-format-pattern)

   支持富有表现力的日期[格式模式](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html#date-format-pattern)

Response:

```console-result
{
  ...
  "aggregations": {
    "sales_over_time": {
      "buckets": [
        {
          "key_as_string": "2015-01-01",
          "key": 1420070400000,
          "doc_count": 3
        },
        {
          "key_as_string": "2015-02-01",
          "key": 1422748800000,
          "doc_count": 2
        },
        {
          "key_as_string": "2015-03-01",
          "key": 1425168000000,
          "doc_count": 2
        }
      ],
      "interval": "1M"
    }
  }
}
```

###  Intervals

The interval of the returned buckets is selected based on the data collected by the aggregation so that the number of buckets returned is less than or equal to the number requested. The possible intervals returned are:

根据聚合收集的数据选择返回桶的间隔，使得返回桶的数量小于或等于请求的数量。返回的可能间隔是：

| seconds | In multiples of 1, 5, 10 and 30<br />1、5、10 和 30 的倍数   |
| ------- | ------------------------------------------------------------ |
| minutes | In multiples of 1, 5, 10 and 30<br />1、5、10 和 30 的倍数   |
| hours   | In multiples of 1, 3 and 12<br />1、3、12的倍数              |
| days    | In multiples of 1, and 7<br />1 和 7 的倍数                  |
| months  | In multiples of 1, and 3<br />1 和 3 的倍数                  |
| years   | In multiples of 1, 5, 10, 20, 50 and 100<br />1、5、10、20、50和100的倍数 |

In the worst case, where the number of daily buckets are too many for the requested number of buckets, the number of buckets returned will be 1/7th of the number of buckets requested.

在最坏的情况下，如果每天的桶数对于请求的桶数来说太多，返回的桶数将是请求的桶数的 1/7。

###  Time Zone

Date-times are stored in Elasticsearch in UTC. By default, all bucketing and rounding is also done in UTC. The `time_zone` parameter can be used to indicate that bucketing should use a different time zone.

日期时间以 UTC 格式存储在 Elasticsearch 中。默认情况下，所有分桶和舍入也是在 UTC 中完成的。该`time_zone`参数可用于指示分桶应使用不同的时区。

Time zones may either be specified as an ISO 8601 UTC offset (e.g. `+01:00` or `-08:00`) or as a timezone id, an identifier used in the TZ database like `America/Los_Angeles`.

时区可以指定为 ISO 8601 UTC 偏移量（例如`+01:00`或 `-08:00`），也可以指定为时区 ID，即 TZ 数据库中使用的标识符，如 `America/Los_Angeles`。

Consider the following example:

```console
PUT my-index-000001/_doc/1?refresh
{
  "date": "2015-10-01T00:30:00Z"
}

PUT my-index-000001/_doc/2?refresh
{
  "date": "2015-10-01T01:30:00Z"
}

PUT my-index-000001/_doc/3?refresh
{
  "date": "2015-10-01T02:30:00Z"
}

GET my-index-000001/_search?size=0
{
  "aggs": {
    "by_day": {
      "auto_date_histogram": {
        "field":     "date",
        "buckets" : 3
      }
    }
  }
}
```

UTC is used if no time zone is specified, three 1-hour buckets are returned starting at midnight UTC on 1 October 2015:

如果未指定时区，则使用 UTC，从 2015 年 10 月 1 日午夜 UTC 开始返回三个 1 小时时段：

```console-result
{
  ...
  "aggregations": {
    "by_day": {
      "buckets": [
        {
          "key_as_string": "2015-10-01T00:00:00.000Z",
          "key": 1443657600000,
          "doc_count": 1
        },
        {
          "key_as_string": "2015-10-01T01:00:00.000Z",
          "key": 1443661200000,
          "doc_count": 1
        },
        {
          "key_as_string": "2015-10-01T02:00:00.000Z",
          "key": 1443664800000,
          "doc_count": 1
        }
      ],
      "interval": "1h"
    }
  }
}
```

If a `time_zone` of `-01:00` is specified, then midnight starts at one hour before midnight UTC:

如果`time_zone`的`-01:00`指定，然后在午夜开始于1小时午夜UTC之前：

```console
GET my-index-000001/_search?size=0
{
  "aggs": {
    "by_day": {
      "auto_date_histogram": {
        "field":     "date",
        "buckets" : 3,
        "time_zone": "-01:00"
      }
    }
  }
}
```

Now three 1-hour buckets are still returned but the first bucket starts at 11:00pm on 30 September 2015 since that is the local time for the bucket in the specified time zone.

现在仍然返回三个 1 小时时段，但第一个时段在 2015 年 9 月 30 日晚上 11:00 开始，因为这是指定时区中时段的本地时间。

```console-result
{
  ...
  "aggregations": {
    "by_day": {
      "buckets": [
        {
          "key_as_string": "2015-09-30T23:00:00.000-01:00", (1)
          "key": 1443657600000,
          "doc_count": 1
        },
        {
          "key_as_string": "2015-10-01T00:00:00.000-01:00",
          "key": 1443661200000,
          "doc_count": 1
        },
        {
          "key_as_string": "2015-10-01T01:00:00.000-01:00",
          "key": 1443664800000,
          "doc_count": 1
        }
      ],
      "interval": "1h"
    }
  }
}
```

1.  The `key_as_string` value represents midnight on each day in the specified time zone.

   该`key_as_string`值表示指定时区中每天的午夜。

> WARNING: When using time zones that follow DST (daylight savings time) changes, buckets close to the moment when those changes happen can have slightly different sizes than neighbouring buckets. For example, consider a DST start in the `CET` time zone: on 27 March 2016 at 2am, clocks were turned forward 1 hour to 3am local time. If the result of the aggregation was daily buckets, the bucket covering that day will only hold data for 23 hours instead of the usual 24 hours for other buckets. The same is true for shorter intervals like e.g. 12h. Here, we will have only a 11h bucket on the morning of 27 March when the DST shift happens.
>
> 当使用遵循 DST（夏令时）更改的时区时，接近这些更改发生时刻的存储桶的大小可能与相邻存储桶的大小略有不同。例如，考虑在`CET`时区开始夏令时：2016 年 3 月 27 日凌晨 2 点，时钟向前调 1 小时至当地时间凌晨 3 点。如果聚合的结果是每天的桶，那么覆盖当天的桶只会保存 23 小时的数据，而不是其他桶通常的 24 小时。对于较短的间隔（例如 12 小时）也是如此。在这里，我们将在 3 月 27 日上午发生夏令时转换时只有 11 小时的存储桶。

###  Minimum Interval parameter

The `minimum_interval` allows the caller to specify the minimum rounding interval that should be used. This can make the collection process more efficient, as the aggregation will not attempt to round at any interval lower than `minimum_interval`.

`minimum_interval`允许调用者指定最小舍入间隔应该被使用。这可以使收集过程更有效，因为聚合不会尝试以低于 的任何间隔进行舍入`minimum_interval`。

The accepted units for `minimum_interval` are:

接受的单位`minimum_interval`是：

- year
- month
- day
- hour
- minute
- second

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sale_date": {
      "auto_date_histogram": {
        "field": "date",
        "buckets": 10,
        "minimum_interval": "minute"
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
    "sale_date": {
      "auto_date_histogram": {
        "field": "date",
        "buckets": 10,
        "missing": "2000/01/01" (1)
      }
    }
  }
}
```

1.  Documents without a value in the `publish_date` field will fall into the same bucket as documents that have the value `2000-01-01`.

   该`publish_date`字段中没有值的文档将与具有该值的文档落入同一个桶中`2000-01-01`。

#  Children aggregation

A special single bucket aggregation that selects child documents that have the specified type, as defined in a [`join` field](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html).

一种特殊的单桶聚合，它选择具有指定类型的子文档，如[`join`字段中](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)所定义。

This aggregation has a single option:

此聚合有一个选项：

- `type` - The child type that should be selected.

  `type` - 应该选择的子类型。

For example, let’s say we have an index of questions and answers. The answer type has the following `join` field in the mapping:

例如，假设我们有一个问题和答案的索引。答案类型`join`在映射中具有以下字段：

```console
PUT child_example
{
  "mappings": {
    "properties": {
      "join": {
        "type": "join",
        "relations": {
          "question": "answer"
        }
      }
    }
  }
}
```

The `question` document contain a tag field and the `answer` documents contain an owner field. With the `children` aggregation the tag buckets can be mapped to the owner buckets in a single request even though the two fields exist in two different kinds of documents.

该`question`文件包含标签字段和`answer`文档包含所有者场。通过`children` 聚合，即使两个字段存在于两种不同类型的文档中，标签桶也可以在单个请求中映射到所有者桶。

An example of a question document:

问题文档示例：

```console
PUT child_example/_doc/1
{
  "join": {
    "name": "question"
  },
  "body": "<p>I have Windows 2003 server and i bought a new Windows 2008 server...",
  "title": "Whats the best way to file transfer my site from server to a newer one?",
  "tags": [
    "windows-server-2003",
    "windows-server-2008",
    "file-transfer"
  ]
}
```

Examples of `answer` documents:

`answer`文件示例：

```console
PUT child_example/_doc/2?routing=1
{
  "join": {
    "name": "answer",
    "parent": "1"
  },
  "owner": {
    "location": "Norfolk, United Kingdom",
    "display_name": "Sam",
    "id": 48
  },
  "body": "<p>Unfortunately you're pretty much limited to FTP...",
  "creation_date": "2009-05-04T13:45:37.030"
}

PUT child_example/_doc/3?routing=1&refresh
{
  "join": {
    "name": "answer",
    "parent": "1"
  },
  "owner": {
    "location": "Norfolk, United Kingdom",
    "display_name": "Troll",
    "id": 49
  },
  "body": "<p>Use Linux...",
  "creation_date": "2009-05-05T13:45:37.030"
}
```

The following request can be built that connects the two together:

可以构建以下请求将两者连接在一起：

```console
POST child_example/_search?size=0
{
  "aggs": {
    "top-tags": {
      "terms": {
        "field": "tags.keyword",
        "size": 10
      },
      "aggs": {
        "to-answers": {
          "children": {
            "type" : "answer" (1)
          },
          "aggs": {
            "top-names": {
              "terms": {
                "field": "owner.display_name.keyword",
                "size": 10
              }
            }
          }
        }
      }
    }
  }
}
```

1. The `type` points to type / mapping with the name `answer`.

    所述`type`点输入/映射的名称`answer`。

The above example returns the top question tags and per tag the top answer owners.

上面的示例返回最热门的问题标签，每个标签返回最热门的答案所有者。

Possible response:

可能的响应：

```console-result
{
  "took": 25,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "top-tags": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "file-transfer",
          "doc_count": 1, 
          "to-answers": {
            "doc_count": 2, 
            "top-names": {
              "doc_count_error_upper_bound": 0,
              "sum_other_doc_count": 0,
              "buckets": [
                {
                  "key": "Sam",
                  "doc_count": 1
                },
                {
                  "key": "Troll",
                  "doc_count": 1
                }
              ]
            }
          }
        },
        {
          "key": "windows-server-2003",
          "doc_count": 1, 
          "to-answers": {
            "doc_count": 2, 
            "top-names": {
              "doc_count_error_upper_bound": 0,
              "sum_other_doc_count": 0,
              "buckets": [
                {
                  "key": "Sam",
                  "doc_count": 1
                },
                {
                  "key": "Troll",
                  "doc_count": 1
                }
              ]
            }
          }
        },
        {
          "key": "windows-server-2008",
          "doc_count": 1, (1)
          "to-answers": {
            "doc_count": 2, (2)
            "top-names": {
              "doc_count_error_upper_bound": 0,
              "sum_other_doc_count": 0,
              "buckets": [
                {
                  "key": "Sam",
                  "doc_count": 1
                },
                {
                  "key": "Troll",
                  "doc_count": 1
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

1. The number of question documents with the tag `file-transfer`, `windows-server-2003`, etc.

   与标签问题的文件的数量`file-transfer`，`windows-server-2003`等等。

2.  The number of answer documents that are related to question documents with the tag `file-transfer`, `windows-server-2003`, etc.

   这涉及到问题的文件与标签回答文档的数量`file-transfer`，`windows-server-2003`等等。

# Composite aggregation

A multi-bucket aggregation that creates composite buckets from different sources.

一种多桶聚合，可从不同来源创建复合桶。

Unlike the other `multi-bucket` aggregations, you can use the `composite` aggregation to paginate **all** buckets from a multi-level aggregation efficiently. This aggregation provides a way to stream **all** buckets of a specific aggregation, similar to what [scroll](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results) does for documents.

与其他`multi-bucket`聚合不同，您可以使用聚合有效地对多级聚合中的**所有**存储桶`composite` 进行分页。这种聚合提供了一种流式传输特定聚合的**所有**桶的方法，类似于 [滚动](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results)对文档的作用。

The composite buckets are built from the combinations of the values extracted/created for each document and each combination is considered as a composite bucket.

复合桶是根据为每个文档提取/创建的值的组合构建的，每个组合都被视为一个复合桶。

For example, consider the following document:

例如，考虑以下文档：

```js
{
  "keyword": ["foo", "bar"],
  "number": [23, 65, 76]
}
```

Using `keyword` and `number` as source fields for the aggregation results in the following composite buckets:

使用`keyword`和`number`作为聚合的源字段会产生以下复合存储桶：

```js
{ "keyword": "foo", "number": 23 }
{ "keyword": "foo", "number": 65 }
{ "keyword": "foo", "number": 76 }
{ "keyword": "bar", "number": 23 }
{ "keyword": "bar", "number": 65 }
{ "keyword": "bar", "number": 76 }
```

###  Value sources

The `sources` parameter defines the source fields to use when building composite buckets. The order that the `sources` are defined controls the order that the keys are returned.

该`sources`参数定义了构建复合存储桶时要使用的源字段。该订单`sources`被定义控件返回键的顺序。

> NOTE: You must use a unique name when defining `sources`.
>
> 定义时必须使用唯一的名称`sources`。

The `sources` parameter can be any of the following types:

该`sources`参数可以是下列任何类型：

- [Terms](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html#_terms)

  [词语](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html#_terms)

- [Histogram](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html#_histogram)

  [直方图](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html#_histogram)

- [Date histogram](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html#_date_histogram)

  [日期直方图](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html#_date_histogram)

- [GeoTile grid](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html#_geotile_grid)

  [GeoTile 网格](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html#_geotile_grid)

####  Terms（词语）

The `terms` value source is equivalent to a simple `terms` aggregation. The values are extracted from a field exactly like the `terms` aggregation.

`terms`值源相当于一个简单的`terms`聚集。这些值是从与`terms`聚合完全一样的字段中提取的。

Example:

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "product": { "terms": { "field": "product" } } }
        ]
      }
    }
  }
}
```

Like the `terms` aggregation, it’s possible to use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) to create values for the composite buckets:

与`terms`聚合一样，可以使用 [运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)为复合存储桶创建值：

```console
GET /_search
{
  "runtime_mappings": {
    "day_of_week": {
      "type": "keyword",
      "script": """
        emit(doc['timestamp'].value.dayOfWeekEnum
          .getDisplayName(TextStyle.FULL, Locale.ROOT))
      """
    }
  },
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          {
            "dow": {
              "terms": { "field": "day_of_week" }
            }
          }
        ]
      }
    }
  }
}
```

####  Histogram（直方图）

The `histogram` value source can be applied on numeric values to build fixed size interval over the values. The `interval` parameter defines how the numeric values should be transformed. For instance an `interval` set to 5 will translate any numeric values to its closest interval, a value of `101` would be translated to `100` which is the key for the interval between 100 and 105.

`histogram`值源可以在数值上的值被施加到生成的固定大小的时间间隔。该`interval`参数定义应如何转换数值。例如，`interval`设置为 5 会将任何数值转换为其最接近的区间，`101`将转换为`100`100 和 105 之间区间的键的值。

Example:

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "histo": { "histogram": { "field": "price", "interval": 5 } } }
        ]
      }
    }
  }
}
```

Like the `histogram` aggregation it’s possible to use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) to create values for the composite buckets:

与`histogram`聚合一样，可以使用 [运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)为复合存储桶创建值：

```console
GET /_search
{
  "runtime_mappings": {
    "price.discounted": {
      "type": "double",
      "script": """
        double price = doc['price'].value;
        if (doc['product'].value == 'mad max') {
          price *= 0.8;
        }
        emit(price);
      """
    }
  },
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          {
            "price": {
              "histogram": {
                "interval": 5,
                "field": "price.discounted"
              }
            }
          }
        ]
      }
    }
  }
}
```

#### Date histogram（日期直方图）

The `date_histogram` is similar to the `histogram` value source except that the interval is specified by date/time expression:

`date_histogram`是类似于`histogram`不同之处在于由日期/时间表达式指定的时间间隔值来源：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d" } } }
        ]
      }
    }
  }
}
```

The example above creates an interval per day and translates all `timestamp` values to the start of its closest intervals. Available expressions for interval: `year`, `quarter`, `month`, `week`, `day`, `hour`, `minute`, `second`

上面的示例每天创建一个间隔，并将所有`timestamp`值转换为其最近间隔的开始。可用的区间表达式：`year`, `quarter`, `month`, `week`, `day`, `hour`, `minute`,`second`

Time values can also be specified via abbreviations supported by [time units](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units) parsing. Note that fractional time values are not supported, but you can address this by shifting to another time unit (e.g., `1.5h` could instead be specified as `90m`).

时间值也可以通过[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units)解析支持的缩写来指定。请注意，不支持小数时间值，但您可以通过转换到另一个时间单位来解决这个问题（例如，`1.5h`可以改为指定为`90m`）。

**Format**

Internally, a date is represented as a 64 bit number representing a timestamp in milliseconds-since-the-epoch. These timestamps are returned as the bucket keys. It is possible to return a formatted date string instead using the format specified with the format parameter:

在内部，日期表示为 64 位数字，表示以毫秒为单位的时间戳。这些时间戳作为存储桶键返回。可以使用 format 参数指定的格式返回格式化的日期字符串：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          {
            "date": {
              "date_histogram": {
                "field": "timestamp",
                "calendar_interval": "1d",
                "format": "yyyy-MM-dd"         (1)
              }
            }
          }
        ]
      }
    }
  }
}
```

1. Supports expressive date [format pattern](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html#date-format-pattern)

   支持富有表现力的日期[格式模式](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html#date-format-pattern)

**Time Zone**

Date-times are stored in Elasticsearch in UTC. By default, all bucketing and rounding is also done in UTC. The `time_zone` parameter can be used to indicate that bucketing should use a different time zone.

日期时间以 UTC 格式存储在 Elasticsearch 中。默认情况下，所有分桶和舍入也是在 UTC 中完成的。该`time_zone`参数可用于指示分桶应使用不同的时区。

Time zones may either be specified as an ISO 8601 UTC offset (e.g. `+01:00` or `-08:00`) or as a timezone id, an identifier used in the TZ database like `America/Los_Angeles`.

时区可以指定为 ISO 8601 UTC 偏移量（例如`+01:00`或 `-08:00`），也可以指定为时区 ID，即 TZ 数据库中使用的标识符，如 `America/Los_Angeles`。

**Offset**

Use the `offset` parameter to change the start value of each bucket by the specified positive (`+`) or negative offset (`-`) duration, such as `1h` for an hour, or `1d` for a day. See [Time units](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units) for more possible time duration options.

使用该`offset`参数按指定的正 ( `+`) 或负偏移 ( `-`) 持续时间更改每个存储区的起始值，例如`1h`一小时或`1d`一天。有关更多可能的持续时间选项，请参阅[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units)。

For example, when using an interval of `day`, each bucket runs from midnight to midnight. Setting the `offset` parameter to `+6h` changes each bucket to run from 6am to 6am:

例如，当使用间隔为 时`day`，每个桶从午夜运行到午夜。将`offset`参数设置为`+6h`将每个存储桶从早上 6 点运行到早上 6 点：

```console
PUT my-index-000001/_doc/1?refresh
{
  "date": "2015-10-01T05:30:00Z"
}

PUT my-index-000001/_doc/2?refresh
{
  "date": "2015-10-01T06:30:00Z"
}

GET my-index-000001/_search?size=0
{
  "aggs": {
    "my_buckets": {
      "composite" : {
        "sources" : [
          {
            "date": {
              "date_histogram" : {
                "field": "date",
                "calendar_interval": "day",
                "offset": "+6h",
                "format": "iso8601"
              }
            }
          }
        ]
      }
    }
  }
}
```

Instead of a single bucket starting at midnight, the above request groups the documents into buckets starting at 6am:

上面的请求不是从午夜开始的单个存储桶，而是从早上 6 点开始将文档分组到存储桶中：

```console-result
{
  ...
  "aggregations": {
    "my_buckets": {
      "after_key": { "date": "2015-10-01T06:00:00.000Z" },
      "buckets": [
        {
          "key": { "date": "2015-09-30T06:00:00.000Z" },
          "doc_count": 1
        },
        {
          "key": { "date": "2015-10-01T06:00:00.000Z" },
          "doc_count": 1
        }
      ]
    }
  }
}
```

> NOTE: The start `offset` of each bucket is calculated after `time_zone` adjustments have been made.
>
> `offset`在进行`time_zone` 调整后计算每个桶的开始。

#### GeoTile grid（GeoTile网格）

The `geotile_grid` value source works on `geo_point` fields and groups points into buckets that represent cells in a grid. The resulting grid can be sparse and only contains cells that have matching data. Each cell corresponds to a [map tile](https://en.wikipedia.org/wiki/Tiled_web_map) as used by many online map sites. Each cell is labeled using a "{zoom}/{x}/{y}" format, where zoom is equal to the user-specified precision.

`geotile_grid`值源适用于`geo_point`字段和组分成代表细胞在网格桶。生成的网格可以是稀疏的，并且只包含具有匹配数据的单元格。每个单元格对应于许多在线地图站点使用的 [地图图块](https://en.wikipedia.org/wiki/Tiled_web_map)。每个单元格都使用“{zoom}/{x}/{y}”格式进行标记，其中缩放等于用户指定的精度。

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "tile": { "geotile_grid": { "field": "location", "precision": 8 } } }
        ]
      }
    }
  }
}
```

**Precision**

The highest-precision geotile of length 29 produces cells that cover less than 10cm by 10cm of land. This precision is uniquely suited for composite aggregations as each tile does not have to be generated and loaded in memory.

长度为 29 的最高精度的土工瓦产生的细胞覆盖面积小于 10 厘米乘以 10 厘米的土地。这种精度非常适合复合聚合，因为不必在内存中生成和加载每个图块。

See [Zoom level documentation](https://wiki.openstreetmap.org/wiki/Zoom_levels) on how precision (zoom) correlates to size on the ground. Precision for this aggregation can be between 0 and 29, inclusive.

请参阅[缩放级别文档](https://wiki.openstreetmap.org/wiki/Zoom_levels) ，了解精度（缩放）与地面尺寸的相关性。此聚合的精度可以介于 0 和 29 之间，包括 0 和 29。

**Bounding box filtering**

The geotile source can optionally be constrained to a specific geo bounding box, which reduces the range of tiles used. These bounds are useful when only a specific part of a geographical area needs high precision tiling.

可以选择将 geotile 源限制为特定的地理边界框，从而减少使用的 tile 范围。当只有地理区域的特定部分需要高精度平铺时，这些边界很有用。

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          {
            "tile": {
              "geotile_grid": {
                "field": "location",
                "precision": 22,
                "bounds": {
                  "top_left": "52.4, 4.9",
                  "bottom_right": "52.3, 5.0"
                }
              }
            }
          }
        ]
      }
    }
  }
}
```

####  Mixing different value sources

The `sources` parameter accepts an array of value sources. It is possible to mix different value sources to create composite buckets. For example:

该`sources`参数接受一组值源。可以混合不同的价值来源来创建复合桶。例如：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d" } } },
          { "product": { "terms": { "field": "product" } } }
        ]
      }
    }
  }
}
```

This will create composite buckets from the values created by two value sources, a `date_histogram` and a `terms`. Each bucket is composed of two values, one for each value source defined in the aggregation. Any type of combinations is allowed and the order in the array is preserved in the composite buckets.

这将从两个值源 a`date_histogram`和 a创建的值创建复合桶`terms`。每个存储桶由两个值组成，一个用于聚合中定义的每个值源。允许任何类型的组合，并且数组中的顺序保留在复合桶中。

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "shop": { "terms": { "field": "shop" } } },
          { "product": { "terms": { "field": "product" } } },
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d" } } }
        ]
      }
    }
  }
}
```

###  Order（排序）

By default the composite buckets are sorted by their natural ordering. Values are sorted in ascending order of their values. When multiple value sources are requested, the ordering is done per value source, the first value of the composite bucket is compared to the first value of the other composite bucket and if they are equals the next values in the composite bucket are used for tie-breaking. This means that the composite bucket `[foo, 100]` is considered smaller than `[foobar, 0]` because `foo` is considered smaller than `foobar`. It is possible to define the direction of the sort for each value source by setting `order` to `asc` (default value) or `desc` (descending order) directly in the value source definition. For example:

默认情况下，复合桶按其自然顺序排序。值按其值的升序排序。当请求多个值源时，对每个值源进行排序，将复合桶的第一个值与另一个复合桶的第一个值进行比较，如果它们相等，则复合桶中的下一个值用于绑定打破。这意味着复合桶 `[foo, 100]`被认为小于 ，`[foobar, 0]`因为`foo`被认为小于`foobar`。可以通过在值源定义中直接设置`order`为`asc`（默认值）或`desc`（降序）来定义每个值源的排序方向。例如：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d", "order": "desc" } } },
          { "product": { "terms": { "field": "product", "order": "asc" } } }
        ]
      }
    }
  }
}
```

... will sort the composite bucket in descending order when comparing values from the `date_histogram` source and in ascending order when comparing values from the `terms` source.

... 将在比较`date_histogram`源中的值时按降序对复合桶进行排序，在比较源中的值时按升序对复合桶进行排序`terms`。

###  Missing bucket（缺少存储桶）

By default documents without a value for a given source are ignored. It is possible to include them in the response by setting `missing_bucket` to `true` (defaults to `false`):

默认情况下，没有给定源值的文档将被忽略。可以通过设置`missing_bucket`为 `true`（默认为`false`）将它们包含在响应中：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "product_name": { "terms": { "field": "product", "missing_bucket": true } } }
        ]
      }
    }
  }
}
```

In the example above the source `product_name` will emit an explicit `null` value for documents without a value for the field `product`. The `order` specified in the source dictates whether the `null` values should rank first (ascending order, `asc`) or last (descending order, `desc`).

在上面的示例中，源`product_name`将为`null`没有字段值的文档发出显式值`product`。所述`order`源使然指定的是否`null`值应当排名第一（升序，`asc`）或最后（降序，`desc`）。

### Size

The `size` parameter can be set to define how many composite buckets should be returned. Each composite bucket is considered as a single bucket, so setting a size of 10 will return the first 10 composite buckets created from the value sources. The response contains the values for each composite bucket in an array containing the values extracted from each value source. Defaults to `10`.

`size`可以设置该参数来定义应该返回多少个复合桶。每个复合桶都被视为一个单一的桶，因此将大小设置为 10 将返回从值源创建的前 10 个复合桶。响应包含数组中每个复合桶的值，该数组包含从每个值源提取的值。默认为`10`.

###  Pagination

If the number of composite buckets is too high (or unknown) to be returned in a single response it is possible to split the retrieval in multiple requests. Since the composite buckets are flat by nature, the requested `size` is exactly the number of composite buckets that will be returned in the response (assuming that they are at least `size` composite buckets to return). If all composite buckets should be retrieved it is preferable to use a small size (`100` or `1000` for instance) and then use the `after` parameter to retrieve the next results. For example:

如果复合桶的数量太多（或未知）而无法在单个响应中返回，则可以将检索拆分为多个请求。由于复合桶本质上是扁平的，请求`size`的正是将在响应中返回的复合桶的数量（假设它们至少是`size`要返回的复合桶）。如果应该检索所有复合桶，最好使用小尺寸（`100`或`1000`例如），然后使用`after`参数检索下一个结果。例如：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "size": 2,
        "sources": [
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d" } } },
          { "product": { "terms": { "field": "product" } } }
        ]
      }
    }
  }
}
```

... returns:

```console-result
{
  ...
  "aggregations": {
    "my_buckets": {
      "after_key": {
        "date": 1494288000000,
        "product": "mad max"
      },
      "buckets": [
        {
          "key": {
            "date": 1494201600000,
            "product": "rocky"
          },
          "doc_count": 1
        },
        {
          "key": {
            "date": 1494288000000,
            "product": "mad max"
          },
          "doc_count": 2
        }
      ]
    }
  }
}
```

To get the next set of buckets, resend the same aggregation with the `after` parameter set to the `after_key` value returned in the response. For example, this request uses the `after_key` value provided in the previous response:

要获取下一组存储桶，请重新发送相同的聚合，并将`after` 参数设置为`after_key`响应中返回的值。例如，此请求使用`after_key`先前响应中提供的值：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "size": 2,
        "sources": [
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d", "order": "desc" } } },
          { "product": { "terms": { "field": "product", "order": "asc" } } }
        ],
        "after": { "date": 1494288000000, "product": "mad max" } (1)
      }
    }
  }
}
```

1.  Should restrict the aggregation to buckets that sort **after** the provided values.

   应将聚合限制为在提供的值**之后**排序的存储桶。

> NOTE: The `after_key` is **usually** the key to the last bucket returned in the response, but that isn’t guaranteed. Always use the returned `after_key` instead of derriving it from the buckets.
>
> `after_key`是**通常**的关键，在响应中返回的最后一个桶，但不能保证。始终使用返回的`after_key`而不是从存储桶中获取它。

### Early termination（提前终止）

For optimal performance the [index sort](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules-index-sorting.html) should be set on the index so that it matches parts or fully the source order in the composite aggregation. For instance the following index sort:

为了获得最佳性能，应在[索引](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules-index-sorting.html)上设置[索引排序](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules-index-sorting.html)，以便它与复合聚合中的部分或全部源顺序匹配。例如以下索引排序：

```console
PUT my-index-000001
{
  "settings": {
    "index": {
      "sort.field": [ "username", "timestamp" ],   (1)
      "sort.order": [ "asc", "desc" ]              (2)
    }
  },
  "mappings": {
    "properties": {
      "username": {
        "type": "keyword",
        "doc_values": true
      },
      "timestamp": {
        "type": "date"
      }
    }
  }
}
```

1. This index is sorted by `username` first then by `timestamp`.

   该索引按`username`first 然后按 排序`timestamp`。

2.  … in ascending order for the `username` field and in descending order for the `timestamp` field.

    ...`username`字段的升序和字段的降序`timestamp`。

   1. could be used to optimize these composite aggregations:

      可用于优化这些复合聚合：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "user_name": { "terms": { "field": "user_name" } } }     (1)
        ]
      }
    }
  }
}
```

1. `user_name` is a prefix of the index sort and the order matches (`asc`).

   `user_name`是索引排序的前缀，并且顺序匹配 ( `asc`)。

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "user_name": { "terms": { "field": "user_name" } } }, (1)
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d", "order": "desc" } } } (2)
        ]
      }
    }
  }
}
```

1. `user_name` is a prefix of the index sort and the order matches (`asc`).

   `user_name`是索引排序的前缀，并且顺序匹配 ( `asc`)。

2. `timestamp` matches also the prefix and the order matches (`desc`).

   `timestamp`也匹配前缀和顺序匹配 ( `desc`)。

In order to optimize the early termination it is advised to set `track_total_hits` in the request to `false`. The number of total hits that match the request can be retrieved on the first request and it would be costly to compute this number on every page:

为了优化提前终止，建议`track_total_hits`在请求中设置为`false`。可以在第一个请求中检索与请求匹配的总点击次数，并且在每个页面上计算此数字的成本很高：

```console
GET /_search
{
  "size": 0,
  "track_total_hits": false,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "user_name": { "terms": { "field": "user_name" } } },
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d", "order": "desc" } } }
        ]
      }
    }
  }
}
```

Note that the order of the source is important, in the example below switching the `user_name` with the `timestamp` would deactivate the sort optimization since this configuration wouldn’t match the index sort specification. If the order of sources do not matter for your use case you can follow these simple guidelines:

请注意，源的顺序很重要，在下面的示例中`user_name`，`timestamp` 将切换为将停用排序优化，因为此配置与索引排序规范不匹配。如果源的顺序对您的用例无关紧要，您可以遵循以下简单准则：

- Put the fields with the highest cardinality first.

  将基数最高的字段放在最前面。

- Make sure that the order of the field matches the order of the index sort.

  确保字段的顺序与索引排序的顺序匹配。

- Put multi-valued fields last since they cannot be used for early termination.

  将多值字段放在最后，因为它们不能用于提前终止。

> WARNING: [
> index sort](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules-index-sorting.html) can slowdown indexing, it is very important to test index sorting with your specific use case and dataset to ensure that it matches your requirement. If it doesn’t note that `composite` aggregations will also try to early terminate on non-sorted indices if the query matches all document (`match_all` query).
>
> [
> 索引排序](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules-index-sorting.html)会减慢索引速度，使用您的特定用例和数据集测试索引排序以确保它符合您的要求非常重要。如果它没有注意到，`composite` 如果查询匹配所有文档（`match_all`查询），聚合也将尝试在未排序的索引上提前终止。

###  Sub-aggregations

Like any `multi-bucket` aggregations the `composite` aggregation can hold sub-aggregations. These sub-aggregations can be used to compute other buckets or statistics on each composite bucket created by this parent aggregation. For instance the following example computes the average value of a field per composite bucket:

像任何`multi-bucket`聚合一样，`composite`聚合可以包含子聚合。这些子聚合可用于计算此父聚合创建的每个复合存储桶的其他存储桶或统计信息。例如，以下示例计算每个复合桶的字段平均值：

```console
GET /_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          { "date": { "date_histogram": { "field": "timestamp", "calendar_interval": "1d", "order": "desc" } } },
          { "product": { "terms": { "field": "product" } } }
        ]
      },
      "aggregations": {
        "the_avg": {
          "avg": { "field": "price" }
        }
      }
    }
  }
}
```

... returns:



```console-result
{
  ...
  "aggregations": {
    "my_buckets": {
      "after_key": {
        "date": 1494201600000,
        "product": "rocky"
      },
      "buckets": [
        {
          "key": {
            "date": 1494460800000,
            "product": "apocalypse now"
          },
          "doc_count": 1,
          "the_avg": {
            "value": 10.0
          }
        },
        {
          "key": {
            "date": 1494374400000,
            "product": "mad max"
          },
          "doc_count": 1,
          "the_avg": {
            "value": 27.0
          }
        },
        {
          "key": {
            "date": 1494288000000,
            "product": "mad max"
          },
          "doc_count": 2,
          "the_avg": {
            "value": 22.5
          }
        },
        {
          "key": {
            "date": 1494201600000,
            "product": "rocky"
          },
          "doc_count": 1,
          "the_avg": {
            "value": 10.0
          }
        }
      ]
    }
  }
}
```

###  Pipeline aggregations

The composite agg is not currently compatible with pipeline aggregations, nor does it make sense in most cases. E.g. due to the paging nature of composite aggs, a single logical partition (one day for example) might be spread over multiple pages. Since pipeline aggregations are purely post-processing on the final list of buckets, running something like a derivative on a composite page could lead to inaccurate results as it is only taking into account a "partial" result on that page.

复合聚合目前与管道聚合不兼容，在大多数情况下也没有意义。例如，由于复合 aggs 的分页特性，单个逻辑分区（例如一天）可能分布在多个页面上。由于管道聚合纯粹是对最终存储桶列表进行后处理，因此在复合页面上运行类似衍生的东西可能会导致结果不准确，因为它只考虑了该页面上的“部分”结果。

Pipeline aggs that are self contained to a single bucket (such as `bucket_selector`) might be supported in the future.

`bucket_selector`将来可能会支持自包含到单个存储桶（例如）的管道聚合。

# Date histogram aggregation

This multi-bucket aggregation is similar to the normal [histogram](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-histogram-aggregation.html), but it can only be used with date or date range values. Because dates are represented internally in Elasticsearch as long values, it is possible, but not as accurate, to use the normal `histogram` on dates as well. The main difference in the two APIs is that here the interval can be specified using date/time expressions. Time-based data requires special support because time-based intervals are not always a fixed length.

这种多桶聚合类似于普通的 [histogram](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-histogram-aggregation.html)，但它只能与日期或日期范围值一起使用。因为日期在 Elasticsearch 内部表示为长值，所以也可以`histogram`在日期上使用普通值，但不是那么准确。这两个 API 的主要区别在于，这里可以使用日期/时间表达式指定间隔。基于时间的数据需要特殊支持，因为基于时间的间隔并不总是固定长度。

Like the histogram, values are rounded **down** into the closest bucket. For example, if the interval is a calendar day, `2020-01-03T07:00:01Z` is rounded to `2020-01-03T00:00:00Z`. Values are rounded as follows:

像直方图，数值四舍五入**下来**到最接近的水桶。例如，如果间隔是一个日历日，`2020-01-03T07:00:01Z`则四舍五入为 `2020-01-03T00:00:00Z`。值四舍五入如下：

```java
bucket_key = Math.floor(value / interval) * interval
```

### Calendar and fixed intervals

When configuring a date histogram aggregation, the interval can be specified in two manners: calendar-aware time intervals, and fixed time intervals.

配置日期直方图聚合时，可以通过两种方式指定时间间隔：日历感知时间间隔和固定时间间隔。

Calendar-aware intervals understand that daylight savings changes the length of specific days, months have different amounts of days, and leap seconds can be tacked onto a particular year.

日历感知间隔知道夏令时会改变特定天的长度，月份有不同的天数，闰秒可以附加到特定的年份。

Fixed intervals are, by contrast, always multiples of SI units and do not change based on calendaring context.

相比之下，固定间隔始终是 SI 单位的倍数，并且不会根据日历上下文而改变。

> NOTE:  Combined `interval` field is deprecated
>
> [7.2] Deprecated in 7.2. `interval` field is deprecatedHistorically both calendar and fixed intervals were configured in a single `interval` field, which led to confusing semantics. Specifying `1d` would be assumed as a calendar-aware time, whereas `2d` would be interpreted as fixed time. To get "one day" of fixed time, the user would need to specify the next smaller unit (in this case, `24h`).
>
> This combined behavior was often unknown to users, and even when knowledgeable about the behavior it was difficult to use and confusing.
>
> This behavior has been deprecated in favor of two new, explicit fields: `calendar_interval` and `fixed_interval`.
>
> By forcing a choice between calendar and intervals up front, the semantics of the interval are clear to the user immediately and there is no ambiguity. The old `interval` field will be removed in the future.
>
> ### 不推荐使用组合`interval`字段
>
> [ 7.2 ] 7.2 中已弃用。`interval`字段已弃用历史上，日历和固定间隔都配置在一个`interval`字段中，这导致语义混乱。指定`1d`将被假定为日历感知时间，而`2d`将被解释为固定时间。要获得“一天”的固定时间，用户需要指定下一个较小的单位（在本例中为`24h`）。
>
> 用户通常不知道这种组合行为，即使了解这种行为，也很难使用和混淆。
>
> 此行为已被弃用，取而代之的是两个新的显式字段：`calendar_interval` 和`fixed_interval`。
>
> 通过预先强制在日历和间隔之间进行选择，用户可以立即清楚间隔的语义并且没有歧义。旧`interval`字段将在将来被删除。

### Calendar intervals

Calendar-aware intervals are configured with the `calendar_interval` parameter. You can specify calendar intervals using the unit name, such as `month`, or as a single unit quantity, such as `1M`. For example, `day` and `1d` are equivalent. Multiple quantities, such as `2d`, are not supported.

日历感知间隔使用`calendar_interval`参数配置。您可以使用单位名称（例如`month`）或单个单位数量（例如 ）来指定日历间隔`1M`。例如，`day`和`1d`是等价的。`2d`不支持多个数量，例如。

The accepted calendar intervals are:

接受的日历间隔是：

- **`minute`, `1m`**

  All minutes begin at 00 seconds. One minute is the interval between 00 seconds of the first minute and 00 seconds of the following minute in the specified time zone, compensating for any intervening leap seconds, so that the number of minutes and seconds past the hour is the same at the start and end.

  所有分钟都从 00 秒开始。一分钟是指定时区中第一分钟的 00 秒和下一分钟的 00 秒之间的间隔，补偿任何中间的闰秒，以便一小时后的分钟数和秒数在开始和开始时相同结尾。

- **`hour`, `1h`**

  All hours begin at 00 minutes and 00 seconds. One hour (1h) is the interval between 00:00 minutes of the first hour and 00:00 minutes of the following hour in the specified time zone, compensating for any intervening leap seconds, so that the number of minutes and seconds past the hour is the same at the start and end.

  所有小时都从 00 分 00 秒开始。一小时 (1h) 是指定时区中第一小时的 00:00 分钟和下一小时的 00:00 分钟之间的间隔，补偿任何中间的闰秒，以便该小时过去的分钟数和秒数开头和结尾是一样的。

- **`day`, `1d`**

  All days begin at the earliest possible time, which is usually 00:00:00 (midnight). One day (1d) is the interval between the start of the day and the start of the following day in the specified time zone, compensating for any intervening time changes.

  每一天都从最早的时间开始，通常是 00:00:00（午夜）。一天 (1d) 是指定时区中一天的开始和第二天的开始之间的间隔，用于补偿任何中间的时间变化。

- **`week`, `1w`**

  One week is the interval between the start day_of_week:hour:minute:second and the same day of the week and time of the following week in the specified time zone.

  一周是指定时区中开始日_of_week:hour:minute:second 与一周的同一天和下一周的时间之间的间隔。

- **`month`, `1M`**

  One month is the interval between the start day of the month and time of day and the same day of the month and time of the following month in the specified time zone, so that the day of the month and time of day are the same at the start and end.

  一个月是指定时区中当月的开始日和时间与当月的同一天和下个月的时间之间的间隔，以便当月的日期和时间相同开始和结束。

- **`quarter`, `1q`**

  One quarter is the interval between the start day of the month and time of day and the same day of the month and time of day three months later, so that the day of the month and time of day are the same at the start and end. 

  一个季度是一个月的开始日和一天的时间与三个月后的同一天和一天的时间之间的间隔，这样开始和结束时的一个月的一天和一天的时间是相同的.

- **`year`, `1y`**

  One year is the interval between the start day of the month and time of day and the same day of the month and time of day the following year in the specified time zone, so that the date and time are the same at the start and end.

  一年是指定时区中月份的开始日期和时间与月份的同一天和下一年的时间之间的间隔，以便开始和结束的日期和时间相同.

####  Calendar interval examples

As an example, here is an aggregation requesting bucket intervals of a month in calendar time:

例如，这里是一个聚合请求日历时间中一个月的存储桶间隔：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      }
    }
  }
}
```

If you attempt to use multiples of calendar units, the aggregation will fail because only singular calendar units are supported:

如果您尝试使用多个日历单位，聚合将失败，因为仅支持单个日历单位：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "2d"
      }
    }
  }
}
```

```js
{
  "error" : {
    "root_cause" : [...],
    "type" : "x_content_parse_exception",
    "reason" : "[1:82] [date_histogram] failed to parse field [calendar_interval]",
    "caused_by" : {
      "type" : "illegal_argument_exception",
      "reason" : "The supplied interval [2d] could not be parsed as a calendar interval.",
      "stack_trace" : "java.lang.IllegalArgumentException: The supplied interval [2d] could not be parsed as a calendar interval."
    }
  }
}
```

### Fixed intervals

Fixed intervals are configured with the `fixed_interval` parameter.

固定间隔通过`fixed_interval`参数配置。

In contrast to calendar-aware intervals, fixed intervals are a fixed number of SI units and never deviate, regardless of where they fall on the calendar. One second is always composed of `1000ms`. This allows fixed intervals to be specified in any multiple of the supported units.

与日历感知间隔相反，固定间隔是固定数量的 SI 单位并且永远不会偏离，无论它们在日历上的哪个位置。一秒总是由 组成`1000ms`。这允许以任何受支持单位的倍数指定固定间隔。

However, it means fixed intervals cannot express other units such as months, since the duration of a month is not a fixed quantity. Attempting to specify a calendar interval like month or quarter will throw an exception.

但是，这意味着固定间隔不能表示其他单位，例如月，因为一个月的持续时间不是固定数量。尝试指定像月或季度这样的日历间隔将引发异常。

The accepted units for fixed intervals are:

固定间隔的可接受单位是：

- **milliseconds (`ms`)**

  A single millisecond. This is a very, very small interval.

  一毫秒。这是一个非常非常小的间隔。

- **seconds (`s`)**

  Defined as 1000 milliseconds each.

  每个定义为 1000 毫秒。

- **minutes (`m`)**

  Defined as 60 seconds each (60,000 milliseconds). All minutes begin at 00 seconds.

  定义为每个 60 秒（60,000 毫秒）。所有分钟都从 00 秒开始。

- **hours (`h`)**

  Defined as 60 minutes each (3,600,000 milliseconds). All hours begin at 00 minutes and 00 seconds.

  定义为每个 60 分钟（3,600,000 毫秒）。所有小时都从 00 分 00 秒开始。

- **days (`d`)**

  Defined as 24 hours (86,400,000 milliseconds). All days begin at the earliest possible time, which is usually 00:00:00 (midnight).

  定义为 24 小时（86,400,000 毫秒）。每一天都从最早的时间开始，通常是 00:00:00（午夜）。

####  Fixed interval examples

If we try to recreate the "month" `calendar_interval` from earlier, we can approximate that with 30 fixed days:

如果我们尝试重新创建之前的“月份” `calendar_interval`，我们可以用 30 个固定天数来近似：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "fixed_interval": "30d"
      }
    }
  }
}
```

But if we try to use a calendar unit that is not supported, such as weeks, we’ll get an exception:

但是如果我们尝试使用不受支持的日历单位，例如周，我们会得到一个异常：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "fixed_interval": "2w"
      }
    }
  }
}
```

```js
{
  "error" : {
    "root_cause" : [...],
    "type" : "x_content_parse_exception",
    "reason" : "[1:82] [date_histogram] failed to parse field [fixed_interval]",
    "caused_by" : {
      "type" : "illegal_argument_exception",
      "reason" : "failed to parse setting [date_histogram.fixedInterval] with value [2w] as a time value: unit is missing or unrecognized",
      "stack_trace" : "java.lang.IllegalArgumentException: failed to parse setting [date_histogram.fixedInterval] with value [2w] as a time value: unit is missing or unrecognized"
    }
  }
}
```

### Date histogram usage notes

In all cases, when the specified end time does not exist, the actual end time is the closest available time after the specified end.

在所有情况下，当指定的结束时间不存在时，实际结束时间为指定结束后最近的可用时间。

Widely distributed applications must also consider vagaries such as countries that start and stop daylight savings time at 12:01 A.M., so end up with one minute of Sunday followed by an additional 59 minutes of Saturday once a year, and countries that decide to move across the international date line. Situations like that can make irregular time zone offsets seem easy.

广泛分布的应用程序还必须考虑变幻莫测的情况，例如在凌晨 12:01 开始和停止夏令时的国家/地区，因此以周日的 1 分钟和每年一次的额外 59 分钟的周六结束，以及决定跨越的国家/地区国际日期变更线。像这样的情况可以使不规则的时区偏移看起来很容易。

As always, rigorous testing, especially around time-change events, will ensure that your time interval specification is what you intend it to be.

与往常一样，严格的测试，尤其是围绕时间变化事件，将确保您的时间间隔规范符合您的预期。

> WARNING: To avoid unexpected results, all connected servers and clients must sync to a reliable network time service.
>
> 为避免意外结果，所有连接的服务器和客户端都必须同步到可靠的网络时间服务。

> NOTE: Fractional time values are not supported, but you can address this by shifting to another time unit (e.g., `1.5h` could instead be specified as `90m`).
>
> 不支持小数时间值，但您可以通过转换到另一个时间单位来解决这个问题（例如，`1.5h`可以改为指定为`90m`）。

> NOTE: You can also specify time values using abbreviations supported by [time units](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units) parsing.
>
> 您还可以使用[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units)解析支持的缩写来指定时间值 。

###  Keys

Internally, a date is represented as a 64 bit number representing a timestamp in milliseconds-since-the-epoch (01/01/1970 midnight UTC). These timestamps are returned as the `key` name of the bucket. The `key_as_string` is the same timestamp converted to a formatted date string using the `format` parameter specification:

在内部，日期表示为 64 位数字，表示以毫秒为单位的时间戳（UTC 时间 01/01/1970 午夜）。这些时间戳作为`key`存储桶的名称返回。这`key_as_string`是使用`format`参数规范转换为格式化日期字符串的相同时间戳：

> TIP: If you don’t specify `format`, the first date [format](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html) specified in the field mapping is used.
>
> 如果未指定`format`，则使用字段映射中指定的第一个日期 [格式](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)。

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M",
        "format": "yyyy-MM-dd" (1)
      }
    }
  }
}
```

1. Supports expressive date [format pattern](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html#date-format-pattern)

   支持富有表现力的日期[格式模式](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html#date-format-pattern)

Response:

```console-result
{
  ...
  "aggregations": {
    "sales_over_time": {
      "buckets": [
        {
          "key_as_string": "2015-01-01",
          "key": 1420070400000,
          "doc_count": 3
        },
        {
          "key_as_string": "2015-02-01",
          "key": 1422748800000,
          "doc_count": 2
        },
        {
          "key_as_string": "2015-03-01",
          "key": 1425168000000,
          "doc_count": 2
        }
      ]
    }
  }
}
```

### Time zone

Elasticsearch stores date-times in Coordinated Universal Time (UTC). By default, all bucketing and rounding is also done in UTC. Use the `time_zone` parameter to indicate that bucketing should use a different time zone.

Elasticsearch 以协调世界时 (UTC) 存储日期时间。默认情况下，所有分桶和舍入也是在 UTC 中完成的。使用该`time_zone`参数指示分桶应使用不同的时区。

For example, if the interval is a calendar day and the time zone is `America/New_York` then `2020-01-03T01:00:01Z` is : # Converted to `2020-01-02T18:00:01` # Rounded down to `2020-01-02T00:00:00` # Then converted back to UTC to produce `2020-01-02T05:00:00:00Z` # Finally, when the bucket is turned into a string key it is printed in `America/New_York` so it’ll display as `"2020-01-02T00:00:00"`.

例如，如果间隔是一个日历日和时区是 `America/New_York`则`2020-01-03T01:00:01Z`是：＃转换为`2020-01-02T18:00:01` ＃向下舍入到`2020-01-02T00:00:00` ＃再转换回UTC，以产生`2020-01-02T05:00:00:00Z` ＃最后，当铲斗变成一个字符串键它被印刷在 `America/New_York`这样它会显示为`"2020-01-02T00:00:00"`.

It looks like:

```java
bucket_key = localToUtc(Math.floor(utcToLocal(value) / interval) * interval))
```

You can specify time zones as an ISO 8601 UTC offset (e.g. `+01:00` or `-08:00`) or as an IANA time zone ID, such as `America/Los_Angeles`.

您可以将时区指定为 ISO 8601 UTC 偏移量（例如`+01:00`或 `-08:00`）或 IANA 时区 ID，例如`America/Los_Angeles`。

Consider the following example:

```console
PUT my-index-000001/_doc/1?refresh
{
  "date": "2015-10-01T00:30:00Z"
}

PUT my-index-000001/_doc/2?refresh
{
  "date": "2015-10-01T01:30:00Z"
}

GET my-index-000001/_search?size=0
{
  "aggs": {
    "by_day": {
      "date_histogram": {
        "field":     "date",
        "calendar_interval":  "day"
      }
    }
  }
}
```

If you don’t specify a time zone, UTC is used. This would result in both of these documents being placed into the same day bucket, which starts at midnight UTC on 1 October 2015:

如果您不指定时区，则使用 UTC。这将导致这两个文档被放入同一天存储桶中，该存储桶从 2015 年 10 月 1 日午夜 UTC 开始：

```console-result
{
  ...
  "aggregations": {
    "by_day": {
      "buckets": [
        {
          "key_as_string": "2015-10-01T00:00:00.000Z",
          "key":           1443657600000,
          "doc_count":     2
        }
      ]
    }
  }
}
```

If you specify a `time_zone` of `-01:00`, midnight in that time zone is one hour before midnight UTC:

如果您指定 a `time_zone`of `-01:00`，则该时区的午夜是 UTC 午夜前一小时：

```console
GET my-index-000001/_search?size=0
{
  "aggs": {
    "by_day": {
      "date_histogram": {
        "field":     "date",
        "calendar_interval":  "day",
        "time_zone": "-01:00"
      }
    }
  }
}
```

Now the first document falls into the bucket for 30 September 2015, while the second document falls into the bucket for 1 October 2015:

现在，第一个文档属于 2015 年 9 月 30 日的存储桶，而第二个文档属于 2015 年 10 月 1 日的存储桶：

```console-result
{
  ...
  "aggregations": {
    "by_day": {
      "buckets": [
        {
          "key_as_string": "2015-09-30T00:00:00.000-01:00", (1)
          "key": 1443574800000,
          "doc_count": 1
        },
        {
          "key_as_string": "2015-10-01T00:00:00.000-01:00", (1)
          "key": 1443661200000,
          "doc_count": 1
        }
      ]
    }
  }
}
```

1. The `key_as_string` value represents midnight on each day in the specified time zone.

   该`key_as_string`值表示指定时区中每天的午夜。

> WARNING: Many time zones shift their clocks for daylight savings time. Buckets close to the moment when those changes happen can have slightly different sizes than you would expect from the `calendar_interval` or `fixed_interval`. For example, consider a DST start in the `CET` time zone: on 27 March 2016 at 2am, clocks were turned forward 1 hour to 3am local time. If you use `day` as the `calendar_interval`, the bucket covering that day will only hold data for 23 hours instead of the usual 24 hours for other buckets. The same is true for shorter intervals, like a `fixed_interval` of `12h`, where you’ll have only a 11h bucket on the morning of 27 March when the DST shift happens.
>
> 许多时区会根据夏令时调整时钟。接近发生这些变化时的存储桶的大小可能与您对`calendar_interval`or 的预期略有不同`fixed_interval`。例如，考虑在`CET`时区开始夏令时：2016 年 3 月 27 日凌晨 2 点，时钟向前调 1 小时至当地时间凌晨 3 点。如果您使用`day`as `calendar_interval`，则覆盖当天的存储桶将仅保存 23 小时的数据，而不是其他存储桶通常的 24 小时。对于较短的间隔也是如此，例如 a `fixed_interval`of `12h`，在 3 月 27 日早上发生 DST 转变时，您将只有 11 小时的存储桶。

### Offset

Use the `offset` parameter to change the start value of each bucket by the specified positive (`+`) or negative offset (`-`) duration, such as `1h` for an hour, or `1d` for a day. See [Time units](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units) for more possible time duration options.

使用该`offset`参数按指定的正 ( `+`) 或负偏移 ( `-`) 持续时间更改每个存储区的起始值，例如`1h`一小时或`1d`一天。有关更多可能的持续时间选项，请参阅[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units)。

For example, when using an interval of `day`, each bucket runs from midnight to midnight. Setting the `offset` parameter to `+6h` changes each bucket to run from 6am to 6am:

例如，当使用间隔为 时`day`，每个桶从午夜运行到午夜。将`offset`参数设置为`+6h`将每个存储桶从早上 6 点运行到早上 6 点：

```console
PUT my-index-000001/_doc/1?refresh
{
  "date": "2015-10-01T05:30:00Z"
}

PUT my-index-000001/_doc/2?refresh
{
  "date": "2015-10-01T06:30:00Z"
}

GET my-index-000001/_search?size=0
{
  "aggs": {
    "by_day": {
      "date_histogram": {
        "field":     "date",
        "calendar_interval":  "day",
        "offset":    "+6h"
      }
    }
  }
}
```

Instead of a single bucket starting at midnight, the above request groups the documents into buckets starting at 6am:

上面的请求不是从午夜开始的单个存储桶，而是从早上 6 点开始将文档分组到存储桶中：

```console-result
{
  ...
  "aggregations": {
    "by_day": {
      "buckets": [
        {
          "key_as_string": "2015-09-30T06:00:00.000Z",
          "key": 1443592800000,
          "doc_count": 1
        },
        {
          "key_as_string": "2015-10-01T06:00:00.000Z",
          "key": 1443679200000,
          "doc_count": 1
        }
      ]
    }
  }
}
```

> NOTE: The start `offset` of each bucket is calculated after `time_zone` adjustments have been made.
>
> `offset`在进行`time_zone` 调整后计算每个桶的开始。

### Keyed Response

Setting the `keyed` flag to `true` associates a unique string key with each bucket and returns the ranges as a hash rather than an array:

将`keyed`标志设置为`true`将唯一的字符串键与每个存储桶相关联，并将范围作为散列而不是数组返回：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "1M",
        "format": "yyyy-MM-dd",
        "keyed": true
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
    "sales_over_time": {
      "buckets": {
        "2015-01-01": {
          "key_as_string": "2015-01-01",
          "key": 1420070400000,
          "doc_count": 3
        },
        "2015-02-01": {
          "key_as_string": "2015-02-01",
          "key": 1422748800000,
          "doc_count": 2
        },
        "2015-03-01": {
          "key_as_string": "2015-03-01",
          "key": 1425168000000,
          "doc_count": 2
        }
      }
    }
  }
}
```

### Scripts

If the data in your documents doesn’t exactly match what you’d like to aggregate, use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) . For example, if the revenue for promoted sales should be recognized a day after the sale date:

如果文档中的数据与您想要聚合的数据不完全匹配，请使用[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)。例如，如果促销销售的收入应在销售日期后一天确认：

```console
POST /sales/_search?size=0
{
  "runtime_mappings": {
    "date.promoted_is_tomorrow": {
      "type": "date",
      "script": """
        long date = doc['date'].value.toInstant().toEpochMilli();
        if (doc['promoted'].value) {
          date += 86400;
        }
        emit(date);
      """
    }
  },
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date.promoted_is_tomorrow",
        "calendar_interval": "1M"
      }
    }
  }
}
```

### Parameters

You can control the order of the returned buckets using the `order` settings and filter the returned buckets based on a `min_doc_count` setting (by default all buckets between the first bucket that matches documents and the last one are returned). This histogram also supports the `extended_bounds` setting, which enables extending the bounds of the histogram beyond the data itself, and `hard_bounds` that limits the histogram to specified bounds. For more information, see [`Extended Bounds`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-histogram-aggregation.html#search-aggregations-bucket-histogram-aggregation-extended-bounds) and [`Hard Bounds`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-histogram-aggregation.html#search-aggregations-bucket-histogram-aggregation-hard-bounds).

您可以使用`order` 设置控制返回的存储桶的顺序，并根据设置过滤返回的存储桶`min_doc_count`（默认情况下，将返回匹配文档的第一个存储桶和最后一个存储桶之间的所有存储桶）。此直方图还支持`extended_bounds` 设置，可以将直方图的边界扩展到数据本身之外，并将`hard_bounds`直方图限制为指定的边界。有关更多信息，请参阅 [`Extended Bounds`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-histogram-aggregation.html#search-aggregations-bucket-histogram-aggregation-extended-bounds)和 [`Hard Bounds`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-histogram-aggregation.html#search-aggregations-bucket-histogram-aggregation-hard-bounds)。

#### Missing value

The `missing` parameter defines how to treat documents that are missing a value. By default, they are ignored, but it is also possible to treat them as if they have a value.

该`missing`参数定义了如何处理缺少值的文档。默认情况下，它们会被忽略，但也可以将它们视为具有值。

```console
POST /sales/_search?size=0
{
  "aggs": {
    "sale_date": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "year",
        "missing": "2000/01/01" (1)
      }
    }
  }
}
```

1. Documents without a value in the `publish_date` field will fall into the same bucket as documents that have the value `2000-01-01`.

    该`publish_date`字段中没有值的文档将与具有该值的文档落入同一个桶中`2000-01-01`。

####  Order

By default the returned buckets are sorted by their `key` ascending, but you can control the order using the `order` setting. This setting supports the same `order` functionality as [`Terms Aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order).

默认情况下，返回的桶按`key`升序排序，但您可以使用`order`设置控制顺序。此设置支持相同的`order`功能 [`Terms Aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)。

####  Using a script to aggregate by day of the week

When you need to aggregate the results by day of the week, run a `terms` aggregation on a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) that returns the day of the week:

当您需要按星期几聚合结果[时](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)，请在返回星期几`terms` 的[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)上运行聚合：

```console
POST /sales/_search?size=0
{
  "runtime_mappings": {
    "date.day_of_week": {
      "type": "keyword",
      "script": "emit(doc['date'].value.dayOfWeekEnum.getDisplayName(TextStyle.FULL, Locale.ROOT))"
    }
  },
  "aggs": {
    "day_of_week": {
      "terms": { "field": "date.day_of_week" }
    }
  }
}
```

Response:

```console-result
{
  ...
  "aggregations": {
    "day_of_week": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Sunday",
          "doc_count": 4
        },
        {
          "key": "Thursday",
          "doc_count": 3
        }
      ]
    }
  }
}
```

The response will contain all the buckets having the relative day of the week as key : 1 for Monday, 2 for Tuesday… 7 for Sunday.

响应将包含所有以一周中的相对日期为关键字的桶：1 表示星期一，2 表示星期二……7 表示星期日。

# Date range aggregation

A range aggregation that is dedicated for date values. The main difference between this aggregation and the normal [range](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-range-aggregation.html) aggregation is that the `from` and `to` values can be expressed in [Date Math](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math) expressions, and it is also possible to specify a date format by which the `from` and `to` response fields will be returned. Note that this aggregation includes the `from` value and excludes the `to` value for each range.

专用于日期值的范围聚合。此聚合与正常[范围](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-range-aggregation.html) 聚合之间的主要区别在于 ，`from`和`to`值可以用 [日期数学](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)表达式表示，并且还可以指定返回`from`和`to`响应字段的日期格式。请注意，此聚合包括每个范围的`from`值并排除该`to`值。

Example:

```console
POST /sales/_search?size=0
{
  "aggs": {
    "range": {
      "date_range": {
        "field": "date",
        "format": "MM-yyyy",
        "ranges": [
          { "to": "now-10M/M" },  (1)
          { "from": "now-10M/M" } (2)
        ]
      }
    }
  }
}
```

1. < now minus 10 months, rounded down to the start of the month.

    < 现在减 10 个月，四舍五入到月初。

2. \>= now minus 10 months, rounded down to the start of the month.

   \>= 现在减去 10 个月，四舍五入到月初。

In the example above, we created two range buckets, the first will "bucket" all documents dated prior to 10 months ago and the second will "bucket" all documents dated since 10 months ago

在上面的例子中，我们创建了两个范围存储桶，第一个将“存储”所有日期在 10 个月之前的文档，第二个将“存储”所有日期为 10 个月前的文档

Response:

```console-result
{
  ...
  "aggregations": {
    "range": {
      "buckets": [
        {
          "to": 1.4436576E12,
          "to_as_string": "10-2015",
          "doc_count": 7,
          "key": "*-10-2015"
        },
        {
          "from": 1.4436576E12,
          "from_as_string": "10-2015",
          "doc_count": 0,
          "key": "10-2015-*"
        }
      ]
    }
  }
}
```

> WARNING: If a format or date value is incomplete, the date range aggregation replaces any missing components with default values. See [Missing date components](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html#missing-date-components).
>
> 如果格式或日期值不完整，日期范围聚合会用默认值替换任何缺失的组件。请参阅 [缺少日期组件](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html#missing-date-components)。

###  Missing Values

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value. This is done by adding a set of fieldname : value mappings to specify default values per field.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。这是通过添加一组 fieldname : value 映射来指定每个字段的默认值来完成的。

```console
POST /sales/_search?size=0
{
   "aggs": {
       "range": {
           "date_range": {
               "field": "date",
               "missing": "1976/11/30",
               "ranges": [
                  {
                    "key": "Older",
                    "to": "2016/02/01"
                  }, (1)
                  {
                    "key": "Newer",
                    "from": "2016/02/01",
                    "to" : "now/d"
                  }
              ]
          }
      }
   }
}
```

1.  Documents without a value in the `date` field will be added to the "Older" bucket, as if they had a date value of "1976-11-30".

   `date`字段中没有值的文档将被添加到“旧”存储桶中，就好像它们的日期值为“1976-11-30”。

###  Date Format/Pattern

> NOTE: this information was copied from [DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html)

All ASCII letters are reserved as format pattern letters, which are defined as follows:

所有 ASCII 字母都保留为格式模式字母，其定义如下：

| Symbol                  | Meaning                    | Presentation            | Examples                                       |
| ----------------------- | -------------------------- | ----------------------- | ---------------------------------------------- |
| G                       | era                        | text                    | AD; Anno Domini; A                             |
| u                       | year                       | year                    | 2004; 04                                       |
| y                       | year-of-era                | year                    | 2004; 04                                       |
| D                       | day-of-year                | number                  | 189                                            |
| M/L                     | month-of-year              | number/text             | 7; 07; Jul; July; J                            |
| d                       | day-of-month               | number                  | 10                                             |
| Q/q                     | quarter-of-year            | number/text             | 3; 03; Q3; 3rd quarter                         |
| Y                       | week-based-year            | year                    | 1996; 96                                       |
| w                       | week-of-week-based-year    | number                  | 27                                             |
| W                       | week-of-month              | number                  | 4                                              |
| E                       | day-of-week                | text                    | Tue; Tuesday; T                                |
| e/c                     | localized day-of-week      | number/text             | 2; 02; Tue; Tuesday; T                         |
| F                       | week-of-month              | number                  | 3                                              |
| a                       | am-pm-of-day               | text                    | PM                                             |
| h                       | clock-hour-of-am-pm (1-12) | number                  | 12                                             |
| K                       | hour-of-am-pm (0-11)       | number                  | 0                                              |
| k                       | clock-hour-of-am-pm (1-24) | number                  | 0                                              |
| H                       | hour-of-day (0-23)         | number                  | 0                                              |
| m                       | minute-of-hour             | number                  | 30                                             |
| s                       | second-of-minute           | number                  | 55                                             |
| S                       | fraction-of-second         | fraction                | 978                                            |
| A                       | milli-of-day               | number                  | 1234                                           |
| n                       | nano-of-second             | number                  | 987654321                                      |
| N                       | nano-of-day                | number                  | 1234000000                                     |
| V                       | time-zone ID               | zone-id                 | America/Los_Angeles; Z; -08:30                 |
| z                       | time-zone name             | zone-name               | Pacific Standard Time; PST                     |
| O                       | localized zone-offset      | offset-O                | GMT+8; GMT+08:00; UTC-08:00;                   |
| X                       | zone-offset *Z* for zero   | offset-X                | Z; -08; -0830; -08:30; -083015; -08:30:15;     |
| x                       | zone-offset                | offset-x                | +0000; -08; -0830; -08:30; -083015; -08:30:15; |
| Z                       | zone-offset                | offset-Z                | +0000; -0800; -08:00;                          |
| p                       | pad next                   | pad modifier            | 1                                              |
| '                       | escape for text            | delimiter               | ''                                             |
| single quote            | literal                    | '                       | [                                              |
| optional section start  | ]                          | optional section end    | #                                              |
| reserved for future use | {                          | reserved for future use | }                                              |

The count of pattern letters determines the format.

模式字母的数量决定了格式。

- **Text**

  The text style is determined based on the number of pattern letters used. Less than 4 pattern letters will use the short form. Exactly 4 pattern letters will use the full form. Exactly 5 pattern letters will use the narrow form. Pattern letters `L`, `c`, and `q` specify the stand-alone form of the text styles.

  文本样式是根据使用的模式字母的数量确定的。少于 4 个模式字母将使用简写形式。正好 4 个模式字母将使用完整形式。正好 5 个模式字母将使用窄形式。模式字母`L`、`c`和`q`指定文本样式的独立形式。

- **Number**

  If the count of letters is one, then the value is output using the minimum number of digits and without padding. Otherwise, the count of digits is used as the width of the output field, with the value zero-padded as necessary. The following pattern letters have constraints on the count of letters. Only one letter of `c` and `F` can be specified. Up to two letters of `d`, `H`, `h`, `K`, `k`, `m`, and `s` can be specified. Up to three letters of `D` can be specified.

  如果字母数为 1，则使用最小位数输出该值并且不进行填充。否则，将使用数字计数作为输出字段的宽度，并根据需要填充零值。以下模式字母对字母数有限制。只能指定`c`和 的一个字母`F`。最多可以指定两个字母`d`, `H`, `h`, `K`, `k`, `m`, 和`s`。最多`D`可以指定三个字母。

- **Number/Text**

  If the count of pattern letters is 3 or greater, use the Text rules above. Otherwise use the Number rules above.

  如果模式字母的数量为 3 或更多，请使用上面的文本规则。否则使用上面的数字规则。

- **Fraction**

  Outputs the nano-of-second field as a fraction-of-second. The nano-of-second value has nine digits, thus the count of pattern letters is from 1 to 9. If it is less than 9, then the nano-of-second value is truncated, with only the most significant digits being output.

  以几分之一秒的形式输出纳秒级字段。纳秒值有九位数字，因此模式字母的计数是从 1 到 9。如果它小于 9，那么纳秒值将被截断，只输出最重要的数字。

- **Year**

  The count of letters determines the minimum field width below which padding is used. If the count of letters is two, then a reduced two digit form is used. For printing, this outputs the rightmost two digits. For parsing, this will parse using the base value of 2000, resulting in a year within the range 2000 to 2099 inclusive. If the count of letters is less than four (but not two), then the sign is only output for negative years as per `SignStyle.NORMAL`. Otherwise, the sign is output if the pad width is exceeded, as per `SignStyle.EXCEEDS_PAD`.

  字母数决定了使用填充的最小字段宽度。如果字母数为两个，则使用减少的两位数形式。对于打印，这会输出最右边的两位数字。对于解析，这将使用基值 2000 进行解析，从而生成 2000 到 2099 范围内的年份。如果字母数少于四个（但不是两个），则符号仅按 输出负年份`SignStyle.NORMAL`。否则，如果超过焊盘宽度，则按照 输出符号`SignStyle.EXCEEDS_PAD`。

- **ZoneId**

  This outputs the time-zone ID, such as `Europe/Paris`. If the count of letters is two, then the time-zone ID is output. Any other count of letters throws `IllegalArgumentException`.

  这将输出时区 ID，例如`Europe/Paris`. 如果字母数为 2，则输出时区 ID。任何其他计数的字母都会抛出`IllegalArgumentException`。

- **Zone names**

  This outputs the display name of the time-zone ID. If the count of letters is one, two or three, then the short name is output. If the count of letters is four, then the full name is output. Five or more letters throws `IllegalArgumentException`.

  这将输出时区 ID 的显示名称。如果字母数为 1、2 或 3，则输出短名称。如果字母数为四，则输出全名。五个或更多的字母抛出`IllegalArgumentException`。

- **Offset X and x**

  This formats the offset based on the number of pattern letters. One letter outputs just the hour, such as `+01`, unless the minute is non-zero in which case the minute is also output, such as `+0130`. Two letters outputs the hour and minute, without a colon, such as `+0130`. Three letters outputs the hour and minute, with a colon, such as `+01:30`. Four letters outputs the hour and minute and optional second, without a colon, such as `+013015`. Five letters outputs the hour and minute and optional second, with a colon, such as `+01:30:15`. Six or more letters throws `IllegalArgumentException`. Pattern letter `X` (upper case) will output `Z` when the offset to be output would be zero, whereas pattern letter `x` (lower case) will output `+00`, `+0000`, or `+00:00`.

  这根据模式字母的数量格式化偏移量。一个字母只输出小时，例如`+01`，除非分钟不为零，在这种情况下也会输出分钟，例如 `+0130`。两个字母输出小时和分钟，没有冒号，例如 `+0130`. 三个字母输出小时和分钟，带有冒号，例如 `+01:30`. 四个字母输出小时和分钟以及可选的秒，没有冒号，例如`+013015`. 五个字母输出小时和分钟以及可选的秒，带有冒号，例如`+01:30:15`. 六个或更多字母抛出`IllegalArgumentException`。图案字母`X`（大写）将输出`Z`当偏移要输出将为零，而图案字母`x`（小写）将输出`+00`，`+0000`或 `+00:00`。

- **Offset O**

  This formats the localized offset based on the number of pattern letters. One letter outputs the short form of the localized offset, which is localized offset text, such as `GMT`, with hour without leading zero, optional 2-digit minute and second if non-zero, and colon, for example `GMT+8`. Four letters outputs the full form, which is localized offset text, such as `GMT, with 2-digit hour and minute field, optional second field if non-zero, and colon, for example `GMT+08:00`. Any other count of letters throws `IllegalArgumentException`.

  这根据模式字母的数量格式化本地化的偏移量。一个字母输出本地化偏移量的简写形式，即本地化偏移量文本，例如`GMT`，带小时不带前导零，可选的两位数分钟和秒（如果非零）和冒号，例如`GMT+8`。四个字母输出完整形式，即本地化的偏移文本，例如`GMT, with 2-digit hour and minute field, optional second field if non-zero, and colon, for example `GMT+08:00`. 任何其他计数的字母都会抛出 `IllegalArgumentException`。

- **Offset Z**

  This formats the offset based on the number of pattern letters. One, two or three letters outputs the hour and minute, without a colon, such as `+0130`. The output will be `+0000` when the offset is zero. Four letters outputs the full form of localized offset, equivalent to four letters of Offset-O. The output will be the corresponding localized offset text if the offset is zero. Five letters outputs the hour, minute, with optional second if non-zero, with colon. It outputs `Z` if the offset is zero. Six or more letters throws IllegalArgumentException.

  这根据模式字母的数量格式化偏移量。一、二或三个字母输出小时和分钟，不带冒号，例如`+0130`. 输出将是`+0000`当偏移量为零时。四个字母输出局部偏移的完整形式，相当于Offset-O的四个字母。如果偏移为零，则输出将是相应的本地化偏移文本。五个字母输出小时、分钟，如果非零，可选秒，带冒号。`Z`如果偏移为零，则输出。六个或更多字母抛出 IllegalArgumentException。

- **Optional section**

  The optional section markers work exactly like calling `DateTimeFormatterBuilder.optionalStart()` and `DateTimeFormatterBuilder.optionalEnd()`.

  可选部分标记的工作方式与调用`DateTimeFormatterBuilder.optionalStart()`和 完全一样 `DateTimeFormatterBuilder.optionalEnd()`。

- **Pad modifier**

  Modifies the pattern that immediately follows to be padded with spaces. The pad width is determined by the number of pattern letters. This is the same as calling `DateTimeFormatterBuilder.padNext(int)`.

  修改紧随其后的模式以填充空格。焊盘宽度由图案字母的数量决定。这与调用`DateTimeFormatterBuilder.padNext(int)`.

For example, `ppH` outputs the hour-of-day padded on the left with spaces to a width of 2.

例如，`ppH`输出在左侧用空格填充到宽度为 2 的小时。

Any unrecognized letter is an error. Any non-letter character, other than `[`, `]`, `{`, `}`, `#` and the single quote will be output directly. Despite this, it is recommended to use single quotes around all characters that you want to output directly to ensure that future changes do not break your application.

任何无法识别的字母都是错误。除了`[`, `]`, `{`, `}`,`#`和单引号之外的任何非字母字符 都将直接输出。尽管如此，还是建议在要直接输出的所有字符周围使用单引号，以确保将来的更改不会破坏您的应用程序。

###  Time zone in date range aggregations

Dates can be converted from another time zone to UTC by specifying the `time_zone` parameter.

通过指定`time_zone`参数，可以将日期从另一个时区转换为 UTC 。

Time zones may either be specified as an ISO 8601 UTC offset (e.g. +01:00 or -08:00) or as one of the time zone ids from the TZ database.

时区可以指定为 ISO 8601 UTC 偏移量（例如 +01:00 或 -08:00）或指定为 TZ 数据库中的时区 ID 之一。

The `time_zone` parameter is also applied to rounding in date math expressions. As an example, to round to the beginning of the day in the CET time zone, you can do the following:

该`time_zone`参数还应用于日期数学表达式的舍入。例如，要四舍五入到 CET 时区的一天开始，您可以执行以下操作：

```console
POST /sales/_search?size=0
{
   "aggs": {
       "range": {
           "date_range": {
               "field": "date",
               "time_zone": "CET",
               "ranges": [
                  { "to": "2016/02/01" }, (1)
                  { "from": "2016/02/01", "to" : "now/d" }, (2)
                  { "from": "now/d" }
              ]
          }
      }
   }
}
```

1.  This date will be converted to `2016-02-01T00:00:00.000+01:00`.

    此日期将转换为`2016-02-01T00:00:00.000+01:00`.

2. `now/d` will be rounded to the beginning of the day in the CET time zone.

   `now/d` 将四舍五入到 CET 时区的一天开始。

###  Keyed Response

Setting the `keyed` flag to `true` will associate a unique string key with each bucket and return the ranges as a hash rather than an array:

将`keyed`标志设置为`true`将与每个存储桶关联一个唯一的字符串键，并将范围作为散列而不是数组返回：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "range": {
      "date_range": {
        "field": "date",
        "format": "MM-yyy",
        "ranges": [
          { "to": "now-10M/M" },
          { "from": "now-10M/M" }
        ],
        "keyed": true
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
    "range": {
      "buckets": {
        "*-10-2015": {
          "to": 1.4436576E12,
          "to_as_string": "10-2015",
          "doc_count": 7
        },
        "10-2015-*": {
          "from": 1.4436576E12,
          "from_as_string": "10-2015",
          "doc_count": 0
        }
      }
    }
  }
}
```

It is also possible to customize the key for each range:

还可以为每个范围自定义键：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "range": {
      "date_range": {
        "field": "date",
        "format": "MM-yyy",
        "ranges": [
          { "from": "01-2015", "to": "03-2015", "key": "quarter_01" },
          { "from": "03-2015", "to": "06-2015", "key": "quarter_02" }
        ],
        "keyed": true
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
    "range": {
      "buckets": {
        "quarter_01": {
          "from": 1.4200704E12,
          "from_as_string": "01-2015",
          "to": 1.425168E12,
          "to_as_string": "03-2015",
          "doc_count": 5
        },
        "quarter_02": {
          "from": 1.425168E12,
          "from_as_string": "03-2015",
          "to": 1.4331168E12,
          "to_as_string": "06-2015",
          "doc_count": 2
        }
      }
    }
  }
}
```

#  Diversified sampler aggregation

Like the `sampler` aggregation this is a filtering aggregation used to limit any sub aggregations' processing to a sample of the top-scoring documents. The `diversified_sampler` aggregation adds the ability to limit the number of matches that share a common value such as an "author".

与`sampler`聚合一样，这是一个过滤聚合，用于将任何子聚合的处理限制为得分最高的文档样本。该`diversified_sampler`聚集增加了限制共享一个共同的价值匹配的数量的能力，如“作者”。

> NOTE: Any good market researcher will tell you that when working with samples of data it is important that the sample represents a healthy variety of opinions rather than being skewed by any single voice. The same is true with aggregations and sampling with these diversify settings can offer a way to remove the bias in your content (an over-populated geography, a large spike in a timeline or an over-active forum spammer).
>
> 任何优秀的市场研究人员都会告诉您，在处理数据样本时，重要的是样本代表健康的各种观点，而不是被任何单一的声音所歪曲。使用这些多样化设置的聚合和抽样也是如此，可以提供一种消除内容偏差的方法（人口过多的地理位置、时间线中的大高峰或过度活跃的论坛垃圾邮件发送者）。

**Example use cases:**

- Tightening the focus of analytics to high-relevance matches rather than the potentially very long tail of low-quality matches

  将分析重点集中在高相关性匹配上，而不是潜在的低质量匹配的长尾

- Removing bias from analytics by ensuring fair representation of content from different sources

  通过确保来自不同来源的内容的公平表示来消除分析中的偏见

- Reducing the running cost of aggregations that can produce useful results using only samples e.g. `significant_terms`

  降低仅使用样本即可产生有用结果的聚合的运行成本，例如 `significant_terms`

The `field` setting is used to provide values used for de-duplication and the `max_docs_per_value` setting controls the maximum number of documents collected on any one shard which share a common value. The default setting for `max_docs_per_value` is 1.

该`field`设置用于提供用于重复数据删除的值，并且该`max_docs_per_value`设置控制在共享公共值的任何一个分片上收集的最大文档数。的默认设置为`max_docs_per_value`1。

The aggregation will throw an error if the `field` produces multiple values for a single document (de-duplication using multi-valued fields is not supported due to efficiency concerns).

如果`field`为单个文档生成多个值，聚合将抛出错误（由于效率问题，不支持使用多值字段的重复数据删除）。

Example:

We might want to see which tags are strongly associated with `#elasticsearch` on StackOverflow forum posts but ignoring the effects of some prolific users with a tendency to misspell #Kibana as #Cabana.

我们可能想`#elasticsearch`在 StackOverflow 论坛帖子上查看哪些标签与哪些标签密切相关，但忽略了一些倾向于将#Kibana 拼错为 #Cabana 的许多用户的影响。

```console
POST /stackoverflow/_search?size=0
{
  "query": {
    "query_string": {
      "query": "tags:elasticsearch"
    }
  },
  "aggs": {
    "my_unbiased_sample": {
      "diversified_sampler": {
        "shard_size": 200,
        "field": "author"
      },
      "aggs": {
        "keywords": {
          "significant_terms": {
            "field": "tags",
            "exclude": [ "elasticsearch" ]
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
  ...
  "aggregations": {
    "my_unbiased_sample": {
      "doc_count": 151,           (1)
      "keywords": {               (2)
        "doc_count": 151,
        "bg_count": 650,
        "buckets": [
          {
            "key": "kibana",
            "doc_count": 150,
            "score": 2.213,
            "bg_count": 200
          }
        ]
      }
    }
  }
}
```

1. 151 documents were sampled in total.

   共抽取151份文档。

2. The results of the significant_terms aggregation are not skewed by any single author’s quirks because we asked for a maximum of one post from any one author in our sample.

   `significant_terms` 聚合的结果不受任何单个作者的怪癖的影响，因为我们要求样本中任何一位作者最多发表一篇文章

###  Scripted example

In this scenario we might want to diversify on a combination of field values. We can use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) to produce a hash of the multiple values in a tags field to ensure we don’t have a sample that consists of the same repeated combinations of tags.

在这种情况下，我们可能希望对字段值的组合进行多样化。我们可以使用[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)来生成标签字段中多个值的散列，以确保我们没有包含相同重复标签组合的样本。

```console
POST /stackoverflow/_search?size=0
{
  "query": {
    "query_string": {
      "query": "tags:kibana"
    }
  },
  "runtime_mappings": {
    "tags.hash": {
      "type": "long",
      "script": "emit(doc['tags'].hashCode())"
    }
  },
  "aggs": {
    "my_unbiased_sample": {
      "diversified_sampler": {
        "shard_size": 200,
        "max_docs_per_value": 3,
        "field": "tags.hash"
      },
      "aggs": {
        "keywords": {
          "significant_terms": {
            "field": "tags",
            "exclude": [ "kibana" ]
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
  ...
  "aggregations": {
    "my_unbiased_sample": {
      "doc_count": 6,
      "keywords": {
        "doc_count": 6,
        "bg_count": 650,
        "buckets": [
          {
            "key": "logstash",
            "doc_count": 3,
            "score": 2.213,
            "bg_count": 50
          },
          {
            "key": "elasticsearch",
            "doc_count": 3,
            "score": 1.34,
            "bg_count": 200
          }
        ]
      }
    }
  }
}
```

###  shard_size

The `shard_size` parameter limits how many top-scoring documents are collected in the sample processed on each shard. The default value is 100.

该`shard_size`参数限制了在每个分片上处理的样本中收集了多少得分最高的文档。默认值为 100。

###  max_docs_per_value

The `max_docs_per_value` is an optional parameter and limits how many documents are permitted per choice of de-duplicating value. The default setting is "1".

这`max_docs_per_value`是一个可选参数，用于限制每个重复数据删除值选择允许的文档数量。默认设置为“1”。

###  execution_hint

The optional `execution_hint` setting can influence the management of the values used for de-duplication. Each option will hold up to `shard_size` values in memory while performing de-duplication but the type of value held can be controlled as follows:

可选`execution_hint`设置可以影响用于重复数据删除的值的管理。`shard_size`在执行重复数据删除时，每个选项将在内存中保存最多值，但保存的值类型可以如下控制：

- hold field values directly (`map`)

  直接保存字段值 ( `map`)

- hold ordinals of the field as determined by the Lucene index (`global_ordinals`)

  保存由 Lucene 索引 ( `global_ordinals`) 确定的字段的序数

- hold hashes of the field values - with potential for hash collisions (`bytes_hash`)

  保存字段值的散列 - 有可能发生散列冲突 ( `bytes_hash`)

The default setting is to use [`global_ordinals`](https://www.elastic.co/guide/en/elasticsearch/reference/master/eager-global-ordinals.html) if this information is available from the Lucene index and reverting to `map` if not. The `bytes_hash` setting may prove faster in some cases but introduces the possibility of false positives in de-duplication logic due to the possibility of hash collisions. Please note that Elasticsearch will ignore the choice of execution hint if it is not applicable and that there is no backward compatibility guarantee on these hints.

默认设置是[`global_ordinals`](https://www.elastic.co/guide/en/elasticsearch/reference/master/eager-global-ordinals.html)在此信息可从 Lucene 索引中获得时使用，`map`如果没有则恢复为。`bytes_hash`在某些情况下，设置可能会更快，但由于散列冲突的可能性，在重复数据删除逻辑中引入了误报的可能性。请注意，如果执行提示的选择不适用，并且这些提示没有向后兼容性保证，则 Elasticsearch 将忽略它。

###  Limitations

####  Cannot be nested under `breadth_first` aggregations

Being a quality-based filter the diversified_sampler aggregation needs access to the relevance score produced for each document. It therefore cannot be nested under a `terms` aggregation which has the `collect_mode` switched from the default `depth_first` mode to `breadth_first` as this discards scores. In this situation an error will be thrown.

作为基于质量的过滤器，diversity_sampler 聚合需要访问为每个文档生成的相关性分数。因此，它不能嵌套在从默认模式切换到的`terms`聚合下，因为这会丢弃分数。在这种情况下，将抛出错误。`collect_mode``depth_first``breadth_first`

####  Limited de-dup logic.

The de-duplication logic applies only at a shard level so will not apply across shards.

重复数据删除逻辑仅适用于分片级别，因此不会跨分片应用。

####  No specialized syntax for geo/date fields

Currently the syntax for defining the diversifying values is defined by a choice of `field` or `script` - there is no added syntactical sugar for expressing geo or date units such as "7d" (7 days). This support may be added in a later release and users will currently have to create these sorts of values using a script.

目前，定义多样化值的语法是通过选择`field`或 定义的`script`- 没有添加用于表示地理或日期单位的语法糖，例如“7d”（7 天）。此支持可能会在以后的版本中添加，用户目前必须使用脚本创建这些类型的值。

#  Filter aggregation

A single bucket aggregation that narrows the set of documents to those that match a [query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html).

将文档集缩小到与[查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html)匹配的文档集的单个桶聚合。

Example:

```console
POST /sales/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "avg_price": { "avg": { "field": "price" } },
    "t_shirts": {
      "filter": { "term": { "type": "t-shirt" } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

The previous example calculates the average price of all sales as well as the average price of all T-shirt sales.

前面的示例计算所有销售的平均价格以及所有 T 恤销售的平均价格。

Response:

```console-result
{
  "aggregations": {
    "avg_price": { "value": 140.71428571428572 },
    "t_shirts": {
      "doc_count": 3,
      "avg_price": { "value": 128.33333333333334 }
    }
  }
}
```

###  Use a top-level `query` to limit all aggregations

To limit the documents on which all aggregations in a search run, use a top-level `query`. This is faster than a single `filter` aggregation with sub-aggregations.

要限制搜索中运行所有聚合的文档，请使用顶级`query`. 这比`filter`具有子聚合的单个聚合更快。

For example, use this:

```console
POST /sales/_search?size=0&filter_path=aggregations
{
  "query": { "term": { "type": "t-shirt" } },
  "aggs": {
    "avg_price": { "avg": { "field": "price" } }
  }
}
```

Instead of this:

```console
POST /sales/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "t_shirts": {
      "filter": { "term": { "type": "t-shirt" } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

###  Use the `filters` aggregation for multiple filters

To group documents using multiple filters, use the [`filters` aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-filters-aggregation.html). This is faster than multiple `filter` aggregations.

要使用多个过滤器对文档进行分组，请使用 [`filters`聚合](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-filters-aggregation.html)。这比多个`filter`聚合更快。

For example, use this:

```console
POST /sales/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "f": {
      "filters": {
        "filters": {
          "hats": { "term": { "type": "hat" } },
          "t_shirts": { "term": { "type": "t-shirt" } }
        }
      },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

Instead of this:

```console
POST /sales/_search?size=0&filter_path=aggregations
{
  "aggs": {
    "hats": {
      "filter": { "term": { "type": "hat" } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    },
    "t_shirts": {
      "filter": { "term": { "type": "t-shirt" } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

#  Filters aggregation

A multi-bucket aggregation where each bucket contains the documents that match a [query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html).

多桶聚合，其中每个桶包含与[查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html)匹配的文档。

Example:

```console
PUT /logs/_bulk?refresh
{ "index" : { "_id" : 1 } }
{ "body" : "warning: page could not be rendered" }
{ "index" : { "_id" : 2 } }
{ "body" : "authentication error" }
{ "index" : { "_id" : 3 } }
{ "body" : "warning: connection timed out" }

GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : {
          "errors" :   { "match" : { "body" : "error"   }},
          "warnings" : { "match" : { "body" : "warning" }}
        }
      }
    }
  }
}
```

In the above example, we analyze log messages. The aggregation will build two collection (buckets) of log messages - one for all those containing an error, and another for all those containing a warning.

在上面的例子中，我们分析了日志消息。聚合将构建两个日志消息集合（存储桶）——一个用于所有包含错误的消息，另一个用于所有包含警告的消息。

Response:

```console-result
{
  "took": 9,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "messages": {
      "buckets": {
        "errors": {
          "doc_count": 1
        },
        "warnings": {
          "doc_count": 2
        }
      }
    }
  }
}
```

###  Anonymous filters

The filters field can also be provided as an array of filters, as in the following request:

过滤器字段也可以作为过滤器数组提供，如以下请求所示：

```console
GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "filters" : [
          { "match" : { "body" : "error"   }},
          { "match" : { "body" : "warning" }}
        ]
      }
    }
  }
}
```

The filtered buckets are returned in the same order as provided in the request. The response for this example would be:

过滤后的存储桶按与请求中提供的顺序相同的顺序返回。此示例的响应将是：

```console-result
{
  "took": 4,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "messages": {
      "buckets": [
        {
          "doc_count": 1
        },
        {
          "doc_count": 2
        }
      ]
    }
  }
}
```

### `Other` Bucket

The `other_bucket` parameter can be set to add a bucket to the response which will contain all documents that do not match any of the given filters. The value of this parameter can be as follows:

`other_bucket`参数可以设置为向响应添加一个存储桶，该存储桶将包含与任何给定过滤器不匹配的所有文档。该参数的值可以如下：

- **`false`**

  Does not compute the `other` bucket

  不计算`other`桶

- **`true`**

  Returns the `other` bucket either in a bucket (named `_other_` by default) if named filters are being used, or as the last bucket if anonymous filters are being used

  如果使用命名过滤器，则 返回`other`存储桶中的存储桶（`_other_`默认命名），或者如果使用匿名过滤器，则返回最后一个存储桶

The `other_bucket_key` parameter can be used to set the key for the `other` bucket to a value other than the default `_other_`. Setting this parameter will implicitly set the `other_bucket` parameter to `true`.

该`other_bucket_key`参数可用于将`other`存储桶的键设置为默认值`_other_`以外的值。设置此参数会将参数隐式设置`other_bucket`为`true`。

The following snippet shows a response where the `other` bucket is requested to be named `other_messages`.

以下代码段显示了`other`请求将存储桶命名为的响应`other_messages`。

```console
PUT logs/_doc/4?refresh
{
  "body": "info: user Bob logged out"
}

GET logs/_search
{
  "size": 0,
  "aggs" : {
    "messages" : {
      "filters" : {
        "other_bucket_key": "other_messages",
        "filters" : {
          "errors" :   { "match" : { "body" : "error"   }},
          "warnings" : { "match" : { "body" : "warning" }}
        }
      }
    }
  }
}
```

The response would be something like the following:

```console-result
{
  "took": 3,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "messages": {
      "buckets": {
        "errors": {
          "doc_count": 1
        },
        "warnings": {
          "doc_count": 2
        },
        "other_messages": {
          "doc_count": 1
        }
      }
    }
  }
}
```

# Geo-distance aggregation

A multi-bucket aggregation that works on `geo_point` fields and conceptually works very similar to the [range](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-range-aggregation.html) aggregation. The user can define a point of origin and a set of distance range buckets. The aggregation evaluate the distance of each document value from the origin point and determines the buckets it belongs to based on the ranges (a document belongs to a bucket if the distance between the document and the origin falls within the distance range of the bucket).

适用于`geo_point`字段的多桶聚合在概念上与[范围](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-range-aggregation.html)聚合非常相似。用户可以定义一个原点和一组距离范围桶。聚合评估每个文档值与原点的距离，并根据范围确定它所属的桶（如果文档和原点之间的距离在桶的距离范围内，则文档属于一个桶）。

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
  "aggs": {
    "rings_around_amsterdam": {
      "geo_distance": {
        "field": "location",
        "origin": "52.3760, 4.894",
        "ranges": [
          { "to": 100000 },
          { "from": 100000, "to": 300000 },
          { "from": 300000 }
        ]
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
    "rings_around_amsterdam": {
      "buckets": [
        {
          "key": "*-100000.0",
          "from": 0.0,
          "to": 100000.0,
          "doc_count": 3
        },
        {
          "key": "100000.0-300000.0",
          "from": 100000.0,
          "to": 300000.0,
          "doc_count": 1
        },
        {
          "key": "300000.0-*",
          "from": 300000.0,
          "doc_count": 2
        }
      ]
    }
  }
}
```

The specified field must be of type `geo_point` (which can only be set explicitly in the mappings). And it can also hold an array of `geo_point` fields, in which case all will be taken into account during aggregation. The origin point can accept all formats supported by the [`geo_point` type](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html):

指定的字段必须是类型`geo_point`（只能在映射中明确设置）。并且它还可以保存一个`geo_point`字段数组，在这种情况下，在聚合过程中将考虑所有字段。原点可以接受[`geo_point`type](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)支持的所有格式：

- Object format: `{ "lat" : 52.3760, "lon" : 4.894 }` - this is the safest format as it is the most explicit about the `lat` & `lon` values

  对象格式：`{ "lat" : 52.3760, "lon" : 4.894 }`- 这是最安全的格式，因为它对`lat`&`lon`值 最明确

- String format: `"52.3760, 4.894"` - where the first number is the `lat` and the second is the `lon`

  字符串格式：`"52.3760, 4.894"`- 其中第一个数字是`lat`，第二个是`lon`

- Array format: `[4.894, 52.3760]` - which is based on the `GeoJson` standard and where the first number is the `lon` and the second one is the `lat`

  数组格式：`[4.894, 52.3760]`- 基于`GeoJson`标准，第一个数字是`lon`，第二个是`lat`

By default, the distance unit is `m` (meters) but it can also accept: `mi` (miles), `in` (inches), `yd` (yards), `km` (kilometers), `cm` (centimeters), `mm` (millimeters).

默认情况下，距离单位是`m`（米），但它也可以接受：（`mi`英里）、`in`（英寸）、`yd`（码）、`km`（公里）、`cm`（厘米）、`mm`（毫米）。

```console
POST /museums/_search?size=0
{
  "aggs": {
    "rings": {
      "geo_distance": {
        "field": "location",
        "origin": "52.3760, 4.894",
        "unit": "km", (1)
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 300 },
          { "from": 300 }
        ]
      }
    }
  }
}
```

1.  The distances will be computed in kilometers

   距离将以公里计算

There are two distance calculation modes: `arc` (the default), and `plane`. The `arc` calculation is the most accurate. The `plane` is the fastest but least accurate. Consider using `plane` when your search context is "narrow", and spans smaller geographical areas (~5km). `plane` will return higher error margins for searches across very large areas (e.g. cross continent search). The distance calculation type can be set using the `distance_type` parameter:

有两种距离计算模式：（`arc`默认）和`plane`。该`arc`计算是最准确的。这`plane`是最快但最不准确的。`plane`当您的搜索上下文“狭窄”并且跨越较小的地理区域（约 5 公里）时，请考虑使用。`plane`将在非常大的区域（例如跨大陆搜索）中进行搜索返回更高的错误率。可以使用`distance_type`参数设置距离计算类型：

```console
POST /museums/_search?size=0
{
  "aggs": {
    "rings": {
      "geo_distance": {
        "field": "location",
        "origin": "52.3760, 4.894",
        "unit": "km",
        "distance_type": "plane",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 300 },
          { "from": 300 }
        ]
      }
    }
  }
}
```

###  Keyed Response

Setting the `keyed` flag to `true` will associate a unique string key with each bucket and return the ranges as a hash rather than an array:

将`keyed`标志设置为`true`将与每个存储桶关联一个唯一的字符串键，并将范围作为散列而不是数组返回：

```console
POST /museums/_search?size=0
{
  "aggs": {
    "rings_around_amsterdam": {
      "geo_distance": {
        "field": "location",
        "origin": "52.3760, 4.894",
        "ranges": [
          { "to": 100000 },
          { "from": 100000, "to": 300000 },
          { "from": 300000 }
        ],
        "keyed": true
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
    "rings_around_amsterdam": {
      "buckets": {
        "*-100000.0": {
          "from": 0.0,
          "to": 100000.0,
          "doc_count": 3
        },
        "100000.0-300000.0": {
          "from": 100000.0,
          "to": 300000.0,
          "doc_count": 1
        },
        "300000.0-*": {
          "from": 300000.0,
          "doc_count": 2
        }
      }
    }
  }
}
```

It is also possible to customize the key for each range:

还可以为每个范围自定义键：

```console
POST /museums/_search?size=0
{
  "aggs": {
    "rings_around_amsterdam": {
      "geo_distance": {
        "field": "location",
        "origin": "52.3760, 4.894",
        "ranges": [
          { "to": 100000, "key": "first_ring" },
          { "from": 100000, "to": 300000, "key": "second_ring" },
          { "from": 300000, "key": "third_ring" }
        ],
        "keyed": true
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
    "rings_around_amsterdam": {
      "buckets": {
        "first_ring": {
          "from": 0.0,
          "to": 100000.0,
          "doc_count": 3
        },
        "second_ring": {
          "from": 100000.0,
          "to": 300000.0,
          "doc_count": 1
        },
        "third_ring": {
          "from": 300000.0,
          "doc_count": 2
        }
      }
    }
  }
}
```

#  Geohash grid aggregation

A multi-bucket aggregation that groups [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) and [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) values into buckets that represent a grid. The resulting grid can be sparse and only contains cells that have matching data. Each cell is labeled using a [geohash](https://en.wikipedia.org/wiki/Geohash) which is of user-definable precision.

一种多桶聚合该基团[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)和 [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html)值到水桶表示的网格。生成的网格可以是稀疏的，并且只包含具有匹配数据的单元格。每个单元格都使用具有用户可定义精度的[geohash](https://en.wikipedia.org/wiki/Geohash)进行标记。

- High precision geohashes have a long string length and represent cells that cover only a small area.

  高精度 geohashes 具有很长的字符串长度，代表仅覆盖小区域的单元格。

- Low precision geohashes have a short string length and represent cells that each cover a large area.

  低精度 geohashes 具有较短的字符串长度，代表每个覆盖大面积的单元格。

Geohashes used in this aggregation can have a choice of precision between 1 and 12.

此聚合中使用的 Geohashes 可以选择 1 到 12 之间的精度。

> WARNING: The highest-precision geohash of length 12 produces cells that cover less than a square metre of land and so high-precision requests can be very costly in terms of RAM and result sizes. Please see the example below on how to first filter the aggregation to a smaller geographic area before requesting high-levels of detail.
>
> 长度为 12 的最高精度 geohash 生成覆盖不到一平方米土地的单元格，因此高精度请求在 RAM 和结果大小方面可能非常昂贵。请参阅下面的示例，了解如何在请求高级别的详细信息之前先将聚合过滤到较小的地理区域。

You can only use `geohash_grid` to aggregate an explicitly mapped `geo_point` or `geo_shape` field. If the `geo_point` field contains an array, `geohash_grid` aggregates all the array values.

您只能用于`geohash_grid`聚合显式映射的`geo_point`或 `geo_shape`字段。如果`geo_point`字段包含数组，则`geohash_grid` 聚合所有数组值。

###  Simple low-precision request

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
  "aggregations": {
    "large-grid": {
      "geohash_grid": {
        "field": "location",
        "precision": 3
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
  "large-grid": {
    "buckets": [
      {
        "key": "u17",
        "doc_count": 3
      },
      {
        "key": "u09",
        "doc_count": 2
      },
      {
        "key": "u15",
        "doc_count": 1
      }
    ]
  }
}
}
```

###  High-precision requests

When requesting detailed buckets (typically for displaying a "zoomed in" map) a filter like [geo_bounding_box](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html) should be applied to narrow the subject area otherwise potentially millions of buckets will be created and returned.

当请求详细的存储桶（通常用于显示“放大”地图）时，应应用类似[geo_bounding_box](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html)的过滤器来缩小主题区域，否则可能会创建和返回数百万个存储桶。

```console
POST /museums/_search?size=0
{
  "aggregations": {
    "zoomed-in": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": "52.4, 4.9",
            "bottom_right": "52.3, 5.0"
          }
        }
      },
      "aggregations": {
        "zoom1": {
          "geohash_grid": {
            "field": "location",
            "precision": 8
          }
        }
      }
    }
  }
}
```

The geohashes returned by the `geohash_grid` aggregation can be also used for zooming in. To zoom into the first geohash `u17` returned in the previous example, it should be specified as both `top_left` and `bottom_right` corner:

聚合返回的geohash`geohash_grid`也可以用于放大。要放大`u17`前面例子中返回的第一个geohash ，应该指定为both`top_left`和`bottom_right`corner：

```console
POST /museums/_search?size=0
{
  "aggregations": {
    "zoomed-in": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": "u17",
            "bottom_right": "u17"
          }
        }
      },
      "aggregations": {
        "zoom1": {
          "geohash_grid": {
            "field": "location",
            "precision": 8
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
  "aggregations": {
    "zoomed-in": {
      "doc_count": 3,
      "zoom1": {
        "buckets": [
          {
            "key": "u173zy3j",
            "doc_count": 1
          },
          {
            "key": "u173zvfz",
            "doc_count": 1
          },
          {
            "key": "u173zt90",
            "doc_count": 1
          }
        ]
      }
    }
  }
}
```

For "zooming in" on the system that don’t support geohashes, the bucket keys should be translated into bounding boxes using one of available geohash libraries. For example, for javascript the [node-geohash](https://github.com/sunng87/node-geohash) library can be used:

为了在不支持 geohash 的系统上“放大”，应该使用可用的 geohash 库之一将存储桶键转换为边界框。例如，对于 javascript，可以使用[node-geohash](https://github.com/sunng87/node-geohash)库：

```js
var geohash = require('ngeohash');

// bbox will contain [ 52.03125, 4.21875, 53.4375, 5.625 ]
//                   [   minlat,  minlon,  maxlat, maxlon]
var bbox = geohash.decode_bbox('u17');
```

###  Requests with additional bounding box filtering

The `geohash_grid` aggregation supports an optional `bounds` parameter that restricts the cells considered to those that intersects the bounds provided. The `bounds` parameter accepts the bounding box in all the same [accepted formats](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html#query-dsl-geo-bounding-box-query-accepted-formats) of the bounds specified in the Geo Bounding Box Query. This bounding box can be used with or without an additional `geo_bounding_box` query filtering the points prior to aggregating. It is an independent bounding box that can intersect with, be equal to, or be disjoint to any additional `geo_bounding_box` queries defined in the context of the aggregation.

`geohash_grid`聚合支持可选的`bounds`，其限制被认为是那些相交的边界提供的细胞参数。该`bounds`参数接受与Geo Bounding Box Query 中指定的边界相同的所有[可接受格式](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html#query-dsl-geo-bounding-box-query-accepted-formats)的边界框。`geo_bounding_box`在聚合之前，可以使用或不使用过滤点的附加查询来使用此边界框。它是一个独立的边界框，可以与`geo_bounding_box`聚合上下文中定义的任何其他查询相交、相等或不相交。

```console
POST /museums/_search?size=0
{
  "aggregations": {
    "tiles-in-bounds": {
      "geohash_grid": {
        "field": "location",
        "precision": 8,
        "bounds": {
          "top_left": "53.4375, 4.21875",
          "bottom_right": "52.03125, 5.625"
        }
      }
    }
  }
}
```

```console-result
{
  ...
  "aggregations": {
    "tiles-in-bounds": {
      "buckets": [
        {
          "key": "u173zy3j",
          "doc_count": 1
        },
        {
          "key": "u173zvfz",
          "doc_count": 1
        },
        {
          "key": "u173zt90",
          "doc_count": 1
        }
      ]
    }
  }
}
```

###  Cell dimensions at the equator

The table below shows the metric dimensions for cells covered by various string lengths of geohash. Cell dimensions vary with latitude and so the table is for the worst-case scenario at the equator.

下表显示了由 geohash 的各种字符串长度覆盖的单元格的度量维度。像元尺寸随纬度而变化，因此该表是针对赤道的最坏情况。

| **GeoHash length** | **Area width x height** |
| ------------------ | ----------------------- |
| 1                  | 5,009.4km x 4,992.6km   |
| 2                  | 1,252.3km x 624.1km     |
| 3                  | 156.5km x 156km         |
| 4                  | 39.1km x 19.5km         |
| 5                  | 4.9km x 4.9km           |
| 6                  | 1.2km x 609.4m          |
| 7                  | 152.9m x 152.4m         |
| 8                  | 38.2m x 19m             |
| 9                  | 4.8m x 4.8m             |
| 10                 | 1.2m x 59.5cm           |
| 11                 | 14.9cm x 14.9cm         |
| 12                 | 3.7cm x 1.9cm           |

####  Aggregating `geo_shape` fields

Aggregating on [Geo-shape](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) fields works just as it does for points, except that a single shape can be counted for in multiple tiles. A shape will contribute to the count of matching values if any part of its shape intersects with that tile. Below is an image that demonstrates this:

聚合[地理形状](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html)字段的工作方式与对点的聚合相同，不同之处在于可以在多个图块中计算单个形状。如果形状的任何部分与该图块相交，则该形状将有助于匹配值的计数。下面是一张图片，证明了这一点：

![geoshape grid](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/spatial/geoshape_grid.png)

###  Options

| field      | Mandatory. The name of the field indexed with GeoPoints.<br />强制的。用 GeoPoints 索引的字段的名称。 |
| ---------- | ------------------------------------------------------------ |
| precision  | Optional. The string length of the geohashes used to define cells/buckets in the results. Defaults to 5. The precision can either be defined in terms of the integer precision levels mentioned above. Values outside of [1,12] will be rejected. Alternatively, the precision level can be approximated from a distance measure like "1km", "10m". The precision level is calculate such that cells will not exceed the specified size (diagonal) of the required precision. When this would lead to precision levels higher than the supported 12 levels, (e.g. for distances <5.6cm) the value is rejected.<br />可选的。用于定义结果中的单元格/存储桶的地理散列的字符串长度。默认为 5。精度可以根据上述整数精度级别定义。[1,12] 之外的值将被拒绝。或者，精度水平可以从“1km”、“10m”等距离度量中近似。计算精度级别以使单元格不会超过所需精度的指定大小（对角线）。如果这会导致精度级别高于支持的 12 个级别（例如距离 <5.6cm），则拒绝该值。 |
| bounds     | Optional. The bounding box to filter the points in the bucket.<br />可选的。用于过滤桶中点的边界框。 |
| size       | Optional. The maximum number of geohash buckets to return (defaults to 10,000). When results are trimmed, buckets are prioritised based on the volumes of documents they contain.<br />可选的。要返回的 geohash 存储桶的最大数量（默认为 10,000）。当结果被修剪时，桶会根据它们包含的文档量进行优先级排序。 |
| shard_size | Optional. To allow for more accurate counting of the top cells returned in the final result the aggregation defaults to returning `max(10,(size x number-of-shards))` buckets from each shard. If this heuristic is undesirable, the number considered from each shard can be over-ridden using this parameter.<br />可选的。为了更准确地计算最终结果中返回的顶部单元格，聚合默认为`max(10,(size x number-of-shards))`从每个分片返回桶。如果这种启发式方法不受欢迎，则可以使用此参数覆盖从每个分片中考虑的数字。 |

#  Geotile grid aggregation

A multi-bucket aggregation that groups [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) and [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) values into buckets that represent a grid. The resulting grid can be sparse and only contains cells that have matching data. Each cell corresponds to a [map tile](https://en.wikipedia.org/wiki/Tiled_web_map) as used by many online map sites. Each cell is labeled using a "{zoom}/{x}/{y}" format, where zoom is equal to the user-specified precision.

一种多桶聚合该基团[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)和 [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html)值到水桶表示的网格。生成的网格可以是稀疏的，并且只包含具有匹配数据的单元格。每个单元格对应于许多在线地图站点使用的 [地图图块](https://en.wikipedia.org/wiki/Tiled_web_map)。每个单元格都使用“{zoom}/{x}/{y}”格式进行标记，其中缩放等于用户指定的精度。

- High precision keys have a larger range for x and y, and represent tiles that cover only a small area.

  高精度键具有更大的 x 和 y 范围，代表仅覆盖小区域的图块。

- Low precision keys have a smaller range for x and y, and represent tiles that each cover a large area.

  低精度键的 x 和 y 范围较小，代表每个覆盖大面积的图块。

See [Zoom level documentation](https://wiki.openstreetmap.org/wiki/Zoom_levels) on how precision (zoom) correlates to size on the ground. Precision for this aggregation can be between 0 and 29, inclusive.

请参阅[缩放级别文档](https://wiki.openstreetmap.org/wiki/Zoom_levels) ，了解精度（缩放）与地面尺寸的相关性。此聚合的精度可以介于 0 和 29 之间，包括 0 和 29。

> WARNING: The highest-precision geotile of length 29 produces cells that cover less than a 10cm by 10cm of land and so high-precision requests can be very costly in terms of RAM and result sizes. Please see the example below on how to first filter the aggregation to a smaller geographic area before requesting high-levels of detail.
>
> 长度为 29 的最高精度 geotile 生成的单元覆盖不到 10 厘米乘 10 厘米的土地，因此就 RAM 和结果大小而言，高精度请求可能非常昂贵。请参阅下面的示例，了解如何在请求高级别的详细信息之前先将聚合过滤到较小的地理区域。

You can only use `geotile_grid` to aggregate an explicitly mapped `geo_point` or `geo_shape` field. If the `geo_point` field contains an array, `geotile_grid` aggregates all the array values.

您只能用于`geotile_grid`聚合显式映射的`geo_point`或 `geo_shape`字段。如果`geo_point`字段包含数组，则`geotile_grid` 聚合所有数组值。

###  Simple low-precision request

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
  "aggregations": {
    "large-grid": {
      "geotile_grid": {
        "field": "location",
        "precision": 8
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
    "large-grid": {
      "buckets": [
        {
          "key": "8/131/84",
          "doc_count": 3
        },
        {
          "key": "8/129/88",
          "doc_count": 2
        },
        {
          "key": "8/131/85",
          "doc_count": 1
        }
      ]
    }
  }
}
```

###  High-precision requests

When requesting detailed buckets (typically for displaying a "zoomed in" map) a filter like [geo_bounding_box](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html) should be applied to narrow the subject area otherwise potentially millions of buckets will be created and returned.

当请求详细的存储桶（通常用于显示“放大”地图）时，应应用类似[geo_bounding_box](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html)的过滤器来缩小主题区域，否则可能会创建和返回数百万个存储桶。

```console
POST /museums/_search?size=0
{
  "aggregations": {
    "zoomed-in": {
      "filter": {
        "geo_bounding_box": {
          "location": {
            "top_left": "52.4, 4.9",
            "bottom_right": "52.3, 5.0"
          }
        }
      },
      "aggregations": {
        "zoom1": {
          "geotile_grid": {
            "field": "location",
            "precision": 22
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
  "aggregations": {
    "zoomed-in": {
      "doc_count": 3,
      "zoom1": {
        "buckets": [
          {
            "key": "22/2154412/1378379",
            "doc_count": 1
          },
          {
            "key": "22/2154385/1378332",
            "doc_count": 1
          },
          {
            "key": "22/2154259/1378425",
            "doc_count": 1
          }
        ]
      }
    }
  }
}
```

###  Requests with additional bounding box filtering

The `geotile_grid` aggregation supports an optional `bounds` parameter that restricts the cells considered to those that intersects the bounds provided. The `bounds` parameter accepts the bounding box in all the same [accepted formats](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html#query-dsl-geo-bounding-box-query-accepted-formats) of the bounds specified in the Geo Bounding Box Query. This bounding box can be used with or without an additional `geo_bounding_box` query for filtering the points prior to aggregating. It is an independent bounding box that can intersect with, be equal to, or be disjoint to any additional `geo_bounding_box` queries defined in the context of the aggregation.

`geotile_grid`聚合支持可选的`bounds`，其限制被认为是那些相交的边界提供的单元格参数。该`bounds`参数接受与Geo Bounding Box Query 中指定的边界相同的所有[可接受格式](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html#query-dsl-geo-bounding-box-query-accepted-formats)的边界框。这个边界框可以在有或没有附加`geo_bounding_box`查询的情况下使用，用于在聚合之前过滤点。它是一个独立的边界框，可以与`geo_bounding_box`聚合上下文中定义的任何其他查询相交、相等或不相交。

```console
POST /museums/_search?size=0
{
  "aggregations": {
    "tiles-in-bounds": {
      "geotile_grid": {
        "field": "location",
        "precision": 22,
        "bounds": {
          "top_left": "52.4, 4.9",
          "bottom_right": "52.3, 5.0"
        }
      }
    }
  }
}
```

```console-result
{
  ...
  "aggregations": {
    "tiles-in-bounds": {
      "buckets": [
        {
          "key": "22/2154412/1378379",
          "doc_count": 1
        },
        {
          "key": "22/2154385/1378332",
          "doc_count": 1
        },
        {
          "key": "22/2154259/1378425",
          "doc_count": 1
        }
      ]
    }
  }
}
```

####  Aggregating `geo_shape` fields

Aggregating on [Geo-shape](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) fields works just as it does for points, except that a single shape can be counted for in multiple tiles. A shape will contribute to the count of matching values if any part of its shape intersects with that tile. Below is an image that demonstrates this:

聚合[地理形状](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html)字段的工作方式与对点的聚合相同，不同之处在于可以在多个图块中计算单个形状。如果形状的任何部分与该图块相交，则该形状将有助于匹配值的计数。下面是一张图片，证明了这一点：

![geoshape grid](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/spatial/geoshape_grid.png)

###  Options

| field      | Mandatory. The name of the field indexed with GeoPoints.<br />强制的。用 GeoPoints 索引的字段的名称。 |
| ---------- | ------------------------------------------------------------ |
| precision  | Optional. The integer zoom of the key used to define cells/buckets in the results. Defaults to 7. Values outside of [0,29] will be rejected.<br />可选的。用于在结果中定义单元格/存储桶的键的整数缩放。默认为 7。[0,29] 之外的值将被拒绝。 |
| bounds     | Optional. The bounding box to filter the points in the bucket.<br />可选。用于过滤桶中点的边界框。 |
| size       | Optional. The maximum number of geohash buckets to return (defaults to 10,000). When results are trimmed, buckets are prioritised based on the volumes of documents they contain.<br />可选的。要返回的 geohash 存储桶的最大数量（默认为 10,000）。当结果被修剪时，桶会根据它们包含的文档量进行优先级排序。 |
| shard_size | Optional. To allow for more accurate counting of the top cells returned in the final result the aggregation defaults to returning `max(10,(size x number-of-shards))` buckets from each shard. If this heuristic is undesirable, the number considered from each shard can be over-ridden using this parameter.<br />可选的。为了更准确地计算最终结果中返回的顶部单元格，聚合默认为`max(10,(size x number-of-shards))`从每个分片返回桶。如果这种启发式方法不受欢迎，则可以使用此参数覆盖从每个分片中考虑的数字。 |

#  Global aggregation

Defines a single bucket of all the documents within the search execution context. This context is defined by the indices and the document types you’re searching on, but is **not** influenced by the search query itself.

定义搜索执行上下文中所有文档的单个存储桶。此上下文由索引和您搜索的文档类型定义，但**不受**搜索查询本身的影响。

> NOTE: Global aggregators can only be placed as top level aggregators because it doesn’t make sense to embed a global aggregator within another bucket aggregator.
>
> 全局聚合器只能作为顶级聚合器放置，因为将全局聚合器嵌入另一个存储桶聚合器没有意义。

Example:

```console
POST /sales/_search?size=0
{
  "query": {
    "match": { "type": "t-shirt" }
  },
  "aggs": {
    "all_products": {
      "global": {}, (1)
      "aggs": {     (2)
      "avg_price": { "avg": { "field": "price" } }
      }
    },
    "t_shirts": { "avg": { "field": "price" } }
  }
}
```

1. The `global` aggregation has an empty body

    该`global`集合有一个空体

2.  The sub-aggregations that are registered for this `global` aggregation

   为此`global`聚合注册的子聚合

The above aggregation demonstrates how one would compute aggregations (`avg_price` in this example) on all the documents in the search context, regardless of the query (in our example, it will compute the average price over all products in our catalog, not just on the "shirts").

上面的聚合演示了如何`avg_price`在搜索上下文中的所有文档上计算聚合（在本例中），而不考虑查询（在我们的示例中，它将计算我们目录中所有产品的平均价格，而不仅仅是“衬衫”）。

The response for the above aggregation:

```console-result
{
  ...
  "aggregations": {
    "all_products": {
      "doc_count": 7, (1)
      "avg_price": {
        "value": 140.71428571428572 (2)
      }
    },
    "t_shirts": {
      "value": 128.33333333333334 (3)
    }
  }
}
```

1. The number of documents that were aggregated (in our case, all documents within the search context)

   聚合的文档数量（在我们的例子中，搜索上下文中的所有文档）

2. The average price of all products in the index

   指数中所有产品的平均价格

3. The average price of all t-shirts

4.  所有 T 恤的平均价格

#  Histogram aggregation

A multi-bucket values source based aggregation that can be applied on numeric values or numeric range values extracted from the documents. It dynamically builds fixed size (a.k.a. interval) buckets over the values. For example, if the documents have a field that holds a price (numeric), we can configure this aggregation to dynamically build buckets with interval `5` (in case of price it may represent $5). When the aggregation executes, the price field of every document will be evaluated and will be rounded down to its closest bucket - for example, if the price is `32` and the bucket size is `5` then the rounding will yield `30` and thus the document will "fall" into the bucket that is associated with the key `30`. To make this more formal, here is the rounding function that is used:

基于多桶值源的聚合，可应用于从文档中提取的数值或数值范围值。它动态地在值上构建固定大小（又名间隔）的桶。例如，如果文档有一个包含价格（数字）的字段，我们可以配置此聚合以动态构建具有间隔的存储桶`5`（在价格的情况下，它可能代表 5 美元）。当聚合执行时，每个文档的 price 字段将被评估，并将向下舍入到最接近的存储桶 - 例如，如果价格是`32`，存储桶大小是，`5`则舍入将产生`30`，因此文档将“落入”与 key 关联的存储桶`30`。为了使这更正式，这里是使用的舍入函数：

```java
bucket_key = Math.floor((value - offset) / interval) * interval + offset
```

For range values, a document can fall into multiple buckets. The first bucket is computed from the lower bound of the range in the same way as a bucket for a single value is computed. The final bucket is computed in the same way from the upper bound of the range, and the range is counted in all buckets in between and including those two.

对于范围值，一个文档可以分为多个桶。第一个桶是从范围的下限计算的，与计算单个值的桶相同。最后一个桶是从范围的上界以相同的方式计算的，范围被计算在这两个桶之间的所有桶中，包括这两个桶。

The `interval` must be a positive decimal, while the `offset` must be a decimal in `[0, interval)` (a decimal greater than or equal to `0` and less than `interval`)

The`interval`必须是一个正小数，而 the`offset`必须是一个小数`[0, interval)` （大于等于`0`小于的小数`interval`）

The following snippet "buckets" the products based on their `price` by interval of `50`:

以下代码段根据产品`price`的间隔“存储”产品`50`：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50
      }
    }
  }
}
```

And the following may be the response:

```console-result
{
  ...
  "aggregations": {
    "prices": {
      "buckets": [
        {
          "key": 0.0,
          "doc_count": 1
        },
        {
          "key": 50.0,
          "doc_count": 1
        },
        {
          "key": 100.0,
          "doc_count": 0
        },
        {
          "key": 150.0,
          "doc_count": 2
        },
        {
          "key": 200.0,
          "doc_count": 3
        }
      ]
    }
  }
}
```

###  Minimum document count

The response above show that no documents has a price that falls within the range of `[100, 150)`. By default the response will fill gaps in the histogram with empty buckets. It is possible change that and request buckets with a higher minimum count thanks to the `min_doc_count` setting:

上面的响应表明没有任何文件的价格在 范围内`[100, 150)`。默认情况下，响应将用空桶填充直方图中的空白。由于设置，可以更改并请求具有更高最小计数的存储桶`min_doc_count`：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50,
        "min_doc_count": 1
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
    "prices": {
      "buckets": [
        {
          "key": 0.0,
          "doc_count": 1
        },
        {
          "key": 50.0,
          "doc_count": 1
        },
        {
          "key": 150.0,
          "doc_count": 2
        },
        {
          "key": 200.0,
          "doc_count": 3
        }
      ]
    }
  }
}
```

By default the `histogram` returns all the buckets within the range of the data itself, that is, the documents with the smallest values (on which with histogram) will determine the min bucket (the bucket with the smallest key) and the documents with the highest values will determine the max bucket (the bucket with the highest key). Often, when requesting empty buckets, this causes a confusion, specifically, when the data is also filtered.

默认`histogram`返回数据本身范围内的所有桶，即具有最小值（其上有直方图）的文档将确定最小桶（具有最小键的桶）和具有最高值的文档将确定最大桶（具有最高键的桶）。通常，当请求空桶时，这会导致混淆，特别是当数据也被过滤时。

To understand why, let’s look at an example:

为了理解原因，让我们看一个例子：

Lets say the you’re filtering your request to get all docs with values between `0` and `500`, in addition you’d like to slice the data per price using a histogram with an interval of `50`. You also specify `"min_doc_count" : 0` as you’d like to get all buckets even the empty ones. If it happens that all products (documents) have prices higher than `100`, the first bucket you’ll get will be the one with `100` as its key. This is confusing, as many times, you’d also like to get those buckets between `0 - 100`.

假设您正在过滤您的请求以获取值介于`0`和之间的所有文档`500`，此外您想使用间隔为 的直方图对每个价格的数据进行切片`50`。您还可以指定`"min_doc_count" : 0`要获取所有存储桶，甚至是空存储桶。如果碰巧所有产品（文件）的价格都高于`100`，您将获得的第一个桶将是`100`它的关键。这令人困惑，因为很多时候，您还希望在`0 - 100`.

With `extended_bounds` setting, you now can "force" the histogram aggregation to start building buckets on a specific `min` value and also keep on building buckets up to a `max` value (even if there are no documents anymore). Using `extended_bounds` only makes sense when `min_doc_count` is 0 (the empty buckets will never be returned if `min_doc_count` is greater than 0).

通过`extended_bounds`设置，您现在可以“强制”直方图聚合以开始在特定 `min`值上构建存储桶，并继续构建高达某个`max`值的存储桶（即使不再有文档）。`extended_bounds`仅在`min_doc_count`0时使用 才有意义（如果`min_doc_count` 大于 0，则永远不会返回空桶）。

Note that (as the name suggest) `extended_bounds` is **not** filtering buckets. Meaning, if the `extended_bounds.min` is higher than the values extracted from the documents, the documents will still dictate what the first bucket will be (and the same goes for the `extended_bounds.max` and the last bucket). For filtering buckets, one should nest the histogram aggregation under a range `filter` aggregation with the appropriate `from`/`to` settings.

需要注意的是（正如其名字暗示的）`extended_bounds`是**不**过滤桶。意思是，如果`extended_bounds.min`高于从文档中提取的值，文档仍将决定第一个存储桶是什么（对于`extended_bounds.max`最后一个存储桶也是如此）。对于过滤桶，应该将直方图聚合嵌套在`filter`具有适当`from`/`to`设置的范围聚合下。

Example:

```console
POST /sales/_search?size=0
{
  "query": {
    "constant_score": { "filter": { "range": { "price": { "to": "500" } } } }
  },
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50,
        "extended_bounds": {
          "min": 0,
          "max": 500
        }
      }
    }
  }
}
```

When aggregating ranges, buckets are based on the values of the returned documents. This means the response may include buckets outside of a query’s range. For example, if your query looks for values greater than 100, and you have a range covering 50 to 150, and an interval of 50, that document will land in 3 buckets - 50, 100, and 150. In general, it’s best to think of the query and aggregation steps as independent - the query selects a set of documents, and then the aggregation buckets those documents without regard to how they were selected. See [note on bucketing range fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-range-field-note.html) for more information and an example.

聚合范围时，存储桶基于返回文档的值。这意味着响应可能包含查询范围之外的桶。例如，如果您的查询查找大于 100 的值，并且您的范围涵盖 50 到 150，并且间隔为 50，则该文档将落在 3 个桶中 - 50、100 和 150。一般来说，最好是将查询和聚合步骤视为独立的 - 查询选择一组文档，然后聚合将这些文档存储在桶中，而不考虑它们是如何被选择的。有关更多信息和示例，请参阅[有关分桶范围字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-range-field-note.html)的说明。

The `hard_bounds` is a counterpart of `extended_bounds` and can limit the range of buckets in the histogram. It is particularly useful in the case of open [data ranges](https://www.elastic.co/guide/en/elasticsearch/reference/master/range.html) that can result in a very large number of buckets.

`hard_bounds`是的对应`extended_bounds`，并且可以限制在直方图桶的范围内。它在可能导致大量桶的开放[数据范围](https://www.elastic.co/guide/en/elasticsearch/reference/master/range.html)的情况下特别有用。

Example:

```console
POST /sales/_search?size=0
{
  "query": {
    "constant_score": { "filter": { "range": { "price": { "to": "500" } } } }
  },
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50,
        "hard_bounds": {
          "min": 100,
          "max": 200
        }
      }
    }
  }
}
```

In this example even though the range specified in the query is up to 500, the histogram will only have 2 buckets starting at 100 and 150. All other buckets will be omitted even if documents that should go to this buckets are present in the results.

在此示例中，即使查询中指定的范围最大为 500，直方图也将只有 2 个从 100 和 150 开始的存储桶。即使结果中存在应进入此存储桶的文档，也将忽略所有其他存储桶。

###  Order

By default the returned buckets are sorted by their `key` ascending, though the order behaviour can be controlled using the `order` setting. Supports the same `order` functionality as the [`Terms Aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order).

默认情况下，返回的存储桶按`key`升序排序，但可以使用`order`设置控制顺序行为。支持相同`order`的功能的[`Terms Aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)。

###  Offset

By default the bucket keys start with 0 and then continue in even spaced steps of `interval`, e.g. if the interval is `10`, the first three buckets (assuming there is data inside them) will be `[0, 10)`, `[10, 20)`, `[20, 30)`. The bucket boundaries can be shifted by using the `offset` option.

默认情况下，bucket 键从 0 开始，然后以 的均匀间隔步长继续`interval`，例如，如果间隔为`10`，则前三个bucket（假设其中有数据）将为`[0, 10)`, `[10, 20)`, `[20, 30)`。可以使用该`offset`选项移动存储区边界。

This can be best illustrated with an example. If there are 10 documents with values ranging from 5 to 14, using interval `10` will result in two buckets with 5 documents each. If an additional offset `5` is used, there will be only one single bucket `[5, 15)` containing all the 10 documents.

这可以用一个例子来最好地说明。如果有 10 个文档的值在 5 到 14 之间，则使用 interval`10`将导致两个桶，每个桶有 5 个文档。如果使用额外的偏移量`5`，则只有一个存储桶`[5, 15)`包含所有 10 个文档。

###  Response Format

By default, the buckets are returned as an ordered array. It is also possible to request the response as a hash instead keyed by the buckets keys:

默认情况下，桶作为有序数组返回。也可以将响应请求为散列，而不是由桶键键控：

```console
POST /sales/_search?size=0
{
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50,
        "keyed": true
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
    "prices": {
      "buckets": {
        "0.0": {
          "key": 0.0,
          "doc_count": 1
        },
        "50.0": {
          "key": 50.0,
          "doc_count": 1
        },
        "100.0": {
          "key": 100.0,
          "doc_count": 0
        },
        "150.0": {
          "key": 150.0,
          "doc_count": 2
        },
        "200.0": {
          "key": 200.0,
          "doc_count": 3
        }
      }
    }
  }
}
```

### Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
POST /sales/_search?size=0
{
  "aggs": {
    "quantity": {
      "histogram": {
        "field": "quantity",
        "interval": 10,
        "missing": 0 (1)
      }
    }
  }
}
```

1. Documents without a value in the `quantity` field will fall into the same bucket as documents that have the value `0`.

   该`quantity`字段中没有值的文档将与具有该值的文档落入同一个桶中`0`。

###  Histogram fields

Running a histogram aggregation over histogram fields computes the total number of counts for each interval.

在直方图字段上运行直方图聚合计算每个间隔的总计数。

For example, executing a histogram aggregation against the following index that stores pre-aggregated histograms with latency metrics (in milliseconds) for different networks:

例如，针对以下索引执行直方图聚合，该索引存储具有不同网络的延迟指标（以毫秒为单位）的预聚合直方图：

```console
PUT metrics_index/_doc/1
{
  "network.name" : "net-1",
  "latency_histo" : {
      "values" : [1, 3, 8, 12, 15],
      "counts" : [3, 7, 23, 12, 6]
   }
}

PUT metrics_index/_doc/2
{
  "network.name" : "net-2",
  "latency_histo" : {
      "values" : [1, 6, 8, 12, 14],
      "counts" : [8, 17, 8, 7, 6]
   }
}

POST /metrics_index/_search?size=0
{
  "aggs": {
    "latency_buckets": {
      "histogram": {
        "field": "latency_histo",
        "interval": 5
      }
    }
  }
}
```

The `histogram` aggregation will sum the counts of each interval computed based on the `values` and return the following output:

`histogram`聚合将总结每个间隔的计数来计算基于所述`values`并返回以下输出：

```console-result
{
  ...
  "aggregations": {
    "prices": {
      "buckets": [
        {
          "key": 0.0,
          "doc_count": 18
        },
        {
          "key": 5.0,
          "doc_count": 48
        },
        {
          "key": 10.0,
          "doc_count": 25
        },
        {
          "key": 15.0,
          "doc_count": 6
        }
      ]
    }
  }
}
```

> IMPORTANT: 
> Histogram aggregation is a bucket aggregation, which partitions documents into buckets rather than calculating metrics over fields like metrics aggregations do. Each bucket represents a collection of documents which sub-aggregations can run on. On the other hand, a histogram field is a pre-aggregated field representing multiple values inside a single field: buckets of numerical data and a count of items/documents for each bucket. This mismatch between the histogram aggregations expected input (expecting raw documents) and the histogram field (that provides summary information) limits the outcome of the aggregation to only the doc counts for each bucket.
>
> **Consequently, when executing a histogram aggregation over a histogram field, no sub-aggregations are allowed.**
>
> 
> 直方图聚合是一种桶聚合，它将文档划分为桶，而不是像度量聚合那样在字段上计算度量。每个存储桶代表子聚合可以运行的文档集合。另一方面，直方图字段是一个预聚合字段，表示单个字段内的多个值：数字数据桶和每个桶的项目/文档计数。直方图聚合预期输入（期望原始文档）和直方图字段（提供摘要信息）之间的这种不匹配将聚合结果限制为每个桶的文档计数。
>
> **因此，在对直方图字段执行直方图聚合时，不允许子聚合。**

Also, when running histogram aggregation over histogram field the `missing` parameter is not supported.

此外，在直方图字段上运行直方图聚合时，`missing`不支持该参数。

#  IP range aggregation

Just like the dedicated [date](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html) range aggregation, there is also a dedicated range aggregation for IP typed fields:

就像专用[日期](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html)范围聚合一样，IP 类型字段也有专用范围聚合：

Example:

```console
GET /ip_addresses/_search
{
  "size": 10,
  "aggs": {
    "ip_ranges": {
      "ip_range": {
        "field": "ip",
        "ranges": [
          { "to": "10.0.0.5" },
          { "from": "10.0.0.5" }
        ]
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
    "ip_ranges": {
      "buckets": [
        {
          "key": "*-10.0.0.5",
          "to": "10.0.0.5",
          "doc_count": 10
        },
        {
          "key": "10.0.0.5-*",
          "from": "10.0.0.5",
          "doc_count": 260
        }
      ]
    }
  }
}
```

IP ranges can also be defined as CIDR masks:

IP 范围也可以定义为 CIDR 掩码：

```console
GET /ip_addresses/_search
{
  "size": 0,
  "aggs": {
    "ip_ranges": {
      "ip_range": {
        "field": "ip",
        "ranges": [
          { "mask": "10.0.0.0/25" },
          { "mask": "10.0.0.127/25" }
        ]
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
    "ip_ranges": {
      "buckets": [
        {
          "key": "10.0.0.0/25",
          "from": "10.0.0.0",
          "to": "10.0.0.128",
          "doc_count": 128
        },
        {
          "key": "10.0.0.127/25",
          "from": "10.0.0.0",
          "to": "10.0.0.128",
          "doc_count": 128
        }
      ]
    }
  }
}
```

###  Keyed Response

Setting the `keyed` flag to `true` will associate a unique string key with each bucket and return the ranges as a hash rather than an array:

将`keyed`标志设置为`true`将与每个存储桶关联一个唯一的字符串键，并将范围作为散列而不是数组返回：

```console
GET /ip_addresses/_search
{
  "size": 0,
  "aggs": {
    "ip_ranges": {
      "ip_range": {
        "field": "ip",
        "ranges": [
          { "to": "10.0.0.5" },
          { "from": "10.0.0.5" }
        ],
        "keyed": true
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
    "ip_ranges": {
      "buckets": {
        "*-10.0.0.5": {
          "to": "10.0.0.5",
          "doc_count": 10
        },
        "10.0.0.5-*": {
          "from": "10.0.0.5",
          "doc_count": 260
        }
      }
    }
  }
}
```

It is also possible to customize the key for each range:

还可以为每个范围自定义键：

```console
GET /ip_addresses/_search
{
  "size": 0,
  "aggs": {
    "ip_ranges": {
      "ip_range": {
        "field": "ip",
        "ranges": [
          { "key": "infinity", "to": "10.0.0.5" },
          { "key": "and-beyond", "from": "10.0.0.5" }
        ],
        "keyed": true
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
    "ip_ranges": {
      "buckets": {
        "infinity": {
          "to": "10.0.0.5",
          "doc_count": 10
        },
        "and-beyond": {
          "from": "10.0.0.5",
          "doc_count": 260
        }
      }
    }
  }
}
```

##  Missing aggregation

A field data based single bucket aggregation, that creates a bucket of all documents in the current document set context that are missing a field value (effectively, missing a field or having the configured NULL value set). This aggregator will often be used in conjunction with other field data bucket aggregators (such as ranges) to return information for all the documents that could not be placed in any of the other buckets due to missing field data values.

基于字段数据的单桶聚合，它在当前文档集上下文中创建一个包含所有缺少字段值的文档的桶（实际上，缺少字段或设置了配置的 NULL 值）。此聚合器通常与其他字段数据桶聚合器（例如范围）结合使用，以返回由于缺少字段数据值而无法放入任何其他桶中的所有文档的信息。

Example:

```console
POST /sales/_search?size=0
{
  "aggs": {
    "products_without_a_price": {
      "missing": { "field": "price" }
    }
  }
}
```

In the above example, we get the total number of products that do not have a price.

在上面的例子中，我们得到了没有价格的产品总数。

Response:

```console-result
{
  ...
  "aggregations": {
    "products_without_a_price": {
      "doc_count": 00
    }
  }
}
```

#  Multi Terms aggregation

A multi-bucket value source based aggregation where buckets are dynamically built - one per unique set of values. The multi terms aggregation is very similar to the [`terms aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order), however in most cases it will be slower than the terms aggregation and will consume more memory. Therefore, if the same set of fields is constantly used, it would be more efficient to index a combined key for this fields as a separate field and use the terms aggregation on this field.

一种基于多桶值源的聚合，其中桶是动态构建的 - 每个唯一的一组值。多术语聚合与 非常相似[`terms aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)，但在大多数情况下，它会比术语聚合慢，并且会消耗更多内存。因此，如果经常使用同一组字段，则将此字段的组合键索引为单独的字段并在此字段上使用术语聚合会更有效。

The multi_term aggregations are the most useful when you need to sort by a number of document or a metric aggregation on a composite key and get top N results. If sorting is not required and all values are expected to be retrieved using nested terms aggregation or [`composite aggregations`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html) will be a faster and more memory efficient solution.

当您需要按文档数量或组合键上的度量聚合进行排序并获得前 N 个结果时，multi_term 聚合最有用。如果不需要排序并且希望使用嵌套术语聚合来检索所有值，或者 [`composite aggregations`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html)将是一种更快且内存效率更高的解决方案。

Example:

```console
GET /products/_search
{
  "aggs": {
    "genres_and_products": {
      "multi_terms": {
        "terms": [{
          "field": "genre" (1)
        }, {
          "field": "product"
        }]
      }
    }
  }
}
```

1. `multi_terms` aggregation can work with the same field types as a [`terms aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order) and supports most of the terms aggregation parameters.

   `multi_terms`聚合可以使用与 a 相同的字段类型， [`terms aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)并支持大多数术语聚合参数。

Response:



```console-result
{
  ...
  "aggregations" : {
    "genres_and_products" : {
      "doc_count_error_upper_bound" : 0,  (1)
      "sum_other_doc_count" : 0,          (2)
      "buckets" : [                       (3)
        {
          "key" : [                       (4)
            "rock",
            "Product A"
          ],
          "key_as_string" : "rock|Product A",
          "doc_count" : 2
        },
        {
          "key" : [
            "electronic",
            "Product B"
          ],
          "key_as_string" : "electronic|Product B",
          "doc_count" : 1
        },
        {
          "key" : [
            "jazz",
            "Product B"
          ],
          "key_as_string" : "jazz|Product B",
          "doc_count" : 1
        },
        {
          "key" : [
            "rock",
            "Product B"
          ],
          "key_as_string" : "rock|Product B",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

1.  an upper bound of the error on the document counts for each term, see <<search-aggregations-bucket-multi-terms-aggregation-approximate-counts,below>

   每个术语的文档计数错误的上限，请参阅<<search-aggregations-bucket-multi-terms-aggregation-approximate-counts,below>

2. when there are lots of unique terms, Elasticsearch only returns the top terms; this number is the sum of the document counts for all buckets that are not part of the response

   当有很多独特的术语时，Elasticsearch 只返回顶部的术语；此数字是不属于响应的所有存储桶的文档计数总和

3.  the list of the top buckets.

   顶级桶的列表。

4. the keys are arrays of values ordered the same ways as expression in the `terms` parameter of the aggregation

   键是值的数组，排序方式`terms`与聚合参数中的表达式相同

By default, the `multi_terms` aggregation will return the buckets for the top ten terms ordered by the `doc_count`. One can change this default behaviour by setting the `size` parameter.

默认情况下，`multi_terms`聚合将返回按 排序的前十个术语的桶`doc_count`。可以通过设置`size`参数来更改此默认行为。

###  Aggregation Parameters

The following parameters are supported. See [`terms aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order) for more detailed explanation of these parameters.

支持以下参数。有关[`terms aggregation`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)这些参数的更详细说明，请参见。

| size                      | Optional. Defines how many term buckets should be returned out of the overall terms list. Defaults to 10.<br />可选的。定义应从整个术语列表中返回多少个术语桶。默认为 10。 |
| ------------------------- | ------------------------------------------------------------ |
| shard_size                | Optional. The higher the requested `size` is, the more accurate the results will be, but also, the more expensive it will be to compute the final results. The default `shard_size` is `(size * 1.5 + 10)`.<br />可选的。要求越高`size`，结果就越准确，但计算最终结果的成本也越高。默认`shard_size`为`(size * 1.5 + 10)`。 |
| show_term_doc_count_error | Optional. Calculates the doc count error on per term basis. Defaults to `false`<br />可选的。计算每个术语的文档计数错误。默认为`false` |
| order                     | Optional. Specifies the order of the buckets. Defaults to the number of documents per bucket. The bucket terms value is used as a tiebreaker for buckets with the same document count.<br />可选的。指定桶的顺序。默认为每个存储桶的文档数。存储桶条款值用作具有相同文档计数的存储桶的决胜局。 |
| min_doc_count             | Optional. The minimal number of documents in a bucket for it to be returned. Defaults to 1.<br />可选的。存储桶中要返回的最小文档数。默认为 1。 |
| shard_min_doc_count       | Optional. The minimal number of documents in a bucket on each shard for it to be returned. Defaults to `min_doc_count`.<br />可选的。每个分片上存储桶中要返回的最小文档数。默认为 `min_doc_count`. |
| collect_mode              | Optional. Specifies the strategy for data collection. The `depth_first` or `breadth_first` modes are supported. Defaults to `breadth_first`.<br />可选的。指定数据收集策略。在`depth_first`或`breadth_first`模式的支持。默认为`breadth_first`. |

###  Script

Generating the terms using a script:

使用脚本生成术语：

```console
GET /products/_search
{
  "runtime_mappings": {
    "genre.length": {
      "type": "long",
      "script": "emit(doc['genre'].value.length())"
    }
  },
  "aggs": {
    "genres_and_products": {
      "multi_terms": {
        "terms": [
          {
            "field": "genre.length"
          },
          {
            "field": "product"
          }
        ]
      }
    }
  }
}
```

Response:



```console-result
{
  ...
  "aggregations" : {
    "genres_and_products" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : [
            4,
            "Product A"
          ],
          "key_as_string" : "4|Product A",
          "doc_count" : 2
        },
        {
          "key" : [
            4,
            "Product B"
          ],
          "key_as_string" : "4|Product B",
          "doc_count" : 2
        },
        {
          "key" : [
            10,
            "Product B"
          ],
          "key_as_string" : "10|Product B",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default if any of the key components are missing the entire document will be ignored but it is also possible to treat them as if they had a value by using the `missing` parameter.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，如果缺少任何关键组件，整个文档将被忽略，但也可以通过使用`missing`参数将它们视为具有值。

```console
GET /products/_search
{
  "aggs": {
    "genres_and_products": {
      "multi_terms": {
        "terms": [
          {
            "field": "genre"
          },
          {
            "field": "product",
            "missing": "Product Z"
          }
        ]
      }
    }
  }
}
```

Response:



```console-result
{
   ...
   "aggregations" : {
    "genres_and_products" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : [
            "rock",
            "Product A"
          ],
          "key_as_string" : "rock|Product A",
          "doc_count" : 2
        },
        {
          "key" : [
            "electronic",
            "Product B"
          ],
          "key_as_string" : "electronic|Product B",
          "doc_count" : 1
        },
        {
          "key" : [
            "electronic",
            "Product Z"
          ],
          "key_as_string" : "electronic|Product Z",  (1)
          "doc_count" : 1
        },
        {
          "key" : [
            "jazz",
            "Product B"
          ],
          "key_as_string" : "jazz|Product B",
          "doc_count" : 1
        },
        {
          "key" : [
            "rock",
            "Product B"
          ],
          "key_as_string" : "rock|Product B",
          "doc_count" : 1
        }
      ]
    }
  }
}
```

1. Documents without a value in the `product` field will fall into the same bucket as documents that have the value `Product Z`.

   该`product`字段中没有值的文档将与具有该值的文档落入同一个桶中`Product Z`。

###  Mixing field types

> WARNING: When aggregating on multiple indices the type of the aggregated field may not be the same in all indices. Some types are compatible with each other (`integer` and `long` or `float` and `double`) but when the types are a mix of decimal and non-decimal number the terms aggregation will promote the non-decimal numbers to decimal numbers. This can result in a loss of precision in the bucket values.
>
> 当聚合多个索引时，聚合字段的类型在所有索引中可能不同。某些类型彼此兼容（`integer`和`long`或`float`和`double`），但是当类型是十进制数和非十进制数的混合时，术语聚合会将非十进制数提升为十进制数。这可能会导致存储桶值的精度损失。

###  Sub aggregation and sorting examples

As most bucket aggregations the `multi_term` supports sub aggregations and ordering the buckets by metrics sub-aggregation:

与大多数存储桶聚合一样，`multi_term`支持子聚合并按指标子聚合对存储桶进行排序：

```console
GET /products/_search
{
  "aggs": {
    "genres_and_products": {
      "multi_terms": {
        "terms": [
          {
            "field": "genre"
          },
          {
            "field": "product"
          }
        ],
        "order": {
          "total_quantity": "desc"
        }
      },
      "aggs": {
        "total_quantity": {
          "sum": {
            "field": "quantity"
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
    "genres_and_products" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : [
            "jazz",
            "Product B"
          ],
          "key_as_string" : "jazz|Product B",
          "doc_count" : 1,
          "total_quantity" : {
            "value" : 10.0
          }
        },
        {
          "key" : [
            "rock",
            "Product A"
          ],
          "key_as_string" : "rock|Product A",
          "doc_count" : 2,
          "total_quantity" : {
            "value" : 9.0
          }
        },
        {
          "key" : [
            "electronic",
            "Product B"
          ],
          "key_as_string" : "electronic|Product B",
          "doc_count" : 1,
          "total_quantity" : {
            "value" : 3.0
          }
        },
        {
          "key" : [
            "rock",
            "Product B"
          ],
          "key_as_string" : "rock|Product B",
          "doc_count" : 1,
          "total_quantity" : {
            "value" : 1.0
          }
        }
      ]
    }
  }
}
```

# Nested aggregation

A special single bucket aggregation that enables aggregating nested documents.

一种特殊的单桶聚合，可以聚合嵌套文档。

For example, lets say we have an index of products, and each product holds the list of resellers - each having its own price for the product. The mapping could look like:

```console
PUT /products
{
  "mappings": {
    "properties": {
      "resellers": { (1)
        "type": "nested",
        "properties": {
          "reseller": { "type": "text" },
          "price": { "type": "double" }
        }
      }
    }
  }
}
```

1. `resellers` is an array that holds nested documents.

   `resellers` 是一个包含嵌套文档的数组。

The following request adds a product with two resellers:

以下请求添加了具有两个经销商的产品：

```console
PUT /products/_doc/0
{
  "name": "LED TV", (1)
  "resellers": [
    {
      "reseller": "companyA",
      "price": 350
    },
    {
      "reseller": "companyB",
      "price": 500
    }
  ]
}
```

1.  We are using a dynamic mapping for the `name` attribute.

   我们对`name`属性使用动态映射。

The following request returns the minimum price a product can be purchased for:

以下请求返回可以购买产品的最低价格：

```console
GET /products/_search
{
  "query": {
    "match": { "name": "led tv" }
  },
  "aggs": {
    "resellers": {
      "nested": {
        "path": "resellers"
      },
      "aggs": {
        "min_price": { "min": { "field": "resellers.price" } }
      }
    }
  }
}
```

As you can see above, the nested aggregation requires the `path` of the nested documents within the top level documents. Then one can define any type of aggregation over these nested documents.

正如您在上面看到的，嵌套聚合需要`path`顶级文档中的嵌套文档。然后可以在这些嵌套文档上定义任何类型的聚合。

Response:

```console-result
{
  ...
  "aggregations": {
    "resellers": {
      "doc_count": 2,
      "min_price": {
        "value": 350
      }
    }
  }
}
```

#  Parent aggregation

A special single bucket aggregation that selects parent documents that have the specified type, as defined in a [`join` field](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html).

一种特殊的单桶聚合，它选择具有指定类型的父文档，如[`join`字段中](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)所定义。

This aggregation has a single option:

此聚合有一个选项：

- `type` - The child type that should be selected.

  `type` - 应该选择的子类型。

For example, let’s say we have an index of questions and answers. The answer type has the following `join` field in the mapping:

例如，假设我们有一个问题和答案的索引。答案类型`join`在映射中具有以下字段：

```console
PUT parent_example
{
  "mappings": {
     "properties": {
       "join": {
         "type": "join",
         "relations": {
           "question": "answer"
         }
       }
     }
  }
}
```

The `question` document contain a tag field and the `answer` documents contain an owner field. With the `parent` aggregation the owner buckets can be mapped to the tag buckets in a single request even though the two fields exist in two different kinds of documents.

该`question`文件包含标签字段和`answer`文档包含所有者场。通过`parent` 聚合，所有者桶可以在单个请求中映射到标签桶，即使这两个字段存在于两种不同类型的文档中。

An example of a question document:

```console
PUT parent_example/_doc/1
{
  "join": {
    "name": "question"
  },
  "body": "<p>I have Windows 2003 server and i bought a new Windows 2008 server...",
  "title": "Whats the best way to file transfer my site from server to a newer one?",
  "tags": [
    "windows-server-2003",
    "windows-server-2008",
    "file-transfer"
  ]
}
```

Examples of `answer` documents:

```console
PUT parent_example/_doc/2?routing=1
{
  "join": {
    "name": "answer",
    "parent": "1"
  },
  "owner": {
    "location": "Norfolk, United Kingdom",
    "display_name": "Sam",
    "id": 48
  },
  "body": "<p>Unfortunately you're pretty much limited to FTP...",
  "creation_date": "2009-05-04T13:45:37.030"
}

PUT parent_example/_doc/3?routing=1&refresh
{
  "join": {
    "name": "answer",
    "parent": "1"
  },
  "owner": {
    "location": "Norfolk, United Kingdom",
    "display_name": "Troll",
    "id": 49
  },
  "body": "<p>Use Linux...",
  "creation_date": "2009-05-05T13:45:37.030"
}
```

The following request can be built that connects the two together:

可以构建以下请求将两者连接在一起：

```console
POST parent_example/_search?size=0
{
  "aggs": {
    "top-names": {
      "terms": {
        "field": "owner.display_name.keyword",
        "size": 10
      },
      "aggs": {
        "to-questions": {
          "parent": {
            "type" : "answer" (1)
          },
          "aggs": {
            "top-tags": {
              "terms": {
                "field": "tags.keyword",
                "size": 10
              }
            }
          }
        }
      }
    }
  }
}
```

1.  The `type` points to type / mapping with the name `answer`.

    所述`type`点输入/映射的名称`answer`。

The above example returns the top answer owners and per owner the top question tags.

上面的示例返回最热门的答案所有者和每个所有者的最热门问题标签。

Possible response:

```console-result
{
  "took": 9,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total" : {
      "value": 3,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "top-names": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "Sam",
          "doc_count": 1, (1)
          "to-questions": {
            "doc_count": 1, (2)
            "top-tags": {
              "doc_count_error_upper_bound": 0,
              "sum_other_doc_count": 0,
              "buckets": [
                {
                  "key": "file-transfer",
                  "doc_count": 1
                },
                {
                  "key": "windows-server-2003",
                  "doc_count": 1
                },
                {
                  "key": "windows-server-2008",
                  "doc_count": 1
                }
              ]
            }
          }
        },
        {
          "key": "Troll",
          "doc_count": 1,
          "to-questions": {
            "doc_count": 1,
            "top-tags": {
              "doc_count_error_upper_bound": 0,
              "sum_other_doc_count": 0,
              "buckets": [
                {
                  "key": "file-transfer",
                  "doc_count": 1
                },
                {
                  "key": "windows-server-2003",
                  "doc_count": 1
                },
                {
                  "key": "windows-server-2008",
                  "doc_count": 1
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

1. The number of answer documents with the tag `Sam`, `Troll`, etc.

    与标签应答文件的数量`Sam`，`Troll`等等。

2. The number of question documents that are related to answer documents with the tag `Sam`, `Troll`, etc.

    这涉及到回答文档与标签问题的文件的数量`Sam`，`Troll`等等。

#  Range aggregation

A multi-bucket value source based aggregation that enables the user to define a set of ranges - each representing a bucket. During the aggregation process, the values extracted from each document will be checked against each bucket range and "bucket" the relevant/matching document. Note that this aggregation includes the `from` value and excludes the `to` value for each range.

基于多桶值源的聚合，使用户能够定义一组范围 - 每个范围代表一个桶。在聚合过程中，将从每个文档中提取的值根据每个存储区范围进行检查，并将相关/匹配文档“存储”到“存储区”中。请注意，此聚合包括每个范围的`from`值并排除该`to`值。

Example:

```console
GET /_search
{
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100.0 },
          { "from": 100.0, "to": 200.0 },
          { "from": 200.0 }
        ]
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
    "price_ranges": {
      "buckets": [
        {
          "key": "*-100.0",
          "to": 100.0,
          "doc_count": 2
        },
        {
          "key": "100.0-200.0",
          "from": 100.0,
          "to": 200.0,
          "doc_count": 2
        },
        {
          "key": "200.0-*",
          "from": 200.0,
          "doc_count": 3
        }
      ]
    }
  }
}
```

###  Keyed Response

Setting the `keyed` flag to `true` will associate a unique string key with each bucket and return the ranges as a hash rather than an array:

将`keyed`标志设置为`true`将与每个存储桶关联一个唯一的字符串键，并将范围作为散列而不是数组返回：

```console
GET /_search
{
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "keyed": true,
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 200 },
          { "from": 200 }
        ]
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
    "price_ranges": {
      "buckets": {
        "*-100.0": {
          "to": 100.0,
          "doc_count": 2
        },
        "100.0-200.0": {
          "from": 100.0,
          "to": 200.0,
          "doc_count": 2
        },
        "200.0-*": {
          "from": 200.0,
          "doc_count": 3
        }
      }
    }
  }
}
```

It is also possible to customize the key for each range:

还可以为每个范围自定义键：

```console
GET /_search
{
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "keyed": true,
        "ranges": [
          { "key": "cheap", "to": 100 },
          { "key": "average", "from": 100, "to": 200 },
          { "key": "expensive", "from": 200 }
        ]
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
    "price_ranges": {
      "buckets": {
        "cheap": {
          "to": 100.0,
          "doc_count": 2
        },
        "average": {
          "from": 100.0,
          "to": 200.0,
          "doc_count": 2
        },
        "expensive": {
          "from": 200.0,
          "doc_count": 3
        }
      }
    }
  }
}
```

###  Script

If the data in your documents doesn’t exactly match what you’d like to aggregate, use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html). For example, if you need to apply a particular currency conversion rate:

如果文档中的数据与您想要聚合的数据不完全匹配，请使用[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)。例如，如果您需要应用特定的货币兑换率：

```console
GET /_search
{
  "runtime_mappings": {
    "price.euros": {
      "type": "double",
      "script": {
        "source": """
          emit(doc['price'].value * params.conversion_rate)
        """,
        "params": {
          "conversion_rate": 0.835526591
        }
      }
    }
  },
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price.euros",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 200 },
          { "from": 200 }
        ]
      }
    }
  }
}
```

###  Sub Aggregations

The following example, not only "bucket" the documents to the different buckets but also computes statistics over the prices in each price range

以下示例不仅将文档“存储”到不同的存储桶，还计算每个价格范围内的价格统计信息

```console
GET /_search
{
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 200 },
          { "from": 200 }
        ]
      },
      "aggs": {
        "price_stats": {
          "stats": { "field": "price" }
        }
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
    "price_ranges": {
      "buckets": [
        {
          "key": "*-100.0",
          "to": 100.0,
          "doc_count": 2,
          "price_stats": {
            "count": 2,
            "min": 10.0,
            "max": 50.0,
            "avg": 30.0,
            "sum": 60.0
          }
        },
        {
          "key": "100.0-200.0",
          "from": 100.0,
          "to": 200.0,
          "doc_count": 2,
          "price_stats": {
            "count": 2,
            "min": 150.0,
            "max": 175.0,
            "avg": 162.5,
            "sum": 325.0
          }
        },
        {
          "key": "200.0-*",
          "from": 200.0,
          "doc_count": 3,
          "price_stats": {
            "count": 3,
            "min": 200.0,
            "max": 200.0,
            "avg": 200.0,
            "sum": 600.0
          }
        }
      ]
    }
  }
}
```

#  Rare terms aggregation

A multi-bucket value source based aggregation which finds "rare" terms — terms that are at the long-tail of the distribution and are not frequent. Conceptually, this is like a `terms` aggregation that is sorted by `_count` ascending. As noted in the [terms aggregation docs](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order), actually ordering a `terms` agg by count ascending has unbounded error. Instead, you should use the `rare_terms` aggregation

一种基于多桶价值源的聚合，它可以找到“稀有”术语——分布的长尾术语并且不频繁。从概念上讲，这就像一个`terms`按`_count`升序排序的聚合。如[术语聚合文档中所述](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)，实际上`terms`按计数升序对[聚合进行](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)排序具有无限错误。相反，您应该使用`rare_terms` 聚合

###  Syntax

A `rare_terms` aggregation looks like this in isolation:

单独的`rare_terms`聚合看起来像这样：

```js
{
  "rare_terms": {
    "field": "the_field",
    "max_doc_count": 1
  }
}
```

**Table 45. `rare_terms` Parameters**

| Parameter Name  | Description                                                  | Required | Default Value |
| --------------- | ------------------------------------------------------------ | -------- | ------------- |
| `field`         | The field we wish to find rare terms in<br />我们希望在其中找到稀有术语的领域 | Required |               |
| `max_doc_count` | The maximum number of documents a term should appear in.<br />一个术语应该出现在的最大文档数。 | Optional | `1`           |
| `precision`     | The precision of the internal CuckooFilters. Smaller precision leads to better approximation, but higher memory usage. Cannot be smaller than `0.00001`<br />内部 CuckooFilters 的精度。更小的精度导致更好的近似，但更高的内存使用。不能小于`0.00001` | Optional | `0.01`        |
| `include`       | Terms that should be included in the aggregation<br />应包含在聚合中的术语 | Optional |               |
| `exclude`       | Terms that should be excluded from the aggregation<br />应从聚合中排除的术语 | Optional |               |
| `missing`       | The value that should be used if a document does not have the field being aggregated<br />如果文档没有要聚合的字段，则应使用的值 | Optional |               |

Example:

```console
GET /_search
{
  "aggs": {
    "genres": {
      "rare_terms": {
        "field": "genre"
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
    "genres": {
      "buckets": [
        {
          "key": "swing",
          "doc_count": 1
        }
      ]
    }
  }
}
```

In this example, the only bucket that we see is the "swing" bucket, because it is the only term that appears in one document. If we increase the `max_doc_count` to `2`, we’ll see some more buckets:

在这个例子中，我们看到的唯一存储桶是“swing”存储桶，因为它是一个文档中唯一出现的术语。如果我们增加了`max_doc_count`对`2`，我们会看到一些桶：

```console
GET /_search
{
  "aggs": {
    "genres": {
      "rare_terms": {
        "field": "genre",
        "max_doc_count": 2
      }
    }
  }
}
```

This now shows the "jazz" term which has a `doc_count` of 2":

这现在显示了“爵士”术语，它的`doc_count`值为 2”：

```console-result
{
  ...
  "aggregations": {
    "genres": {
      "buckets": [
        {
          "key": "swing",
          "doc_count": 1
        },
        {
          "key": "jazz",
          "doc_count": 2
        }
      ]
    }
  }
}
```

### Maximum document count

The `max_doc_count` parameter is used to control the upper bound of document counts that a term can have. There is not a size limitation on the `rare_terms` agg like `terms` agg has. This means that terms which match the `max_doc_count` criteria will be returned. The aggregation functions in this manner to avoid the order-by-ascending issues that afflict the `terms` aggregation.

该`max_doc_count`参数用于控制术语可以具有的文档计数的上限。没有`rare_terms`像`terms`agg那样对agg的大小限制。这意味着`max_doc_count`将返回符合条件的术语。聚合以这种方式起作用以避免影响`terms`聚合的按升序排序问题。

This does, however, mean that a large number of results can be returned if chosen incorrectly. To limit the danger of this setting, the maximum `max_doc_count` is 100.

但是，这确实意味着如果选择不正确，可能会返回大量结果。为了限制此设置的危险，最大值`max_doc_count`为 100。

###  Max Bucket Limit

The Rare Terms aggregation is more liable to trip the `search.max_buckets` soft limit than other aggregations due to how it works. The `max_bucket` soft-limit is evaluated on a per-shard basis while the aggregation is collecting results. It is possible for a term to be "rare" on a shard but become "not rare" once all the shard results are merged together. This means that individual shards tend to collect more buckets than are truly rare, because they only have their own local view. This list is ultimately pruned to the correct, smaller list of rare terms on the coordinating node… but a shard may have already tripped the `max_buckets` soft limit and aborted the request.

由于其工作方式，罕见术语聚合`search.max_buckets`比其他聚合更容易突破软限制。`max_bucket`当聚合收集结果时，软限制是在每个分片的基础上评估的。一个词条在分片上可能是“罕见的”，但一旦所有分片结果合并在一起就变成“不罕见”。这意味着单个分片倾向于收集比真正稀有的桶更多的桶，因为它们只有自己的局部视图。该列表最终会被修剪为协调节点上正确的、较小的稀有术语列表......但分片可能已经`max_buckets`超出了软限制并中止了请求。

When aggregating on fields that have potentially many "rare" terms, you may need to increase the `max_buckets` soft limit. Alternatively, you might need to find a way to filter the results to return fewer rare values (smaller time span, filter by category, etc), or re-evaluate your definition of "rare" (e.g. if something appears 100,000 times, is it truly "rare"?)

在聚合可能具有许多“稀有”术语的字段时，您可能需要增加`max_buckets`软限制。或者，您可能需要找到一种方法来过滤结果以返回较少的稀有值（较小的时间跨度、按类别过滤等），或重新评估您对“稀有”的定义（例如，如果某物出现 100,000 次，是不是真的“稀有”？）

###  Document counts are approximate

The naive way to determine the "rare" terms in a dataset is to place all the values in a map, incrementing counts as each document is visited, then return the bottom `n` rows. This does not scale beyond even modestly sized data sets. A sharded approach where only the "top n" values are retained from each shard (ala the `terms` aggregation) fails because the long-tail nature of the problem means it is impossible to find the "top n" bottom values without simply collecting all the values from all shards.

确定数据集中“稀有”术语的天真方法是将所有值放在映射中，在访问每个文档时递增计数，然后返回底部`n`行。这甚至不会超出中等规模的数据集。仅保留每个分片中的“前 n”个值（即`terms`聚合）的分片方法失败，因为问题的长尾性质意味着如果不简单地收集所有值就不可能找到“前 n”个底部值来自所有碎片。

Instead, the Rare Terms aggregation uses a different approximate algorithm:

相反，稀有术语聚合使用不同的近似算法：

1. Values are placed in a map the first time they are seen.

   值在第一次被看到时被放置在地图中。

2. Each addition occurrence of the term increments a counter in the map

   该术语的每次添加都会增加映射中的计数器

3. If the counter > the `max_doc_count` threshold, the term is removed from the map and placed in a [CuckooFilter](https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf)

   如果计数器 >`max_doc_count`阈值，则该术语将从地图中删除并放置在 [CuckooFilter 中](https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf)

4. The CuckooFilter is consulted on each term. If the value is inside the filter, it is known to be above the threshold already and skipped.

   每个术语都会咨询 CuckooFilter。如果该值在过滤器内，则已知它已经高于阈值并被跳过。

After execution, the map of values is the map of "rare" terms under the `max_doc_count` threshold. This map and CuckooFilter are then merged with all other shards. If there are terms that are greater than the threshold (or appear in a different shard’s CuckooFilter) the term is removed from the merged list. The final map of values is returned to the user as the "rare" terms.

执行后，值的映射是`max_doc_count`阈值下“稀有”术语的映射。然后将此地图和 CuckooFilter 与所有其他分片合并。如果存在大于阈值（或出现在不同分片的 CuckooFilter 中）的术语，则该术语将从合并列表中删除。最终的值映射作为“稀有”术语返回给用户。

CuckooFilters have the possibility of returning false positives (they can say a value exists in their collection when it actually does not). Since the CuckooFilter is being used to see if a term is over threshold, this means a false positive from the CuckooFilter will mistakenly say a value is common when it is not (and thus exclude it from it final list of buckets). Practically, this means the aggregations exhibits false-negative behavior since the filter is being used "in reverse" of how people generally think of approximate set membership sketches.

CuckooFilters 有可能返回误报（他们可以说他们的集合中存在一个值，但实际上并不存在）。由于 CuckooFilter 被用于查看一个术语是否超过阈值，这意味着来自 CuckooFilter 的误报会错误地认为一个值是常见的，而实际上它不是（因此将它从它的最终存储桶列表中排除）。实际上，这意味着聚合表现出假阴性行为，因为过滤器的使用与人们通常对近似集合成员草图的看法“相反”。

CuckooFilters are described in more detail in the paper:

CuckooFilters 在论文中有更详细的描述：

[Fan, Bin, et al. "Cuckoo filter: Practically better than bloom."](https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf) Proceedings of the 10th ACM International on Conference on emerging Networking Experiments and Technologies. ACM, 2014.

[范斌等人。“布谷鸟过滤器：实际上比布隆好。” ](https://www.cs.cmu.edu/~dga/papers/cuckoo-conext2014.pdf)第 10 届 ACM 国际新兴网络实验和技术会议论文集。ACM，2014 年。

###  Precision

Although the internal CuckooFilter is approximate in nature, the false-negative rate can be controlled with a `precision` parameter. This allows the user to trade more runtime memory for more accurate results.

虽然内部 CuckooFilter 本质上是近似的，但可以通过`precision`参数控制假阴性率 。这允许用户用更多的运行时内存来换取更准确的结果。

The default precision is `0.001`, and the smallest (e.g. most accurate and largest memory overhead) is `0.00001`. Below are some charts which demonstrate how the accuracy of the aggregation is affected by precision and number of distinct terms.

默认精度是`0.001`，最小的（例如最准确和最大的内存开销）是`0.00001`。下面是一些图表，展示了聚合的准确性如何受到不同术语的精度和数量的影响。

The X-axis shows the number of distinct values the aggregation has seen, and the Y-axis shows the percent error. Each line series represents one "rarity" condition (ranging from one rare item to 100,000 rare items). For example, the orange "10" line means ten of the values were "rare" (`doc_count == 1`), out of 1-20m distinct values (where the rest of the values had `doc_count > 1`)

X 轴显示聚合所看到的不同值的数量，Y 轴显示百分比误差。每个系列代表一个“稀有”条件（从一件稀有物品到 100,000 件稀有物品）。例如，橙色的“10”线表示其中 10 个值是“稀有”( `doc_count == 1`)，在 1-20m 不同值中（其余值具有`doc_count > 1`）

This first chart shows precision `0.01`:

第一个图表显示了精度`0.01`：

![accuracy 01](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/rare_terms/accuracy_01.png)

And precision `0.001` (the default):

和精度`0.001`（默认）：

![accuracy 001](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/rare_terms/accuracy_001.png)

And finally `precision 0.0001`:

最后`precision 0.0001`：

![accuracy 0001](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/rare_terms/accuracy_0001.png)

The default precision of `0.001` maintains an accuracy of < 2.5% for the tested conditions, and accuracy slowly degrades in a controlled, linear fashion as the number of distinct values increases.

的默认精度`0.001`在测试条件下保持 < 2.5% 的精度，并且随着不同值数量的增加，精度以受控的线性方式缓慢下降。

The default precision of `0.001` has a memory profile of `1.748⁻⁶ * n` bytes, where `n` is the number of distinct values the aggregation has seen (it can also be roughly eyeballed, e.g. 20 million unique values is about 30mb of memory). The memory usage is linear to the number of distinct values regardless of which precision is chosen, the precision only affects the slope of the memory profile as seen in this chart:

的默认精度`0.001`有一个`1.748⁻⁶ * n`字节的内存配置文件，其中`n`是聚合看到的不同值的数量（也可以粗略地观察，例如 2000 万个唯一值大约是 30mb 的内存）。无论选择哪种精度，内存使用量都与不同值的数量呈线性关系，精度仅影响内存配置文件的斜率，如下图所示：

![memory](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/rare_terms/memory.png)

For comparison, an equivalent terms aggregation at 20 million buckets would be roughly `20m * 69b == ~1.38gb` (with 69 bytes being a very optimistic estimate of an empty bucket cost, far lower than what the circuit breaker accounts for). So although the `rare_terms` agg is relatively heavy, it is still orders of magnitude smaller than the equivalent terms aggregation

相比之下，2000 万个存储桶的等效项聚合将大致为 `20m * 69b == ~1.38gb`（69 字节是对空存储桶成本的非常乐观的估计，远低于断路器所考虑的成本）。所以虽然`rare_terms`agg 比较重，但仍然比等效项聚合小几个数量级

###  Filtering Values

It is possible to filter the values for which buckets will be created. This can be done using the `include` and `exclude` parameters which are based on regular expression strings or arrays of exact values. Additionally, `include` clauses can filter using `partition` expressions.

可以过滤将为其创建存储桶的值。这可以使用基于正则表达式字符串或精确值数组的`include`和 `exclude`参数来完成。此外， `include`子句可以使用`partition`表达式进行过滤。

####  Filtering Values with regular expressions

```console
GET /_search
{
  "aggs": {
    "genres": {
      "rare_terms": {
        "field": "genre",
        "include": "swi*",
        "exclude": "electro*"
      }
    }
  }
}
```

In the above example, buckets will be created for all the tags that starts with `swi`, except those starting with `electro` (so the tag `swing` will be aggregated but not `electro_swing`). The `include` regular expression will determine what values are "allowed" to be aggregated, while the `exclude` determines the values that should not be aggregated. When both are defined, the `exclude` has precedence, meaning, the `include` is evaluated first and only then the `exclude`.

在上面的示例中，将为所有以 开头的标签创建桶`swi`，除了以 开头`electro`的标签（因此标签`swing`将被聚合但不会被聚合`electro_swing`）。该`include`正则表达式将决定什么样的价值观是“允许”被聚集，而`exclude`决定不应该聚合的价值。当两者都定义时， the`exclude`具有优先级，含义，`include`首先评估 ，然后才评估`exclude`。

The syntax is the same as [regexp queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html).

语法与正则[表达式查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html)相同。

####  Filtering Values with exact values

For matching based on exact values the `include` and `exclude` parameters can simply take an array of strings that represent the terms as they are found in the index:

对于基于精确值的匹配，`include`和`exclude`参数可以简单地采用表示在索引中找到的术语的字符串数组：

```console
GET /_search
{
  "aggs": {
    "genres": {
      "rare_terms": {
        "field": "genre",
        "include": [ "swing", "rock" ],
        "exclude": [ "jazz" ]
      }
    }
  }
}
```

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
GET /_search
{
  "aggs": {
    "genres": {
      "rare_terms": {
        "field": "genre",
        "missing": "N/A" (1)
      }
    }
  }
}
```

1.  Documents without a value in the `tags` field will fall into the same bucket as documents that have the value `N/A`.
2.  该`tags`字段中没有值的文档将与具有该值的文档落入同一个桶中`N/A`。

###  Nested, RareTerms, and scoring sub-aggregations

The RareTerms aggregation has to operate in `breadth_first` mode, since it needs to prune terms as doc count thresholds are breached. This requirement means the RareTerms aggregation is incompatible with certain combinations of aggregations that require `depth_first`. In particular, scoring sub-aggregations that are inside a `nested` force the entire aggregation tree to run in `depth_first` mode. This will throw an exception since RareTerms is unable to process `depth_first`.

RareTerms 聚合必须在`breadth_first`模式下运行，因为它需要在违反文档计数阈值时修剪术语。此要求意味着 RareTerms 聚合与某些需要`depth_first`. 特别是，对内部的子聚合进行评分会`nested`强制整个聚合树以`depth_first`模式运行。由于 RareTerms 无法处理，这将引发异常`depth_first`。

As a concrete example, if `rare_terms` aggregation is the child of a `nested` aggregation, and one of the child aggregations of `rare_terms` needs document scores (like a `top_hits` aggregation), this will throw an exception.

作为一个具体的例子，如果`rare_terms`聚合是聚合的子级`nested`，并且`rare_terms` 需要文档分数的子聚合之一（如`top_hits`聚合），这将引发异常。

#  Reverse nested aggregation

A special single bucket aggregation that enables aggregating on parent docs from nested documents. Effectively this aggregation can break out of the nested block structure and link to other nested structures or the root document, which allows nesting other aggregations that aren’t part of the nested object in a nested aggregation.

一种特殊的单桶聚合，可以从嵌套文档中聚合父文档。有效地，此聚合可以打破嵌套块结构并链接到其他嵌套结构或根文档，这允许在嵌套聚合中嵌套不属于嵌套对象的其他聚合。

The `reverse_nested` aggregation must be defined inside a `nested` aggregation.

该`reverse_nested`聚合必须在内部被定义`nested`聚集。

**Options:**

- `path` - Which defines to what nested object field should be joined back. The default is empty, which means that it joins back to the root / main document level. The path cannot contain a reference to a nested object field that falls outside the `nested` aggregation’s nested structure a `reverse_nested` is in.

  `path`- 它定义了应该连接回哪个嵌套对象字段。默认为空，这意味着它连接回根/主文档级别。路径不能包含对位于`nested`聚合的嵌套结构 a `reverse_nested`is in之外的嵌套对象字段的引用。

For example, lets say we have an index for a ticket system with issues and comments. The comments are inlined into the issue documents as nested documents. The mapping could look like:

例如，假设我们有一个带有问题和评论的工单系统的索引。注释作为嵌套文档内联到问题文档中。映射可能如下所示：

```console
PUT /issues
{
  "mappings": {
    "properties": {
      "tags": { "type": "keyword" },
      "comments": {                            (1)
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

1. The `comments` is an array that holds nested documents under the `issue` object.

   `comments`是保持下嵌套文档的阵列`issue`对象。

The following aggregations will return the top commenters' username that have commented and per top commenter the top tags of the issues the user has commented on:

以下聚合将返回评论最多的评论者的用户名，以及每个评论者返回用户评论过的问题的顶部标签：

```console
GET /issues/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "comments": {
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "top_usernames": {
          "terms": {
            "field": "comments.username"
          },
          "aggs": {
            "comment_to_issue": {
              "reverse_nested": {}, (1)
              "aggs": {
                "top_tags_per_comment": {
                  "terms": {
                    "field": "tags"
                  }
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

As you can see above, the `reverse_nested` aggregation is put in to a `nested` aggregation as this is the only place in the dsl where the `reverse_nested` aggregation can be used. Its sole purpose is to join back to a parent doc higher up in the nested structure.

正如您在上面看到的，`reverse_nested`聚合被放入`nested`聚合中，因为这是 dsl 中唯一`reverse_nested`可以使用聚合的地方。它的唯一目的是连接回嵌套结构中更高的父文档。

1.  A `reverse_nested` aggregation that joins back to the root / main document level, because no `path` has been defined. Via the `path` option the `reverse_nested` aggregation can join back to a different level, if multiple layered nested object types have been defined in the mapping

   `reverse_nested`连接回根/主文档级别的聚合，因为没有`path`定义。如果在映射中定义了多个分层嵌套对象类型，则通过该`path`选项，`reverse_nested`聚合可以连接回不同的级别

Possible response snippet:

```console-result
{
  "aggregations": {
    "comments": {
      "doc_count": 1,
      "top_usernames": {
        "doc_count_error_upper_bound" : 0,
        "sum_other_doc_count" : 0,
        "buckets": [
          {
            "key": "username_1",
            "doc_count": 1,
            "comment_to_issue": {
              "doc_count": 1,
              "top_tags_per_comment": {
                "doc_count_error_upper_bound" : 0,
                "sum_other_doc_count" : 0,
                "buckets": [
                  {
                    "key": "tag_1",
                    "doc_count": 1
                  }
                  ...
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

# Sampler aggregation

A filtering aggregation used to limit any sub aggregations' processing to a sample of the top-scoring documents.

过滤聚合，用于将任何子聚合的处理限制为得分最高的文档样本。

**Example use cases:**

- Tightening the focus of analytics to high-relevance matches rather than the potentially very long tail of low-quality matches

  将分析重点集中在高相关性匹配上，而不是潜在的低质量匹配的长尾

- Reducing the running cost of aggregations that can produce useful results using only samples e.g. `significant_terms`

  降低仅使用样本即可产生有用结果的聚合的运行成本，例如 `significant_terms`

Example:

A query on StackOverflow data for the popular term `javascript` OR the rarer term `kibana` will match many documents - most of them missing the word Kibana. To focus the `significant_terms` aggregation on top-scoring documents that are more likely to match the most interesting parts of our query we use a sample.

在 StackOverflow 数据上查询流行词`javascript`或稀有词 `kibana`将匹配许多文档 - 其中大多数都缺少 Kibana 一词。为了将`significant_terms`聚合集中在更有可能匹配我们查询中最有趣部分的得分最高的文档上，我们使用了一个示例。

```console
POST /stackoverflow/_search?size=0
{
  "query": {
    "query_string": {
      "query": "tags:kibana OR tags:javascript"
    }
  },
  "aggs": {
    "sample": {
      "sampler": {
        "shard_size": 200
      },
      "aggs": {
        "keywords": {
          "significant_terms": {
            "field": "tags",
            "exclude": [ "kibana", "javascript" ]
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
  ...
  "aggregations": {
    "sample": {
      "doc_count": 200, (1)
      "keywords": {
        "doc_count": 200,
        "bg_count": 650,
        "buckets": [
          {
            "key": "elasticsearch",
            "doc_count": 150,
            "score": 1.078125,
            "bg_count": 200
          },
          {
            "key": "logstash",
            "doc_count": 50,
            "score": 0.5625,
            "bg_count": 50
          }
        ]
      }
    }
  }
}
```

1. 200 documents were sampled in total. The cost of performing the nested significant_terms aggregation was therefore limited rather than unbounded.

   总共抽取了200份文件。因此，执行嵌套的 important_terms 聚合的成本是有限的而不是无限的。

Without the `sampler` aggregation the request query considers the full "long tail" of low-quality matches and therefore identifies less significant terms such as `jquery` and `angular` rather than focusing on the more insightful Kibana-related terms.

如果没有`sampler`聚合，请求查询会考虑低质量匹配的完整“长尾”，因此识别不太重要的术语，例如`jquery`和 ，`angular`而不是专注于更有洞察力的 Kibana 相关术语。

```console
POST /stackoverflow/_search?size=0
{
  "query": {
    "query_string": {
      "query": "tags:kibana OR tags:javascript"
    }
  },
  "aggs": {
    "low_quality_keywords": {
      "significant_terms": {
        "field": "tags",
        "size": 3,
        "exclude": [ "kibana", "javascript" ]
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
    "low_quality_keywords": {
      "doc_count": 600,
      "bg_count": 650,
      "buckets": [
        {
          "key": "angular",
          "doc_count": 200,
          "score": 0.02777,
          "bg_count": 200
        },
        {
          "key": "jquery",
          "doc_count": 200,
          "score": 0.02777,
          "bg_count": 200
        },
        {
          "key": "logstash",
          "doc_count": 50,
          "score": 0.0069,
          "bg_count": 50
        }
      ]
    }
  }
}
```

###  shard_size

The `shard_size` parameter limits how many top-scoring documents are collected in the sample processed on each shard. The default value is 100.

该`shard_size`参数限制了在每个分片上处理的样本中收集了多少得分最高的文档。默认值为 100。

###  Limitations

####  Cannot be nested under `breadth_first` aggregations

Being a quality-based filter the sampler aggregation needs access to the relevance score produced for each document. It therefore cannot be nested under a `terms` aggregation which has the `collect_mode` switched from the default `depth_first` mode to `breadth_first` as this discards scores. In this situation an error will be thrown.

作为基于质量的过滤器，采样器聚合需要访问为每个文档生成的相关性分数。因此，它不能嵌套在从默认模式切换到的`terms`聚合下，因为这会丢弃分数。在这种情况下，将抛出错误。`collect_mode``depth_first``breadth_first`

#  Significant terms aggregation

An aggregation that returns interesting or unusual occurrences of terms in a set.

返回集合中有趣或异常出现的术语的聚合。

**Example use cases:**

- Suggesting "H5N1" when users search for "bird flu" in text

  用户在文本中搜索“禽流感”时提示“H5N1”

- Identifying the merchant that is the "common point of compromise" from the transaction history of credit card owners reporting loss

  从信用卡持卡人挂失的交易历史中识别出“共同妥协点”的商户

- Suggesting keywords relating to stock symbol $ATI for an automated news classifier

  为自动新闻分类器建议与股票代码 $ATI 相关的关键字

- Spotting the fraudulent doctor who is diagnosing more than their fair share of whiplash injuries

  发现欺诈性医生诊断出的鞭伤伤害超过了他们的公平份额

- Spotting the tire manufacturer who has a disproportionate number of blow-outs

  发现爆胎次数不成比例的轮胎制造商

In all these cases the terms being selected are not simply the most popular terms in a set. They are the terms that have undergone a significant change in popularity measured between a *foreground* and *background* set. If the term "H5N1" only exists in 5 documents in a 10 million document index and yet is found in 4 of the 100 documents that make up a user’s search results that is significant and probably very relevant to their search. 5/10,000,000 vs 4/100 is a big swing in frequency.

在所有这些情况下，所选择的术语不仅仅是一组中最流行的术语。它们是在*前景*和*背景*集之间测量的流行度发生显着变化的术语。如果术语“H5N1”仅存在于 1000 万个文档索引中的 5 个文档中，但在构成用户搜索结果的 100 个文档中的 4 个中找到，这些文档很重要并且可能与他们的搜索非常相关。5/10,000,000 对 4/100 是一个很大的频率摆动。

### Single-set analysis

In the simplest case, the *foreground* set of interest is the search results matched by a query and the *background* set used for statistical comparisons is the index or indices from which the results were gathered.

在最简单的情况下，感兴趣的*前景*集是与查询匹配的搜索结果， 用于统计比较的*背景*集是从中收集结果的一个或多个索引。

Example:

```console
GET /_search
{
  "query": {
    "terms": { "force": [ "British Transport Police" ] }
  },
  "aggregations": {
    "significant_crime_types": {
      "significant_terms": { "field": "crime_type" }
    }
  }
}
```

Response:

```console-result
{
  ...
  "aggregations": {
    "significant_crime_types": {
      "doc_count": 47347,
      "bg_count": 5064554,
      "buckets": [
        {
          "key": "Bicycle theft",
          "doc_count": 3640,
          "score": 0.371235374214817,
          "bg_count": 66799
        }
              ...
      ]
    }
  }
}
```

When querying an index of all crimes from all police forces, what these results show is that the British Transport Police force stand out as a force dealing with a disproportionately large number of bicycle thefts. Ordinarily, bicycle thefts represent only 1% of crimes (66799/5064554) but for the British Transport Police, who handle crime on railways and stations, 7% of crimes (3640/47347) is a bike theft. This is a significant seven-fold increase in frequency and so this anomaly was highlighted as the top crime type.

当查询所有警察部队的所有犯罪指数时，这些结果表明，英国交通警察部队作为处理不成比例的大量自行车盗窃的部队脱颖而出。通常，自行车盗窃仅占犯罪的 1% (66799/5064554)，但对于处理铁路和车站犯罪的英国交通警察来说，7% 的犯罪 (3640/47347) 是自行车盗窃。这是频率显着增加的七倍，因此这种异常被强调为顶级犯罪类型。

The problem with using a query to spot anomalies is it only gives us one subset to use for comparisons. To discover all the other police forces' anomalies we would have to repeat the query for each of the different forces.

使用查询来发现异常的问题在于它只给了我们一个用于比较的子集。要发现所有其他警察部队的异常情况，我们必须对每个不同的部队重复查询。

This can be a tedious way to look for unusual patterns in an index

这可能是一种在索引中查找异常模式的乏味方法

###  Multi-set analysis

A simpler way to perform analysis across multiple categories is to use a parent-level aggregation to segment the data ready for analysis.

跨多个类别执行分析的一种更简单的方法是使用父级聚合对准备进行分析的数据进行分段。

Example using a parent aggregation for segmentation:

使用父聚合进行分段的示例：

```console
GET /_search
{
  "aggregations": {
    "forces": {
      "terms": { "field": "force" },
      "aggregations": {
        "significant_crime_types": {
          "significant_terms": { "field": "crime_type" }
        }
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
    "forces": {
        "doc_count_error_upper_bound": 1375,
        "sum_other_doc_count": 7879845,
        "buckets": [
            {
                "key": "Metropolitan Police Service",
                "doc_count": 894038,
                "significant_crime_types": {
                    "doc_count": 894038,
                    "bg_count": 5064554,
                    "buckets": [
                        {
                            "key": "Robbery",
                            "doc_count": 27617,
                            "score": 0.0599,
                            "bg_count": 53182
                        }
                        ...
                    ]
                }
            },
            {
                "key": "British Transport Police",
                "doc_count": 47347,
                "significant_crime_types": {
                    "doc_count": 47347,
                    "bg_count": 5064554,
                    "buckets": [
                        {
                            "key": "Bicycle theft",
                            "doc_count": 3640,
                            "score": 0.371,
                            "bg_count": 66799
                        }
                        ...
                    ]
                }
            }
        ]
    }
  }
}
```

Now we have anomaly detection for each of the police forces using a single request.

现在，我们使用单个请求对每个警察部队进行异常检测。

We can use other forms of top-level aggregations to segment our data, for example segmenting by geographic area to identify unusual hot-spots of a particular crime type:

我们可以使用其他形式的顶级聚合来分割我们的数据，例如按地理区域分割以识别特定犯罪类型的异常热点：

```console
GET /_search
{
  "aggs": {
    "hotspots": {
      "geohash_grid": {
        "field": "location",
        "precision": 5
      },
      "aggs": {
        "significant_crime_types": {
          "significant_terms": { "field": "crime_type" }
        }
      }
    }
  }
}
```

This example uses the `geohash_grid` aggregation to create result buckets that represent geographic areas, and inside each bucket we can identify anomalous levels of a crime type in these tightly-focused areas e.g.

这个例子使用`geohash_grid`聚合来创建代表地理区域的结果桶，在每个桶内，我们可以识别这些高度集中的区域中犯罪类型的异常级别，例如

- Airports exhibit unusual numbers of weapon confiscations

  机场的武器没收数量异常

- Universities show uplifts of bicycle thefts

  大学显示自行车盗窃率上升

At a higher geohash_grid zoom-level with larger coverage areas we would start to see where an entire police-force may be tackling an unusual volume of a particular crime type.

在具有更大覆盖区域的更高 geohash_grid 缩放级别上，我们将开始看到整个警察部队可能在何处处理特定犯罪类型的异常数量。

Obviously a time-based top-level segmentation would help identify current trends for each point in time where a simple `terms` aggregation would typically show the very popular "constants" that persist across all time slots.

显然，基于时间的顶级细分将有助于识别每个时间点的当前趋势，其中简单的`terms`聚合通常会显示在所有时间段内持续存在的非常流行的“常量”。

> **How are the scores calculated?**
>
> The numbers returned for scores are primarily intended for ranking different suggestions sensibly rather than something easily understood by end users. The scores are derived from the doc frequencies in *foreground* and *background* sets. In brief, a term is considered significant if there is a noticeable difference in the frequency in which a term appears in the subset and in the background. The way the terms are ranked can be configured, see "Parameters" section.
>
> **分数是如何计算的？**
>
> 返回的分数数字主要用于对不同的建议进行合理的排名，而不是最终用户容易理解的内容。分数来自*前景*和*背景*集中的文档频率。简而言之，如果一个词在子集和背景中出现的频率存在显着差异，则该词被认为是显着的。可以配置术语的排名方式，请参阅“参数”部分。

###  Use on free-text fields

The significant_terms aggregation can be used effectively on tokenized free-text fields to suggest:

important_terms 聚合可以有效地用于标记化的自由文本字段，以建议：

- keywords for refining end-user searches

  用于优化最终用户搜索的关键字

- keywords for use in percolator queries

  用于过滤器查询的关键字

> WARNING: Picking a free-text field as the subject of a significant terms analysis can be expensive! It will attempt to load every unique word into RAM. It is recommended to only use this on smaller indices.
>
> 选择一个自由文本字段作为重要术语分析的主题可能会很昂贵！它将尝试将每个唯一的字加载到 RAM 中。建议只在较小的索引上使用它。

>**Use the \*"like this but not this"\* pattern**
>
>You can spot mis-categorized content by first searching a structured field e.g. `category:adultMovie` and use significant_terms on the free-text "movie_description" field. Take the suggested words (I’ll leave them to your imagination) and then search for all movies NOT marked as category:adultMovie but containing these keywords. You now have a ranked list of badly-categorized movies that you should reclassify or at least remove from the "familyFriendly" category.
>
>The significance score from each term can also provide a useful `boost` setting to sort matches. Using the `minimum_should_match` setting of the `terms` query with the keywords will help control the balance of precision/recall in the result set i.e a high setting would have a small number of relevant results packed full of keywords and a setting of "1" would produce a more exhaustive results set with all documents containing *any* keyword.
>
>**使用\*“like this but not this”\*模式**
>
>您可以通过首先搜索结构化字段（例如`category:adultMovie`，在自由文本“movie_description”字段上使用重要术语）来发现分类错误的内容。使用建议的词（我将让您自行想象），然后搜索所有未标记为 category:adultMovie 但包含这些关键字的电影。您现在有一个分类不当的电影的排名列表，您应该重新分类或至少从“家庭友好”类别中删除这些电影。
>
>每个术语的显着性分数也可以提供一个有用的`boost`设置来对匹配进行排序。使用带有关键字`minimum_should_match`的`terms`查询设置将有助于控制结果集中的精确率/召回率的平衡，即高设置将使少量相关结果充满关键字，而设置“1”将产生更详尽的结果包含*任何*关键字的所有文档的结果集。

> TIP: 
> Show significant_terms in context Free-text significant_terms are much more easily understood when viewed in context. Take the results of `significant_terms` suggestions from a free-text field and use them in a `terms` query on the same field with a `highlight` clause to present users with example snippets of documents. When the terms are presented unstemmed, highlighted, with the right case, in the right order and with some context, their significance/meaning is more readily apparent.
>
> **
> 在上下文中显示重要术语 在上下文中查看时，**自由文本重要术语更容易理解。`significant_terms`从自由文本字段中获取建议的结果，并`terms`在同一字段的查询中使用它们使用`highlight`子句向用户展示示例文档片段。当这些术语不加词干、突出显示、大小写正确、顺序正确并具有某些上下文时，它们的重要性/含义更容易显而易见。

###  Custom background sets

Ordinarily, the foreground set of documents is "diffed" against a background set of all the documents in your index. However, sometimes it may prove useful to use a narrower background set as the basis for comparisons. For example, a query on documents relating to "Madrid" in an index with content from all over the world might reveal that "Spanish" was a significant term. This may be true but if you want some more focused terms you could use a `background_filter` on the term *spain* to establish a narrower set of documents as context. With this as a background "Spanish" would now be seen as commonplace and therefore not as significant as words like "capital" that relate more strongly with Madrid. Note that using a background filter will slow things down - each term’s background frequency must now be derived on-the-fly from filtering posting lists rather than reading the index’s pre-computed count for a term.

通常，前景文档集与索引中所有文档的背景集“不同”。然而，有时使用较窄的背景集作为比较的基础可能会被证明是有用的。例如，对包含来自世界各地的内容的索引中与“Madrid”相关的文档的查询可能会显示“Spanish”是一个重要术语。这可能是真的，但如果你想要一些更集中的术语，你可以`background_filter` 在术语*西班牙*使用 a建立一个更窄的文档集作为上下文。以此为背景，“西班牙语”现在将被视为司空见惯，因此不如“首都”等与马德里更密切相关的词那么重要。请注意，使用背景过滤器会减慢速度 - 现在必须从过滤发布列表中即时得出每个术语的背景频率，而不是读取索引的预先计算的术语计数。

###  Limitations

####  Significant terms must be indexed values

Unlike the terms aggregation it is currently not possible to use script-generated terms for counting purposes. Because of the way the significant_terms aggregation must consider both *foreground* and *background* frequencies it would be prohibitively expensive to use a script on the entire index to obtain background frequencies for comparisons. Also DocValues are not supported as sources of term data for similar reasons.

与术语聚合不同，目前无法使用脚本生成的术语进行计数。由于 significant_terms聚合必须同时考虑*前景*和*背景*频率的方式，在整个索引上使用脚本来获取背景频率进行比较的成本会非常高。出于类似的原因，也不支持 DocValues 作为术语数据的来源。

####  No analysis of floating point fields

Floating point fields are currently not supported as the subject of significant_terms analysis. While integer or long fields can be used to represent concepts like bank account numbers or category numbers which can be interesting to track, floating point fields are usually used to represent quantities of something. As such, individual floating point terms are not useful for this form of frequency analysis.

目前不支持将浮点字段作为 important_terms 分析的主题。虽然整数或长字段可用于表示诸如银行帐号或类别编号之类的概念，这些概念可能很有趣，但浮点字段通常用于表示某物的数量。因此，单个浮点项对于这种形式的频率分析没有用处。

####  Use as a parent aggregation

If there is the equivalent of a `match_all` query or no query criteria providing a subset of the index the significant_terms aggregation should not be used as the top-most aggregation - in this scenario the *foreground* set is exactly the same as the *background* set and so there is no difference in document frequencies to observe and from which to make sensible suggestions.

如果有等价的`match_all`查询或没有提供索引子集的查询条件，那么重要的聚合不应该用作最顶层的聚合 - 在这种情况下，*前景*集与*背景*集完全相同，因此有要观察的文件频率和从中提出合理建议的文件频率没有区别。

Another consideration is that the significant_terms aggregation produces many candidate results at shard level that are only later pruned on the reducing node once all statistics from all shards are merged. As a result, it can be inefficient and costly in terms of RAM to embed large child aggregations under a significant_terms aggregation that later discards many candidate terms. It is advisable in these cases to perform two searches - the first to provide a rationalized list of significant_terms and then add this shortlist of terms to a second query to go back and fetch the required child aggregations.

另一个考虑因素是，重要的术语聚合会在分片级别产生许多候选结果，只有在合并所有分片的所有统计信息后，才会在减少节点上修剪这些候选结果。因此，就 RAM 而言，将大型子聚合嵌入到随后丢弃许多候选术语的重要项聚合下可能是低效且成本高昂的。在这些情况下，建议执行两次搜索 - 第一次提供合理化的重要术语列表，然后将此术语候选列表添加到第二个查询以返回并获取所需的子聚合。

####  Approximate counts

The counts of how many documents contain a term provided in results are based on summing the samples returned from each shard and as such may be:

包含结果中提供的术语的文档数量基于从每个分片返回的样本的总和，因此可能是：

- low if certain shards did not provide figures for a given term in their top sample

  如果某些分片未在其顶级样本中提供给定术语的数字，则为低

- high when considering the background frequency as it may count occurrences found in deleted documents

  考虑背景频率时高，因为它可能会计算已删除文档中的出现次数

Like most design decisions, this is the basis of a trade-off in which we have chosen to provide fast performance at the cost of some (typically small) inaccuracies. However, the `size` and `shard size` settings covered in the next section provide tools to help control the accuracy levels.

与大多数设计决策一样，这是我们选择以牺牲一些（通常很小）不准确为代价提供快速性能的权衡基础。但是，下一节中介绍的`size`和`shard size`设置提供了帮助控制准确度级别的工具。

###  Parameters

####  JLH score

The JLH score can be used as a significance score by adding the parameter

通过添加参数，JLH 分数可以用作显着性分数

```js
"jlh": {
	 }
```

The scores are derived from the doc frequencies in *foreground* and *background* sets. The *absolute* change in popularity (foregroundPercent - backgroundPercent) would favor common terms whereas the *relative* change in popularity (foregroundPercent/ backgroundPercent) would favor rare terms. Rare vs common is essentially a precision vs recall balance and so the absolute and relative changes are multiplied to provide a sweet spot between precision and recall.

分数来自*前景*和*背景*集中的文档频率。流行度的*绝对*变化 (foregroundPercent - backgroundPercent) 有利于常用词，而流行度的*相对*变化 (foregroundPercent/ backgroundPercent) 有利于罕见词。罕见与普通本质上是精度与召回的平衡，因此绝对和相对变化相乘以提供精度和召回之间的最佳位置。

####  Mutual information

Mutual information as described in "Information Retrieval", Manning et al., Chapter 13.5.1 can be used as significance score by adding the parameter

通过添加参数，Manning 等人，第 13.5.1 章“信息检索”中描述的互信息可以用作显着性分数

```js
	 "mutual_information": {
	      "include_negatives": true
	 }
```

Mutual information does not differentiate between terms that are descriptive for the subset or for documents outside the subset. The significant terms therefore can contain terms that appear more or less frequent in the subset than outside the subset. To filter out the terms that appear less often in the subset than in documents outside the subset, `include_negatives` can be set to `false`.

互信息不区分描述子集或子集外文档的术语。因此，重要项可以包含在子集中出现的频率比在子集外出现的频率更高或更低的术语。要过滤掉子集中出现频率低于子集外文档的术语，`include_negatives`可以设置为`false`。

Per default, the assumption is that the documents in the bucket are also contained in the background. If instead you defined a custom background filter that represents a different set of documents that you want to compare to, set

默认情况下，假设存储桶中的文档也包含在背景中。相反，如果您定义了一个自定义背景过滤器来表示要与之比较的一组不同的文档，请设置

```js
"background_is_superset": false
```

####  Chi square

Chi square as described in "Information Retrieval", Manning et al., Chapter 13.5.2 can be used as significance score by adding the parameter

在“信息检索”，曼宁等人，第 13.5.2 章中描述的卡方可以通过添加参数用作显着性分数

```js
 "chi_square": {
	 }
```

Chi square behaves like mutual information and can be configured with the same parameters `include_negatives` and `background_is_superset`.

卡方的行为类似于互信息，可以使用相同的参数`include_negatives`和进行配置`background_is_superset`。

####  Google normalized distance

Google normalized distance as described in ["The Google Similarity Distance", Cilibrasi and Vitanyi, 2007](https://arxiv.org/pdf/cs/0412098v3.pdf) can be used as significance score by adding the parameter

谷歌归一化距离，如[“谷歌相似距离”，Cilibrasi 和 Vitanyi，2007 年所述，](https://arxiv.org/pdf/cs/0412098v3.pdf)可以通过添加参数作为显着性得分

```js
	 "gnd": {
	 }
```

`gnd` also accepts the `background_is_superset` parameter.

`gnd`也接受`background_is_superset`参数。

####  Percentage

A simple calculation of the number of documents in the foreground sample with a term divided by the number of documents in the background with the term. By default this produces a score greater than zero and less than one.

简单计算前台样本中带有term的文档数除以带有term的后台文档数。默认情况下，这会产生大于零且小于 1 的分数。

The benefit of this heuristic is that the scoring logic is simple to explain to anyone familiar with a "per capita" statistic. However, for fields with high cardinality there is a tendency for this heuristic to select the rarest terms such as typos that occur only once because they score 1/1 = 100%.

这种启发式方法的好处是评分逻辑很容易向熟悉“人均”统计数据的任何人解释。但是，对于具有高基数的字段，这种启发式方法倾向于选择最罕见的术语，例如仅出现一次的拼写错误，因为它们的得分为 1/1 = 100%。

It would be hard for a seasoned boxer to win a championship if the prize was awarded purely on the basis of percentage of fights won - by these rules a newcomer with only one fight under their belt would be impossible to beat. Multiple observations are typically required to reinforce a view so it is recommended in these cases to set both `min_doc_count` and `shard_min_doc_count` to a higher value such as 10 in order to filter out the low-frequency terms that otherwise take precedence.

如果仅仅根据赢得比赛的百分比来颁发奖项，那么经验丰富的拳击手很难赢得冠军——根据这些规则，一个只有一场比赛的新人是不可能被击败的。通常需要多次观察来强化视图，因此在这些情况下，建议将两者都设置为`min_doc_count`和`shard_min_doc_count`更高的值，例如 10，以过滤掉优先的低频项。

```js
	 "percentage": {
	 }
```

####  Which one is best?

Roughly, `mutual_information` prefers high frequent terms even if they occur also frequently in the background. For example, in an analysis of natural language text this might lead to selection of stop words. `mutual_information` is unlikely to select very rare terms like misspellings. `gnd` prefers terms with a high co-occurrence and avoids selection of stopwords. It might be better suited for synonym detection. However, `gnd` has a tendency to select very rare terms that are, for example, a result of misspelling. `chi_square` and `jlh` are somewhat in-between.

粗略地说，`mutual_information`即使它们在后台也经常出现，也更喜欢高频词。例如，在对自然语言文本的分析中，这可能会导致选择停用词。`mutual_information`不太可能选择拼写错误等非常罕见的术语。`gnd`更喜欢高共现的术语并避免选择停用词。它可能更适合同义词检测。但是，`gnd`倾向于选择非常罕见的术语，例如拼写错误的结果。`chi_square`并且`jlh`介于两者之间。

It is hard to say which one of the different heuristics will be the best choice as it depends on what the significant terms are used for (see for example [Yang and Pedersen, "A Comparative Study on Feature Selection in Text Categorization", 1997](http://courses.ischool.berkeley.edu/i256/f06/papers/yang97comparative.pdf) for a study on using significant terms for feature selection for text classification).

这是很难说哪一种不同的启发之一将是最好的选择，因为它依赖于用于什么显著范围（见例如[杨和Pedersen，“比较研究在文本特征选择”，1997年](http://courses.ischool.berkeley.edu/i256/f06/papers/yang97comparative.pdf)的使用重要术语进行文本分类特征选择的研究）。

If none of the above measures suits your usecase than another option is to implement a custom significance measure:

如果上述措施都不适合您的用例，那么另一种选择是实施自定义重要性措施：

####  Scripted

Customized scores can be implemented via a script:

自定义分数可以通过脚本实现：

```js
  "script_heuristic": {
              "script": {
	        "lang": "painless",
	        "source": "params._subset_freq/(params._superset_freq - params._subset_freq + 1)"
	      }
            }
```

Scripts can be inline (as in above example), indexed or stored on disk. For details on the options, see [script documentation](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html).

脚本可以是内联的（如上例所示）、索引或存储在磁盘上。有关选项的详细信息，请参阅[脚本文档](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html)。

Available parameters in the script are

脚本中可用的参数是

| `_subset_freq`   | Number of documents the term appears in the subset.<br />该术语出现在子集中的文档数。 |
| ---------------- | ------------------------------------------------------------ |
| `_superset_freq` | Number of documents the term appears in the superset.<br />该术语出现在超集中的文档数。 |
| `_subset_size`   | Number of documents in the subset.<br />子集中的文档数。     |
| `_superset_size` | Number of documents in the superset.<br />超集中的文档数。   |

####  Size & Shard Size

The `size` parameter can be set to define how many term buckets should be returned out of the overall terms list. By default, the node coordinating the search process will request each shard to provide its own top term buckets and once all shards respond, it will reduce the results to the final list that will then be returned to the client. If the number of unique terms is greater than `size`, the returned list can be slightly off and not accurate (it could be that the term counts are slightly off and it could even be that a term that should have been in the top size buckets was not returned).

`size`可以设置该参数来定义应从整个术语列表中返回多少个术语桶。默认情况下，协调搜索过程的节点将请求每个分片提供自己的顶级词桶，一旦所有分片都响应，它会将结果减少到最终列表，然后返回给客户端。如果唯一术语的数量大于`size`，则返回的列表可能会略微偏离且不准确（可能是术语计数略有偏离，甚至可能是本应位于最大存储桶中的术语未回）。

To ensure better accuracy a multiple of the final `size` is used as the number of terms to request from each shard (`2 * (size * 1.5 + 10)`). To take manual control of this setting the `shard_size` parameter can be used to control the volumes of candidate terms produced by each shard.

为了确保更好的准确性，使用 final 的倍数`size`作为从每个分片 ( `2 * (size * 1.5 + 10)`)请求的术语数。要手动控制此设置，该`shard_size`参数可用于控制每个分片产生的候选词的数量。

Low-frequency terms can turn out to be the most interesting ones once all results are combined so the significant_terms aggregation can produce higher-quality results when the `shard_size` parameter is set to values significantly higher than the `size` setting. This ensures that a bigger volume of promising candidate terms are given a consolidated review by the reducing node before the final selection. Obviously large candidate term lists will cause extra network traffic and RAM usage so this is quality/cost trade off that needs to be balanced. If `shard_size` is set to -1 (the default) then `shard_size` will be automatically estimated based on the number of shards and the `size` parameter.

一旦所有结果组合在一起，低频项就会成为最有趣的项，因此当`shard_size`参数设置为明显高于设置的值时，重要项聚合可以产生更高质量的结果`size`。这确保了在最终选择之前，减少节点对大量有希望的候选术语进行了综合审查。显然，大型候选术语列表会导致额外的网络流量和 RAM 使用量，因此这是需要平衡的质量/成本权衡。如果`shard_size`设置为 -1（默认值），则将`shard_size`根据分片数量和`size`参数自动估计。

> NOTE: `shard_size` cannot be smaller than `size` (as it doesn’t make much sense). When it is, Elasticsearch will override it and reset it to be equal to `size`.
>
> `shard_size`不能小于`size`（因为它没有多大意义）。如果是，Elasticsearch 将覆盖它并将其重置为等于`size`。

####  Minimum document count

It is possible to only return terms that match more than a configured number of hits using the `min_doc_count` option:

使用该`min_doc_count`选项可以只返回匹配超过配置数量的匹配项：

```console
GET /_search
{
  "aggs": {
    "tags": {
      "significant_terms": {
        "field": "tag",
        "min_doc_count": 10
      }
    }
  }
}
```

The above aggregation would only return tags which have been found in 10 hits or more. Default value is `3`.

上述聚合只会返回在 10 个或更多点击中找到的标签。默认值为`3`。

Terms that score highly will be collected on a shard level and merged with the terms collected from other shards in a second step. However, the shard does not have the information about the global term frequencies available. The decision if a term is added to a candidate list depends only on the score computed on the shard using local shard frequencies, not the global frequencies of the word. The `min_doc_count` criterion is only applied after merging local terms statistics of all shards. In a way the decision to add the term as a candidate is made without being very *certain* about if the term will actually reach the required `min_doc_count`. This might cause many (globally) high frequent terms to be missing in the final result if low frequent but high scoring terms populated the candidate lists. To avoid this, the `shard_size` parameter can be increased to allow more candidate terms on the shards. However, this increases memory consumption and network traffic.

得分高的术语将在分片级别收集，并在第二步与从其他分片收集的术语合并。但是，分片没有关于可用的全局词频的信息。是否将术语添加到候选列表的决定仅取决于使用本地分片频率在分片上计算的分数，而不是单词的全局频率。该`min_doc_count`标准仅在合并所有分片的本地术语统计信息后应用。从某种意义上说，决定将这个任期添加为候选人是在*不确定*该任期是否真的会达到要求的情况下做出的`min_doc_count`. 如果低频率但高评分的术语填充了候选列表，这可能会导致最终结果中缺少许多（全局）高频率术语。为避免这种情况，`shard_size`可以增加该参数以允许分片上有更多候选词。但是，这会增加内存消耗和网络流量。

#### `shard_min_doc_count`

The parameter `shard_min_doc_count` regulates the *certainty* a shard has if the term should actually be added to the candidate list or not with respect to the `min_doc_count`. Terms will only be considered if their local shard frequency within the set is higher than the `shard_min_doc_count`. If your dictionary contains many low frequent terms and you are not interested in those (for example misspellings), then you can set the `shard_min_doc_count` parameter to filter out candidate terms on a shard level that will with a reasonable certainty not reach the required `min_doc_count` even after merging the local counts. `shard_min_doc_count` is set to `0` per default and has no effect unless you explicitly set it.

该参数`shard_min_doc_count`规定了一个分片是否应该实际添加到候选列表中的*确定性*`min_doc_count`。仅当它们在集合中的本地分片频率高于`shard_min_doc_count`. 如果您的字典包含许多低频词并且您对这些词不感兴趣（例如拼写错误），那么您可以设置`shard_min_doc_count`参数以在分片级别上过滤候选词，`min_doc_count`即使合并后也不会达到要求的分片级别当地计数。`shard_min_doc_count`设置为`0`默认值，除非您明确设置，否则无效。

> WARNING: Setting `min_doc_count` to `1` is generally not advised as it tends to return terms that are typos or other bizarre curiosities. Finding more than one instance of a term helps reinforce that, while still rare, the term was not the result of a one-off accident. The default value of 3 is used to provide a minimum weight-of-evidence. Setting `shard_min_doc_count` too high will cause significant candidate terms to be filtered out on a shard level. This value should be set much lower than `min_doc_count/#shards`.
>
> 通常不建议设置`min_doc_count`为，`1`因为它往往会返回拼写错误或其他奇怪的术语。找到一个术语的多个实例有助于强化这一点，虽然仍然很少见，但该术语不是一次性事故的结果。默认值 3 用于提供最小证据权重。设置`shard_min_doc_count`太高会导致重要的候选词在分片级别被过滤掉。此值应设置得远低于`min_doc_count/#shards`。

####  Custom background context

The default source of statistical information for background term frequencies is the entire index and this scope can be narrowed through the use of a `background_filter` to focus in on significant terms within a narrower context:

背景术语频率的默认统计信息来源是整个索引，可以通过使用 a`background_filter`来缩小范围，以在更窄的上下文中关注重要术语：

```console
GET /_search
{
  "query": {
    "match": {
      "city": "madrid"
    }
  },
  "aggs": {
    "tags": {
      "significant_terms": {
        "field": "tag",
        "background_filter": {
          "term": { "text": "spain" }
        }
      }
    }
  }
}
```

The above filter would help focus in on terms that were peculiar to the city of Madrid rather than revealing terms like "Spanish" that are unusual in the full index’s worldwide context but commonplace in the subset of documents containing the word "Spain".

上述过滤器将有助于关注马德里市特有的术语，而不是揭示诸如“西班牙文”之类的术语，这些术语在完整索引的全球范围内不常见，但在包含“西班牙”一词的文档子集中很常见。

> WARNING: Use of background filters will slow the query as each term’s postings must be filtered to determine a frequency
>
> 使用后台过滤器会减慢查询速度，因为必须过滤每个术语的帖子以确定频率

####  Filtering Values

It is possible (although rarely required) to filter the values for which buckets will be created. This can be done using the `include` and `exclude` parameters which are based on a regular expression string or arrays of exact terms. This functionality mirrors the features described in the [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) documentation.

可以（虽然很少需要）过滤将为其创建存储桶的值。这可以使用基于正则表达式字符串或精确术语数组的`include`和 `exclude`参数来完成。此功能反映了[术语聚合](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html)文档中描述的功能。

###  Collect mode

To avoid memory issues, the `significant_terms` aggregation always computes child aggregations in `breadth_first` mode. A description of the different collection modes can be found in the [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-collect) documentation.

为避免内存问题，`significant_terms`聚合始终以`breadth_first`模式计算子聚合。可以在[术语聚合](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-collect)文档中找到不同收集模式的描述 。

###  Execution hint

There are different mechanisms by which terms aggregations can be executed:

可以通过不同的机制来执行术语聚合：

- by using field values directly in order to aggregate data per-bucket (`map`)

  通过直接使用字段值来聚合每个存储桶的数据 ( `map`)

- by using [global ordinals](https://www.elastic.co/guide/en/elasticsearch/reference/master/eager-global-ordinals.html) of the field and allocating one bucket per global ordinal (`global_ordinals`)

  通过使用字段的[全局序数](https://www.elastic.co/guide/en/elasticsearch/reference/master/eager-global-ordinals.html)并为每个全局序数分配一个桶 ( `global_ordinals`)

Elasticsearch tries to have sensible defaults so this is something that generally doesn’t need to be configured.

Elasticsearch 尝试使用合理的默认值，因此这通常不需要配置。

`global_ordinals` is the default option for `keyword` field, it uses global ordinals to allocates buckets dynamically so memory usage is linear to the number of values of the documents that are part of the aggregation scope.

`global_ordinals`是`keyword`字段的默认选项，它使用全局序数动态分配桶，因此内存使用与作为聚合范围一部分的文档的值数量呈线性关系。

`map` should only be considered when very few documents match a query. Otherwise the ordinals-based execution mode is significantly faster. By default, `map` is only used when running an aggregation on scripts, since they don’t have ordinals.

`map`仅当很少有文档与查询匹配时才应考虑。否则，基于序数的执行模式要快得多。默认情况下，`map`仅在对脚本运行聚合时使用，因为它们没有序数。

```console
GET /_search
{
  "aggs": {
    "tags": {
      "significant_terms": {
        "field": "tags",
        "execution_hint": "map" (1)
      }
    }
  }
}
```

1.  the possible values are `map`, `global_ordinals`

    可能的值是`map`，`global_ordinals`

Please note that Elasticsearch will ignore this execution hint if it is not applicable.

请注意，如果此执行提示不适用，Elasticsearch 将忽略它。

#  Significant text aggregation

An aggregation that returns interesting or unusual occurrences of free-text terms in a set. It is like the [significant terms](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-significantterms-aggregation.html) aggregation but differs in that:

返回集合中有趣或异常出现的自由文本术语的聚合。它类似于[重要术语](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-significantterms-aggregation.html)聚合，但不同之处在于：

- It is specifically designed for use on type `text` fields

  它专门设计用于类型`text`字段

- It does not require field data or doc-values

  它不需要字段数据或文档值

- It re-analyzes text content on-the-fly meaning it can also filter duplicate sections of noisy text that otherwise tend to skew statistics.

  它即时重新分析文本内容，这意味着它还可以过滤嘈杂文本的重复部分，否则这些部分往往会扭曲统计数据。

> WARNING: Re-analyzing *large* result sets will require a lot of time and memory. It is recommended that the significant_text aggregation is used as a child of either the [sampler](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-sampler-aggregation.html) or [diversified sampler](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-diversified-sampler-aggregation.html) aggregation to limit the analysis to a *small* selection of top-matching documents e.g. 200. This will typically improve speed, memory use and quality of results.
>
> 重新分析*大型*结果集将需要大量时间和内存。建议将重要文本聚合用作[采样器](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-sampler-aggregation.html)或 [多样化采样器](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-diversified-sampler-aggregation.html)聚合的子项，以将分析限制为*少数*最佳匹配文档，例如 200。这通常会提高速度、内存使用和结果质量。

**Example use cases:**

- Suggesting "H5N1" when users search for "bird flu" to help expand queries

  用户搜索“禽流感”时提示“H5N1”有助于扩大查询

- Suggesting keywords relating to stock symbol $ATI for use in an automated news classifier

  建议与股票代码 $ATI 相关的关键字用于自动新闻分类器

In these cases the words being selected are not simply the most popular terms in results. The most popular words tend to be very boring (*and, of, the, we, I, they* …). The significant words are the ones that have undergone a significant change in popularity measured between a *foreground* and *background* set. If the term "H5N1" only exists in 5 documents in a 10 million document index and yet is found in 4 of the 100 documents that make up a user’s search results that is significant and probably very relevant to their search. 5/10,000,000 vs 4/100 is a big swing in frequency.

在这些情况下，被选择的词不仅仅是结果中最流行的词。最流行的词往往很无聊（*和，的，我们，我，他们*......）。重要词是在*前景*和*背景*集之间测量的流行度发生了显着变化的词。如果术语“H5N1”仅存在于 1000 万个文档索引中的 5 个文档中，但在构成用户搜索结果的 100 个文档中的 4 个中找到，这些文档很重要并且可能与他们的搜索非常相关。5/10,000,000 对 4/100 是一个很大的频率摆动。

###  Basic use

In the typical use case, the *foreground* set of interest is a selection of the top-matching search results for a query and the _background_set used for statistical comparisons is the index or indices from which the results were gathered.

在典型的用例中，感兴趣的*前景*集是查询的最佳匹配搜索结果的选择，用于统计比较的 _background_set 是从中收集结果的一个或多个索引。

Example:

```console
GET news/_search
{
  "query": {
    "match": { "content": "Bird flu" }
  },
  "aggregations": {
    "my_sample": {
      "sampler": {
        "shard_size": 100
      },
      "aggregations": {
        "keywords": {
          "significant_text": { "field": "content" }
        }
      }
    }
  }
}
```

Response:

```console-result
{
  "took": 9,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
    "aggregations" : {
        "my_sample": {
            "doc_count": 100,
            "keywords" : {
                "doc_count": 100,
                "buckets" : [
                    {
                        "key": "h5n1",
                        "doc_count": 4,
                        "score": 4.71235374214817,
                        "bg_count": 5
                    }
                    ...
                ]
            }
        }
    }
}
```

The results show that "h5n1" is one of several terms strongly associated with bird flu. It only occurs 5 times in our index as a whole (see the `bg_count`) and yet 4 of these were lucky enough to appear in our 100 document sample of "bird flu" results. That suggests a significant word and one which the user can potentially add to their search.

结果表明，“h5n1”是与禽流感密切相关的几个术语之一。它在我们的索引中作为一个整体仅出现 5 次（参见`bg_count`），但其中 4 次幸运地出现在我们的 100 个“禽流感”结果文档样本中。这暗示了一个重要的词，并且用户可以将其添加到他们的搜索中。

###  Dealing with noisy data using `filter_duplicate_text`

Free-text fields often contain a mix of original content and mechanical copies of text (cut-and-paste biographies, email reply chains, retweets, boilerplate headers/footers, page navigation menus, sidebar news links, copyright notices, standard disclaimers, addresses).

自由文本字段通常包含原始内容和文本机械副本的混合（剪切和粘贴传记、电子邮件回复链、转发、样板页眉/页脚、页面导航菜单、侧边栏新闻链接、版权声明、标准免责声明、地址）。

In real-world data these duplicate sections of text tend to feature heavily in `significant_text` results if they aren’t filtered out. Filtering near-duplicate text is a difficult task at index-time but we can cleanse the data on-the-fly at query time using the `filter_duplicate_text` setting.

在现实世界的数据中，这些重复的文本部分`significant_text`如果没有被过滤掉，往往会在结果中占据重要地位。在索引时过滤几乎重复的文本是一项艰巨的任务，但我们可以在查询时使用`filter_duplicate_text`设置即时清理数据 。

First let’s look at an unfiltered real-world example using the [Signal media dataset](https://research.signalmedia.co/newsir16/signal-dataset.html) of a million news articles covering a wide variety of news. Here are the raw significant text results for a search for the articles mentioning "elasticsearch":

首先让我们看一个未经过滤的真实世界示例，该示例使用[Signal 媒体数据集](https://research.signalmedia.co/newsir16/signal-dataset.html)，该[数据集](https://research.signalmedia.co/newsir16/signal-dataset.html)包含涵盖各种新闻的一百万篇新闻文章。以下是搜索提及“elasticsearch”的文章的原始重要文本结果：

```js
{
  ...
  "aggregations": {
    "sample": {
      "doc_count": 35,
      "keywords": {
        "doc_count": 35,
        "buckets": [
          {
            "key": "elasticsearch",
            "doc_count": 35,
            "score": 28570.428571428572,
            "bg_count": 35
          },
          ...
          {
            "key": "currensee",
            "doc_count": 8,
            "score": 6530.383673469388,
            "bg_count": 8
          },
          ...
          {
            "key": "pozmantier",
            "doc_count": 4,
            "score": 3265.191836734694,
            "bg_count": 4
          },
          ...

}
```

The uncleansed documents have thrown up some odd-looking terms that are, on the face of it, statistically correlated with appearances of our search term "elasticsearch" e.g. "pozmantier". We can drill down into examples of these documents to see why pozmantier is connected using this query:

未清理的文档抛出了一些看起来很奇怪的词，从表面上看，这些词与我们的搜索词“elasticsearch”（例如“pozmantier”）的出现在统计上相关。我们可以深入查看这些文档的示例，以了解为什么使用此查询连接 pozmantier：

```console
GET news/_search
{
  "query": {
    "simple_query_string": {
      "query": "+elasticsearch  +pozmantier"
    }
  },
  "_source": [
    "title",
    "source"
  ],
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
```

The results show a series of very similar news articles about a judging panel for a number of tech projects:

结果显示了一系列非常相似的新闻文章，这些文章涉及多个技术项目的评审团：

```console
{
  ...
  "hits": {
    "hits": [
      {
        ...
        "_source": {
          "source": "Presentation Master",
          "title": "T.E.N. Announces Nominees for the 2015 ISE® North America Awards"
        },
        "highlight": {
          "content": [
            "City of San Diego Mike <em>Pozmantier</em>, Program Manager, Cyber Security Division, Department of",
            " Janus, Janus <em>ElasticSearch</em> Security Visualization Engine "
          ]
        }
      },
      {
        ...
        "_source": {
          "source": "RCL Advisors",
          "title": "T.E.N. Announces Nominees for the 2015 ISE(R) North America Awards"
        },
        "highlight": {
          "content": [
            "Mike <em>Pozmantier</em>, Program Manager, Cyber Security Division, Department of Homeland Security S&T",
            "Janus, Janus <em>ElasticSearch</em> Security Visualization Engine"
          ]
        }
      },
      ...
```

Mike Pozmantier was one of many judges on a panel and elasticsearch was used in one of many projects being judged.

Mike Pozmantier 是小组中的众多评委之一，elasticsearch 被用于许多被评判的项目之一。

As is typical, this lengthy press release was cut-and-paste by a variety of news sites and consequently any rare names, numbers or typos they contain become statistically correlated with our matching query.

通常，这个冗长的新闻稿被各种新闻网站剪切和粘贴，因此它们包含的任何稀有名称、数字或拼写错误都会与我们的匹配查询在统计上相关。

Fortunately similar documents tend to rank similarly so as part of examining the stream of top-matching documents the significant_text aggregation can apply a filter to remove sequences of any 6 or more tokens that have already been seen. Let’s try this same query now but with the `filter_duplicate_text` setting turned on:

幸运的是，相似的文档往往排名相似，因此作为检查顶级匹配文档流的一部分，重要文本聚合可以应用过滤器来删除任何 6 个或更多已经看到的标记序列。现在让我们尝试相同的查询，但`filter_duplicate_text`设置已打开：

```console
GET news/_search
{
  "query": {
    "match": {
      "content": "elasticsearch"
    }
  },
  "aggs": {
    "sample": {
      "sampler": {
        "shard_size": 100
      },
      "aggs": {
        "keywords": {
          "significant_text": {
            "field": "content",
            "filter_duplicate_text": true
          }
        }
      }
    }
  }
}
```

The results from analysing our deduplicated text are obviously of higher quality to anyone familiar with the elastic stack:

对于熟悉弹性堆栈的人来说，分析我们的去重文本的结果显然质量更高：

```js
{
  ...
  "aggregations": {
    "sample": {
      "doc_count": 35,
      "keywords": {
        "doc_count": 35,
        "buckets": [
          {
            "key": "elasticsearch",
            "doc_count": 22,
            "score": 11288.001166180758,
            "bg_count": 35
          },
          {
            "key": "logstash",
            "doc_count": 3,
            "score": 1836.648979591837,
            "bg_count": 4
          },
          {
            "key": "kibana",
            "doc_count": 3,
            "score": 1469.3020408163263,
            "bg_count": 5
          }
        ]
      }
    }
  }
}
```

Mr Pozmantier and other one-off associations with elasticsearch no longer appear in the aggregation results as a consequence of copy-and-paste operations or other forms of mechanical repetition.

由于复制和粘贴操作或其他形式的机械重复，Pozmantier 先生和其他与 elasticsearch 的一次性关联不再出现在聚合结果中。

If your duplicate or near-duplicate content is identifiable via a single-value indexed field (perhaps a hash of the article’s `title` text or an `original_press_release_url` field) then it would be more efficient to use a parent [diversified sampler](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-diversified-sampler-aggregation.html) aggregation to eliminate these documents from the sample set based on that single key. The less duplicate content you can feed into the significant_text aggregation up front the better in terms of performance.

如果您的重复或接近重复的内容可以通过单值索引字段（可能是文章`title`文本或`original_press_release_url`字段的哈希）识别，那么使用父[多元化采样器](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-diversified-sampler-aggregation.html)聚合从样本集中消除这些文档会更有效基于那个单一的键。在性能方面，您可以预先输入重要文本聚合的重复内容越少越好。

> **How are the significance scores calculated?**
>
> The numbers returned for scores are primarily intended for ranking different suggestions sensibly rather than something easily understood by end users. The scores are derived from the doc frequencies in *foreground* and *background* sets. In brief, a term is considered significant if there is a noticeable difference in the frequency in which a term appears in the subset and in the background. The way the terms are ranked can be configured, see "Parameters" section.
>
> **显着性分数是如何计算的？**
>
> 返回的分数数字主要用于对不同的建议进行合理的排名，而不是最终用户容易理解的内容。分数来自*前景*和*背景*集中的文档频率。简而言之，如果一个词在子集和背景中出现的频率存在显着差异，则该词被认为是显着的。可以配置术语的排名方式，请参阅“参数”部分。

> **Use the \*"like this but not this"\* pattern**
>
> You can spot mis-categorized content by first searching a structured field e.g. `category:adultMovie` and use significant_text on the text "movie_description" field. Take the suggested words (I’ll leave them to your imagination) and then search for all movies NOT marked as category:adultMovie but containing these keywords. You now have a ranked list of badly-categorized movies that you should reclassify or at least remove from the "familyFriendly" category.
>
> The significance score from each term can also provide a useful `boost` setting to sort matches. Using the `minimum_should_match` setting of the `terms` query with the keywords will help control the balance of precision/recall in the result set i.e a high setting would have a small number of relevant results packed full of keywords and a setting of "1" would produce a more exhaustive results set with all documents containing *any* keyword.
>
> **使用\*“like this but not this”\*模式**
>
> 您可以通过首先搜索结构化字段（例如`category:adultMovie`，在文本“movie_description”字段上使用重要文本）来发现分类错误的内容。使用建议的词（我将让您自行想象），然后搜索所有未标记为 category:adultMovie 但包含这些关键字的电影。您现在有一个分类不当的电影的排名列表，您应该重新分类或至少从“家庭友好”类别中删除这些电影。
>
> 每个术语的显着性分数也可以提供一个有用的`boost`设置来对匹配进行排序。使用带有关键字`minimum_should_match`的`terms`查询设置将有助于控制结果集中的精确率/召回率的平衡，即高设置将使少量相关结果充满关键字，而设置“1”将产生更详尽的结果包含*任何*关键字的所有文档的结果集。

###  Limitations

####  No support for child aggregations

The significant_text aggregation intentionally does not support the addition of child aggregations because:

important_text 聚合有意不支持添加子聚合，因为：

- It would come with a high memory cost

  它会带来很高的内存成本

- It isn’t a generally useful feature and there is a workaround for those that need it

  它不是一个普遍有用的功能，对于那些需要它的人来说，有一个解决方法

The volume of candidate terms is generally very high and these are pruned heavily before the final results are returned. Supporting child aggregations would generate additional churn and be inefficient. Clients can always take the heavily-trimmed set of results from a `significant_text` request and make a subsequent follow-up query using a `terms` aggregation with an `include` clause and child aggregations to perform further analysis of selected keywords in a more efficient fashion.

候选术语的数量通常非常高，并且在返回最终结果之前对其进行了大量修剪。支持子聚合会产生额外的流失并且效率低下。客户端总是可以从`significant_text`请求中获取大量修剪的结果集，并使用`terms`带有`include`子句和子聚合的聚合进行后续的后续查询，以更有效的方式对选定的关键字进行进一步分析。

####  No support for nested objects

The significant_text aggregation currently also cannot be used with text fields in nested objects, because it works with the document JSON source. This makes this feature inefficient when matching nested docs from stored JSON given a matching Lucene docID.

important_text 聚合目前也不能与嵌套对象中的文本字段一起使用，因为它适用于文档 JSON 源。在给定匹配的 Lucene docID 的情况下，从存储的 JSON 中匹配嵌套文档时，这使得此功能效率低下。

####  Approximate counts

The counts of how many documents contain a term provided in results are based on summing the samples returned from each shard and as such may be:

包含结果中提供的术语的文档数量基于从每个分片返回的样本的总和，因此可能是：

- low if certain shards did not provide figures for a given term in their top sample

  如果某些分片未在其顶级样本中提供给定术语的数字，则为低

- high when considering the background frequency as it may count occurrences found in deleted documents

  考虑背景频率时高，因为它可能会计算已删除文档中的出现次数

Like most design decisions, this is the basis of a trade-off in which we have chosen to provide fast performance at the cost of some (typically small) inaccuracies. However, the `size` and `shard size` settings covered in the next section provide tools to help control the accuracy levels.

与大多数设计决策一样，这是我们选择以牺牲一些（通常很小）不准确为代价提供快速性能的权衡基础。但是，下一节中介绍的`size`和`shard size`设置提供了帮助控制准确度级别的工具。

###  Parameters

####  Significance heuristics

This aggregation supports the same scoring heuristics (JLH, mutual_information, gnd, chi_square etc) as the [significant terms](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-significantterms-aggregation.html) aggregation

此聚合支持与[重要术语](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-significantterms-aggregation.html)聚合相同的评分试探法（JLH、mutual_information、gnd、chi_square 等）

####  Size & Shard Size

The `size` parameter can be set to define how many term buckets should be returned out of the overall terms list. By default, the node coordinating the search process will request each shard to provide its own top term buckets and once all shards respond, it will reduce the results to the final list that will then be returned to the client. If the number of unique terms is greater than `size`, the returned list can be slightly off and not accurate (it could be that the term counts are slightly off and it could even be that a term that should have been in the top size buckets was not returned).

`size`可以设置该参数来定义应从整个术语列表中返回多少个术语桶。默认情况下，协调搜索过程的节点将请求每个分片提供自己的顶级词桶，一旦所有分片都响应，它会将结果减少到最终列表，然后返回给客户端。如果唯一术语的数量大于`size`，则返回的列表可能会略微偏离且不准确（可能是术语计数略有偏离，甚至可能是本应位于最大大小存储桶中的术语未回）。

To ensure better accuracy a multiple of the final `size` is used as the number of terms to request from each shard (`2 * (size * 1.5 + 10)`). To take manual control of this setting the `shard_size` parameter can be used to control the volumes of candidate terms produced by each shard.

为了确保更好的准确性，使用 final 的倍数`size`作为从每个分片 ( `2 * (size * 1.5 + 10)`)请求的术语数。要手动控制此设置，该`shard_size`参数可用于控制每个分片产生的候选词的数量。

Low-frequency terms can turn out to be the most interesting ones once all results are combined so the significant_terms aggregation can produce higher-quality results when the `shard_size` parameter is set to values significantly higher than the `size` setting. This ensures that a bigger volume of promising candidate terms are given a consolidated review by the reducing node before the final selection. Obviously large candidate term lists will cause extra network traffic and RAM usage so this is quality/cost trade off that needs to be balanced. If `shard_size` is set to -1 (the default) then `shard_size` will be automatically estimated based on the number of shards and the `size` parameter.

一旦组合了所有结果，低频项可能会成为最有趣的项，因此当`shard_size`参数设置为明显高于设置的值时，重要项聚合可以产生更高质量的结果`size`。这确保了在最终选择之前，减少节点对大量有希望的候选术语进行了综合审查。显然，大型候选术语列表会导致额外的网络流量和 RAM 使用量，因此这是需要平衡的质量/成本权衡。如果`shard_size`设置为 -1（默认值），则将`shard_size`根据分片数量和`size`参数自动估计。

> NOTE: `shard_size` cannot be smaller than `size` (as it doesn’t make much sense). When it is, elasticsearch will override it and reset it to be equal to `size`.
>
> `shard_size`不能小于`size`（因为它没有多大意义）。如果是，elasticsearch 将覆盖它并将其重置为等于`size`。

####  Minimum document count

It is possible to only return terms that match more than a configured number of hits using the `min_doc_count` option. The Default value is 3.

使用该`min_doc_count`选项可以只返回匹配超过配置数量的匹配项。默认值为 3。

Terms that score highly will be collected on a shard level and merged with the terms collected from other shards in a second step. However, the shard does not have the information about the global term frequencies available. The decision if a term is added to a candidate list depends only on the score computed on the shard using local shard frequencies, not the global frequencies of the word. The `min_doc_count` criterion is only applied after merging local terms statistics of all shards. In a way the decision to add the term as a candidate is made without being very *certain* about if the term will actually reach the required `min_doc_count`. This might cause many (globally) high frequent terms to be missing in the final result if low frequent but high scoring terms populated the candidate lists. To avoid this, the `shard_size` parameter can be increased to allow more candidate terms on the shards. However, this increases memory consumption and network traffic.

得分高的术语将在分片级别收集，并在第二步与从其他分片收集的术语合并。但是，分片没有关于可用的全局词频的信息。是否将术语添加到候选列表的决定仅取决于使用本地分片频率在分片上计算的分数，而不是单词的全局频率。该`min_doc_count`标准仅在合并所有分片的本地术语统计信息后应用。从某种意义上说，决定将这个任期添加为候选人是在*不确定*该任期是否真的会达到要求的情况下做出的`min_doc_count`. 如果低频率但高评分的术语填充了候选列表，这可能会导致最终结果中缺少许多（全局）高频率术语。为避免这种情况，`shard_size`可以增加该参数以允许分片上有更多候选词。但是，这会增加内存消耗和网络流量。

##### `shard_min_doc_count`

The parameter `shard_min_doc_count` regulates the *certainty* a shard has if the term should actually be added to the candidate list or not with respect to the `min_doc_count`. Terms will only be considered if their local shard frequency within the set is higher than the `shard_min_doc_count`. If your dictionary contains many low frequent terms and you are not interested in those (for example misspellings), then you can set the `shard_min_doc_count` parameter to filter out candidate terms on a shard level that will with a reasonable certainty not reach the required `min_doc_count` even after merging the local counts. `shard_min_doc_count` is set to `0` per default and has no effect unless you explicitly set it.

该参数`shard_min_doc_count`规定了一个分片是否应该实际添加到候选列表中的*确定性*`min_doc_count`。仅当它们在集合中的本地分片频率高于`shard_min_doc_count`. 如果您的字典包含许多低频词并且您对这些词不感兴趣（例如拼写错误），那么您可以设置`shard_min_doc_count`参数以在分片级别上过滤候选词，`min_doc_count`即使合并后也不会达到要求的分片级别当地计数。`shard_min_doc_count`设置为`0`默认值，除非您明确设置，否则无效。

> WARNING: Setting `min_doc_count` to `1` is generally not advised as it tends to return terms that are typos or other bizarre curiosities. Finding more than one instance of a term helps reinforce that, while still rare, the term was not the result of a one-off accident. The default value of 3 is used to provide a minimum weight-of-evidence. Setting `shard_min_doc_count` too high will cause significant candidate terms to be filtered out on a shard level. This value should be set much lower than `min_doc_count/#shards`.
>
> 通常不建议设置`min_doc_count`为，`1`因为它往往会返回拼写错误或其他奇怪的术语。找到一个术语的多个实例有助于强化这一点，虽然仍然很少见，但该术语不是一次性事故的结果。默认值 3 用于提供最小证据权重。设置`shard_min_doc_count`太高会导致重要的候选词在分片级别被过滤掉。此值应设置得远低于`min_doc_count/#shards`。

####  Custom background context

The default source of statistical information for background term frequencies is the entire index and this scope can be narrowed through the use of a `background_filter` to focus in on significant terms within a narrower context:

背景术语频率的默认统计信息来源是整个索引，可以通过使用 a`background_filter`来缩小范围，以在更窄的上下文中关注重要术语：

```console
GET news/_search
{
  "query": {
    "match": {
      "content": "madrid"
    }
  },
  "aggs": {
    "tags": {
      "significant_text": {
        "field": "content",
        "background_filter": {
          "term": { "content": "spain" }
        }
      }
    }
  }
}
```

The above filter would help focus in on terms that were peculiar to the city of Madrid rather than revealing terms like "Spanish" that are unusual in the full index’s worldwide context but commonplace in the subset of documents containing the word "Spain".

上述过滤器将有助于关注马德里市特有的术语，而不是揭示诸如“西班牙文”之类的术语，这些术语在完整索引的全球范围内不常见，但在包含“西班牙”一词的文档子集中很常见。

> WARNING: Use of background filters will slow the query as each term’s postings must be filtered to determine a frequency
>
> 使用后台过滤器会减慢查询速度，因为必须过滤每个术语的帖子以确定频率

####  Dealing with source and index mappings

Ordinarily the indexed field name and the original JSON field being retrieved share the same name. However with more complex field mappings using features like `copy_to` the source JSON field(s) and the indexed field being aggregated can differ. In these cases it is possible to list the JSON _source fields from which text will be analyzed using the `source_fields` parameter:

通常，索引字段名称和正在检索的原始 JSON 字段共享相同的名称。但是，对于使用`copy_to`源 JSON 字段和被聚合的索引字段等功能的更复杂的字段映射可能会有所不同。在这些情况下，可以使用`source_fields`参数列出将从中分析文本的 JSON _source 字段：

```console
GET news/_search
{
  "query": {
    "match": {
      "custom_all": "elasticsearch"
    }
  },
  "aggs": {
    "tags": {
      "significant_text": {
        "field": "custom_all",
        "source_fields": [ "content", "title" ]
      }
    }
  }
}
```

####  Filtering Values

It is possible (although rarely required) to filter the values for which buckets will be created. This can be done using the `include` and `exclude` parameters which are based on a regular expression string or arrays of exact terms. This functionality mirrors the features described in the [terms aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) documentation.

可以（虽然很少需要）过滤将为其创建存储桶的值。这可以使用基于正则表达式字符串或精确术语数组的`include`和 `exclude`参数来完成。此功能反映了[术语聚合](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html)文档中描述的功能。

#  Terms aggregation

A multi-bucket value source based aggregation where buckets are dynamically built - one per unique value.

一种基于多桶值源的聚合，其中桶是动态构建的 - 每个唯一值一个。

Example:

```console
GET /_search
{
  "aggs": {
    "genres": {
      "terms": { "field": "genre" }
    }
  }
}
```

Response:

```console-result
{
  ...
  "aggregations": {
    "genres": {
      "doc_count_error_upper_bound": 0,   (1)
      "sum_other_doc_count": 0,           (2)
      "buckets": [                        (3)
        {
          "key": "electronic",
          "doc_count": 6
        },
        {
          "key": "rock",
          "doc_count": 3
        },
        {
          "key": "jazz",
          "doc_count": 2
        }
      ]
    }
  }
}
```

1. an upper bound of the error on the document counts for each term, see [below](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#terms-agg-doc-count-error)

   每个术语的文档计数错误的上限，见[下文](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#terms-agg-doc-count-error)

2.  when there are lots of unique terms, Elasticsearch only returns the top terms; this number is the sum of the document counts for all buckets that are not part of the response

   当有很多独特的术语时，Elasticsearch 只返回顶部的术语；此数字是不属于响应的所有存储桶的文档计数总和

3. the list of the top buckets, the meaning of `top` being defined by the [order](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)

    顶级桶的列表，`top`由[订单](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-order)定义的含义

By default, the `terms` aggregation will return the buckets for the top ten terms ordered by the `doc_count`. One can change this default behaviour by setting the `size` parameter.

默认情况下，`terms`聚合将返回按 排序的前十个术语的桶`doc_count`。可以通过设置`size`参数来更改此默认行为。

The `field` can be [Keyword](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html), [Numeric](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html), [`ip`](https://www.elastic.co/guide/en/elasticsearch/reference/master/ip.html), [`boolean`](https://www.elastic.co/guide/en/elasticsearch/reference/master/boolean.html), or [`binary`](https://www.elastic.co/guide/en/elasticsearch/reference/master/binary.html).

该`field`可[关键词](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html)，[数字](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)，[`ip`](https://www.elastic.co/guide/en/elasticsearch/reference/master/ip.html)，[`boolean`](https://www.elastic.co/guide/en/elasticsearch/reference/master/boolean.html)，或[`binary`](https://www.elastic.co/guide/en/elasticsearch/reference/master/binary.html)。

> NOTE: By default, you cannot run a `terms` aggregation on a `text` field. Use a `keyword` [sub-field](https://www.elastic.co/guide/en/elasticsearch/reference/master/multi-fields.html) instead. Alternatively, you can enable [`fielddata`](https://www.elastic.co/guide/en/elasticsearch/reference/master/fielddata.html) on the `text` field to create buckets for the field’s [analyzed](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) terms. Enabling `fielddata` can significantly increase memory usage.
>
> 默认情况下，您无法`terms`对`text`字段运行聚合。请改用 `keyword` [子字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/multi-fields.html)。或者，您可以[`fielddata`](https://www.elastic.co/guide/en/elasticsearch/reference/master/fielddata.html)在该`text`字段上启用 以为该字段的[分析](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)术语创建存储桶 。启用`fielddata`可以显着增加内存使用量。

###  Size

The `size` parameter can be set to define how many term buckets should be returned out of the overall terms list. By default, the node coordinating the search process will request each shard to provide its own top `size` term buckets and once all shards respond, it will reduce the results to the final list that will then be returned to the client. This means that if the number of unique terms is greater than `size`, the returned list is slightly off and not accurate (it could be that the term counts are slightly off and it could even be that a term that should have been in the top size buckets was not returned).

`size`可以设置该参数来定义应从整个术语列表中返回多少个术语桶。默认情况下，协调搜索过程的节点将请求每个分片提供自己的顶级`size`词桶，一旦所有分片都响应，它会将结果减少到最终列表，然后返回给客户端。这意味着，如果唯一术语的数量大于`size`，则返回的列表会略微偏离且不准确（可能是术语计数略有偏离，甚至可能是该术语本应位于顶级存储桶中没有退回）。

> NOTE: If you want to retrieve **all** terms or all combinations of terms in a nested `terms` aggregation you should use the [Composite](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html) aggregation which allows to paginate over all possible terms rather than setting a size greater than the cardinality of the field in the `terms` aggregation. The `terms` aggregation is meant to return the `top` terms and does not allow pagination.
>
> 如果要检索嵌套聚合中的**所有**术语或**所有**术语组合，`terms`您应该使用[Composite](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-composite-aggregation.html)聚合，它允许对所有可能的术语进行分页，而不是设置大于`terms`聚合中字段基数的大小 。该`terms`聚集是为了回报`top`条款，不允许分页。

###  Shard Size

The higher the requested `size` is, the more accurate the results will be, but also, the more expensive it will be to compute the final results (both due to bigger priority queues that are managed on a shard level and due to bigger data transfers between the nodes and the client).

请求的越高`size`，结果就越准确，但计算最终结果的成本也越高（这都是由于在分片级别管理的优先级队列更大，也由于在分片级别之间进行了更大的数据传输）节点和客户端）。

The `shard_size` parameter can be used to minimize the extra work that comes with bigger requested `size`. When defined, it will determine how many terms the coordinating node will request from each shard. Once all the shards responded, the coordinating node will then reduce them to a final result which will be based on the `size` parameter - this way, one can increase the accuracy of the returned terms and avoid the overhead of streaming a big list of buckets back to the client.

该`shard_size`参数可用于最小化更大的 requests 带来的额外工作`size`。定义后，它将确定协调节点将从每个分片请求多少项。一旦所有的分片都响应了，协调节点就会将它们归结为基于`size`参数的最终结果——这样，可以提高返回术语的准确性并避免将大量存储桶流回的开销客户端。

> NOTE: `shard_size` cannot be smaller than `size` (as it doesn’t make much sense). When it is, Elasticsearch will override it and reset it to be equal to `size`.
>
> `shard_size`不能小于`size`（因为它没有多大意义）。当它是时，Elasticsearch 将覆盖它并将其重置为等于`size`。

The default `shard_size` is `(size * 1.5 + 10)`.

默认`shard_size`值为`(size * 1.5 + 10)`.

###  Document count error

`doc_count` values for a `terms` aggregation may be approximate. As a result, any sub-aggregations on the `terms` aggregation may also be approximate.

`terms`聚合`doc_count`值可能是近似值。因此，聚合上的任何子`terms`聚合也可能是近似的。

To calculate `doc_count` values, each shard provides its own top terms and document counts. The aggregation combines these shard-level results to calculate its final `doc_count` values. To measure the accuracy of `doc_count` values, the aggregation results include the following properties:

为了计算`doc_count`值，每个分片都提供了自己的顶级术语和文档计数。聚合结合这些分片级结果来计算其最终`doc_count`值。为了衡量`doc_count`值的准确性，聚合结果包括以下属性：

- **`sum_other_doc_count`**

  (integer) The total document count for any terms not included in the results.

  （整数）未包含在结果中的任何术语的文档总数。

- **`doc_count_error_upper_bound`**

  (integer) The highest possible document count for any single term not included in the results. If `0`, `doc_count` values are accurate.

  （整数）未包含在结果中的任何单个术语的最高可能文档计数。如果`0`，`doc_count`值是准确的。

###  Per bucket document count error

To get the `doc_count_error_upper_bound` for each term, set `show_term_doc_count_error` to `true`:

要获取`doc_count_error_upper_bound`每个术语的 ，请设置 `show_term_doc_count_error`为`true`：

```console
GET /_search
{
  "aggs": {
    "products": {
      "terms": {
        "field": "product",
        "size": 5,
        "show_term_doc_count_error": true
      }
    }
  }
}
```

This shows an error value for each term returned by the aggregation which represents the *worst case* error in the document count and can be useful when deciding on a value for the `shard_size` parameter. This is calculated by summing the document counts for the last term returned by all shards which did not return the term.

这显示了聚合返回的每个术语的错误值，它表示文档计数中的*最坏情况*错误，并且在决定`shard_size`参数值时非常有用。这是通过将所有未返回该术语的分片返回的最后一个术语的文档计数相加来计算的。

These errors can only be calculated in this way when the terms are ordered by descending document count. When the aggregation is ordered by the terms values themselves (either ascending or descending) there is no error in the document count since if a shard does not return a particular term which appears in the results from another shard, it must not have that term in its index. When the aggregation is either sorted by a sub aggregation or in order of ascending document count, the error in the document counts cannot be determined and is given a value of -1 to indicate this.

只有当术语按文档计数降序排序时，才能以这种方式计算这些错误。当聚合按条款值本身（升序或降序）排序时，文档计数中没有错误，因为如果分片不返回出现在另一个分片结果中的特定条款，则该条款不得包含在它的索引。当聚合按子聚合排序或按文档计数升序排序时，文档计数中的错误无法确定，并被赋予值 -1 以表明这一点。

###  Order

The order of the buckets can be customized by setting the `order` parameter. By default, the buckets are ordered by their `doc_count` descending. It is possible to change this behaviour as documented below:

可以通过设置`order`参数来自定义桶的顺序。默认情况下，桶按`doc_count`降序排列。可以更改此行为，如下所述：

> WARNING: Sorting by ascending `_count` or by sub aggregation is discouraged as it increases the [error](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#terms-agg-doc-count-error) on document counts. It is fine when a single shard is queried, or when the field that is being aggregated was used as a routing key at index time: in these cases results will be accurate since shards have disjoint values. However otherwise, errors are unbounded. One particular case that could still be useful is sorting by [`min`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-min-aggregation.html) or [`max`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-max-aggregation.html) aggregation: counts will not be accurate but at least the top buckets will be correctly picked.
>
> `_count`不鼓励按升序或按子聚合排序，因为它会增加文档计数的 [错误](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#terms-agg-doc-count-error)。当查询单个分片时，或者正在聚合的字段在索引时用作路由键时，这很好：在这些情况下，结果将是准确的，因为分片具有不相交的值。然而，否则，错误是无限的。一种仍然有用的特殊情况是按排序[`min`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-min-aggregation.html)或 [`max`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-metrics-max-aggregation.html)聚合：计数将不准确，但至少会正确选择顶部的存储桶。

Ordering the buckets by their doc `_count` in an ascending manner:

`_count`以升序方式按文档对存储桶进行排序：

```console
GET /_search
{
  "aggs": {
    "genres": {
      "terms": {
        "field": "genre",
        "order": { "_count": "asc" }
      }
    }
  }
}
```

Ordering the buckets alphabetically by their terms in an ascending manner:

按字母顺序以升序方式对桶进行排序：

```console
GET /_search
{
  "aggs": {
    "genres": {
      "terms": {
        "field": "genre",
        "order": { "_key": "asc" }
      }
    }
  }
}
```

> WARNING: Deprecated in 6.0.0.
>
> Use `_key` instead of `_term` to order buckets by their term
>
> ###  在 6.0.0 中已弃用。
>
> 使用`_key`而不是`_term`按术语对存储桶进行排序

Ordering the buckets by single value metrics sub-aggregation (identified by the aggregation name):

按单值指标子聚合（由聚合名称标识）对存储桶进行排序：

```console
GET /_search
{
  "aggs": {
    "genres": {
      "terms": {
        "field": "genre",
        "order": { "max_play_count": "desc" }
      },
      "aggs": {
        "max_play_count": { "max": { "field": "play_count" } }
      }
    }
  }
}
```

Ordering the buckets by multi value metrics sub-aggregation (identified by the aggregation name):

按多值指标子聚合（由聚合名称标识）对存储桶进行排序：

```console
GET /_search
{
  "aggs": {
    "genres": {
      "terms": {
        "field": "genre",
        "order": { "playback_stats.max": "desc" }
      },
      "aggs": {
        "playback_stats": { "stats": { "field": "play_count" } }
      }
    }
  }
}
```

> NOTE:  Pipeline aggs cannot be used for sorting
>
> [Pipeline aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html) are run during the reduce phase after all other aggregations have already completed. For this reason, they cannot be used for ordering.
>
> ### 管道 aggs 不能用于排序
>
> 在所有其他聚合已经完成之后，[管道聚合](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline.html)在 reduce 阶段运行。因此，它们不能用于订购。

It is also possible to order the buckets based on a "deeper" aggregation in the hierarchy. This is supported as long as the aggregations path are of a single-bucket type, where the last aggregation in the path may either be a single-bucket one or a metrics one. If it’s a single-bucket type, the order will be defined by the number of docs in the bucket (i.e. `doc_count`), in case it’s a metrics one, the same rules as above apply (where the path must indicate the metric name to sort by in case of a multi-value metrics aggregation, and in case of a single-value metrics aggregation the sort will be applied on that value).

还可以根据层次结构中的“更深”聚合对存储桶进行排序。只要聚合路径是单桶类型，就支持这一点，其中路径中的最后一个聚合可能是单桶或指标。如果是单存储桶类型，则顺序将由存储桶中的文档数量定义（即`doc_count`），如果是度量标准，则适用与上述相同的规则（其中路径必须指示要排序的度量标准名称）在多值指标聚合的情况下，以及在单值指标聚合的情况下，排序将应用于该值）。

The path must be defined in the following form:

路径必须按以下形式定义：

```ebnf
AGG_SEPARATOR       =  '>' ;
METRIC_SEPARATOR    =  '.' ;
AGG_NAME            =  <the name of the aggregation> ;
METRIC              =  <the name of the metric (in case of multi-value metrics aggregation)> ;
PATH                =  <AGG_NAME> [ <AGG_SEPARATOR>, <AGG_NAME> ]* [ <METRIC_SEPARATOR>, <METRIC> ] ;
```

```console
GET /_search
{
  "aggs": {
    "countries": {
      "terms": {
        "field": "artist.country",
        "order": { "rock>playback_stats.avg": "desc" }
      },
      "aggs": {
        "rock": {
          "filter": { "term": { "genre": "rock" } },
          "aggs": {
            "playback_stats": { "stats": { "field": "play_count" } }
          }
        }
      }
    }
  }
}
```

The above will sort the artist’s countries buckets based on the average play count among the rock songs.

以上将根据摇滚歌曲中的平均播放次数对艺术家的国家/地区进行排序。

Multiple criteria can be used to order the buckets by providing an array of order criteria such as the following:

通过提供一系列排序条件，可以使用多个条件对存储桶进行排序，如下所示：

```console
GET /_search
{
  "aggs": {
    "countries": {
      "terms": {
        "field": "artist.country",
        "order": [ { "rock>playback_stats.avg": "desc" }, { "_count": "desc" } ]
      },
      "aggs": {
        "rock": {
          "filter": { "term": { "genre": "rock" } },
          "aggs": {
            "playback_stats": { "stats": { "field": "play_count" } }
          }
        }
      }
    }
  }
}
```

The above will sort the artist’s countries buckets based on the average play count among the rock songs and then by their `doc_count` in descending order.

以上将根据摇滚歌曲的平均播放次数对艺术家的国家/地区进行排序，然后按`doc_count`降序排列。

> NOTE: In the event that two buckets share the same values for all order criteria the bucket’s term value is used as a tie-breaker in ascending alphabetical order to prevent non-deterministic ordering of buckets.
>
> 如果两个存储桶的所有顺序标准共享相同的值，则存储桶的术语值用作按字母升序排列的决胜局，以防止存储桶的非确定性排序。

###  Minimum document count

It is possible to only return terms that match more than a configured number of hits using the `min_doc_count` option:

使用该`min_doc_count`选项可以只返回匹配超过配置数量的匹配项：

```console
GET /_search
{
  "aggs": {
    "tags": {
      "terms": {
        "field": "tags",
        "min_doc_count": 10
      }
    }
  }
}
```

The above aggregation would only return tags which have been found in 10 hits or more. Default value is `1`.

上述聚合只会返回在 10 个或更多点击中找到的标签。默认值为`1`。

Terms are collected and ordered on a shard level and merged with the terms collected from other shards in a second step. However, the shard does not have the information about the global document count available. The decision if a term is added to a candidate list depends only on the order computed on the shard using local shard frequencies. The `min_doc_count` criterion is only applied after merging local terms statistics of all shards. In a way the decision to add the term as a candidate is made without being very *certain* about if the term will actually reach the required `min_doc_count`. This might cause many (globally) high frequent terms to be missing in the final result if low frequent terms populated the candidate lists. To avoid this, the `shard_size` parameter can be increased to allow more candidate terms on the shards. However, this increases memory consumption and network traffic.

在分片级别收集和排序术语，并在第二步中与从其他分片收集的术语合并。但是，分片没有关于可用的全局文档计数的信息。是否将术语添加到候选列表的决定仅取决于使用本地分片频率在分片上计算的顺序。该`min_doc_count`标准仅在合并所有分片的本地术语统计信息后应用。从某种意义上说，决定将这个任期添加为候选人是在*不确定*该任期是否真正达到要求的情况下做出的`min_doc_count`。如果低频率术语填充了候选列表，这可能会导致最终结果中缺少许多（全局）高频率术语。为避免这种情况，`shard_size`可以增加参数以在分片上允许更多候选词。但是，这会增加内存消耗和网络流量。

#### `shard_min_doc_count`

The parameter `shard_min_doc_count` regulates the *certainty* a shard has if the term should actually be added to the candidate list or not with respect to the `min_doc_count`. Terms will only be considered if their local shard frequency within the set is higher than the `shard_min_doc_count`. If your dictionary contains many low frequent terms and you are not interested in those (for example misspellings), then you can set the `shard_min_doc_count` parameter to filter out candidate terms on a shard level that will with a reasonable certainty not reach the required `min_doc_count` even after merging the local counts. `shard_min_doc_count` is set to `0` per default and has no effect unless you explicitly set it.

该参数`shard_min_doc_count`规定了一个分片是否应该实际添加到候选列表中的*确定性*`min_doc_count`。仅当它们在集合中的本地分片频率高于`shard_min_doc_count`. 如果您的字典包含许多低频词并且您对这些词不感兴趣（例如拼写错误），那么您可以设置`shard_min_doc_count`参数以在分片级别上过滤候选词，`min_doc_count`即使合并后也不会达到要求的分片级别当地计数。`shard_min_doc_count`设置为`0`默认值，除非您明确设置，否则无效。

> NOTE: Setting `min_doc_count`=`0` will also return buckets for terms that didn’t match any hit. However, some of the returned terms which have a document count of zero might only belong to deleted documents or documents from other types, so there is no warranty that a `match_all` query would find a positive document count for those terms.
>
> 设置`min_doc_count`=`0`还将返回与任何命中都不匹配的术语的桶。但是，某些返回的文档计数为零的术语可能仅属于已删除的文档或来自其他类型的文档，因此无法保证`match_all`查询会找到这些术语的正文档计数。

> WARNING: When NOT sorting on `doc_count` descending, high values of `min_doc_count` may return a number of buckets which is less than `size` because not enough data was gathered from the shards. Missing buckets can be back by increasing `shard_size`. Setting `shard_min_doc_count` too high will cause terms to be filtered out on a shard level. This value should be set much lower than `min_doc_count/#shards`.
>
> 当不按`doc_count`降序排序时， 的高值`min_doc_count`可能会返回多个桶，该桶数小于 ，`size`因为没有从分片中收集到足够的数据。丢失的桶可以通过增加`shard_size`. 设置`shard_min_doc_count`太高会导致术语在分片级别被过滤掉。此值应设置得远低于`min_doc_count/#shards`。

###  Script

Use a [runtime field](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) if the data in your documents doesn’t exactly match what you’d like to aggregate. If, for example, "anthologies" need to be in a special category then you could run this:

如果文档中的数据与您想要聚合的数据不完全匹配，请使用[运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)。例如，如果“选集”需要属于特殊类别，那么您可以运行以下命令：

```console
GET /_search
{
  "size": 0,
  "runtime_mappings": {
    "normalized_genre": {
      "type": "keyword",
      "script": """
        String genre = doc['genre'].value;
        if (doc['product'].value.startsWith('Anthology')) {
          emit(genre + ' anthology');
        } else {
          emit(genre);
        }
      """
    }
  },
  "aggs": {
    "genres": {
      "terms": {
        "field": "normalized_genre"
      }
    }
  }
}
```

Which will look like:

看起来像：

```console-result
{
  "aggregations": {
    "genres": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "electronic",
          "doc_count": 4
        },
        {
          "key": "rock",
          "doc_count": 3
        },
        {
          "key": "electronic anthology",
          "doc_count": 2
        },
        {
          "key": "jazz",
          "doc_count": 2
        }
      ]
    }
  },
  ...
}
```

This is a little slower because the runtime field has to access two fields instead of one and because there are some optimizations that work on non-runtime `keyword` fields that we have to give up for for runtime `keyword` fields. If you need the speed, you can index the `normalized_genre` field.

这有点慢，因为运行时字段必须访问两个字段而不是一个字段，并且因为有一些优化适用于非运行时`keyword`字段，我们必须放弃运行时 `keyword`字段。如果您需要速度，您可以索引该 `normalized_genre`字段。

###  Filtering Values

It is possible to filter the values for which buckets will be created. This can be done using the `include` and `exclude` parameters which are based on regular expression strings or arrays of exact values. Additionally, `include` clauses can filter using `partition` expressions.

可以过滤将为其创建存储桶的值。这可以使用基于正则表达式字符串或精确值数组的`include`和 `exclude`参数来完成。此外， `include`子句可以使用`partition`表达式进行过滤。

####  Filtering Values with regular expressions

```console
GET /_search
{
  "aggs": {
    "tags": {
      "terms": {
        "field": "tags",
        "include": ".*sport.*",
        "exclude": "water_.*"
      }
    }
  }
}
```

In the above example, buckets will be created for all the tags that has the word `sport` in them, except those starting with `water_` (so the tag `water_sports` will not be aggregated). The `include` regular expression will determine what values are "allowed" to be aggregated, while the `exclude` determines the values that should not be aggregated. When both are defined, the `exclude` has precedence, meaning, the `include` is evaluated first and only then the `exclude`.

在上面的示例中，将为其中包含单词的所有标签创建桶`sport`，除了以开头`water_`的标签（因此标签`water_sports`不会被聚合）。该`include`正则表达式将决定什么样的价值观是“允许”被聚集，而`exclude`决定不应该聚合的价值。当两者都定义时， the`exclude`具有优先级，含义，`include`首先评估 ，然后才评估`exclude`。

The syntax is the same as [regexp queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html).

语法与正则[表达式查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html)相同。

####  Filtering Values with exact values

For matching based on exact values the `include` and `exclude` parameters can simply take an array of strings that represent the terms as they are found in the index:

对于基于精确值的匹配，`include`和`exclude`参数可以简单地采用表示在索引中找到的术语的字符串数组：

```console
GET /_search
{
  "aggs": {
    "JapaneseCars": {
      "terms": {
        "field": "make",
        "include": [ "mazda", "honda" ]
      }
    },
    "ActiveCarManufacturers": {
      "terms": {
        "field": "make",
        "exclude": [ "rover", "jensen" ]
      }
    }
  }
}
```

####  Filtering Values with partitions

Sometimes there are too many unique terms to process in a single request/response pair so it can be useful to break the analysis up into multiple requests. This can be achieved by grouping the field’s values into a number of partitions at query-time and processing only one partition in each request. Consider this request which is looking for accounts that have not logged any access recently:

有时在单个请求/响应对中需要处理太多独特的术语，因此将分析分解为多个请求会很有用。这可以通过在查询时将字段的值分组到多个分区中并在每个请求中仅处理一个分区来实现。考虑这个请求，它正在寻找最近没有记录任何访问的帐户：

```console
GET /_search
{
   "size": 0,
   "aggs": {
      "expired_sessions": {
         "terms": {
            "field": "account_id",
            "include": {
               "partition": 0,
               "num_partitions": 20
            },
            "size": 10000,
            "order": {
               "last_access": "asc"
            }
         },
         "aggs": {
            "last_access": {
               "max": {
                  "field": "access_date"
               }
            }
         }
      }
   }
}
```

This request is finding the last logged access date for a subset of customer accounts because we might want to expire some customer accounts who haven’t been seen for a long while. The `num_partitions` setting has requested that the unique account_ids are organized evenly into twenty partitions (0 to 19). and the `partition` setting in this request filters to only consider account_ids falling into partition 0. Subsequent requests should ask for partitions 1 then 2 etc to complete the expired-account analysis.

此请求正在查找客户帐户子集的上次记录访问日期，因为我们可能希望使一些长时间未访问的客户帐户过期。该`num_partitions`设置要求将唯一的 account_id 均匀地组织成 20 个分区（0 到 19）。并且`partition`此请求过滤器中的设置仅考虑落入分区 0 的 account_ids。后续请求应请求分区 1 然后 2 等以完成过期帐户分析。

Note that the `size` setting for the number of results returned needs to be tuned with the `num_partitions`. For this particular account-expiration example the process for balancing values for `size` and `num_partitions` would be as follows:

请注意，`size`返回结果数量的设置需要使用`num_partitions`. 对于这个特定的帐户到期示例，平衡`size`和值的过程`num_partitions`如下：

1. Use the `cardinality` aggregation to estimate the total number of unique account_id values

   使用`cardinality`聚合来估计唯一 account_id 值的总数

2. Pick a value for `num_partitions` to break the number from 1) up into more manageable chunks

   选择一个值`num_partitions`以将数字从 1) 分解为更易于管理的块

3. Pick a `size` value for the number of responses we want from each partition

   从每个分区中为我们想要的响应数量` size`选择一个值

4. Run a test request

   运行测试请求

If we have a circuit-breaker error we are trying to do too much in one request and must increase `num_partitions`. If the request was successful but the last account ID in the date-sorted test response was still an account we might want to expire then we may be missing accounts of interest and have set our numbers too low. We must either

如果我们有一个断路器错误，我们试图在一个请求中做太多事情并且必须增加`num_partitions`。如果请求成功，但按日期排序的测试响应中的最后一个帐户 ID 仍然是我们可能想要过期的帐户，那么我们可能缺少感兴趣的帐户并且将我们的数字设置得太低。我们必须要么

- increase the `size` parameter to return more results per partition (could be heavy on memory) or

  增加`size`参数以在每个分区返回更多结果（可能会占用大量内存）或

- increase the `num_partitions` to consider less accounts per request (could increase overall processing time as we need to make more requests)

  增加`num_partitions`每个请求考虑更少的帐户（可能会增加整体处理时间，因为我们需要提出更多请求）

Ultimately this is a balancing act between managing the Elasticsearch resources required to process a single request and the volume of requests that the client application must issue to complete a task.

归根结底，这是在管理处理单个请求所需的 Elasticsearch 资源与客户端应用程序必须发出以完成任务的请求量之间的平衡行为。

> WARNING: Partitions cannot be used together with an `exclude` parameter.
>
> 分区不能与`exclude`参数一起使用。

###  Multi-field terms aggregation

The `terms` aggregation does not support collecting terms from multiple fields in the same document. The reason is that the `terms` agg doesn’t collect the string term values themselves, but rather uses [global ordinals](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-execution-hint) to produce a list of all of the unique values in the field. Global ordinals results in an important performance boost which would not be possible across multiple fields.

该`terms`集合不支持从同一个文档中的多个领域收集方面。原因是`terms`agg 本身不收集字符串术语值，而是使用 [全局序数](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-execution-hint) 来生成字段中所有唯一值的列表。全局序数导致重要的性能提升，这在多个领域是不可能的。

There are three approaches that you can use to perform a `terms` agg across multiple fields:

您可以使用三种方法`terms`跨多个字段执行聚合：

- **[Script](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html#search-aggregations-bucket-terms-aggregation-script)**

  Use a script to retrieve terms from multiple fields. This disables the global ordinals optimization and will be slower than collecting terms from a single field, but it gives you the flexibility to implement this option at search time.

  使用脚本从多个字段中检索术语。这会禁用全局序数优化，并且比从单个字段收集术语要慢，但它使您可以在搜索时灵活地实施此选项。

- **[`copy_to` field](https://www.elastic.co/guide/en/elasticsearch/reference/master/copy-to.html)**

  If you know ahead of time that you want to collect the terms from two or more fields, then use `copy_to` in your mapping to create a new dedicated field at index time which contains the values from both fields. You can aggregate on this single field, which will benefit from the global ordinals optimization.

  如果您提前知道要从两个或多个字段中收集术语，则`copy_to`在映射中使用以在索引时创建一个新的专用字段，其中包含来自两个字段的值。您可以在这个单一字段上进行聚合，这将受益于全局序数优化。

- **[`multi_terms` aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-multi-terms-aggregation.html)**

  Use multi_terms aggregation to combine terms from multiple fields into a compound key. This also disables the global ordinals and will be slower than collecting terms from a single field. It is faster but less flexible than using a script.

  使用 multi_terms 聚合将来自多个字段的术语组合成一个复合键。这也会禁用全局序数，并且比从单个字段收集术语要慢。它比使用脚本更快但不那么灵活。

###  Collect mode

Deferring calculation of child aggregations

推迟子聚合的计算

For fields with many unique terms and a small number of required results it can be more efficient to delay the calculation of child aggregations until the top parent-level aggs have been pruned. Ordinarily, all branches of the aggregation tree are expanded in one depth-first pass and only then any pruning occurs. In some scenarios this can be very wasteful and can hit memory constraints. An example problem scenario is querying a movie database for the 10 most popular actors and their 5 most common co-stars:

对于具有许多唯一术语和少量所需结果的字段，将子聚合的计算延迟到顶级父级 agg 被修剪后会更有效。通常，聚合树的所有分支都在一次深度优先传递中扩展，然后才会发生任何修剪。在某些情况下，这可能非常浪费并且可能会遇到内存限制。一个示例问题场景是在电影数据库中查询 10 位最受欢迎的演员及其 5 位最常见的联合主演：

```console
GET /_search
{
  "aggs": {
    "actors": {
      "terms": {
        "field": "actors",
        "size": 10
      },
      "aggs": {
        "costars": {
          "terms": {
            "field": "actors",
            "size": 5
          }
        }
      }
    }
  }
}
```

Even though the number of actors may be comparatively small and we want only 50 result buckets there is a combinatorial explosion of buckets during calculation - a single actor can produce n² buckets where n is the number of actors. The sane option would be to first determine the 10 most popular actors and only then examine the top co-stars for these 10 actors. This alternative strategy is what we call the `breadth_first` collection mode as opposed to the `depth_first` mode.

即使演员的数量可能相对较少，并且我们只需要 50 个结果桶，但在计算过程中桶的组合爆炸 - 单个演员可以产生 n² 桶，其中 n 是演员的数量。明智的选择是首先确定 10 位最受欢迎的演员，然后再检查这 10 位演员的顶级联合主演。这种替代策略就是我们所说的`breadth_first`收集模式，而不是`depth_first`模式。

> NOTE: The `breadth_first` is the default mode for fields with a cardinality bigger than the requested size or when the cardinality is unknown (numeric fields or scripts for instance). It is possible to override the default heuristic and to provide a collect mode directly in the request:
>
> 这`breadth_first`是基数大于请求大小或基数未知（例如数字字段或脚本）的字段的默认模式。可以覆盖默认启发式并直接在请求中提供收集模式：

```console
GET /_search
{
  "aggs": {
    "actors": {
      "terms": {
        "field": "actors",
        "size": 10,
        "collect_mode": "breadth_first" (1)
      },
      "aggs": {
        "costars": {
          "terms": {
            "field": "actors",
            "size": 5
          }
        }
      }
    }
  }
}
```

1. the possible values are `breadth_first` and `depth_first`

    可能的值是`breadth_first`和`depth_first`

When using `breadth_first` mode the set of documents that fall into the uppermost buckets are cached for subsequent replay so there is a memory overhead in doing this which is linear with the number of matching documents. Note that the `order` parameter can still be used to refer to data from a child aggregation when using the `breadth_first` setting - the parent aggregation understands that this child aggregation will need to be called first before any of the other child aggregations.

当使用`breadth_first`mode 时，落入最上面的存储桶的文档集会被缓存以供后续重放，因此这样做的内存开销与匹配文档的数量成线性关系。请注意，`order`使用该`breadth_first`设置时，该参数仍可用于引用来自子聚合的数据- 父聚合理解需要先调用此子聚合，然后再调用任何其他子聚合。

> WARNING: Nested aggregations such as `top_hits` which require access to score information under an aggregation that uses the `breadth_first` collection mode need to replay the query on the second pass but only for the documents belonging to the top buckets.
>
> 嵌套聚合（例如`top_hits`需要访问使用`breadth_first` 集合模式的聚合下的分数信息）需要在第二次通过时重放查询，但仅针对属于顶级存储桶的文档。

###  Execution hint

There are different mechanisms by which terms aggregations can be executed:

可以通过不同的机制来执行术语聚合：

- by using field values directly in order to aggregate data per-bucket (`map`)

  通过直接使用字段值来聚合每个存储桶的数据 ( `map`)

- by using global ordinals of the field and allocating one bucket per global ordinal (`global_ordinals`)

  通过使用字段的全局序数并为每个全局序数分配一个桶 ( `global_ordinals`)

Elasticsearch tries to have sensible defaults so this is something that generally doesn’t need to be configured.

Elasticsearch 尝试使用合理的默认值，因此这通常不需要配置。

`global_ordinals` is the default option for `keyword` field, it uses global ordinals to allocates buckets dynamically so memory usage is linear to the number of values of the documents that are part of the aggregation scope.

`global_ordinals`是`keyword`字段的默认选项，它使用全局序数动态分配桶，因此内存使用与作为聚合范围一部分的文档的值数量呈线性关系。

`map` should only be considered when very few documents match a query. Otherwise the ordinals-based execution mode is significantly faster. By default, `map` is only used when running an aggregation on scripts, since they don’t have ordinals.

`map`仅当很少有文档与查询匹配时才应考虑。否则，基于序数的执行模式要快得多。默认情况下，`map`仅在对脚本运行聚合时使用，因为它们没有序数。

```console
GET /_search
{
  "aggs": {
    "tags": {
      "terms": {
        "field": "tags",
        "execution_hint": "map" (1)
      }
    }
  }
}
```

1. The possible values are `map`, `global_ordinals`
2. 可能的值是`map`，`global_ordinals`

Please note that Elasticsearch will ignore this execution hint if it is not applicable and that there is no backward compatibility guarantee on these hints.

请注意，如果此执行提示不适用，并且这些提示没有向后兼容性保证，则 Elasticsearch 将忽略该提示。

###  Missing value

The `missing` parameter defines how documents that are missing a value should be treated. By default they will be ignored but it is also possible to treat them as if they had a value.

该`missing`参数定义应如何处理缺少值的文档。默认情况下，它们将被忽略，但也可以将它们视为具有值。

```console
GET /_search
{
  "aggs": {
    "tags": {
      "terms": {
        "field": "tags",
        "missing": "N/A" (1)
      }
    }
  }
}
```

1.  Documents without a value in the `tags` field will fall into the same bucket as documents that have the value `N/A`.

    该`tags`字段中没有值的文档将与具有该值的文档落入同一个桶中`N/A`。

###  Mixing field types

> WARNING: When aggregating on multiple indices the type of the aggregated field may not be the same in all indices. Some types are compatible with each other (`integer` and `long` or `float` and `double`) but when the types are a mix of decimal and non-decimal number the terms aggregation will promote the non-decimal numbers to decimal numbers. This can result in a loss of precision in the bucket values.
>
> 当聚合多个索引时，聚合字段的类型在所有索引中可能不同。某些类型彼此兼容（`integer`和`long`或`float`和`double`），但是当类型是十进制数和非十进制数的混合时，术语聚合会将非十进制数提升为十进制数。这可能会导致存储桶值的精度损失。

####  Troubleshooting

####  Failed Trying to Format Bytes

When running a terms aggregation (or other aggregation, but in practice usually terms) over multiple indices, you may get an error that starts with "Failed trying to format bytes…". This is usually caused by two of the indices not having the same mapping type for the field being aggregated.

在多个索引上运行术语聚合（或其他聚合，但实际上通常是术语）时，您可能会收到以“尝试格式化字节失败...”开头的错误。这通常是由于两个索引对于聚合的字段没有相同的映射类型造成的。

**Use an explicit `value_type`** Although it’s best to correct the mappings, you can work around this issue if the field is unmapped in one of the indices. Setting the `value_type` parameter can resolve the issue by coercing the unmapped field into the correct type.

**使用显式`value_type`** 尽管最好更正映射，但如果该字段未在其中一个索引中映射，您可以解决此问题。设置`value_type`参数可以通过将未映射的字段强制为正确的类型来解决问题。

```console
GET /_search
{
  "aggs": {
    "ip_addresses": {
      "terms": {
        "field": "destination_ip",
        "missing": "0.0.0.0",
        "value_type": "ip"
      }
    }
  }
}
```

#  Variable width histogram aggregation

This is a multi-bucket aggregation similar to [Histogram](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-histogram-aggregation.html). However, the width of each bucket is not specified. Rather, a target number of buckets is provided and bucket intervals are dynamically determined based on the document distribution. This is done using a simple one-pass document clustering algorithm that aims to obtain low distances between bucket centroids. Unlike other multi-bucket aggregations, the intervals will not necessarily have a uniform width.

这是一个类似于[Histogram](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-histogram-aggregation.html)的多桶聚合。但是，没有指定每个桶的宽度。相反，提供了目标数量的桶，并且桶间隔是基于文档分布动态确定的。这是使用简单的一次性文档聚类算法完成的，该算法旨在获得桶质心之间的低距离。与其他多桶聚合不同，间隔不一定具有统一的宽度。

> TIP: The number of buckets returned will always be less than or equal to the target number.
>
> 返回的桶数将始终小于或等于目标数。

Requesting a target of 2 buckets.

请求 2 个存储桶的目标。

```console
POST /sales/_search?size=0
{
  "aggs": {
    "prices": {
      "variable_width_histogram": {
        "field": "price",
        "buckets": 2
      }
    }
  }
}
```

Response:

回复：

```console-result
{
  ...
  "aggregations": {
    "prices": {
      "buckets": [
        {
          "min": 10.0,
          "key": 30.0,
          "max": 50.0,
          "doc_count": 2
        },
        {
          "min": 150.0,
          "key": 185.0,
          "max": 200.0,
          "doc_count": 5
        }
      ]
    }
  }
}
```

> IMPORTANT: This aggregation cannot currently be nested under any aggregation that collects from more than a single bucket.
>
> 此聚合当前不能嵌套在从多个存储桶收集的任何聚合下。

###  Clustering Algorithm

Each shard fetches the first `initial_buffer` documents and stores them in memory. Once the buffer is full, these documents are sorted and linearly separated into `3/4 * shard_size buckets`. Next each remaining documents is either collected into the nearest bucket, or placed into a new bucket if it is distant from all the existing ones. At most `shard_size` total buckets are created.

每个分片获取第一个`initial_buffer`文档并将它们存储在内存中。一旦缓冲区已满，这些文档就会被排序并线性分离为`3/4 * shard_size buckets`. 接下来，每个剩余的文档要么被收集到最近的存储桶中，要么被放入一个新的存储桶，如果它与所有现有的文件都相距甚远。最多`shard_size`创建总存储桶。

In the reduce step, the coordinating node sorts the buckets from all shards by their centroids. Then, the two buckets with the nearest centroids are repeatedly merged until the target number of buckets is achieved. This merging procedure is a form of [agglomerative hierarchical clustering](https://en.wikipedia.org/wiki/Hierarchical_clustering).

在reduce 步骤中，协调节点按质心对所有分片中的桶进行排序。然后，重复合并具有最近质心的两个桶，直到达到目标桶数。这种合并过程是[凝聚层次聚类的](https://en.wikipedia.org/wiki/Hierarchical_clustering)一种形式。

> TIP: A shard can return fewer than `shard_size` buckets, but it cannot return more.
>
> 分片可以返回少于`shard_size`桶，但不能返回更多。

###  Shard size

The `shard_size` parameter specifies the number of buckets that the coordinating node will request from each shard. A higher `shard_size` leads each shard to produce smaller buckets. This reduce the likelihood of buckets overlapping after the reduction step. Increasing the `shard_size` will improve the accuracy of the histogram, but it will also make it more expensive to compute the final result because bigger priority queues will have to be managed on a shard level, and the data transfers between the nodes and the client will be larger.

该`shard_size`参数指定协调节点将从每个分片请求的桶数。更高的`shard_size`导致每个分片产生更小的桶。这减少了在减少步骤之后桶重叠的可能性。增加`shard_size`将提高直方图的准确性，但也会使计算最终结果的成本更高，因为必须在分片级别上管理更大的优先级队列，并且节点和客户端之间的数据传输将更大.

> TIP: Parameters `buckets`, `shard_size`, and `initial_buffer` are optional. By default, `buckets = 10`, `shard_size = buckets * 50`, and `initial_buffer = min(10 * shard_size, 50000)`.
>
> 参数`buckets`、`shard_size`和`initial_buffer`是可选的。默认情况下`buckets = 10`，`shard_size = buckets * 50`、 和`initial_buffer = min(10 * shard_size, 50000)`。

###  Initial Buffer

The `initial_buffer` parameter can be used to specify the number of individual documents that will be stored in memory on a shard before the initial bucketing algorithm is run. Bucket distribution is determined using this sample of `initial_buffer` documents. So, although a higher `initial_buffer` will use more memory, it will lead to more representative clusters.

该`initial_buffer`参数可用于指定在运行初始分桶算法之前将存储在分片内存中的单个文档的数量。使用此`initial_buffer`文档样本确定存储桶分布。因此，尽管更高`initial_buffer`会使用更多内存，但它会导致更具代表性的集群。

###  Bucket bounds are approximate

During the reduce step, the master node continuously merges the two buckets with the nearest centroids. If two buckets have overlapping bounds but distant centroids, then it is possible that they will not be merged. Because of this, after reduction the maximum value in some interval (`max`) might be greater than the minimum value in the subsequent bucket (`min`). To reduce the impact of this error, when such an overlap occurs the bound between these intervals is adjusted to be `(max + min) / 2`.

在reduce 步骤中，主节点不断地将两个bucket 与最近的质心合并。如果两个桶有重叠的边界但质心很远，那么它们可能不会合并。因此，减少后某个区间 ( `max`) 中的最大值可能大于后续存储区 ( `min`) 中的最小值。为了减少此错误的影响，当发生这种重叠时，这些间隔之间的界限被调整为`(max + min) / 2`。

> TIP: Bucket bounds are very sensitive to outliers
>
> 桶边界对异常值非常敏感

#  Subtleties of bucketing range fields

###  Documents are counted for each bucket they land in

Since a range represents multiple values, running a bucket aggregation over a range field can result in the same document landing in multiple buckets. This can lead to surprising behavior, such as the sum of bucket counts being higher than the number of matched documents. For example, consider the following index:

由于范围代表多个值，因此在范围字段上运行存储区聚合可能会导致同一文档进入多个存储区。这可能会导致令人惊讶的行为，例如存储桶计数的总和高于匹配文档的数量。例如，考虑以下索引：

```console
PUT range_index
{
  "settings": {
    "number_of_shards": 2
  },
  "mappings": {
    "properties": {
      "expected_attendees": {
        "type": "integer_range"
      },
      "time_frame": {
        "type": "date_range",
        "format": "yyyy-MM-dd||epoch_millis"
      }
    }
  }
}

PUT range_index/_doc/1?refresh
{
  "expected_attendees" : {
    "gte" : 10,
    "lte" : 20
  },
  "time_frame" : {
    "gte" : "2019-10-28",
    "lte" : "2019-11-04"
  }
}
```

The range is wider than the interval in the following aggregation, and thus the document will land in multiple buckets.

范围比后面聚合中的区间更宽，因此文档将落在多个桶中。

```console
POST /range_index/_search?size=0
{
  "aggs": {
    "range_histo": {
      "histogram": {
        "field": "expected_attendees",
        "interval": 5
      }
    }
  }
}
```

Since the interval is `5` (and the offset is `0` by default), we expect buckets `10`, `15`, and `20`. Our range document will fall in all three of these buckets.

由于间隔`5`（偏移量为`0`默认值），我们预计水桶`10`， `15`和`20`。我们的范围文档将落入所有这三个类别中。

```console-result
{
  ...
  "aggregations" : {
    "range_histo" : {
      "buckets" : [
        {
          "key" : 10.0,
          "doc_count" : 1
        },
        {
          "key" : 15.0,
          "doc_count" : 1
        },
        {
          "key" : 20.0,
          "doc_count" : 1
        }
      ]
    }
  }
}
```

A document cannot exist partially in a bucket; For example, the above document cannot count as one-third in each of the above three buckets. In this example, since the document’s range landed in multiple buckets, the full value of that document would also be counted in any sub-aggregations for each bucket as well.

文档不能部分存在于存储桶中；例如，上面的文档不能算作上述三个桶中每一个的三分之一。在此示例中，由于文档的范围落在多个存储桶中，因此该文档的完整值也将计入每个存储桶的任何子聚合中。

###  Query bounds are not aggregation filters

Another unexpected behavior can arise when a query is used to filter on the field being aggregated. In this case, a document could match the query but still have one or both of the endpoints of the range outside the query. Consider the following aggregation on the above document:

当使用查询过滤正在聚合的字段时，可能会出现另一种意外行为。在这种情况下，文档可以匹配查询，但仍然具有查询之外的范围的一个或两个端点。考虑对上述文档的以下聚合：

```console
POST /range_index/_search?size=0
{
  "query": {
    "range": {
      "time_frame": {
        "gte": "2019-11-01",
        "format": "yyyy-MM-dd"
      }
    }
  },
  "aggs": {
    "november_data": {
      "date_histogram": {
        "field": "time_frame",
        "calendar_interval": "day",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```

Even though the query only considers days in November, the aggregation generates 8 buckets (4 in October, 4 in November) because the aggregation is calculated over the ranges of all matching documents.

即使查询仅考虑 11 月的天数，聚合也会生成 8 个存储桶（10 月 4 个，11 月 4 个），因为聚合是在所有匹配文档的范围内计算的。

```console-result
{
  ...
  "aggregations" : {
    "november_data" : {
      "buckets" : [
              {
          "key_as_string" : "2019-10-28",
          "key" : 1572220800000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2019-10-29",
          "key" : 1572307200000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2019-10-30",
          "key" : 1572393600000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2019-10-31",
          "key" : 1572480000000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2019-11-01",
          "key" : 1572566400000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2019-11-02",
          "key" : 1572652800000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2019-11-03",
          "key" : 1572739200000,
          "doc_count" : 1
        },
        {
          "key_as_string" : "2019-11-04",
          "key" : 1572825600000,
          "doc_count" : 1
        }
      ]
    }
  }
}
```

Depending on the use case, a `CONTAINS` query could limit the documents to only those that fall entirely in the queried range. In this example, the one document would not be included and the aggregation would be empty. Filtering the buckets after the aggregation is also an option, for use cases where the document should be counted but the out of bounds data can be safely ignored.

根据用例，`CONTAINS`查询可以将文档限制为完全落在查询范围内的文档。在此示例中，不会包含一个文档，并且聚合将为空。在聚合后过滤桶也是一种选择，适用于应该计算文档但可以安全地忽略越界数据的用例。