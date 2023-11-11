
## 1. 三种场景

 - **最佳字段**（Best Fields） 
 当字段之间相互竞争，又相互关联。例如 title 和 body 这样的字段，评分来自最匹配字段
 - **多数字段**（Most Fields） 
 处理英文内容时：一种常见的手段是，在主字段（English
   Analyzer），抽取词干，加入同义词，以匹配更多的文档。相同的文本，加入子字段（Standard Analyzer），以提供更加精确的匹配。其他字段作为匹配文档提高性相关度的信号。匹配字段越多越好
 - **混合字段**（Cross Field）
   对于某些实体，例如人名，地址，图书信息。需要在多个字段中确定信息，单个字段只能作为整体的一部分。希望在任何这些列出的字段中尽可能找出多的词
## 2. Multi Match Query
Best Fields 是默认类型，可不指定
`Minimum should match` 等参数可以传递到生成的 query 中

```bash
POST blogs/_search
{
  "query": {
    "multi_match": {
      "type": "best_fields",
      "query": "Quick pets",
      "fields": ["title","body"],
      "tie_breaker": 0.2,
      "minimum_should_match": "20%"
    }
  }
}
```
## 3. 查询案例

```bash
PUT /titles
{
  "mappings": {
    "properties": {
      "title":{
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}

POST titles/_bulk
{"index":{"_id":1}}
{"title":"My dog barks"}
{"index":{"_id":2}}
{"title":"I see a lot of barking dogs on the road "}

GET titles/_search
{
  "query": {
    "match": {
      "title": "barking dogs"
    }
  }
}
//结果 因为是english 分词 ，且短 则 id 排第一个
"hits" : [
      {
        "_index" : "titles",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.24399278,
        "_source" : {
          "title" : "My dog barks"
        }
      },
      {
        "_index" : "titles",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.1854345,
        "_source" : {
          "title" : "I see a lot of barking dogs on the road "
        }
      }
    ]
```
## 4. 重新设置 mapping

```bash
DELETE titles

PUT /titles
{
  "mappings": {
    "properties": {
      "title":{
        "type": "text",
        "analyzer": "english",
        "fields": {
          "std":{
            "type":"text",
            "analyzer":"standard"
          }
        }
      }
    }
  }
}
POST titles/_bulk
{"index":{"_id":1}}
{"title":"My dog barks"}
{"index":{"_id":2}}
{"title":"I see a lot of barking dogs on the road "}
//multi_match 查询
GET titles/_search
{
  "query": {
    "multi_match": {
      "query": "barking dogs",
      "type": "most_fields", //默认是best_fields
      "fields": ["title","title.std"]//累计叠加
    }
  }
}
//返回
"hits" : [
      {
        "_index" : "titles",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.4569323,
        "_source" : {
          "title" : "I see a lot of barking dogs on the road "
        }
      },
      {
        "_index" : "titles",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.42221838,
        "_source" : {
          "title" : "My dog barks"
        }
      }
    ]
```
## 5. 使用多字段匹配解决

 - 用广度匹配字段 title 包括尽可能多的文档 - 以提高召回率 ，同时又使用字段 title.std
   作为信息将相关度更高的文档结至于文档顶部
 - 每个字段对于最终评分的贡献可以通过自定义值 boost 来控制。比如，使 title 字段更为重要，这样同时也降低了其他信号字段的作用

```bash
GET titles/_search
{
  "query": {
    "multi_match": {
      "query": "barking dogs",
      "type": "most_fields", 
      "fields": ["title^10","title.std"]
    }
  }
}
```
结果

```bash
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 4.344906,
    "hits" : [
      {
        "_index" : "titles",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 4.344906,
        "_source" : {
          "title" : "I see a lot of barking dogs on the road "
        }
      },
      {
        "_index" : "titles",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 4.2221837,
        "_source" : {
          "title" : "My dog barks"
        }
      }
    ]
  }
}
```
## 6. 跨字段搜索

 - `most_fields` 无法使用 opeartor
 - 可以用 `copy_to` 解决，但是需要额外的储存空间
 - `cross_fields` 可以支持 operator
 - 与 `copy_to` 相比，其中一个优势就是可以在搜索时为某个字段提升权重

```bash
PUT address/_doc/1
{
  "street":"5 Poland Street",
  "city" : "Lodon",
  "country":"United Kingdom",
  "postcode" : "W1V 3DG"
}

POST address/_search
{
  "query":{
    "multi_match": {
      "query": "Poland Street W1V",
      "type": "cross_fields",  //most_fields查询为空
      "operator": "and", 
      "fields": ["street","city","country","postcode"]
    }
  }
}
"hits" : [
      {
        "_index" : "address",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.8630463,
        "_source" : {
          "street" : "5 Poland Street",
          "city" : "Lodon",
          "country" : "United Kingdom",
          "postcode" : "W1V 3DG"
        }
      }
    ]
```
参考资料：
极客时间：Elasticsearch核心技术与实战
相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
[docker-compose安装elasticsearch集群](https://blog.csdn.net/xixihahalelehehe/article/details/109389780)
[Elasticsearch 7.X之文档、索引、REST API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406518)
[Elasticsearch节点，集群，分片及副本详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406875)
[Elasticsearch倒排索引介绍](https://blog.csdn.net/xixihahalelehehe/article/details/109440345)
[Elasticsearch Analyzer 进行分词详解](https://blog.csdn.net/xixihahalelehehe/article/details/109447777)
[Elasticsearch search API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109449425)
[Elasticsearch URI Search 查询方法详解](https://blog.csdn.net/xixihahalelehehe/article/details/109453253)
[Elasticsearch Request Body 与 Query DSL详解](https://blog.csdn.net/xixihahalelehehe/article/details/109458983)
[Elasticsearch Dynamic Mapping 和常见字段类型详解](https://blog.csdn.net/xixihahalelehehe/article/details/109463294)
[Eelasticsearch 多字段特性及 Mapping 中配置自定义 Analyzer详解](https://blog.csdn.net/xixihahalelehehe/article/details/109476672)
[Elasticsearch index template与dynamic template详解](https://blog.csdn.net/xixihahalelehehe/article/details/109595303)
[Elasticsearch 聚合分析简介](https://blog.csdn.net/xixihahalelehehe/article/details/109625376)
[Elasticsearch 第一阶段总结与测试](https://blog.csdn.net/xixihahalelehehe/article/details/109626903)
[Elasticsearch 基于词项和基于全文的搜索详解](https://blog.csdn.net/xixihahalelehehe/article/details/109669976)
[Elasticsearch 结构化搜索详解](https://blog.csdn.net/xixihahalelehehe/article/details/109674952)
[Elasticsearch 搜索的相关性算分详解](https://blog.csdn.net/xixihahalelehehe/article/details/109720759)
