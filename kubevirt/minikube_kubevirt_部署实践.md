

## 准备条件

-  [Ubuntu 18.04 通过 Minikube 安装 Kubernetes v1.20](http://t.csdn.cn/RxECj)

```bash
minikube start --cni=flannel
```

添加 alias

```bash
alias kubectl='minikube kubectl --'
```
安装 kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
VERSION=$(minikube kubectl version | head -1 | awk -F', ' {'print $3'} | awk -F':' {'print $2'} | sed s/\"//g)
sudo install ${HOME}/.minikube/cache/linux/${VERSION}/kubectl /usr/local/bin
```

## 选项

### 多节点 Minikube
Minikube 支持向集群添加额外节点。这对于在 minikube 上试验 KubeVirt 很有帮助，因为节点关联或实时迁移等一些操作需要多个集群节点来演示。

### 容器网络接口
默认情况下，minikube 使用虚拟机设备或容器设置 kubernetes 集群。对于单节点设置，本地网络连接就足够了。在涉及多个节点的情况下，即使在同一主机上使用容器或虚拟机，kubernetes 也需要定义一个共享网络，以允许一台主机上的 pod 与另一台主机上的 pod 进行通信。为此，minikube 支持许多容器网络接口（CNI）插件， 其中最简单的是flannel。

### 更新 minikube 启动命令
要让 minikube 在两个节点上使用 flannel CNI 插件启动，请更改 minikube 启动命令：

```bash
minikube start --nodes=2 --cni=flannel
```

> 核心 DNS 竞争条件
据报告 ，多节点 minikube 中的pod coredns提供了错误的 IP 地址。如果发生这种情况，kubevirt 将无法正确安装。要解决此问题，请coredns从 kube-system 命名空间中删除 pod，并在 minikube 中禁用/启用 kubevirt 插件。


## 部署 KubeVirt

### 一键安装

```bash
minikube addons enable kubevirt
```

### yaml 安装

```bash
export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | sort -r | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
echo $VERSION
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
```
> 嵌套虚拟化
如果 minikube 集群在虚拟机上运行，​​请考虑启用嵌套虚拟化。请按照此处描述的说明进行操作。如果由于任何原因无法启用嵌套虚拟化，请按如下方式启用 KubeVirt 模拟：

```bash
kubectl -n kubevirt patch kubevirt kubevirt --type=merge --patch '{"spec":{"configuration":{"developerConfiguration":{"useEmulation":true}}}}'
```
再次用于kubectl部署 KubeVirt 自定义资源定义：


```bash
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```
## 验证组件
默认情况下，KubeVirt 将部署 7 个 Pod、3 个服务、1 个守护进程集、3 个部署应用程序、3 个副本集。

检查部署情况：

```bash
kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.phase}"
```
检查组件：

```bash
$ kubectl get all -n kubevirt
Warning: kubevirt.io/v1 VirtualMachineInstancePresets is now deprecated and will be removed in v2.
NAME                                  READY   STATUS    RESTARTS      AGE
pod/virt-api-dc9c659d5-wt6zt          1/1     Running   0             46h
pod/virt-controller-c97c96576-q55cg   1/1     Running   0             46h
pod/virt-controller-c97c96576-rglgv   1/1     Running   1 (45h ago)   46h
pod/virt-handler-rwqtp                1/1     Running   0             46h
pod/virt-operator-74bfd88c66-sndk5    1/1     Running   1 (45h ago)   46h
pod/virt-operator-74bfd88c66-zzkw8    1/1     Running   0             46h

NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubevirt-operator-webhook     ClusterIP   10.103.206.79   <none>        443/TCP   46h
service/kubevirt-prometheus-metrics   ClusterIP   None            <none>        443/TCP   46h
service/virt-api                      ClusterIP   10.110.53.201   <none>        443/TCP   46h
service/virt-exportproxy              ClusterIP   10.109.84.174   <none>        443/TCP   46h

NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/virt-handler   1         1         1       1            1           kubernetes.io/os=linux   46h

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/virt-api          1/1     1            1           46h
deployment.apps/virt-controller   2/2     2            2           46h
deployment.apps/virt-operator     2/2     2            2           46h

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/virt-api-dc9c659d5          1         1         1       46h
replicaset.apps/virt-controller-c97c96576   2         2         2       46h
replicaset.apps/virt-operator-74bfd88c66    2         2         2       46h

NAME                            AGE   PHASE
kubevirt.kubevirt.io/kubevirt   46h   Deployed

$ kubectl get crd |grep kubevirt
kubevirts.kubevirt.io                                         2023-09-17T15:55:59Z
migrationpolicies.migrations.kubevirt.io                      2023-09-17T15:57:26Z
virtualmachineclones.clone.kubevirt.io                        2023-09-17T15:57:27Z
virtualmachineclusterinstancetypes.instancetype.kubevirt.io   2023-09-17T15:57:26Z
virtualmachineclusterpreferences.instancetype.kubevirt.io     2023-09-17T15:57:27Z
virtualmachineexports.export.kubevirt.io                      2023-09-17T15:57:27Z
virtualmachineinstancemigrations.kubevirt.io                  2023-09-17T15:57:24Z
virtualmachineinstancepresets.kubevirt.io                     2023-09-17T15:57:23Z
virtualmachineinstancereplicasets.kubevirt.io                 2023-09-17T15:57:24Z
virtualmachineinstances.kubevirt.io                           2023-09-17T15:57:23Z
virtualmachineinstancetypes.instancetype.kubevirt.io          2023-09-17T15:57:26Z
virtualmachinepools.pool.kubevirt.io                          2023-09-17T15:57:26Z
virtualmachinepreferences.instancetype.kubevirt.io            2023-09-17T15:57:26Z
virtualmachinerestores.snapshot.kubevirt.io                   2023-09-17T15:57:25Z
virtualmachines.kubevirt.io                                   2023-09-17T15:57:24Z
virtualmachinesnapshotcontents.snapshot.kubevirt.io           2023-09-17T15:57:25Z
virtualmachinesnapshots.snapshot.kubevirt.io                  2023-09-17T15:57:25Z
```
镜像列表：

```bash
$ kubectl get pod -n kubevirt -oyaml |grep image: | awk '{print $NF}' | sort -r | uniq
quay.io/kubevirt/virt-operator:v1.0.0
quay.io/kubevirt/virt-launcher:v1.0.0
quay.io/kubevirt/virt-handler:v1.0.0
quay.io/kubevirt/virt-controller:v1.0.0
quay.io/kubevirt/virt-api:v1.0.0
```

使用 `minikube KubeVirt` 插件时，检查 `kubevirt-install-manager pod` 的日志：

```bash
kubectl logs pod/kubevirt-install-manager -n kube-system
```

##  Virtctl
KubeVirt 提供了一个名为virtctl的附加二进制文件，用于快速访问虚拟机的串行和图形端口，并处理启动/停止操作。
virtctl可以从 KubeVirt github 页面的发布页面检索。

运行以下命令：

```bash
VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
echo ${ARCH}
curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}
chmod +x virtctl
sudo install virtctl /usr/local/bin
```

