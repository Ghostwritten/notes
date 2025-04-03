![](https://i-blog.csdnimg.cn/blog_migrate/dd9456878c47313732f4aea386b300d0.jpeg)

## 1. 背景

当使用 Prometheus 监控多个 Kubernetes 集群时,如果没有合适的标签管理,alertmanager 在接收到警告时可能无法识别是哪个集群产生的警告。这可能会带来一些挑战:

- 警告上下文缺失: 当 alertmanager 接收到警告时,如果没有集群标识信息,很难确定警告来自哪个集群。这会降低故障排查和恢复的效率。
- 警告分类困难: 对于管理员来说,难以根据集群信息对警告进行分类和管理。这可能会导致警告混乱,影响及时响应。
- 跨集群视图缺失: 如果无法区分不同集群的警告,监控系统就很难提供一个全局的视图,难以了解整个基础设施的健康状况。

为了解决这个问题,可以考虑以下几点:

1. 在 Prometheus 配置中,为每个 Kubernetes 集群添加一个独特的标签,如 cluster: cluster-a。这样在警告中就可以包含集群信息。
2. 在 alertmanager 配置中,利用这些标签对警告进行路由和分组。例如根据 cluster 标签将警告划分到不同的接收器。
3. 在报警规则中,尽可能包含更多上下文信息,如节点名称、pod 名称等,以便 alertmanager 生成更丰富的警告内容。
4. 考虑使用 Grafana 等可视化工具,通过仪表盘展示跨集群的警告情况,帮助管理员快速定位问题根源。

通过这些措施,就可以确保 alertmanager 能够正确识别来自不同 Kubernetes 集群的警告,提升故障排查和整体监控的效率。


## 2. 添加静态元数据

```bash
vim prometheus-config.yaml
```

```bash
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['192.168.118.85:9100']
    relabel_configs:
      - source_labels: []
        regex: .*
        replacement: '192.168.118.20'
        action: replace
        target_label: cluster_vip
      - source_labels: []
        regex: .*
        replacement: 'production'
        action: replace
        target_label: cluster_type
      - source_labels: []
        regex: .*
        replacement: 'a-t-k8sv2'
        action: replace
        target_label: cluster_prefix
      - source_labels: []
        regex: .*
        replacement: '192.168.118.83'
        action: replace
        target_label: cluster_master01_ip
```

执行

```bash
kubectl apply -f prometheus-config.yaml
```

重载生效

```bash
curl -X POST http://192.168.118.83:30003/-/reload
```

> 注意：注意：当第二次追加metric_relabel_configs 参数，更新配置顺序，否则配置无法生效。





