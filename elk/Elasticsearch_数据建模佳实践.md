

---
## 1. 建模建议（一）：如何处理关联关系

 1. Object ： 优先考虑 Denormailzation
 2. Nested ： 当数据包含多数值对象（对个演员），同时有查询需求
 3. Child/Parent: 关联文档更新非常频繁时

## 2. Kibana

 1. Kibana 目前暂不支持 nested 类型 和 parent /child 类型，在未来有可能会支持
 2. 如果需要使用 Kibana 进行数据分析，在数据建模时仍需要对嵌套和父子关联类型作出取舍

## 3. 建模建议（二）：避免过多字段
一个文档中，最好避免大量的字段

 - 过多的字段数不容易维护
 - Mapping 信息保存在 Cluster State 中， 数据量过大，对集群性能会有影响（Cluster State
   信息需要和所有的节点同步）
 - 删除或者修改数据需要 reindex

默认最大字段数是 1000，可以设置 index.mapping.total_fields.limit 限制最大的字段数
什么原因会导致文档中会有成百上千的字段？
## 4. Dynamic v.s Strict
Dynamic （生产环境中，尽量不要打开 Dynamic）

 - true - 未知字段会被自动加入
 - false - 新字段不会被索引，但是会保存在 _source
 - strict - 新增字段不会被索引，文档写入失败

Strict

 - 可以控制到字段级别

## 5. 一个例子： Cookie Service 的数据
来自 Cookie Service 的数据

 - Cookie 的键值对很多
 - 当 Dynamic 设置为 True
 - 同时采用扁平化的设计，必然导致字段数量的膨胀
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310150955162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
###### Cookie Service

##索引数据，dynamic mapping 会不断加入新增字段
PUT cookie_service/_doc/1
{
 "url":"www.google.com",
 "cookies":{
   "username":"tom",
   "age":32
 }
}

PUT cookie_service/_doc/2
{
 "url":"www.amazon.com",
 "cookies":{
   "login":"2019-01-01",
   "email":"xyz@abc.com"
 }
}



GET cookie_service

DELETE cookie_service
#使用 Nested 对象，增加key/value
PUT cookie_service
{
  "mappings": {
    "properties": {
      "cookies": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "keyword"
          },
          "dateValue": {
            "type": "date"
          },
          "keywordValue": {
            "type": "keyword"
          },
          "IntValue": {
            "type": "integer"
          }
        }
      },
      "url": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}


##写入数据，使用key和合适类型的value字段
PUT cookie_service/_doc/1
{
 "url":"www.google.com",
 "cookies":[
    {
      "name":"username",
      "keywordValue":"tom"
    },
    {
       "name":"age",
      "intValue":32

    }

   ]
 }


PUT cookie_service/_doc/2
{
 "url":"www.amazon.com",
 "cookies":[
    {
      "name":"login",
      "dateValue":"2019-01-01"
    },
    {
       "name":"email",
      "IntValue":32

    }

   ]
 }


# Nested 查询，通过bool查询进行过滤
POST cookie_service/_search
{
  "query": {
    "nested": {
      "path": "cookies",
      "query": {
        "bool": {
          "filter": [
            {
            "term": {
              "cookies.name": "age"
            }},
            {
              "range":{
                "cookies.intValue":{
                  "gte":30
                }
              }
            }
          ]
        }
      }
    }
  }
}

返回输出：
{
  "took" : 467,
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
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "cookie_service",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.0,
        "_source" : {
          "url" : "www.google.com",
          "cookies" : [
            {
              "name" : "username",
              "keywordValue" : "tom"
            },
            {
              "name" : "age",
              "intValue" : 32
            }
          ]
        }
      }
    ]
  }
}
```
## 6. 通过 Nested 对象保存 Key / Value 的一些不足
可以减少字段数量，解决 Cluster State 中 保存过多 Meta 信息的问题，但是

 - 导致查询语句复杂度增加
 - Nested 对象 ，不利于在 Kibana 汇总实现可视化分析

## 7. 建模建议（三）：避免正则查询
问题：

 - 正则，通配符查询，前缀查询属于 Term 查询，但是性能不够好
 - 特别是将通配符放在开头，会导致性能的灾难

案例：

 - 文档中某个字段包含了 ES 的版本信息，例如 version：“7.1.0”
 - 搜索所有是 bug fix 的版本？每个主要版本号所关联的文档？

```bash

# 在Mapping中加入元信息，便于管理
PUT softwares/
{
  "mappings": {
    "_meta": {
      "software_version_mapping": "1.0"
    }
  }
}

GET softwares/_mapping
PUT softwares/_doc/1
{
  "software_version":"7.1.0"
}
```
### 7.1 解决方案：将字符串转换为对象
```bash

DELETE softwares
# 优化,使用inner object
PUT softwares/
{
  "mappings": {
    "_meta": {
      "software_version_mapping": "1.1"
    },
    "properties": {
      "version": {
        "properties": {
          "display_name": {
            "type": "keyword"
          },
          "hot_fix": {
            "type": "byte"
          },
          "marjor": {
            "type": "byte"
          },
          "minor": {
            "type": "byte"
          }
        }
      }
    }
  }
}


#通过 Inner Object 写入多个文档
PUT softwares/_doc/1
{
  "version":{
  "display_name":"7.1.0",
  "marjor":7,
  "minor":1,
  "hot_fix":0  
  }

}

PUT softwares/_doc/2
{
  "version":{
  "display_name":"7.2.0",
  "marjor":7,
  "minor":2,
  "hot_fix":0  
  }
}

PUT softwares/_doc/3
{
  "version":{
  "display_name":"7.2.1",
  "marjor":7,
  "minor":2,
  "hot_fix":1  
  }
}


# 通过 bool 查询，
POST softwares/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "match":{
            "version.marjor":7
          }
        },
        {
          "match":{
            "version.minor":2
          }
        }

      ]
    }
  }
}

返回输出：
{
  "took" : 1,
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
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "softwares",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.0,
        "_source" : {
          "version" : {
            "display_name" : "7.2.0",
            "marjor" : 7,
            "minor" : 2,
            "hot_fix" : 0
          }
        }
      },
      {
        "_index" : "softwares",
        "_type" : "_doc",
        "_id" : "3",
        "_score" : 0.0,
        "_source" : {
          "version" : {
            "display_name" : "7.2.1",
            "marjor" : 7,
            "minor" : 2,
            "hot_fix" : 1
          }
        }
      }
    ]
  }
}

```

## 8. 建模建议（四）：避免空置引起的聚合不准
```bash
# Not Null 解决聚合的问题
DELETE ratings
PUT ratings
{
  "mappings": {
      "properties": {
        "rating": {
          "type": "float",
          "null_value": 1.0
        }
      }
    }
}


PUT ratings/_doc/1
{
 "rating":5
}
PUT ratings/_doc/2
{
 "rating":null
}


POST ratings/_search
POST ratings/_search
{
  "size": 0,
  "aggs": {
    "avg": {
      "avg": {
        "field": "rating"
      }
    }
  }
}

POST ratings/_search
{
  "query": {
    "term": {
      "rating": {
        "value": 1
      }
    }
  }
}
```
## 9. 建模建议（五）：为索引的 Mapping 加入 Meta 的信息
Mappings 设置非常重要，需要从两个维度进行考虑

 - 功能：索引，聚合，排序
 - 性能：存储的开销，内存的开销，搜索的性能

Mappings 设置是一个迭代的过程

 - 加入新的字段容易（必要时需要 update_by_query）
 - 更新删除字段不允许（需要 Reindex 重建数据）
 - 最好能对 Mappings 加入 Meta 信息，更好的进行版本管理
 - 可以考虑 Mapping 文件上传 git 进行管理

```c
PUT softwares/
{
  "mappings": {
    "_meta": {
      "software_version_mapping": "1.0"
    }
  }
}
```

