# ElasticSearch的中文分词器

# 一、简介

elasticsearch官方默认的分词插件，对中文分词效果不理想。

ik 带有两个分词器

- **ik_max_word** ：会将文本做最细粒度的拆分；尽可能多的拆分出词语。比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合；
- **ik_smart**：会做最粗粒度的拆分；已被分出的词语将不会再次被其它词语占有。比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”。

Ik插件Github: https://github.com/medcl/elasticsearch-analysis-ik

# 二、安装

在线安装（所有ES节点都要安装）

```bash
ES_JAVA_OPTS="-Dhttp.proxyHost=100.129.58.13 -Dhttp.proxyPort=3128 -Dhttps.proxyHost=100.129.58.13 -Dhttps.proxyPort=3128" /usr/share/elasticsearch/bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.3.0/elasticsearch-analysis-ik-6.3.0.zip
```

重启ES集群（一个节点一个节点地重启，直到重启的节点上分片已平衡，保证服务不中断）

# 三、分词验证

## 1、创建Index

```bash
curl -XPUT http://localhost:9200/index
```

## 2、创建Mapping时指定分词器

```bash
curl -XPOST http://localhost:9200/index/fulltext/_mapping -H 'Content-Type:application/json' -d'
{
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word"
            }
        }
}'
```

## 3、插入数据Doc

```bash
curl -XPOST http://localhost:9200/index/fulltext/1 -H 'Content-Type:application/json' -d'
{"content":"美国留给伊拉克的是个烂摊子吗"}
'
curl -XPOST http://localhost:9200/index/fulltext/2 -H 'Content-Type:application/json' -d'
{"content":"公安部：各地校车将享最高路权"}
'
curl -XPOST http://localhost:9200/index/fulltext/3 -H 'Content-Type:application/json' -d'
{"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}
'
curl -XPOST http://localhost:9200/index/fulltext/4 -H 'Content-Type:application/json' -d'
{"content":"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}
'
```

## 4、中文关键词查询

```bash
curl -XPOST http://localhost:9200/index/fulltext/_search  -H 'Content-Type:application/json' -d'
{
    "query" : { "match" : { "content" : "中国" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}
'
```

## 5、对比结果

```json
{
    "took": 14,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 2,
        "hits": [
            {
                "_index": "index",
                "_type": "fulltext",
                "_id": "4",
                "_score": 2,
                "_source": {
                    "content": "中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"
                },
                "highlight": {
                    "content": [
                        "<tag1>中国</tag1>驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首 "
                    ]
                }
            },
            {
                "_index": "index",
                "_type": "fulltext",
                "_id": "3",
                "_score": 2,
                "_source": {
                    "content": "中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"
                },
                "highlight": {
                    "content": [
                        "均每天扣1艘<tag1>中国</tag1>渔船 "
                    ]
                }
            }
        ]
    }
}
```

# 四、配置IK分词插件的热词更新

修改 IK 的配置文件`IKAnalyzer.cfg.xml` ：`{conf}/analysis-ik/config/IKAnalyzer.cfg.xml` 或者 `{plugins}/elasticsearch-analysis-ik-*/config/IKAnalyzer.cfg.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>IK Analyzer 扩展配置</comment>
    <!--用户可以在这里配置自己的扩展字典 -->
    <entry key="ext_dict">custom/mydict.dic;custom/single_word_low_freq.dic</entry>
     <!--用户可以在这里配置自己的扩展停止词字典-->
    <entry key="ext_stopwords">custom/ext_stopword.dic</entry>
    <!--用户可以在这里配置远程扩展字典 -->
    <entry key="remote_ext_dict">location</entry>
    <!--用户可以在这里配置远程扩展停止词字典-->
    <entry key="remote_ext_stopwords">http://xxx.com/xxx.dic</entry>
</properties>
```

其中 `location` 是指一个 url，比如 `http://yoursite.com/getCustomDict`，该请求只需满足以下两点即可完成分词热更新。

1. 该 http 请求需要返回两个头部(header)，一个是 `Last-Modified`，一个是 `ETag`，这两者都是字符串类型，只要有一个发生变化，该插件就会去抓取新的分词进而更新词库。
2. 该 http 请求返回的内容格式是一行一个分词，换行符用 `\n` 即可。

满足上面两点要求就可以实现热更新分词了，不需要重启 ES 实例。

可以将需自动更新的热词放在一个 UTF-8 编码的 .txt 文件里，放在 nginx 或其他简易 http server 下，当  .txt 文件修改时，http server 会在客户端请求该文件时自动返回相应的 Last-Modified 和  ETag。可以另外做一个工具来从业务系统提取相关词汇，并更新这个 .txt 文件。