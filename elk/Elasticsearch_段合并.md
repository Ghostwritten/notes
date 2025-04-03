

---
## 1. 问题

 - 1、 segment是不是合并到一个最好，及max_num_segments=1
 - 2、合并的时候，通过

```bash
POST /my_index/_forcemerge?max_num_segments=1
```
 - 会不会吃光所有的机器资源，造成服务暂时不可用(`optimize?max_num_segments=1`就会吃光所有资源)，但是我没有从官方文档找到`_forcemerger`这种方式的资源消耗。
 - 3、在es 6.7及以上中index.merge 相关参数有需要特别注意和调整的地方吗？ （目前我全部使用的默认值）


## 2. 什么是段
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ba2053f4974f1c803d3b30a06a89cd1d.png)
如图所示，自顶向下看，

 - 一个集群包含1个或多个节点；
 - 一个节点包含1个或多个索引；
 - 一个索引：类似 Mysql 中的数据库；
 - 每个索引又由一个或多个分片组成；
 - 每个分片都是一个 Lucene 索引实例，您可以将其视作一个独立的搜索引擎，它能够对 Elasticsearch
   集群中的数据子集进行索引并处理相关查询；
 - 每个分片包含多个segment（段），每一个segment都是一个倒排索引。

在查询的时，会把所有的segment查询结果汇总归并为最终的分片查询结果返回。除了上面提到的”段(segment)”的概念，Lucene还增加了一个”提交点(commit point)”的概念，”提交点(commit point)”用于列出了所有已知的”段”。

## 3. 倒排索引有什么好处？
Elasticsearch可以对全文进行检索主要归功于倒排索引，倒排索引被写入磁盘后是不可改变的，永远不能被修改。倒排索引的不变性有几个好处：

 - 因为索引不能更新，不需要锁
 - 文件系统缓存亲和性，由于索引不会改变，只要系统内存足够，大部分读请求直接命中内存，可以极大提高性能
 - 其他缓存，如filter缓存，在索引的生命周期内始终有效
 - 写入单个大的倒排索引允许数据被压缩，减少磁盘I/O和需要被缓存到内存的索引的使用量

但倒排索引的不变性，同样意味着当需要新增文档时，需要对整个索引进行重建，当数据更新频繁时，这个问题将会变成灾难。那Elasticsearch索引近似实时性，是如何解决这个问题的呢？

## 4. 索引更新过程
索引的更新过程可以通过refresh api和flush API来说明。

refresh API
从内存索引缓冲区把数据写入新段(segment)中，并打开，可供检索，但这部分数据仍在缓存中，未写入磁盘。默认间隔是1s，这个时间会影响段的大小，对段的合并策略有影响，后面会分析。可以进行手动刷新：

```bash
# 刷新所有索引
POST /_refresh

# 指定刷新索引
POST /index_name/_refresh
flush API
```

执行一个提交并且截断translog的行为在Elasticsearch被称作一次flush。每30分钟或者translog太大时会进行flush，所以可以通过translog的设置来调节flush的行为。完成一次flush会有以下过程：

 - 所有在内存缓冲区的文档都被写入一个新的段。
 - 缓冲区被清空。
 - 一个提交点被写入硬盘。
 - 文件系统缓存通过fsync被刷新(flush)。
 - 老的translog被删除。

## 5. 为什么 段是不可变的？
在 lucene 中，为了实现高索引速度，故使用了segment 分段架构存储。

一批写入数据保存在一个段中，其中每个段是磁盘中的单个文件。

由于两次写入之间的文件操作非常繁重，因此将一个段设为不可变的，以便所有后续写入都转到New段。

## 6. 什么是段合并(segment merge)？

每次refresh都产生一个新段(segment)，频繁的refresh会导致段数量的暴增。段数量过多会导致过多的消耗文件句柄、内存和CPU时间，影响查询速度。基于这个原因，Lucene会通过合并段来解决这个问题。
由于自动刷新流程每秒会创建一个新的段（由动态配置参数：`refresh_interval` 决定），这样会导致短时间内的段数量暴增。

而段数目太多会带来较大的麻烦。

消耗资源：每一个段都会消耗文件句柄、内存和cpu运行周期；
搜索变慢：每个搜索请求都必须轮流检查每个段； 所以段越多，搜索也就越慢。
Elasticsearch 通过在后台进行段合并来解决这个问题。

小的段被合并到大的段，然后这些大的段再被合并到更大的段。


## 7. 段合并做了什么？
段合并的时候会将那些旧的已删除文档从文件系统中清除。

被删除的文档（或被更新文档的旧版本）不会被拷贝到新的大段中。

启动段合并不需要你做任何事。进行索引和搜索时会自动进行。

当索引的时候，刷新（refresh）操作会创建新的段并将段打开以供搜索使用。
合并进程选择一小部分大小相似的段，并且在后台将它们合并到更大的段中。这并不会中断索引和搜索。

## 8. 为什么要进行段合并？
索引段的个数越多，搜索性能越低并且消耗更多的内存。

索引段是不可变的，你并不能物理上从中删除信息。

可以物理上删除document，但只是做了删除标记，物理上并没有删除。

当段合并时，这些被标记为删除的文档并没有被拷贝至新的索引段中，这样，减少了最终的索引段中的 document 数目。

## 9. 段合并的好处是什么？
减少索引段的数量并提高检索速度；

减少索引的容量（文档数）

原因：段合并会移除被标记为已删除的那些文档。

## 10. 段合并可能带来的问题？
磁盘IO操作的代价；

速度慢的系统中，段合并会显著影响性能。

## 11. 关于合并段的大小（通常为1个）——针对问题1
早期版本的文档有说明如下：

optimize API（现在已废弃，原理一致）

optimize API大可看做是 强制合并 API。它会将一个分片强制合并到 max_num_segments 参数指定大小的段数目。

这样做的意图是减少段的数量（通常减少到一个），来提升搜索性能。


## 12. 关于段合并资源消耗——针对问题2
资源消耗的官方解读

> orce merge should only be called against an index after you have
> finished writing to it. Force merge can cause very large (>5GB)
> segments to be produced, and if you continue to write to such an index
> then the automatic merge policy will never consider these segments for
> future merges until they mostly consist of deleted documents. This can
> cause very large segments to remain in the index which can result in
> increased disk usage and worse search performance.

一句话：导致磁盘io消耗和影响检索性能。

Force merge API

[https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html)

以下是老版本文档解读，原理一致可参考，api已过时。

段合并

[https://www.elastic.co/guide/cn/elasticsearch/guide/current/merge-process.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/merge-process.html)

请注意，使用 optimize API 触发段合并的操作不会受到任何资源上的限制。

这可能会消耗掉你节点上全部的 I/O 资源, 使其没有“富裕”资源来处理搜索请求，从而有可能使集群失去响应。

如果你想要对索引执行 optimize，你需要先使用分片分配（查看 迁移旧索引）把索引移到一个安全的节点，再执行。

是的，非常耗费资源，建议在非业务密集实践操作。

我的线上环境，我都是凌晨1点段合并（脚本控制，晚上没人操作系统）


##  13. 可推荐的参数——针对问题3
减少段的产生频率，修改`refresh_inteval:`默认1s，如果时效性要求不高，建议改成30s。

`index.merge.scheduler.max_thread_count`：根据cpu核数修改

推荐：

[https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-merge.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-merge.html)

老版本参数修改参考价值不大，也建议看一下：

索引性能技巧

[https://www.elastic.co/guide/cn/elasticsearch/guide/current/indexing-performance.html#segments-and-merging](https://www.elastic.co/guide/cn/elasticsearch/guide/current/indexing-performance.html#segments-and-merging)


参考连接：
[https://my.oschina.net/elastic6/blog/4666668](https://my.oschina.net/elastic6/blog/4666668)
