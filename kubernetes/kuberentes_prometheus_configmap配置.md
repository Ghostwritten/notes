


```bash
apiVersion: v1
data:
  monitoring-prometheus-k8s-rules.yaml: |
    groups:
    - name: node-exporter.rules
      rules:
      - expr: |
          count without (cpu) (
            count without (mode) (
              node_cpu_seconds_total{cluster="mycluster"}
            )
          )
        record: instance:node_num_cpu:sum
      - expr: |
          1 - avg without (cpu, mode) (
            rate(node_cpu_seconds_total{cluster="mycluster",mode="idle"}[1m])
          )
        record: instance:node_cpu_utilisation:rate1m
      - expr: |
          (
            node_load1{cluster="mycluster"}
          /
            instance:node_num_cpu:sum{cluster="mycluster"}
          )
        record: instance:node_load1_per_cpu:ratio
      - expr: |
          1 - (
            node_memory_MemAvailable_bytes{cluster="mycluster"}
          /
            node_memory_MemTotal_bytes{cluster="mycluster"}
          )
        record: instance:node_memory_utilisation:ratio
      - expr: |
          (
            rate(node_vmstat_pgpgin{cluster="mycluster"}[1m])
          +
            rate(node_vmstat_pgpgout{cluster="mycluster"}[1m])
          )
        record: instance:node_memory_swap_io_pages:rate1m
      - expr: |
          rate(node_disk_io_time_seconds_total{cluster="mycluster",device=~"nvme.+|rbd.+|sd.+|vd.+|xvd.+|dm-.+"}[1m])
        record: instance_device:node_disk_io_time_seconds:rate1m
      - expr: |
          rate(node_disk_io_time_weighted_seconds_total{cluster="mycluster",device=~"nvme.+|rbd.+|sd.+|vd.+|xvd.+|dm-.+"}[1m])
        record: instance_device:node_disk_io_time_weighted_seconds:rate1m
      - expr: |
          sum without (device) (
            rate(node_network_receive_bytes_total{cluster="mycluster",device!="lo"}[1m])
          )
        record: instance:node_network_receive_bytes_excluding_lo:rate1m
      - expr: |
          sum without (device) (
            rate(node_network_transmit_bytes_total{cluster="mycluster",device!="lo"}[1m])
          )
        record: instance:node_network_transmit_bytes_excluding_lo:rate1m
      - expr: |
          sum without (device) (
            rate(node_network_receive_drop_total{cluster="mycluster",device!="lo"}[1m])
          )
        record: instance:node_network_receive_drop_excluding_lo:rate1m
      - expr: |
          sum without (device) (
            rate(node_network_transmit_drop_total{cluster="mycluster",device!="lo"}[1m])
          )
        record: instance:node_network_transmit_drop_excluding_lo:rate1m
    - name: k8s.rules
      rules:
      - expr: |
          sum(rate(container_cpu_usage_seconds_total{cluster="mycluster",job="kubelet", image!="", container!="POD"}[5m])) by (namespace,cluster)
        record: namespace:container_cpu_usage_seconds_total:sum_rate
      - expr: |
          sum by (namespace, pod, container,cluster) (
            rate(container_cpu_usage_seconds_total{cluster="mycluster",job="kubelet", image!="", container!="POD"}[5m])
          )
        record: namespace_pod_container:container_cpu_usage_seconds_total:sum_rate
      - expr: |
          sum(container_memory_usage_bytes{cluster="mycluster",job="kubelet", image!="", container!="POD"}) by (namespace,cluster)
        record: namespace:container_memory_usage_bytes:sum
      - expr: |
          sum by (namespace, label_name,cluster) (
              sum(kube_pod_container_resource_requests_memory_bytes{cluster="mycluster",job="kube-state-metrics"} * on (cluster, endpoint, instance, job, namespace, pod, service) group_left(phase) (kube_pod_status_phase{cluster="mycluster",phase=~"^(Pending|Running)$"} == 1)) by (cluster, namespace, pod)
            * on (cluster, namespace, pod)
              group_left(label_name) kube_pod_labels{cluster="mycluster", job="kube-state-metrics"}
          )
        record: namespace:kube_pod_container_resource_requests_memory_bytes:sum
      - expr: |
          sum by (namespace, label_name,cluster) (
              sum(kube_pod_container_resource_requests_cpu_cores{cluster="mycluster", job="kube-state-metrics"} * on (cluster, endpoint, instance, job, namespace, pod, service) group_left(phase) (kube_pod_status_phase{cluster="mycluster",phase=~"^(Pending|Running)$"} == 1)) by (cluster, namespace, pod)
            * on (namespace, pod)
              group_left(label_name) kube_pod_labels{cluster="mycluster", job="kube-state-metrics"}
          )
        record: namespace:kube_pod_container_resource_requests_cpu_cores:sum
      - expr: |
          sum(
            label_replace(
              label_replace(
                kube_pod_owner{cluster="mycluster", job="kube-state-metrics", owner_kind="ReplicaSet"},
                "replicaset", "$1", "owner_name", "(.*)"
              ) * on(cluster, replicaset, namespace) group_left(owner_name) kube_replicaset_owner{cluster="mycluster", job="kube-state-metrics"},
              "workload", "$1", "owner_name", "(.*)"
            )
          ) by (cluster, namespace, workload, pod,cluster)
        labels:
          workload_type: deployment
        record: mixin_pod_workload
      - expr: |
          sum(
            label_replace(
              kube_pod_owner{cluster="mycluster", job="kube-state-metrics", owner_kind="DaemonSet"},
              "workload", "$1", "owner_name", "(.*)"
            )
          ) by (cluster, namespace, workload, pod,cluster)
        labels:
          workload_type: daemonset
        record: mixin_pod_workload
      - expr: |
          sum(
            label_replace(
              kube_pod_owner{cluster="mycluster", job="kube-state-metrics", owner_kind="StatefulSet"},
              "workload", "$1", "owner_name", "(.*)"
            )
          ) by (cluster, namespace, workload, pod,cluster)
        labels:
          workload_type: statefulset
        record: mixin_pod_workload
    - name: kube-scheduler.rules
      rules:
      - expr: |
          histogram_quantile(0.99, sum(rate(scheduler_e2e_scheduling_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.99"
        record: cluster_quantile:scheduler_e2e_scheduling_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.99, sum(rate(scheduler_scheduling_algorithm_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.99"
        record: cluster_quantile:scheduler_scheduling_algorithm_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.99, sum(rate(scheduler_binding_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.99"
        record: cluster_quantile:scheduler_binding_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.9, sum(rate(scheduler_e2e_scheduling_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.9"
        record: cluster_quantile:scheduler_e2e_scheduling_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.9, sum(rate(scheduler_scheduling_algorithm_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.9"
        record: cluster_quantile:scheduler_scheduling_algorithm_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.9, sum(rate(scheduler_binding_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.9"
        record: cluster_quantile:scheduler_binding_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.5, sum(rate(scheduler_e2e_scheduling_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.5"
        record: cluster_quantile:scheduler_e2e_scheduling_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.5, sum(rate(scheduler_scheduling_algorithm_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.5"
        record: cluster_quantile:scheduler_scheduling_algorithm_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.5, sum(rate(scheduler_binding_duration_seconds_bucket{cluster="mycluster", job="kube-scheduler"}[5m])) without(instance, pod))
        labels:
          quantile: "0.5"
        record: cluster_quantile:scheduler_binding_duration_seconds:histogram_quantile
    - name: kube-apiserver.rules
      rules:
      - expr: |
          histogram_quantile(0.99, sum(rate(apiserver_request_duration_seconds_bucket{cluster="mycluster", job="apiserver"}[5m])) without(instance, pod))
        labels:
          quantile: "0.99"
        record: cluster_quantile:apiserver_request_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.9, sum(rate(apiserver_request_duration_seconds_bucket{cluster="mycluster", job="apiserver"}[5m])) without(instance, pod))
        labels:
          quantile: "0.9"
        record: cluster_quantile:apiserver_request_duration_seconds:histogram_quantile
      - expr: |
          histogram_quantile(0.5, sum(rate(apiserver_request_duration_seconds_bucket{cluster="mycluster", job="apiserver"}[5m])) without(instance, pod))
        labels:
          quantile: "0.5"
        record: cluster_quantile:apiserver_request_duration_seconds:histogram_quantile
    - name: node.rules
      rules:
      - expr: sum(min(kube_pod_info{cluster="mycluster"}) by (node))
        record: ':kube_pod_info_node_count:'
      - expr: |
          max(label_replace(kube_pod_info{cluster="mycluster", job="kube-state-metrics"}, "pod", "$1", "pod", "(.*)")) by (cluster, node, namespace, pod)
        record: 'node_namespace_pod:kube_pod_info:'
      - expr: |
          count by (node) (sum by (node, cpu) (
            node_cpu_seconds_total{cluster="mycluster"}
          * on (namespace, pod) group_left(node)
            node_namespace_pod:kube_pod_info:{cluster="mycluster"}
          ))
        record: node:node_num_cpu:sum
      - expr: |
          sum(node_memory_MemFree_bytes{cluster="mycluster"} + node_memory_Cached_bytes{cluster="mycluster"} + node_memory_Buffers_bytes{cluster="mycluster"})
        record: :node_memory_MemFreeCachedBuffers_bytes:sum
    - name: kube-prometheus-node-recording.rules
      rules:
      - expr: sum(rate(node_cpu_seconds_total{cluster="mycluster", mode!="idle",mode!="iowait"}[3m]))
          BY (instance,cluster)
        record: instance:node_cpu:rate:sum
      - expr: sum((node_filesystem_size_bytes{cluster="mycluster", mountpoint="/"} - node_filesystem_free_bytes{cluster="mycluster",
          mountpoint="/"})) BY (instance,cluster)
        record: instance:node_filesystem_usage:sum
      - expr: sum(rate(node_network_receive_bytes_total{cluster="mycluster"}[3m])) BY
          (instance,cluster)
        record: instance:node_network_receive_bytes:rate:sum
      - expr: sum(rate(node_network_transmit_bytes_total{cluster="mycluster"}[3m])) BY
          (instance,cluster)
        record: instance:node_network_transmit_bytes:rate:sum
      - expr: sum(rate(node_cpu_seconds_total{cluster="mycluster", mode!="idle",mode!="iowait"}[5m]))
          WITHOUT (cpu, mode) / ON(instance) GROUP_LEFT() count(sum(node_cpu_seconds_total{cluster="mycluster"})
          BY (instance, cpu)) BY (instance,cluster)
        record: instance:node_cpu:ratio
      - expr: sum(rate(node_cpu_seconds_total{cluster="mycluster", mode!="idle",mode!="iowait"}[5m]))
        record: cluster:node_cpu:sum_rate5m
    - name: node-exporter
      rules:
      - alert: 节点文件系统使用率超过阈值
        annotations:
          message: xxxx平台{{ $labels.instance }}节点文件系统{{ $labels.mountpoint
            }}使用率超过阈值85%， 目前可用空间为{{ printf "%.2f" $value }}%
          summary: Filesystem has less than 15% space left.
        expr: |
          (
            node_filesystem_avail_bytes{cluster="mycluster"} / node_filesystem_size_bytes{cluster="mycluster"} * 100 < 15
          and
            node_filesystem_readonly == 0
          )
        labels:
          severity: error
      - alert: 文件系统只读状态
        annotations:
          message: xxxxx平台{{ $labels.instance }}节点文件系统{{ $labels.mountpoint
            }}状态为只读
          summary: Filesystem has been ReadOnly status.
        expr: node_filesystem_readonly{cluster="mycluster"} == 1
        for: 1m
        labels:
          severity: error
      - alert: 节点inode使用率超过阈值
        annotations:
          message: xxxxx平台{{ $labels.instance }}节点文件系统{{ $labels.mountpoint
            }} inode使用率超过阈值70%， 目前可用inode空间为{{ printf "%.2f" $value }}%
          summary: Filesystem has less than 30% inodes left.
        expr: |
          (
            node_filesystem_files_free{cluster="mycluster"} / node_filesystem_files{cluster="mycluster"} * 100 < 30
          and
            node_filesystem_readonly{cluster="mycluster"} == 0
          )
        labels:
          severity: warning
      - alert: 节点网卡接收错误率超过阈值
        annotations:
          message: xxxxx平台{{ $labels.instance }}节点网卡{{ $labels.device }}接收错误率在2分钟内超过10次
          summary: Network interface is reporting many receive errors.
        expr: |
          increase(node_network_receive_errs_total{cluster="mycluster"}[2m]) > 10
        for: 10m
        labels:
          severity: error
      - alert: 节点网卡发送错误率超过阈值
        annotations:
          message: xxxxx平台{{ $labels.instance }}节点网卡{{ $labels.device }}发送错误率在2分钟内超过10次
          summary: Network interface is reporting many transmit errors.
        expr: |
          increase(node_network_transmit_errs_total{cluster="mycluster"}[2m]) > 10
        for: 10m
        labels:
          severity: error
      - alert: 节点CPU使用率高
        annotations:
          message: xxxxx平台节点{{ $labels.instance }} cpu {{ $labels.cpu }} 使用率为{{
            printf "%.2f" $value }}% 超过阈值 90%
          summary: Node cpu utilization is over threshold
        expr: |
          100*(1 - rate(node_cpu_seconds_total{cluster="mycluster",mode="idle"}[5m]))> 90
        for: 10m
        labels:
          severity: warning
      - alert: 节点CPU IOwait高
        annotations:
          message: xxxxxx平台节点{{ $labels.instance }} cpu {{ $labels.cpu }} IOwait为
            {{ printf "%.2f" $value }}% 超过阈值 90%
          summary: Node cpu iowait is over threshold
        expr: |
          100*(rate(node_cpu_seconds_total{cluster="mycluster",mode="iowait"}[5m])) > 90
        for: 10m
        labels:
          severity: warning
      - alert: 节点内存使用率高
        annotations:
          message: xxxxx平台节点{{ $labels.instance }} 内存使用率为 {{ printf "%.2f" $value
            }}% 超过阈值 90% '
          summary: Node memory utilization is over threshold
        expr: |
          100*((node_memory_MemTotal_bytes{cluster="mycluster"}-node_memory_MemFree_bytes{cluster="mycluster"}-node_memory_Buffers_bytes{cluster="mycluster"}-node_memory_Cached_bytes{cluster="mycluster"})/node_memory_MemTotal_bytes{cluster="mycluster"}) > 90
        for: 5m
        labels:
          severity: warning
      - alert: 物理机被重启
        annotations:
          message: xxxxxxxxxxxx平台节点{{ $labels.instance }} 物理服务器重启 '
          summary: Node reboot
        expr: |
          time()-node_boot_time_seconds{cluster="mycluster"} < 1800
        labels:
          severity: error
      - alert: 物理节点夯机或宕机
        annotations:
          message: xxxxxxxxxx平台节点{{ $labels.node }} 物理服务器夯机或宕机 '
          summary: Node down
        expr: |
          label_replace(sum(up{job="node-exporter"})by(instance),"node","$1","instance","(.*)")+ on(node) sum(up{job="kubelet"})by(node)==0
        labels:
          severity: critical
    - name: kubernetes-absent
      rules:
      - alert: Alertmanager状态异常
        annotations:
          message: xxxxx平台Alertmanager {{ $labels.pod }}状态异常
        expr: |
          up{cluster="mycluster",job="alertmanager-main",namespace="monitoring"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: CoreDNS状态异常
        annotations:
          message: xxxxxx平台CoreDNS {{ $labels.pod }}状态异常
        expr: |
          up{cluster="mycluster",job="kube-dns"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: Apiserver状态异常
        annotations:
          message: xxxxx平台KubeAPIserver {{ $labels.instance }} 状态异常
        expr: |
          up{cluster="mycluster",job="apiserver"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: ControllerManager状态异常
        annotations:
          message: xxxxxx平台KubeControllerManager {{ $labels.instance }} 状态异常
        expr: |
          up{cluster="mycluster",job="kube-controller-manager"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: Scheduler状态异常
        annotations:
          message: xxxxxxx平台KubeScheduler {{ $labels.instance }} 状态异常
        expr: |
          up{cluster="mycluster",job="kube-scheduler"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: KubeStateMetric状态异常
        annotations:
          message: xxxxx平台KubeStateMetrics {{ $labels.pod }} 状态异常
        expr: |
          up{cluster="mycluster",job="kube-state-metrics"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: Kubelet状态异常
        annotations:
          message: xxxxx平台Kubelet {{ $labels.node }} 状态异常
        expr: |
          up{cluster="mycluster",job="kubelet"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: Etcd状态异常
        annotations:
          message: xxxxx平台Etcd {{ $labels.instance }} 状态异常
        expr: |
          up{cluster="mycluster",job="etcd"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: NodeExporter状态异常
        annotations:
          message: xxxxxx平台NodeExporter {{ $labels.instance }} 状态异常
        expr: |
          up{cluster="mycluster",job=~"node-exporter.*"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: Prometheus状态异常
        annotations:
          message: xxxxx平台Prometheus {{ $labels.pod }} 状态异常
        expr: |
          up{cluster="mycluster",job="prometheus-k8s",namespace="monitoring"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: PrometheusOperator状态异常
        annotations:
          message: xxxxx平台PrometheusOperator {{ $labels.pod }} 状态异常
        expr: |
          up{cluster="mycluster",job="prometheus-operator",namespace="monitoring"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: mytest前端Cayenne状态异常
        annotations:
          message: xxxxx平台Mizar后端Cayenne {{ $labels.cayenneurl }} 状态异常
        expr: |
          cayenne_status==0
        for: 2m
        labels:
          severity: critical
      - alert: Harbor状态异常
        annotations:
          message: xxxxxx平台Harbor {{ $labels.harborurl }} 状态异常
        expr: |
          harbor_status==0
        labels:
          severity: critical
      - alert: infra节点haproxy状态异常
        annotations:
          message: xxxxxxx平台 infra节点 {{ $labels.instance }} haproxy-mycluster
            状态异常
        expr: |
          node_systemd_unit_state{name="haproxy.service",state="active"}==0
        labels:
          severity: critical
      - alert: infra节点docker状态异常
        annotations:
          message: xxxxxx平台 infra节点 {{ $labels.instance }} docker 状态异常
        expr: |
          node_systemd_unit_state{name="docker.service",state="active"}==0
        labels:
          severity: critical
      - alert: infra节点keepalived状态异常
        annotations:
          message: xxxxxxxxxxx平台 infra节点 {{ $labels.instance }} keepalived 状态异常
        expr: |
          node_systemd_unit_state{name="keepalived.service",state="active"}==0
        labels:
          severity: critical
    - name: kubernetes-apps
      rules:
      - alert: POD中容器重启
        annotations:
          message: xxxx平台Pod {{ $labels.namespace }}/{{ $labels.pod }} ({{
            $labels.container }}) 正在重启 {{ printf "%.2f" $value }} 次 / 5 minutes.
        expr: |
          (rate(kube_pod_container_status_restarts_total{cluster="mycluster",job="kube-state-metrics"}[15m]) * 60 * 5 > 0) and on(namespace,pod) (time()-kube_pod_start_time{cluster="mycluster"}>300)
        for: 5m
        labels:
          severity: error
      - alert: POD状态异常
        annotations:
          message: xxxxxx平台Pod {{ $labels.namespace }}/{{ $labels.pod }} 状态异常
        expr: |
          sum by (namespace, pod) (kube_pod_status_phase{cluster="mycluster",job="kube-state-metrics", phase=~"Failed|Pending|Unknown"}) > 0
        for: 2m
        labels:
          severity: error
      - alert: 容器状态异常
        annotations:
          message: xxxxxxxxxx平台容器 {{ $labels.namespace }}/{{ $labels.pod }}/{{ $labels.container
            }} 状态异常，原因是{{ $labels.reason }}
        expr: |
          kube_pod_container_status_waiting_reason{cluster="mycluster"}==1 or kube_pod_container_status_terminated_reason{cluster="mycluster",reason!="Completed"}==1
        for: 2m
        labels:
          severity: error
      - alert: POD中有容器非Ready状态
        annotations:
          message: xxxxxxxx平台容器 {{ $labels.namespace }}/{{ $labels.pod }}/{{ $labels.container
            }} 为非Ready状态
        expr: |
          kube_pod_container_status_ready{cluster="mycluster",pod!~"mysqlmanagerbackup.+|etcdbackup.+"} == 0
        for: 5m
        labels:
          severity: error
      - alert: Deployment下有未成功启动的容器
        annotations:
          message: xxxxxxxxxx平台Deployment {{ $labels.namespace }}/{{ $labels.deployment
            }} 有未成功启动的容器在5分钟内
        expr: |
          kube_deployment_spec_replicas{cluster="mycluster",job="kube-state-metrics"}
            !=
          kube_deployment_status_replicas_available{cluster="mycluster",job="kube-state-metrics"}
        for: 5m
        labels:
          severity: error
      - alert: StatefulSet下有未成功启动的容器
        annotations:
          message: xxxxxxx平台StatefulSet {{ $labels.namespace }}/{{ $labels.statefulset
            }} 有未成功启动的容器在5分钟内
        expr: |
          kube_statefulset_status_replicas_ready{cluster="mycluster",job="kube-state-metrics"}
            !=
          kube_statefulset_status_replicas{cluster="mycluster",job="kube-state-metrics"}
        for: 5m
        labels:
          severity: error
      - alert: DaemonSet下有未成功启动的容器
        annotations:
          message: xxxxxxx平台DaemonSet {{ $labels.namespace }}/{{ $labels.daemonset
            }} 有未成功启动的容器在5分钟内
        expr: |
          kube_daemonset_status_number_ready{cluster="mycluster",job="kube-state-metrics"}
            /
          kube_daemonset_status_desired_number_scheduled{cluster="mycluster",job="kube-state-metrics"} * 100 < 100
        for: 5m
        labels:
          severity: error
      - alert: CronJob运行超过1小时未结束
        annotations:
          message: xxxxxx平台CronJob {{ $labels.namespace }}/{{ $labels.cronjob
            }}运行超过1小时未结束
        expr: |
          time() - kube_cronjob_next_schedule_time{cluster="mycluster",job="kube-state-metrics"} > 3600
        for: 1h
        labels:
          severity: warning
      - alert: Job非正常结束
        annotations:
          message: xxxxx平台Job {{ $labels.namespace }}/{{ $labels.job_name }}
            非正常结束
        expr: |
          kube_job_status_failed{cluster="mycluster",job="kube-state-metrics"}  > 0
        for: 2m
        labels:
          severity: error
    - name: kubernetes-resources
      rules:
      - alert: 集群已分配CPU过量
        annotations:
          message: xxxxxxxxx平台集群已分配CPU过量
        expr: |
          sum(namespace:kube_pod_container_resource_requests_cpu_cores:sum{cluster="mycluster"})
            /
          sum(kube_node_status_allocatable_cpu_cores{cluster="mycluster"})
            >
          (count(kube_node_status_allocatable_cpu_cores{cluster="mycluster"})-1) / count(kube_node_status_allocatable_cpu_cores{cluster="mycluster"})
        for: 5m
        labels:
          severity: warning
      - alert: 集群已分配Memory过量
        annotations:
          message: xxxxxxx平台集群已分配Memory过量
        expr: |
          sum(namespace:kube_pod_container_resource_requests_memory_bytes:sum{cluster="mycluster"})
            /
          sum(kube_node_status_allocatable_memory_bytes{cluster="mycluster"})
            >
          (count(kube_node_status_allocatable_memory_bytes{cluster="mycluster"})-1)
            /
          count(kube_node_status_allocatable_memory_bytes{cluster="mycluster"})
        for: 5m
        labels:
          severity: warning
      - alert: namespace中已分配资源将达到上限
        annotations:
          message: xxxxxx平台Namespace {{ $labels.namespace }} 已使用 {{ printf "%0.0f"
            $value }}% 的{ $labels.resource }}总量上限.
        expr: |
          100 * kube_resourcequota{cluster="mycluster",job="kube-state-metrics", type="used"}
            / ignoring(instance, job, type)
          (kube_resourcequota{cluster="mycluster",job="kube-state-metrics", type="hard"} > 0)
            > 90
        for: 5m
        labels:
          severity: warning
      - alert: 容器CPU使用率高
        annotations:
          message: xxxxx平台{{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container
            }}) cpu 使用率为{{ printf "%0.0f" $value }}% 超过阈值 90%
        expr: (namespace_pod_container:container_cpu_usage_seconds_total:sum_rate{cluster="mycluster"}
          / on(namespace,pod,container) kube_pod_container_resource_limits_cpu_cores{cluster="mycluster"})*100
          > 90
        for: 5m
        labels:
          severity: warning
      - alert: 容器内存使用率高
        annotations:
          message: xxxxxx平台{{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container
            }}) 内存使用率为 {{ printf "%0.0f" $value }}% 超过阈值 90%
        expr: 100*((container_memory_usage_bytes{cluster="mycluster"}-container_memory_cache{cluster="mycluster"})/(container_spec_memory_limit_bytes{cluster="mycluster"}
          != 0))> 90
        for: 5m
        labels:
          severity: warning
      - alert: 容器CACHE内存使用率超过阈值
        annotations:
          message: xxxxxx平台{{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container
            }}) Cache内存使用率为 {{ printf "%0.0f" $value }}% 超过阈值 90%
        expr: 100 * ( container_memory_cache{cluster="mycluster"} / (container_spec_memory_limit_bytes{cluster="mycluster"}!=0))
          > 90
        for: 5m
        labels:
          severity: warning
      - alert: 容器根目录文件系统使用率高
        annotations:
          message: xxxxxx平台{{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container
            }}) {{ $labels.device }} 使用率为 {{ printf "%0.0f" $value }}% 超过阈值 90%
        expr: 100*(container_fs_usage_bytes{cluster="mycluster",container!="POD",pod=~".+",device="/dev/mapper/vgdata-lvdocker"}/(30*1024*1024*1024))
          > 90
        labels:
          severity: error
    - name: kubernetes-storage
      rules:
      - alert: PV使用率超过阈值
        annotations:
          message: xxxxx平台namespace{{ $labels.namespace }}下PVC{{ $labels.persistentvolumeclaim
            }} 超过阈值85%，目前剩余{{ printf "%0.2f" $value}}%
        expr: |
          100 * kubelet_volume_stats_available_bytes{cluster="mycluster",job="kubelet"}
            /
          kubelet_volume_stats_capacity_bytes{cluster="mycluster",job="kubelet"}
            < 15
        for: 1m
        labels:
          severity: critical
      - alert: PV将在四天内耗尽
        annotations:
          message: Based on recent sampling, the PersistentVolume claimed by {{ $labels.persistentvolumeclaim
            }} in Namespace {{ $labels.namespace }} is expected to fill up within four
            days. Currently {{ printf "%0.2f" $value }}% is available.
        expr: |
          100 * (
            kubelet_volume_stats_available_bytes{cluster="mycluster",job="kubelet"}
              /
            kubelet_volume_stats_capacity_bytes{cluster="mycluster",job="kubelet"}
          ) < 15
          and
          predict_linear(kubelet_volume_stats_available_bytes{cluster="mycluster",job="kubelet"}[6h], 4 * 24 * 3600) < 0
        for: 5m
        labels:
          severity: warning
      - alert: PV fast-disk-large 数量将耗尽
        annotations:
          message: PV fast-disk-large is being used up. Now is {{ $value }} remained.
        expr: "(count(kube_persistentvolume_capacity_bytes==106300440576)\n-\n(count((kube_persistentvolume_status_phase{phase=\"Bound\"}==1)
          \n  and ignoring(phase) \n(kube_persistentvolume_capacity_bytes==106300440576))))
          < 5\n"
        labels:
          severity: warning
      - alert: PV fast-disk-big 数量将耗尽
        annotations:
          message: PV fast-disk-big is being used up. Now is {{ $value }} remained.
        expr: "(count(kube_persistentvolume_capacity_bytes==52613349376)\n-\n(count((kube_persistentvolume_status_phase{phase=\"Bound\"}==1)
          \n  and ignoring(phase) \n(kube_persistentvolume_capacity_bytes==52613349376))))
          < 5\n"
        labels:
          severity: warning
      - alert: PV fast-disk-medium 数量将耗尽
        annotations:
          message: PV fast-disk-medium is being used up. Now is {{ $value }} remained.
        expr: "(count(kube_persistentvolume_capacity_bytes==20401094656)\n-\n(count((kube_persistentvolume_status_phase{phase=\"Bound\"}==1)
          \n  and ignoring(phase) \n(kube_persistentvolume_capacity_bytes==20401094656))))
          < 5\n"
        labels:
          severity: warning
      - alert: PV fast-disk-small 数量将耗尽
        annotations:
          message: PV fast-disk-small is being used up. Now is {{ $value }} remained.
        expr: "(count(kube_persistentvolume_capacity_bytes==10726932480)\n-\n(count((kube_persistentvolume_status_phase{phase=\"Bound\"}==1)
          \n  and ignoring(phase) \n(kube_persistentvolume_capacity_bytes==10726932480))))
          < 5\n"
        labels:
          severity: warning
      - alert: PV状态异常
        annotations:
          message: xxxxx平台PV {{ $labels.persistentvolume }} 状态为 {{ $labels.phase
            }}
        expr: |
          kube_persistentvolume_status_phase{cluster="mycluster",phase=~"Failed|Pending",job="kube-state-metrics"} > 0
        for: 5m
        labels:
          severity: error
    - name: kubernetes-system
      rules:
      - alert: 节点状态异常
        annotations:
          message: xxxxx平台节点{{ $labels.node }} 状态异常
        expr: |
          kube_node_status_condition{cluster="mycluster",job="kube-state-metrics",condition="Ready",status="true"} == 0
        for: 2m
        labels:
          severity: critical
      - alert: KubeAPIClient请求错误率超过阈值
        annotations:
          message: xxxxxxxxxxx平台节点Kubernetes API server client {{ $labels.job }}/{{
            $labels.instance }} 请求错误率为{{ printf "%0.0f" $value }}%
        expr: |
          (sum(rate(rest_client_requests_total{cluster="mycluster",code=~"5.."}[5m])) by (instance, job)
            /
          sum(rate(rest_client_requests_total{cluster="mycluster"}[5m])) by (instance, job))
          * 100 > 1
        for: 5m
        labels:
          severity: critical
      - alert: 节点运行容器数量超过阈值
        annotations:
          message: xxxxxxxxx平台节点{{ $labels.instance }} 运行容器数量为{{ $value }}
        expr: |
          kubelet_running_pod_count{cluster="mycluster",job="kubelet"} > 110 * 0.9
        for: 2m
        labels:
          severity: warning
      - alert: KubeAPI返回请求延迟过高
        annotations:
          message: xxxxxxxxxx平台API server has a 99th percentile latency of {{ $value
            }} seconds for {{ $labels.verb }} {{ $labels.resource }}.
        expr: |
          cluster_quantile:apiserver_request_duration_seconds:histogram_quantile{cluster="mycluster",job="apiserver",quantile="0.99",subresource!="log",verb!~"^(?:LIST|WATCH|WATCHLIST|PROXY|CONNECT)$"} > 3
        for: 2m
        labels:
          severity: critical
      - alert: KubeAPI返回请求错误率超过阈值
        annotations:
          message: xxxxxxxxx平台API server 返回请求错误率为 {{ $value }}%
        expr: |
          sum(rate(apiserver_request_total{cluster="mycluster",job="apiserver",code=~"^(?:5..)$"}[5m]))
            /
          sum(rate(apiserver_request_total{cluster="mycluster",job="apiserver"}[5m])) * 100 > 3
        for: 2m
        labels:
          severity: critical
    - name: alertmanager.rules
      rules:
      - alert: Alertmanager配置文件不一致
        annotations:
          message: xxxxxxxxx平台Alertmanager配置文件不一致
        expr: |
          avg(alertmanager_config_hash)/max(alertmanager_config_hash) !=1
        for: 5m
        labels:
          severity: warning
      - alert: Alertmanager配置文件加载失败
        annotations:
          message: xxxxxxxxxx平台Alertmanager配置文件加载失败{{ $labels.namespace }}/{{ $labels.pod}}.
        expr: |
          alertmanager_config_last_reload_successful{job="alertmanager-main",namespace="monitoring"} == 0
        for: 5m
        labels:
          severity: warning
      - alert: Alertmanager实例数量异常
        annotations:
          message: xxxxxxxxx平台Alertmanager实例数量异常
        expr: |
          alertmanager_cluster_members{job="alertmanager-main",namespace="monitoring"}
            != on (service) GROUP_LEFT()
          count by (service) (alertmanager_cluster_members{job="alertmanager-main",namespace="monitoring"})
        for: 5m
        labels:
          severity: warning
    - name: general.rules
      rules:
      - alert: 监控客户端状态异常
        annotations:
          message: xxxxxxxxxx平台{{ $labels.namespace }}/{{ $labels.pod }}监控客户端{{ $labels.job
            }} 状态异常
        expr: up{cluster="mycluster"} == 0
        for: 2m
        labels:
          severity: warning
      - alert: 监控系统心跳监测
        annotations:
          message: xxxxxxxxx平台监控系统心跳监测，收到这条信息标识监控系统运行正常
        expr: vector(1)
        labels:
          severity: info
      - alert: 节点NTP时钟offset
        annotations:
          message: xxxxxxxx平台 node-exporter {{ $labels.namespace }}/{{ $labels.pod
            }}发现时钟状态异常
        expr: |
          time()-node_time_seconds{cluster="mycluster"} > 300
        for: 2m
        labels:
          severity: warning
    - name: node-network
      rules:
      - alert: 节点网卡状态发生改变
        annotations:
          message: xxxxxxxx平台节点{{ $labels.instance }} 网卡 {{ $labels.device }}状态发生改变
        expr: |
          changes(node_network_up{cluster="mycluster",device!~"veth.+"}[2m]) > 0
        for: 2m
        labels:
          severity: warning
    - name: cayenne-mysql
      rules:
      - alert: mytest前端MYSQL状态异常
        annotations:
          message: xxxxxxx平台 前端MYSQL {{ $labels.job }}状态异常
        expr: |
          mysql_up{cluster="mycluster"}==0
        labels:
          severity: critical
      - alert: mytest前端MYSQL主节点为ReadOnly
        annotations:
          message: xxxxxxx平台 前端MYSQL主节点{{ $labels.job }}为ReadOnly
        expr: |
          mysql_global_variables_read_only{cluster="mycluster",job="cayenne-mysql-master"} == 1
        labels:
          severity: critical
      - alert: mytest前端MYSQL从节点为非ReadOnly
        annotations:
          message: xxxxxxxx平台 前端MYSQL主节点{{ $labels.job }}为ReadOnly
        expr: |
          mysql_global_variables_read_only{cluster="mycluster",job="cayenne-mysql-slave"} == 0
        labels:
          severity: critical
      - alert: mytest前端MYSQL IO_THREAD线程状态异常
        annotations:
          message: xxxxxx平台 前端MYSQL从节点{{ $labels.job }}IO_THREAD线程状态异常
        expr: |
          mysql_slave_status_slave_io_running{cluster="mycluster",job="cayenne-mysql-slave"} == 0
        labels:
          severity: error
      - alert: mytest前端MYSQL SQL_THREAD线程状态异常
        annotations:
          message: xxxxxxxxxx平台 前端MYSQL从节点{{ $labels.job }}SQL_THREAD线程状态异常
        expr: |
          mysql_slave_status_slave_sql_running{cluster="mycluster",job="cayenne-mysql-slave"} == 0
        labels:
          severity: error
      - alert: mytest前端MYSQL主从同步差距超过阈值
        annotations:
          message: xxxxxxxxxx平台 前端MYSQL主从同步差距超过阈值
        expr: |
          mysql_slave_status_seconds_behind_master{cluster="mycluster"}>300
        labels:
          severity: error
kind: ConfigMap
metadata:
  labels:
    managed-by: prometheus-operator
    prometheus-name: k8s
  name: prometheus-k8s-rulefiles-0
  namespace: monitoring
  ownerReferences:
  - apiVersion: monitoring.coreos.com/v1
    blockOwnerDeletion: true
    controller: true
    kind: Prometheus
    name: k8s
    uid: 16bbd837-ae35-4321-8034-22d2c644c996
```

