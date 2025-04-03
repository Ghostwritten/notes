

![](https://i-blog.csdnimg.cn/blog_migrate/1e58bc4b8924856f4bcddc498d09726c.png)




## 1. 前言
在上一节课，我为你介绍了什么是金丝雀发布以及如何实施自动化金丝雀发布。在实施金丝雀发布的过程中，我们通过 Argo Rollout 的金丝雀策略将发布过程分成了 3 个阶段，每个阶段金丝雀的流量比例都不同，经过一段时间之后，金丝雀环境变成了新的生产环境。实际上，这也是一种渐进式的交付方式**，它通过延长发布时间来保护生产环境，降低了发生生产事故的概率**。不过，这种渐进式的交付方式存在一个明显的缺点：**无法自动判断金丝雀环境是否出错**。这可能会导致一种情况，**当金丝雀环境在接收生产流量之后，它产生了大量的请求错误，在缺少人工介入的情况下，发布仍然按照计划进行，最终导致生产环境故障。**

为了解决这个问题，我们希望渐进式交付变得更加智能，一个好的工程实践方式是：通过指标分析来自动判断金丝雀发布的质量，如果符合预期，就继续金丝雀步骤；如果不符合预期，则进行回滚。这样，也就能够避免将金丝雀环境的故障带到生产环境中了，这种分析方法也叫做金丝雀分析。这节课，我们就来学习如何将 Argo Rollout 和 Prometheus 结合，实现自动渐进式交付。

## 2. 准备
在开始今天的学习之前，你需要做好下面这些准备。
- 按照第一章第 2 讲的内容在本地配置好 Kind 集群，安装 `Ingress-Nginx`，并暴露 80 和 443 端口。
- 配置好 Kubectl，使其能够访问 Kind 集群。
- 按照第 24 讲的内容安装好 Argo Rollout 以及 kubectl 插件。


## 3. 自动渐进式交付概述
![](https://i-blog.csdnimg.cn/blog_migrate/b4a19b3a7e05bcc6dbf59dfb4737898c.png)
相比较金丝雀发布，自动渐进式交付增加了 `Prometheus`、`Analysis Template` 和 `AnalysisRun` 对象。其中，Analysis Template 定义用于分析的模板，AnalysisRun 是分析模板的实例化，Prometheus 是用来存储指标的数据库。

在这节课的例子中，我设计的自动渐进式交付流程会按照下面这张流程图来进行。

![](https://i-blog.csdnimg.cn/blog_migrate/dd3f41fb6df14af0449336d4fee48f83.png)
自动渐进式交付开始时，首先会先将金丝雀环境的流量比例设置为 20% 并持续两分钟，然后将金丝雀环境的流量比例设置为 40% 并持续两分钟，然后再以此类推到 60%、80%，直到将金丝雀环境提升为生产环境为止。

从第二个阶段开始，自动金丝雀分析开始运行，在持续运行的过程中，如果金丝雀分析失败，那么金丝雀环境将进行自动回滚。这样就达到了自动渐进式交付的目的。

## 4. 自动渐进式交付实战
接下来，我们进入到渐进式交付的实战环节，实战过程大致分成下面几个步骤。

- 创建生产环境，包括 Rollout 对象、Service 和 Ingress。
- 创建用于自动金丝雀分析的 AnalysisTemplate 模板。
- 安装 Prometheus 并配置 Ingress-Nginx。
- 修改镜像版本，启动渐进式交付。

### 4.1 创建生产环境
首先，我们需要创建用于模拟生产环境的 `Rollout` 对象、Service 和 Ingress。将下面的内容保存为 `rollout-with-analysis.yaml` 文件。

上面的内容相比较[第 25 讲金丝雀发布](https://blog.csdn.net/xixihahalelehehe/article/details/129120109)的 Rollout 对象并没有太大差异，只是在 `canary` 字段下面增加了 `analysis` 字段，它的作用是指定金丝雀分析的模板，模板内容我们会在稍后创建。另外，这里同样使用了 `argoproj/rollouts-demo:blue` 镜像来模拟生产环境。

然后，使用 kubectl apply 命令将它应用到集群内。

```bash
$ kubectl apply -f rollout-with-analysis.yaml
rollout.argoproj.io/canary-demo created
```
接下来，我们还需要创建 Service 对象。在这里，我们可以一并创建生产环境和金丝雀环境所需要用到的 Service ，将下面的内容保存为 `canary-demo-service.yaml`。

```bash
apiVersion: v1
kind: Service
metadata:
  name: canary-demo
  labels: 
    app: canary-demo
spec:
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http
  selector:
    app: canary-demo
---
apiVersion: v1
kind: Service
metadata:
  name: canary-demo-canary
  labels: 
    app: canary-demo
spec:
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http
  selector:
    app: canary-demo
```
然后，使用 kubectl apply 命令将它应用到集群内。

```bash
$ kubectl apply -f canary-demo-service.yaml
service/canary-demo created
service/canary-demo-canary created
```
最后，再创建 Ingress 对象。将下面的内容保存为 `canary-demo-ingress.yaml` 文件。

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: canary-demo
  labels:
    app: canary-demo
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: progressive.auto
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: canary-demo
                port:
                  name: http
```
在这个 Ingress 对象中，指定了 `progressive.auto` 作为访问域名。然后，使用 `kubectl apply` 命令将它应用到集群内。


```bash
$ kubectl apply -f canary-demo-ingress.yaml
ingress.networking.k8s.io/canary-demo created
```

### 4.2 创建 AnalysisTemplate
于我们在 `Rollout` 对象中指定了名为 `success-rate` 的金丝雀分析模板，所以我们还需要创建它。将下面的内容保存为 `analysis-success.yaml` 文件。

```bash
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
  - name: ingress
  metrics:
  - name: success-rate
    interval: 10s
    failureLimit: 3
    successCondition: result[0] > 0.90
    provider:
      prometheus:
        address: http://prometheus-kube-prometheus-prometheus.prometheus:9090
        query: >+
          sum(
            rate(nginx_ingress_controller_requests{ingress="{{args.ingress}}",status!~"[4-5].*"}[60s]))
            /
            sum(rate(nginx_ingress_controller_requests{ingress="{{args.ingress}}"}[60s])
          )
```
这里我简单介绍一下 `AnalysisTemplate` 对象字段的含义。

先 `spec.args` 字段定义了参数，该参数会在后续的 `query` 语句中使用，它的值是从 Rollout 对象的 `canary.analysis.args` 字段传递进来的。

`spec.metrics` 字段定义了自动分析的相关配置。其中，`interval` 字段为频率，每 10 秒钟执行一次分析。`failureLimit` 字段代表“连续 3 次失败则金丝雀分析失败”，此时要执行回滚动作。`successCondition` 字段代表判断条件，这里的 `result[0]` 是一个表达式，代表的含义是当查询语句的返回值大于 `0.90` 时，说明本次金丝雀分析成功了。

最后，`spec.metrics.provider` 字段定义了分析数据来源于 `Prometheus`，还定义了 Prometheus Server 的连接地址，我们将在稍后部署 Prometheus。

`query` 字段是金丝雀分析的查询语句。这条查询语句的含义你可以简单地理解成：在 60 秒内 HTTP 状态码不为 `4xx` 和 `5xx` 的请求占所有请求的比例。换句话说，当 HTTP 请求成功的比例大于 0.90 时，代表一次金丝雀分析成功。

### 4.3 访问生产环境
接下来，为了访问生产环境，你还需要先配置 Hosts。

```bash
127.0.0.1 progressive.auto
```
接下来，使用浏览器访问 `http://progressive.auto`，你应该能看到如下页面。
![](https://i-blog.csdnimg.cn/blog_migrate/85f0a045d0af724e6d3a3e79f4354b58.png)

### 4.4 安装 Prometheus
Prometheus  是 Kubernetes 平台开源的监控和报警系统。由于金丝雀分析需要用到 Prometheus 来查询指标，所以我们需要先部署它。这里我使用 Helm 的方式进行部署。

```bash

$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm upgrade prometheus prometheus-community/kube-prometheus-stack \
--namespace prometheus  --create-namespace --install \
--set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \
--set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false

Release "prometheus" does not exist. Installing it now.
......
STATUS: deployed
```
我这里 `helm repo add`卡住了：
配置[clash for linux](https://blog.csdn.net/xixihahalelehehe/article/details/129147359)

添加成功。
安装过程中下载[kube-prometheus-stack-45.2.0.tgz](https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-45.2.0/kube-prometheus-stack-45.2.0.tgz)失败。


我用了另一种方法：
浏览器下载：[https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-45.2.0/kube-prometheus-stack-45.2.0.tgz](https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-45.2.0/kube-prometheus-stack-45.2.0.tgz)

拷贝至 linux

```bash
$ tar -zxvf kube-prometheus-stack-45.2.0.tgz
kube-prometheus-stack/Chart.yaml
kube-prometheus-stack/Chart.lock
kube-prometheus-stack/values.yaml
kube-prometheus-stack/templates/NOTES.txt
kube-prometheus-stack/templates/_helpers.tpl
kube-prometheus-stack/templates/alertmanager/alertmanager.yaml
kube-prometheus-stack/templates/alertmanager/extrasecret.yaml
kube-prometheus-stack/templates/alertmanager/ingress.yaml
kube-prometheus-stack/templates/alertmanager/ingressperreplica.yaml
kube-prometheus-stack/templates/alertmanager/podDisruptionBudget.yaml
kube-prometheus-stack/templates/alertmanager/psp-role.yaml
kube-prometheus-stack/templates/alertmanager/psp-rolebinding.yaml
kube-prometheus-stack/templates/alertmanager/psp.yaml
kube-prometheus-stack/templates/alertmanager/secret.yaml
kube-prometheus-stack/templates/alertmanager/service.yaml
kube-prometheus-stack/templates/alertmanager/serviceaccount.yaml
kube-prometheus-stack/templates/alertmanager/servicemonitor.yaml
kube-prometheus-stack/templates/alertmanager/serviceperreplica.yaml
kube-prometheus-stack/templates/exporters/core-dns/service.yaml
kube-prometheus-stack/templates/exporters/core-dns/servicemonitor.yaml
kube-prometheus-stack/templates/exporters/kube-api-server/servicemonitor.yaml
kube-prometheus-stack/templates/exporters/kube-controller-manager/endpoints.yaml
kube-prometheus-stack/templates/exporters/kube-controller-manager/service.yaml
kube-prometheus-stack/templates/exporters/kube-controller-manager/servicemonitor.yaml
kube-prometheus-stack/templates/exporters/kube-dns/service.yaml
kube-prometheus-stack/templates/exporters/kube-dns/servicemonitor.yaml
kube-prometheus-stack/templates/exporters/kube-etcd/endpoints.yaml
kube-prometheus-stack/templates/exporters/kube-etcd/service.yaml
kube-prometheus-stack/templates/exporters/kube-etcd/servicemonitor.yaml
kube-prometheus-stack/templates/exporters/kube-proxy/endpoints.yaml
kube-prometheus-stack/templates/exporters/kube-proxy/service.yaml
kube-prometheus-stack/templates/exporters/kube-proxy/servicemonitor.yaml
kube-prometheus-stack/templates/exporters/kube-scheduler/endpoints.yaml
kube-prometheus-stack/templates/exporters/kube-scheduler/service.yaml
kube-prometheus-stack/templates/exporters/kube-scheduler/servicemonitor.yaml
kube-prometheus-stack/templates/exporters/kubelet/servicemonitor.yaml
kube-prometheus-stack/templates/grafana/configmap-dashboards.yaml
kube-prometheus-stack/templates/grafana/configmaps-datasources.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/alertmanager-overview.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/apiserver.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/cluster-total.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/controller-manager.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/etcd.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/grafana-overview.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/k8s-coredns.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/k8s-resources-cluster.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/k8s-resources-namespace.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/k8s-resources-node.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/k8s-resources-pod.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/k8s-resources-workload.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/k8s-resources-workloads-namespace.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/kubelet.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/namespace-by-pod.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/namespace-by-workload.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/node-cluster-rsrc-use.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/node-rsrc-use.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/nodes-darwin.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/nodes.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/persistentvolumesusage.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/pod-total.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/prometheus-remote-write.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/prometheus.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/proxy.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/scheduler.yaml
kube-prometheus-stack/templates/grafana/dashboards-1.14/workload-total.yaml
kube-prometheus-stack/templates/prometheus/_rules.tpl
kube-prometheus-stack/templates/prometheus/additionalAlertRelabelConfigs.yaml
kube-prometheus-stack/templates/prometheus/additionalAlertmanagerConfigs.yaml
kube-prometheus-stack/templates/prometheus/additionalPrometheusRules.yaml
kube-prometheus-stack/templates/prometheus/additionalScrapeConfigs.yaml
kube-prometheus-stack/templates/prometheus/clusterrole.yaml
kube-prometheus-stack/templates/prometheus/clusterrolebinding.yaml
kube-prometheus-stack/templates/prometheus/csi-secret.yaml
kube-prometheus-stack/templates/prometheus/extrasecret.yaml
kube-prometheus-stack/templates/prometheus/ingress.yaml
kube-prometheus-stack/templates/prometheus/ingressThanosSidecar.yaml
kube-prometheus-stack/templates/prometheus/ingressperreplica.yaml
kube-prometheus-stack/templates/prometheus/podDisruptionBudget.yaml
kube-prometheus-stack/templates/prometheus/podmonitors.yaml
kube-prometheus-stack/templates/prometheus/prometheus.yaml
kube-prometheus-stack/templates/prometheus/psp-clusterrole.yaml
kube-prometheus-stack/templates/prometheus/psp-clusterrolebinding.yaml
kube-prometheus-stack/templates/prometheus/psp.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/alertmanager.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/config-reloaders.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/etcd.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/general.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/k8s.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kube-apiserver-availability.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kube-apiserver-burnrate.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kube-apiserver-histogram.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kube-apiserver-slos.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kube-prometheus-general.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kube-prometheus-node-recording.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kube-scheduler.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kube-state-metrics.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubelet.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-apps.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-resources.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-storage.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-system-apiserver.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-system-controller-manager.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-system-kube-proxy.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-system-kubelet.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-system-scheduler.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/kubernetes-system.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/node-exporter.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/node-exporter.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/node-network.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/node.rules.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/prometheus-operator.yaml
kube-prometheus-stack/templates/prometheus/rules-1.14/prometheus.yaml
kube-prometheus-stack/templates/prometheus/service.yaml
kube-prometheus-stack/templates/prometheus/serviceThanosSidecar.yaml
kube-prometheus-stack/templates/prometheus/serviceThanosSidecarExternal.yaml
kube-prometheus-stack/templates/prometheus/serviceaccount.yaml
kube-prometheus-stack/templates/prometheus/servicemonitor.yaml
kube-prometheus-stack/templates/prometheus/servicemonitorThanosSidecar.yaml
kube-prometheus-stack/templates/prometheus/servicemonitors.yaml
kube-prometheus-stack/templates/prometheus/serviceperreplica.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/clusterrole.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/clusterrolebinding.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/job-createSecret.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/job-patchWebhook.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/networkpolicy-createSecret.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/networkpolicy-patchWebhook.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/psp.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/role.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/rolebinding.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/job-patch/serviceaccount.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/mutatingWebhookConfiguration.yaml
kube-prometheus-stack/templates/prometheus-operator/admission-webhooks/validatingWebhookConfiguration.yaml
kube-prometheus-stack/templates/prometheus-operator/aggregate-clusterroles.yaml
kube-prometheus-stack/templates/prometheus-operator/certmanager.yaml
kube-prometheus-stack/templates/prometheus-operator/clusterrole.yaml
kube-prometheus-stack/templates/prometheus-operator/clusterrolebinding.yaml
kube-prometheus-stack/templates/prometheus-operator/deployment.yaml
kube-prometheus-stack/templates/prometheus-operator/networkpolicy.yaml
kube-prometheus-stack/templates/prometheus-operator/psp-clusterrole.yaml
kube-prometheus-stack/templates/prometheus-operator/psp-clusterrolebinding.yaml
kube-prometheus-stack/templates/prometheus-operator/psp.yaml
kube-prometheus-stack/templates/prometheus-operator/service.yaml
kube-prometheus-stack/templates/prometheus-operator/serviceaccount.yaml
kube-prometheus-stack/templates/prometheus-operator/servicemonitor.yaml
kube-prometheus-stack/templates/prometheus-operator/verticalpodautoscaler.yaml
kube-prometheus-stack/templates/thanos-ruler/extrasecret.yaml
kube-prometheus-stack/templates/thanos-ruler/ingress.yaml
kube-prometheus-stack/templates/thanos-ruler/podDisruptionBudget.yaml
kube-prometheus-stack/templates/thanos-ruler/ruler.yaml
kube-prometheus-stack/templates/thanos-ruler/service.yaml
kube-prometheus-stack/templates/thanos-ruler/serviceaccount.yaml
kube-prometheus-stack/templates/thanos-ruler/servicemonitor.yaml
kube-prometheus-stack/.helmignore
kube-prometheus-stack/CONTRIBUTING.md
kube-prometheus-stack/README.md
kube-prometheus-stack/crds/crd-alertmanagerconfigs.yaml
kube-prometheus-stack/crds/crd-alertmanagers.yaml
kube-prometheus-stack/crds/crd-podmonitors.yaml
kube-prometheus-stack/crds/crd-probes.yaml
kube-prometheus-stack/crds/crd-prometheuses.yaml
kube-prometheus-stack/crds/crd-prometheusrules.yaml
kube-prometheus-stack/crds/crd-servicemonitors.yaml
kube-prometheus-stack/crds/crd-thanosrulers.yaml
kube-prometheus-stack/charts/grafana/Chart.yaml
kube-prometheus-stack/charts/grafana/values.yaml
kube-prometheus-stack/charts/grafana/templates/NOTES.txt
kube-prometheus-stack/charts/grafana/templates/_helpers.tpl
kube-prometheus-stack/charts/grafana/templates/_pod.tpl
kube-prometheus-stack/charts/grafana/templates/clusterrole.yaml
kube-prometheus-stack/charts/grafana/templates/clusterrolebinding.yaml
kube-prometheus-stack/charts/grafana/templates/configmap-dashboard-provider.yaml
kube-prometheus-stack/charts/grafana/templates/configmap.yaml
kube-prometheus-stack/charts/grafana/templates/dashboards-json-configmap.yaml
kube-prometheus-stack/charts/grafana/templates/deployment.yaml
kube-prometheus-stack/charts/grafana/templates/extra-manifests.yaml
kube-prometheus-stack/charts/grafana/templates/headless-service.yaml
kube-prometheus-stack/charts/grafana/templates/hpa.yaml
kube-prometheus-stack/charts/grafana/templates/image-renderer-deployment.yaml
kube-prometheus-stack/charts/grafana/templates/image-renderer-hpa.yaml
kube-prometheus-stack/charts/grafana/templates/image-renderer-network-policy.yaml
kube-prometheus-stack/charts/grafana/templates/image-renderer-service.yaml
kube-prometheus-stack/charts/grafana/templates/image-renderer-servicemonitor.yaml
kube-prometheus-stack/charts/grafana/templates/ingress.yaml
kube-prometheus-stack/charts/grafana/templates/networkpolicy.yaml
kube-prometheus-stack/charts/grafana/templates/poddisruptionbudget.yaml
kube-prometheus-stack/charts/grafana/templates/podsecuritypolicy.yaml
kube-prometheus-stack/charts/grafana/templates/pvc.yaml
kube-prometheus-stack/charts/grafana/templates/role.yaml
kube-prometheus-stack/charts/grafana/templates/rolebinding.yaml
kube-prometheus-stack/charts/grafana/templates/secret-env.yaml
kube-prometheus-stack/charts/grafana/templates/secret.yaml
kube-prometheus-stack/charts/grafana/templates/service.yaml
kube-prometheus-stack/charts/grafana/templates/serviceaccount.yaml
kube-prometheus-stack/charts/grafana/templates/servicemonitor.yaml
kube-prometheus-stack/charts/grafana/templates/statefulset.yaml
kube-prometheus-stack/charts/grafana/templates/tests/test-configmap.yaml
kube-prometheus-stack/charts/grafana/templates/tests/test-podsecuritypolicy.yaml
kube-prometheus-stack/charts/grafana/templates/tests/test-role.yaml
kube-prometheus-stack/charts/grafana/templates/tests/test-rolebinding.yaml
kube-prometheus-stack/charts/grafana/templates/tests/test-serviceaccount.yaml
kube-prometheus-stack/charts/grafana/templates/tests/test.yaml
kube-prometheus-stack/charts/grafana/.helmignore
kube-prometheus-stack/charts/grafana/README.md
kube-prometheus-stack/charts/grafana/ci/default-values.yaml
kube-prometheus-stack/charts/grafana/ci/with-affinity-values.yaml
kube-prometheus-stack/charts/grafana/ci/with-dashboard-json-values.yaml
kube-prometheus-stack/charts/grafana/ci/with-dashboard-values.yaml
kube-prometheus-stack/charts/grafana/ci/with-extraconfigmapmounts-values.yaml
kube-prometheus-stack/charts/grafana/ci/with-image-renderer-values.yaml
kube-prometheus-stack/charts/grafana/ci/with-persistence.yaml
kube-prometheus-stack/charts/grafana/dashboards/custom-dashboard.json
kube-prometheus-stack/charts/kube-state-metrics/Chart.yaml
kube-prometheus-stack/charts/kube-state-metrics/values.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/NOTES.txt
kube-prometheus-stack/charts/kube-state-metrics/templates/_helpers.tpl
kube-prometheus-stack/charts/kube-state-metrics/templates/clusterrolebinding.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/deployment.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/kubeconfig-secret.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/pdb.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/podsecuritypolicy.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/psp-clusterrole.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/psp-clusterrolebinding.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/rbac-configmap.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/role.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/rolebinding.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/service.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/serviceaccount.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/servicemonitor.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/stsdiscovery-role.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/stsdiscovery-rolebinding.yaml
kube-prometheus-stack/charts/kube-state-metrics/templates/verticalpodautoscaler.yaml
kube-prometheus-stack/charts/kube-state-metrics/.helmignore
kube-prometheus-stack/charts/kube-state-metrics/README.md
kube-prometheus-stack/charts/prometheus-node-exporter/Chart.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/values.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/NOTES.txt
kube-prometheus-stack/charts/prometheus-node-exporter/templates/_helpers.tpl
kube-prometheus-stack/charts/prometheus-node-exporter/templates/daemonset.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/endpoints.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/psp-clusterrole.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/psp-clusterrolebinding.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/psp.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/service.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/serviceaccount.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/servicemonitor.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/templates/verticalpodautoscaler.yaml
kube-prometheus-stack/charts/prometheus-node-exporter/.helmignore
kube-prometheus-stack/charts/prometheus-node-exporter/README.md
kube-prometheus-stack/charts/prometheus-node-exporter/ci/port-values.yaml
```

执行：


```bash
helm upgrade prometheus ./kube-prometheus-stack \
--namespace prometheus  --create-namespace --install \
--set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \
--set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false
```

在上面的安装命令中，我使用 `--set` 对安装参数进行了配置，这是为了让它后续能够顺利获取到 `Ingress-Nginx` 的监控指标。

然后，你需要等待 `prometheus` 命名空间下的工作负载处于就绪状态。


```bash
$ kubectl wait --for=condition=Ready pods --all -n prometheus
pod/alertmanager-prometheus-kube-prometheus-alertmanager-0 condition met
pod/prometheus-grafana-64b6c46fb5-6hz2z condition met
pod/prometheus-kube-prometheus-operator-696cc64986-pv9rg condition met
pod/prometheus-kube-state-metrics-649f8795d4-glbcq condition met
pod/prometheus-prometheus-kube-prometheus-prometheus-0 condition met
pod/prometheus-prometheus-node-exporter-mqnrw condition met
```
到这里，Prometheus 就部署完成了。

### 4.5 配置 Ingress-Nginx 和 ServiceMonitor
为了让 Prometheus 能够顺利地获取到 HTTP 请求指标，我们需要打开 `Ingress-Nginx` Metric 指标端口。首先需要为 `Ingress-Nginx Deployment` 添加容器的指标端口，你可以执行下面的命令来完成。


```bash
$ kubectl patch deployment ingress-nginx-controller -n ingress-nginx --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/ports/-", "value": {"name": "prometheus","containerPort":10254}}]'
deployment.apps/ingress-nginx-controller patched
```
然后，为 `Ingrss-Nginx Service` 添加指标端口。

```bash
$ kubectl patch service ingress-nginx-controller -n ingress-nginx --type='json' -p='[{"op": "add", "path": "/spec/ports/-", "value": {"name": "prometheus","port":10254,"targetPort":"prometheus"}}]'
service/ingress-nginx-controller patched
```
最后，为了让 Prometheus 能够抓取到 `Ingress-Nginx` 指标，我们还需要创建 `ServiceMonitor` 对象，它可以为 Prometheus 配置指标获取的策略。将下面的内容保存为 `servicemonitor.yaml` 文件。

```bash
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-ingress-controller-metrics
  namespace: prometheus
  labels:
    app: nginx-ingress
    release: prometheus-operator
spec:
  endpoints:
  - interval: 10s
    port: prometheus
  selector:
    matchLabels:
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  namespaceSelector:
    matchNames:
    - ingress-nginx
```
然后，通过 kubectl 将它部署到集群内。

```bash
$ kubectl apply -f servicemonitor.yaml
servicemonitor.monitoring.coreos.com/nginx-ingress-controller-metrics created
```
当把 `ServiceMonitor` 应用到集群后，Prometheus 会按照标签来匹配 `Ingress-Nginx Pod`，并且会每 10s 主动拉取一次指标数据，并保存到 Prometheus 时序数据库中。

### 4.6 验证 Ingress-Nginx 指标
接下来，我们验证 Prometheus 是否已经成功获取到了 `Ingress-Nginx` 指标，这将决定自动金丝雀分析是否能成功获取到数据。我们可以进入 Prometheus 控制台验证是否成功获取了 `Ingress-Nginx` 指标。首先，使用 `kubectl port-forward` 命令将 Prometheus 转发到本地。


```bash
$ kubectl port-forward service/prometheus-kube-prometheus-prometheus 9090:9090 -n prometheus
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090 
```
接下来，使用浏览器打开 `http://127.0.0.1:9090` 进入控制台，在搜索框中输入 `nginx_ingress`，如果出现一系列指标，则说明 Prometheus 和 `Ingress-Nginx` 已经配置完成，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/e49b8674538aae60fee9415272c6eb9a.png)

## 5. 自动渐进式交付实验
在，所有的准备工作都已经完成了，接下来我们进行自动渐进式交付实验。让我们重新回忆一下我在前面提到的这张流程图。
![](https://i-blog.csdnimg.cn/blog_migrate/beda11d4817f1ed8daa61ab08ee5fcdb.png)
在实验过程过程中，我会按照这张流程图分别进行两个实验。

- 自动渐进式交付成功（图中①号链路）。
- 自动渐进式交付失败（图中②号链路）。

### 5.1 自动渐进式交付成功
接下来，我们进行自动渐进式交付成功的实验。要开始实验，只要更新 Rollout 对象的镜像版本即可。在[第 25 讲金丝雀发布](https://blog.csdn.net/xixihahalelehehe/article/details/129120109)中，我提到了编辑 Rollout 对象并通过 kubectl apply 的方法来更新镜像版本。这节课，我们使用另一种更新镜像的方法，通过 `Argo Rollout kubectl` 插件来更新镜像。


```bash
$ kubectl argo rollouts set image canary-demo canary-demo=argoproj/rollouts-demo:green
rollout "canary-demo" image updated
```
接下来，使用浏览器打开 `http://progressive.auto` 返回应用，过一会儿看到绿色方块开始出现，流量占比约为 20%，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/019f3ec312dc548b96d9b5fd65e0af22.png)
现在，你可以尝试打开 Argo Rollout 控制台。


```bash
$ kubectl argo rollouts dashboard
INFO[0000] Argo Rollouts Dashboard is now available at http://localhost:3100/rollouts
```
使用浏览器访问 `http://localhost:3100/rollouts` 进入控制台，观察自动渐进式交付过程。可以看到目前处在 20% 金丝雀流量的下一阶段，也就是暂停 2 分钟的阶段。

![](https://i-blog.csdnimg.cn/blog_migrate/6f048f989f5da418b67bd2f2ce84f483.png)
2 分钟后，将进入到 40% 金丝雀流量阶段，从这个阶段开始，自动金丝雀分析开始工作，直到最后金丝雀发布完成，金丝雀环境提升为了生产环境，这时自动分析也完成了，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/97c6ec605960ae824d7b547583c28e46.png)
到这里，一次完整的自动渐进式交付就完成了。


### 5.2 自动渐进式交付失败
在上面的实验中，由于应用返回的 HTTP 状态码都是 200 ，所以金丝雀分析自然是会成功的。接下来，我们来尝试进行自动渐进式交付失败的实验。经过了自动渐进式交付成功的实验之后，当前生产环境中的镜像为 `argoproj/rollouts-demo:green`，我们继续使用 Argo Rollout kubectl 插件来更新镜像，并将镜像版本修改为 `yellow` 版本。


```bash
$ kubectl argo rollouts set image canary-demo canary-demo=argoproj/rollouts-demo:yellow
rollout "canary-demo" image updated
```

接下来，重新返回 `http://progressive.auto` 打开应用，等待一段时间后，你会看到请求开始出现黄色方块，如下图所示。
![](https://i-blog.csdnimg.cn/blog_migrate/746807eed0bbc713e2943d85d3c2e045.png)
接下来，我们让应用返回错误的 HTTP 状态码。你可以滑动界面上的 ERROR 滑动块，将错误率设置为 50%，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/cc8e93aa454388988fc152be2be3774d.png)
现在，你会在黄色方块中看到带有红色描边的方块，这代表本次请求返回的 HTTP 状态码不等于 `200`，说明我们成功控制了一部分请求返回错误。2 分钟后，金丝雀发布会进入到 40% 流量的阶段，此时自动分析将开始进行。现在，我们进入 `Argo Rollout` 控制台。


```bash
$ kubectl argo rollouts dashboard
INFO[0000] Argo Rollouts Dashboard is now available at http://localhost:3100/rollouts
```
使用浏览器打开 `http://localhost:3100/rollouts`，进入发布详情，等待一段时间后，金丝雀分析将失败，如下图所示。

![](https://i-blog.csdnimg.cn/blog_migrate/e39f1f3d866118ec5ea149adac227e2b.png)
此时，Argo Rollout 将执行自动回滚操作，这时候重新返回 `http://progressive.auto` 打开应用，你会看到黄色方块的流量消失，所有请求被绿色方块取代，说明已经完成回滚了，如下图所示。


到这里，一次完整的渐进式交付失败实验就完成了。


## 6. 总结
这节课，我为你介绍了什么是渐进式交付，以及如何借助 ArgoCD 实施渐进式交付，它的发布流程和我们在 25 讲提到的金丝雀发布非常类似。

不同的是，渐进式交付在金丝雀发布的过程中加入了自动金丝雀分析，它可以验证新版本在生产环境中的表现，而这是单纯的金丝雀发布所无法实现的。借助渐进式交付，我们可以在发布过程通过指标实时分析金丝雀环境，兼顾发布的安全性和效率。

值得注意的是，为了能够查询到示例应用的 HTTP 指标，我开启了 Ingress-Nginx 的指标开关，这样所有经过 Ingress-Nginx 的流量都会被记录下来，结合 Prometheus ServiceMonitor 实现了 HTTP 请求指标的采集。

此外，为了让 ArgoCD 在渐进式交付时顺利运行金丝雀分析，我们还需要创建 AnalysisTemplate 对象，它实际上是 PromQL 编写的查询语句，ArgoCD 在交付过程中会用这条语句去 Prometheus 查询，并将返回的结果和预定义的阈值进行对比，以此控制渐进式交付应该继续进行还是回滚。

在实际的业务场景中，如果你希望验证多个维度的指标，你可以创建多个 AnalysisTemplate 并将它配置到 Rollout 对象中，进一步提高分析的可靠性。另外，你还可以在金丝雀发布的 steps 阶段里配置“内联”的分析步骤，比如在金丝雀环境 20% 和 40% 流量阶段的下一阶段分别运行不同的金丝雀分析，具体配置方法你可以参考[这份文档](https://argoproj.github.io/argo-rollouts/features/analysis/)。

这节课我并没有深入介绍 Prometheus。在生产环境下，我们通常会使用它来构建强大的监控和告警系统，我将在后续的课程为你详细介绍。


## 7. 思考

最后，给你留一道思考题吧。当自动分析失败导致回滚时，是否有必要将镜像版本回写到 GitOps 应用定义的仓库中呢？

