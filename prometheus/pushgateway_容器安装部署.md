## 简介
Pushgateway 是 Prometheus 生态中一个重要工具，使用它的原因主要是：
Prometheus 采用 pull 模式，可能由于不在一个子网或者防火墙原因，导致 Prometheus 无法直接拉取各个 target 数据。
在监控业务数据的时候，需要将不同数据汇总, 由 Prometheus 统一收集。
由于以上原因，不得不使用 pushgateway，但在使用之前，有必要了解一下它的一些弊端：
将多个节点数据汇总到 pushgateway, 如果 pushgateway 挂了，受影响比多个 target 大。
Prometheus 拉取状态 up 只针对 pushgateway, 无法做到对每个节点有效。
Pushgateway 可以持久化推送给它的所有监控数据。
因此，即使你的监控已经下线，prometheus 还会拉取到旧的监控数据，需要手动清理 pushgateway 不要的数据

缺点：
1. pushgateway 会形成⼀个单点瓶颈，假如好多个 脚本同时 发送给 ⼀个pushgateway的进程  如果这个进程没了，那么监控数据也就没了
 2. pushgateway 并不能对发送过来的 脚本采集数据 进⾏更智 能的判断 假如脚本中间采集出问题了 那么有问题的数据 pushgateway⼀样照单全收 发送给 prometheus
