



---

参考链接：

 1. [https://dbaplus.cn/news-134-3247-1.html](https://dbaplus.cn/news-134-3247-1.html)
 2. [https://github.com/aiopstack/Prometheus-book](https://github.com/aiopstack/Prometheus-book)
 3. [kubernetes ingress-nginx通过外网访问您的应用](https://ghostwritten.blog.csdn.net/article/details/112831123)
 4. [https://github.com/kubernetes/kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
 5. [https://github.com/kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)
 6. [https://www.cnblogs.com/jiangwenhui/p/11989470.html](https://www.cnblogs.com/jiangwenhui/p/11989470.html)
 7. [https://cloud.tencent.com/developer/article/1536967](https://cloud.tencent.com/developer/article/1536967)

## 1. 前言
本文首先从Prometheus是如何监控Kubernetes入手，介绍Prometheus Operator组件。接着详细介绍基于Kubernetes的两种Prometheus部署方式，最后介绍服务配置、监控对象以及数据展示和告警。通过本文，大家可以在Kubernetes集群的基础上学习和搭建完善的Prometheus监控系统。

----
## 2. Prometheus与Kubernetes完美结合
### 2.1 Kubernetes Operator
在Kubernetes的支持下，管理和伸缩Web应用、移动应用后端以及API服务都变得比较简单了。因为这些应用一般都是无状态的，所以Deployment这样的基础Kubernetes API对象就可以在无需附加操作的情况下，对应用进行伸缩和故障恢复了。


而对于数据库、缓存或者监控系统等有状态应用的管理，就是挑战了。这些系统需要掌握应用领域的知识，正确地进行伸缩和升级，当数据丢失或不可用的时候，要进行有效的重新配置。我们希望这些应用相关的运维技能可以编码到软件之中，从而借助Kubernetes 的能力，正确地运行和管理复杂应用。

 
`Operator`这种软件，使用TPR（第三方资源，现在已经升级为CRD）机制对Kubernetes API进行扩展，将特定应用的知识融入其中，让用户可以创建、配置和管理应用。与Kubernetes的内置资源一样，Operator操作的不是一个单实例应用，而是集群范围内的多实例。

### 2.3 Prometheus Operator
Kubernetes的`Prometheus Operator`为Kubernetes服务和Prometheus实例的部署和管理提供了简单的监控定义。

 

安装完毕后，Prometheus Operator提供了以下功能：

 1. 创建/毁坏。在Kubernetes namespace中更容易启动一个Prometheus实例，一个特定的应用程序或团队更容易使用的Operato。
 2. 简单配置。配Prometheus的基础东西，比如在Kubernetes的本地资源`versions`， `persistence`，`retention policies`和`replicas`。
 3. Target Services通过标签。基于常见的Kubernetes label查询，自动生成监控target配置；不需要学习Prometheus特定的配置语言。


Prometheus Operator架构如图1所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118144725530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

架构中的各组成部分以不同的资源方式运行在Kubernetes集群中，它们各自有不同的作用。

 1. `Operator`：Operator资源会根据自定义资源（`Custom Resource Definition，CRD`）来部署和管理`Prometheus Server`，同时监控这些自定义资源事件的变化来做相应的处理，是整个系统的控制中心。
 2. `Prometheus`： Prometheus资源是声明性地描述Prometheus部署的期望状态。
 3. `Prometheus Server`： Operator根据自定义资源Prometheus类型中定义的内容而部署的`Prometheus Server`集群，这些自定义资源可以看作用来管理Prometheus Server 集群的StatefulSets资源。
 4. `ServiceMonitor`：ServiceMonitor也是一个自定义资源，它描述了一组被Prometheus监控的target列表。该资源通过标签来选取对应的Service Endpoint，让Prometheus Server通过选取的Service来获取Metrics信息。
 5. `Service`：Service资源主要用来对应Kubernetes集群中的Metrics Server Pod，提供给ServiceMonitor选取，让Prometheus Server来获取信息。简单说就是Prometheus监控的对象，例如`Node Exporter Service`、`Mysql Exporter Service`等。
 6. `Alertmanager`：Alertmanager也是一个自定义资源类型，由Operator根据资源描述内容来部署Alertmanager集群。

-----
##  3. 在Kubernetes上部署Prometheus的传统方式
本节详细介绍Kubernetes通过YAML文件方式部署Prometheus的过程，即按顺序部署了`Prometheus`、`kube-state-metrics`、`node-exporter`以及`Grafana`。图2展示了各个组件的调用关系。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011814523315.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
在Kubernetes Node上部署`Node exporter`，获取该节点物理机或者虚拟机的监控信息，在Kubernetes Master上部署`kube-state-metrics`获取Kubernetes集群的状态。所有信息汇聚到Prometheus进行处理和存储，然后通过Grafana进行展示。

> **下载介质，也可以不用下载，直接对文章的yaml复制黏贴，当然无论怎么样都需要根据自己的环境修改恰当的配置参数。**

```bash
git clone https://github.com/aiopstack/Prometheus-book
cd Prometheus-book-master/scripts-config/Chapter11/
ls -l
ls -l 
total 64
-rw-r--r-- 1 root root 1756 Jan 17 23:52 grafana-deploy.yaml
-rw-r--r-- 1 root root  261 Jan 17 23:52 grafana-ingress.yaml
-rw-r--r-- 1 root root  207 Jan 17 23:52 grafana-service.yaml
-rw-r--r-- 1 root root  444 Jan 17 23:52 kube-state-metrics-deploy.yaml
-rw-r--r-- 1 root root 1087 Jan 17 23:52 kube-state-metrics-rbac.yaml
-rw-r--r-- 1 root root  293 Jan 17 23:52 kube-state-metrics-service.yaml
-rw-r--r-- 1 root root 1728 Jan 17 23:52 node_exporter-daemonset.yaml
-rw-r--r-- 1 root root  390 Jan 17 23:52 node_exporter-service.yaml
-rw-r--r-- 1 root root   65 Jan 17 23:52 ns-monitoring.yaml
-rw-r--r-- 1 root root 5052 Jan 17 23:52 prometheus-core-cm.yaml
-rw-r--r-- 1 root root 1651 Jan 18 01:59 prometheus-deploy.yaml
-rw-r--r-- 1 root root  269 Jan 17 23:52 prometheus_Ingress.yaml
-rw-r--r-- 1 root root  679 Jan 17 23:52 prometheus-rbac.yaml
-rw-r--r-- 1 root root 2429 Jan 17 23:52 prometheus-rules-cm.yaml
-rw-r--r-- 1 root root  325 Jan 17 23:52 prometheus-service.yaml
```
---
### 3.1 Kubernetes部署Prometheus   
部署对外可访问Prometheus:

 1. 首先需要创建Prometheus所在命名空间,
 2. 然后创建Prometheus使用的RBAC规则,
 3. 创建Prometheus的configmap来保存配置文件,
 4. 创建service进行固定集群IP访问,
 5. 创建deployment部署带有Prometheus容器的pod,
 6. 最后创建ingress实现外部域名访问Prometheus。


部署顺序如图3所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210118145457830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


#### 3.1.1 创建命名空间monitoring
相关对象都部署到该命名空间，使用以下命令创建命名空间：

```bash
$ kubectl create namespace monitoring
```
或者

```bash
$ kubectl create -f ns-monitoring.yaml
```
代码如下
```bash
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```
可以看到该YAML文件使用的apiVersion版本是v1，kind是Namespace，命名空间的名字是monitoring。
使用以下命令确认名为monitoring的ns已经创建成功：

```bash
$ kubectl get ns monitoring
NAME         STATUS    AGE
monitoring   Active    1d
```
#### 3.1.2 创建RBAC规则
创建RBAC规则，包含`ServiceAccount`、`ClusterRole`、`ClusterRoleBinding`三类YAML文件。Service Account 是面向命名空间的，ClusterRole、ClusterRoleBinding是面向整个集群所有命名空间的，可以看到ClusterRole、ClusterRoleBinding对象并没有指定任何命名空间。ServiceAccount中可以看到，名字是prometheus-k8s，在monitoring命名空间下。ClusterRole一条规则由apiGroups、resources、verbs共同组成。ClusterRoleBinding中subjects是访问API的主体，subjects包含users、groups、service accounts三种类型，我们使用的是ServiceAccount类型，使用以下命令创建RBAC：

```bash
kubectl create -f prometheus-rbac.yaml
```

rometheus-rbac.yaml文件内容如下：

```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus-k8s
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources: ["nodes", "services", "endpoints", "pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
```
使用以下命令确认RBAC是否创建成功：

```bash
$ kubectl get sa prometheus-k8s -n monitoring
NAME             SECRETS   AGE
prometheus-k8s   1         2m17s
$ kubectl get clusterrole prometheus
NAME         CREATED AT
prometheus   2021-01-18T07:42:36Z
$ kubectl get clusterrolebinding prometheus
NAME         ROLE                        AGE
prometheus   ClusterRole/cluster-admin   2m24s
```
#### 3.1.3 创建ConfigMap类型的Prometheus配置文件
使用ConfigMap方式创建Prometheus配置文件，YAML文件中使用的类型是ConfigMap，命名空间为monitoring，名称为`prometheus-core`，apiVersion是v1，data数据中包含`prometheus.yaml`文件，内容是prometheus.yaml: |这行下面的内容。使用以下命令创建Prometheus的配置文件：

```bash
$ kubectl create -f prometheus-core-cm.yaml
```
prometheus-core-cm.yaml文件内容如下：

```bash
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: prometheus-core
  namespace: monitoring
apiVersion: v1
data:
  prometheus.yaml: |
    global:
      scrape_interval: 15s
      scrape_timeout: 15s
      evaluation_interval: 15s
    alerting:
      alertmanagers:
      - static_configs:
        - targets: ["$alertmanagerIP:9093"]
    rule_files:
      - "/etc/prometheus-rules/*.yml"
    scrape_configs:
    - job_name: 'kubernetes-apiservers'
    
      kubernetes_sd_configs:
      - role: endpoints
    
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        
        
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https
    - job_name: 'kubernetes-nodes'
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    
      kubernetes_sd_configs:
      - role: node
    
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: $kube-apiserverIP:6443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics
    
    - job_name: 'kubernetes-cadvisor'
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    
      kubernetes_sd_configs:
      - role: node
    
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: 36.111.140.20:6443

      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
    
    - job_name: 'kubernetes-service-endpoints'
    
      kubernetes_sd_configs:
      - role: endpoints
    
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name
    
    - job_name: 'kubernetes-services'
    
      metrics_path: /probe
      params:
        module: [http_2xx]
    
      kubernetes_sd_configs:
      - role: service
    
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: kubernetes_name
    
    - job_name: 'kubernetes-pods'
    
      kubernetes_sd_configs:
      - role: pod
    
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name 
```


使用以下命令查看已创建的配置文件prometheus-core：

```bash
$ kubectl get cm prometheus-core -n monitoring 
NAME              DATA   AGE
prometheus-core   1      44s
```
#### 3.1.4 创建ConfigMap类型的prometheus rules配置文件
创建prometheus rules配置文件，使用ConfigMap方式创建prometheus rules配置文件，包含的内容是两个文件，分别是`node-up.yml`和`cpu-usage.yml`。使用以下命令创建Prometheus的另外两个配置文件：

```bash
$ kubectl create -f prometheus-rules-cm.yaml
```
`prometheus-rules-cm.yaml`文件内容如下：

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: prometheus-rules
  namespace: monitoring
data:
  node-up.yml: |
    groups:
    - name: server_rules
      rules:
      - alert: 机器宕机
        expr: up{component="node-exporter"} != 1
        for: 1m
        labels:
          severity: "warning"
          instance: "{{ $labels.instance }}"
        annotations:
          summary: "机器 {{ $labels.instance }} 处于down的状态"
          description: "{{ $labels.instance }} of job {{ $labels.job }} 已经处于down状态超过1分钟，请及时处理"
  cpu-usage.yml: |
    groups:
    - name: cpu_rules
      rules:
      - alert: cpu 剩余量过低
        expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 1m
        labels:
          severity: "warning"
          instance: "{{ $labels.instance }}"
        annotations:
          summary: "机器 {{ $labels.instance }} cpu 已用超过设定值"
          description: "{{ $labels.instance }} CPU 用量已超过 85% (current value is: {{ $value }})，请及时处理"
  low-disk-space.yml: |
    groups:
    - name: disk_rules
      rules:
      - alert: disk 剩余量过低
        expr: 1 - node_filesystem_free_bytes{fstype!~"rootfs|selinuxfs|autofs|rpc_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse.*",device=~"/dev/.*"} /
              node_filesystem_size_bytes{fstype!~"rootfs|selinuxfs|autofs|rpc_pipefs|tmpfs|udev|none|devpts|sysfs|debugfs|fuse.*",device=~"/dev/.*"} > 0.85
        for: 1m
        labels:
          severity: "warning"
          instance: "{{ $labels.instance }}"
        annotations: 
          summary: "{{$labels.instance}}: 磁盘剩余量低于设定值"
          description: "{{$labels.instance}}:  磁盘用量超过 85% (current value is: {{ $value }})，请及时处理"
  mem-usage.yml: |
    groups:
    - name: memory_rules
      rules:
      - alert: memory 剩余量过低
        expr: (node_memory_MemTotal_bytes - (node_memory_MemFree_bytes+node_memory_Buffers_bytes+node_memory_Cached_bytes )) / node_memory_MemTotal_bytes * 100 > 85
        for: 1m
        labels:
          severity: "warning"
          instance: "{{ $labels.instance }}"
        annotations:
          summary: "{{$labels.instance}}: 内存剩余量过低"
          description: "{{$labels.instance}}: 内存使用量超过 85% (current value is: {{ $value }} ，请及时处理"
```

使用以下命令查看已下发的配置文件`prometheus-rules`：

```bash
$ kubectl get cm -n monitoring prometheus-rules
NAME               DATA   AGE
prometheus-rules   4      2s
```
#### 3.1.5 创建prometheus svc
会生成一个`CLUSTER-IP`进行集群内部的访问，CLUSTER-IP也可以自己指定。使用以下命令创建Prometheus要用的service：

```bash
$ kubectl create -f prometheus-service.yaml
```
prometheus-service.yaml文件内容如下：

```bash
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
  labels:
    app: prometheus
    component: core
  annotations:
    prometheus.io/scrape: 'true'
spec:
  ports:
    - port: 9090
      targetPort: 9090
      protocol: TCP
      name: webui
  selector:
    app: prometheus
    component: core
```
查看已创建的名为prometheus的service：
```bash
$ kubectl get svc prometheus  -n monitoring
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
prometheus   ClusterIP   10.98.66.13   <none>        9090/TCP   11s
```
#### 3.1.6 deployment方式创建prometheus实例
可以根据自己的环境需求选择部署节点，我计划部署在node1

```bash
$ kubectl label node node1 app=prometheus
$ kubectl label node node1 component=core
$ kubectl create -f prometheus-deploy.yaml
```
prometheus-deploy.yaml文件内容如下：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-core
  namespace: monitoring
  labels:
    app: prometheus
    component: core
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
      component: core
  template:
    metadata:
      name: prometheus-main
      labels:
        app: prometheus
        component: core
    spec:
      serviceAccountName: prometheus-k8s
#      nodeSelector:
#        kubernetes.io/hostname: 192.168.211.40
      containers:
      - name: prometheus
        image: zqdlove/prometheus:v2.0.0
        args:
          - '--storage.tsdb.retention=15d'
          - '--config.file=/etc/prometheus/prometheus.yaml'
          - '--storage.tsdb.path=/home/prometheus_data'
          - '--web.enable-lifecycle' 
        ports:
        - name: webui
          containerPort: 9090
        resources:
          requests:
            cpu: 1000m
            memory: 1000M
          limits:
            cpu: 1000m
            memory: 1000M
        securityContext:
          privileged: true
        volumeMounts:
        - name: data
          mountPath: /home/prometheus_data
        - name: config-volume
          mountPath: /etc/prometheus
        - name: rules-volume
          mountPath: /etc/prometheus-rules
        - name: time
          mountPath: /etc/localtime
      volumes:
      - name: data
        hostPath:
          path: /home/cdnadmin/prometheus_data 
      - name: config-volume
        configMap:
          name: prometheus-core
      - name: rules-volume
        configMap:
          name: prometheus-rules
      - name: time
        hostPath:
          path: /etc/localtime
```
使用以下命令查看已创建的名字为prometheus-core的deployment的状态：

```bash
$ kubectl get deployments.apps -n monitoring 
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
prometheus-core   1/1     1            1           75s
$ kubectl get pods -n monitoring 
NAME                               READY   STATUS    RESTARTS   AGE
prometheus-core-6544fbc888-m58hf   1/1     Running   0          78s
```
返回信息表示部署期望的pod有1个，当前有1个，更新到最新状态的有1个，可用的有1个，pod当前的年龄是1天。

#### 3.1.7 创建prometheus ingress实现外部域名访问
使用以下命令创建Ingress：

```bash
$ kubectl create -f prometheus_Ingress.yaml
```
prometheus_Ingress.yaml文件内容如下：

```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-prometheus
  namespace: monitoring
spec:
  rules:
  - host: prometheus.test.com
    http:
      paths:
      - path: /
        backend:
          serviceName: prometheus
          servicePort: 9090
```
将prometheus.test.com域名解析到Ingress服务器，此时可以通过prometheus.test.com访问Prometheus的监控数据的界面了。

使用以下命令查看已创建Ingress的状态：

```bash
$ kubectl get ing traefik-prometheus -n monitoring
NAME                 CLASS    HOSTS                 ADDRESS   PORTS   AGE
traefik-prometheus   <none>   prometheus.test.com             80      52s
```
#### 3.1.8 测试登录prometheus
将`prometheus.test.com`解析到Ingress服务器，此时可以通过`grafana.test.com`访问Grafana的监控展示的界面。
linux文件/etc/hosts添加：

```bash
#任意node_ip
192.168.211.41 prometheus.test.com
```
执行：`30304`是nginx-ingress的统一对外开方端口，[【kubernets如何集群安装ningx-ingress】](https://ghostwritten.blog.csdn.net/article/details/112831123)

```bash
$ curl prometheus.test.com:30304
<a href="/graph">Found</a>.

```
windows添加`C:\Windows\System32\drivers\etc\hosts`

```bash
192.168.211.41 prometheus.test.com
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119193945524.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


----
###  3.2 Kubernetes部署kube-state-metrics
`kube-state-metrics`使用名为monitoring的命名空间，在上节已创建，不需要再次创建。创
#### 3.2.1 创建RBAC
包含`ServiceAccount`、`ClusterRole`、`ClusterRoleBinding`三类YAML文件，本节RBAC内容结构和上节中内容类似。使用以下命令创建kube-state-metrics RBAC：

```bash
$ kubectl create -f kube-state-metrics-rbac.yaml
```
`kube-state-metrics-rbac.yaml`文件内容如下：

```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-state-metrics
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kube-state-metrics
rules:
- apiGroups: [""]
  resources: ["nodes","pods","services","resourcequotas","replicationcontrollers","limitranges"]
  verbs: ["list", "watch"]
- apiGroups: ["extensions"]
  resources: ["daemonsets","deployments","replicasets"]
  verbs: ["list", "watch"]
- apiGroups: ["batch/v1"]
  resources: ["job"]
  verbs: ["list", "watch"]
- apiGroups: ["v1"]
  resources: ["persistentvolumeclaim"]
  verbs: ["list", "watch"]
- apiGroups: ["apps"]
  resources: ["statefulset"]
  verbs: ["list", "watch"]
- apiGroups: ["batch/v2alpha1"]
  resources: ["cronjob"]
  verbs: ["list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-state-metrics
#  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kube-state-metrics
  namespace: monitoring
```
使用以下命令确认RBAC是否创建成功，命令分别获取已创建的ServiceAccount、ClusterRole、ClusterRoleBinding：

```bash
$ kubectl get sa kube-state-metrics -n monitoring
NAME                 SECRETS   AGE
kube-state-metrics   1         20s
$ kubectl get clusterrole kube-state-metrics
NAME                 CREATED AT
kube-state-metrics   2021-01-19T08:44:39Z
$ kubectl get clusterrolebinding kube-state-metrics
NAME                 ROLE                             AGE
kube-state-metrics   ClusterRole/kube-state-metrics   32s
```
#### 3.2.2 创建kube-state-metrics Service

```bash
kubectl create -f kube-state-metrics-service.yaml
```
`kube-state-metrics-service.yaml`文件内容如下：

```bash
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  name: kube-state-metrics
  namespace: monitoring
  labels:
    app: kube-state-metrics
spec:
  ports:
  - name: kube-state-metrics
    port: 8080
    protocol: TCP
  selector:
    app: kube-state-metrics
```
使用以下命令查看名为kube-state-metrics的Service：

```bash
$ kubectl get svc kube-state-metrics -n monitoring
NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kube-state-metrics   ClusterIP   10.101.0.15   <none>        8080/TCP   41s
```
#### 3.2.3 创建kube-state-metrics的deployment
用来部署kube-state-metrics 容器：
可以根据自己的环境需求选择部署节点，我计划部署在node2

```bash
$ kubectl label node node2 app=kube-state-metrics  
$ kubectl create -f kube-state-metrics-deploy.yaml
```
`kube-state-metrics-deploy.yaml`文件内容如下：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-state-metrics
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-state-metrics  
  template:
    metadata:
      labels:
        app: kube-state-metrics
    spec:
      serviceAccountName: kube-state-metrics
      nodeSelector:
        type: k8smaster
      containers:
      - name: kube-state-metrics
        image: zqdlove/kube-state-metrics:v1.0.1
        ports:
        - containerPort: 8080

```
使用以下命令查看monitoring命名空间下名为kube-state-metrics的deployment的状态信息：

```bash
kubectl get deployment  kube-state-metrics -n monitoring
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
kube-state-metrics   1/1     1            1           8m9s

```
#### 3.2.4 prometheus配置kube-state-metrics 的target
[prometheus relabel_configs 实现自定义标签及分类](https://ghostwritten.blog.csdn.net/article/details/113355804)



-----
### 3.3 Kubernetes部署node-exporter
在Prometheus中负责数据汇报的程序统一称为Exporter，而不同的Exporter负责不同的业务。它们具有统一命名格式，即xx_exporter，例如，负责主机信息收集的node_exporter。本节为安装node_exporter的教程。node_exporter主要用于*NIX系统监控，用Golang编写。
node-exporter使用名为monitoring的命名空间，上节已创建。

#### 3.3.1 部署node-exporter service

```bash
$ kubectl create -f node_exporter-service.yaml
```

`node_exporter-service.yaml`文件内容如下：

```bash
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: 'true'
  name: prometheus-node-exporter
  namespace: monitoring
  labels:
    app: prometheus
    component: node-exporter
spec:
  clusterIP: None
  ports:
    - name: prometheus-node-exporter
      port: 9100
      protocol: TCP
  selector:
    app: prometheus
    component: node-exporter
  type: ClusterIP
```

使用以下命令查看monitoring命名空间下名为prometheus-node-exporter的service：

```bash
$ kubectl get svc prometheus-node-exporter -n monitoring 
NAME                       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
prometheus-node-exporter   ClusterIP   None         <none>        9100/TCP   20s
```
#### 3.3.2 daemonset方式创建node-exporter容器

```bash
$ kubectl label node --all node-exporter=node-exporter
$ kubectl create -f node_exporter-daemonset.yaml
```
查看monitoring命令空间下名为prometheus-node-exporter的daemonset的状态，命令如下：

```bash
$ kubectl get ds prometheus-node-exporter -n monitoring
NAME                       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR             AGE
prometheus-node-exporter   2         2         2       2            2           
```
 

`node_exporter-daemonset.yaml`文件详细内容如下：

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: prometheus-node-exporter
  namespace: monitoring
  labels:
#    app: prometheus
#    component: node-exporter
    node-exporter: node-exporter
spec:
  selector:
    matchLabels:
#      app: prometheus
      node-exporter: node-exporter
      #component: node-exporter
  template:
    metadata:
      name: prometheus-node-exporter
      labels:
        node-exporter: node-exporter
        #app: prometheus
        #component: node-exporter
    spec:
      nodeSelector:
        node-exporter: node-exporter
      containers:
      - image: zqdlove/node-exporter:v0.16.0
        name: prometheus-node-exporter
        ports:
        - name: prom-node-exp
          #^ must be an IANA_SVC_NAME (at most 15 characters, ..)
          containerPort: 9100
          # hostPort: 9100
        resources:
          requests:
           # cpu: 20000m
            cpu: "0.6"
            memory: 100M
          limits:
            cpu: "0.6"
            #cpu: 20000m
            memory: 100M
        securityContext:
          privileged: true

        command:
        - /bin/node_exporter
        - --path.procfs
        - /host/proc
        - --path.sysfs
        - /host/sys
        -  --collector.filesystem.ignored-mount-points
        - ^/(sys|proc|dev|host|etc)($|/)
        volumeMounts:
        - name: dev
          mountPath: /host/dev
        - name: proc
          mountPath: /host/proc
        - name: sys
          mountPath: /host/sys
        - name: root
          mountPath: /rootfs
      tolerations:
      - key: "node-role.kubernetes.io/master"
        operator: "Exists"
        effect: "NoSchedule"
      volumes:
      - name: dev
        hostPath:
          path: /dev
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
      - name: root
        hostPath:
          path: /
#      affinity:
#        nodeAffinity:
#          requiredDuringSchedulingIgnoredDuringExecution:
#            nodeSelectorTerms:
#            - matchExpressions:
#                - key: kubernetes.io/hostname
#                  operator: NotIn
#                  values:
#                  - $YOUR_IP
#
      hostNetwork: true
      hostIPC: true
      hostPID: true

```
`node_exporter-daemonset.yaml`文件说明：

```bash
 - hostPID:true
 - hostIPC:true
 - hostNetwork:true
```

这三个配置主要用于主机的`PID namespace`、`IPC namespace`以及主机网络

```bash
  tolerations:
  - key: "node-role.kubernetes.io/master"
    operator: "Exists"
    effect: "NoSchedule"
```
```bash
$ kubectl describe nodes |grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule
Taints:             <none>
Taints:             <none>
```
查看有污点的节点为master，如果想把daemonset pod部署到master，需要容忍这个节点的污点，也可以称之为过滤。具体关于容忍度与污点详解请参考：[kubernetes 【调度和驱逐】【1】污点和容忍度](https://ghostwritten.blog.csdn.net/article/details/113109564)


#### 3.3.3 prometheus配置node-exporter的target
[prometheus relabel_configs 实现自定义标签及分类](https://ghostwritten.blog.csdn.net/article/details/113355804)


----
### 3.4 Kubernetes部署Grafana
Grafana使用名为monitoring的命名空间，前面小节已经创建，不需要再次创建.
#### 3.4.1 创建Grafana Service

```bash
$ kubectl create -f grafana-service.yaml
```
`grafana-service.yaml`文件内容如下：

```bash
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
  labels:
    app: grafana
    component: core
spec:
  ports:
    - port: 3000
  selector:
    app: grafana
    component: core
```
使用以下命令查看monitoring命令空间下名为grafana的service的信息：

```bash
$ kubectl get svc grafana -n monitoring
NAME      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
grafana   ClusterIP   10.109.232.15   <none>        3000/TCP   18s
```
#### 3.4.2 deployment方式部署Grafana
根据自己需求选择部署的节点，我计划在node2
```bash
$ kubectl label node node2 grafana=grafana
#要预先配置好grafana的配置文件
```
node2执行
```bash
$ docker run -tid zqdlove/grafana:v5.0.0 --name grafana-tmp bash
$ docker cp practical_morse:/et/grafana/grafana.ini /etc/grafana/
$ docker kill grafana-tmp
$ docker rm grafana-tmp
$ useradd grafana
$ chown grafana:grafana /etc/grafana/grafana.ini
$ chmod 777 /etc/grafana/grafana.ini
```
master执行
```bash
$ kubectl create -f grafana-deploy.yaml
```

 

`grafana-deploy.yaml`文件内容如下：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-core
  namespace: monitoring
  labels:
    app: grafana
    component: core
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
      component: core
  template:
    metadata:
      labels:
        app: grafana
        component: core
    spec:
      nodeSelector:
        #kubernetes.io/hostname: 192.168.211.42
        grafana: grafana
      containers:
      - image: zqdlove/grafana:v5.0.0
        name: grafana-core
        imagePullPolicy: IfNotPresent
        #securityContext:
         # privileged: true
        # env:
        resources:
          # keep request = limit to keep this container in guaranteed class
          limits:
            cpu: 500m
            memory: 1200Mi
          requests:
            cpu: 500m
            memory: 1200Mi
        env:
          # The following env variables set up basic auth twith the default admin user and admin password.
          - name: GF_AUTH_BASIC_ENABLED
            value: "true"
          - name: GF_AUTH_ANONYMOUS_ENABLED
            value: "false"
          # - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          #   value: Admin
          # does not really work, because of template variables in exported dashboards:
          # - name: GF_DASHBOARDS_JSON_ENABLED
          #   value: "true"
        readinessProbe:
          httpGet:
            path: /login
            port: 3000
          # initialDelaySeconds: 30
          # timeoutSeconds: 1
        volumeMounts:
        - name: grafana-persistent-storage
          mountPath: /var
        - name: grafana
          mountPath: /etc/grafana    
      imagePullSecrets:
      - name: bjregistry
      volumes:
      - name: grafana-persistent-storage
        emptyDir: {}
      - name: grafana
        hostPath:
          path: /etc/grafana

```


查看monitoring命令空间下名为grafana-core的deployment的状态，信息如下：

```bash
$ kubectl get deployment grafana-core -n monitoring
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
grafana-core   1/1     1            1           8m32s
```
#### 3.4.3 创建grafana ingress实现外部域名访问

```bash
$ kubectl create -f grafana-ingress.yaml
```

 

grafana-ingress.yaml文件内容如下：

```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-grafana
  namespace: monitoring
spec:
  rules:
  - host: grafana.test.com
    http:
      paths:
      - path: /
        backend:
          serviceName: grafana
          servicePort: 3000
```


查看monitoring命名空间下名为traefik-grafana的Ingress，使用以下命令：

```bash
$ kubectl get ingress traefik-grafana -n monitoring
NAME              CLASS    HOSTS              ADDRESS   PORTS   AGE
traefik-grafana   <none>   grafana.test.com             80      30s
```
#### 3.4.4 测试登录grafana
将`grafana.test.com`解析到Ingress服务器，此时可以通过grafana.test.com访问Grafana的监控展示的界面。
linux文件/etc/hosts添加：

```bash
192.168.211.41 grafana.test.com
```
执行：`30304`是nginx-ingress的统一对外开方端口

```bash
$ curl grafana.test.com:30304
<a href="/login">Found</a>.
```
windows添加`C:\Windows\System32\drivers\etc\hosts`

```bash
192.168.211.41 grafana.test.com
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119193346792.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


