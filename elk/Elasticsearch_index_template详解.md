
## 1. 场景应用
### 1.1 管理多个索引
使用多个索引可以让你的更好的管理的你数据，提高性能

 - logs-2020-05-01
 - logs-2020-05-02
 - logs-2020-05-03

## 2. 什么是index template

 - 官网：[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html)

Index Templates 帮助你设定 `Mapping` 和 `Settings`，并按照一定的规则，自动匹配到新创建的索引之上

 - 模板仅在一个索引被新创建时，才会产生作用。修改模板不会影响已创建的索引
 - 可以设定多个索引模板，这些设置会被 `merge` 在一起
 - 可以指定 `order` 的数值，控制 merging 的过程

### 2.1 index template格式
`index_patterns` 索引名称 `[“*”] [“test*”]` 表示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201110112137109.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

### 2.2 Index Template 工作方式
当一个索引被新创建时

 - 应用 Elasticsearch默认的 `settings` 和 `mappings`
 - 应该 `order` 数值低的 Index Template 中的设定
 - 应用 order 高的 Index Template 中的设定，之前的设定会被覆盖
 - 应用创建索引时，用户所指定的 Settings 和 Mappings，并覆盖之前模板中的设定

### 2.3 实例1
#### 2.3.1 测试默认的索引类型

```bash
#测试下默认的mapping
PUT ttemplate/_doc/1
{
  "someNumber" :"1",
  "someDate" : "2019/01/01"
}
//发现someDate推算正确，但是someNumber没有推算称number类型
GET ttemplate/_mapping
{
  "ttemplate" : {
    "mappings" : {
      "properties" : {
        "someDate" : {
          "type" : "date",
          "format" : "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis"
        },
        "someNumber" : {
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
}
```
#### 2.3.2 创建 template

```bash
#创建一个默认的template
PUT _template/template_default
{
  "index_patterns" : ["*"],
  "order" : 0,
  "version": 1,
  "settings" :{
    "number_of_shards" :1,
    "number_of_replicas" :1
  }
}
#匹配test开头的patterns
PUT _template/template_test
{
    "index_patterns" : ["test*"],
    "order" : 1,
    "settings" : {
        "number_of_shards": 1,
        "number_of_replicas" : 2
    },
    "mappings" : {
        "date_detection": false,
        "numeric_detection": true
    }
}
```
#### 2.3.3 查看 template 信息

```bash
GET _template/template_default
GET _template/temp* //通配符查看所有的
```
#### 2.3.4 写入新的数据，index 以 test 开头

```bash
PUT testtemplate/_doc/1
{
    "someNumber":"1",
    "someDate":"2019/01/01"
}
GET testtemplate/_mapping
{
  "testtemplate" : {
    "mappings" : {
      "date_detection" : false,
      "numeric_detection" : true,
      "properties" : {
        "someDate" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "someNumber" : {
          "type" : "long"
        }
      }
    }
  }
}

GET testtemplate/_settings
{
  "testtemplate" : {
    "settings" : {
      "index" : {
        "creation_date" : "1572275616083",
        "number_of_shards" : "1",
        "number_of_replicas" : "2",
        "uuid" : "s3B3NEaZRDSq586qDVbCqA",
        "version" : {
          "created" : "7030099"
        },
        "provided_name" : "testtemplate"
      }
    }
  }
}
```
#### 2.3.5 设置 test 开头 settings

```bash
PUT testmy
{
    "settings":{
        "number_of_replicas":5
    }
}

PUT testmy/_doc/1
{
  "key":"value"
}


//testmy 设置完之后，会覆盖test*开头是settings
GET testmy/_settings
{
  "testmy" : {
    "settings" : {
      "index" : {
        "creation_date" : "1572275856520",
        "number_of_shards" : "1",
        "number_of_replicas" : "5",
        "uuid" : "yJJ02g77S6ubgs1w3OVY2g",
        "version" : {
          "created" : "7030099"
        },
        "provided_name" : "testmy"
      }
    }
  }
}
```

### 2.4 实例2

```bash
#启动elasticsearch
[elastic@slave1 elasticsearch-7.2.0]$ tar -zxvf elasticsearch-7.2.0-linux-x86_64.tar.gz 
[elastic@slave1 elasticsearch-7.2.0]$ cd elasticsearch-7.2.0/
[elastic@slave1 elasticsearch-7.2.0]$ ./bin/elasticsearch -d
[elastic@slave1 elasticsearch-7.2.0]$ curl http://127.0.0.1:9200
{
  "name" : "slave1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "z4AYaKxPTVmIoe2CQsh9Tw",
  "version" : {
    "number" : "7.2.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "508c38a",
    "build_date" : "2019-06-20T15:54:18.811730Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

#创建模板template_1，匹配所有的以zhouls开头的索引
[elastic@slave1 elasticsearch-7.2.0]$ curl -H "Content-Type:application/json" -XPUT 127.0.0.1:9200/_template/template_1 -d '
> {
>     "template" : "zhouls*",
>     "order" : 0,
>     "settings" : {
>         "number_of_shards" : 1
>     },
>     "mappings" : {
>         "_source" : { "enabled" : false }
>     }
> }
> '
{"acknowledged":true}[elastic@slave1 elasticsearch-7.2.0]$ 

#给索引zhouls10赋值
[elastic@slave1 elasticsearch-7.2.0]$ curl -H "Content-Type:application/json" -XPUT '127.0.0.1:9200/zhouls10/emp/1' -d '{"name":"zs"}'

#查看模板配置
[elastic@slave1 elasticsearch-7.2.0]$ curl -H "Content-Type:application/json" -XGET 127.0.0.1:9200/_template/template_1

## 查看索引配置
[elastic@slave1 elasticsearch-7.2.0]$  curl -H "Content-Type:application/json"  -XGET http://127.0.0.1:9200/zhouls10/_settings?pretty 
{
  "zhouls10" : {
    "settings" : {
      "index" : {
        "creation_date" : "1616571416544",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "2zja346mRvehMzsFux5MSg",
        "version" : {
          "created" : "7020099"
        },
        "provided_name" : "zhouls10"
      }
    }
  }
}

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
[Elasticsearch Analyzer 进行分词详解](https://blog.csdn.net/xixihahalelehehe/article/details/109447777)
[Elasticsearch search API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109449425)
[Elasticsearch URI Search 查询方法详解](https://blog.csdn.net/xixihahalelehehe/article/details/109453253)
[Elasticsearch Request Body 与 Query DSL详解](https://blog.csdn.net/xixihahalelehehe/article/details/109458983)
[Elasticsearch Dynamic Mapping 和常见字段类型详解](https://blog.csdn.net/xixihahalelehehe/article/details/109463294)
[Eelasticsearch 多字段特性及 Mapping 中配置自定义 Analyzer详解](https://blog.csdn.net/xixihahalelehehe/article/details/109476672)
