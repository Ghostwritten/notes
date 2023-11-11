
## 1. 结构化数据
1.结构化搜索（Structured search） 是指对结构化数据的搜索

 - 日期，布尔类型和数字都是结构化

2.文本也可以是结构化的

 - 如彩色笔可以有离散的颜色集合：红（red）、绿（green）、蓝（blue）
 - 一个博客可能被标记了标签，例如，分布式（distributed）和搜索（search）
 - 电商网站上的商品都有 UPCs（通用产品码 Universal Product Codes）或其他的唯一标识，它们都遵从严格规定的、结构化的格式

## 2. ES 中的机构化搜索

 - `布尔、时间，日期和数字`这类结构化数据：有精确的格式，我们可以对这些格式进行逻辑操作。包括比较数字或时间的范围，或判断两个值的大小
 - 结构化的文本可以做到`精确匹配`或者`部分匹配`
`Term` 查询 / `Prefix 前缀`查询
 - 结构化结构只有 “是” 或 “否” 两个值
根据场景需要，可以决定结构化搜索是否需要打分

## 3. Demo

```bash
DELETE products
POST /products/_bulk
{"index":{"_id":1}}
{"price":10,"avaliable":true,"date":"2018-01-01","productID":"XHDK-A-1293-#fJ3"}
{"index":{"_id":2}}
{"price":20,"avaliable":true,"date":"2019-01-01","productID":"KDKE-B-9947-#kL5"}
{"index":{"_id":3}}
{"price":30,"avaliable":true,"productID":"JODL-X-1937-#pV7"}
{"index":{"_id":4}}
{"price":30,"avaliable":false,"productID":"QQPX-R-3956-#aD8"}

#查看mapping
GET products/_mapping
{
  "products" : {
    "mappings" : {
      "properties" : {
        "avaliable" : {
          "type" : "boolean"
        },
        "date" : {
          "type" : "date"
        },
        "price" : {
          "type" : "long"
        },
        "productID" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```
#### 3.1 对布尔值 match 查询，有算分

```bash
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "avaliable": true
    }
  }
}
```
#### 3.2 对布尔值，通过 constant score 转成 filtering，没有算分

```bash
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "avaliable": true
        }
      },
      "boost": 1.2
    }
  }
}
```
#### 3.3 数字类型 Term

```bash
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "price": 30
        }
      },
      "boost": 1.2
    }
  }
}
```
#### 3.4 数字类型 terms

```bash
POST products/_search
{
  "profile": "true",
  "explain": true,  //explain 返回文档的评分解释
  "query": {
    "constant_score": {
      "filter": {
        "terms": {
          "price": [
              "20",
              "30"
            ]
        }
      }
    }
  }
}
```
#### 3.5 数字 Range 查询

```bash
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "price": {
             "gte": 20,
             "lte":30
          }
        }
      }
    }
  }
}
```

 - gt 大于
 - lt 小于
 - gte 大于等于
 - lte 小于等于
#### 3.6 日期 range

```bash
POST products/_search
{
"query": {
  "constant_score": {
    "filter": {
      "range": {
        "date": {
           "gte": "now-1y"  //当前时间减1天
        }
      }
    }
  }
}
}
```
Date Match Expressions

```bash
2014-01-01 00:00:00 || +1M
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201113165423787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
#### 3.7 exists 查询 - 非空查询

```bash
POST products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "exists": {
                    "field":"date"
                }
            }
        }
    }
}
```
#### 3.8 字符类型 terms

```bash
POST products/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "terms": {
          "productID.keyword": [
            "QQPX-R-3956-#aD8",
            "JODL-X-1937-#pV7"
          ]
        }
      }
    }
  }
}
```
#### 3.9 处理多值字段

```bash
#demo
POST /movies/_bulk
{"index":{"_id":1}}
{"title":"Father of the Bridge Part II","year":1995,"genre":"Comedy"}
{"index":{"_id":2}}
{"title":"Dave","year":1993,"genre":["Comedy","Romance"]}
```
#### 4.0 处理多值字段，term 查询是包含，而不是等于

```bash
//返回2条数据
POST movies/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "genre.keyword": "Comedy"
        }
      }
    }
  }
}
```
#### 4.1 Match 跟 term 对比

```bash
POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "term": {
      "date": "2019-01-01"
    }
  }
}

POST products/_search
{
  "profile": "true",
  "explain": true,
  "query": {
    "match": {
      "date": "2019-01-01"
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
