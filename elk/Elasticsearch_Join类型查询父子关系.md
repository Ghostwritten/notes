


----
## 1. arent / Child
对象和 Nested 对象的局限性

 - 每次更新，需要重新索引整个对象（包括跟对象和嵌套对象）

ES 提供了类似关系型数据库中 Join 的实现。使用 Join 数据类型实现，可以通过 Parent / Child 的关系，从而分离两个对象

 - 父文档和子文档是两个独立的文档
 - 更新父文档无需重新索引整个子文档。子文档被新增，更改和删除也不会影响到父文档和其他子文档。

## 2. 父子关系
定义父子关系的几个步骤

 1. 设置索引的 Mapping
 2. 索引父文档
 3. 索引子文档
 4. 按需查询文档

## 3. 示例1
###  3.1 设置 Mapping
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306203346577.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
DELETE my_blogs

# 设定 Parent/Child Mapping
PUT my_blogs
{
  "settings": {
    "number_of_shards": 2
  },
  "mappings": {
    "properties": {
      "blog_comments_relation": {
        "type": "join",
        "relations": {
          "blog": "comment"
        }
      },
      "content": {
        "type": "text"
      },
      "title": {
        "type": "keyword"
      }
    }
  }
}
```

### 3.2 索引父文档
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306203455420.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
#索引父文档
PUT my_blogs/_doc/blog1
{
  "title":"Learning Elasticsearch",
  "content":"learning ELK @ geektime",
  "blog_comments_relation":{
    "name":"blog"
  }
}

#索引父文档
PUT my_blogs/_doc/blog2
{
  "title":"Learning Hadoop",
  "content":"learning Hadoop",
    "blog_comments_relation":{
    "name":"blog"
  }
}
```

### 3.3 索引子文档
父文档和子文档必须存在相同的分片上

 - 确保查询 join 的性能

当指定文档时候，必须指定它的父文档 ID

 - 使用 route 参数来保证，分配到相同的分片
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306203545949.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
#索引子文档
PUT my_blogs/_doc/comment1?routing=blog1
{
  "comment":"I am learning ELK",
  "username":"Jack",
  "blog_comments_relation":{
    "name":"comment",
    "parent":"blog1"
  }
}

#索引子文档
PUT my_blogs/_doc/comment2?routing=blog2
{
  "comment":"I like Hadoop!!!!!",
  "username":"Jack",
  "blog_comments_relation":{
    "name":"comment",
    "parent":"blog2"
  }
}

#索引子文档
PUT my_blogs/_doc/comment3?routing=blog2
{
  "comment":"Hello Hadoop",
  "username":"Bob",
  "blog_comments_relation":{
    "name":"comment",
    "parent":"blog2"
  }
}
```
### 3.4 查询

```bash
# 查询所有文档
POST my_blogs/_search
{

}
返回输出：
...

#根据父文档ID查看
GET my_blogs/_doc/blog2

返回输出：
{
  "_index" : "my_blogs",
  "_type" : "_doc",
  "_id" : "blog2",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "title" : "Learning Hadoop",
    "content" : "learning Hadoop",
    "blog_comments_relation" : {
      "name" : "blog"
    }
  }
}


# Parent Id 查询
POST my_blogs/_search
{
  "query": {
    "parent_id": {
      "type": "comment",
      "id": "blog2"
    }
  }
}

返回输出：
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.44183275,
    "hits" : [
      {
        "_index" : "my_blogs",
        "_type" : "_doc",
        "_id" : "comment2",
        "_score" : 0.44183275,
        "_routing" : "blog2",
        "_source" : {
          "comment" : "I like Hadoop!!!!!",
          "username" : "Jack",
          "blog_comments_relation" : {
            "name" : "comment",
            "parent" : "blog2"
          }
        }
      },
      {
        "_index" : "my_blogs",
        "_type" : "_doc",
        "_id" : "comment3",
        "_score" : 0.44183275,
        "_routing" : "blog2",
        "_source" : {
          "comment" : "Hello Hadoop",
          "username" : "Bob",
          "blog_comments_relation" : {
            "name" : "comment",
            "parent" : "blog2"
          }
        }
      }
    ]
  }
}


# Has Child 查询,返回父文档
POST my_blogs/_search
{
  "query": {
    "has_child": {
      "type": "comment",
      "query" : {
                "match": {
                    "username" : "Jack"
                }
            }
    }
  }
}

返回输出：
{
  "took" : 43,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
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
        "_index" : "my_blogs",
        "_type" : "_doc",
        "_id" : "blog1",
        "_score" : 1.0,
        "_source" : {
          "title" : "Learning Elasticsearch",
          "content" : "learning ELK @ geektime",
          "blog_comments_relation" : {
            "name" : "blog"
          }
        }
      },
      {
        "_index" : "my_blogs",
        "_type" : "_doc",
        "_id" : "blog2",
        "_score" : 1.0,
        "_source" : {
          "title" : "Learning Hadoop",
          "content" : "learning Hadoop",
          "blog_comments_relation" : {
            "name" : "blog"
          }
        }
      }
    ]
  }
}


# Has Parent 查询，返回相关的子文档
POST my_blogs/_search
{
  "query": {
    "has_parent": {
      "parent_type": "blog",
      "query" : {
                "match": {
                    "title" : "Learning Hadoop"
                }
            }
    }
  }
}

返回输出：
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
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
        "_index" : "my_blogs",
        "_type" : "_doc",
        "_id" : "comment2",
        "_score" : 1.0,
        "_routing" : "blog2",
        "_source" : {
          "comment" : "I like Hadoop!!!!!",
          "username" : "Jack",
          "blog_comments_relation" : {
            "name" : "comment",
            "parent" : "blog2"
          }
        }
      },
      {
        "_index" : "my_blogs",
        "_type" : "_doc",
        "_id" : "comment3",
        "_score" : 1.0,
        "_routing" : "blog2",
        "_source" : {
          "comment" : "Hello Hadoop",
          "username" : "Bob",
          "blog_comments_relation" : {
            "name" : "comment",
            "parent" : "blog2"
          }
        }
      }
    ]
  }
}



#通过ID ，访问子文档
GET my_blogs/_doc/comment3
返回输出：
{
  "_index" : "my_blogs",
  "_type" : "_doc",
  "_id" : "comment3",
  "found" : false
}


#通过ID和routing ，访问子文档
GET my_blogs/_doc/comment3?routing=blog2

返回输出：
{
  "_index" : "my_blogs",
  "_type" : "_doc",
  "_id" : "comment3",
  "_version" : 1,
  "_seq_no" : 5,
  "_primary_term" : 1,
  "_routing" : "blog2",
  "found" : true,
  "_source" : {
    "comment" : "Hello Hadoop",
    "username" : "Bob",
    "blog_comments_relation" : {
      "name" : "comment",
      "parent" : "blog2"
    }
  }
}

#更新子文档
PUT my_blogs/_doc/comment3?routing=blog2
{
    "comment": "Hello Hadoop??",
    "blog_comments_relation": {
      "name": "comment",
      "parent": "blog2"
    }
}

返回输出：
{
  "_index" : "my_blogs",
  "_type" : "_doc",
  "_id" : "comment3",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 6,
  "_primary_term" : 1
}
```


## 4. 示例2
### 4.1 Mapping定义
Join类型的Mapping如下:
核心
• 1） "my_join_field"为join的名称。
• 2）"question": "answer" 指：qustion为answer的父类。

```bash
PUT join_ext_index
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "question": [
            "answer",
            "comment"
          ]
        }
      }
    }
  }
}
```
### 4.2 join类型定义父文档
文档类型为父类型:"question"。

```bash
PUT my_join_index/_doc/1?refresh
{
  "text": "This is a question",
  "my_join_field": "question" 
}
PUT my_join_index/_doc/2?refresh
{
  "text": "This is another question",
  "my_join_field": "question"
}
```
###  4.3 join类型定义子文档
• 路由值是强制性的，因为父文件和子文件必须在相同的分片上建立索引。
• "answer"是此子文档的加入名称。
• 指定此子文档的父文档ID：1。

```bash
PUT my_join_index/_doc/3?routing=1&refresh 
{
  "text": "This is an answer",
  "my_join_field": {
    "name": "answer", 
    "parent": "1" 
  }
}
PUT my_join_index/_doc/4?routing=1&refresh
{
  "text": "This is another answer",
  "my_join_field": {
    "name": "answer",
    "parent": "1"
  }
}
```
### 4.4 Join类型约束
1. 每个索引只允许一个Join类型Mapping定义；
2. 父文档和子文档必须在同一个分片上编入索引；这意味着，当进行删除、更新、查找子文档时候需要提供相同的路由值。
3. 一个文档可以有多个子文档，但只能有一个父文档。
4. 可以为已经存在的Join类型添加新的关系。
5. 当一个文档已经成为父文档后，可以为该文档添加子文档。


### 4.5 Join全量检索

```bash
GET my_join_index/_search
{
  "query": {
    "match_all": {}
  },
  "sort": ["_id"]
}
```

返回结果如下：

```bash
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": null,
    "hits": [
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "1",
        "_score": null,
        "_source": {
          "text": "This is a question",
          "my_join_field": "question"
        },
        "sort": [
          "1"
        ]
      },
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "2",
        "_score": null,
        "_source": {
          "text": "This is another question",
          "my_join_field": "question"
        },
        "sort": [
          "2"
        ]
      },
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "3",
        "_score": null,
        "_routing": "1",
        "_source": {
          "text": "This is an answer",
          "my_join_field": {
            "name": "answer",
            "parent": "1"
          }
        },
        "sort": [
          "3"
        ]
      },
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "4",
        "_score": null,
        "_routing": "1",
        "_source": {
          "text": "This is another answer",
          "my_join_field": {
            "name": "answer",
            "parent": "1"
          }
        },
        "sort": [
          "4"
        ]
      }
    ]
  }
}
```
### 4.6 基于父文档查找子文档

```bash
GET my_join_index/_search
{
    "query": {
        "has_parent" : {
            "parent_type" : "question",
            "query" : {
                "match" : {
                    "text" : "This is"
                }
            }
        }
    }
}
```

返回结果：

```bash
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "3",
        "_score": 1,
        "_routing": "1",
        "_source": {
          "text": "This is an answer",
          "my_join_field": {
            "name": "answer",
            "parent": "1"
          }
        }
      },
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "4",
        "_score": 1,
        "_routing": "1",
        "_source": {
          "text": "This is another answer",
          "my_join_field": {
            "name": "answer",
            "parent": "1"
          }
        }
      }
    ]
  }
}
```
### 4.7 基于子文档查找父文档

```bash
GET my_join_index/_search
{
"query": {
        "has_child" : {
            "type" : "answer",
            "query" : {
                "match" : {
                    "text" : "This is question"
                }
            }
        }
    }
}
```

返回结果：

```bash
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "text": "This is a question",
          "my_join_field": "question"
        }
      }
    ]
  }
}
```
### 4.8 Join聚合操作实战
以下操作含义如下：
• 1）parent_id是特定的检索方式，用于检索属于特定父文档id=1的，子文档类型为answer的文档的个数。
• 2）基于父文档类型question进行聚合；
• 3）基于指定的field处理。

```bash
GET my_join_index/_search
{
  "query": {
    "parent_id": { 
      "type": "answer",
      "id": "1"
    }
  },
  "aggs": {
    "parents": {
      "terms": {
        "field": "my_join_field#question", 
        "size": 10
      }
    }
  },
  "script_fields": {
    "parent": {
      "script": {
         "source": "doc['my_join_field#question']" 
      }
    }
  }
}
```

返回结果：

```bash
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.13353139,
    "hits": [
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "3",
        "_score": 0.13353139,
        "_routing": "1",
        "fields": {
          "parent": [
            "1"
          ]
        }
      },
      {
        "_index": "my_join_index",
        "_type": "_doc",
        "_id": "4",
        "_score": 0.13353139,
        "_routing": "1",
        "fields": {
          "parent": [
            "1"
          ]
        }
      }
    ]
  },
  "aggregations": {
    "parents": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "1",
          "doc_count": 2
        }
      ]
    }
  }
}
```
## 5. Join 一对多实战
### 5.1 一对多定义
如下，一个父文档question与多个子文档answer，comment的映射定义。

```bash
PUT join_ext_index
{
  "mappings": {
    "_doc": {
      "properties": {
        "my_join_field": {
          "type": "join",
          "relations": {
            "question": ["answer", "comment"]  
          }
        }
      }
    }
  }
}
```

### 5.2 一对多对多定义
实现如下图的祖孙三代关联关系的定义。

```bash
question
    /    \
   /      \
comment  answer
           |
           |
          vote
```

```bash
PUT join_multi_index
{
  "mappings": {
    "properties": {
      "my_join_field": {
        "type": "join",
        "relations": {
          "question": [
            "answer",
            "comment"
          ],
          "answer": "vote"
        }
      }
    }
  }
}
```

孙子文档导入数据，如下所示：

```bash
PUT join_multi_index/_doc/3?routing=1&refresh 
{
  "text": "This is a vote",
  "my_join_field": {
    "name": "vote",
    "parent": "2" 
  }
}
```

注意：
- 孙子文档所在分片必须与其父母和祖父母相同
- 孙子文档的父代号（必须指向其父亲answer文档）
