考试真题
链接：[https://pan.baidu.com/s/1BYiqKqFA6GMGjzeM8qAP8Q](https://pan.baidu.com/s/1BYiqKqFA6GMGjzeM8qAP8Q)
提取码：mcfj
复制这段内容后打开百度网盘手机App，操作更方便哦
答案地址：
[https://github.com/mingyitianxia/elastic-certified-engineer/blob/master/review-practice/0011_zhenti.md](https://github.com/mingyitianxia/elastic-certified-engineer/blob/master/review-practice/0011_zhenti.md)


## 0、文档入口地址
0.1 核心文档 95%+
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index.html)
0.2 script文档 5%
[https://www.elastic.co/guide/en/elasticsearch/painless/7.2/index.html](https://www.elastic.co/guide/en/elasticsearch/painless/7.2/index.html)
注意：script部分是很容易忽视的！

## 1、Installation and Configuration
elasticsearch.yml 配置样例
**考试提示1：**
考试的时候，就是下面的样例，机器配置一堆配置，没有像咱们官方下载的那种有分隔线，所以，要适应这种配置样式。
**考试提示2：**
- 10个试题，不同试题对应不同的机器。
- 拿到试题后，你首先看到的第一页就是讲解集群配置的，所以，要先大致过一遍。
- 大致会有4个左右集群，每个集群节点(node)个数不一致，有的单节点，有的多个节点。
- 一般kibana都会启动了（5601，5602, 5603等多个端口），ES如果没有启动，需要自己手动启动。
- 切记不要打错集群了

```bash
cluster.name: cluster1 
node.name: node1
node.master: true 
node.data: true 
node.ingest: true
node.ml: false 
cluster.remote.connect: false 
network.host: 172.17.0.17
http.port: 9200
discovery.seed_hosts: ["172.17.0.17:9300","172.17.0.17:9301"]
cluster.initial_master_nodes: ["172.17.0.17:9300","172.17.0.17:9301"]
xpack.security.enabled: true
path.repo: ["/home/elasticsearch/elasticsearch-7.2.0/backup"]
node.attr.hot_warm_type: warm 
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```
### 1.1 Deploy and start an Elasticsearch cluster that satisfies a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/setup.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/setup.html)
**考点梳理：**
1、elasticsearch.yml 核心配置的含义

```bash
cluster.name: cluster1  ——含义：集群名称，多个node的，集群名称必须一致。

node.name: node1 —— 含义：节点名称，每个节点名称是唯一的。

network.host: 172.17.0.17 ——含义：这台机器的内网ip地址

http.port: 9200 ——含义：http端口（对外提供服务的端口）

transport.port：9300——含义：集群之间通信的端口，若不指定默认：9300

discovery.seed_hosts: ["172.17.0.17:9300","172.17.0.17:9301"]
——节点发现需要配置一些种子节点，与7.X之前老版本：disvoery.zen.ping.unicast.hosts类似，一般配置集群中的全部节点

cluster.initial_master_nodes: ["172.17.0.17:9300","172.17.0.17:9301"]
——含义：指定集群初次选举中用到的具有主节点资格的节点，称为集群引导，只在第一次形成集群时需要。
注意：
1）不具备主节点资格的节点，以及新节点加入现有集群，无需配置inital_master_nodes
2)各个节点配置的inital_master_nodes值应该相同。
3）小心配置，可能会引发脑裂。
```
### 1.2 Configure the nodes of a cluster to satisfy a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/important-settings.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/important-settings.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-node.html)
**考点梳理：**
区分节点角色，能配置：主节点、数据节点、路由节点（协调节点）、Ingest节点等。

```bash
主节点：
node.master: true 
node.data: false 
node.ingest: false
node.ml: false 
数据节点：
node.master: false 
node.data: true 
node.ingest: false
node.ml: false 
路由节点：
node.master: false 
node.data: false 
node.ingest: false
node.ml: false 
Ingest节点：
node.master: false 
node.data: false 
node.ingest: true
node.ml: false
```
### 1.3 Secure a cluster using Elasticsearch Security
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-settings.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-settings.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/configuring-security.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/configuring-security.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/setup-passwords.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/setup-passwords.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/encrypting-communications.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/encrypting-communications.html)
**考点梳理：**
• x-pack 安全基础配置

```bash
xpack.security.enabled: true
```

——默认没有这条配置，x-pack 相关的都需要手动配置启动。
• 设置用户名、密码（注意：可以kibana操作），新设置完，一定验证是否成功，切记！！
• tls 加密通信（没有明确说不考，建议也过一下）

### 1.4 Define role-based access control using Elasticsearch Security
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-role.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-role.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-user.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/security-api-put-user.html)
**考点梳理：**
- x-pack 一般会结合role 角色一起考
- 新建角色
- 新建用户&密码，修改密码
- 官方我咨询过：命令行或者kibana操作都可以，但要确保结果对。建议kibana，毕竟比较简洁。
- kibana权限设置，一定要加上能访问kibana，否则新建了用户会无法登录（可能会扣分）
举例：设置x-pack属性后（默认未开启），设置用户名、密码（可以kibana设置）、设置访问权限等。

## 2、Indexing Data
### 2.1 Define an index that satisfies a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-create-index.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-create-index.html)
**考点梳理：**
- 创建满足给定条件的索引
- 主分片数、副本分片数
- 修改主分片、副本分片数
- setting设置（参数建议都过一下，如：刷新频率等）
### 2.2 Perform index, create, read, update, and delete operations on the documents of an index
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs.html)
考点梳理：
- 文档单个新增及批量新增（bulk）
- 文档修改（单个修改：update，批量修改：update_by_query）
- 文档读取(get)
- 文档删除(单个删除：delete，批量删除：delete_by_query)
- 注意组合考点（必考难点）：更新结合ingest，更新结合script-painless 删除结合：script-painless
### 2.3 Define and use index aliases
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-aliases.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-aliases.html)
考点梳理：
- 新建索引指定别名
- 新建模板指定别名
- 为已有索引添加别名
### 2.4 Define and use an index template for a given pattern that satisfies a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/indices-templates.html)
考点梳理：
- 创建满足给定条件的索引模板
- 组合考点创建模板同时：指定mapping，指定setting，指定ingest，指定analyzer，指定别名，指定order优先级
### 2.5 Define and use a dynamic template that satisfies a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/dynamic-templates.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/dynamic-templates.html)
考点梳理：
- 创建满足给定模板条件的索引，如：text_*开头指定为text类型
- 创建满足给定模板条件的模板，可以结合2.4 一起考！
### 2.6 Use the Reindex API and Update By Query API to reindex and/or update documents
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-reindex.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-reindex.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docs-update-by-query.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting.html)
**考点梳理：**
- 单纯redinex操作
- reindex结合query 条件
- 组合考点：reindex结合script-painless操作
- 组合考点：（难点）redinex结合 ingest操作
### 2.7 Define and use an ingest pipeline that satisfies a given set of requirements, including the use of Painless to modify documents
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/ingest.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-scripting.html)
[https://www.elastic.co/guide/en/elasticsearch/painless/7.2/painless-lang-spec.html](https://www.elastic.co/guide/en/elasticsearch/painless/7.2/painless-lang-spec.html)
考点梳理：
- 单纯 ingest操作（修改字段，新增字段等）
- （难点）redinex结合 ingest操作
- （难点）update 组合 ingest操作

## 3、Queries
### 3.1 Write and execute a search query for terms and/or phrases in one or more fields of an index
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl.html)
考点梳理：
- 区分 全文检索（打分）和非评分检索（无需打分）， filter 和 query
- 各种query 都要熟悉
- bool组合query
- 自定义评分的四种方式检索
### 3.2 Write and execute a search query that is a Boolean combination of multiple queries and filters
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-bool-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-bool-query.html)
考点梳理：
- （重点）bool组合query
- filter, should, must, must_not 组合及嵌套使用
- minimum_should_match 的正确使用
### 3.3 Highlight the search terms in the response of a query
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-highlighting.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-highlighting.html)
考点梳理：
- 为指定字段设置给定条件的高亮 highlight
### 3.4 Sort the results of a query by a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-sort.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-sort.html)
考点梳理：
- 为指定字段设置给定条件的排序 sort
- 一个字段排序，多个字段不同条件排序（升序、降序）
### 3.5 Implement pagination of the results of a search query
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-from-size.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-from-size.html)
考点梳理：
- 为指定字段设置给定条件的分页（from， size   from——起始值默认为0）
- 会和query，sort等一起考，作为其中一个条件
- 注意：聚合的时候，不需要返回查询的时候，size设置为0。
### 3.6 Use the scroll API to retrieve large numbers of results
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-scroll.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-scroll.html)
考点梳理：
- 会使用scroll遍历数据
- 会使用scroll-after 遍历数据
### 3.7 Apply fuzzy matching to a query
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-fuzzy-query.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-fuzzy-query.html)
考点梳理：
- 会使用fuzzy模糊匹配
- 注意不同参数都要过一遍
### 3.8 Define and use a search template
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-template.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-template.html)
考点梳理：
- 设置满足给定条件的search_template
- 有难度，wood大叔（携程首席架构师，Elastic中文社区排名第一名）这道题都做错了（原因：平时不大用）
### 3.9 Write and execute a query that searches across multiple clusters
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html)
考点梳理：
- 跨集群设置（setting可以动态配置，每个集群都要设置的）
- 跨集群组合检索

## 4、Aggregations
### 4.1 Write and execute metric and bucket aggregations
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-metrics.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-metrics.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-bucket.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-bucket.html)
考点梳理：
- 要区分不同的聚合
- 根据考试要求选定指定的聚合
- metric：理解为min等指标聚合
- bucket：理解为具体的分桶聚合（举例：mysql的group by聚合）
-
 每篇细节的参数都要过一遍
### 4.2 Write and execute aggregations that contain sub-aggregations
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-pipeline.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-aggregations-pipeline.html)
考点梳理：
- 有难度，实现基于聚合条件的子聚合
- 区分：silbing子聚合和child自聚合写法不一样
- 复杂聚合可能容易逻辑混乱，注意：格式化快捷键“ctrl + i”（关键时候提高排错效率）

## 5、 Mappings and Text Analysis
### 5.1 Define a mapping that satisfies a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping.html)
考点梳理：
- 设置给定条件的mapping
- 不同字段类型选型（简单字段：如keyword，integer；复杂字段如：join，nested等）
- 不同字段的细节配置（如：正排索引docvalue取消等）
### 5.2 Define and use a custom analyzer that satisfies a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-custom-analyzer.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/analysis-custom-analyzer.html)
考点梳理：
- 自定义分词器
- 能区分：analyzer的含义（如：keyword，whitespace，standard等）
- 难点：能实现给定条件自定义分词（很重要，有难度）
## 5.3 Define and use multi-fields with different data types and/or analyzers
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/multi-fields.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/multi-fields.html)
考点梳理：
- 一个字段设置不同的分词器（必考）
- 考试100%不会考中文分词器，一般会是：standard，english或者其他组合
- 语法要熟悉，一直往后罗列即可
## 5.4 Configure an index so that it properly maintains the relationships of nested arrays of objects
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/nested.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/nested.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/parent-join.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/parent-join.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-inner-hits.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/search-request-inner-hits.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/joining-queries.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/joining-queries.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-terms-query.html#query-dsl-terms-lookup](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/query-dsl-terms-query.html#query-dsl-terms-lookup)
考点梳理：
- 组合类型：nested类型
- 组合类型：join类型
- nested类型的查询和聚合
- join类型的查询（子查父，父查子）和聚合
- 注意：inner_hits的潜在考点
- 隐藏考点：terms-lookup（没有明确不考，注意下）

## 6、Cluster Administration
### 6.1 Allocate the shards of an index to specific nodes based on a given set of requirements
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index-modules-allocation.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index-modules-allocation.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cluster.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cluster.html)
考点梳理：
- 分片分配策略
- 指定分片分配到指定的节点上

 注意：elasticsearch.yml的配置
### 6.2 Configure shard allocation awareness and forced awareness for an index
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-awareness.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/allocation-awareness.html)
考点梳理：
- 跨机架集群分片分配感知分配策略
- 一般会有多个节点（考试的时候4个节点）

### 6.3 Diagnose shard issues and repair a cluster’s health
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-health.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-health.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-allocation-explain.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-allocation-explain.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-reroute.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/cluster-reroute.html)
考点梳理：
- cat api使用（很多，都要熟悉）
- 诊断集群健康状态，找到黄色或红色非健康能找到原因，并变成健康绿色状态
- 诊断集群分配未分配的原因，并恢复正常
- 集群分配迁移等重新路由实现
### 6.4 Backup and restore a cluster and/or specific indices
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/backup-cluster.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/backup-cluster.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-snapshots.html)
考点梳理：
- 快照备份集群并恢复
- 快照备份指定索引并恢复
- 一定要验证一下恢复是否正确，是否满足给定题目的条件
### 6.5 Configure a cluster for use with a hot/warm architecture
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shard-allocation-filtering.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/shard-allocation-filtering.html)
考点梳理：
- 冷热集群部署
- 冷热集群架构设置
- 热数据迁移到冷节点上
### 6.6 Configure a cluster for cross cluster search
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-remote-clusters.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-remote-clusters.html)
[https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html](https://www.elastic.co/guide/en/elasticsearch/reference/7.2/modules-cross-cluster-search.html)
考点梳理：
- 能实现跨集群检索配置
- 能实现跨集群检索
- 考试的时候，一定要验证返回结果是不同集群返回的才可以
