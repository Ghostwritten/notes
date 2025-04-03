
## 1. 为什么学习elasticsearch？
主要功能

 - 分布式搜索引擎
 - 大数据实时分析引擎
主要特性：
 - 高性能
 - 容易使用，容易扩展
## 2. 达到目的，三个层面
 - 开发 产品基本功能，底层工作原理，数据建模最佳实践
 - 运维 容易规划，性能优化，问题诊断，滚动升级
 - 方案 搜索与如何解决搜索得相关性问题 大数据分析实践与项目实战，理论知识运用到实际场景

## 3. 如何学好？

 - 勤动手 本地搭建多节点得集群环境，理解分布式工作原理，执行教程中得每个范例
 - 多思考 结合实际得业务场景，深入思考，学会查阅技术文档
 - 定目标 做一次分享，开发具体项目，参加考试

## 4. 从开源到上市
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d95a907c7fcc15fc6a8c45c7602ddb6.png#pic_center)

## 5. elastic客户
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de84e833cb0172670ecbc3c3d9a9bdd5.png#pic_center)
## 6. elastic简介
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7fce73bf95ea46be1899bb26915e0d13.png#pic_center)
## 7. Lucene起源
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d01c41f595665bf64a1cff4e06c092d4.png#pic_center)
这个看着很帅气的男人他叫Doug Cutting，是Lucene的创始人，同时他也是Hadoop的创始人，被称为Hadoop之父。重点还是来看下Lucene有哪些特点：

（1）基于Java语言开大的搜索引擎类库
（2）创建于1999年，2005年成为Apache顶级开源项目
（3）Lucene具有高性能、以扩展的有点
（4）Lucene的局限性：

只能基于Java语言开发
类库的接口学习成本高
原生并不支持水平扩展（这对于搜索引擎来说是一个非常大的问题）
## 8. elastic诞生
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/664ec0099fc7cdc150f283dfe18e67d0.png#pic_center)
## 9. ElasticSearch分布式架构
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27b36484dcee86d584e922cc42b42247.png#pic_center)
集群规模可以从单个节点扩展至数百个节点
高可用 & 水平扩展：从服务和数据两个维度
支持不同的节点类型：支持Hot & Warm架构
## 10. 支持多种方式集成接入
多种编程语言的类库（https://www.elastic.co/guide/en/elasticsearch/client/index.html）
Java / .NET / Python / Ruby / PHP / Groovy /Perl
RESTful API v.s Transport API: 9200 v.s 9300端口（建议使用RESTful API）
最新版支持JDBC & ODBC接入方式



## 11. ElasticSearch主要功能

 - 海量数据的分布式存储及集群管理能力：服务于数据的高可用，水平扩展
 - 近实时搜索，性能卓越：结构化 / 全文 / 地理位置 / 自动完成
 - 海量数据的近实时分析: 指聚合功能

## 12. Elastic的版本升级
0.4：2010年2月第一次发布
1.0：2014年1月
2.0：2015年10月
5.0：2016年10月
6.0：2017年10月
7.0：2019年4月

### 12.1 新特性5.X
Lucene 6.x（表示此时以来的Lucene版本）性能升级，默认打分机制从TF-IDF（计算分词相似的一个算法）改为BM 25
支持Ingest节点/Painiess Scripting / Completion suggested支持 / 原生的Java REST客户端
Type标记成deprecated(过时，以前我们在创建索引的时候是需要创建一个Type标记的，现在可以不创建它)，支持了keyword类型
性能优化：

 1. 内部引擎移除了避免同一文档并发更新的竞争锁，带来15% - 20%的性能提升
 2. 支持分片上聚合的缓存
 3. 新增了Prefile API

### 12.2 新特性6.X
Lucene 7.x
新功能

 1. 跨级群复制（CCR）
 2. 索引声明周期管理
 3. SQL的支持

更友好的升级及数据迁移

 1. 在主要版本之间的迁移更为简化，体检升级
 2. 全新的基于操作的数据复制框架，可加快恢复数据、

性能优化

 1. 有效存储稀疏字段的新方法，降低了存储成本
 2. 在索引时进行排序，可加快排序的查询性能

### 12.3  新特性 7.X
Lucene 8.0
重大改进 - 正式废除单个索引下多Type的支持
7.1开始，Security功能免费使用
**ECK - Elasticsearch Operator on Kubernetes**
新功能

 1. New Cluster coordination
 2. Feature-Complete High Level REST Client
 3. Script Score Query、

性能优化

 1. 默认的Permary Shard数从5改为1，避免Over Sharding
 2. 性能优化，更快的Top K

## 13. elastic stack 生态圈
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3737fde5bdb3483ce3d6374264315edd.png#pic_center)
## 14. logstash 数据处理管道
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6b9de7df1e90cf0d0a703aae1595ec6b.png#pic_center)
[功能特性介绍](https://www.elastic.co/cn/logstash)
[实用介绍](https://www.elastic.co/cn/blog/a-practical-introduction-to-logstash)

## 15. kibana可视化分析利器
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aa2b03ba0331b2b8789e066c0829e4a5.png#pic_center)

[功能特性介绍](https://www.elastic.co/cn/kibana)
## 16. elastic发展
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d040f123a5eda06e82999f43d47689f3.png#pic_center)
## 17. BEATS 轻量数据采集器
![,color_FFFFFF,t_70#pic_center)](https://i-blog.csdnimg.cn/blog_migrate/26aa8f21dad0ceea86fb278df8c7c26c.png#pic_center)
## 18. X-pack 商业化套件
## 19. 应用场景
网站搜索
垂直搜素
代码搜索
日志管理与分析
安全指标监控
应用性能监控
web抓取舆情
## 20. 日志管理
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e2b260bf9962c2cc4ab20458525d244.png#pic_center)
## 21. elastic与数据库集成
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/38d72b9dfbee6f2c3e71d198915eaa56.png#pic_center)



## 22. 指标分析与日志分析
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0cb950ceba1ff62f297a4015c8349e02.png#pic_center)

