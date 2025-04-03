

----
## 1. 并发控制的必要性
两个 Web 程序同时更新某个文档，如果缺乏有效的并发，会导致更改的数据丢失
悲观并发控制

 - 假设有变更冲突的可能，会对资源加锁，防止冲突。例如数据库行锁

乐观并发控制

 - 假设突然是不会发生的，不会阻塞正在尝试的操作。如果数据在读写中被修改，更新将会失败。应用程序决定如何解决冲突，例如重试更新，使用新的数据，或者将错误报告给用户
 - ES 采用的乐观并发控制
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a62b27df6c644281be6636a89c911dc2.png)
## 2. ES 的乐观并发控制
ES 中的文档是不可变更的。如果你更新一个文档，会将会文档标记为删除，同时增加一个全新当文档，同时文档的 version 字段加 1
内部版本控制
 - If_seq_no + If_primary_term



使用外部版本（使用其他数据库作为主要数据存储）

 - version + version_type = external

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fa2eff34fa58f6f1c95d19c2561d01d9.png)
## 3. demo

```bash
DELETE products
PUT products
PUT products/_doc/1
{
  "title":"iphone",
  "count":100
}
返回输出：
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,        
  "_primary_term" : 1
}


GET products/_doc/1
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "title" : "iphone",
    "count" : 100
  }
}


//只能执行一次
PUT products/_doc/1?if_seq_no=0&if_primary_term=1
{
  "title":"iphone",
  "count":110
}

返回输出：_seq_no增1
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

#如果默认另外一个人查询
PUT products/_doc/1?if_seq_no=0&if_primary_term=1
{
  "title":"iphone",
  "count":113
}
返回输出：报错
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[1]: version conflict, required seqNo [0], primary term [1]. current document has seqNo [1] and primary term [1]",
        "index_uuid": "V8OgL-0mQq6WST217HUCgA",
        "shard": "0",
        "index": "products"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[1]: version conflict, required seqNo [0], primary term [1]. current document has seqNo [1] and primary term [1]",
    "index_uuid": "V8OgL-0mQq6WST217HUCgA",
    "shard": "0",
    "index": "products"
  },
  "status": 409
}


//数据库版本号为主
PUT products/_doc/1?version=23&version_type=external
{
  "title":"iphone",
  "count":130
}

返回输出：
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 23,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}

#查询

GET products/_doc/1
返回输出：
{
  "_index" : "products",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 23,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "title" : "iphone",
    "count" : 130
  }
}
```

