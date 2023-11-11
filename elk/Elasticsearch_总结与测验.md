

----
## 1. 搜索与算分
结构化搜索与⾮结构化搜索
○ Term 查询和基于全⽂本 Match 搜索的区别
○ 对于需要做精确匹配的字段，需要做聚合分析的字段，字段类型设置为 Keyword

Query Context v.s Filter Context
○ Filter Context 可以避免算分，并且利⽤缓存
○ Bool 查询中 Filter 和 Must Not 都属于 Filter Context


## 2. 搜索与算分
● 搜索的算分
○ TF-IDF / 字段 Boosting
● 单字符串多字段查询：multi-match
○ Best_Field / Most_Fields / Cross_Field

● 提⾼搜索的相关性
○ 多语⾔：设置⼦字段和不同的分词器提升搜索的效果
○ Search Template 分离代码逻辑和搜索 DSL
○ 多测试，监控及分析⽤户的搜索语句和搜索效果


## 3. 聚合 / 分⻚
● 聚合
○ Bucket / Metric / Pipeline

● 分⻚
○ From & Size / Search After / Scroll API
○ 要避免深度分⻚，对于数据导出等操作，可以使⽤ Scroll API


## 4. Elasticsearch 的分布式模型
● ⽂档的分布式存储
○ ⽂档通过 hash 算法， route 并存储到相应的分⽚

● 分⽚及其内部的⼯作机制
○ Segment / Transaction Log / Refresh / Merge

● 分布式查询和聚合分析的内部机制
○ Query Then Fetch；IDF 不是基于全局，⽽是基于分⽚计算，，因此，数据量少的时候，算分不准
○ 增加 “shard_size” 可以提⾼ Terms 聚合的精准度


## 5. 数据建模及重要性
● 数据建模
○ ES 如何处理管理关系 / 数据建模的常⻅步骤 / 建模的最佳实践

● 建模相关的⼯具
○ Index Template / Dynamic Template / Ingest Node / Update By Query / Reindex / Index Alias

● 最佳实践
○ 避免过多的字段 / 避免 wildcard 查询 / 在 Mapping 中设置合适的字段


## 阅读官方文档
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310163323115.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310163410378.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031016342118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


## 6. 测试
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310163619474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
DELETE test
PUT test/_doc/1
{
  "content":"Hello World"
}

POST test/_search
{
  "profile": "true",
  "query": {
    "match": {
      "content": "Hello World"
    }
  }
}

POST test/_search
{
  "profile": "true",
  "query": {
    "match": {
      "content": "hello world"
    }
  }
}

POST test/_search
{
  "profile": "true",
  "query": {
    "match": {
      "content.keyword": "Hello World"
    }
  }
}

POST test/_search
{
  "profile": "true",
  "query": {
    "match": {
      "content.keyword": "hello world"
    }
  }
}

POST test/_search
{
  "profile": "true",
  "query": {
    "term": {
      "content": "Hello World"
    }
  }
}

POST test/_search
{
  "profile": "true",
  "query": {
    "term": {
      "content": "hello world"
    }
  }
}

POST test/_search
{
  "profile": "true",
  "query": {
    "term": {
      "content.keyword": "Hello World"
    }
  }
}
```

1. 判断题：⽣产环境中，对索引使⽤ Index Alias 是⼀个好的实践 ，
答：`对`，不用改变已有数据和索引
2. 在 Terms 聚合分析中，有哪些⽅法可以提⾼查询的精准度
两种方法：
1.数据量不大的时候，分片数量设置为1，就不会存在聚合分析精准度的问题。
2.当数据量多大，不得不分片的时候，term


3. 如何通过聚合分析知道，每天⽹站中的访客来⾃多少不同的 IP
4. 请描述 “multi_match” 查询中 “best_field”的⾏为
答： 最高分搜索
5. 对搜索结果分⻚时，所采⽤的两个参数
from和size
6. 判断题：使⽤ Scroll API 导出数据时，即使中途有新的数据写⼊，这些数据也能被导出 
答： `错`，新写入的数据不能导出，因为是通过快照的方式
