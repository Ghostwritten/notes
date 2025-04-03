

----
## 1. 日志类应用的部署结构
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3a56d2126d881b12b1ac8dad43751fe.png)
## 2. 什么是 Hot & Warm Architecture
什么是 Hot & Warm Architecture

 - 数据通常不会有 Update 操作；适用于 Time based 索引数据（生命周期管理），同时数据量比较大的 场景
 - 引入 Warm 节点，低配置大容量的机器存放老数据，以降低部署成本

两类数据节点，不同的硬件配置

 - Hot 节点（通常使用 SSD）：索引有不断有新文档写入。通常使用 SSD
 - Warm 节点（通常使用 HDD）：索引不存在新数据的写入；同时也不存在大量的数据查询

## 3. Hot Nodes

 - 用于数据的写入
 - Indexing 对 CPU 和 IO 都有很高的要求。所以需要使用高配置的机器
 - 存储的性能要好。建议使用 SSD
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ba584beb0321db3619b9c1bba5e60ef8.png)
## 4. Warm Nodes
 - 用于保存只读的索引，比较旧的数据
 - 通常使用大容量的磁盘（通常是 Spinning Disks）
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/569b19d8bcd395cbf93760aa9d1e07db.png)
## 5. 配置 Hot & Warm Architecture
使用 Shard Filtering，步骤分为以下几步
 - 标记节点 （Tagging）
 - 配置索引到 Hot Node
 - 配置索引到 Warm 节点

### 5.1 标记节点

 - 需要通过 “node.attr” 来标记一个节点
 - 节点的 attribute 可以是任何的 key/value
 - 可以通过 elasticsearch.yml 或者通过 –E 命 令指定

ios本机执行：
```bash
# 标记一个 Hot 节点
bin/elasticsearch  -E node.name=hotnode -E cluster.name=geektime -E path.data=hot_data -E node.attr.my_node_type=hot

# 标记一个 warm 节点
bin/elasticsearch  -E node.name=warmnode -E cluster.name=geektime -E path.data=warm_data -E node.attr.my_node_type=warm
```

linux：

```bash
# 标记一个 Hot 节点
bin/elasticsearch  -E node.name=hotnode -E cluster.name=geektime -E path.data=hot_data -E node.attr.my_node_type=hot -E network.host=192.168.211.61 -E http.port=9200 -E discovery.seed_hosts=192.168.211.61:9300,192.168.211.61:9301 -E cluster.initial_master_nodes=192.168.211.61:9300
# 标记一个 warm 节点
bin/elasticsearch  -E node.name=warmnode -E cluster.name=geektime -E path.data=warm_data -E node.attr.my_node_type=warm -E network.host=192.168.211.61 -E http.port=9201 -E discovery.seed_hosts=192.168.211.61:9300,192.168.211.61:9301 -E cluster.initial_master_nodes=192.168.211.61:9300
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c59b63ecbe90222ef1aa6bc6390ea6d0.png)
运行kibana

```bash
[elastic@slave1 kibana-7.3.1-linux-x86_64]$ ./bin/kibana
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/493460ac24aaa26a384a8306088bcd6d.png)




### 5.2 配置 Hot 数据

```bash
创建索引时候，指定将其创建在 hot 节点上
PUT log-2020-03-04
{
"settings": {
  "number_of_shards": 2,
  "number_of_replicas": 0,
  "index.routing.allocation.require.my_node_type":"hot"
}
}
//return
index                shard prirep state   docs  store ip         node
log-2020-03-04       1     p      STARTED    0   230b 172.18.0.6 es7_hot
log-2020-03-04       0     p      STARTED    0   230b 172.18.0.6 es7_hot
```

### 5.3 旧数据移动到 Warm 节点
`Index.routing.allocation` 是一个索引级的 `dynamic setting`，可以通过 API 在后期进行设定

 - Curator / Index Life Cycle Management Tool

```bash
PUT log-2020-03-04/_settings
{
"index.routing.allocation.require.my_node_type":"warm"
}
//return 
index                shard prirep state   docs  store ip         node
log-2020-03-04       1     p      STARTED    0   230b 172.18.0.4 es7_warm
log-2020-03-04       0     p      STARTED    0   230b 172.18.0.2 es7_warm
```

### 5.4 实战demo
```bash
# 配置到 Hot节点
PUT logs-2019-06-27
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":0,
    "index.routing.allocation.require.my_node_type":"hot"
  }
}

#插入一条数据
PUT my_index1/_doc/1
{
  "key":"value"
}

GET _cat/shards?v
#return
index                shard prirep state   docs  store ip             node
.kibana_task_manager 0     p      STARTED    2 45.6kb 192.168.211.61 warmnode
.kibana_task_manager 0     r      STARTED    2   21kb 192.168.211.61 hotnode
my_index1            0     p      STARTED    1  3.3kb 192.168.211.61 warmnode
my_index1            0     r      STARTED    1  3.4kb 192.168.211.61 hotnode
.kibana_1            0     p      STARTED    4 22.5kb 192.168.211.61 warmnode
.kibana_1            0     r      STARTED    4 19.7kb 192.168.211.61 hotnode
logs-2019-06-27      1     p      STARTED    0   230b 192.168.211.61 hotnode
logs-2019-06-27      0     p      STARTED    0   230b 192.168.211.61 hotnode



# 配置到 warm 节点
PUT PUT logs-2019-06-27/_settings
{  
  "index.routing.allocation.require.my_node_type":"warm"
}

GET _cat/shards?v
#return
index                shard prirep state   docs  store ip             node
.kibana_task_manager 0     p      STARTED    2 45.6kb 192.168.211.61 warmnode
.kibana_task_manager 0     r      STARTED    2   21kb 192.168.211.61 hotnode
my_index1            0     p      STARTED    1  3.5kb 192.168.211.61 warmnode
my_index1            0     r      STARTED    1  3.5kb 192.168.211.61 hotnode
.kibana_1            0     p      STARTED    4 22.9kb 192.168.211.61 warmnode
.kibana_1            0     r      STARTED    4   20kb 192.168.211.61 hotnode
logs-2019-06-27      1     p      STARTED    0   283b 192.168.211.61 warmnode
logs-2019-06-27      0     p      STARTED    0   283b 192.168.211.61 warmnode
```
## 6. Rack Awareness
ES 的节点可能分布在不同的机架

 - 当一个机架断电，可能会同时丢失几个节点
 - 如果一个索引相同的主分片和副本分片，同时在这个机架上，就 有可能导致数据的丢失
 - 通过 Rack Awareness 的机制，就可以尽可能避免将同一个索引 的主副分片同时分配在一个机架的节点上
 - 一个机架断电，数据可以恢复
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/42ac7b52af62de58c543754481c7c49d.png)
### 6.1 标记 Rack 节点 + 配置集群
 - `cluster.routing.allocation.awareness.attributes": "my_rack_id`
表示将主从副本分配到不同rack节点上

 **实战demo**
ios本机
```bash
# 标记一个 rack 1
bin/elasticsearch  -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -E node.attr.my_rack_id=rack1

# 标记一个 rack 2
bin/elasticsearch  -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -E node.attr.my_rack_id=rack2
```

linux两台机器：

```bash
# 标记一个 rack 1
bin/elasticsearch  -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -E node.attr.my_rack_id=rack1 -E network.host=192.168.211.61 -E discovery.seed_hosts=192.168.211.61:9300,192.168.211.62:9300 -E cluster.initial_master_nodes=192.168.211.61:9300


# 标记一个 rack 2
 bin/elasticsearch  -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -E node.attr.my_rack_id=rack2 -E network.host=192.168.211.62 -E discovery.seed_hosts=192.168.211.61:9300,192.168.211.62:9300 -E cluster.initial_master_nodes=192.168.211.61:9300
```
测试

```bash
GET /_cat/nodeattrs?v
#return
node  host           ip             attr              value
node1 192.168.211.61 192.168.211.61 ml.machine_memory 3954188288
node1 192.168.211.61 192.168.211.61 xpack.installed   true
node1 192.168.211.61 192.168.211.61 ml.max_open_jobs  20
node1 192.168.211.61 192.168.211.61 my_rack_id        rack1
node2 192.168.211.62 192.168.211.62 ml.machine_memory 3954188288
node2 192.168.211.62 192.168.211.62 ml.max_open_jobs  20
node2 192.168.211.62 192.168.211.62 xpack.installed   true
node2 192.168.211.62 192.168.211.62 my_rack_id        rack2

#配置rack参数
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "my_rack_id"
  }
}

#一个副本两个分片
PUT my_index1
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":1
  }
}

#写入数据
PUT my_index1/_doc/1
{
  "key":"value"
}


#查看
GET _cat/shards?v
#return
index                shard prirep state   docs  store ip             node
.tasks               0     p      STARTED    1  6.3kb 192.168.211.61 node1
.tasks               0     r      STARTED    1  6.3kb 192.168.211.62 node2
.kibana_task_manager 0     p      STARTED    2 45.6kb 192.168.211.61 node1
.kibana_task_manager 0     r      STARTED    2 45.6kb 192.168.211.62 node2
.kibana_1            0     p      STARTED    1  6.7kb 192.168.211.61 node1
.kibana_1            0     r      STARTED    1  6.7kb 192.168.211.62 node2
.kibana_2            0     p      STARTED    3 14.7kb 192.168.211.61 node1
.kibana_2            0     r      STARTED    3 14.7kb 192.168.211.62 node2
my_index1            1     r      STARTED    0   230b 192.168.211.61 node1
my_index1            1     p      STARTED    0   230b 192.168.211.62 node2
my_index1            0     p      STARTED    1   230b 192.168.211.61 node1
my_index1            0     r      STARTED    1  3.3kb 192.168.211.62 node2

DELETE my_index1/_doc/1

```
### 6.2 Fore awareness
ios本机：
```bash
# 标记一个 rack 1
bin/elasticsearch  -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -E node.attr.my_rack_id=rack1

# 标记一个 rack 1
bin/elasticsearch  -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -E node.attr.my_rack_id=rack1
```
linux：

```bash
# 标记一个 rack 1
bin/elasticsearch  -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -E node.attr.my_rack_id=rack1 -E network.host=192.168.211.61 -E discovery.seed_hosts=192.168.211.61:9300,192.168.211.62:9300  cluster.initial_master_nodes=192.168.211.61:9300

# 标记一个 rack 1
bin/elasticsearch  -E node.name=node2 -E cluster.name=geektime -E path.data=node2_data -E node.attr.my_rack_id=rack1 -E network.host=192.168.211.62 -E discovery.seed_hosts=192.168.211.61:9300,192.168.211.62:9300 -E cluster.initial_master_nodes=192.168.211.61:9300
```

```bash
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "my_rack_id",
    "cluster.routing.allocation.awareness.force.my_rack_id.values": "rack1,rack2"
  }
}
GET _cluster/settings
#return
{
  "persistent" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "awareness" : {
            "attributes" : "my_rack_id",
            "force" : {
              "my_rack_id" : {
                "values" : "rack1,rack2"
              }
            }
          }
        }
      }
    }
  },
  "transient" : { }
}


#创建索引
PUT my_index1
{
  "settings":{
    "number_of_shards":2,
    "number_of_replicas":1
  }
}

#插入数据
PUT my_index1/_doc/1
{
  "key":"value"
}

# 集群黄色
GET _cluster/health
#return
{
  "cluster_name" : "geektime",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 3,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 66.66666666666666
}

# 副本无法分配
GET _cat/shards?v
#return
index     shard prirep state      docs  store ip             node
.kibana   0     r      STARTED       2  7.2kb 192.168.211.62 node2
.kibana   0     p      STARTED       2 10.6kb 192.168.211.61 node1
my_index1 1     p      STARTED       0   283b 192.168.211.61 node1
my_index1 1     r      UNASSIGNED                            
my_index1 0     p      STARTED       0   230b 192.168.211.62 node2
my_index1 0     r      UNASSIGNED                            


GET _cluster/allocation/explain?pretty
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5bfc7a48249b3933c2c6f25daa8f725.png)
## 7. Shard Filtering

 - `node.attr` - 标记节点
 - `index.routing.allocation` – 分配索引到节点


| 设置                                      | 分配索引到节点，节点的属性规则 |
|-----------------------------------------|-----------------|
| Index.routing.allocation.include.{attr} | 至少包含一个值         |
| Index.routing.allocation.exclude.{attr} | 不能包含任何一个值       |
| Index.routing.allocation.require.{attr} | 所有值都需要包含        |

