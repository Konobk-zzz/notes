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