

---

 - [https://metallb.universe.tf/installation/](https://metallb.universe.tf/installation/)

---



## 1. 什么是MetalLB
根据 [官方文档](https://metallb.universe.tf/)  MetalLB 是裸机Kubernetes 集群的负载均衡器实现 ，使用标准路由协议。

##  2. 为什么需要它
如果您在云解决方案（AWS、GCP）之外部署 Kubernetes 集群，您需要一种在集群之外公开服务的方法。您可以使用 NodePort 使用端口范围 30000–32767 公开您的服务，但是每次您的客户需要访问您的服务时，他们都需要指定这个高阶端口。如果您尝试在裸机中使用 LoadBalancer 服务，您将看到您的服务将永远挂起。在这些情况下，您可以使用 MetlabLB。

##  3. 要求
MetalLB 需要以下功能才能运行：

 - 一个 [Kubernetes](https://kubernetes.io/) 集群，运行 Kubernetes 1.13.0 或更高版本，还没有网络负载平衡功能。
 - 一个 [集群网络配置](https://metallb.universe.tf/installation/network-addons/) 可以与MetalLB共存。
 - 一些供 MetalLB 分发的 IPv4 地址。
 - 使用 [BGP](https://en.wikipedia.org/wiki/Border_Gateway_Protocol) 操作模式时，您需要一台或多台能够通话的路由器 BGP.
 - 节点之间必须允许端口 7946（TCP 和 UDP）上的流量，如 [会员名单](https://github.com/hashicorp/memberlist).

##  4. 准备
如果您在 IPVS 模式下使用 `kube-proxy`，从 `Kubernetes v1.14.2` 开始，您必须启用严格的 `ARP` 模式。

请注意，如果您使用 `kube-router` 作为服务代理，则不需要这个，因为它默认启用严格的 ARP。

您可以通过在当前集群中编辑 kube-proxy 配置来实现这一点：

```bash
$ kubectl edit configmap -n kube-system kube-proxy
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```
您还可以将此配置片段添加到您的 kubeadm-config 中，只需将其附加---到主配置之后。

如果您尝试自动执行此更改，这些 shell 片段可能会帮助您：

```bash
# 查看将进行哪些更改，如果不同则返回非零返回码
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# 实际应用更改，仅在错误时返回非零返回码
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```
##  5. 安装
### 5.1 通过清单安装

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml
```
这会将 `MetalLB` 部署到`metallb-system` 命名空间下的集群。清单中的组件是：

 - 该`metallb-system/controller`  deployment。这是处理 IP 地址分配的集群范围的控制器。
 - 该`metallb-system/speaker` daemonset。这是使用您选择的协议以使服务可访问的组件。
 - 控制器和扬声器的Service accounts，以及组件运行所需的 RBAC 权限。

安装清单不包含配置文件。MetalLB 的组件仍将启动，但将保持空闲状态，直到您 [定义和部署配置图](https://metallb.universe.tf/configuration/).

###  5.2 Kustomize 安装
您可以使用 [Kustomize](https://github.com/kubernetes-sigs/kustomize) 安装 MetalLB  通过指向远程 kustomization 文件：

```bash
# kustomization.yml
namespace: metallb-system

resources:
  - github.com/metallb/metallb//manifests?ref=v0.11.0
  - configmap.yml 
```
如果你想使用一个 `configMapGenerator` 对于配置文件，您想告诉 Kustomize 不要将哈希附加到配置映射，因为 MetalLB 正在等待名为的配置映射config （请参阅 [https://github.com/kubernetes-sigs/kustomize/blob/master/examples/generatorOptions.md](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/generatorOptions.md)):

```bash
# kustomization.yml
namespace: metallb-system

resources:
  - github.com/metallb/metallb//manifests?ref=v0.11.0

configMapGenerator:
- name: config
  files:
    - configs/config

generatorOptions:
 disableNameSuffixHash: true
```
###  5.3 Helm 安装
使用 Helm 图表存储库： [https://metallb.github.io/metallb](https://metallb.github.io/metallb)

```bash
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb
```
可以在安装时指定值文件。建议在 Helm 值中提供配置：

```bash
helm install metallb metallb/metallb -f values.yaml
```
MetalLB `CONFIGS`在设置`values.yaml`下`configInLine`：

```bash
configInline:
  address-pools:
   - name: default
     protocol: layer2
     addresses:
     - 198.51.100.0/24
```

##  6. 示例
kind创建kubernets

```bash
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

$ kind get  clusters 
kind
```


```bash
docker pull nginx:1.14.2
kind load docker-image nginx:1.14.2 --name kind
kubectl create deployment nginx:1.14.2 --image=nginx
kubectl expose deploy nginx --port 80 --type LoadBalancer
```
如果检查部署情况，则`external-ip`状态为`pending`

```bash
$ kubectl get svc nginx          
NAME    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx   LoadBalancer   10.96.255.195   <pending>     80:30681/TCP   46s
```
安装metal LB

```bash
$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml
namespace/metallb-system created


$ kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/controller created
podsecuritypolicy.policy/speaker created
serviceaccount/controller created
serviceaccount/speaker created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
role.rbac.authorization.k8s.io/config-watcher created
role.rbac.authorization.k8s.io/pod-lister created
role.rbac.authorization.k8s.io/controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/config-watcher created
rolebinding.rbac.authorization.k8s.io/pod-lister created
rolebinding.rbac.authorization.k8s.io/controller created
daemonset.apps/speaker created
deployment.apps/controller createdl


$ kubectl get all -n metallb-system
NAME                             READY   STATUS    RESTARTS   AGE
pod/controller-77c44876d-b8gsw   1/1     Running   0          42s
pod/speaker-wqxlq                1/1     Running   0          42s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   42s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           42s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-77c44876d   1         1         1       42sm
```
如果您查看pod，您将在应用清单之后创建两种类型的pod，即`controller`和`speaker`。 该控制器是集群范围的`MetalLB`控制器，负责IP分配。 `speaker`是一个`deamonset`，它将安装在集群中的每个节点上，并使用各种广告策略使用指定的ip发布服务。 

作为最后一步，您需要定义configmap。 根据环境的不同，可以选择Layer 2模式或BGP模式，也可以选择`external IP`范围。 在这个演示中，我将使用协议作为layer2，地址为`172.18.0.200-172.18.0.250`，因为我的Kubernetes节点在子网范围内运行(检查`kubectl get nodes -o wide`命令的输出)。

```bash
$ kubectl get nodes -o wide

NAME                 STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION     CONTAINER-RUNTIME

kind-control-plane   Ready    control-plane,master   24h   v1.21.1   172.18.0.2    <none>     

$ cat cm.yaml                                  
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: my-ip-space
      protocol: layer2
      addresses:
      - 172.18.0.200-172.18.0.250

$ kubectl create -f cm.yaml
```
如果您检查服务，您将看到metallb从您在configmap下定义的池中为它分配了`external ip`

```bash
$ kubectl get svc nginx

NAME  TYPE      CLUSTER-IP   EXTERNAL-IP  PORT(S)    AGE

nginx  LoadBalancer  10.96.131.55  172.18.0.200  80:30355/TCP  10m
```

