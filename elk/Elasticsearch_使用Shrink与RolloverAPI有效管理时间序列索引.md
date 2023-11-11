

-----
## 1. 索引管理 API

● Open / Close Index： 索引关闭后无法进行读写，但是索引数据不会被
删除
● Shrink Index：可以将索引的主分片数收缩到较小的值
● Split Index：可以扩大主分片个数
● Rollover Index：类似 Log4J 记录日志的方式，索引尺寸或者时间超过
一定值后，创建新的
● Rollup Index：对数据进行处理后，重新写入，减少数据量

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210316102315801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. Open / Close Index API
● 索引关闭后，对集群的相关开销基本降低为 0
● 但是无法被读取和搜索
● 当需要的时候，可以重新打开

## 3. 打开关闭索引

```bash
DELETE test
#查看索引是否存在
HEAD test

PUT test/_doc/1
{
  "key":"value"
}

#关闭索引
POST /test/_close
#索引存在
HEAD test
# 无法查询
POST test/_count

#打开索引
POST /test/_open
POST test/_search
{
  "query": {
    "match_all": {}
  }
}
POST test/_count
```

## 4. Shrink API
● ES 5.x 后推出的一个新功能，使用场景
○ 索引保存的数据量比较小，需要重新设定主分片数
○ 索引从 Hot 移动到 Warm 后，需要降低主分片数
● 会使用和源索引相同的配置创建一个新的索引，仅仅降低主分片数
○ 源分片数必须是目标分片数的倍数。如果源分片数是素数，目标分片数只能为 1
○ 如果文件系统支持硬链接，会将 Segments 硬连接到目标索引，所以性能好
● 完成后，可以删除源索引


![## Shrink API](https://img-blog.csdnimg.cn/20210316102804715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
• 分片必须只读
• 所有的分片必须在同一个节点上
• 集群健康状态为 Green

```bash
# 在一个 hot-warm-cold的集群上进行测试
GET _cat/nodes
GET _cat/nodeattrs

DELETE my_source_index
DELETE my_target_index
PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0
 }
}

PUT my_source_index/_doc/1
{
  "key":"value"
}

GET _cat/shards/my_source_index

# 分片数3，会失败
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 3,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}



# 报错，因为没有置成 readonly
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}

#将 my_source_index 设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}

# 报错，必须都在一个节点
POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}

DELETE my_source_index
## 确保分片都在 hot
PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0,
   "index.routing.allocation.include.box_type":"hot"
 }
}

PUT my_source_index/_doc/1
{
  "key":"value"
}

GET _cat/shards/my_source_index

#设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}


POST my_source_index/_shrink/my_target_index
{
  "settings": {
    "index.number_of_replicas": 0,
    "index.number_of_shards": 2,
    "index.codec": "best_compression"
  },
  "aliases": {
    "my_search_indices": {}
  }
}


GET _cat/shards/my_target_index

# My target_index状态为也只读
PUT my_target_index/_doc/1
{
  "key":"value"
}
```

## 5. Split API
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210316103428309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210316103534528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
# Split Index
DELETE my_source_index
DELETE my_target_index

PUT my_source_index
{
 "settings": {
   "number_of_shards": 4,
   "number_of_replicas": 0
 }
}

PUT my_source_index/_doc/1
{
  "key":"value"
}

GET _cat/shards/my_source_index

# 必须是倍数
POST my_source_index/_split/my_target
{
  "settings": {
    "index.number_of_shards": 10
  }
}

# 必须是只读
POST my_source_index/_split/my_target
{
  "settings": {
    "index.number_of_shards": 8
  }
}


#设置为只读
PUT /my_source_index/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}


POST my_source_index/_split/my_target_index
{
  "settings": {
    "index.number_of_shards": 8,
    "index.number_of_replicas":0
  }
}

GET _cat/shards/my_target_index



# write block
PUT my_target_index/_doc/1
{
  "key":"value"
}
```

## 6. 一个时间序列索引的实际场景	
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210316103734159.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 7. Rollover API


● 当满足一系列的条件，Rollover API 支持将一个 Alias 指向一个新的索引
○ 存活的时间 / 最大文档数 / 最大的文件尺寸
● 应用场景
○ 当一个索引数据量过大
● 一般需要和 `Index Lifecycle Management Policies` 结合使用
○ 只有调用 Rollover API 时，才会去做相应的检测。ES 并不会自动去监控这些索引

```bash
#Rollover API
DELETE nginx-logs*
# 不设定 is_write_true
# 名字符合命名规范
PUT /nginx-logs-000001
{
  "aliases": {
    "nginx_logs_write": {}
  }
}

# 多次写入文档
POST nginx_logs_write/_doc
{
  "log":"something"
}


POST /nginx_logs_write/_rollover
{
  "conditions": {
    "max_age":   "1d",
    "max_docs":  5,
    "max_size":  "5gb"
  }
}

GET /nginx_logs_write/_count
# 查看 Alias信息
GET /nginx_logs_write


DELETE apache-logs*


# 设置 is_write_index
PUT apache-logs1
{
  "aliases": {
    "apache_logs": {
      "is_write_index":true
    }
  }
}
POST apache_logs/_count

POST apache_logs/_doc
{
  "key":"value"
}

# 需要指定 target 的名字
POST /apache_logs/_rollover/apache-logs8xxxx
{
  "conditions": {
    "max_age":   "1d",
    "max_docs":  1,
    "max_size":  "5gb"
  }
}


# 查看 Alias信息
GET /apache_logs
```

