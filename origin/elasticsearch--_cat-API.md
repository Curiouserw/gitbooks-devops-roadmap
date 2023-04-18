# Elasticsearch _cat APIs

官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/cat.html

## 查看_cat API支持的所有Endpoint

```json
GET /_cat
curl -XGET http://127.0.0.1:9200/_cat
```

```json
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/tasks
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/thread_pool/{thread_pools}
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
/_cat/nodeattrs
/_cat/repositories
/_cat/snapshots/{repository}
/_cat/templates
```
## 查询Endpoint参数

    GET /_cat/health?help
    curl -XGET "http://127.0.0.1:9200/_cat/health?help"


    # 参数全称             | 参数缩写    							   | 参数详解
    ----------------------------------------------------------------------------------------------------
    epoch                 | t,time                                   | seconds since 1970-01-01 00:00:00  
    timestamp             | ts,hms,hhmmss                            | time in HH:MM:SS                   
    cluster               | cl                                       | cluster name                       
    status                | st                                       | health status                      
    node.total            | nt,nodeTotal                             | total number of nodes              
    node.data             | nd,nodeData                              | number of nodes that can store data
    shards                | t,sh,shards.total,shardsTotal            | total number of shards             
    pri                   | p,shards.primary,shardsPrimary           | number of primary shards           
    relo                  | r,shards.relocating,shardsRelocating     | number of relocating nodes         
    init                  | i,shards.initializing,shardsInitializing | number of initializing nodes       
    unassign              | u,shards.unassigned,shardsUnassigned     | number of unassigned shards        
    pending_tasks         | pt,pendingTasks                          | number of pending tasks            
    max_task_wait_time    | mtwt,maxTaskWaitTime                     | wait time of longest task pending  
    active_shards_percent | asp,activeShardsPercent                  | active number of shards in percent 

## 使用参数控制查询条件

    GET /_cat/health?h=st,t
    #带表头
    GET /_cat/health?v&h=st,t

## 控制查询的输出排序

```bash
GET _cat/indices?v&h=index,store.size,creation.date&s=store.size:desc,creation.date
# 查询出来的Index将会以store.size的大小降序输出。只输出Index名，store.size大小，创建时间戳

curl -XGET "http://elasticsearch-service.logger.svc:9200/_cat/indices?v&h=index,store.size,creation.date&s=store.size:desc,creation.date"
```

## 控制查询的输出格式

    GET _cat/indices?v&h=index,store.size,creation.date&s=store.size:desc,creation.date&format=yaml

* yaml
    ```bash
    - index: "test-test-2019.05.21"
      store.size: "4.1gb"
      creation.date: "1558432572904"
    - index: ".monitoring-es-7-2019.06.17"
      store.size: "1.2gb"
      creation.date: "1560729605158"
    ```
* json
    ```bash
    [
    {
        "index" : "test-test-2019.05.21",
        "store.size" : "4.1gb",
        "creation.date" : "1558432572904"
    },
    {
        "index" : ".monitoring-es-7-2019.06.17",
        "store.size" : "1.2gb",
        "creation.date" : "1560729605158"
    }
    ]
    ```
* text (default)
    ```bash
    index                           store.size creation.date
    test-test-2019.05.21                 4.1gb 1558432572904
    .monitoring-es-7-2019.06.17          1.2gb 1560729605158
    ```
* cbor
* smile