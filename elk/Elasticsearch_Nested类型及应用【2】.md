

----
## 1. 问题背景
在elasticsearch中，我们可以将密切相关的实体存储在单个文档中。 例如，我们可以通过传递一系列评论来存储博客文章及其所有评论。
举例：

```bash
{
  "title": "Invest Money",
  "body": "Please start investing money as soon...",
  "tags": ["money", "invest"],
  "published_on": "18 Oct 2017",
  "comments": [
    {
      "name": "William",
      "age": 34,
      "rating": 8,
      "comment": "Nice article..",
      "commented_on": "30 Nov 2017"
    },
    {
      "name": "John",
      "age": 38,
      "rating": 9,
      "comment": "I started investing after reading this.",
      "commented_on": "25 Nov 2017"
    },
    {
      "name": "Smith",
      "age": 33,
      "rating": 7,
      "comment": "Very good post",
      "commented_on": "20 Nov 2017"
    }
  ]
}
```

如上所示，所以我们有一个文档描述了一个帖子和一个包含帖子上所有评论的内部对象评论。
但是Elasticsearch搜索中的内部对象并不像我们期望的那样工作。

## 2. 问题出现
现在假设我们想查找用户{name：john，age：34}评论过的所有博客帖子。 让我们再看一下上面的示例文档，找到评论过的用户。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df56e4f5758e41194365d369ccebddaf.png)

从列表中我们可以清楚地看到，没有34岁的用户John。为简单起见，我们在elasticsearch索引中只有1个文档。让我们通过查询索引来验证它：

```bash
GET /blog/_search?pretty
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "comments.name": "John"
          }
        },
        {
          "match": {
            "comments.age": 34
          }
        }
      ]
    }
  }
}
```
我们的示例文档作为回复返回。 很惊讶，这是为什么呢？
## 3. 原因分析
这就是为什么我说：elasticsearch中的内部对象无法按预期工作，这里的问题是elasticsearch（lucene）使用的库没有内部对象的概念，因此内部对象被扁平化为一个简单的字段名称和值列表。
我们的文档内部存储为：

```bash
{
  "title":                    [ invest, money ],
  "body":                     [ as, investing, money, please, soon, start ],
  "tags":                     [ invest, money ],
  "published_on":             [ 18 Oct 2017 ]
  "comments.name":            [ smith, john, william ],
  "comments.comment":         [ after, article, good, i, investing, nice, post, reading, started, this, very ],
  "comments.age":             [ 33, 34, 38 ],
  "comments.rating":          [ 7, 8, 9 ],
  "comments.commented_on":    [ 20 Nov 2017, 25 Nov 2017, 30 Nov 2017 ]
}
```
如上，您可以清楚地看到，`comments.name`和`comments.age`之间的关系已丢失。
这就是为什么我们的文档匹配john和34的查询。

## 4. 如何解决呢？
要解决这个问题，我们只需要对elasticsearch的映射进行一些小改动。如果您查看索引的映射，您会发现comments字段的类型是object。我们需要更新它的类型为nested。我们可以通过运行以下查询来简单地更新索引的映射：

```bash
PUT /blog_new
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "body": {
        "type": "text"
      },
      "tags": {
        "type": "keyword"
      },
      "published_on": {
        "type": "keyword"
      },
      "comments": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text"
          },
          "comment": {
            "type": "text"
          },
          "age": {
            "type": "short"
          },
          "rating": {
            "type": "short"
          },
          "commented_on": {
            "type": "text"
          }
        }
      }
    }
  }
}

```
将映射更改为Nested类型后，我们可以查询索引的方式略有变化。 我们需要使用Nested查询。
下面给出了Nested查询示例：

```bash
GET /blog_new/_search?pretty
{
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "path": "comments",
            "query": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "comments.name": "john"
                    }
                  },
                  {
                    "match": {
                      "comments.age": 34
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  }
}
```
由于用户`{name：john，age：34}`没有匹配，上面的查询将不返回任何文档。
再次感到惊讶？ 只需一个小小的改变即可解决问题。
这可能是我们理解的一个较小的变化，但是在elasticsearch存储我们的文档的方式上有很多变化。
在内部，嵌套对象将数组中的每个对象索引为单独的隐藏文档，这意味着可以独立于其他对象查询每个嵌套对象。
下面给出了更改映射后样本文档的内部表示：

```bash
{
  {
    "comments.name":    [ john ],
    "comments.comment": [ after i investing started reading this ],
    "comments.age":     [ 38 ],
    "comments.rating":  [ 9 ],
    "comments.date":    [ 25 Nov 2017 ]
  },
  {
    "comments.name":    [ william ],
    "comments.comment": [ article, nice ],
    "comments.age":     [ 34 ],
    "comments.rating":   [ 8 ],
    "comments.date":    [ 30 Nov 2017 ]
  },
  {
    "comments.name":    [ smith ],
    "comments.comment": [ good, post, very],
    "comments.age":     [ 33 ],
    "comments.rating":   [ 7 ],
    "comments.date":    [ 20 Nov 2017 ]
  },
  {
    "title":            [ invest, money ],
    "body":             [ as, investing, money, please, soon, start ],
    "tags":             [ invest, money ],
    "published_on":     [ 18 Oct 2017 ]
  }
}
```
如您所见，每个内部对象都在内部存储为单独的隐藏文档。 这保持了他们的领域之间的关系。

## 5. Nested类型的作用？
从上一小节，可以清晰的看出nested类型的特别之处。
nested类型是对象数据类型的专用版本，它允许对象数组以可以彼此独立查询的方式进行索引。


## 6. Nested类型的适用场景
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eff6ec4f4ee8f53368a0d9526eaef129.png)
## 7. Nested类型的增、删、改、查、聚合操作详解
还是以第2节的blog_new索引示例，Nested类型的增、删、改、查操作。
###  7.1 Nested类型——增

新增blog和评论

```bash
POST blog_new/_doc/2
{
  "title": "Hero",
  "body": "Hero test body...",
  "tags": ["Heros", "happy"],
  "published_on": "6 Oct 2018",
  "comments": [
    {
      "name": "steve",
      "age": 24,
      "rating": 18,
      "comment": "Nice article..",
      "commented_on": "3 Nov 2018"
    }
  ]
}
```
### 7.2 Nested类型——删
序号为1的评论原来有三条，现在删除John的评论数据，删除后评论数为2条。

```bash
POST blog_new/_update/2
{
  "script": {
    "lang": "painless",
    "source": "ctx._source.comments.removeIf(it -> it.name == 'John');"
  }
}
```
### 7.3 Nested类型——改
将steve评论内容中的age值调整为25，同时调整了评论内容。

```bash
POST blog_new/_update/2
{
  "script": {
    "source": "for(e in ctx._source.comments){if (e.name == 'steve') {e.age = 25; e.comment= 'very very good article...';}}"
  }
}
```
### 7.4 Nested类型——查
如前所述，查询评论字段中评论姓名=William并且评论age=34的blog信息。

```bash
GET /blog_new/_search?pretty
{
  "query": {
    "bool": {
      "must": [
        {
          "nested": {
            "path": "comments",
            "query": {
              "bool": {
                "must": [
                  {
                    "match": {
                      "comments.name": "William"
                    }
                  },
                  {
                    "match": {
                      "comments.age": 34
                    }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  }
}
```
### 7.5 Nested类型——聚合
知前提：nested聚合隶属于聚合分类中的Bucket聚合分类。
聚合blog_new 中评论者年龄最小的值。

```bash
GET blog_new/_search
{
  "size": 0,
  "aggs": {
    "comm_aggs": {
      "nested": {
        "path": "comments"
      },
      "aggs": {
        "min_age": {
          "min": {
            "field": "comments.age"
          }
        }
      }
    }
  }
}
```
小结
如果您在索引中使用内部对象并做查询操作，请验证内部对象的类型是否为nested类型。 否则查询可能会返回无效的结果文档。
更新认知是非常痛苦的，不确定的问题只有亲手实践才能检验真知
