

---
## 1. 什么是搜索建议

 - 现代的搜索引擎，一般都会提供 Suggest as you type 的功能
 - 帮助用户在输入搜索的过程中，进行自动补全或者纠错。通过协助用户输入更加精准的关键词，提高后续搜索阶段文档匹配的程度
 - 在 google 上搜索，一开始会自动补全。当输入到一定长度，如因为单词拼写错误无法补全，就会开始提示相似的词或者句子、

## 2. Elasticsearch Suggester API
搜索引擎中类似的功能，在 ES 中通过 `Sugester API` 实现的
原理：将输入的文档分解为 Token，然后在索引的字段里查找相似的 Term 并返回
根据不同的使用场景，ES 设计了 4 种类别的 Suggesters

 - `Term & Phrase Suggester`
 - `Complete & Context Suggester`

一般搜索：
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
        "field": "title_completion"  //查询 title_completion 字段以 elk 开头的所有文档
      }
    }
  }
}

返回输出：
{
  "took" : 996,
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
            "_id" : "cTmozXcBVWBR55DpWY-3",
            "_score" : 1.0,
            "_source" : {
              "title_completion" : "Elk stack rocks"
            }
          },
          {
            "text" : "Elk stack rocks",
            "_index" : "articles",
            "_type" : "_doc",
            "_id" : "KjmrzXcBVWBR55DpHJCd",
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

## 3. Term Suggester
 - Suggester 就是一种特殊类型的搜索。“text” 里是调用时候提供的文本，通常来自用户界面上用户输入的内容
 - 用户输入的 “lucen” 是一个错误的拼写
 - 会到 指定的字段 “body” 上搜索，当无法搜索到结果时（missing），返回建议的词



搜索 “l`ucen rock`”：
每个建议都包含了一个算分，相似性是通过 Levenshtein Edit Distance 的算法实现的。核心思想就是一个词改动多少字段就可以和另外一个词一致。提供了很多可选参数来控制相似性的模糊程度。
几种 Suggestion Mode

 - Missing - 如索引中已存在，就不提供建议
 - Popular - 推荐出现频率更加高的词
 - Always - 无论是否存在，都提供建议

### 3.1 missing Mode

```bash
DELETE articles

POST articles/_bulk
{ "index" : { } }
{ "body": "lucene is very cool"}
{ "index" : { } }
{ "body": "Elasticsearch builds on top of lucene"}
{ "index" : { } }
{ "body": "Elasticsearch rocks"}
{ "index" : { } }
{ "body": "elastic is the company behind ELK stack"}
{ "index" : { } }
{ "body": "Elk stack rocks"}
{ "index" : {} }
{  "body": "elasticsearch is rock solid"}


POST _analyze
{
  "analyzer": "standard",
  "text": ["Elk stack  rocks rock"]
}
返回输出：
{
  "tokens" : [
    {
      "token" : "elk",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "stack",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "rocks",
      "start_offset" : 11,
      "end_offset" : 16,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "rock",
      "start_offset" : 17,
      "end_offset" : 21,
      "type" : "<ALPHANUM>",
      "position" : 3
    }
  ]
}

--
POST /articles/_search
{
  "size": 1,
  "query": {
    "match": {
      "body": "lucen rock"
    }
  },
  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "missing",
        "field": "body"
      }
    }
  }
}

返回输出：
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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.5904956,
    "hits" : [
      {
        "_index" : "articles",
        "_type" : "_doc",
        "_id" : "UDmvzXcBVWBR55DpdZHm",
        "_score" : 1.5904956,
        "_source" : {
          "body" : "elasticsearch is rock solid"
        }
      }
    ]
  },
  "suggest" : {
    "term-suggestion" : [
      {
        "text" : "lucen",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "lucene",
            "score" : 0.8,
            "freq" : 2
          }
        ]
      },
      {
        "text" : "rock",
        "offset" : 6,
        "length" : 4,
        "options" : [ ]
      }
    ]
  }
}


```

### 3.2 popular mode
```bash
POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "popular",
        "field": "body"
      }
    }
  }
}
返回输出：

{
  "took" : 16,
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
    "term-suggestion" : [
      {
        "text" : "lucen",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "lucene",
            "score" : 0.8,
            "freq" : 2
          }
        ]
      },
      {
        "text" : "rock",
        "offset" : 6,
        "length" : 4,
        "options" : [
          {
            "text" : "rocks",
            "score" : 0.75,
            "freq" : 2
          }
        ]
      }
    ]
  }
}
```

### 3.3 always mode

```bash
POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "always",
        "field": "body",
      }
    }
  }
}
返回输出：
{
  "took" : 4,
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
    "term-suggestion" : [
      {
        "text" : "lucen",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "lucene",
            "score" : 0.8,
            "freq" : 2
          }
        ]
      },
      {
        "text" : "rock",
        "offset" : 6,
        "length" : 4,
        "options" : [
          {
            "text" : "rocks",
            "score" : 0.75,
            "freq" : 2
          }
        ]
      }
    ]
  }
}

```

```bash
POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen hocks",
      "term": {
        "suggest_mode": "always",
        "field": "body",
        "prefix_length":0,
        "sort": "frequency"
      }
    }
  }
}

返回输出：
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
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "suggest" : {
    "term-suggestion" : [
      {
        "text" : "lucen",
        "offset" : 0,
        "length" : 5,
        "options" : [
          {
            "text" : "lucene",
            "score" : 0.8,
            "freq" : 2
          }
        ]
      },
      {
        "text" : "hocks",
        "offset" : 6,
        "length" : 5,
        "options" : [
          {
            "text" : "rocks",
            "score" : 0.8,
            "freq" : 2
          }
        ]
      }
    ]
  }
}

```
## 4. Phrase Suggester
Phrase Suggesetr 上增加了一些额外的逻辑
一些参数

 - Suggeset Mode ： missing,popular ,always
 - Max Errors: 最多可以拼错的 Terms 数
 - Condfidence ： 限制返回结果数，默认为 1


```bash
POST /articles/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "lucne and elasticsear rock hello world ",
      "phrase": {
        "field": "body",
        "max_errors":2,
        "confidence":0,
        "direct_generator":[{
          "field":"body",
          "suggest_mode":"always"
        }],
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}


返回输出：
{
  "took" : 47,
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
    "my-suggestion" : [
      {
        "text" : "lucne and elasticsear rock hello world ",
        "offset" : 0,
        "length" : 39,
        "options" : [
          {
            "text" : "lucene and elasticsearch rock hello world",
            "highlighted" : "<em>lucene</em> and <em>elasticsearch</em> rock hello world",
            "score" : 1.5788074E-4
          },
          {
            "text" : "lucne and elasticsearch rocks hello world",
            "highlighted" : "lucne and <em>elasticsearch rocks</em> hello world",
            "score" : 1.136111E-4
          },
          {
            "text" : "lucne and elasticsearch rock hello world",
            "highlighted" : "lucne and <em>elasticsearch</em> rock hello world",
            "score" : 1.05567684E-4
          },
          {
            "text" : "lucene and elasticsear rocks hello world",
            "highlighted" : "<em>lucene</em> and elasticsear <em>rocks</em> hello world",
            "score" : 9.929376E-5
          },
          {
            "text" : "lucene and elasticsear rock hello world",
            "highlighted" : "<em>lucene</em> and elasticsear rock hello world",
            "score" : 9.2263974E-5
          }
        ]
      }
    ]
  }
}
```

