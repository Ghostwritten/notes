
## 1. 相关性和相关性算分
相关性 - Relevance

 - 搜索的相关性算分，描述了一个文档和查询语句匹配的程度。ES 会对每个匹配查询条件的结构进行算分_score
 - 打分的本质是排序 , 需要把最符合用户需求的文档排在前面。ES 5 之前，默认的相关性打分采用 TF-IDF，现在采用 BM25
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4cb5296a3d9930a6a06bd75fd9a87c9b.png#pic_center)
## 2. 词频 TF
 - Term Frequency ：检查词在一篇文档里出现的频率
检查词出现的次数除以文档的总字数
 - 度量一条查询和结果文档县管辖的建档方法：简单讲搜索每一个词的 TF 进行相加
TF（区块链） + TF（的）+ TF（应用）
 - Stop Word
“的” 在文档中出现了很多次，但是对贡献相关度几乎没有用处，不应该考虑他们的 TF

## 3. 逆文档频率 IDF
`DF`：检索词在所有文档中出现的频率

 - “区块链” 在相对比较少的文档中出现
 - “应用” 在相对比较多的文档中出现
 - “Stop Word” 在大量的文档中出现

`Inverse Document Frequency` ：简单说 = log（全部文档书 / 检索词出现过的文档总数）
TF-IDF 本质上就是将 TF 求和变成了加权求和

 - TF（区块链）* IDF（区块链） + TF（的）* IDF（的）+ TF（应用）* IDF（应用）
## 4. TF-IDF 的概念
**TF-IDF 被公认为是信息检索领域最重要的发明**
除了在信息检索，再文献分类和其他相关领域有着非常广泛的应用
IDF 的概念，最早是剑桥大学的 “斯巴达。琼斯” 提出
 - 1972 年 ——“关键词特殊性的统计解释和它在文献检索中的应用”
 - 但是没有从理论上件事 IDF 应该是用 log（全部文档书 / 检索词出现过的文档总数），而不是其他函数。也没有做进一步的研究

1970,1980 年代萨尔顿和罗宾逊，进行了进一步的证明和研究，并用香农信息做了证明
现代搜索引擎，对 TF-IDF 进行了大量细微的优化

## 5. Lucene 中的 TF-IDF 评分公式
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cfe9aa4a373ec9ae345dd671774b4235.png#pic_center)
## 6. BM25
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eb49c346753ee80a285f33237e0f6513.png#pic_center)
## 7. 定制 Similarity
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48d3bd686643320c8c38f3e43335a23a.png#pic_center)
## 8. 通过 Explain API 查看 TF-IDF

```bash
PUT testscore/_bulk
{"index":{"_id":1}}
{"content":"we use Elasticsearch to power the search"}
{"index":{"_id":2}}
{"content":"we like elasticsearch"}
{"index":{"_id":3}}
{"content":"The scoring of documents is caculated by the scoring formula"}
{"index":{"_id":4}}
{"content":"you know, for search"}
//查询
POST /testscore/_search
{
  "explain": true,
  "query": {
    "match": {
     // "content":"you"
      "content": "elasticsearch"
      //"content":"the"
      //"content": "the elasticsearch"
    }
  }
}
```
## 9. Boosting Relevance
Boosting 是控制相关度的一种手段

 - 索引，字段或查询子条件

参数 boost 的含义

 - 当 boost > 1 时，打分的相关度相对性提高
 - 当 0 < boost < 1 时，打分的权重相对性降低
 - 当 boost < 0 时，贡献度负分

```bash
POST testscore/_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "content" : "elasticsearch"
                }
            },
            "negative" : {
                 "term" : {
                     "content" : "like"
                }
            },
            "negative_boost" : 0.2
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
[Eelasticsearch 多字段特性及 Mapping 中配置自定义 Analyzer详解](https://blog.csdn.net/xixihahalelehehe/article/details/109476672)
[Elasticsearch index template与dynamic template详解](https://blog.csdn.net/xixihahalelehehe/article/details/109595303)
[Elasticsearch 聚合分析简介](https://blog.csdn.net/xixihahalelehehe/article/details/109625376)
[Elasticsearch 第一阶段总结与测试](https://blog.csdn.net/xixihahalelehehe/article/details/109626903)
[Elasticsearch 基于词项和基于全文的搜索详解](https://blog.csdn.net/xixihahalelehehe/article/details/109669976)
[Elasticsearch 结构化搜索详解](https://blog.csdn.net/xixihahalelehehe/article/details/109674952)
