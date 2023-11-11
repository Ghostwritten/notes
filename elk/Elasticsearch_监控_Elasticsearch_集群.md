

---
## 1. Elasticsearch Stats 相关的 API
Elasticsearch 提供了多个监控相关的 API

 - Node Stats： _nodes/stats
 - Cluster Stats: _cluster/stats
 - Index Stats: index_name/_stats

## 2. Elasticsearch Task API
查看 Task 相关的 API

 - Pending Cluster Tasks API: GET _cluster/pending_tasks
 - Task Management API :GET _tasks (可以用来 Cancel 一个 Task)

监控 Thread Pools

 - GET _nodes/thread_pool
 - GET _nodes/stats/thread_pool
 - GET _cat/thread_pool?v
 - GET _nodes/hot_threads

## 3. The Index & Query Slow Log

 - 支持将分片上， `Search` 和 `Fetch` 阶段的慢 查询写入文件
 - 支持为 `Query` 和 `Fetch` 分别定义阈值
 - 索引级的动态设置，可以按需设置，或者通 过 `Index Template` 统一设定 
 - Slog log 文件通过


   log4j2.properties 配置

![监控 Elasticsearch 集群]

```bash
PUT my_index/
{
  "settings": {
    "index.search.slowlog.threshold": {
      "query.warn": "10s",
      "query.info": "3s",
      "query.debug": "2s",
      "query.trace": "0s",
      "fetch.warn": "1s",
      "fetch.info": "600ms",
      "fetch.debug": "400ms",
      "fetch.trace": "0s"
    }
  }
}
```

## 4. 如何创建监控 Dashboard

 - 开发 `Elasticsearch plugin`，通过读取相关的监控 API，将数据发送到 ES，或者 TSDB
 - 使用 Metricbeats 搜集相关指标
 - 使用 Kibana 或 Grafana 创建 Dashboard
 - 可以开发 `Elasticsearch Exporter`，通过 `Prometheus` 监控 Elasticsearch 集群

## 5. demo

```bash
# Node Stats：
GET _nodes/stats

#Cluster Stats:
GET _cluster/stats

#Index Stats:
GET kibana_sample_data_ecommerce/_stats

#Pending Cluster Tasks API:
GET _cluster/pending_tasks

# 查看所有的 tasks，也支持 cancel task
GET _tasks


GET _nodes/thread_pool
GET _nodes/stats/thread_pool
GET _cat/thread_pool?v
GET _nodes/hot_threads
GET _nodes/stats/thread_pool


# 设置 Index Slowlogs
# the first 1000 characters of the doc's source will be logged
PUT my_index/_settings
{
  "index.indexing.slowlog":{
    "threshold.index":{
      "warn":"10s",
      "info": "4s",
      "debug":"2s",
      "trace":"0s"
    },
    "level":"trace",
    "source":1000  
  }
}

# 设置查询
DELETE my_index
//"0" logs all queries
PUT my_index/
{
  "settings": {
    "index.search.slowlog.threshold": {
      "query.warn": "10s",
      "query.info": "3s",
      "query.debug": "2s",
      "query.trace": "0s",
      "fetch.warn": "1s",
      "fetch.info": "600ms",
      "fetch.debug": "400ms",
      "fetch.trace": "0s"
    }
  }
}

GET my_index
```


