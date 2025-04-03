
## 1. Search API
**URL Search**

 - 在 URL 中使用查询参数

**Request Body Search**

 - 使用 Elasticsearch 提供的，基于 JSON 格式的格式更加完备的 Query Dpmain Specific
   Language (DSL)
 

指定查询的索引
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6d1be4f33550343a2252c60c259cd562.png#pic_center)
### 1.1 URL 查询

 - 使用 “q” , 指定查询字符串
 - “query string syntax” , KV 键值对
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/74336d2c446bc593918db43d202c1071.png#pic_center)
### 1.2 Request Body
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0acb33c8084770d55049554354aa413c.png#pic_center)
### 1.3 搜索response
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d0470037d7983fb84d7492650e9ea4b8.png#pic_center)
### 1.4 搜索的相关性 Relevance
 - 搜索是用户和搜索引擎的对话
 - 用户关心的是搜索结果的相关性

1. 是否找到所有相关的内容
2. 有多少不相关的内容被返回了
3. 文档的打分是否合理
4. 结合业务需求，平衡结果排名

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c7c7ce88c67d116893da632d16d3740.png#pic_center)
### 1.5 WEB 搜索
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9757a34699e178c7c2af3ffa85c67107.png#pic_center)
### 1.6 电商搜索
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/672bdcf6c0eb44b0b3cdadf1ace474fd.png#pic_center)
### 1.7 衡量相关性
Information Retrieval

 - Precision （查准率） - 尽可能返回较少的无关文档
 - Recall （查全率） - 尽量返回较多的相关文档
 - Ranking - 是否能够按照相关度进行排序
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e4a9938a30b44b94be8fc6bcbdb00ee.png#pic_center)
参考资料：
极客时间：Elasticsearch核心技术与实战
相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
[docker-compose安装elasticsearch集群](https://blog.csdn.net/xixihahalelehehe/article/details/109389780)
[Elasticsearch 7.X之文档、索引、REST API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406518)
[Elasticsearch节点，集群，分片及副本详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406875)
[Elasticsearch倒排索引介绍](https://blog.csdn.net/xixihahalelehehe/article/details/109440345)
[elasticsearch Analyzer 进行分词详解](https://blog.csdn.net/xixihahalelehehe/article/details/109447777)
