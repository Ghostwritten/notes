
## 1. term基于词项查询
term 的 重要性
Term 是表达语意的最小单位。搜索和利用统计语言模型进行自然语言处理都需要处理 Term
### 1.1 特点

 - Term Level Query：Term Query / Range Query / Exists Query / Prefix
   Query / Wildcard Query
 - 在 ES 中，Term查询，对输入不做分词。会将输入作为一个整体，在倒排索引中查找准确的词项，并且使用相关度算分公式为每个包含该词项的文档进行相关度算分 -例如 “Apple Store”
 - 可以通过 Constant Score 将查询转换换成一个 Filtering，避免算分，并利用缓存，提交性能

### 1.2 demo
插入数据
```bash
POST /products/_bulk
{ "index": { "_id": 1 }}
{ "productID" : "XHDK-A-1293-#fJ3","desc":"iPhone" }
{ "index": { "_id": 2 }}
{ "productID" : "KDKE-B-9947-#kL5","desc":"iPad" }
{ "index": { "_id": 3 }}
{ "productID" : "JODL-X-1937-#pV7","desc":"MBP" }
```
查询

```bash
POST /products/_search
{
"query": {
  "term": {
    "desc.keyword": {
      "value": "iPhone" //查不多数据，term查询
      //"value":"iphone" // 查到数据 term查询会做分词
    }
  }
}
}
POST /products/_search
{
"query": {
  "term": {
    "productID": {
      "value": "XHDK-A-1293-#fJ3"  // 无结果
      "value": "xhdk" //有一条数据
      "value": "xhdk-a-1293-#fj3"
    }
  }
}
}
//如果对值进行查询
POST /products/_search
{
//"explain": true,
"query": {
  "term": {
    "productID.keyword": {
      "value": "XHDK-A-1293-#fJ3"
    }
  }
}
}
```
### 1.3 复合查询 - Constant Score 转为 Filter

 - 将 Query 转成 Filter，忽略 TF-IDF 计算，避免相关性算分的开销
 - Filter 可以有效利用缓存

```bash
POST /products/_search
{
  "explain": true,
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "productID.keyword": "XHDK-A-1293-#fJ3"
        }
      }
    }
  }
}
```


## 2. 基于全文本的查找

 - `Match Query` / `Match Phrase Query` / `Query String Query`

### 2.1 特点

 - 索引和搜索时会进行分词，查询字符串先传递到一个合适的分词器，然后生成一个供查询的词项列表
 - 查询时候，先会对输入的查询进行分词。然后每个词项逐个进行底层的查询，最终将结果进行合并。并未每个文档生成一个算分。 例如查 “Martix reloaded”, 会查到包括 Matrix 或者 reload 的所有结果

### 2.2. Match Query Result

```bash
POST movies/_search
{
  "query":{
    "match": {
      "title":{
        "query": "Matrix reloaded"
      }
    }
  }
}
```
### 2.3 Operator


```bash
POST movies/_search
{
  "query":{
    "match": {
      "title":{
        "query": "Matrix reloaded",
        "operator":"AND"
      }
    }
  }
}
```
### 2.4 Minimun_should_match

```c
POST movies/_search
{
  "query":{
    "match": {
      "title":{
        "query": "Matrix reloaded",
        "minimum_should_match":2"
      }
    }
  }
}
```
### 2.5 Match Phrase Query

```bash
POST movies/_search
{
  "profile":true,   //启用Profile API
  "query":{
    "match_phrase": {
      "title":{
        "query": "Matrix reloaded",
        "slop":1"
      }
    }
  }
}
```
### 2.6 Match Query 查询过程
基于全文本的查找

 - Match Query / Match Phrase Query / Query String Query

基于全文本的查询的特点

 - 索引和搜索时都会进行分词，查询字符串先传递到一个合适的分词器，然后生成一个供查询的词项列表
 - 查询会对每个词项逐个进行底层的查询，再将结果进行合并，并未每个文档生成一个算分
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201113144210801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
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
[Elasticsearch index template与dynamic template详解](https://blog.csdn.net/xixihahalelehehe/article/details/109595303)
[Elasticsearch 聚合分析简介](https://blog.csdn.net/xixihahalelehehe/article/details/109625376)
[Elasticsearch 第一阶段总结与测试](https://blog.csdn.net/xixihahalelehehe/article/details/109626903)
