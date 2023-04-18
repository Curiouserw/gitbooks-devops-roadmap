# Elasticsearch数据的分配路由

# 一、简介



# 二、索引分片的路由分配



## 2、给节点打上标签

elasticsearch.yml

```bash
node.attr.size: medium
```

或启动命令

```bash
./bin/elasticsearch -Enode.attr.size=medium
```

## 3. 

```bash
PUT test/_settings
{
  "index.routing.allocation.include.size": "big",
  "index.routing.allocation.include.rack": "rack1"
}
```

## 4. 

#### ](https://github.com/elastic/elasticsearch/edit/7.5/docs/reference/index-modules/allocation/filtering.asciidoc)

- index.routing.allocation.include.{attribute}`**

  Assign the index to a node whose `{attribute}` has at least one of the comma-separated values.

- **`index.routing.allocation.require.{attribute}`**

  Assign the index to a node whose `{attribute}` has *all* of the comma-separated values.

- **`index.routing.allocation.exclude.{attribute}`**

  Assign the index to a node whose `{attribute}` has *none* of the comma-separated values.

内置的attribute：

| `_name`       | Match nodes by node name                                     |
| ------------- | ------------------------------------------------------------ |
| `_host_ip`    | Match nodes by host IP address (IP associated with hostname) |
| `_publish_ip` | Match nodes by publish IP address                            |
| `_ip`         | Match either `_host_ip` or `_publish_ip`                     |
| `_host`       | Match nodes by hostname                                      |
| `_id`         | Match nodes by node id                                       |

```bash
PUT test/_settings
{
  "index.routing.allocation.include._ip": "192.168.2.*"
}
```

## 5. 

```bash
PUT loginmac-201905/_settings
{
  "index": {
    "routing": {
      "allocation": {
        "require": {
          "box_type": "warm"
        }
      }
    }
  }
}
```

## 6. 

```bash
POST /_cluster/reroute
{
  "commands": [
    {
      "move": {
        "index": "loginmac-201905",
        "shard": 2,
        "from_node": "node-248",
        "to_node": "node-12"
      }
    }
  ]
}
```





# 参考

1. https://www.elastic.co/guide/en/elasticsearch/reference/7.5/shard-allocation-filtering.html