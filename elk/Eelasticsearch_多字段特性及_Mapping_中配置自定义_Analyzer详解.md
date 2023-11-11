
## 1. 多字段特性

1.厂家名字实现精确匹配

 - 增加一个 keyword 字段

2.使用不同的 analyzer

 - 不同语言
 - pinyin 字段的搜索
 - 还支持为搜索和索引指定不同的 analyzer
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103190052914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 2. Excat values(精词) v.s Full Text（全文本）
 - Excat Values ：包括数字 / 日期 / 具体一个字符串 （例如 “Apple Store”）
**Elasticsearch 中的 keyword**
 - 全文本，非结构化的文本数据
**Elasticsearch 中的 text**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103190229443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 3. Excat values不需要分词
Elaticsearch 为每一个字段创建一个倒排索引
Exact Value 在索引时，不需要做特殊的分词处理

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103193323356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 4. 自定义分词器
当 Elasticsearch 自带的分词器无法满足时，可以自定义分词器。通过自组合不同的组件实现：

 - Character Filter
 - Tokenizer
 - Token Filter

### 4.1 Character Filters
在 `Tokenizer` 之前对文本进行处理，例如增加删除及替换字符。可以配置多个 `Character Filters`。会影响 Tokenizer 的 position 和 offset 信息
 一些自带的 Character Filters：
 - `HTML strip` - 去除 html 标签
 - `Mapping` - 字符串替换
 - `Pattern replace` - 正则匹配替换

### 4.2 Tokenizer

 - 将原始的文本按照一定的规则，切分为词（term or token）
 - Elasticsearch 内置的 Tokenizers：`whitespace` | `standard` | `uax_url_email` | `pattern` | `keyword` | `path hierarchy`
 - 可以用 JAVA 开发插件，实现自己的 Tokenizer

### 4.3 Token Filters

 - 将 Tokenizer 输出的单词，进行增加、修改、删除
 - 自带的 Token Filters：`Lowercase` |`stop`| `synonym`（添加近义词）

## 5. Tokenizer
### 5.1 Tokenizer与char_filter

#### 5.1.1 使用keyword与html_strip方法处理网页

```bash
POST _analyze
{
"tokenizer":"keyword",
"char_filter":["html_strip"],
"text": "<b>hello world</b>"
}
//结果
{
"tokens" : [
  {
    "token" : "hello world",
    "start_offset" : 3,
    "end_offset" : 18,
    "type" : "word",
    "position" : 0
  }
]
}
```

#### 5.1.2 使用 `standard`与`mapping` 方法进行替换

standard:按词切分，小写处理

```bash
POST _analyze
{
"tokenizer": "standard",
"char_filter": [
    {
      "type" : "mapping",
      "mappings" : [ "- => _"]
    }
  ],
"text": "123-456, I-test! test-990 650-555-1234"
}
//返回
{
"tokens" : [
  {
    "token" : "123_456",
    "start_offset" : 0,
    "end_offset" : 7,
    "type" : "<NUM>",
    "position" : 0
  },
  {
    "token" : "I_test",
    "start_offset" : 9,
    "end_offset" : 15,
    "type" : "<ALPHANUM>",
    "position" : 1
  },
  {
    "token" : "test_990",
    "start_offset" : 17,
    "end_offset" : 25,
    "type" : "<ALPHANUM>",
    "position" : 2
  },
  {
    "token" : "650_555_1234",
    "start_offset" : 26,
    "end_offset" : 38,
    "type" : "<NUM>",
    "position" : 3
  }
]
}
```

#### 5.1.3 standard与mapping替换表情符号
 

```bash
 POST _analyze
{
"tokenizer": "standard",
"char_filter": [
    {
      "type" : "mapping",
      "mappings" : [ ":) => happy", ":( => sad"]
    }
  ],
  "text": ["I am felling :)", "Feeling :( today"]
}
//返回
{
"tokens" : [
  {
    "token" : "I",
    "start_offset" : 0,
    "end_offset" : 1,
    "type" : "<ALPHANUM>",
    "position" : 0
  },
  {
    "token" : "am",
    "start_offset" : 2,
    "end_offset" : 4,
    "type" : "<ALPHANUM>",
    "position" : 1
  },
  {
    "token" : "felling",
    "start_offset" : 5,
    "end_offset" : 12,
    "type" : "<ALPHANUM>",
    "position" : 2
  },
  {
    "token" : "happy",
    "start_offset" : 13,
    "end_offset" : 15,
    "type" : "<ALPHANUM>",
    "position" : 3
  },
  {
    "token" : "Feeling",
    "start_offset" : 16,
    "end_offset" : 23,
    "type" : "<ALPHANUM>",
    "position" : 104
  },
  {
    "token" : "sad",
    "start_offset" : 24,
    "end_offset" : 26,
    "type" : "<ALPHANUM>",
    "position" : 105
  },
  {
    "token" : "today",
    "start_offset" : 27,
    "end_offset" : 32,
    "type" : "<ALPHANUM>",
    "position" : 106
  }
]
}
```

#### 5.1.4 standard与正则表达式pattern_replace

```bash
GET _analyze
{
"tokenizer": "standard",
"char_filter": [
    {
      "type" : "pattern_replace",
      "pattern" : "http://(.*)",
      "replacement" : "$1"
    }
  ],
  "text" : "http://www.elastic.co"
}
//返回
{
"tokens" : [
  {
    "token" : "www.elastic.co",
    "start_offset" : 0,
    "end_offset" : 21,
    "type" : "<ALPHANUM>",
    "position" : 0
  }
]
}
```
### 5.2 tokenizer与text

 - 通过路劲切分

```bash
POST _analyze
{
"tokenizer":"path_hierarchy",
"text":"/user/ymruan/a"
}
//返回
{
"tokens" : [
  {
    "token" : "/user",
    "start_offset" : 0,
    "end_offset" : 5,
    "type" : "word",
    "position" : 0
  },
  {
    "token" : "/user/ymruan",
    "start_offset" : 0,
    "end_offset" : 12,
    "type" : "word",
    "position" : 0
  },
  {
    "token" : "/user/ymruan/a",
    "start_offset" : 0,
    "end_offset" : 14,
    "type" : "word",
    "position" : 0
  }
]
}
```
### 5.3 tokenizer与filter（token_filters）

```bash
GET _analyze
{
"tokenizer": "whitespace", 
"filter": ["stop"], //on the a
"text": ["The gilrs in China are playing this game!"]
}

//返回，保留名词复数
{
  "tokens" : [
    {
      "token" : "The",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "gilrs",
      "start_offset" : 4,
      "end_offset" : 9,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "China",
      "start_offset" : 13,
      "end_offset" : 18,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "playing",
      "start_offset" : 23,
      "end_offset" : 30,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "game!",
      "start_offset" : 36,
      "end_offset" : 41,
      "type" : "word",
      "position" : 7
    }
  ]
}
```

`snowball`将名称复数变为单数
```bash
GET _analyze
{
"tokenizer": "whitespace", 
"filter": ["stop","snowball"], //on the a
"text": ["The gilrs in China are playing this game!"]
}
{
"tokens" : [
  {
    "token" : "The", //大写的The 不做过滤
    "start_offset" : 0,
    "end_offset" : 3,
    "type" : "word",
    "position" : 0
  },
  {
    "token" : "gilr",
    "start_offset" : 4,
    "end_offset" : 9,
    "type" : "word",
    "position" : 1
  },
  {
    "token" : "China",
    "start_offset" : 13,
    "end_offset" : 18,
    "type" : "word",
    "position" : 3
  },
  {
    "token" : "play",
    "start_offset" : 23,
    "end_offset" : 30,
    "type" : "word",
    "position" : 5
  },
  {
    "token" : "game!",
    "start_offset" : 36,
    "end_offset" : 41,
    "type" : "word",
    "position" : 7
  }
]
}
```
加入 `lowercase` 后，The 被当成 stopword 删除

```bash
GET _analyze
{
"tokenizer": "whitespace",
"filter": ["lowercase","stop","snowball"],
"text": ["The gilrs in China are playing this game!"]
}
{
"tokens" : [
  {
    "token" : "gilr",
    "start_offset" : 4,
    "end_offset" : 9,
    "type" : "word",
    "position" : 1
  },
  {
    "token" : "china",
    "start_offset" : 13,
    "end_offset" : 18,
    "type" : "word",
    "position" : 3
  },
  {
    "token" : "play",
    "start_offset" : 23,
    "end_offset" : 30,
    "type" : "word",
    "position" : 5
  },
  {
    "token" : "game!",
    "start_offset" : 36,
    "end_offset" : 41,
    "type" : "word",
    "position" : 7
  }
]
}
```
## 6. 设置标准的 analyzer
### 6.1 官网自定义分词器的标准格式

```bash
PUT /my_index
{
  "settings": {
      "analysis": {
          "char_filter": { ... custom character filters ... },//字符过滤器
          "tokenizer": { ... custom tokenizers ... },//分词器
          "filter": { ... custom token filters ... }, //词单元过滤器
          "analyzer": { ... custom analyzers ... }
      }
  }
}
```


### 6.2 定义自己的分词器

```bash
PUT my_index
{
"settings": {
  "analysis": {
    "analyzer": {
      "my_custom_analyzer":{
        "type":"custom",
        "char_filter":[
          "emoticons"
        ],
        "tokenizer":"punctuation",
        "filter":[
          "lowercase",
          "english_stop"
        ]
      }
    },
    "tokenizer": {
      "punctuation":{
        "type":"pattern",
        "pattern": "[ .,!?]"
      }
    },
    "char_filter": {
      "emoticons":{
        "type":"mapping",
        "mappings" : [ 
          ":) => happy",
          ":( => sad"
        ]
      }
    },
    "filter": {
      "english_stop":{
        "type":"stop",
        "stopwords":"_english_"
      }
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
