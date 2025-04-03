![](https://i-blog.csdnimg.cn/blog_migrate/34a9ed7b2a373b3344a944235bd54407.png)






## 1. 什么是 OpenEBS LocalPV？
[OpenEBS](https://github.com/openebs/openebs) LocalPV 是一个 Kubernetes 存储插件，它将本地磁盘转化为 Kubernetes 存储卷。它允许您使用本地存储作为动态分配的存储卷，从而提高了存储的性能和可靠性。使用 OpenEBS LocalPV，您可以轻松地将本地存储作为动态分配的存储卷，而无需使用任何外部存储解决方案。

如何使用大体需要这样一个流程：
- **安装 OpenEBS**：您可以使用其 Helm chart 在 Kubernetes 集群上安装 OpenEBS，该 Helm chart 可以在 OpenEBS GitHub 存储库中找到。

- **创建一个 StorageClass**：创建一个使用 OpenEBS LocalPV 存储插件的 Kubernetes StorageClass。 此 StorageClass 将用于创建可由您的应用程序用于存储数据的 PersistentVolumeClaims (PVC)。

- **创建 PVC**：使用您在上一步中创建的 StorageClass 创建 PersistentVolumeClaim (PVC)。 此 PVC 将从 OpenEBS LocalPV 存储插件请求存储。

- **使用 PVC**：创建 PVC 后，您可以在应用程序中使用它来存储数据。 您可以将 PVC 作为卷安装在应用程序的容器中。


有些应用部署需要依赖集群已有的存储类型（StorageClass），若集群还没有准备存储类型（StorageClass），可参考本文档，在 K8s 集群中安装 OpenEBS 并创建 LocalPV 的存储类型，从而可以在集群快速安装测试 KubeSphere。

注意：基于 OpenEBS 创建 LocalPV 的存储类型仅适用于开发测试环境，不建议在生产环境使用。生产环境建议准备符合 Kubernetes 要求的持久化存储（如 GlusterFS、Ceph、NFS、Neonsan 等分布式存储，或云上的块存储），然后再创建对应的 StorageClass

## 2. 以本地目录 /var/openebs/local 作为存储
### 2.1  前提条件

安装好 kubernetes 集群：
- [kubespray v2.21.0 部署 kubernetes v1.24.0 集群](https://blog.csdn.net/xixihahalelehehe/article/details/129952069)

关于第二个前提条件，是由于安装 OpenEBS 时它有一个初始化的 Pod 需要在 master 节点启动并创建 PV 给 kubernetes 的有状态应用挂载。因此，若您的 master 节点存在 `Taint`，建议在安装 OpenEBS 之前手动取消 Taint，待 OpenEBS 与 KubeSphere 安装完成后，再对 master 打上 Taint，以下步骤供参考：


例如本示例有一个 master 节点，节点名称即 master，可通过以下命令查看节点名称：

```bash
$ kubectl get node -o wide
NAME        STATUS   ROLES           AGE   VERSION    INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                      KERNEL-VERSION                 CONTAINER-RUNTIME
109node     Ready    <none>          42h   v1.24.10   192.168.10.109   <none>        Rocky Linux 9.1 (Blue Onyx)   5.14.0-162.18.1.el9_1.x86_64   containerd://1.6.15
110node     Ready    <none>          42h   v1.24.10   192.168.10.110   <none>        Rocky Linux 9.1 (Blue Onyx)   5.14.0-162.18.1.el9_1.x86_64   containerd://1.6.15
111node     Ready    <none>          42h   v1.24.10   192.168.10.111   <none>        Rocky Linux 9.1 (Blue Onyx)   5.14.0-162.18.1.el9_1.x86_64   containerd://1.6.15
control01   Ready    <none>          42h   v1.24.10   192.168.10.107   <none>        Rocky Linux 9.1 (Blue Onyx)   5.14.0-162.18.1.el9_1.x86_64   containerd://1.6.15
master01    Ready    control-plane   42h   v1.24.10   192.168.10.106   <none>        Rocky Linux 9.1 (Blue Onyx)   5.14.0-162.18.1.el9_1.x86_64   containerd://1.6.15
prom01      Ready    <none>          42h   v1.24.10   192.168.10.108   <none>        Rocky Linux 9.1 (Blue Onyx)   5.14.0-162.18.1.el9_1.x86_64   containerd://1.6.15
```
确认 master 节点是否有 Taint，如下看到 master 节点有 Taint。

```bash
$ kubectl describe node master01 | grep Taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```
去掉 master 节点的 Taint：

```bash
$ kubectl taint nodes master01 node-role.kubernetes.io/master:NoSchedule-
```

### 2.2 安装 OpenEBS
配置 openebs-localpv 源

```bash
ansible all -m shell -a "mkdir  -p /var/openebs/local"
helm repo add openebs-localpv https://openebs.github.io/dynamic-localpv-provisioner
helm repo update
```
默认安装

```bash
helm upgrade openebs-localpv-hostpath openebs-localpv/localpv-provisioner --namespace openebs --create-namespace --install
```
除此之外 还可以通过 kubectl 命令安装：

```bash
kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml
```
安装 OpenEBS 后将自动创建 2 个 StorageClass，查看创建的 StorageClass：

```bash
$ k get sc -n openebs
NAME               PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device     openebs.io/local   Delete          WaitForFirstConsumer   false                  49m
openebs-hostpath   openebs.io/local   Delete          WaitForFirstConsumer   false                  49m
```
如下将 openebs-hostpath设置为 默认的 StorageClass：

```bash
$ kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/openebs-hostpath patched

$ k get sc -n openebs
NAME                         PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device               openebs.io/local   Delete          WaitForFirstConsumer   false                  51m
openebs-hostpath (default)   openebs.io/local   Delete          WaitForFirstConsumer   false                  51m
```
至此，OpenEBS 的 LocalPV 已作为默认的存储类型创建成功。可以通过命令 kubectl get pod -n openebs来查看 OpenEBS 相关 Pod 的状态，若 Pod 的状态都是 running，则说明存储安装成功。

```bash
$ kubectl get pod -n openebs
NAME                                                             READY   STATUS    RESTARTS   AGE
openebs-localpv-hostpath-localpv-provisioner-5cdbcc6f6f-xk2m8    1/1     Running   0          53m
openebs-localpv-hostpath-openebs-ndm-2lrj5                       1/1     Running   0          53m
openebs-localpv-hostpath-openebs-ndm-7zdds                       1/1     Running   0          53m
openebs-localpv-hostpath-openebs-ndm-g59zk                       1/1     Running   0          53m
openebs-localpv-hostpath-openebs-ndm-mplqt                       1/1     Running   0          53m
openebs-localpv-hostpath-openebs-ndm-nsd6x                       1/1     Running   0          53m
openebs-localpv-hostpath-openebs-ndm-operator-56859f9f79-cztm8   1/1     Running   0          53m
```

### 2.3  测试
helm 创建 influxdb 会创建 pvc 申请绑定pv，我们可以检查sc 能否为influxdb 自动分配 pv。

```bash
helm upgrade my-release-influxdb2 bitnami/influxdb -n influxdb --create-namespace --install
```
输出：

```bash
$ k get pv,pvc -n influxdb
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                               STORAGECLASS       REASON   AGE
persistentvolume/pvc-d7963db4-9198-4ec3-a821-f035cafcc7e7   8Gi        RWO            Delete           Bound    influxdb/my-release-influxdb2       openebs-hostpath            14m

NAME                                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
persistentvolumeclaim/my-release-influxdb2   Bound    pvc-d7963db4-9198-4ec3-a821-f035cafcc7e7   8Gi        RWO            openebs-hostpath   26m
```

## 3. 以磁盘 /dev/sdb 作为存储

- [https://github.com/upmio/upm-deploy/tree/main/addons/openebs-lvmlocalpv](https://github.com/upmio/upm-deploy/tree/main/addons/openebs-lvmlocalpv)

### 预备条件

```bash
$ kubectl get node
NAME            STATUS   ROLES           AGE    VERSION
kube-master01   Ready    control-plane   154d   v1.26.5
kube-master02   Ready    control-plane   154d   v1.26.5
kube-master03   Ready    control-plane   154d   v1.26.5
kube-node01     Ready    <none>          154d   v1.26.5
kube-node02     Ready    <none>          154d   v1.26.5
kube-node03     Ready    <none>          154d   v1.26.5
kube-node04     Ready    <none>          154d   v1.26.5
```

### 3.1 创建VolumeGroup
注意⚠️：每个数据节点都必须保证VolumeGroup 存在，才能部署 openebs-lvmlocalpv。

在LVM PV的工作节点，创建对应与VG_NAME同名的VolumeGroup，用于供应Storageclass的本地PV资源。

dev_name 是磁盘设备名称，例如 /dev/sdb。

```bash
pvcreate /dev/sdb
vgcreate local_vg_HDD /dev/sdb
```

### 3.2 设置必要的环境变量
- `OPENEBS_CONTROLLER_NODE_NAMES`：指定安装controller pod的节点名称，节点名称可以使用","作为分隔符，表示多个节点名称，安装程序会对节点进行label固定安装节点。
- `OPENEBS_DATA_NODE_NAMES`：指定LVM PV的工作负载节点名称，节点名称可以使用","作为分隔符，表示多个节点名称，安装程序会对节点进行label固定安装节点。
- `OPENEBS_STORAGECLASS_NAME`：指定Storageclass名称。
- `OPENEBS_VG_NAME`：指定Storageclass对应的VolumeGroup名称。
- `OPENEBS_KUBE_NAMESPACE`：指定安装命名空间，非必填项，默认值为openebs。

```bash
export OPENEBS_CONTROLLER_NODE_NAMES="kube-node01"
export OPENEBS_DATA_NODE_NAMES="kube-node01,kube-node02,kube-node03,kube-node04"
export OPENEBS_STORAGECLASS_NAME="openebs-lvmsc-hdd"
export OPENEBS_VG_NAME="local_vg_HDD"
export OPENEBS_CREATE_STORAGECLASS="true"
export OPENEBS_KUBE_NAMESPACE="openebs"
```

### 3.3 运行安装脚本
> 注意⚠️：如果找不到 Helm3，将自动安装。
>注意⚠️：安装脚本会对指定节点进行添加label的操作。
>注意⚠️：确保集群可以下载访问github内容，否则需要手动下载两个文件，再分别执行。
- [https://raw.githubusercontent.com/upmio/upm-deploy/main/addons/openebs-lvmlocalpv/install_el7.sh](https://raw.githubusercontent.com/upmio/upm-deploy/main/addons/openebs-lvmlocalpv/install_el7.sh) 
- [https://raw.githubusercontent.com/upmio/upm-deploy/main/addons/openebs-lvmlocalpv/yaml/storageclass.yaml](https://raw.githubusercontent.com/upmio/upm-deploy/main/addons/openebs-lvmlocalpv/yaml/storageclass.yaml)


运行安装脚本

```bash
# BASH
curl -sSL https://raw.githubusercontent.com/upmio/upm-deploy/main/addons/openebs-lvmlocalpv/install_el7.sh | sh -
```

等几分钟。 如果所有 openebs-lvmlocalpv pod 都在运行，则 openebs-lvmlocalpv 将成功安装。

```bash
$  helm ls -n openebs
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
openebs-lvmlocalpv      openebs         1               2023-12-17 10:17:03.74864032 -0500 EST  deployed        lvm-localpv-1.4.0       1.4.0  

$ kubectl get pod -n openebs
NAME                                          READY   STATUS    RESTARTS   AGE
openebs-lvmlocalpv-lvm-localpv-controller-0   5/5     Running   0          28m
openebs-lvmlocalpv-lvm-localpv-node-45b69     2/2     Running   0          28m
openebs-lvmlocalpv-lvm-localpv-node-hpgft     2/2     Running   0          28m
openebs-lvmlocalpv-lvm-localpv-node-vzk8l     2/2     Running   0          28m
openebs-lvmlocalpv-lvm-localpv-node-xgstx     2/2     Running   0          28m

$ kubectl get sc
NAME                PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-lvmsc-hdd   local.csi.openebs.io   Delete          WaitForFirstConsumer   true                   21m
```

### 3.4 使用 Helm 卸载 Charts

```bash
# Helm
helm uninstall openebs-lvmlocalpv -n openebs

# clean storageClass
kubectl delete storageclasses "${OPENEBS_STORAGECLASS_NAME}"
```

这将删除与 Charts 关联的所有 Kubernetes 组件并删除发布。

参考：

 - [https://openebs.io/docs](https://openebs.io/docs)
 - [OpenEBS Local PV Hostpath User Guide](https://openebs.io/docs/user-guides/localpv-hostpath#install)
 - [https://openebs.github.io/charts/](https://openebs.github.io/charts/)
 - [OpenEBS local-pv-hostpath 实践](https://weiliang-ms.github.io/wl-awesome/2.%E5%AE%B9%E5%99%A8/k8s/storage/OpenEBS.html#local-pv-hostpath%E5%AE%9E%E8%B7%B5)
