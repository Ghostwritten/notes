

-----

## 1. 需求：修复与增强写入的数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210308105720602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

Tags 字段中，逗号分割的文本应该是数组，而不是一个字符串

 - 需求：后期需要对 Tags 进行 `Aggregation` 统计

## 2. Ingest Node
Elasticsearch 5.0 后，引入的一种新的节点类型。默认配置下，每个节点都是 `Ingest Node`

 - 具有预处理数据的能力，可拦截 `Index` 或者 `Bulck API` 的请求
 - 对数据进行转换，并重新返回给 Index 和 Bluck API

无需 Logstash ，就可以进行数据的预处理，例如

 - 为某个字段设置默认值；重命名某个字段的字段名；对字段值进行 `Split` 操作
 - 支持设置 Painless 脚本，对数据进行更加复杂的加工

## 3. Pipeline & Processor
`Pipeline` - 管道会对通过的数据（文档），按照顺序进行加工
`Processor` - Elasticsearch 对一些加工的行为进行了抽象包装
Elasticsearch 有很多内置的 Processors。也支持通过插件的方式，实现自己的 Processsor

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210308105944661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. 使用 Pipeline 切分字符串
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210308110012603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
DELETE tech_blogs

#Blog数据，包含3个字段，tags用逗号间隔
PUT tech_blogs/_doc/1
{
  "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data"
}


# 测试split tags
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "to split blog tags",
    "processors": [
      {
        "split": {
          "field": "tags",
          "separator": ","
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_id": "id",
      "_source": {
        "title": "Introducing big data......",
        "tags": "hadoop,elasticsearch,spark",
        "content": "You konw, for big data"
      }
    },
    {
      "_index": "index",
      "_id": "idxx",
      "_source": {
        "title": "Introducing cloud computering",
        "tags": "openstack,k8s",
        "content": "You konw, for cloud"
      }
    }
  ]
}

返回输出：
{
  "docs" : [
    {
      "doc" : {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "id",
        "_source" : {
          "title" : "Introducing big data......",
          "content" : "You konw, for big data",
          "tags" : [
            "hadoop",
            "elasticsearch",
            "spark"
          ]
        },
        "_ingest" : {
          "timestamp" : "2021-03-08T03:21:26.220387Z"
        }
      }
    },
    {
      "doc" : {
        "_index" : "index",
        "_type" : "_doc",
        "_id" : "idxx",
        "_source" : {
          "title" : "Introducing cloud computering",
          "content" : "You konw, for cloud",
          "tags" : [
            "openstack",
            "k8s"
          ]
        },
        "_ingest" : {
          "timestamp" : "2021-03-08T03:21:26.220411Z"
        }
      }
    }
  ]
}
```

```bash
#同时为文档，增加一个字段。blog查看量
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "description": "to split blog tags",
    "processors": [
      {
        "split": {
          "field": "tags",
          "separator": ","
        }
      },

      {
        "set":{
          "field": "views",
          "value": 0
        }
      }
    ]
  },

  "docs": [
    {
      "_index":"index",
      "_id":"id",
      "_source":{
        "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data"
      }
    },


    {
      "_index":"index",
      "_id":"idxx",
      "_source":{
        "title":"Introducing cloud computering",
  "tags":"openstack,k8s",
  "content":"You konw, for cloud"
      }
    }

    ]
}


```

## 5. 为 ES添加一个 Pipeline
```bash

PUT _ingest/pipeline/blog_pipeline
{
  "description": "a blog pipeline",
  "processors": [
      {
        "split": {
          "field": "tags",
          "separator": ","
        }
      },

      {
        "set":{
          "field": "views",
          "value": 0
        }
      }
    ]
}

#查看Pipleline
GET _ingest/pipeline/blog_pipeline


#测试pipeline
POST _ingest/pipeline/blog_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "title": "Introducing cloud computering",
        "tags": "openstack,k8s",
        "content": "You konw, for cloud"
      }
    }
  ]
}
```
## 6. Pipeline API
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310140822641.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 7. 添加 Pipeline 并测试
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310140901556.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
# 为ES添加一个 Pipeline
PUT _ingest/pipeline/blog_pipeline
{
  "description": "a blog pipeline",
  "processors": [
    {
      "split": {
        "field": "tags",
        "separator": ","
      }
    },
    {
      "set": {
        "field": "views",
        "value": 0
      }
    }
  ]
}
#测试pipeline
POST _ingest/pipeline/blog_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "title": "Introducing cloud computering",
        "tags": "openstack,k8s",
        "content": "You konw, for cloud"
      }
    }
  ]
}
```

## 8. Index & Update By Query
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310141009175.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
#不使用pipeline更新数据
PUT tech_blogs/_doc/1
{
  "title":"Introducing big data......",
  "tags":"hadoop,elasticsearch,spark",
  "content":"You konw, for big data"
}

#使用pipeline更新数据
PUT tech_blogs/_doc/2?pipeline=blog_pipeline
{
  "title": "Introducing cloud computering",
  "tags": "openstack,k8s",
  "content": "You konw, for cloud"
}

#查看两条数据，一条被处理，一条未被处理
POST tech_blogs/_search
{}

#update_by_query 会导致错误
POST tech_blogs/_update_by_query?pipeline=blog_pipeline
{
}

#增加update_by_query的条件
POST tech_blogs/_update_by_query?pipeline=blog_pipeline
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "views"
        }
      }
    }
  }
}
```

## 10. 一些内置的 Processors
[https://www.elastic.co/guide/en/elasticsea..](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/ingest-processors.html)

 - Split Processor （例如：将给定字段分成一个数组）
 - Remove / Rename Processor （移除一个重命名字段）
 - Append（为商品增加一个新的标签）
 - Convert （将商品价格，从字符串转换成 float 类型）
 - Date / JSON （日期格式转换，字符串转 JSON 对象）
 - Date Index Name Processor （将通过该处理器的文档，分配到指定时间格式的索引中）
 - Fail Processor （一旦出现异常，该 Pipeline 指定的错误信息能返回给用户）
 - Foreach Process （数组字段，数组的每个元素都会使用到一个相同的处理器）
 - Grok Processor （日志的日志格式切割）
 - Gsub / Join / Split （字符串替换、数组转字符串、字符串转数组）
 - Lowercase / Upcase（大小写转换）


## 11. Ingest Node v.s Logstash
|| Logstash| Ingest Node|
|–|–|
| 数据输入与输出 | 支持从不同的数据源读取，并写入不同的数据源 | 支持从 ES REST API 获取数据，并且写入 ES|
| 数据源缓冲 | 实现了简单的数据队列，支持重写 | 不支持缓冲 |
| 数据处理 | 支持大量的的插件，也支持定制开发 | 内置的插件，可以开发 Plugin 进行扩展（Plugin 更新需要重启）|
| 配置和使用 | 增加了一定的架构复杂度 | 无需额外部署 |

[https://www.elastic.co/cn/blog/should-i-us...](https://www.elastic.co/cn/blog/should-i-use-logstash-or-elasticsearch-ingest-nodes)



