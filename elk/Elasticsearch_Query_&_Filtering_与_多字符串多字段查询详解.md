
## 1. Query Context & Filter Context
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118144741841.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

高级搜索的功能，支持多想文本输入，针对多个字段进行搜索
搜索引擎一般也提供时间，价格等条件过滤
在 ES 中，有 Query 和 Filter 两种 Context

 - Query Context ：相关性算分
 - Filter Context ：不需要算分（YES OR NO）, 可以利用 Cache 获得更好的性能
## 2. 条件组合
假设搜索一本电影
 - 评论中包含了 Guitar ，用户打分高于 3 分，同时上映时间在 1993 到 2000 年之间

这个搜索包含了 3 段逻辑，针对不同的字段
 - 评论字段中要包含 Guitar 、用户评论大于 3、上映时间日期在给定范围内

同时包含这三个逻辑，并且有比较好的性能
 - 复合查询： bool Query
## 3. bool 查询
一个 bool 查询，是一个或者多个查询子句的组合
总共包含 4 种子句，其中 2 种会影响算分，2 种不影响算分
相关性并不只是全文本搜索的专利。也适合 yes | no 的子句，匹配的子句越多，相关性评分越高。如果多条查询子句被合并为一条复合查询语句，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118144815247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 4. bool 查询语句
 - 子查询可以任意顺序出现
 - 可以嵌套多个查询
 - 如果你的 bool 查询中，没有 must 条件，should 中必须满足一条查询

```bash
//插入数据
POST /products/_bulk
{"index":{"_id":1}}
{"price":10,"avaliable":true,"date":"2018-01-01","productID":"XHDK-A-1293-#fJ3"}
{"index":{"_id":2}}
{"price":20,"avaliable":true,"date":"2019-01-01","productID":"KDKE-B-9947-#kL5"}
{"index":{"_id":3}}
{"price":30,"avaliable":true,"productID":"JODL-X-1937-#pV7"}
{"index":{"_id":4}}
{"price":30,"avaliable":false,"productID":"QQPX-R-3956-#aD8"}
//查询
POST /products/_search
{
"query": {
  "bool": {
    "must": {
      "term": {
        "price": "30"
      }
    },
    "filter": {
      "term": {
        "avaliable": "true"
      }
    },
    "must_not": {
      "range": {
        "price": {
          "lte": 10
        }
      }
    },
    "should": [
      {
        "term": {
          "productID.keyword": "JODL-X-1937-#pV7"
        }
      },
      {
        "term": {
          "productID.keyword": "XHDK-A-1293-#fJ3"
        }
      }
    ],
    "minimum_should_match": 1
  }
}
}
```
## 5. 如何解决结构化查询 -“包含而不是相等” 的问题
### 5.1 增加 count 字段，使用 bool 查询
从业务角度，按需改进数据模型

```bash
POST /newmovies/_bulk
{"index":{"_id":1}}
{"title":"Father of the Bridge Part II","year":1995,"genre":"Comedy","genre_count":1}
{"index":{"_id":2}}
{"title":"Dave","year":1993,"genre":["Comedy","Romance"],"genre_count":2}
# must  有算分
POST /newmovies/_search
{
"query": {
  "bool": {
    "must": [
      {"term": {"genre.keyword": {"value": "Comedy"}}},
      {"term": {"genre_count": {"value": 1}}}
    ]
  }
}
}
#Filter。不参与算分，结果的score是0
POST /newmovies/_search
{
"query": {
  "bool": {
    "filter": [
      {"term": {"genre.keyword": {"value": "Comedy"}}},
      {"term": {"genre_count": {"value": 1}}}
      ]
  }
}
}
```
## 6 Filter Context - 不影响算分
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118145736604.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 7. Query Context - 影响算分

```bash
POST /products/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "productID.keyword": {
              "value": "JODL-X-1937-#pV7"}}
        },
        {"term": {"avaliable": {"value": true}}
        }
      ]
    }
  }
}
// 算分
"hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.5606477, //算分的
    "hits" : [
      {
        "_index" : "products",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.5606477,
        "_source" : {
          "price" : 30,
          "avaliable" : true,
          "productID" : "JODL-X-1937-#pV7"
        }
      },
      {
        "_index" : "products",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.35667494, 
        "_source" : {
          "price" : 10,
          "avaliable" : true,
          "date" : "2018-01-01",
          "productID" : "XHDK-A-1293-#fJ3"
        }
      },
      {
        "_index" : "products",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.35667494,
        "_source" : {
          "price" : 20,
          "avaliable" : true,
          "date" : "2019-01-01",
          "productID" : "KDKE-B-9947-#kL5"
        }
      }
    ]
  }
```
## 8. bool 嵌套
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111814593618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 9. 查询语句的结构，会对相关度算分产生影响
同一层级下的竞争字段，具有相同的权重
通过嵌套 bool 查询，可以改变对算分的影响
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118150019820.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 10. 控制字段的 Boosting
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118151000382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

## 11. Not Quite Not
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118151043167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 12 Boosting Query
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118151233758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 13 知识点回顾

 - Query Context vs Filter Context
 - Bool Query - 更多的条件组合
 - 查询结构与相关性算分
 - 如何控制查询的精确度
 - Boosting & Boosting Query

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
