#  Joining queries

Performing full SQL-style joins in a distributed system like Elasticsearch is prohibitively expensive. Instead, Elasticsearch offers two forms of join which are designed to scale horizontally.

在像 Elasticsearch 这样的分布式系统中执行完整的 SQL 风格的连接是非常昂贵的。相反，Elasticsearch 提供了两种形式的连接，旨在水平扩展。

**[`nested` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-nested-query.html)**

- Documents may contain fields of type [`nested`](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html). These fields are used to index arrays of objects, where each object can be queried (with the `nested` query) as an independent document.

  文档可能包含类型为 [`nested`](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html) 的字段。这些字段用于索引对象数组，其中每个对象都可以作为独立文档进行查询（使用`nested`查询）。

**[`has_child`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-has-child-query.html) and [`has_parent`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-has-parent-query.html) queries**

- A [`join` field relationship](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) can exist between documents within a single index. The `has_child` query returns parent documents whose child documents match the specified query, while the `has_parent` query returns child documents whose parent document matches the specified query.

  [`join`字段关系](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)可以在单个索引中的文档之间存在。`has_child`查询返回父文档，其子文档匹配指定的查询，而 `has_parent`查询返回子文档的父文件指定的查询相匹配。

Also see the [terms-lookup mechanism](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-query.html#query-dsl-terms-lookup) in the `terms` query, which allows you to build a `terms` query from values contained in another document.

另请参阅[条款查找机制](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-query.html#query-dsl-terms-lookup)在`terms` 查询中，它允许你建立一个`terms`由包含在另一个文件值的查询。

##  Notes

###  Allow expensive queries

Joining queries will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) 设置为 false，则不会执行联接查询。

#  Nested query

Wraps another query to search [nested](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html) fields.

包装另一个查询以搜索[嵌套](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html)字段。

The `nested` query searches nested field objects as if they were indexed as separate documents. If an object matches the search, the `nested` query returns the root parent document.

该`nested`查询搜索嵌套Field对象，好像他们是索引作为单独的文档。如果对象与搜索匹配，则`nested`查询返回根父文档。

##  Example request

###  Index setup

To use the `nested` query, your index must include a [nested](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html) field mapping. For example:

要使用`nested`查询，您的索引必须包含[嵌套](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html)字段映射。例如：

```console
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "obj1": {
        "type": "nested"
      }
    }
  }
}
```

###  Example query

```console
GET /my-index-000001/_search
{
  "query": {
    "nested": {
      "path": "obj1",
      "query": {
        "bool": {
          "must": [
            { "match": { "obj1.name": "blue" } },
            { "range": { "obj1.count": { "gt": 5 } } }
          ]
        }
      },
      "score_mode": "avg"
    }
  }
}
```

##  Top-level parameters for `nested`

- **`path`**

  (Required, string) Path to the nested object you wish to search.

  （必需，字符串）要搜索的嵌套对象的路径。

- **`query`**

  (Required, query object) Query you wish to run on nested objects in the `path`. If an object matches the search, the `nested` query returns the root parent document.

  （必需，查询对象）您希望在`path`. 如果对象与搜索匹配，则`nested`查询返回根父文档。

  You can search nested fields using dot notation that includes the complete path, such as `obj1.name`.

  您可以使用包含完整路径的点表示法搜索嵌套字段，例如`obj1.name`.

  Multi-level nesting is automatically supported, and detected, resulting in an inner nested query to automatically match the relevant nesting level, rather than root, if it exists within another nested query.

  自动支持和检测多级嵌套，从而导致内部嵌套查询自动匹配相关嵌套级别，而不是根（如果它存在于另一个嵌套查询中）。

  See [Multi-level nested queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-nested-query.html#multi-level-nested-query-ex) for an example.

  有关示例，请参阅[多级嵌套查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-nested-query.html#multi-level-nested-query-ex)。

- **`score_mode`**

  (Optional, string) Indicates how scores for matching child objects affect the root parent document’s [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores). Valid values are:

  （可选，字符串）指示匹配子对象的分数如何影响根父文档的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)。有效值为：

  - **`avg` (Default)**

    Use the mean relevance score of all matching child objects.

    使用所有匹配子对象的平均相关性分数。

  - **`max`**

    Uses the highest relevance score of all matching child objects.

    使用所有匹配子对象的最高相关性分数。

  - **`min`**

    Uses the lowest relevance score of all matching child objects.

    使用所有匹配子对象的最低相关性分数。

  - **`none`**

    Do not use the relevance scores of matching child objects. The query assigns parent documents a score of `0`.

    不要使用匹配子对象的相关性分数。该查询为父文档分配一个分数`0`。

  - **`sum`**

    Add together the relevance scores of all matching child objects.

    将所有匹配子对象的相关性分数加在一起。

- **`ignore_unmapped`**

  (Optional, Boolean) Indicates whether to ignore an unmapped `path` and not return any documents instead of an error. Defaults to `false`.

  （可选，布尔值）指示是否忽略未映射`path`且不返回任何文档而不是错误。默认为`false`.

  If `false`, Elasticsearch returns an error if the `path` is an unmapped field.

  如果`false`，如果`path`是未映射的字段，Elasticsearch 将返回错误。

  You can use this parameter to query multiple indices that may not contain the field `path`.

  您可以使用此参数查询可能不包含该字段的多个索引`path`。

##  Notes

###  Multi-level nested queries

To see how multi-level nested queries work, first you need an index that has nested fields. The following request defines mappings for the `drivers` index with nested `make` and `model` fields.

要了解多级嵌套查询的工作原理，首先需要一个具有嵌套字段的索引。以下请求定义了`drivers`具有嵌套`make`和`model`字段的索引的映射。

```console
PUT /drivers
{
  "mappings": {
    "properties": {
      "driver": {
        "type": "nested",
        "properties": {
          "last_name": {
            "type": "text"
          },
          "vehicle": {
            "type": "nested",
            "properties": {
              "make": {
                "type": "text"
              },
              "model": {
                "type": "text"
              }
            }
          }
        }
      }
    }
  }
}
```

Next, index some documents to the `drivers` index.

接下来，将一些文档`drivers`索引到索引中。

```console
PUT /drivers/_doc/1
{
  "driver" : {
        "last_name" : "McQueen",
        "vehicle" : [
            {
                "make" : "Powell Motors",
                "model" : "Canyonero"
            },
            {
                "make" : "Miller-Meteor",
                "model" : "Ecto-1"
            }
        ]
    }
}

PUT /drivers/_doc/2?refresh
{
  "driver" : {
        "last_name" : "Hudson",
        "vehicle" : [
            {
                "make" : "Mifune",
                "model" : "Mach Five"
            },
            {
                "make" : "Miller-Meteor",
                "model" : "Ecto-1"
            }
        ]
    }
}
```

You can now use a multi-level nested query to match documents based on the `make` and `model` fields.

您现在可以使用多级嵌套查询来匹配基于`make`和`model`字段的文档。

```console
GET /drivers/_search
{
  "query": {
    "nested": {
      "path": "driver",
      "query": {
        "nested": {
          "path": "driver.vehicle",
          "query": {
            "bool": {
              "must": [
                { "match": { "driver.vehicle.make": "Powell Motors" } },
                { "match": { "driver.vehicle.model": "Canyonero" } }
              ]
            }
          }
        }
      }
    }
  }
}
```

The search request returns the following response:

搜索请求返回以下响应：

```console
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 3.7349272,
    "hits" : [
      {
        "_index" : "drivers",
        "_id" : "1",
        "_score" : 3.7349272,
        "_source" : {
          "driver" : {
            "last_name" : "McQueen",
            "vehicle" : [
              {
                "make" : "Powell Motors",
                "model" : "Canyonero"
              },
              {
                "make" : "Miller-Meteor",
                "model" : "Ecto-1"
              }
            ]
          }
        }
      }
    ]
  }
}
```

#  Has child query

Returns parent documents whose [joined](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) child documents match a provided query. You can create parent-child relationships between documents in the same index using a [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field mapping.

返回其[加入的](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)子文档与提供的查询匹配的父文档。您可以使用[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)字段映射在同一索引中的文档之间创建父子关系。

> **WARNING:** 
> Because it performs a join, the `has_child` is slow compared to other queries. Its performance degrades as the number of matching child documents pointing to unique parent documents increases. Each `has_child` query in a search can increase query time significantly.
>
> If you care about query performance, do not use this query. If you need to use the `has_child` query, use it as rarely as possible.
>
> 
> 因为它执行连接，所以`has_child`与其他查询相比速度较慢。随着指向唯一父文档的匹配子文档数量的增加，其性能会下降。`has_child`搜索中的每个查询都会显着增加查询时间。
>
> 如果您关心查询性能，请不要使用此查询。如果您需要使用`has_child`查询，请尽可能少地使用它。

##  Example request

###  Index setup

To use the `has_child` query, your index must include a [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field mapping. For example:

要使用`has_child`查询，您的索引必须包含[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) 字段映射。例如：

```console
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "my-join-field": {
        "type": "join",
        "relations": {
          "parent": "child"
        }
      }
    }
  }
}
```

###  Example query

```console
GET /_search
{
  "query": {
    "has_child": {
      "type": "child",
      "query": {
        "match_all": {}
      },
      "max_children": 10,
      "min_children": 2,
      "score_mode": "min"
    }
  }
}
```

##  Top-level parameters for `has_child`

- **`type`**

  (Required, string) Name of the child relationship mapped for the [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field.

  （必需，字符串）为[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)字段映射的子关系的名称 。

- **`query`**

  (Required, query object) Query you wish to run on child documents of the `type` field. If a child document matches the search, the query returns the parent document.

  （必需，查询对象）您希望在该`type` 字段的子文档上运行的查询。如果子文档与搜索匹配，则查询返回父文档。

- **`ignore_unmapped`**

  (Optional, Boolean) Indicates whether to ignore an unmapped `type` and not return any documents instead of an error. Defaults to `false`.

  （可选，布尔值）指示是否忽略未映射`type`且不返回任何文档而不是错误。默认为`false`.

  If `false`, Elasticsearch returns an error if the `type` is unmapped.

  如果`false`，如果`type`未映射，Elasticsearch 将返回错误。

  You can use this parameter to query multiple indices that may not contain the `type`.

  您可以使用此参数查询可能不包含 `type`.

- **`max_children`**

  (Optional, integer) Maximum number of child documents that match the `query` allowed for a returned parent document. If the parent document exceeds this limit, it is excluded from the search results.

  （可选，整数）与`query` 返回的父文档所允许的匹配的最大子文档数。如果父文档超过此限制，则将其从搜索结果中排除。

- **`min_children`**

  (Optional, integer) Minimum number of child documents that match the `query` required to match the query for a returned parent document. If the parent document does not meet this limit, it is excluded from the search results.

  （可选，整数）`query` 与返回的父文档的查询匹配所需的最小子文档数。如果父文档不符合此限制，则将其从搜索结果中排除。

- **`score_mode`**

  (Optional, string) Indicates how scores for matching child documents affect the root parent document’s [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores). Valid values are:

  （可选，字符串）指示匹配子文档的分数如何影响根父文档的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)。有效值为：

  - **`none` (Default)**

    Do not use the relevance scores of matching child documents. The query assigns parent documents a score of `0`.

    不要使用匹配子文档的相关性分数。该查询为父文档分配一个分数`0`。

  - **`avg`**

    Use the mean relevance score of all matching child documents.

    使用所有匹配子文档的平均相关性分数。

  - **`max`**

    Uses the highest relevance score of all matching child documents.

    使用所有匹配子文档的最高相关性分数。

  - **`min`**

    Uses the lowest relevance score of all matching child documents.

    使用所有匹配子文档的最低相关性分数。

  - **`sum`**

    Add together the relevance scores of all matching child documents.

    将所有匹配的子文档的相关性分数加在一起。

##  Notes

###  Sorting

You cannot sort the results of a `has_child` query using standard [sort options](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html).

您不能`has_child`使用标准[排序选项](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html)对查询 结果进行[排序](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html)。

If you need to sort returned documents by a field in their child documents, use a `function_score` query and sort by `_score`. For example, the following query sorts returned documents by the `click_count` field of their child documents.

如果您需要按子文档中的字段对返回的文档进行排序，请使用`function_score`查询和排序依据`_score`。例如，以下查询`click_count`按其子文档的字段对返回的文档进行排序。

```console
GET /_search
{
  "query": {
    "has_child": {
      "type": "child",
      "query": {
        "function_score": {
          "script_score": {
            "script": "_score * doc['click_count'].value"
          }
        }
      },
      "score_mode": "max"
    }
  }
}
```

#  Has parent query

Returns child documents whose [joined](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) parent document matches a provided query. You can create parent-child relationships between documents in the same index using a [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field mapping.

返回其[加入的](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)父文档与提供的查询匹配的子文档。您可以使用[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)字段映射在同一索引中的文档之间创建父子关系。

> **WARNING:** Because it performs a join, the `has_parent` query is slow compared to other queries. Its performance degrades as the number of matching parent documents increases. Each `has_parent` query in a search can increase query time significantly.
>
> 由于它执行连接，因此`has_parent`与其他查询相比，查询速度较慢。随着匹配父文档数量的增加，其性能会下降。`has_parent`搜索中的每个查询都会显着增加查询时间。

##  Example request

###  Index setup

To use the `has_parent` query, your index must include a [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field mapping. For example:

要使用`has_parent`查询，您的索引必须包含[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) 字段映射。例如：

```console
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "my-join-field": {
        "type": "join",
        "relations": {
          "parent": "child"
        }
      },
      "tag": {
        "type": "keyword"
      }
    }
  }
}
```

###  Example query

```console
GET /my-index-000001/_search
{
  "query": {
    "has_parent": {
      "parent_type": "parent",
      "query": {
        "term": {
          "tag": {
            "value": "Elasticsearch"
          }
        }
      }
    }
  }
}
```

##  Top-level parameters for `has_parent`

- **`parent_type`**

  (Required, string) Name of the parent relationship mapped for the [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field.

  （必需，字符串）为[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)字段映射的父关系的名称 。

- **`query`**

  (Required, query object) Query you wish to run on parent documents of the `parent_type` field. If a parent document matches the search, the query returns its child documents.

  （必需，查询对象）您希望在`parent_type`字段的父文档上运行的查询 。如果父文档与搜索匹配，则查询返回其子文档。

- **`score`**

  (Optional, Boolean) Indicates whether the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html) of a matching parent document is aggregated into its child documents. Defaults to `false`.

  （可选，布尔值）指示匹配父文档的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html)是否聚合到其子文档中。默认为`false`.

  If `false`, Elasticsearch ignores the relevance score of the parent document. Elasticsearch also assigns each child document a relevance score equal to the `query`'s `boost`, which defaults to `1`.

  如果`false`，Elasticsearch 会忽略父文档的相关性分数。Elasticsearch 还为每个子文档分配一个与`query`'s相等的相关性分数`boost`，默认为`1`。

  If `true`, the relevance score of the matching parent document is aggregated into its child documents' relevance scores.

  如果`true`，则匹配父文档的相关性分数被聚合到其子文档的相关性分数中。

- **`ignore_unmapped`**

  (Optional, Boolean) Indicates whether to ignore an unmapped `parent_type` and not return any documents instead of an error. Defaults to `false`.

  （可选，布尔值）指示是否忽略未映射`parent_type`且不返回任何文档而不是错误。默认为`false`.

  If `false`, Elasticsearch returns an error if the `parent_type` is unmapped.

  如果`false`，如果`parent_type`未映射，Elasticsearch 将返回错误。

  You can use this parameter to query multiple indices that may not contain the `parent_type`.

  您可以使用此参数查询可能不包含 `parent_type`.

##  Notes

###  Sorting

You cannot sort the results of a `has_parent` query using standard [sort options](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html).

您不能`has_parent`使用标准[排序选项](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html)对查询 结果进行[排序](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html)。

If you need to sort returned documents by a field in their parent documents, use a `function_score` query and sort by `_score`. For example, the following query sorts returned documents by the `view_count` field of their parent documents.

如果您需要按父文档中的字段对返回的文档进行排序，请使用`function_score`查询和排序依据`_score`。例如，以下查询按`view_count`其父文档的字段对返回的文档进行排序。

```console
GET /_search
{
  "query": {
    "has_parent": {
      "parent_type": "parent",
      "score": true,
      "query": {
        "function_score": {
          "script_score": {
            "script": "_score * doc['view_count'].value"
          }
        }
      }
    }
  }
}
```

#  Parent ID query

Returns child documents [joined](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) to a specific parent document. You can use a [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field mapping to create parent-child relationships between documents in the same index.

返回[加入](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)特定父文档的子文档。您可以使用[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)字段映射在同一索引中的文档之间创建父子关系。

##  Example request

###  Index setup

To use the `parent_id` query, your index must include a [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field mapping. To see how you can set up an index for the `parent_id` query, try the following example.

要使用`parent_id`查询，您的索引必须包含[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) 字段映射。要了解如何为`parent_id`查询设置索引，请尝试以下示例。

1. Create an index with a [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field mapping.

   使用[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)字段映射创建索引。

   ```console
   PUT /my-index-000001
   {
     "mappings": {
       "properties": {
         "my-join-field": {
           "type": "join",
           "relations": {
             "my-parent": "my-child"
           }
         }
       }
     }
   }
   ```

2. Index a parent document with an ID of `1`.

   索引 ID 为 的父文档`1`。

   ```console
   PUT /my-index-000001/_doc/1?refresh
   {
     "text": "This is a parent document.",
     "my-join-field": "my-parent"
   }
   ```

3. Index a child document of the parent document.

   索引父文档的子文档。

   ```console
   PUT /my-index-000001/_doc/2?routing=1&refresh
   {
     "text": "This is a child document.",
     "my_join_field": {
       "name": "my-child",
       "parent": "1"
     }
   }
   ```

##  Example query

The following search returns child documents for a parent document with an ID of `1`.

以下搜索返回 ID 为 的父文档的子文档 `1`。

```console
GET /my-index-000001/_search
{
  "query": {
      "parent_id": {
          "type": "my-child",
          "id": "1"
      }
  }
}
```

##  Top-level parameters for `parent_id`

- **`type`**

  (Required, string) Name of the child relationship mapped for the [join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) field.

  （必需，字符串）为[连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)字段映射的子关系的名称 。

- **`id`**

  (Required, string) ID of the parent document. The query will return child documents of this parent document.

  （必需，字符串）父文档的 ID。查询将返回此父文档的子文档。

- **`ignore_unmapped`**

  (Optional, Boolean) Indicates whether to ignore an unmapped `type` and not return any documents instead of an error. Defaults to `false`.

  （可选，布尔值）指示是否忽略未映射`type`且不返回任何文档而不是错误。默认为`false`.

  If `false`, Elasticsearch returns an error if the `type` is unmapped.

  如果`false`，如果`type`未映射，Elasticsearch 将返回错误。

  You can use this parameter to query multiple indices that may not contain the `type`.

  您可以使用此参数查询可能不包含 type.