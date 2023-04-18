# Elasticsearch的Index API 

# 一、DDL 数据定义(索引的创建与删除)

**数据定义语言：Data Definition Language**

## 1.  创建索引

```bash
PUT /index_name?pretty
========================================================
curl -sk -u username:userpassword -XPUT "http://localhost:9200/index_name?pretty"
```

#### 创建索引时设置参数

```bash
curl -sk -u username:userpassword -XPUT "http://localhost:9200/index_name?pretty" \
-H 'Content-Type: application/json' -d'
{
 "settings" : {
   "number_of_replicas" : 0
  }
}'
```

## 2. 删除Index

```bash
DELET /index_name
========================================================
curl -sk -u username:userpassword -XDELETE "http://127.0.0.1:9200/index_name"
```

# 二、DCL 数据控制(索引的配置)

**数据控制语言：Data Control Language**

## 1. 查看索引的设置

```bash
GET /index_name/_settings
========================================================
curl -sk -u username:userpassword "http://127.0.0.1:9200/index_name/_settings"
```

## 2. 查看索引的Mapping

```bash
GET /index_name/_mapping
========================================================
curl -sk -u username:userpassword "http://127.0.0.1:9200/index_name/_mapping"
```

## 3. 设置索引Mapping

```bash
PUT /index_name
{
  "mappings": {
    "index_name": {
      "dynamic":"false",
      "properties": {
        "id": {
            "type": "long"
        },
        "prd_id": {
           "type": "long"
        },
        "mer_id": {
          "type": "long"
        },
        "data_status": {
          "type": "text"
        },
        "datachange_createtime": {
          "type": "date",
          "format": "strict_date_optional_time||epoch_millis"
        },
        "datachange_lasttime": {
          "type": "date",
          "format": "strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}
========================================================
curl -sk -u username:userpassword -XPUT "http://127.0.0.1:9200/index_name" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "index_name": {
      "dynamic":"false",
      "properties": {
        "id": {
            "type": "long"
        },
        "prd_id": {
           "type": "long"
        },
        "mer_id": {
          "type": "long"
        },
        "data_status": {
          "type": "text"
        },
        "datachange_createtime": {
          "type": "date",
          "format": "strict_date_optional_time||epoch_millis"
        },
        "datachange_lasttime": {
          "type": "date",
          "format": "strict_date_optional_time||epoch_millis"
        }
      }
    }
  }
}'
```



# 三、DML 数据操作(文档的增、删、改)

**数据操作语言：Data Manipulation Language**

## 1. 向索引中插入一个文档

向索引中插入一个ID为1的文档

```bash
PUT /index_name/_doc/1 
{
  "name": "test"
} 
========================================================
curl -sk -u username:userpassword \
-XPUT "http://localhost:9200/index_name/_doc/1" \
-H 'Content-Type: application/json' \
-d'{  "name": "test" }'
```

## 2. 向索引中批量插入文档

详见[Elasticsearch索引文档批量操作](../origin/elasticsearch-bulk-api.md)

## 3. 更新指定文档

```bash
PUT /index_name/_doc/1?pretty
{
  "doc": {"name": "test1"}
}
========================================================
curl -sk -u username:userpassword \
-XPUT "http://localhost:9200/index_name/_doc/1" \
-H 'Content-Type: application/json' \
-d '
{
  "doc": {"name": "test1"}
}
'
```

## 4. 指定文档新增字段

```bash
PUT /index_name/_doc/1?pretty
{
  "doc": {"name": "test1"，"new_field": "testN"}
}
========================================================
curl -sk -u username:userpassword \
-XPUT "http://localhost:9200/index_name/_doc/1" \
-H 'Content-Type: application/json' \
-d '
{
  "doc": {"name": "test1"，"new_field": "testN"}
}
'
```

## 5. 删除文档

```bash
curl -sk -u username:userpassword \
-XPOST "http://localhost:9200/index_name/_delete_by_query" \
-H 'Content-Type: application/json' \
-d '
{
  "query":{
    "term":{
        "name" : "test1"
    }
  }
}'
```



# 三、**DQL 数据查询** (文档的查询)

**数据查询语言：Data Query Language**

## 1. 查询索引中的所有文档

只显示前十条

```bash
GET /index_name/_search?pretty 
========================================================
curl -sk -u username:userpassword "http://localhost:9200/index_name/_search?pretty"
```

## 2. 查询_id为1的文档

```bash
GET /index_name/_doc/1?pretty
========================================================
curl -sk -u username:userpassword "http://localhost:9200/index_name/_doc/1?pretty"
```

## 3. 查询_id为1的文档的元数据

```bash
GET index_name/_doc/1/_source
========================================================
curl -sk -u username:userpassword "http://localhost:9200/index_name/_doc/1/_source?pretty"
```

## 4. 查询符合指定条件的文档

```bash
GET /index_name/_search?q=name:test1
========================================================
curl -sk -u username:userpassword "http://localhost:9200/index_name/_search?q=name:test1"
```

## 5. 复杂查询 

```json
GET /employee/_doc/_search   
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith"
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 }
                }
            }
        }
    }
}
#这条语句翻译成sql：select * from employee where last_name='smith' and age > 40
```







