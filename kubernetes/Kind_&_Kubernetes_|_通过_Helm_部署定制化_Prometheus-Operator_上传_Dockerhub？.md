![](https://img-blog.csdnimg.cn/7f9770d0330f4561ab081a46bffc3e26.png)



##  1. 环境准备
- [安装系统 Centos 8.2](https://blog.csdn.net/xixihahalelehehe/article/details/127616480)
- [初始化  Centos 8.2](https://blog.csdn.net/xixihahalelehehe/article/details/127641551)
- [安装 Podman](https://blog.csdn.net/xixihahalelehehe/article/details/127953530)，它是 Podman 最初由红帽工程师联合开源社区一同开发的无守护进程的下一代容器管理工具，即 Docker 替代者。
- [安装 Kubectl](https://kubernetes.io/docs/tasks/tools/) 
 [安装 Kind](https://blog.csdn.net/xixihahalelehehe/article/details/121968488)， 它是一个 [Kubernetes 孵化项目](https://www.cncf.io/projects/)，一套开箱即用的 Kubernetes 环境搭建方案。顾名思义，就是将 Kubernetes 所需要的所有组件，全部部署在一个 Docker 容器中。

```bash
curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.17.0/kind-linux-amd64
chmod +x ./kind
mv ./kind /usr/local/bin/kind
```
- [安装 helm 命令](https://helm.sh/docs/intro/install/)

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
或者

```bash
$ wget https://get.helm.sh/helm-v3.11.0-linux-amd64.tar.gz
$ tar -xzvf helm-v3.11.0-linux-amd64.tar.gz
$ cp linux-amd64/helm /usr/local/bin/
$ helm version
version.BuildInfo{Version:"v3.11.0", GitCommit:"d14138609b01886f544b2025f5000351c9eb092e", GitTreeState:"clean", GoVersion:"go1.17.5"}
```
## 2. Kind 部署 Kubernetes
- `config.yaml`
```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
```
创建 K8s 集群:

```bash
$ kind create cluster --config config.yaml
enabling experimental podman provider
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

### 2.1 安装 Ingress-nginx 组件

```bash
kubectl create -f https://ghproxy.com/https://raw.githubusercontent.com/Ghostwritten/resource/main/ingress-nginx/ingress-nginx.yaml
```

### 2.2 安装 Metric Server 组件
系统资源的采集均使用Metrics-Server服务，可以通过Metrics-Server服务采集节点和Pod的内存、磁盘、CPU和网络的使用率等信息。Metric Server组件是实现服务自动扩容不可或缺的组件。

```bash
kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/Ghostwritten/resource/main/metrics/metrics.yaml
```
等待 Metric 工作负载就绪

```bash
kubectl wait deployment -n kube-system metrics-server --for condition=Available=True --timeout=90s
```
Metric Server 就绪后，我们通过 `kubectl autoscale` 命令来为 Deployment 创建自动扩容策略。但这篇不是重点。




## 3.  helm 快速安装 Prometheus-Operator 
默认安装方式：

```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
$ helm upgrade prometheus prometheus-community/kube-prometheus-stack \--namespace prometheus  --create-namespace --install \--set prometheusOperator.admissionWebhooks.patch.image.registry=docker.io --set prometheusOperator.admissionWebhooks.patch.image.repository=dyrnq/kube-webhook-certgen \--set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \--set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false 

Release "prometheus" does not exist. Installing it now.
NAME: prometheus
LAST DEPLOYED: Tue Mar 28 16:22:32 2023
NAMESPACE: prometheus
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace prometheus get pods -l "release=prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```
虽然部署成功了，但是查看 pod 报以下错误：

```bash
$k get pod -n prometheus
NAME                                                     READY   STATUS         RESTARTS      AGE
alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running        1 (33s ago)   36s
prometheus-grafana-6f77bc5bc9-9qkf6                      3/3     Running        0             41s
prometheus-kube-prometheus-operator-6d7bf45ccc-mlnzm     1/1     Running        0             41s
prometheus-kube-state-metrics-678b896dcf-ntz5r           0/1     ErrImagePull   0             41s
prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running        0             33s
prometheus-prometheus-node-exporter-4dxf5                1/1     Running        0             41s

$ k describe pod -n prometheus prometheus-kube-state-metrics-678b896dcf-2qvwt
.......
  Events:
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Normal   Scheduled  36s   default-scheduler  Successfully assigned prometheus/prometheus-kube-state-metrics-678b896dcf-2qvwt to kind-control-plane
  Normal   Pulling    36s   kubelet            Pulling image "registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0"
  Warning  Failed     5s    kubelet            Failed to pull image "registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0": rpc error: code = Unknown desc = failed to pull and unpack image "registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0": failed to resolve reference "registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0": failed to do request: Head "https://asia-east1-docker.pkg.dev/v2/k8s-artifacts-prod/images/kube-state-metrics/kube-state-metrics/manifests/v2.8.0": dial tcp 142.251.170.82:443: i/o timeout
  Warning  Failed     5s    kubelet            Error: ErrImagePull
  Normal   BackOff    5s    kubelet            Back-off pulling image "registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0"
  Warning  Failed     5s    kubelet            Error: ImagePullBackOff
........
```

没有真正的全球互联网，我们无法拉取 `registry.k8s.io`。需要对重新定制自己的 prometheus helm charts。

### 3.1 获取镜像方法
但由于隐形墙的原因，我们无法下载拉取以下：

- `registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0`

所以，你可以获取`registry.k8s.io`的方法：
- 购买国外 `VPS` 拉取镜像推送至 `DockerHub`
- 通过配置代理拉取镜像，需施展要魔法技能。
- 去 [DockerHub](https://hub.docker.com/)、[quay.io](https://quay.io/) 寻找他人推送的对应版本的镜像。
- [利用 Github Action 实现镜像自动迁移](https://github.com/Ghostwritten/hub-mirror)。fork来自@[togettoyou](https://github.com/togettoyou/hub-mirror/issues/408) YYDS

这里我利用 Github Action 实现镜像自动迁移。


### 3.2 定制内容
- 下载最新版本(20230328) [kube-prometheus-stack-45.8.0.tgz](https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-45.8.0/kube-prometheus-stack-45.8.0.tgz)

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
第二步：为了和官方 prometheus helm charts 做一个区分需要修改一下 `helm charts`的名字。打开`vim Chart.yaml`

```bash
 50 name: kube-prometheus-stack
```
修改为：
```bash
 50 name: ghostwritten-kube-prometheus-stack
```

### 3.3 打包

```bash
$ cd ../
$ helm package ./kube-prometheus-stack
Successfully packaged chart and saved it to: /root/ghostwritten-kube-prometheus-stack-45.2.0.tgz
```

### 3.4  推送 Helm Package 至 DockerHub

> 如何上传 [Github](https://github.com/) 或者 [Harbor](https://goharbor.io/) 请参考这篇文章：[kind & kubernetes 集群内如何通过 helm 部署定制化 Prometheus-Operator？](https://ghostwritten.blog.csdn.net/article/details/129247543)


创建 `Access token`

![](https://img-blog.csdnimg.cn/2e7ef8c68dda4809b6b3c2e7ebb8ec18.png)
![](https://img-blog.csdnimg.cn/ba83bbd6c7454174a63946d28dc5354f.png)
![](https://img-blog.csdnimg.cn/a30375a22fe744a3b814f931dc2c58dc.png)

```bash

$ export REG_PAT=<token>

$  echo $REG_PAT | helm registry login registry-1.docker.io -u ghostwritten --password-stdin
Login Succeeded

$ helm push ghostwritten-kube-prometheus-stack-45.8.0.tgz oci://registry-1.docker.io/ghostwritten
Pushed: registry-1.docker.io/ghostwritten/ghostwritten-kube-prometheus-stack:45.8.0
Digest: sha256:20bb9e1e22d5bb422bd285a829061b58458adde6ebc26f5ee81cca8f31ef0729
```
登陆 [DockerHub](https://hub.docker.com/) 验证是否上传成功。
![](https://img-blog.csdnimg.cn/c5d225f1b2fc4bb094c4034da7a4fbd1.png)


## 4. helm 安装定制化的 ghostwritten-kube-prometheus-stack
我这里选择从刚刚上传的公共 [dockerhub](https://github.com/users/Ghostwritten/packages/container/package/helm/ghostwritten-kube-prometheus-stack)上的`ghostwritten-kube-prometheus-stack`包进行安装。


```bash
helm upgrade prometheus oci://registry-1.docker.io/ghostwritten/ghostwritten-kube-prometheus-stack  --version 45.8.0 \--namespace prometheus  --create-namespace --install \--set prometheusOperator.admissionWebhooks.patch.image.registry=docker.io --set prometheusOperator.admissionWebhooks.patch.image.repository=dyrnq/kube-webhook-certgen \--set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \--set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false 
```

输出：

```bash
Release "prometheus" does not exist. Installing it now.
Pulled: registry-1.docker.io/ghostwritten/ghostwritten-kube-prometheus-stack:45.8.0
Digest: sha256:20bb9e1e22d5bb422bd285a829061b58458adde6ebc26f5ee81cca8f31ef0729
NAME: prometheus
LAST DEPLOYED: Tue Mar 28 17:28:51 2023
NAMESPACE: prometheus
STATUS: deployed
REVISION: 1
NOTES:
ghostwritten-kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace prometheus get pods -l "release=prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```
接下来，等待 Prometheus 所有组件处于 Ready 状态。

```bash
$ kubectl wait --for=condition=Ready pods --all -n prometheus --timeout=300s
pod/alertmanager-prometheus-ghostwritten-ku-alertmanager-0 condition met
pod/prometheus-ghostwritten-ku-operator-67d79fc6f7-cx5kk condition met
pod/prometheus-grafana-79486fb946-mhng2 condition met
pod/prometheus-kube-state-metrics-85c6bbbcb-c7kms condition met
pod/prometheus-prometheus-ghostwritten-ku-prometheus-0 condition met
pod/prometheus-prometheus-node-exporter-5bdrl condition met

$  k get pods -n prometheus
NAME                                                     READY   STATUS    RESTARTS      AGE
alertmanager-prometheus-ghostwritten-ku-alertmanager-0   2/2     Running   1 (72s ago)   75s
prometheus-ghostwritten-ku-operator-67d79fc6f7-cx5kk     1/1     Running   0             79s
prometheus-grafana-79486fb946-mhng2                      3/3     Running   0             79s
prometheus-kube-state-metrics-85c6bbbcb-c7kms            1/1     Running   0             79s
prometheus-prometheus-ghostwritten-ku-prometheus-0       2/2     Running   0             72s
prometheus-prometheus-node-exporter-5bdrl                1/1     Running   0             79s
```
当所有 Pod 准备就绪后，代表 `helm` 部署的最新版本 `kube-prometheus-stack` 已经安装完成了。

