# Search your data

## Search across clusters

**Cross-cluster search** 跨节点搜索可以让你通过一个搜索请求，请求一个或多个远程节点。可以通过该搜索过滤和分析存储在不同节点的数据中心的日志。

### Supported APIs

- [Search](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html)
- [Async search](https://www.elastic.co/guide/en/elasticsearch/reference/master/async-search.html)
- [Multi search](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-multi-search.html)
- [Search template](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-template.html)
- [Multi search template](https://www.elastic.co/guide/en/elasticsearch/reference/master/multi-search-template.html)
- [Field capabilities](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-field-caps.html)

### Cross-cluster search examples

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

### Skip unavailable clusters

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

###  Selecting gateway and seed nodes in sniff mode

使用sniff mode（嗅觉模式）去搜索网关和种子节点

For remote clusters using the [sniff connection](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-remote-clusters.html#sniff-mode) mode, gateway and seed nodes need to be accessible from the local cluster via your network.

By default, any non-[master-eligible](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-node.html#master-node) node can act as a gateway node. If wanted, you can define the gateway nodes for a cluster by setting `cluster.remote.node.attr.gateway` to `true`.

For cross-cluster search, we recommend you use gateway nodes that are capable of serving as [coordinating nodes](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-node.html#coordinating-node) for search requests. If wanted, the seed nodes for a cluster can be a subset of these gateway nodes.

当集群使用嗅觉连接模式时，网关和种子节点需要通过网络从本地集群节点变为可用。

默认，任何非主节点候选节点都可以作为网关节点。可以定义网关节点从集群设置`cluster.remote.node.attr.gateway` 为 `true`。

对于跨集群检索，我们建议使用这个有能力作为协同节点服务请求的网关节点。如果想要的话种子节点可以成为这些网关节点的子节点。

###  Cross-cluster search in proxy mode

[Proxy mode](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-remote-clusters.html#proxy-mode) remote cluster connections support cross-cluster search. All remote connections connect to the configured `proxy_address`. Any desired connection routing to gateway or [coordinating nodes](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-node.html#coordinating-node) must be implemented by the intermediate proxy at this configured address.

代理模式的远程节点连接是支持跨集群检索的。所有的远程连接连接到配置的`proxy_address`（代理路径）。任何连接想要路由到网关或者[coordinating nodes](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-node.html#coordinating-node) （协同节点）的话必须通过这个配置的中间代理节点地址实现。

### How cross-cluster search handles network delays

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

### Minimize network roundtrips

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

### Don’t minimize network roundtrips

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

### Supported configurations

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

为了防止硬件故障增加搜索容量，ES存储了一个索引数据的备份在多个分片多个节点上。当运行一个检索请求的时候，ES会选择存在这个索引数据副本的节点并将搜索请求转发到这个节点的粉脸。这个过程被称为*搜索分片路由* 或 *路由*。

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

ES支持通过数组或多值字段排序。该`mode`选项控制选择哪个数组值来对它所属的文档进行排序。该`mode`选项可以具有以下值：

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

在上一个例子中，`index_long` 索引的值被转成了double类型以便于和`index_double`索引产生的值兼容。当然也可以将浮点字段为`long`，但是请注意这个例子中浮点数被替换成了一个范围更大的值（范围大于或等于如果这个值是负数）作为参数，他等于一个精确的整数。

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

 `missing`参数指定

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

