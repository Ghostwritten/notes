

----
## 1. 节点类型
不同角色的节点

 - `Master eligible / Data / Ingest / Coordinating / Machine Learning`

在开发环境中，一个节点可承担多种角色

在生产环境中

 - 根据数据量，写入和查询的吞吐量，选择适合的部署方式
 - 建议设置单一角色的节点（dedicated node）

## 2. 节点参数配置
一个节点在默认情况下会同时扮演： `master eligible` ，`data node` 和 `ingest node`

| 节点类型              | 配置参数        | 默认值                     |
|-------------------|-------------|-------------------------|
| master eligible   | node.master | true                    |
| data              | node.data   | true                    |
| ingest            | node.ingest | true                    |
| coodrinating only | 无           | 设置上面三个参数全部为 false       |
| machine learning  | node.ml     | true (需要 enable x-pack) |


## 3. 单一职责的节点
一个节点只承担一个角色
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312101449714.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. 单一角色：职责分离的好处
Dedicated master eligible nodes：负责集群状态（cluster state）的管理

 - 使用低配置的 CPU ,RAM 和磁盘

Dedicated data nodes: 负责数据存储及处理客户端请求

 - 使用高配置的 CPU,RAM 和磁盘

Dedicated ingest nodes: 负责数据处理

 - 使用高配置的 CPU ; 中等配置的 RAM; 低配置的磁盘

## 5. Dedicate Coordinating Only Node (Client Node)
配置：将 Master ，Data ，Ingest 都配置成 Flase

 - Medium / High CUP; Medium / High RAM;Low Disk

生产环境中，建议为一些大的集群配置 Coordinating Only Nodes

 - 扮演 Load Balancers。 降低 Master 和 Data Nodes 的负载
 - 负载搜索结果的 Gather / Reduce
 - 有时候无法预知客户端会发生怎样的请求
大量占用内存的结合操作，一个深度聚合可能引发 OOM
## 6. Dedicate Master Node
 从高可用 & 避免脑裂的角色出发
 - 一般在生产环境中配置 3 台
 - 一个集群只有 1 台活跃的主节点
 - 负载分片管理，索引创建，集群管理等操作

如果和数据节点或者 Coordinate 节点混合部署

 - 数据节点相对有比较大的内存占用
 - Coordinate 节点有时候可能会有开销很高的查询，导致 OOM
 - 这些都有可能影响 Master 节点，导致集群的不稳定

## 7. 基本部署：增减节点，水平扩展
当磁盘容量无法满足需求时，可以增加数据节点；磁盘读写压力大时，增加数据节点
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312102646110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


## 8. 水平扩展：Coordinating Only Node
当系统中有大量的复杂查询及聚合时候，增加 Coordinating 节点，增加查询的性能
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312102706865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 9. 读写分离
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312102726758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 10. 在集群里部署 Kibana
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031210281127.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 11. 异地多活的部署
集群处在三个数据中心；数据三写；GTM 分发读请求
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312102835358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

