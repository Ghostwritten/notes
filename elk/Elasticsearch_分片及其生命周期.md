

----
## 1. 分片的内部原理
什么是 ES 的分片

 - ES 中最小的工作单元 / 是一个 Lucence 的 Index

一些问题：

 - 为什么 ES 的搜索时近实时的（1 秒后被搜到）
 - ES 如何保证在断电时数据也不会丢失
 - 为甚删除文档，并不会立刻释放空间

## 2. 倒排索引的不可变性
倒排索引采用 `Immutable Design`, 一旦生成，不可更改
不可变性，带来了的好处如下：

 - 不许考虑并发写文件的问题，避免了锁机制带来的性能问题
 - 一旦读入内核的文件系统缓存，便留在那里，只要文件系统存有足够的空间，大部分请求就会直接请求内存，不会命中磁盘，提高了很大的性能
 - 缓存容易生成和维护 / 数据可以被压缩

不可变更性，带来了的挑战：如果需要让一个新的文档可以被搜索，需要重建整个索引
## 3. Lucence Index

 - 在 `Lucene` 中，单个倒排索引文件被称为 `Segment`。Segment 是自包含的，不可变更的。多个 Segment 汇总在一起，称为 Lucene 的 `Index`，其对应的就是 ES 中的 Shard
 - 当有新文档写入时，会生成新的 Segment, 查询时会同时查询所有的 Segment，并且对结果汇总。Luncene中有个文件，用来记录所有的 Segment 的信息，叫做 `Commit Point`
 - 删除的文档信息，保存在”.del” 文件中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303163125820.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. 什么是refresh

 - 将 `Index buffer` 写入 `Segment` 的过程叫做 `Refresh`。Refresh 不执行 `fsync` 操作
 - Refresh 频率：默认 1 秒发生一次，可通过 `index.refresj_interval` 配置。Refresh
   后，数据就可以被搜索到了。这也就是为什么 ES 被称为**近实时搜索**
 - 如果系统有大量的数据写入，那就会产生很多的 Segment
 - `Index Buffer` 被占满时，会触发 Refresh, 默认值是 JVM 的 10%
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303163347876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 5. 什么是Transaction Log
 - Segment 写入磁盘的过程相对耗时，借助文件系统缓存，Refresh 时，先将 `Segment` 写入`缓存`以开放查询
 - 为了保证数据不会丢失。所有在 Index 文档时，同时写 `Transaction Log`，高版本开始，ra 默认落盘。每个分片都有一个`Transaction Log`
 - 当 ES Refresh 时，Index Buffer 被清空，Transaction Log 不会清空

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303163728623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 6. 什么是flush
ES Flush & Lucene Commit

 - 调用 Refresh ，`Index Buffer` 清空并且 `Refresh`
 - 调用 `fsync`, 将缓存中的 `Segments` 写入磁盘
 - 清空（删除）Transaction Log
 - 默认 30 分钟调用一次
 - Transaction Log 满（默认 512M）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303163947668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 7. 什么是merge
Segment 很多，需要定期被合并
 - 减少 Segment/ 删除已经删除的文档

ES 和 Lucene 会自动进行 Merge 操作

 - POST my_index/_forcemerge

