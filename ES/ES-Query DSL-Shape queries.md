# Shape queries

Like [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) Elasticsearch supports the ability to index arbitrary two dimension (non Geospatial) geometries making it possible to map out virtual worlds, sporting venues, theme parks, and CAD diagrams.

就像[`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html)Elasticsearch 支持索引任意二维（非地理空间）几何的能力，这使得绘制虚拟世界、体育场馆、主题公园和 CAD 图表成为可能。

Elasticsearch supports two types of cartesian data: [`point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/point.html) fields which support x/y pairs, and [`shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/shape.html) fields, which support points, lines, circles, polygons, multi-polygons, etc.

Elasticsearch 支持两种类型的笛卡尔数据： [`point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/point.html)支持 x/y 对的 [`shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/shape.html)字段，以及支持点、线、圆、多边形、多多边形等的字段。

The queries in this group are:

该组中的查询是：

**[`shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-shape-query.html) query**

Finds documents with:

查找具有以下内容的文档：

- `shapes` which either intersect, are contained by, are within or do not intersect with the specified shape

  `shapes` 与指定的形状相交、包含在其中、在其中或不相交

- `points` which intersect the specified shape

  points 与指定的形状相交

#  Shape query

Queries documents that contain fields indexed using the `shape` type.

查询包含使用该`shape`类型索引的字段的文档。

Requires the [`shape` Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/master/shape.html).

需要[`shape`Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/master/shape.html)。

The query supports two ways of defining the target shape, either by providing a whole shape definition, or by referencing the name, or id, of a shape pre-indexed in another index. Both formats are defined below with examples.

该查询支持两种定义目标形状的方法，一种是提供整个形状定义，另一种是通过引用在另一个索引中预先索引的形状的名称或 ID。下面通过示例定义了这两种格式。

##  Inline Shape Definition

Similar to the `geo_shape` query, the `shape` query uses [GeoJSON](http://geojson.org/) or [Well Known Text](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) (WKT) to represent shapes.

与`geo_shape`查询类似，`shape`查询使用 [GeoJSON](http://geojson.org/)或 [众所周知的文本](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) (WKT) 来表示形状。

Given the following index:

鉴于以下索引：

```console
PUT /example
{
  "mappings": {
    "properties": {
      "geometry": {
        "type": "shape"
      }
    }
  }
}

PUT /example/_doc/1?refresh=wait_for
{
  "name": "Lucky Landing",
  "geometry": {
    "type": "point",
    "coordinates": [ 1355.400544, 5255.530286 ]
  }
}
```

The following query will find the point using the Elasticsearch’s `envelope` GeoJSON extension:

以下查询将使用 Elasticsearch 的`envelope`GeoJSON 扩展找到该点 ：

```console
GET /example/_search
{
  "query": {
    "shape": {
      "geometry": {
        "shape": {
          "type": "envelope",
          "coordinates": [ [ 1355.0, 5355.0 ], [ 1400.0, 5200.0 ] ]
        },
        "relation": "within"
      }
    }
  }
}
```

##  Pre-Indexed Shape

The Query also supports using a shape which has already been indexed in another index. This is particularly useful for when you have a pre-defined list of shapes which are useful to your application and you want to reference this using a logical name (for example *New Zealand*) rather than having to provide their coordinates each time. In this situation it is only necessary to provide:

查询还支持使用已在另一个索引中编入索引的形状。当您有一个对您的应用程序有用的预定义形状列表并且您想使用逻辑名称（例如*New Zealand*）引用它而不是每次都必须提供它们的坐标时，这特别有用。在这种情况下，只需提供：

- `id` - The ID of the document that containing the pre-indexed shape.

  `id` - 包含预索引形状的文档的 ID。

- `index` - Name of the index where the pre-indexed shape is. Defaults to *shapes*.

  `index`- 预索引形状所在的索引名称。默认为*shape*。

- `path` - The field specified as path containing the pre-indexed shape. Defaults to *shape*.

  `path`- 指定为包含预索引形状的路径的字段。默认为*shape*。

- `routing` - The routing of the shape document if required.

  `routing` - 如果需要，形状文件的路由。

The following is an example of using the Filter with a pre-indexed shape:

以下是使用具有预索引形状的过滤器的示例：

```console
PUT /shapes
{
  "mappings": {
    "properties": {
      "geometry": {
        "type": "shape"
      }
    }
  }
}

PUT /shapes/_doc/footprint
{
  "geometry": {
    "type": "envelope",
    "coordinates": [ [ 1355.0, 5355.0 ], [ 1400.0, 5200.0 ] ]
  }
}

GET /example/_search
{
  "query": {
    "shape": {
      "geometry": {
        "indexed_shape": {
          "index": "shapes",
          "id": "footprint",
          "path": "geometry"
        }
      }
    }
  }
}
```

##  Spatial Relations

The following is a complete list of spatial relation operators available:

以下是可用空间关系运算符的完整列表：

- `INTERSECTS` - (default) Return all documents whose `shape` field intersects the query geometry.

  `INTERSECTS`-（默认）返回`shape`字段与查询几何相交的所有文档。

- `DISJOINT` - Return all documents whose `shape` field has nothing in common with the query geometry.

  `DISJOINT`- 返回其`shape`字段与查询几何没有共同之处的所有文档。

- `WITHIN` - Return all documents whose `shape` field is within the query geometry.

  `WITHIN`- 返回其`shape`字段在查询几何范围内的所有文档。

- `CONTAINS` - Return all documents whose `shape` field contains the query geometry.

  `CONTAINS`- 返回其`shape`字段包含查询几何的所有文档。

##  Ignore Unmapped

When set to `true` the `ignore_unmapped` option will ignore an unmapped field and will not match any documents for this query. This can be useful when querying multiple indexes which might have different mappings. When set to `false` (the default value) the query will throw an exception if the field is not mapped.

当设置`true`为该`ignore_unmapped`选项时，将忽略未映射的字段，并且不会匹配此查询的任何文档。这在查询可能具有不同映射的多个索引时非常有用。当设置为 `false`（默认值）时，如果字段未映射，查询将抛出异常。