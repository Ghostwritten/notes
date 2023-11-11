

--
## 1. 数据的关联关系

真实世界中很多重要的关联关系

 - 博客、作者、评论
 - 银行账户有多次交易记录
 - 客户有很多银行账户
 - 目录文件有很多文件和子目录


![在这里插入图片描述](https://img-blog.csdnimg.cn/2021033014132060.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


## 2. 关系型数据库的范式化设计

 - 范式化设计（Normalization）的主要目标是 “减少不必要的更新”
 - 副作用：一个完全范式化设计的数据库经常面临 “查询缓慢” 的问题
数据库余额范式化，就需要 Join 越多的表
 - 范式化节省了储存空间，但是储存空间越来越便宜
 - 范式化简化了更新，但是数据 “读” 取操作可能越多
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306195831127.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 3. Denormalization
反范式化设计
 - 数据 “Flattening”, 不使用关联关系，而是在文档中保存冗余的数据拷贝

优点：无需处理 Joins 操作，数据读取性能好

 - Elasticsearch 通过压缩_source 字段，减少磁盘空间的开销

缺点：不适合在数据频繁修改的场景

 - 一条数据（用户名）的改动，可能会引起很多数据的更新

## 4. 在 Elasticsearch 中处理关联关系
关系型数据库，一般会考虑 Normalize 数据；在 Elasticsearch，往往考虑 Denormalize 数据

 - Denormalize 的好处：读的速度变快、无需表连接、无需行锁

Elasticsearch 并不擅长处理关联关系，我们一般采用以下四种方法处理关联

 - 对象类型
 - 嵌套对象（Nested Object）
 - 父子关联关系（Parent 、Child）
 - 应用端关联

## 5. 案例 1：博客和其作者信息
对象类型

 - 在每个博客的问下中都保留作者的信息
 - 如果作者信息发生变化，需要修改相关的博客文档
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306200110594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 5.1 设置blog的mapping
```bash
PUT /blog
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text"
      },
      "time": {
        "type": "date"
      },
      "user": {
        "properties": {
          "city": {
            "type": "text"
          },
          "userid": {
            "type": "long"
          },
          "username": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### 5.2 插入一条 Blog 信息
```bash
PUT blog/_doc/1
{
  "content":"I like Elasticsearch",
  "time":"2019-01-01T00:00:00",
  "user":{
    "userid":1,
    "username":"Jack",
    "city":"Shanghai"
  }
}

返回输出：
{
  "_index" : "blog",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

```


### 5.3 查询 Blog 信息

```bash
POST blog/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"content": "Elasticsearch"}},
        {"match": {"user.username": "Jack"}}
      ]
    }
  }
}

返回输出：
{
  "took" : 28,
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
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "blog",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.5753642,
        "_source" : {
          "content" : "I like Elasticsearch",
          "time" : "2019-01-01T00:00:00",
          "user" : {
            "userid" : 1,
            "username" : "Jack",
            "city" : "Shanghai"
          }
        }
      }
    ]
  }
}
```
## 6. 案例 2：包含对象数组的文档
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306200819118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306200832778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
###  6.1 电影的Mapping信息

```bash
PUT my_movies
{
      "mappings" : {
      "properties" : {
        "actors" : {
          "properties" : {
            "first_name" : {
              "type" : "keyword"
            },
            "last_name" : {
              "type" : "keyword"
            }
          }
        },
        "title" : {
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
```
### 6.2 写入一条电影信息

```bash
POST my_movies/_doc/1
{
  "title":"Speed",
  "actors":[
    {
      "first_name":"Keanu",
      "last_name":"Reeves"
    },

    {
      "first_name":"Dennis",
      "last_name":"Hopper"
    }

  ]
}

```

### 6.3 查询电影信息

```bash
POST my_movies/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"actors.first_name": "Keanu"}},
        {"match": {"actors.last_name": "Hopper"}}
      ]
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
  }
}

```
### 6.4 为啥搜不到

 - 储存时，内部对象的边界没有在考虑在内，JSON 格式被处理成扁平键值对的结构
 - 当对多个字段进行查询时，导致了意外的搜索结果
 - 可以用 Nested Data Type 解决这个问题

## 7. Nested Data Type

 - Nested 数据类型：允许对象数组中的对象呗独立索引
 - 使用 Nested 和 Properties 关键词，将所有 actors 索引到对个分隔的文档
 - 在内部，Nested 文档会被保存在两个 Lucene 文档中，查询时做 join 处理

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306201454511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 7.1 PUT my_movies

```bash
{
      "mappings" : {
      "properties" : {
        "actors" : {
          "type": "nested",
          "properties" : {
            "first_name" : {"type" : "keyword"},
            "last_name" : {"type" : "keyword"}
          }},
        "title" : {
          "type" : "text",
          "fields" : {"keyword":{"type":"keyword","ignore_above":256}}
        }
      }
    }
}


POST my_movies/_doc/1
{
  "title":"Speed",
  "actors":[
    {
      "first_name":"Keanu",
      "last_name":"Reeves"
    },

    {
      "first_name":"Dennis",
      "last_name":"Hopper"
    }

  ]
}
```

### 7.2 Nested 查询

```bash
POST my_movies/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"title": "Speed"}},
        {
          "nested": {
            "path": "actors",
            "query": {
              "bool": {
                "must": [
                  {"match": {
                    "actors.first_name": "Keanu"
                  }},

                  {"match": {
                    "actors.last_name": "Hopper"
                  }}
                ]
              }
            }
          }
        }
      ]
    }
  }
}
返回输出：
{
  "took" : 12,
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
  }
}
```


### 7.3 Nested Aggregation

```bash
POST my_movies/_search
{
  "size": 0,
  "aggs": {
    "actors": {
      "nested": {
        "path": "actors"
      },
      "aggs": {
        "actor_name": {
          "terms": {
            "field": "actors.first_name",
            "size": 10
          }
        }
      }
    }
  }
}

返回输出：
{
  "took" : 10,
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
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "actors" : {
      "doc_count" : 2,
      "actor_name" : {
        "doc_count_error_upper_bound" : 0,
        "sum_other_doc_count" : 0,
        "buckets" : [
          {
            "key" : "Dennis",
            "doc_count" : 1
          },
          {
            "key" : "Keanu",
            "doc_count" : 1
          }
        ]
      }
    }
  }
}

```

### 7.4 普通 aggregation不工作

```bash
POST my_movies/_search
{
  "size": 0,
  "aggs": {
    "NAME": {
      "terms": {
        "field": "actors.first_name",
        "size": 10
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
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "NAME" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [ ]
    }
  }
}

```



