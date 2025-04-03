

![](https://i-blog.csdnimg.cn/blog_migrate/28cfc3ba87eb58cbb949f4736684c062.png)


## 1. 简介

kube-prometheus-stack是一个基于Prometheus和Grafana的开源软件套件，用于在Kubernetes集群中进行监控和可视化。它提供了一套完整的工具和组件，用于收集、存储、查询和展示监控指标数据。

组件：
kube-prometheus-stack由多个关键组件组成，包括：

- Prometheus Operator：用于在Kubernetes上部署和管理Prometheus实例的控制器。
- Alertmanager：用于管理和处理Prometheus生成的告警通知。
- Prometheus：一个开源的监控系统，用于收集和存储时间序列数据。
- Grafana：一个功能强大的数据可视化和仪表板工具，用于展示监控指标和创建仪表板。
- kube-state-metrics：用于从Kubernetes API中导出集群状态指标的工具。
- node-exporter：用于收集主机级别的指标数据。
- Kubelet、kube-proxy和cAdvisor等其他组件，用于收集和导出Kubernetes集群的相关指标。

特性和功能：
kube-prometheus-stack具有以下特性和功能：

- 快速部署和配置Prometheus和Grafana，无需手动编写复杂的配置文件。
- 自动化的Prometheus实例管理和自动发现服务和Pod。
- 提供预定义的Prometheus规则和警报，用于监控常见的Kubernetes和系统指标。
- 支持水平扩展和高可用性，以满足大规模集群的监控需求。
- 提供丰富的数据可视化和仪表板功能，通过Grafana创建和共享自定义仪表板。
- 支持告警通知和集成到外部通知系统，如Slack、PagerDuty等。
- 提供强大的查询语言和灵活的数据处理能力，以便进行高级监控和分析。

总之，kube-prometheus-stack为Kubernetes集群提供了一个强大而灵活的监控解决方案，使您能够方便地监控、分析和可视化集群的健康状态和性能指标。
## 2. 简单部署

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```bash
helm upgrade prometheus prometheus-community/kube-prometheus-stack --version 55.0.0 --debug --namespace prometheus --create-namespace --install --timeout 600s --wait
```


 查看

```bash
$ kubectl get crd |grep -i monitor
alertmanagerconfigs.monitoring.coreos.com                  2023-03-01T07:02:27Z
alertmanagers.monitoring.coreos.com                        2023-03-01T07:02:27Z
podmonitors.monitoring.coreos.com                          2023-03-01T07:02:27Z
probes.monitoring.coreos.com                               2023-03-01T07:02:27Z
prometheusagents.monitoring.coreos.com                     2023-12-08T02:56:31Z
prometheuses.monitoring.coreos.com                         2023-12-08T02:56:31Z
prometheusrules.monitoring.coreos.com                      2023-12-08T02:56:31Z
scrapeconfigs.monitoring.coreos.com                        2023-12-08T02:13:50Z
servicemonitors.monitoring.coreos.com                      2023-03-01T07:02:27Z
thanosrulers.monitoring.coreos.com                         2023-03-01T07:02:27Z

```

## 3. 数据持久化部署
确保数据持久存储，通过 OpenEBS LocalPV 实现：[How to helm install OpenEBS LocalPV](https://blog.csdn.net/xixihahalelehehe/article/details/129967490)

### 3.1 设置必要的环境变量
- `PROM_STORAGECLASS_NAME`：指定Storageclass名称, 使用 kubectl get storageclasses 获取可用的 Storageclass 名称。

- `PROM_PVC_SIZE_G`：指定持久化卷的大小，单位为Gi。

- `PROM_NODE_NAMES`：指定安装Prometheus pod的节点名称，节点名称可以使用","作为分隔符，表示多个节点名称，安装程序会对节点进行label固定安装节点。

```bash
export PROM_STORAGECLASS_NAME="openebs-lvmsc-hdd"
export PROM_PVC_SIZE_G="20"
export PROM_NODE_NAMES="kube-node01,kube-node02,kube-node03"
```

### 3.2 运行安装脚本
> 注意⚠️：如果找不到 Helm3，将自动安装。
> 注意⚠️：安装脚本会对指定节点进行添加label的操作。


```bash
curl -sSL https://raw.githubusercontent.com/upmio/upm-deploy/main/addons/prometheus/install_el7.sh | sh -
```
等几分钟。 如果所有 prometheus pod 都在运行，则 prometheus 将成功安装。

### 3.3 查看

```bash
$ kubectl get pod -n prometheus
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   0          5m51s
prometheus-grafana-8df95c687-rdlng                       3/3     Running   0          6m32s
prometheus-kube-prometheus-operator-f8d88b6bb-jh4tg      1/1     Running   0          6m32s
prometheus-kube-state-metrics-84846764b9-smg97           1/1     Running   0          6m32s
prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0          5m50s
prometheus-prometheus-node-exporter-4lgvk                1/1     Running   0          6m32s
prometheus-prometheus-node-exporter-4z6n9                1/1     Running   0          6m32s
prometheus-prometheus-node-exporter-57s9z                1/1     Running   0          6m32s
prometheus-prometheus-node-exporter-gtfkf                1/1     Running   0          6m32s
prometheus-prometheus-node-exporter-txzxs                1/1     Running   0          6m32s
prometheus-prometheus-node-exporter-w47rm                1/1     Running   0          6m32s
prometheus-prometheus-node-exporter-x95dk                1/1     Running   0          6m32s


#查看所依赖的镜像
$ kubectl get pod -n prometheus -oyaml |grep 'image:' | sort | uniq
      image: docker.io/dbscale/kube-state-metrics:v2.9.2
      image: docker.io/grafana/grafana:10.2.2
      image: quay.io/kiwigrid/k8s-sidecar:1.25.2
      image: quay.io/prometheus-operator/prometheus-config-reloader:v0.70.0
      image: quay.io/prometheus-operator/prometheus-operator:v0.70.0
      image: quay.io/prometheus/alertmanager:v0.26.0
      image: quay.io/prometheus/node-exporter:v1.7.0
      image: quay.io/prometheus/prometheus:v2.48.1

#查看 svc
$ kubectl  get svc -n prometheus
NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   7m47s
prometheus-grafana                        ClusterIP   10.233.48.28    <none>        80/TCP                       8m27s
prometheus-kube-prometheus-alertmanager   ClusterIP   10.233.21.53    <none>        9093/TCP,8080/TCP            8m27s
prometheus-kube-prometheus-operator       ClusterIP   10.233.54.20    <none>        443/TCP                      8m27s
prometheus-kube-prometheus-prometheus     ClusterIP   10.233.20.146   <none>        9090/TCP,8080/TCP            8m27s
prometheus-kube-state-metrics             ClusterIP   10.233.30.192   <none>        8080/TCP                     8m27s
prometheus-operated                       ClusterIP   None            <none>        9090/TCP                     7m45s
prometheus-prometheus-node-exporter       ClusterIP   10.233.50.226   <none>        9100/TCP                     8m27s

#查询crd
$  kubectl get crd |grep monitor
alertmanagerconfigs.monitoring.coreos.com             2023-12-17T16:21:04Z
alertmanagers.monitoring.coreos.com                   2023-12-17T16:21:05Z
podmonitors.monitoring.coreos.com                     2023-12-17T16:21:05Z
probes.monitoring.coreos.com                          2023-12-17T16:21:05Z
prometheusagents.monitoring.coreos.com                2023-12-17T16:21:06Z
prometheuses.monitoring.coreos.com                    2023-12-17T16:21:06Z
prometheusrules.monitoring.coreos.com                 2023-12-17T16:21:07Z
scrapeconfigs.monitoring.coreos.com                   2023-12-17T16:21:07Z
servicemonitors.monitoring.coreos.com                 2023-12-17T16:21:07Z
thanosrulers.monitoring.coreos.com                    2023-12-17T16:21:07Z
```

查看存储是否自动分配卷

```bash
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                                                                                               STORAGECLASS        REASON   AGE
pvc-6a000349-b47d-47bf-a80f-7e9b825db4cb   20Gi       RWO            Delete           Bound       prometheus/prometheus-prometheus-kube-prometheus-prometheus-db-prometheus-prometheus-kube-prometheus-prometheus-0   openebs-lvmsc-hdd            16m
task-pv-volume                             10Gi       RWO            Retain           Available                                                                                                                       manual                       144d

$ kubectl get pvc -n prometheus
NAME                                                                                                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
prometheus-prometheus-kube-prometheus-prometheus-db-prometheus-prometheus-kube-prometheus-prometheus-0   Bound    pvc-6a000349-b47d-47bf-a80f-7e9b825db4cb   20Gi       RWO            openebs-lvmsc-hdd   16m
```

参考：

- [https://prometheus.io/](https://prometheus.io/)
- [https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [Kind & Kubernetes | 通过 Helm 部署定制化 Prometheus-Operator 上传 Dockerhub？](https://blog.csdn.net/xixihahalelehehe/article/details/129820019)
- [https://docs.youdianzhishi.com/prometheus/](https://docs.youdianzhishi.com/prometheus/)
- [promethues 云原生手册](https://ghostwritten.blog.csdn.net/article/details/113100748)
