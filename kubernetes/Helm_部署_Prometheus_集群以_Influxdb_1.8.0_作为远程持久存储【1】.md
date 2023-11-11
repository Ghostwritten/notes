
![](https://img-blog.csdnimg.cn/5d07bb8aa0f549ff997f25078cec9880.png)




## 1. 简介
InfluxDB 是用Go语言编写的一个用于存储和分析时间序列数据的开源数据库，无需外部依赖。

优点：

- 专为时间序列数据编写的自定义高性能数据存储。 TSM引擎允许高摄取速度和数据压缩
- 完全用 Go 语言编写。 它编译成单个二进制文件，没有外部依赖项
- 简单，高性能的写入和查询HTTP API
- 专为类似SQL的查询语言量身定制，可轻松查询聚合数据
- 标签允许对系列进行索引以实现快速有效的查询
- 保留策略有效地自动使过时数据过期
- 连续查询自动计算聚合数据，以提高频繁查询的效率

缺点：

- 开源版本没有集群功能,集群版本需要收费




![](https://img-blog.csdnimg.cn/e1f827d7f6094bf19f5e54cf4185e341.png)
Prometheus 对 InfluxDB 1.x 和 2.0 进行双远程写入区别：

- 当您开始向 InfluxDB 1.x 远程写入数据时，您可能已经配置了prometheus.yml包含 InfluxDB 运行端口的 URL；
- 为了双重写入 InfluxDB 2.0，我们将向您的配置添加一个额外的远程写入端点。这可以简单地通过将另一个-url:指向您的 Telegraf 端点的点添加到remote_write 您的prometheus.yml.


```bash
remote_write:
  - url: "http://localhost:8086/api/v1/prom/write?db=prometheus"
  - url: "http://localhost:1234/receive"
```


这篇文章以 `InfluxDB 1.8.0` 作为 `prometheus` 的远程存储。
## 2. helm 部署 influxdb 1.8.0
- [https://bitnami.com/stack/influxdb/helm](https://bitnami.com/stack/influxdb/helm)没有 influxdb 1.x版本，只有influxdb2.x 版本，所以这里使用helm源 [https://github.com/influxdata/helm-charts](https://github.com/influxdata/helm-charts)，当然你也可以打包属于自己的源。

```bash
$ helm repo add influxdata https://helm.influxdata.com/
$ helm search repo influxdata
NAME                            CHART VERSION   APP VERSION     DESCRIPTION
influxdata/chronograf           1.2.5           1.9.4           Open-source web application written in Go and R...
influxdata/influxdb             4.12.1          1.8.10          Scalable datastore for metrics, events, and rea...
influxdata/influxdb-enterprise  0.1.22          1.10.0          Run InfluxDB Enterprise on Kubernetes
influxdata/influxdb2            2.1.1           2.3.0           A Helm chart for InfluxDB v2
influxdata/kapacitor            1.4.6           1.6.4           InfluxDB's native data processing engine. It ca...
influxdata/telegraf             1.8.26          1.25.2          Telegraf is an agent written in Go for collecti...
influxdata/telegraf-ds          1.1.8           1.25.2          Telegraf is an agent written in Go for collecti...
influxdata/telegraf-operator    1.3.11          v1.3.10         A Helm chart for Kubernetes to deploy telegraf-...

#部署1.8.10版本
$ helm install my-release-influxdb influxdata/influxdb
NAME: my-release-influxdb
LAST DEPLOYED: Mon Apr  3 14:31:45 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
InfluxDB can be accessed via port 8086 on the following DNS name from within your cluster:

  http://my-release-influxdb.default:8086

You can connect to the remote instance with the influx CLI. To forward the API port to localhost:8086, run the following:

  kubectl port-forward --namespace default $(kubectl get pods --namespace default -l app=my-release-influxdb -o jsonpath='{ .items[0].metadata.name }') 8086:8086

You can also connect to the influx CLI from inside the container. To open a shell session in the InfluxDB pod, run the following:

  kubectl exec -i -t --namespace default $(kubectl get pods --namespace default -l app=my-release-influxdb -o jsonpath='{.items[0].metadata.name}') /bin/sh

To view the logs for the InfluxDB pod, run the following:

  kubectl logs -f --namespace default $(kubectl get pods --namespace default -l app=my-release-influxdb -o jsonpath='{ .items[0].metadata.name }')
```
报错：

```bash
$ helm install my-release influxdata/influxdb
Error: INSTALLATION FAILED: Get "https://github.com/influxdata/helm-charts/releases/download/influxdb-4.12.1/influxdb-4.12.1.tgz": unexpected EOF
```


查看 influxdb 资源

```bash
$ k get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/my-release-influxdb-57b47c9c79-7p72c   1/1     Running   0          9h

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/my-release-influxdb   ClusterIP   10.96.31.13     <none>        8086/TCP,8088/TCP   9h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-release-influxdb   1/1     1            1           9h

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/my-release-influxdb-57b47c9c79   1         1         1       9h


$ k get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
persistentvolume/pvc-81bb824e-3fc1-45cd-a43c-543a91923f45   8Gi        RWO            Delete           Bound    default/my-release-influxdb   standard                9h

NAME                                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/my-release-influxdb   Bound    pvc-81bb824e-3fc1-45cd-a43c-543a91923f45   8Gi        RWO
  standard       9h

$ k get secret
NAME                               TYPE                 DATA   AGE
my-release-influxdb                Opaque               2      9h
```

## 3. 安装 influx CLI 1.8.0

- [https://docs.influxdata.com/influxdb/v1.8/tools/shell/](https://docs.influxdata.com/influxdb/v1.8/tools/shell/)

rpm 方式：
```bash
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.8.10.x86_64.rpm
sudo yum -y localinstall influxdb-1.8.10.x86_64.rpm
```
tar 方式：

```bash
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.8.10_linux_amd64.tar.gz
 tar xvfz influxdb-1.8.10_linux_amd64.tar.gz
```



正在为您搜索：influx命令的版本有哪些

根据官方文档[1]，InfluxDB 1.x有三个版本的CLI命令：

- `InfluxDB Shell`（influx命令）：这是InfluxDB 1.x的原生CLI工具，可以与本地或远程InfluxDB实例进行交互。influx命令提供了大量的功能，例如创建、删除数据库、写入、查询数据等，并且支持交互式和非交互式使用。

- `InfluxDB HTTP API`：此API允许通过HTTP请求与InfluxDB 1.x进行通信，支持使用POST和GET方法进行写入和查询操作。在使用该API进行交互时，需要指定InfluxDB实例的IP地址和端口号，并且需要授权访问。

- `InfluxDB Client Libraries`：InfluxDB提供了多种语言的客户端库，包括C#、Java、Go、Python等。这些客户端库可帮助开发人员轻松地在应用程序中集成InfluxDB，并使用简单的API来执行操作。



## 4. port-forward

```bash
$ kubectl port-forward --namespace default $(kubectl get pods --namespace default -l app.kubernetes.io/instance=my-release-influxdb -o jsonpath='{ .items[0].metadata.name }') 8086:8086
Forwarding from 127.0.0.1:8086 -> 8086
Forwarding from [::1]:8086 -> 8086
Handling connection for 8086
Handling connection for 8086
```
客户端访问：

```bash
$ influx --host 127.0.0.1 --port 8086
Connected to http://127.0.0.1:8086 version 1.8.10
InfluxDB shell version: 1.8.10
> show databases
name: databases
name
```

```bash
$ kubectl port-forward --namespace default $(kubectl get pods --namespace default -l app.kubernetes.io/instance=my-release-influxdb -o jsonpath='{ .items[0].metadata.name }') --address=192.168.10.29 8086:8086
Forwarding from 192.168.10.29:8086 -> 8086
Handling connection for 8086
Handling connection for 8086
Handling connection for 8086

$ influx --host 192.168.10.29 --port 8086
Connected to http://192.168.10.29:8086 version 1.8.10
InfluxDB shell version: 1.8.10
> show databases
name: databases
```



## 5. helm  部署 prometheus

更多细节请参考这里：[Kind & Kubernetes | 通过 Helm 部署定制化 Prometheus-Operator 上传 Dockerhub？](https://ghostwritten.blog.csdn.net/article/details/129820019)

```bash
helm upgrade prometheus oci://registry-1.docker.io/ghostwritten/ghostwritten-kube-prometheus-stack  --version 45.8.0 \--namespace prometheus  --create-namespace --install \--set prometheusOperator.admissionWebhooks.patch.image.registry=docker.io --set prometheusOperator.admissionWebhooks.patch.image.repository=dyrnq/kube-webhook-certgen \--set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \--set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false 
```

添加远程存储配置：

```bash
helm upgrade prometheus oci://registry-1.docker.io/ghostwritten/ghostwritten-kube-prometheus-stack  --version 45.8.0 \--namespace prometheus  --create-namespace --install \--set prometheusOperator.admissionWebhooks.patch.image.registry=docker.io --set prometheusOperator.admissionWebhooks.patch.image.repository=dyrnq/kube-webhook-certgen \--set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \--set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false \--set prometheus.prometheusSpec.remoteWrite[0].url=http://192.168.10.29:8086/api/v1/prom/write?db=prometheus \--set prometheus.prometheusSpec.remoteRead[0].url=http://192.168.10.29:8086/api/v1/prom/read?db=prometheus
```


或者手动下载原版 helm 版本 kube-prometheus-stack 修改配置部署

下载最新版本(20230328) [kube-prometheus-stack-45.8.0.tgz](https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-45.8.0/kube-prometheus-stack-45.8.0.tgz)

```bash
wget https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-45.8.0/kube-prometheus-stack-45.8.0.tgz
tar -zxvf kube-prometheus-stack-45.8.0.tgz
```

修改镜像版本
- `registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0`改为`docker.io/ghostwritten/kube-state-metrics:v2.8.0`


首先：`vim charts/kube-state-metrics/values.yaml`

```bash
  3 image:
  4   registry: registry.k8s.io
  5   repository: kube-state-metrics/kube-state-metrics
  6   # If unset use v + .Charts.appVersion
  7   tag: ""
```
修改为：

```bash
  3 image:
  4   registry: docker.io
  5   repository: ghostwritten/registry.k8s.io.kube-state-metrics.kube-state-metrics
  6   # If unset use v + .Charts.appVersion
  7   tag: "v2.8.0"
```
修改 `value.yaml`在末尾添加以下配置：

```bash
remoteWrite:
  - url: "http://192.168.10.29:8086/api/v1/prom/write?db=prometheus"
remoteRead:
  - url: "http://192.168.10.29:8086/api/v1/prom/read?db=prometheus"
```

部署：

```bash
helm upgrade prometheus ./kube-prometheus-stack  -f ./kube-prometheus-stack/values.yaml  --version 45.8.0 \--namespace prometheus  --create-namespace --install \--set prometheusOperator.admissionWebhooks.patch.image.registry=docker.io --set prometheusOperator.admissionWebhooks.patch.image.repository=dyrnq/kube-webhook-certgen \--set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \--set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false
```


### 5.1 更新 prometheus.yaml

如果配置需要更新，我们如何在不重新部署的情况下修改配置文件并生效。

生成 `prometheus.yaml`

```bash
$ k get secret -n prometheus
NAME                                                                TYPE                 DATA   AGE
alertmanager-prometheus-ghostwritten-ku-alertmanager                Opaque               1      5d7h
alertmanager-prometheus-ghostwritten-ku-alertmanager-generated      Opaque               1      5d7h
alertmanager-prometheus-ghostwritten-ku-alertmanager-tls-assets-0   Opaque               0      5d7h
alertmanager-prometheus-ghostwritten-ku-alertmanager-web-config     Opaque               1      5d7h
prometheus-ghostwritten-ku-admission                                Opaque               3      5d7h
prometheus-grafana                                                  Opaque               3      5d7h
prometheus-prometheus-ghostwritten-ku-prometheus                    Opaque               1      5d7h
prometheus-prometheus-ghostwritten-ku-prometheus-tls-assets-0       Opaque               1      5d7h
prometheus-prometheus-ghostwritten-ku-prometheus-web-config         Opaque               1      5d7h
sh.helm.release.v1.prometheus.v1                                    helm.sh/release.v1   1      5d7h


$ k get secret -n prometheus prometheus-prometheus-ghostwritten-ku-prometheus -oyaml
apiVersion: v1
data:
  prometheus.yaml.gz: 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
kind: Secret
metadata:
  annotations:
    generated: "true"
  creationTimestamp: "2023-03-28T09:29:33Z"
  labels:
    managed-by: prometheus-operator
  name: prometheus-prometheus-ghostwritten-ku-prometheus
  namespace: prometheus
  ownerReferences:
  - apiVersion: monitoring.coreos.com/v1
    blockOwnerDeletion: true
    controller: true
    kind: Prometheus
    name: prometheus-ghostwritten-ku-prometheus
    uid: b774698a-8122-447a-8e68-61cf3300abeb
  resourceVersion: "6312864"
  uid: 40f116bb-2cd2-40be-b163-5577c1d8512f
type: Opaque
```


```bash
echo -n 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 | base64 -d | gunzip > prometheus.yaml
```

修改 `prometheus.yaml` 重新
内容：

方法1：

```bash
remote_write:
    - url: "https://ts-1234abcd.influxdata.rds.aliyuncs.com:3242/api/v1/prom/write?db=prometheus&u=prom&p=mypassword"

remote_read:
    - url: "https://ts-1234abcd.influxdata.rds.aliyuncs.com:3242/api/v1/prom/read?db=prometheus&u=prom&p=mypassword"
```

方法2：

```bash
prometheus:
  prometheusSpec:
    remoteWrite:
    - url: "http://influxdbui.demo.com:8086/api/v1/prom/write?db=prometheus"
      basicAuth:
          username:
            name: kubepromsecret
            key: username
          password:
            name: kubepromsecret
            key: password
```


方法3：

```bash
remote_write:
- url: "http://influxdbui.demo.com:8086/api/v1/prom/write?db=prometheus"
  basic_auth:
    username: admin
    password: djuNjAZ79v
```

```bash
remote_write:
    - url: "http://192.168.10.29:8086/api/v1/prom/write?db=prometheus"

remote_read:
    - url: "http://192.168.10.29:8086/api/v1/prom/read?db=prometheus"
```

把 `prometheus.yaml` 转 `base64`

```bash
$ cat prometheus.yaml | gzip | base64 -w0
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
```

拷贝 `base64` 密文更新配置

```bash
 k get secret prometheus-prometheus-ghostwritten-ku-prometheus -n prometheus
NAME                                               TYPE     DATA   AGE
prometheus-prometheus-ghostwritten-ku-prometheus   Opaque   1      5d16h


$ k get secret prometheus-prometheus-ghostwritten-ku-prometheus -n prometheus -oyaml
apiVersion: v1
data:
  prometheus.yaml.gz: H4sIAAAAAAAA/+ydz27jONLA73kKHvrg9MCyne5M9yinAeb7sIed2cHOcbEgaLJscyyRXJJyEqz23ReUKFvyn6zdrThOUgcDsigWq8gi6ydKIueZnrIsvSIEViwrmJdaUak82BXLUvJp7K4IcdwyAzun4cGDVSyjGZtC5oIQQozVOfgFFC5tHY82h8P5Qjt/b6X3oIbLYrhJ2hJALZhMcpaSD4Pf//YL/e3nX//v+soWGdCZzMClV0MyAs9bwkchtVPaUQUPQ7ZK5nA8+pg8sjy7ikZzrWZyXpX1p55SxXJIiQO7khx+1Up6bdvlq7lUD0Op5hacG3KtvNVZBnaYg7eSu9H4ipCFVto2tUZmLHNwRciymIJV4MFRJzblEjIkVmeQElDCaKl8XU9BE2cYh1jx8UzzZ0gaJSqV9rXipGpFC5Ui3QKdLiyHTsNW9ldHntk5+DoxJZT63NBWqzXVVAliPHhUSpYApsq8VzKlOXhG2zVQ13B9HWXGtBOlplI5zxSH4wQYCw6U/19yLMzhISWDTs1d33lbPK8xsba+y5C1jOcyovE+arT1O8V1OvEB/3lCJhMiKEujay2lEidnWqvkwDDLvLYpuWvr+JsWcDdIPl7HkyZjHHJQPiUf/j35zx7fVlrA5drzuxanmWO0OMWa9Qizr2ZaaUcLbFx5bVlXZkw+RaLR4pC0E40NksJwzaQCe0jm+oJOTxJWn9STQlFmwVy3v/4/kxmI8o+CcwAB4rrHmm1G7n1OMty6tnHH3QzH9fHGi+nBWLFgblEl5loUWQCFSfW3qc+Qnj/deh0xsQY/DP74y89//+W6IyuOcnX0pduxjvzjn0fG9ScwgmVgfc4Um4M9Y3jvtEakC2qYX6RkFP8GRlNsmgFdeG9uUhKDwCsJ+SeHxE5/2mqmKbR5z3nGl52W6z3MW8ig6eNHG9HO1BiyUbt3HR1kM5rX3n6aojs5G22Dhs9OG8Gdh/cwvdzYjKyBrIGs8e2scUwPf3+kYWTId1bMEDBjRear+YMFBNVD04TMPnNRfn21VA54YYG6pTR0BVbOHjd6EVKrHu3fqFmlcVZN6aRktGJ2ZAs1csAteDfaXJhIPYrVxjjXhfIjzhJug25TYBYs9XoJ6pskVTlfCRpxnRutIHabo0N2N1szFKx9qne4MFavpIATwaKTq1Fyk+ksaIFzGMgVyBWviit6GEgPFVeblvxw2EdPw5o3M3vyVBGhEaMRQ5J1vGQdcaiFfxXgPBWFrR88OeBaCUenBV+CvxuMk8ltOU5uynHyKfzCn8/hFw5+LMfJl3KcfC3HyU/lJLm5LSdJ+H25LW/KT+Wn5Lb8XH5Obssfyy/l1/KncnJb3tyWn8fl7bhrZ+X9341nXFsQ6pyPeKppDffoPOTvDYLOMT8UG/SdTA0dMeWymVe8UJxAPEI8QjzqCY/+1NO/hoOz0tExY8xrgqReJn6q8NR+meTsT5u6pPFtU0ExQuFsz8WBzgH/QvBB8EHwQfBB8EHweUHwAc8FTqq8KdYITYp0gXSBdIF0gXSBdPGCdGGsfnhEvHhTeFG1KfIF8gXyBfIF8gXyxQvyheMLEEWGT2sQcfpEnLVbIeYg5iDmIOYg5iDmvBzmZOD38E0c9RFv3hDenGPtkuhRvaPN8qs7ndDamfpX8IivkRBsEGwQbN4N2LTHm7N9k3TkGLNeZGQv3bQv6EBILA+paQ81TV6GmvavGDPiTKykq9b0QK5CrkKuQq5CrkKuQq5Crnp+rjryg/JYwMb3uSnogM8c9Qurvc9ArL8k99qzrMw0E5StwLI50MnYlTUGbV1VOLDdU3u+Ev9GFWeODqSmvLAWlC+lpl7msKVAc/Ye5Hzhd6ywwISjOdg5iHjKAffa0jqlcyqQJjTn4p921v5MyyHX9pEOcmYMiArVSnfPTH8lDIJMKsBxK43X1pWeuaWjzjMPpV/EimEP/RXpDPDk4wniZBMu61F8LTL54W5XSm83DzcXdfNgrJ6Cw1sHvHXAW4eXRm28dcBbB7x1wFuHd3Lr0MuUrDZQDT3ne5LdWcT7exfY3GDT1qYoHKx3kXzaNPLUUqMil85JrShncVxurd55RB2+Hno6GZU6Hf6I9/WaKsE39XB9TcQmxKbXiE0nkQtCxd5dxi5ucxAM0HWAfsYdNi4zROMuIEgTSBNIE6+BJnAXEMtmTLEXm5Sodg31Mgdd+Gbr14NI0Z7DeD2EsfXw41k3F33GQH6mR0HRITG+Y3zH+I7xva+HLFv7gJ/tk773zhf1qx2eeXhik/JzoAbCwhuEhV3vOgs3XG6MRWZAZkBm6IkZnhiAzsUP75odWodhlBrCQxiRX24fdJx+ePtEccjncJEh5AvkC+QLXGToAsnCeW3ZHIJs78S0LkMXnuoZ1VaAjV8+SSX0fUrG7oplYL1U1buP1fH+z8UaFaqktY9GZVuRO9SyrN5lrITFHY6ikOr1WWNhJh9SMqq9vEsST3DLIXLZxy5b9LLDL4QwI+kKrKuMWt1Ea3YM3xvl9vbNI7vSnjrbfSm0VXM96HAg2u7O//83AAD//+IHCXilkAAA
kind: Secret
metadata:
  annotations:
    generated: "true"
  creationTimestamp: "2023-03-28T09:29:33Z"
  labels:
    managed-by: prometheus-operator
  name: prometheus-prometheus-ghostwritten-ku-prometheus
  namespace: prometheus
  ownerReferences:
  - apiVersion: monitoring.coreos.com/v1
    blockOwnerDeletion: true
    controller: true
    kind: Prometheus
    name: prometheus-ghostwritten-ku-prometheus
    uid: b774698a-8122-447a-8e68-61cf3300abeb
  resourceVersion: "7164057"
  uid: 40f116bb-2cd2-40be-b163-5577c1d8512f
type: Opaque
 
```

添加远程存储配置

```bash

$ k edit secret prometheus-prometheus-ghostwritten-ku-prometheus -n prometheus
```

或者

```bash
$ kubectl patch secret prometheus-prometheus-ghostwritten-ku-prometheus -n prometheus --type merge -p '{"data": {"promethes.yaml.gz": "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"}}'
```


更新 prometheus pod

```bash
$ k delete pod prometheus-prometheus-ghostwritten-ku-prometheus-0 -n prometheus
```





## 6. 验证 influxDB 1.8.0 存储

```bash
# 进入 influxDB 
$ influx --host 192.168.10.29 --port 8086
Connected to http://192.168.10.29:8086 version 1.8.10
InfluxDB shell version: 1.8.10
#显示库名列表
> show databases
name: databases
name
----
_internal
prometheus
切换至 prometheus 库
> use prometheus
Using database prometheus
# 显示表名
> show measurements
name: measurements
name
----
:node_memory_MemAvailable_bytes:sum
ALERTS
ALERTS_FOR_STATE
access_evaluation_duration_bucket
access_evaluation_duration_count
access_evaluation_duration_sum
access_permissions_duration_bucket
access_permissions_duration_count
access_permissions_duration_sum
aggregator_openapi_v2_regeneration_count
aggregator_openapi_v2_regeneration_duration
aggregator_unavailable_apiservice
alertmanager_alerts
alertmanager_alerts_invalid_total
alertmanager_alerts_received_total
alertmanager_build_info
alertmanager_cluster_enabled
alertmanager_config_hash
alertmanager_config_last_reload_success_timestamp_seconds
alertmanager_config_last_reload_successful
alertmanager_dispatcher_aggregation_groups
alertmanager_dispatcher_alert_processing_duration_seconds_count
alertmanager_dispatcher_alert_processing_duration_seconds_sum
alertmanager_http_concurrency_limit_exceeded_total
alertmanager_http_request_duration_seconds_b
......

显示 表内容，例如：access_evaluation_duration_bucket
> select * from access_evaluation_duration_bucket;
name: access_evaluation_duration_bucket
time                __name__                          container endpoint instance          job                le      namespace  pod                                 prometheus                                       prometheus_replica                                 service            value
----                --------                          --------- -------- --------          ---                --      ---------  ---                                 ----------                                       ------------------                                 -------            -----
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana +Inf    prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 53
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana 0.00016 prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 53
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana 0.00064 prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 53
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana 0.00256 prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 53
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana 0.01024 prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 53
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana 0.04096 prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 53
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana 0.16384 prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 53
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana 0.65536 prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 53
1680512513140000000 access_evaluation_duration_bucket grafana   http-web 10.244.0.146:3000 prometheus-grafana 1e-05   prometheus prometheus-grafana-6f77bc5bc9-2lptr prometheus/prometheus-ghostwritten-ku-prometheus prometheus-prometheus-ghostwritten-ku-prometheus-0 prometheus-grafana 2
```



参考：
-  [Prometheus Remote Write Support with InfluxDB 2.0](https://www.influxdata.com/blog/prometheus-remote-write-support-with-influxdb-2-0/)
- [Prometheus vs. InfluxDB: A Monitoring Comparison](https://logz.io/blog/prometheus-influxdb/)
- [Configure remote_write with Helm and Prometheus](https://grafana.com/docs/grafana-cloud/kubernetes-monitoring/other-methods/prometheus/remote_write_helm_prometheus/)
- [Configure remote_write with Helm and kube-prometheus-stack](https://grafana.com/docs/grafana-cloud/kubernetes-monitoring/other-methods/prometheus/remote_write_helm_operator/)
- [Integrate Prometheus with TSDB for InfluxDB®️ Service](https://www.alibabacloud.com/help/en/time-series-database/latest/integrate-prometheus-with-tsdb-for-influxdb-service)
- [How To Deploy InfluxDB / Telegraf / Grafana on K8s?](https://octoperf.com/blog/2019/09/19/kraken-kubernetes-influxdb-grafana-telegraf/#prerequisites)
- [AIX: Installing InfluxDB 1.8 and Grafana 7](http://grafana.loki.com/?orgId=1)
- [Prometheus vs. InfluxDB: A Monitoring Comparison](https://logz.io/blog/prometheus-influxdb/)
