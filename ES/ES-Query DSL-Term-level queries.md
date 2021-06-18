#  Term-level queries

You can use **term-level queries** to find documents based on precise values in structured data. Examples of structured data include date ranges, IP addresses, prices, or product IDs.

您可以使用**术语级查询**根据结构化数据中的精确值查找文档。结构化数据的示例包括日期范围、IP 地址、价格或产品 ID。

Unlike [full-text queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/full-text-queries.html), term-level queries do not analyze search terms. Instead, term-level queries match the exact terms stored in a field.

与[全文查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/full-text-queries.html)不同，词级查询不分析搜索词。相反，术语级查询与存储在字段中的确切术语相匹配。

> NOTE: Term-level queries still normalize search terms for `keyword` fields with the `normalizer` property. For more details, see [`normalizer`](https://www.elastic.co/guide/en/elasticsearch/reference/master/normalizer.html).
>
> 术语级别的查询仍然对`keyword`具有该`normalizer`属性的字段的 搜索术语进行规范化。有关更多详细信息，请参阅[`normalizer`](https://www.elastic.co/guide/en/elasticsearch/reference/master/normalizer.html)。

##  Types of term-level queries

**[`exists` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-exists-query.html)**

Returns documents that contain any indexed value for a field.

返回包含字段的任何索引值的文档。

**[`fuzzy` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-fuzzy-query.html)**

Returns documents that contain terms similar to the search term. Elasticsearch measures similarity, or fuzziness, using a [Levenshtein edit distance](https://en.wikipedia.org/wiki/Levenshtein_distance).

返回包含与搜索词相似的词的文档。Elasticsearch 使用[Levenshtein 编辑距离](https://en.wikipedia.org/wiki/Levenshtein_distance)来测量相似性或模糊性 。

**[`ids` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-ids-query.html)**

Returns documents based on their [document IDs](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html).

根据[文档 ID](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html)返回文档。

**[`prefix` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-prefix-query.html)**

Returns documents that contain a specific prefix in a provided field.

返回在提供的字段中包含特定前缀的文档。

**[`range` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html)**

Returns documents that contain terms within a provided range.

返回包含提供范围内的术语的文档。

**[`regexp` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-regexp-query.html)**

Returns documents that contain terms matching a [regular expression](https://en.wikipedia.org/wiki/Regular_expression).

返回包含与[正则表达式](https://en.wikipedia.org/wiki/Regular_expression)匹配的术语的文档 。

**[`term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)**

Returns documents that contain an exact term in a provided field.

返回在提供的字段中包含确切术语的文档。

**[`terms` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-query.html)**

Returns documents that contain one or more exact terms in a provided field.

返回在提供的字段中包含一个或多个确切术语的文档。

**[`terms_set` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-set-query.html)**

Returns documents that contain a minimum number of exact terms in a provided field. You can define the minimum number of matching terms using a field or script.

返回在提供的字段中包含最少数量的精确术语的文档。您可以使用字段或脚本定义匹配术语的最小数量。

**[`wildcard` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-wildcard-query.html)**

Returns documents that contain terms matching a wildcard pattern.

返回包含匹配通配符模式的术语的文档。

#  Exists query

Returns documents that contain an indexed value for a field.

返回包含字段索引值的文档。

An indexed value may not exist for a document’s field due to a variety of reasons:

由于多种原因，文档字段可能不存在索引值：

- The field in the source JSON is `null` or `[]`

  源 JSON 中的字段是`null`或`[]`

- The field has `"index" : false` set in the mapping

  该字段已`"index" : false`在映射中设置

- The length of the field value exceeded an `ignore_above` setting in the mapping

  字段值的长度超过`ignore_above`映射中的设置

- The field value was malformed and `ignore_malformed` was defined in the mapping

  字段值格式错误并`ignore_malformed`在映射中定义

##  Example request

```console
GET /_search
{
  "query": {
    "exists": {
      "field": "user"
    }
  }
}
```

### Top-level parameters for `exists`

**`field`**

(Required, string) Name of the field you wish to search.

（必填，字符串）您要搜索的字段的名称。

While a field is deemed non-existent if the JSON value is `null` or `[]`, these values will indicate the field does exist:

如果 JSON 值为`null`或`[]`，则字段被视为不存在，但这些值将表明该字段确实存在：

- Empty strings, such as `""` or `"-"`

  空字符串，例如`""`或`"-"`

- Arrays containing `null` and another value, such as `[null, "foo"]`

  包含`null`和另一个值的数组，例如`[null, "foo"]`

- A custom [`null-value`](https://www.elastic.co/guide/en/elasticsearch/reference/master/null-value.html), defined in field mapping

  自定义[`null-value`](https://www.elastic.co/guide/en/elasticsearch/reference/master/null-value.html)，在字段映射中定义

###  Notes

####  Find documents missing indexed values

To find documents that are missing an indexed value for a field, use the `must_not` [boolean query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html) with the `exists` query.

要查找缺少字段索引值的文档，请在 查询中使用`must_not` [布尔](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html)`exists`查询。

The following search returns documents that are missing an indexed value for the `user.id` field.

以下搜索返回缺少`user.id`字段索引值的文档。

```console
GET /_search
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "user.id"
        }
      }
    }
  }
}
```

# Fuzzy query

Returns documents that contain terms similar to the search term, as measured by a [Levenshtein edit distance](https://en.wikipedia.org/wiki/Levenshtein_distance).

返回包含与搜索词相似的词的文档，由[Levenshtein 编辑距离](https://en.wikipedia.org/wiki/Levenshtein_distance)测量。

An edit distance is the number of one-character changes needed to turn one term into another. These changes can include:

编辑距离是将一个术语转换为另一个术语所需的一个字符更改的数量。这些变化可能包括：

- Changing a character (**b**ox → **f**ox)
- Removing a character (**b**lack → lack)
- Inserting a character (sic → sic**k**)
- Transposing two adjacent characters (**ac**t → **ca**t)

To find similar terms, the `fuzzy` query creates a set of all possible variations, or expansions, of the search term within a specified edit distance. The query then returns exact matches for each expansion.

为了查找相似的术语，`fuzzy`查询会在指定的编辑距离内创建一组搜索术语的所有可能变体或扩展。然后查询返回每个扩展的精确匹配。

###  Example requests

####  Simple example

```console
GET /_search
{
  "query": {
    "fuzzy": {
      "user.id": {
        "value": "ki"
      }
    }
  }
}
```

####  Example using advanced parameters

```console
GET /_search
{
  "query": {
    "fuzzy": {
      "user.id": {
        "value": "ki",
        "fuzziness": "AUTO",
        "max_expansions": 50,
        "prefix_length": 0,
        "transpositions": true,
        "rewrite": "constant_score"
      }
    }
  }
}
```

###  Top-level parameters for `fuzzy`

**`<field>`**

(Required, object) Field you wish to search.

（必需，对象）要搜索的字段。

###  Parameters for `<field>`

**`value`**

(Required, string) Term you wish to find in the provided `<field>`.

（必需，字符串）您希望在提供的`<field>`.

**`fuzziness`**

(Optional, string) Maximum edit distance allowed for matching. See [Fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness) for valid values and more information.

（可选，字符串）允许匹配的最大编辑距离。有关 有效值和更多信息，请参阅[模糊度](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#fuzziness)。

**`max_expansions`**

(Optional, integer) Maximum number of variations created. Defaults to `50`.

（可选，整数）创建的最大变体数。默认为`50`.

> WARNING: Avoid using a high value in the `max_expansions` parameter, especially if the `prefix_length` parameter value is `0`. High values in the `max_expansions` parameter can cause poor performance due to the high number of variations examined.
>
> 避免在`max_expansions`参数中使用高值，尤其是当`prefix_length`参数值为 时`0`。`max_expansions`由于检查了大量的变化，参数中的高值 会导致性能不佳。

**`prefix_length`**

(Optional, integer) Number of beginning characters left unchanged when creating expansions. Defaults to `0`.

（可选，整数）创建扩展时保持不变的起始字符数。默认为`0`.

**`transpositions`**

(Optional, Boolean) Indicates whether edits include transpositions of two adjacent characters (ab → ba). Defaults to `true`.

（可选，布尔值）指示编辑是否包括两个相邻字符的换位（ab → ba）。默认为`true`.

**`rewrite`**

(Optional, string) Method used to rewrite the query. For valid values and more information, see the [`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html).

（可选，字符串）用于重写查询的方法。有关有效值和更多信息，请参阅[`rewrite`参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html)。

###  Notes

Fuzzy queries will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) 设置为 false，则不会执行模糊查询。

# IDs

Returns documents based on their IDs. This query uses document IDs stored in the [`_id`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html) field.

根据文档的 ID 返回文档。此查询使用存储在[`_id`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html)字段中的文档 ID 。

###  Example request

```console
GET /_search
{
  "query": {
    "ids" : {
      "values" : ["1", "4", "100"]
    }
  }
}
```

###  Top-level parameters for `ids`

**`values`**

(Required, array of strings) An array of [document IDs](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html).

（必需，字符串数组）[文档 ID](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html)数组。

#  Prefix query

Returns documents that contain a specific prefix in a provided field.

返回在提供的字段中包含特定前缀的文档。

###  Example request

The following search returns documents where the `user.id` field contains a term that begins with `ki`.

以下搜索返回`user.id`字段包含以 开头的术语的文档`ki`。

```console
GET /_search
{
  "query": {
    "prefix": {
      "user.id": {
        "value": "ki"
      }
    }
  }
}
```

###  Top-level parameters for `prefix`

**`<field>`**

(Required, object) Field you wish to search.

（必需，对象）要搜索的字段。

###  Parameters for `<field>`

**`value`**

(Required, string) Beginning characters of terms you wish to find in the provided `<field>`.

（必需，字符串）您希望在提供的`<field>`.

**`rewrite`**

(Optional, string) Method used to rewrite the query. For valid values and more information, see the [`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html).

（可选，字符串）用于重写查询的方法。有关有效值和更多信息，请参阅[`rewrite`参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html)。

**`case_insensitive` \**[\*\*7.10.0\*\*]\*\*Added in 7.10.0.\*\**\***

(Optional, Boolean) Allows ASCII case insensitive matching of the value with the indexed field values when set to true. Default is false which means the case sensitivity of matching depends on the underlying field’s mapping.

（可选，布尔值）当设置为 true 时，允许值与索引字段值的 ASCII 不区分大小写匹配。默认为 false，这意味着匹配的区分大小写取决于基础字段的映射。

###  Notes

####  Short request example

You can simplify the `prefix` query syntax by combining the `<field>` and `value` parameters. For example:

您可以`prefix`通过组合`<field>`和 `value`参数来简化查询语法。例如：

```console
GET /_search
{
  "query": {
    "prefix" : { "user" : "ki" }
  }
}
```

####  Speed up prefix queries

You can speed up prefix queries using the [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html) mapping parameter. If enabled, Elasticsearch indexes prefixes between 2 and 5 characters in a separate field. This lets Elasticsearch run prefix queries more efficiently at the cost of a larger index.

您可以使用[`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html) mapping 参数加速前缀查询。如果启用，Elasticsearch 会在单独的字段中索引 2 到 5 个字符之间的前缀。这让 Elasticsearch 以更大的索引为代价更有效地运行前缀查询。

#### 允许昂贵的查询

####  Allow expensive queries

Prefix queries will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false. However, if [`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html) are enabled, an optimised query is built which is not considered slow, and will be executed in spite of this setting.

如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) 设置为 false，则不会执行前缀查询。但是，如果[`index_prefixes`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-prefixes.html)启用，则会构建一个优化的查询，该查询不会被认为是缓慢的，并且尽管有此设置也会执行。

#  Range query

Returns documents that contain terms within a provided range.

返回包含提供范围内的术语的文档。

###  Example request

The following search returns documents where the `age` field contains a term between `10` and `20`.

以下搜索返回`age`字段包含介于`10`和之间的术语的文档`20`。

```console
GET /_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20,
        "boost": 2.0
      }
    }
  }
}
```

###  Top-level parameters for `range`

**`<field>`**

(Required, object) Field you wish to search.

（必需，对象）要搜索的字段。

###  Parameters for `<field>`

**`gt`**

(Optional) Greater than.

（可选）大于。

**`gte`**

(Optional) Greater than or equal to.

（可选）大于或等于。

**`lt`**

(Optional) Less than.

（可选）小于。

**`lte`**

(Optional) Less than or equal to.

（可选）小于或等于。

**`format`**

(Optional, string) Date format used to convert `date` values in the query.

（可选，字符串）用于转换`date`查询中值的日期格式。

By default, Elasticsearch uses the [date `format`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html) provided in the `<field>`'s mapping. This value overrides that mapping format.

默认情况下，Elasticsearch 使用的映射中提供 的[日期`format`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)`<field>`。此值会覆盖该映射格式。

For valid syntax, see [`format`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html).

有关有效语法，请参阅[`format`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-date-format.html)。

> WARNING: If a format or date value is incomplete, the range query replaces any missing components with default values. See [Missing date components](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html#missing-date-components).
>
> 如果格式或日期值不完整，范围查询会用默认值替换任何缺失的组件。请参阅[缺少日期组件](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html#missing-date-components)。

**`relation`**

(Optional, string) Indicates how the range query matches values for `range` fields. Valid values are:

（可选，字符串）指示范围查询如何匹配`range` 字段的值。有效值为：

- **`INTERSECTS` (Default)**

  Matches documents with a range field value that intersects the query’s range.

  匹配具有与查询范围相交的范围字段值的文档。

- **`CONTAINS`**

  Matches documents with a range field value that entirely contains the query’s range.

  匹配具有完全包含查询范围的范围字段值的文档。

- **`WITHIN`**

  Matches documents with a range field value entirely within the query’s range.

  匹配具有完全在查询范围内的范围字段值的文档。

**`time_zone`**

(Optional, string) [Coordinated Universal Time (UTC) offset](https://en.wikipedia.org/wiki/List_of_UTC_time_offsets) or [IANA time zone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) used to convert `date` values in the query to UTC.

（可选，字符串） 用于将查询中的值转换为 UTC 的[协调世界时 (UTC) 偏移量](https://en.wikipedia.org/wiki/List_of_UTC_time_offsets)或 [IANA 时区](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)`date`。

Valid values are ISO 8601 UTC offsets, such as `+01:00` or -`08:00`, and IANA time zone IDs, such as `America/Los_Angeles`.

有效值为 ISO 8601 UTC 偏移量，例如`+01:00`or -`08:00`和 IANA 时区 ID，例如`America/Los_Angeles`。

For an example query using the `time_zone` parameter, see [Time zone in `range` queries](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html#range-query-time-zone).

对于使用一个例子查询`time_zone`参数，看 [在时区`range`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-range-query.html#range-query-time-zone)。

> NOTE: 
> The `time_zone` parameter does **not** affect the [date math](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math) value of `now`. `now` is always the current system time in UTC.
>
> However, the `time_zone` parameter does convert dates calculated using `now` and [date math rounding](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math). For example, the `time_zone` parameter will convert a value of `now/d`.
>
> 
> 该`time_zone`参数不**不**影响[日期数学](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)的价值`now`。`now`始终是 UTC 中的当前系统时间。
>
> 但是，该`time_zone`参数会转换使用`now`和 [日期数学四舍五入](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)计算的[日期](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)。例如，该`time_zone`参数将转换为 的值`now/d`。

**`boost`**

(Optional, float) Floating point number used to decrease or increase the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of a query. Defaults to `1.0`.

（可选，浮点数）用于减少或增加查询[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)的浮点数 。默认为`1.0`.

You can use the `boost` parameter to adjust relevance scores for searches containing two or more queries.

您可以使用该`boost`参数来调整包含两个或更多查询的搜索的相关性分数。

Boost values are relative to the default value of `1.0`. A boost value between `0` and `1.0` decreases the relevance score. A value greater than `1.0` increases the relevance score.

Boost 值相对于 的默认值`1.0`。`0`和之间的提升值会 `1.0`降低相关性分数。大于 的值会`1.0` 增加相关性分数。

###  Notes

####  Using the `range` query with `text` and `keyword` fields

Range queries on [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) or [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html) fields will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

如果设置为 false，则不会执行对[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html)或[`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html)字段的 范围查询[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries)。

####  Using the `range` query with `date` fields

When the `<field>` parameter is a [`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html) field data type, you can use [date math](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math) with the following parameters:

当`<field>`参数是[`date`](https://www.elastic.co/guide/en/elasticsearch/reference/master/date.html)字段数据类型时，您可以使用 带有以下参数的[日期数学](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)：

- `gt`
- `gte`
- `lt`
- `lte`

For example, the following search returns documents where the `timestamp` field contains a date between today and yesterday.

例如，以下搜索返回`timestamp`字段包含今天和昨天之间的日期的文档。

```console
GET /_search
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-1d/d",
        "lt": "now/d"
      }
    }
  }
}
```

#####  Missing date components

For range queries and [date range](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html) aggregations, Elasticsearch replaces missing date components with the following values. Missing year components are not replaced.

对于范围查询和[日期范围](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket-daterange-aggregation.html)聚合，Elasticsearch 使用以下值替换缺失的日期组件。缺少年份的组件不会被替换。

```text
MONTH_OF_YEAR:    01
DAY_OF_MONTH:     01
HOUR_OF_DAY:      23
MINUTE_OF_HOUR:   59
SECOND_OF_MINUTE: 59
NANO_OF_SECOND:   999_999_999
```

For example, if the format is `yyyy-MM`, Elasticsearch converts a `gt` value of `2099-12` to `2099-12-01T23:59:59.999_999_999Z`. This date uses the provided year (`2099`) and month (`12`) but uses the default day (`01`), hour (`23`), minute (`59`), second (`59`), and nanosecond (`999_999_999`).

例如，如果格式为`yyyy-MM`，Elasticsearch 会将`gt`值转换`2099-12` 为`2099-12-01T23:59:59.999_999_999Z`。此日期使用提供的年 ( `2099`) 和月 ( `12`)，但使用默认的日 ( `01`)、小时 ( `23`)、分钟 ( `59`)、秒 ( `59`) 和纳秒 ( `999_999_999`)。

##### Numeric date range value

When no date format is specified and the range query is targeting a date field, numeric values are interpreted representing milliseconds-since-the-epoch. If you want the value to represent a year, e.g. 2020, you need to pass it as a String value (e.g. "2020") that will be parsed according to the default format or the set format.

当未指定日期格式并且范围查询针对日期字段时，数值被解释为表示自纪元以来的毫秒数。如果您希望该值代表一年，例如 2020，则需要将其作为 String 值（例如“2020”）传递，该值将根据默认格式或设置格式进行解析。

#####  Date math and rounding

Elasticsearch rounds [date math](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math) values in parameters as follows:

Elasticsearch对参数中的[日期数学](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#date-math)值进行四舍五入，如下所示：

- **`gt`**

  Rounds up to the first millisecond not covered by the rounded date.

  向上舍入到舍入日期未涵盖的第一个毫秒。

  For example, `2014-11-18||/M` rounds up to `2014-12-01T00:00:00.000`, excluding the entire month of November.

  例如，`2014-11-18||/M`四舍五入到`2014-12-01T00:00:00.000`，不包括整个 11 月。

- **`gte`**

  Rounds down to the first millisecond.

  向下舍入到第一个毫秒。

  For example, `2014-11-18||/M` rounds down to `2014-11-01T00:00:00.000`, including the entire month.

  例如，`2014-11-18||/M`向下舍入为`2014-11-01T00:00:00.000`，包括整个月。

- **`lt`**

  Rounds down to the last millisecond before the rounded value.

  向下舍入到舍入值之前的最后一毫秒。

  For example, `2014-11-18||/M` rounds down to `2014-10-31T23:59:59.999`, excluding the entire month of November.

  例如，`2014-11-18||/M`向下`2014-10-31T23:59:59.999`舍入为，不包括整个 11 月。

- **`lte`**

  Rounds up to the latest millisecond in the rounding interval.

  向上舍入到舍入间隔中的最新毫秒。

  For example, `2014-11-18||/M` rounds up to `2014-11-30T23:59:59.999`, including the entire month.

  例如，`2014-11-18||/M`向上舍入到`2014-11-30T23:59:59.999`，包括整个月。

####  Example query using `time_zone` parameter

You can use the `time_zone` parameter to convert `date` values to UTC using a UTC offset. For example:

您可以使用该`time_zone`参数`date`使用 UTC 偏移量将值转换为 UTC。例如：

```console
GET /_search
{
  "query": {
    "range": {
      "timestamp": {
        "time_zone": "+01:00",        (1)
        "gte": "2020-01-01T00:00:00", (2)
        "lte": "now"                  (3)
      }
    }
  }
}
```

1. Indicates that `date` values use a UTC offset of `+01:00`.

   表示`date`值使用 UTC 偏移量`+01:00`。

2. With a UTC offset of `+01:00`, Elasticsearch converts this date to `2019-12-31T23:00:00 UTC`.

    使用 UTC 偏移量时`+01:00`，Elasticsearch 将此日期转换为 `2019-12-31T23:00:00 UTC`.

3. The `time_zone` parameter does not affect the `now` value.

    该`time_zone`参数不影响该`now`值。

#  Regexp query

Returns documents that contain terms matching a [regular expression](https://en.wikipedia.org/wiki/Regular_expression).

返回包含与[正则表达式](https://en.wikipedia.org/wiki/Regular_expression)匹配的术语的文档 。

A regular expression is a way to match patterns in data using placeholder characters, called operators. For a list of operators supported by the `regexp` query, see [Regular expression syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html).

正则表达式是一种使用占位符字符（称为运算符）匹配数据模式的方法。有关`regexp`查询支持的运算符列表 ，请参阅[正则表达式语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html)。

### Example request

The following search returns documents where the `user.id` field contains any term that begins with `k` and ends with `y`. The `.*` operators match any characters of any length, including no characters. Matching terms can include `ky`, `kay`, and `kimchy`.

以下搜索返回`user.id`字段包含以 开头`k`和结尾的任何术语的文档`y`。的`.*`运营商匹配任何长度的任何字符，包括无字符。匹配条件可以包括`ky`，`kay`，和`kimchy`。

```console
GET /_search
{
  "query": {
    "regexp": {
      "user.id": {
        "value": "k.*y",
        "flags": "ALL",
        "case_insensitive": true,
        "max_determinized_states": 10000,
        "rewrite": "constant_score"
      }
    }
  }
}
```

###  Top-level parameters for `regexp`

**`<field>`**

(Required, object) Field you wish to search.

（必需，对象）要搜索的字段。

###  Parameters for `<field>`

**`value`**

(Required, string) Regular expression for terms you wish to find in the provided `<field>`. For a list of supported operators, see [Regular expression syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html).

（必需，字符串）您希望在提供的 `<field>`. 有关支持的运算符列表，请参阅[正则表达式语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html)。

By default, regular expressions are limited to 1,000 characters. You can change this limit using the [`index.max_regex_length`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-regex-length) setting.

默认情况下，正则表达式限制为 1,000 个字符。您可以使用[`index.max_regex_length`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-regex-length) 设置更改此限制。

> WARNING: The performance of the `regexp` query can vary based on the regular expression provided. To improve performance, avoid using wildcard patterns, such as `.*` or `.*?+`, without a prefix or suffix.
>
> `regexp`查询的性能可能因提供的正则表达式而异。要提高性能，请避免使用通配符模式，例如`.*`或 `.*?+`，而没有前缀或后缀。

**`flags`**

(Optional, string) Enables optional operators for the regular expression. For valid values and more information, see [Regular expression syntax](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html#regexp-optional-operators).

(Optional, string) 为正则表达式启用可选运算符。有关有效值和更多信息，请参阅[正则表达式语法](https://www.elastic.co/guide/en/elasticsearch/reference/master/regexp-syntax.html#regexp-optional-operators)。

**`case_insensitive` \**[\*\*7.10.0\*\*]\*\*Added in 7.10.0.\*\**\***

(Optional, Boolean) Allows case insensitive matching of the regular expression value with the indexed field values when set to true. Default is false which means the case sensitivity of matching depends on the underlying field’s mapping.

（可选，布尔值）当设置为 true 时，允许将正则表达式值与索引字段值进行不区分大小写的匹配。默认为 false，这意味着匹配的区分大小写取决于基础字段的映射。

**`max_determinized_states`**

(Optional, integer) Maximum number of [automaton states](https://en.wikipedia.org/wiki/Deterministic_finite_automaton) required for the query. Default is `10000`.

（可选，整数） 查询所需的最大[自动机状态](https://en.wikipedia.org/wiki/Deterministic_finite_automaton)数 。默认为`10000`。

Elasticsearch uses [Apache Lucene](https://lucene.apache.org/core/) internally to parse regular expressions. Lucene converts each regular expression to a finite automaton containing a number of determinized states.

Elasticsearch 在内部使用[Apache Lucene](https://lucene.apache.org/core/)来解析正则表达式。Lucene 将每个正则表达式转换为包含多个确定状态的有限自动机。

You can use this parameter to prevent that conversion from unintentionally consuming too many resources. You may need to increase this limit to run complex regular expressions.

您可以使用此参数来防止该转换无意中消耗过多资源。您可能需要增加此限制才能运行复杂的正则表达式。

**`rewrite`**

(Optional, string) Method used to rewrite the query. For valid values and more information, see the [`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html).

（可选，字符串）用于重写查询的方法。有关有效值和更多信息，请参阅[`rewrite`参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html)。

###  Notes

####  Allow expensive queries

Regexp queries will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) 设置为 false，则不会执行 Regexp 查询。

#  Term query

Returns documents that contain an **exact** term in a provided field.

返回在提供的字段中包含**确切**术语的文档。

You can use the `term` query to find documents based on a precise value such as a price, a product ID, or a username.

您可以使用`term`查询根据价格、产品 ID 或用户名等精确值来查找文档。

> WARNING: 
> Avoid using the `term` query for [`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html) fields.
>
> 避免`term`对[`text`](https://www.elastic.co/guide/en/elasticsearch/reference/master/text.html)字段使用查询。
>
> By default, Elasticsearch changes the values of `text` fields as part of [analysis](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html). This can make finding exact matches for `text` field values difficult.
>
> 默认情况下，`text`作为[analysis 的](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis.html)一部分，Elasticsearch 会更改字段的值。这会使查找`text`字段值的精确匹配变得困难。
>
> To search `text` field values, use the [`match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html) query instead.
>
> 要搜索`text`字段值，请改用[`match`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-match-query.html)查询。

###  Example request

```console
GET /_search
{
  "query": {
    "term": {
      "user.id": {
        "value": "kimchy",
        "boost": 1.0
      }
    }
  }
}
```

###  Top-level parameters for `term`

**`<field>`**

(Required, object) Field you wish to search.

（必需，对象）要搜索的字段。

###  Parameters for `<field>`

**`value`**

(Required, string) Term you wish to find in the provided `<field>`. To return a document, the term must exactly match the field value, including whitespace and capitalization.

（必需，字符串）您希望在提供的`<field>`. 要返回文档，该术语必须与字段值完全匹配，包括空格和大写。

**`boost`**

(Optional, float) Floating point number used to decrease or increase the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of a query. Defaults to `1.0`.

（可选，浮点数）用于减少或增加查询[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)的浮点数 。默认为`1.0`.

You can use the `boost` parameter to adjust relevance scores for searches containing two or more queries.

您可以使用该`boost`参数来调整包含两个或更多查询的搜索的相关性分数。

Boost values are relative to the default value of `1.0`. A boost value between `0` and `1.0` decreases the relevance score. A value greater than `1.0` increases the relevance score.

Boost 值相对于 的默认值`1.0`。`0`和之间的提升值会 `1.0`降低相关性分数。大于 的值会`1.0` 增加相关性分数。

**`case_insensitive` \**[\*\*7.10.0\*\*]\*\*Added in 7.10.0.\*\**\***

(Optional, Boolean) Allows ASCII case insensitive matching of the value with the indexed field values when set to true. Default is false which means the case sensitivity of matching depends on the underlying field’s mapping.

（可选，布尔值）当设置为 true 时，允许值与索引字段值的 ASCII 不区分大小写匹配。默认为 false，这意味着匹配的区分大小写取决于基础字段的映射。

###  Notes

####  Avoid using the `term` query for `text` fields

By default, Elasticsearch changes the values of `text` fields during analysis. For example, the default [standard analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-standard-analyzer.html) changes `text` field values as follows:

默认情况下，Elasticsearch`text`在分析期间更改字段的值。例如，默认的[标准分析器](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-standard-analyzer.html)更改 `text`字段值如下：

- Removes most punctuation

  删除大部分标点符号

- Divides the remaining content into individual words, called [tokens](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-tokenizers.html)

  将剩余的内容分成单独的单词，称为 [标记](https://www.elastic.co/guide/en/elasticsearch/reference/master/analysis-tokenizers.html)

- Lowercases the tokens

  小写标记

To better search `text` fields, the `match` query also analyzes your provided search term before performing a search. This means the `match` query can search `text` fields for analyzed tokens rather than an exact term.

为了更好地搜索`text`字段，`match`查询还会在执行搜索之前分析您提供的搜索词。这意味着`match`查询可以搜索 `text`分析标记的字段，而不是精确的术语。

The `term` query does **not** analyze the search term. The `term` query only searches for the **exact** term you provide. This means the `term` query may return poor or no results when searching `text` fields.

该`term`查询并**没有**分析搜索词。该`term`查询仅搜索您提供的**确切**术语。这意味着`term`在搜索`text`字段时查询可能返回较差的结果或没有结果。

To see the difference in search results, try the following example.

要查看搜索结果的差异，请尝试以下示例。

1. Create an index with a `text` field called `full_text`.

   使用`text`名为的字段创建索引`full_text`。

   ```console
   PUT my-index-000001
   {
     "mappings": {
       "properties": {
         "full_text": { "type": "text" }
       }
     }
   }
   ```

2. Index a document with a value of `Quick Brown Foxes!` in the `full_text` field.

   与指数的值的文件`Quick Brown Foxes!`在`full_text` 字段。

   ```console
   PUT my-index-000001/_doc/1
   {
     "full_text":   "Quick Brown Foxes!"
   }
   ```

   Because `full_text` is a `text` field, Elasticsearch changes `Quick Brown Foxes!` to `[quick, brown, fox]` during analysis.

   因为`full_text`是一个`text`场，Elasticsearch改变`Quick Brown Foxes!`对 `[quick, brown, fox]`分析过程中。

3. Use the `term` query to search for `Quick Brown Foxes!` in the `full_text` field. Include the `pretty` parameter so the response is more readable.

   使用`term`查询`Quick Brown Foxes!`在`full_text` 字段中进行搜索。包括`pretty`参数，以便响应更具可读性。

   ```console
   GET my-index-000001/_search?pretty
   {
     "query": {
       "term": {
         "full_text": "Quick Brown Foxes!"
       }
     }
   }
   ```

   Because the `full_text` field no longer contains the **exact** term `Quick Brown Foxes!`, the `term` query search returns no results.

   由于该`full_text`字段不再包含**确切的**term `Quick Brown Foxes!`，`term`查询搜索不会返回任何结果。

4. Use the `match` query to search for `Quick Brown Foxes!` in the `full_text` field.

   使用`match`查询`Quick Brown Foxes!`在`full_text` 字段中进行搜索。

   ```console
   GET my-index-000001/_search?pretty
   {
     "query": {
       "match": {
         "full_text": "Quick Brown Foxes!"
       }
     }
   }
   ```

   Unlike the `term` query, the `match` query analyzes your provided search term, `Quick Brown Foxes!`, before performing a search. The `match` query then returns any documents containing the `quick`, `brown`, or `fox` tokens in the `full_text` field.

   与`term`查询不同，查询会在执行搜索之前`match`分析您提供的搜索词 `Quick Brown Foxes!`。该`match`查询然后返回包含任何文件`quick`，`brown`或`fox`在标记 `full_text`字段中。

   Here’s the response for the `match` query search containing the indexed document in the results.

   下面是`match`对结果中包含索引文档的查询搜索的响应。

   ```console-result
   {
     "took" : 1,
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
       "max_score" : 0.8630463,
       "hits" : [
         {
           "_index" : "my-index-000001",
           "_id" : "1",
           "_score" : 0.8630463,
           "_source" : {
             "full_text" : "Quick Brown Foxes!"
           }
         }
       ]
     }
   }
   ```

#  Terms query

Returns documents that contain one or more **exact** terms in a provided field.

返回在提供的字段中包含一个或多个**确切**术语的文档。

The `terms` query is the same as the [`term` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html), except you can search for multiple values.

该`terms`查询相同的[`term`查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-term-query.html)，除了可以搜索多个值。

###  Example request

The following search returns documents where the `user.id` field contains `kimchy` or `elkbee`.

以下搜索返回`user.id`字段包含`kimchy` or `elkbee`的文档。

```console
GET /_search
{
  "query": {
    "terms": {
      "user.id": [ "kimchy", "elkbee" ],
      "boost": 1.0
    }
  }
}
```

### Top-level parameters for `terms`

**`<field>`**

(Optional, object) Field you wish to search.

（可选，对象）您要搜索的字段。

The value of this parameter is an array of terms you wish to find in the provided field. To return a document, one or more terms must exactly match a field value, including whitespace and capitalization.

此参数的值是您希望在提供的字段中查找的术语数组。要返回文档，一个或多个术语必须与字段值完全匹配，包括空格和大写。

By default, Elasticsearch limits the `terms` query to a maximum of 65,536 terms. You can change this limit using the [`index.max_terms_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-terms-count) setting.

默认情况下，Elasticsearch 将`terms`查询限制为最多 65,536 个术语。您可以使用[`index.max_terms_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-terms-count)设置更改此限制。

> NOTE: To use the field values of an existing document as search terms, use the [terms lookup](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-query.html#query-dsl-terms-lookup) parameters.
>
> 要将现有文档的字段值用作搜索词，请使用 [词条查找](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-query.html#query-dsl-terms-lookup)参数。

**`boost`**

(Optional, float) Floating point number used to decrease or increase the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of a query. Defaults to `1.0`.

（可选，浮点数）用于减少或增加查询[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)的浮点数 。默认为`1.0`.

You can use the `boost` parameter to adjust relevance scores for searches containing two or more queries.

您可以使用该`boost`参数来调整包含两个或更多查询的搜索的相关性分数。

Boost values are relative to the default value of `1.0`. A boost value between `0` and `1.0` decreases the relevance score. A value greater than `1.0` increases the relevance score.

Boost 值相对于 的默认值`1.0`。`0`和之间的提升值会 `1.0`降低相关性分数。大于 的值会`1.0` 增加相关性分数。

###  Notes

####  Highlighting `terms` queries

[Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html) is best-effort only. Elasticsearch may not return highlight results for `terms` queries depending on:

[突出显示](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html)只是尽力而为。Elasticsearch 可能不会返回`terms`查询的突出显示结果，具体取决于：

- Highlighter type

  荧光笔类型

- Number of terms in the query

  查询中的术语数

####  Terms lookup

Terms lookup fetches the field values of an existing document. Elasticsearch then uses those values as search terms. This can be helpful when searching for a large set of terms.

术语查找获取现有文档的字段值。Elasticsearch 然后使用这些值作为搜索词。这在搜索大量术语时会很有帮助。

Because terms lookup fetches values from a document, the [`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-source-field.html) mapping field must be enabled to use terms lookup. The `_source` field is enabled by default.

由于术语查找从文档中获取值，因此[`_source`](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-source-field.html)必须启用映射字段才能使用术语查找。该`_source` 字段默认启用。

> NOTE: By default, Elasticsearch limits the `terms` query to a maximum of 65,536 terms. This includes terms fetched using terms lookup. You can change this limit using the [`index.max_terms_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-terms-count) setting.
>
> 默认情况下，Elasticsearch 将`terms`查询限制为最多 65,536 个术语。这包括使用术语查找获取的术语。您可以使用[`index.max_terms_count`](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules.html#index-max-terms-count)设置更改此限制。

To perform a terms lookup, use the following parameters.

要执行术语查找，请使用以下参数。

##### Terms lookup parameters

**`index`**

(Required, string) Name of the index from which to fetch field values.

（必需，字符串）从中获取字段值的索引的名称。

**`id`**

(Required, string) [ID](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html) of the document from which to fetch field values.

（必需，字符串）从中获取字段值的文档的[ID](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-id-field.html)。

**`path`**

(Required, string) Name of the field from which to fetch field values. Elasticsearch uses these values as search terms for the query.

（必需，字符串）从中获取字段值的字段的名称。Elasticsearch 使用这些值作为查询的搜索词。

If the field values include an array of nested inner objects, you can access those objects using dot notation syntax.

如果字段值包含嵌套内部对象的数组，则可以使用点符号语法访问这些对象。

**`routing`**

(Optional, string) Custom [routing value](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-routing-field.html) of the document from which to fetch term values. If a custom routing value was provided when the document was indexed, this parameter is required.

（可选，字符串）要从中获取术语值的文档的自定义[路由](https://www.elastic.co/guide/en/elasticsearch/reference/master/mapping-routing-field.html)值。如果在索引文档时提供了自定义路由值，则此参数是必需的。

#####  Terms lookup example

To see how terms lookup works, try the following example.

要查看术语查找的工作原理，请尝试以下示例。

1. Create an index with a `keyword` field named `color`.

   使用`keyword`名为的字段创建索引`color`。

   ```console
   PUT my-index-000001
   {
     "mappings": {
       "properties": {
         "color": { "type": "keyword" }
       }
     }
   }
   ```

2. Index a document with an ID of 1 and values of `["blue", "green"]` in the `color` field.

   索引用的1 ID和的值的文档`["blue", "green"]`中的 `color`字段。

   ```console
   PUT my-index-000001/_doc/1
   {
     "color":   ["blue", "green"]
   }
   ```

3. Index another document with an ID of 2 and value of `blue` in the `color` field.

   索引用的2 ID和价值另一个文档`blue`中的`color` 字段。

   ```console
   PUT my-index-000001/_doc/2
   {
     "color":   "blue"
   }
   ```

4. Use the `terms` query with terms lookup parameters to find documents containing one or more of the same terms as document 2. Include the `pretty` parameter so the response is more readable.

   使用`terms`带有词条查找参数的查询来查找包含一个或多个与文档 2 相同的词条的文档。包括`pretty` 参数以便响应更具可读性。

   ```console
   GET my-index-000001/_search?pretty
   {
     "query": {
       "terms": {
           "color" : {
               "index" : "my-index-000001",
               "id" : "2",
               "path" : "color"
           }
       }
     }
   }
   ```

   Because document 2 and document 1 both contain `blue` as a value in the `color` field, Elasticsearch returns both documents.

   因为文档 2 和文档 1`blue`在`color` 字段中都包含一个值，所以Elasticsearch 会返回这两个文档。

   ```console-result
   {
     "took" : 17,
     "timed_out" : false,
     "_shards" : {
       "total" : 1,
       "successful" : 1,
       "skipped" : 0,
       "failed" : 0
     },
     "hits" : {
       "total" : {
         "value" : 2,
         "relation" : "eq"
       },
       "max_score" : 1.0,
       "hits" : [
         {
           "_index" : "my-index-000001",
           "_id" : "1",
           "_score" : 1.0,
           "_source" : {
             "color" : [
               "blue",
               "green"
             ]
           }
         },
         {
           "_index" : "my-index-000001",
           "_id" : "2",
           "_score" : 1.0,
           "_source" : {
             "color" : "blue"
           }
         }
       ]
     }
   }
   ```

#  Terms set query

Returns documents that contain a minimum number of **exact** terms in a provided field.

返回在提供的字段中包含最少数量的**精确**术语的文档。

The `terms_set` query is the same as the [`terms` query](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-query.html), except you can define the number of matching terms required to return a document. For example:

该`terms_set`查询相同的[`terms` 查询](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-query.html)，但您可以定义返回文档所需的匹配项数。例如：

- A field, `programming_languages`, contains a list of known programming languages, such as `c++`, `java`, or `php` for job candidates. You can use the `terms_set` query to return documents that match at least two of these languages.

  字段，`programming_languages`包含已知的编程语言，如列表`c++`，`java`或`php`为求职者。您可以使用 `terms_set`查询来返回至少匹配其中两种语言的文档。

- A field, `permissions`, contains a list of possible user permissions for an application. You can use the `terms_set` query to return documents that match a subset of these permissions.

  字段`permissions`包含应用程序可能的用户权限列表。您可以使用`terms_set`查询返回与这些权限的子集匹配的文档。

###  Example request

####  Index setup

In most cases, you’ll need to include a [numeric](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) field mapping in your index to use the `terms_set` query. This numeric field contains the number of matching terms required to return a document.

在大多数情况下，您需要在索引中包含[数字](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)字段映射才能使用`terms_set`查询。此数字字段包含返回文档所需的匹配项数。

To see how you can set up an index for the `terms_set` query, try the following example.

要了解如何为`terms_set`查询设置索引，请尝试以下示例。

1. Create an index, `job-candidates`, with the following field mappings:

   `job-candidates`使用以下字段映射创建索引：

   - `name`, a [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html) field. This field contains the name of the job candidate.

     `name`，一个[`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html)字段。此字段包含求职者的姓名。

   - `programming_languages`, a [`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html) field. This field contains programming languages known by the job candidate.

     `programming_languages`，一个[`keyword`](https://www.elastic.co/guide/en/elasticsearch/reference/master/keyword.html)字段。该字段包含求职者已知的编程语言。

   - `required_matches`, a [numeric](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) `long` field. This field contains the number of matching terms required to return a document.

     `required_matches`,[数字](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) `long`字段。此字段包含返回文档所需的匹配项数。

   ```console
   PUT /job-candidates
   {
     "mappings": {
       "properties": {
         "name": {
           "type": "keyword"
         },
         "programming_languages": {
           "type": "keyword"
         },
         "required_matches": {
           "type": "long"
         }
       }
     }
   }
   ```

2. Index a document with an ID of `1` and the following values:

   使用 ID`1`和以下值索引文档：

   - `Jane Smith` in the `name` field.

     `Jane Smith`在该`name`领域。

   - `["c++", "java"]` in the `programming_languages` field.

     `["c++", "java"]`在该`programming_languages`领域。

   - `2` in the `required_matches` field.

     `2`在该`required_matches`领域。

   Include the `?refresh` parameter so the document is immediately available for search.

   包括`?refresh`参数，以便文档立即可用于搜索。

   ```console
   PUT /job-candidates/_doc/1?refresh
   {
     "name": "Jane Smith",
     "programming_languages": [ "c++", "java" ],
     "required_matches": 2
   }
   ```

3. Index another document with an ID of `2` and the following values:

   使用 ID`2`和以下值索引另一个文档：

   - `Jason Response` in the `name` field.

     `Jason Response`在该`name`领域。

   - `["java", "php"]` in the `programming_languages` field.

     `["java", "php"]`在该`programming_languages`领域。

   - `2` in the `required_matches` field.

     `2`在该`required_matches`领域。

   

   ```console
   PUT /job-candidates/_doc/2?refresh
   {
     "name": "Jason Response",
     "programming_languages": [ "java", "php" ],
     "required_matches": 2
   }
   ```

You can now use the `required_matches` field value as the number of matching terms required to return a document in the `terms_set` query.

您现在可以使用`required_matches`字段值作为在`terms_set`查询中返回文档所需的匹配项数。

####  Example query

The following search returns documents where the `programming_languages` field contains at least two of the following terms:

以下搜索返回`programming_languages`字段包含至少两个以下术语的文档：

- `c++`
- `java`
- `php`

The `minimum_should_match_field` is `required_matches`. This means the number of matching terms required is `2`, the value of the `required_matches` field.

该`minimum_should_match_field`是`required_matches`。这意味着所需的匹配项数是 字段`2`的值`required_matches`。

```console
GET /job-candidates/_search
{
  "query": {
    "terms_set": {
      "programming_languages": {
        "terms": [ "c++", "java", "php" ],
        "minimum_should_match_field": "required_matches"
      }
    }
  }
}
```

###  Top-level parameters for `terms_set`

**`<field>`**

(Required, object) Field you wish to search.

（必需，对象）要搜索的字段。

### Parameters for `<field>`

**`terms`**

(Required, array of strings) Array of terms you wish to find in the provided `<field>`. To return a document, a required number of terms must exactly match the field values, including whitespace and capitalization.

（必需，字符串数组）您希望在提供的 `<field>`. 要返回文档，所需数量的术语必须与字段值完全匹配，包括空格和大写。

The required number of matching terms is defined in the `minimum_should_match_field` or `minimum_should_match_script` parameter.

所需的匹配项数在`minimum_should_match_field`or`minimum_should_match_script`参数中定义 。

**`minimum_should_match_field`**

(Optional, string) [Numeric](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html) field containing the number of matching terms required to return a document.

（可选，字符串）包含返回文档所需的匹配项数的[数字](https://www.elastic.co/guide/en/elasticsearch/reference/master/number.html)字段。

**`minimum_should_match_script`**

(Optional, string) Custom script containing the number of matching terms required to return a document.

（可选，字符串）包含返回文档所需的匹配项数的自定义脚本。

For parameters and valid values, see [Scripting](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html).

有关参数和有效值，请参阅[脚本](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting.html)。

For an example query using the `minimum_should_match_script` parameter, see [How to use the `minimum_should_match_script` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-set-query.html#terms-set-query-script).

有关使用`minimum_should_match_script`参数的示例查询，请参阅 [如何使用`minimum_should_match_script` 参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-terms-set-query.html#terms-set-query-script)。

###  Notes

####  How to use the `minimum_should_match_script` parameter

You can use `minimum_should_match_script` to define the required number of matching terms using a script. This is useful if you need to set the number of required terms dynamically.

您可以使用`minimum_should_match_script`脚本定义所需数量的匹配术语。如果您需要动态设置所需术语的数量，这将非常有用。

#####  Example query using `minimum_should_match_script`

The following search returns documents where the `programming_languages` field contains at least two of the following terms:

以下搜索返回`programming_languages`字段包含至少两个以下术语的文档：

- `c++`
- `java`
- `php`

The `source` parameter of this query indicates:

该`source`查询的参数表示：

- The required number of terms to match cannot exceed `params.num_terms`, the number of terms provided in the `terms` field.

  需要匹配的术语数不能超过字段中`params.num_terms`提供的术语数`terms`。

- The required number of terms to match is `2`, the value of the `required_matches` field.

  需要匹配的术语数是字段`2`的值 `required_matches`。

```console
GET /job-candidates/_search
{
  "query": {
    "terms_set": {
      "programming_languages": {
        "terms": [ "c++", "java", "php" ],
        "minimum_should_match_script": {
          "source": "Math.min(params.num_terms, doc['required_matches'].value)"
        },
        "boost": 1.0
      }
    }
  }
}
```

#  Wildcard query

Returns documents that contain terms matching a wildcard pattern.

返回包含匹配通配符模式的术语的文档。

A wildcard operator is a placeholder that matches one or more characters. For example, the `*` wildcard operator matches zero or more characters. You can combine wildcard operators with other characters to create a wildcard pattern.

通配符运算符是匹配一个或多个字符的占位符。例如，`*`通配符运算符匹配零个或多个字符。您可以将通配符运算符与其他字符组合以创建通配符模式。

### Example request

The following search returns documents where the `user.id` field contains a term that begins with `ki` and ends with `y`. These matching terms can include `kiy`, `kity`, or `kimchy`.

以下搜索返回`user.id`字段包含以 开头`ki`和结尾的术语的文档`y`。这些匹配条件可以包括`kiy`， `kity`或`kimchy`。

```console
GET /_search
{
  "query": {
    "wildcard": {
      "user.id": {
        "value": "ki*y",
        "boost": 1.0,
        "rewrite": "constant_score"
      }
    }
  }
}
```

### Top-level parameters for `wildcard`

**`<field>`**

(Required, object) Field you wish to search.

（必需，对象）要搜索的字段。

###  Parameters for `<field>`

**`value`**

(Required, string) Wildcard pattern for terms you wish to find in the provided `<field>`.

（必需，字符串）您希望在提供的 `<field>`.

This parameter supports two wildcard operators:

此参数支持两个通配符运算符：

- `?`, which matches any single character

  `?`, 匹配任何单个字符

- `*`, which can match zero or more characters, including an empty one

  `*`, 可以匹配零个或多个字符，包括空字符

  > WARNING: Avoid beginning patterns with `*` or `?`. This can increase the iterations needed to find matching terms and slow search performance.
  >
  > 避免以`*`或开始模式`?`。这会增加查找匹配项所需的迭代次数并降低搜索性能。

**`boost`**

(Optional, float) Floating point number used to decrease or increase the [relevance scores](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores) of a query. Defaults to `1.0`.

（可选，浮点数）用于减少或增加查询[相关性分数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-filter-context.html#relevance-scores)的浮点数 。默认为`1.0`.

You can use the `boost` parameter to adjust relevance scores for searches containing two or more queries.

您可以使用该`boost`参数来调整包含两个或更多查询的搜索的相关性分数。

Boost values are relative to the default value of `1.0`. A boost value between `0` and `1.0` decreases the relevance score. A value greater than `1.0` increases the relevance score.

Boost 值相对于 的默认值`1.0`。`0`和之间的提升值会 `1.0`降低相关性分数。大于 的值会`1.0` 增加相关性分数。

**`rewrite`**

(Optional, string) Method used to rewrite the query. For valid values and more information, see the [`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html).

（可选，字符串）用于重写查询的方法。有关有效值和更多信息，请参阅 [`rewrite`参数](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-multi-term-rewrite.html)。

**`case_insensitive` \**[\*\*7.10.0\*\*]\*\*Added in 7.10.0.\*\**\***

(Optional, Boolean) Allows case insensitive matching of the pattern with the indexed field values when set to true. Default is false which means the case sensitivity of matching depends on the underlying field’s mapping.

（可选，布尔值）设置为 true 时，允许模式与索引字段值不区分大小写匹配。默认为 false，这意味着匹配的区分大小写取决于基础字段的映射。

###  Notes

####  Allow expensive queries

Wildcard queries will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) 设置为 false，则不会执行通配符查询。