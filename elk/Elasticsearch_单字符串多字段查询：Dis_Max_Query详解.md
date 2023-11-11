
## 1. 单字符串查询
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118152837989.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
DEMO

```bash
PUT /blogs/_doc/1
{
  "title": "Quick brown rabbits",
  "body": "Brown rabbits are commonly seen."
}
PUT /blogs/_doc/2
{
  "title": "Keeping pets healthy",
  "body": "My quick brown fox eats rabbits on a regular basis."
}
//查询语句
POST /blogs/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": "Brown fox"
          }
        },
        {
          "match": {
            "body": "Brown fox"
          }
        }
      ]
    }
  }
}
```

```bash
"hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.90425634,
    "hits" : [
      {
        "_index" : "blogs",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.90425634, // 因为2个字段都有brown
        "_source" : {
          "title" : "Quick brown rabbits",
          "body" : "Brown rabbits are commonly seen."
        }
      },
      {
        "_index" : "blogs",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.77041256,
        "_source" : {
          "title" : "Keeping pets healthy",
          "body" : "My quick brown fox eats rabbits on a regular basis."
        }
      }
    ]
  }
```
## 2. 算分过程

 - 查询 should 语句中的两个查询
 - 加和两个查询的评分
 - 乘以匹配语句的总数
 - 除以所有语句的总数

结果


![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118154710116.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 3. Disjunction Max Query 查询
上列中，`title` 和 `body` 相互竞争

 - 不应该将分数简单叠加，而是应该找个单个最佳匹配的字段的评分

Disjunction Max Query

 - 将任何与任一查询匹配的文档作为结果返回。采用字段上最匹配的评分返回

```bash
POST /blogs/_search
{
  "query": {
    "dis_max": {
      "queries": [
        {
          "match": {
            "title": "Quick fox"
          }
        },
        {
          "match": {
            "body": "Quick fox"
          }
        }
      ]
    }
  }
}
```
结果

```bash
{
  "took" : 7,
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
    "max_score" : 1.2199391,
    "hits" : [
      {
        "_index" : "blogs",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.2199391,
        "_source" : {
          "title" : "Keeping pets healthy",
          "body" : "My quick brown fox eats rabbits on a regular basis."
        }
      },
      {
        "_index" : "blogs",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.6931472,
        "_source" : {
          "title" : "Quick brown rabbits",
          "body" : "Brown rabbits are commonly seen."
        }
      }
    ]
  }
}
```
## 4. 最佳字段查询调优
有一些情况下，同时匹配 title 和 body 字段的文档比只与一个字段匹配的文档的相关度更高
但 disjunction max query 查询指挥简单的使用单个最佳匹配语句的评分_scoce 作为整体评分

## 5. 通过 Tie Breaker 参数调整

 - 获得最佳匹配语句的评分
 - 将其他匹配语句的评分 与 tie_breaker 相乘
 - 对以上评分求和并规范化
 - Tie Breanker 是一个介于 0-1 之间的浮点数。**0 代表使用最佳匹配 l;1 代表所有语句同等重要**

```bash
POST blogs/_search
{
"query": {
  "dis_max": {
      "queries": [
          { "match": { "title": "Quick pets" }},
          { "match": { "body":  "Quick pets" }}
      ],
      "tie_breaker": 0.2
  }
}
}
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
[Elasticsearch Query & Filtering 与 多字符串多字段查询详解](https://blog.csdn.net/xixihahalelehehe/article/details/109773574)
