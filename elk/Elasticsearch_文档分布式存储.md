


----
## 1. 文档储存在分片上

 文档会存储在具体的某个主分片和副本分片上：例如文档 1，会储存在 P0 R0 分片上 
 文档到分片的映射算法

 - 确保文档能均匀分布在所用分片上，充分利用硬件资源，避免部分机器空闲，部门机器繁忙

潜在的算法

 - 随机 / Round Robin. 当查询文档 1，分片数很多，需要多次查询才能查档文档 1
 - 维护文档到分片的映射关系，当文档数据量大的时候，维护成本高
 - 实时计算，通过文档 1，自动算出，需要去哪个分片上获取文档

## 2. 文档到分片的路由算法

```bash
shard = hash(_routing) % number_of_primary_shards
```

 - Hash 算法确保文档均匀分散到分片中
 - 默认的`_routing` 值是文档 id
 - 可以自行制定 routing 数值，例如用相同国家的商品，都分配到制定的 shard
 - **设置 Index Setting 后，Primary 数，不能随意修改的根本原因**

## 3. 更新文档
顺序： `index -> hash -> route -> delete -> index -> success -> response`

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303162300906.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. 删除一个文档
顺序 ：`detele -> hash&route -> delete -> delete replica -> success -> deleted -> response`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303162501532.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

