


----
## 1. 集群健康度
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bbf1a52320f66c80f02275b3378c00a9.png)

**分片健康**

 - 红：至少有一个主分片没有分配
 - 黄：至少有一个副本没有分配
 - 绿：主副本分片全部正常分配

**索引健康**：最差的分片的状态
**集群健康**：最差的索引的状态

## 2. Health 相关的 API
| GET _cluster/health               | 集群的状态（检查 节点数量）      |
|-----------------------------------|---------------------|
| GET _cluster/health?level=indices | 所有索引的健康状态 （查看有问题的索引 |
| GET _cluster/health/my_index      | 单个索引的健康状态（查看具体的索引）  |
| GET _cluster/health?level=shards  | 分片级的索引              |
| GET _cluster/allocation/explain   | 返回第一个未分配 Shard 的原因  |
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cb6c3edfdad3d9abf052320e4249b188.png)
## 3. 案例 1
[Elasticsearch docker-compose部署hot、warm、cold的elasticsearch集群](https://ghostwritten.blog.csdn.net/article/details/114820354)

 - 症状：集群变红
 - 分析：通过 `Allocation Explain API` 发现 创建索引失败，因为无法找到标记了相应 box type 的节点
 - 解决：删除索引，集群变绿。重新创建索引，并且指定正确的 routing box type，索引创建成 功。集群保持绿色状态


```bash
PUT mytest
{
  "settings":{
    "number_of_shards":3,
    "number_of_replicas":0,
    "index.routing.allocation.require.box_type":"hott"
  }
}
# 检查集群状态，查看是否有节点丢失，有多少分片无法分配
GET /_cluster/health/
#return
{
  "cluster_name" : "geektime-hwc",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 5,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 3,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 76.92307692307693
}

# 查看索引级别,找到红色的索引
GET /_cluster/health?level=indices
#return
{
  "cluster_name" : "geektime-hwc",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 5,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 3,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 76.92307692307693,
  "indices" : {
    "mytest" : {
      "status" : "red",
      "number_of_shards" : 3,
      "number_of_replicas" : 0,
      "active_primary_shards" : 0,
      "active_shards" : 0,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 3
    },
    ".kibana_task_manager" : {
      "status" : "green",
      "number_of_shards" : 1,
      "number_of_replicas" : 1,
      "active_primary_shards" : 1,
      "active_shards" : 2,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 0
    }
....

#查看索引的分片
GET _cluster/health?level=shards
#return
{
  "cluster_name" : "geektime-hwc",
  "status" : "red",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 5,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 3,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 76.92307692307693,
  "indices" : {
    "mytest" : {
      "status" : "red",
      "number_of_shards" : 3,
      "number_of_replicas" : 0,
      "active_primary_shards" : 0,
      "active_shards" : 0,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 3,
      "shards" : {
        "0" : {
          "status" : "red",
          "primary_active" : false,
          "active_shards" : 0,
          "relocating_shards" : 0,
          "initializing_shards" : 0,
          "unassigned_shards" : 1
        },
        "1" : {
          "status" : "red",
          "primary_active" : false,
          "active_shards" : 0,
          "relocating_shards" : 0,
          "initializing_shards" : 0,
          "unassigned_shards" : 1
        },
        "2" : {
          "status" : "red",
          "primary_active" : false,
          "active_shards" : 0,
          "relocating_shards" : 0,
          "initializing_shards" : 0,
          "unassigned_shards" : 1
        }
      }
    },

# Explain 变红的原因
GET /_cluster/allocation/explain
//return 
"deciders" : [
        {
          "decider" : "filter",
          "decision" : "NO",
          "explanation" : """node does not match index setting [index.routing.allocation.require] filters [box_type:"hott"]"""
        }
      ]

GET /_cat/shards/mytest

GET _cat/nodeattrs

DELETE mytest
# 查看集群 集群变绿
GET /_cluster/health/

PUT mytest
{
  "settings":{
    "number_of_shards":3,
    "number_of_replicas":0,
    "index.routing.allocation.require.box_type":"hot"
  }
}
```

## 4. 案例 2

 - 症状：集群变黄
 - 分析：通过 Allocation Explain API 发现无法在相同的节点上创建副本
 - 解决：将索引的副本数设置为 0，或者通过增加节点解决


```bash
PUT mytest
{
"settings":{
  "number_of_shards":2,
  "number_of_replicas":1,
  "index.routing.allocation.require.box_type":"hot"
}
}
GET _cluster/health
#return
{
  "cluster_name" : "geektime-hwc",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 7,
  "active_shards" : 12,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 85.71428571428571
}

GET _cat/shards/mytest
GET /_cluster/allocation/explain
#return,副本和分片不能放到一个节点上，但hot节点只有一个，所以变黄
    {
      "node_id" : "jppJ74VhT5usiF7-L5lN0g",
      "node_name" : "es7_hot",
      "transport_address" : "172.20.0.4:9300",
      "node_attributes" : {
        "ml.machine_memory" : "3954188288",
        "ml.max_open_jobs" : "20",
        "box_type" : "hot",
        "xpack.installed" : "true"
      },
      "node_decision" : "no",
      "weight_ranking" : 3,
      "deciders" : [
        {
          "decider" : "same_shard",
          "decision" : "NO",
          "explanation" : "the shard cannot be allocated to the same node on which a copy of the shard already exists [[mytest][1], node[jppJ74VhT5usiF7-L5lN0g], [P], s[STARTED], a[id=J_Wyn1mvR16s6bAV0MnGBA]]"
        }
      ]
    }
  ]
}

PUT mytest/_settings
{
  "number_of_replicas": 0
}
```

## 5. 分片没有被分配的一些原因

 - `INDEX_CREATE`: 创建索引导致。在索引的全部分片分配完成之前，会有短暂的 Red，不一定代表有问题
 - CLUSTER_RECOVER：集群重启阶段，会有这个问题
 - `INDEX_REOPEN`：Open 一个之前 Close 的索引
 - `DANGLING_INDEX_IMPORTED`：一个节点离开集群期间，有索引被删除。这个节点重新返回时，会导致 Dangling 的问题

## 6. 常见问题与解决方法

 - 集群变红，需要检查是否有节点离线。如果有，通常通过重启离线的节点可以解决问题
 - 由于配置导致的问题，需要修复相关的配置（例如错误的 box_type，错误的副本数）
 - 如果是测试的索引，可以直接删除
 - 因为磁盘空间限制，分片规则（Shard Filtering）引发的，需要调整规则或者增加节点
 - 对于节点返回集群，导致的 dangling 变红，可直接删除 dangling 索引

## 7. 集群 Red & Yellow 问题的总结

 - Red & Yellow 是集群运维中常见的问题
 - 除了集群故障，一些创建，增加副本等操作，都会导致集群短暂的 Red 和 Yellow，所以监控和报警时需要设置一定的延时
 - 通过检查节点数，使用 ES 提供的相关 API，找到真正的原因
 - 可以指定 Move 或者 Reallocate 分片

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/13167dbd244483ce73fbe37c2d2ce32c.png)

