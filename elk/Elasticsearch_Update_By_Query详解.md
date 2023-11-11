

----
## 1. 使用场景
一般在以下几种情况时，我们需要重建索引：

 - 索引的 Mappings 发生变更：字段类型更改，分词器及字典更新
 - 索引的 Setting 发生变更：索引的主分片数发生改变
 - 集群内，集群间需要做数据迁移

ElastiicSearch 的内置提供的 API

 - Update By Query : 在现有索引上重建
 - Reindex：在其他索引上重建索引

## 2. Update By Query
### 2.1 案例一： 为索引增加子字段

 - 改变 Mapping ， 增加子字段，使用英文分词器
 - 此时尝试对子字段进行查询
 - 虽然有数据已经存在，但是没有返回结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210308095608396.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

执行 Update By Query
尝试对 Multi-Fields 查询查询
返回结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210308095802892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
# 写入文档
PUT blogs/_doc/1
{
  "content":"Hadoop is cool",
  "keyword":"hadoop"
}
# 修改 Mapping，增加子字段，使用英文分词器
PUT blogs/_mapping
{
  "properties" : {
    "content" : {
      "type" : "text",
      "fields" : {
        "english" : {
          "type" : "text",
          "analyzer":"english"
        }
      }
    }
  }
}

    # 写入文档
PUT blogs/_doc/2
{
  "content":"Elasticsearch rocks",
  "keyword":"elasticsearch"
}

# 查询新写入文档
POST blogs/_search
{
  "query": {
    "match": {
      "content.english": "Elasticsearch"
    }
  }
}

# 查询 Mapping 变更前写入的文档
POST blogs/_search
{
  "query": {
    "match": {
      "content.english": "hadoop"
    }
  }
}

# Update所有文档
POST blogs/_update_by_query
{

}
```
### 2.2 案例二：更改已有字段类型的 Mappings

 - ES 不允许在原有 Mapping 上对字段类型进行修改
 - 只能创建新的索引，并设定正确的字段类型，在重新导入数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210308102033951.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash

# 查询
GET blogs/_mapping

PUT blogs/_mapping
{
        "properties" : {
        "content" : {
          "type" : "text",
          "fields" : {
            "english" : {
              "type" : "text",
              "analyzer" : "english"
            }
          }
        },
        "keyword" : {
          "type" : "keyword"
        }
      }
}

返回输出：
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "mapper [keyword] of different type, current_type [text], merged_type [keyword]"
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "mapper [keyword] of different type, current_type [text], merged_type [keyword]"
  },
  "status": 400
}

# 创建新的索引并且设定新的Mapping
PUT blogs_fix/
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "fields": {
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      },
      "keyword": {
        "type": "keyword"
      }
    }
  }
}

# Reindx API
POST  _reindex
{
  "source": {
    "index": "blogs"
  },
  "dest": {
    "index": "blogs_fix"
  }
}
返回输出：
{
  "took" : 17,
  "timed_out" : false,
  "total" : 2,
  "updated" : 0,
  "created" : 2,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}

GET  blogs_fix/_doc/1
返回输出：
{
  "_index" : "blogs_fix",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "content" : "Hadoop is cool",
    "keyword" : "hadoop"
  }
}

# 测试 Term Aggregation
POST blogs_fix/_search
{
  "size": 0,
  "aggs": {
    "blog_keyword": {
      "terms": {
        "field": "keyword",
        "size": 10
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
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "blog_keyword" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "elasticsearch",
          "doc_count" : 1
        },
        {
          "key" : "hadoop",
          "doc_count" : 1
        }
      ]
    }
  }
}
```



