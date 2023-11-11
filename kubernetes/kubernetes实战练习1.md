

[kubernetes实战练习1](https://ghostwritten.blog.csdn.net/article/details/121186089)
[kubernetes实战练习2](https://ghostwritten.blog.csdn.net/article/details/121194823)

-----------
## 1. 启动单节点 Kubernetes 集群
Minikube已经安装并配置到环境中。通过运行`minikube version`命令检查它是否已正确安装，通过运行`minikube start`命令启动集群

### 1.1 创建集群
```bash
$ minikube version
minikube version: v1.8.1
commit: cbda04cf6bbe65e987ae52bb393c10099ab62014


$ minikube start --wait=false
* minikube v1.8.1 on Ubuntu 18.04
* Using the none driver based on user configuration
* Running on localhost (CPUs=2, Memory=2460MB, Disk=145651MB) ...
* OS release is Ubuntu 18.04.4 LTS
* Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
* Launching Kubernetes ... 
* Enabling addons: default-storageclass, storage-provisioner
* Configuring local host environment ...
* Done! kubectl is now configured to use "minikube"


$ kubectl cluster-info
Kubernetes master is running at https://172.17.0.37:8443
KubeDNS is running at https://172.17.0.37:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

#如果节点被标记为NotReady则它仍在启动组件。
#此命令显示可用于托管我们的应用程序的所有节点。现在我们只有一个节点，可以看到它的状态是ready
$ kubectl get nodes
NAME       STATUS     ROLES    AGE   VERSION
minikube   NotReady   master   12s   v1.17.3
```

### 1.2 启动容器
```bash

$ kubectl create deployment first-deployment --image=katacoda/docker-http-server
deployment.apps/first-deployment created


$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
first-deployment-666c48b44-vqsvl   1/1     Running   0          20s


#容器运行后，它可以根据要求通过不同的网络选项公开。一种可能的解决方案是 NodePort，它为容器提供动态端口。
$ kubectl expose deployment first-deployment --port=80 --type=NodePort
service/first-deployment exposed


#下面的命令查找分配的端口并执行 HTTP 请求。
$ export PORT=$(kubectl get svc first-deployment -o go-template='{{range.spec.ports}}{{if .nodePort}}{{.nodePort}}{{"\n"}}{{end}}{{end}}')
$ echo "Accessing host01:$PORT"
Accessing host01:31400
$ curl host01:$PORT
<h1>This request was processed by host: first-deployment-666c48b44-vqsvl</h1>
#结果是处理请求的容器。
```
### 1.3 创建界面
使用 Minikube 和命令启用仪表板

```bash
$ minikube addons enable dashboard
* The 'dashboard' addon is enabled
```
通过部署以下 YAML 定义使 Kubernetes 仪表板可用。这应该只用于 `Katacoda`。

```bash
$ cat /opt/kubernetes-dashboard.yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/minikube-addons: dashboard
  name: kubernetes-dashboard
  selfLink: /api/v1/namespaces/kubernetes-dashboard
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: kubernetes-dashboard
  name: kubernetes-dashboard-katacoda
  namespace: kubernetes-dashboard
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 9090
    nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
```
Kubernetes 仪表板允许您在 UI 中查看您的应用程序。在此部署中，仪表板已在端口30000上可用，但可能需要一段时间才能启动。要查看仪表板启动的进度，请使用以下命令查看kube-system命名空间中的Pod

```bash
$ kubectl get pods -n kubernetes-dashboard -w
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-7b64584c5c-479th   1/1     Running   0          110s
kubernetes-dashboard-79d9cd965-lhlxl         1/1     Running   0          110s
```
##  2. Kubeadm 入门
###  2.1 初始化 Master
Kubeadm 已经安装在节点上。软件包适用于 Ubuntu 16.04+、CentOS 7 或 HypriotOS v1.0.1+。

初始化集群的第一阶段是启动主节点。Master 负责运行控制平面组件、etcd 和 API 服务器。客户端将与 API 通信以安排工作负载并管理集群状态。

```bash
controlplane $ kubeadm init --token=102952.1a7dd4cc8d1f4cc5 --kubernetes-version $(kubeadm version -o short)
[init] Using Kubernetes version: v1.14.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [controlplane localhost] and IPs [172.17.0.18 127.0.0.1 ::1]
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [controlplane localhost] and IPs [172.17.0.18 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [controlplane kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.17.0.18]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s


[apiclient] All control plane components are healthy after 19.505046 seconds
[upload-config] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.14" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --experimental-upload-certs
[mark-control-plane] Marking the node controlplane as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node controlplane as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 102952.1a7dd4cc8d1f4cc5
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.17.0.18:6443 --token 102952.1a7dd4cc8d1f4cc5 \
    --discovery-token-ca-cert-hash sha256:1672bc53b2f7bb2e545ae130dc43fc0cc05f3ddf3e3be037a75317290bd66fcb 
```
在生产中，建议排除导致 kubeadm 代表您生成一个的令牌。

要管理 Kubernetes 集群，需要客户端配置和证书。这个配置是在kubeadm初始化集群时创建的。该命令将配置复制到用户主目录并设置用于 CLI 的环境变量。

```bash
controlplane $ sudo cp /etc/kubernetes/admin.conf $HOME/
controlplane $ sudo chown $(id -u):$(id -g) $HOME/admin.conf
controlplane $ export KUBECONFIG=$HOME/admin.conf
```
###  2.2 部署容器网络接口 (CNI)
容器网络接口 (CNI) 定义了不同节点及其工作负载应如何通信。有多个网络提供商可用。

在这个场景中，我们将使用 `Wea​​veWorks`。

```bash
controlplane $ cat /opt/weave-kube.yaml
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
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
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
      - apiGroups:
          - ''
        resources:
          - nodes/status
        verbs:
          - patch
          - update
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
    roleRef:
      kind: ClusterRole
      name: weave-net
      apiGroup: rbac.authorization.k8s.io
    subjects:
      - kind: ServiceAccount
        name: weave-net
        namespace: kube-system
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
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
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
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
  - apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: weave-net
      annotations:
        cloud.weave.works/launcher-info: |-
          {
            "original-request": {
              "url": "/k8s/v1.10/net.yaml?k8s-version=v1.16.0",
              "date": "Mon Oct 28 2019 18:38:09 GMT+0000 (UTC)"
            },
            "email-address": "support@weave.works"
          }
      labels:
        name: weave-net
      namespace: kube-system
    spec:
      minReadySeconds: 5
      selector:
        matchLabels:
          name: weave-net
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
                - name: IPALLOC_RANGE
                  value: 10.32.0.0/24
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'docker.io/weaveworks/weave-kube:2.6.0'
              readinessProbe:
                httpGet:
                  host: 127.0.0.1
                  path: /status
                  port: 6784
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
              env:
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.nodeName
              image: 'docker.io/weaveworks/weave-npc:2.6.0'
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
                type: FileOrCreate
```
执行创建

```bash
controlplane $ kubectl apply -f /opt/weave-kube.yaml
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created

#查看状态
controlplane $ kubectl get pod -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-22wq9                1/1     Running   0          21m
coredns-fb8b8dccf-k6tx5                1/1     Running   0          21m
etcd-controlplane                      1/1     Running   0          20m
kube-apiserver-controlplane            1/1     Running   0          20m
kube-controller-manager-controlplane   1/1     Running   0          19m
kube-proxy-scxbp                       1/1     Running   0          21m
kube-scheduler-controlplane            1/1     Running   1          20m
weave-net-9dmqb                        2/2     Running   0          37s
```
在集群上安装 `Weave` 时，请访问[https://www.weave.works/docs/net/latest/kube-addon/](%E5%9C%A8%E9%9B%86%E7%BE%A4%E4%B8%8A%E5%AE%89%E8%A3%85%20Weave%20%E6%97%B6%EF%BC%8C%E8%AF%B7%E8%AE%BF%E9%97%AEhttps://www.weave.works/docs/net/latest/kube-addon/%E4%BA%86%E8%A7%A3%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF%E3%80%82)了解详细信息。

###  2.3 加入集群
一旦 Master 和 CNI 初始化，其他节点只要拥有正确的令牌就可以加入集群。`kubeadm token`例如，可以通过 来管理令牌

```bash
controlplane $ kubeadm token list
TOKEN                     TTL       EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
102952.1a7dd4cc8d1f4cc5   23h       2021-11-08T02:56:07Z   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token
```
这与 Master 初始化后提供的命令相同。

```bash
node01 $ kubeadm join --discovery-token-unsafe-skip-ca-verification --token=102952.1a7dd4cc8d1f4cc5 172.17.0.18:6443
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.14" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

该`--discovery-token-unsafe-skip-ca-verification`标签用于绕过 `Discovery Token` 验证。由于此令牌是动态生成的，因此我们无法将其包含在步骤中。在生产中，使用由 提供的令牌`kubeadm init`。

### 2.4 查看节点
```bash
controlplane $ kubectl get nodes
NAME           STATUS   ROLES    AGE    VERSION
controlplane   Ready    master   35m    v1.14.0
node01         Ready    <none>   105s   v1.14.0
```
###  2.5 部署pod

```bash
controlplane $ kubectl create deployment http --image=katacoda/docker-http-server:latest
deployment.apps/http created


controlplane $ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
http-7f8cbdf584-5p7dd   1/1     Running   0          18s


node01 $ docker ps | grep docker-http-server
9a6f2a14b008        katacoda/docker-http-server   "/app"                   32 seconds ago      Up 32 seconds                           k8s_docker-http-server_http-7f8cbdf584-5p7dd_default_4e464e6d-3f7b-11ec-9bfb-0242ac110012_0
```

###  2.6 部署仪表板

```bash
controlplane $ cat dashboard.yaml 
# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# ------------------- Dashboard Secret ------------------- #

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque

---
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Role & Role Binding ------------------- #

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to create 'kubernetes-dashboard-key-holder' secret.
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create"]
  # Allow Dashboard to create 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["kubernetes-dashboard-settings"]
  verbs: ["get", "update"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]
- apiGroups: [""]
  resources: ["services/proxy"]
  resourceNames: ["heapster", "http:heapster:", "https:heapster:"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: apps/v1beta2
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
          - --auto-generate-certificates
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  externalIPs:
  - 172.17.0.28
  ports:
    - port: 8443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboardcontrolplane $ 
```
创建dashborad

```bash
controlplane $ kubectl apply -f dashboard.yaml
secret/kubernetes-dashboard-certs created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created

```
查看

```bash
controlplane $ kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-22wq9                 1/1     Running   0          61m
coredns-fb8b8dccf-k6tx5                 1/1     Running   0          61m
etcd-controlplane                       1/1     Running   0          60m
kube-apiserver-controlplane             1/1     Running   0          60m
kube-controller-manager-controlplane    1/1     Running   0          60m
kube-proxy-c66jf                        1/1     Running   0          28m
kube-proxy-scxbp                        1/1     Running   0          61m
kube-scheduler-controlplane             1/1     Running   1          60m
kubernetes-dashboard-5f57845f9d-mrhrv   1/1     Running   0          48s
weave-net-78dch                         2/2     Running   1          28m
weave-net-9dmqb                         2/2     Running   0          41m
```
需要 `ServiceAccount` 才能登录。`ClusterRoleBinding` 用于为新的 `ServiceAccount` ( `admin-user` )分配集群上的 `cluster -admin`角色。

```bash
cat <<EOF | kubectl create -f - 
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF
```
这意味着他们可以控制 Kubernetes 的所有方面。通过 ClusterRoleBinding 和 RBAC，可以根据安全要求定义不同级别的权限。可以在[Dashboard 文档](https://github.com/kubernetes/dashboard/wiki/)中找到有关为 Dashboard 创建用户的更多信息。

创建 `ServiceAccount` 后，可以通过以下方式找到登录令牌：

```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
部署仪表板时，它使用 `externalIPs` 将服务绑定到端口 8443。这使得仪表板可供集群外部使用，并可在`https://2886795292-8443-ollie02.environments.katacoda.com/`查看

使用管理员用户令牌访问仪表板。

对于生产，建议使用`kubectl proxy`来访问仪表板，而不是 `externalIP` 。在`https://github.com/kubernetes/dashboard` 上查看更多详细信息。

##  3. 使用 Kubectl 启动容器
在此场景中，您将学习如何使用 Kubectl 创建和启动部署、复制控制器并通过服务公开它们，而无需编写yaml定义。

###  3.1 启动集群

```bash
$ minikube start --wait=false
* minikube v1.8.1 on Ubuntu 18.04
* Using the none driver based on user configuration
* Running on localhost (CPUs=2, Memory=2460MB, Disk=145651MB) ...
* OS release is Ubuntu 18.04.4 LTS
* Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
* Launching Kubernetes ... 
* Enabling addons: default-storageclass, storage-provisioner
* Configuring local host environment ...
* Done! kubectl is now configured to use "minikube"


$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   2m13s   v1.17.3
```
###  3.2 Kubectl 运行
启动一个基于 Docker 镜像`katacoda/docker-http-server:latest`的容器

```bash
$ kubectl run http --image=katacoda/docker-http-server:latest --replicas=1
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/http created


$ kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
http   1/1     1            1           25s


$ kubectl describe deployment http
Name:                   http
Namespace:              default
CreationTimestamp:      Sun, 07 Nov 2021 04:53:59 +0000
Labels:                 run=http
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=http
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=http
  Containers:
   http:
    Image:        katacoda/docker-http-server:latest
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   http-774bb756bb (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  31s   deployment-controller  Scaled up replica set http-774bb756bb to 1
```

###  3.3 Kubectl 暴露
创建部署后，我们可以使用kubectl创建一个服务，该服务在特定端口上公开Pod。

通过kubectl 暴露新部署的http部署。该命令允许您定义服务的不同参数以及如何公开部署。
使用下面的命令暴露主机8000上的容器端口80绑定到主机的`external-ip`。

```bash
$ kubectl expose deployment http --external-ip="172.17.0.34" --port=8000 --target-port=80
service/http exposed


$ curl http://172.17.0.34:8000
<h1>This request was processed by host: http-774bb756bb-thlz6</h1>
```
###  3.4 Kubectl 运行和公开
使用kubectl run可以创建部署并将其公开为单个命令。

```bash
$ kubectl run httpexposed --image=katacoda/docker-http-server:latest --replicas=1 --port=80 --hostport=8001
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/httpexposed created

$ curl http://172.17.0.34:8001
<h1>This request was processed by host: httpexposed-68cb8c8d4-7jxl5</h1>

$ kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
http         ClusterIP   10.99.57.91   172.17.0.34   8000/TCP   5m26s
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP    21m


$ docker ps | grep httpexposed
305d27b7ef0a        katacoda/docker-http-server   "/app"                   About a minute ago   Up About a minute                          k8s_httpexposed_httpexposed-68cb8c8d4-7jxl5_default_d8535fb7-f2d9-4a50-a4b3-abc5d3166f2e_0
928f66157451        k8s.gcr.io/pause:3.1          "/pause"                 About a minute ago   Up About a minute   0.0.0.0:8001->80/tcp   k8s_POD_httpexposed-68cb8c8d4-7jxl5_default_d8535fb7-f2d9-4a50-a4b3-abc5d3166f2e_0
```
您应该能够使用它访问它 `curl http://172.17.0.34:8001`

在幕后，这通过 Docker 端口映射公开了 Pod。因此，您不会看到使用列出的服务`kubectl get svc`

要查找您可以使用的详细信息 `docker ps | grep httpexposed`

运行上面的命令，你会注意到端口暴露在 Pod 上，而不是 http 容器本身。Pause 容器负责为 Pod 定义网络。Pod 中的其他容器共享相同的网络命名空间。这提高了网络性能并允许多个容器通过同一网络接口进行通信。

###  3.5 扩展容器
随着我们的部署运行，我们现在可以使用kubectl来扩展副本的数量。

扩展部署将请求 Kubernetes 启动额外的 Pod。然后，这些 Pod 将使用公开的服务自动进行负载平衡。
`kubectl scale`命令允许我们调整为特定部署或复制控制器运行的Pod数量。

```bash
$ kubectl scale --replicas=3 deployment http
deployment.apps/http scaled
```
列出所有 pod，您应该会看到三个正在运行的http部署

```bash
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
http-774bb756bb-b79wn         1/1     Running   0          34s
http-774bb756bb-bspt4         1/1     Running   0          34s
http-774bb756bb-thlz6         1/1     Running   0          12m
httpexposed-68cb8c8d4-7jxl5   1/1     Running   0          4m34s
```
一旦每个 Pod 启动，它就会被添加到负载均衡器服务中。通过描述服务，您可以查看包含的端点和关联的 Pod。

```bash
$ kubectl describe svc http
Name:              http
Namespace:         default
Labels:            run=http
Annotations:       <none>
Selector:          run=http
Type:              ClusterIP
IP:                10.99.57.91
External IPs:      172.17.0.34
Port:              <unset>  8000/TCP
TargetPort:        80/TCP
Endpoints:         172.18.0.4:80,172.18.0.6:80,172.18.0.7:80
Session Affinity:  None
Events:            <none>
```
向服务发出请求将请求在不同的节点中处理请求。

```bash
$ curl http://172.17.0.34:8000
<h1>This request was processed by host: http-774bb756bb-thlz6</h1>
```

##  4.  YAML 部署容器

在此场景中，您将学习如何使用 Kubectl 创建和启动部署、复制控制器，并通过编写yaml定义通过服务公开它们。
YAML 定义定义了计划部署的 Kubernetes 对象。可以更新对象并将其重新部署到集群以更改配置。
###  4.1 创建deployment

deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp1
  template:
    metadata:
      labels:
        app: webapp1
    spec:
      containers:
      - name: webapp1
        image: katacoda/docker-http-server:latest
        ports:
        - containerPort: 80
```

```bash
$ kubectl create -f deployment.yaml
deployment.apps/webapp1 created

$ kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp1   1/1     1            1           29s


$ kubectl describe deployment webapp1
Name:                   webapp1
Namespace:              default
CreationTimestamp:      Sun, 07 Nov 2021 10:05:15 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=webapp1
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=webapp1
  Containers:
   webapp1:
    Image:        katacoda/docker-http-server:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   webapp1-6b54fb89d9 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  32s   deployment-controller  Scaled up replica set webapp1-6b54fb89d9 to 1
```
###  4.2 创建服务
Kubernetes 具有强大的网络功能，可以控制应用程序的通信方式。这些网络配置也可以通过 YAML 进行控制。

service.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: webapp1-svc
  labels:
    app: webapp1
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: webapp1
```

```bash
$ kubectl create -f service.yaml
service/webapp1-svc created
$ kubectl get svc
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP        3m3s
webapp1-svc   NodePort    10.107.139.217   <none>        80:30080/TCP   2s
$ kubectl describe svc webapp1-svc
Name:                     webapp1-svc
Namespace:                default
Labels:                   app=webapp1
Annotations:              <none>
Selector:                 app=webapp1
Type:                     NodePort
IP:                       10.107.139.217
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                172.18.0.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$ curl host01:30080
<h1>This request was processed by host: webapp1-6b54fb89d9-hfc88</h1>
```
### 4.3 扩容
更改`replicas: 4`

```bash
$ kubectl apply -f deployment.yaml
deployment.apps/webapp1 configured
$ kubectl get deployment
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp1   1/4     4            1           4m27s
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
webapp1-6b54fb89d9-hfc88   1/1     Running   0          4m31s
webapp1-6b54fb89d9-j9gdt   1/1     Running   0          6s
webapp1-6b54fb89d9-w7j2w   1/1     Running   0          6s
webapp1-6b54fb89d9-xczm7   1/1     Running   0          6s
$ curl host01:30080
<h1>This request was processed by host: webapp1-6b54fb89d9-hfc88</h1>
```

