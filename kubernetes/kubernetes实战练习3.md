


[kubernetes实战练习1](https://ghostwritten.blog.csdn.net/article/details/121186089)
[kubernetes实战练习2](https://ghostwritten.blog.csdn.net/article/details/121194823)
[kubernetes实战练习3](https://ghostwritten.blog.csdn.net/article/details/121239607)
[**kubernetes 快速学习手册**](https://ghostwritten.blog.csdn.net/article/details/108562082)

----

##  1. CRI-O 和 Kubeadm 入门
在此场景中，您将学习如何使用 Kubeadm 引导 Kubernetes 集群。

`Kubeadm` 解决了处理 TLS 加密配置、部署核心 Kubernetes 组件以及确保额外节点可以轻松加入集群的问题。生成的集群通过 RBAC 等机制开箱即用。

有关 Kubeadm 的更多详细信息，请访问https://github.com/kubernetes/kubeadm

###  1.1 初始化 Master
Kubeadm 已经安装在节点上。软件包适用于 `Ubuntu 16.04+`、`CentOS 7` 或 `HypriotOS v1.0.1+`。

初始化集群的第一阶段是启动主节点。Master 负责运行控制平面组件、etcd 和 API 服务器。客户端将与 API 通信以安排工作负载并管理集群状态。

下面将使用 `CRI-O`，这是 Kubernetes 的轻量级容器运行时。当前存在一个错误，这意味着需要在开始之前重新启动 `CRI-O`。执行解决方法

```bash
$ systemctl restart crio
```
下面的命令将使用已知令牌初始化集群以简化以下步骤。该命令指向备用容器运行时接口 (CRI)，在本例中为 `CRI-O`。

```bash
$ kubeadm init --cri-socket=/var/run/crio/crio.sock --kubernetes-version $(kubeadm version -o short)
[init] Using Kubernetes version: v1.9.3
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.17.0.17]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 32.502591 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node master01 as master by adding a label and a taint
[markmaster] Master master01 tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 92bf0a.dd15deba9805244f
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 92bf0a.dd15deba9805244f 172.17.0.17:6443 --discovery-token-ca-cert-hash sha256:dd2bdbbd5a0e2ff140cf9e869ae752299a633945fd3e1378fb6175cc8a6738b6
```
在生产中，建议排除导致 kubeadm 代表您生成一个的令牌。

请注意，没有运行 Docker 容器。

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
相反，一切都通过 `CRI-O` 进行管理。可以通过以下方式探索其状态

```bash
$ crictl images
IMAGE                                                    TAG                 IMAGE ID            SIZE
docker.io/kubernetes/pause                               latest              f9d5de0795395       251kB
docker.io/weaveworks/weave-kube                          2.2.0               222ab9e78a839       96.7MB
docker.io/weaveworks/weave-npc                           2.2.0               765b48853ac02       47MB
gcr.io/google_containers/etcd-amd64                      3.1.11              59d36f27cceb9       194MB
gcr.io/google_containers/kube-apiserver-amd64            v1.9.3              360d55f91cbf4       211MB
gcr.io/google_containers/kube-controller-manager-amd64   v1.9.3              83dbda6ee8104       138MB
gcr.io/google_containers/kube-proxy-amd64                v1.9.3              35fdc6da5fd83       111MB
gcr.io/google_containers/kube-scheduler-amd64            v1.9.3              d3534b539b764       62.9MB


$ crictl ps
CONTAINER ID        IMAGE                                                                                                                            CREATED             STATE               NAME                      ATTEMPT
0510ca475b069       gcr.io/google_containers/kube-proxy-amd64@sha256:19277373ca983423c3ff82dbb14f079a2f37b84926a4c569375314fa39a4ee96                36 minutes ago      CONTAINER_RUNNING   kube-proxy                0
30c348901818e       gcr.io/google_containers/etcd-amd64@sha256:54889c08665d241e321ca5ce976b2df0f766794b698d53faf6b7dacb95316680                      36 minutes ago      CONTAINER_RUNNING   etcd                      0
842f2a756f4bf       gcr.io/google_containers/kube-scheduler-amd64@sha256:2c17e637c8e4f9202300bd5fc26bc98a7099f49559ca0a8921cf692ffd4a1675            36 minutes ago      CONTAINER_RUNNING   kube-scheduler            0
6d2b34ecc3062       gcr.io/google_containers/kube-apiserver-amd64@sha256:a5382344aa373a90bc87d3baa4eda5402507e8df5b8bfbbad392c4fff715f043            36 minutes ago      CONTAINER_RUNNING   kube-apiserver            0
599c2f875303c       gcr.io/google_containers/kube-controller-manager-amd64@sha256:3ac295ae3e78af5c9f88164ae95097c2d7af03caddf067cb35599769d0b7251e   36 minutes ago      CONTAINER_RUNNING   kube-controller-manager   0
```
因为 `CRI-O` 是为 Kubernetes 构建的，这意味着没有暂停容器。这只是为 Kubernetes 设计的容器运行时的众多优势之一。

默认情况下，`crictlCLI` 被配置为通过配置文件与运行时通信

```bash
$ cat /etc/crictl.yaml
runtime-endpoint: /var/run/crio/crio.sock
```
### 1.2 查看节点
要管理 `Kubernetes` 集群，需要客户端配置和证书。这个配置是在kubeadm初始化集群时创建的。该命令将配置复制到用户主目录并设置用于 CLI 的环境变量。

```bash
$ sudo cp /etc/kubernetes/admin.conf $HOME/
$ sudo chown $(id -u):$(id -g) $HOME/admin.conf
$ export KUBECONFIG=$HOME/admin.conf
```
`Kubernetes CLI`（称为kubectl）现在可以使用配置访问集群。例如，下面的命令将返回我们集群中的两个节点。

```bash
$ kubectl get nodes
NAME       STATUS    ROLES     AGE       VERSION
master01   Ready     master    45m       v1.9.3
```
我们可以通过描述节点来验证使用的`Container Runtime`。

```bash
$ kubectl describe node master01  | grep "Container Runtime Version:"
 Container Runtime Version:  cri-o://1.9.10-dev
```
###  1.3 Taint Master
在这种情况下，已配置单个节点。删除污点以将应用程序部署到主节点。

```bash
$ kubectl taint nodes --all node-role.kubernetes.io/master-
node "master01" untainted
```
Kubernetes 抽象了底层容器运行时，这意味着命令的行为符合预期。

```bash
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                               READY     STATUS              RESTARTS   AGE
kube-system   etcd-master01                      1/1       Running             0          48m
kube-system   kube-apiserver-master01            1/1       Running             0          48m
kube-system   kube-controller-manager-master01   1/1       Running             0          49m
kube-system   kube-dns-6f4fd4bdf-lxwrd           0/3       ContainerCreating   0          49m
kube-system   kube-proxy-2jh7x                   1/1       Running             0          49m
kube-system   kube-scheduler-master01            1/1       Running             0          48m
```
###  1.4 部署容器网络接口 (CNI)
容器网络接口 (CNI) 定义了不同节点及其工作负载应如何通信。有多个网络提供商可用，[此处](https://kubernetes.io/docs/concepts/cluster-administration/addons/)列出了一些。
在这个场景中，我们将使用 Wea​​veWorks

```bash
$ cat /opt/weave-kube.yaml
apiVersion: v1
kind: List
items:
  - apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.8/net.yaml?k8s-version=Q2xpZW50IFZlcnNpb246IHZlcnNpb24uSW5mb3tNYWpvcjoiMSIsIE1pbm9yOiI5IiwgR2l0VmVyc2lvbjoidjEuOS4zIiwgR2l0Q29tbWl0OiJkMjgzNTQxNjU0NGYyOThjOTE5ZTJlYWQzYmUzZDA4NjRiNTIzMjNiIiwgR2l0VHJlZVN0YXRlOiJjbGVhbiIsIEJ1aWxkRGF0ZToiMjAxOC0wMi0wN1QxMjoyMjoyMVoiLCBHb1ZlcnNpb246ImdvMS45LjIiLCBDb21waWxlcjoiZ2MiLCBQbGF0Zm9ybToibGludXgvYW1kNjQifQo=",
              "date": "Sun Mar 11 2018 19:17:47 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRole
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.8/net.yaml?k8s-version=Q2xpZW50IFZlcnNpb246IHZlcnNpb24uSW5mb3tNYWpvcjoiMSIsIE1pbm9yOiI5IiwgR2l0VmVyc2lvbjoidjEuOS4zIiwgR2l0Q29tbWl0OiJkMjgzNTQxNjU0NGYyOThjOTE5ZTJlYWQzYmUzZDA4NjRiNTIzMjNiIiwgR2l0VHJlZVN0YXRlOiJjbGVhbiIsIEJ1aWxkRGF0ZToiMjAxOC0wMi0wN1QxMjoyMjoyMVoiLCBHb1ZlcnNpb246ImdvMS45LjIiLCBDb21waWxlcjoiZ2MiLCBQbGF0Zm9ybToibGludXgvYW1kNjQifQo=",
              "date": "Sun Mar 11 2018 19:17:47 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    rules:
      - apiGroups:
          - ''
        resources:
          - pods
          - namespaces
          - nodes
        verbs:
          - get
          - list
          - watch
      - apiGroups:
          - networking.k8s.io
        resources:
          - networkpolicies
        verbs:
          - get
          - list
          - watch
  - apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.8/net.yaml?k8s-version=Q2xpZW50IFZlcnNpb246IHZlcnNpb24uSW5mb3tNYWpvcjoiMSIsIE1pbm9yOiI5IiwgR2l0VmVyc2lvbjoidjEuOS4zIiwgR2l0Q29tbWl0OiJkMjgzNTQxNjU0NGYyOThjOTE5ZTJlYWQzYmUzZDA4NjRiNTIzMjNiIiwgR2l0VHJlZVN0YXRlOiJjbGVhbiIsIEJ1aWxkRGF0ZToiMjAxOC0wMi0wN1QxMjoyMjoyMVoiLCBHb1ZlcnNpb246ImdvMS45LjIiLCBDb21waWxlcjoiZ2MiLCBQbGF0Zm9ybToibGludXgvYW1kNjQifQo=",
              "date": "Sun Mar 11 2018 19:17:47 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    roleRef:
      kind: ClusterRole
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: Role
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.8/net.yaml?k8s-version=Q2xpZW50IFZlcnNpb246IHZlcnNpb24uSW5mb3tNYWpvcjoiMSIsIE1pbm9yOiI5IiwgR2l0VmVyc2lvbjoidjEuOS4zIiwgR2l0Q29tbWl0OiJkMjgzNTQxNjU0NGYyOThjOTE5ZTJlYWQzYmUzZDA4NjRiNTIzMjNiIiwgR2l0VHJlZVN0YXRlOiJjbGVhbiIsIEJ1aWxkRGF0ZToiMjAxOC0wMi0wN1QxMjoyMjoyMVoiLCBHb1ZlcnNpb246ImdvMS45LjIiLCBDb21waWxlcjoiZ2MiLCBQbGF0Zm9ybToibGludXgvYW1kNjQifQo=",
              "date": "Sun Mar 11 2018 19:17:47 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    rules:
      - apiGroups:
          - ''
        resourceNames:
          - weave-net
        resources:
          - configmaps
        verbs:
          - get
          - update
      - apiGroups:
          - ''
        resources:
          - configmaps
        verbs:
          - create
  - apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: RoleBinding
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.8/net.yaml?k8s-version=Q2xpZW50IFZlcnNpb246IHZlcnNpb24uSW5mb3tNYWpvcjoiMSIsIE1pbm9yOiI5IiwgR2l0VmVyc2lvbjoidjEuOS4zIiwgR2l0Q29tbWl0OiJkMjgzNTQxNjU0NGYyOThjOTE5ZTJlYWQzYmUzZDA4NjRiNTIzMjNiIiwgR2l0VHJlZVN0YXRlOiJjbGVhbiIsIEJ1aWxkRGF0ZToiMjAxOC0wMi0wN1QxMjoyMjoyMVoiLCBHb1ZlcnNpb246ImdvMS45LjIiLCBDb21waWxlcjoiZ2MiLCBQbGF0Zm9ybToibGludXgvYW1kNjQifQo=",
              "date": "Sun Mar 11 2018 19:17:47 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    roleRef:
      kind: Role
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: extensions/v1beta1
    kind: DaemonSet
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.8/net.yaml?k8s-version=Q2xpZW50IFZlcnNpb246IHZlcnNpb24uSW5mb3tNYWpvcjoiMSIsIE1pbm9yOiI5IiwgR2l0VmVyc2lvbjoidjEuOS4zIiwgR2l0Q29tbWl0OiJkMjgzNTQxNjU0NGYyOThjOTE5ZTJlYWQzYmUzZDA4NjRiNTIzMjNiIiwgR2l0VHJlZVN0YXRlOiJjbGVhbiIsIEJ1aWxkRGF0ZToiMjAxOC0wMi0wN1QxMjoyMjoyMVoiLCBHb1ZlcnNpb246ImdvMS45LjIiLCBDb21waWxlcjoiZ2MiLCBQbGF0Zm9ybToibGludXgvYW1kNjQifQo=",
              "date": "Sun Mar 11 2018 19:17:47 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    spec:
      template:
        metadata:
          labels:
            name: weave-net
        spec:
          containers:
            - name: weave
              command:
                - /home/weave/launch.sh
              env:
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'docker.io/weaveworks/weave-kube:2.2.0'
              livenessProbe:
                httpGet:
                  host: 127.0.0.1
                  path: /status
                  port: 6784
                initialDelaySeconds: 30
              resources:
                requests:
                  cpu: 10m
              securityContext:
                privileged: true
              volumeMounts:
                - name: weavedb
                  mountPath: /weavedb
                - name: cni-bin
                  mountPath: /host/opt
                - name: cni-bin2
                  mountPath: /host/home
                - name: cni-conf
                  mountPath: /host/etc
                - name: dbus
                  mountPath: /host/var/lib/dbus
                - name: lib-modules
                  mountPath: /lib/modules
                - name: xtables-lock
                  mountPath: /run/xtables.lock
            - name: weave-npc
              args: []
              env:
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'docker.io/weaveworks/weave-npc:2.2.0'
              resources:
                requests:
                  cpu: 10m
              securityContext:
                privileged: true
              volumeMounts:
                - name: xtables-lock
                  mountPath: /run/xtables.lock
          hostNetwork: true
          hostPID: true
          restartPolicy: Always
          securityContext:
            seLinuxOptions: {}
          serviceAccountName: weave-net
          tolerations:
            - effect: NoSchedule
              operator: Exists
          volumes:
            - name: weavedb
              hostPath:
                path: /var/lib/weave
            - name: cni-bin
              hostPath:
                path: /opt
            - name: cni-bin2
              hostPath:
                path: /home
            - name: cni-conf
              hostPath:
                path: /etc
            - name: dbus
              hostPath:
                path: /var/lib/dbus
            - name: lib-modules
              hostPath:
                path: /lib/modules
            - name: xtables-lock
              hostPath:
                path: /run/xtables.lock
      updateStrategy:
        type: RollingUpdate
```
执行

```bash
$ kubectl apply -f /opt/weave-kube.yaml
serviceaccount "weave-net" created
clusterrole "weave-net" created
clusterrolebinding "weave-net" created
role "weave-net" created
rolebinding "weave-net" created
daemonset "weave-net" created


$ kubectl get pod -n kube-system
NAME                               READY     STATUS              RESTARTS   AGE
etcd-master01                      1/1       Running             0          53m
kube-apiserver-master01            1/1       Running             0          53m
kube-controller-manager-master01   1/1       Running             0          53m
kube-dns-6f4fd4bdf-lxwrd           0/3       ContainerCreating   0          54m
kube-proxy-2jh7x                   1/1       Running             0          54m
kube-scheduler-master01            1/1       Running             0          53m
weave-net-ssfkk                    2/2       Running             0          1m
```
在集群上安装 Weave 时，请访问[https://www.weave.works/docs/net/latest/kube-addon/](https://www.weave.works/docs/net/latest/kube-addon/)了解详细信息。


###  1.5 部署 Pod
集群中两个节点的状态现在应该是 Ready。这意味着我们的部署可以被安排和启动。

使用 Kubectl，可以部署 pod。命令总是为 Master 发出，每个节点只负责执行工作负载。

下面的命令基于 Docker Image `katacoda/docker-http-server`创建一个 Pod 。

```bash
$ kubectl run http --image=katacoda/docker-http-server:latest --replicas=1
deployment "http" created

$ kubectl get pods
NAME                    READY     STATUS              RESTARTS   AGE
http-85ddfcb674-khrqn   0/1       ImageInspectError   0          1m
```
您会注意到这是 `ImageInspectError`。这是预期的。这是因为所有镜像都需要以 `Container Image Registry` 为前缀，例如 `Docker Hub` 的 `docker.io`。

通过更改图像以包含注册表 URL 来解决此问题。

```bash
$ kubectl set image deployment/http http=docker.io/katacoda/docker-http-server:latest
deployment "http" image updated
```
因为我们污染了master，一旦运行，就可以看到在master节点上运行的Container。

```bash
$ crictl ps | grep docker-http-server
7bd5c4819c6ac       docker.io/katacoda/docker-http-server@sha256:76dc8a47fd019f80f2a3163aba789faf55b41b2fb06397653610c754cb12d3ee                    36 seconds ago      CONTAINER_RUNNING   http         
```

###  1.6 Deploy Dashboard
Kubernetes 有一个基于 Web 的仪表板 UI，提供对 Kubernetes 集群的可见性。

```bash
kubectl apply -f dashboard.yaml
kubectl get pods -n kube-system
```
部署dashboad时，它被分配了 30000 的 NodePort。这使得dashboad可用于集群外部查看


##  2. 运行Stateful Services 
###  2.1 部署 NFS 服务器
NFS 是一种允许节点通过网络读/写数据的协议。该协议的工作原理是让主节点运行 NFS 守护程序并存储数据。此主节点使某些目录可通过网络使用。

客户端访问通过驱动器挂载共享的主服务器。从应用程序的角度来看，它们正在写入本地磁盘。在幕后，NFS 协议将其写入主服务器。

在此场景中，出于演示和学习目的，NFS 服务器的角色由自定义容器处理。容器通过 NFS 提供目录并将数据存储在容器内。在生产中，建议配置专用的 `NFS Server`。

```bash
controlplane $ docker run -d --net=host \
>    --privileged --name nfs-server \
>    katacoda/contained-nfs-server:centos7 \
>    /exports/data-0001 /exports/data-0002
Unable to find image 'katacoda/contained-nfs-server:centos7' locally
centos7: Pulling from katacoda/contained-nfs-server
8d30e94188e7: Pull complete 
2b2b27f1f462: Pull complete 
133e63cf95fe: Pull complete 
Digest: sha256:5f2ea4737fe27f26be5b5cabaa23e24180079a4dce8d5db235492ec48c5552d1
Status: Downloaded newer image for katacoda/contained-nfs-server:centos7
f649240cba8f1dfb482c2799269b3e2ed24f2c30158c8aa9b7d547f939129cc0
```
NFS 服务器公开两个目录，`data-0001`和`data-0002`。在接下来的步骤中，这将用于存储数据。

###  2.2 Deploy Persistent Volume
为了让 Kubernetes 了解可用的 NFS 共享，它需要一个`PersistentVolume`配置。该`PersistentVolume`支持用于存储数据，如`AWS EBS`卷，GCE存储，OpenStack Cinder，`Glusterfs`和NFS不同的协议。该配置提供了存储和 API 之间的抽象，从而实现了一致的体验。

在 NFS 的情况下，一个`PersistentVolume`与一个 `NFS` 目录相关。当容器处理完卷后，可以保留数据以备将来使用，也可以回收卷，这意味着所有数据都将被删除。该策略由`persistentVolumeReclaimPolicy`选项定义。
定义：

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <friendly-name>
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: <server-name>
    path: <shared-path>
```
该规范定义了关于持久卷的额外元数据，包括有多少可用空间以及它是否具有读/写访问权限。创建两个新的 `PersistentVolume` 定义以指向两个可用的 NFS 共享。

```bash
controlplane $ cat nfs-0001.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-0001
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 172.17.0.11
    path: /exports/data-0001
controlplane $ cat nfs-0002.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-0002
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 172.17.0.11
    path: /exports/data-0002

controlplane $ kubectl create -f nfs-0001.yaml
persistentvolume/nfs-0001 created
controlplane $ kubectl create -f nfs-0002.yaml
persistentvolume/nfs-0002 created


controlplane $ kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
nfs-0001   2Gi        RWO,RWX        Recycle          Available                                   60s
nfs-0002   5Gi        RWO,RWX        Recycle          Available                                   58s
```

###  2.3 Deploy Persistent Volume Claim

一旦持久卷可用，应用程序就可以声明该卷供其使用。该声明旨在阻止应用程序意外写入同一卷并导致冲突和数据损坏。

声明指定了卷的要求。这包括所需的读/写访问和存储空间。一个例子如下：

```bash
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
为两个不同的应用程序创建两个声明。MySQL Pod 将使用一个声明，另一个由 HTTP 服务器使用。

```bash
controlplane $ cat pvc-mysql.yaml pvc-http.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-mysql
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-http
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
controlplane $ kubectl create -f pvc-mysql.yaml
persistentvolumeclaim/claim-mysql created
controlplane $ kubectl create -f pvc-http.yaml
persistentvolumeclaim/claim-http created

controlplane $ kubectl get pvc
NAME          STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
claim-http    Bound    nfs-0001   2Gi        RWO,RWX                       26s
claim-mysql   Bound    nfs-0002   5Gi        RWO,RWX                       29s
```
###  2.4 Use Volume
定义部署后，它可以将自己分配给先前的声明。以下代码段定义了目录`/var/lib/mysql/data`的卷挂载，该目录映射到存储`mysql-persistent-storage`。名为`mysql-persistent-storage` 的存储映射到名为`claim-mysql` 的声明。

```bash
  spec:
      volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql/data
  volumes:
    - name: mysql-persistent-storage
      persistentVolumeClaim:
        claimName: claim-mysql
```
推出两个具有 `Persistent Volume Claims` 的新 Pod。当 Pod 开始允许应用程序像本地目录一样读/写时，卷被映射到正确的目录。

```bash
controlplane $ cat pod-mysql.yaml pod-www.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    name: mysql
spec:
  containers:
  - name: mysql
    image: openshift/mysql-55-centos7
    env:
      - name: MYSQL_ROOT_PASSWORD
        value: yourpassword
      - name: MYSQL_USER
        value: wp_user
      - name: MYSQL_PASSWORD
        value: wp_pass
      - name: MYSQL_DATABASE
        value: wp_db
    ports:
      - containerPort: 3306
        name: mysql
    volumeMounts:
      - name: mysql-persistent-storage
        mountPath: /var/lib/mysql/data
  volumes:
    - name: mysql-persistent-storage
      persistentVolumeClaim:
        claimName: claim-mysql
apiVersion: v1
kind: Pod
metadata:
  name: www
  labels:
    name: www
spec:
  containers:
  - name: www
    image: nginx:alpine
    ports:
      - containerPort: 80
        name: www
    volumeMounts:
      - name: www-persistent-storage
        mountPath: /usr/share/nginx/html
  volumes:
    - name: www-persistent-storage
      persistentVolumeClaim:
        claimName: claim-http
```

```bash
controlplane $ kubectl create -f pod-mysql.yaml
pod/mysql created
controlplane $ kubectl create -f pod-www.yaml
pod/www created

controlplane $ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
mysql   1/1     Running   0          64s
www     1/1     Running   0          62s
```
推出两个具有 `Persistent Volume Claims` 的新 Pod。当 Pod 开始允许应用程序像本地目录一样读/写时，卷被映射到正确的目录。

如果 `Persistent Volume Claim` 未分配给 `Persistent Volume`，则 Pod 将处于Pending模式，直到它变为可用。在下一步中，我们将向卷读/写数据。

###  2.5 Read/Write Data
我们的 Pod 现在可以读/写。MySQL 将所有数据库更改存储到 NFS 服务器，而 HTTP 服务器将从 NFS 驱动器提供静态服务。升级、重新启动或将容器移动到不同的机器时，数据仍然可以访问。

要测试 HTTP 服务器，请编写一个“Hello World” index.html主页。在这种情况下，我们知道 HTTP 目录将基于`data-0001`，因为卷定义没有驱动足够的空间来满足 MySQL 大小要求。

```bash
controlplane $ docker exec -it nfs-server bash -c "echo 'Hello World' > /exports/data-0001/index.html"
```
根据 Pod 的 IP，访问 Pod 时，应该返回预期的响应。

```bash
controlplane $ ip=$(kubectl get pod www -o yaml |grep podIP | awk '{split($0,a,":"); print a[2]}'); echo $ip
10.32.0.6

controlplane $ curl $ip
Hello World
```
更新数据
当 NFS 共享上的数据发生变化时，Pod 会读取新更新的数据。

```bash
controlplane $ docker exec -it nfs-server bash -c "echo 'Hello NFS World' > /exports/data-0001/index.html"

controlplane $ curl $ip
Hello NFS World
```
###  2.6 Recreate Pod
因为远程 NFS 服务器存储数据，如果 Pod 或主机宕机，那么数据仍然可用。删除 Pod 将导致它删除对任何持久卷的声明。新 Pod 可以获取并重新使用 NFS 共享。

```bash
controlplane $ kubectl delete pod www
pod "www" deleted


controlplane $ cat pod-www2.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: www2
  labels:
    name: www2
spec:
  containers:
  - name: www2
    image: nginx:alpine
    ports:
      - containerPort: 80
        name: www2
    volumeMounts:
      - name: www-persistent-storage
        mountPath: /usr/share/nginx/html
  volumes:
    - name: www-persistent-storage
      persistentVolumeClaim:
        claimName: claim-http


controlplane $ kubectl create -f pod-www2.yaml
pod/www2 created

controlplane $ ip=$(kubectl get pod www2 -o yaml |grep podIP | awk '{split($0,a,":"); print a[2]}'); curl $ip
Hello NFS World
```
应用程序现在使用远程 NFS 进行数据存储。根据要求，这种相同的方法适用于其他存储引擎，例如 GlusterFS、AWS EBS、GCE 存储或 OpenStack Cinder。

##  3. Use Kubernetes to manage Secrets
在此场景中，您将了解如何使用 Kubernetes 管理Secrets。Kubernetes 允许您创建通过环境变量或作为卷挂载到 pod 的Secrets。

这允许Secrets（例如 SSL 证书或密码）只能通过基础架构团队以安全的方式进行管理，而不是将Secrets存储在应用程序的部署工件中。

###  3.1 Create Secrets
Kubernetes 要求将机密编码为 `Base64` 字符串。

使用命令行工具，我们可以创建 `Base64` 字符串并将它们存储为变量以在文件中使用。

```bash
username=$(echo -n "admin" | base64)
password=$(echo -n "a62fjbd37942dcs" | base64)
```
Secrets是使用yaml定义的。下面我们将使用上面定义的变量，并为它们提供我们的应用程序可以使用的友好标签。这将创建一个可以通过名称访问的键/值Secrets集合，在本例中为test-secret

```bash
controlplane $  secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  username: $username
  password: $password" 
```
这个yaml文件可以与 `Kubectl` 一起使用来创建我们的秘密。在启动需要访问密钥的 pod 时，我们将通过友好名称引用集合。

```bash
controlplane $ kubectl create -f secret.yaml
secret/test-secret created

controlplane $ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-rgbmd   kubernetes.io/service-account-token   3      8m12s
test-secret           Opaque                                2      62s
```
### 3.2 secret配置环境变量
在文件`secret-env.yaml` 中，我们定义了一个 Pod，它具有从先前创建的秘密填充的环境变量。

```bash
controlplane $ cat secret-env.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
    - name: mycontainer
      image: alpine:latest
      command: ["sleep", "9999"]
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: test-secret
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: test-secret
              key: password
  restartPolicy: Never
```
为了填充环境变量，我们定义了名称，在本例中为 `SECRET_USERNAME`，以及机密集合的名称和包含数据的密钥。

```bash
- name: SECRET_USERNAME
valueFrom:
 secretKeyRef:
   name: test-secret
   key: username
```

```bash
controlplane $ kubectl create -f secret-env.yaml
pod/secret-env-pod created


controlplane $ kubectl exec -it secret-env-pod env | grep SECRET_
SECRET_USERNAME=admin
SECRET_PASSWORD=a62fjbd37942dcs
```
Kubernetes 在填充环境变量时解码 `base64` 值。您应该会看到我们定义的原始用户名/密码组合。这些变量现在可用于访问 API、数据库等。

```bash
controlplane $ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
secret-env-pod   1/1     Running   0          78s
```
### 3.3 通过卷挂载secret
使用环境变量在内存中存储机密可能会导致它们意外泄漏。推荐的方法是将它们安装为卷。

```bash
controlplane $ cat secret-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-pod
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: test-secret
  containers:
    - name: test-container
      image: alpine:latest
      command: ["sleep", "9999"]
      volumeMounts:
          - name: secret-volume
            mountPath: /etc/secret-volume
```
要将secret安装为卷，我们首先定义一个具有众所周知名称的卷，在本例中为volume卷，并为其提供我们存储的volume。

```bash
volumes:
 - name: secret-volume
   secret:
     secretName: test-secret
```
当我们定义容器时，我们将创建的卷挂载到特定目录。应用程序将从该路径读取secret作为文件

```bash
volumeMounts:
 - name: secret-volume
   mountPath: /etc/secret-volume
```

```bash
controlplane $ kubectl create -f secret-pod.yaml
pod/secret-vol-pod created


controlplane $ kubectl exec -it secret-vol-pod ls /etc/secret-volume
password  username

controlplane $ kubectl exec -it secret-vol-pod cat /etc/secret-volume/username
admin


admincontrolplane $ kubectl exec -it secret-vol-pod cat /etc/secret-volume/password
a62fjbd37942dcs
```

##  4. Deploy Docker Compose with Kompose
这个场景中，您将如何使用 Kompose 将现有的 Docker Compose 文件部署到 Kubernetes。

Kompose 是一个帮助熟悉 docker-compose 的用户迁移到 Kubernetes 的工具。它需要一个 Docker Compose 文件并将其转换为 Kubernetes 资源。更多细节可以在[http://kompose.io/](http://kompose.io/)找到

###  4.1 安装Kompose
Kompose 作为二进制文件部署到客户端。要在 Katacoda 上安装 Kompose，请运行命令

```bash
curl -L https://github.com/kubernetes/kompose/releases/download/v1.9.0/kompose-linux-amd64 -o /usr/bin/kompose && chmod +x /usr/bin/kompose
```
有关如何为您的操作系统安装 Kompose 的详细信息，请访问[https://github.com/kubernetes/kompose/releases](https://github.com/kubernetes/kompose/releases)

### 4.2 kompose 启动
`Kompose` 采用现有的 `Docker Compose` 文件，并使它们能够部署到 Kubernetes 上。`Compose` 是一个用于定义和运行多容器 Docker 应用程序的工具。借助 Compose，您可以使用 `Compose` 文件来配置应用程序的服务。

将示例 Docker Compose 文件复制到编辑器。

```bash
version: "2"
services:
  redis-master:
    image: redis:latest
    ports:
      - "6379"
  redis-slave:
    image: gcr.io/google_samples/gb-redisslave:v1
    ports:
      - "6379"
    environment:
      - GET_HOSTS_FROM=dns
  frontend:
    image: gcr.io/google-samples/gb-frontend:v3
    ports:
      - "80:80"
    environment:
      - GET_HOSTS_FROM=dns
```
与 Docker Compose 一样，`Kompose` 允许使用单个命令部署`image`

```bash
controlplane $ kompose up
INFO We are going to create Kubernetes Deployments, Services and PersistentVolumeClaims for your Dockerized application. If you need different kind of resources, use the 'kompose convert' and 'kubectl create -f' commands instead. 
 
INFO Deploying application in "default" namespace 
INFO Successfully created Service: frontend       
INFO Successfully created Service: redis-master   
INFO Successfully created Service: redis-slave    
INFO Successfully created Deployment: frontend    
INFO Successfully created Deployment: redis-master 
INFO Successfully created Deployment: redis-slave 

Your application has been deployed to Kubernetes. You can run 'kubectl get deployment,svc,pods,pvc' for details.
```
可以使用 Kubernetes CLI kubectl发现已部署内容的详细信息。

```bash
controlplane $ kubectl get deployment,svc,pods,pvc
NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/frontend       1/1     1            1           35s
deployment.extensions/redis-master   1/1     1            1           35s
deployment.extensions/redis-slave    1/1     1            1           35s

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/frontend       ClusterIP   10.101.107.67    <none>        80/TCP     35s
service/kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP    2m10s
service/redis-master   ClusterIP   10.109.66.160    <none>        6379/TCP   35s
service/redis-slave    ClusterIP   10.110.203.137   <none>        6379/TCP   35s

NAME                               READY   STATUS    RESTARTS   AGE
pod/frontend-64bcc8dd75-n8lrx      1/1     Running   0          35s
pod/redis-master-fc59d57fd-wl2bd   1/1     Running   0          35s
pod/redis-slave-8676777656-xwhrk   1/1     Running   0          35s
```
###  4.3 转换
`Kompose` 还能够获取现有的 `Compose` 文件并生成相关的 `Kubernetes Manifest` 文件。

```bash
controlplane $ kompose convert
INFO Kubernetes file "frontend-service.yaml" created 
INFO Kubernetes file "redis-master-service.yaml" created 
INFO Kubernetes file "redis-slave-service.yaml" created 
INFO Kubernetes file "frontend-deployment.yaml" created 
INFO Kubernetes file "redis-master-deployment.yaml" created 
INFO Kubernetes file "redis-slave-deployment.yaml" created 


controlplane $ ls
docker-compose.yml        frontend-service.yaml         redis-master-service.yaml    redis-slave-service.yaml
frontend-deployment.yaml  redis-master-deployment.yaml  redis-slave-deployment.yaml
```
###  4.4 Kubectl 创建
转换文件后，它们也可以使用Kubectl进行部署。这将匹配通过kompose up应用的现有部署。

```bash
controlplane $ kubectl apply -f frontend-service.yaml,redis-master-service.yaml,redis-slave-service.yaml,frontend-deployment.yaml,redis-master-deployment.yaml,redis-slave-deployment.yaml

```
###  4.5 OpenShift
Kompose 还支持不同的 Kubernetes 发行版，例如 OpenShift。

```bash
controlplane $ kompose --provider openshift convert
INFO OpenShift file "frontend-service.yaml" created 
INFO OpenShift file "redis-master-service.yaml" created 
INFO OpenShift file "redis-slave-service.yaml" created 
INFO OpenShift file "frontend-deploymentconfig.yaml" created 
INFO OpenShift file "frontend-imagestream.yaml" created 
INFO OpenShift file "redis-master-deploymentconfig.yaml" created 
INFO OpenShift file "redis-master-imagestream.yaml" created 
INFO OpenShift file "redis-slave-deploymentconfig.yaml" created 
INFO OpenShift file "redis-slave-imagestream.yaml" created 

```
###  4.6 转换为 Json
默认情况下，Kompose 会生成 YAML 文件。可以通过指定-j参数来生成基于 JSON 的文件。

```bash
controlplane $ kompose convert -j
INFO Kubernetes file "frontend-service.json" created 
INFO Kubernetes file "redis-master-service.json" created 
INFO Kubernetes file "redis-slave-service.json" created 
INFO Kubernetes file "frontend-deployment.json" created 
INFO Kubernetes file "redis-master-deployment.json" created 
INFO Kubernetes file "redis-slave-deployment.json" created 
```

