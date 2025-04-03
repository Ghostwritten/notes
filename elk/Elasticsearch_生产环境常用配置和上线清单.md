


---
## 1. Development vs. Production Mode
从 ES 5 开始，支持 `Development` 和 `Production` 两种运行模式

 - 开发模式
 - 生产模式
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc33bb0ae9a2efd808e19b3d147dc009.png)
## 2. Bootstrap Checks
一个集群在 Production Mode 时，启动时必须通过所有 Bootstrap 检测，否则会启动失败
Bootstrap Checks 可以分为两类：`JVM` & `Linux Checks`。Linux Checks 只针对 Linux 系统
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe88bbaebad22b6e446f52a9e48d742a.png)
[https://www.elastic.co/guide/en/elasticsea...](https://www.elastic.co/guide/en/elasticsearch/reference/master/bootstrap-checks.html)
## 3. JVM 设定
从 ES 6 开始， 只支持 64 位 的 JVM
 - 配置 config /jvm.options

避免修改默认配置

 - Xmx 设置不要超过物理内存的 50%；单个节点上，最大内存建议不要超过 32 G 内存
 - [https://www.elastic.co/blog/a-heap-of-trou...](https://www.elastic.co/cn/blog/a-heap-of-trouble)

生产环境，JVM 必须使用 Server 模式
关闭 `JVM Swapping`
## 4. 集群的 API 设定
静态设置和动态设定

 - 静态配置文件尽量简洁：按照文档设置所有相 关系统参数。 `elasticsearch.yml` 配置文件 中尽量只写必备参数

其他的设置项可以通过 API 动态进行设定。 动态设定分 `transient` 和 `persistent` 两种， 都会覆盖 elasticsearch.yaml 中的设置

 - `Transient` 在集群重启后会丢失
 - `Persistent` 在集群中重启后不会丢失

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b09e1c07319b16b185967062ee813661.png)
## 5. 系统设置
参照文档 “Setup Elasticsearch> Important System Configuration”

 - [https://www.elastic.co/guide/en/elasticsea...](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/system-config.html)

`Disable Swapping` ， `Increase file descriptor`，虚拟内存，number of thread
## 6. 最佳实践：网络

 - 单个集群不要跨数据中心进行部署（不要使用 WAN）
 - 节点之间的 hops 越少越好
 - 如果有多块网卡，最好将 transport 和 http 绑定到不同的网卡，并设置不同的防火墙 Rules
 - 按需为 `Coordinating Node` 或 `Ingest Node` 配置负载均衡

## 7. 最佳实践：内存设定计算实例
内存大小要根据 Node 需要存储的数据来进行估算

 - 搜索类的比例建议： 1:16
 - 日志类： 1:48 - 1:96 之间

总数据量 1 T， 设置一个副本 = 2T 总数据量

 - 如果搜索类的项目，每个节点 31 *16 = 496 G，加上预留空间。所以每个节点最多 400 G 数据，至少需 要 5 个数据节点
 - 如果是日志类项目，每个节点 31*50 = 1550 GB，2 个数据节点 即可

## 8. 最佳实践：存储

 - 推荐使用 SSD，使用本地存储（Local Disk）。避免使用 SAN NFS / AWS / Azure filesystem
 - 可以在本地指定多个 “path.data”，以支持使用多块磁盘
 - ES 本身提供了很好的 HA 机制；无需使用 RAID 1/5/10
 - 可以在 Warm 节点上使用 Spinning Disk，但是需要关闭 Concurrent Merges

```bash
Index.merge.scheduler.max_thread_count: 1
```

 - Trim 你的 SSD
 - [https://www.elastic.co/blog/is-your-elasti...](https://www.elastic.co/cn/blog/is-your-elasticsearch-trimmed)

## 9. 最佳实践：服务器硬件

 - 建议使用中等配置的机器，不建议使用过于强劲的硬件配置 `Medium machine over large machine`
 - 不建议在一台服务器上运行多个节点

## 10. 集群设置：Throttles 限流
为 `Relocation` 和 `Recovery` 设置限流，避免过多任务对集群产生性能影响
Recovery

 - `Cluster.routing.allocation.node_concurrent_recoveries: 2`

Relocation

 - `Cluster.routing.allocation.cluster_concurrent_rebalance: 2`

## 11. 集群设置：关闭 Dynamic Indexes
可以考虑关闭动态索引创建的功能

```bash
PUT _cluster/settings
{
"persistent": {
  "action.auto_create_index":false
}
}
```

或者通过模版设置白名单

```bash
PUT _cluster/settings
{
"persistent": {
  "action.auto_create_index":"logstash-*,.kibana*"
}
}
```

## 12. 集群安全设定

 1. 打开 Authentication & Authorization
 2. 实现索引和和字段级的安全控制
 3. 节点间通信加密
 4. Enable HTTPS
 5. Audit logs

