
## 1. 倒排索引
### 1.1 书的目录就是书的索引
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101225727960.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 1.2 倒排索引
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101225802502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 1.3 图书和索引引擎的类比
图书

 - 正排索引 - 目录页
 - 倒排索引 - 索引页

搜索引擎

 - 正排索引 - 文档 id 到文档内容和单词的关联
 - 倒排索引 - 单词到文档 id 的关系
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101225908763.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 1.4 倒排索引的核心组成
倒排索引包含两个部分
 - 单词词典（Term Dictionary） ，记录所有文档的单词，记录单词到倒排列表的关联关系 单词词典比较大，可以通过 B + 树 或者哈希拉链法实现，以满足高性能的插入与查询
 - 倒排列表（Postion List）- 记录了单词对应的文档结合，由倒排索引项组成

倒排索引

 - 文档 ID
 - 词频 TF - 单词在文档中的分词的位置。用于语句搜索（phrase query）
 - 偏移（Offset） - 记录单词的开始结束时间，实现高亮显示

### 1.5 Elasticsearch倒排索引实例
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201101232000662.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
### 1.6 Elasticsearch 倒排索引特点
Elasticsearch 的 JSON 文档中的每个字段，都有自己的倒排索引
可以指定对某些字段不做索引

 - 优点：节省储存空间
 - 缺点：字段无法被搜索
参考资料：
极客时间：Elasticsearch核心技术与实战

相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
[docker-compose安装elasticsearch集群](https://blog.csdn.net/xixihahalelehehe/article/details/109389780)
[Elasticsearch 7.X之文档、索引、REST API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406518)
[Elasticsearch节点，集群，分片及副本详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406875)
[elasticsearch倒排索引介绍](https://blog.csdn.net/xixihahalelehehe/article/details/109440345)
