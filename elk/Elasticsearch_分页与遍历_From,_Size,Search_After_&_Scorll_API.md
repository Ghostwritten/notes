

----
## 1. From / Size
默认情况下，查询按照相关度算分排序，返回前 10 条记录
容易理解的分页方案

 - From ： 开始位置
 - Size：期望获取文档的总数
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ecc292f917393523a39db2b2442365e4.png)
from与size设置超过10000会报错
## 2. 分布式系统中深度分页的问题
ES 天生就是分布式，查询信息，但是数据分别保存在多个分片，多台机器，ES 天生就需要满足排序的需要（按照相关性算分）
当一个查询：From = 990 ，Size =10

 - 会在每个分片上先获取 1000 个文档。然后，通过 `Coordinating Node` 聚合所有结果。最后在通过排序选取前 1000个文档
 - 页数越深，占用内容越多。为了避免深度分页带来的内存开销。ES 有个设定，默认限定到 10000 个文档
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b3b26f3b0f109b7eb1ca986ed5e21d4.png)

```bash
POST tmdb/_search
{
  "from": 10000,
  "size": 1,
  "query": {
    "match_all": {}
  }
}
//
```
## 3. Search After 避免深度分页的问题
避免深度分页的性能问题，可以实时获取下一页文档信息

 - 不支持指定页数（From）
 - 不能往下翻

第一步搜索需要指定 `sort`，并且保证值是唯一的（可以通过加入_id 保证唯一性）
然后使用上一次，最后一个文档的 sort 值进行查询

```bash
POST users/_doc
{"name":"user1","age":10}
POST users/_doc
{"name":"user2","age":11}
POST users/_doc
{"name":"user2","age":12}
POST users/_doc
{"name":"user2","age":13}
POST users/_count
POST users/_search
{
  "size": 1,
  "query": {
      "match_all": {}
  },
  "sort": [
      {"age": "desc"} ,
      {"_id": "asc"}    
  ]
}
//返回
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "QlE9AHgBkFqZmZhjeUx_",
        "_score" : null,
        "_source" : {
          "name" : "user2",
          "age" : 13
        },
        "sort" : [
          13,
          "QlE9AHgBkFqZmZhjeUx_"
        ]
      }
    ]
  }
}
POST users/_search
{
  "size": 1,
  "query": {
      "match_all": {}
  },
  "search_after":
       [
          13,
          "QlE9AHgBkFqZmZhjeUx_"
      ],
  "sort": [
      {"age": "desc"} ,
      {"_id": "asc"}    
  ]
}

返回输出
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 6,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "RFE_AHgBkFqZmZhj30zN",
        "_score" : null,
        "_source" : {
          "name" : "user2",
          "age" : 13
        },
        "sort" : [
          13,
          "RFE_AHgBkFqZmZhj30zN"
        ]
      }
    ]
  }
}

```

## 4. Scoll API
创建一个快照，有新的数据写入以后，无法被查找
每次查询后，输入上一次的 `Sroll Id`


```bash
DELETE users
POST users/_doc
{"name":"user1","age":10}
POST users/_doc
{"name":"user2","age":20}
POST users/_doc
{"name":"user3","age":30}
POST users/_doc
{"name":"user4","age":40}

#查询文档个数
POST users/_count
{
  "count" : 4,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  }
}



POST /users/_search?scroll=5m
{
  "size": 1,
  "query": {
      "match_all" : {
      }
  }
}
返回输出：保存_scroll_id
{
  "_scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAXIWdFBMMS1DMklUNi1lVmJ2NUZmRVdsZw==",
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "RlFDAHgBkFqZmZhj3UwF",
        "_score" : 1.0,
        "_source" : {
          "name" : "user1",
          "age" : 10
        }
      }
    ]
  }
}


POST /_search/scroll
{
  "scroll" : "1m",
  "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAXIWdFBMMS1DMklUNi1lVmJ2NUZmRVdsZw=="
}

返回输出：
{
  "_scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAXIWdFBMMS1DMklUNi1lVmJ2NUZmRVdsZw==",
  "took" : 12,
  "timed_out" : false,
  "terminated_early" : true,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "users",
        "_type" : "_doc",
        "_id" : "R1FDAHgBkFqZmZhj40zs",
        "_score" : 1.0,
        "_source" : {
          "name" : "user2",
          "age" : 20
        }
      }
    ]
  }
}

```

## 5. 不同的搜索类型和使用场景

 - Regular ：需要实时获取顶部的部分文档。例如查询最新的订单
 - Scorll ：需要全部文档，例如导出全部数据
 - Pagination ：From 和 Size 如何需要深度分页，则选用 Search After

