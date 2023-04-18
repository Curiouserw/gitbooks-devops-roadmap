# Elasticsearch索引文档的批量操作API：\_bulk

# 一、简介

[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)

## 1. API请求URL格式

```json
POST /_bulk
{ "index动作": { "_index" : "索引名", "_id" : "文档ID" } }{ "字段名" : "字段值" }
{ "delete动作": { "_index" : "索引名", "_id" : "文档ID" } }
{ "create动作": { "_index" : "索引名", "_id" : "文档ID" } }{ "字段名" : "字段值" }
{ "update动作": { "_index" : "索引名", "_id" : "文档ID" } }{ "doc" : { "字段名" : "字段值" } }
```

```json
POST /索引名/_bulk
{"index":{"_id":"文档ID"}}{ "字段名": "字段值" }
{"index":{"_id":"文档ID"}}{ "字段名": "字段值" }
```

## 2. 支持的文档操作动作

- **`index`**

   如果索引中已经存在具有相同名称的文档，则创建失败，索引将根据需要添加或替换文档

- **`create`**

   如果索引中已经存在具有相同名称的文档，则创建失败，索引将根据需要添加或替换文档

- **`delete`**

   不期望下一行有文档数据。具有与标准delete API相同的语义

- **`update`** 

   期望在下一行中指定部分文档、upsert和脚本及其选项

## 3. 将文档操作数据存储在文本

文本格式

```json
动作及元数据\n
数据\n
动作及元数据\n
数据\n
....
动作及元数据\n
数据\n
```

例如操作数据文本test.json数据如下：

```json
{"index": {"_index": "test", "_type": "_doc", "_id": 1}}
{"doc": {"name": "test1"}}
{"index": {"_index": "test", "_type": "_doc", "_id": 2}}
{"doc": {"name": "test2"}}
========================================================================
{"index":{"_id":"1"}}
{ "name": "test1" }
{"index":{"_id":"2"}}
{ "name": "test2" }
{"index":{"_id":"3"}}
{ "name": "test3" }
```

操作API的Curl命令

```bash
curl -X POST "localhost:9200/_bulk" -H 'Content-Type: application/json' --data-binary @test.json
========================================================================
curl -X POST "localhost:9200/test/_bulk" -H 'Content-Type: application/json' --data-binary @test.json
```

## 4. 注意事项

- 批量操作的响应可能是很大的JSON数据，其中包含执行的每个操作的结果，显示的顺序与请求中出现的操作顺序相同。单个操作的失败不会影响其余操作。 
- 批量操作的响应中没有标识`操作成功`的计数字段

# 二、API请求的参数



# 三、Update动作的参数

- `doc` (partial document)
- `upsert`
- `doc_as_upsert`
- `script`
- `params` (for script)
- `lang` (for script)
- `_source` 

```json

POST _bulk
{ "update" : {"_id" : "1", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"} }
{ "update" : { "_id" : "0", "_index" : "index1", "retry_on_conflict" : 3} }
{ "script" : { "source": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}
{ "update" : {"_id" : "2", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"}, "doc_as_upsert" : true }
{ "update" : {"_id" : "3", "_index" : "index1", "_source" : true} }
{ "doc" : {"field" : "value"} }
{ "update" : {"_id" : "4", "_index" : "index1"} }
{ "doc" : {"field" : "value"}, "_source": true}
```

# 四、操作示例

## 1. 向指定索引批量插入文档

- **Kibana Dev Tools Console**

  ```json
  POST _bulk
  { "index" : { "_index" : "test", "_id" : "1" } }{ "name" : "test1" }
  { "index" : { "_index" : "test", "_id" : "2" } }{ "name" : "test2" }
  { "index" : { "_index" : "test", "_id" : "3" } }{ "name" : "test3" }
  ========================================================================
  POST /test/_bulk
  {"index":{"_id":"1"}}{ "name": "test1" }
  {"index":{"_id":"2"}}{ "name": "test2" }
  {"index":{"_id":"3"}}{ "name": "test3" }
  ```

- **Curl命令**

  ```json
  curl -XPOST "http://localhost:9200/_bulk" \
  -H 'Content-Type: application/json' \
  -d '
  { "index" : { "_index" : "test", "_id" : "1" } }{ "name" : "test1" }
  { "index" : { "_index" : "test", "_id" : "2" } }{ "name" : "test2" }
  { "index" : { "_index" : "test", "_id" : "3" } }{ "name" : "test3" }
  '
  ========================================================================
  curl -XPOST "http://localhost:9200/test/_bulk" \
  -H 'Content-Type: application/json' \
  -d '
  {"index":{"_id":"1"}}{ "name": "test1" }
  {"index":{"_id":"2"}}{ "name": "test2" }
  {"index":{"_id":"3"}}{ "name": "test3" }
  '
  ```

## 2. 针对索引文档进行批量操作

- **Kibana Dev Tools Console**

  ```json
  POST _bulk
  { "index" :  { "_index" : "test", "_id" : "1" } }{ "field1" : "value1" }
  { "delete" : { "_index" : "test", "_id" : "2" } }
  { "create" : { "_index" : "test", "_id" : "3" } }{ "field1" : "value3" }
  { "update" : { "_index" : "test", "_id" : "1" } }{ "doc" : { "field2" : "value2"} }
  ```

- **Curl命令**

  ```json
  curl -X POST "localhost:9200/_bulk?pretty" \
  -H 'Content-Type: application/json' \
  -d '
  { "index" :  { "_index" : "test", "_id" : "1" } } { "field1" : "value1" }
  { "delete" : { "_index" : "test", "_id" : "2" } }
  { "create" : { "_index" : "test", "_id" : "3" } } { "field1" : "value3" }
  { "update" : { "_index" : "test", "_id" : "1" } } { "doc" : {"field2" : "value2"} }
  '
  ```