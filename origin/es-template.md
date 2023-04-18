# ElasticSearch模板

# 一、简介

索引模板是预先定义好的在创建新索引时自动应用的模板，主要包括索引设置、映射和模板优先级等配置。





# 二、模板管理

## 1、查看模板

```bash
GET /_template/template_

# 通过模糊匹配得到多个模板信息
GET /_template/template* 

# 批量查看模板
GET /_template/template_1,template_2
```

## 2、创建模板

```
PUT /_template/template_1
{
    "template" : "*",
    "order" : 0,
    "settings" : {
        "number_of_shards" : 1
    },
    "version": 123
}
```

## 3、删除模板

```
DELETE /_template/template_1
```

# 三、模板详解

template大致分成`setting`和`mappings`两部分：

- **settings**：作用于index的一些相关配置信息，如分片数、副本数，tranlog同步条件、refresh等。
- **mappings: **主要是一些说明信息，大致又分为_all、_source、prpperties这三部分：
  - **all**：主要指的是`AllField`字段，我们可以将一个或多个都包含进来，在进行检索时无需指定字段的情况下检索多个字段。设置`“all" : {"enabled" : true}`
  - **source**：主要指的是`SourceField`字段，Source可以理解为ES除了将数据保存在索引文件中，另外还有一份源数据。_source字段在我们进行检索时相当重要，如果在{"enabled" : false}情况下默认检索只会返回ID， 你需要通过Fields字段去到索引中去取数据，效率不是很高。但是enabled设置为true时，索引会比较大，这时可以通过Compress进行压缩和inclueds、excludes来在字段级别上进行一些限制，自定义哪些字段允许存储。
  - **properties**：这是最重要的步骤，主要针对索引结构和字段级别上的一些设置。

  1. 咱们通常在elasticsearch中 post mapping信息，每重新创建索引便到设置mapping，分片，副本信息。非常繁琐。强烈建议大家通过设置template方式设置索引信息。设置索引名，通过正则匹配的方式匹配到相应的模板。



- 直接修改mapping的优先级>索引template。索引匹配了多个template，当属性等配置出现不一致的，以order的最大值为准，order默认值为0
- 多个模板同时匹配，以order顺序倒排，order越大，优先级越高**



# 四、常用操作

## 1、调整主分片个数

对于数据规模较小、索引个数较多的场景，建议调小主分片个数，以减轻索引元数据对堆内存的压力

```
{
 "index_patterns" : ["*"],
   "order" : 2, // 请确保模板中 order 字段的值大于1
   "settings" : {
       "index": {
           "number_of_shards" : 1
       }
   }
}
```

## 2、调整refresh时间

希望让索引的文档在10s之后就能被搜索到，并应用于所有的 `search-*` 索引

```
{
   "index_patterns" : ["search-*"],
   "order" : 2, // 请确保模板中 order 字段的值大于1
   "settings" : {
       "index": {
           "refresh_interval": "10s"
       }
   }
}
```

## 3、调整字段类型

在默认模板中，我们将 string 类型字段动态映射为 keyword 类型，以防止对所有文本类型数据都进行全文索引。您可以根据业务需求，修改指定 string 类型字段为 text，使其可以全文索引。

```
{
 "index_patterns" : ["*"],
   "order" : 2, // 请确保模板中 order 字段的值大于1
   "mappings": {
     "properties": {
       "字段名": {
         "type":  "text"
       }
   }
 }
}
```