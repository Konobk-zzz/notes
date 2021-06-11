#  Geo queries

Elasticsearch supports two types of geo data: [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) fields which support lat/lon pairs, and [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) fields, which support points, lines, circles, polygons, multi-polygons, etc.

Elasticsearch支持两种类型的地理数据： [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)支持纬度/经度对的[`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html)字段和 支持点，线，圆，多边形，多面体等的字段。

The queries in this group are:

- **[`geo_bounding_box`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html) query**

  Finds documents with geo-points that fall into the specified rectangle.

  查找具有落入指定矩形的地理位置的文档。

- **[`geo_distance`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-distance-query.html) query**

  Finds documents with geo-points within the specified distance of a central point.

  查找地理点在中心点指定距离内的文档。

- **[`geo_polygon`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-polygon-query.html) query**

  Find documents with geo-points within the specified polygon.

  查找具有指定多边形内的地理点的文档。

- **[`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-shape-query.html) query**

  Finds documents with:

  查找具有以下内容的文档：

  - `geo-shapes` which either intersect, are contained by, or do not intersect with the specified geo-shape

    与指定的几何形状相交，包含于其中或不与指定的几何形状相交的对象

  - `geo-points` which intersect the specified geo-shape

    与指定的地理形状相交

#  Geo-bounding box query

Matches [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) and [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) values that intersect a bounding box.

匹配 [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) 和[`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) 与边界框相交的值。

##  Example

Assume the following the following documents are indexed:

假设以下文档已建立索引：

```console
PUT /my_locations
{
  "mappings": {
    "properties": {
      "pin": {
        "properties": {
          "location": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}

PUT /my_locations/_doc/1
{
  "pin": {
    "location": {
      "lat": 40.12,
      "lon": -71.34
    }
  }
}

PUT /my_geoshapes
{
  "mappings": {
    "properties": {
      "pin": {
        "properties": {
          "location": {
            "type": "geo_shape"
          }
        }
      }
    }
  }
}

PUT /my_geoshapes/_doc/1
{
  "pin": {
    "location": {
      "type" : "polygon",
      "coordinates" : [[[13.0 ,51.5], [15.0, 51.5], [15.0, 54.0], [13.0, 54.0], [13.0 ,51.5]]]
    }
  }
}
```

Use a `geo_bounding_box` filter to match `geo_point` values that intersect a bounding box. To define the box, provide geopoint values for two opposite corners.

使用`geo_bounding_box`过滤器来匹配`geo_point`与边界框相交的值。要定义该框，请提供两个相对角的geopoint值。

```console
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

Use the same filter to match `geo_shape` values that intersect the bounding box:

使用相同的过滤器来匹配`geo_shape`与边界框相交的值：

```console
GET my_geoshapes/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

To match both `geo_point` and `geo_shape` values, search both indices:

要同时匹配`geo_point`和`geo_shape`值，请搜索两个索引：

```console
GET my_locations,my_geoshapes/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

## Query Options

| Option              | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `_name`             | Optional name field to identify the filter<br />可选名称字段，用于标识过滤器 |
| `validation_method` | Set to `IGNORE_MALFORMED` to accept geo points with invalid latitude or longitude, set to `COERCE` to also try to infer correct latitude or longitude. (default is `STRICT`).<br />设置为`IGNORE_MALFORMED`接受具有无效纬度或经度的地理点，设置 `COERCE`为还尝试推断正确的纬度或经度。（默认为`STRICT`）。 |
| `type`              | Set to one of `indexed` or `memory` to defines whether this filter will be executed in memory or indexed. See [Type](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html#geo-bbox-type) below for further details Default is `memory`.<br />设置为`indexed`或`memory`定义此过滤器是在内存中执行还是在索引中定义。有关更多详细信息，请参见下面的[类型](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-bounding-box-query.html#geo-bbox-type)。默认值为`memory`。 |

##  Accepted Formats

In much the same way the geo_point type can accept different representations of the geo point, the filter can accept it as well:

几乎相同的geo_point 类型可以通过不同的表示形式来表示地理点，过滤器也可以接受它：

##  Lat Lon As Properties

```console
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.01,
              "lon": -71.12
            }
          }
        }
      }
    }
  }
}
```

##  Lat Lon As Array

Format in `[lon, lat]`, note, the order of lon/lat here in order to conform with [GeoJSON](http://geojson.org/).

请在`[lon, lat]`此处格式化lon / lat的顺序，以符合[GeoJSON](http://geojson.org/)。

```console
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": [ -74.1, 40.73 ],
            "bottom_right": [ -71.12, 40.01 ]
          }
        }
      }
    }
  }
}
```

##  Lat Lon As String

Format in `lat,lon`.

格式为`lat,lon`.

```console
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": "40.73, -74.1",
            "bottom_right": "40.01, -71.12"
          }
        }
      }
    }
  }
}
```

##  Bounding Box as Well-Known Text (WKT)

```console
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "wkt": "BBOX (-74.1, -71.12, 40.73, 40.01)"
          }
        }
      }
    }
  }
}
```

## Geohash

```console
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": "dr5r9ydj2y73",
            "bottom_right": "drj7teegpus6"
          }
        }
      }
    }
  }
}
```

When geohashes are used to specify the bounding the edges of the bounding box, the geohashes are treated as rectangles. The bounding box is defined in such a way that its top left corresponds to the top left corner of the geohash specified in the `top_left` parameter and its bottom right is defined as the bottom right of the geohash specified in the `bottom_right` parameter.

当 geohashes 用于指定边界框的边界时，geohashes 被视为矩形。边界框的定义方式是，其左上角对应于`top_left`参数中指定的 geohash 的左上角，其右下角定义为`bottom_right`参数中指定的 geohash 的右下角。

In order to specify a bounding box that would match entire area of a geohash the geohash can be specified in both `top_left` and `bottom_right` parameters:

为了指定与 geohash 的整个区域匹配的边界框，可以在`top_left`和 `bottom_right`参数中指定 geohash ：

```console
GET my_locations/_search
{
  "query": {
    "geo_bounding_box": {
      "pin.location": {
        "top_left": "dr",
        "bottom_right": "dr"
      }
    }
  }
}
```

In this example, the geohash `dr` will produce the bounding box query with the top left corner at `45.0,-78.75` and the bottom right corner at `39.375,-67.5`.

在此示例中，geohash`dr`将生成左上角为`45.0,-78.75`、右下角为 的边界框查询`39.375,-67.5`。

##  Vertices

The vertices of the bounding box can either be set by `top_left` and `bottom_right` or by `top_right` and `bottom_left` parameters. More over the names `topLeft`, `bottomRight`, `topRight` and `bottomLeft` are supported. Instead of setting the values pairwise, one can use the simple names `top`, `left`, `bottom` and `right` to set the values separately.

边界框的顶点可以通过`top_left`和 `bottom_right`或通过`top_right`和`bottom_left`参数来设置。更过名`topLeft`，`bottomRight`，`topRight`和`bottomLeft` 支持。相反，在设置这些值成对的，一个可以使用简单的名字`top`，`left`，`bottom`并`right`分别设置的值。

```console
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top": 40.73,
            "left": -74.1,
            "bottom": 40.01,
            "right": -71.12
          }
        }
      }
    }
  }
}
```

##  Multi Location Per Document

The filter can work with multiple locations / points per document. Once a single location / point matches the filter, the document will be included in the filter

过滤器可以处理每个文档的多个位置/点。一旦单个位置/点与过滤器匹配，则文档将包含在过滤器中

##  Type

The type of the bounding box execution by default is set to `memory`, which means in memory checks if the doc falls within the bounding box range. In some cases, an `indexed` option will perform faster (but note that the `geo_point` type must have lat and lon indexed in this case). Note, when using the indexed option, multi locations per document field are not supported. Here is an example:

边界框执行的类型默认设置为`memory`，这意味着在内存中检查文档是否在边界框范围内。在某些情况下，`indexed`选项的执行速度会更快（但请注意，`geo_point`在这种情况下，类型必须具有lat和lon索引）。请注意，使用索引选项时，不支持每个文档字段的多个位置。下面是一个例子：

```console
GET my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_bounding_box": {
          "pin.location": {
            "top_left": {
              "lat": 40.73,
              "lon": -74.1
            },
            "bottom_right": {
              "lat": 40.10,
              "lon": -71.12
            }
          },
          "type": "indexed"
        }
      }
    }
  }
}
```

##  Ignore Unmapped

When set to `true` the `ignore_unmapped` option will ignore an unmapped field and will not match any documents for this query. This can be useful when querying multiple indexes which might have different mappings. When set to `false` (the default value) the query will throw an exception if the field is not mapped.

当设置`true`为该`ignore_unmapped`选项时，将忽略未映射的字段，并且不会匹配此查询的任何文档。这在查询可能具有不同映射的多个索引时非常有用。当设置为 `false`（默认值）时，如果字段未映射，查询将抛出异常。

##  Notes on Precision

Geopoints have limited precision and are always rounded down during index time. During the query time, upper boundaries of the bounding boxes are rounded down, while lower boundaries are rounded up. As a result, the points along on the lower bounds (bottom and left edges of the bounding box) might not make it into the bounding box due to the rounding error. At the same time points alongside the upper bounds (top and right edges) might be selected by the query even if they are located slightly outside the edge. The rounding error should be less than 4.20e-8 degrees on the latitude and less than 8.39e-8 degrees on the longitude, which translates to less than 1cm error even at the equator.

地理点的精度有限，并且在索引期间总是向下取整。在查询期间，边界框的上边界向下舍入，而下边界向上舍入。因此，由于舍入误差，下边界（边界框的底部和左边缘）上的点可能无法进入边界框。同时，查询可能会选择上边界（顶部和右侧边缘）旁边的点，即使它们稍微位于边缘之外。纬度上的舍入误差应小于 4.20e-8 度，经度上的舍入误差应小于 8.39e-8 度，这意味着即使在赤道上误差也小于 1 厘米。

Geoshapes also have limited precision due to rounding. Geoshape edges along the bounding box’s bottom and left edges may not match a `geo_bounding_box` query. Geoshape edges slightly outside the box’s top and right edges may still match the query.

由于舍入，几何形状的精度也受到限制。边界框底部和左侧边缘的Geoshape边缘可能与`geo_bounding_box`查询不匹配。略微位于框顶部和右侧边缘之外的Geoshape边缘可能仍与查询匹配。

#  Geo-distance query

Matches [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) and [`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) values within a given distance of a geopoint.

地理点给定距离内的匹配项[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)和[`geo_shape`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html)值。

##  Example

Assume the following the following documents are indexed:

假设以下文档已编入索引：

```console
PUT /my_locations
{
  "mappings": {
    "properties": {
      "pin": {
        "properties": {
          "location": {
            "type": "geo_point"
          }
        }
      }
    }
  }
}

PUT /my_locations/_doc/1
{
  "pin": {
    "location": {
      "lat": 40.12,
      "lon": -71.34
    }
  }
}

PUT /my_geoshapes
{
  "mappings": {
    "properties": {
      "pin": {
        "properties": {
          "location": {
            "type": "geo_shape"
          }
        }
      }
    }
  }
}

PUT /my_geoshapes/_doc/1
{
  "pin": {
    "location": {
      "type" : "polygon",
      "coordinates" : [[[13.0 ,51.5], [15.0, 51.5], [15.0, 54.0], [13.0, 54.0], [13.0 ,51.5]]]
    }
  }
}
```

Use a `geo_distance` filter to match `geo_point` values within a specified distance of another geopoint:

使用`geo_distance`过滤器匹配`geo_point`另一个地理点指定距离内的值：

```console
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "200km",
          "pin.location": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

Use the same filter to match `geo_shape` values within the given distance:

使用相同的过滤器匹配`geo_shape`给定距离内的值：

```console
GET my_geoshapes/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "200km",
          "pin.location": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

To match both `geo_point` and `geo_shape` values, search both indices:

要同时匹配`geo_point`和`geo_shape`值，请搜索两个索引：

```console
GET my_locations,my_geoshapes/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "200km",
          "pin.location": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

##  Accepted Formats

In much the same way the `geo_point` type can accept different representations of the geo point, the filter can accept it as well:

与`geo_point`类型可以接受地理点的不同表示形式大致相同，过滤器也可以接受它：

###  Lat Lon As Properties

```console
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "12km",
          "pin.location": {
            "lat": 40,
            "lon": -70
          }
        }
      }
    }
  }
}
```

###  Lat Lon As Array

Format in `[lon, lat]`, note, the order of lon/lat here in order to conform with [GeoJSON](http://geojson.org/).

`[lon, lat]`请注意，这里的 lon/lat 顺序是为了符合[GeoJSON](http://geojson.org/)格式。

```console
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "12km",
          "pin.location": [ -70, 40 ]
        }
      }
    }
  }
}
```

###  Lat Lon As String

Format in `lat,lon`.

格式为`lat,lon`.

```console
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "12km",
          "pin.location": "40,-70"
        }
      }
    }
  }
}
```

### Geohash

```console
GET /my_locations/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_distance": {
          "distance": "12km",
          "pin.location": "drm3btev3e86"
        }
      }
    }
  }
}
```

##  Options

The following are options allowed on the filter:

以下是过滤器允许的选项：

| `distance`          | The radius of the circle centred on the specified location. Points which fall into this circle are considered to be matches. The `distance` can be specified in various units. See [Distance Units](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#distance-units).<br />以指定位置为中心的圆的半径。落入这个圆圈的点被认为是匹配的。在`distance`可以以各种单元指定。请参阅[距离单位](https://www.elastic.co/guide/en/elasticsearch/reference/master/common-options.html#distance-units)。 |
| ------------------- | ------------------------------------------------------------ |
| `distance_type`     | How to compute the distance. Can either be `arc` (default), or `plane` (faster, but inaccurate on long distances and close to the poles).<br />如何计算距离。可以是`arc`（默认）或`plane`（更快，但在长距离和靠近两极时不准确）。 |
| `_name`             | Optional name field to identify the query<br />用于标识查询的可选名称字段 |
| `validation_method` | Set to `IGNORE_MALFORMED` to accept geo points with invalid latitude or longitude, set to `COERCE` to additionally try and infer correct coordinates (default is `STRICT`).<br />设置为`IGNORE_MALFORMED`接受具有无效纬度或经度的地理点，设置`COERCE`为另外尝试推断正确的坐标（默认为`STRICT`）。 |

##  Multi Location Per Document

The `geo_distance` filter can work with multiple locations / points per document. Once a single location / point matches the filter, the document will be included in the filter.

该`geo_distance`过滤器可以用每份文件的多个位置/点工作。一旦单个位置/点与过滤器匹配，该文档将包含在过滤器中。

##  Ignore Unmapped

When set to `true` the `ignore_unmapped` option will ignore an unmapped field and will not match any documents for this query. This can be useful when querying multiple indexes which might have different mappings. When set to `false` (the default value) the query will throw an exception if the field is not mapped.

当设置`true`为该`ignore_unmapped`选项时，将忽略未映射的字段，并且不会匹配此查询的任何文档。这在查询可能具有不同映射的多个索引时非常有用。当设置为 `false`（默认值）时，如果字段未映射，查询将抛出异常。

# Geo-polygon query

> **WARNING:**  Deprecated in 7.12.
>
> Use [Geo-shape](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-shape-query.html) instead where polygons are defined in GeoJSON or [Well-Known Text (WKT)](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html).
>
> ###  7.12 中已弃用。
>
> 使用[Geo-shape](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-geo-shape-query.html)而不是在 GeoJSON 或[Well-Known Text (WKT)](http://docs.opengeospatial.org/is/18-010r7/18-010r7.html)中定义多边形的地方。

A query returning hits that only fall within a polygon of points. Here is an example:

返回仅落入点多边形内的命中的查询。下面是一个例子：

```console
GET /_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_polygon": {
          "person.location": {
            "points": [
              { "lat": 40, "lon": -70 },
              { "lat": 30, "lon": -80 },
              { "lat": 20, "lon": -90 }
            ]
          }
        }
      }
    }
  }
}
```

##  Query Options

| Option              | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `_name`             | Optional name field to identify the filter<br />用于标识过滤器的可选名称字段 |
| `validation_method` | Set to `IGNORE_MALFORMED` to accept geo points with invalid latitude or longitude, `COERCE` to try and infer correct latitude or longitude, or `STRICT` (default is `STRICT`).<br />设置为`IGNORE_MALFORMED`接受具有无效纬度或经度的地理点，`COERCE`以尝试推断正确的纬度或经度，或`STRICT`（默认为`STRICT`）。 |

##  Allowed Formats

###  Lat Long as Array

Format as `[lon, lat]`

格式为 `[lon, lat]`

Note: the order of lon/lat here must conform with [GeoJSON](http://geojson.org/).

注意：这里的 lon/lat 顺序必须符合[GeoJSON](http://geojson.org/)。

```console
GET /_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_polygon": {
          "person.location": {
            "points": [
              [ -70, 40 ],
              [ -80, 30 ],
              [ -90, 20 ]
            ]
          }
        }
      }
    }
  }
}
```

###  Lat Lon as String

Format in `lat,lon`.

格式为`lat,lon`.

```console
GET /_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_polygon": {
          "person.location": {
            "points": [
              "40, -70",
              "30, -80",
              "20, -90"
            ]
          }
        }
      }
    }
  }
}
```

###  Geohash

```console
GET /_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_polygon": {
          "person.location": {
            "points": [
              "drn5x1g8cu2y",
              "30, -80",
              "20, -90"
            ]
          }
        }
      }
    }
  }
}
```

## geo_point Type

The query **requires** the [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html) type to be set on the relevant field.

查询**需要**的[`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)要在相关领域的集合类型。

##  Ignore Unmapped

When set to `true` the `ignore_unmapped` option will ignore an unmapped field and will not match any documents for this query. This can be useful when querying multiple indexes which might have different mappings. When set to `false` (the default value) the query will throw an exception if the field is not mapped.

当设置`true`为该`ignore_unmapped`选项时，将忽略未映射的字段，并且不会匹配此查询的任何文档。这在查询可能具有不同映射的多个索引时非常有用。当设置为 `false`（默认值）时，如果字段未映射，查询将抛出异常。

#  Geo-shape query

Filter documents indexed using the `geo_shape` or `geo_point` type.

过滤使用`geo_shape`or`geo_point`类型索引的文档。

Requires the [`geo_shape` Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html) or the [`geo_point` Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html).

需要[`geo_shape`Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html)或 [`geo_point`Mapping](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-point.html)。

The `geo_shape` query uses the same grid square representation as the `geo_shape` mapping to find documents that have a shape that intersects with the query shape. It will also use the same Prefix Tree configuration as defined for the field mapping.

`geo_shape`查询使用相同的方格表示为 `geo_shape`映射发现有一个形状文档与查询形状相交。它还将使用为字段映射定义的相同前缀树配置。

The query supports two ways of defining the query shape, either by providing a whole shape definition, or by referencing the name of a shape pre-indexed in another index. Both formats are defined below with examples.

查询支持两种定义查询形状的方法，一种是提供整个形状定义，另一种是引用在另一个索引中预先编入索引的形状的名称。下面通过示例定义了这两种格式。

##  Inline Shape Definition

Similar to the `geo_shape` type, the `geo_shape` query uses [GeoJSON](http://geojson.org/) to represent shapes.

与`geo_shape`类型类似，`geo_shape`查询使用 [GeoJSON](http://geojson.org/)来表示形状。

Given the following index with locations as `geo_shape` fields:

给定以下索引，将位置作为`geo_shape`字段：

```console
PUT /example
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_shape"
      }
    }
  }
}

POST /example/_doc?refresh
{
  "name": "Wind & Wetter, Berlin, Germany",
  "location": {
    "type": "point",
    "coordinates": [ 13.400544, 52.530286 ]
  }
}
```

The following query will find the point using Elasticsearch’s `envelope` GeoJSON extension:

以下查询将使用 Elasticsearch 的`envelope`GeoJSON 扩展找到该点：

```console
GET /example/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_shape": {
          "location": {
            "shape": {
              "type": "envelope",
              "coordinates": [ [ 13.0, 53.0 ], [ 14.0, 52.0 ] ]
            },
            "relation": "within"
          }
        }
      }
    }
  }
}
```

The above query can, similarly, be queried on `geo_point` fields.

类似地，上面的查询可以在`geo_point`字段上进行查询。

```console
PUT /example_points
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

PUT /example_points/_doc/1?refresh
{
  "name": "Wind & Wetter, Berlin, Germany",
  "location": [13.400544, 52.530286]
}
```

Using the same query, the documents with matching `geo_point` fields are returned.

使用相同的查询，`geo_point`返回具有匹配字段的文档。

```console
GET /example_points/_search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "geo_shape": {
          "location": {
            "shape": {
              "type": "envelope",
              "coordinates": [ [ 13.0, 53.0 ], [ 14.0, 52.0 ] ]
            },
            "relation": "intersects"
          }
        }
      }
    }
  }
}
```

```json
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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "example_points",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name": "Wind & Wetter, Berlin, Germany",
          "location": [13.400544, 52.530286]
        }
      }
    ]
  }
}
```

##  Pre-Indexed Shape

The query also supports using a shape which has already been indexed in another index. This is particularly useful for when you have a pre-defined list of shapes and you want to reference the list using a logical name (for example *New Zealand*) rather than having to provide coordinates each time. In this situation, it is only necessary to provide:

该查询还支持使用已在另一个索引中编入索引的形状。当您有一个预定义的形状列表并且您想要使用逻辑名称（例如*New Zealand*）引用该列表而不必每次都提供坐标时，这尤其有用。在这种情况下，只需要提供：

- `id` - The ID of the document that containing the pre-indexed shape.

  `id` - 包含预索引形状的文档的 ID。

- `index` - Name of the index where the pre-indexed shape is. Defaults to *shapes*.

  `index`- 预索引形状所在的索引名称。默认为 *shape*。

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
      "location": {
        "type": "geo_shape"
      }
    }
  }
}

PUT /shapes/_doc/deu
{
  "location": {
    "type": "envelope",
    "coordinates" : [[13.0, 53.0], [14.0, 52.0]]
  }
}

GET /example/_search
{
  "query": {
    "bool": {
      "filter": {
        "geo_shape": {
          "location": {
            "indexed_shape": {
              "index": "shapes",
              "id": "deu",
              "path": "location"
            }
          }
        }
      }
    }
  }
}
```

##  Spatial Relations

The [geo_shape strategy](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html#spatial-strategy) mapping parameter determines which spatial relation operators may be used at search time.

所述[geo_shape策略](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html#spatial-strategy)映射参数确定其可在搜索时间被使用的空间关系的运营商。

The following is a complete list of spatial relation operators available when searching a geo field:

以下是搜索地理字段时可用的空间关系运算符的完整列表：

- `INTERSECTS` - (default) Return all documents whose `geo_shape` or `geo_point` field intersects the query geometry.

  `INTERSECTS`-（默认）返回其`geo_shape`或`geo_point`字段与查询几何相交的所有文档。

- `DISJOINT` - Return all documents whose `geo_shape` or `geo_point` field has nothing in common with the query geometry.

  `DISJOINT`- 返回其`geo_shape`或`geo_point`字段与查询几何没有共同之处的所有文档。

- `WITHIN` - Return all documents whose `geo_shape` or `geo_point` field is within the query geometry. Line geometries are not supported.

  `WITHIN`- 返回其`geo_shape`或`geo_point`字段在查询几何范围内的所有文档。不支持线几何。

- `CONTAINS` - Return all documents whose `geo_shape` or `geo_point` field contains the query geometry.

  `CONTAINS`- 返回其`geo_shape`或`geo_point`字段包含查询几何的所有文档。

##  Ignore Unmapped

When set to `true` the `ignore_unmapped` option will ignore an unmapped field and will not match any documents for this query. This can be useful when querying multiple indexes which might have different mappings. When set to `false` (the default value) the query will throw an exception if the field is not mapped.

当设置`true`为该`ignore_unmapped`选项时，将忽略未映射的字段，并且不会匹配此查询的任何文档。这在查询可能具有不同映射的多个索引时非常有用。当设置为 `false`（默认值）时，如果字段未映射，查询将抛出异常。

##  Notes

- Geo-shape queries on geo-shapes implemented with [`PrefixTrees`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html#prefix-trees) will not be executed if [`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries) is set to false.

  如果[`search.allow_expensive_queries`](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl.html#query-dsl-allow-expensive-queries)设置为 false ，则不会执行[`PrefixTrees`](https://www.elastic.co/guide/en/elasticsearch/reference/master/geo-shape.html#prefix-trees) 对实现的地理形状的地理形状查询 。

- When data is indexed in a `geo_shape` field as an array of shapes, the arrays are treated as one shape. For this reason, the following requests are equivalent.

  当数据在`geo_shape`字段中作为形状数组进行索引时，这些数组将被视为一个形状。因此，以下请求是等效的。

```console
PUT /test/_doc/1
{
  "location": [
    {
      "coordinates": [46.25,20.14],
      "type": "point"
    },
    {
      "coordinates": [47.49,19.04],
      "type": "point"
    }
  ]
}
```

```console
PUT /test/_doc/1
{
  "location":
    {
      "coordinates": [[46.25,20.14],[47.49,19.04]],
      "type": "multipoint"
    }
}
```

