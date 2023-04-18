# 一、xpack的monitoring功能“导致”failed to flush export bulks和 there are no ingest nodes in this cluster”报错

**原因**：

xpack的monitoring功能需要定义exporter用于导出监控数据， 默认的exporter是local exporter，也就是直接写入本地的集群，并且要求节点开启了ingest选项。

**解决方案**:

1. 将集群的结点配置里的ingest角色打开
2. 或者在集群设置elasticsearch.yml里，将local exporter的use ingest关掉
   ```yaml
    xpack.monitoring.exporters.my_local:
      type: local
      use_ingest: false
   ```
   但一般的，使用local cluster监控自己存在很大的问题，故障发生时，监控也没法看到了。 生产上最好是设置一个单独的监控集群，然后可以配置一个HTTP exporter，将监控数据送往这个监控集群

**参考**：

1. https://www.elastic.co/guide/en/x-pack/5.5/monitoring-cluster.html#http-exporter-reference
2. https://elasticsearch.cn/question/1915



# 二、监控日志索引Index的保存期限为7天

Elasticsearch的监控日志索引Index为"`.monitoring-*`"开头的，保存期限为7天，7天之后会自动删除。

**参考**

1. https://discuss.elastic.co/t/how-system-index-like-monitoring-es-6-2018-02-06-are-being-deleted-automatically/119578

# 三、字段过大导致kibana搜索是分片失败

**报错**：

```bash
The length of [response.keyword] field of [SwiBc3YBv0gFs9LK4P1_] doc of [docc-2020-12-18] index has exceeded [1000000] - maximum allowed to be analyzed for highlighting. This maximum can be set by changing the [index.highlight.max_analyzed_offset] index level setting. For large texts, indexing with offsets or term vectors is recommended!
```

**原因**：某个字段超出了字符偏移量上限

**解决方案**

```bash
PUT /分片失败的索引/_settings
 {
    "index" : {
        "highlight.max_analyzed_offset" : 60000000
    }
}
```

# 四、单节点ES默认索引副本与分片的问题

## 1. 修改索引模板中的默认值

```bash
curl -X PUT http://localhost:9200/_template/default -H 'Content-Type: application/json' -d '{"index_patterns": ["*"],"order": -1,"settings": {"number_of_shards": "1","number_of_replicas": "0"}}' 

# 或者
PUT /_template/default
{"index_patterns": ["*"],"order": -1,"settings": {"number_of_shards": "1","number_of_replicas": "0"}}
```

## 2. 对于已创建的索引

```bash
curl -X PUT http://localhost:9200/_settings -H 'Content-Type: application/json' -d '{"index": {"number_of_shards": "1","number_of_replicas": "0"}}'
```

会报以下的错误

```
{"error":{"root_cause":[{"type":"cluster_block_exception","reason":"blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"}],"type":"cluster_block_exception","reason":"blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"},"status":403}
```

执行以下命令进行修复

```bash
curl -X PUT http://localhost:9200/_settings -H 'Content-Type: application/json' -d '{"index": {"blocks": {"read_only_allow_delete": "false"}}}'
```

参考：https://gist.github.com/angristan/9d251d853d11f265899b8a4725bff756
