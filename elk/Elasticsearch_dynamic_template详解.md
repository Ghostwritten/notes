

----
## 1. 介绍
动态模板允许你定义可以用于动态添加的字段的自定义映射：

 - 由Elasticsearch中的`match_mapping_type`检测到数据类型。
 - 字段的名称可以是`match`(匹配)和`unmatch`(不匹配)或match_pattern(模式匹配)。
 - 全点路径的字段可以是`path_match`(路径匹配)和`path_unmatch`(不匹配路径)。

原始字段名称{name}和检测导的数据类型{dynamic_type}模板变量可以在映射规范中用作占位符。
 - 所有的字符串类型都设定称 Keyword，或者关闭 keyword 字段
 - is 开头的字段都设置成 boolean
 - long_ 开头的都设置成 long 类型

模板按顺序进行处理-第一个匹配模板达到要求。可以使用PUT mapping API将新的模板附加到列表的尾部。如果新的模板与现有的模板同名，它将会替换旧的版本。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71166239720d2cdfe023237f8e4177cd.png#pic_center)
## 2. 匹配规则参数
### 2.1 match_mapping_type（匹配映射类型）
`match_mapping_type`通过动态字段映射检测数据类型的匹配，换句话说，就是Elasticsearch认为该字段应该具有的数据类型。只能自动检测以下数据类型：`boolean`(布尔类型),`date`(日期),`double`(浮点型),`long`(长整型),`object`(对象类型),`string`(字符类型)。同时，它也接受*来匹配所有数据类型。

例如，如果我们要将所有整数字段映射为`integer`(整型)而不是`long`(长整型),并且所有string(字符串类型)字段都是text(文本)和keyword(关键词),我们可以使用以下模板：

```bash
PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_templates": [
        {
          "integers": {
            "match_mapping_type": "long",
            "mapping": {
              "type": "integer"
            }
          }
        },
        {
          "strings": {
            "match_mapping_type": "string",
            "mapping": {
              "type": "text",
              "fields": {
                "raw": {
                  "type":  "keyword",
                  "ignore_above": 256
                }
              }
            }
          }
        }
      ]
    }
  }}

PUT my_index/my_type/1{
  "my_integer": 5,                 #1
  "my_string": "Some string" }     #2
```
`my_integer`字段被映射为integer(整数类型)。
`my_string`字段被映射为text(文本类型),并且是keword(关键字)多字段


### 2.2 match and unmatch（匹配和不匹配）
match(匹配)参数使用模式匹配字段名称，而unmatch(不匹配)使用模式排除match(匹配)的字段。

以下示例匹配名称为`long`_开头(除了以_text结尾的字段字符串除外)的所有`string`(字符串类型)字段，并将其映射为long(长整型)字段：

```bash
PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_templates": [
        {
          "longs_as_strings": {
            "match_mapping_type": "string",
            "match":   "long_*",
            "unmatch": "*_text",
            "mapping": {
              "type": "long"
            }
          }
        }
      ]
    }
  }}

PUT my_index/my_type/1{
  "long_num": "5",              #1
  "long_text": "foo" }          #2
```
 `long_num`字段被映射为long(长整型)。 
 `long_text`字段使用默认的`string`(字符串类型)映射

### 2.3 match_pattern（模式匹配）
`match_pattern`参数调整match参数的行为，使其在字段名称上支持匹配完整的Java正则表达式，而不是简单的通配符，例如：

```bash
  "match_pattern": "regex",
  "match": "^profit_\d+$"
```
### 2.4 path_match and path_unmatch(路径匹配和不匹配路径)
`path_match`和`path_unmatch`参数的工作方式与`match`和`unmath`相同，但是字段运行在完整的路径上，而不仅仅是最终的名称，例如：`some_object.*.some_field`。

此示例将name对象中的任何字段的值复制到顶级full_name字段，但middle字段除外：

```bash
PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_templates": [
        {
          "full_name": {
            "path_match":   "name.*",
            "path_unmatch": "*.middle",
            "mapping": {
              "type":       "text",
              "copy_to":    "full_name"
            }
          }
        }
      ]
    }
  }}

PUT my_index/my_type/1{
  "name": {
    "first":  "Alice",
    "middle": "Mary",
    "last":   "White"  }}
```

### 2.5 {name} and {dynamic_type}
在映射中{name}和{dynamic_type}占位符会被替换为字段名称和检测到的动态类型。以下示例将所有的字符串类型设置为使用与该字段名称相同的`analyzer`(分析器)，并禁用所有非字符串字段的`doc_values`：

```bash
PUT my_index
{
  "mappings": {
    "my_type": {
      "dynamic_templates": [
        {
          "named_analyzers": {
            "match_mapping_type": "string",
            "match": "*",
            "mapping": {
              "type": "text",
              "analyzer": "{name}"
            }
          }
        },
        {
          "no_doc_values": {
            "match_mapping_type":"*",
            "mapping": {
              "type": "{dynamic_type}",
              "doc_values": false
            }
          }
        }
      ]
    }
  }}

PUT my_index/my_type/1{

  "english": "Some English text",            #1
  "count":   5 }                             #2
```

`english`字段通过`english`分析器映射为`string`（字符串）字段。
 count字段通过禁用的doc_values映射为long（长整型）字段。 

##  3. 实例
```bash
DELETE my_index
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
            {
        "strings_as_boolean": {
          "match_mapping_type":   "string",
          "match":"is*",
          "mapping": {
            "type": "boolean"
          }
        }
      },
      {
        "strings_as_keywords": {
          "match_mapping_type":   "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}

PUT my_index/_doc/1
{
  "firstName":"Li",
  "isVIP":"true"
}
GET my_index/_mapping
//返回部分
"properties" : {
        "firstName" : {
          "type" : "keyword"
        },
        "isVIP" : {
          "type" : "boolean"
        }
      }
```
设置新的 `my_index`

```bash
DELETE my_index
#结合路径
PUT my_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "full_name": {
          "path_match":   "name.*",
          "path_unmatch": "*.middle",
          "mapping": {
            "type":       "text",
            "copy_to":    "full_name"
          }
        }
      }
    ]
  }
}
PUT my_index/_doc/1
{
  "name": {
    "first":  "John",
    "middle": "Winston",
    "last":   "Lennon"
  }
}
//可查询到
GET my_index/_search?q=full_name:John

//返回
{
  "took" : 7,
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
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "name" : {
            "first" : "John",
            "middle" : "Winston",
            "last" : "Lennon"
          }
        }
      }
    ]
  }
}

```



