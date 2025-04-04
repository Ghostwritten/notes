

----
## 1. 单个分片

 - 7.0 开始，新创建一个索引时，默认只有一个主分片
 - 单个分片，查询算分，聚合不准的问题都可以得以避免
 - 单个索引，单个分片时候，集群无法实现水平扩展
 - 即使增加新的节点，无法实现水平扩展

## 2. 两个分片
集群增加一个节点后，Elasticsearch 会自动进行分片的移动，也叫 `Shard Rebalancing`
## 3. 如何设计分片数
当分片数 > 节点数时

 - 一旦集群中有新的数据节点加入，分片就可以自动进行分配
 - 分片在重新分配时，系统不会有 downtime

多分片的好处：一个索引如果分布在不同的节点，多个节点可以并行执行

 - 查询可以并行执行
 - 数据写入可以分散到多个机器

## 4. 一些例子
案例 1

 - 每天 1 GB 的数据，一个索引一个主分片，一个副本分片
 - 需保留半年的数据，接近 360 GB 的数据量

案例 2

 - 5 个不同的日志，每天创建一个日志索引。每个日志索引创建 10 个主分片
 - 保留半年的数据
 - 5 * 10 * 30 * 6 = 9000 个分片

## 5. 分片过多所带来的副作用
`Shard` 是 Elasticsearch 实现集群水平扩展的最小单位
过多设置分片数会带来一些潜在的问题

 - 每个分片是一个 Lucene 的 索引，会使用机器的资源。过多的分片会导致额外的性能开销
 - Lucene Indices / File descriptors / RAM / CPU
 - 每次搜索的请求，需要从每个分片上获取数据
 - 分片的 Meta 信息由 Master 节点维护。过多，会增加管理的负担。经验值，控制分片总数在 10 W 以内

## 6. 如何确定主分片数
从存储的物理角度看

 - 日志类应用，单个分片不要大于 50 GB
 - 搜索类应用，单个分片不要超过 20 GB

为什么要控制分片存储大小

 - 提高 Update 的性能
 - Merge 时，减少所需的资源
 - 丢失节点后，具备更快的恢复速度 / 便于分片在集群内 `Rebalancing`

## 7. 如何确定副本分片数
副本是主分片的拷贝

 - 提高系统可用性：相应查询请求，防止数据丢失
 - 需要占用和主分片一样的资源

对性能的影响

 - 副本会降低数据的索引速度：有几份副本就会有几倍的 CPU 资源消耗在索引上
 - 会减缓对主分片的查询压力，但是会消耗同样的内存资源
 - 如果机器资源充分，提高副本数，可以提高整体的查询 QPS

## 8. 调整分片总数设定，避免分配不均衡
ES 的分片策略会尽量保证节点上的分片数大 致相同
扩容的新节点没有数据，导致新索引集中在新 的节点
热点数据过于集中，可能会产生新能问题

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/817c0bb3cd670e5a8a335e76354dc58c.png)

