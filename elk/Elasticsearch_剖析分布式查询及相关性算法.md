

----
## 1. 分布式搜索的运行机制
ES 的搜索，会分两阶段进行

 - 第一阶段 - QUERY
 - 第二阶段 - Fetch

Query - then - Fetch
## 2. Query 阶段

 - 用户发出搜索请求到 ES 节点。节点收到请求后，会以 `Coordinating` 节点的身份，在 6 个主副分片中随机选择 3个分片，发送查询请求
 - 被选中的分片执行查询，进行排序。然后，每个分片都会返回 `From + Size` 个排序后的文档 Id 和排序值给 `Coordinating`节点
 - ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303165031830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


## 3. Fetch 阶段
Coordinating Node 会将 Query 阶段，从每个分片获取的排序后的文档 Id 列表，重新进行排序。选取 From 到 From + Size 个文档的 Id
以 `multi get` 请求的方式，到相应的分片获取详细的文档数据


## 4. Query Then Fetch 潜在的问题
性能问题

 - 每个分片上需要查的文档个数 = from + size
 - 最终协调节点需要处理：number_of_shard * (from + size)
 - 深度分页

相关性算分

 - 每个分片都基于自己的分片上的数据进行相关度计算。这会导致打分偏离的情况，特别是数据量很少时，
 - 如果文档总数很好的情况下，如果主分片大于 1，主分片越多，相关性算分会越不准。

## 5. 解决算分不准的方法
数据量不大的时候，可以将主分片数设置为 1

 - 当数据量足够大时候，只要保证文档均匀分散在各个分片上，结果一般就不会出现偏差 使用 DFS Query Then Fetch
 - 搜索的 URL 中指定参数 “`_search?search_type=dfs_query_then_fetch`”
 - 到每个分片把各分片的词频和文档频率进行搜集，然后完整的进行一次相关性算分，消耗更加多的 CPU 和内存，执行性能低下，一般不建议使用

## 6. 相关性算分问题 DEMO

 - 写入 3 条记录 “Good” / “Good moring” / “good morning everyone”
 - 使用 1 个主分片测试，Good 应该排在第一，Good DF 数值应该是 3
 - 和 20 个主分片测试
 - 当多个主分片时，3 个文档的算分都一样。可以通过 Explain API 进行分析
 - 在 3 个主分片上执行 DFS Query Then Fetch ，结果和一个分片上一致

Demo

```bash
DELETE message
PUT message
{
  "settings": {
    "number_of_shards": 20
  }
}
GET message
POST message/_doc?routing=1
{
  "content":"good"
}
POST message/_doc?routing=2
{
  "content":"good morning"
}
POST message/_doc?routing=3
{
  "content":"good morning everyone"
}
POST message/_search
{
  "explain": true,
  "query": {
    "match_all": {}
  }
}

POST message/_search
{
  "explain": true,
  "query": {
    "term": {
      "content": {
        "value": "good"
      }
    }
  }
}

POST message/_search?search_type=dfs_query_then_fetch
{
  "query": {
    "term": {
      "content": {
        "value": "good"
      }
    }
  }
}
```


