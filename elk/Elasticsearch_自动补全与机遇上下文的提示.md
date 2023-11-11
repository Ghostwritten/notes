

---
## 1. The Completion Suggester
Completion Suggester 提供了 “自动完成”（Auto Complete）的功能。用户每输入一个字符，就需要即时发送一个插叙请求到后端查询匹配项

 - 对性能要求比较苛刻。ES 采用了不同的数据结构，并非通过倒排索引来完成。而是将 Analyze 的数据编码成 FST和索引一起存放。FST 会被 ES 整个加载进内容，速度很快
 - FST 只能用于前缀查找

## 2. 使用 Completion Suggester 一些步骤

 - 定义 Mapping，使用 “completion” type
 - 索引数据
 - 运行 “suggest” 查询，得到搜索建议

```bash
DELETE articles
PUT articles
{
  "mappings": {
    "properties": {
      "title_completion":{
        "type": "completion"
      }
    }
  }
}

POST articles/_bulk
{ "index" : { } }
{ "title_completion": "lucene is very cool"}
{ "index" : { } }
{ "title_completion": "Elasticsearch builds on top of lucene"}
{ "index" : { } }
{ "title_completion": "Elasticsearch rocks"}
{ "index" : { } }
{ "title_completion": "elastic is the company behind ELK stack"}
{ "index" : { } }
{ "title_completion": "Elk stack rocks"}
{ "index" : {} }


POST articles/_search?pretty
{
  "size": 0,
  "suggest": {
    "article-suggester": {
      "prefix": "elk ",
      "completion": {
        "field": "title_completion"
      }
    }
  }
}

返回输出：
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "article-suggester" : [
      {
        "text" : "elk ",
        "offset" : 0,
        "length" : 4,
        "options" : [
          {
            "text" : "Elk stack rocks",
            "_index" : "articles",
            "_type" : "_doc",
            "_id" : "ITkIzncBVWBR55DpjKhv",
            "_score" : 1.0,
            "_source" : {
              "title_completion" : "Elk stack rocks"
            }
          }
        ]
      }
    ]
  }
}

```
## 3. 什么是 Context Suggester
Completion Suggester 的扩展
可以在搜索中加入耕读偶读上下文信息，例如，输入 “star”

 - 咖啡相关：starbucks
 - 电影相关：star wars

## 4. 实现 Context Suggester
可以定义两种类型的 Context

 - Category - 任意的字符串
 - Geo - 地理信息位置

实现 Context Suggester

 - 定制一个 Mapping
 - 索引数据，并且为每个文档加入 Conetxt 信息
 - 结合 Context 进行 Suggestion 查询

```bash
DELETE comments
PUT comments
PUT comments/_mapping
{
  "properties": {
    "comment_autocomplete":{
      "type": "completion",
      "contexts":[{
        "type":"category",
        "name":"comment_category"
      }]
    }
  }
}

POST comments/_doc
{
  "comment":"I love the star war movies",
  "comment_autocomplete":{
    "input":["star wars"],
    "contexts":{
      "comment_category":"movies"
    }
  }
}

POST comments/_doc
{
  "comment":"Where can I find a Starbucks",
  "comment_autocomplete":{
    "input":["starbucks"],
    "contexts":{
      "comment_category":"coffee"
    }
  }
}


POST comments/_search
{
  "suggest": {
    "MY_SUGGESTION": {
      "prefix": "sta",
      "completion":{
        "field":"comment_autocomplete",
        "contexts":{
          "comment_category":"coffee"
        }
      }
    }
  }
}

返回输出：
{
  "took" : 958,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "MY_SUGGESTION" : [
      {
        "text" : "sta",
        "offset" : 0,
        "length" : 3,
        "options" : [
          {
            "text" : "starbucks",
            "_index" : "comments",
            "_type" : "_doc",
            "_id" : "4DkSzncBVWBR55DpQ6rd",
            "_score" : 1.0,
            "_source" : {
              "comment" : "Where can I find a Starbucks",
              "comment_autocomplete" : {
                "input" : [
                  "starbucks"
                ],
                "contexts" : {
                  "comment_category" : "coffee"
                }
              }
            },
            "contexts" : {
              "comment_category" : [
                "coffee"
              ]
            }
          }
        ]
      }
    ]
  }
}
```

