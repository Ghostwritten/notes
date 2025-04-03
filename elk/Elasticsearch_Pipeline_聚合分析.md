

一个例子：Pipeline： `min_bucket`
在员工数最多的工种里，找出平均工资最低的工种

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/335c1e9bb0e86ce093b8e31a22fd9938.png)

关键词：`buckets_path`
## 1. Pipeline
管道的概念：支持对聚合分析的结果，再次进行聚合分析
Pipeline 的分析结果会输出到原结果汇总，根据位置的不同，分为两类

 - Sibling - 结果和现有分析结果同级
 Max，min，Avg&Sum Bucket
Stats ， Extened Status Bucket
Percentiles Bucket
 - Parent - 结果内嵌到现有的聚合分析结果之中
Derivative（求导）
Cumultive Sum（累计求和）
Moving Function（滑动窗口）

## 2. Sibling Pipeline 的例子
对不同类型工作的，平均工资

 - 求最大
 - 平均
 - 统计信息
 - 百分位数

### 2.1 插入数据
```bash
DELETE employees
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


### 2.2 平均工资最低的工作类型

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "min_salary_by_job":{
      "min_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```
返回输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/00667d73a52b809db1452918ef13bc91.png)


### 2.3 平均工资最高的工作类型

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "max_salary_by_job":{
      "max_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```
返回输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/312b6a35d212a6a7481174c11e076086.png)


### 2.4 平均工资的平均工资

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "avg_salary_by_job":{
      "avg_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```
返回输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/469b64c9fbe0991a8c581511c2c03240.png)


### 2.5 平均工资的统计分析

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "stats_salary_by_job":{
      "stats_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```
返回输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/013c964e87e087adfa317dbe62ffe7d3.png)

### 2.6 平均工资的百分位数

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "jobs": {
      "terms": {
        "field": "job.keyword",
        "size": 10
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        }
      }
    },
    "percentiles_salary_by_job":{
      "percentiles_bucket": {
        "buckets_path": "jobs>avg_salary"
      }
    }
  }
}
```
返回输出：

```bash
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
    "jobs" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "Java Programmer",
          "doc_count" : 7,
          "avg_salary" : {
            "value" : 25571.428571428572
          }
        },
        {
          "key" : "Javascript Programmer",
          "doc_count" : 4,
          "avg_salary" : {
            "value" : 19250.0
          }
        },
        {
          "key" : "QA",
          "doc_count" : 3,
          "avg_salary" : {
            "value" : 21000.0
          }
        },
        {
          "key" : "DBA",
          "doc_count" : 2,
          "avg_salary" : {
            "value" : 25000.0
          }
        },
        {
          "key" : "Web Designer",
          "doc_count" : 2,
          "avg_salary" : {
            "value" : 20000.0
          }
        },
        {
          "key" : "Dev Manager",
          "doc_count" : 1,
          "avg_salary" : {
            "value" : 50000.0
          }
        },
        {
          "key" : "Product Manager",
          "doc_count" : 1,
          "avg_salary" : {
            "value" : 35000.0
          }
        }
      ]
    },
    "percentiles_salary_by_job" : {
      "values" : {
        "1.0" : 19250.0,
        "5.0" : 19250.0,
        "25.0" : 21000.0,
        "50.0" : 25000.0,
        "75.0" : 35000.0,
        "95.0" : 50000.0,
        "99.0" : 50000.0
      }
    }
  }
}
```

### 2.7 按照年龄对平均工资求导

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "age": {
      "histogram": {
        "field": "age",
        "min_doc_count": 1,
        "interval": 1
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        },
        "derivative_avg_salary":{
          "derivative": {
            "buckets_path": "avg_salary"
          }
        }
      }
    }
  }
}
```
返回输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e076418999221f1cc6f187028181e0fa.png)


### 2.8 Cumulative_sum（累计求和）

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "age": {
      "histogram": {
        "field": "age",
        "min_doc_count": 1,
        "interval": 1
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        },
        "cumulative_salary":{
          "cumulative_sum": {
            "buckets_path": "avg_salary"
          }
        }
      }
    }
  }
}
```
返回输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e2982c93167246bd1101436a3ddd6480.png)

### 2.9 Moving Function（移动平均）

```bash
POST employees/_search
{
  "size": 0,
  "aggs": {
    "age": {
      "histogram": {
        "field": "age",
        "min_doc_count": 1,
        "interval": 1
      },
      "aggs": {
        "avg_salary": {
          "avg": {
            "field": "salary"
          }
        },
        "moving_avg_salary":{
          "moving_fn": {
            "buckets_path": "avg_salary",
            "window":10,
            "script": "MovingFunctions.min(values)"
          }
        }
      }
    }
  }
}
```

返回输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/297d7bcfdb3f07df3535ec87f3b3fb2d.png)




