

## 1. 什么是数据建模

 - 数据建模（Data modeling），是创建数据模型的过程。
 - 数据模型是对真实世界进行抽象描述的一种工具和方法，实现对现实世界的映射
 - 博客、作者、用户评论
 - 三个过程：概念模型 =》逻辑模型 =》数据模型（第三范式）
 - 数据模型：结合具体的数据库，在满足业务读写性能等需求的前提下，确定最终的定义


## 2. 数据建模：功能需求 + 性能需求
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310142825453.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 3. 如何对字段进行建模
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031014284598.png)
## 4. 字段类型 ： Text v.s Keyword
Text

 - 用于全文本字段，文本会被 Analyzer 分词
 - 默认不支持聚合分析及排序。需要设置 fielddata 为 true

Keyword

 - 用于 id ，枚举及不需要分词的文本。例如电话号码，email 地址，手机号码，邮政编码，性别等
 - 适用于 Filter（精确匹配），Sorting 和 Aggregations

设置多字段类型

 - 默认会为文本类型设置成 text ，并且设置一个 keyword 的子字段
 - 在处理人类语言时，通过增加 “英文”，“拼音” 和 “标准” 分词器，提高搜索结构

## 5. 字段类型：结构化数据
数据类型

 - 尽量选择贴近的类型。例如可以用 byte，就不要用 long

枚举类型

 - 设置为 keyword 。即便是数字，也应该设置成 keyword ，获取更好的性能

其他

 - 日期、布尔、地理信息

## 6. 检索
如不需要检索，排序和聚合分析

 - Enable 设置成 false

如不需要检索

 - index 设置成 false

对需要检索的字段，可以通过如下配置，设置存储粒度

 - Index_options / Norms : 不需要归一化数据时，可以关闭

## 7. 聚合及排序
如不需要检索，排序和聚合分析

 - Enable 设置成 false

如不需要排序或者聚合分析功能

 - Doc_values /fielddata 设置成 false

更新频繁，聚合查询频繁的 keyword 类型的字段

 - 推荐将 eager_global_ordinals 设置为 true


## 8. 额外的存储
是否需要专门存储当前字段数据

 - Store 设置为 true ，可以存储该字段的原始数据
 - 一般结合 _source 的 enabled 为 false 时候使用

Disable_source ： 节约磁盘，适用于指标型数据

 - 一般建议先考虑增加压缩比
 - 无法看到 _source 字段，无法做 ReIndex，无法做 Update
 - Kibana 中无法做 discovery

## 9. 一个数据建模的实例
图书的索引

 - 书名
 - 简介
 - 作者
 - 发行日期
 - 图书封面
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310143358567.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
优化片段设定
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310145607502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
PUT books/_doc/1
{
  "title":"Mastering ElasticSearch 5.0",
  "description":"Master the searching, indexing, and aggregation features in ElasticSearch Improve users’ search experience with Elasticsearch’s functionalities and develop your own Elasticsearch plugins",
  "author":"Bharvi Dixit",
  "public_date":"2017",
  "cover_url":"https://images-na.ssl-images-amazon.com/images/I/51OeaMFxcML.jpg"
}
GET books/_mapping
# return 
{
  "books" : {
    "mappings" : {
      "properties" : {
        "author" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "cover_url" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "description" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "public_date" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
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
}

#优化字段类型
PUT books
{
      "mappings" : {
      "properties" : {
        "author" : {"type" : "keyword"},
        "cover_url" : {"type" : "keyword","index": false},
        "description" : {"type" : "text"},
        "public_date" : {"type" : "date"},
        "title" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 100
            }
          }
        }
      }
    }
}

#Cover URL index 设置成false，无法对该字段进行搜索
POST books/_search
{
  "query": {
    "term": {
      "cover_url": {
        "value": "https://images-na.ssl-images-amazon.com/images/I/51OeaMFxcML.jpg"
      }
    }
  }
}

#Cover URL index 设置成false，依然支持聚合分析
POST books/_search
{
  "aggs": {
    "cover": {
      "terms": {
        "field": "cover_url",
        "size": 10
      }
    }
  }
}



```

## 10. 优化需求变更
新需求：增加图书内容的字段，并要求能被搜索同时支持高亮显示
新需求会导致 _source 的内容过大

 - Source Filtering 只是传输给客户端进行过滤， Fetch 数据时， ES 节点还是会传输 _source 中的数据

解决方法

 - 关闭 _source
 - 然后将每个字段的 “store” 设置成 true

```bash
DELETE books
#新增 Content字段。数据量很大。选择将Source 关闭
PUT books
{
  "mappings" : {
    "_source": {"enabled": false},
    "properties" : {
      "author" : {"type" : "keyword","store": true},
      "cover_url" : {"type" : "keyword","index": false,"store": true},
      "description" : {"type" : "text","store": true},
       "content" : {"type" : "text","store": true},
      "public_date" : {"type" : "date","store": true},
      "title" : {
        "type" : "text",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 100
          }
        },
        "store": true
      }
    }
  }
}





```

## 11. 查询图书：解决字段过大引发的性能问题
返回结果不包含 _source 字段
对于需要显示的信息，可以在查询中指定 “store_fields”
禁止 _source 字段后，还是支持使用 hignlights API ，高亮显示 content 中的匹配的相关信息

```bash
# Index 一本书的信息,包含Content
PUT books/_doc/1
{
  "title":"Mastering ElasticSearch 5.0",
  "description":"Master the searching, indexing, and aggregation features in ElasticSearch Improve users’ search experience with Elasticsearch’s functionalities and develop your own Elasticsearch plugins",
  "content":"The content of the book......Indexing data, aggregation, searching.    something else. something in the way............",
  "author":"Bharvi Dixit",
  "public_date":"2017",
  "cover_url":"https://images-na.ssl-images-amazon.com/images/I/51OeaMFxcML.jpg"
}

#查询结果中，Source不包含数据
POST books/_search
{}

#搜索，通过store 字段显示数据，同时高亮显示 conent的内容
POST books/_search
{
  "stored_fields": ["title","author","public_date"],
  "query": {
    "match": {
      "content": "searching"
    }
  },

  "highlight": {
    "fields": {
      "content":{}
    }
  }
}

返回输出：
{
  "took" : 59,
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
        "_index" : "books",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "fields" : {
          "title" : [
            "Mastering ElasticSearch 5.0"
          ],
          "public_date" : [
            "1970-01-01T00:00:02.017Z"
          ],
          "author" : [
            "Bharvi Dixit"
          ]
        },
        "highlight" : {
          "content" : [
            "The content of the book......Indexing data, aggregation, <em>searching</em>."
          ]
        }
      }
    ]
  }
}
```
## 12. Mapping 字段的相关设置
[https://www.elastic.co/guide/en/elasticsea...](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html)

![映射](https://img-blog.csdnimg.cn/20210329163518920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210329163539435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 13. 一些相关的 API
Index Template & Dynamic Template

 - 根据索引的名字匹配不同的 Mappings 和 Settings
 - 可以在⼀个 Mapping 上动态的设定字段类型

Index Alias

 - ⽆需停机，⽆需修改程序，即可进⾏修改

Update By Query & Reindex
数据建模对功能与性能⾄关重要

## 14. ES 万能Mapping 模板参考
以下的索引 Mapping中，_source设置为false，同时各个字段的store根据需求设置了true和false。
url的doc_values设置为false，该字段url不用于聚合和排序操作。

```bash
PUT blog_index
{
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 100
          }
        },
        "store": true
      },
      "publish_date": {
        "type": "date",
        "store": true
      },
      "author": {
        "type": "keyword",
        "ignore_above": 100,
        "store": true
      },
      "abstract": {
        "type": "text",
        "store": true
      },
      "content": {
        "type": "text",
        "store": true
      },
      "url": {
        "type": "keyword",
        "doc_values": false,
        "norms": false,
        "ignore_above": 100,
        "store": true
      }
    }
  }
}
```

## 15. 不可回避——ES多表关联
实际业务问题：多层数据结构，一对多关系，如何用一个查询查询所有的数据？
比如数据结构如下：帖子--帖子评论--评论用户  3层。
现在需要查询一条帖子，最好能查询到帖子下的评论，还有评论下面的用户数据，一个查询能搞定吗？目前两层我可以查询到，3层就不行了。
如果一次查询不到，那如何设计数据结构？又应该如何查询呢？
目前ES主要有以下4种常用的方法来处理数据实体间的关联关系：
## 16. Application-side joins（服务端Join或客户端Join）
这种方式，索引之间完全独立（利于对数据进行标准化处理，如便于上述两种增量同步的实现），由应用端的多次查询来实现近似关联关系查询。这种方法适用于第一个实体只有少量的文档记录的情况（使用ES的terms查询具有上限，默认1024，具体可在elasticsearch.yml中修改），并且最好它们很少改变。这将允许应用程序对结果进行缓存，并避免经常运行第一次查询。
## 17. Data denormalization（数据的非规范化）
这种方式，通俗点就是通过字段冗余，以一张大宽表来实现粗粒度的index，这样可以充分发挥扁平化的优势。但是这是以牺牲索引性能及灵活度为代价的。使用的前提：冗余的字段应该是很少改变的；比较适合与一对少量关系的处理。当业务数据库并非采用非规范化设计时，这时要将数据同步到作为二级索引库的ES中，就很难使用上述增量同步方案，必须进行定制化开发，基于特定业务进行应用开发来处理join关联和实体拼接。
ps：宽表处理在处理一对多、多对多关系时，会有字段冗余问题，适合“一对少量”且这个“一”更新不频繁的应用场景。宽表化处理，在查询阶段如果只需要“一”这部分时，需要进行结果去重处理（可以使用ES5.x的字段折叠特性，但无法准确获取分页总数，产品设计上需采用上拉加载分页方式）
## 18. Nested objects（嵌套文档）
索引性能和查询性能二者不可兼得，必须进行取舍。嵌套文档将实体关系嵌套组合在单文档内部（类似与json的一对多层级结构），这种方式牺牲索引性能（文档内任一属性变化都需要重新索引该文档）来换取查询性能，可以同时返回关系实体，比较适合于一对少量的关系处理。
ps: 当使用嵌套文档时，使用通用的查询方式是无法访问到的，必须使用合适的查询方式（nested query、nested filter、nested facet等），很多场景下，使用嵌套文档的复杂度在于索引阶段对关联关系的组织拼装。
## 19. Parent/child relationships（父子文档）
父子文档牺牲了一定的查询性能来换取索引性能，适用于一对多的关系处理。其通过两种type的文档来表示父子实体，父子文档的索引是独立的。父-子文档ID映射存储在 Doc Values 中。当映射完全在内存中时， Doc Values 提供对映射的快速处理能力，另一方面当映射非常大时，可以通过溢出到磁盘提供足够的扩展能力。 在查询parent-child替代方案时，发现了一种filter-terms的语法，要求某一字段里有关联实体的ID列表。基本的原理是在terms的时候，对于多项取值，如果在另外的index或者type里已知主键id的情况下，某一字段有这些值，可以直接嵌套查询。具体可参考官方文档的示例：通过用户里的粉丝关系，微博和用户的关系，来查询某个用户的粉丝发表的微博列表。
ps：父子文档相比嵌套文档较灵活，但只适用于“一对大量”且这个“一”不是海量的应用场景，该方式比较耗内存和CPU，这种方式查询比嵌套方式慢5~10倍，且需要使用特定的has_parent和has_child过滤器查询语法，查询结果不能同时返回父子文档（一次join查询只能返回一种类型的文档）。而受限于父子文档必须在同一分片上，ES父子文档在滚动索引、多索引场景下对父子关系存储和联合查询支持得不好，而且子文档type删除比较麻烦（子文档删除必须提供父文档ID）。
如果业务端对查询性能要求很高的话，还是建议使用宽表化处理*的方式，这样也可以比较好地应对聚合的需求。在索引阶段需要做join处理，查询阶段可能需要做去重处理，分页方式可能也得权衡考虑下。

参考连接：
[https://www.yuque.com/books/share/69ed7241-26ee-4784-983a-c4eaf56469db/gm4s19](https://www.yuque.com/books/share/69ed7241-26ee-4784-983a-c4eaf56469db/gm4s19)
