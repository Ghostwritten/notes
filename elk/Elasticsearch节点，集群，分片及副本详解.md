
## 1. elasticsearch 分布式
### 1.1 分布式系统的可用性与扩展性

高可用性

 - 服务可用性-允许有节点停止服务
 - 数据可用性-部分节点丢失，不会丢失数据

可扩展性

 - 请求量提升/数据的不断增长（将数据分布到所有节点上）

### 1.2 分布式特性

elasticsearch的分布式架构好处

 - 存储的水平扩容
 - 提高系统的可用性，部分节停止服务，整个集群的服务不受影响

ElasticSearch的分布式架构

 - 不同的集群通过不同的名字来区分，默认名字”elasticsearch“
 - 通过配置文件修改，或者在命令行中 -E cluster.name=cluster1进行设定
 - 一个集群可以用多个节点

## 2. 节点

 - 是一个elasticsearch的实例，本质上是一个java进程，一台机器上可以运行多个elasticsearch进程，但是生产环境一般建议一台机器上运行一个elasticsearch实例
 - 每一个节点都有名字，通过配置文件配置，或者启动时候 -E node.name=node1 指定
 - 每一个节点在启动之后，会分配一个UID，保存在data目录下

**不同的节点承担了不同的角色**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9b1cf5174ba800bbb2949086c10e70ef.png#pic_center)

### 2.1 Master-eligible nodes和Master Node

 - 每个节点启动后，默认就是一个Master eligible节点，可以设置node.master:false禁止
 - Master-eligible节点可以参加选主流程，成为master节点
 - 当第一个节点启动时候，它会将自己选举成Master节点
 - 每个节点上保存了集群的状态，只有master节点才能修改集群的状态信息
 集群状态（Cluster State)，维护了一个集群中，必要的信息：
    所有的节点信息；
    所有的索引和其相关的Mapping与Setting信息；
    分片的路由信息；
 - 任意节点都能修改信息会导致数据的不一致性

### 2.2 Data Node & Coordinate Node

#### 2.2.1 Data Node

可以保存数据的节点，叫做Data Node，负责保存分片数据。在数据扩展上起到了至关重要的作用
#### 2.2.2 Coordinate Node

负责接受Client的请求，将请求分发到合适的节点，最终将结果汇集到一起
每个节点默认起到了Coordinate Node的职责



### 2.3 其他的节点类型
#### 2.3.1 Hot & Warm Node
不同硬件配置的Data Node，用来实现Hot & Warm架构，降低集群部署的成本
#### 2.3.2 Machine Learning Node
负责跑机器学习的 Job,用来做异常检测
#### 2.3.3 Tribe Node
5.3开始使用Cross Cluster Search ，Tribe Node连接到不同的Elasticsearch集群，并且支持将这些集群当成一个单独的集群处理

### 2.4 配置节点类型

| 节点类型              | 配置参数                   | 默认值                              |
|-------------------|------------------------|----------------------------------|
| master            | eligible\.node\.master | TRUE                             |
| data              | node\.data             | TRUE                             |
| ingest            | node\.ingest           | TRUE                             |
| coordinatingonly  | 无                      | 每个节点默认都是coordinate节点设置其他类型为false |
| machine learning  | node\.ml               | true\(需设置enablex\-pack\)         |


## 3. 分片（Primary Shard & Replica Shard)

 - 主分片，用以解决数据水平扩展的问题。通过主分片，可以将数据分布到集群内的所有节点之上
1.一个分片是一个运行的 Lucene 的实例
2.主分片数在索引创建时指定，后续不允许修改，除非 通过Reindex方式进行
 - 副本，用以解决数据高可用的问题，分片是主分片的拷贝
1.副本分片数，可以动态调整
2.增加副本数，还可以在一定程度上提高服务的可用性（读取的吞吐）
 - 一个三节点的集群中，blogs索引的分片分布情况

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97c9d7e1e5bf356cd8bc0ef48769f1c0.png#pic_center)
一个主分片分散到三个节点上，每个节点存在一个副本分片
### 3.1 分片的设定

对于生成环境分片的设定，需要提前做好容量规划

 - 分片数设置过小：
1.导致后续无法增加节点实现水平扩展
2.单个分片的数据量太大，导致数据重新分配耗时
 - 分片数设置过大，7.0开始，默认主分片设置为1，解决了over-sharding的问题：
1.影响搜索结果的相关性打分，影响统计结果的准确性
2.单个节点上过多的分片，会导致资源浪费，同时也会影响性能

查看集群的健康状况

```bash
GET _cluster/health
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 6,
  "active_shards" : 6,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 1,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 85.71428571428571
}
```

 - Green-主分片与副本都正常分配
 - Yellow-主分片全部正常分配，副本分片未能正常分配
 - Red-有主分片未能分配

例如，当服务器的磁盘容量超过85%时，去创建一个新的索引
## 4. 操作示例

```bash
get _cat/nodes?v
GET /_nodes/es7_01,es7_02
GET /_cat/nodes?v
GET /_cat/nodes?v&h=id,ip,port,v,m


GET _cluster/health
GET _cluster/health?level=shards
GET /_cluster/health/kibana_sample_data_ecommerce,kibana_sample_data_flights
GET /_cluster/health/kibana_sample_data_flights?level=shards

#### cluster state
The cluster state API allows access to metadata representing the state of the whole cluster. This includes information such as
GET /_cluster/state

#cluster get settings
GET /_cluster/settings
GET /_cluster/settings?include_defaults=true

GET _cat/shards
GET _cat/shards?h=index,shard,prirep,state,unassigned.reason
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8331db487b1d70b73a4ea56aed0ee436.png#pic_center)
参考资料：
极客时间：Elasticsearch核心技术与实战

相关阅读：
[初学elasticsearch入门](https://blog.csdn.net/xixihahalelehehe/article/details/109380768)
[Elasticsearch本地安装与简单配置](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)
[docker-compose安装elasticsearch集群](https://blog.csdn.net/xixihahalelehehe/article/details/109389780)
[Elasticsearch 7.X之文档、索引、REST API详解](https://blog.csdn.net/xixihahalelehehe/article/details/109406518)
