
## 1. 聚合（Aggregation）
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111116143492.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
Elasticsearch 除搜索以外，提供的针对 ES 数据进行统计分析的功能

 - 实时性
 - Hadoop (T+1)

通过聚合，我们会得到一个数据的概念，是分析和总结全套的数据，而不是寻找单个文档

 - 尖沙咀和香港岛的客房数量
 - 不同的价格区间，可预定的经济型酒店和五星级酒店的数量

高性能，只需要一条语句，就可以从 ES 得到分析结果

 - 无需再客户端自己去实现分析逻辑
### 1.1 Kibana 可视化报表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201111161615383.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 1.2 聚合的分类
 - `Bucket Aggregation` - 一些列满足特定条件的文档的集合
 - `Metric Aggregation` - 一些数学运算，可以对文档字段进行统计分析
 - `Pipeline Aggregation` - 对其他的聚合结果进行二次聚合
 - `Matrix Aggregation` - 支持对多个字段的操作并提供一个结果矩阵
### 1.3 Bucket & Metric
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201111161817441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
#### 1.3.1 Bucket
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201111161906128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
#### 1.3.2 Metric
Metric 会基于数据集计算结果，除了支持在字段上进行计算，同样也支持在脚本（painless script）产生的结果之上进行计算
大多数 Metric 是数学计算，仅输出一个值
 - `min / max / sum / avg /cardinality`

部分 metric 支持输出多个数值

 - `stats / percentiles / percentile_ranks`


### 1.4 Demo
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111116232768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201111162426325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201111162458171.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
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
