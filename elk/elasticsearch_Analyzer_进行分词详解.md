

----
## 1. Anaiysis 与 Analyzer

 - Analysis - 文本分析是吧全文本转换成一系列的单词（term /token）的过程，也叫分词
 - Analysis 是通过 Analyzer 来实现的，可使用 Elasticesearch 内置的分析器 或者按需求定制化分析器
 - 除了在数据写入时转换词条，匹配 Query 语句时候也需要用相同的分析器会查询语句进行分析
## 2. Analyzer 的组成
分词器是专门处理分词的组件，Analyzer 由三部分组成
 - `Character Filters` （针对原始文本处理，例如去除 html）
 - `Tokenizer`（按照规则切分为单词）
 - `Token Filter` （将切分的单词进行加工，小写，删除 stopwords，增加同义语）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201102140051984.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 3. Elastocsearch 的内置分词器
 - `Standard Analyzer` - 默认分词器，按词切分，小写处理
 - `Simple Analyzer` - 按照非字母切分（符号被过滤），小写处理
 - `Stop Analyzer` - 小写处理，停用词过滤（the ，a，is）
 - `Whitespace Analyzer` - 按照空格切分，不转小写
 - `Keyword Analyzer` - 不分词，直接将输入当做输出
 - `Pattern Analyzer` - 正则表达式，默认 \W+
 - `Language` - 提供了 30 多种常见语言的分词器
 - `Customer Analyzer` 自定义分词器
### 3.1 使用 _analyzer Api
#### 3.1.1 直接指定 Analyzer 进行测试

```bash
GET _analyze
 {
   "analyzer": "standard",
   "text" : "Mastering Elasticsearch , elasticsearch in Action"
 }
 //返回结果
 {
   "tokens" : [
     {
       "token" : "mastering",
       "start_offset" : 0,
       "end_offset" : 9,
       "type" : "<ALPHANUM>",
       "position" : 0
     },
     {
       "token" : "elasticsearch",
       "start_offset" : 10,
       "end_offset" : 23,
       "type" : "<ALPHANUM>",
       "position" : 1
     },
     {
       "token" : "elasticsearch",
       "start_offset" : 26,
       "end_offset" : 39,
       "type" : "<ALPHANUM>",
       "position" : 2
     },
     {
       "token" : "in",
       "start_offset" : 40,
       "end_offset" : 42,
       "type" : "<ALPHANUM>",
       "position" : 3
     },
     {
       "token" : "action",
       "start_offset" : 43,
       "end_offset" : 49,
       "type" : "<ALPHANUM>",
       "position" : 4
     }
   ]
 }
```

#### 3.1.2 指定索引的字段进行测试

```bash
POST books/_analyze
{
"field": "title",
"text": "Mastering Elasticesearch"
}
```
#### 3.1.3 自定义分词进行测试

```bash
  POST /_analyze
  {
    "tokenizer": "standard", 
    "filter": ["lowercase"],
    "text": "Mastering Elasticesearch"
  }
  //结果返回
  {
    "tokens" : [
      {
        "token" : "mastering",
        "start_offset" : 0,
        "end_offset" : 9,
        "type" : "<ALPHANUM>",
        "position" : 0
      },
      {
        "token" : "elasticesearch",
        "start_offset" : 10,
        "end_offset" : 24,
        "type" : "<ALPHANUM>",
        "position" : 1
      }
    ]
  }
```
### 3.2 Standard Analyzer

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201102141234399.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

```bash
#standard
GET _analyze
{
"analyzer": "standard",
"text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}   
```

### 3.3  Simple Analyzer
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201102141424270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

```bash
#simple 去除非字母的 ：2 -  xi
GET _analyze
{
"analyzer": "simple",
"text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

### 3.4 Whitespace Analyzer
![空格切分](https://img-blog.csdnimg.cn/20201102141656639.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)


```bash
#stop
GET _analyze
{
"analyzer": "whitespace",
"text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

#return

{
  "tokens" : [
    {
      "token" : "2",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "Quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "brown-foxes",
      "start_offset" : 16,
      "end_offset" : 27,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "in",
      "start_offset" : 48,
      "end_offset" : 50,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "the",
      "start_offset" : 51,
      "end_offset" : 54,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "evening.",
      "start_offset" : 62,
      "end_offset" : 70,
      "type" : "word",
      "position" : 11
    }
  ]
}

```
### 3.5  Stop Analyzer
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201102141731268.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

```bash
GET _analyze
{
"analyzer": "stop",
"text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

#return
{
  "tokens" : [
    {
      "token" : "running",
      "start_offset" : 2,
      "end_offset" : 9,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "quick",
      "start_offset" : 10,
      "end_offset" : 15,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "brown",
      "start_offset" : 16,
      "end_offset" : 21,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "foxes",
      "start_offset" : 22,
      "end_offset" : 27,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "leap",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 4
    },
    {
      "token" : "over",
      "start_offset" : 33,
      "end_offset" : 37,
      "type" : "word",
      "position" : 5
    },
    {
      "token" : "lazy",
      "start_offset" : 38,
      "end_offset" : 42,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "dogs",
      "start_offset" : 43,
      "end_offset" : 47,
      "type" : "word",
      "position" : 7
    },
    {
      "token" : "summer",
      "start_offset" : 55,
      "end_offset" : 61,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "evening",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "word",
      "position" : 11
    }
  ]
}

```

### 3.6  Keyword Analyzer
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201102141834259.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

```bash
#keyword
GET _analyze
{
"analyzer": "keyword",
"text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

#return
{
  "tokens" : [
    {
      "token" : "2 running Quick brown-foxes leap over lazy dogs in the summer evening.",
      "start_offset" : 0,
      "end_offset" : 70,
      "type" : "word",
      "position" : 0
    }
  ]
}
```

### 3.7 Pattern Analyzer
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201102141914186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

```bash
GET _analyze
{
"analyzer": "pattern",
"text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```
### 3.8  Language Analyzer
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020110214200235.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

```bash
#english
GET _analyze
{
"analyzer": "english",
"text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

## 4. 中文分词的难点

 - 中文句子，切分成一个一个次（不是一个个字）
 - 英文中，单词有自然的空格作为分隔
 - 一句中文，在不同的上下文，有不同的理解：
  1. 这个苹果，不大好吃 / 这个苹果，不大，好吃！
2. 他说的确实在理 / 这事的确定不下来

### 4.1 ICU Analyzer
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020110214243960.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
安装的 es 安装插件

```bash
    cd /var/docker/docker-es-7.3/
    docker exec -it es7_01 bash
    bin/elasticsearch-plugin install analysis-icu
    exit
    docker exec -it es7_02 bash
    bin/elasticsearch-plugin install analysis-icu
    exit
    docker-compose restart

#icu analyzer    
POST _analyze
{
  "analyzer": "icu_analyzer",
  "text": "他说的确实在理”"
}
```
### 4.2 更多的中文分词器
IK
支持自定义词库，支持热更新分词字典
https://github.com/medcl/elasticsearch-ana...
THULAC
THU Lexucal Analyzer for Chinese, 清华大学自然语言处理和社会人文计算实验室的一套中文分词器
https://github.com/microbun/elasticearch-t...


参考资料：
极客时间：Elasticsearch核心技术与实战
相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
[docker-compose安装elasticsearch集群](https://blog.csdn.net/xixihahalelehehe/article/details/109389780)
[Elasticsearch 7.X之文档、索引、REST API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406518)
[Elasticsearch节点，集群，分片及副本详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406875)
[elasticsearch倒排索引介绍](https://blog.csdn.net/xixihahalelehehe/article/details/109440345)

