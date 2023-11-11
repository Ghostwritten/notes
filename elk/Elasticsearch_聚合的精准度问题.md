



----
## 1. 分布式系统的近似统计算法#

## ![](https://img-blog.csdnimg.cn/20210305201721260.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. Min 聚合分析的执行流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305201749344.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 3. terms Aggregation 的返回值
在 Terms Aggregation 的返回中有两个特殊的数值

 - `doc_count_error_upper_bound`：被遗漏的 term 分桶，包含的文档，有可能的最大值
 - `sum_other_doc_count`: 处理返回结果 bucket 的 terms 以外，其他 terms 的文档总数（总数 -返回的总数）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306101757584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. Terms 聚合分析的执行流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021030610190293.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 5. Terms 不正确的案例
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306102040826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 6. 如何解决 Terms 不准的问题：提升 shard_size 的参数
 - Terms 聚合分析不准的原因，数据分散在多个分片上，Coordinating Node 无法获取数据全貌
 - 解决方案 1：当数据量不大时，设置 `Primary Shard` 为 1；实现准确性
 - 解决方案 2：在分布式数据上，设置 `shard_size` 参数，提高精确度
 - 原理：每次从 Shard 上额外多获取数据，提升准确率
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306102204220.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 7. 打开 show_term_doc_count_error
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210306102254479.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 8. shard_size 设定
调整 `shard size` 大小，降低 `doc_count_error_upper_bound` 来提升准确度
 - 增加整体计算量，提高了准确率，但会降低相应时间

Shard Size 默认大小设定

 - shard size = size * 1.5 +10

## 9. demo
### 9.1 插入数据

```bash
DELETE my_flights
PUT my_flights
{
  "settings": {
    "number_of_shards": 20
  },
  "mappings" : {
      "properties" : {
        "AvgTicketPrice" : {
          "type" : "float"
        },
        "Cancelled" : {
          "type" : "boolean"
        },
        "Carrier" : {
          "type" : "keyword"
        },
        "Dest" : {
          "type" : "keyword"
        },
        "DestAirportID" : {
          "type" : "keyword"
        },
        "DestCityName" : {
          "type" : "keyword"
        },
        "DestCountry" : {
          "type" : "keyword"
        },
        "DestLocation" : {
          "type" : "geo_point"
        },
        "DestRegion" : {
          "type" : "keyword"
        },
        "DestWeather" : {
          "type" : "keyword"
        },
        "DistanceKilometers" : {
          "type" : "float"
        },
        "DistanceMiles" : {
          "type" : "float"
        },
        "FlightDelay" : {
          "type" : "boolean"
        },
        "FlightDelayMin" : {
          "type" : "integer"
        },
        "FlightDelayType" : {
          "type" : "keyword"
        },
        "FlightNum" : {
          "type" : "keyword"
        },
        "FlightTimeHour" : {
          "type" : "keyword"
        },
        "FlightTimeMin" : {
          "type" : "float"
        },
        "Origin" : {
          "type" : "keyword"
        },
        "OriginAirportID" : {
          "type" : "keyword"
        },
        "OriginCityName" : {
          "type" : "keyword"
        },
        "OriginCountry" : {
          "type" : "keyword"
        },
        "OriginLocation" : {
          "type" : "geo_point"
        },
        "OriginRegion" : {
          "type" : "keyword"
        },
        "OriginWeather" : {
          "type" : "keyword"
        },
        "dayOfWeek" : {
          "type" : "integer"
        },
        "timestamp" : {
          "type" : "date"
        }
      }
    }
}
```
### 9.2 将索引kibana_sample_data_flights数据导入my_flights

```bash
POST _reindex
{
  "source": {
    "index": "kibana_sample_data_flights"
  },
  "dest": {
    "index": "my_flights"
  }
}

返回输出
{
  "took" : 3221,
  "timed_out" : false,
  "total" : 13059,
  "updated" : 0,
  "created" : 13059,
  "deleted" : 0,
  "batches" : 14,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```

```bash
GET kibana_sample_data_flights/_count
GET my_flights/_count
返回输出：
{
  "count" : 13059,
  "_shards" : {
    "total" : 20,
    "successful" : 20,
    "skipped" : 0,
    "failed" : 0
  }
}


get kibana_sample_data_flights/_search
```

```bash
GET kibana_sample_data_flights/_search
{
  "size": 0,
  "aggs": {
    "weather": {
      "terms": {
        "field":"OriginWeather",
        "size":5,
        "show_term_doc_count_error":true
      }
    }
  }
}

返回输出：
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "weather" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 2932,
      "buckets" : [
        {
          "key" : "Clear",
          "doc_count" : 2324,
          "doc_count_error_upper_bound" : 0
        },
        {
          "key" : "Cloudy",
          "doc_count" : 2319,
          "doc_count_error_upper_bound" : 0
        },
        {
          "key" : "Rain",
          "doc_count" : 2214,
          "doc_count_error_upper_bound" : 0
        },
        {
          "key" : "Sunny",
          "doc_count" : 2209,
          "doc_count_error_upper_bound" : 0
        },
        {
          "key" : "Thunder & Lightning",
          "doc_count" : 1061,
          "doc_count_error_upper_bound" : 0
        }
      ]
    }
  }
}

```


```bash
GET my_flights/_search
{
  "size": 0,
  "aggs": {
    "weather": {
      "terms": {
        "field":"OriginWeather",
        "size":1,
        "shard_size":1,
        "show_term_doc_count_error":true
      }
    }
  }
}

返回输出：
{
  "took" : 18,
  "timed_out" : false,
  "_shards" : {
    "total" : 20,
    "successful" : 20,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "weather" : {
      "doc_count_error_upper_bound" : 2511,
      "sum_other_doc_count" : 12022,
      "buckets" : [
        {
          "key" : "Clear",
          "doc_count" : 1037,
          "doc_count_error_upper_bound" : 1474
        }
      ]
    }
  }
}

```



