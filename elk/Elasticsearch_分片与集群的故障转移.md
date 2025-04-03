

----

**7.0版本开始，默认主分片为1，副分片为0**
## 1. Primary Shard - 提升系统存储容量
分片是 ES 分布式储存的基石

 - 主分片 / 副本分片

通过主分片，将数据分布在所有节点上

 - Primary Shard , 可以将一份索引的数据，分散在多个 Data Node 上，实现储存的水平扩展
 - 主分片（Primary Shard）数在索引创建时候指定，后续默认不能修改，如要修改，需重建索引

## 2. Replica Shard - 提高数据可用性
数据可用性

 - 通过引入副本分片（Replica Shard）提高数据的可用性。一旦主分片丢失，副本分片可以在 `Promote`成主分片。副本分片数可以动态调整的。每个节点上都有完备的数据。如果不设置副本分片，一旦出现节点硬件故障，就有可能造成数据丢失。

提高系统的读取性能

 - 副本分片由主分片（Primary Shard）同步。通过支持增加 Replica 个数，一定程度可以提高读取的吞吐量

## 3. 分片数的设置
如何规划一个索引的主分片数和副本分片数

 - 主分片数过小：例如创建一个 1 个 Primary Shard 的 index，如果该索引增长很快，集群无法通过增加节点实现对这个索引的数据扩展
 - 主分片数设置过大：导致单个 Shard 容量很小，引发一个节点上有过多分片，影响性能 
 - 副本分片设置过多，会降低集群整体的写入性能

## 4. 单节点集群
副本无法分片，集群状态为黄色

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7de140f83ec472e0846c6596d2f2ea2a.png)

## 5. 增加一个数据节点

 - 集群状态转为绿色
 - 集群具备故障转移能力
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d3a0efa4c4e52c3a77ecb3a37a99d62.png)
## 6. 故障转移
 - 3 个节点共同组成。包含 1 个索引，索引设置了 3 个 Primary Shard 和 1 个 Replica 节点 1 是 Master 节点，节点意外出现故障。集群重新选举 Master 节点
 - Node3 上的 R0 提升成 P0 ，集群变黄
 - R0 R1 分配，集群变绿
 - 需要集群具备能力，必须将索引的副本分片设置为 1，否则一丢失节点，就会造成数据丢失

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ebb457053410b860559dbb200df81064.png)
## 7. 集群健康状态

 - Green : 健康状态，所有的主分片和副本分片都可用
 - Yellow: 亚健康，所有的主分片可用，部分副本分片不可用
 - Red：不健康状态，部分主分片不可用

```bash
GET /_cluster/health
{
"cluster_name" : "lsk",
"status" : "yellow",
"timed_out" : false,
"number_of_nodes" : 2,
"number_of_data_nodes" : 2,
"active_primary_shards" : 16,
"active_shards" : 32,
"relocating_shards" : 0,
"initializing_shards" : 0,
"unassigned_shards" : 6,
"delayed_unassigned_shards" : 0,
"number_of_pending_tasks" : 0,
"number_of_in_flight_fetch" : 0,
"task_max_waiting_in_queue_millis" : 0,
"active_shards_percent_as_number" : 84.21052631578947
}
```
## 8. demo

 - 启动一个节点，3 个 Primary shard，1 个 Replica，集群黄色，因为无法分片 Replica
 - 启动 3 个节点，1 个索引上包含 3 个 Primary Shard，一个 Replica
 - 关闭 Node 1（Master）
 - 查看 Master Node 重新选举
 - 集群变黄，然后重新分配
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f3a2fc49e31426f4129baf2406377e1.png)
我杀掉第三个节点
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/31d35a2e0e7605a2592813e1dde52587.png)

