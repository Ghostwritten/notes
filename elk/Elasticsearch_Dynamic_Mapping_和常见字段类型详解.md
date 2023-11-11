
## 1. Mapping介绍
Mapping 类似数据库的 schema 的定义，作用如下

 - 定义索引中的字段名称
 - 定义字段的数据类型，例如字符串，数字，布尔…
 - 字段，倒排索引的相关配置，(Analyzed or Not Analyzed,Analyzer)

Mapping 会把 JSON 文档映射称 Lucene 所需的扁平格式
一个 Mapping 属于一个索引的 Type

 - 每个文档都属于一个 Type
 - 一个 Type 有一个 Mapping 定义
 - 7.0 开始，不需要在 Mapping 定义指定 type 信息

## 2. 元字段
各种元字段，它们都以一个下划线开头，例如 _type 、 _id 和 _source。
## 2.1 Identity 元字段
• `_index`：表示它所属的文档的索引。
• `uid`：type和_id的组合键。
• `_id`：表示文档的ID。
### 2.2 文档源元字段
• `_source`：表示代表文档正文的原始JSON对象
• `size`：它表示source字段的大小（以字节为单位）
### 2.3 索引元字段
• `_field_names`：表示给定文档中包含非空值的所有字段。
• `_timestamp`：与每个文档相关联的手动或自动生成的时间戳。
• `_ttl`：表示应该保持活动状态的时间，之后该时间将被删除。
### 2.4 路由元字段
• `_parent`：当必须创建父子关系时，使用此方法。
• `_routing`：一个专有值，有助于将给定文档路由到指定的分片。
### 2.5 其他元字段
• `_meta`：表示应用程序特定的元数据。

## 3. 字段的数据类型

### 3.1 核心数据类型：
核心数据类型是大多数系统都可用并支持的基本数据类型，例如
• 整型 integer,
• 双精度浮点型 double,
• 长整型 long,
• 短整型 short,
• 字节类型 byte,
• 单精度浮点型 float,
• 字符串类型 string（text 和 keyword）,
• 布尔类型 Boolean,
• 日期类型 date,
• 二进制类型 binary。




### 3.2 复杂数据类型
复杂数据类型是核心数据类型的组合，如：
• 数组类型 arrays,
• JSON 对象类型：Object,
• 嵌套数据类型: nested。
数组类型需要多啰嗦几句。
第一： 任何类型都可以包含一个或者多个元素，当数据包含多个元素的时候，它就是数组类型。
第二：数据类型要求一个组内的数据类型一致。
实战举例如下：

```bash
PUT my_index_010/_doc/1
{
  "class_tags": [
    "新闻",
    "论坛",
    "博客",
    "电子报"
  ],
  "info_array": [
    {
      "name": "Mary",
      "age": 12
    },
    {
      "name": "John",
      "age": 10
    }
  ],
  "size_tags": [
    0,
    50,
    100
  ]
}
```

如上示例，定义三种数组类型：
• class_tags：媒体分类数组，数组元素是字符串类型
• info_array：个人信息数组，数组元素是object类型
• size_tags： 大小规模数组，数组元素是整型
### 3.3 Geo 数据类型
地理数据类型是用于保存详细信息（例如地点的地理位置）的数据类型，例如：
• geo_point用于标识纬度和经度。

### 3.4 专用数据类型
专用数据类型是那些具有本质上唯一的详细信息的数据类型，例如：
• IP地址，自动完成建议以及从字符串中计数令牌。
• completion， 补全建议导航搜索功能。

### 3.5 多字段类型multi-fields
上个例子，说明问题（考题中一种举例）。

```bash
PUT my_index_011
{
  "mappings": {
    "properties": {
      "cont": {
        "type": "text",
        "analyzer": "english",
        "fields": {
          "keyword": {
            "type": "keyword"
          },
          "stand":{
            "type":"text",
            "analyzer":"standard"
          }
        }
      }
    }
  }
}
```

上述demo定义了类型cont，使用english分词器，基于英文关键词全文检索。
同时为cont定义了两个扩展类型：
• 其一：keyword，用于精准匹配。
• 其二：standard，用于全文检索。
公司项目中实战，我们往往对需要全文检索的字段设置：text类型，并且指定：ik_max_word等中文分词器。
除此之外，如果这个字段还需要整个字段聚合或者排序等操作，我们往往还会设置其为keyword类型。
举例如下：

```bash
PUT my_index_012
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

以上，在你的公司里面是不是也很常见？
multi-fields是上面所有知识点的综合，一个知识点覆盖了Mapping 类型部分的绝大部分认知，所以是考试的重中之重。




## 3. Dynamic Mapping
**在写入文档的时候，如果索引不存在，会自动创建索引**

 - Dynamic Mapping 的机制，使得我们无需手动定义 Mappings。Elasticsearch
   会自动根据文档信息，推算出字段的类型
 - 但是会有时候推算不对。例如地理位置信息
 - 当类型如果设置不对时，会导致一些功能无法正常运行，例如 Range 查询
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103140315848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

## 4. 类型的自动识别
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103140404832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

   

```c
//写入文档
    PUT mapping_test/_doc/1
    {
      "firstName":"Lee",
      "lastName":"Crazy",
      "loginDate":"2019-10-22T21:08:48"
    }
    //查看Mapping 文件
    GET mapping_test/_mapping

//放回结果
    {
      "mapping_test" : {
        "mappings" : {
          "properties" : {
            "firstName" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "lastName" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "loginDate" : {
              "type" : "date"
            }
          }
        }
      }
    }
```



```c
//dynamic mapping 推断字符的类型
PUT mapping_test/_doc/1
{
  "uid":"123",
  "isVip": false,
  "isAdmin":"true",
  "age": 18,
  "heigh" : 180
}
GET mapping_test/_mapping
//返回结果
{
  "mapping_test" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "heigh" : {
          "type" : "long"
        },
        "isAdmin" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "isVip" : {
          "type" : "boolean"
        },
        "uid" : {
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
## 5. 能否更改 Mapping 的字段类型
两种情况
### 5.1 新增字段

 - Dynamic 设置为 true 时，一定有新增字段的文档写入，Mapping 也同时被更新
 - Dynamic 设为 false，Mapping 不会被更新，自增字段的数据无法被索引，但是信息会出现在_source 中
 - Dynamic 设置成 Strict 文档写入失败

### 5.2 已有字段

 - 一旦已经有数据写入，就不在支持修改字段定义
 - Luene 实现的倒排索引，一旦生成后，就不允许修改
 - 如果希望改变字段类型，必须 Reindex API，重建索引

原因

 - 如果修改了字段的数据类型，会导致已被索引的属于无法被搜索
 - 但是如果是增加新的字段，就不会有这样的影响


### 5.3 控制 Dynamic Mappings
|             | TRUE | FALSE | strict |
|-------------|------|-------|--------|
| 文档可索引       | YES  | YES   | NO     |
| 字段可索引       | YES  | NO    | NO     |
| Mapping 被更新 | YES  | NO    | NO     |

 - 当 dynamic 被设置成 false 时候，存在新增字段的数据写入，该数据可以被索引，当时新增字段被废弃
 - 当设置成 Strict 模式的时候，数据写入直接出错


```c
 //默认Mapping支持dynamic，写入的文档加入新的字段
  
  PUT dynamic_mapping_test/_doc/1
  {
    "newField":"someValue"
  }
  //能被搜索到
  POST dynamic_mapping_test/_search
  {
    "query": {
      "match": {
        "newField": "someValue"
      }
    }
  }
  ------------------------------------------------
  //修改为dynamic false
  PUT dynamic_mapping_test/_mapping
  {
    "dynamic":false
  }
  //新增anotherField 成功
  PUT dynamic_mapping_test/_doc/10
  {
    "anotherField":"someValue"
  }
  //重新去查询，但是anotherField 未被搜索到
  POST dynamic_mapping_test/_search
  {
    "query": {
      "match": {
        "anotherField": "someValue"
      }
    }
  }
  //查看mapping，无anotherField
  GET dynamic_mapping_test/_mapping
----------------------------------------------  
  //修改为dynamic strict
  PUT dynamic_mapping_test/_mapping
  {
    "dynamic": "strict"
  }
  //新增newField 报错
  PUT dynamic_mapping_test/_doc/12
  {
    "lastField":"value"
  }

```
## 6. 如何显示定义一个 Mapping

```bash
PUT movies
{
    "mappings" : {
        // define your mappings here
    }
}
```
自定义 Mapping 的一些建议:
1.可以参考 API 手册，纯手写
2.为了减少输入的工作量，减少出错率，依照以下步骤:
 - 创建一个临时的 index，写入一些样本数据
 - 通过访问 Mapping API 获得该临时文件的动态 Mapping 定义
 - 修改后用，使用该配置创建的索引
 - 删除临时索引

## 7. 控制当前字段是否被索引
index - 控制当前字段是否被索引。默认为 true。如果设置成 false，该字段不可被搜索

```bash
DELETE users
//定义mappings
PUT users
{
  "mappings" : {
    "properties" : {
      "firstName" : {
        "type" : "text"
      },
      "lastName" : {
        "type" : "text"
      },
      "mobile" : {
        "type" : "text",
        "index": false
      }
    }
  }
}
//写入数据
PUT users/_doc/1
{
  "firstname":"Ruan",
  "lastname":"Yiming",
  "mobile":12345678
}
//查询数据
POST /users/_search
{
  "query":{
    "match": {
      "mobile": "12345678"
    }
  }
}
返回内容为400，"reason": "Cannot search on field [mobile] since it is not indexed."
```

### 7.1 Index Options
四种不同级别的 Index Options 配置，可以控制倒排索引记录的内容

 - `docs` - 记录 doc id
 - `freqs` - 记录 doc id 和 term frequencies
 - `positions` - 记录 doc id /term frequencies /term position
 - `offsets` - doc id / term frequencies / term posistion / character offects

Text 类型默认记录 postions，其他默认为 docs
记录内容越多，占用存储空间越大

```bash
PUT users
{
  "mappings" : {
    "properties" : {
      "firstName" : {
        "type" : "text"
      },
      "lastName" : {
        "type" : "text"
      },
      "mobile" : {
        "type" : "text",
        "index": false
      },
      "bio": {
        "type" : "text",
        "index_options": "offsets"
      }
    }
  }
}
```

### 7.2 null_value

 - 需要对 NULL 值实现搜索
 - 只有 Keyword 类型支持设定 Null_Value

```bash
DELETE users
PUT users
{
  "mappings" : {
    "properties" : {
      "firstName" : {
        "type" : "text"
      },
      "lastName" : {
        "type" : "text"
      },
      "mobile" : {
        "type" : "keyword", //这个如果是text 无法设置为空
        "null_value": "NULL"
      }
    }
  }
}
//推送带有空值得数据
PUT users/_doc/2
{
"firstName":"Li",
"lastName": "Sunke",
"mobile": null
}

//查询mobile为null的数据
GET users/_search?q=mobile:NULL
或者
GET users/_search
{
  "query":{
    "match": {
      "mobile": "NULL"
    }
  }
}

//搜索结果
"_source" : {
        "firstName" : "Li",
        "lastName" : "Sunke",
        "mobile" : null
      }
```

### 7.3 copy_to

 - _all 在 7 中已经被 copy_to 所替代
 - 满足一些特定的搜索需求
 - copy_to 将字段的数值拷贝到目标字段，实现类似 _all 的作用
 - copy_to 的目标字段不出现在_source 中

```bash
DELETE users

//设定带copy_to的mapping
PUT users
{
"mappings": {
  "properties": {
    "firstName":{
      "type": "text",
      "copy_to": "fullName"
    },
    "lastName":{
      "type": "text",
      "copy_to": "fullName"
    }
  }
}
}

//写入数据
PUT users/_doc/1
{
"firstName":"Ruan",
"lastName": "Yiming"
}

//get查询
GET users/_search?q=fullName:(Ruan Yiming)
//返回一个值

//post查询
POST users/_search
{
  "query": {
    "match": {
      "fullName": {
        "query": "Ruan Yiming",
        "operator": "and"
      }
    }
  }
}
//返回一个值

```

### 7.4 数组类型
Elasticsearch 中不提供专门的数组类型。但是任何字段，都可以包含多个相同类型的数值.

```bash
DELETE users
PUT users/_doc/1
{
"name":"onebird",
"interests":"reading"
}
PUT users/_doc/1
{
"name":"twobirds",
"interests":["reading","music"]
}
GET users/_mapping 
//部分代码
"interests" : {
        "type" : "text", //类型还是text，不是数组
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        }
      }
```

##  8. 动态映射的弊端
### 8.1 字段匹配不准确
如：date类型匹配为keyword类型。
举例：

```bash
DELETE my_index_014
PUT my_index_014/_doc/1
{
  "create_date": "2020-12-26 12:00:00"
}
GET my_index_014/_mapping
```

返回结果可知：create_date 是 text和keyword类型。不是我们期望的date类型。
是有解决方案的，如下，但也需要提前设置匹配规则。

```bash
DELETE my_index_014
PUT my_index_014
{
  "mappings": {
    "dynamic_date_formats": ["yyyy-MM-dd HH:mm:ss"]
  }
}
```

### 8.2 字段匹配相对准确，但不是用户期望的。
举例: 用户期望text类型支持ik分词，但默认的是standard标准分词器。
当然也会有解决方案，借助动态 template解决。
### 8.3 占据多余的存储空间。
举例：string类型匹配为：text和keyword两种类型。意味着两次索引。
但实际用户极有可能只期望排序和聚合的keyword类型。
或者极有可能只需要存储text类型，如：网页正文内容只需要全文检索，不需要排序和聚合操作。
### 8.4 Mapping可能错误泛滥。
不小心写错的查询语句，由于使用了put操作，很可能写入Mapping中也经常见。
基于此，实际工程开发实战中，建议：使用静态Mapping，提前定义好字段。



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


