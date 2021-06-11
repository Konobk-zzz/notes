#  Collapse search results

You can use the `collapse` parameter to collapse search results based on field values. The collapsing is done by selecting only the top sorted document per collapse key.

你可以使用`collapse`参数基于字段来折叠搜索结果。折叠是通过选择每个折叠键排序最靠前的文档来完成的。

For example, the following search collapses results by `user.id` and sorts them by `http.response.bytes`.

举例，下面这个检索折叠结果通过`user.id`字段和通过`http.response.bytes`排序。

```json
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "user.id"                
  },
  "sort": [ "http.response.bytes" ],  
  "from": 10                          
}
```

1. Collapse the result set using the "user.id" field
2.  Sort the results by `http.response.bytes`
3.  define the offset of the first collapsed result

> **WARNING:** The total number of hits in the response indicates the number of matching documents without collapsing. The total number of distinct group is unknown.
>
> 警告：响应中命中的文档总数是不包括折叠的。不同分组的总数是未知的。

The field used for collapsing must be a single valued [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html) or [`numeric`](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) field with [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/master/doc-values.html) activated

被使用于折叠的字段必须是单独值的 [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html) 或[`numeric`](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) 字段，且 [`doc_values`](https://www.elastic.co/guide/en/elasticsearch/reference/master/doc-values.html) 是激活的。

> **NOTE:** The collapsing is applied to the top hits only and does not affect aggregations.
>
> 注意：折叠只被应用与最顶端的命中且不会影响聚合。

## Expand collapse results

It is also possible to expand each collapsed top hits with the `inner_hits` option.

也可以带上`inner_hits` 选项来展开每一个被折叠的顶端命中。

```json
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "user.id",                       
    "inner_hits": {
      "name": "most_recent",                  
      "size": 5,                              
      "sort": [ { "@timestamp": "asc" } ]     
    },
    "max_concurrent_group_searches": 4        
  },
  "sort": [ "http.response.bytes" ]
}

```

1. collapse the result set using the "user.id" field

2. the name used for the inner hit section in the response

3. the number of inner_hits to retrieve per collapse key

4. how to sort the document inside each group

5. the number of concurrent requests allowed to retrieve the inner_hits per group

See [inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/master/inner-hits.html) for the complete list of supported options and the format of the response.

查看 [inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/master/inner-hits.html) 支持选项的完整列表和格式化响应。

It is also possible to request multiple `inner_hits` for each collapsed hit. This can be useful when you want to get multiple representations of the collapsed hits.

也可以在请求中为每一个折叠命中使用多个 `inner_hits` 。当您想要获得折叠后的匹配的多种表示形式时，此功能很有用。

```json
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "user.id",                      
      "inner_hits": [
      {
        "name": "largest_responses",         
        "size": 3,
        "sort": [ "http.response.bytes" ]
      },
      {
        "name": "most_recent",               
        "size": 3,
        "sort": [ { "@timestamp": "asc" } ]
      }
    ]
  },
  "sort": [ "http.response.bytes" ]
}
```

1. collapse the result set using the "user.id" field

2. return the three largest HTTP responses for the user

3. return the three most recent HTTP responses for the user

The expansion of the group is done by sending an additional query for each `inner_hit` request for each collapsed hit returned in the response. This can significantly slow things down if you have too many groups and/or `inner_hit` requests.

展开分组是通过发送一个额外请求查询每一个折叠的命中的 `inner_hit`请求实现的。如果你使用了很多分组或`inner_hit`会极大程度的拖慢查询速度。

The `max_concurrent_group_searches` request parameter can be used to control the maximum number of concurrent searches allowed in this phase. The default is based on the number of data nodes and the default search thread pool size.

 `max_concurrent_group_searches` 请求参数可以控制查询当前阶段所允许的最大并发数量。

> **WARNING:** `collapse` cannot be used in conjunction with [scroll](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results), [rescore](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#rescore) or [search after](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#search-after).
>
> 警告：`collapse`不能用于连接词如 [scroll](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results), [rescore](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#rescore) or [search after](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#search-after).

##  Second level of collapsing

Second level of collapsing is also supported and is applied to `inner_hits`.

支持二级折叠，二级折叠被应用于`inner_hits`

For example, the following search collapses results by `geo.country_name`. Within each `geo.country_name`, inner hits are collapsed by `user.id`.

举例，下面的检索结果通过`geo.country_name`折叠。每一个`geo.country_name`内部命中通过 `user.id`折叠。

```json
GET /my-index-000001/_search
{
  "query": {
    "match": {
      "message": "GET /search"
    }
  },
  "collapse": {
    "field": "geo.country_name",
    "inner_hits": {
      "name": "by_location",
      "collapse": { "field": "user.id" },
      "size": 3
    }
  }
}
```

Response:

```json
{
  ...
  "hits": [
    {
      "_index": "my-index-000001",
      "_type": "_doc",
      "_id": "9",
      "_score": ...,
      "_source": {...},
      "fields": { "geo": { "country_name": [ "UK" ] }},
      "inner_hits": {
        "by_location": {
          "hits": {
            ...,
            "hits": [
              {
                ...
                "fields": { "user": "id": { [ "user124" ] }}
              },
              {
                ...
                "fields": { "user": "id": { [ "user589" ] }}
              },
              {
                ...
                "fields": { "user": "id": { [ "user001" ] }}
              }
            ]
          }
        }
      }
    },
    {
      "_index": "my-index-000001",
      "_type": "_doc",
      "_id": "1",
      "_score": ..,
      "_source": {...
      },
      "fields": { "geo": { "country_name": [ "Canada" ] }},
      "inner_hits": {
        "by_location": {
          "hits": {
            ...,
            "hits": [
              {
                ...
                "fields": { "user": "id": { [ "user444" ] }}
              },
              {
                ...
                "fields": { "user": "id": { [ "user1111" ] }
              },
              {
                ...
                  "fields": { "user": "id": { [ "user999" ] }}
              }
            ]
          }
        }
      }
    },
    ...
  ]
}
```

> **NOTE:** Second level of collapsing doesn’t allow `inner_hits`.
>
> 注意：二级折叠不允许使用`inner_hits`

#  Filter search results

You can use two methods to filter search results:

你有两种方法来过滤检索结果：

- Use a boolean query with a `filter` clause. Search requests apply [boolean filters](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) to both search hits and [aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations.html).

  使用一个boolean查询带`filter`条件。检索请求应用 [boolean filters](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) 同时检索命中和 [聚合](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations.html).

- Use the search API’s `post_filter` parameter. Search requests apply [post filters](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#post-filter) only to search hits, not aggregations. You can use a post filter to calculate aggregations based on a broader result set, and then further narrow the results.

  使用检索API `post_filter`  参数。检索请求只对检索命中使用 [post filters](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#post-filter) ，不包括聚合。你可以使用 后置过滤器 去计算基于广泛结果集合的聚合，然后进一步缩小结果。
  
  You can also [rescore](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#rescore) hits after the post filter to improve relevance and reorder results.
  
  你也可以在后置过滤器后对命中进行 [rescore](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#rescore) （重新计算分数）来提升关联性和重排序结果。

## Post filter

When you use the `post_filter` parameter to filter search results, the search hits are filtered after the aggregations are calculated. A post filter has no impact on the aggregation results.

当你使用`post_filter` 参数来过滤检索结果，检索命中在聚合后才被计算过滤。一个后置过滤不能对聚合结果造成影响。

For example, you are selling shirts that have the following properties:

举例，你正在销售的短袖有如下属性：

```json
PUT /shirts
{
  "mappings": {
    "properties": {
      "brand": { "type": "keyword"},
      "color": { "type": "keyword"},
      "model": { "type": "keyword"}
    }
  }
}

PUT /shirts/_doc/1?refresh
{
  "brand": "gucci",
  "color": "red",
  "model": "slim"
}
```

Imagine a user has specified two filters:

想象一个用户指定了两点过滤：

`color:red` and `brand:gucci`. You only want to show them red shirts made by Gucci in the search results. Normally you would do this with a [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html):

`color:red` 和 `brand:gucci`。你只想要在结果展示Gucci 制造的红色短袖。通常你会通过 [`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html):

``` json
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  }
}
```

However, you would also like to use *faceted navigation* to display a list of other options that the user could click on. Perhaps you have a `model` field that would allow the user to limit their search results to red Gucci `t-shirts` or `dress-shirts`.

然而你可能想要使用 *分面导航* 来展示一个其他选项的列表使用户可以点击。也许你有一个 `model` 字段来允许用户限制他们的检索为Gucci红色的 `t-shirts `或 `dress-shirts`

This can be done with a [`terms` aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html):

这可以通过 [`terms` aggregation](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-terms-aggregation.html) 来实现

```json
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  },
  "aggs": {
    "models": {
      "terms": { "field": "model" } 
    }
  }
}
```

1.  Returns the most popular models of red shirts by Gucci.

But perhaps you would also like to tell the user how many Gucci shirts are available in **other colors**. If you just add a `terms` aggregation on the `color` field, you will only get back the color `red`, because your query returns only red shirts by Gucci.

也许你也想要告诉用户有多少其他颜色 Gucci shirts 可用。如果你只是添加一个 `terms` 聚合 `color` 字段，将会只返回红色，因为查询只返回Gucci的红色短袖。

Instead, you want to include shirts of all colors during aggregation, then apply the `colors` filter only to the search results. This is the purpose of the `post_filter`:

最为替代，你想要包含所有颜色短袖在聚合期间， `colors` 过滤只应用在检索结果。这就是 `post_filter`的目的。

```json
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "brand": "gucci" } 
      }
    }
  },
  "aggs": {
    "colors": {
      "terms": { "field": "color" } 
    },
    "color_red": {
      "filter": {
        "term": { "color": "red" } 
      },
      "aggs": {
        "models": {
          "terms": { "field": "model" } 
        }
      }
    }
  },
  "post_filter": { 
    "term": { "color": "red" }
  }
}
```

1. The main query now finds all shirts by Gucci, regardless of color.

2. The colors agg returns popular colors for shirts by Gucci.

3. The color_red agg limits the models sub-aggregation to red Gucci shirts.

4. Finally, the post_filter removes colors other than red from the search hits.

##  Rescore filtered search results

Rescoring can help to improve precision by reordering just the top (eg 100 - 500) documents returned by the [`query`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-query) and [`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#post-filter) phases, using a secondary (usually more costly) algorithm, instead of applying the costly algorithm to all documents in the index.

通过使用辅助（通常更昂贵）算法，而不是对索引中的所有文档应用昂贵的算法，仅通过重新排序由[`query`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-query)和 [`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#post-filter)阶段返回的最前面的文档（例如100-500），排序可以帮助提高精度 。

A `rescore` request is executed on each shard before it returns its results to be sorted by the node handling the overall search request.

`rescore`在每个分片上执行请求之前，该请求将返回其结果，并由处理整个搜索请求的节点将其排序。

Currently the rescore API has only one implementation: the query rescorer, which uses a query to tweak the scoring. In the future, alternative rescorers may be made available, for example, a pair-wise rescorer.

当前，rescore API仅具有一种实现：查询rescorer，该查询使用查询来调整得分。将来，可能会提供替代的记录器，例如，成对记录器。

> **NOTE:** An error will be thrown if an explicit [`sort`](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html) (other than `_score` in descending order) is provided with a `rescore` query.
>
> 注意：如果为查询提供显式[`sort`](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html) （而不是`_score`降序排列），则会引发错误`rescore`。

>  **NOTE:** when exposing pagination to your users, you should not change `window_size` as you step through each page (by passing different `from` values) since that can alter the top hits causing results to confusingly shift as the user steps through pages.
>
>  注意：当像用户展示分页时，不应该改变作为步长的 每页（不同的 `from` 值）`window_size` （窗口大小），因为这可能会更改顶部匹配，从而导致结果在用户逐步浏览页面时引起混乱。

## Query rescorer

The query rescorer executes a second query only on the Top-K results returned by the [`query`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-query) and [`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#post-filter) phases. The number of docs which will be examined on each shard can be controlled by the `window_size` parameter, which defaults to 10.

查询记录器仅对[`query`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-query)和 [`post_filter`](https://www.elastic.co/guide/en/elasticsearch/reference/master/filter-search-results.html#post-filter)阶段返回的前K个结果执行第二个查询。每个分片上要检查的文档数量可以由`window_size`参数控制，该参数默认为10。

By default the scores from the original query and the rescore query are combined linearly to produce the final `_score` for each document. The relative importance of the original query and of the rescore query can be controlled with the `query_weight` and `rescore_query_weight` respectively. Both default to `1`.

每一个文档的最终得分 `_score` 根据原始查询分数和重新计算得分查询线性组合得到。重计算得分查询相较原始查询的关联性提升可以被 `query_weight` 和 `rescore_query_weight`分别控制。默认都是1。

For example:

```json
POST /_search
{
   "query" : {
      "match" : {
         "message" : {
            "operator" : "or",
            "query" : "the quick brown"
         }
      }
   },
   "rescore" : {
      "window_size" : 50,
      "query" : {
         "rescore_query" : {
            "match_phrase" : {
               "message" : {
                  "query" : "the quick brown",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }
}
```

The way the scores are combined can be controlled with the `score_mode`:

分数的组合方式可以通过以下方式控制`score_mode`：

| Score Mode | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| `total`    | Add the original score and the rescore query score. The default.<br />原始分数和重新评分查询分数相加。默认值。 |
| `multiply` | Multiply the original score by the rescore query score. Useful for [`function query`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html) rescores.<br />将原始分数乘以重新评分查询分数。对于[`function query`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-function-score-query.html)重新计分很有用。 |
| `avg`      | Average the original score and the rescore query score.<br />平均原始分数和重新评分查询分数。 |
| `max`      | Take the max of original score and the rescore query score.<br />取得原始分数和重新评分查询分数的最大值。 |
| `min`      | Take the min of the original score and the rescore query score.<br />取原始分数和重新评分查询分数的最小值。 |

## Multiple rescores

It is also possible to execute multiple rescores in sequence:

也可以按顺序执行多个重新评分：

```json
POST /_search
{
   "query" : {
      "match" : {
         "message" : {
            "operator" : "or",
            "query" : "the quick brown"
         }
      }
   },
   "rescore" : [ {
      "window_size" : 100,
      "query" : {
         "rescore_query" : {
            "match_phrase" : {
               "message" : {
                  "query" : "the quick brown",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }, {
      "window_size" : 10,
      "query" : {
         "score_mode": "multiply",
         "rescore_query" : {
            "function_score" : {
               "script_score": {
                  "script": {
                    "source": "Math.log10(doc.count.value + 2)"
                  }
               }
            }
         }
      }
   } ]
}
```

The first one gets the results of the query then the second one gets the results of the first, etc. The second rescore will "see" the sorting done by the first rescore so it is possible to use a large window on the first rescore to pull documents into a smaller window for the second rescore.

第一个获得查询的结果，然后第二个获得查询的结果，依此类推。第二个重新评分将“看到”第一个重新评分完成的排序，因此可以在第一个重新评分上使用大窗口来查询将文档拉入较小的窗口中以进行第二次重新评分。

#  Highlighting

Highlighters enable you to get highlighted snippets from one or more fields in your search results so you can show users where the query matches are. When you request highlights, the response contains an additional `highlight` element for each search hit that includes the highlighted fields and the highlighted fragments.

> **NOTE:** Highlighters don’t reflect the boolean logic of a query when extracting terms to highlight. Thus, for some complex boolean queries (e.g nested boolean queries, queries using `minimum_should_match` etc.), parts of documents may be highlighted that don’t correspond to query matches.

Highlighting requires the actual content of a field. If the field is not stored (the mapping does not set `store` to `true`), the actual `_source` is loaded and the relevant field is extracted from `_source`.

For example, to get highlights for the `content` field in each search hit using the default highlighter, include a `highlight` object in the request body that specifies the `content` field:

```json
GET /_search
{
  "query": {
    "match": { "content": "kimchy" }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
```

Elasticsearch supports three highlighters: `unified`, `plain`, and `fvh` (fast vector highlighter). You can specify the highlighter `type` you want to use for each field.

## Unified highlighter

The `unified` highlighter uses the Lucene Unified Highlighter. This highlighter breaks the text into sentences and uses the BM25 algorithm to score individual sentences as if they were documents in the corpus. It also supports accurate phrase and multi-term (fuzzy, prefix, regex) highlighting. This is the default highlighter.

## Plain highlighter

The `plain` highlighter uses the standard Lucene highlighter. It attempts to reflect the query matching logic in terms of understanding word importance and any word positioning criteria in phrase queries.

> **WARNING:** The `plain` highlighter works best for highlighting simple query matches in a single field. To accurately reflect query logic, it creates a tiny in-memory index and re-runs the original query criteria through Lucene’s query execution planner to get access to low-level match information for the current document. This is repeated for every field and every document that needs to be highlighted. If you want to highlight a lot of fields in a lot of documents with complex queries, we recommend using the `unified` highlighter on `postings` or `term_vector` fields.

## Fast vector highlighter

The `fvh` highlighter uses the Lucene Fast Vector highlighter. This highlighter can be used on fields with `term_vector` set to `with_positions_offsets` in the mapping. The fast vector highlighter:

- Can be customized with a [`boundary_scanner`](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#boundary-scanners).
- Requires setting `term_vector` to `with_positions_offsets` which increases the size of the index
- Can combine matches from multiple fields into one result. See `matched_fields`
- Can assign different weights to matches at different positions allowing for things like phrase matches being sorted above term matches when highlighting a Boosting Query that boosts phrase matches over term matches

> **WARNING:** The `fvh` highlighter does not support span queries. If you need support for span queries, try an alternative highlighter, such as the `unified` highlighter.

## Offsets strategy

To create meaningful search snippets from the terms being queried, the highlighter needs to know the start and end character offsets of each word in the original text. These offsets can be obtained from:

- The postings list. If `index_options` is set to `offsets` in the mapping, the `unified` highlighter uses this information to highlight documents without re-analyzing the text. It re-runs the original query directly on the postings and extracts the matching offsets from the index, limiting the collection to the highlighted documents. This is important if you have large fields because it doesn’t require reanalyzing the text to be highlighted. It also requires less disk space than using `term_vectors`.
- Term vectors. If `term_vector` information is provided by setting `term_vector` to `with_positions_offsets` in the mapping, the `unified` highlighter automatically uses the `term_vector` to highlight the field. It’s fast especially for large fields (> `1MB`) and for highlighting multi-term queries like `prefix` or `wildcard` because it can access the dictionary of terms for each document. The `fvh` highlighter always uses term vectors.
- Plain highlighting. This mode is used by the `unified` when there is no other alternative. It creates a tiny in-memory index and re-runs the original query criteria through Lucene’s query execution planner to get access to low-level match information on the current document. This is repeated for every field and every document that needs highlighting. The `plain` highlighter always uses plain highlighting.

> **WARNING:** Plain highlighting for large texts may require substantial amount of time and memory. To protect against this, the maximum number of text characters that will be analyzed has been limited to 1000000. This default limit can be changed for a particular index with the index setting [`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-analyzed-offset).

##  Highlighting settings

Highlighting settings can be set on a global level and overridden at the field level.

### **boundary_chars**

A string that contains each boundary character. Defaults to `.,!? \t\n`.

### **boundary_max_scan**

How far to scan for boundary characters. Defaults to `20`.

### **boundary_scanner**

Specifies how to break the highlighted fragments: `chars`, `sentence`, or `word`. Only valid for the `unified` and `fvh` highlighters. Defaults to `sentence` for the `unified` highlighter. Defaults to `chars` for the `fvh` highlighter.

- **`chars`**

  Use the characters specified by `boundary_chars` as highlighting boundaries. The `boundary_max_scan` setting controls how far to scan for boundary characters. Only valid for the `fvh`highlighter.

- **`sentence`**

  Break highlighted fragments at the next sentence boundary, as determined by Java’s [BreakIterator](https://docs.oracle.com/javase/8/docs/api/java/text/BreakIterator.html). You can specify the locale to use with `boundary_scanner_locale`.

  > **NOTE:** When used with the `unified` highlighter, the `sentence` scanner splits sentences bigger than `fragment_size` at the first word boundary next to `fragment_size`. You can set `fragment_size` to 0 to never split any sentence.

- **`word`**

  Break highlighted fragments at the next word boundary, as determined by Java’s [BreakIterator](https://docs.oracle.com/javase/8/docs/api/java/text/BreakIterator.html). You can specify the locale to use with `boundary_scanner_locale`.

### **boundary_scanner_locale**

Controls which locale is used to search for sentence and word boundaries. This parameter takes a form of a language tag, e.g. `"en-US"`, `"fr-FR"`, `"ja-JP"`. More info can be found in the [Locale Language Tag](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#forLanguageTag-java.lang.String-) documentation. The default value is [Locale.ROOT](https://docs.oracle.com/javase/8/docs/api/java/util/Locale.html#ROOT).

### **encoder**

Indicates if the snippet should be HTML encoded: `default` (no encoding) or `html` (HTML-escape the snippet text and then insert the highlighting tags)

### **fields**

Specifies the fields to retrieve highlights for. You can use wildcards to specify fields. For example, you could specify `comment_*` to get highlights for all [text](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) and [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html) fields that start with `comment_`.

> **NOTE:** Only text and keyword fields are highlighted when you use wildcards. If you use a custom mapper and want to highlight on a field anyway, you must explicitly specify that field name.

### **force_source**

Highlight based on the source even if the field is stored separately. Defaults to `false`.

### **fragmenter**

Specifies how text should be broken up in highlight snippets: `simple` or `span`. Only valid for the `plain` highlighter. Defaults to `span`.

- **`simple`**

  Breaks up text into same-sized fragments.

- **`span`**

  Breaks up text into same-sized fragments, but tries to avoid breaking up text between highlighted terms. This is helpful when you’re querying for phrases. Default.

### **fragment_offset**

Controls the margin from which you want to start highlighting. Only valid when using the `fvh` highlighter.

### **fragment_size**

The size of the highlighted fragment in characters. Defaults to 100.

### **highlight_query**

Highlight matches for a query other than the search query. This is especially useful if you use a rescore query because those are not taken into account by highlighting by default.

> **IMPORTANT:** Elasticsearch does not validate that `highlight_query` contains the search query in any way so it is possible to define it so legitimate query results are not highlighted. Generally, you should include the search query as part of the `highlight_query`.

### **matched_fields**

Combine matches on multiple fields to highlight a single field. This is most intuitive for multifields that analyze the same string in different ways. All `matched_fields` must have `term_vector` set to `with_positions_offsets`, but only the field to which the matches are combined is loaded so only that field benefits from having `store` set to `yes`. Only valid for the `fvh` highlighter.

### **no_match_size**

The amount of text you want to return from the beginning of the field if there are no matching fragments to highlight. Defaults to 0 (nothing is returned).

### **number_of_fragments**

The maximum number of fragments to return. If the number of fragments is set to 0, no fragments are returned. Instead, the entire field contents are highlighted and returned. This can be handy when you need to highlight short texts such as a title or address, but fragmentation is not required. If `number_of_fragments` is 0, `fragment_size` is ignored. Defaults to 5.

### **order**

Sorts highlighted fragments by score when set to `score`. By default, fragments will be output in the order they appear in the field (order: `none`). Setting this option to `score` will output the most relevant fragments first. Each highlighter applies its own logic to compute relevancy scores. See the document [How highlighters work internally](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#how-es-highlighters-work-internally) for more details how different highlighters find the best fragments.

### **phrase_limit**

Controls the number of matching phrases in a document that are considered. Prevents the `fvh` highlighter from analyzing too many phrases and consuming too much memory. When using `matched_fields`, `phrase_limit` phrases per matched field are considered. Raising the limit increases query time and consumes more memory. Only supported by the `fvh` highlighter. Defaults to 256.

### **pre_tags**

Use in conjunction with `post_tags` to define the HTML tags to use for the highlighted text. By default, highlighted text is wrapped in `<em>` and `</em>` tags. Specify as an array of strings.

### **post_tags**

Use in conjunction with `pre_tags` to define the HTML tags to use for the highlighted text. By default, highlighted text is wrapped in `<em>` and `</em>` tags. Specify as an array of strings.

### **require_field_match**

By default, only fields that contains a query match are highlighted. Set `require_field_match` to `false` to highlight all fields. Defaults to `true`.

### **max_analyzed_offset**

By default, the maximum number of characters analyzed for a highlight request is bounded by the value defined in the [`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-analyzed-offset) setting, and when the number of characters exceeds this limit an error is returned. If this setting is set to a non-negative value, the highlighting stops at this defined maximum limit, and the rest of the text is not processed, thus not highlighted and no error is returned. The [`max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#max-analyzed-offset) query setting does **not** override the [`index.highlight.max_analyzed_offset`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-analyzed-offset) which prevails when it’s set to lower value than the query setting.

### **tags_schema**

Set to `styled` to use the built-in tag schema. The `styled` schema defines the following `pre_tags` and defines `post_tags` as `</em>`.

```json
<em class="hlt1">, <em class="hlt2">, <em class="hlt3">,
<em class="hlt4">, <em class="hlt5">, <em class="hlt6">,
<em class="hlt7">, <em class="hlt8">, <em class="hlt9">,
<em class="hlt10">
```

### **type**

The highlighter to use: `unified`, `plain`, or `fvh`. Defaults to `unified`.

##  Highlighting examples

- [Override global settings](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#override-global-settings)
- [Specify a highlight query](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#specify-highlight-query)
- [Set highlighter type](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#set-highlighter-type)
- [Configure highlighting tags](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#configure-tags)
- [Highlight source](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#highlight-source)
- [Highlight all fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#highlight-all)
- [Combine matches on multiple fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#matched-fields)
- [Explicitly order highlighted fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#explicit-field-order)
- [Control highlighted fragments](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#control-highlighted-frags)
- [Highlight using the postings list](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#highlight-postings-list)
- [Specify a fragmenter for the plain highlighter](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html#specify-fragmenter)

##  Override global settings

You can specify highlighter settings globally and selectively override them for individual fields.

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "number_of_fragments" : 3,
    "fragment_size" : 150,
    "fields" : {
      "body" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] },
      "blog.title" : { "number_of_fragments" : 0 },
      "blog.author" : { "number_of_fragments" : 0 },
      "blog.comment" : { "number_of_fragments" : 5, "order" : "score" }
    }
  }
}
```

##  Specify a highlight query

You can specify a `highlight_query` to take additional information into account when highlighting. For example, the following query includes both the search query and rescore query in the `highlight_query`. Without the `highlight_query`, highlighting would only take the search query into account.

```json
GET /_search
{
  "query": {
    "match": {
      "comment": {
        "query": "foo bar"
      }
    }
  },
  "rescore": {
    "window_size": 50,
    "query": {
      "rescore_query": {
        "match_phrase": {
          "comment": {
            "query": "foo bar",
            "slop": 1
          }
        }
      },
      "rescore_query_weight": 10
    }
  },
  "_source": false,
  "highlight": {
    "order": "score",
    "fields": {
      "comment": {
        "fragment_size": 150,
        "number_of_fragments": 3,
        "highlight_query": {
          "bool": {
            "must": {
              "match": {
                "comment": {
                  "query": "foo bar"
                }
              }
            },
            "should": {
              "match_phrase": {
                "comment": {
                  "query": "foo bar",
                  "slop": 1,
                  "boost": 10.0
                }
              }
            },
            "minimum_should_match": 0
          }
        }
      }
    }
  }
}
```

##  Set highlighter type

The `type` field allows to force a specific highlighter type. The allowed values are: `unified`, `plain` and `fvh`. The following is an example that forces the use of the plain highlighter:

```json
GET /_search
{
  "query": {
    "match": { "user.id": "kimchy" }
  },
  "highlight": {
    "fields": {
      "comment": { "type": "plain" }
    }
  }
}
```

##  Configure highlighting tags

By default, the highlighting will wrap highlighted text in `<em>` and `</em>`. This can be controlled by setting `pre_tags` and `post_tags`, for example:

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "pre_tags" : ["<tag1>"],
    "post_tags" : ["</tag1>"],
    "fields" : {
      "body" : {}
    }
  }
}
```

When using the fast vector highlighter, you can specify additional tags and the "importance" is ordered.

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "pre_tags" : ["<tag1>", "<tag2>"],
    "post_tags" : ["</tag1>", "</tag2>"],
    "fields" : {
      "body" : {}
    }
  }
}
```

You can also use the built-in `styled` tag schema:

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "tags_schema" : "styled",
    "fields" : {
      "comment" : {}
    }
  }
}
```

##  Highlight on source

Forces the highlighting to highlight fields based on the source even if fields are stored separately. Defaults to `false`.

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "fields" : {
      "comment" : {"force_source" : true}
    }
  }
}
```

##  Highlight in all fields

By default, only fields that contains a query match are highlighted. Set `require_field_match` to `false` to highlight all fields.

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "require_field_match": false,
    "fields": {
      "body" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] }
    }
  }
}
```

##  Combine matches on multiple fields

> **WARNING:** This is only supported by the `fvh` highlighter

The Fast Vector Highlighter can combine matches on multiple fields to highlight a single field. This is most intuitive for multifields that analyze the same string in different ways. All `matched_fields` must have `term_vector` set to `with_positions_offsets` but only the field to which the matches are combined is loaded so only that field would benefit from having `store` set to `yes`.

In the following examples, `comment` is analyzed by the `english` analyzer and `comment.plain` is analyzed by the `standard` analyzer.

```json
GET /_search
{
  "query": {
    "query_string": {
      "query": "comment.plain:running scissors",
      "fields": [ "comment" ]
    }
  },
  "highlight": {
    "order": "score",
    "fields": {
      "comment": {
        "matched_fields": [ "comment", "comment.plain" ],
        "type": "fvh"
      }
    }
  }
}
```

The above matches both "run with scissors" and "running with scissors" and would highlight "running" and "scissors" but not "run". If both phrases appear in a large document then "running with scissors" is sorted above "run with scissors" in the fragments list because there are more matches in that fragment.

```json
GET /_search
{
  "query": {
    "query_string": {
      "query": "running scissors",
      "fields": ["comment", "comment.plain^10"]
    }
  },
  "highlight": {
    "order": "score",
    "fields": {
      "comment": {
        "matched_fields": ["comment", "comment.plain"],
        "type" : "fvh"
      }
    }
  }
}
```

The above highlights "run" as well as "running" and "scissors" but still sorts "running with scissors" above "run with scissors" because the plain match ("running") is boosted.

```json
GET /_search
{
  "query": {
    "query_string": {
      "query": "running scissors",
      "fields": [ "comment", "comment.plain^10" ]
    }
  },
  "highlight": {
    "order": "score",
    "fields": {
      "comment": {
        "matched_fields": [ "comment.plain" ],
        "type": "fvh"
      }
    }
  }
}
```

The above query wouldn’t highlight "run" or "scissor" but shows that it is just fine not to list the field to which the matches are combined (`comment`) in the matched fields.

> **NOTE:** Technically it is also fine to add fields to `matched_fields` that don’t share the same underlying string as the field to which the matches are combined. The results might not make much sense and if one of the matches is off the end of the text then the whole query will fail.

> **NOTE:** There is a small amount of overhead involved with setting `matched_fields` to a non-empty array so always prefer
>
> ```json
>  "highlight": {
>         "fields": {
>             "comment": {}
>         }
>     }
> ```
>
> to
>
> ```json
>  "highlight": {
>         "fields": {
>             "comment": {
>                 "matched_fields": ["comment"],
>                 "type" : "fvh"
>             }
>         }
>     }
> ```

##  Explicitly order highlighted fields

Elasticsearch highlights the fields in the order that they are sent, but per the JSON spec, objects are unordered. If you need to be explicit about the order in which fields are highlighted specify the `fields` as an array:

```json
GET /_search
{
  "highlight": {
    "fields": [
      { "title": {} },
      { "text": {} }
    ]
  }
}
```

None of the highlighters built into Elasticsearch care about the order that the fields are highlighted but a plugin might.

##  Control highlighted fragments

Each field highlighted can control the size of the highlighted fragment in characters (defaults to `100`), and the maximum number of fragments to return (defaults to `5`). For example:

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "fields" : {
      "comment" : {"fragment_size" : 150, "number_of_fragments" : 3}
    }
  }
}
```

On top of this it is possible to specify that highlighted fragments need to be sorted by score:

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "order" : "score",
    "fields" : {
      "comment" : {"fragment_size" : 150, "number_of_fragments" : 3}
    }
  }
}
```

If the `number_of_fragments` value is set to `0` then no fragments are produced, instead the whole content of the field is returned, and of course it is highlighted. This can be very handy if short texts (like document title or address) need to be highlighted but no fragmentation is required. Note that `fragment_size` is ignored in this case.

```json
GET /_search
{
  "query" : {
    "match": { "user.id": "kimchy" }
  },
  "highlight" : {
    "fields" : {
      "body" : {},
      "blog.title" : {"number_of_fragments" : 0}
    }
  }
}
```

When using `fvh` one can use `fragment_offset` parameter to control the margin to start highlighting from.

In the case where there is no matching fragment to highlight, the default is to not return anything. Instead, we can return a snippet of text from the beginning of the field by setting `no_match_size` (default `0`) to the length of the text that you want returned. The actual length may be shorter or longer than specified as it tries to break on a word boundary.

```json
GET /_search
{
  "query": {
    "match": { "user.id": "kimchy" }
  },
  "highlight": {
    "fields": {
      "comment": {
        "fragment_size": 150,
        "number_of_fragments": 3,
        "no_match_size": 150
      }
    }
  }
}
```

##  Highlight using the postings list

Here is an example of setting the `comment` field in the index mapping to allow for highlighting using the postings:

```json
PUT /example
{
  "mappings": {
    "properties": {
      "comment" : {
        "type": "text",
        "index_options" : "offsets"
      }
    }
  }
}
```

Here is an example of setting the `comment` field to allow for highlighting using the `term_vectors` (this will cause the index to be bigger):

```json
PUT /example
{
  "mappings": {
    "properties": {
      "comment" : {
        "type": "text",
        "term_vector" : "with_positions_offsets"
      }
    }
  }
}
```

##  Specify a fragmenter for the plain highlighter

When using the `plain` highlighter, you can choose between the `simple` and `span` fragmenters:

```json
GET my-index-000001/_search
{
  "query": {
    "match_phrase": { "message": "number 1" }
  },
  "highlight": {
    "fields": {
      "message": {
        "type": "plain",
        "fragment_size": 15,
        "number_of_fragments": 3,
        "fragmenter": "simple"
      }
    }
  }
}
```

Response:

```json
{
  ...
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.6011951,
    "hits": [
      {
        "_index": "my-index-000001",
        "_id": "1",
        "_score": 1.6011951,
        "_source": {
          "message": "some message with the number 1",
          "context": "bar"
        },
        "highlight": {
          "message": [
            " with the <em>number</em>",
            " <em>1</em>"
          ]
        }
      }
    ]
  }
}
```

```json
GET my-index-000001/_search
{
  "query": {
    "match_phrase": { "message": "number 1" }
  },
  "highlight": {
    "fields": {
      "message": {
        "type": "plain",
        "fragment_size": 15,
        "number_of_fragments": 3,
        "fragmenter": "span"
      }
    }
  }
}
```

Response:

```json
{
  ...
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.6011951,
    "hits": [
      {
        "_index": "my-index-000001",
        "_id": "1",
        "_score": 1.6011951,
        "_source": {
          "message": "some message with the number 1",
          "context": "bar"
        },
        "highlight": {
          "message": [
            " with the <em>number</em> <em>1</em>"
          ]
        }
      }
    ]
  }
}
```

If the `number_of_fragments` option is set to `0`, `NullFragmenter` is used which does not fragment the text at all. This is useful for highlighting the entire contents of a document or field.

##  How highlighters work internally

Given a query and a text (the content of a document field), the goal of a highlighter is to find the best text fragments for the query, and highlight the query terms in the found fragments. For this, a highlighter needs to address several questions:

- How break a text into fragments?
- How to find the best fragments among all fragments?
- How to highlight the query terms in a fragment?

##  How to break a text into fragments?

Relevant settings: `fragment_size`, `fragmenter`, `type` of highlighter, `boundary_chars`, `boundary_max_scan`, `boundary_scanner`, `boundary_scanner_locale`.

Plain highlighter begins with analyzing the text using the given analyzer, and creating a token stream from it. Plain highlighter uses a very simple algorithm to break the token stream into fragments. It loops through terms in the token stream, and every time the current term’s end_offset exceeds `fragment_size` multiplied by the number of created fragments, a new fragment is created. A little more computation is done with using `span` fragmenter to avoid breaking up text between highlighted terms. But overall, since the breaking is done only by `fragment_size`, some fragments can be quite odd, e.g. beginning with a punctuation mark.

Unified or FVH highlighters do a better job of breaking up a text into fragments by utilizing Java’s `BreakIterator`. This ensures that a fragment is a valid sentence as long as `fragment_size` allows for this.

##  How to find the best fragments?

Relevant settings: `number_of_fragments`.

To find the best, most relevant, fragments, a highlighter needs to score each fragment in respect to the given query. The goal is to score only those terms that participated in generating the *hit* on the document. For some complex queries, this is still work in progress.

The plain highlighter creates an in-memory index from the current token stream, and re-runs the original query criteria through Lucene’s query execution planner to get access to low-level match information for the current text. For more complex queries the original query could be converted to a span query, as span queries can handle phrases more accurately. Then this obtained low-level match information is used to score each individual fragment. The scoring method of the plain highlighter is quite simple. Each fragment is scored by the number of unique query terms found in this fragment. The score of individual term is equal to its boost, which is by default is 1. Thus, by default, a fragment that contains one unique query term, will get a score of 1; and a fragment that contains two unique query terms, will get a score of 2 and so on. The fragments are then sorted by their scores, so the highest scored fragments will be output first.

FVH doesn’t need to analyze the text and build an in-memory index, as it uses pre-indexed document term vectors, and finds among them terms that correspond to the query. FVH scores each fragment by the number of query terms found in this fragment. Similarly to plain highlighter, score of individual term is equal to its boost value. In contrast to plain highlighter, all query terms are counted, not only unique terms.

Unified highlighter can use pre-indexed term vectors or pre-indexed terms offsets, if they are available. Otherwise, similar to Plain Highlighter, it has to create an in-memory index from the text. Unified highlighter uses the BM25 scoring model to score fragments.

##  How to highlight the query terms in a fragment?

Relevant settings: `pre-tags`, `post-tags`.

The goal is to highlight only those terms that participated in generating the *hit* on the document. For some complex boolean queries, this is still work in progress, as highlighters don’t reflect the boolean logic of a query and only extract leaf (terms, phrases, prefix etc) queries.

Plain highlighter given the token stream and the original text, recomposes the original text to highlight only terms from the token stream that are contained in the low-level match information structure from the previous step.

FVH and unified highlighter use intermediate data structures to represent fragments in some raw form, and then populate them with actual text.

A highlighter uses `pre-tags`, `post-tags` to encode highlighted terms.

##  An example of the work of the unified highlighter

Let’s look in more details how unified highlighter works.

First, we create a index with a text field `content`, that will be indexed using `english` analyzer, and will be indexed without offsets or term vectors.

``` json
PUT test_index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

We put the following document into the index:

```json
PUT test_index/_doc/doc1
{
  "content" : "For you I'm only a fox like a hundred thousand other foxes. But if you tame me, we'll need each other. You'll be the only boy in the world for me. I'll be the only fox in the world for you."
}
```

And we ran the following query with a highlight request:

```json
GET test_index/_search
{
  "query": {
    "match_phrase" : {"content" : "only fox"}
  },
  "highlight": {
    "type" : "unified",
    "number_of_fragments" : 3,
    "fields": {
      "content": {}
    }
  }
}
```

After `doc1` is found as a hit for this query, this hit will be passed to the unified highlighter for highlighting the field `content` of the document. Since the field `content` was not indexed either with offsets or term vectors, its raw field value will be analyzed, and in-memory index will be built from the terms that match the query:

```
{"token":"onli","start_offset":12,"end_offset":16,"position":3},
{"token":"fox","start_offset":19,"end_offset":22,"position":5},
{"token":"fox","start_offset":53,"end_offset":58,"position":11},
{"token":"onli","start_offset":117,"end_offset":121,"position":24},
{"token":"onli","start_offset":159,"end_offset":163,"position":34},
{"token":"fox","start_offset":164,"end_offset":167,"position":35}
```

Our complex phrase query will be converted to the span query: `spanNear([text:onli, text:fox], 0, true)`, meaning that we are looking for terms "onli: and "fox" within 0 distance from each other, and in the given order. The span query will be run against the created before in-memory index, to find the following match:

```
{"term":"onli", "start_offset":159, "end_offset":163},
{"term":"fox", "start_offset":164, "end_offset":167}
```

In our example, we have got a single match, but there could be several matches. Given the matches, the unified highlighter breaks the text of the field into so called "passages". Each passage must contain at least one match. The unified highlighter with the use of Java’s `BreakIterator` ensures that each passage represents a full sentence as long as it doesn’t exceed `fragment_size`. For our example, we have got a single passage with the following properties (showing only a subset of the properties here):

```
Passage:
    startOffset: 147
    endOffset: 189
    score: 3.7158387
    matchStarts: [159, 164]
    matchEnds: [163, 167]
    numMatches: 2
```

Notice how a passage has a score, calculated using the BM25 scoring formula adapted for passages. Scores allow us to choose the best scoring passages if there are more passages available than the requested by the user `number_of_fragments`. Scores also let us to sort passages by `order: "score"` if requested by the user.

As the final step, the unified highlighter will extract from the field’s text a string corresponding to each passage:

```
"I'll be the only fox in the world for you."
```

and will format with the tags <em> and </em> all matches in this string using the passages’s `matchStarts` and `matchEnds` information:

```
I'll be the <em>only</em> <em>fox</em> in the world for you.
```

This kind of formatted strings are the final result of the highlighter returned to the user.

#  Long-running searches

Elasticsearch generally allows you to quickly search across big amounts of data. There are situations where a search executes on many shards, possibly against [frozen indices](https://www.elastic.co/guide/en/elasticsearch/reference/master/frozen-indices.html) and spanning multiple [remote clusters](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-remote-clusters.html), for which results are not expected to be returned in milliseconds. When you need to execute long-running searches, synchronously waiting for its results to be returned is not ideal. Instead, Async search lets you submit a search request that gets executed *asynchronously*, monitor the progress of the request, and retrieve results at a later stage. You can also retrieve partial results as they become available but before the search has completed.

Elasticsearch通常允许您快速搜索大量数据。在某些情况下，可能对[冻结的索引](https://www.elastic.co/guide/en/elasticsearch/reference/master/frozen-indices.html)并跨越多个 [远程群集的](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-remote-clusters.html)许多碎片执行搜索 ，因此预期结果不会在毫秒内返回。当您需要执行长时间运行的搜索时，同步等待返回结果是不理想的。相反，异步搜索使您可以提交*异步*执行的搜索请求，监视请求的进度并在以后的阶段检索结果。您也可以在部分结果可用时但在搜索完成之前检索它们。

You can submit an async search request using the [submit async search](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html#submit-async-search) API. The [get async search](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html#get-async-search) API allows you to monitor the progress of an async search request and retrieve its results. An ongoing async search can be deleted through the [delete async search](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html#delete-async-search) API.

您可以使用[提交异步搜索](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html#submit-async-search)API提交异步搜索请求。使用[get async search](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html#get-async-search) API，您可以监视异步搜索请求的进度并检索其结果。正在进行的异步搜索可以通过[Delete async search](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html#delete-async-search) API[删除](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html#delete-async-search)。

#  Near real-time search

The overview of [documents and indices](https://www.elastic.co/guide/en/elasticsearch/reference/master/documents-indices.html) indicates that when a document is stored in Elasticsearch, it is indexed and fully searchable in *near real-time*--within 1 second. What defines near real-time search?

[文档和索引](https://www.elastic.co/guide/en/elasticsearch/reference/master/documents-indices.html)的概述表明，当文档存储在Elasticsearch中时，将在1秒内以*几乎实时的方式*对其进行索引和完全搜索。什么定义了近实时搜索？

Lucene, the Java libraries on which Elasticsearch is based, introduced the concept of per-segment search. A *segment* is similar to an inverted index, but the word *index* in Lucene means "a collection of segments plus a commit point". After a commit, a new segment is added to the commit point and the buffer is cleared.

Lucene是Elasticsearch所基于的Java库，它引入了按段搜索的概念。*段* 类似于一个倒排索引，但*索引*这个词在Lucene中指“段的集合加提交点”。提交后，将新段添加到提交点并清除缓冲区。

Sitting between Elasticsearch and the disk is the filesystem cache. Documents in the in-memory indexing buffer ([Figure 1](https://www.elastic.co/guide/en/elasticsearch/reference/master/near-real-time.html#img-pre-refresh)) are written to a new segment ([Figure 2](https://www.elastic.co/guide/en/elasticsearch/reference/master/near-real-time.html#img-post-refresh)). The new segment is written to the filesystem cache first (which is cheap) and only later is it flushed to disk (which is expensive). However, after a file is in the cache, it can be opened and read just like any other file.

文件系统缓存位于Elasticsearch和磁盘之间。内存中索引缓冲区（[图1](https://www.elastic.co/guide/en/elasticsearch/reference/master/near-real-time.html#img-pre-refresh)）中的文档被写入新段（[图2](https://www.elastic.co/guide/en/elasticsearch/reference/master/near-real-time.html#img-post-refresh)）。新段首先被写入文件系统缓存（便宜），然后才刷新到磁盘（昂贵）。但是，将文件放入高速缓存后，可以像打开其他文件一样打开和读取该文件。

![A Lucene index with new documents in the in-memory buffer](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/lucene-in-memory-buffer.png)

**Figure 1. A Lucene index with new documents in the in-memory buffer**

Lucene allows new segments to be written and opened, making the documents they contain visible to search without performing a full commit. This is a much lighter process than a commit to disk, and can be done frequently without degrading performance.

Lucene允许编写和打开新的段，使它们包含的文档可见以进行搜索而无需执行完整的提交。这比提交磁盘要轻得多，并且可以经常执行而不会降低性能。

![The buffer contents are written to a segment, which is searchable, but is not yet committed](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/lucene-written-not-committed.png)

**Figure 2. The buffer contents are written to a segment, which is searchable, but is not yet committed**

In Elasticsearch, this process of writing and opening a new segment is called a *refresh*. A refresh makes all operations performed on an index since the last refresh available for search. You can control refreshes through the following means:

在Elasticsearch中，这种写入和打开新段的过程称为*refresh*。刷新使自上次刷新以来对索引执行的所有操作都可用于搜索。您可以通过以下方式控制刷新：

- Waiting for the refresh interval
- 等待刷新间隔
- Setting the [?refresh](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-refresh.html) option
- 设置[？refresh](https://www.elastic.co/guide/en/elasticsearch/reference/master/docs-refresh.html)选项
- Using the [Refresh API](https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-refresh.html) to explicitly complete a refresh (`POST _refresh`)
- 使用[Refresh API](https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-refresh.html)显式完成刷新（`POST _refresh`）

By default, Elasticsearch periodically refreshes indices every second, but only on indices that have received one search request or more in the last 30 seconds. This is why we say that Elasticsearch has *near* real-time search: document changes are not visible to search immediately, but will become visible within this timeframe.

默认情况下，Elasticsearch定期每秒刷新一次索引，但仅刷新在最近30秒内已收到一个或多个搜索请求的索引。这就是为什么我们说Elasticsearch具有*几乎*实时的搜索功能：文档更改不可见，无法立即搜索，但在此时间范围内将变为可见。

#  Paginate search results

By default, searches return the top 10 matching hits. To page through a larger set of results, you can use the [search API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)'s `from` and `size` parameters. The `from` parameter defines the number of hits to skip, defaulting to `0`. The `size` parameter is the maximum number of hits to return. Together, these two parameters define a page of results.

默认情况下，搜索会返回前10个匹配的匹配项。要检索较大的结果集，你可以使用[搜索API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)的`from`和`size` 参数。该`from`参数定义要跳过的匹配数，默认为`0`。该`size`参数是命中返回的最大数量。这两个参数共同定义了结果页面。

```json
GET /_search
{
  "from": 5,
  "size": 20,
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

Avoid using `from` and `size` to page too deeply or request too many results at once. Search requests usually span multiple shards. Each shard must load its requested hits and the hits for any previous pages into memory. For deep pages or large sets of results, these operations can significantly increase memory and CPU usage, resulting in degraded performance or node failures.

避免使用`from`和`size`分页太深或一次请求太多结果。搜索请求通常跨越多个分片。每个分片必须将其请求的命中以及任何先前页面的命中加载到内存中。对于较深的页面或大量结果，这些操作会显着增加内存和CPU使用率，从而导致性能下降或节点故障。

By default, you cannot use `from` and `size` to page through more than 10,000 hits. This limit is a safeguard set by the [`index.max_result_window`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-result-window) index setting. If you need to page through more than 10,000 hits, use the [`search_after`](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#search-after) parameter instead.

默认情况下，您不能使用`from`和`size`分页浏览超过10,000个匹配项。此限制是由[`index.max_result_window`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-result-window)索引设置设置的保护措施 。如果您需要分页浏览超过10,000个匹配项，请改用[`search_after`](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#search-after) 参数。

> **WARNING:** Elasticsearch uses Lucene’s internal doc IDs as tie-breakers. These internal doc IDs can be completely different across replicas of the same data. When paging search hits, you might occasionally see that documents with the same sort values are not ordered consistently.
>
> 警告：Elasticsearch使用Lucene的内部文档ID作为决断因素。这些内部文档ID在相同数据的副本之间可能完全不同。当分页搜索命中时，您可能偶尔会看到具有相同排序值的文档的排列顺序不一致。

##  Search after

You can use the `search_after` parameter to retrieve the next page of hits using a set of [sort values](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html) from the previous page.

您可以使用`search_after`参数使用前一页中的一组[排序值](https://www.elastic.co/guide/en/elasticsearch/reference/master/sort-search-results.html)来检索匹配的下一页。

Using `search_after` requires multiple search requests with the same `query` and `sort` values. If a [refresh](https://www.elastic.co/guide/en/elasticsearch/reference/master/near-real-time.html) occurs between these requests, the order of your results may change, causing inconsistent results across pages. To prevent this, you can create a [point in time (PIT)](https://www.elastic.co/guide/en/elasticsearch/reference/master/point-in-time-api.html) to preserve the current index state over your searches.

使用`search_after`需要具有`query`和 `sort`值的多个搜索请求。如果在这些请求之间进行[刷新](https://www.elastic.co/guide/en/elasticsearch/reference/master/near-real-time.html)，则结果的顺序可能会更改，从而导致页面之间的结果不一致。为防止这种情况，您可以创建一个[时间点（PIT）](https://www.elastic.co/guide/en/elasticsearch/reference/master/point-in-time-api.html)以在搜索过程中保留当前索引状态。

```http
POST /my-index-000001/_pit?keep_alive=1m
```

The API returns a PIT ID.

API返回一个PIT ID。

```json
{
  "id": "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
}
```

To get the first page of results, submit a search request with a `sort` argument. If using a PIT, specify the PIT ID in the `pit.id` parameter and omit the target data stream or index from the request path.

要获得结果的第一页，请提交带有`sort` 参数的搜索请求。如果使用PIT，请在`pit.id`参数中指定PIT ID，并从请求路径中忽略目标数据流或索引。

> **IMPORTANT:** All PIT search requests add an implicit sort tiebreaker field called `_shard_doc`, which can also be provided explicitly. If you cannot use a PIT, we recommend that you include a tiebreaker field in your `sort`. This tiebreaker field should contain a unique value for each document. If you don’t include a tiebreaker field, your paged results could miss or duplicate hits. 
>
> 重点：所有PIT搜索请求都会添加一个称为`_shard_doc`的隐式排序决断字段，也可以显式提供。如果您不能使用PIT，建议您在中添加一个决断字段`sort`。决断字段应为每个文档包含唯一值。如果您不包含决断字段，则分页结果可能会丢失或重复命中。

> **NODE:** Search after requests have optimizations that make them faster when the sort order is `_shard_doc` and total hits are not tracked. If you want to iterate over all documents regardless of the order, this is the most efficient option.
>
> 注意：请求后搜索具有优化功能，可以优化排序顺序`_shard_doc`并且不跟踪总命中率。如果要遍历所有文档而不考虑顺序，这是最有效的选择。

> **IMPORTANT:** If the `sort` field is a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html) in some target data streams or indices but a [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) field in other targets, use the `numeric_type` parameter to convert the values to a single resolution and the `format` parameter to specify a [date format](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html) for the `sort` field. Otherwise, Elasticsearch won’t interpret the search after parameter correctly in each request.
>
> 重点：如果该`排序`字段在某些目标数据流或索引中是[`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html) ，但在其他目标中是一个[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html)字段，则使用`numeric_type`参数将值转换为单个分辨率，并使用`format`参数指定该字段的 [日期格式](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)`sort`。否则，Elasticsearch将不会在每个请求中正确解释参数之后的搜索。

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
	    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
	    "keep_alive": "1m"
  },
  "sort": [
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos", "numeric_type" : "date_nanos" }}
  ]
}
```

1. PIT ID for the search.

2. Sorts hits for the search with an implicit tiebreak on _shard_doc ascending.

The search response includes an array of `sort` values for each hit. If you used a PIT, a tiebreaker is included as the last `sort` values for each hit. This tiebreaker called `_shard_doc` is added automatically on every search requests that use a PIT. The `_shard_doc` value is the combination of the shard index within the PIT and the Lucene’s internal doc ID, it is unique per document and constant within a PIT. You can also add the tiebreaker explicitly in the search request to customize the order:

搜索响应包括`sort`每个匹配项的值数组。如果您使用的是PIT，则决断字段将作为`sort`每个命中的最后一个值。这个决断字段被称为`_shard_doc`在使用PIT的每个搜索请求中，都会自动添加。该`_shard_doc`值是PIT中的分片索引和Lucene的内部doc ID的组合，它对于每个文档都是唯一的，并且在PIT中是常量。您还可以在搜索请求中显式添加决断字段以自定义顺序：

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
	    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
	    "keep_alive": "1m"
  },
  "sort": [ 
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos"}},
    {"_shard_doc": "desc"}
  ]
}
```

1. PIT ID for the search.

2. Sorts hits for the search with an explicit tiebreak on _shard_doc descending.

```json
{
  "pit_id" : "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
  "took" : 17,
  "timed_out" : false,
  "_shards" : ...,
  "hits" : {
    "total" : ...,
    "max_score" : null,
    "hits" : [
      ...
      {
        "_index" : "my-index-000001",
        "_id" : "FaslK3QBySSL_rrj9zM5",
        "_score" : null,
        "_source" : ...,
        "sort" : [                                
          "2021-05-20T05:30:04.832Z",
          4294967298                              
        ]
      }
    ]
  }
}
```

1. Updated id for the point in time.

2. Sort values for the last returned hit.

3. The tiebreaker value, unique per document within the pit_id.

To get the next page of results, rerun the previous search using the last hit’s sort values (including the tiebreaker) as the `search_after` argument. If using a PIT, use the latest PIT ID in the `pit.id` parameter. The search’s `query` and `sort` arguments must remain unchanged. If provided, the `from` argument must be `0` (default) or `-1`.

要获取下一页结果，请使用最后一个匹配项的排序值（包括决胜字段）作为`search_after`参数，重新运行上一个搜索。如果使用PIT，请在`pit.id`参数中使用最新的PIT ID 。搜索的`query`和`sort`参数必须保持不变。如果提供，则`from`参数必须为`0`（默认）或`-1`。

```json
GET /_search
{
  "size": 10000,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  },
  "pit": {
	    "id":  "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA==", 
	    "keep_alive": "1m"
  },
  "sort": [
    {"@timestamp": {"order": "asc", "format": "strict_date_optional_time_nanos"}}
  ],
  "search_after": [                                
    "2021-05-20T05:30:04.832Z",
    4294967298
  ],
  "track_total_hits": false                        
}
```

1. PIT ID returned by the previous search.

2. Sort values from the previous search’s last hit.

3. Disable the tracking of total hits to speed up pagination.

You can repeat this process to get additional pages of results. If using a PIT, you can extend the PIT’s retention period using the `keep_alive` parameter of each search request.

您可以重复此过程以获取其他结果页面。如果使用PIT，则可以使用`keep_alive`每个搜索请求的参数来延长PIT的保留期限 。

When you’re finished, you should delete your PIT.

完成后，您应该删除您的PIT。

```json
DELETE /_pit
{
    "id" : "46ToAwMDaWR5BXV1aWQyKwZub2RlXzMAAAAAAAAAACoBYwADaWR4BXV1aWQxAgZub2RlXzEAAAAAAAAAAAEBYQADaWR5BXV1aWQyKgZub2RlXzIAAAAAAAAAAAwBYgACBXV1aWQyAAAFdXVpZDEAAQltYXRjaF9hbGw_gAAAAA=="
}
```

##  Scroll search results

> **IMPORTANT:** We no longer recommend using the scroll API for deep pagination. If you need to preserve the index state while paging through more than 10,000 hits, use the [`search_after`](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#search-after) parameter with a point in time (PIT).
>
> 重要：我们不再建议使用scroll API进行深度分页。如果在分页浏览超过10,000个匹配项时需要保留索引状态，请使用[`search_after`](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#search-after)带有时间点（PIT）的参数。

While a `search` request returns a single “page” of results, the `scroll` API can be used to retrieve large numbers of results (or even all results) from a single search request, in much the same way as you would use a cursor on a traditional database.

当`search`请求返回单个“页面”结果时，该`scroll` API可用于从单个搜索请求中检索大量结果（甚至所有结果），其方式与在传统数据库上使用游标的方式几乎相同。

Scrolling is not intended for real time user requests, but rather for processing large amounts of data, e.g. in order to reindex the contents of one data stream or index into a new data stream or index with a different configuration.

滚动并非旨在用于实时用户请求，而是用于处理大量数据，例如，以便将一个数据流或索引的内容重新索引为具有不同配置的新数据流或索引。

**Client support for scrolling and reindexing**

Some of the officially supported clients provide helpers to assist with scrolled searches and reindexing:

一些受官方支持的客户提供了一些助手来帮助您进行滚动搜索和重新编制索引：

- **Perl**

  See [Search::Elasticsearch::Client::5_0::Bulk](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Bulk) and [Search::Elasticsearch::Client::5_0::Scroll](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Scroll)

- **Python**

  See [elasticsearch.helpers.*](https://elasticsearch-py.readthedocs.org/en/master/helpers.html)

- **JavaScript**

  See [client.helpers.*](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/client-helpers.html)

> **NOTE:** The results that are returned from a scroll request reflect the state of the data stream or index at the time that the initial `search` request was made, like a snapshot in time. Subsequent changes to documents (index, update or delete) will only affect later search requests.
>
> 从滚动请求返回的结果反映了发出初始`search`请求时数据流或索引的状态，如时间快照。随后对文档的更改（索引，更新或删除）只会影响以后的搜索请求。

In order to use scrolling, the initial search request should specify the `scroll` parameter in the query string, which tells Elasticsearch how long it should keep the “search context” alive (see [Keeping the search context alive](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-context)), eg `?scroll=1m`.

为了使用滚动，初始搜索请求应`scroll`在查询字符串中指定参数，该 参数告诉Elasticsearch它应将“搜索上下文”[保持](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-context)活动状态的时间（请参阅[使搜索上下文保持活动状态](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-context)），例如`?scroll=1m`。

```json
POST /my-index-000001/_search?scroll=1m
{
  "size": 100,
  "query": {
    "match": {
      "message": "foo"
    }
  }
}
```

The result from the above request includes a `_scroll_id`, which should be passed to the `scroll` API in order to retrieve the next batch of results.

上述请求的结果包括`_scroll_id`，应将其传递给`scroll`API，以检索下一批结果。

```json
POST /_search/scroll                                                               
{
  "scroll" : "1m",                                                                 
  "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==" 
}
```

1. GET or POST can be used and the URL should not include the index name — this is specified in the original search request instead.

2. The scroll parameter tells Elasticsearch to keep the search context open for another 1m.

3. The scroll_id parameter

The `size` parameter allows you to configure the maximum number of hits to be returned with each batch of results. Each call to the `scroll` API returns the next batch of results until there are no more results left to return, ie the `hits` array is empty.

该`size`参数允许您配置每批结果返回的最大匹配数。每次对`scroll`API的调用都会返回下一批结果，直到没有其他要返回的结果为止，即 `hits`数组为空。

> **IMPORTANT:** The initial search request and each subsequent scroll request each return a `_scroll_id`. While the `_scroll_id` may change between requests, it doesn’t always change — in any case, only the most recently received `_scroll_id` should be used.
>
> 重要：初始搜索请求和随后的每个滚动请求均返回`_scroll_id`。尽管`_scroll_id`请求之间的可能会有所变化，但并非总是如此-在任何情况下，仅`_scroll_id`应使用最新接收到的数据。

> **NOTE:** If the request specifies aggregations, only the initial search response will contain the aggregations results.
>
> 提示：如果请求指定聚合，则仅初始搜索响应将包含聚合结果。

> **NOTE:** Scroll requests have optimizations that make them faster when the sort order is `_doc`. If you want to iterate over all documents regardless of the order, this is the most efficient option:
>
> 提示：滚动请求具有优化功能，可以使排序顺序为`_doc`时更快响应。如果要遍历所有文档而不考虑顺序，这是最有效的选择：

```json
GET /_search?scroll=1m
{
  "sort": [
    "_doc"
  ]
}
```

##  Keeping the search context alive

A scroll returns all the documents which matched the search at the time of the initial search request. It ignores any subsequent changes to these documents. The `scroll_id` identifies a *search context* which keeps track of everything that Elasticsearch needs to return the correct documents. The search context is created by the initial request and kept alive by subsequent requests.

滚动返回在初始搜索请求时与搜索匹配的所有文档。它忽略了对这些文档的任何后续更改。该`scroll_id`标识一个*搜索上下文*它记录身边的一切Elasticsearch需要返回正确的文件。搜索上下文由初始请求创建，并由后续请求保持活动状态。

The `scroll` parameter (passed to the `search` request and to every `scroll` request) tells Elasticsearch how long it should keep the search context alive. Its value (e.g. `1m`, see [Time units](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units)) does not need to be long enough to process all data — it just needs to be long enough to process the previous batch of results. Each `scroll` request (with the `scroll` parameter) sets a new expiry time. If a `scroll` request doesn’t pass in the `scroll` parameter, then the search context will be freed as part of *that* `scroll` request.

该`scroll`参数（传递到`search`请求和每个`scroll` 请求）告诉Elasticsearch应该保持多长时间的搜索上下文活着。它的值（例如`1m`，请参阅“[时间单位”](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#time-units)）不需要足够长的时间来处理所有数据，而只需要足够长的时间即可处理前一批结果。每个`scroll`请求（带有`scroll`参数）都设置一个新的到期时间。如果`scroll`没有在`scroll` 参数中传递请求，则搜索上下文将作为*该* `scroll` 请求的一部分被释放。

Normally, the background merge process optimizes the index by merging together smaller segments to create new, bigger segments. Once the smaller segments are no longer needed they are deleted. This process continues during scrolling, but an open search context prevents the old segments from being deleted since they are still in use.

通常，后台合并过程通过将较小的段合并在一起以创建新的较大的段来优化索引。一旦不再需要较小的段，则将其删除。在滚动过程中，此过程继续进行，但是开放的搜索上下文可防止旧段被删除，因为它们仍在使用中。

> **TIP:** Keeping older segments alive means that more disk space and file handles are needed. Ensure that you have configured your nodes to have ample free file handles. See [File Descriptors](https://www.elastic.co/guide/en/elasticsearch/reference/master/file-descriptors.html).
>
> 提示：使较旧的段保持活动状态意味着需要更多的磁盘空间和文件句柄。确保已将节点配置为具有足够的空闲文件句柄。请参阅[文件描述符](https://www.elastic.co/guide/en/elasticsearch/reference/master/file-descriptors.html)。

Additionally, if a segment contains deleted or updated documents then the search context must keep track of whether each document in the segment was live at the time of the initial search request. Ensure that your nodes have sufficient heap space if you have many open scrolls on an index that is subject to ongoing deletes or updates.

此外，如果某个段包含已删除或更新的文档，则搜索上下文必须跟踪该段中的每个文档在初始搜索请求时是否处于活动状态。如果索引上有许多打开的滚动，这些索引会不断进行删除或更新，请确保节点具有足够的堆空间。

> **NOTE:** To prevent against issues caused by having too many scrolls open, the user is not allowed to open scrolls past a certain limit. By default, the maximum number of open scrolls is 500. This limit can be updated with the `search.max_open_scroll_context` cluster setting.
>
> 注意：为了防止打开太多滚动条引起的问题，不允许用户打开超过特定限制的滚动条。默认情况下，最大打开滚动数为500。可以使用`search.max_open_scroll_context`群集设置更新此限制 。

You can check how many search contexts are open with the [nodes stats API](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-nodes-stats.html):

您可以使用[node stats API](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-nodes-stats.html)检查打开了多少个搜索上下文 ：

```http
GET /_nodes/stats/indices/search
```

##  Clear scroll

Search context are automatically removed when the `scroll` timeout has been exceeded. However keeping scrolls open has a cost, as discussed in the [previous section](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-context) so scrolls should be explicitly cleared as soon as the scroll is not being used anymore using the `clear-scroll` API:

`scroll`超过超时时间后，搜索上下文将自动删除。但是，如上[一节](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-context)所述，保持滚动打开是有代价的， 因此，一旦不再使用`clear-scroll`API使用滚动，则应明确清除滚动 ：

```json
DELETE /_search/scroll
{
  "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ=="
}
```

Multiple scroll IDs can be passed as array:

多个滚动ID可以作为数组传递：

```json
DELETE /_search/scroll
{
  "scroll_id" : [
    "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==",
    "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAABFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAAAxZrUllkUVlCa1NqNmRMaUhiQlZkMWFBAAAAAAAAAAIWa1JZZFFZQmtTajZkTGlIYkJWZDFhQQAAAAAAAAAFFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAABBZrUllkUVlCa1NqNmRMaUhiQlZkMWFB"
  ]
}
```

All search contexts can be cleared with the `_all` parameter:

可以使用以下`_all`参数清除所有搜索上下文：

```http
DELETE /_search/scroll/_all
```

The `scroll_id` can also be passed as a query string parameter or in the request body. Multiple scroll IDs can be passed as comma separated values:

`scroll_id`也可以作为查询字符串参数或请求体通过。多个滚动ID可以作为逗号分隔的值传递：

```http
DELETE /_search/scroll/DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==,DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAABFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAAAxZrUllkUVlCa1NqNmRMaUhiQlZkMWFBAAAAAAAAAAIWa1JZZFFZQmtTajZkTGlIYkJWZDFhQQAAAAAAAAAFFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAABBZrUllkUVlCa1NqNmRMaUhiQlZkMWFB
```

##  Sliced scroll

For scroll queries that return a lot of documents it is possible to split the scroll in multiple slices which can be consumed independently:

对于返回大量文档的滚动查询，可以将滚动分为多个切片，这些切片可以独立使用：

```json
GET /my-index-000001/_search?scroll=1m
{
  "slice": {
    "id": 0,                      
    "max": 2                      
  },
  "query": {
    "match": {
      "message": "foo"
    }
  }
}
GET /my-index-000001/_search?scroll=1m
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "query": {
    "match": {
      "message": "foo"
    }
  }
}
```

1. The id of the slice

2. The maximum number of slices

The result from the first request returned documents that belong to the first slice (id: 0) and the result from the second request returned documents that belong to the second slice. Since the maximum number of slices is set to 2 the union of the results of the two requests is equivalent to the results of a scroll query without slicing. By default the splitting is done on the shards first and then locally on each shard using the _id field with the following formula: `slice(doc) = floorMod(hashCode(doc._id), max)` For instance if the number of shards is equal to 2 and the user requested 4 slices then the slices 0 and 2 are assigned to the first shard and the slices 1 and 3 are assigned to the second shard.

第一个请求的结果返回了属于第一个片（id：0）的文档，第二个请求的结果返回了属于第二个片的文档。由于切片的最大数量设置为2，所以两个请求的结果的并集等效于不切片的滚动查询的结果。默认情况下，首先在碎片上进行拆分，然后使用具有以下公式的_id字段在每个碎片上进行本地拆分： `slice(doc) = floorMod(hashCode(doc._id), max)` 例如，如果碎片数等于2，并且用户请求了4个切片，则将切片0和2分配给该切片将第一个分片分配给第一个分片，并将分片1和3分配给第二个分片。

Each scroll is independent and can be processed in parallel like any scroll request.

每个滚动都是独立的，并且可以像任何滚动请求一样并行处理。

> **NOTE:** If the number of slices is bigger than the number of shards the slice filter is very slow on the first calls, it has a complexity of O(N) and a memory cost equals to N bits per slice where N is the total number of documents in the shard. After few calls the filter should be cached and subsequent calls should be faster but you should limit the number of sliced query you perform in parallel to avoid the memory explosion.
>
> 注意：如果切片的数量大于分片（ES节点分片）的数量，则切片过滤器在首次调用时非常慢，它的复杂度为O（N），内存成本等于每个切片N位，其中N是文档总数在碎片中。几次调用后，应将筛选器缓存起来，随后的调用应更快，但应限制并行执行的切片查询的数量，以避免内存爆掉。

To avoid this cost entirely it is possible to use the `doc_values` of another field to do the slicing but the user must ensure that the field has the following properties:

为了完全避免这种成本，可以使用`doc_values`另一个字段的进行切片，但是用户必须确保该字段具有以下属性：

- The field is numeric.

  该字段是数字。

- `doc_values` are enabled on that field

  `doc_values` 在该字段上启用

- Every document should contain a single value. If a document has multiple values for the specified field, the first value is used.

  每个文档应包含一个值。如果文档的指定字段具有多个值，则使用第一个值。

- The value for each document should be set once when the document is created and never updated. This ensures that each slice gets deterministic results.

  每个文档的值在创建文档时都应该设置一次，并且永远不要更新。这样可以确保每个切片得到确定的结果。

- The cardinality of the field should be high. This ensures that each slice gets approximately the same amount of documents.

  该字段的基数应该很高。这样可以确保每个切片获得的文档数量大致相同。

```json
GET /my-index-000001/_search?scroll=1m
{
  "slice": {
    "field": "@timestamp",
    "id": 0,
    "max": 10
  },
  "query": {
    "match": {
      "message": "foo"
    }
  }
}
```

For append only time-based indices, the `timestamp` field can be used safely.

对于仅追加基于时间的索引，`timestamp`可以安全地使用该字段。

> **NOTE:** By default the maximum number of slices allowed per scroll is limited to 1024. You can update the `index.max_slices_per_scroll` index setting to bypass this limit.
>
> 注意：默认情况下，每个滚动所允许的最大切片数限制为1024。您可以更新`index.max_slices_per_scroll`索引设置以绕过此限制。

#  Retrieve inner hits

The [parent-join](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html) and [nested](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html) features allow the return of documents that have matches in a different scope. In the parent/child case, parent documents are returned based on matches in child documents or child documents are returned based on matches in parent documents. In the nested case, documents are returned based on matches in nested inner objects.

该[父连接](https://www.elastic.co/guide/en/elasticsearch/reference/master/parent-join.html)和[嵌套](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html)功能允许有不同范围的匹配文件的回报。在父/子情况下，根据子文档中的匹配返回父文档，或者根据父文档中的匹配返回子文档。在嵌套的情况下，基于嵌套内部对象中的匹配返回文档。

In both cases, the actual matches in the different scopes that caused a document to be returned are hidden. In many cases, it’s very useful to know which inner nested objects (in the case of nested) or children/parent documents (in the case of parent/child) caused certain information to be returned. The inner hits feature can be used for this. This feature returns per search hit in the search response additional nested hits that caused a search hit to match in a different scope.

在这两种情况下，隐藏了导致返回文档的不同范围中的实际匹配项。在许多情况下，了解哪些内部嵌套对象（对于嵌套的情况）或子/父文档（对于父/子的情况）导致某些信息返回非常有用。内部点击功能可用于此目的。此功能在搜索响应中为每个搜索命中返回附加的嵌套命中，这些嵌套命中导致搜索命中匹配在不同范围内。

Inner hits can be used by defining an `inner_hits` definition on a `nested`, `has_child` or `has_parent` query and filter. The structure looks like this:

内命中可以通过定义一个`inner_hits`在上`nested`使用，`has_child`或者`has_parent`查询和过滤器。结构如下：

```json
"<query>" : {
    "inner_hits" : {
        <inner_hits_options>
    }
}
```

If `inner_hits` is defined on a query that supports it then each search hit will contain an `inner_hits` json object with the following structure:

如果`inner_hits`在支持查询的查询中定义，则每个搜索命中将包含`inner_hits`具有以下结构的json对象：

```json
"hits": [
     {
        "_index": ...,
        "_type": ...,
        "_id": ...,
        "inner_hits": {
           "<inner_hits_name>": {
              "hits": {
                 "total": ...,
                 "hits": [
                    {
                       "_id": ...,
                       ...
                    },
                    ...
                 ]
              }
           }
        },
        ...
     },
     ...
]
```

##  Options

Inner hits support the following options:

| `from` | The offset from where the first hit to fetch for each `inner_hits` in the returned regular search hits.<br />`inner_hits`所返回的常规搜索匹配中 每个匹配的第一个匹配获取位置的偏移量。 |
| ------ | ------------------------------------------------------------ |
| `size` | The maximum number of hits to return per `inner_hits`. By default the top three matching hits are returned.<br />每个返回的最大命中数`inner_hits`。默认情况下，返回前三个匹配项。 |
| `sort` | How the inner hits should be sorted per `inner_hits`. By default the hits are sorted by the score.<br />每个`inner_hits`内部命中应如何排序。默认情况下，命中按得分排序。 |
| `name` | The name to be used for the particular inner hit definition in the response. Useful when multiple inner hits have been defined in a single search request. The default depends in which query the inner hit is defined. For `has_child` query and filter this is the child type, `has_parent` query and filter this is the parent type and the nested query and filter this is the nested path.<br />响应中用于特定内部匹配定义的名称。如果在单个搜索请求中定义了多个内部匹配，则很有用。默认值取决于定义内部匹配的查询。对于`has_child`查询和过滤器，这是子类型；对于查询和过滤`has_parent`器，这是父类型；对于嵌套查询和过滤器，这是嵌套路径。 |

Inner hits also supports the following per document features:

内部命中还支持以下每个文档功能

- [Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html)

  [突出显示](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html)

- [Explain](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-explain)

  [解释](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-explain)

- [Search fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#search-fields-param)

  [搜寻栏位](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#search-fields-param)

- [Source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-request-body.html#request-body-search-source-filtering)

  [源过滤](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-request-body.html#request-body-search-source-filtering)

- [Script fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#script-fields)

  [脚本字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#script-fields)

- [Doc value fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#docvalue-fields)

  [文件值栏位](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#docvalue-fields)

- [Include versions](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-version)

  [包含版本](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-version)

- [Include Sequence Numbers and Primary Terms](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-seq-no-primary-term)

  [包括序列号和主要术语](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-seq-no-primary-term)

##  Nested inner hits

The nested `inner_hits` can be used to include nested inner objects as inner hits to a search hit.

嵌套`inner_hits`可用于将嵌套的内部对象包括为搜索命中的内部命中。

```json
PUT test
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested"
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "number": 1
    },
    {
      "author": "nik9000",
      "number": 2
    }
  ]
}

POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.number" : 2}
      },
      "inner_hits": {} 
    }
  }
}
```

1. The inner hit definition in the nested query. No other options need to be defined.

   嵌套查询中的内部匹配定义。无需定义其他选项。

An example of a response snippet that could be generated from the above search request:

可以从上述搜索请求中生成响应片段的示例：

```json
{
  ...,
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "test",
        "_id": "1",
        "_score": 1.0,
        "_source": ...,
        "inner_hits": {
          "comments": { 
            "hits": {
              "total" : {
                  "value": 1,
                  "relation": "eq"
              },
              "max_score": 1.0,
              "hits": [
                {
                  "_index": "test",
                  "_id": "1",
                  "_nested": {
                    "field": "comments",
                    "offset": 1
                  },
                  "_score": 1.0,
                  "_source": {
                    "author": "nik9000",
                    "number": 2
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

1. The name used in the inner hit definition in the search request. A custom key can be used via the `name` option.

   搜索请求中内部匹配定义中使用的名称。可以通过该`name`选项使用自定义密钥。

The `_nested` metadata is crucial in the above example, because it defines from what inner nested object this inner hit came from. The `field` defines the object array field the nested hit is from and the `offset` relative to its location in the `_source`. Due to sorting and scoring the actual location of the hit objects in the `inner_hits` is usually different than the location a nested inner object was defined.

该`_nested`元数据是在上面的例子中是至关重要的，因为它是从什么内部嵌套对象这一内命中是从哪里来的定义。在`field`定义了对象阵列字段嵌套命中是来自与`offset`相对于它在位置`_source`。由于排序和评分，命中对象在中的实际位置`inner_hits`通常与定义嵌套内部对象的位置不同。

By default the `_source` is returned also for the hit objects in `inner_hits`, but this can be changed. Either via `_source` filtering feature part of the source can be returned or be disabled. If stored fields are defined on the nested level these can also be returned via the `fields` feature.

默认情况下，`_source`还会为中的命中对象返回`inner_hits`，但是可以更改。通过 `_source`过滤功能，可以返回或禁用源的一部分。如果在嵌套级别定义了存储字段，则也可以通过`fields`功能部件返回这些字段。

An important default is that the `_source` returned in hits inside `inner_hits` is relative to the `_nested` metadata. So in the above example only the comment part is returned per nested hit and not the entire source of the top level document that contained the comment.

一个重要的默认值是，内部`_source`返回的匹配`inner_hits`是相对于`_nested`元数据的。因此，在上面的示例中，每个嵌套的匹配仅返回注释部分，而不返回包含注释的顶级文档的整个源。

##  Nested inner hits and `_source`

Nested document don’t have a `_source` field, because the entire source of document is stored with the root document under its `_source` field. To include the source of just the nested document, the source of the root document is parsed and just the relevant bit for the nested document is included as source in the inner hit. Doing this for each matching nested document has an impact on the time it takes to execute the entire search request, especially when `size` and the inner hits' `size` are set higher than the default. To avoid the relatively expensive source extraction for nested inner hits, one can disable including the source and solely rely on doc values fields. Like this:

嵌套文档没有`_source`字段，因为整个文档源都与根文档一起存储在其`_source`字段下。为了仅包含嵌套文档的源，将分析根文档的源，并且在内部匹配中仅包含嵌套文档的相关位作为源。对每个匹配的嵌套文档执行此操作都会影响执行整个搜索请求所花费的时间，尤其是当`size`内部匹配项和内部匹配项`size` 设置为高于默认值时。为避免嵌套内部匹配的源代码提取相对昂贵，可以禁用源代码，而只能依靠doc值字段。像这样：

```json
PUT test
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested"
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text"
    },
    {
      "author": "nik9000",
      "text": "words words words"
    }
  ]
}

POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": {"comments.text" : "words"}
      },
      "inner_hits": {
        "_source" : false,
        "docvalue_fields" : [
          "comments.text.keyword"
        ]
      }
    }
  }
}
```

##  Hierarchical levels of nested object fields and inner hits.

If a mapping has multiple levels of hierarchical nested object fields each level can be accessed via dot notated path. For example if there is a `comments` nested field that contains a `votes` nested field and votes should directly be returned with the root hits then the following path can be defined:

如果映射具有多层嵌套对象字段层次，则可以通过点标记路径访问每个层次。例如，如果存在一个`comments`包含`votes`嵌套字段的嵌套字段，并且投票应直接与根匹配一起返回，则可以定义以下路径：

```json
PUT test
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested",
        "properties": {
          "votes": {
            "type": "nested"
          }
        }
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "title": "Test title",
  "comments": [
    {
      "author": "kimchy",
      "text": "comment text",
      "votes": []
    },
    {
      "author": "nik9000",
      "text": "words words words",
      "votes": [
        {"value": 1 , "voter": "kimchy"},
        {"value": -1, "voter": "other"}
      ]
    }
  ]
}

POST test/_search
{
  "query": {
    "nested": {
      "path": "comments.votes",
        "query": {
          "match": {
            "comments.votes.voter": "kimchy"
          }
        },
        "inner_hits" : {}
    }
  }
}
```

Which would look like:

看起来像：

```json
{
  ...,
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 0.6931471,
    "hits": [
      {
        "_index": "test",
        "_id": "1",
        "_score": 0.6931471,
        "_source": ...,
        "inner_hits": {
          "comments.votes": { 
            "hits": {
              "total" : {
                  "value": 1,
                  "relation": "eq"
              },
              "max_score": 0.6931471,
              "hits": [
                {
                  "_index": "test",
                  "_id": "1",
                  "_nested": {
                    "field": "comments",
                    "offset": 1,
                    "_nested": {
                      "field": "votes",
                      "offset": 0
                    }
                  },
                  "_score": 0.6931471,
                  "_source": {
                    "value": 1,
                    "voter": "kimchy"
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

This indirect referencing is only supported for nested inner hits.

此间接引用仅支持嵌套内部匹配使用。

##  Parent/child inner hits

The parent/child `inner_hits` can be used to include parent or child:

父母/子女`inner_hits`可以用来包括父母或子女：

```json
PUT test
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "my_parent": "my_child"
        }
      }
    }
  }
}

PUT test/_doc/1?refresh
{
  "number": 1,
  "my_join_field": "my_parent"
}

PUT test/_doc/2?routing=1&refresh
{
  "number": 1,
  "my_join_field": {
    "name": "my_child",
    "parent": "1"
  }
}

POST test/_search
{
  "query": {
    "has_child": {
      "type": "my_child",
      "query": {
        "match": {
          "number": 1
        }
      },
      "inner_hits": {}    
    }
  }
}
```

1. The inner hit definition like in the nested example.

    内部命中定义类似于嵌套示例中的内容。

An example of a response snippet that could be generated from the above search request:

可以从上述搜索请求中生成响应片段的示例：

```json
{
  ...,
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "test",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "number": 1,
          "my_join_field": "my_parent"
        },
        "inner_hits": {
          "my_child": {
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.0,
              "hits": [
                {
                  "_index": "test",
                  "_id": "2",
                  "_score": 1.0,
                  "_routing": "1",
                  "_source": {
                    "number": 1,
                    "my_join_field": {
                      "name": "my_child",
                      "parent": "1"
                    }
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```

#  Retrieve selected fields from a search

By default, each hit in the search response includes the document [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-source-field.html), which is the entire JSON object that was provided when indexing the document. There are two recommended methods to retrieve selected fields from a search query:

默认情况下，搜索响应中的每个[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-source-field.html)匹配都包括document ，这是对文档建立索引时提供的整个JSON对象。有两种推荐的方法可用于从搜索查询中检索选定的字段：

- Use the [`fields` option](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#search-fields-param) to extract the values of fields present in the index mapping

  使用该[`fields`选项](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#search-fields-param)提取索引映射中存在的字段的值

- Use the [`_source` option](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#source-filtering) if you need to access the original data that was passed at index time

  如果您需要访问在索引时传递的原始数据， 请使用该[`_source`选项](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#source-filtering)

You can use both of these methods, though the `fields` option is preferred because it consults both the document data and index mappings. In some instances, you might want to use [other methods](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#field-retrieval-methods) of retrieving data.

您可以使用这两种方法，尽管`fields`首选该选项，因为它可以同时查阅文档数据和索引映射。在某些情况下，您可能想使用[其他](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#field-retrieval-methods)检索数据的[方法](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#field-retrieval-methods)。

##  The `fields` option

To retrieve specific fields in the search response, use the `fields` parameter. Because it consults the index mappings, the `fields` parameter provides several advantages over referencing the `_source` directly. Specifically, the `fields` parameter:

要在搜索响应中检索特定字段，请使用`fields`参数。因为它查询索引映射，所以`fields`与`_source`直接引用相比，该参数具有多个优点。具体来说，`fields` 参数：

- Returns each value in a standardized way that matches its mapping type

  以符合其映射类型的标准化方式返回每个值

- Accepts [multi-fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/multi-fields.html) and [field aliases](https://www.elastic.co/guide/en/elasticsearch/reference/master/alias.html)

  接受[多字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/multi-fields.html)和[字段别名](https://www.elastic.co/guide/en/elasticsearch/reference/master/alias.html)

- Formats dates and spatial data types

  格式化日期和空间数据类型

- Retrieves [runtime field values](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime-retrieving-fields.html)

  检索[运行时字段值](https://www.elastic.co/guide/en/elasticsearch/reference/master/runtime-retrieving-fields.html)

- Returns fields calculated by a script at index time

  返回由脚本在索引时间计算的字段

Other mapping options are also respected, including [`ignore_above`](https://www.elastic.co/guide/en/elasticsearch/reference/master/ignore-above.html), [`ignore_malformed`](https://www.elastic.co/guide/en/elasticsearch/reference/master/ignore-malformed.html), and [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/master/null-value.html).

其他映射选项也被推崇，包括 [`ignore_above`](https://www.elastic.co/guide/en/elasticsearch/reference/master/ignore-above.html)，[`ignore_malformed`](https://www.elastic.co/guide/en/elasticsearch/reference/master/ignore-malformed.html)，和 [`null_value`](https://www.elastic.co/guide/en/elasticsearch/reference/master/null-value.html)。

The `fields` option returns values in the way that matches how Elasticsearch indexes them. For standard fields, this means that the `fields` option looks in `_source` to find the values, then parses and formats them using the mappings.

`fields`选项以与Elasticsearch索引它们的方式匹配的方式返回值。对于标准字段，这意味着该`fields`选项在 `_source`中查找值，然后使用映射对其进行解析和格式化。

##  Search for specific fields

The following search request uses the `fields` parameter to retrieve values for the `user.id` field, all fields starting with `http.response.`, and the `@timestamp` field.

以下搜索请求使用该`fields`参数来检索`user.id`字段，以开头的所有字段`http.response.`以及 `@timestamp`字段的值。

Using object notation, you can pass a `format` parameter for certain fields to apply a custom format for the field’s values:

使用对象表示法，您可以`format`为某些字段传递参数，以将自定义格式应用于该字段的值：

- [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html) and [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) fields accept a [date format](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)

  [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html)和[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html)字段接受[日期格式](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)

- [Spatial fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-types.html#spatial_datatypes) accept either `geojson` for [GeoJSON](http://www.geojson.org/) (the default) or `wkt` for [Well Known Text](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)

  [空间字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-types.html#spatial_datatypes)接受要么`geojson`为[GeoJSON的](http://www.geojson.org/)（默认值）或`wkt`为 [众所周知的文本](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)

Other field types do not support the `format` parameter.

其他字段类型不支持该`format`参数。

```json
POST my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "fields": [
    "user.id",
    "http.response.*",         
    {
      "field": "@timestamp",
      "format": "epoch_millis" 
    }
  ],
  "_source": false
}
```

1. Both full field names and wildcard patterns are accepted.

   完整字段名和通配符模式都可以接受。

2. Use the format parameter to apply a custom format for the field’s values.

    使用`format`参数为字段的值应用自定义格式。

##  Response always returns an array

The `fields` response always returns an array of values for each field, even when there is a single value in the `_source`. This is because Elasticsearch has no dedicated array type, and any field could contain multiple values. The `fields` parameter also does not guarantee that array values are returned in a specific order. See the mapping documentation on [arrays](https://www.elastic.co/guide/en/elasticsearch/reference/master/array.html) for more background.

`fields`响应时总是为每个字段返回一个数组，即使在一个单一的值`_source`。这是因为Elasticsearch没有专用的数组类型，并且任何字段都可以包含多个值。 `fields`参数也不能保证以特定顺序返回数组值。有关更多背景信息，请参见[数组](https://www.elastic.co/guide/en/elasticsearch/reference/master/array.html)上的映射文档。

The response includes values as a flat list in the `fields` section for each hit. Because the `fields` parameter doesn’t fetch entire objects, only leaf fields are returned.

响应在`fields`每个匹配项的部分中均包含作为平面列表的值。由于fields`参数不能获取整个对象，因此仅返回叶字段。

```json
{
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_id" : "0",
        "_score" : 1.0,
        "fields" : {
          "user.id" : [
            "kimchy"
          ],
          "@timestamp" : [
            "4098435132000"
          ],
          "http.response.bytes": [
            1070000
          ],
          "http.response.status_code": [
            200
          ]
        }
      }
    ]
  }
}
```

##  Retrieve nested fields

The `fields` response for [`nested` fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html) is slightly different from that of regular object fields. While leaf values inside regular `object` fields are returned as a flat list, values inside `nested` fields are grouped to maintain the independence of each object inside the original nested array. For each entry inside a nested field array, values are again returned as a flat list unless there are other `nested` fields inside the parent nested object, in which case the same procedure is repeated again for the deeper nested fields.

[嵌套字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/nested.html)的`fields`响应与常规对象字段的响应略有不同。将常规字段内的叶值作为平面列表返回时，将嵌套字段的值按原始嵌套数组分组保持内的每个对象的独立性。对于嵌套字段数组中的每个条目，除非父嵌套对象内还有其他嵌套字段，否则值将再次作为一个平面列表返回，在这种情况下，将对较深的嵌套字段再次重复相同的过程。

Given the following mapping where `user` is a nested field, after indexing the following document and retrieving all fields under the `user` field:

给定以下映射where`user`是嵌套字段，在索引以下文档并检索该字段下的所有字段之后`user`：

```json
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "group" : { "type" : "keyword" },
      "user": {
        "type": "nested",
        "properties": {
          "first" : { "type" : "keyword" },
          "last" : { "type" : "keyword" }
        }
      }
    }
  }
}

PUT my-index-000001/_doc/1?refresh=true
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}

POST my-index-000001/_search
{
  "fields": ["*"],
  "_source": false
}
```

The response will group `first` and `last` name instead of returning them as a flat list.

响应将以`first`和`last`名进行分组，而不是将其作为平面列表返回。

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [{
      "_index": "my-index-000001",
      "_id": "1",
      "_score": 1.0,
      "fields": {
        "group" : ["fans"],
        "user": [{
            "first": ["John"],
            "last": ["Smith"],
          },
          {
            "first": ["Alice"],
            "last": ["White"],
          }
        ]
      }
    }]
  }
}
```

Nested fields will be grouped by their nested paths, no matter the pattern used to retrieve them. For example, if you query only for the `user.first` field from the previous example:

无论用于检索它们的模式如何，嵌套字段都将按其嵌套路径进行分组。例如，如果您在上一个示例中仅查询`user.first`的字段：

```json
POST my-index-000001/_search
{
  "fields": ["user.first"],
  "_source": false
}
```

The response returns only the user’s first name, but still maintains the structure of the nested `user` array:

响应仅返回用户的名字，但仍保留嵌套`user`数组的结构：

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [{
      "_index": "my-index-000001",
      "_id": "1",
      "_score": 1.0,
      "fields": {
        "user": [{
            "first": ["John"],
          },
          {
            "first": ["Alice"],
          }
        ]
      }
    }]
  }
}
```

However, when the `fields` pattern targets the nested `user` field directly, no values will be returned because the pattern doesn’t match any leaf fields.

但是，当`fields`模式`user`直接将嵌套字段作为目标时，将不返回任何值，因为该模式与任何叶字段都不匹配。

##  Retrieve unmapped fields

By default, the `fields` parameter returns only values of mapped fields. However, Elasticsearch allows storing fields in `_source` that are unmapped, such as setting [dynamic field mapping](https://www.elastic.co/guide/en/elasticsearch/reference/master/dynamic-field-mapping.html) to `false` or by using an object field with `enabled: false`. These options disable parsing and indexing of the object content.

默认情况下，该`fields`参数仅返回映射字段的值。但是，Elasticsearch允许存储`_source`未映射的字段，例如将[动态字段映射](https://www.elastic.co/guide/en/elasticsearch/reference/master/dynamic-field-mapping.html)设置为`false`或通过将对象字段设置为`enabled: false`。这些选项禁用对象内容的解析和索引编制。

To retrieve unmapped fields in an object from `_source`, use the `include_unmapped` option in the `fields` section:

要从`_source`中检索对象中未映射的字段，请使用`fields`中的 `include_unmapped`选项：

```json
PUT my-index-000001
{
  "mappings": {
    "enabled": false 
  }
}

PUT my-index-000001/_doc/1?refresh=true
{
  "user_id": "kimchy",
  "session_data": {
     "object": {
       "some_field": "some_value"
     }
   }
}

POST my-index-000001/_search
{
  "fields": [
    "user_id",
    {
      "field": "session_data.object.*",
      "include_unmapped" : true 
    }
  ],
  "_source": false
}
```

1. Disable all mappings.

    禁用所有映射。

2. Include unmapped fields matching this field pattern.

   包括与该字段模式匹配的未映射字段。

The response will contain field results under the `session_data.object.*` path, even if the fields are unmapped. The `user_id` field is also unmapped, but it won’t be included in the response because `include_unmapped` isn’t set to `true` for that field pattern.

即使未映射字段，响应也会在`session_data.object.*`路径下包含字段结果 。该`user_id`字段也未映射，但由于`include_unmapped`未设置该字段模式为`true` ，因此不会包含在响应中。

```json
{
  "took" : 2,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "session_data.object.some_field": [
            "some_value"
          ]
        }
      }
    ]
  }
}
```

##  The `_source` option

You can use the `_source` parameter to select what fields of the source are returned. This is called *source filtering*.

您可以使用该`_source`参数选择返回源的哪些字段。这称为*源过滤*。

The following search API request sets the `_source` request body parameter to `false`. The document source is not included in the response.

以下搜索API请求将`_source`请求正文参数设置为 `false`。该文档来源不包括在响应中。

```json
GET /_search
{
  "_source": false,
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

To return only a subset of source fields, specify a wildcard (`*`) pattern in the `_source` parameter. The following search API request returns the source for only the `obj` field and its properties.

要仅返回源字段的子集，请在`_source`参数中指定通配符（`*`）模式。以下搜索API请求仅返回该`obj`字段及其属性的源。

```json
GET /_search
{
  "_source": "obj.*",
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

You can also specify an array of wildcard patterns in the `_source` field. The following search API request returns the source for only the `obj1` and `obj2` fields and their properties.

您也可以在`_source`字段中指定一个通配符模式数组。以下搜索API请求仅返回`obj1`和 `obj2`字段及其属性的源。

```json
GET /_search
{
  "_source": [ "obj1.*", "obj2.*" ],
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

For finer control, you can specify an object containing arrays of `includes` and `excludes` patterns in the `_source` parameter.

为了更好地控制，您可以在 `_source` 参数中指定以`includes`和 `excludes`  为参数名的数组。

If the `includes` property is specified, only source fields that match one of its patterns are returned. You can exclude fields from this subset using the `excludes` property.

如果`includes`指定了属性，则仅返回与其模式之一匹配的源字段。您可以使用`excludes`属性从此子集中排除字段 。

If the `includes` property is not specified, the entire document source is returned, excluding any fields that match a pattern in the `excludes` property.

如果`includes`未指定该属性，则返回整个文档源，但不包括与该`excludes`属性中的模式匹配的任何字段。

The following search API request returns the source for only the `obj1` and `obj2` fields and their properties, excluding any child `description` fields.

以下搜索API请求仅返回`obj1`和 `obj2`字段及其属性的源，不包括任何子`description`字段。

```json
GET /_search
{
  "_source": {
    "includes": [ "obj1.*", "obj2.*" ],
    "excludes": [ "*.description" ]
  },
  "query": {
    "term": {
      "user.id": "kimchy"
    }
  }
}
```

##  Other methods of retrieving data

- **Using `fields` is typically better**

  **使用`fields`通常更好**
  
  These options are usually not required. Using the `fields` option is typically the better choice, unless you absolutely need to force loading a stored or `docvalue_fields`.
  
  通常不需要这些选项。`fields`除非您绝对需要强制加载存储的或，否则使用`docvalue_fields`选项通常是更好的选择 。

A document’s `_source` is stored as a single field in Lucene. This structure means that the whole `_source` object must be loaded and parsed even if you’re only requesting part of it. To avoid this limitation, you can try other options for loading fields:

文档的文档`_source`在Lucene中存储为单个字段。这种结构意味着`_source`即使只请求对象的一部分，也必须加载并解析整个对象。为避免此限制，您可以尝试其他选项来加载字段：

- Use the [`docvalue_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#docvalue-fields) parameter to get values for selected fields. This can be a good choice when returning a fairly small number of fields that support doc values, such as keywords and dates.

- 使用[`docvalue_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#docvalue-fields) 参数获取选定字段的值。当返回相当少量的支持doc值的字段（例如关键字和日期）时，这是一个不错的选择。

- Use the [`stored_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-request-body.html#request-body-search-stored-fields) parameter to get the values for specific stored fields (fields that use the [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-store.html) mapping option).

  使用[`stored_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-request-body.html#request-body-search-stored-fields)参数获取特定存储字段（使用[`store`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-store.html)映射选项的字段 ）的值。

Elasticsearch always attempts to load values from `_source`. This behavior has the same implications of source filtering where Elasticsearch needs to load and parse the entire `_source` to retrieve just one field.

Elasticsearch总是尝试从中加载值`_source`。此行为与源过滤具有相同的含义，在此情况下，仅检索一个字段Elasticsearch需要加载并解析整个 `_source`字段。

##  Doc value fields

You can use the [`docvalue_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#docvalue-fields) parameter to return [doc values](https://www.elastic.co/guide/en/elasticsearch/reference/master/doc-values.html) for one or more fields in the search response.

您可以使用[`docvalue_fields`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#docvalue-fields)参数返回 搜索响应中一个或多个字段的[doc值](https://www.elastic.co/guide/en/elasticsearch/reference/master/doc-values.html)。

Doc values store the same values as the `_source` but in an on-disk, column-based structure that’s optimized for sorting and aggregations. Since each field is stored separately, Elasticsearch only reads the field values that were requested and can avoid loading the whole document `_source`.

Doc值存储与doc相同的值，`_source`但在磁盘上基于列的结构中进行了优化，该结构针对排序和聚合进行了优化。由于每个字段都是单独存储的，因此Elasticsearch仅读取请求的字段值，并且可以避免加载整个文档`_source`。

Doc values are stored for supported fields by default. However, doc values are not supported for [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) or [`text_annotated`](https://www.elastic.co/guide/en/elasticsearch/plugins/master/mapper-annotated-text-usage.html) fields.

默认情况下，将为支持的字段存储文档值。但是，[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html)或 [`text_annotated`](https://www.elastic.co/guide/en/elasticsearch/plugins/master/mapper-annotated-text-usage.html)字段不支持doc值。

The following search request uses the `docvalue_fields` parameter to retrieve doc values for the `user.id` field, all fields starting with `http.response.`, and the `@timestamp` field:

以下搜索请求使用`docvalue_fields`来检索DOC值参数`user.id`，各个字段以`http.response.`开始，和`@timestamp`字段：

```json
GET my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "docvalue_fields": [
    "user.id",
    "http.response.*", 
    {
      "field": "date",
      "format": "epoch_millis" 
    }
  ]
}
```

1. Both full field names and wildcard patterns are accepted.

   完整字段名和通配符模式都可以接受。

2. Using object notation, you can pass a format parameter to apply a custom format for the field’s doc values. Date fields support a date format. Numeric fields support a DecimalFormat pattern. Other field datatypes do not support the format parameter.

    使用对象表示法，您可以传递`format`参数以对字段的doc值应用自定义格式。[日期字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html)支持 [日期`format`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)。[数值字段](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)支持 [DecimalFormat模式](https://docs.oracle.com/javase/8/docs/api/java/text/DecimalFormat.html)。其他字段数据类型不支持该`format`参数。

> **TIP:** You cannot use the `docvalue_fields` parameter to retrieve doc values for nested objects. If you specify a nested object, the search returns an empty array (`[ ]`) for the field. To access nested fields, use the [`inner_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/inner-hits.html) parameter’s `docvalue_fields` property.
>
> 提示：您不能使用该`docvalue_fields`参数来检索嵌套对象的doc值。如果指定嵌套对象，则搜索将为该字段返回一个空数组（`[ ]`）。要访问嵌套字段，请使用 [`inner_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/inner-hits.html)参数的`docvalue_fields` 属性。

##  Stored fields

It’s also possible to store an individual field’s values by using the [`store`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-store.html) mapping option. You can use the `stored_fields` parameter to include these stored values in the search response.

也可以通过使用[`store`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-store.html)映射选项来存储单个字段的值 。您可以使用 `stored_fields`参数将这些存储的值包括在搜索响应中。

> **WARNING:** The `stored_fields` parameter is for fields that are explicitly marked as stored in the mapping, which is off by default and generally not recommended. Use [source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#source-filtering) instead to select subsets of the original source document to be returned.
>
> 警告：该`stored_fields`参数用于显式标记为存储在映射中的字段，默认情况下处于关闭状态，通常不建议这样做。而是使用[源过滤](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#source-filtering)来选择要返回的原始源文档的子集。

Allows to selectively load specific stored fields for each document represented by a search hit.

允许有选择地为搜索命中表示的每个文档加载特定的存储字段。

```json
GET /_search
{
  "stored_fields" : ["user", "postDate"],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

`*` can be used to load all stored fields from the document.

`*` 可用于从文档加载所有存储的字段。

An empty array will cause only the `_id` and `_type` for each hit to be returned, for example:

空数组将仅使`_id`与`_type`每个命中被返回，例如：

```json
GET /_search
{
  "stored_fields" : [],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

If the requested fields are not stored (`store` mapping set to `false`), they will be ignored.

如果未存储请求的字段（`store`映射设置为`false`），则将忽略它们。

Stored field values fetched from the document itself are always returned as an array. On the contrary, metadata fields like `_routing` are never returned as an array.

从文档本身获取的存储字段值始终以数组形式返回。相反，像`_routing`这样的元数据字段永远不会作为数组返回。

Also only leaf fields can be returned via the `stored_fields` option. If an object field is specified, it will be ignored.

也只能通过该`stored_fields`选项返回叶字段。如果指定了对象字段，它将被忽略。

> **NOTE:** On its own, `stored_fields` cannot be used to load fields in nested objects — if a field contains a nested object in its path, then no data will be returned for that stored field. To access nested fields, `stored_fields` must be used within an [`inner_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/inner-hits.html) block.
>
> 注意：就其本身而言，`stored_fields`不能被用于加载嵌套的对象字段-如果一个字段包含在其路径嵌套的对象，那么将没有数据返回存储领域。要访问嵌套字段，`stored_fields` 必须在一个[`inner_hits`](https://www.elastic.co/guide/en/elasticsearch/reference/master/inner-hits.html)块内使用。

##  Disable stored fields

To disable the stored fields (and metadata fields) entirely use: `_none_`:

要完全禁用存储的字段（和元数据字段），请使用`_none_`：

```json
GET /_search
{
  "stored_fields": "_none_",
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

> **NOTE:** [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#source-filtering) and [`version`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-version) parameters cannot be activated if `_none_` is used.
>
> 注意：`_none_`使用时，[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-fields.html#source-filtering)和[`version`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#request-body-search-version)参数不能被激活。

##  Script fields

You can use the `script_fields` parameter to retrieve a [script evaluation](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html) (based on different fields) for each hit. For example:

您可以使用该`script_fields`参数为每个匹配检索[脚本评估](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html)（基于不同的字段）。例如：

```json
GET /_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "test1": {
      "script": {
        "lang": "painless",
        "source": "doc['price'].value * 2"
      }
    },
    "test2": {
      "script": {
        "lang": "painless",
        "source": "doc['price'].value * params.factor",
        "params": {
          "factor": 2.0
        }
      }
    }
  }
}
```

Script fields can work on fields that are not stored (`price` in the above case), and allow to return custom values to be returned (the evaluated value of the script).

脚本字段可以在未存储的字段上工作（在上述情况下`price`），并允许返回要返回的自定义值（脚本的评估值）。

Script fields can also access the actual `_source` document and extract specific elements to be returned from it by using `params['_source']`. Here is an example:

脚本字段还可以使用来访问实际`_source`文档并提取要从中返回的特定元素`params['_source']`。这是一个例子：

```json
GET /_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "test1": {
      "script": "params['_source']['message']"
    }
  }
}
```

Note the `_source` keyword here to navigate the json-like model.

注意`_source`此处的关键字以浏览类似json的模型。

It’s important to understand the difference between `doc['my_field'].value` and `params['_source']['my_field']`. The first, using the doc keyword, will cause the terms for that field to be loaded to memory (cached), which will result in faster execution, but more memory consumption. Also, the `doc[...]` notation only allows for simple valued fields (you can’t return a json object from it) and makes sense only for non-analyzed or single term based fields. However, using `doc` is still the recommended way to access values from the document, if at all possible, because `_source` must be loaded and parsed every time it’s used. Using `_source` is very slow.

理解`doc['my_field'].value` 和 `params['_source']['my_field']`之间的区别时很重要的。首先使用doc关键字，将导致将该字段的术语加载到内存中（缓存），这将导致执行速度更快，但会占用更多内存。另外，该`doc[...]`表示法仅允许使用简单值字段（您不能从中返回json对象），并且仅对未分析或基于单个术语的字段有意义。但是，`doc`仍然建议使用using来访问文档中的值（如果可能的话），因为`_source`每次使用时都必须加载和解析它。使用`_source`非常慢。

# Search across clusters

**Cross-cluster search** 跨节点搜索可以让你通过一个搜索请求，请求一个或多个远程节点。可以通过该搜索过滤和分析存储在不同节点的数据中心的日志。

## Supported APIs

- [Search](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)
- [Async search](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html)
- [Multi search](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-multi-search.html)
- [Search template](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-template.html)
- [Multi search template](https://www.elastic.co/guide/en/elasticsearch/reference/master/multi-search-template.html)
- [Field capabilities](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-field-caps.html)

### **Cross-cluster search examples**

#### Remote cluster setup

要进行跨节点搜索至少需要一个远程节点

例子：可以使用[cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-update-settings.html) API来添加三个远程节点。

```json
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster_one": {
          "seeds": [
            "127.0.0.1:9300"
          ]
        },
        "cluster_two": {
          "seeds": [
            "127.0.0.1:9301"
          ]
        },
        "cluster_three": {
          "seeds": [
            "127.0.0.1:9302"
          ]
        }
      }
    }
  }
}
```

#### Search a single remote cluster

```json
GET /cluster_one:my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "_source": ["user.id", "message", "http.response.status_code"]
}
```

response:

```json
{
  "took": 150,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0,
    "skipped": 0
  },
  "_clusters": {
    "total": 1,
    "successful": 1,
    "skipped": 0
  },
  "hits": {
    "total" : {
        "value": 1,
        "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "cluster_one:my-index-000001", 
        "_id": "0",
        "_score": 1,
        "_source": {
          "user": {
            "id": "kimchy"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "http": {
            "response":
              {
                "status_code": 200
              }
          }
        }
      }
    ]
  }
}
```

#### Search multiple remote clusters

同样使用[search](https://www.elastic.co/guide/en/elasticsearch/reference/master/search.html)API查询my-index-000001索引在三个节点上：

- 本地节点
- 两个远程节点，cluster_one 和 cluster_two

```json
GET /my-index-000001,cluster_one:my-index-000001,cluster_two:my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "_source": ["user.id", "message", "http.response.status_code"]
}
```



The API returns the following response:

```json
{
  "took": 150,
  "timed_out": false,
  "num_reduce_phases": 4,
  "_shards": {
    "total": 3,
    "successful": 3,
    "failed": 0,
    "skipped": 0
  },
  "_clusters": {
    "total": 3,
    "successful": 3,
    "skipped": 0
  },
  "hits": {
    "total" : {
        "value": 3,
        "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "my-index-000001", 
        "_id": "0",
        "_score": 2,
        "_source": {
          "user": {
            "id": "kimchy"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "http": {
            "response":
              {
                "status_code": 200
              }
          }
        }
      },
      {
        "_index": "cluster_one:my-index-000001", 
        "_id": "0",
        "_score": 1,
        "_source": {
          "user": {
            "id": "kimchy"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "http": {
            "response":
              {
                "status_code": 200
              }
          }
        }
      },
      {
        "_index": "cluster_two:my-index-000001", 
        "_id": "0",
        "_score": 1,
        "_source": {
          "user": {
            "id": "kimchy"
          },
          "message": "GET /search HTTP/1.1 200 1070000",
          "http": {
            "response":
              {
                "status_code": 200
              }
          }
        }
      }
    ]
  }
}
```

本地节点的响应_index中是不包含节点名的

## Skip unavailable clusters

By default, a cross-cluster search returns an error if **any** cluster in the request is unavailable.

To skip an unavailable cluster during a cross-cluster search, set the [`skip_unavailable`](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-remote-info.html#skip-unavailable) cluster setting to `true`.

The following [cluster update settings](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-update-settings.html) API request changes `cluster_two`'s `skip_unavailable` setting to `true`.

默认情况下，任何一个节点不可用的时候跨集群检索都会返回错误。

想要在跨集群检索的时候跳过不可用节点可以设置集群设置（cluster setting）中的 `skip_unavailable`为`true`



```json
PUT _cluster/settings
{
  "persistent": {
    "cluster.remote.cluster_two.skip_unavailable": true
  }
}
```

If `cluster_two` is disconnected or unavailable during a cross-cluster search, Elasticsearch won’t include matching documents from that cluster in the final results.

如果节点二在跨集群检索的时候失去连接或者不可用，则最终返回的结果中不会包含该节点的文档

##  Selecting gateway and seed nodes in sniff mode

使用sniff mode（嗅觉模式）去搜索网关和种子节点

For remote clusters using the [sniff connection](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-remote-clusters.html#sniff-mode) mode, gateway and seed nodes need to be accessible from the local cluster via your network.

By default, any non-[master-eligible](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-node.html#master-node) node can act as a gateway node. If wanted, you can define the gateway nodes for a cluster by setting `cluster.remote.node.attr.gateway` to `true`.

For cross-cluster search, we recommend you use gateway nodes that are capable of serving as [coordinating nodes](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-node.html#coordinating-node) for search requests. If wanted, the seed nodes for a cluster can be a subset of these gateway nodes.

当集群使用嗅觉连接模式时，网关和种子节点需要通过网络从本地集群节点变为可用。

默认，任何非主节点候选节点都可以作为网关节点。可以定义网关节点从集群设置`cluster.remote.node.attr.gateway` 为 `true`。

对于跨集群检索，我们建议使用这个有能力作为协同节点服务请求的网关节点。如果想要的话种子节点可以成为这些网关节点的子节点。

##  Cross-cluster search in proxy mode

[Proxy mode](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-remote-clusters.html#proxy-mode) remote cluster connections support cross-cluster search. All remote connections connect to the configured `proxy_address`. Any desired connection routing to gateway or [coordinating nodes](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-node.html#coordinating-node) must be implemented by the intermediate proxy at this configured address.

代理模式的远程节点连接是支持跨集群检索的。所有的远程连接连接到配置的`proxy_address`（代理路径）。任何连接想要路由到网关或者[coordinating nodes](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-node.html#coordinating-node) （协同节点）的话必须通过这个配置的中间代理节点地址实现。

## How cross-cluster search handles network delays

Because cross-cluster search involves sending requests to remote clusters, any network delays can impact search speed. To avoid slow searches, cross-cluster search offers two options for handling network delays:

- **[Minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-cross-cluster-search.html#ccs-min-roundtrips)**

  By default, Elasticsearch reduces the number of network roundtrips between remote clusters. This reduces the impact of network delays on search speed. However, Elasticsearch can’t reduce network roundtrips for large search requests, such as those including a [scroll](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results) or [inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/master/inner-hits.html).See [Minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-cross-cluster-search.html#ccs-min-roundtrips) to learn how this option works.

- **[Don’t minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-cross-cluster-search.html#ccs-unmin-roundtrips)**

  For search requests that include a scroll or inner hits, Elasticsearch sends multiple outgoing and ingoing requests to each remote cluster. You can also choose this option by setting the [`ccs_minimize_roundtrips`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#ccs-minimize-roundtrips) parameter to `false`. While typically slower, this approach may work well for networks with low latency.See [Don’t minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-cross-cluster-search.html#ccs-unmin-roundtrips) to learn how this option works.

因为跨集群检索包含了向远程节点发送请求的过程，任何网络延迟都可能影响到检索的速度。为了避免慢检索，跨集群检索提供两点来控制网络延迟：

- 最小化网络往返

  默认情况下，ES会减少在两个远程节点的网络往返次数。从而减少检索速度受到网络延迟的影响。然而，ES不能减少大检索请求的网络往返，像那些包含[scroll](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results) 或 [inner hits](https://www.elastic.co/guide/en/elasticsearch/reference/master/inner-hits.html).

  查看 [Minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-cross-cluster-search.html#ccs-min-roundtrips) 来学习这个选项是怎么工作的。

- 不最小化网络往返

  当检索请求包含scroll或inner hits，ES会发送多个向外和向内的请求给每一个远程节点。你也可以选择这个选项，通过设置[`ccs_minimize_roundtrips`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#ccs-minimize-roundtrips) 参数为`false` 。通常网络缓慢的时候这个方法可能会工作的不错，带来比较低的延迟。

  查看 [Don’t minimize network roundtrips](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-cross-cluster-search.html#ccs-unmin-roundtrips)来学习这个选项是怎么工作的。

## Minimize network roundtrips

Here’s how cross-cluster search works when you minimize network roundtrips.

1. You send a cross-cluster search request to your local cluster. A coordinating node in that cluster receives and parses the request.

   你可以发送一个跨集群检索请求给你的本地节点。该集群的一个协同节点会解析这个请求。

   ![ccs min roundtrip client request](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-min-roundtrip-client-request.svg)

2. The coordinating node sends a single search request to each cluster, including the local cluster. Each cluster performs the search request independently, applying its own cluster-level settings to the request.

   这个协同节点会给每一个集群发送一个单独查询请求，包括本地节点。每一个集群都是单独执行这个请求，将其自己的群集级别设置应用于该请求。

   ![ccs min roundtrip cluster search](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-min-roundtrip-cluster-search.svg)

3. Each remote cluster sends its search results back to the coordinating node.

   每个远程集群将他们的检索结果发送回协同节点。

   ![ccs min roundtrip cluster results](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-min-roundtrip-cluster-results.svg)

4. After collecting results from each cluster, the coordinating node returns the final results in the cross-cluster search response.

   在搜集了每一个集群的返回结果后，协同节点会在跨集群检索返回最终的结果。

   ![ccs min roundtrip client response](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-min-roundtrip-client-response.svg)

## Don’t minimize network roundtrips

Here’s how cross-cluster search works when you don’t minimize network roundtrips.

1. You send a cross-cluster search request to your local cluster. A coordinating node in that cluster receives and parses the request.

   你可以发送跨集群检索请求到本地集群。该集群的协同节点会解析这个请求。

   ![ccs min roundtrip client request](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-min-roundtrip-client-request.svg)

2. The coordinating node sends a [search shards](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-shards.html) API request to each remote cluster.

   这个协同节点发送一个 [search shards](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-shards.html) API 请求到每一个远程集群。

   ![ccs min roundtrip cluster search](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-min-roundtrip-cluster-search.svg)

3. Each remote cluster sends its response back to the coordinating node. This response contains information about the indices and shards the cross-cluster search request will be executed on.

   每个远程集群将响应发送回协同节点。这个响应包含的信息是关于哪些索引和分片是这个跨集群检索请求将要执行的。

   ![ccs min roundtrip cluster results](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-min-roundtrip-cluster-results.svg)

4. The coordinating node sends a search request to each shard, including those in its own cluster. Each shard performs the search request independently.

   协同节点发送检索请求到每一个分片（包括自身集群的分片），每一个分片独立执行检索请求。

   When network roundtrips aren’t minimized, the search is executed as if all data were in the coordinating node’s cluster. We recommend updating cluster-level settings that limit searches, such as `action.search.shard_count.limit`, `pre_filter_shard_size`, and `max_concurrent_shard_requests`, to account for this. If these limits are too low, the search may be rejected.

   当网络往返不是最小化的时候，该请求执行方法所有数据都在协同节点上。我们建议更新集群级别的设置，以限制搜索，例如`action.search.shard_count.limit`， `pre_filter_shard_size`和`max_concurrent_shard_requests`，考虑到这一点。如果这些限制太低，则搜索可能会被拒绝。

   ![ccs dont min roundtrip shard search](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-dont-min-roundtrip-shard-search.svg)

5. Each shard sends its search results back to the coordinating node.

   每一个分片将结果发送回协同节点。

   ![ccs dont min roundtrip shard results](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-dont-min-roundtrip-shard-results.svg)

6. After collecting results from each cluster, the coordinating node returns the final results in the cross-cluster search response.

   在收集了每一个集群的结果后，协同节点将在跨集群检索响应中返回最终结果。

   ![ccs min roundtrip client response](https://www.elastic.co/guide/en/elasticsearch/reference/master/images/ccs/ccs-min-roundtrip-client-response.svg)

## Supported configurations

Generally, [cross cluster search](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-remote-clusters.html#gateway-nodes-selection) can search remote clusters that are one major version ahead or behind the coordinating node’s version. Cross cluster search can also search remote clusters that are being [upgraded](https://www.elastic.co/guide/en/elasticsearch/reference/master/rolling-upgrades.html) so long as both the "upgrade from" and "upgrade to" version are compatible with the gateway node.

通常来说，跨集群检索可以检索比协同节点版本高或低的远程集群。跨群集搜索还可以搜索要[升级的](https://www.elastic.co/guide/en/elasticsearch/reference/master/rolling-upgrades.html)远程群集， 只要“升级前”和“升级后”版本都与网关节点兼容即可。

For example, a coordinating node running Elasticsearch 5.6 can search a remote cluster running Elasticsearch 6.8, but that cluster can not be upgraded to 7.1. In this case you should first upgrade the coordinating node to 7.1 and then upgrade remote cluster.

举个例子，一个协同节点运行ES版本5.6可以去检索远程集群ES版本6.8，但是这个集群不能被升级到7.1。在这个案例你应该先将协同节点的ES版本升级到7.1然后升级远程集群的版本。

Running multiple versions of Elasticsearch in the same cluster beyond the duration of an upgrade is not supported.

不支持在升级持续时间内在同一集群中运行多个版本的Elasticsearch。

Only features that exist across all searched clusters are supported. Using a recent feature with a remote cluster where the feature is not supported will result in undefined behavior.

只有所有集群上都存在的特性才被支持。使用最新的特性在不同的远程集群中如果有集群不支持该特性将会返回未定义行为。



#  Search multiple data streams and indices

To search multiple data streams and indices, add them as comma-separated values in the [search API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)'s request path.

检索多个数据流和索引，可以将他们以逗号分隔添加到 [search API](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)'s的请求路径中。

The following request searches the `my-index-000001` and `my-index-000002` indices.

如下检索`my-index-000001` 和`my-index-000002`索引。

``` json
GET /my-index-000001,my-index-000002/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

You can also search multiple data streams and indices using an index pattern.

也可以用索引模式检索多个数据流和索引。

The following request targets the `my-index-*` index pattern. The request searches any data streams or indices in the cluster that start with `my-index-`.

以下请求以`my-index-*`为索引模式目标。这个请求检索在集群节点中以 `my-index-`开头的数据流和索引。

``` json
GET /my-index-*/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

To search all data streams and indices in a cluster, omit the target from the request path. Alternatively, you can use `_all` or `*`.

要检索集群上的全部数据流和索引，可以忽略请求中的路径目标。或者可以使用 `_all` 或 `*`。

The following requests are equivalent and search all data streams and indices in the cluster.

下列请求检索集群上的所有数据流和索引是等价的。

``` json
GET /_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}

GET /_all/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}

GET /*/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

## Index boost

When searching multiple indices, you can use the `indices_boost` parameter to boost results from one or more specified indices. This is useful when hits coming from some indices matter more than hits from other.

当检索多个数据流时，你可以使用`indices_boost`参数来增强从一个或多个指定的索引。当一些索引的命中结果比其他命中更重要的时候，这个参数十分有用。

You cannot use `indices_boost` with data streams.

不能把`indices_boost` 和数据流一起使用。

```json
GET /_search
{
  "indices_boost": [
    { "my-index-000001": 1.4 },
    { "my-index-000002": 1.3 }
  ]
}
```

Index aliases and index patterns can also be used:

索引别名和索引参数一样能够使用：

```json
GET /_search
{
  "indices_boost": [
    { "my-alias":  1.4 },
    { "my-index*": 1.3 }
  ]
}
```

If multiple matches are found, the first match will be used. For example, if an index is included in `alias1` and matches the `my-index*` pattern, a boost value of `1.4` is applied.

如果找到了多个匹配，最先被找到的匹配将会生效。举个例子，如果一个索引被`alias1` 包含又被`my-index*`参数匹配，`1.4`这个增强值将会生效。

PS:似乎不能用来nested检索（v7.2）

# Search shard routing

To protect against hardware failure and increase search capacity, Elasticsearch can store copies of an index’s data across multiple shards on multiple nodes. When running a search request, Elasticsearch selects a node containing a copy of the index’s data and forwards the search request to that node’s shards. This process is known as *search shard routing* or *routing*.

为了防止硬件故障增加搜索容量，ES存储了一个索引数据的备份在多个分片多个节点上。当运行一个检索请求的时候，ES会选择存在这个索引数据副本的节点并将搜索请求转发到这个节点的分片。这个过程被称为*搜索分片路由* 或 *路由*。

## Adaptive replica selection

By default, Elasticsearch uses *adaptive replica selection* to route search requests. This method selects an eligible node using [shard allocation awareness](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-cluster.html#shard-allocation-awareness) and the following criteria:

- Response time of prior requests between the coordinating node and the eligible node
- How long the eligible node took to run previous searches
- Queue size of the eligible node’s `search` [threadpool](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-threadpool.html)

Adaptive replica selection is designed to decrease search latency. However, you can disable adaptive replica selection by setting `cluster.routing.use_adaptive_replica_selection` to `false` using the [cluster settings API](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-update-settings.html). If disabled, Elasticsearch routes search requests using a round-robin method, which may result in slower searches.

默认，ES使用*自适应副本选择* 来路由检索请求。这个方法使用 [分片分配意识](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-cluster.html#shard-allocation-awareness)和以下标准选择一个合适的节点：

- 在协同节点和合适节点间优先请求响应时间。

- 合适节点执行上一个检索花费的时间。

- 合适节点的`search`[线程池](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-threadpool.html)的队列大小。

自适应副本选择时被设计来减少搜索延迟的。然而，你可以通过使用 [cluster settings API](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-update-settings.html)设置`cluster.routing.use_adaptive_replica_selection` 为 `false`来禁用自适应副本选择。如果被禁用，ES会使用循环方法路由检索请求，这可能会导致搜索请求速度变慢。

## Set a preference

By default, adaptive replica selection chooses from all eligible nodes and shards. However, you may only want data from a local node or want to route searches to a specific node based on its hardware. Or you may want to send repeated searches to the same shard to take advantage of caching.

默认，自适应副本选择在所有合格的节点和分片中。然而，你可能只想要获取数据从本地节点或者路由检索指定的节点在他们的硬件上。或者你可能想要发送重复的检索在相同的分片来享受缓存的好处。

To limit the set of nodes and shards eligible for a search request, use the search API’s [`preference`](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html#search-preference) query parameter.

可以用search API’s的`preference`查询参数来限制检索用的合格的节点和分片集合。

For example, the following request searches `my-index-000001` with a `preference` of `_local`. This restricts the search to shards on the local node. If the local node contains no shard copies of the index’s data, the request uses adaptive replica selection to another eligible node as a fallback.

举个例子，如下请求带倾向`_local`的`preference`偏好来检索 `my-index-000001`。这个限制了检索只在本地节点上。如果本地节点不存在数据的副本，这个请求会使用自适应副本选择到其他合法节点作为返回。

```json
GET /my-index-000001/_search?preference=_local
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

You can also use the `preference` parameter to route searches to specific shards based on a provided string. If the cluster state and selected shards do not change, searches using the same `preference` string are routed to the same shards in the same order.

你也可以使用 `preference` 参数路由检索到指定的分片基于指定提供者。如果集群的状态和选中分片的没有改变，检索会使用同样的`preference` 字符串来路由到相同的分片以相同的顺序。

We recommend using a unique `preference` string, such as a user name or web session ID. This string cannot start with a `_`.

我们推荐使用唯一的 `preference` 字符串，就像用户名或者web session ID。这个字符串不能以 _ 开头。

You can use this option to serve cached results for frequently used and resource-intensive searches. If the shard’s data doesn’t change, repeated searches with the same `preference` string retrieve results from the same [shard request cache](https://www.elastic.co/guide/en/elasticsearch/reference/master/shard-request-cache.html). For time series use cases, such as logging, data in older indices is rarely updated and can be served directly from this cache.

你可以使用这个选项来提供带缓存的结果给那些频繁使用且占用大量资源的搜索。如果该分片的数据没有变动，重复的检索将会使用相同的`preference`字符串来从同样的 [分片请求缓存](https://www.elastic.co/guide/en/elasticsearch/reference/master/shard-request-cache.html)取回结果集。对于具有时间顺序的案例，例如日志，数据在旧的索引中很少被更新可以直接返回缓存的数据。

The following request searches `my-index-000001` with a `preference` string of `my-custom-shard-string`.

如下请求使用`my-custom-shard-string`来 `preference` 偏好字符串用于检索 `my-index-000001`。

```json
GET /my-index-000001/_search?preference=my-custom-shard-string
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

If the cluster state or selected shards change, the same `preference` string may not route searches to the same shards in the same order. This can occur for a number of reasons, including shard relocations and shard failures. A node can also reject a search request, which Elasticsearch would re-route to another node.

如果集群的状态或者选中的分片的状态改变了，则相同的 `preference` 偏好字符串可能不能路由检索到相同的分片上并使用相同的顺序。这可能会发生与数字类型的结果，包含分片重定位和分片故障。一个节点也会拒绝检索请求，ES会重新路由检索请求到其他节点。

##  Use a routing value

When you index a document, you can specify an optional [routing value](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-routing-field.html), which routes the document to a specific shard.

当索引一个文档的时候，可以指定路由值，将文档路由到这个指定的路由值。

For example, the following indexing request routes a document using `my-routing-value`.

下面这个例子，使用`my-routing-value`作为索引请求的路由。

```json
POST /my-index-000001/_doc?routing=my-routing-value
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
```

You can use the same routing value in the search API’s `routing` query parameter. This ensures the search runs on the same shard used to index the document.

你可以使用相同的路由值在`routing`检索API的请求参数中。这确保这个检索运行在相同的分片中去索引这个文档。

```json
GET /my-index-000001/_search?routing=my-routing-value
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

You can also provide multiple comma-separated routing values:

你也可以使用多种逗号分隔的路由：

```json
GET /my-index-000001/_search?routing=my-routing-value,my-routing-value-2
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

##  Search concurrency and parallelism

By default, Elasticsearch doesn’t reject search requests based on the number of shards the request hits. However, hitting a large number of shards can significantly increase CPU and memory usage.

默认情况下，Elasticsearch不会基于请求命中的分片数量拒绝搜索请求。但是，命中大量分片会大大增加CPU和内存使用率。

For tips on preventing indices with large numbers of shards, see [Avoid oversharding](https://www.elastic.co/guide/en/elasticsearch/reference/master/avoid-oversharding.html).

有关防止具有大量分片的索引的提示，请参阅 [避免过度分片](https://www.elastic.co/guide/en/elasticsearch/reference/master/avoid-oversharding.html)。

You can use the `max_concurrent_shard_requests` query parameter to control maximum number of concurrent shards a search request can hit per node. This prevents a single request from overloading a cluster. The parameter defaults to a maximum of `5`.

您可以使用`max_concurrent_shard_requests`查询参数来控制搜索请求可以在每个节点上命中的最大并发分片数。这样可以防止单个请求使群集过载。参数默认最大为`5`。

```json
GET /my-index-000001/_search?max_concurrent_shard_requests=3
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

You can also use the `action.search.shard_count.limit` cluster setting to set a search shard limit and reject requests that hit too many shards. You can configure `action.search.shard_count.limit` using the [cluster settings API](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-update-settings.html).

您还可以使用`action.search.shard_count.limit`群集设置来设置搜索分片限制，并拒绝命中太多分片的请求。您可以`action.search.shard_count.limit`使用 [集群设置API](https://www.elastic.co/guide/en/elasticsearch/reference/master/cluster-update-settings.html)进行配置。

# Sort search results

Allows you to add one or more sorts on specific fields. Each sort can be reversed as well. The sort is defined on a per field level, with special field name for `_score` to sort by score, and `_doc` to sort by index order.

允许添加一种或多种排序在指定字段上。每个排序可以被反转。排序被定义在每一个字段级别上。通过得分排序对 `_score`有特殊的字段名称，`_doc`被索引顺序所排序。

Assuming the following index mapping:

假如像如下索引：

```json
PUT /my-index-000001
{
  "mappings": {
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
```

```json
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"order" : "asc", "format": "strict_date_optional_time_nanos"}},
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

`_doc` has no real use-case besides being the most efficient sort order. So if you don’t care about the order in which documents are returned, then you should sort by `_doc`. This especially helps when [scrolling](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results).

`_doc`除了作为最有效的排序顺序之外没有其他实际作用。所以如果你不关心文档返回的顺序，则应该按`_doc`排序。这在 [scrolling](https://www.elastic.co/guide/en/elasticsearch/reference/master/paginate-search-results.html#scroll-search-results)滚动时特别有用。

##  Sort Values

The search response includes `sort` values for each document. Use the `format` parameter to specify a [date format](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html#built-in-date-formats) for the `sort` values of [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html) and [`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html) fields. The following search returns `sort` values for the `post_date` field in the `strict_date_optional_time_nanos` format.

检索响应每一个文档都会包含 `sort` 值。使用`format` 参数指定日期格式化格式为 [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html)  和[`date_nanos`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date_nanos.html)字段类型作为排序值。下面的检索为`post_date`字段返回 `sort` 排序值使用 `strict_date_optional_time_nanos` 格式化。

```json
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"format": "strict_date_optional_time_nanos"}}
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

##  Sort Order

The `order` option can have the following values:

 `order` 参数可以使用如下选项值：

| `asc`  | Sort in ascending order  |
| ------ | ------------------------ |
| `desc` | Sort in descending order |

The order defaults to `desc` when sorting on the `_score`, and defaults to `asc` when sorting on anything else.

当使用`_score`进行排序时默认时降序的，当使用其他字段的时候默认是升序。

##  Sort mode option

Elasticsearch supports sorting by array or multi-valued fields. The `mode` option controls what array value is picked for sorting the document it belongs to. The `mode` option can have the following values:

ES支持对数组或多值字段排序。该`mode`选项控制选择哪个数组值来对它所属的文档进行排序。该`mode`选项可以具有以下值：

| `min`    | Pick the lowest value. 选择最小值                            |
| -------- | ------------------------------------------------------------ |
| `max`    | Pick the highest value. 选择最大值                           |
| `sum`    | Use the sum of all values as sort value. Only applicable for number based array fields. 使用所有值的总和作为排序值。仅适用于基于数字的数组字段。 |
| `avg`    | Use the average of all values as sort value. Only applicable for number based array fields.使用平均值来作为排序值。仅适用于基于数字的数组字段。 |
| `median` | Use the median of all values as sort value. Only applicable for number based array fields.使用中位值作为排序值。仅适用于基于数字的数组字段。 |

The default sort mode in the ascending sort order is `min` — the lowest value is picked. The default sort mode in the descending order is `max` — the highest value is picked.

升序排序的默认排序模式是`min` -选择最小值。降序排序的默认排序模式是`max` -选择最大值。

##  Sort mode example usage

In the example below the field price has multiple prices per document. In this case the result hits will be sorted by price ascending based on the average price per document.

在下面这个例子中每个文档字段有多种价值。这个例子中命中结果将会基于每个文档的平均价值排序。

```json
PUT /my-index-000001/_doc/1?refresh
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

## Sorting numeric fields

For numeric fields it is also possible to cast the values from one type to another using the `numeric_type` option. This option accepts the following values: [`"double", "long", "date", "date_nanos"`] and can be useful for searches across multiple data streams or indices where the sort field is mapped differently.

对于数值类型字段也可以使用`numeric_type` 选项将值从一种类型转换成另一种。这个选项接受的值包括：`"double", "long", "date", "date_nanos"`，对跨多数据流检索或索引（排序字段映射不同）的索引十分有用。

Consider for instance these two indices:

思考下面两个例子：

```json
PUT /index_double
{
  "mappings": {
    "properties": {
      "field": { "type": "double" }
    }
  }
}
```

```json
PUT /index_long
{
  "mappings": {
    "properties": {
      "field": { "type": "long" }
    }
  }
}
```

Since `field` is mapped as a `double` in the first index and as a `long` in the second index, it is not possible to use this field to sort requests that query both indices by default. However you can force the type to one or the other with the `numeric_type` option in order to force a specific type for all indices:

第一个索引使用`double` 作为字段映射，第二个索引使用 `long`作为字段映射，在默认情况这两个索引不可能使用这个字段来做排序请求。然而你可以使用`numeric_type` 选项强制转换类型成同一个，以便于对所有索引使用指定的类型排序。

```json
POST /index_long,index_double/_search
{
   "sort" : [
      {
        "field" : {
            "numeric_type" : "double"
        }
      }
   ]
}
```

In the example above, values for the `index_long` index are casted to a double in order to be compatible with the values produced by the `index_double` index. It is also possible to transform a floating point field into a `long` but note that in this case floating points are replaced by the largest value that is less than or equal (greater than or equal if the value is negative) to the argument and is equal to a mathematical integer.

在上一个例子中，`index_long` 索引的值被转成了double类型以便于和`index_double`索引产生的值兼容。当然也可以将浮点字段为`long`，但是请注意这个例子中浮点数被替换成了一个范围更大的值（如果这个值大于或等于范围则是负数）作为参数，他等于一个精确的整数。

This option can also be used to convert a `date` field that uses millisecond resolution to a `date_nanos` field with nanosecond resolution. Consider for instance these two indices:

这个选项也可以被用于转换`date`日期字段使用毫秒解析为一个 `date_nanos` 字段。思考下面两个例子：

```json
PUT /index_double
{
  "mappings": {
    "properties": {
      "field": { "type": "date" }
    }
  }
}
```

```json
PUT /index_long
{
  "mappings": {
    "properties": {
      "field": { "type": "date_nanos" }
    }
  }
}
```

Values in these indices are stored with different resolutions so sorting on these fields will always sort the `date` before the `date_nanos` (ascending order). With the `numeric_type` type option it is possible to set a single resolution for the sort, setting to `date` will convert the `date_nanos` to the millisecond resolution while `date_nanos` will convert the values in the `date` field to the nanoseconds resolution:

这些索引的值被使用不同的粒度排序，所以在这些字段上 `date` 总是在`date_nanos`之前以升序排序。通过`numeric_type` 类型选项使得使用同一种粒度镜像解析排序成为了可能，设置`date`类型会将date_nanos`以毫秒级粒度解析，设置为`date_nanos`将会以纳秒级粒度解析：

```json
POST /index_long,index_double/_search
{
   "sort" : [
      {
        "field" : {
            "numeric_type" : "date_nanos"
        }
      }
   ]
}
```

To avoid overflow, the conversion to `date_nanos` cannot be applied on dates before 1970 and after 2262 as nanoseconds are represented as longs.

为了避免越界，向`date_nanos`转换不能用1970年前或2262年后的纳秒来表示。

##  Sorting within nested objects.

Elasticsearch also supports sorting by fields that are inside one or more nested objects. The sorting by nested field support has a `nested` sort option with the following properties:

ES也支持对一个或多个嵌套对象排序。排序嵌套字段支持一个 `nested` 排序选项支持如下参数：

- **`path`**

  Defines on which nested object to sort. The actual sort field must be a direct field inside this nested object. When sorting by nested field, this field is mandatory.

  定义在哪个嵌套对象上排序。实际排序字段必须是此嵌套对象内的一个直接字段。当使用嵌套对象字段排序，此字段是必填的。

- **`filter`**

  A filter that the inner objects inside the nested path should match with in order for its field values to be taken into account by sorting. Common case is to repeat the query / filter inside the nested filter or query. By default no `filter` is active.

  一个嵌套路径的内部对象与其匹配的过滤器，以便于控制哪些字段需要被计入排序。内部对象的 query / filter 和普通的查询/过滤是一样的。默认没有过滤器活动。

- **`max_children`**

  The maximum number of children to consider per root document when picking the sort value. Defaults to unlimited.

  限制每个根文档在排序的时候需要考虑的最大子文档数量。默认是无限制的。

- **`nested`**

  Same as top-level `nested` but applies to another nested path within the current nested object.

  与顶级 `nested` 相同，当前嵌套对象中可以继续使用嵌套路径。

Elasticsearch will throw an error if a nested field is defined in a sort without a `nested` context.

注意：如果不在嵌套的上下文环境中定义嵌套字段ES将会抛出错误。

## Nested sorting examples

In the below example `offer` is a field of type `nested`. The nested `path` needs to be specified; otherwise, Elasticsearch doesn’t know on what nested level sort values need to be captured.

下面的例子中`offer` 字段是 `nested`类型的。需要指定嵌套路径；否则，ES不知道嵌套多少层的排序值需要被捕获。

```json
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

In the below example `parent` and `child` fields are of type `nested`. The `nested.path` needs to be specified at each level; otherwise, Elasticsearch doesn’t know on what nested level sort values need to be captured.

下面这个例子中`parent`和child字段是 `nested`类型的。 `nested.path`嵌套路径需要被明确指定到每一个层级；否则，ES是不知道排序值需要捕获到哪一个层级。

```json
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

Nested sorting is also supported when sorting by scripts and sorting by geo distance.

嵌套排序也支持使用脚本和地理距离排序。

##  Missing Values

The `missing` parameter specifies how docs which are missing the sort field should be treated: The `missing` value can be set to `_last`, `_first`, or a custom value (that will be used for missing docs as the sort value). The default is `_last`.

 `missing`参数指定在文档排序字段缺失时应该怎么处理： `missing` 值可以被设置为 `_last`, `_first`或者一个自定义的值（这将会被用在缺失文档作为排序值）。默认设置是 `_last`。

For example:

```json
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

If a nested inner object doesn’t match with the `nested.filter` then a missing value is used.

如果一个嵌套内部对象不被`nested.filter`匹配那么缺失值将会生效。

##  Ignoring Unmapped Fields

By default, the search request will fail if there is no mapping associated with a field. The `unmapped_type` option allows you to ignore fields that have no mapping and not sort by them. The value of this parameter is used to determine what sort values to emit. Here is an example of how it can be used:

默认，在字段没有关联映射时检索请求将会失败。`unmapped_type`选项允许忽视那些没有映射的字段且不对他们进行排序。这个参数的值被用于确定使用什么样的排序值。下面是一个如何使用的例子。

```json
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

If any of the indices that are queried doesn’t have a mapping for `price` then Elasticsearch will handle it as if there was a mapping of type `long`, with all documents in this index having no value for this field.

如果检索请求中的索引都没有对 `price` 字段的映射那么ES将会把这个索引中这个字段缺失值的全部文档的这个字段类型作为`long`来处理。

##  Geo Distance Sorting

Allow to sort by `_geo_distance`. Here is an example, assuming `pin.location` is a field of type `geo_point`:

允许通过`_geo_distance`地理距离排序。这是一个例子，假如`pin.location` 是一个 `geo_point`类型字段：

```json
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

**`distance_type`** 

How to compute the distance. Can either be `arc` (default), or `plane` (faster, but inaccurate on long distances and close to the poles).

如何计算距离。可以通过 `arc` （默认）也可以通过 `plane` （更快，但在长距离和极点附近不准确）

**`mode`**

What to do in case a field has several geo points. By default, the shortest distance is taken into account when sorting in ascending order and the longest distance when sorting in descending order. Supported values are `min`, `max`, `median` and `avg`.

如果一个字段有多个地理位置怎么办。默认情况下，按升序排序时会考虑最短距离，而按降序排序时会考虑最长距离。支持的值有 `min`, `max`, `median`和 `avg`。

**`unit`**

The unit to use when computing sort values. The default is `m` (meters).

单位被用于计算排序值。默认是`m`(米)

**`ignore_unmapped`**

Indicates if the unmapped field should be treated as a missing value. Setting it to `true` is equivalent to specifying an `unmapped_type` in the field sort. The default is `false` (unmapped field cause the search to fail).

索引中如果有未映射字段需要被处理为丢失值。把**`ignore_unmapped`**设置为 `true` 等同于在排序字段中指定 `unmapped_type` 。默认是`false`(为映射的字段可能会导致检索失败)。

geo distance sorting does not support configurable missing values: the distance will always be considered equal to `Infinity` when a document does not have values for the field that is used for distance computation.

地理距离排序不支持配置丢失值：当文档被用于计算距离的字段没有值时，距离应当被考虑成 `Infinity` （无限远的点）。

The following formats are supported in providing the coordinates:

提供坐标时支持以下格式：

- Lat Lon as Properties

  ```json
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

- Lat Lon as String

  Format in `lat,lon`.

  ```json
  GET /_search
  {
    "sort": [
      {
        "_geo_distance": {
          "pin.location": "40,-70",
          "order": "asc",
          "unit": "km"
        }
      }
    ],
    "query": {
      "term": { "user": "kimchy" }
    }
  }
  ```

- Geohash

  ```json
  GET /_search
  {
    "sort": [
      {
        "_geo_distance": {
          "pin.location": "drm3btev3e86",
          "order": "asc",
          "unit": "km"
        }
      }
    ],
    "query": {
      "term": { "user": "kimchy" }
    }
  }
  ```

- Lat Lon as Array

  Format in `[lon, lat]`, note, the order of lon/lat here in order to conform with [GeoJSON](http://geojson.org/).

  请在`[lon, lat]`此处格式化lon / lat的顺序，以符合[GeoJSON](http://geojson.org/)。

  ```json
  GET /_search
  {
    "sort": [
      {
        "_geo_distance": {
          "pin.location": [ -70, 40 ],
          "order": "asc",
          "unit": "km"
        }
      }
    ],
    "query": {
      "term": { "user": "kimchy" }
    }
  }
  ```

- Multiple reference points

  Multiple geo points can be passed as an array containing any `geo_point` format, for example

  多个地理位置可以作为包含 `geo_point` 格式作为数组传递，例如

  ```json
  GET /_search
  {
    "sort": [
      {
        "_geo_distance": {
          "pin.location": [ [ -70, 40 ], [ -71, 42 ] ],
          "order": "asc",
          "unit": "km"
        }
      }
    ],
    "query": {
      "term": { "user": "kimchy" }
    }
  }
  ```

  and so forth.

  等等。

  The final distance for a document will then be `min`/`max`/`avg` (defined via `mode`) distance of all points contained in the document to all points given in the sort request.

  最终一个文档的最终距离是根据`min`/ `max`/ `avg`（通过定义`mode`）包含在文档中的排序请求中给出的所有点中的所有点的距离来决定的。

- Script Based Sorting

  Allow to sort based on custom scripts, here is an example:

  支持基于自定义脚本排序，例如：

  ```json
  GET /_search
  {
    "query": {
      "term": { "user": "kimchy" }
    },
    "sort": {
      "_script": {
        "type": "number",
        "script": {
          "lang": "painless",
          "source": "doc['field_name'].value * params.factor",
          "params": {
            "factor": 1.1
          }
        },
        "order": "asc"
      }
    }
  }
  ```

- Track Scores

  When sorting on a field, scores are not computed. By setting `track_scores` to true, scores will still be computed and tracked.

  在字段上排序时，不计算分数。通过设置 `track_scores`为true，将会计算和跟踪分数。

  ```json
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

- Memory Considerations

  When sorting, the relevant sorted field values are loaded into memory. This means that per shard, there should be enough memory to contain them. For string based types, the field sorted on should not be analyzed / tokenized. For numeric types, if possible, it is recommended to explicitly set the type to narrower types (like `short`, `integer` and `float`).

  在排序时，排序相关字段值将会被加载到内存中。这意味着每一个分片需要足够的内存来包含他们。基于string类型，排序时不应该被分析/符号化。基于numeric类型，如果有可能，推荐明确指定最小的类型（像 `short`, `integer` 和 `float`）。