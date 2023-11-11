

----
## 1. Bucket & Metric Aggregation

 - Metric 一些系列的统计方法
 - Bucket 一组满足条件的文档
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305185028503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 2. Aggregation 的语法
Aggregation 属于 Search 的一部分。一般情况下，建议将其 Size 指定为 0
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305110909164.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
demo
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305110920541.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 3. Mertric Aggregation
单值分析：只输出一个分析结果
 - `min，max，avg，sum`
 - `Cardinality`（类似 distinct Count）

多值分析：输出多个分析结果

 - `stats ,extended stats`
 - `percentile, percentile rank`
 - `top hits` （排在前面的示例）

### 3.1 Metric 聚合的具体 Demo
查看最低工资
查看最高工资
一个聚合输出多个值
一次查询包含多个聚合

 - 同时查看最低 最高 和平均工资

```bash
DELETE /employees
#做一个员工表的定义
PUT /employees/
{
  "mappings" : {
      "properties" : {
        "age" : {
          "type" : "integer"
        },
        "gender" : {
          "type" : "keyword"
        },
        "job" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 50
            }
          }
        },
        "name" : {
          "type" : "keyword"
        },
        "salary" : {
          "type" : "integer"
        }
      }
    }
}
```
插入数据

```bash
PUT /employees/_bulk
{ "index" : {  "_id" : "1" } }
{ "name" : "Emma","age":32,"job":"Product Manager","gender":"female","salary":35000 }
{ "index" : {  "_id" : "2" } }
{ "name" : "Underwood","age":41,"job":"Dev Manager","gender":"male","salary": 50000}
{ "index" : {  "_id" : "3" } }
{ "name" : "Tran","age":25,"job":"Web Designer","gender":"male","salary":18000 }
{ "index" : {  "_id" : "4" } }
{ "name" : "Rivera","age":26,"job":"Web Designer","gender":"female","salary": 22000}
{ "index" : {  "_id" : "5" } }
{ "name" : "Rose","age":25,"job":"QA","gender":"female","salary":18000 }
{ "index" : {  "_id" : "6" } }
{ "name" : "Lucy","age":31,"job":"QA","gender":"female","salary": 25000}
{ "index" : {  "_id" : "7" } }
{ "name" : "Byrd","age":27,"job":"QA","gender":"male","salary":20000 }
{ "index" : {  "_id" : "8" } }
{ "name" : "Foster","age":27,"job":"Java Programmer","gender":"male","salary": 20000}
{ "index" : {  "_id" : "9" } }
{ "name" : "Gregory","age":32,"job":"Java Programmer","gender":"male","salary":22000 }
{ "index" : {  "_id" : "10" } }
{ "name" : "Bryant","age":20,"job":"Java Programmer","gender":"male","salary": 9000}
{ "index" : {  "_id" : "11" } }
{ "name" : "Jenny","age":36,"job":"Java Programmer","gender":"female","salary":38000 }
{ "index" : {  "_id" : "12" } }
{ "name" : "Mcdonald","age":31,"job":"Java Programmer","gender":"male","salary": 32000}
{ "index" : {  "_id" : "13" } }
{ "name" : "Jonthna","age":30,"job":"Java Programmer","gender":"female","salary":30000 }
{ "index" : {  "_id" : "14" } }
{ "name" : "Marshall","age":32,"job":"Javascript Programmer","gender":"male","salary": 25000}
{ "index" : {  "_id" : "15" } }
{ "name" : "King","age":33,"job":"Java Programmer","gender":"male","salary":28000 }
{ "index" : {  "_id" : "16" } }
{ "name" : "Mccarthy","age":21,"job":"Javascript Programmer","gender":"male","salary": 16000}
{ "index" : {  "_id" : "17" } }
{ "name" : "Goodwin","age":25,"job":"Javascript Programmer","gender":"male","salary": 16000}
{ "index" : {  "_id" : "18" } }
{ "name" : "Catherine","age":29,"job":"Javascript Programmer","gender":"female","salary": 20000}
{ "index" : {  "_id" : "19" } }
{ "name" : "Boone","age":30,"job":"DBA","gender":"male","salary": 30000}
{ "index" : {  "_id" : "20" } }
{ "name" : "Kathy","age":29,"job":"DBA","gender":"female","salary": 20000}
```

### 3.2 Metric 聚合，找到最低的工资

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "min_salary": {
      "min": {
        "field":"salary"
      }
    }
  }
}
返回输出：
{
  "took" : 943,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "min_salary" : {
      "value" : 9000.0
    }
  }
}
```


### 3.3 Metric 聚合，找到最高的工资

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "max_salary": {
      "max": {
        "field":"salary"
      }
    }
  }
}
返回输出：
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "max_salary" : {
      "value" : 50000.0
    }
  }
}
```

### 3.4 多个 Metric 聚合，找到最低最高和平均工资

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "max_salary": {
      "max": {
        "field": "salary"
      }
    },
    "min_salary": {
      "min": {
        "field": "salary"
      }
    },
    "avg_salary": {
      "avg": {
        "field": "salary"
      }
    }
  }
}

返回输出：
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "max_salary" : {
      "value" : 50000.0
    },
    "avg_salary" : {
      "value" : 24700.0
    },
    "min_salary" : {
      "value" : 9000.0
    }
  }
}

```

### 3.5 一个聚合，输出多值

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "stats_salary": {
      "stats": {
        "field":"salary"
      }
    }
  }
}

返回输出：
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "stats_salary" : {
      "count" : 20,
      "min" : 9000.0,
      "max" : 50000.0,
      "avg" : 24700.0,
      "sum" : 494000.0
    }
  }
}

```


## 4. bucket Aggregation
按照一定的规则，将文档分配到不同的桶中，从而达到分类的目的。ES 提供的一些常见的 Bucket Aggregation

 - `Term`
 - 数字类型：`Range 、Date Range、Histogram / Data Histogram`

支持嵌套：也就在桶里在做分桶
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305112245995.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

### 4.1 Terms Aggregation
字段需要打开 fielddata，才能进行 Terms Aggregation

 - Keyword 默认支持 `doc_values`
 - Text 需要在 Mapping 中 enable ，会按照分词后的结果进行分

Demo

 - 对 job 和 job.keyword 进行聚合
 - 对性别进行 Terms 聚合
 - 指定 bucket size

### 4.2 对keword 进行聚合

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword"
      }
    }
  }
}
返回输出：

{
  "took" : 12,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "jobs" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Java Programmer",
          "doc_count" : 7
        },
        {
          "key" : "Javascript Programmer",
          "doc_count" : 4
        },
        {
          "key" : "QA",
          "doc_count" : 3
        },
        {
          "key" : "DBA",
          "doc_count" : 2
        },
        {
          "key" : "Web Designer",
          "doc_count" : 2
        },
        {
          "key" : "Dev Manager",
          "doc_count" : 1
        },
        {
          "key" : "Product Manager",
          "doc_count" : 1
        }
      ]
    }
  }
}
```


### 4.3 对 Text 字段进行 terms 聚合查询，失败

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job"
      }
    }
  }
}

返回输出：
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [job] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "employees",
        "node": "tPL1-C2IT6-eVbv5FfEWlg",
        "reason": {
          "type": "illegal_argument_exception",
          "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [job] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
        }
      }
    ],
    "caused_by": {
      "type": "illegal_argument_exception",
      "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [job] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.",
      "caused_by": {
        "type": "illegal_argument_exception",
        "reason": "Fielddata is disabled on text fields by default. Set fielddata=true on [job] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead."
      }
    }
  },
  "status": 400
}
```

### 4.4 对 Text 字段打开 fielddata，支持terms aggregation

```bash
PUT employees/_mapping
{
  "properties" : {
    "job":{
       "type":     "text",
       "fielddata": true
    }
  }
}
```

### 4.5 对 Text 字段进行 terms 分词。分词后的terms

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job"
      }
    }
  }
}

返回输出：
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "jobs" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "programmer",
          "doc_count" : 11
        },
        {
          "key" : "java",
          "doc_count" : 7
        },
        {
          "key" : "javascript",
          "doc_count" : 4
        },
        {
          "key" : "qa",
          "doc_count" : 3
        },
        {
          "key" : "dba",
          "doc_count" : 2
        },
        {
          "key" : "designer",
          "doc_count" : 2
        },
        {
          "key" : "manager",
          "doc_count" : 2
        },
        {
          "key" : "web",
          "doc_count" : 2
        },
        {
          "key" : "dev",
          "doc_count" : 1
        },
        {
          "key" : "product",
          "doc_count" : 1
        }
      ]
    }
  }
}



POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword"
      }
    }
  }
}

返回输出：
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "jobs" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Java Programmer",
          "doc_count" : 7
        },
        {
          "key" : "Javascript Programmer",
          "doc_count" : 4
        },
        {
          "key" : "QA",
          "doc_count" : 3
        },
        {
          "key" : "DBA",
          "doc_count" : 2
        },
        {
          "key" : "Web Designer",
          "doc_count" : 2
        },
        {
          "key" : "Dev Manager",
          "doc_count" : 1
        },
        {
          "key" : "Product Manager",
          "doc_count" : 1
        }
      ]
    }
  }
}

```



### 4.6 对job.keyword 和 job 进行 terms 聚合，分桶的总数并不一样

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "cardinate": {
      "cardinality": {
        "field": "job"
      }
    }
  }
}

返回输出：
{
  "took" : 30,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "cardinate" : {
      "value" : 10
    }
  }
}

```


```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "cardinate": {
      "cardinality": {
        "field": "job。keyword"
      }
    }
  }
}

返回输出：
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "cardinate" : {
      "value" : 0
    }
  }
}
```

### 4.7 对性别的 keyword 进行聚合

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "gender": {
      "terms": {
        "field":"gender"
      }
    }
  }
}


返回输出：
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "gender" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "male",
          "doc_count" : 12
        },
        {
          "key" : "female",
          "doc_count" : 8
        }
      ]
    }
  }
}

```

### 4.8 指定 bucket 的 size

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "ages_5": {
      "terms": {
        "field":"age",
        "size":3
      }
    }
  }
}

返回输出：
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "ages_5" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 12,
      "buckets" : [
        {
          "key" : 25,
          "doc_count" : 3
        },
        {
          "key" : 32,
          "doc_count" : 3
        },
        {
          "key" : 27,
          "doc_count" : 2
        }
      ]
    }
  }
}
```


### 4.9 指定size，不同工种中，年纪最大的3个员工的具体信息

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field":"job.keyword"
      },
      "aggs":{
        "old_employee":{
          "top_hits":{
            "size":3,
            "sort":[
              {
                "age":{
                  "order":"desc"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```
### 4.10 优化Terms聚合性能
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305191339625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


## 5. Range & Histogram聚合
### 5.1 Salary Ranges 分桶，可以自己定义 key

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "salary_range": {
      "range": {
        "field":"salary",
        "ranges":[
          {
            "to":10000
          },
          {
            "from":10000,
            "to":20000
          },
          {
            "key":">20000",
            "from":20000
          }
        ]
      }
    }
  }
}

返回输出：

{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 20,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "salary_range" : {
      "buckets" : [
        {
          "key" : "*-10000.0",
          "to" : 10000.0,
          "doc_count" : 1
        },
        {
          "key" : "10000.0-20000.0",
          "from" : 10000.0,
          "to" : 20000.0,
          "doc_count" : 4
        },
        {
          "key" : ">20000",
          "from" : 20000.0,
          "doc_count" : 15
        }
      ]
    }
  }
}

```


### 5.2 Salary Histogram,工资0到10万，以 5000一个区间进行分桶

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "salary_histrogram": {
      "histogram": {
        "field":"salary",
        "interval":5000,
        "extended_bounds":{
          "min":0,
          "max":100000

        }
      }
    }
  }
}
```



### 5.3 嵌套聚合1，按照工作类型分桶，并统计工资信息

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "Job_salary_stats": {
      "terms": {
        "field": "job.keyword"
      },
      "aggs": {
        "salary": {
          "stats": {
            "field": "salary"
          }
        }
      }
    }
  }
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305192224437.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 5.4 多次嵌套。根据工作类型分桶，然后按照性别分桶，计算工资的统计信息

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "Job_gender_stats": {
      "terms": {
        "field": "job.keyword"
      },
      "aggs": {
        "gender_stats": {
          "terms": {
            "field": "gender"
          },
          "aggs": {
            "salary_stats": {
              "stats": {
                "field": "salary"
              }
            }
          }
        }
      }
    }
  }
}
```


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210305192352275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)




