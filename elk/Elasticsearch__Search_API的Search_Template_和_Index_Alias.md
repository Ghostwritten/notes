

----
## 1. 为什么需要索引模板？
在实际工作中针对一批大量数据存储的时候需要使用多个索引库，如果手工指定每个索引库的配置信息(settings和mappings)的话就很麻烦了。

所以，这个时候，就存在创建索引模板的必要了
**索引可使用预定义的模板进行创建,这个模板称作Index templates。模板设置包括settings和mappings，通过模式匹配的方式使得多个索引重用一个模板。**
[官网elasticsearch template](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/indices-templates.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/5.6/search-template.html](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/search-template.html)
## 2. 索引别名的应用场景
比如，公司使用es收集应用的运行日志，每个星期创建一个索引库，这样时间长了就会创建很多的索引库，操作和管理的时候很不方便。由于新增索引数据只会操作当前这个星期的索引库，所以就创建了两个别名。

 - curr_week：这个别名指向这个星期的索引库，新增数据操作这个索引库。
 - last_3_month：这个别名指向最近三个月的所有索引库，因为我们的需求是查询最近三个月的日志信息。

后期只需要修改这两个别名和索引库之间的指向关系即可。应用层代码不需要任何改动。
还要把三个月以前的索引库close掉，留存最近一年的日志数据，一年以前的数据删除掉。说明：可以类似，指定多个索引库查询。定义一个索引别名，如zhouls_all，将索引zhouls1映射到这个别名上，把索引zhouls2，把索引zhoulsn，也映射到这个别名上。
那么，在通过别名来查询时，直接同查询别名zhouls_all，就可以把对应所有的索引zhouls,1,2,...n都一次性查询完了。但是，如果你是具体要插入和操作数据，则，就不方便使用别名了。而是具体到某个索引zhoulsn了。

在学习Search Template之前，我们需要先掌握[mustache模板语法](https://ghostwritten.blog.csdn.net/article/details/113951240)，因为在ES中默认使用mustache语言来定义模板。使用它而不是内置的 painless 脚本方式，是因为他有更好的模板处理能力，例如 {{#tojson}}XXX{{/tojson}} 的自动扩展。


## 3. Search Template （查询模板）

 - Elasticsearch 的查询语句: 对相关性算分 / 查询性能都至关重要
 - 在开发初期，虽然可以明确查询参数，但是往往还不能最终定义查询的 DSL 的具体结构 通过 Search Template 定义一个Contract
 - 各司其职，解耦 开发人员 / 搜索工程师 / 性能工程师
### 3.1 创建自定义模块
```bash
POST _scripts/tmdb
{
  "script": {
    "lang": "mustache",  #一种使用模板的语言
    "source": {
      "_source": [
        "title","overview"
      ],
      "size": 20,
      "query": {
        "multi_match": {
          "query": "{{q}}",
          "fields": ["title","overview"]
        }
      }
    }
  }
}

```

### 3.2 查询
```bash
GET _scripts/tmdb

POST tmdb/_search/template
{
    "id":"tmdb",
    "params": {
        "q": "basketball with cartoon aliens"
    }
}
输出：
{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 84,
      "relation" : "eq"
    },
    "max_score" : 12.311834,
    "hits" : [
      {
        "_index" : "tmdb",
        "_type" : "_doc",
        "_id" : "2300",
        "_score" : 12.311834,
        "_source" : {
          "overview" : "Michael Jordan agrees to help the Looney Tunes play a basketball game against alien slavers to determine their freedom.",
          "title" : "Space Jam"
        }
      },
      {
        "_index" : "tmdb",
        "_type" : "_doc",
        "_id" : "348",
        "_score" : 7.307767,
        "_source" : {
          "overview" : "During its return to the earth, commercial spaceship Nostromo intercepts a distress signal from a distant planet. When a three-member team of the crew discovers a chamber containing thousands of eggs on the planet, a creature inside one of the eggs attacks an explorer. The entire crew is unaware of the impending nightmare set to descend upon them when the alien parasite planted inside its unfortunate host is birthed.",
          "title" : "Alien"
        }
      },
      {
        "_index" : "tmdb",
        "_type" : "_doc",
        "_id" : "8077",
        "_score" : 7.307767,
        "_source" : {
          "overview" : "After escaping with Newt and Hicks from the alien planet, Ripley crash lands on Fiorina 161, a prison planet and host to a correctional facility. Unfortunately, although Newt and Hicks do not survive the crash, a more unwelcome visitor does. The prison does not allow weapons of any kind, and with aid being a long time away, the prisoners must simply survive in any way they can.",
          "title" : "Alien³"
        }
      },

DELETE _scripts/tmdb
```
## 4. Index Alias（索引别名）
### 4.1 使用 Index Alias 实现零停机运维

```bash
PUT movies-2019/_doc/1
{
  "name":"the matrix",
  "rating":5
}

PUT movies-2019/_doc/2
{
  "name":"Speed",
  "rating":3
}

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies-2019",
        "alias": "movies-latest"
      }
    }
  ]
}


POST movies-latest/_search
{
  "query": {
    "match_all": {}
  }
}

返回：
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
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movies-2019",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "the matrix",
          "rating" : 5
        }
      },
      {
        "_index" : "movies-2019",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "Speed",
          "rating" : 3
        }
      }
    ]
  }
}


```
### 4.2 使用 Alias 创建不同查询的视图

```bash
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies-2019",
        "alias": "movies-lastest-highrate",
        "filter": {
          "range": {
            "rating": {
              "gte": 4
            }
          }
        }
      }
    }
  ]
}


POST movies-lastest-highrate/_search
{
  "query": {
    "match_all": {}
  }
}
返回：
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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movies-2019",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "the matrix",
          "rating" : 5
        }
      }
    ]
  }
}



```







