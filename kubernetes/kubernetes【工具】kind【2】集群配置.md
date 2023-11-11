

----

更多阅读：

 - [kubernetes【工具】kind【1】入门实践](https://ghostwritten.blog.csdn.net/article/details/121968488)
 - [kubernetes【工具】kind【2】集群配置](https://ghostwritten.blog.csdn.net/article/details/122171462)

 - [https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/)


---

##  1. 多节点集群demo
kind-example-config.yaml
```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
kubeadmConfigPatches:
- |
  apiVersion: kubelet.config.k8s.io/v1beta1
  kind: KubeletConfiguration
  evictionHard:
    nodefs.available: "0%"
kubeadmConfigPatchesJSON6902:
- group: kubeadm.k8s.io
  version: v1beta2
  kind: ClusterConfiguration
  patch: |
    - op: add
      path: /apiServer/certSANs/-
      value: my-hostname
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
```
创建一个多节点集群
```bash
$ kind create cluster --config kind-example-config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼
 ✓ Preparing nodes 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

$ kind get clusters
kind

$ docker ps
CONTAINER ID        IMAGE                                                                                          COMMAND                  CREATED             STATUS              PORTS                       NAMES
c0671c3223c5        kindest/node:v1.21.1@sha256:fae9a58f17f18f06aeac9772ca8b5ac680ebbed985e266f711d936e91d113bad   "/usr/local/bin/en..."   5 minutes ago       Up 5 minutes        127.0.0.1:37346->6443/tcp   kind-control-plane
bb9edd673c90        kindest/node:v1.21.1@sha256:fae9a58f17f18f06aeac9772ca8b5ac680ebbed985e266f711d936e91d113bad   "/usr/local/bin/en..."   5 minutes ago       Up 5 minutes                                    kind-worker2
0bf3a45f2d9a        kindest/node:v1.21.1@sha256:fae9a58f17f18f06aeac9772ca8b5ac680ebbed985e266f711d936e91d113bad   "/usr/local/bin/en..."   5 minutes ago       Up 5 minutes                                    kind-worker


$ kubectl get node
NAME                 STATUS   ROLES                  AGE     VERSION
kind-control-plane   Ready    control-plane,master   3m31s   v1.21.1
kind-worker          Ready    <none>                 2m59s   v1.21.1
kind-worker2         Ready    <none>                 2m59s   v1.21.1

```

## 2. 定制log

```bash
$ kind export logs
Exported logs to: /tmp/396758314

$ kind export logs ./somedir
Exported logs to: ./somedir

#日志结构
$ tree 
.
├── docker-info.txt
└── kind-control-plane/
    ├── containers
    ├── docker.log
    ├── inspect.json
    ├── journal.log
    ├── kubelet.log
    ├── kubernetes-version.txt
    └── pods/
```


## 3. 集群配置

###  3.1 集群名字

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: app-1-cluster
```
###  3.2 特性门控

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
  # any feature gate can be enabled here with "Name": true
  # or disabled here with "Name": false
  # not all feature gates are tested, however
  "CSIMigration": true
```

###  3.3 Runtime Config
Kubernetes API服务器运行时配置可以使用runtimeConfig键来切换，该键映射到--runtime-config [kube-apiserver flag](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)标志。这可以用来禁用`beta / alpha api`。

###  3.4 Networking
集群网络的多个细节可以在网络字段下定制，KIND支持IPv4、IPv6和双栈集群，可以通过设置从默认IPv4切换，如果运行docker容器的主机支持IPv6，可以使用kind运行IPv6单栈集群。大多数操作系统/发行版默认都启用了IPv6，但是你可以在Linux上用下面的命令检查:

```bash
sudo sysctl net.ipv6.conf.all.disable_ipv6
```

```bash
net.ipv6.conf.all.disable_ipv6 = 0
```
如果你在Windows或Mac上使用Docker，你将需要使用一个IPv4端口转发的API服务器，因为IPv6端口转发不工作在这些平台上，你可以这样做:


```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  ipFamily: ipv6
  apiServerAddress: 127.0.0.1
```
On Linux all you need is:

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  ipFamily: ipv6
```
### 3.5 API Server 配置
API服务器监听地址和端口可以通过以下方式定制:

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  # WARNING: It is _strongly_ recommended that you keep this the default
  # (127.0.0.1) for security reasons. However it is possible to change this.
  apiServerAddress: "127.0.0.1"
  # By default the API server listens on a random open port.
  # You may choose a specific port but probably don't need to in most cases.
  # Using a random port makes it easier to spin up multiple clusters.
  apiServerPort: 6443
```
### 3.6 Pod Subnet 
通过“设置”配置pod ip所使用的子网

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  podSubnet: "10.244.0.0/16"
```
###  3.7 Service Subnet 
通过“设置”配置业务ip使用的Kubernetes业务子网

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  serviceSubnet: "10.96.0.0/12"
```
###  3.8 Disable Default CNI 
`KIND`提供了一个简单的网络实现(“kindnetd”)，它基于标准的CNI插件(ptp, host-local，…)和简单的`netlink`路由。
这个CNI也处理IP伪装。您可以禁用默认设置来安装不同的CNI。这是一个支持有限的高级用户特性，但已知有许多常见的CNI清单可以工作，例如Calico。

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  # the default CNI will not be installed
  disableDefaultCNI: true
```
###  3.9 kube-proxy mode
可以在iptables和ipvs之间配置`kube-proxy`模式。缺省情况下使用iptables

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  kubeProxyMode: "ipvs"
```

### 3.10 Multi-node clusters 
```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```
### 3.11 Control-plane HA 

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker
```

###  3.12 设置Kubernetes 版本

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
 - role: control-plane
  image: 
 - kindest/node:v1.16.4@sha256:b91a2c2317a000f3a783489dfb755064177dbc3a0b2f4147d50f04825d016f55
 - role: worker
  image: kindest/node:v1.16.4@sha256:b91a2c2317a000f3a783489dfb755064177dbc3a0b2f4147d50f04825d016f55
```
###  3.13 配置代理
如果您在一个需要代理的环境中运行kind，您可能需要配置kind来使用它。
 - HTTP_PROXY or http_proxy
 - HTTPS_PROXY or https_proxy
 - NO_PROXY or no_proxy

### 3.14 挂载
额外的挂载可以通过主机上的存储传递到一个类型节点，用于持久化数据、通过代码进行挂载等。

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  # add a mount from /path/to/my/files on the host to /files on the node
  extraMounts:
  - hostPath: /path/to/my/files/
    containerPath: /files
    # optional: if set, the mount is read-only.
    # default false
    readOnly: true
    # optional: if set, the mount needs SELinux relabeling.
    # default false
    selinuxRelabel: false
    # optional: set propagation mode (None, HostToContainer or Bidirectional)
    # see https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation
    # default None
    propagation: HostToContainer
```

###  3.15 将端口映射到主机
可以使用额外的端口映射将端口转发到类节点。这是一个跨平台的选项，可以让流量进入你的集群。使用Linux上的docker，您可以简单地将来自主机的流量发送到节点ip

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  # port forward 80 on the host to 80 on this node
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    # optional: set the bind address on the host
    # 0.0.0.0 is the current default
    listenAddress: "127.0.0.1"
    # optional: set the protocol to one of TCP, UDP, SCTP.
    # TCP is the default
    protocol: TCP
```
使用实例http pod将主机端口映射到容器端口。

```bash
kind: Pod
apiVersion: v1
metadata:
  name: foo
spec:
  containers:
  - name: foo
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=foo"
    ports:
    - containerPort: 5678
      hostPort: 80
```
### 3.16 `NodePort` 端口映射
要使用与NodePort的端口映射，类节点`containerPort`和服务`NodePort`需要相等。

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30950
    hostPort: 80
```
然后将`nodePort`设置为`30950`

```bash
kind: Pod
apiVersion: v1
metadata:
  name: foo
  labels:
    app: foo
spec:
  containers:
  - name: foo
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=foo"
    ports:
    - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: foo
spec:
  type: NodePort
  ports:
  - name: http
    nodePort: 30950
    port: 5678
  selector:
    app: foo
```

###  3.17 Kubeadm Config Patches
KIND使用[kubeadm](https://kind.sigs.k8s.io/docs/design/principles/#leverage-existing-tooling)配置集群节点，通常，KIND在第一个控制平面节点上运行`kubeadm init`，我们可以使用[kubeadm InitConfiguration](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file)(规范)来定制标志。

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
        node-labels: "my-label=true"
```
如果你想做更多的自定义，在`kubeadm init`中有四种配置类型:`InitConfiguration`, `ClusterConfiguration`, `KubeProxyConfiguration`, `KubeletConfiguration`。例如，我们可以使用kubeadm覆盖apiserver标志[ClusterConfiguration](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/control-plane-flags/) (spec):

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: ClusterConfiguration
    apiServer:
        extraArgs:
          enable-admission-plugins: NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook
```
在KIND集群、worker或控制平面(在HA模式下)中配置的每个额外节点上，KIND运行`kubeadm join`，可以使用`JoinConfiguration`(规范)配置该join。

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "my-label2=true"
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "my-label3=true"
```







