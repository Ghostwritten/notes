

---
## 1. Request Body Search（query.match_all）

 - 将查询语句通过 HTTP Request Body 发送给 Elasticsearch
 - Query DSL

```bash
POST /movies,404_idx/_search?ignore_unavailable=true
{
  "profile": "true",
  "query": {
    "match_all": {}
  }
}
//返回全部
```
空查询（empty search） —{}— 在功能上等价于使用 match_all 查询， 正如其名字一样，匹配所有文档。在没有指定查询方式时，它是默认的查询：`. { "match_all": {}}`. 拷贝为cURL在Sense 中查看. 它经常与`filter` 结合使用
### 1.1 分页

 - From 从 0 开始 默认返回 10 个结果
 - 获取靠后的翻页，成本较高

```bash
POST /kibana_sample_data_ecommerce/_search
{
  "from":10,
  "size":20,
  "query": {
    "match_all":{}
  }
}
```
### 1.2 排序

 - 最好在 “数字型” 与 “日期型” 字段上排序
 - 因为对于多值类型或分析过的字段排序，系统会选一个值，无法得知该值

```bash
POST /kibana_sample_data_ecommerce/_search
{
  "sort":[{"order_date":"desc"}],
  "from":10,
  "size":20,
  "query": {
    "match_all":{}
  }
}
//返回10个并按照日期排序
```

### 1.3  _source 过滤

 - 如果_source 没有存储，那就只返回匹配的文档元数据
 - _source 支持使用通配符
 - _source[“name* “,”desc*”]

```bash
GET /kibana_sample_data_ecommerce/_search
{
  "_source": ["order_date","order_date","category.keyword"],
  "from":10,
  "size":20,
  "query": {
    "match_all":{}
  }
}
//返回_source含有"order_date","order_date","category.keyword"的值
例如
{
  "took" : 13,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4675,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "kibana_sample_data_ecommerce",
        "_type" : "_doc",
        "_id" : "EFGvgnUBh8v6HnS077dk",
        "_score" : 1.0,
        "_source" : {
          "order_date" : "2020-10-30T16:29:17+00:00"
        }
      },
```
### 1.4 脚本字段

 - **能算出新的字段**
 - **用例：订单中有不同的汇率，需要结合汇率对订单价格进行排序**
 

```bash
 #脚本字段
GET kibana_sample_data_ecommerce/_search
{
  "script_fields": {
    "new_field": {
      "script": {
        "lang": "painless",
        "source": "doc['order_date'].value+'hello'"
      }
    }
  },
  "query": {
    "match_all": {}
  }
}
```
### 1.5 使用查询表达式 - Match（query.match）

```bash
POST movies/_search
{
  "query": {
    "match": {
      "title": "Last Christmas"  // 相当于 OR 可出现其中1个
    }
  }
}

POST movies/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Last Christmas",
        "operator": "AND"             //相当于 and 只能两个都出现
      }
    }
  }
}
```
### 1.6 短语搜索 - Match Phrase（query.match_phrase）

```bash
POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love"

      }
    }
  }
}
返回0

POST movies/_search
{
  "query": {
    "match_phrase": {
      "title":{
        "query": "one love",
        "slop":1 
      }
    }
  }
}
返回1个
```
### 1.7 query.query_string

 - 类似 URI Query – 把查询条件放在 POST 里面

```bash
//准备工作
  PUT /users/_doc/3
  {
    "name" : "Li Sunke",
    "about": "php,elasticsearch,redis,nginx,swoole"
  }

  PUT /users/_doc/4
  {
    "name" : "Qu Sunke",
    "about": "mysql,php"
  }
  //query_string
  POST /users/_search
  {
    "query": {
      "query_string": {
        "query": "Li AND Sunke",
        "default_field": "name"
      }
    }
  }
  //返回1个结果
  "hits" : [
        {
          "_index" : "users",
          "_type" : "_doc",
          "_id" : "3",
          "_score" : 0.87546873,
          "_source" : {
            "name" : "Li Sunke",
            "about" : "php,elasticsearch,redis,nginx,swoole"
          }
        }
      ]
  //query string 支持分组查询多个字段
  // 返回 doc_3
  POST /users/_search
  {
    "query": {
      "query_string": {
        "fields": ["name","about"],
        "query": "(Li AND Sunke) OR (php AND nginx)"
      }
    }
  }
  返回一个结果
  "hits" : [
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 1.9899296,
        "_source" : {
          "name" : "Li Sunke",
          "about" : "php,elasticsearch,redis,nginx,swoole"
        }
      }
    ]
```
### 1.8 query.simple_query_string

 - 类似 Query String , 但是会忽略错误的语法同时只支持部分查询语句
 - 不支持 AND OR NOT , 但会当作字符串处理
 - Term 之间默认的关系是 OR, 可以指定 Operator
 - 支持 部分逻辑

- **·替代 AND**
- **| 替代 OR**
- **-替代 NOT**

```bash
// Simple Query 默认的operator 是 Or
// AND 会当做一个字符串,所以会 3个字段OR查询

POST /users/_search
{
"query": {
 "simple_query_string": {
   "query": "Li  AND Sunke", 
   "fields": ["name"],
 }
}
}
//返回两个

POST /users/_search
{
"query": {
 "simple_query_string": {
   "query": "Li  AND Sunke", 
   "fields": ["name"],
   "default_operator": "AND"
 }
}
}
//返回0个



POST /users/_search
{
"query": {
"simple_query_string": {
"query": "Li Sunke",
"fields": ["name"],
"default_operator": "AND"
}
}
}
//返回1个
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
[elasticsearch Analyzer 进行分词详解](https://blog.csdn.net/xixihahalelehehe/article/details/109447777)
[elasticsearch search API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109449425)
[Elasticsearch URI Search 查询方法详解](https://blog.csdn.net/xixihahalelehehe/article/details/109453253)
