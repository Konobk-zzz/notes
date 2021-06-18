#  Query DSL

Elasticsearch provides a full Query DSL (Domain Specific Language) based on JSON to define queries. Think of the Query DSL as an AST (Abstract Syntax Tree) of queries, consisting of two types of clauses:

Elasticsearch提供了基于JSON的完整查询DSL（特定于域的语言）来定义查询。将查询DSL视为查询的AST（抽象语法树），它由两种子句组成：

## **Leaf query clauses**

Leaf query clauses look for a particular value in a particular field, such as the [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html), [`term`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html) or [`range`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html) queries. These queries can be used by themselves.

叶查询子句中寻找一个特定的值在某一特定领域，如 [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)，[`term`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)或 [`range`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html)查询。这些查询可以自己使用。

## **Compound query clauses**

Compound query clauses wrap other leaf **or** compound queries and are used to combine multiple queries in a logical fashion (such as the [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) or [`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-dis-max-query.html) query), or to alter their behaviour (such as the [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-constant-score-query.html) query).

复合查询子句包装其他叶查询**或**复合查询，并用于以逻辑方式组合多个查询（例如 [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)或[`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-dis-max-query.html)查询），或更改其行为（例如 [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-constant-score-query.html)查询）。

Query clauses behave differently depending on whether they are used in [query context or filter context](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html).

查询子句的行为有所不同，具体取决于它们是在 [查询上下文中还是在过滤器上下文中使用](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html)。

## **Allow expensive queries**

Certain types of queries will generally execute slowly due to the way they are implemented, which can affect the stability of the cluster. Those queries can be categorised as follows:

某些类型的查询由于其实现方式而通常会缓慢执行，这可能会影响群集的稳定性。这些查询可以分类如下：

- Queries that need to do linear scans to identify matches:

  需要进行线性扫描以识别匹配项的查询：

  - [`script` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-query.html)

    [`脚本`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-query.html)

- Queries that have a high up-front cost:

  前期费用高的查询：

  - [`fuzzy` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-fuzzy-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type) fields)

    [`模糊`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-fuzzy-query.html)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)字段上除外 ）

  - [`regexp` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-regexp-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type) fields)

    [`正则`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-regexp-query.html)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)字段上除外 ）

  - [`prefix` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type) fields or those without [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html))

    [`前缀`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html) （[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)字段或不包含的 [查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html)除外 [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html)）

  - [`wildcard` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-wildcard-query.html) (except on [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type) fields)

    [`通配符`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-wildcard-query.html)（[`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html#wildcard-field-type)字段上除外 ）

  - [`range` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html) on [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) and [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html) fields

    [`范围`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html)上[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html)和 [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html)领域

- [Joining queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/joining-queries.html)

  [联接查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/joining-queries.html)

- Queries on [deprecated geo-shapes](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html#prefix-trees)

  关于[过时的地理形状的](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html#prefix-trees)查询

- Queries that may have a high per-document cost:

  每个文档的查询费用可能很高：

  - [`script_score` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html)

    [`脚本_评分` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html)

  - [`percolate` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-percolate-query.html)

    [`过滤` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-percolate-query.html)

The execution of such queries can be prevented by setting the value of the `search.allow_expensive_queries` setting to `false` (defaults to `true`).

可以通过将设置`search.allow_expensive_queries` 的值设置为`false`（默认为`true`）来阻止执行此类查询。

# Query and filter context

### Relevance scores

By default, Elasticsearch sorts matching search results by **relevance score**, which measures how well each document matches a query.

默认情况下，Elasticsearch按**相关性得分对**匹配的搜索结果进行**排序**，该**得分**衡量每个文档与查询的匹配程度。

The relevance score is a positive floating point number, returned in the `_score` metadata field of the [search](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-request-body.html) API. The higher the `_score`, the more relevant the document. While each query type can calculate relevance scores differently, score calculation also depends on whether the query clause is run in a **query** or **filter** context.

相关性分数是一个正浮点数`_score`，在[搜索](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-request-body.html)API的元数据字段中返回 。越高 `_score`，文档越相关。虽然每种查询类型可以不同地计算相关性分数，但是分数计算还取决于查询子句是在**查询**上下文中还是在**过滤器**上下文中运行。

### Query context

In the query context, a query clause answers the question “*How well does this document match this query clause?*” Besides deciding whether or not the document matches, the query clause also calculates a relevance score in the `_score` metadata field.

在查询上下文中，查询子句回答问题“*此文档与该查询子句的匹配程度如何？*”除了确定文档是否匹配之外，查询子句还计算`_score`元数据字段中的相关性得分 。

Query context is in effect whenever a query clause is passed to a `query` parameter, such as the `query` parameter in the [search](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-query) API.

查询上下文是影响每当查询子句被传递给一个`query` 参数，如`query`该参数 [搜索](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-query)API。

### Filter context

In a filter context, a query clause answers the question “*Does this document match this query clause?*” The answer is a simple Yes or No — no scores are calculated. Filter context is mostly used for filtering structured data, e.g.

在过滤器上下文中，查询子句回答问题“*此文档是否与此查询子句匹配？*答案是简单的“是”或“否”-不计算分数。过滤器上下文主要用于过滤结构化数据，例如

- *Does this `timestamp` fall into the range 2015 to 2016?*

  *这是否`timestamp`属于2015年至2016年的范围？*

- *Is the `status` field set to `"published"`*?

  *该`status` 字段设置为`"published"`*吗？

Frequently used filters will be cached automatically by Elasticsearch, to speed up performance.

常用的过滤器将由Elasticsearch自动缓存，以提高性能。

Filter context is in effect whenever a query clause is passed to a `filter` parameter, such as the `filter` or `must_not` parameters in the [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) query, the `filter` parameter in the [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-constant-score-query.html) query, or the [`filter`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-filter-aggregation.html) aggregation.

每当将查询子句传递到`filter` 参数（例如查询中的`filter`或`must_not`参数， [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)查询中的`filter`参数 [`constant_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-constant-score-query.html)或[`filter`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-filter-aggregation.html)聚合）时， 过滤器上下文即有效。

### Example of query and filter contexts

Below is an example of query clauses being used in query and filter context in the `search` API. This query will match documents where all of the following conditions are met:

以下是在`search`API的查询和过滤器上下文中使用的查询子句的示例。此查询将匹配满足以下所有条件的文档：

- The `title` field contains the word `search`.

  `title`字段包含单词`search`。

- The `content` field contains the word `elasticsearch`.

  `content`字段包含单词`elasticsearch`。

- The `status` field contains the exact word `published`.

  `status`字段包含确切的单词`published`。

- The `publish_date` field contains a date from 1 Jan 2015 onwards.

  `publish_date`字段包含从2015年1月1日开始的日期。

```json
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
```

1.  The `query` parameter indicates query context.

   `query`参数表示查询上下文。

2.  The `bool` and two `match` clauses are used in query context, which means that they are used to score how well each document matches.

   `bool`的两个`match`子句被用于查询上下文，意味着它们被用来评定每个文档有多匹配的得分。

3.  The `filter` parameter indicates filter context. Its `term` and `range` clauses are used in filter context. They will filter out documents which do not match, but they will not affect the score for matching documents.

   `filter`参数指示过滤器上下文。它的`term`and `range`子句在过滤器上下文中使用。它们将过滤出不匹配的文档，但不会影响匹配文档的分数。

> **WARNING:** Scores calculated for queries in query context are represented as single precision floating point numbers; they have only 24 bits for significand’s precision. Score calculations that exceed the significand’s precision will be converted to floats with loss of precision.
>
> **警告：** 在查询上下文中为查询计算的分数表示为单精度浮点数；他们只有24位有效数字的精度。超过有效位数精度的分数计算将转换为浮点数而失去精度。

> **TIP:** Use query clauses in query context for conditions which should affect the score of matching documents (i.e. how well does the document match), and use all other query clauses in filter context.
>
> **提示：** 在查询上下文中使用查询子句以应对可能影响匹配文档得分（即文档匹配程度）的条件，并在过滤器上下文中使用所有其他查询子句。

#  Compound queries

Compound queries wrap other compound or leaf queries, either to combine their results and scores, to change their behaviour, or to switch from query to filter context.

复合查询包装其他复合查询或叶查询，以组合其结果和分数，更改其行为或从查询切换到过滤器上下文。

The queries in this group are:

**[`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)**

The default query for combining multiple leaf or compound query clauses, as `must`, `should`, `must_not`, or `filter` clauses. The `must` and `should` clauses have their scores combined — the more matching clauses, the better — while the `must_not` and `filter` clauses are executed in filter context.

用于组合多个叶或复合查询子句，作为默认查询 `must`，`should`，`must_not`，或`filter`子句。在`must`和`should` 子句有他们的分数相结合-更匹配的子句，更好地-当`must_not`和`filter`子句在过滤器上下文中执行。

**[`boosting` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-boosting-query.html)**

Return documents which match a `positive` query, but reduce the score of documents which also match a `negative` query.

返回与`positive`查询匹配的文档，但减少与`negative`查询匹配的文档的分数。

**[`constant_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-constant-score-query.html)**

A query which wraps another query, but executes it in filter context. All matching documents are given the same “constant” `_score`.

一个查询包装另一个查询，但在过滤器上下文中执行该查询。所有匹配的文档都被赋予相同的“常量” `_score`。

**[`dis_max` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-dis-max-query.html)**

A query which accepts multiple queries, and returns any documents which match any of the query clauses. While the `bool` query combines the scores from all matching queries, the `dis_max` query uses the score of the single best- matching query clause.

一个查询，它接受多个查询，并返回与任何查询子句匹配的任何文档。当`bool`查询合并所有匹配查询的分数时，`dis_max`查询将使用单个最佳匹配查询子句的分数。

**[`function_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html)**

Modify the scores returned by the main query with functions to take into account factors like popularity, recency, distance, or custom algorithms implemented with scripting.

使用函数修改主查询返回的分数，以考虑诸如受欢迎程度，新近度，距离或使用脚本实现的自定义算法之类的因素。

## Boolean query

A query that matches documents matching boolean combinations of other queries. The bool query maps to Lucene `BooleanQuery`. It is built using one or more boolean clauses, each clause with a typed occurrence. The occurrence types are:

与文档匹配的查询，这些文档与其他查询的布尔组合匹配。布尔查询映射到Lucene `BooleanQuery`。它是使用一个或多个布尔子句构建的，每个子句都具有类型的出现。发生类型为

| Occur      | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `must`     | The clause (query) must appear in matching documents and will contribute to the score.<br />子句（查询）必须出现在匹配的文档中，并将有助于得分。 |
| `filter`   | The clause (query) must appear in matching documents. However unlike `must` the score of the query will be ignored. Filter clauses are executed in [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html), meaning that scoring is ignored and clauses are considered for caching.<br />子句（查询）必须出现在匹配的文档中。但是不像 `must`查询的分数将被忽略。Filter子句在[filter上下文](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html)中执行，这意味着计分被忽略，并且子句被考虑用于缓存。 |
| `should`   | The clause (query) should appear in the matching document.<br />子句（查询）应出现在匹配的文档中。 |
| `must_not` | The clause (query) must not appear in the matching documents. Clauses are executed in [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html) meaning that scoring is ignored and clauses are considered for caching. Because scoring is ignored, a score of `0` for all documents is returned.<br />子句（查询）不得出现在匹配的文档中。子句在[过滤器上下文](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html)中执行，这意味着计分被忽略，并且子句被视为用于缓存。由于计分被忽略，因此`0`为返回所有文档的分数。 |

The `bool` query takes a *more-matches-is-better* approach, so the score from each matching `must` or `should` clause will be added together to provide the final `_score` for each document.

该`bool`查询采用的*是“更好匹配”的*方法，因此每个匹配项`must`或`should`子句的得分将加在一起，以提供`_score`每个文档的最终结果。

```json
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user.id" : "kimchy" }
      },
      "filter": {
        "term" : { "tags" : "production" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tags" : "env1" } },
        { "term" : { "tags" : "deployed" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

#### Using `minimum_should_match`

You can use the `minimum_should_match` parameter to specify the number or percentage of `should` clauses returned documents *must* match.

您可以使用`minimum_should_match`参数指定`should`返回的文档*必须*匹配的子句的数量或百分比。

If the `bool` query includes at least one `should` clause and no `must` or `filter` clauses, the default value is `1`. Otherwise, the default value is `0`.

如果`bool`查询包含至少一个`should`子句，而没有`must`或 `filter`子句，则默认值为`1`。否则，默认值为`0`。

For other valid values, see the [`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html).

有关其他有效值，请参见 [`minimum_should_match`参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html)。

#### Scoring with `bool.filter`

Queries specified under the `filter` element have no effect on scoring — scores are returned as `0`. Scores are only affected by the query that has been specified. For instance, all three of the following queries return all documents where the `status` field contains the term `active`.

在`filter`元素下指定的查询对得分没有影响-得分以`0`返回。分数仅受指定查询的影响。例如，以下所有三个查询返回该`status`字段包含术语的所有文档`active`。

This first query assigns a score of `0` to all documents, as no scoring query has been specified:

第一个查询指定所有文档得分为`0`，作为没有指定任何评分查询的分数：

```json
GET _search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

This `bool` query has a `match_all` query, which assigns a score of `1.0` to all documents.

该`bool`查询具有一个`match_all`查询，该查询`1.0`为所有文档分配分数。

```json
GET _search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

This `constant_score` query behaves in exactly the same way as the second example above. The `constant_score` query assigns a score of `1.0` to all documents matched by the filter.

`constant_score`查询的行为与上面的第二个示例完全相同。该`constant_score`查询为`1.0`过滤器匹配的所有文档分配分数。

```json
GET _search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

#### Named queries

Each query accepts a `_name` in its top level definition. You can use named queries to track which queries matched returned documents. If named queries are used, the response includes a `matched_queries` property for each hit.

每个查询都在其顶层定义中接受一个`_name`。您可以使用命名查询来跟踪哪些查询与返回的文档匹配。如果使用命名查询，则响应的每个匹配文档包括`matched_queries`属性。

```json
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "name.first": { "query": "shay", "_name": "first" } } },
        { "match": { "name.last": { "query": "banon", "_name": "last" } } }
      ],
      "filter": {
        "terms": {
          "name.last": [ "banon", "kimchy" ],
          "_name": "test"
        }
      }
    }
  }
}
```

##  Boosting query

Returns documents matching a `positive` query while reducing the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of documents that also match a `negative` query.

返回与`positive`查询匹配的文档，同时降低也与查询匹配的文档 的 [相关性得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)`negative`。

You can use the `boosting` query to demote certain documents without excluding them from the search results.

您可以使用`boosting`查询来降级某些文档，而不必将它们从搜索结果中排除。

#### Example request

```json
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "text": "apple"
        }
      },
      "negative": {
        "term": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

#### Top-level parameters for `boosting`

**`positive`**

(Required, query object) Query you wish to run. Any returned documents must match this query.

（必需的查询对象）您要运行的查询。返回的所有文档都必须与此查询匹配。

**`negative`**

(Required, query object) Query used to decrease the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of matching documents.

（必需的查询对象）用于降低匹配文档的[相关性得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)的查询。

If a returned document matches the `positive` query and this query, the `boosting` query calculates the final [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) for the document as follows:

如果返回的文档与`positive`查询和此查询匹配，则该 `boosting`查询将如下计算该文档的最终[相关性得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)：

1. Take the original relevance score from the `positive` query.

   从`positive`查询中获取原始的相关性分数。

2. Multiply the score by the `negative_boost` value.

   将分数乘以`negative_boost`值。

**`negative_boost`**

(Required, float) Floating point number between `0` and `1.0` used to decrease the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of documents matching the `negative` query.

（必需，浮点数）介于之间的浮点数`0`，`1.0`用于降低[与](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)`negative`查询匹配的文档 的[相关性得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)。

##  Constant score query

Wraps a [filter query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) and returns every matching document with a [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) equal to the `boost` parameter value.

包装[过滤器查询，](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)并返回[相关得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)等于`boost` 参数值的每个匹配文档。

```json
GET /_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": { "user.id": "kimchy" }
      },
      "boost": 1.2
    }
  }
}
```

###### Top-level parameters for `constant_score`

**`filter`**

(Required, query object) [Filter query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) you wish to run. Any returned documents must match this query.

（必需的查询对象）[过滤](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)要运行的[查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)。返回的所有文档都必须与此查询匹配。

Filter queries do not calculate [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores). To speed up performance, Elasticsearch automatically caches frequently used filter queries.

过滤查询不计算[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)。为了提高性能，Elasticsearch自动缓存经常使用的过滤器查询。

**`boost`**

(Optional, float) Floating point number used as the constant [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) for every document matching the `filter` query. Defaults to `1.0`.

（可选，float）浮点数，用作与查询匹配的每个文档 的恒定 相关性得分filter。默认为1.0。

##  Disjunction max query

Returns documents matching one or more wrapped queries, called query clauses or clauses.

返回与一个或多个包装查询（称为查询子句或子句）匹配的文档。

If a returned document matches multiple query clauses, the `dis_max` query assigns the document the highest relevance score from any matching clause, plus a tie breaking increment for any additional matching subqueries.

如果返回的文档与多个查询子句匹配，则`dis_max`查询会为该文档分配来自任何匹配子句的最高相关性得分，再加上任何其他匹配子查询的决定性增量。

#### Example request

```json
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "term": { "title": "Quick pets" } },
        { "term": { "body": "Quick pets" } }
      ],
      "tie_breaker": 0.7
    }
  }
}
```

####  Top-level parameters for `dis_max`

**`queries`**

(Required, array of query objects) Contains one or more query clauses. Returned documents **must match one or more** of these queries. If a document matches multiple queries, Elasticsearch uses the highest [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html).

（必需的查询对象数组）包含一个或多个查询子句。返回的文档**必须与**这些查询**中的一个或多个匹配**。如果一个文档匹配多个查询，Elasticsearch将使用最高的[相关性得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html)。

**`tie_breaker`**

(Optional, float) Floating point number between `0` and `1.0` used to increase the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of documents matching multiple query clauses. Defaults to `0.0`.

（可选，float）介于之间的浮点数`0`，`1.0`用于增加与多个查询子句匹配的文档的[相关性得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)。默认为`0.0`。

You can use the `tie_breaker` value to assign higher relevance scores to documents that contain the same term in multiple fields than documents that contain this term in only the best of those multiple fields, without confusing this with the better case of two different terms in the multiple fields.

您可以使用该`tie_breaker`值为多个字段中包含相同术语的文档分配比仅在多个字段中最好的包含该术语的文档更高的相关性分数，而不必将其与多个字段中两个不同术语的更好情况相混淆。

If a document matches multiple clauses, the `dis_max` query calculates the relevance score for the document as follows:

如果一个文档与多个子句匹配，`dis_max`查询将计算该文档的相关性得分，如下所示：

1. Take the relevance score from a matching clause with the highest score.

   从具有最高分数的匹配子句中获取相关性分数。

2. Multiply the score from any other matching clauses by the `tie_breaker` value.

   将任何其他匹配子句的分数乘以该`tie_breaker`值。

3. Add the highest score to the multiplied scores.

   将最高分数加到相乘的分数上。

If the `tie_breaker` value is greater than `0.0`, all matching clauses count, but the clause with the highest score counts most.

如果该`tie_breaker`值大于`0.0`，则所有匹配的子句都计数，但是得分最高的子句计数最多。

##  Function score query

The `function_score` allows you to modify the score of documents that are retrieved by a query. This can be useful if, for example, a score function is computationally expensive and it is sufficient to compute the score on a filtered set of documents.

 `function_score` 允许你修改查询返回的文档得分。例如，如果一个评分函数计算的代价是昂贵的并且足以在过滤后的一组文档上计算分数，则此功能将非常有用。

To use `function_score`, the user has to define a query and one or more functions, that compute a new score for each document returned by the query.

要使用 `function_score`，用户需要定义一个查询和一个或多个函数去为查询返回的每一个文档计算新的得分。

`function_score` can be used with only one function like this:

`function_score`只能与以下一种功能一起使用：

```json
GET /_search
{
  "query": {
    "function_score": {
      "query": { "match_all": {} },
      "boost": "5",
      "random_score": {},  (1)
      "boost_mode": "multiply"
    }
  }
}
```

1.  See [Function score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#score-functions) for a list of supported functions.

   有关支持的功能列表，请参见[功能得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#score-functions)。

Furthermore, several functions can be combined. In this case one can optionally choose to apply the function only if a document matches a given filtering query

此外，可以组合几个功能。在这种情况下，可以选择仅在文档与给定的过滤查询匹配时才应用该功能。

```json
GET /_search
{
  "query": {
    "function_score": {
      "query": { "match_all": {} },
      "boost": "5",  (1)
      "functions": [
        {
          "filter": { "match": { "test": "bar" } },
          "random_score": {},  (2)
          "weight": 23
        },
        {
          "filter": { "match": { "test": "cat" } },
          "weight": 42
        }
      ],
      "max_boost": 42,
      "score_mode": "max",
      "boost_mode": "multiply",
      "min_score": 42
    }
  }
}
```

1.  Boost for the whole query.

   提高整个查询的效率。

2. See [Function score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#score-functions) for a list of supported functions.

   有关支持的功能列表，请参见[功能得分](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#score-functions)。

> **NOTE:** The scores produced by the filtering query of each function do not matter.
>
> **注意：**每个函数的过滤查询所产生的分数无关紧要。

If no filter is given with a function this is equivalent to specifying `"match_all": {}`

如果没有给函数提供过滤器，则等同于指定 `"match_all": {}`

First, each document is scored by the defined functions. The parameter `score_mode` specifies how the computed scores are combined:

首先，通过定义的功能对每个文档进行评分。该参数 `score_mode`指定如何组合计算出的分数：

| `multiply` | scores are multiplied (default)<br />分数相乘（默认）        |
| ---------- | ------------------------------------------------------------ |
| `sum`      | scores are summed<br />分数相加                              |
| `avg`      | scores are averaged<br />分数取平均值                        |
| `first`    | the first function that has a matching filter is applied<br />具有匹配过滤器的第一个函数被应用 |
| `max`      | maximum score is used<br /> 使用最高分                       |
| `min`      | minimum score is used<br />使用最低分                        |

Because scores can be on different scales (for example, between 0 and 1 for decay functions but arbitrary for `field_value_factor`) and also because sometimes a different impact of functions on the score is desirable, the score of each function can be adjusted with a user defined `weight`. The `weight` can be defined per function in the `functions` array (example above) and is multiplied with the score computed by the respective function. If weight is given without any other function declaration, `weight` acts as a function that simply returns the `weight`.

因为分数不能被用作不同的尺度（例如，在0到1的衰减函数被应用在任意的 `field_value_factor`）并且有时希望函数对分数有不同的影响，每一个的得分可以用用户定义的 `weight`来调整。 `weight`可以被每一个函数定义在`functions`数组中（像上面的例子中一样）并在每个函数中以相乘计算得分。如果赋予了`weight`但没有其他函数定义，`weight`在函数中的行为仅仅是返回`weight`。

In case `score_mode` is set to `avg` the individual scores will be combined by a **weighted** average. For example, if two functions return score 1 and 2 and their respective weights are 3 and 4, then their scores will be combined as `(1*3+2*4)/(3+4)` and **not** `(1*3+2*4)/2`.

如果 `score_mode`被设置为 `avg` 那么各个分数将由加权平均合并。例如，如果两个函数返回分数为1和2并且他们各自的权重是3和4，他们的分数将会被组合为 `(1*3+2*4)/(3+4)`而不是`(1*3+2*4)/2`。

The new score can be restricted to not exceed a certain limit by setting the `max_boost` parameter. The default for `max_boost` is FLT_MAX.

新分数可以限制不超过某个确定的值，这个值可以通过`max_boost`参数来设置。 `max_boost`的默认值是FLT_MAX（最大浮点数）。

The newly computed score is combined with the score of the query. The parameter `boost_mode` defines how:

新计算的分数会和查询得分组合。该参数`boost_mode`如何定义：

| `multiply` | query score and function score is multiplied (default)<br />查询分数和功能分数相乘（默认） |
| ---------- | ------------------------------------------------------------ |
| `replace`  | only function score is used, the query score is ignored<br />仅使用功能得分，查询得分将被忽略 |
| `sum`      | query score and function score are added<br />查询分数和功能分数相加 |
| `avg`      | average<br />平均数                                          |
| `max`      | max of query score and function score<br />查询分数和功能分数的最大值 |
| `min`      | min of query score and function score<br />查询分数和功能分数的最小值 |

By default, modifying the score does not change which documents match. To exclude documents that do not meet a certain score threshold the `min_score` parameter can be set to the desired score threshold.

默认情况下，修改分数不会更改匹配的文档。要排除不符合特定分数阈值的文档，`min_score`可以将参数设置为所需分数阈值。

> **NOTE:** For `min_score` to work, **all** documents returned by the query need to be scored and then filtered out one by one.
>
> **注意：**为了`min_score`工作，需要对查询返回的**所有**文档进行打分，然后一一过滤掉。

The `function_score` query provides several types of score functions.

`function_score`查询提供了几种类型的得分函数。

- [`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#function-script-score)
- [`weight`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#function-weight)
- [`random_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#function-random)
- [`field_value_factor`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#function-field-value-factor)
- [decay functions](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#function-decay): `gauss`, `linear`, `exp`

#### Script score

The `script_score` function allows you to wrap another query and customize the scoring of it optionally with a computation derived from other numeric field values in the doc using a script expression. Here is a simple sample:

`script_score`函数允许您包装另一个查询，并可以选择使用脚本表达式从文档中其他数字字段值派生的计算来自定义另一个查询的得分。这是一个简单的示例：

```json
GET /_search
{
  "query": {
    "function_score": {
      "query": {
        "match": { "message": "elasticsearch" }
      },
      "script_score": {
        "script": {
          "source": "Math.log(2 + doc['my-int'].value)"
        }
      }
    }
  }
}
```

> **IMPORTANT:**  In Elasticsearch, all document scores are positive 32-bit floating point numbers.
>
>
> 在Elasticsearch中，所有文档分数均为正的32位浮点数。
>
> If the `script_score` function produces a score with greater precision, it is converted to the nearest 32-bit float.
>
> 如果`script_score`函数产生的精度更高，则将其转换为最接近的32位浮点数。
>
> Similarly, scores must be non-negative. Otherwise, Elasticsearch returns an error.
>
> 同样，分数必须为非负数。否则，Elasticsearch返回错误。

On top of the different scripting field values and expression, the `_score` script parameter can be used to retrieve the score based on the wrapped query.

除了不同的脚本字段值和表达式之外， `_score`还可以使用script参数基于包装的查询来检索分数。

Scripts compilation is cached for faster execution. If the script has parameters that it needs to take into account, it is preferable to reuse the same script, and provide parameters to it:

脚本编译被缓存以加快执行速度。如果脚本具有需要考虑的参数，则最好重用相同的脚本并为其提供参数：

```json
GET /_search
{
  "query": {
    "function_score": {
      "query": {
        "match": { "message": "elasticsearch" }
      },
      "script_score": {
        "script": {
          "params": {
            "a": 5,
            "b": 1.2
          },
          "source": "params.a / Math.pow(params.b, doc['my-int'].value)"
        }
      }
    }
  }
}
```

Note that unlike the `custom_score` query, the score of the query is multiplied with the result of the script scoring. If you wish to inhibit this, set `"boost_mode": "replace"`

请注意，与`custom_score`查询不同，查询的分数将与脚本评分的结果相乘。如果您想禁止这种情况，请设置`"boost_mode": "replace"`

####  Weight

The `weight` score allows you to multiply the score by the provided `weight`. This can sometimes be desired since boost value set on specific queries gets normalized, while for this score function it does not. The number value is of type float.

`weight`分数可以让你乘上提供的分数 `weight`。有时可能会希望这样做，因为在特定查询上设置的提升值会被标准化，而对于此得分函数则不会。该数字值为float类型。

```json
"weight" : number
```

####  Random

The `random_score` generates scores that are uniformly distributed from 0 up to but not including 1. By default, it uses the internal Lucene doc ids as a source of randomness, which is very efficient but unfortunately not reproducible since documents might be renumbered by merges.

`random_score`产生被均匀地从0分布直到但不包括1.默认情况下，它采用了内部Lucene的文档ID作为随机源，这是非常有效的，但不幸的是不可再现的，因为文档可能通过合并重新编号分数。

In case you want scores to be reproducible, it is possible to provide a `seed` and `field`. The final score will then be computed based on this seed, the minimum value of `field` for the considered document and a salt that is computed based on the index name and shard id so that documents that have the same value but are stored in different indexes get different scores. Note that documents that are within the same shard and have the same value for `field` will however get the same score, so it is usually desirable to use a field that has unique values for all documents. A good default choice might be to use the `_seq_no` field, whose only drawback is that scores will change if the document is updated since update operations also update the value of the `_seq_no` field.

如果您希望分数是可复制的，可以提供`seed` 和`field`。然后将基于该种子，所`field`考虑文档的最小值和根据索引名称和分片ID计算的盐计算最终分数，以便具有相同值但存储在不同索引中的文档得到不同的结果分数。请注意，位于相同分片内且具有相同值的文档`field` 将获得相同的分数，因此通常希望使用对所有文档都具有唯一值的字段。一个很好的默认选择是使用该 `_seq_no`字段，其唯一的缺点是，如果文档被更新，则分数会改变，因为更新操作也会更新该`_seq_no`字段的值。

> **NOTE:** It was possible to set a seed without setting a field, but this has been deprecated as this requires loading fielddata on the `_id` field which consumes a lot of memory.
>
> **注意：**可以在不设置字段的情况下设置种子，但是已弃用该方法，因为这需要在`_id`消耗大量内存的字段上加载字段数据。

```json
GET /_search
{
  "query": {
    "function_score": {
      "random_score": {
        "seed": 10,
        "field": "_seq_no"
      }
    }
  }
}
```

####  Field Value factor

The `field_value_factor` function allows you to use a field from a document to influence the score. It’s similar to using the `script_score` function, however, it avoids the overhead of scripting. If used on a multi-valued field, only the first value of the field is used in calculations.

`field_value_factor`功能使您可以使用文档中的字段来影响得分。它类似于使用该`script_score`函数，但是它避免了脚本编写的开销。如果用于多值字段，则在计算中仅使用该字段的第一个值。

As an example, imagine you have a document indexed with a numeric `my-int` field and wish to influence the score of a document with this field, an example doing so would look like:

例如，假设您有一个用数字`my-int` 字段编制索引的文档，并希望通过该字段影响文档的分数，则示例如下所示：

```json
GET /_search
{
  "query": {
    "function_score": {
      "field_value_factor": {
        "field": "my-int",
        "factor": 1.2,
        "modifier": "sqrt",
        "missing": 1
      }
    }
  }
}
```

Which will translate into the following formula for scoring:

它将转化为以下得分公式：

```
sqrt(1.2 * doc['my-int'].value)
```

There are a number of options for the `field_value_factor` function:

该功能有许多选项`field_value_factor`：

| `field`    | Field to be extracted from the document.<br />要从文档中提取的字段。 |
| ---------- | ------------------------------------------------------------ |
| `factor`   | Optional factor to multiply the field value with, defaults to `1`.<br />字段值乘以的可选因子，默认为`1`。 |
| `modifier` | Modifier to apply to the field value, can be one of: `none`, `log`, `log1p`, `log2p`, `ln`, `ln1p`, `ln2p`, `square`, `sqrt`, or `reciprocal`. Defaults to `none`.<br />修改适用于该字段的值，可以是一个：`none`，`log`， `log1p`，`log2p`，`ln`，`ln1p`，`ln2p`，`square`，`sqrt`，或`reciprocal`。默认为`none`。 |

| Modifier<br />**修饰符** | Meaning<br />**意义**                                        |
| ------------------------ | ------------------------------------------------------------ |
| `none`                   | Do not apply any multiplier to the field value<br />不要对字段值应用任何乘数 |
| `log`                    | Take the [common logarithm](https://en.wikipedia.org/wiki/Common_logarithm) of the field value. Because this function will return a negative value and cause an error if used on values between 0 and 1, it is recommended to use `log1p` instead.<br />取字段值的[常用对数](https://en.wikipedia.org/wiki/Common_logarithm)。由于此函数将返回负值，并且如果将其用于0到1之间的值，则会导致错误，因此建议改用它`log1p`。 |
| `log1p`                  | Add 1 to the field value and take the common logarithm<br />将1加到字段值并取公共对数 |
| `log2p`                  | Add 2 to the field value and take the common logarithm<br />在字段值上加2并取公共对数 |
| `ln`                     | Take the [natural logarithm](https://en.wikipedia.org/wiki/Natural_logarithm) of the field value. Because this function will return a negative value and cause an error if used on values between 0 and 1, it is recommended to use `ln1p` instead.<br />取字段值的[自然对数](https://en.wikipedia.org/wiki/Natural_logarithm)。由于此函数将返回负值，并且如果将其用于0到1之间的值，则会导致错误，因此建议改用它`ln1p`。 |
| `ln1p`                   | Add 1 to the field value and take the natural logarithm<br />将1加到栏位值并取自然对数 |
| `ln2p`                   | Add 2 to the field value and take the natural logarithm<br />将2加到栏位值并取自然对数 |
| `square`                 | Square the field value (multiply it by itself)<br />对字段值求平方（乘以它本身） |
| `sqrt`                   | Take the [square root](https://en.wikipedia.org/wiki/Square_root) of the field value<br />取字段值的[平方根](https://en.wikipedia.org/wiki/Square_root) |
| `reciprocal`             | [Reciprocate](https://en.wikipedia.org/wiki/Multiplicative_inverse) the field value, same as `1/x` where `x` is the field’s value<br />[响应](https://en.wikipedia.org/wiki/Multiplicative_inverse)字段值，同`1/x`那里`x`是该字段的值 |

**`missing`**

Value used if the document doesn’t have that field. The modifier and factor are still applied to it as though it were read from the document.

如果文档没有该字段，则使用该值。虽然从文档中读取了修饰符和因子，修饰符和因数仍然适用于它。

> **NOTE:** Scores produced by the `field_value_score` function must be non-negative, otherwise an error will be thrown. The `log` and `ln` modifiers will produce negative values if used on values between 0 and 1. Be sure to limit the values of the field with a range filter to avoid this, or use `log1p` and `ln1p`.
>
> **注意：**该`field_value_score`函数产生的分数必须为非负数，否则将引发错误。的`log`和`ln`如果在所使用的值0和1之间一定要限制的字段的值与一系列过滤器，以避免这种情况，或使用改性剂会产生负值`log1p`和 `ln1p`。

> **NOTE:** Keep in mind that taking the log() of 0, or the square root of a negative number is an illegal operation, and an exception will be thrown. Be sure to limit the values of the field with a range filter to avoid this, or use `log1p` and `ln1p`.
>
> **注意：**请记住，将log（）设为0或负数的平方根是非法操作，并且将引发异常。为避免这种情况，请务必使用范围过滤器限制该字段的值，或使用 `log1p`和`ln1p`。

####  Decay functions

Decay functions score a document with a function that decays depending on the distance of a numeric field value of the document from a user given origin. This is similar to a range query, but with smooth edges instead of boxes.
衰减函数对文档进行评分，该函数的衰减取决于文档的数字字段值距用户给定原点的距离。这类似于范围查询，但具有平滑的边缘而不是框。

To use distance scoring on a query that has numerical fields, the user has to define an `origin` and a `scale` for each field. The `origin` is needed to define the “central point” from which the distance is calculated, and the `scale` to define the rate of decay. The decay function is specified as

要在具有数字字段的查询上使用距离计分，用户必须为每个字段定义an`origin`和一个`scale`。定义从“中心点”该计算出的距离需要定义`origin` ，并且`scale`定义衰减率。衰减函数指定为

```json
"DECAY_FUNCTION": { （1）
    "FIELD_NAME": { （2）
          "origin": "11, 12",
          "scale": "2km",
          "offset": "0km",
          "decay": 0.33
    }
}
```

1. The `DECAY_FUNCTION` should be one of `linear`, `exp`, or `gauss`.

   `DECAY_FUNCTION`应该是一个`linear`，`exp`或`gauss`。

2. The specified field must be a numeric, date, or geo-point field.

    指定的字段必须是数字，日期或地理点字段。

In the above example, the field is a [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) and origin can be provided in geo format. `scale` and `offset` must be given with a unit in this case. If your field is a date field, you can set `scale` and `offset` as days, weeks, and so on. Example:

在上面的示例中，字段是一个 [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)，可以按地理格式提供原点。在这种情况下`scale`，`offset`必须提供一个单位。如果您的字段是日期字段，则可以将`scale`和设置`offset`为天，周等。例子：

```json
GET /_search
{
  "query": {
    "function_score": {
      "gauss": {
        "@timestamp": {
          "origin": "2013-09-17", （1）
          "scale": "10d",
          "offset": "5d",         （2）
          "decay": 0.5            （2）
        }
      }
    }
  }
}
```

1.  The date format of the origin depends on the [`format`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html) defined in your mapping. If you do not define the origin, the current time is used.

   原点的日期格式取决于[`format`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)映射中的定义。如果未定义原点，则使用当前时间。

2.  The `offset` and `decay` parameters are optional.

    在`offset`和`decay`参数都是可选的。

| `origin` | The point of origin used for calculating distance. Must be given as a number for numeric field, date for date fields and geo point for geo fields. Required for geo and numeric field. For date fields the default is `now`. Date math (for example `now-1h`) is supported for origin.<br />用于计算距离的原点。对于数字字段，必须指定为数字；对于日期字段，必须指定为日期；对于地理字段，必须指定为地理点。地理位置和数字字段必填。对于日期字段，默认值为`now`。`now-1h`原点支持日期数学（例如）。 |
| -------- | ------------------------------------------------------------ |
| `scale`  | Required for all types. Defines the distance from origin + offset at which the computed score will equal `decay` parameter. For geo fields: Can be defined as number+unit (1km, 12m,…). Default unit is meters. For date fields: Can to be defined as a number+unit ("1h", "10d",…). Default unit is milliseconds. For numeric field: Any number.<br />所有类型均必需。定义到原点的距离+偏移，计算出的分数将等于该距离`decay`。对于地理字段：可以定义为数字+单位（1km，12m，...）。默认单位是米。对于日期字段：可以定义为数字+单位（“ 1h”，“ 10d”，…。）。默认单位是毫秒。对于数字字段：任何数字。 |
| `offset` | If an `offset` is defined, the decay function will only compute the decay function for documents with a distance greater than the defined `offset`. The default is 0.<br />如果`offset`定义了，衰减函数将仅计算距离大于定义的文档的衰减函数 `offset`。默认值为0。 |
| `decay`  | The `decay` parameter defines how documents are scored at the distance given at `scale`. If no `decay` is defined, documents at the distance `scale` will be scored 0.5.<br />该`decay`参数定义了如何在处给定的距离处对文档进行评分`scale`。如果`decay`未定义，则远处的文档 `scale`得分为0.5。 |

In the first example, your documents might represents hotels and contain a geo location field. You want to compute a decay function depending on how far the hotel is from a given location. You might not immediately see what scale to choose for the gauss function, but you can say something like: "At a distance of 2km from the desired location, the score should be reduced to one third." The parameter "scale" will then be adjusted automatically to assure that the score function computes a score of 0.33 for hotels that are 2km away from the desired location.

在第一个示例中，您的文档可能代表酒店，并且包含地理位置字段。您要根据酒店距指定位置的距离来计算衰减函数。您可能不会立即看到为高斯功能选择哪种比例，但是您可以这样说：“在距所需位置2公里的距离处，分数应减少到三分之一。” 然后将自动调整参数“规模”，以确保得分功能为距离期望位置2公里的酒店计算出0.33的得分

In the second example, documents with a field value between 2013-09-12 and 2013-09-22 would get a weight of 1.0 and documents which are 15 days from that date a weight of 0.5.

在第二个示例中，字段值在2013-09-12和2013-09-22之间的文档的权重为1.0，从该日期起15天的文档的权重为0.5。

####  Supported decay functions

The `DECAY_FUNCTION` determines the shape of the decay:

所述`DECAY_FUNCTION`确定衰减的形状：

**`gauss`**

Normal decay, computed as:

正常衰减，计算如下：

![Gaussian](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/Gaussian.png)where ![sigma](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/sigma.png) is computed to assure that the score takes the value `decay` at distance `scale` from `origin`+-`offset`

其中![西格玛](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/sigma.png)被计算以确保得分取值`decay`在距离`scale`从`origin`+ -`offset`

![sigma calc](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/sigma_calc.png)

See [Normal decay, keyword `gauss`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#gauss-decay) for graphs demonstrating the curve generated by the `gauss` function.

请参见[正态衰减，关键字`gauss`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#gauss-decay)以显示表示该`gauss`函数生成的曲线的图形。

**`exp`**

Exponential decay, computed as:

指数衰减，计算如下：

![Exponential](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/Exponential.png)

where again the parameter ![lambda](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/lambda.png) is computed to assure that the score takes the value `decay` at distance `scale` from `origin`+-`offset`

其中![西格玛](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/sigma.png)被计算以确保得分取值`decay`在距离`scale`从`origin`+ -`offset`

![lambda calc](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/lambda_calc.png)

See [Exponential decay, keyword `exp`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#exp-decay) for graphs demonstrating the curve generated by the `exp` function.

请参阅[指数衰减，`exp`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#exp-decay)有关显示该`exp`函数生成的曲线的图形的[关键字](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#exp-decay)。

**`linear`**

Linear decay, computed as:

线性衰减，计算如下：

![Linear](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/Linear.png)where again the parameter `s` is computed to assure that the score takes the value `decay` at distance `scale` from `origin`+-`offset`

其中再次参数`s`被计算，以确保该得分取值`decay`在距离`scale`从`origin`+ -`offset`

![s calc](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/s_calc.png)

In contrast to the normal and exponential decay, this function actually sets the score to 0 if the field value exceeds twice the user given scale value.

与正常和指数衰减相反，如果字段值超过用户给定标度值的两倍，则此函数实际上将分数设置为0。

For single functions the three decay functions together with their parameters can be visualized like this (the field in this example called "age"):

对于单个函数，三个衰减函数及其参数可以像这样可视化（在此示例中，该字段称为“年龄”）：

![decay 2d](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/decay_2d.png)

####  Multi-values fields

If a field used for computing the decay contains multiple values, per default the value closest to the origin is chosen for determining the distance. This can be changed by setting `multi_value_mode`.

如果用于计算衰减的字段包含多个值，则默认情况下会选择最接近原点的值来确定距离。可以通过设置更改`multi_value_mode`。

| `min` | Distance is the minimum distance<br /> 距离是最小距离        |
| ----- | ------------------------------------------------------------ |
| `max` | Distance is the maximum distance<br />距离是最大距离         |
| `avg` | Distance is the average distance<br /> 距离是平均距离        |
| `sum` | Distance is the sum of all distances<br />距离是所有距离的总和 |

Example:

```json
   "DECAY_FUNCTION": {
        "FIELD_NAME": {
              "origin": ...,
              "scale": ...
        },
        "multi_value_mode": "avg"
    }
```

####  Detailed example

Suppose you are searching for a hotel in a certain town. Your budget is limited. Also, you would like the hotel to be close to the town center, so the farther the hotel is from the desired location the less likely you are to check in.

假设您正在寻找某个城镇的酒店。您的预算有限。另外，您希望酒店离市中心很近，因此酒店离所需位置越远，则办理登机手续的可能性就越小。

You would like the query results that match your criterion (for example, "hotel, Nancy, non-smoker") to be scored with respect to distance to the town center and also the price.

您希望根据距市中心的距离以及价格来对与您的条件相匹配的查询结果（例如“酒店，南希，不吸烟者”）进行评分。

Intuitively, you would like to define the town center as the origin and maybe you are willing to walk 2km to the town center from the hotel.
In this case your **origin** for the location field is the town center and the **scale** is ~2km.

直观地讲，您想将市中心定义为起点，也许您愿意从酒店步行2公里到市中心。
在这种情况下，您的**起源**的位置字段是市中心和**范围**是〜2公里。

If your budget is low, you would probably prefer something cheap above something expensive. For the price field, the **origin** would be 0 Euros and the **scale** depends on how much you are willing to pay, for example 20 Euros.

如果您的预算低，您可能更喜欢便宜的东西而不是昂贵的东西。对于价格字段，**原点**为0欧元，**范围**取决于您愿意支付的金额，例如20欧元。

In this example, the fields might be called "price" for the price of the hotel and "location" for the coordinates of this hotel.

在此示例中，字段可能被称为“价格”作为酒店的价格，而“位置”为该酒店的坐标。

The function for `price` in this case would be

`price`在这种情况下的功能是

```json
"gauss": { 
    "price": {
          "origin": "0",
          "scale": "20"
    }
}
```

1.  This decay function could also be `linear` or `exp`.

   此衰减函数也可以是`linear`或`exp`。

and for `location`:

和为`location`：

```json
"gauss": { （1）
    "location": {
          "origin": "11, 12",
          "scale": "2km"
    }
}
```

1. This decay function could also be `linear` or `exp`.

   此衰减函数也可以是`linear`或`exp`。

Suppose you want to multiply these two functions on the original score, the request would look like this:

假设您要将这两个函数乘以原始得分，则请求将如下所示：

```json
GET /_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "gauss": {
            "price": {
              "origin": "0",
              "scale": "20"
            }
          }
        },
        {
          "gauss": {
            "location": {
              "origin": "11, 12",
              "scale": "2km"
            }
          }
        }
      ],
      "query": {
        "match": {
          "properties": "balcony"
        }
      },
      "score_mode": "multiply"
    }
  }
}
```

Next, we show how the computed score looks like for each of the three possible decay functions.

接下来，我们显示三个可能衰减函数中的每一个的计算得分如何。

####  Normal decay, keyword `gauss`

When choosing `gauss` as the decay function in the above example, the contour and surface plot of the multiplier looks like this:

`gauss`在上面的示例中选择作为衰减函数时，乘数的轮廓和曲面图如下所示：

![cd0e18a6 e898 11e2 9b3c f0145078bd6f](https://f.cloud.github.com/assets/4320215/768157/cd0e18a6-e898-11e2-9b3c-f0145078bd6f.png)

![ec43c928 e898 11e2 8e0d f3c4519dbd89](https://f.cloud.github.com/assets/4320215/768160/ec43c928-e898-11e2-8e0d-f3c4519dbd89.png)

Suppose your original search results matches three hotels :

假设您的原始搜索结果与三家酒店匹配：

- "Backback Nap"
- "Drink n Drive"
- "BnB Bellevue".

"Drink n Drive" is pretty far from your defined location (nearly 2 km) and is not too cheap (about 13 Euros) so it gets a low factor a factor of 0.56. "BnB Bellevue" and "Backback Nap" are both pretty close to the defined location but "BnB Bellevue" is cheaper, so it gets a multiplier of 0.86 whereas "Backpack Nap" gets a value of 0.66.

“ Drink n Drive”距离您定义的位置很近（近2公里），而且价格也不便宜（约13欧元），因此它的系数低至0.56。“ BnB Bellevue”和“ Backback Nap”都非常接近定义的位置，但是“ BnB Bellevue”更便宜，因此它的乘数为0.86，而“ Backpack Nap”的值为0.66。

####  Exponential decay, keyword `exp`

When choosing `exp` as the decay function in the above example, the contour and surface plot of the multiplier looks like this:

`exp`在上面的示例中选择作为衰减函数时，乘数的轮廓和曲面图如下所示：

![082975c0 e899 11e2 86f7 174c3a729d64](https://f.cloud.github.com/assets/4320215/768161/082975c0-e899-11e2-86f7-174c3a729d64.png)

![0b606884 e899 11e2 907b aefc77eefef6](https://f.cloud.github.com/assets/4320215/768162/0b606884-e899-11e2-907b-aefc77eefef6.png)

####  Linear decay, keyword `linear`

When choosing `linear` as the decay function in the above example, the contour and surface plot of the multiplier looks like this:

`linear`在上面的示例中选择作为衰减函数时，乘数的轮廓和曲面图如下所示：

![1775b0ca e899 11e2 9f4a 776b406305c6](https://f.cloud.github.com/assets/4320215/768164/1775b0ca-e899-11e2-9f4a-776b406305c6.png)

![19d8b1aa e899 11e2 91bc 6b0553e8d722](https://f.cloud.github.com/assets/4320215/768165/19d8b1aa-e899-11e2-91bc-6b0553e8d722.png)

####  Supported fields for decay functions

Only numeric, date, and geo-point fields are supported.

仅支持数字，日期和地理位置字段。

####  What if a field is missing?

If the numeric field is missing in the document, the function will return 1.

如果文档中缺少数字字段，该函数将返回1。



# Full text queries

The full text queries enable you to search [analyzed text fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) such as the body of an email. The query string is processed using the same analyzer that was applied to the field during indexing.

全文查询使您可以搜索已[分析的文本字段，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)例如电子邮件的正文。使用在索引期间应用于字段的同一分析器来处理查询字符串。

The queries in this group are:

该组中的查询是：

- **[`intervals` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html)**

  A full text query that allows fine-grained control of the ordering and proximity of matching terms.

  全文查询，可以对匹配项的顺序和接近度进行细粒度控制。

- **[`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)**

  The standard query for performing full text queries, including fuzzy matching and phrase or proximity queries.

  用于执行全文查询的标准查询，包括模糊匹配和短语或接近查询。

- **[`match_bool_prefix` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-bool-prefix-query.html)**

  Creates a `bool` query that matches each term as a `term` query, except for the last term, which is matched as a `prefix` query

  创建一个`bool`每个字词的匹配作为查询`term`的查询，除了最后一项，它匹配的`prefix`查询

- **[`match_phrase` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html)**

  Like the `match` query but used for matching exact phrases or word proximity matches.

  与`match`查询类似，但用于匹配确切的短语或单词接近度匹配。

- **[`match_phrase_prefix` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase-prefix.html)**

  Like the `match_phrase` query, but does a wildcard search on the final word.

  与`match_phrase`查询类似，但是对最后一个单词进行通配符搜索。

- **[`multi_match` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html)**

  The multi-field version of the `match` query.

  `match`查询 的多字段版本。

- **[`combined_fields` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-combined-fields-query.html)**

  Matches over multiple fields as if they had been indexed into one combined field.

  匹配多个字段，就好像它们已被索引到一个组合字段中一样。

- **[`query_string` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html)**

  Supports the compact Lucene [query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-syntax), allowing you to specify AND|OR|NOT conditions and multi-field search within a single query string. For expert users only.

  支持紧凑的Lucene[查询字符串语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-syntax)，允许您在单个查询字符串中指定AND | OR | NOT条件和多字段搜索。仅适用于专家用户。

- **[`simple_query_string` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html)**

  A simpler, more robust version of the `query_string` syntax suitable for exposing directly to users.

  query_string适用于直接向用户公开 的语法的更简单，更可靠的版本。

## Intervals query

Returns documents based on the order and proximity of matching terms.

根据匹配项的顺序和接近程度返回文档。

The `intervals` query uses **matching rules**, constructed from a small set of definitions. These rules are then applied to terms from a specified `field`.

`intervals`查询使用**匹配规则**，这些**规则是**由一小组定义构成的。然后，将这些规则应用于指定的terms`field`。

The definitions produce sequences of minimal intervals that span terms in a body of text. These intervals can be further combined and filtered by parent sources.

这些定义产生的最小间隔序列跨越了文本主体中的各个terms。这些间隔可以由父源进一步组合和过滤。

### Example request

The following `intervals` search returns documents containing `my favorite food` without any gap, followed by `hot water` or `cold porridge` in the `my_text` field.

以下`intervals`搜索将返回`my favorite food`不包含任何间隙的文档，其后跟`hot water`或`cold porridge`在该 `my_text`字段中。

This search would match a `my_text` value of `my favorite food is cold porridge` but not `when it's cold my favorite food is porridge`.

此搜索将匹配`my_text`值`my favorite food is cold porridge`但不匹配`when it's cold my favorite food is porridge`。

```json
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "all_of" : {
          "ordered" : true,
          "intervals" : [
            {
              "match" : {
                "query" : "my favorite food",
                "max_gaps" : 0,
                "ordered" : true
              }
            },
            {
              "any_of" : {
                "intervals" : [
                  { "match" : { "query" : "hot water" } },
                  { "match" : { "query" : "cold porridge" } }
                ]
              }
            }
          ]
        }
      }
    }
  }
}
```

### Top-level parameters for `intervals`

**`<field>`**

(Required, rule object) Field you wish to search.

（必填，规则对象）您要搜索的字段。

The value of this parameter is a rule object used to match documents based on matching terms, order, and proximity.

此参数的值是一个规则对象，用于根据匹配的术语，顺序和接近度来匹配文档。

Valid rules include:

有效规则包括：

- [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#intervals-match)
- [`prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#intervals-prefix)
- [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#intervals-wildcard)
- [`fuzzy`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#intervals-fuzzy)
- [`all_of`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#intervals-all_of)
- [`any_of`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#intervals-any_of)

### `match` rule parameters

The `match` rule matches analyzed text.

该`match`规则匹配分析文本。

- **`query`**

  (Required, string) Text you wish to find in the provided `<field>`.

  （必需，字符串）您希望在提供的中找到的文本`<field>`。	

- **`max_gaps`**

  (Optional, integer) Maximum number of positions between the matching terms. Terms further apart than this are not considered matches. Defaults to `-1`.

  （可选，整数）匹配项之间的最大位置数。除此以外的术语均不视为匹配项。默认为 `-1`。

  If unspecified or set to `-1`, there is no width restriction on the match. If set to `0`, the terms must appear next to each other.

  如果未指定或设置为`-1`，则匹配项没有宽度限制。如果设置为`0`，则这些术语必须彼此相邻出现。

- **`ordered`**

  (Optional, Boolean) If `true`, matching terms must appear in their specified order. Defaults to `false`.

  （可选，布尔值）如果为`true`，则匹配词必须以其指定顺序出现。默认为 `false`。

- **`analyzer`**

  (Optional, string) [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to analyze terms in the `query`. Defaults to the top-level `<field>`'s analyzer.

  （可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于分析中的术语`query`。默认为顶级`<field>`的分析器。

- **`filter`**

  (Optional, [interval filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval_filter) rule object) An optional interval filter.

  （可选，[间隔过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval_filter)规则对象）可选间隔过滤器。

- **`use_field`**

  (Optional, string) If specified, then match intervals from this field rather than the top-level `<field>`. Terms are analyzed using the search analyzer from this field. This allows you to search across multiple fields as if they were all the same field; for example, you could index the same text into stemmed and unstemmed fields, and search for stemmed tokens near unstemmed ones.

  （可选，字符串）如果指定，则匹配该字段而不是top-level的间隔`<field>`。使用该字段中的搜索分析器分析术语。这样，您就可以跨多个字段进行搜索，就好像它们都是同一字段一样。例如，您可以将相同的文本编入词干和未词干字段中，并在未词干的字段附近搜索词干标记。

### `prefix` rule parameters

The `prefix` rule matches terms that start with a specified set of characters. This prefix can expand to match at most 128 terms. If the prefix matches more than 128 terms, Elasticsearch returns an error. You can use the [`index-prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html) option in the field mapping to avoid this limit.

`prefix`规则匹配以指定字符集开头的术语。该前缀可以扩展为最多匹配128个术语。如果前缀匹配的术语超过128个，Elasticsearch将返回错误。您可以[`index-prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html)在字段映射中使用该 选项来避免此限制。

- **`prefix`**

  (Required, string) Beginning characters of terms you wish to find in the top-level `<field>`.

  （必需，字符串）您希望在顶级中找到的术语的开始字符`<field>`。

- **`analyzer`**

  (Optional, string) [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to normalize the `prefix`. Defaults to the top-level `<field>`'s analyzer.

  （可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于规范化`prefix`。默认为顶级`<field>`的分析器。

- **`use_field`**

  (Optional, string) If specified, then match intervals from this field rather than the top-level `<field>`.

  （可选，字符串）如果指定，则匹配该字段而不是top-level的间隔`<field>`。

  The `prefix` is normalized using the search analyzer from this field, unless a separate `analyzer` is specified.

  `prefix`除非另行`analyzer`指定，否则使用此字段中的搜索分析器将标准化。

### `wildcard` rule parameters

The `wildcard` rule matches terms using a wildcard pattern. This pattern can expand to match at most 128 terms. If the pattern matches more than 128 terms, Elasticsearch returns an error.

该`wildcard`规则使用通配符模式匹配术语。此模式可以扩展以匹配最多128个术语。如果该模式匹配超过128个术语，则Elasticsearch返回错误。

**`pattern`**

(Required, string) Wildcard pattern used to find matching terms.

必需，字符串）通配符模式，用于查找匹配的字词。

This parameter supports two wildcard operators:

该参数支持两个通配符：

- `?`, which matches any single character

  `?`，它与任何单个字符匹配

- `*`, which can match zero or more characters, including an empty one

  `*`，可以匹配零个或多个字符，包括一个空字符

  > **WARNING:** Avoid beginning patterns with `*` or `?`. This can increase the iterations needed to find matching terms and slow search performance.
  >
  > **警告：**避免使用`*`或开头的模式`?`。这会增加查找匹配项所需的迭代次数，并降低搜索性能。

**`analyzer`**

(Optional, string) [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to normalize the `pattern`. Defaults to the top-level `<field>`'s analyzer.

（可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于规范化`pattern`。默认为顶级`<field>`的分析器。

**`use_field`**

(Optional, string) If specified, match intervals from this field rather than the top-level `<field>`.

（可选，字符串）如果指定，则匹配该字段而不是top-level的间隔`<field>`。

The `pattern` is normalized using the search analyzer from this field, unless `analyzer` is specified separately.

除非`analyzer`另行指定`pattern`，否则使用此字段中的搜索分析器将标准化 。

### `fuzzy` rule parameters

The `fuzzy` rule matches terms that are similar to the provided term, within an edit distance defined by [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness). If the fuzzy expansion matches more than 128 terms, Elasticsearch returns an error.

`fuzzy`规则在[Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness)定义的编辑距离内匹配与提供的术语相似的术语。如果模糊扩展匹配超过128个项，Elasticsearch将返回错误。

**`term`**

(Required, string) The term to match

（必需，字符串）要匹配的术语

**`prefix_length`**

(Optional, integer) Number of beginning characters left unchanged when creating expansions. Defaults to `0`.

（可选，整数）创建扩展时保留不变的开始字符数。默认为`0`。

**`transpositions`**

(Optional, Boolean) Indicates whether edits include transpositions of two adjacent characters (ab → ba). Defaults to `true`.

（可选，布尔值）指示编辑是否包括两个相邻字符的转置（ab→ba）。默认为`true`。

**`fuzziness`**

(Optional, string) Maximum edit distance allowed for matching. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness) for valid values and more information. Defaults to `auto`.

（可选，字符串）匹配允许的最大编辑距离。有关 有效值和更多信息，请参见[模糊性](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness)。默认为`auto`。

**`analyzer`**

(Optional, string) [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to normalize the `term`. Defaults to the top-level `<field>` 's analyzer.

（可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于规范化`term`。默认为顶级`<field>`的分析器。

**`use_field`**

(Optional, string) If specified, match intervals from this field rather than the top-level `<field>`.

（可选，字符串）如果指定，则匹配该字段而不是top-level的间隔`<field>`。

The `term` is normalized using the search analyzer from this field, unless `analyzer` is specified separately.

`term`除非`analyzer`另行指定，否则使用此字段中的搜索分析器将标准化 。

### `all_of` rule parameters

The `all_of` rule returns matches that span a combination of other rules.

该`all_of`规则返回跨越其他规则组合的匹配项。

**`intervals`**

(Required, array of rule objects) An array of rules to combine. All rules must produce a match in a document for the overall source to match.

（必需，规则对象数组）要组合的规则数组。所有规则都必须在文档中产生一个匹配项，以使整个源匹配。

**`max_gaps`**

(Optional, integer) Maximum number of positions between the matching terms. Intervals produced by the rules further apart than this are not considered matches. Defaults to `-1`.

（可选，整数）匹配项之间的最大位置数。由规则产生的间隔比此间隔更远的间隔不视为匹配项。默认为`-1`。

If unspecified or set to `-1`, there is no width restriction on the match. If set to `0`, the terms must appear next to each other.

如果未指定或设置为`-1`，则匹配项没有宽度限制。如果设置为`0`，则这些术语必须彼此相邻出现。

**`ordered`**

(Optional, Boolean) If `true`, intervals produced by the rules should appear in the order in which they are specified. Defaults to `false`.

（可选，布尔值）如果为`true`，则规则产生的间隔应按指定的顺序出现。默认为`false`。

**`filter`**

(Optional, [interval filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval_filter) rule object) Rule used to filter returned intervals.

（可选，[时间间隔过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval_filter)规则对象）用于过滤返回的时间间隔的规则。

### `any_of` rule parameters

The `any_of` rule returns intervals produced by any of its sub-rules.

该`any_of`规则返回其任何子规则产生的间隔。

**`intervals`**

(Required, array of rule objects) An array of rules to match.

（必需，规则对象数组）要匹配的规则数组。

**`filter`**

(Optional, [interval filter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval_filter) rule object) Rule used to filter returned intervals.

（可选，[时间间隔过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval_filter)规则对象）用于过滤返回的时间间隔的规则。

### `filter` rule parameters

The `filter` rule returns intervals based on a query. See [Filter example](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval-filter-rule-ex) for an example.

该`filter`基于查询规则的回报区间。有关[示例](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval-filter-rule-ex)，请参 [见过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval-filter-rule-ex)示例。

**`after`**

(Optional, query object) Query used to return intervals that follow an interval from the `filter` rule.

（可选，查询对象）查询用于返回遵循`filter`规则间隔的间隔。

**`before`**

(Optional, query object) Query used to return intervals that occur before an interval from the `filter` rule.

（可选，查询对象）查询用于返回在`filter`规则间隔之前出现的间隔。

**`contained_by`**

(Optional, query object) Query used to return intervals contained by an interval from the `filter` rule.

（可选，查询对象）用于从`filter`规则返回间隔所包含的间隔的查询。

**`containing`**

(Optional, query object) Query used to return intervals that contain an interval from the `filter` rule.

（可选，查询对象）用于返回包含来自`filter`规则的间隔的间隔的查询。

**`not_contained_by`**

(Optional, query object) Query used to return intervals that are **not** contained by an interval from the `filter` rule.

（可选，查询对象）查询用于返回规则的间隔中**不** 包含的间隔`filter`。

**`not_containing`**

(Optional, query object) Query used to return intervals that do **not** contain an interval from the `filter` rule.

（可选，查询对象）用于返回**不**包含来自`filter`规则的间隔的间隔的查询。

**`not_overlapping`**

(Optional, query object) Query used to return intervals that do **not** overlap with an interval from the `filter` rule.

（可选，查询对象）查询用于返回与规则中的间隔**不**重叠的间隔`filter`。

**`overlapping`**

(Optional, query object) Query used to return intervals that overlap with an interval from the `filter` rule.

（可选，查询对象）查询用于返回与`filter`规则中的间隔重叠的间隔。

**`script`**

(Optional, [script object](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-using.html)) Script used to return matching documents. This script must return a boolean value, `true` or `false`. See [Script filters](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval-script-filter) for an example.

（可选，[脚本对象](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-using.html)）用于返回匹配文档的脚本。该脚本必须返回布尔值`true`或`false`。有关示例，请参见[脚本过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-intervals-query.html#interval-script-filter)。

### Notes

#### Filter example

The following search includes a `filter` rule. It returns documents that have the words `hot` and `porridge` within 10 positions of each other, without the word `salty` in between:

以下搜索包括一个`filter`规则。它返回包含以下单词的文档，`hot`并且`porridge`彼此`salty`之间的位置不超过10个位置：

```json
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "hot porridge",
          "max_gaps" : 10,
          "filter" : {
            "not_containing" : {
              "match" : {
                "query" : "salty"
              }
            }
          }
        }
      }
    }
  }
}
```

####  Script filters

You can use a script to filter intervals based on their start position, end position, and internal gap count. The following `filter` script uses the `interval` variable with the `start`, `end`, and `gaps` methods:

您可以使用脚本根据间隔的开始位置，结束位置和内部间隔计数来过滤间隔。下面的`filter`脚本使用 `interval`与变量`start`，`end`和`gaps`方法：

```json
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "hot porridge",
          "filter" : {
            "script" : {
              "source" : "interval.start > 10 && interval.end < 20 && interval.gaps == 0"
            }
          }
        }
      }
    }
  }
}
```

####  Minimization

The intervals query always minimizes intervals, to ensure that queries can run in linear time. This can sometimes cause surprising results, particularly when using `max_gaps` restrictions or filters. For example, take the following query, searching for `salty` contained within the phrase `hot porridge`:

间隔查询始终将间隔最小化，以确保查询可以线性运行。有时这可能会导致令人惊讶的结果，尤其是在使用`max_gaps`限制或过滤器时。例如，采用以下查询，搜索`salty`短语中包含的内容`hot porridge`：

```json
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "match" : {
          "query" : "salty",
          "filter" : {
            "contained_by" : {
              "match" : {
                "query" : "hot porridge"
              }
            }
          }
        }
      }
    }
  }
}
```

This query does **not** match a document containing the phrase `hot porridge is salty porridge`, because the intervals returned by the match query for `hot porridge` only cover the initial two terms in this document, and these do not overlap the intervals covering `salty`.

此查询与包含该短语的文档**不**匹配`hot porridge is salty porridge`，因为match查询返回的间隔`hot porridge`仅覆盖了该文档中的前两个术语，并且与覆盖的间隔不重叠`salty`。

Another restriction to be aware of is the case of `any_of` rules that contain sub-rules which overlap. In particular, if one of the rules is a strict prefix of the other, then the longer rule can never match, which can cause surprises when used in combination with `max_gaps`. Consider the following query, searching for `the` immediately followed by `big` or `big bad`, immediately followed by `wolf`:

要注意的另一个限制是`any_of`规则包含子规则重叠的情况。尤其是，如果其中一个规则是另一个规则的严格前缀，则更长的规则将永远无法匹配，当与结合使用时，可能会引起意外`max_gaps`。考虑以下查询，搜索`the`紧随其后的是`big`还是`big bad`，紧随其后的是`wolf`：

```json
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "all_of" : {
          "intervals" : [
            { "match" : { "query" : "the" } },
            { "any_of" : {
                "intervals" : [
                    { "match" : { "query" : "big" } },
                    { "match" : { "query" : "big bad" } }
                ] } },
            { "match" : { "query" : "wolf" } }
          ],
          "max_gaps" : 0,
          "ordered" : true
        }
      }
    }
  }
}
```

Counter-intuitively, this query does **not** match the document `the big bad wolf`, because the `any_of` rule in the middle only produces intervals for `big` - intervals for `big bad` being longer than those for `big`, while starting at the same position, and so being minimized away. In these cases, it’s better to rewrite the query so that all of the options are explicitly laid out at the top level:

与直觉相反，此查询与文档**不**匹配`the big bad wolf`，因为在同一位置开始时，`any_of`中间的规则仅生成`big`-的间隔`big bad`比-的间隔长`big`，因此在同一位置开始，因此被最小化。在这些情况下，最好重写查询，以便所有选项都明确地显示在顶层：

```json
POST _search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "any_of" : {
          "intervals" : [
            { "match" : {
                "query" : "the big bad wolf",
                "ordered" : true,
                "max_gaps" : 0 } },
            { "match" : {
                "query" : "the big wolf",
                "ordered" : true,
                "max_gaps" : 0 } }
           ]
        }
      }
    }
  }
}
```

## Match query

Returns documents that match a provided text, number, date or boolean value. The provided text is analyzed before matching.

返回与提供的文本，数字，日期或布尔值匹配的文档。匹配之前对提供的文本进行分析。

The `match` query is the standard query for performing a full-text search, including options for fuzzy matching.

该`match`查询是用于执行全文搜索的标准查询，其中包括模糊匹配的选项。

### Example request

```json
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test"
      }
    }
  }
}
```

###  Top-level parameters for `match`

**`<field>`**

(Required, object) Field you wish to search.

（必填，对象）您要搜索的字段。

###  Parameters for `<field>`

**`query`**

(Required) Text, number, boolean value or date you wish to find in the provided `<field>`.

（必填）您希望在提供的中找到的文本，数字，布尔值或日期 `<field>`。

The `match` query [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) any provided text before performing a search. This means the `match` query can search [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) fields for analyzed tokens rather than an exact term.

该`match`查询[分析](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)执行搜索之前的任何提供的文本。这意味着`match`查询可以在[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html)字段中搜索分析的标记，而不是确切的词。

**`analyzer`**

(Optional, string) [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to convert the text in the `query` value into tokens. Defaults to the [index-time analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/specify-analyzer.html#specify-index-time-analyzer) mapped for the `<field>`. If no analyzer is mapped, the index’s default analyzer is used.

（可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于将`query` 值中的文本转换为标记。默认为为映射的[索引时间分析器](https://www.elastic.co/guide/en/elasticsearch/reference/master/specify-analyzer.html#specify-index-time-analyzer)`<field>`。如果未映射任何分析器，则使用索引的默认分析器。

**`auto_generate_synonyms_phrase_query`**

(Optional, Boolean) If `true`, [match phrase](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html) queries are automatically created for multi-term synonyms. Defaults to `true`.

（可选，布尔值）如果为`true`， 则会自动为多个术语同义词创建[匹配词组](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html)查询。默认为`true`。

See [Use synonyms with match query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-synonyms) for an example.

有关示例，请参阅[将同义词与匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-synonyms)一起使用。

**`fuzziness`**

(Optional, string) Maximum edit distance allowed for matching. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness) for valid values and more information. See [Fuzziness in the match query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-fuzziness) for an example.

（可选，字符串）匹配允许的最大编辑距离。有关 有效值和更多信息，请参见[模糊性](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness)。有关 示例，请参见[匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-fuzziness)中的[模糊性](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-fuzziness)。

**`max_expansions`**

(Optional, integer) Maximum number of terms to which the query will expand. Defaults to `50`.

（可选，整数）查询将扩展到的最大术语数。默认为`50`。

**`prefix_length`**

(Optional, integer) Number of beginning characters left unchanged for fuzzy matching. Defaults to `0`.

（可选，整数）为模糊匹配保留的起始字符数。默认为`0`。

**`fuzzy_transpositions`**

(Optional, Boolean) If `true`, edits for fuzzy matching include transpositions of two adjacent characters (ab → ba). Defaults to `true`.

（可选，布尔值）如果`true`为，则模糊匹配的编辑内容包括两个相邻字符的变位（ab→ba）。默认为`true`。

**`fuzzy_rewrite`**

(Optional, string) Method used to rewrite the query. See the [`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html) for valid values and more information.

（可选，字符串）用于重写查询的方法。请参阅 [`rewrite`参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html)以获取有效值和更多信息。

If the `fuzziness` parameter is not `0`, the `match` query uses a `fuzzy_rewrite` method of `top_terms_blended_freqs_${max_expansions}` by default.

如果`fuzziness`参数不是`0`，则`match`查询默认使用的`fuzzy_rewrite` 方法`top_terms_blended_freqs_${max_expansions}`。

**`lenient`**

(Optional, Boolean) If `true`, format-based errors, such as providing a text `query` value for a [numeric](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) field, are ignored. Defaults to `false`.

（可选，布尔值）如果为`true`，则将忽略基于格式的错误，例如为[数字](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)字段提供文本 `query`值。默认为`false`。

**`operator`**

(Optional, string) Boolean logic used to interpret text in the `query` value. Valid values are:

（可选，字符串）布尔逻辑，用于解释`query`值中的文本。有效值为：

- **`OR` (Default)**

  For example, a `query` value of `capital of Hungary` is interpreted as `capital OR of OR Hungary`.

  例如，`query`值`capital of Hungary`解释为`capital OR of OR Hungary`。

- **`AND`**

  For example, a `query` value of `capital of Hungary` is interpreted as `capital AND of AND Hungary`.

  例如，`query`值`capital of Hungary`解释为`capital AND of AND Hungary`。

**`minimum_should_match`**

(Optional, string) Minimum number of clauses that must match for a document to be returned. See the [`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html) for valid values and more information.

（可选，字符串）要返回的文档必须匹配的最小子句数。请参阅[`minimum_should_match` 参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html)以获取有效值和更多信息。

**`zero_terms_query`**

(Optional, string) Indicates whether no documents are returned if the `analyzer` removes all tokens, such as when using a `stop` filter. Valid values are:

（可选，字符串）指示如果`analyzer` 删除所有标记（例如使用`stop`过滤器时），是否不返回任何文档。有效值为：

- **`none` (Default)**

  No documents are returned if the `analyzer` removes all tokens.

  如果`analyzer`删除所有标记，则不会返回任何文档。

- **`all`**

  Returns all documents, similar to a [`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-all-query.html) query.

  返回所有文档，类似于[`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-all-query.html) 查询。

See [Zero terms query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-zero) for an example.

有关示例，请参见[零项查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-zero)。

###  Notes

#### Short request example

You can simplify the match query syntax by combining the `<field>` and `query` parameters. For example:

您可以通过组合`<field>`和`query` 参数来简化匹配查询语法。例如：

```json
GET /_search
{
  "query": {
    "match": {
      "message": "this is a test"
    }
  }
}
```

####  How the match query works

The `match` query is of type `boolean`. It means that the text provided is analyzed and the analysis process constructs a boolean query from the provided text. The `operator` parameter can be set to `or` or `and` to control the boolean clauses (defaults to `or`). The minimum number of optional `should` clauses to match can be set using the [`minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html) parameter.

该`match`查询类型`boolean`。这意味着将对提供的文本进行分析，并且分析过程将从提供的文本中构造一个布尔查询。所述`operator`参数可以被设置为`or`或者`and` 以控制布尔条款（默认为`or`）。`should`可以使用[`minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html) 参数设置要匹配的可选子句 的最小数量。

Here is an example with the `operator` parameter:

这是带有`operator`参数的示例：

```json
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test",
        "operator": "and"
      }
    }
  }
}
```

The `analyzer` can be set to control which analyzer will perform the analysis process on the text. It defaults to the field explicit mapping definition, or the default search analyzer.

所述`analyzer`可以被设置为控制哪个分析器将在文本上执行的分析过程。它默认为字段显式映射定义或默认的搜索分析器。

The `lenient` parameter can be set to `true` to ignore exceptions caused by data-type mismatches, such as trying to query a numeric field with a text query string. Defaults to `false`.

该`lenient`参数可以设置为`true`忽略造成的数据类型不匹配的异常，如想查询的数字字段与文本查询字符串。默认为`false`。

####  Fuzziness in the match query

`fuzziness` allows *fuzzy matching* based on the type of field being queried. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness) for allowed settings.

`fuzziness`允许根据要查询的字段类型进行*模糊匹配*。有关允许的设置，请参见[模糊性](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness)。

The `prefix_length` and `max_expansions` can be set in this case to control the fuzzy process. If the fuzzy option is set the query will use `top_terms_blended_freqs_${max_expansions}` as its [rewrite method](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html) the `fuzzy_rewrite` parameter allows to control how the query will get rewritten.

`prefix_length`和 `max_expansions`可以在这种情况下，以控制模糊处理来设定。如果设置了模糊选项，则查询将`top_terms_blended_freqs_${max_expansions}` 用作其[重写方法，](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html)该`fuzzy_rewrite`参数允许控制如何重写查询。

Fuzzy transpositions (`ab` → `ba`) are allowed by default but can be disabled by setting `fuzzy_transpositions` to `false`.

默认情况下允许使用模糊转置（`ab`→ `ba`），但可以通过将其设置`fuzzy_transpositions`为来禁用模糊转置`false`。

> **NOTE:** Fuzzy matching is not applied to terms with synonyms or in cases where the analysis process produces multiple tokens at the same position. Under the hood these terms are expanded to a special synonym query that blends term frequencies, which does not support fuzzy expansion.
>
> **注意：** 模糊匹配不适用于具有同义词的术语，或者在分析过程中在同一位置产生多个标记的情况。这些术语在引擎盖下被扩展为混合了术语频率的特殊同义词查询，这不支持模糊扩展。

```json
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a testt",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

#### Zero terms query

If the analyzer used removes all tokens in a query like a `stop` filter does, the default behavior is to match no documents at all. In order to change that the `zero_terms_query` option can be used, which accepts `none` (default) and `all` which corresponds to a `match_all` query.

如果使用的分析器像`stop`过滤器一样删除查询中的所有标记，则默认行为是根本不匹配任何文档。为了更改`zero_terms_query`可以使用该选项，该选项接受 `none`（默认值）并且`all`与`match_all`查询相对应。

```json
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "to be or not to be",
        "operator": "and",
        "zero_terms_query": "all"
      }
    }
  }
}
```

####  Synonyms

The `match` query supports multi-terms synonym expansion with the [synonym_graph](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-graph-tokenfilter.html) token filter. When this filter is used, the parser creates a phrase query for each multi-terms synonyms. For example, the following synonym: `"ny, new york"` would produce:

该`match`查询通过[synonym_graph](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-graph-tokenfilter.html)标记过滤器支持多词同义词扩展。使用此过滤器时，解析器为每个多词同义词创建短语查询。例如，以下同义词：`"ny, new york"`将产生：

`(ny OR ("new york"))`

It is also possible to match multi terms synonyms with conjunctions instead:

也可以用连接词来匹配多词同义词：

```json
GET /_search
{
   "query": {
       "match" : {
           "message": {
               "query" : "ny city",
               "auto_generate_synonyms_phrase_query" : false
           }
       }
   }
}
```

The example above creates a boolean query:

上面的示例创建一个布尔查询：

`(ny OR (new AND york)) city`

that matches documents with the term `ny` or the conjunction `new AND york`. By default the parameter `auto_generate_synonyms_phrase_query` is set to `true`.

与带有术语`ny`或连词的文档相匹配`new AND york`。默认情况下，参数`auto_generate_synonyms_phrase_query`设置为`true`。

## Match boolean prefix query

A `match_bool_prefix` query analyzes its input and constructs a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) from the terms. Each term except the last is used in a `term` query. The last term is used in a `prefix` query. A `match_bool_prefix` query such as

一个`match_bool_prefix`查询分析其输入，并构造 [`bool`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)子句。`term`查询中使用除最后一个词以外的每个词。最后一项在`prefix`查询中使用。一个 `match_bool_prefix`查询如下

```json
GET /_search
{
  "query": {
    "match_bool_prefix" : {
      "message" : "quick brown f"
    }
  }
}
```

where analysis produces the terms `quick`, `brown`, and `f` is similar to the following `bool` query

分析产生术语 `quick`, `brown`, 和 `f` 就像下面的`bool`查询

```json
GET /_search
{
  "query": {
    "bool" : {
      "should": [
        { "term": { "message": "quick" }},
        { "term": { "message": "brown" }},
        { "prefix": { "message": "f"}}
      ]
    }
  }
}
```

An important difference between the `match_bool_prefix` query and [`match_phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase-prefix.html) is that the `match_phrase_prefix` query matches its terms as a phrase, but the `match_bool_prefix` query can match its terms in any position. The example `match_bool_prefix` query above could match a field containing `quick brown fox`, but it could also match `brown fox quick`. It could also match a field containing the term `quick`, the term `brown` and a term starting with `f`, appearing in any position.

`match_bool_prefix`查询与[`match_phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase-prefix.html)之间的一个重要区别是 `match_phrase_prefix`查询将其术语作为短语进行匹配，但是 `match_bool_prefix`查询可以在任何位置匹配其术语。`match_bool_prefix`上面的示例 查询可以匹配包含的字段 `quick brown fox`，但也可以匹配`brown fox quick`。它还可以匹配一个包含术语`quick`，术语`brown`和以开头的术语的字段，该字段`f`出现在任何位置。

###  Parameters

By default, `match_bool_prefix` queries' input text will be analyzed using the analyzer from the queried field’s mapping. A different search analyzer can be configured with the `analyzer` parameter

默认情况下，`match_bool_prefix`将使用分析器从查询字段的映射中分析查询的输入文本。可以使用`analyzer`参数配置其他搜索分析器

```json
GET /_search
{
  "query": {
    "match_bool_prefix": {
      "message": {
        "query": "quick brown f",
        "analyzer": "keyword"
      }
    }
  }
}
```

`match_bool_prefix` queries support the [`minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html) and `operator` parameters as described for the [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-boolean), applying the setting to the constructed `bool` query. The number of clauses in the constructed `bool` query will in most cases be the number of terms produced by analysis of the query text.

`match_bool_prefix`查询支持 [`minimum_should_match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html)和`operator` 如所描述的参数 [`match`的查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-boolean)，将所述设定到所构建的`bool`查询。`bool` 在大多数情况下，构造查询中的子句数将是通过分析查询文本产生的术语数。

The [`fuzziness`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-fuzziness), `prefix_length`, `max_expansions`, `fuzzy_transpositions`, and `fuzzy_rewrite` parameters can be applied to the `term` subqueries constructed for all terms but the final term. They do not have any effect on the prefix query constructed for the final term.

[`fuzziness`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-fuzziness)，`prefix_length`， `max_expansions`，`fuzzy_transpositions`，和`fuzzy_rewrite`参数可被应用到`term`所有术语，而是最终术语构建子查询。它们对为最终术语构造的前缀查询没有任何影响。

## Match phrase query

The `match_phrase` query analyzes the text and creates a `phrase` query out of the analyzed text. For example:

`match_phrase`查询分析文本，并创建一个`phrase`查询出来的分析文字。例如：

```console
GET /_search
{
  "query": {
    "match_phrase": {
      "message": "this is a test"
    }
  }
}
```

A phrase query matches terms up to a configurable `slop` (which defaults to 0) in any order. Transposed terms have a slop of 2.

词组查询可以按`slop` 任意顺序匹配最多可配置（默认值为0）的字词。转置字词的斜率是2。

The `analyzer` can be set to control which analyzer will perform the analysis process on the text. It defaults to the field explicit mapping definition, or the default search analyzer, for example:

所述`analyzer`可以被设置为控制哪个分析器将在文本上执行的分析过程。它默认为字段显式映射定义或默认的搜索分析器，例如：

```console
GET /_search
{
  "query": {
    "match_phrase": {
      "message": {
        "query": "this is a test",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

This query also accepts `zero_terms_query`, as explained in [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html).

该查询也接受`zero_terms_query`，在解释[`match`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)。

##  Match phrase prefix query

Returns documents that contain the words of a provided text, in the **same order** as provided. The last term of the provided text is treated as a [prefix](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html), matching any words that begin with that term.

返回包含提供文本且与提供文本顺序一致的文档。提供文本的最后一个属于被当作[前缀](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html)，匹配任何一个开始于这个术语的单词。

###  Example request

The following search returns documents that contain phrases beginning with `quick brown f` in the `message` field.

以下搜索返回`message`字段包含以`quick brown f`开头的短语的文档 。

This search would match a `message` value of `quick brown fox` or `two quick brown ferrets` but not `the fox is quick and brown`.

该搜索将匹配`message`的值`quick brown fox`或`two quick brown ferrets`而不是`the fox is quick and brown`。

```console
GET /_search
{
  "query": {
    "match_phrase_prefix": {
      "message": {
        "query": "quick brown f"
      }
    }
  }
}
```

###  Top-level parameters for `match_phrase_prefix`

**`<field>`**

(Required, object) Field you wish to search.

（必填，对象）您要搜索的字段。

###  Parameters for `<field>`

**`query`**

(Required, string) Text you wish to find in the provided `<field>`.

（必需，字符串）您希望在提供的中找到的文本`<field>`。

The `match_phrase_prefix` query [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) any provided text into tokens before performing a search. The last term of this text is treated as a [prefix](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html), matching any words that begin with that term.

该`match_phrase_prefix`查询[分析](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)执行搜索之前的任何提供的文本为标记。此文本的最后一个术语被视为 [前缀](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html)，匹配以该术语开头的所有单词。

**`analyzer`**

(Optional, string) [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to convert text in the `query` value into tokens. Defaults to the [index-time analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/specify-analyzer.html#specify-index-time-analyzer) mapped for the `<field>`. If no analyzer is mapped, the index’s default analyzer is used.

（可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于将`query` 值中的文本转换为标记。默认为为映射的[索引时间分析器](https://www.elastic.co/guide/en/elasticsearch/reference/master/specify-analyzer.html#specify-index-time-analyzer)`<field>`。如果未映射任何分析器，则使用索引的默认分析器。

**`max_expansions`**

(Optional, integer) Maximum number of terms to which the last provided term of the `query` value will expand. Defaults to `50`.

（可选，整数）该`query`值的最后提供的项将扩展到的最大项数（以最后项开头的取？个加入到短语匹配中）。默认为`50`。

**`slop`**

(Optional, integer) Maximum number of positions allowed between matching tokens. Defaults to `0`. Transposed terms have a slop of `2`.

（可选，整数）匹配标记之间允许的最大位置数。默认为`0`。换位词的斜度为`2`。

**`zero_terms_query`**

(Optional, string) Indicates whether no documents are returned if the `analyzer` removes all tokens, such as when using a `stop` filter. Valid values are:

（可选，字符串）指示如果`analyzer` 删除所有标记（例如使用`stop`过滤器时），是否不返回任何文档。有效值为：

- **`none` (Default)**

  No documents are returned if the `analyzer` removes all tokens.

  如果`analyzer`删除所有标记，则不会返回任何文档。

- **`all`**

  Returns all documents, similar to a [`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-all-query.html) query.
  
  返回所有文档，类似于[`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-all-query.html) 查询。

###  Notes

####  Using the match phrase prefix query for search autocompletion

While easy to set up, using the `match_phrase_prefix` query for search autocompletion can sometimes produce confusing results.

虽然易于设置，但使用`match_phrase_prefix`查询进行自动补全有时会产生令人困惑的结果。

For example, consider the query string `quick brown f`. This query works by creating a phrase query out of `quick` and `brown` (i.e. the term `quick` must exist and must be followed by the term `brown`). Then it looks at the sorted term dictionary to find the first 50 terms that begin with `f`, and adds these terms to the phrase query.

例如，考虑查询字符串`quick brown f`。该查询通过在`quick`和`brown`之外创建词组查询来工作（即，该术语`quick`必须存在，并且`brown`必须紧随其后）。然后，它查看排序的术语词典，以找到以`f`开头的前50个术语，并将这些术语添加到短语查询中。

The problem is that the first 50 terms may not include the term `fox` so the phrase `quick brown fox` will not be found. This usually isn’t a problem as the user will continue to type more letters until the word they are looking for appears.

问题在于前50个术语可能不包含该术语`fox`，因此`quick brown fox`找不到该短语。这通常不是问题，因为用户将继续输入更多字母，直到他们要查找的单词出现为止。

For better solutions for *search-as-you-type* see the [completion suggester](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-suggesters.html#completion-suggester) and the [`search_as_you_type` field type](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-as-you-type.html).

有关按需*搜索的*更好解决方案，请参见 [完成建议器](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-suggesters.html#completion-suggester)和[`search_as_you_type`字段类型](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-as-you-type.html)。

##  Combined fields

The `combined_fields` query supports searching multiple text fields as if their contents had been indexed into one combined field. It takes a term-centric view of the query: first it analyzes the query string into individual terms, then looks for each term in any of the fields. This query is particularly useful when a match could span multiple text fields, for example the `title`, `abstract` and `body` of an article:

该`combined_fields`查询支持搜索多个文本字段，就好像它们的内容已被索引到一个组合字段中一样。它以查询的术语为中心：首先将查询字符串分析为单个术语，然后在任何字段中查找每个术语。该查询时特别有用的匹配可以跨越多个文本字段，例如`title`， `abstract`和`body`文章的：

```console
GET /_search
{
  "query": {
    "combined_fields" : {
      "query":      "database systems",
      "fields":     [ "title", "abstract", "body"],
      "operator":   "and"
    }
  }
}
```

The `combined_fields` query takes a principled approach to scoring based on the simple BM25F formula described in [The Probabilistic Relevance Framework: BM25 and Beyond](http://www.staff.city.ac.uk/~sb317/papers/foundations_bm25_review.pdf). When scoring matches, the query combines term and collection statistics across fields. This allows it to score each match as if the specified fields had been indexed into a single combined field. (Note that this is a best attempt — `combined_fields` makes some approximations and scores will not obey this model perfectly.)

`combined_fields`查询采用基于[概率相关框架BM25和Beyond中](http://www.staff.city.ac.uk/~sb317/papers/foundations_bm25_review.pdf)描述的简单BM25F公式的有原则的评分方法 。当得分匹配时，查询将跨字段组合术语和集合统计信息。这使它可以对每个匹配项进行评分，就好像指定字段已被索引到单个组合字段中一样。（请注意，这是最佳尝试- `combined_fields`做出一些近似，并且分数将不能完全遵循该模型。）

> **WARNING:** **Field number limit**
>
> There is a limit on the number of fields that can be queried at once. It is defined by the `indices.query.bool.max_clause_count` [Search settings](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html) which defaults to 1024.
>
> 一次可以查询的字段数有限制。它由“`indices.query.bool.max_clause_count` [搜索”设置](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html)定义 ，默认为1024。

###  Per-field boosting

Individual fields can be boosted with the caret (`^`) notation:

可以使用插入号（`^`）表示法增强各个字段：

```json
GET /_search
{
  "query": {
    "combined_fields" : {
      "query" : "distributed consensus",
      "fields" : [ "title^2", "body" ] 
    }
  }
}
```

Field boosts are interpreted according to the combined field model. For example, if the `title` field has a boost of 2, the score is calculated as if each term in the title appeared twice in the synthetic combined field.

根据组合的场模型来解释场增强。例如，如果`title`字段的增幅为2，则计算分数，就好像标题中的每个术语在合成组合字段中出现两次一样。

> **NOTE:**The `combined_fields` query requires that field boosts are greater than or equal to 1.0. Field boosts are allowed to be fractional.
>
> 该`combined_fields`查询要求字段增强大于或等于1.0。字段增强可以是分数。

###  Top-level parameters for `combined_fields`

**`fields`**

(Required, array of strings) List of fields to search. Field wildcard patterns are allowed. Only [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) fields are supported, and they must all have the same search [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/master/analyzer.html).

（必需，字符串数组）要搜索的字段列表。允许使用字段通配符模式。仅[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html)支持字段，并且它们都必须具有相同的搜索[`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/master/analyzer.html)。

**`query`**

(Required, string) Text to search for in the provided `<fields>`.

（必需，字符串）在提供的中搜索的文本`<fields>`。

The `combined_fields` query [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) the provided text before performing a search.

该`combined_fields`查询在执行搜索之前会[分析](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)提供的文本。

**`auto_generate_synonyms_phrase_query`**

(Optional, Boolean) If `true`, [match phrase](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html) queries are automatically created for multi-term synonyms. Defaults to `true`.

（可选，布尔值）如果为`true`， 则会自动为多个术语同义词创建[匹配词组](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html)查询。默认为`true`。

See [Use synonyms with match query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-synonyms) for an example.

有关示例，请参阅[将同义词与匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-synonyms)一起使用。

**`operator`**

(Optional, string) Boolean logic used to interpret text in the `query` value. Valid values are:

（可选，字符串）布尔逻辑，用于解释`query`值中的文本。有效值为：

- **`or` (Default)**

  For example, a `query` value of `database systems` is interpreted as `database OR systems`.

  例如，`query`值`database systems`解释为`database OR systems`。

- **`and`**

  For example, a `query` value of `database systems` is interpreted as `database AND systems`.

  例如，`query`值`database systems`解释为`database AND systems`。

**`minimum_should_match`**

(Optional, string) Minimum number of clauses that must match for a document to be returned. See the [`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html) for valid values and more information.

（可选，字符串）要返回的文档必须匹配的最小子句数。请参阅[`minimum_should_match` 参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html)以获取有效值和更多信息。

**`zero_terms_query`**

(Optional, string) Indicates whether no documents are returned if the `analyzer` removes all tokens, such as when using a `stop` filter. Valid values are:

（可选，字符串）指示如果`analyzer` 删除所有标记（例如使用`stop`过滤器时），是否不返回任何文档。有效值为：

- **`none` (Default)**

  No documents are returned if the `analyzer` removes all tokens.

  如果`analyzer`删除所有标记，则不会返回任何文档。

- **`all`**

  Returns all documents, similar to a [`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-all-query.html) query.

  返回所有文档，类似于[`match_all`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-all-query.html) 查询。

See [Zero terms query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-zero) for an example.

有关示例，请参见[零项查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html#query-dsl-match-query-zero)。

###  Comparison to `multi_match` query

The `combined_fields` query provides a principled way of matching and scoring across multiple [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) fields. To support this, it requires that all fields have the same search [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/master/analyzer.html).

`combined_fields`查询提供了跨多个[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html)字段进行匹配和评分的原则方法。为了支持这一点，它要求所有字段都具有相同的search [`analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/master/analyzer.html)。

If you want a single query that handles fields of different types like keywords or numbers, then the [`multi_match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html) query may be a better fit. It supports both text and non-text fields, and accepts text fields that do not share the same analyzer.

如果您希望使用一个查询来处理不同类型的字段（例如关键字或数字），那么该[`multi_match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html) 查询可能更合适。它同时支持文本字段和非文本字段，并接受不共享同一分析器的文本字段。

The main `multi_match` modes `best_fields` and `most_fields` take a field-centric view of the query. In contrast, `combined_fields` is term-centric: `operator` and `minimum_should_match` are applied per-term, instead of per-field. Concretely, a query like

主要`multi_match`模式`best_fields`和`most_fields`采取以字段为中心的查询视图。相反，`combined_fields`是以术语为中心的：`operator`和`minimum_should_match`按术语而不是按字段应用。具体来说，像

```console
GET /_search
{
  "query": {
    "combined_fields" : {
      "query":      "database systems",
      "fields":     [ "title", "abstract"],
      "operator":   "and"
    }
  }
}
```

is executed as

`+(combined("database", fields:["title" "abstract"]))`

`+(combined("systems", fields:["title", "abstract"]))`

In other words, each term must be present in at least one field for a document to match.

换句话说，每个术语都必须存在于至少一个字段中才能与文档匹配。

The `cross_fields` `multi_match` mode also takes a term-centric approach and applies `operator` and `minimum_should_match per-term`. The main advantage of `combined_fields` over `cross_fields` is its robust and interpretable approach to scoring based on the BM25F algorithm.

该`cross_fields` `multi_match`模式还采用以术语为中心的方法，并应用`operator`和`minimum_should_match per-term`。`combined_fields`超过 `cross_fields`的主要优点是其基于BM25F算法的稳健且可解释的评分方法。

> **NOTE: ** **Custom similarities**
>
> The `combined_fields` query currently only supports the `BM25` similarity (which is the default unless a [custom similarity](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules-similarity.html) is configured). [Per-field similarities](https://www.elastic.co/guide/en/elasticsearch/reference/master/similarity.html) are also not allowed. Using `combined_fields` in either of these cases will result in an error.
>
> 该`combined_fields`查询当前仅支持`BM25`相似性（除非 配置了[自定义相似性，](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules-similarity.html)否则这是默认设置）。也不允许[每个字段相似](https://www.elastic.co/guide/en/elasticsearch/reference/master/similarity.html)。使用`combined_fields`在这两种情况下会导致错误。

##  Multi-match query

The `multi_match` query builds on the [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html) to allow multi-field queries:

在`multi_match`查询基础上的[`match`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html) ，允许多领域的查询：

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 	 （1）
      "fields": [ "subject", "message" ] （2） 
    }
  }
}
```

1.  The query string.

   查询字符串。

2.  The fields to be queried.

   要查询的字段。

### `fields` and per-field boosting

Fields can be specified with wildcards, eg:

字段可以用通配符指定，例如：

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "Will Smith",
      "fields": [ "title", "*_name" ] （1）
    }
  }
}
```

1. Query the `title`, `first_name` and `last_name` fields.

Individual fields can be boosted with the caret (`^`) notation:

可以使用插入号（`^`）表示法增强各个字段：

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query" : "this is a test",
      "fields" : [ "subject^3", "message" ] （1）
    }
  }
}
```

1. The query multiplies the `subject` field’s score by three but leaves the `message` field’s score unchanged.

   该查询将`subject`字段的分数乘以三，但 `message`字段的分数保持不变。

If no `fields` are provided, the `multi_match` query defaults to the `index.query.default_field` index settings, which in turn defaults to `*`. `*` extracts all fields in the mapping that are eligible to term queries and filters the metadata fields. All extracted fields are then combined to build a query.

如果未`fields`提供，则`multi_match`查询默认为`index.query.default_field` 索引设置，而索引设置依次为`*`。`*`提取映射中符合词条查询条件的所有字段，并过滤元数据字段。然后将所有提取的字段组合起来以构建查询。

> **WARNING:** There is a limit on the number of fields that can be queried at once. It is defined by the `indices.query.bool.max_clause_count` [Search settings](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html) which defaults to 1024.
>
> 一次可以查询的字段数有限制。它由“`indices.query.bool.max_clause_count` [搜索”设置](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html)定义 ，默认为1024。

###  Types of `multi_match` query:

The way the `multi_match` query is executed internally depends on the `type` parameter, which can be set to:

`multi_match`内部执行查询的方式取决于`type` 参数，可以将其设置为：

| `best_fields`   | (**default**) Finds documents which match any field, but uses the `_score` from the best field. See [`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-best-fields).<br />（**默认**）查找与任何字段匹配但使用`_score`最佳的文档字段 。请参阅[`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-best-fields)。 |
| --------------- | ------------------------------------------------------------ |
| `most_fields`   | Finds documents which match any field and combines the `_score` from each field. See [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-most-fields).<br /> 查找与任何字段匹配的文档，并将`_score`每个字段中的合并。请参阅[`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-most-fields)。 |
| `cross_fields`  | Treats fields with the same `analyzer` as though they were one big field. Looks for each word in **any** field. See [`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-cross-fields).<br />对待每一个字段都使用相同的`analyzer`即使他是一个大字段。在**任何** 字段中查找每个单词。请参阅[`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-cross-fields)。 |
| `phrase`        | Runs a `match_phrase` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-phrase).<br />`match_phrase`在每个字段上 运行查询，并使用`_score` 最佳字段中的。请参阅[`phrase`和`phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-phrase)。 |
| `phrase_prefix` | Runs a `match_phrase_prefix` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-phrase).<br />`match_phrase_prefix`在每个字段上 运行查询，并使用`_score`最佳字段中的。请参阅[`phrase`和`phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-phrase)。 |
| `bool_prefix`   | Creates a `match_bool_prefix` query on each field and combines the `_score` from each field. See [`bool_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-bool-prefix).<br />`match_bool_prefix`在每个字段上 创建查询，并将`_score`每个字段中的合并。请参阅 [`bool_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-bool-prefix)。 |

### `best_fields`

The `best_fields` type is most useful when you are searching for multiple words best found in the same field. For instance “brown fox” in a single field is more meaningful than “brown” in one field and “fox” in the other.

`best_fields`当您搜索在同一字段中最好找到的多个单词时，该类型最有用。例如，单个字段中的“棕狐”比一个字段中的“棕”和另一字段中的“ fox”更有意义。

The `best_fields` type generates a [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html) for each field and wraps them in a [`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-dis-max-query.html) query, to find the single best matching field. For instance, this query:

该`best_fields`类型为每个字段生成一个[`match`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)，并将它们包装在[`dis_max`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-dis-max-query.html)查询中，以找到最匹配的单个字段。例如，此查询：

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "brown fox",
      "type":       "best_fields",
      "fields":     [ "subject", "message" ],
      "tie_breaker": 0.3
    }
  }
}
```

would be executed as:

将执行为：

```json
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "subject": "brown fox" }},
        { "match": { "message": "brown fox" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
```

Normally the `best_fields` type uses the score of the **single** best matching field, but if `tie_breaker` is specified, then it calculates the score as follows:

通常，该`best_fields`类型使用**单个**最佳匹配字段的分数，但是如果`tie_breaker`指定了该分数，则它将按以下方式计算分数：

- the score from the best matching field

  最佳匹配领域的分数

- plus `tie_breaker * _score` for all other matching fields

  加上`tie_breaker * _score`所有其他匹配字段

Also, accepts `analyzer`, `boost`, `operator`, `minimum_should_match`, `fuzziness`, `lenient`, `prefix_length`, `max_expansions`, `fuzzy_rewrite`, `zero_terms_query`, `auto_generate_synonyms_phrase_query` and `fuzzy_transpositions`, as explained in [match query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html).

此外，接受`analyzer`，`boost`，`operator`，`minimum_should_match`， `fuzziness`，`lenient`，`prefix_length`，`max_expansions`，`fuzzy_rewrite`，`zero_terms_query`， `auto_generate_synonyms_phrase_query`和`fuzzy_transpositions`，作为解释[匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)。

> **IMPORTANT:** `operator` and `minimum_should_match`
>
> The `best_fields` and `most_fields` types are *field-centric* — they generate a `match` query **per field**. This means that the `operator` and `minimum_should_match` parameters are applied to each field individually, which is probably not what you want.
>
> `best_fields`和`most_fields`类型是*字段为中心* -它们产生`match`查询**每个字段**。这意味着`operator`和 `minimum_should_match`参数分别应用于每个字段，这可能不是您想要的。
>
> Take this query for example:
>
> ```console
> GET /_search
> {
>   "query": {
>     "multi_match" : {
>       "query":      "Will Smith",
>       "type":       "best_fields",
>       "fields":     [ "first_name", "last_name" ],
>       "operator":   "and"  （1）
>     }
>   }
> }
> ```
>
> 1.  All terms must be present.
>
>     所有术语都必须存在。
>
> This query is executed as:
>
> 该查询的执行方式为：
>
> ```
>   (+first_name:will +first_name:smith)
> | (+last_name:will  +last_name:smith)
> ```
>
> In other words, **all terms** must be present **in a single field** for a document to match.
>
> 换句话说，**所有术语都**必须存在**于单个字段中**才能与文档匹配。
>
> The [`combined_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-combined-fields-query.html) query offers a term-centric approach that handles `operator` and `minimum_should_match` on a per-term basis. The other multi-match mode [`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-cross-fields) also addresses this issue.
>
> 该[`combined_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-combined-fields-query.html)查询提供了一个长期为中心的方法来处理`operator`，并`minimum_should_match`在每个术语的基础。另一种多重匹配模式[`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-cross-fields)也解决了这个问题。

### `most_fields`

The `most_fields` type is most useful when querying multiple fields that contain the same text analyzed in different ways. For instance, the main field may contain synonyms, stemming and terms without diacritics. A second field may contain the original terms, and a third field might contain shingles. By combining scores from all three fields we can match as many documents as possible with the main field, but use the second and third fields to push the most similar results to the top of the list.

`most_fields`当查询包含以不同方式分析的相同文本的多个字段时，该类型最有用。例如，主字段可能包含同义词，词干和术语，而没有附加符号。第二个字段可能包含原始术语，第三个字段可能包含单独一个。通过合并所有三个字段的得分，我们可以将尽可能多的文档与主字段匹配，但是使用第二和第三字段将最相似的结果推到列表的顶部。

This query:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown fox",
      "type":       "most_fields",
      "fields":     [ "title", "title.original", "title.shingles" ]
    }
  }
}
```

would be executed as:

```console
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":          "quick brown fox" }},
        { "match": { "title.original": "quick brown fox" }},
        { "match": { "title.shingles": "quick brown fox" }}
      ]
    }
  }
}
```

The score from each `match` clause is added together, then divided by the number of `match` clauses.

每个`match`子句的分数相加，然后除以`match`子句数。

Also, accepts `analyzer`, `boost`, `operator`, `minimum_should_match`, `fuzziness`, `lenient`, `prefix_length`, `max_expansions`, `fuzzy_rewrite`, and `zero_terms_query`.

此外，接受`analyzer`，`boost`，`operator`，`minimum_should_match`， `fuzziness`，`lenient`，`prefix_length`，`max_expansions`，`fuzzy_rewrite`，和`zero_terms_query`。

### `phrase` and `phrase_prefix`

The `phrase` and `phrase_prefix` types behave just like [`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-best-fields), but they use a `match_phrase` or `match_phrase_prefix` query instead of a `match` query.

`phrase`和`phrase_prefix`类型的行为很像[`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-best-fields)，但它们使用`match_phrase`或`match_phrase_prefix`查询，而不是一个的 `match`查询。

This query:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown f",
      "type":       "phrase_prefix",
      "fields":     [ "subject", "message" ]
    }
  }
}
```

would be executed as:

```console
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match_phrase_prefix": { "subject": "quick brown f" }},
        { "match_phrase_prefix": { "message": "quick brown f" }}
      ]
    }
  }
}
```

Also, accepts `analyzer`, `boost`, `lenient` and `zero_terms_query` as explained in [Match](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html), as well as `slop` which is explained in [Match phrase](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html). Type `phrase_prefix` additionally accepts `max_expansions`.

此外，接受`analyzer`，`boost`，`lenient`和`zero_terms_query`如在解释[匹配](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)，以及`slop`其中解释[匹配短语](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html)。类型`phrase_prefix`另外接受`max_expansions`。

> **IMPORTANT:** `phrase`, `phrase_prefix` and `fuzziness`
>
> The `fuzziness` parameter cannot be used with the `phrase` or `phrase_prefix` type.
>
> `fuzziness`参数不能与`phrase`或`phrase_prefix`类型一起使用。

### `cross_fields`

The `cross_fields` type is particularly useful with structured documents where multiple fields **should** match. For instance, when querying the `first_name` and `last_name` fields for “Will Smith”, the best match is likely to have “Will” in one field and “Smith” in the other.

`cross_fields`类型对于多个字段**应**匹配的结构化文档特别有用。例如，当在`first_name` 和`last_name`字段中查询“ Will Smith”时，最匹配的一个字段可能是“ Will”，而另一个字段中是“ Smith”。

> This sounds like a job for [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-most-fields) but there are two problems with that approach. The first problem is that `operator` and `minimum_should_match` are applied per-field, instead of per-term (see [explanation above](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#operator-min)).
>
> [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-most-fields)听起来像是一项工作，但是这种方法存在两个问题。第一个问题是`operator`和 `minimum_should_match`是按字段而不是按术语应用的（请参阅 [上面的说明](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#operator-min)）。
>
> The second problem is to do with relevance: the different term frequencies in the `first_name` and `last_name` fields can produce unexpected results.
>
> 第二个问题与相关性有关：`first_name`和`last_name`字段中不同的词频可能会产生意想不到的结果。
>
> For instance, imagine we have two people: “Will Smith” and “Smith Jones”. “Smith” as a last name is very common (and so is of low importance) but “Smith” as a first name is very uncommon (and so is of great importance).
>
> 例如，假设我们有两个人：“威尔·史密斯”和“史密斯·琼斯”。“ Smith”作为姓很常见（因此重要性不高），但“ Smith”作为名却很少见（因此重要性很强）。
>
> If we do a search for “Will Smith”, the “Smith Jones” document will probably appear above the better matching “Will Smith” because the score of `first_name:smith` has trumped the combined scores of `first_name:will` plus `last_name:smith`.
>
> 如果我们搜索“威尔·史密斯”，“史密斯·琼斯”文件可能会出现在匹配度更高的“威尔·史密斯”上方，因为的得分 `first_name:smith`比`first_name:will`加 `last_name:smith`的总和还要高。

One way of dealing with these types of queries is simply to index the `first_name` and `last_name` fields into a single `full_name` field. Of course, this can only be done at index time.

处理这些类型的查询的一种方法是简单地将`first_name`and`last_name`字段索引 到单个`full_name`字段中。当然，这只能在索引时完成。

The `cross_field` type tries to solve these problems at query time by taking a *term-centric* approach. It first analyzes the query string into individual terms, then looks for each term in any of the fields, as though they were one big field.

该`cross_field`类型尝试通过以*术语为中心的*方法在查询时解决这些问题 。它首先将查询字符串分析为单个术语，然后在任何字段中查找每个术语，就好像它们是一个大字段一样。

A query like:

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Will Smith",
      "type":       "cross_fields",
      "fields":     [ "first_name", "last_name" ],
      "operator":   "and"
    }
  }
}
```

is executed as:

```
+(first_name:will last_name:will)
+(first_name:smith last_name:smith)
```

In other words, **all terms** must be present **in at least one field** for a document to match. (Compare this to [the logic used for `best_fields` and `most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#operator-min).)

换句话说，**所有术语都**必须存在**于至少一个字段中**才能与文档匹配。（将此与[用于`best_fields`和的逻辑`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#operator-min)进行比较 。）

That solves one of the two problems. The problem of differing term frequencies is solved by *blending* the term frequencies for all fields in order to even out the differences.

这解决了两个问题之一。通过*混合*所有字段的术语频率以使差异均匀来解决术语频率不同的问题。

In practice, `first_name:smith` will be treated as though it has the same frequencies as `last_name:smith`, plus one. This will make matches on `first_name` and `last_name` have comparable scores, with a tiny advantage for `last_name` since it is the most likely field that contains `smith`.

在实践中，`first_name:smith`将被视为具有与相同的频率`last_name:smith`，再加上一个。这将使匹配继续进行 `first_name`并`last_name`获得可比分数，`last_name`这是一个极小的优势，因为它是包含的最可能字段`smith`。

Note that `cross_fields` is usually only useful on short string fields that all have a `boost` of `1`. Otherwise boosts, term freqs and length normalization contribute to the score in such a way that the blending of term statistics is not meaningful anymore.

请注意，这`cross_fields`通常仅对全部为`boost`的短字符串字段有用`1`。否则，助推，术语频率和长度归一化会以一种使得术语统计不再有意义的方式对得分有所贡献。

If you run the above query through the [Validate](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-validate.html), it returns this explanation:

如果通过[Validate](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-validate.html)运行以上查询，则返回以下说明：

```
+blended("will",  fields: [first_name, last_name])
+blended("smith", fields: [first_name, last_name])
```

Also, accepts `analyzer`, `boost`, `operator`, `minimum_should_match`, `lenient` and `zero_terms_query`.

此外，接受`analyzer`，`boost`，`operator`，`minimum_should_match`， `lenient`和`zero_terms_query`。

> **WARNING:** The `cross_fields` type blends field statistics in a way that does not always produce well-formed scores (for example scores can become negative). As an alternative, you can consider the [`combined_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-combined-fields-query.html) query, which is also term-centric but combines field statistics in a more robust way.
>
> `cross_fields`类型以不总是产生格式正确的分数的方式混合字段统计信息（例如，分数可能变为负数）。作为替代方案，您可以考虑 [`combined_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-combined-fields-query.html)查询，该查询也以术语为中心，但以更可靠的方式组合了字段统计信息。

### `cross_field` and analysis

The `cross_field` type can only work in term-centric mode on fields that have the same analyzer. Fields with the same analyzer are grouped together as in the example above. If there are multiple groups, the query will use the best score from any group.

该`cross_field`类型只能在以术语为中心的模式下用于具有相同分析器的字段。具有相同分析器的字段按上述示例分组在一起。如果有多个组，则查询将使用任何组中的最佳分数。

For instance, if we have a `first` and `last` field which have the same analyzer, plus a `first.edge` and `last.edge` which both use an `edge_ngram` analyzer, this query:

例如，如果我们有一个`first`和`last`字段具有相同的分析器，再加上一个`first.edge`和`last.edge`都使用一个`edge_ngram`分析器，则此查询：

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "Jon",
      "type":       "cross_fields",
      "fields":     [
        "first", "first.edge",
        "last",  "last.edge"
      ]
    }
  }
}
```

would be executed as:

```
    blended("jon", fields: [first, last])
| (
    blended("j",   fields: [first.edge, last.edge])
    blended("jo",  fields: [first.edge, last.edge])
    blended("jon", fields: [first.edge, last.edge])
)
```

In other words, `first` and `last` would be grouped together and treated as a single field, and `first.edge` and `last.edge` would be grouped together and treated as a single field.

换句话说，`first`和`last`将被组合在一起并被视为一个字段，`first.edge`并且`last.edge`将被组合在一起并被视为一个字段。

Having multiple groups is fine, but when combined with `operator` or `minimum_should_match`, it can suffer from the [same problem](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#operator-min) as `most_fields` or `best_fields`.

有多个组是好的，但是当联合`operator`或者 `minimum_should_match`，它可以从遭受[同样的问题，](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#operator-min) 如`most_fields`或`best_fields`。

You can easily rewrite this query yourself as two separate `cross_fields` queries combined with a `dis_max` query, and apply the `minimum_should_match` parameter to just one of them:

您可以轻松地将自己作为两个单独的`cross_fields` 查询与一个`dis_max`查询合并重写该查询，并将`minimum_should_match` 参数仅应用于其中一个：

```console
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        {
          "multi_match" : {
            "query":      "Will Smith",
            "type":       "cross_fields",
            "fields":     [ "first", "last" ],
            "minimum_should_match": "50%"  （1）
          }
        },
        {
          "multi_match" : {
            "query":      "Will Smith",
            "type":       "cross_fields",
            "fields":     [ "*.edge" ]
          }
        }
      ]
    }
  }
}
```

1.  Either `will` or `smith` must be present in either of the `first` or `last` fields

    任一`will`或`smith`必须在任一的存在`first` 或`last`字段

You can force all fields into the same group by specifying the `analyzer` parameter in the query.

您可以通过`analyzer` 在查询中指定参数来将所有字段都归为同一组。

```console
GET /_search
{
  "query": {
   "multi_match" : {
      "query":      "Jon",
      "type":       "cross_fields",
      "analyzer":   "standard", （1）
      "fields":     [ "first", "last", "*.edge" ]
    }
  }
}
```

1. Use the `standard` analyzer for all fields.

   将`standard`分析仪用于所有字段。

which will be executed as:

```
blended("will",  fields: [first, first.edge, last.edge, last])
blended("smith", fields: [first, first.edge, last.edge, last])
```

### `tie_breaker`

By default, each per-term `blended` query will use the best score returned by any field in a group. Then when combining scores across groups, the query uses the best score from any group. The `tie_breaker` parameter can change the behavior for both of these steps:

默认情况下，每个按词`blended`查询将使用组中任何字段返回的最佳分数。然后，在组合各个组的分数时，查询将使用任何组中的最佳分数。该`tie_breaker`参数可以更改以下两个步骤的行为：

| `0.0`           | Take the single best score out of (eg) `first_name:will` and `last_name:will` (default)<br />从（例如）`first_name:will` 和`last_name:will`（默认）中 取一个最佳分数 |
| --------------- | ------------------------------------------------------------ |
| `1.0`           | Add together the scores for (eg) `first_name:will` and `last_name:will`<br />将（例如）`first_name:will`和 `last_name:will` |
| `0.0 < n < 1.0` | Take the single best score plus `tie_breaker` multiplied by each of the scores from other matching fields/ groups<br />取单个最佳分数，再`tie_breaker`乘以其他匹配字段/组的每个分数 |

> **IMPORTANT:** `cross_fields` and `fuzziness`
>
> The `fuzziness` parameter cannot be used with the `cross_fields` type.
>
> `fuzziness`参数不能与`cross_fields`类型一起使用。

### `bool_prefix`

The `bool_prefix` type’s scoring behaves like [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-most-fields), but using a [`match_bool_prefix` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-bool-prefix-query.html) instead of a `match` query.

该`bool_prefix`类型的得分的行为像[`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-most-fields)，但使用 [`match_bool_prefix`的查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-bool-prefix-query.html)，而不是一个的 `match`查询。

```console
GET /_search
{
  "query": {
    "multi_match" : {
      "query":      "quick brown f",
      "type":       "bool_prefix",
      "fields":     [ "subject", "message" ]
    }
  }
}
```

The `analyzer`, `boost`, `operator`, `minimum_should_match`, `lenient`, `zero_terms_query`, and `auto_generate_synonyms_phrase_query` parameters as explained in [match query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html) are supported. The `fuzziness`, `prefix_length`, `max_expansions`, `fuzzy_rewrite`, and `fuzzy_transpositions` parameters are supported for the terms that are used to construct term queries, but do not have an effect on the prefix query constructed from the final term.

`analyzer`，`boost`，`operator`，`minimum_should_match`，`lenient`， `zero_terms_query`，和`auto_generate_synonyms_phrase_query`在解释的参数[匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)的支持。`fuzziness`，`prefix_length`，`max_expansions`，`fuzzy_rewrite`，和 `fuzzy_transpositions`参数都支持了用于构建长期的查询条件，但没有对前缀查询从最终期限结构的影响。

The `slop` parameter is not supported by this query type.

该`slop`查询类型不支持该参数。

##  Query string query

Returns documents based on a provided query string, using a parser with a strict syntax.

使用具有严格语法的解析器，根据提供的查询字符串返回文档。

This query uses a [syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-syntax) to parse and split the provided query string based on operators, such as `AND` or `NOT`. The query then [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) each split text independently before returning matching documents.

此查询使用[语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-syntax)根据运算符（例如`AND`或`NOT`）来解析和拆分提供的查询字符串。然后查询在返回匹配的文档之前独立[分析](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)每个拆分的文本。

You can use the `query_string` query to create a complex search that includes wildcard characters, searches across multiple fields, and more. While versatile, the query is strict and returns an error if the query string includes any invalid syntax.

您可以使用该`query_string`查询创建一个复杂的搜索，其中包括通配符，跨多个字段的搜索等等。尽管用途广泛，但查询是严格的，如果查询字符串包含任何无效语法，则返回错误。

> **WARNING：**
> Because it returns an error for any invalid syntax, we don’t recommend using the `query_string` query for search boxes.
>
> 由于它针对任何无效的语法返回错误，因此我们不建议`query_string`对搜索框使用查询。
>
> If you don’t need to support a query syntax, consider using the [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html) query. If you need the features of a query syntax, use the [`simple_query_string`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html) query, which is less strict.
>
> 如果不需要支持查询语法，请考虑使用 [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)查询。如果需要查询语法的功能，请使用[`simple_query_string`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html) 不太严格的查询。

###  Example request

When running the following search, the `query_string` query splits `(new york city) OR (big apple)` into two parts: `new york city` and `big apple`. The `content` field’s analyzer then independently converts each part into tokens before returning matching documents. Because the query syntax does not use whitespace as an operator, `new york city` is passed as-is to the analyzer.

运行以下搜索时，`query_string`查询分隔`(new york city) OR (big apple)`为两部分：`new york city`和`big apple`。然后`content`在字段的分析器在返回匹配的文档之前，将每个部分独立地转换为标记。由于查询语法不使用空格作为运算符，`new york city`因此按原样传递给分析器。

```console
GET /_search
{
  "query": {
    "query_string": {
      "query": "(new york city) OR (big apple)",
      "default_field": "content"
    }
  }
}
```

###  Top-level parameters for `query_string`

**`query`**

(Required, string) Query string you wish to parse and use for search. See [Query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-syntax).

（必填，字符串）您要解析并用于搜索的查询字符串。请参阅 [查询字符串语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-syntax)。

**`default_field`**

(Optional, string) Default field you wish to search if no field is provided in the query string.

（可选，字符串）如果查询字符串中未提供任何字段，则为您要搜索的默认字段。

Defaults to the `index.query.default_field` index setting, which has a default value of `*`. The `*` value extracts all fields that are eligible for term queries and filters the metadata fields. All extracted fields are then combined to build a query if no `prefix` is specified.

默认为`index.query.default_field`索引设置，其默认值为`*`。该`*`值提取符合条件查询的所有字段，并过滤元数据字段。如果未`prefix`指定，则合并所有提取的字段以构建查询。

Searching across all eligible fields does not include [nested documents](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html). Use a [`nested` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-nested-query.html) to search those documents.

在所有符合条件的字段中搜索不包括[嵌套文档](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html)。使用[`nested`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-nested-query.html)来搜索那些文档。

For mappings with a large number of fields, searching across all eligible fields could be expensive.

对于具有大量字段的映射，在所有符合条件的字段中进行搜索可能会很昂贵。

There is a limit on the number of fields that can be queried at once. It is defined by the `indices.query.bool.max_clause_count` [search setting](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html), which defaults to 1024.

一次可以查询的字段数有限制。它由`indices.query.bool.max_clause_count` [搜索设置](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html)定义，默认为1024。

**`allow_leading_wildcard`**

(Optional, Boolean) If `true`, the wildcard characters `*` and `?` are allowed as the first character of the query string. Defaults to `true`.

（可选，布尔值）如果为`true`，则允许使用通配符`*`和`?`作为查询字符串的第一个字符。默认为`true`。

**`analyze_wildcard`**

(Optional, Boolean) If `true`, the query attempts to analyze wildcard terms in the query string. Defaults to `false`.

（可选，布尔值）如果为`true`，则查询尝试分析查询字符串中的通配符术语。默认为`false`。

**`analyzer`**

(Optional, string) [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to convert text in the query string into tokens. Defaults to the [index-time analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/specify-analyzer.html#specify-index-time-analyzer) mapped for the `default_field`. If no analyzer is mapped, the index’s default analyzer is used.

（可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于将查询字符串中的文本转换为标记。默认为为映射`default_field` 的 [索引时间分析器](https://www.elastic.co/guide/en/elasticsearch/reference/master/specify-analyzer.html#specify-index-time-analyzer)。如果未映射任何分析器，则使用索引的默认分析器。

**`auto_generate_synonyms_phrase_query`**

(Optional, Boolean) If `true`, [match phrase](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html) queries are automatically created for multi-term synonyms. Defaults to `true`. See [Synonyms and the `query_string` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-synonyms) for an example.

（可选，布尔值）如果为`true`， 则会自动为多个术语同义词创建[匹配词组](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html)查询。默认为`true`。有关示例，请参见[同义词和`query_string`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-synonyms)。

**`boost`**

(Optional, float) Floating point number used to decrease or increase the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of the query. Defaults to `1.0`.

（可选，float）用于减少或增加查询的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)的浮点数 。默认为`1.0`。

Boost values are relative to the default value of `1.0`. A boost value between `0` and `1.0` decreases the relevance score. A value greater than `1.0` increases the relevance score.

提升值是相对于默认值的`1.0`。介于`0`和之间的提升值会 `1.0`降低相关性得分。大于的值会`1.0` 增加相关性得分。

**`default_operator`**

(Optional, string) Default boolean logic used to interpret text in the query string if no operators are specified. Valid values are:

（可选，字符串）如果未指定运算符，则用于解释查询字符串中的文本的默认布尔逻辑。有效值为：

- **`OR` (Default)**

  For example, a query string of `capital of Hungary` is interpreted as `capital OR of OR Hungary`.

  例如，查询字符串`capital of Hungary`解释为`capital OR of OR Hungary`。

- **`AND`**

  For example, a query string of `capital of Hungary` is interpreted as `capital AND of AND Hungary`.

  例如，查询字符串`capital of Hungary`解释为`capital AND of AND Hungary`。

**`enable_position_increments`**

(Optional, Boolean) If `true`, enable position increments in queries constructed from a `query_string` search. Defaults to `true`.

（可选，布尔值）如果为`true`，则在通过`query_string`搜索构造的查询中启用位置增量。默认为`true`。

**`fields`**

(Optional, array of strings) Array of fields you wish to search.

（可选，字符串数组）您要搜索的字段数组。

You can use this parameter query to search across multiple fields. See [Search multiple fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-multi-field).

您可以使用此参数查询来搜索多个字段。请参阅 [搜索多个字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-multi-field)。

**`fuzziness`**

(Optional, string) Maximum edit distance allowed for matching. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness) for valid values and more information.

（可选，字符串）匹配允许的最大编辑距离。有关 有效值和更多信息，请参见[模糊性](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness)。

**`fuzzy_max_expansions`**

(Optional, integer) Maximum number of terms to which the query expands for fuzzy matching. Defaults to `50`.

（可选，整数）查询为模糊匹配而扩展到的最大术语数。默认为`50`。

**`fuzzy_prefix_length`**

(Optional, integer) Number of beginning characters left unchanged for fuzzy matching. Defaults to `0`.

（可选，整数）为模糊匹配保留的起始字符数。默认为`0`。

**`fuzzy_transpositions`**

(Optional, Boolean) If `true`, edits for fuzzy matching include transpositions of two adjacent characters (ab → ba). Defaults to `true`.

（可选，布尔值）如果`true`为，则模糊匹配的编辑内容包括两个相邻字符的变位（ab→ba）。默认为`true`。

**`lenient`**

(Optional, Boolean) If `true`, format-based errors, such as providing a text value for a [numeric](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) field, are ignored. Defaults to `false`.

（可选，布尔值）如果为`true`，则将忽略基于格式的错误，例如为[数字](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)字段提供文本值。默认为`false`。

**`max_determinized_states`**

(Optional, integer) Maximum number of [automaton states](https://en.wikipedia.org/wiki/Deterministic_finite_automaton) required for the query. Default is `10000`.

（可选，整数） 查询所需的[自动机状态的](https://en.wikipedia.org/wiki/Deterministic_finite_automaton)最大数量 。默认值为`10000`。

Elasticsearch uses [Apache Lucene](https://lucene.apache.org/core/) internally to parse regular expressions. Lucene converts each regular expression to a finite automaton containing a number of determinized states.

Elasticsearch在内部使用[Apache Lucene](https://lucene.apache.org/core/)解析正则表达式。Lucene将每个正则表达式转换为包含许多确定状态的有限自动机。

You can use this parameter to prevent that conversion from unintentionally consuming too many resources. You may need to increase this limit to run complex regular expressions.

您可以使用此参数来防止该转换无意中消耗过多的资源。您可能需要增加此限制才能运行复杂的正则表达式。

**`minimum_should_match`**

(Optional, string) Minimum number of clauses that must match for a document to be returned. See the [`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html) for valid values and more information. See [How `minimum_should_match` works](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-min-should-match) for an example.

（可选，字符串）要返回的文档必须匹配的最小子句数。请参阅[`minimum_should_match` 参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html)以获取有效值和更多信息。请参阅 [如何`minimum_should_match`工作](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-min-should-match)的一个例子。

**`quote_analyzer`**

(Optional, string) [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to convert quoted text in the query string into tokens. Defaults to the [`search_quote_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/master/analyzer.html#search-quote-analyzer) mapped for the `default_field`.

（可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于将查询字符串中带引号的文本转换为标记。默认为的 [`search_quote_analyzer`](https://www.elastic.co/guide/en/elasticsearch/reference/master/analyzer.html#search-quote-analyzer)映射 `default_field`。

For quoted text, this parameter overrides the analyzer specified in the `analyzer` parameter.

对于带引号的文本，此参数将覆盖参数中指定的分析器 `analyzer`。

**`phrase_slop`**

(Optional, integer) Maximum number of positions allowed between matching tokens for phrases. Defaults to `0`. If `0`, exact phrase matches are required. Transposed terms have a slop of `2`.

对于带引号的文本，此参数将覆盖参数中指定的分析器 `analyzer`。

**`quote_field_suffix`**

(Optional, string) Suffix appended to quoted text in the query string.

（可选，字符串）后缀附加到查询字符串中的带引号的文本中。

You can use this suffix to use a different analysis method for exact matches. See [Mixing exact search with stemming](https://www.elastic.co/guide/en/elasticsearch/reference/master/mixing-exact-search-with-stemming.html).

您可以使用此后缀为精确匹配使用其他分析方法。请参阅[将精确搜索与词干混合在一起](https://www.elastic.co/guide/en/elasticsearch/reference/master/mixing-exact-search-with-stemming.html)。

**`rewrite`**

(Optional, string) Method used to rewrite the query. For valid values and more information, see the [`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html).

（可选，字符串）用于重写查询的方法。有关有效值和更多信息，请参阅[`rewrite`参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html)。

**`time_zone`**

(Optional, string) [Coordinated Universal Time (UTC) offset](https://en.wikipedia.org/wiki/List_of_UTC_time_offsets) or [IANA time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) used to convert `date` values in the query string to UTC.

（可选，字符串） 用于将查询字符串中的值转换为UTC的[协调世界时（UTC）偏移量](https://en.wikipedia.org/wiki/List_of_UTC_time_offsets)或 [IANA时区](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)`date`。

Valid values are ISO 8601 UTC offsets, such as `+01:00` or -`08:00`, and IANA time zone IDs, such as `America/Los_Angeles`.

有效值是ISO 8601 UTC偏移，如`+01:00`或-`08:00`和IANA时区的ID，如`America/Los_Angeles`。

> **NOTE:** The `time_zone` parameter does **not** affect the [date math](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math) value of `now`. `now` is always the current system time in UTC. However, the `time_zone` parameter does convert dates calculated using `now` and [date math rounding](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math). For example, the `time_zone` parameter will convert a value of `now/d`.
>
> 该`time_zone`参数不**不**影响[日期数学](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)的价值`now`。`now`始终是UTC的当前系统时间。但是，该 `time_zone`参数会转换使用`now`和 [日期数学取整](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)计算的[日期](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)。例如，该`time_zone`参数将转换为的值`now/d`。

###  Notes

####  Query string syntax

The query string “mini-language” is used by the [Query string](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html) and by the `q` query string parameter in the [`search` API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html).

应用于查询字符串的的“迷你语言”是在 [`search` API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)中使用`q`查询参数。

The query string is parsed into a series of *terms* and *operators*. A term can be a single word — `quick` or `brown` — or a phrase, surrounded by double quotes — `"quick brown"` — which searches for all the words in the phrase, in the same order.

查询字符串被解析为一系列*术语*和*运算符*。一个术语可以是一个字- `quick`或`brown` -或短语，用双引号包围-  `"quick brown"` -其中搜索一语中的所有单词，以相同的顺序。

Operators allow you to customize the search — the available options are explained below.

运算符允许您自定义搜索-可用选项如下所述。

####  Field names

You can specify fields to search in the query syntax:

您可以在查询语法中指定要搜索的字段：

- where the `status` field contains `active`

  `status`字段包含`active`

  ```
  status:active
  ```

- where the `title` field contains `quick` or `brown`

  `title`字段包含`quick`或`brown`

  ```
  title:(quick OR brown)
  ```

- where the `author` field contains the exact phrase `"john smith"`

  `author`字段包含`"john smith"`的抽取解析

  ```
  author:"John Smith"
  ```

- where the `first name` field contains `Alice` (note how we need to escape the space with a backslash)

  `first name`字段包含`Alice`（请注意我们需要如何使用反斜杠来转义空格）

  ```
  first\ name:Alice
  ```

- where any of the fields `book.title`, `book.content` or `book.date` contains `quick` or `brown` (note how we need to escape the `*` with a backslash):

  其中任何字段book.title，book.content或book.date包含 quick或brown（请注意我们需要如何*使用反斜杠将其转义）：

  ```
  book.\*:(quick OR brown)
  ```

- where the field `title` has any non-null value:

  其中该字段title具有任何非空值：

  ```
  _exists_:title
  ```

####  Wildcards

Wildcard searches can be run on individual terms, using `?` to replace a single character, and `*` to replace zero or more characters:

通配符搜索可以在单个条件下运行，`?`用于替换单个字符，以及`*`替换零个或多个字符：

```
qu?ck bro*
```

Be aware that wildcard queries can use an enormous amount of memory and perform very badly — just think how many terms need to be queried to match the query string `"a* b* c*"`.

请注意，通配符查询会占用大量内存，并且执行效果非常差-只需考虑要查询多少项以匹配查询字符串即可`"a* b* c*"`。

> **WARNING:** Pure wildcards `\*` are rewritten to [`exists`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-exists-query.html) queries for efficiency. As a consequence, the wildcard `"field:*"` would match documents with an empty value like the following:
>
> 纯通配符`\*`将重写为[`exists`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-exists-query.html)查询以提高效率。结果，通配符`"field:*"`将匹配具有空值的文档，如下所示：
>
> ```
> {
>   "field": ""
> }
> ```
>
> ... and would **not** match if the field is missing or set with an explicit null value like the following:
>
> ...，并且如果该字段丢失或设置为显式null值（例如，如下所示），则将**不**匹配：
>
> ```
> {
>   "field": null
> }
> ```

> **WARNING:** Allowing a wildcard at the beginning of a word (eg `"*ing"`) is particularly heavy, because all terms in the index need to be examined, just in case they match. Leading wildcards can be disabled by setting `allow_leading_wildcard` to `false`.
>
> 在单词（例如`"*ing"`）的开头允许使用通配符特别繁重，因为索引中的所有术语都需要进行检查，以防万一它们匹配。设置`allow_leading_wildcard`为可以禁用前导通配符 `false`。

Only parts of the analysis chain that operate at the character level are applied. So for instance, if the analyzer performs both lowercasing and stemming, only the lowercasing will be applied: it would be wrong to perform stemming on a word that is missing some of its letters.

仅应用在字符级别上运行的分析链的各个部分。因此，例如，如果分析器同时执行小写和词干分析，则仅应用小写：对缺少某些字母的单词执行词干分析是错误的。

By setting `analyze_wildcard` to true, queries that end with a `*` will be analyzed and a boolean query will be built out of the different tokens, by ensuring exact matches on the first N-1 tokens, and prefix match on the last token.

通过设置`analyze_wildcard`为true，`*`将分析以a结尾的查询，并通过确保前N-1个标记的完全匹配以及最后一个标记的前缀匹配，从不同的标记中构建一个布尔查询。

####  Regular expressions

Regular expression patterns can be embedded in the query string by wrapping them in forward-slashes (`"/"`):

通过将正则表达式模式包装在正斜杠（`"/"`）中，可以将其嵌入查询字符串中：

```
name:/joh?n(ath[oa]n)/
```

The supported regular expression syntax is explained in [*Regular expression syntax*](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html).

支持的正则表达式语法在[*正*](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html)则表达式语法中进行了说明。

> **WARNING:** 
> The `allow_leading_wildcard` parameter does not have any control over regular expressions. A query string such as the following would force Elasticsearch to visit every term in the index:
>
> 该`allow_leading_wildcard`参数对正则表达式没有任何控制。如下所示的查询字符串将迫使Elasticsearch访问索引中的每个术语：
>
> ```
> /.*n/
> ```
>
> Use with caution!
>
> 请谨慎使用！

####  Fuzziness

We can search for terms that are similar to, but not exactly like our search terms, using the “fuzzy” operator:

我们可以使用“模糊”运算符来搜索与我们的搜索词相似但不完全相同的词：

```
quikc~ brwn~ foks~
```

This uses the [Damerau-Levenshtein distance](https://en.wikipedia.org/wiki/Damerau-Levenshtein_distance) to find all terms with a maximum of two changes, where a change is the insertion, deletion or substitution of a single character, or transposition of two adjacent characters.

这使用 [Damerau-Levenshtein距离](https://en.wikipedia.org/wiki/Damerau-Levenshtein_distance) 查找最大两个变化的所有术语，其中变化是单个字符的插入，删除或替换，或者两个相邻字符的转置。

The default *edit distance* is `2`, but an edit distance of `1` should be sufficient to catch 80% of all human misspellings. It can be specified as:

默认的*编辑距离*是`2`，但是编辑距离`1`应该足以捕捉所有人类拼写错误的80％。可以指定为：

```
quikc~1
```

> **WARNING:**  Avoid mixing fuzziness with wildcards
>
> Mixing [fuzzy](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness) and [wildcard](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-wildcard) operators is *not* supported. When mixed, one of the operators is not applied. For example, you can search for `app~1` (fuzzy) or `app*` (wildcard), but searches for `app*~1` do not apply the fuzzy operator (`~1`).
>
> *不*支持混合使用[模糊](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness)运算符 和[通配符](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html#query-string-wildcard)。混合使用时，不会应用其中一个运算符。例如，您可以搜索`app~1`（模糊）或`app*`（通配符），但是搜索 `app*~1`不应用模糊运算符（`~1`）。

####  Proximity searches

While a phrase query (eg `"john smith"`) expects all of the terms in exactly the same order, a proximity query allows the specified words to be further apart or in a different order. In the same way that fuzzy queries can specify a maximum edit distance for characters in a word, a proximity search allows us to specify a maximum edit distance of words in a phrase:

词组查询（例如`"john smith"`）期望所有术语都以完全相同的顺序进行，而邻近查询则允许指定的单词进一步分开或以不同的顺序进行。与模糊查询可以为单词中的字符指定最大编辑距离的方式相同，邻近搜索使我们可以指定短语中单词的最大编辑距离：

```
"fox quick"~5
```

The closer the text in a field is to the original order specified in the query string, the more relevant that document is considered to be. When compared to the above example query, the phrase `"quick fox"` would be considered more relevant than `"quick brown fox"`.

字段中的文本越接近查询字符串中指定的原始顺序，则该文档被认为越相关。与上述示例查询相比，该短语`"quick fox"`会被认为比更为相关`"quick brown fox"`。

####  Ranges

Ranges can be specified for date, numeric or string fields. Inclusive ranges are specified with square brackets `[min TO max]` and exclusive ranges with curly brackets `{min TO max}`.

可以为日期，数字或字符串字段指定范围。包含范围用方括号指定，`[min TO max]`排除范围用大括号指定`{min TO max}`。

- All days in 2012:

  ```
  date:[2012-01-01 TO 2012-12-31]
  ```

- Numbers 1..5

  ```
  count:[1 TO 5]
  ```

- Tags between `alpha` and `omega`, excluding `alpha` and `omega`:

  ```
  tag:{alpha TO omega}
  ```

- Numbers from 10 upwards

  ```
  count:[10 TO *]
  ```

- Dates before 2012

  ```
  date:{* TO 2012-01-01}
  ```

Curly and square brackets can be combined:

弯括号和方括号可以组合使用：

- Numbers from 1 up to but not including 5

  ```
  count:[1 TO 5}
  ```

Ranges with one side unbounded can use the following syntax:

一侧无界的范围可以使用以下语法：

```
age:>10
age:>=10
age:<10
age:<=10
```

> **NOTE:** To combine an upper and lower bound with the simplified syntax, you would need to join two clauses with an `AND` operator:
>
> 要将上限和下限与简化语法结合在一起，您需要使用`AND`运算符将两个子句连接起来：
>
> ```
> age:(>=10 AND <20)
> age:(+>=10 +<20)
> ```

The parsing of ranges in query strings can be complex and error prone. It is much more reliable to use an explicit [`range` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html).

查询字符串中范围的解析可能很复杂并且容易出错。使用显式[`range`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html)要可靠得多。

####  Boosting

Use the *boost* operator `^` to make one term more relevant than another. For instance, if we want to find all documents about foxes, but we are especially interested in quick foxes:

使用*boost*运算符`^`可使一个术语比另一个术语更相关。例如，如果我们想查找有关狐狸的所有文档，但是我们对速狐特别感兴趣：

```
quick^2 fox
```

The default `boost` value is 1, but can be any positive floating point number. Boosts between 0 and 1 reduce relevance.

默认`boost`值为1，但可以是任何正浮点数。介于0和1之间的提升会降低相关性。

Boosts can also be applied to phrases or to groups:

升压还可以应用于短语或组：

```
"john smith"^2   (foo bar)^4
```

####  Boolean operators

By default, all terms are optional, as long as one term matches. A search for `foo bar baz` will find any document that contains one or more of `foo` or `bar` or `baz`. We have already discussed the `default_operator` above which allows you to force all terms to be required, but there are also *boolean operators* which can be used in the query string itself to provide more control.

默认情况下，所有条件都是可选的，只要一个条件匹配即可。搜索`foo bar baz`将找到包含一个或多个`foo`或`bar`或的任何文档 `baz`。我们已经讨论了`default_operator` 上面的内容，上面的内容允许您强制要求所有条件，但是还可以在查询字符串本身中使用*布尔运算符*来提供更多控制。

The preferred operators are `+` (this term **must** be present) and `-` (this term **must not** be present). All other terms are optional. For example, this query:

首选运算符为`+`（**必须**存在此术语）和 `-` （**不得**存在此术语）。所有其他术语都是可选的。例如，此查询：

```
quick brown +fox -news
```

states that:

- `fox` must be present

  `fox` 必须存在

- `news` must not be present

  `news` 一定不能出现

- `quick` and `brown` are optional — their presence increases the relevance

  `quick`并且`brown`是可选的-它们的存在增加了相关性

The familiar boolean operators `AND`, `OR` and `NOT` (also written `&&`, `||` and `!`) are also supported but beware that they do not honor the usual precedence rules, so parentheses should be used whenever multiple operators are used together. For instance the previous query could be rewritten as:

熟悉的布尔运算符`AND`，`OR`以及`NOT`（也写作`&&`，`||` 和`!`）也支持，但要小心，他们不遵守通常的优先级规则，所以每当多个运营商一起使用时，应使用括号。例如，先前的查询可以重写为：

- **`((quick AND fox) OR (brown AND fox) OR fox) AND NOT news`**

  This form now replicates the logic from the original query correctly, but the relevance scoring bears little resemblance to the original.

  现在，此表单可以正确地复制原始查询中的逻辑，但是相关性评分与原始查询几乎没有相似之处。

In contrast, the same query rewritten using the [`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html) would look like this:

相反，使用该[`match`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)重写的同一[查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html) 将如下所示：

```
{
    "bool": {
        "must":     { "match": "fox"         },
        "should":   { "match": "quick brown" },
        "must_not": { "match": "news"        }
    }
}
```

####  Grouping

Multiple terms or clauses can be grouped together with parentheses, to form sub-queries:

多个术语或子句可以与括号组合在一起，以形成子查询：

```
(quick OR brown) AND fox
```

Groups can be used to target a particular field, or to boost the result of a sub-query:

组可用于定位特定字段或提升子查询的结果：

```
status:(active OR pending) title:(full text search)^2
```

####  Reserved characters

If you need to use any of the characters which function as operators in your query itself (and not as operators), then you should escape them with a leading backslash. For instance, to search for `(1+1)=2`, you would need to write your query as `\(1\+1\)\=2`. When using JSON for the request body, two preceding backslashes (`\\`) are required; the backslash is a reserved escaping character in JSON strings.

如果您需要使用在查询本身中充当运算符的任何字符（而不是运算符），则应使用反斜杠将其转义。例如，要进行搜索`(1+1)=2`，您需要将查询写为`\(1\+1\)\=2`。在请求正文中使用JSON时，必须使用两个前面的反斜杠（`\\`）；反斜杠是JSON字符串中的保留转义字符。

```console
GET /my-index-000001/_search
{
  "query" : {
    "query_string" : {
      "query" : "kimchy\\!",
      "fields"  : ["user.id"]
    }
  }
}
```

The reserved characters are: `+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /`

保留字符为： `+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /`

Failing to escape these special characters correctly could lead to a syntax error which prevents your query from running.

未能正确转义这些特殊字符可能会导致语法错误，从而阻止查询运行。

> **NOTE:** `<` and `>` can’t be escaped at all. The only way to prevent them from attempting to create a range query is to remove them from the query string entirely.
>
> `<`and`>`根本无法逃脱。阻止它们尝试创建范围查询的唯一方法是将它们从查询字符串中完全删除。

####  Whitespaces and empty queries

Whitespace is not considered an operator.

空白不被视为运算符。

If the query string is empty or only contains whitespaces the query will yield an empty result set.

如果查询字符串为空或仅包含空格，则查询将产生一个空结果集。

####  Avoid using the `query_string` query for nested documents

`query_string` searches do not return [nested](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html) documents. To search nested documents, use the [`nested` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-nested-query.html).

`query_string`搜索不会返回[嵌套](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html)文档。要搜索嵌套文档，请使用[`nested`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-nested-query.html)。

####  Search multiple fields

You can use the `fields` parameter to perform a `query_string` search across multiple fields.

您可以使用该`fields`参数在多个字段中执行`query_string`搜索。

The idea of running the `query_string` query against multiple fields is to expand each query term to an OR clause like this:

`query_string`针对多个字段运行查询的想法是将每个查询词扩展为一个OR子句，如下所示：

```
field1:query_term OR field2:query_term | ...
```

For example, the following query

例如下面的查询

```console
GET /_search
{
  "query": {
    "query_string": {
      "fields": [ "content", "name" ],
      "query": "this AND that"
    }
  }
}
```

matches the same words as

和下述表达的意思是相同的

```console
GET /_search
{
  "query": {
    "query_string": {
      "query": "(content:this OR name:this) AND (content:that OR name:that)"
    }
  }
}
```

Since several queries are generated from the individual search terms, combining them is automatically done using a `dis_max` query with a `tie_breaker`. For example (the `name` is boosted by 5 using `^5` notation):

由于是根据各个搜索字词生成的几个查询，因此使用带有的`dis_max`查询会自动完成对它们的合并`tie_breaker`。例如（`name`使用`^5`表示法将其增加5 ）：

```console
GET /_search
{
  "query": {
    "query_string" : {
      "fields" : ["content", "name^5"],
      "query" : "this AND that OR thus",
      "tie_breaker" : 0
    }
  }
}
```

Simple wildcard can also be used to search "within" specific inner elements of the document. For example, if we have a `city` object with several fields (or inner object with fields) in it, we can automatically search on all "city" fields:

简单的通配符也可以用于在文档的特定内部元素中“搜索”。例如，如果我们有一个`city`包含多个字段的对象（或内部带有字段的对象），则可以自动搜索所有“城市”字段：

```console
GET /_search
{
  "query": {
    "query_string" : {
      "fields" : ["city.*"],
      "query" : "this AND that OR thus"
    }
  }
}
```

Another option is to provide the wildcard fields search in the query string itself (properly escaping the `*` sign), for example: `city.\*:something`:

另一种选择是在查询字符串本身中提供通配符字段搜索（正确地转义`*`符号），例如 `city.\*:something`：

```console
GET /_search
{
  "query": {
    "query_string" : {
      "query" : "city.\\*:(this AND that OR thus)"
    }
  }
}
```

> **NOTE:** Since `\` (backslash) is a special character in json strings, it needs to be escaped, hence the two backslashes in the above `query_string`.
>
> 由于`\`（反斜杠）是json字符串中的特殊字符，因此需要对其进行转义，因此上述两个反斜杠`query_string`。

The fields parameter can also include pattern based field names, allowing to automatically expand to the relevant fields (dynamically introduced fields included). For example:

fields参数还可以包括基于模式的字段名称，从而可以自动扩展到相关字段（包括动态引入的字段）。例如：

```console
GET /_search
{
  "query": {
    "query_string" : {
      "fields" : ["content", "name.*^5"],
      "query" : "this AND that OR thus"
    }
  }
}
```

#### Additional parameters for multiple field searches

When running the `query_string` query against multiple fields, the following additional parameters are supported.

当`query_string`对多个字段运行查询时，支持以下附加参数。

**`type`**

(Optional, string) Determines how the query matches and scores documents. Valid values are:

（可选，字符串）确定查询如何匹配和评分文档。有效值为：

- **`best_fields` (Default)**

  Finds documents which match any field and uses the highest [`_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) from any matching field. See [`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-best-fields).

  查找与任何字段匹配的文档，并在[`_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)任何匹配的字段中使用最高的文档 。请参阅 [`best_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-best-fields)。

- **`bool_prefix`**

  Creates a `match_bool_prefix` query on each field and combines the `_score` from each field. See [`bool_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-bool-prefix).

  `match_bool_prefix`在每个字段上 创建查询，并将`_score`每个字段中的合并。请参阅[`bool_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-bool-prefix)。

- **`cross_fields`**

  Treats fields with the same `analyzer` as though they were one big field. Looks for each word in **any** field. See [`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-cross-fields).

  `analyzer`就像 对待一个大字段一样对待字段。在**任何**字段中查找每个单词。请参阅[`cross_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-cross-fields)。

- **`most_fields`**

  Finds documents which match any field and combines the `_score` from each field. See [`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-most-fields).

  查找与任何字段匹配的文档，并将`_score`每个字段中的合并。请参阅[`most_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-most-fields)。

- **`phrase`**

  Runs a `match_phrase` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-phrase).

  `match_phrase`在每个字段上 运行查询，并使用`_score`最佳字段中的。请参阅[`phrase`和`phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-phrase)。

- **`phrase_prefix`**

  Runs a `match_phrase_prefix` query on each field and uses the `_score` from the best field. See [`phrase` and `phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-phrase).

  `match_phrase_prefix`在每个字段上 运行查询，并使用`_score`最佳字段中的。请参阅[`phrase`和`phrase_prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#type-phrase)。

NOTE: Additional top-level `multi_match` parameters may be available based on the [`type`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#multi-match-types) value.

注意：`multi_match`根据该[`type`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-match-query.html#multi-match-types)值，可能还有其他顶级参数 。

####  Synonyms and the `query_string` query

The `query_string` query supports multi-terms synonym expansion with the [synonym_graph](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-graph-tokenfilter.html) token filter. When this filter is used, the parser creates a phrase query for each multi-terms synonyms. For example, the following synonym: `ny, new york` would produce:

该`query_string`查询通过[synonym_graph](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-synonym-graph-tokenfilter.html)令牌过滤器支持多词同义词扩展。使用此过滤器时，解析器为每个多词同义词创建短语查询。例如，以下同义词：`ny, new york`将产生：

```
(ny OR ("new york"))
```

It is also possible to match multi terms synonyms with conjunctions instead:

也可以用连接词来匹配多词同义词：

```console
GET /_search
{
   "query": {
       "query_string" : {
           "default_field": "title",
           "query" : "ny city",
           "auto_generate_synonyms_phrase_query" : false
       }
   }
}
```

The example above creates a boolean query:

上面的示例创建一个布尔查询：

```
(ny OR (new AND york)) city
```

that matches documents with the term `ny` or the conjunction `new AND york`. By default the parameter`auto_generate_synonyms_phrase_query` is set to `true`.

与具有术语`ny`或连词的文档相匹配`new AND york`。默认情况下，参数`auto_generate_synonyms_phrase_query`设置为`true`。

####  How `minimum_should_match` works

The `query_string` splits the query around each operator to create a boolean query for the entire input. You can use `minimum_should_match` to control how many "should" clauses in the resulting query should match.

该`query_string`拆分各地各运营商查询，以创建整个输入一个布尔查询。您可以`minimum_should_match`用来控制结果查询中应匹配多少个“应该”子句。

```console
GET /_search
{
  "query": {
    "query_string": {
      "fields": [
        "title"
      ],
      "query": "this that thus",
      "minimum_should_match": 2
    }
  }
}
```

The example above creates a boolean query:

上面的示例创建一个布尔查询：

```
(title:this title:that title:thus)~2
```

that matches documents with at least two of the terms `this`, `that` or `thus` in the single field `title`.

与至少两个术语匹配的文档`this`，`that`或`thus` 在单个字段中匹配的文档`title`。

####  How `minimum_should_match` works for multiple fields

```console
GET /_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "content"
      ],
      "query": "this that thus",
      "minimum_should_match": 2
    }
  }
}
```

The example above creates a boolean query:

上面的示例创建一个布尔查询：

```
((content:this content:that content:thus) | (title:this title:that title:thus))
```

that matches documents with the disjunction max over the fields `title` and `content`. Here the `minimum_should_match` parameter can’t be applied.

在字段`title`和 `content`上将文档与相加最大值相匹配的文档进行匹配。在此`minimum_should_match`参数无法应用。

```console
GET /_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "content"
      ],
      "query": "this OR that OR thus",
      "minimum_should_match": 2
    }
  }
}
```

Adding explicit operators forces each term to be considered as a separate clause.

添加显式运算符会强制将每个术语视为一个单独的子句。

The example above creates a boolean query:

上面的示例创建一个布尔查询：

```
((content:this | title:this) (content:that | title:that) (content:thus | title:thus))~2
```

that matches documents with at least two of the three "should" clauses, each of them made of the disjunction max over the fields for each term.

与三个“应”子句中的至少两个子句匹配的文档，每个子句都由每个术语的字段上的析取最大值构成。

####  How `minimum_should_match` works for cross-field searches

A `cross_fields` value in the `type` field indicates fields with the same analyzer are grouped together when the input is analyzed.

字段中的`cross_fields`值`type`指示在分析输入时将具有相同分析器的字段组合在一起。

```console
GET /_search
{
  "query": {
    "query_string": {
      "fields": [
        "title",
        "content"
      ],
      "query": "this OR that OR thus",
      "type": "cross_fields",
      "minimum_should_match": 2
    }
  }
}
```

The example above creates a boolean query:

上面的示例创建一个布尔查询：

```
(blended(terms:[field2:this, field1:this]) blended(terms:[field2:that, field1:that]) blended(terms:[field2:thus, field1:thus]))~2
```

that matches documents with at least two of the three per-term blended queries.

与三个词项混合查询中的至少两个匹配的文档。

####  Allow expensive queries

Query string query can be internally be transformed to a [`prefix query`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html) which means that if the prefix queries are disabled as explained [here](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html#prefix-query-allow-expensive-queries) the query will not be executed and an exception will be thrown.

查询字符串查询内部可以被转换成[`prefix query`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html)这意味着如果前缀查询被禁止作为解释[这里](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html#prefix-query-allow-expensive-queries)查询将不会被执行并且会抛出异常。

## Simple query string query

Returns documents based on a provided query string, using a parser with a limited but fault-tolerant syntax.

使用具有有限但容错语法的解析器，根据提供的查询字符串返回文档。

This query uses a [simple syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-syntax) to parse and split the provided query string into terms based on special operators. The query then [analyzes](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) each term independently before returning matching documents.

该查询使用一种[简单的语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-syntax)来解析提供的查询字符串并将其拆分为基于特殊运算符的术语。然后查询在返回匹配的文档之前独立[分析](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)每个术语。

While its syntax is more limited than the [`query_string` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html), the `simple_query_string` query does not return errors for invalid syntax. Instead, it ignores any invalid parts of the query string.

尽管其语法比[`query_string`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html)更受限制 ，但`simple_query_string` 查询不会针对无效语法返回错误。而是，它将忽略查询字符串的任何无效部分。

###  Example request

```json
GET /_search
{
  "query": {
    "simple_query_string" : {
        "query": "\"fried eggs\" +(eggplant | potato) -frittata",
        "fields": ["title^5", "body"],
        "default_operator": "and"
    }
  }
}
```

###  Top-level parameters for `simple_query_string`

**`query`**

(Required, string) Query string you wish to parse and use for search. See [Simple query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-syntax).

（必填，字符串）您要解析并用于搜索的查询字符串。请参阅[简单查询字符串语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-syntax)。

**`fields`**

(Optional, array of strings) Array of fields you wish to search.

（可选，字符串数组）您要搜索的字段数组。

This field accepts wildcard expressions. You also can boost relevance scores for matches to particular fields using a caret (`^`) notation. See [Wildcards and per-field boosts in the `fields` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-boost) for examples.

该字段接受通配符表达式。您也可以使用插入符号（`^`）表示法来提高与特定字段匹配的相关性得分。见 [通配符和每场提升的`fields`参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-boost)的例子。

Defaults to the `index.query.default_field` index setting, which has a default value of `*`. The `*` value extracts all fields that are eligible to term queries and filters the metadata fields. All extracted fields are then combined to build a query if no `prefix` is specified.

默认为`index.query.default_field`索引设置，其默认值为`*`。该`*`值提取符合条件查询的所有字段，并过滤元数据字段。如果未`prefix`指定，则合并所有提取的字段以构建查询。

> **WARNING:** There is a limit on the number of fields that can be queried at once. It is defined by the `indices.query.bool.max_clause_count` [search setting](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html), which defaults to `1024`.
>
> 一次可以查询的字段数有限制。它由`indices.query.bool.max_clause_count` [搜索设置](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html)定义，默认为`1024`。

**`default_operator`**

(Optional, string) Default boolean logic used to interpret text in the query string if no operators are specified. Valid values are:

（可选，字符串）如果未指定运算符，则用于解释查询字符串中的文本的默认布尔逻辑。有效值为：

- **`OR` (Default)**

  For example, a query string of `capital of Hungary` is interpreted as `capital OR of OR Hungary`.

  例如，查询字符串`capital of Hungary`解释为`capital OR of OR Hungary`。

- **`AND`**

  For example, a query string of `capital of Hungary` is interpreted as `capital AND of AND Hungary`.

  例如，查询字符串`capital of Hungary`解释为`capital AND of AND Hungary`。

**`analyze_wildcard`**

(Optional, Boolean) If `true`, the query attempts to analyze wildcard terms in the query string. Defaults to `false`.

（可选，布尔值）如果为`true`，则查询尝试分析查询字符串中的通配符术语。默认为`false`。

**`analyzer`**

(Optional, string) [Analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html) used to convert text in the query string into tokens. Defaults to the [index-time analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/specify-analyzer.html#specify-index-time-analyzer) mapped for the `default_field`. If no analyzer is mapped, the index’s default analyzer is used.

（可选，字符串）[分析器，](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)用于将查询字符串中的文本转换为标记。默认为为映射 的 [索引时间分析器](https://www.elastic.co/guide/en/elasticsearch/reference/master/specify-analyzer.html#specify-index-time-analyzer)`default_field`。如果未映射任何分析器，则使用索引的默认分析器。

**`auto_generate_synonyms_phrase_query`**

(Optional, Boolean) If `true`, the parser creates a [`match_phrase`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html) query for each [multi-position token](https://www.elastic.co/guide/en/elasticsearch/reference/master/token-graphs.html#token-graphs-multi-position-tokens). Defaults to `true`. For examples, see [Multi-position tokens](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-synonyms).

（可选，布尔值）如果为`true`，则解析器[`match_phrase`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html)为每个[多位置标记](https://www.elastic.co/guide/en/elasticsearch/reference/master/token-graphs.html#token-graphs-multi-position-tokens)创建一个 查询 。默认为`true`。有关示例，请参阅[多位置标记](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-synonyms)。

**`flags`**

(Optional, string) List of enabled operators for the [simple query string syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-syntax). Defaults to `ALL` (all operators). See [Limit operators](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#supported-flags) for valid values.

（可选，字符串）为[简单查询字符串语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#simple-query-string-syntax)启用的运算符的列表 。默认为`ALL` （所有运算符）。有关有效值，请参见[限制运算符](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-simple-query-string-query.html#supported-flags)。

**`fuzzy_max_expansions`**

(Optional, integer) Maximum number of terms to which the query expands for fuzzy matching. Defaults to `50`.

（可选，整数）查询为模糊匹配而扩展到的最大术语数。默认为`50`。

**`fuzzy_prefix_length`**

(Optional, integer) Number of beginning characters left unchanged for fuzzy matching. Defaults to `0`.

（可选，整数）为模糊匹配保留的起始字符数。默认为`0`。

**`fuzzy_transpositions`**

(Optional, Boolean) If `true`, edits for fuzzy matching include transpositions of two adjacent characters (ab → ba). Defaults to `true`.

（可选，布尔值）如果`true`为，则模糊匹配的编辑内容包括两个相邻字符的变位（ab→ba）。默认为`true`。

**`lenient`**

(Optional, Boolean) If `true`, format-based errors, such as providing a text value for a [numeric](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) field, are ignored. Defaults to `false`.

（可选，布尔值）如果为`true`，则将忽略基于格式的错误，例如为[数字](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)字段提供文本值。默认为`false`。

**`minimum_should_match`**

(Optional, string) Minimum number of clauses that must match for a document to be returned. See the [`minimum_should_match` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html) for valid values and more information.

（可选，字符串）要返回的文档必须匹配的最小子句数。请参阅[`minimum_should_match` 参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html)以获取有效值和更多信息。

**`quote_field_suffix`**

(Optional, string) Suffix appended to quoted text in the query string.

（可选，字符串）后缀附加到查询字符串中的带引号的文本中。

You can use this suffix to use a different analysis method for exact matches. See [Mixing exact search with stemming](https://www.elastic.co/guide/en/elasticsearch/reference/master/mixing-exact-search-with-stemming.html).

您可以使用此后缀为精确匹配使用其他分析方法。请参阅[将精确搜索与词干混合在一起](https://www.elastic.co/guide/en/elasticsearch/reference/master/mixing-exact-search-with-stemming.html)。

###  Notes

####  Simple query string syntax

The `simple_query_string` query supports the following operators:

该`simple_query_string`查询支持以下运算符：

- `+` signifies AND operation

  `+` 表示与运算

- `|` signifies OR operation

  `|` 表示或运算

- `-` negates a single token

  `-` 取反单个令牌

- `"` wraps a number of tokens to signify a phrase for searching

  `"` 包装许多标记以表示要搜索的短语

- `*` at the end of a term signifies a prefix query

  `*` 字词末尾表示前缀查询

- `(` and `)` signify precedence

  `(`并`)`表示优先

- `~N` after a word signifies edit distance (fuzziness)

  `~N` 单词表示编辑距离（模糊性）后

- `~N` after a phrase signifies slop amount

  `~N` 在短语表示斜线数量之后

To use one of these characters literally, escape it with a preceding backslash (`\`).

要从字面上使用这些字符之一，请使用前面的反斜杠（`\`）将其转义。

The behavior of these operators may differ depending on the `default_operator` value. For example:

这些运算符的行为可能会有所不同，具体取决于`default_operator` 值。例如：

```json
GET /_search
{
  "query": {
    "simple_query_string": {
      "fields": [ "content" ],
      "query": "foo bar -baz"
    }
  }
}
```

This search is intended to only return documents containing `foo` or `bar` that also do **not** contain `baz`. However because of a `default_operator` of `OR`, this search actually returns documents that contain `foo` or `bar` and any documents that don’t contain `baz`. To return documents as intended, change the query string to `foo bar +-baz`.

该搜索的目的是只返回包含`foo`或`bar`**不**包含`baz` 的文档。但是，由于一个`default_operator`的`OR`，这实际上搜索将返回包含文档`foo`或`bar`与不包含任何文件`baz`。要按预期返回文档，请将查询字符串更改为`foo bar +-baz`。

####  Limit operators

You can use the `flags` parameter to limit the supported operators for the simple query string syntax.

您可以使用该`flags`参数来限制简单查询字符串语法的支持运算符。

To explicitly enable only specific operators, use a `|` separator. For example, a `flags` value of `OR|AND|PREFIX` disables all operators except `OR`, `AND`, and `PREFIX`.

要显式仅启用特定的运算符，请使用`|`分隔符。例如，`flags`的值`OR|AND|PREFIX`禁用所有操作符除了`OR`，`AND`和`PREFIX`。

```json
GET /_search
{
  "query": {
    "simple_query_string": {
      "query": "foo | bar + baz*",
      "flags": "OR|AND|PREFIX"
    }
  }
}
```

####  Valid values

The available flags are:

可用的标志是：

- **`ALL` (Default)**

  Enables all optional operators.

  启用所有可选的运算符。

- **`AND`**

  Enables the `+` AND operator.

  启用`+`AND运算符。

- **`ESCAPE`**

  Enables `\` as an escape character.

  启用`\`作为转义字符。

- **`FUZZY`**

  Enables the `~N` operator after a word, where `N` is an integer denoting the allowed edit distance for matching. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness).

  `~N`在一个单词后 启用运算符，其中`N`整数表示允许匹配的编辑距离。请参阅[模糊性](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness)。

- **`NEAR`**

  Enables the `~N` operator, after a phrase where `N` is the maximum number of positions allowed between matching tokens. Synonymous to `SLOP`.

  `~N`在一个短语之后 启用操作符，其中`N`是匹配标记之间允许的最大位置数。的同义词`SLOP`。

- **`NONE`**

  Disables all operators.

  禁用所有运算符。

- **`NOT`**

  Enables the `-` NOT operator.

  启用`-`NOT运算符。

- **`OR`**

  Enables the `\|` OR operator.

  启用“`\|`或”运算符。

- **`PHRASE`**

  Enables the `"` quotes operator used to search for phrases.

  启用`"`用于搜索短语的引号运算符。

- **`PRECEDENCE`**

  Enables the `(` and `)` operators to control operator precedence.

  使`(`和`)`运算符可以控制运算符优先级。

- **`PREFIX`**

  Enables the `*` prefix operator.

  启用`*`前缀运算符。

- **`SLOP`**

  Enables the `~N` operator, after a phrase where `N` is maximum number of positions allowed between matching tokens. Synonymous to `NEAR`.

  在匹配标记之间允许的最大位置数的`~N`短语之后， 启用运算符`N`。的同义词`NEAR`。

- **`WHITESPACE`**

  Enables whitespace as split characters.

  启用空格作为分割字符。

####  Wildcards and per-field boosts in the `fields` parameter

Fields can be specified with wildcards, eg:

字段可以用通配符指定，例如：

```json
GET /_search
{
  "query": {
    "simple_query_string" : {
      "query":    "Will Smith",
      "fields": [ "title", "*_name" ] (1)
    }
  }
}
```

1. Query the `title`, `first_name` and `last_name` fields.

   查询`title`，`first_name`和`last_name`字段。

Individual fields can be boosted with the caret (`^`) notation:

可以使用插入号（`^`）表示法增强各个字段：

```json
GET /_search
{
  "query": {
    "simple_query_string" : {
      "query" : "this is a test",
      "fields" : [ "subject^3", "message" ] (1)
    }
  }
}
```

1.  The `subject` field is three times as important as the `message` field.

    该`subject`领域的重要性是该`message`领域的三倍。

####  Multi-position tokens

By default, the `simple_query_string` query parser creates a [`match_phrase`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html) query for each [multi-position token](https://www.elastic.co/guide/en/elasticsearch/reference/master/token-graphs.html#token-graphs-multi-position-tokens) in the query string. For example, the parser creates a `match_phrase` query for the multi-word synonym `ny, new york`:

默认情况下，`simple_query_string`查询解析器[`match_phrase`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query-phrase.html)为查询字符串中的每个 [多位置标记](https://www.elastic.co/guide/en/elasticsearch/reference/master/token-graphs.html#token-graphs-multi-position-tokens)创建 查询。例如，解析器`match_phrase`为多字同义词创建查询`ny, new york`：

```
(ny OR ("new york"))
```

To match multi-position tokens with an `AND` conjunction instead, set `auto_generate_synonyms_phrase_query` to `false`:

要使用`AND`连词匹配多位置标记，请设置 `auto_generate_synonyms_phrase_query`为`false`：

```json
GET /_search
{
  "query": {
    "simple_query_string": {
      "query": "ny city",
      "auto_generate_synonyms_phrase_query": false
    }
  }
}
```

For the above example, the parser creates the following [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) query:

对于上面的示例，解析器创建以下 [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)查询：

```
(ny OR (new AND york)) city)
```

This `bool` query matches documents with the term `ny` or the conjunction `new AND york`.

此`bool`查询将术语`ny`或连词 与文档匹配`new AND york`。

#  Match all query

The most simple query, which matches all documents, giving them all a `_score` of `1.0`.

```console
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

The `_score` can be changed with the `boost` parameter:

```console
GET /_search
{
  "query": {
    "match_all": { "boost" : 1.2 }
  }
}
```

### Match None Query

This is the inverse of the `match_all` query, which matches no documents.

```console
GET /_search
{
  "query": {
    "match_none": {}
  }
}
```

# Span queries

Span queries are low-level positional queries which provide expert control over the order and proximity of the specified terms. These are typically used to implement very specific queries on legal documents or patents.

跨度查询是低级位置查询，它提供对指定术语的顺序和接近度的专家控制。这些通常用于对法律文件或专利进行非常具体的查询。

It is only allowed to set boost on an outer span query. Compound span queries, like span_near, only use the list of matching spans of inner span queries in order to find their own spans, which they then use to produce a score. Scores are never computed on inner span queries, which is the reason why boosts are not allowed: they only influence the way scores are computed, not spans.

只允许在外部跨度查询上设置提升。复合跨度查询，如 span_near，仅使用内部跨度查询的匹配跨度列表来查找自己的跨度，然后使用这些跨度来生成分数。永远不会在内部跨度查询上计算分数，这就是不允许提升的原因：它们只影响分数的计算方式，而不是跨度。

Span queries cannot be mixed with non-span queries (with the exception of the `span_multi` query).

跨度查询不能与非跨度查询混合使用（查询除外`span_multi`）。

The queries in this group are:

该组中的查询是：

- **[`span_containing` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-containing-query.html)**

  Accepts a list of span queries, but only returns those spans which also match a second span query.

  接受一个跨度查询列表，但只返回那些也匹配第二个跨度查询的跨度。

- **[`field_masking_span` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-field-masking-query.html)**

  Allows queries like `span-near` or `span-or` across different fields.

  允许类似`span-near`或`span-or`跨不同字段的查询。

- **[`span_first` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-first-query.html)**

  Accepts another span query whose matches must appear within the first N positions of the field.

  接受另一个 span 查询，其匹配项必须出现在该字段的前 N 个位置。

- **[`span_multi` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-multi-term-query.html)**

  Wraps a [`term`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html), [`range`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html), [`prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html), [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-wildcard-query.html), [`regexp`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-regexp-query.html), or [`fuzzy`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-fuzzy-query.html) query.

  包装[`term`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html), [`range`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html), [`prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html), [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-wildcard-query.html), [`regexp`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-regexp-query.html), 或[`fuzzy`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-fuzzy-query.html)查询。

- **[`span_near` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-near-query.html)**

  Accepts multiple span queries whose matches must be within the specified distance of each other, and possibly in the same order.

  接受多个跨度查询，其匹配项必须在彼此指定的距离内，并且可能以相同的顺序。

- **[`span_not` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-not-query.html)**

  Wraps another span query, and excludes any documents which match that query.

  包装另一个 span 查询，并排除与该查询匹配的任何文档。

- **[`span_or` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-or-query.html)**

  Combines multiple span queries — returns documents which match any of the specified queries.

  组合多个跨度查询——返回匹配任何指定查询的文档。

- **[`span_term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-term-query.html)**

  The equivalent of the [`term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html) but for use with other span queries.

  相当于[`term`查询，](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)但用于其他跨度查询。

- **[`span_within` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-span-within-query.html)**

  The result from a single span query is returned as long is its span falls within the spans returned by a list of other span queries.

  只要它的跨度落在其他跨度查询列表返回的跨度内，就会返回单个跨度查询的结果。

## Span containing query

Returns matches which enclose another span query. The span containing query maps to Lucene `SpanContainingQuery`. Here is an example:

返回包含另一个跨度查询的匹配项。包含查询的跨度映射到 Lucene `SpanContainingQuery`。下面是一个例子：

```console
GET /_search
{
  "query": {
    "span_containing": {
      "little": {
        "span_term": { "field1": "foo" }
      },
      "big": {
        "span_near": {
          "clauses": [
            { "span_term": { "field1": "bar" } },
            { "span_term": { "field1": "baz" } }
          ],
          "slop": 5,
          "in_order": true
        }
      }
    }
  }
}
```

The `big` and `little` clauses can be any span type query. Matching spans from `big` that contain matches from `little` are returned.

该`big`和`little`条款可以是任何的跨越式查询。返回`big`包含匹配项的匹配范围`little`。

## Span field masking query



Wrapper to allow span queries to participate in composite single-field span queries by *lying* about their search field. The span field masking query maps to Lucene’s `SpanFieldMaskingQuery`

通过对搜索字段*撒谎*，允许跨度查询参与复合单字段跨度查询的包装器。span 字段屏蔽查询映射到 Lucene 的`SpanFieldMaskingQuery`

This can be used to support queries like `span-near` or `span-or` across different fields, which is not ordinarily permitted.

这可用于支持类似`span-near`或`span-or`跨不同字段的查询，这通常是不允许的。

Span field masking query is invaluable in conjunction with **multi-fields** when same content is indexed with multiple analyzers. For instance we could index a field with the standard analyzer which breaks text up into words, and again with the english analyzer which stems words into their root form.

当使用多个分析器索引相同的内容时，跨字段屏蔽查询与**多字段**结合使用是非常宝贵的。例如，我们可以使用将文本分解为单词的标准分析器来索引字段，并再次使用将单词分解为词根形式的英语分析器来索引字段。

Example:

例子：

```console
GET /_search
{
  "query": {
    "span_near": {
      "clauses": [
        {
          "span_term": {
            "text": "quick brown"
          }
        },
        {
          "field_masking_span": {
            "query": {
              "span_term": {
                "text.stems": "fox"
              }
            },
            "field": "text"
          }
        }
      ],
      "slop": 5,
      "in_order": false
    }
  }
}
```

Note: as span field masking query returns the masked field, scoring will be done using the norms of the field name supplied. This may lead to unexpected scoring behaviour.

注意：由于 span 字段屏蔽查询返回被屏蔽的字段，因此将使用提供的字段名称的规范进行评分。这可能会导致意外的得分行为。

## Span first query

Matches spans near the beginning of a field. The span first query maps to Lucene `SpanFirstQuery`. Here is an example:

匹配跨越字段的开头附近。跨度优先查询映射到 Lucene `SpanFirstQuery`。下面是一个例子：

```console
GET /_search
{
  "query": {
    "span_first": {
      "match": {
        "span_term": { "user.id": "kimchy" }
      },
      "end": 3
    }
  }
}
```

The `match` clause can be any other span type query. The `end` controls the maximum end position permitted in a match.

该`match`子句可以是任何其他跨度类型查询。的`end`控制的最大端部位置在一个匹配允许的。

##  Span multi-term query

The `span_multi` query allows you to wrap a `multi term query` (one of wildcard, fuzzy, prefix, range or regexp query) as a `span query`, so it can be nested. Example:

该`span_multi`查询允许您将 a `multi term query`（通配符、模糊、前缀、范围或正则表达式查询之一）包装为 a `span query`，因此它可以嵌套。例子：

```console
GET /_search
{
  "query": {
    "span_multi": {
      "match": {
        "prefix": { "user.id": { "value": "ki" } }
      }
    }
  }
}
```

A boost can also be associated with the query:

提升也可以与查询相关联：

```console
GET /_search
{
  "query": {
    "span_multi": {
      "match": {
        "prefix": { "user.id": { "value": "ki", "boost": 1.08 } }
      }
    }
  }
}
```

> **WARNING:** `span_multi` queries will hit too many clauses failure if the number of terms that match the query exceeds the boolean query limit (defaults to 1024).To avoid an unbounded expansion you can set the [rewrite method](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html) of the multi term query to `top_terms_*` rewrite. Or, if you use `span_multi` on `prefix` query only, you can activate the [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html) field option of the `text` field instead. This will rewrite any prefix query on the field to a single term query that matches the indexed prefix.
>
> `span_multi`如果与查询匹配的词条数超过布尔查询限制（默认为 1024），则查询将命中太多子句失败。为了避免无限扩展，您可以将多词条查询的[重写方法](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html)设置为`top_terms_*`重写。或者，如果您仅使用`span_multi`on `prefix`query，则可以改为激活[`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html)字段的字段选项`text`。这会将字段上的任何前缀查询重写为与索引前缀匹配的单个术语查询。

##  Span near query

Matches spans which are near one another. One can specify *slop*, the maximum number of intervening unmatched positions, as well as whether matches are required to be in-order. The span near query maps to Lucene `SpanNearQuery`. Here is an example:

匹配彼此接近的跨度。可以指定*slop*，插入不匹配位置的最大数量，以及是否需要按顺序匹配。查询附近的跨度映射到 Lucene `SpanNearQuery`。下面是一个例子：

```console
GET /_search
{
  "query": {
    "span_near": {
      "clauses": [
        { "span_term": { "field": "value1" } },
        { "span_term": { "field": "value2" } },
        { "span_term": { "field": "value3" } }
      ],
      "slop": 12,
      "in_order": false
    }
  }
}
```

The `clauses` element is a list of one or more other span type queries and the `slop` controls the maximum number of intervening unmatched positions permitted.

该`clauses`元素是一个或多个其他跨度类型查询的列表，并`slop`控制允许的最大干预不匹配位置数。

##  Span not query

Removes matches which overlap with another span query or which are within x tokens before (controlled by the parameter `pre`) or y tokens after (controlled by the parameter `post`) another SpanQuery. The span not query maps to Lucene `SpanNotQuery`. Here is an example:

删除与另一个跨度查询重叠的匹配项，或者在另一个 SpanQuery之前（由参数控制`pre`）或之后（由参数控制`post`）的x 个标记内的匹配项。span not query 映射到 Lucene `SpanNotQuery`。下面是一个例子：

```console
GET /_search
{
  "query": {
    "span_not": {
      "include": {
        "span_term": { "field1": "hoya" }
      },
      "exclude": {
        "span_near": {
          "clauses": [
            { "span_term": { "field1": "la" } },
            { "span_term": { "field1": "hoya" } }
          ],
          "slop": 0,
          "in_order": true
        }
      }
    }
  }
}
```

The `include` and `exclude` clauses can be any span type query. The `include` clause is the span query whose matches are filtered, and the `exclude` clause is the span query whose matches must not overlap those returned.

该`include`和`exclude`条款可以是任何的跨越式查询。该 `include`条款是跨度查询，其匹配被过滤，并在 `exclude`从句是跨度查询其比赛不能重叠的返回。

In the above example all documents with the term hoya are filtered except the ones that have *la* preceding them.

在上面的例子中，所有带有 hoya 的文档都被过滤掉了，除了它们前面有*la 的*那些。

Other top level options:

其他顶级选项：

| `pre`  | If set the amount of tokens before the include span can’t have overlap with the exclude span. Defaults to 0.<br />如果在包含跨度之前设置令牌数量不能与排除跨度重叠。默认为 0。 |
| ------ | ------------------------------------------------------------ |
| `post` | If set the amount of tokens after the include span can’t have overlap with the exclude span. Defaults to 0.<br />如果设置包含跨度后的令牌数量不能与排除跨度重叠。默认为 0。 |
| `dist` | If set the amount of tokens from within the include span can’t have overlap with the exclude span. Equivalent of setting both `pre` and `post`.<br />如果设置包含范围内的令牌数量不能与排除范围重叠。相当于同时设置`pre`和`post`。 |

## Span or query

Matches the union of its span clauses. The span or query maps to Lucene `SpanOrQuery`. Here is an example:

匹配其跨度子句的并集。跨度或查询映射到 Lucene `SpanOrQuery`。下面是一个例子：

```console
GET /_search
{
  "query": {
    "span_or" : {
      "clauses" : [
        { "span_term" : { "field" : "value1" } },
        { "span_term" : { "field" : "value2" } },
        { "span_term" : { "field" : "value3" } }
      ]
    }
  }
}
```

The `clauses` element is a list of one or more other span type queries.

该`clauses`元素是一个或多个其他跨度类型查询的列表。

##  Span term query

Matches spans containing a term. The span term query maps to Lucene `SpanTermQuery`. Here is an example:

匹配包含术语的跨度。span 术语查询映射到 Lucene `SpanTermQuery`。下面是一个例子：

```console
GET /_search
{
  "query": {
    "span_term" : { "user.id" : "kimchy" }
  }
}
```

A boost can also be associated with the query:

提升也可以与查询相关联：

```console
GET /_search
{
  "query": {
    "span_term" : { "user.id" : { "value" : "kimchy", "boost" : 2.0 } }
  }
}
```

Or :

或者 ：

```console
GET /_search
{
  "query": {
    "span_term" : { "user.id" : { "term" : "kimchy", "boost" : 2.0 } }
  }
}
```

##  Span within query

Returns matches which are enclosed inside another span query. The span within query maps to Lucene `SpanWithinQuery`. Here is an example:

返回包含在另一个跨度查询中的匹配项。查询中的跨度映射到 Lucene `SpanWithinQuery`。下面是一个例子：

```console
GET /_search
{
  "query": {
    "span_within": {
      "little": {
        "span_term": { "field1": "foo" }
      },
      "big": {
        "span_near": {
          "clauses": [
            { "span_term": { "field1": "bar" } },
            { "span_term": { "field1": "baz" } }
          ],
          "slop": 5,
          "in_order": true
        }
      }
    }
  }
}
```

The `big` and `little` clauses can be any span type query. Matching spans from `little` that are enclosed within `big` are returned.

该`big`和`little`条款可以是任何的跨越式查询。返回其中`little`包含的匹配范围`big`。



# Specialized queries

This group contains queries which do not fit into the other groups:

该组包含不适合其他组的查询：

- **[`distance_feature` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-distance-feature-query.html)**

  A query that computes scores based on the dynamically computed distances between the origin and documents' date, date_nanos and geo_point fields. It is able to efficiently skip non-competitive hits.

  根据原点和文档的日期、date_nanos 和 geo_point 字段之间动态计算的距离计算分数的查询。它能够有效地跳过非竞争性命中。

- **[`more_like_this` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-mlt-query.html)**

  This query finds documents which are similar to the specified text, document, or collection of documents.

  此查询查找与指定文本、文档或文档集合相似的文档。

- **[`percolate` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-percolate-query.html)**

  This query finds queries that are stored as documents that match with the specified document.

  此查询查找存储为与指定文档匹配的文档的查询。

- **[`rank_feature` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html)**

  A query that computes scores based on the values of numeric features and is able to efficiently skip non-competitive hits.

  基于数字特征的值计算分数的查询，并能够有效地跳过非竞争性命中。

- **[`script` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-query.html)**

  This query allows a script to act as a filter. Also see the [`function_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html).

  此查询允许脚本充当过滤器。另请参阅 [`function_score`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html)。

- **[`script_score` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html)**

  A query that allows to modify the score of a sub-query with a script.

  允许使用脚本修改子查询分数的查询。

- **[`wrapper` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-wrapper-query.html)**

  A query that accepts other queries as json or yaml string.

  接受其他查询作为 json 或 yaml 字符串的查询。

- **[`pinned` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-pinned-query.html)**

  A query that promotes selected documents over others matching a given query.

  将所选文档提升到与给定查询匹配的其他文档的查询。

## Distance feature query

Boosts the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of documents closer to a provided `origin` date or point. For example, you can use this query to give more weight to documents closer to a certain date or location.

提高更接近提供日期或点的文档的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)`origin`。例如，您可以使用此查询为更接近某个日期或位置的文档赋予更多权重。

You can use the `distance_feature` query to find the nearest neighbors to a location. You can also use the query in a [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) search’s `should` filter to add boosted relevance scores to the `bool` query’s scores.

您可以使用`distance_feature`查询来查找某个位置的最近邻居。您还可以在[`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) 搜索`should`过滤器中使用查询来将提升的相关性分数添加到`bool`查询的分数中。

### Example request

####  Index setup

To use the `distance_feature` query, your index must include a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html), [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) or [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) field.

要使用`distance_feature`查询，您的索引必须包含[`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html), [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html)或[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)字段。

To see how you can set up an index for the `distance_feature` query, try the following example.

要了解如何为`distance_feature`查询设置索引，请尝试以下示例。

1. Create an `items` index with the following field mapping:

   `items`使用以下字段映射创建索引：

   - `name`, a [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html) field

     `name`，一个[`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html)字段

   - `production_date`, a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html) field

     `production_date`，一个[`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html)字段

   - `location`, a [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) field

     `location`，一个[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)字段

   ```console
   PUT /items
   {
     "mappings": {
       "properties": {
         "name": {
           "type": "keyword"
         },
         "production_date": {
           "type": "date"
         },
         "location": {
           "type": "geo_point"
         }
       }
     }
   }
   ```

2. Index several documents to this index.

   将多个文档索引到此索引。

   ```console
   PUT /items/_doc/1?refresh
   {
     "name" : "chocolate",
     "production_date": "2018-02-01",
     "location": [-71.34, 41.12]
   }
   
   PUT /items/_doc/2?refresh
   {
     "name" : "chocolate",
     "production_date": "2018-01-01",
     "location": [-71.3, 41.15]
   }
   
   
   PUT /items/_doc/3?refresh
   {
     "name" : "chocolate",
     "production_date": "2017-12-01",
     "location": [-71.3, 41.12]
   }
   ```

###  Example queries

####  Boost documents based on date

The following `bool` search returns documents with a `name` value of `chocolate`. The search also uses the `distance_feature` query to increase the relevance score of documents with a `production_date` value closer to `now`.

以下`bool`搜索返回`name`值为 的 文档`chocolate`。搜索还使用`distance_feature`查询来增加`production_date`值更接近的文档的相关性分数`now`。

```console
GET /items/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "name": "chocolate"
        }
      },
      "should": {
        "distance_feature": {
          "field": "production_date",
          "pivot": "7d",
          "origin": "now"
        }
      }
    }
  }
}
```

####  Boost documents based on location

The following `bool` search returns documents with a `name` value of `chocolate`. The search also uses the `distance_feature` query to increase the relevance score of documents with a `location` value closer to `[-71.3, 41.15]`.

以下`bool`搜索返回`name`值为 的 文档`chocolate`。搜索还使用`distance_feature`查询来增加`location`值更接近的文档的相关性分数`[-71.3, 41.15]`。

```console
GET /items/_search
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "name": "chocolate"
        }
      },
      "should": {
        "distance_feature": {
          "field": "location",
          "pivot": "1000m",
          "origin": [-71.3, 41.15]
        }
      }
    }
  }
}
```

###  Top-level parameters for `distance_feature`

**`field`**

(Required, string) Name of the field used to calculate distances. This field must meet the following criteria:

（必需，字符串）用于计算距离的字段的名称。该字段必须满足以下条件：

- Be a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html), [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) or [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) field

  是一个[`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html),[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html)或 [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)字段

- Have an [`index`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-index.html) mapping parameter value of `true`, which is the default

  有一个[`index`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-index.html)映射参数值为`true`，这是默认值

- Have an [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/master/doc-values.html) mapping parameter value of `true`, which is the default

  有一个[`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/master/doc-values.html)映射参数值为`true`，这是默认值

**`origin`**

(Required, string) Date or point of origin used to calculate distances.

（必需，字符串）用于计算距离的日期或原点。

If the `field` value is a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html) or [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) field, the `origin` value must be a [date](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html#date-format-pattern). [Date Math](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math), such as `now-1h`, is supported.

如果`field`值为 a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html)or[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) 字段，则该`origin`值必须为[date](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html#date-format-pattern)。 [支持 Date Math](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)，例如`now-1h`。

If the `field` value is a [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) field, the `origin` value must be a geopoint.

如果`field`值为[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)字段，则该`origin`值必须是地理点。

**`pivot`**

(Required, [time unit](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units) or [distance unit](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#distance-units)) Distance from the `origin` at which relevance scores receive half of the `boost` value.

（必需，[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units)或[距离单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#distance-units)）`origin`与相关性分数收到一半`boost` 值的距离。

If the `field` value is a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html) or [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) field, the `pivot` value must be a [time unit](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units), such as `1h` or `10d`.

如果`field`值为 a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html)or[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) 字段，则该`pivot`值必须是[时间单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units)，例如`1h`or `10d`。

If the `field` value is a [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) field, the `pivot` value must be a [distance unit](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#distance-units), such as `1km` or `12m`.

如果`field`值为[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)字段，则该`pivot`值必须是[距离单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#distance-units)，例如`1km`或`12m`。

**`boost`**

(Optional, float) Floating point number used to multiply the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of matching documents. This value cannot be negative. Defaults to `1.0`.

（可选，浮点数）用于乘以匹配文档的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)的浮点数 。该值不能为负。默认为`1.0`.

###  Notes

####  How the `distance_feature` query calculates relevance scores

The `distance_feature` query dynamically calculates the distance between the `origin` value and a document’s field values. It then uses this distance as a feature to boost the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of closer documents.

该`distance_feature`查询动态计算之间的距离 `origin`值和文档的字段值。然后它使用这个距离作为一个特征来提高更接近文档的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)。

The `distance_feature` query calculates a document’s [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) as follows:

该`distance_feature`查询计算文档的 [相关分值](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)如下：

```
relevance score = boost * pivot / (pivot + distance)
```

The `distance` is the absolute difference between the `origin` value and a document’s field value.

该`distance`是之间的绝对差`origin`值和文档的字段值。

####  Skip non-competitive hits

Unlike the [`function_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html) query or other ways to change [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores), the `distance_feature` query efficiently skips non-competitive hits when the [`track_total_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-uri-request.html) parameter is **not** `true`.

与[`function_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html)查询或其他更改[相关性分数的](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)方法不同，`distance_feature`当[`track_total_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-uri-request.html)参数**不是** 时， 查询有效地跳过非竞争性命中 `true`。

##  More like this query

The More Like This Query finds documents that are "like" a given set of documents. In order to do so, MLT selects a set of representative terms of these input documents, forms a query using these terms, executes the query and returns the results. The user controls the input documents, how the terms should be selected and how the query is formed.

More Like This Query 查找与给定文档集“相似”的文档。为此，MLT 选择这些输入文档的一组代表性术语，使用这些术语形成查询，执行查询并返回结果。用户控制输入文档、应如何选择术语以及如何形成查询。

The simplest use case consists of asking for documents that are similar to a provided piece of text. Here, we are asking for all movies that have some text similar to "Once upon a time" in their "title" and in their "description" fields, limiting the number of selected terms to 12.

最简单的用例包括请求与提供的文本片段相似的文档。在这里，我们要求所有在“标题”和“描述”字段中具有类似于“从前”的文本的电影，将所选术语的数量限制为 12。

```console
GET /_search
{
  "query": {
    "more_like_this" : {
      "fields" : ["title", "description"],
      "like" : "Once upon a time",
      "min_term_freq" : 1,
      "max_query_terms" : 12
    }
  }
}
```

A more complicated use case consists of mixing texts with documents already existing in the index. In this case, the syntax to specify a document is similar to the one used in the [Multi GET API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-multi-get.html).

一个更复杂的用例包括将文本与索引中已经存在的文档混合。在这种情况下，指定文档的语法类似于[Multi GET API 中](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-multi-get.html)使用的语法。

```console
GET /_search
{
  "query": {
    "more_like_this": {
      "fields": [ "title", "description" ],
      "like": [
        {
          "_index": "imdb",
          "_id": "1"
        },
        {
          "_index": "imdb",
          "_id": "2"
        },
        "and potentially some more text here as well"
      ],
      "min_term_freq": 1,
      "max_query_terms": 12
    }
  }
}
```

Finally, users can mix some texts, a chosen set of documents but also provide documents not necessarily present in the index. To provide documents not present in the index, the syntax is similar to [artificial documents](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html#docs-termvectors-artificial-doc).

最后，用户可以混合一些文本、一组选定的文档，但也可以提供不一定出现在索引中的文档。为了提供索引中不存在的文档，语法类似于[人工文档](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html#docs-termvectors-artificial-doc)。

```console
GET /_search
{
  "query": {
    "more_like_this": {
      "fields": [ "name.first", "name.last" ],
      "like": [
        {
          "_index": "marvel",
          "doc": {
            "name": {
              "first": "Ben",
              "last": "Grimm"
            },
            "_doc": "You got no idea what I'd... what I'd give to be invisible."
          }
        },
        {
          "_index": "marvel",
          "_id": "2"
        }
      ],
      "min_term_freq": 1,
      "max_query_terms": 12
    }
  }
}
```

###  How it Works

Suppose we wanted to find all documents similar to a given input document. Obviously, the input document itself should be its best match for that type of query. And the reason would be mostly, according to [Lucene scoring formula](https://lucene.apache.org/core/4_9_0/core/org/apache/lucene/search/similarities/TFIDFSimilarity.html), due to the terms with the highest tf-idf. Therefore, the terms of the input document that have the highest tf-idf are good representatives of that document, and could be used within a disjunctive query (or `OR`) to retrieve similar documents. The MLT query simply extracts the text from the input document, analyzes it, usually using the same analyzer at the field, then selects the top K terms with highest tf-idf to form a disjunctive query of these terms.

假设我们想找到与给定输入文档相似的所有文档。显然，输入文档本身应该是该类型查询的最佳匹配。根据[Lucene 评分公式](https://lucene.apache.org/core/4_9_0/core/org/apache/lucene/search/similarities/TFIDFSimilarity.html)，原因主要是 由于具有最高 tf-idf 的术语。因此，具有最高 tf-idf 的输入文档的术语是该文档的良好代表，并且可以在析取查询（或`OR`）中用于检索类似文档。MLT 查询简单地从输入文档中提取文本，对其进行分析，通常在字段中使用相同的分析器，然后选择具有最高 tf-idf 的前 K 个术语以形成这些术语的析取查询。

> **WARNING:**The fields on which to perform MLT must be indexed and of type `text` or `keyword``. Additionally, when using `like` with documents, either `_source` must be enabled or the fields must be `stored` or store `term_vector`. In order to speed up analysis, it could help to store term vectors at index time.
>
> 要对其执行 MLT 的字段必须编入索引且类型为 `text`或`keyword``。此外，`like`与文档一起使用时， `_source`必须启用或字段必须是`stored`或 存储 `term_vector`。为了加快分析速度，可以在索引时存储术语向量。

For example, if we wish to perform MLT on the "title" and "tags.raw" fields, we can explicitly store their `term_vector` at index time. We can still perform MLT on the "description" and "tags" fields, as `_source` is enabled by default, but there will be no speed up on analysis for these fields.

例如，如果我们希望对“title”和“tags.raw”字段执行 MLT，我们可以`term_vector`在索引时显式存储它们。我们仍然可以在`_source`默认情况下启用的“描述”和“标签”字段上执行 MLT ，但不会加快对这些字段的分析。

```console
PUT /imdb
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "term_vector": "yes"
      },
      "description": {
        "type": "text"
      },
      "tags": {
        "type": "text",
        "fields": {
          "raw": {
            "type": "text",
            "analyzer": "keyword",
            "term_vector": "yes"
          }
        }
      }
    }
  }
}
```

###  Parameters

The only required parameter is `like`, all other parameters have sensible defaults. There are three types of parameters: one to specify the document input, the other one for term selection and for query formation.

唯一需要的参数是`like`，所有其他参数都有合理的默认值。有三种类型的参数：一种用于指定文档输入，另一种用于术语选择和查询形成。

####  Document Input Parameters

| `like`   | The only **required** parameter of the MLT query is `like` and follows a versatile syntax, in which the user can specify free form text and/or a single or multiple documents (see examples above). The syntax to specify documents is similar to the one used by the [Multi GET API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-multi-get.html). When specifying documents, the text is fetched from `fields` unless overridden in each document request. The text is analyzed by the analyzer at the field, but could also be overridden. The syntax to override the analyzer at the field follows a similar syntax to the `per_field_analyzer` parameter of the [Term Vectors API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html#docs-termvectors-per-field-analyzer). Additionally, to provide documents not necessarily present in the index, [artificial documents](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html#docs-termvectors-artificial-doc) are also supported.<br />MLT 查询 的唯一**必需**参数是`like`并遵循通用语法，其中用户可以指定自由格式文本和/或单个或多个文档（参见上面的示例）。指定文档的语法类似于[Multi GET API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-multi-get.html)使用的语法。指定文档时，`fields`除非在每个文档请求中被覆盖，否则将从中获取文本。文本由现场分析器分析，但也可以被覆盖。在字段中覆盖分析器的语法遵循与[Term Vectors API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html#docs-termvectors-per-field-analyzer)的`per_field_analyzer`参数 类似的语法。此外，为了提供索引中不一定存在的 [文档](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html#docs-termvectors-artificial-doc)，还支持[人工文档](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html#docs-termvectors-artificial-doc)。 |
| -------- | ------------------------------------------------------------ |
| `unlike` | The `unlike` parameter is used in conjunction with `like` in order not to select terms found in a chosen set of documents. In other words, we could ask for documents `like: "Apple"`, but `unlike: "cake crumble tree"`. The syntax is the same as `like`.<br />该`unlike`参数与 结合使用`like`，以便不选择在所选文档集中找到的术语。换句话说，我们可以要求文档`like: "Apple"`，但是`unlike: "cake crumble tree"`。语法与`like`. |
| `fields` | A list of fields to fetch and analyze the text from. Defaults to the `index.query.default_field` index setting, which has a default value of `*`. The `*` value matches all fields eligible for [term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/term-level-queries.html), excluding metadata fields.<br />要从中获取和分析文本的字段列表。默认为 `index.query.default_field`索引设置，其默认值为`*`。该 `*`值匹配所有符合[术语级别查询条件的](https://www.elastic.co/guide/en/elasticsearch/reference/master/term-level-queries.html)字段，元数据字段除外。 |

####  Term Selection Parameters

| `max_query_terms` | The maximum number of query terms that will be selected. Increasing this value gives greater accuracy at the expense of query execution speed. Defaults to `25`.<br />将选择的最大查询词数。增加此值会以牺牲查询执行速度为代价提供更高的准确性。默认为 `25`. |
| ----------------- | ------------------------------------------------------------ |
| `min_term_freq`   | The minimum term frequency below which the terms will be ignored from the input document. Defaults to `2`.<br />最小词频，低于该词频将被输入文档忽略。默认为`2`. |
| `min_doc_freq`    | The minimum document frequency below which the terms will be ignored from the input document. Defaults to `5`.<br />最小文档频率，低于该频率将被输入文档忽略。默认为`5`. |
| `max_doc_freq`    | The maximum document frequency above which the terms will be ignored from the input document. This could be useful in order to ignore highly frequent words such as stop words. Defaults to unbounded (`Integer.MAX_VALUE`, which is `2^31-1` or 2147483647).<br />最大文档频率，超过该频率将被输入文档忽略。这对于忽略高频词（例如停用词）很有用。默认为无界 ( `Integer.MAX_VALUE`,`2^31-1` 或 2147483647)。 |
| `min_word_length` | The minimum word length below which the terms will be ignored. Defaults to `0`.<br />最小字长，低于该长度的术语将被忽略。默认为`0`. |
| `max_word_length` | The maximum word length above which the terms will be ignored. Defaults to unbounded (`0`).<br />最大字长，超过该字长将被忽略。默认为无界 ( `0`)。 |
| `stop_words`      | An array of stop words. Any word in this set is considered "uninteresting" and ignored. If the analyzer allows for stop words, you might want to tell MLT to explicitly ignore them, as for the purposes of document similarity it seems reasonable to assume that "a stop word is never interesting".<br />一组停用词。这个集合中的任何单词都被认为是“无趣的”并被忽略。如果分析器允许停用词，您可能希望告诉 MLT 明确忽略它们，因为为了文档相似性，假设“停用词永远不会有趣”似乎是合理的。 |
| `analyzer`        | The analyzer that is used to analyze the free form text. Defaults to the analyzer associated with the first field in `fields`.<br />用于分析自由格式文本的分析器。默认为与 中的第一个字段关联的分析器`fields`。 |

####  Query Formation Parameters

| `minimum_should_match`      | After the disjunctive query has been formed, this parameter controls the number of terms that must match. The syntax is the same as the [minimum should match](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html). (Defaults to `"30%"`).<br />形成析取查询后，此参数控制必须匹配的术语数。语法与[minimum should match](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-minimum-should-match.html)相同。（默认为`"30%"`）。 |
| --------------------------- | ------------------------------------------------------------ |
| `fail_on_unsupported_field` | Controls whether the query should fail (throw an exception) if any of the specified fields are not of the supported types (`text` or `keyword`). Set this to `false` to ignore the field and continue processing. Defaults to `true`.<br />如果任何指定的字段不属于受支持的类型（`text`或`keyword`），则控制查询是否应失败（引发异常）。将此设置`false`为忽略该字段并继续处理。默认为`true`. |
| `boost_terms`               | Each term in the formed query could be further boosted by their tf-idf score. This sets the boost factor to use when using this feature. Defaults to deactivated (`0`). Any other positive value activates terms boosting with the given boost factor.<br />形成的查询中的每个术语都可以通过它们的 tf-idf 分数进一步提升。这将设置使用此功能时要使用的提升因子。默认为停用 ( `0`)。任何其他正值都会使用给定的提升因子激活项提升。 |
| `include`                   | Specifies whether the input documents should also be included in the search results returned. Defaults to `false`.<br />指定输入文档是否也应包含在返回的搜索结果中。默认为`false`. |
| `boost`                     | Sets the boost value of the whole query. Defaults to `1.0`.<br />设置整个查询的提升值。默认为`1.0`. |

###  Alternative

To take more control over the construction of a query for similar documents it is worth considering writing custom client code to assemble selected terms from an example document into a Boolean query with the desired settings. The logic in `more_like_this` that selects "interesting" words from a piece of text is also accessible via the [TermVectors API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html). For example, using the termvectors API it would be possible to present users with a selection of topical keywords found in a document’s text, allowing them to select words of interest to drill down on, rather than using the more "black-box" approach of matching used by `more_like_this`.

为了更好地控制类似文档的查询构造，值得考虑编写自定义客户端代码以将示例文档中的选定术语组合成具有所需设置的布尔查询。`more_like_this`从一段文本中选择“有趣”单词的逻辑也可以通过[TermVectors API](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-termvectors.html)访问。例如，使用 termvectors API 可以向用户展示在文档文本中找到的主题关键字的选择，允许他们选择感兴趣的词进行深入研究，而不是使用更多的“黑盒”方法使用的匹配`more_like_this`。

##  Percolate query

The `percolate` query can be used to match queries stored in an index. The `percolate` query itself contains the document that will be used as query to match with the stored queries.

该`percolate`查询可用于匹配存储在索引中的查询。该`percolate`查询本身包含将被用作查询，以配合存储查询该文档。

###  Sample Usage

Create an index with two fields:

```console
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "message": {
        "type": "text"
      },
      "query": {
        "type": "percolator"
      }
    }
  }
}
```

The `message` field is the field used to preprocess the document defined in the `percolator` query before it gets indexed into a temporary index.

`message`字段是用于在将`percolator`查询中定义的文档编入临时索引之前对其进行预处理的字段。

The `query` field is used for indexing the query documents. It will hold a json object that represents an actual Elasticsearch query. The `query` field has been configured to use the [percolator field type](https://www.elastic.co/guide/en/elasticsearch/reference/master/percolator.html). This field type understands the query dsl and stores the query in such a way that it can be used later on to match documents defined on the `percolate` query.

`query`字段用于索引查询文档。它将保存一个表示实际 Elasticsearch 查询的 json 对象。该`query`字段已配置为使用[percolator 字段类型](https://www.elastic.co/guide/en/elasticsearch/reference/master/percolator.html)。此字段类型理解查询 dsl 并以稍后可用于匹配`percolate`查询中定义的文档的方式存储查询。

Register a query in the percolator:

在 percolator 中注册一个查询：

```console
PUT /my-index-000001/_doc/1?refresh
{
  "query": {
    "match": {
      "message": "bonsai tree"
    }
  }
}
```

Match a document to the registered percolator queries:

将文档与已注册的过滤器查询相匹配：

```console
GET /my-index-000001/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "message": "A new bonsai tree in the office"
      }
    }
  }
}
```

The above request will yield the following response:

上述请求将产生以下响应：

```console-result
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.26152915,
    "hits": [
      { (1)
        "_index": "my-index-000001",
        "_id": "1",
        "_score": 0.26152915,
        "_source": {
          "query": {
            "match": {
              "message": "bonsai tree"
            }
          }
        },
        "fields" : {
          "_percolator_document_slot" : [0] (2)
        }
      }
    ]
  }
}
```

1. The query with id `1` matches our document.

   带有 id 的查询与`1`我们的文档匹配。

2. The `_percolator_document_slot` field indicates which document has matched with this query. Useful when percolating multiple document simultaneously.

   `_percolator_document_slot`字段指示与此查询匹配的文档。在同时渗透多个文档时很有用。

> TIP: To provide a simple example, this documentation uses one index `my-index-000001` for both the percolate queries and documents. This set-up can work well when there are just a few percolate queries registered. However, with heavier usage it is recommended to store queries and documents in separate indices. Please see [How it Works Under the Hood](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-percolate-query.html#how-it-works) for more details.
>
> 为了提供一个简单的例子，本文档`my-index-000001`为渗透查询和文档使用了一个索引。当只注册了几个渗透查询时，这种设置可以很好地工作。但是，如果使用较多，建议将查询和文档存储在单独的索引中。有关更多详细信息，请参阅[引擎盖下的工作原理](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-percolate-query.html#how-it-works)。

####  Parameters

The following parameters are required when percolating a document:

| `field`         | The field of type `percolator` that holds the indexed queries. This is a required parameter.<br />保存索引查询的`percolator`类型字段。这是一个必需的参数。 |
| --------------- | ------------------------------------------------------------ |
| `name`          | The suffix to be used for the `_percolator_document_slot` field in case multiple `percolate` queries have been specified. This is an optional parameter.<br />在指定了`_percolator_document_slot`多个`percolate`查询的情况下 用于该字段的后缀。这是一个可选参数。 |
| `document`      | The source of the document being percolated.<br />被渗透的文档的来源。 |
| `documents`     | Like the `document` parameter, but accepts multiple documents via a json array.<br />与`document`参数类似，但通过 json 数组接受多个文档。 |
| `document_type` | The type / mapping of the document being percolated. This parameter is deprecated and will be removed in Elasticsearch 8.0.<br />被过滤的文档的类型/映射。此参数已弃用，将在 Elasticsearch 8.0 中删除。 |

Instead of specifying the source of the document being percolated, the source can also be retrieved from an already stored document. The `percolate` query will then internally execute a get request to fetch that document.
也可以从已经存储的文档中检索源，而不是指定被过滤的文档的来源。然后`percolate`查询将在内部执行获取请求以获取该文档。

In that case the `document` parameter can be substituted with the following parameters:
在这种情况下，该`document`参数可以替换为以下参数：

| `index`      | The index the document resides in. This is a required parameter.<br />文档所在的索引。这是一个必需参数。 |
| ------------ | ------------------------------------------------------------ |
| `type`       | The type of the document to fetch. This parameter is deprecated and will be removed in Elasticsearch 8.0.<br />要获取的文档的类型。此参数已弃用，将在 Elasticsearch 8.0 中删除。 |
| `id`         | The id of the document to fetch. This is a required parameter.<br />要获取的文档的 ID。这是一个必需的参数。 |
| `routing`    | Optionally, routing to be used to fetch document to percolate.<br />可选地，用于获取要渗透的文档的路由。 |
| `preference` | Optionally, preference to be used to fetch document to percolate.<br />（可选）用于获取要渗透的文档的首选项。 |
| `version`    | Optionally, the expected version of the document to be fetched.<br />（可选）要获取的文档的预期版本。 |

####  Percolating in a filter context

In case you are not interested in the score, better performance can be expected by wrapping the percolator query in a `bool` query’s filter clause or in a `constant_score` query:

如果您对分数不感兴趣，可以通过将过滤器查询包装在`bool`查询的过滤器子句或查询中来获得更好的性能`constant_score`：

```console
GET /my-index-000001/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "percolate": {
          "field": "query",
          "document": {
            "message": "A new bonsai tree in the office"
          }
        }
      }
    }
  }
}
```

At index time terms are extracted from the percolator query and the percolator can often determine whether a query matches just by looking at those extracted terms. However, computing scores requires to deserialize each matching query and run it against the percolated document, which is a much more expensive operation. Hence if computing scores is not required the `percolate` query should be wrapped in a `constant_score` query or a `bool` query’s filter clause.

在索引时从过滤器查询中提取术语，并且过滤器通常可以仅通过查看这些提取的术语来确定查询是否匹配。然而，计算分数需要反序列化每个匹配的查询并针对渗透的文档运行它，这是一个更昂贵的操作。因此，如果不需要计算分数，`percolate`则应将查询包装在`constant_score`查询或`bool`查询的过滤器子句中。

Note that the `percolate` query never gets cached by the query cache.

请注意，`percolate`查询永远不会被查询缓存缓存。

#### Percolating multiple documents

The `percolate` query can match multiple documents simultaneously with the indexed percolator queries. Percolating multiple documents in a single request can improve performance as queries only need to be parsed and matched once instead of multiple times.

该`percolate`查询可以使用索引过滤器查询匹配同时多个文档。在单个请求中渗透多个文档可以提高性能，因为查询只需要解析和匹配一次而不是多次。

The `_percolator_document_slot` field that is being returned with each matched percolator query is important when percolating multiple documents simultaneously. It indicates which documents matched with a particular percolator query. The numbers correlate with the slot in the `documents` array specified in the `percolate` query.

`_percolator_document_slot`当同时渗透多个文档时，每个匹配的渗透器查询返回的字段很重要。它指示哪些文档与特定的过滤器查询匹配。这些数字与查询中`documents`指定的数组中的槽相关`percolate`。

```console
GET /my-index-000001/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "documents": [ (1)
        {
          "message": "bonsai tree"
        },
        {
          "message": "new tree"
        },
        {
          "message": "the office"
        },
        {
          "message": "office tree"
        }
      ]
    }
  }
}
```

1.  The documents array contains 4 documents that are going to be percolated at the same time.

   文档数组包含 4 个将同时被渗透的文档。

```console-result
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.7093853,
    "hits": [
      {
        "_index": "my-index-000001",
        "_id": "1",
        "_score": 0.7093853,
        "_source": {
          "query": {
            "match": {
              "message": "bonsai tree"
            }
          }
        },
        "fields" : {
          "_percolator_document_slot" : [0, 1, 3] (1)
        }
      }
    ]
  }
}
```

1. The `_percolator_document_slot` indicates that the first, second and last documents specified in the `percolate` query are matching with this query.

    在`_percolator_document_slot`表明，在指定的第一个，第二个和最后一个文件`percolate`的查询与此查询匹配。

####  Percolating an Existing Document

In order to percolate a newly indexed document, the `percolate` query can be used. Based on the response from an index request, the `_id` and other meta information can be used to immediately percolate the newly added document.

为了渗透新索引的文档，`percolate`可以使用查询。基于来自索引请求的响应，`_id`可以使用元信息和其他元信息立即渗透新添加的文档。

#####  Example

Based on the previous example.

基于前面的例子。

Index the document we want to percolate:

索引我们想要渗透的文档：

```console
PUT /my-index-000001/_doc/2
{
  "message" : "A new bonsai tree in the office"
}
```

Index response:

索引响应：

```console-result
{
  "_index": "my-index-000001",
  "_id": "2",
  "_version": 1,
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "result": "created",
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

Percolating an existing document, using the index response as basis to build to new search request:

渗透现有文档，使用索引响应作为构建新搜索请求的基础：

```console
GET /my-index-000001/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "index": "my-index-000001",
      "id": "2",
      "version": 1 (1)
    }
  }
}
```

1. The version is optional, but useful in certain cases. We can ensure that we are trying to percolate the document we just have indexed. A change may be made after we have indexed, and if that is the case the search request would fail with a version conflict error.

   该版本是可选的，但在某些情况下很有用。我们可以确保我们正在尝试渗透我们刚刚编入索引的文档。在我们建立索引后可能会进行更改，如果是这种情况，搜索请求将因版本冲突错误而失败。

The search response returned is identical as in the previous example.

返回的搜索响应与前面的示例相同。

####  Percolate query and highlighting

The `percolate` query is handled in a special way when it comes to highlighting. The queries hits are used to highlight the document that is provided in the `percolate` query. Whereas with regular highlighting the query in the search request is used to highlight the hits.

在`percolate`当涉及到高亮查询以特殊的方式处理。查询命中用于突出显示`percolate`查询中提供的文档。而定期突出显示搜索请求中的查询用于突出显示命中。

#####  Example

This example is based on the mapping of the first example.

此示例基于第一个示例的映射。

Save a query:

保存查询：

```console
PUT /my-index-000001/_doc/3?refresh
{
  "query": {
    "match": {
      "message": "brown fox"
    }
  }
}
```

Save another query:

保存另一个查询：

```console
PUT /my-index-000001/_doc/4?refresh
{
  "query": {
    "match": {
      "message": "lazy dog"
    }
  }
}
```

Execute a search request with the `percolate` query and highlighting enabled:

在`percolate`启用查询和突出显示的情况下执行搜索请求：

```console
GET /my-index-000001/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "document": {
        "message": "The quick brown fox jumps over the lazy dog"
      }
    }
  },
  "highlight": {
    "fields": {
      "message": {}
    }
  }
}
```

This will yield the following response.

这将产生以下响应。

```console-result
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 2,
        "relation": "eq"
    },
    "max_score": 0.26152915,
    "hits": [
      {
        "_index": "my-index-000001",
        "_id": "3",
        "_score": 0.26152915,
        "_source": {
          "query": {
            "match": {
              "message": "brown fox"
            }
          }
        },
        "highlight": {
          "message": [
            "The quick <em>brown</em> <em>fox</em> jumps over the lazy dog" 
          ]
        },
        "fields" : {
          "_percolator_document_slot" : [0]
        }
      },
      {
        "_index": "my-index-000001",
        "_id": "4",
        "_score": 0.26152915,
        "_source": {
          "query": {
            "match": {
              "message": "lazy dog"
            }
          }
        },
        "highlight": {
          "message": [
            "The quick brown fox jumps over the <em>lazy</em> <em>dog</em>" (1)
          ]
        },
        "fields" : {
          "_percolator_document_slot" : [0]
        }
      }
    ]
  }
}
```

1. The terms from each query have been highlighted in the document.

    每个查询中的术语已在文档中突出显示。

Instead of the query in the search request highlighting the percolator hits, the percolator queries are highlighting the document defined in the `percolate` query.

不是搜索请求中的查询突出显示过滤器命中，而是过滤器查询突出显示查询中定义的文档`percolate`。

When percolating multiple documents at the same time like the request below then the highlight response is different:

当像下面的请求一样同时渗透多个文档时，高亮响应是不同的：

```console
GET /my-index-000001/_search
{
  "query": {
    "percolate": {
      "field": "query",
      "documents": [
        {
          "message": "bonsai tree"
        },
        {
          "message": "new tree"
        },
        {
          "message": "the office"
        },
        {
          "message": "office tree"
        }
      ]
    }
  },
  "highlight": {
    "fields": {
      "message": {}
    }
  }
}
```

The slightly different response:

略有不同的回应：

```console-result
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.7093853,
    "hits": [
      {
        "_index": "my-index-000001",
        "_id": "1",
        "_score": 0.7093853,
        "_source": {
          "query": {
            "match": {
              "message": "bonsai tree"
            }
          }
        },
        "fields" : {
          "_percolator_document_slot" : [0, 1, 3]
        },
        "highlight" : { (1)
          "0_message" : [
              "<em>bonsai</em> <em>tree</em>"
          ],
          "3_message" : [
              "office <em>tree</em>"
          ],
          "1_message" : [
              "new <em>tree</em>"
          ]
        }
      }
    ]
  }
}
```

1. The highlight fields have been prefixed with the document slot they belong to, in order to know which highlight field belongs to what document.

    高亮字段以它们所属的文档槽为前缀，以便知道哪个高亮字段属于哪个文档。

####  Specifying multiple percolate queries

It is possible to specify multiple `percolate` queries in a single search request:

可以`percolate`在单个搜索请求中指定多个查询：

```console
GET /my-index-000001/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "percolate": {
            "field": "query",
            "document": {
              "message": "bonsai tree"
            },
            "name": "query1" (1)
          }
        },
        {
          "percolate": {
            "field": "query",
            "document": {
              "message": "tulip flower"
            },
            "name": "query2" (1)
          }
        }
      ]
    }
  }
}
```

1. The `name` parameter will be used to identify which percolator document slots belong to what `percolate` query.

    该`name`参数将用于标识哪些渗透器文档槽属于哪个`percolate`查询。

The `_percolator_document_slot` field name will be suffixed with what is specified in the `_name` parameter. If that isn’t specified then the `field` parameter will be used, which in this case will result in ambiguity.

该`_percolator_document_slot`字段名称将与什么是在指定的后缀`_name`参数。如果未指定，则将使用该`field`参数，在这种情况下将导致歧义。

The above search request returns a response similar to this:

上面的搜索请求返回类似这样的响应：

```console-result
{
  "took": 13,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.26152915,
    "hits": [
      {
        "_index": "my-index-000001",
        "_id": "1",
        "_score": 0.26152915,
        "_source": {
          "query": {
            "match": {
              "message": "bonsai tree"
            }
          }
        },
        "fields" : {
          "_percolator_document_slot_query1" : [0] (1)
        }
      }
    ]
  }
}
```

1.  The `_percolator_document_slot_query1` percolator slot field indicates that these matched slots are from the `percolate` query with `_name` parameter set to `query1`.

    该`_percolator_document_slot_query1`渗滤器槽字段表明这些匹配插槽脱离`percolate` 与查询`_name`参数集`query1`。

####  How it Works Under the Hood

When indexing a document into an index that has the [percolator field type](https://www.elastic.co/guide/en/elasticsearch/reference/master/percolator.html) mapping configured, the query part of the document gets parsed into a Lucene query and is stored into the Lucene index. A binary representation of the query gets stored, but also the query’s terms are analyzed and stored into an indexed field.

将文档索引到配置了[percolator 字段类型](https://www.elastic.co/guide/en/elasticsearch/reference/master/percolator.html)映射的索引时，文档的查询部分会被解析为 Lucene 查询并存储到 Lucene 索引中。查询的二进制表示被存储，但查询的术语也会被分析并存储到索引字段中。

At search time, the document specified in the request gets parsed into a Lucene document and is stored in a in-memory temporary Lucene index. This in-memory index can just hold this one document and it is optimized for that. After this a special query is built based on the terms in the in-memory index that select candidate percolator queries based on their indexed query terms. These queries are then evaluated by the in-memory index if they actually match.

在搜索时，请求中指定的文档被解析为 Lucene 文档并存储在内存中的临时 Lucene 索引中。这个内存中的索引只能保存这个文档，并为此进行了优化。在此之后，基于内存索引中的术语构建一个特殊查询，该查询根据索引查询术语选择候选过滤器查询。如果这些查询实际上匹配，则这些查询将通过内存索引进行评估。

The selecting of candidate percolator queries matches is an important performance optimization during the execution of the `percolate` query as it can significantly reduce the number of candidate matches the in-memory index needs to evaluate. The reason the `percolate` query can do this is because during indexing of the percolator queries the query terms are being extracted and indexed with the percolator query. Unfortunately the percolator cannot extract terms from all queries (for example the `wildcard` or `geo_shape` query) and as a result of that in certain cases the percolator can’t do the selecting optimization (for example if an unsupported query is defined in a required clause of a boolean query or the unsupported query is the only query in the percolator document). These queries are marked by the percolator and can be found by running the following search:

候选过滤器查询匹配的选择是查询执行期间重要的性能优化，`percolate`因为它可以显着减少内存索引需要评估的候选匹配的数量。`percolate`查询可以这样做的原因是因为在对 percolator 查询进行索引期间，正在使用 percolator 查询提取查询词并对其进行索引。不幸的是，过滤器不能从所有查询提取条件（例如`wildcard`或`geo_shape`查询），因此在某些情况下，过滤器无法进行选择优化（例如，如果在布尔查询的必需子句中定义了不受支持的查询，或者不受支持的查询是过滤器文档中的唯一查询） . 这些查询由过滤器标记，可以通过运行以下搜索找到：

```console
GET /_search
{
  "query": {
    "term" : {
      "query.extraction_result" : "failed"
    }
  }
}
```

> NOTE: The above example assumes that there is a `query` field of type `percolator` in the mappings.
>
> 上面的例子假设映射中有一个`query`type 字段 `percolator`。

Given the design of percolation, it often makes sense to use separate indices for the percolate queries and documents being percolated, as opposed to a single index as we do in examples. There are a few benefits to this approach:

考虑到渗透的设计，对渗透查询和被渗透的文档使用单独的索引通常是有意义的，而不是像我们在示例中所做的那样使用单个索引。这种方法有几个好处：

- Because percolate queries contain a different set of fields from the percolated documents, using two separate indices allows for fields to be stored in a denser, more efficient way.

  因为渗透查询包含与渗透文档不同的一组字段，所以使用两个单独的索引允许以更密集、更有效的方式存储字段。

- Percolate queries do not scale in the same way as other queries, so percolation performance may benefit from using a different index configuration, like the number of primary shards.

  渗透查询的扩展方式与其他查询不同，因此渗透性能可能会受益于使用不同的索引配置，例如主分片的数量。

###  Notes

####  Allow expensive queries

Percolate queries will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) 设置为 false，则不会执行 Percolate 查询。

##  Rank feature query

Boosts the [relevance score](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of documents based on the numeric value of a [`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html) or [`rank_features`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-features.html) field.

根据 a或 字段的数值提高文档的[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)。[`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html)[`rank_features`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-features.html)

The `rank_feature` query is typically used in the `should` clause of a [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) query so its relevance scores are added to other scores from the `bool` query.

`rank_feature`查询典型地在使用中`should`一个的子句 [`bool`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)查询，以便它的相关性分数被添加到从对方得分`bool`的查询。

With `positive_score_impact` set to `false` for a `rank_feature` or `rank_features` field, we recommend that every document that participates in a query has a value for this field. Otherwise, if a `rank_feature` query is used in the should clause, it doesn’t add anything to a score of a document with a missing value, but adds some boost for a document containing a feature. This is contrary to what we want – as we consider these features negative, we want to rank documents containing them lower than documents missing them.

随着`positive_score_impact`集到`false`一个`rank_feature`或 `rank_features`领域，我们建议每个文档，在查询参与了此字段的值。否则，如果`rank_feature`在 should 子句中使用查询，它不会向具有缺失值的文档的分数添加任何内容，但会为包含特征的文档添加一些提升。这与我们想要的相反——因为我们认为这些特征是负面的，我们希望包含它们的文档的排名低于缺少它们的文档。

Unlike the [`function_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html) query or other ways to change [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores), the `rank_feature` query efficiently skips non-competitive hits when the [`track_total_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-uri-request.html) parameter is **not** `true`. This can dramatically improve query speed.

与[`function_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html)查询或其他更改[相关性分数的](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)方法不同，`rank_feature`当[`track_total_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-uri-request.html)参数**不是** 时， 查询有效地跳过非竞争性命中 `true`。这可以显着提高查询速度。

###  Rank feature functions

To calculate relevance scores based on rank feature fields, the `rank_feature` query supports the following mathematical functions:

为了根据排名特征字段计算相关性分数，该`rank_feature` 查询支持以下数学函数：

- [Saturation](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-saturation)
- [Logarithm](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-logarithm)
- [Sigmoid](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-sigmoid)
- [Linear](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-linear)

If you don’t know where to start, we recommend using the `saturation` function. If no function is provided, the `rank_feature` query uses the `saturation` function by default.

如果您不知道从哪里开始，我们建议您使用该`saturation`功能。如果没有提供函数，则`rank_feature`查询`saturation` 默认使用该函数。

###  Example request

####  Index setup

To use the `rank_feature` query, your index must include a [`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html) or [`rank_features`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-features.html) field mapping. To see how you can set up an index for the `rank_feature` query, try the following example.

要使用`rank_feature`查询，您的索引必须包含一个 [`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html)or[`rank_features`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-features.html)字段映射。要了解如何为`rank_feature`查询设置索引，请尝试以下示例。

Create a `test` index with the following field mappings:

`test`使用以下字段映射创建索引：

- `pagerank`, a [`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html) field which measures the importance of a website

  `pagerank`,[`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html)衡量网站重要性的字段

- `url_length`, a [`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html) field which contains the length of the website’s URL. For this example, a long URL correlates negatively to relevance, indicated by a `positive_score_impact` value of `false`.

  `url_length`，[`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html)包含网站 URL 长度的字段。对于此示例，长 URL 与相关性呈负相关，由`positive_score_impact`值表示`false`。

- `topics`, a [`rank_features`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-features.html) field which contains a list of topics and a measure of how well each document is connected to this topic

  `topics`，[`rank_features`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-features.html)包含主题列表和衡量每个文档与该主题的联系程度的字段

```console
PUT /test
{
  "mappings": {
    "properties": {
      "pagerank": {
        "type": "rank_feature"
      },
      "url_length": {
        "type": "rank_feature",
        "positive_score_impact": false
      },
      "topics": {
        "type": "rank_features"
      }
    }
  }
}
```

Index several documents to the `test` index.

将多个文档`test`索引到索引中。

```console
PUT /test/_doc/1?refresh
{
  "url": "https://en.wikipedia.org/wiki/2016_Summer_Olympics",
  "content": "Rio 2016",
  "pagerank": 50.3,
  "url_length": 42,
  "topics": {
    "sports": 50,
    "brazil": 30
  }
}

PUT /test/_doc/2?refresh
{
  "url": "https://en.wikipedia.org/wiki/2016_Brazilian_Grand_Prix",
  "content": "Formula One motor race held on 13 November 2016",
  "pagerank": 50.3,
  "url_length": 47,
  "topics": {
    "sports": 35,
    "formula one": 65,
    "brazil": 20
  }
}

PUT /test/_doc/3?refresh
{
  "url": "https://en.wikipedia.org/wiki/Deadpool_(film)",
  "content": "Deadpool is a 2016 American superhero film",
  "pagerank": 50.3,
  "url_length": 37,
  "topics": {
    "movies": 60,
    "super hero": 65
  }
}
```

####  Example query

The following query searches for `2016` and boosts relevance scores based on `pagerank`, `url_length`, and the `sports` topic.

对于下面的查询搜索`2016`，并提升相关性得分基础上 `pagerank`，`url_length`和`sports`话题。

```console
GET /test/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "content": "2016"
          }
        }
      ],
      "should": [
        {
          "rank_feature": {
            "field": "pagerank"
          }
        },
        {
          "rank_feature": {
            "field": "url_length",
            "boost": 0.1
          }
        },
        {
          "rank_feature": {
            "field": "topics.sports",
            "boost": 0.4
          }
        }
      ]
    }
  }
}
```

###  Top-level parameters for `rank_feature`

**`field`**

(Required, string) [`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html) or [`rank_features`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-features.html) field used to boost [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores).

（必需，字符串）[`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-feature.html)或 [`rank_features`](https://www.elastic.co/guide/en/elasticsearch/reference/master/rank-features.html)用于提高[相关性分数的](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)字段 。

**`boost`**

(Optional, float) Floating point number used to decrease or increase [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores). Defaults to `1.0`.

（可选，浮点数）用于减少或增加[相关性分数的](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)浮点数 。默认为`1.0`.

Boost values are relative to the default value of `1.0`. A boost value between `0` and `1.0` decreases the relevance score. A value greater than `1.0` increases the relevance score.

Boost 值相对于 的默认值`1.0`。`0`和之间的提升值会 `1.0`降低相关性分数。大于 的值会`1.0` 增加相关性分数。

**`saturation`**

(Optional, [function object](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-saturation)) Saturation function used to boost [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) based on the value of the rank feature `field`. If no function is provided, the `rank_feature` query defaults to the `saturation` function. See [Saturation](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-saturation) for more information.

（可选，[函数对象](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-saturation)）饱和度函数，用于根据排名特征的值提高[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)`field`。如果未提供函数，则`rank_feature` 查询默认为该`saturation`函数。有关更多信息，请参阅 [饱和度](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-saturation)。

Only one function `saturation`, `log`, `sigmoid` or `linear` can be provided.

只能提供一个功能`saturation`、`log`、`sigmoid`或`linear`。

**`log`**

(Optional, [function object](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-logarithm)) Logarithmic function used to boost [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) based on the value of the rank feature `field`. See [Logarithm](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-logarithm) for more information.

（可选，[函数对象](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-logarithm)）对数函数，用于根据排名特征的值提高[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)`field`。有关详细信息，请参阅 [对数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-logarithm)。

Only one function `saturation`, `log`, `sigmoid` or `linear` can be provided.

只能提供一个功能`saturation`、`log`、`sigmoid`或`linear`。

**`sigmoid`**

(Optional, [function object](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-sigmoid)) Sigmoid function used to boost [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) based on the value of the rank feature `field`. See [Sigmoid](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-sigmoid) for more information.

（可选，[函数对象](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-sigmoid)）Sigmoid 函数，用于根据排名特征的值提高[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)`field`。有关更多信息，请参阅[Sigmoid](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-sigmoid)。

Only one function `saturation`, `log`, `sigmoid` or `linear` can be provided.

只能提供一个功能`saturation`、`log`、`sigmoid`或`linear`。

**`linear`**

(Optional, [function object](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-linear)) Linear function used to boost [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) based on the value of the rank feature `field`. See [Linear](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-linear) for more information.

Only one function `saturation`, `log`, `sigmoid` or `linear` can be provided.

（可选，[函数对象](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-linear)）用于根据排名特征值提高[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)的线性函数`field`。有关更多信息，请参阅[线性](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html#rank-feature-query-linear)。

只能提供一个功能`saturation`、`log`、`sigmoid`或`linear`。

###  Notes

####  Saturation

The `saturation` function gives a score equal to `S / (S + pivot)`, where `S` is the value of the rank feature field and `pivot` is a configurable pivot value so that the result will be less than `0.5` if `S` is less than pivot and greater than `0.5` otherwise. Scores are always `(0,1)`.

该`saturation`函数给出的分数等于`S / (S + pivot)`，其中`S`是排名特征字段的值，并且`pivot`是一个可配置的枢轴值，因此结果将小于`0.5`if`S`小于枢轴并大于`0.5`否则。分数总是`(0,1)`。

If the rank feature has a negative score impact then the function will be computed as `pivot / (S + pivot)`, which decreases when `S` increases.

如果排名特征对分数有负面影响，则该函数将计算为`pivot / (S + pivot)`，随着`S`增加而减少。

```console
GET /test/_search
{
  "query": {
    "rank_feature": {
      "field": "pagerank",
      "saturation": {
        "pivot": 8
      }
    }
  }
}
```

If a `pivot` value is not provided, Elasticsearch computes a default value equal to the approximate geometric mean of all rank feature values in the index. We recommend using this default value if you haven’t had the opportunity to train a good pivot value.

如果`pivot`未提供值，Elasticsearch 会计算一个默认值，该值等于索引中所有排名特征值的近似几何平均值。如果您没有机会训练一个好的枢轴值，我们建议您使用此默认值。

```console
GET /test/_search
{
  "query": {
    "rank_feature": {
      "field": "pagerank",
      "saturation": {}
    }
  }
}
```

####  Logarithm

The `log` function gives a score equal to `log(scaling_factor + S)`, where `S` is the value of the rank feature field and `scaling_factor` is a configurable scaling factor. Scores are unbounded.

该`log`函数给出的分数等于`log(scaling_factor + S)`，其中`S` 是排名特征字段的值，`scaling_factor`是一个可配置的缩放因子。分数是无限的。

This function only supports rank features that have a positive score impact.

此函数仅支持具有正分数影响的排名特征。

```console
GET /test/_search
{
  "query": {
    "rank_feature": {
      "field": "pagerank",
      "log": {
        "scaling_factor": 4
      }
    }
  }
}
```

####  Sigmoid

The `sigmoid` function is an extension of `saturation` which adds a configurable exponent. Scores are computed as `S^exp^ / (S^exp^ + pivot^exp^)`. Like for the `saturation` function, `pivot` is the value of `S` that gives a score of `0.5` and scores are `(0,1)`.

该`sigmoid`函数是一个扩展，`saturation`它添加了一个可配置的指数。分数计算如下`S^exp^ / (S^exp^ + pivot^exp^)`。就像 `saturation`函数一样，`pivot`是`S`给出分数的值`0.5` ，分数是`(0,1)`。

The `exponent` must be positive and is typically in `[0.5, 1]`. A good value should be computed via training. If you don’t have the opportunity to do so, we recommend you use the `saturation` function instead.

本`exponent`必须是积极的，是典型的`[0.5, 1]`。一个好的值应该通过训练来计算。如果您没有机会这样做，我们建议您改用该`saturation`功能。

```console
GET /test/_search
{
  "query": {
    "rank_feature": {
      "field": "pagerank",
      "sigmoid": {
        "pivot": 7,
        "exponent": 0.6
      }
    }
  }
}
```

####  Linear

The `linear` function is the simplest function, and gives a score equal to the indexed value of `S`, where `S` is the value of the rank feature field. If a rank feature field is indexed with `"positive_score_impact": true`, its indexed value is equal to `S` and rounded to preserve only 9 significant bits for the precision. If a rank feature field is indexed with `"positive_score_impact": false`, its indexed value is equal to `1/S` and rounded to preserve only 9 significant bits for the precision.

该`linear`函数是最简单的函数，给出的分数等于 的索引值`S`，其中`S`是 rank 特征字段的值。如果排名特征字段用 索引`"positive_score_impact": true`，则其索引值等于`S`并四舍五入以仅保留 9 个有效位以保证精度。如果排名特征字段用 索引`"positive_score_impact": false`，则其索引值等于`1/S`并四舍五入以仅保留 9 个有效位以保证精度。

```console
GET /test/_search
{
  "query": {
    "rank_feature": {
      "field": "pagerank",
      "linear": {}
    }
  }
}
```

##  Script query

> NOTE: Runtime fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html) provide a very similar feature that is more flexible. You write a script to create field values and they are available everywhere, such as [`fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html), [all queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html), and [aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations.html).
>
> [
> 运行时字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime.html)提供了一个非常相似的功能，而且更加灵活。您编写一个脚本来创建字段值，它们随处可用，例如[`fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html)， [所有查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html)和[聚合](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations.html)。

Filters documents based on a provided [script](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-using.html). The `script` query is typically used in a [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html).

根据提供的[脚本](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-using.html)过滤文档。该 `script`查询通常用于[过滤器上下文](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html)。

> WARNING:Using scripts can result in slower search speeds. See [Scripts, caching, and search speed](https://www.elastic.co/guide/en/elasticsearch/reference/master/scripts-and-search-speed.html).
>
> 使用脚本会导致搜索速度变慢。请参阅 [脚本、缓存和搜索速度](https://www.elastic.co/guide/en/elasticsearch/reference/master/scripts-and-search-speed.html)。

###  Example request

```console
GET /_search
{
  "query": {
    "bool": {
      "filter": {
        "script": {
          "script": """
            double amount = doc['amount'].value;
            if (doc['type'].value == 'expense') {
              amount *= -1;
            }
            return amount < 10;
          """
        }
      }
    }
  }
}
```

You can achieve the same results in a search query by using runtime fields. Use the [`fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html) parameter on the `_search` API to fetch values as part of the same query:

您可以使用运行时字段在搜索查询中获得相同的结果。使用API[`fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html)上的 参数 `_search`获取值作为同一查询的一部分：

```console
GET /_search
{
  "runtime_mappings": {
    "amount.signed": {
      "type": "double",
      "script": """
        double amount = doc['amount'].value;
        if (doc['type'].value == 'expense') {
          amount *= -1;
        }
        emit(amount);
      """
    }
  },
  "query": {
    "bool": {
      "filter": {
        "range": {
          "amount.signed": { "lt": 10 }
        }
      }
    }
  },
  "fields": [{"field": "amount.signed"}]
}
```

###  Top-level parameters for `script`

**`script`**

(Required, [script object](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-using.html)) Contains a script to run as a query. This script must return a boolean value, `true` or `false`.

（必需，[脚本对象](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-using.html)）包含作为查询运行的脚本。此脚本必须返回一个布尔值，`true`或`false`.

###  Notes

####  Custom Parameters

Like [filters](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html), scripts are cached for faster execution. If you frequently change the arguments of a script, we recommend you store them in the script’s `params` parameter. For example:

像[过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html)一样，脚本被缓存以加快执行速度。如果您经常更改脚本的`params`参数，我们建议您将它们存储在脚本的参数中。例如：

```console
GET /_search
{
  "query": {
    "bool": {
      "filter": {
        "script": {
          "script": {
            "source": "doc['num1'].value > params.param1",
            "lang": "painless",
            "params": {
              "param1": 5
            }
          }
        }
      }
    }
  }
}
```

####  Allow expensive queries

Script queries will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) 设置为 false，则不会执行脚本查询。

## Script score query

Uses a [script](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html) to provide a custom score for returned documents.

使用[脚本](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html)为返回的文档提供自定义分数。

The `script_score` query is useful if, for example, a scoring function is expensive and you only need to calculate the score of a filtered set of documents.

`script_score`例如，如果评分函数很昂贵并且您只需要计算一组过滤文档的分数，则该查询很有用。

###  Example request

The following `script_score` query assigns each returned document a score equal to the `my-int` field value divided by `10`.

以下`script_score`查询为每个返回的文档分配一个等于`my-int`字段值除以`10`的分数。

```console
GET /_search
{
  "query": {
    "script_score": {
      "query": {
        "match": { "message": "elasticsearch" }
      },
      "script": {
        "source": "doc['my-int'].value / 10 "
      }
    }
  }
}
```

###  Top-level parameters for `script_score`

**`query`**

(Required, query object) Query used to return documents.

（必需，查询对象）用于返回文档的查询。

**`script`**

(Required, [script object](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-using.html)) Script used to compute the score of documents returned by the `query`.

（必需，[脚本对象](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-using.html)）用于计算`query`.

> **IMPORTANT:** Final relevance scores from the `script_score` query cannot be negative. To support certain search optimizations, Lucene requires scores be positive or `0`.
>
> `script_score`查询的最终相关性分数不能为负。为了支持某些搜索优化，Lucene 要求分数为正或`0`。

**`min_score`**

(Optional, float) Documents with a score lower than this floating point number are excluded from the search results.

（可选，浮点数）分数低于此浮点数的文档将从搜索结果中排除。

**`boost`**

(Optional, float) Documents' scores produced by `script` are multiplied by `boost` to produce final documents' scores. Defaults to `1.0`.

（可选，浮点数）产生的文档分数`script`乘以`boost`产生最终文档的分数。默认为`1.0`.

###  Notes

####  Use relevance scores in a script

Within a script, you can [access](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-fields.html#scripting-score) the `_score` variable which represents the current relevance score of a document.

在脚本中，您可以 [访问](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-fields.html#scripting-score)`_score`代表文档当前相关性分数 的变量。

####  Predefined functions

You can use any of the available [painless functions](https://www.elastic.co/guide/en/elasticsearch/painless/master/painless-contexts.html) in your `script`. You can also use the following predefined functions to customize scoring:

您可以使用任何可用的[无痛性功能](https://www.elastic.co/guide/en/elasticsearch/painless/master/painless-contexts.html)在你的`script`。您还可以使用以下预定义函数来自定义评分：

- [Saturation](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#script-score-saturation)
- [Sigmoid](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#script-score-sigmoid)
- [Random score function](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#random-score-function)
- [Decay functions for numeric fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#decay-functions-numeric-fields)
- [Decay functions for geo fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#decay-functions-geo-fields)
- [Decay functions for date fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#decay-functions-date-fields)
- [Functions for vector fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#script-score-functions-vector-fields)

We suggest using these predefined functions instead of writing your own. These functions take advantage of efficiencies from Elasticsearch' internal mechanisms.

我们建议使用这些预定义的函数，而不是自己编写。这些函数利用了 Elasticsearch 内部机制的效率。

#####  Saturation（饱和）

saturation(value,k) = value/(k + value)

```console
"script" : {
    "source" : "saturation(doc['my-int'].value, 1)"
}
```

#####  Sigmoid（S型函数）

sigmoid(value, k, a) = value^a/ (k^a + value^a)

```js
"script" : {
    "source" : "sigmoid(doc['my-int'].value, 2, 1)"
}
```

#####  Random score function（随机评分函数）

`random_score` function generates scores that are uniformly distributed from 0 up to but not including 1.

`random_score` 函数生成从 0 到但不包括 1 的均匀分布的分数。

`randomScore` function has the following syntax: `randomScore(<seed>, <fieldName>)`. It has a required parameter - `seed` as an integer value, and an optional parameter - `fieldName` as a string value.

`randomScore`函数具有以下语法： `randomScore(<seed>, <fieldName>)`. 它有一个必需参数 -`seed`作为整数值，以及一个可选参数 -`fieldName`作为字符串值。

```js
"script" : {
    "source" : "randomScore(100, '_seq_no')"
}
```

If the `fieldName` parameter is omitted, the internal Lucene document ids will be used as a source of randomness. This is very efficient, but unfortunately not reproducible since documents might be renumbered by merges.

如果`fieldName`省略该参数，则内部 Lucene 文档 ID 将用作随机源。这是非常有效的，但不幸的是无法重现，因为文档可能会通过合并重新编号。

```js
"script" : {
    "source" : "randomScore(100)"
}
```

Note that documents that are within the same shard and have the same value for field will get the same score, so it is usually desirable to use a field that has unique values for all documents across a shard. A good default choice might be to use the `_seq_no` field, whose only drawback is that scores will change if the document is updated since update operations also update the value of the `_seq_no` field.

请注意，位于同一分片内且具有相同字段值的文档将获得相同的分数，因此通常希望使用对分片中所有文档具有唯一值的字段。一个好的默认选择可能是使用`_seq_no` 字段，其唯一的缺点是如果文档更新，分数会改变，因为更新操作也会更新`_seq_no`字段的值。

#####  Decay functions for numeric fields

You can read more about decay functions [here](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#function-decay).

您可以[在此处](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html#function-decay)阅读有关衰减函数的更多信息 。

- `double decayNumericLinear(double origin, double scale, double offset, double decay, double docValue)`
- `double decayNumericExp(double origin, double scale, double offset, double decay, double docValue)`
- `double decayNumericGauss(double origin, double scale, double offset, double decay, double docValue)`

```js
"script" : {
    "source" : "decayNumericLinear(params.origin, params.scale, params.offset, params.decay, doc['dval'].value)",
    "params": { (1)
        "origin": 20,
        "scale": 10,
        "decay" : 0.5,
        "offset" : 0
    }
}
```

1. Using `params` allows to compile the script only once, even if params change.

    使用`params`允许只编译一次脚本，即使参数改变。

#####  Decay functions for geo fields

- `double decayGeoLinear(String originStr, String scaleStr, String offsetStr, double decay, GeoPoint docValue)`
- `double decayGeoExp(String originStr, String scaleStr, String offsetStr, double decay, GeoPoint docValue)`
- `double decayGeoGauss(String originStr, String scaleStr, String offsetStr, double decay, GeoPoint docValue)`

```js
"script" : {
    "source" : "decayGeoExp(params.origin, params.scale, params.offset, params.decay, doc['location'].value)",
    "params": {
        "origin": "40, -70.12",
        "scale": "200km",
        "offset": "0km",
        "decay" : 0.2
    }
}
```

##### Decay functions for date fields

- `double decayDateLinear(String originStr, String scaleStr, String offsetStr, double decay, JodaCompatibleZonedDateTime docValueDate)`
- `double decayDateExp(String originStr, String scaleStr, String offsetStr, double decay, JodaCompatibleZonedDateTime docValueDate)`
- `double decayDateGauss(String originStr, String scaleStr, String offsetStr, double decay, JodaCompatibleZonedDateTime docValueDate)`

```js
"script" : {
    "source" : "decayDateGauss(params.origin, params.scale, params.offset, params.decay, doc['date'].value)",
    "params": {
        "origin": "2008-01-01T01:00:00Z",
        "scale": "1h",
        "offset" : "0",
        "decay" : 0.5
    }
}
```

> NOTE: Decay functions on dates are limited to dates in the default format and default time zone. Also calculations with `now` are not supported.
>
> 日期衰减函数仅限于默认格式和默认时区的日期。`now`也不支持计算。

##### Functions for vector fields

[Functions for vector fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#vector-functions) are accessible through `script_score` query.

[矢量字段的函数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#vector-functions)可通过`script_score`查询访问 。

####  Allow expensive queries

Script score queries will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) 设置为 false，则不会执行脚本分数查询。

####  Faster alternatives

The `script_score` query calculates the score for every matching document, or hit. There are faster alternative query types that can efficiently skip non-competitive hits:

该`script_score`查询计算每一个匹配文档，或者命中得分。有更快的替代查询类型可以有效地跳过非竞争性命中：

- If you want to boost documents on some static fields, use the [`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html) query.

  如果要在某些静态字段上提升文档，请使用 [`rank_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-rank-feature-query.html)查询。

- If you want to boost documents closer to a date or geographic point, use the [`distance_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-distance-feature-query.html) query.

  如果要提升更接近日期或地理点的文档，请使用 [`distance_feature`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-distance-feature-query.html)查询。

####  Transition from the function score query

We are deprecating the [`function_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html) query. We recommend using the `script_score` query instead.

我们正在弃用该[`function_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html) 查询。我们建议改用`script_score`查询。

You can implement the following functions from the `function_score` query using the `script_score` query:

您可以`function_score`使用`script_score`查询从查询中实现以下功能：

- [`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#script-score)
- [`weight`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#weight)
- [`random_score`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#random-score)
- [`field_value_factor`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#field-value-factor)
- [`decay` functions](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#decay-functions)

##### `script_score`

What you used in `script_score` of the Function Score query, you can copy into the Script Score query. No changes here.

您在`script_score`Function Score 查询中使用的内容，您可以复制到 Script Score 查询中。这里没有变化。

##### `weight`

`weight` function can be implemented in the Script Score query through the following script:

`weight` 函数可以通过以下脚本在 Script Score 查询中实现：

```js
"script" : {
    "source" : "params.weight * _score",
    "params": {
        "weight": 2
    }
}
```

##### `random_score`

Use `randomScore` function as described in [random score function](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#random-score-function).

使用[随机得分函数中](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#random-score-function)`randomScore`描述的[函数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#random-score-function)。

##### `field_value_factor`

`field_value_factor` function can be easily implemented through script:

`field_value_factor` 功能可以通过脚本轻松实现：

```js
"script" : {
    "source" : "Math.log10(doc['field'].value * params.factor)",
    "params" : {
        "factor" : 5
    }
}
```

For checking if a document has a missing value, you can use `doc['field'].size() == 0`. For example, this script will use a value `1` if a document doesn’t have a field `field`:

要检查文档是否有缺失值，您可以使用 `doc['field'].size() == 0`. 例如，如果文档没有字段，此脚本将使用`1`作为`field`的值：

```js
"script" : {
    "source" : "Math.log10((doc['field'].size() == 0 ? 1 : doc['field'].value()) * params.factor)",
    "params" : {
        "factor" : 5
    }
}
```

This table lists how `field_value_factor` modifiers can be implemented through a script:

下表列出了如何`field_value_factor`通过脚本实现修饰符：

| Modifier     | Implementation in Script Score   |
| ------------ | -------------------------------- |
| `none`       | -                                |
| `log`        | `Math.log10(doc['f'].value)`     |
| `log1p`      | `Math.log10(doc['f'].value + 1)` |
| `log2p`      | `Math.log10(doc['f'].value + 2)` |
| `ln`         | `Math.log(doc['f'].value)`       |
| `ln1p`       | `Math.log(doc['f'].value + 1)`   |
| `ln2p`       | `Math.log(doc['f'].value + 2)`   |
| `square`     | `Math.pow(doc['f'].value, 2)`    |
| `sqrt`       | `Math.sqrt(doc['f'].value)`      |
| `reciprocal` | `1.0 / doc['f'].value`           |

##### `decay` functions

The `script_score` query has equivalent [decay functions](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#decay-functions) that can be used in script.

该`script_score`查询具有 可在脚本中使用的等效[衰减函数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-script-score-query.html#decay-functions)。

####  Functions for vector fields

>  NOTE: During vector functions' calculation, all matched documents are linearly scanned. Thus, expect the query time grow linearly with the number of matched documents. For this reason, we recommend to limit the number of matched documents with a `query` parameter.
>
>  在向量函数的计算过程中，所有匹配的文档都被线性扫描。因此，预计查询时间会随着匹配文档的数量线性增长。因此，我们建议使用`query`参数限制匹配文档的数量。

This is the list of available vector functions and vector access methods:

这是可用向量函数和向量访问方法的列表：

1. `cosineSimilarity` – calculates cosine similarity

   `cosineSimilarity` – 计算余弦相似度

2. `dotProduct` – calculates dot product

   `dotProduct` – 计算点积

3. `l1norm` – calculates L1 distance

   `l1norm`– 计算 L 1距离

4. `l2norm` - calculates L2 distance

   `l2norm`- 计算 L 2距离

5. `doc[<field>].vectorValue` – returns a vector’s value as an array of floats

   `doc[<field>].vectorValue` – 以浮点数组形式返回向量的值

6. `doc[<field>].magnitude` – returns a vector’s magnitude

   `doc[<field>].magnitude` – 返回向量的大小

Let’s create an index with a `dense_vector` mapping and index a couple of documents into it.

让我们创建一个带有`dense_vector`映射的索引，并将几个文档索引到其中。

```console
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "my_dense_vector": {
        "type": "dense_vector",
        "dims": 3
      },
      "status" : {
        "type" : "keyword"
      }
    }
  }
}

PUT my-index-000001/_doc/1
{
  "my_dense_vector": [0.5, 10, 6],
  "status" : "published"
}

PUT my-index-000001/_doc/2
{
  "my_dense_vector": [-0.5, 10, 10],
  "status" : "published"
}

POST my-index-000001/_refresh
```

The `cosineSimilarity` function calculates the measure of cosine similarity between a given query vector and document vectors.

该`cosineSimilarity`函数计算给定查询向量和文档向量之间的余弦相似度度量。

```console
GET my-index-000001/_search
{
  "query": {
    "script_score": {
      "query" : {
        "bool" : {
          "filter" : {
            "term" : {
              "status" : "published" (1)
            }
          }
        }
      },
      "script": {
        "source": "cosineSimilarity(params.query_vector, 'my_dense_vector') + 1.0", (2)
        "params": {
          "query_vector": [4, 3.4, -0.2]  (3)
        }
      }
    }
  }
}
```

1. To restrict the number of documents on which script score calculation is applied, provide a filter.

   要限制应用脚本分数计算的文档数量，请提供过滤器。

2. The script adds 1.0 to the cosine similarity to prevent the score from being negative.

   脚本在余弦相似度上增加 1.0 以防止分数为负。

3. To take advantage of the script optimizations, provide a query vector as a script parameter.

   要利用脚本优化，请提供查询向量作为脚本参数。

>  NOTE: If a document’s dense vector field has a number of dimensions different from the query’s vector, an error will be thrown.
>
>  如果文档的密集向量字段的维数与查询向量的维数不同，则会抛出错误。

The `dotProduct` function calculates the measure of dot product between a given query vector and document vectors.

该`dotProduct`函数计算给定查询向量和文档向量之间的点积度量。

```console
GET my-index-000001/_search
{
  "query": {
    "script_score": {
      "query" : {
        "bool" : {
          "filter" : {
            "term" : {
              "status" : "published"
            }
          }
        }
      },
      "script": {
        "source": """
          double value = dotProduct(params.query_vector, 'my_dense_vector');
          return sigmoid(1, Math.E, -value); (1)
        """,
        "params": {
          "query_vector": [4, 3.4, -0.2]
        }
      }
    }
  }
}
```

1. Using the standard sigmoid function prevents scores from being negative.

   使用标准的 sigmoid 函数可以防止分数为负。

The `l1norm` function calculates L1 distance (Manhattan distance) between a given query vector and document vectors.

该`l1norm`函数计算给定查询向量和文档向量之间的L 1距离（曼哈顿距离）。

```console
GET my-index-000001/_search
{
  "query": {
    "script_score": {
      "query" : {
        "bool" : {
          "filter" : {
            "term" : {
              "status" : "published"
            }
          }
        }
      },
      "script": {
        "source": "1 / (1 + l1norm(params.queryVector, 'my_dense_vector'))", (1)
        "params": {
          "queryVector": [4, 3.4, -0.2]
        }
      }
    }
  }
}
```

1. Unlike `cosineSimilarity` that represent similarity, `l1norm` and `l2norm` shown below represent distances or differences. This means, that the more similar the vectors are, the lower the scores will be that are produced by the `l1norm` and `l2norm` functions. Thus, as we need more similar vectors to score higher, we reversed the output from `l1norm` and `l2norm`. Also, to avoid division by 0 when a document vector matches the query exactly, we added `1` in the denominator.

   不同于`cosineSimilarity`表示的相似性，`l1norm`和 `l2norm`如下所示地表示距离或差异。这意味着，向量越相似，`l1norm`和`l2norm`函数产生的分数就越低。因此，由于我们需要更多相似的向量来获得更高的分数，我们将`l1norm`和的输出反转`l2norm`。此外，为了避免在文档向量与查询完全匹配时被 0 除，我们添加`1`了分母。

The `l2norm` function calculates L2 distance (Euclidean distance) between a given query vector and document vectors.

该`l2norm`函数计算给定查询向量和文档向量之间的L 2距离（欧几里得距离）。

```console
GET my-index-000001/_search
{
  "query": {
    "script_score": {
      "query" : {
        "bool" : {
          "filter" : {
            "term" : {
              "status" : "published"
            }
          }
        }
      },
      "script": {
        "source": "1 / (1 + l2norm(params.queryVector, 'my_dense_vector'))",
        "params": {
          "queryVector": [4, 3.4, -0.2]
        }
      }
    }
  }
}
```

> NOTE:If a document doesn’t have a value for a vector field on which a vector function is executed, an error will be thrown.
>
> 如果文档没有执行向量函数的向量字段的值，则会抛出错误。

You can check if a document has a value for the field `my_vector` by `doc['my_vector'].size() == 0`. Your overall script can look like this:

您可以检查文档中的以下字段的值`my_vector`通过 `doc['my_vector'].size() == 0`。您的整体脚本可能如下所示：

```js
"source": "doc['my_vector'].size() == 0 ? 0 : cosineSimilarity(params.queryVector, 'my_vector')"
```

The recommended way to access dense vectors is through `cosineSimilarity`, `dotProduct`, `l1norm` or `l2norm` functions. But for custom use cases, you can access dense vectors’s values directly through the following functions:

推荐方法访问密矢量是通过`cosineSimilarity`， `dotProduct`，`l1norm`或`l2norm`功能。但是对于自定义用例，您可以直接通过以下函数访问密集向量的值：

- `doc[<field>].vectorValue` – returns a vector’s value as an array of floats

  `doc[<field>].vectorValue` – 以浮点数组形式返回向量的值

- `doc[<field>].magnitude` – returns a vector’s magnitude as a float (for vectors created prior to version 7.5 the magnitude is not stored. So this function calculates it anew every time it is called).

  `doc[<field>].magnitude` – 以浮点数形式返回向量的大小（对于 7.5 版之前创建的向量，不存储大小。因此，该函数每次调用时都会重新计算）。

For example, the script below implements a cosine similarity using these two functions:

例如，下面的脚本使用这两个函数实现余弦相似度：

```console
GET my-index-000001/_search
{
  "query": {
    "script_score": {
      "query" : {
        "bool" : {
          "filter" : {
            "term" : {
              "status" : "published"
            }
          }
        }
      },
      "script": {
        "source": """
          float[] v = doc['my_dense_vector'].vectorValue;
          float vm = doc['my_dense_vector'].magnitude;
          float dotProduct = 0;
          for (int i = 0; i < v.length; i++) {
            dotProduct += v[i] * params.queryVector[i];
          }
          return dotProduct / (vm * (float) params.queryVectorMag);
        """,
        "params": {
          "queryVector": [4, 3.4, -0.2],
          "queryVectorMag": 5.25357
        }
      }
    }
  }
}
```

#####  Explain request

Using an [explain request](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-explain.html) provides an explanation of how the parts of a score were computed. The `script_score` query can add its own explanation by setting the `explanation` parameter:

使用[解释请求](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-explain.html)提供了如何计算分数部分的解释。该`script_score`查询可以通过设置添加自己的解释`explanation`参数：

```console
GET /my-index-000001/_explain/0
{
  "query": {
    "script_score": {
      "query": {
        "match": { "message": "elasticsearch" }
      },
      "script": {
        "source": """
          long count = doc['count'].value;
          double normalizedCount = count / 10;
          if (explanation != null) {
            explanation.set('normalized count = count / 10 = ' + count + ' / 10 = ' + normalizedCount);
          }
          return normalizedCount;
        """
      }
    }
  }
}
```

Note that the `explanation` will be null when using in a normal `_search` request, so having a conditional guard is best practice.

请注意，`explanation`在正常`_search`请求中使用时将为 null ，因此最好使用条件保护。

##  Wrapper query

A query that accepts any other query as base64 encoded string.

接受任何其他查询作为 base64 编码字符串的查询。

```console
GET /_search
{
  "query": {
    "wrapper": {
      "query": "eyJ0ZXJtIiA6IHsgInVzZXIuaWQiIDogImtpbWNoeSIgfX0=" (1)
    }
  }
}
```

1. Base64 encoded string: `{"term" : { "user.id" : "kimchy" }}`

   Base64 编码的字符串： `{"term" : { "user.id" : "kimchy" }}`

This query is more useful in the context of the Java high-level REST client or transport client to also accept queries as json formatted string. In these cases queries can be specified as a json or yaml formatted string or as a query builder (which is a available in the Java high-level REST client).

此查询在 Java 高级 REST 客户端或传输客户端的上下文中更有用，可以将查询作为 json 格式的字符串接受。在这些情况下，可以将查询指定为 json 或 yaml 格式的字符串或查询构建器（在 Java 高级 REST 客户端中可用）。

## Pinned Query

Promotes selected documents to rank higher than those matching a given query. This feature is typically used to guide searchers to curated documents that are promoted over and above any "organic" matches for a search. The promoted or "pinned" documents are identified using the document IDs stored in the [`_id`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html) field.

将选定的文档提升到比匹配给定查询的文档更高的排名。此功能通常用于引导搜索者找到在搜索的任何“有机”匹配项之上提升的精选文档。使用存储在[`_id`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html)字段中的文档 ID 来标识提升或“固定”的文档。

###  Example request

```console
GET /_search
{
  "query": {
    "pinned": {
      "ids": [ "1", "4", "100" ],
      "organic": {
        "match": {
          "description": "iphone"
        }
      }
    }
  }
}
```

###  Top-level parameters for `pinned`

**`ids`**

An array of [document IDs](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html) listed in the order they are to appear in results.

按在结果中出现的顺序列出的 一组[文档 ID](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html)。

**`organic`**

Any choice of query used to rank documents which will be ranked below the "pinned" document ids.

用于对文档进行排名的任何查询选择，这些文档将排在“固定”文档 ID 之下。

# `minimum_should_match` parameter

The `minimum_should_match` parameter’s possible values:

该`minimum_should_match`参数的可能值：

| Type                  | Example       | Description                                                  |
| --------------------- | ------------- | ------------------------------------------------------------ |
| Integer               | `3`           | Indicates a fixed value regardless of the number of optional clauses.<br /> 表示一个固定值，与可选子句的数量无关。 |
| Negative integer      | `-2`          | Indicates that the total number of optional clauses, minus this number should be mandatory.<br />表示可选子句的总数，减去这个数应该是强制性的。 |
| Percentage            | `75%`         | Indicates that this percent of the total number of optional clauses are necessary. The number computed from the percentage is rounded down and used as the minimum.<br />表示这个占可选子句总数的百分比是必要的。从百分比计算的数字向下舍入并用作最小值。 |
| Negative percentage   | `-25%`        | Indicates that this percent of the total number of optional clauses can be missing. The number computed from the percentage is rounded down, before being subtracted from the total to determine the minimum.<br />表示可以缺少占可选子句总数的百分比。从百分比计算的数字向下舍入，然后从总数中减去以确定最小值。 |
| Combination           | `3<90%`       | A positive integer, followed by the less-than symbol, followed by any of the previously mentioned specifiers is a conditional specification. It indicates that if the number of optional clauses is equal to (or less than) the integer, they are all required, but if it’s greater than the integer, the specification applies. In this example: if there are 1 to 3 clauses they are all required, but for 4 or more clauses only 90% are required.<br />一个正整数，后跟小于号，后跟任何前面提到的说明符是条件说明。表示可选子句的个数等于（或小于）整数时，都是必须的，大于整数时，则适用规范。在这个例子中：如果有 1 到 3 个子句，它们都是必需的，但对于 4 个或更多子句，只需要 90%。 |
| Multiple combinations | `2<-25% 9<-3` | Multiple conditional specifications can be separated by spaces, each one only being valid for numbers greater than the one before it. In this example: if there are 1 or 2 clauses both are required, if there are 3-9 clauses all but 25% are required, and if there are more than 9 clauses, all but three are required.<br />多个条件规范可以用空格分隔，每一个仅对大于前一个的数字有效。在此示例中：如果有 1 或 2 个子句都需要，如果有 3-9 个子句，但只需要 75%，如果有 9 个以上的子句，则除了三个子句外都需要。 |

**NOTE:**

When dealing with percentages, negative values can be used to get different behavior in edge cases. 75% and -25% mean the same thing when dealing with 4 clauses, but when dealing with 5 clauses 75% means 3 are required, but -25% means 4 are required.

在处理百分比时，负值可用于在边缘情况下获得不同的行为。75%和-25%在处理4个子句时是同一个意思，但是在处理5个子句时75%表示需要3个，-25%表示需要4个。

If the calculations based on the specification determine that no optional clauses are needed, the usual rules about BooleanQueries still apply at search time (a BooleanQuery containing no required clauses must still match at least one optional clause)

如果基于规范的计算确定不需要可选子句，则有关 BooleanQueries 的通常规则在搜索时仍然适用（不包含必需子句的 BooleanQuery 仍必须至少匹配一个可选子句）

No matter what number the calculation arrives at, a value greater than the number of optional clauses, or a value less than 1 will never be used. (ie: no matter how low or how high the result of the calculation result is, the minimum number of required matches will never be lower than 1 or greater than the number of clauses.

无论计算得出什么数字，都不会使用大于可选子句数量的值或小于 1 的值。（即：无论计算结果的结果多低或多高，所需的最小匹配数永远不会低于1或大于子句数。

# `rewrite` parameter

> WARNING: This parameter is for expert users only. Changing the value of this parameter can impact search performance and relevance.
>
> 此参数仅供专家用户使用。更改此参数的值会影响搜索性能和相关性。

Elasticsearch uses [Apache Lucene](https://lucene.apache.org/core/) internally to power indexing and searching. In their original form, Lucene cannot execute the following queries:

Elasticsearch 在内部使用[Apache Lucene](https://lucene.apache.org/core/)来支持索引和搜索。在其原始形式中，Lucene 无法执行以下查询：

- [`fuzzy`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-fuzzy-query.html)
- [`prefix`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html)
- [`query_string`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html)
- [`regexp`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-regexp-query.html)
- [`wildcard`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-wildcard-query.html)

To execute them, Lucene changes these queries to a simpler form, such as a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) or a [bit set](https://en.wikipedia.org/wiki/Bit_array).

为了执行它们，Lucene 将这些查询更改为更简单的形式，例如 [`bool`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)或 [位集](https://en.wikipedia.org/wiki/Bit_array)。

The `rewrite` parameter determines:

该`rewrite`参数决定：

- How Lucene calculates the relevance scores for each matching document

  Lucene 如何计算每个匹配文档的相关性分数

- Whether Lucene changes the original query to a `bool` query or bit set

  Lucene 是否将原始查询更改为`bool` 查询或位集

- If changed to a `bool` query, which `term` query clauses are included

  如果改为`bool`查询，`term`包含哪些查询子句

###  Valid values

**`constant_score` (Default)**

Uses the `constant_score_boolean` method for fewer matching terms. Otherwise, this method finds all matching terms in sequence and returns matching documents using a bit set.

`constant_score_boolean`对较少匹配项 使用该方法。否则，此方法按顺序查找所有匹配项并使用位集返回匹配文档。

**`constant_score_boolean`**

Assigns each document a relevance score equal to the `boost` parameter.

为每个文档分配一个与`boost` 参数相等的相关性分数。

This method changes the original query to a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html). This `bool` query contains a `should` clause and [`term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html) for each matching term.

此方法将原始查询更改为[`bool` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)。此`bool`查询包含每个匹配项的`should`子句和 [`term`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)。

This method can cause the final `bool` query to exceed the clause limit in the [`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) setting. If the query exceeds this limit, Elasticsearch returns an error.

此方法会导致最终`bool`查询超出[`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) 设置中的子句限制 。如果查询超过此限制，Elasticsearch 将返回错误。

**`scoring_boolean`**

Calculates a relevance score for each matching document.

计算每个匹配文档的相关性分数。

This method changes the original query to a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html). This `bool` query contains a `should` clause and [`term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html) for each matching term.

此方法将原始查询更改为[`bool` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)。此`bool`查询包含每个匹配项的`should`子句和 [`term`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)。

This method can cause the final `bool` query to exceed the clause limit in the [`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) setting. If the query exceeds this limit, Elasticsearch returns an error.

此方法会导致最终`bool`查询超出[`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) 设置中的子句限制 。如果查询超过此限制，Elasticsearch 将返回错误。

**`top_terms_blended_freqs_N`**

Calculates a relevance score for each matching document as if all terms had the same frequency. This frequency is the maximum frequency of all matching terms.

计算每个匹配文档的相关性分数，就好像所有术语都具有相同的频率。该频率是所有匹配项的最大频率。

This method changes the original query to a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html). This `bool` query contains a `should` clause and [`term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html) for each matching term.

此方法将原始查询更改为[`bool` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)。此`bool`查询包含每个匹配项的`should`子句和 [`term`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)。

The final `bool` query only includes `term` queries for the top `N` scoring terms.

最终`bool`查询仅包括`term`对`N`得分最高的术语的查询。

You can use this method to avoid exceeding the clause limit in the [`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) setting.

您可以使用此方法来避免超出[`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) 设置中的子句限制 。

**`top_terms_boost_N`**

Assigns each matching document a relevance score equal to the `boost` parameter.

为每个匹配的文档分配一个与`boost`参数相等的相关性分数。

This method changes the original query to a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html). This `bool` query contains a `should` clause and [`term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html) for each matching term.

此方法将原始查询更改为[`bool` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)。此`bool`查询包含每个匹配项的`should`子句和 [`term`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)。

The final `bool` query only includes `term` queries for the top `N` terms.

最终`bool`查询仅包括`term`对顶级`N`术语的查询。

You can use this method to avoid exceeding the clause limit in the [`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) setting.

您可以使用此方法来避免超出[`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) 设置中的子句限制 。

**`top_terms_N`**

Calculates a relevance score for each matching document.

计算每个匹配文档的相关性分数。

This method changes the original query to a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html). This `bool` query contains a `should` clause and [`term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html) for each matching term.

此方法将原始查询更改为[`bool` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)。此`bool`查询包含每个匹配项的`should`子句和 [`term`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)。

The final `bool` query only includes `term` queries for the top `N` scoring terms.

最终`bool`查询仅包括`term`对`N`得分最高的术语的查询。

You can use this method to avoid exceeding the clause limit in the [`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) setting.

您可以使用此方法来避免超出[`indices.query.bool.max_clause_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-settings.html#indices-query-bool-max-clause-count) 设置中的子句限制 。

###  Performance considerations for the `rewrite` parameter

For most uses, we recommend using the `constant_score`, `constant_score_boolean`, or `top_terms_boost_N` rewrite methods.

对于大多数应用，我们建议使用`constant_score`， `constant_score_boolean`或`top_terms_boost_N`重写方法。

Other methods calculate relevance scores. These score calculations are often expensive and do not improve query results.

其他方法计算相关性分数。这些分数计算通常很昂贵，并且不会改善查询结果。

# Regular expression syntax

A [regular expression](https://en.wikipedia.org/wiki/Regular_expression) is a way to match patterns in data using placeholder characters, called operators.

[正则表达式](https://en.wikipedia.org/wiki/Regular_expression)是一种方法来符合使用占位符的字符，称为操作符。

Elasticsearch supports regular expressions in the following queries:

Elasticsearch 支持以下查询中的正则表达式：

- [`regexp`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-regexp-query.html)
- [`query_string`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-query-string-query.html)

Elasticsearch uses [Apache Lucene](https://lucene.apache.org/core/)'s regular expression engine to parse these queries.

Elasticsearch 使用[Apache Lucene](https://lucene.apache.org/core/)的正则表达式引擎来解析这些查询。

##  Reserved characters

Lucene’s regular expression engine supports all Unicode characters. However, the following characters are reserved as operators:

Lucene 的正则表达式引擎支持所有 Unicode 字符。但是，以下字符被保留为运算符：

```
. ? + * | { } [ ] ( ) " \
```

Depending on the [optional operators](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html#regexp-optional-operators) enabled, the following characters may also be reserved:

根据启用的[可选运算符](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html#regexp-optional-operators)，还可以保留以下字符：

```
# @ & < >  ~
```

To use one of these characters literally, escape it with a preceding backslash or surround it with double quotes. For example:

要从字面上使用这些字符之一，请使用前面的反斜杠对其进行转义或用双引号将其括起来。例如：

```
\@                  # renders as a literal '@'
\\                  # renders as a literal '\'
"john@smith.com"    # renders as 'john@smith.com'
```

##  Standard operators

Lucene’s regular expression engine does not use the [Perl Compatible Regular Expressions (PCRE)](https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions) library, but it does support the following standard operators.

Lucene 的正则表达式引擎不使用 [Perl Compatible Regular Expressions (PCRE)](https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions)库，但它支持以下标准运算符。

- **`.`**

  Matches any character. For example:

  匹配任何字符。例如：

  ```
  ab.     # matches 'aba', 'abb', 'abz', etc.
  ```

- **`?`**

  Repeat the preceding character zero or one times. Often used to make the preceding character optional. For example:

  重复前面的字符零次或一次。通常用于使前面的字符可选。例如：

  ```
  abc?     # matches 'ab' and 'abc'
  ```

- **`+`**


  Repeat the preceding character one or more times. For example:

  重复前面的字符一次或多次。例如：

  ```
  ab+     # matches 'ab', 'abb', 'abbb', etc.
  ```

- **`\*`**

  Repeat the preceding character zero or more times. For example:

  重复前面的字符零次或多次。例如：

  ```
  ab*     # matches 'a', 'ab', 'abb', 'abbb', etc.
  ```

- **`{}`**

  Minimum and maximum number of times the preceding character can repeat. For example:

  前一个字符可以重复的最小和最大次数。例如：

  ```
  a{2}    # matches 'aa'
  a{2,4}  # matches 'aa', 'aaa', and 'aaaa'
  a{2,}   # matches 'a` repeated two or more times
  ```

- **`|`**

  OR operator. The match will succeed if the longest pattern on either the left side OR the right side matches. For example:

  OR 运算符。如果左侧或右侧的最长模式匹配，则匹配将成功。例如：

  ```
  abc|xyz  # matches 'abc' and 'xyz'
  ```

- **`( … )`**

  Forms a group. You can use a group to treat part of the expression as a single character. For example:

  形成一个群体。您可以使用组将部分表达式视为单个字符。例如：

  ```
  abc(def)?  # matches 'abc' and 'abcdef' but not 'abcd'
  ```

- **`[ … ]`**

  Match one of the characters in the brackets. For example:

  匹配括号中的字符之一。例如：

  ```
  [abc]   # matches 'a', 'b', 'c'
  ```

  Inside the brackets, `-` indicates a range unless `-` is the first character or escaped. For example:

  括号内，`-`表示一个范围，除非`-`是第一个字符或转义。例如：

  ```
  [a-c]   # matches 'a', 'b', or 'c'
  [-abc]  # '-' is first character. Matches '-', 'a', 'b', or 'c'
  [abc\-] # Escapes '-'. Matches 'a', 'b', 'c', or '-'
  ```

  A `^` before a character in the brackets negates the character or range. For example:

  `^`括号中字符前的A否定该字符或范围。例如：

  ```
  [^abc]      # matches any character except 'a', 'b', or 'c'
  [^a-c]      # matches any character except 'a', 'b', or 'c'
  [^-abc]     # matches any character except '-', 'a', 'b', or 'c'
  [^abc\-]    # matches any character except 'a', 'b', 'c', or '-'
  ```

##  Optional operators

You can use the `flags` parameter to enable more optional operators for Lucene’s regular expression engine.

您可以使用该`flags`参数为 Lucene 的正则表达式引擎启用更多可选运算符。

To enable multiple operators, use a `|` separator. For example, a `flags` value of `COMPLEMENT|INTERVAL` enables the `COMPLEMENT` and `INTERVAL` operators.

要启用多个运算符，请使用`|`分隔符。例如，`flags`值`COMPLEMENT|INTERVAL`启用`COMPLEMENT`和`INTERVAL`运算符。

##  Valid values

**`ALL` (Default)**

Enables all optional operators.

启用所有可选运算符。

**`COMPLEMENT`**

Enables the `~` operator. You can use `~` to negate the shortest following pattern. For example:

启用`~`操作员。您可以使用`~`来否定最短的跟随模式。例如：

```
a~bc   # matches 'adc' and 'aec' but not 'abc'
```

**`INTERVAL`**

Enables the `<>` operators. You can use `<>` to match a numeric range. For example:

启用`<>`运算符。您可以使用`<>`来匹配数字范围。例如：

```
foo<1-100>      # matches 'foo1', 'foo2' ... 'foo99', 'foo100'
foo<01-100>     # matches 'foo01', 'foo02' ... 'foo99', 'foo100'
```

**`INTERSECTION`**

Enables the `&` operator, which acts as an AND operator. The match will succeed if patterns on both the left side AND the right side matches. For example:

启用`&`作为 AND 运算符的运算符。如果左侧和右侧的模式都匹配，则匹配将成功。例如：

```
aaa.+&.+bbb  # matches 'aaabbb'
```

**`ANYSTRING`**

Enables the `@` operator. You can use `@` to match any entire string.

启用`@`操作员。您可以使用`@`来匹配任何整个字符串。

You can combine the `@` operator with `&` and `~` operators to create an "everything except" logic. For example:

您可以将`@`运算符与`&`和`~`运算符结合使用以创建“除此之外的所有内容”逻辑。例如：

```
@&~(abc.+)  # matches everything except terms beginning with 'abc'
```

###  Unsupported operators

Lucene’s regular expression engine does not support anchor operators, such as `^` (beginning of line) or `$` (end of line). To match a term, the regular expression must match the entire string.

Lucene 的正则表达式引擎不支持锚操作符，例如 `^`(beginning of line) 或`$`(end of line)。要匹配一个术语，正则表达式必须匹配整个字符串。