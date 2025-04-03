

-----
## 1. 算分与排序

 - ES 默认会以文档的相关度算分进行排序
 - 可以通过制定一个或者多个字段进行排序
 - 使用相关性算分（score）排序，不能满足某些特定条件

无法针对相关度，对排序实现更多的控制

## 2. Function Score Query
Function Score Query
可以在查询结束后，对每一个匹配的文档进行一系列的重新算分，根据新生成的分数进行排序
提供了几种默认的计算分值的函数

 - `Weight`：为每一个文档设置一个简单而不被规范化的权重
 - `Field Value Factor`：使用该数值来修改_score，例如将 “热度” 和 “点赞数” 作为算分的参考因素
 - `Random Score`：为每一个用户使用一个不同的，随机算分结果


## 3. 按受欢迎度提升权重
望能够将点赞多的 blog，放在搜索列表相对靠前的位置。同事搜索的评分，还是要作为排序的主要依据
新的算分 = 老的算分 * 投票数

 - 投票数为 0

投票数很大时

```bash
DELETE blogs
PUT /blogs/_doc/1
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   0
}

PUT /blogs/_doc/2
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   100
}

PUT /blogs/_doc/3
{
  "title":   "About popularity",
  "content": "In this post we will talk about...",
  "votes":   1000000
}


POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes"
      }
    }
  }
}
```
## 4. 使用 Modifier 平滑曲线
新的算分 = 老的算分 * log（1 + 投票数）
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d6fc288c17e38fbe620d91d45bc89956.png)

```bash
POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p"
      }
    }
  }
}
```

## 5. 引入 Factor

新的算分 = 老的算分 * log（1 + factor * 投票数）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0354da5bb1e28d10ce4762373e1994a.png)

```bash
POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p" ,
        "factor": 0.1
      }
    }
  }
}
```
## 6. Boost Mode 和 Max Boost#
Boost Mode

 - Multiply：算分和函数值的乘积
 - Sum：算分和函数值的和
 - Min/Max：算分与函数去 最小 / 最大值
 - Replace：使用函数取代算分
 
  Max Boost 可以将算分控制在一个最大值

```bash
POST /blogs/_search
{
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query":    "popularity",
          "fields": [ "title", "content" ]
        }
      },
      "field_value_factor": {
        "field": "votes",
        "modifier": "log1p" ,
        "factor": 0.1
      },
      "boost_mode": "sum",
      "max_boost": 3
    }
  }
}


```
## 7. 一致性随机函数

 - 使用场景：网址的广告需要提高展示率
 - 具体需求：让每个用户看到不同的随机排名，但是也希望同一个用户访问时，结果的相对顺序，保持一致（Consistently Random）

```bash
POST /blogs/_search
{
  "query": {
    "function_score": {
      "random_score": {
        "seed": 911119
      }
    }
  }
}
```

