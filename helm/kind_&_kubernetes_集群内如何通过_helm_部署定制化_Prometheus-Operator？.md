![](https://i-blog.csdnimg.cn/blog_migrate/a977f8959afbf3255656c54bb1e4c900.png)


## 1. Prometheus 简介
[Prometheus](https://github.com/prometheus) 是由前 `Google` 工程师从 2012 年开始在 [Soundcloud](https://soundcloud.com/) 以开源软件的形式进行研发的系统监控和告警工具包，自此以后，许多公司和组织都采用了 `Prometheus` 作为监控告警工具。`Prometheus` 的开发者和用户社区非常活跃，它现在是一个独立的开源项目，可以独立于任何公司进行维护。为了证明这一点，Prometheus 于 2016 年 5 月加入 [CNCF](https://www.cncf.io/) 基金会，成为继 Kubernetes 之后的第二个 CNCF 托管项目。

## 2. Prometheus 优势
- 由指标名称和和键/值对标签标识的时间序列数据组成的多维数据模型。
- 强大的查询语言 PromQL。
- 不依赖分布式存储；单个服务节点具有自治能力。
- 时间序列数据是服务端通过 HTTP 协议主动拉取获得的。
- 也可以通过中间网关来推送时间序列数据。
- 可以通过静态配置文件或服务发现来获取监控目标。
- 支持多种类型的图表和仪表盘

## 3. Prometheus 架构图
![](https://i-blog.csdnimg.cn/blog_migrate/ea5eab060622a21c6e3e5bc185a0456e.png)

组件说明：
- `Prometheus Server`: 用于收集和存储时间序列数据；
- `Client Library`: 客户端库，为需要监控的服务生成相应的 metrics 并暴露给 Prometheus server；
- `PushGateway`: pushgateway是采用被动推送来获取监控数据的 prometheus 插件，它可以单独运行在任何节点上，并不一定要运行在被监控的客户端。而后通过用户自定义编写的脚本把需要监控的数据发送给 pushgateway，pushgateway再将数据推送给prometheus server；
- `Exporters`: 负责监控机器运行状态，提供被监控组件信息的 HTTP 接口被叫做 exporter；
- `Alertmanager`: 从 Prometheus server 端接收到 alerts 后，会进行去除重复数据，分组，并路由到对收的接受方式，发出报警；
- `Grafana`：一个监控仪表系统，它是由 Grafana Labs 公司开源的的一个系统监测工具，它可以大大帮助我们简化监控的复杂度，我们只需要提供需要监控的数据，它就可以帮助生成各种可视化仪表，同时它还有报警功能，可以在系统出现问题时发出通知。

## 4. Prometheus-Operator 简介

- [Kubernetes Operator](https://www.redhat.com/en/topics/containers/what-is-a-kubernetes-operator) 是由 [CoreOS](https://www.redhat.com/en/technologies/cloud-computing/openshift) 公司开发的，用来扩展 Kubernetes API，特定的应用程序控制器，它用来创建、配置和管理复杂的有状态应用，如数据库、缓存和监控系统。Operator基于 Kubernetes 的资源和控制器概念之上构建，但同时又包含了应用程序特定的一些专业知识，比如创建一个数据库的Operator，则必须对创建的数据库的各种运维方式非常了解，创建Operator的关键是CRD（自定义资源）的设计。
- [Kubernetes CRD](https://thenewstack.io/kubernetes-crds-what-they-are-and-why-they-are-useful/) 是对 Kubernetes API 的扩展，Kubernetes 中的每个资源都是一个 API 对象的集合，例如我们在 YAML文件里定义的那些spec都是对 Kubernetes 中的资源对象的定义，所有的自定义资源可以跟 Kubernetes 中内建的资源一样使用 kubectl 操作。
- [Prometheus Operator](https://prometheus-operator.dev/docs/prologue/introduction/) 提供 Kubernetes 原生部署和管理`Prometheus`及相关监控组件。该项目的目的是为 Kubernetes 集群简化和自动化基于 Prometheus 的监控堆栈的配置。
Prometheus Operator 特性：

  - Kubernetes 自定义资源：使用 Kubernetes 自定义资源部署和管理 Prometheus、Alertmanager 及相关组件。
  - 简化的部署配置：配置 Prometheus 的基础知识，例如版本、持久性、保留策略和本地 Kubernetes 资源的副本。
  - Prometheus Target Configuration：根据熟悉的Kubernetes标签查询，自动生成监控目标配置；无需学习普罗米修斯特定的配置语言。

## 5. Prometheus-Operator 架构图
![](https://i-blog.csdnimg.cn/blog_migrate/4cb43ac5811c17eec8f490c58341d6af.png)

这篇文章我将在kind部署的kubernets集群内通过 `helm` 工具安装定制化的 `Prometheus-Operator` 。

##  6. 环境准备
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
## 7. Kind 部署 Kubernetes
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

### 7.1 安装 Ingress-nginx 组件

```bash
kubectl create -f https://ghproxy.com/https://raw.githubusercontent.com/Ghostwritten/resource/main/ingress-nginx/ingress-nginx.yaml
```

### 7.2 安装 Metric Server 组件
系统资源的采集均使用Metrics-Server服务，可以通过Metrics-Server服务采集节点和Pod的内存、磁盘、CPU和网络的使用率等信息。Metric Server组件是实现服务自动扩容不可或缺的组件。

```bash
kubectl apply -f https://ghproxy.com/https://raw.githubusercontent.com/Ghostwritten/resource/main/metrics/metrics.yaml
```
等待 Metric 工作负载就绪

```bash
kubectl wait deployment -n kube-system metrics-server --for condition=Available=True --timeout=90s
```
Metric Server 就绪后，我们通过 `kubectl autoscale` 命令来为 Deployment 创建自动扩容策略。但这篇不是重点。




## 8.  helm 快速安装 Prometheus-Operator 
默认安装方式：

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
但我本地的 linux 环境会有没有“科学网络”，所以部署实际会卡住很长一段时间，最后报:`Error: failed pre-install: timed out waiting for the condition`,查看 pod 报以下错误：

```bash
$ k get pods -n prometheus
NAME                                                READY   STATUS             RESTARTS   AGE
prometheus-kube-prometheus-admission-create-7494p   0/1     ImagePullBackOff   0          6m20s

$ k get pods -n prometheus prometheus-kube-prometheus-admission-create-7494p -oyaml
.......
    state:
      waiting:
        message: Back-off pulling image "registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.3.0"
        reason: ImagePullBackOff
........
```

没有真正的全球互联网，我们无法拉取 `registry.k8s.io`。需要对重新定制自己的 prometheus helm charts

## 9. 定制 Prometheus-Operator  helm charts

### 9.1 获取镜像方法
但由于隐形墙的原因，我们无法下载拉取以下：
- `registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.3.0`
- `registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0`

所以，你可以获取`registry.k8s.io`的方法：
- 购买国外 `VPS` 拉取镜像推送至 `DockerHub`
- 通过配置代理拉取镜像，需施展要魔法技能。
- 去 [DockerHub](https://hub.docker.com/)、[quay.io](https://quay.io/) 寻找他人推送的对应版本的镜像。
- [利用 Github Action 实现镜像自动迁移](https://github.com/Ghostwritten/hub-mirror)。fork来自@[togettoyou](https://github.com/togettoyou/hub-mirror/issues/408) YYDS

这里我由于只是用于测试，直接从 `DockerHub` 获取他人已经推送好的镜像，当然，这种方法不敢保证别人已经做过改动，存在风险，生产谨慎使用。




### 9.2 定制内容
- 下载 [kube-prometheus-stack-45.2.0.tgz](https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-45.2.0/kube-prometheus-stack-45.2.0.tgz)

```bash
wget https://github.com/prometheus-community/helm-charts/releases/download/kube-prometheus-stack-45.2.0/kube-prometheus-stack-45.2.0.tgz
tar -zxvf kube-prometheus-stack-45.2.0.tgz
```
修改镜像版本
- `registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.3.0` 改为`docker.io/ghostwritten/kube-webhook-certgen:v1.3.0`
- `registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.8.0`改为`docker.io/ghostwritten/kube-state-metrics:v2.8.0`

具体操作：第一步：`cd kube-prometheus-stack`目录，`vim values.yaml`文件

```bash
1872         registry: registry.k8s.io
1873         repository: ingress-nginx/kube-webhook-certgen
1874         tag: v1.3.0
```

修改为：

```bash
1872         registry: docker.io
1873         repository: ghostwritten/kube-webhook-certgen
1874         tag: v1.3.0
```
第二步：`vim charts/kube-state-metrics/values.yaml`

```bash
  3 image:
  4   repository: registry.k8s.io/kube-state-metrics/kube-state-metrics
```
修改为：

```bash
  3 image:
  4   repository: docker.io/ghostwritten/kube-state-metrics
```

第三步：为了和官方 prometheus helm charts 做一个区分需要修改一下 `helm charts`的名字。打开`vim Chart.yaml`

```bash
 50 name: kube-prometheus-stack
```
修改为：
```bash
 50 name: ghostwritten-kube-prometheus-stack
```
### 9.3 打包

```bash
$ cd ../
$ helm package ./kube-prometheus-stack
Successfully packaged chart and saved it to: /root/ghostwritten-kube-prometheus-stack-45.2.0.tgz
```
### 9.4  推送 helm package 至 GitHub Package
登陆 `github` 创建一个`Token`，你可以在[这个链接](https://github.com/settings/tokens/new)创建，并勾选 `write:packages` 权限。
![](https://i-blog.csdnimg.cn/blog_migrate/c9a0da429c4be511cfce2d1bd537da2a.png)
点击“`Genarate token`”按钮生成 `Token` 并复制。
![](https://i-blog.csdnimg.cn/blog_migrate/073037acf3bc09c958fbfc7c31361b8e.png)

```bash
$ helm registry login -u ghostwritten https://ghcr.io
Password: <复制 github_token>
Login Succeeded
```
推送 

```bash
$ helm push ghostwritten-kube-prometheus-stack-45.2.0.tgz oci://ghcr.io/ghostwritten/helm
Pushed: ghcr.io/ghostwritten/helm/ghostwritten-kube-prometheus-stack:45.2.0
Digest: sha256:eecc4fcfc2dd3a65dedb692bfbfcd49d1e420e99e2b13aded22b136dae25e146
```
查看位置 `ghostwritten-kube-prometheus-stack-45.2.0.tgz` 存放位置
![](https://i-blog.csdnimg.cn/blog_migrate/98003c785c65f7ea3afe539fb9be51bc.png)
![](https://i-blog.csdnimg.cn/blog_migrate/72f2efe93650464b7968826cad7b3a2c.png)

### 9.5 推送 helm package 至私有 harbor 
- [Centos 部署 harbor 镜像仓库实践](https://ghostwritten.blog.csdn.net/article/details/127920005)

harbor 默认仓库内没有`Helm Charts`栏，需要执行以下命令即可：

```bash
docker-compose stop
./install.sh  --with-chartmuseum
```


拷贝证书：

```bash
scp -r /etc/docker/certs.d/harbor.ghostwritten.com root@192.168.10.29:/etc/containers/certs.d/
```
`/etc/hosts` 文件配置 `192.168.10.81 harbor.ghostwritten.com`
登陆

```bash
$ podman login -u admin -p Harbor12345  harbor.ghostwritten.com
Login Succeeded!

$ helm registry login --insecure harbor.ghostwritten.com
Username: admin
Password:
Login Succeeded
或者
$ helm registry login --insecure harbor.ghostwritten.com -u admin -p Harbor12345
WARNING: Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
```
跟配置 Docker 仓库一样，配置 Helm 仓库也得提前配置证书，首先进入 ca 签名目录

```bash
yum install ca-certificates
cp /etc/containers/certs.d/harbor.ghostwritten.com/ca.crt /etc/pki/ca-trust/source/anchors/
```
执行更新命令，使证书生效:

```bash
update-ca-trust extract 
```
harbor 界面新建一个项目，名字为 helm
![](https://i-blog.csdnimg.cn/blog_migrate/11491e58b1b114767783c664f6dc1d0e.png)
将此项目添加 Helm 仓库:

```bash
$ helm  repo add harbor --username admin --password Harbor12345 https://harbor.ghostwritten.com/chartrepo/helm
"harbor" has been added to your repositories
```
查看 helm 本地库
```bash
helm repo list
NAME                    URL
prometheus-community    https://prometheus-community.github.io/helm-charts
harbor                  https://harbor.ghostwritten.com/chartrepo/helm
```
helm 默认不支持推送仓库，需要安装 `helm-push` 插件

```bash
$ helm plugin install https://github.com/chartmuseum/helm-push
Downloading and installing helm-push v0.10.3 ...
https://github.com/chartmuseum/helm-push/releases/download/v0.10.3/helm-push_0.10.3_linux_amd64.tar.gz
Installed plugin: cm-push
```
推送 `ghostwritten-kube-prometheus-stack-45.2.0.tgz` 至 harbor 

```bash
$ helm cm-push ghostwritten-kube-prometheus-stack-45.2.0.tgz harbor
Pushing ghostwritten-kube-prometheus-stack-45.2.0.tgz to harbor...
Done.
```
已上传成功，效果图如下：
![](https://i-blog.csdnimg.cn/blog_migrate/67d763beab3ae00424576c87ccd75783.png)



## 10. helm 安装定制化的 kube-prometheus-stack
我这里选择从刚刚上传的公共 [github package](https://github.com/users/Ghostwritten/packages/container/package/helm/ghostwritten-kube-prometheus-stack)上的`ghostwritten-kube-prometheus-stack`包进行安装。
```bash
helm upgrade prometheus oci://ghcr.io/ghostwritten/helm/ghostwritten-kube-prometheus-stack \
--version 45.2.0  \
--namespace prometheus  --create-namespace --install \
--set prometheus.prometheusSpec.podMonitorSelectorNilUsesHelmValues=false \
--set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false
```
输出：

```bash
Release "prometheus" does not exist. Installing it now.
Pulled: ghcr.io/ghostwritten/helm/ghostwritten-kube-prometheus-stack:45.2.0
Digest: sha256:2d17d54b97cbd2ce45f34bf2d25b7d424cff43508054ea0232b7d2f485a5b7a2
NAME: prometheus
LAST DEPLOYED: Tue Feb 28 14:32:22 2023
NAMESPACE: prometheus
STATUS: deployed
REVISION: 1
NOTES:
ghostwritten-kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace prometheus get pods -l "release=prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```
查看 helm 部署应用列表

```bash
$ helm list -n prometheus
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART
                                APP VERSION
prometheus      prometheus      1               2023-02-28 14:32:22.526037959 +0800 CST deployed        ghostwritten-kube-prometheus-stack-45.2.0       v0.63.0
```


查看 pod 以及其他资源对象

```bash
$ kubectl --namespace prometheus get pods -l "release=prometheus"
NAME                                                   READY   STATUS    RESTARTS   AGE
prometheus-ghostwritten-ku-operator-7c89d9649c-hl5sx   1/1     Running   0          49s
prometheus-kube-state-metrics-bf46f46cc-944cr          1/1     Running   0          49s
prometheus-prometheus-node-exporter-jbv7n              1/1     Running   0          49s

$ k get all -n prometheus
NAME                                                         READY   STATUS    RESTARTS      AGE
pod/alertmanager-prometheus-ghostwritten-ku-alertmanager-0   2/2     Running   1 (88m ago)   88m
pod/prometheus-ghostwritten-ku-operator-7c89d9649c-fvd4x     1/1     Running   0             88m
pod/prometheus-grafana-6f77bc5bc9-mnsn5                      3/3     Running   0             88m
pod/prometheus-kube-state-metrics-bf46f46cc-dqbrm            1/1     Running   0             88m
pod/prometheus-prometheus-ghostwritten-ku-prometheus-0       2/2     Running   0             88m
pod/prometheus-prometheus-node-exporter-gkd29                1/1     Running   0             88m

NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
 AGE
service/alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   88m
service/prometheus-ghostwritten-ku-alertmanager   ClusterIP   10.96.193.144   <none>        9093/TCP
 88m
service/prometheus-ghostwritten-ku-operator       ClusterIP   10.96.112.10    <none>        443/TCP
 88m
service/prometheus-ghostwritten-ku-prometheus     ClusterIP   10.96.110.97    <none>        9090/TCP
 88m
service/prometheus-grafana                        ClusterIP   10.96.86.245    <none>        80/TCP
 88m
service/prometheus-kube-state-metrics             ClusterIP   10.96.205.51    <none>        8080/TCP
 88m
service/prometheus-operated                       ClusterIP   None            <none>        9090/TCP
 88m
service/prometheus-prometheus-node-exporter       ClusterIP   10.96.154.79    <none>        9100/TCP
 88m

NAME                                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/prometheus-prometheus-node-exporter   1         1         1       1            1           <none>          88m

NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-ghostwritten-ku-operator   1/1     1            1           88m
deployment.apps/prometheus-grafana                    1/1     1            1           88m
deployment.apps/prometheus-kube-state-metrics         1/1     1            1           88m

NAME                                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-ghostwritten-ku-operator-7c89d9649c   1         1         1       88m
replicaset.apps/prometheus-grafana-6f77bc5bc9                    1         1         1       88m
replicaset.apps/prometheus-kube-state-metrics-bf46f46cc          1         1         1       88m

NAME                                                                    READY   AGE
statefulset.apps/alertmanager-prometheus-ghostwritten-ku-alertmanager   1/1     88m
statefulset.apps/prometheus-prometheus-ghostwritten-ku-prometheus       1/1     88m
```




## 11. 域名访问 
当然如果我们想要在外网访问这两个服务的话可以通过创建对应的 Ingress 对象或者使用 NodePort 类型的 Service，使用 NodePort 类型的服务，编辑`prometheus-grafana` 、 `prometheus-k8sprometheus-ghostwritten-ku-prometheus` 、`prometheus-ghostwritten-ku-alertmanager`这三个 Service，将服务类型（type）更改为 `NodePort`。但我更喜欢通过域名访问。

编写 ` prometheus-grafana-alertmanager-ingress.yaml`
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  labels:
    app: ghostwritten-kube-prometheus-stack-prometheus
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: grafana.demo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  prometheus-grafana
                port:
                  name: http-web

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
  labels:
    app: ghostwritten-kube-prometheus-stack-prometheus
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: prometheus.demo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name:  prometheus-ghostwritten-ku-prometheus
                port:
                  name: http-web
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alertmanager-ingress
  labels:
    app: ghostwritten-kube-prometheus-stack-prometheus
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: alertmanager.demo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: prometheus-ghostwritten-ku-alertmanager
                port:
                  name: http-web
```
执行：

```bash
k apply -f prometheus-grafana-alertmanager-ingress.yaml
```

windows: C:\Windows\System32\drivers\etc\hosts 配置：

```bash
192.168.10.29 grafana.demo.com prometheus.demo.com alertmanager.demo.com
```
访问： `http://prometheus.demo.com`
![](https://i-blog.csdnimg.cn/blog_migrate/5319f7540f9c74235ca9e33925dbe9e8.png)
访问：http://alertmanager.demo.com
![](https://i-blog.csdnimg.cn/blog_migrate/1c6a49ed4fe6f957f1e15d8b69cb5132.png)



访问：`http://grafana.demo.com`
![](https://i-blog.csdnimg.cn/blog_migrate/c95ad63f8db1df0686cb24cbb2bab2d8.png)
因为我第一次登陆`admin/admin`, 并没有登陆成功，通过`grafana-cli `命令修改 admin 密码为 `admin`：

```bash
$ grafana-cli admin reset-admin-password admin
INFO [02-28|10:24:48] Starting Grafana                         logger=settings version= commit= branch= compiled=1970-01-01T00:00:00Z
INFO [02-28|10:24:48] Config loaded from                       logger=settings file=/usr/share/grafana/conf/defaults.ini
INFO [02-28|10:24:48] Config overridden from Environment variable logger=settings var="GF_PATHS_DATA=/var/lib/grafana/"
INFO [02-28|10:24:48] Config overridden from Environment variable logger=settings var="GF_PATHS_LOGS=/var/log/grafana"
INFO [02-28|10:24:48] Config overridden from Environment variable logger=settings var="GF_PATHS_PLUGINS=/var/lib/grafana/plugins"
INFO [02-28|10:24:48] Config overridden from Environment variable logger=settings var="GF_PATHS_PROVISIONING=/etc/grafana/provisioning"
INFO [02-28|10:24:48] Config overridden from Environment variable logger=settings var="GF_SECURITY_ADMIN_USER=admin"
INFO [02-28|10:24:48] Config overridden from Environment variable logger=settings var="GF_SECURITY_ADMIN_PASSWORD=*********"
INFO [02-28|10:24:48] Path Home                                logger=settings path=/usr/share/grafana
INFO [02-28|10:24:48] Path Data                                logger=settings path=/var/lib/grafana/
INFO [02-28|10:24:48] Path Logs                                logger=settings path=/var/log/grafana
INFO [02-28|10:24:48] Path Plugins                             logger=settings path=/var/lib/grafana/plugins
INFO [02-28|10:24:48] Path Provisioning                        logger=settings path=/etc/grafana/provisioning
INFO [02-28|10:24:48] App mode production                      logger=settings
INFO [02-28|10:24:48] Connecting to DB                         logger=sqlstore dbtype=sqlite3
INFO [02-28|10:24:48] Starting DB migrations                   logger=migrator
INFO [02-28|10:24:48] migrations completed                     logger=migrator performed=0 skipped=464 duration=2.142374ms
INFO [02-28|10:24:48] Envelope encryption state                logger=secrets enabled=true current provider=secretKey.v1

Admin password changed successfully ✔
```
再次`admin/admin` 登陆成功,直接跳转修改密码，我修改新的密码为 `12345678`
![](https://i-blog.csdnimg.cn/blog_migrate/513a9194428e78c91abf2a2a57e156d1.png)
终于可以正常访问了。
![](https://i-blog.csdnimg.cn/blog_migrate/88c6e736c23a10581be4005eaab31a88.png)


##  12. 清理应用
删除 `prometheus` 应用
```bash
k delete -f prometheus-grafana-alertmanager-ingress.yaml
helm delete prometheus -n prometheus
k delete ns prometheus
```
强制删除命名空间,创建 `delete_ns.sh`

```bash
#!/bin/bash

NAMESPACE=$1
kubectl proxy &
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
```

```bash
bash delete_ns.sh prometheus
```
参考：
- [Prometheus Operator 初体验](https://www.qikqiak.com/post/first-use-prometheus-operator/)
