
![](https://img-blog.csdnimg.cn/cf5b39600a2142abb501daa6c2c437e3.png)



前天，kubevirt 更新了，尝鲜。

![](https://img-blog.csdnimg.cn/302b764469164e45bcae4e4541336889.png)

## 准备条件

- [二进制 Deploy Kubernetes v1.23.17](http://t.csdn.cn/f67XR)
- [Helm 部署 OpenEBS LocalPV 作为伸缩存储](https://blog.csdn.net/xixihahalelehehe/article/details/129967490)

## 下载介质

```bash
export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | sort -r | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
echo $VERSION
wget  https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
wget https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```
下载镜像入库

```bash
#!/bin/bash

# kubevirt组件版本
version='v1.1.0-alpha.0'

# 私有镜像仓库
registry='harbor.ghostwritten.com'

# 私有镜像仓库的namespace
namespace=kubevirt

kubevirtRegistry="quay.io/kubevirt"

readonly APPLIST=(
    virt-operator
    virt-api
    virt-controller
    virt-launcher
    virt-handler
)

for app in "${APPLIST[@]}"; do
    # 拉取镜像
    docker pull ${kubevirtRegistry}/${app}:${version}
    # 重命名
    docker tag ${kubevirtRegistry}/${app}:${version} ${registry}/${namespace}/${app}:${version}
    # 推送镜像
    docker push ${registry}/${namespace}/${app}:${version}
done
```


##. 修改 kubevirt-operator.yaml

```bash
        env:
        - name: VIRT_OPERATOR_IMAGE
          value: harbor.ghostwritten.com/kubevirt/virt-operator:v1.1.0-alpha.0
        - name: WATCH_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['olm.targetNamespaces']
        - name: KUBEVIRT_VERSION
          value: v1.1.0-alpha.0
        image: harbor.ghostwritten.com/kubevirt/virt-operator:v1.1.0-alpha.0
```

## 部署

```bash
kubectl apply -f kubevirt-operator.yaml
kubectl apply -f kubevirt-cr.yaml
kubectl -n kubevirt patch kubevirt kubevirt --type=merge --patch '{"spec":{"configuration":{"developerConfiguration":{"useEmulation":true}}}}'
```

## 查看

```bash
$ kubectl get all  -n kubevirt
Warning: kubevirt.io/v1 VirtualMachineInstancePresets is now deprecated and will be removed in v2.
NAME                                   READY   STATUS    RESTARTS   AGE
pod/virt-api-6bfc8548cf-tvb88          1/1     Running   0          9m24s
pod/virt-api-6bfc8548cf-w7bsd          1/1     Running   0          9m24s
pod/virt-controller-864d658788-6fsm4   1/1     Running   0          8m53s
pod/virt-controller-864d658788-h8n9j   1/1     Running   0          8m53s
pod/virt-handler-7gkwm                 1/1     Running   0          8m52s
pod/virt-handler-ptzzs                 1/1     Running   0          8m52s
pod/virt-handler-zwqsf                 1/1     Running   0          8m52s
pod/virt-operator-56bfbd6756-d9jww     1/1     Running   0          12m
pod/virt-operator-56bfbd6756-kf852     1/1     Running   0          12m

NAME                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubevirt-operator-webhook     ClusterIP   10.255.26.53     <none>        443/TCP   9m28s
service/kubevirt-prometheus-metrics   ClusterIP   None             <none>        443/TCP   9m28s
service/virt-api                      ClusterIP   10.255.48.169    <none>        443/TCP   9m28s
service/virt-exportproxy              ClusterIP   10.255.100.157   <none>        443/TCP   9m28s

NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/virt-handler   3         3         3       3            3           kubernetes.io/os=linux   8m53s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/virt-api          2/2     2            2           9m25s
deployment.apps/virt-controller   2/2     2            2           8m53s
deployment.apps/virt-operator     2/2     2            2           12m

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/virt-api-6bfc8548cf          2         2         2       9m25s
replicaset.apps/virt-controller-864d658788   2         2         2       8m53s
replicaset.apps/virt-operator-56bfbd6756     2         2         2       12m

NAME                            AGE   PHASE
kubevirt.kubevirt.io/kubevirt   10m   Deployed
```
或者

```bash
$ kubectl -n kubevirt wait kv kubevirt --for condition=Available
kubevirt.kubevirt.io/kubevirt condition met

```
某些KubeVirt功能默认情况下是禁用的，必须通过功能门启用。例如，禁用实时迁移和对虚拟机磁盘映像使用HostDisk。启用KubeVirt功能门可以通过更改现有的KubeVirt自定义资源并指定要启用的功能列表来完成。例如，您可以启用实时迁移和使用托管服务
## 安装 virtctl

下载 virctl
```bash
VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
echo ${ARCH}
wget https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}

```

```bash
chmod +x virtctl
sudo install virtctl /usr/local/bin
```


- [安装为 Krew 插件](https://blog.csdn.net/xixihahalelehehe/article/details/129838856)

```bash
kubectl krew install virt
```
输出：
```bash
Updated the local copy of plugin index.
Installing plugin: virt


Installed plugin: virt
\
 | Use this plugin:
 | 	kubectl virt
 | Documentation:
 | 	https://github.com/kubevirt/kubectl-virt-plugin
 | Caveats:
 | \
 |  | virt plugin is a wrapper for virtctl originating from the KubeVirt project. In order to use virtctl you will
 |  | need to have KubeVirt installed on your Kubernetes cluster to use it. See https://kubevirt.io/ for details
 |  |
 |  | See
 |  |
 |  |   https://kubevirt.io/user-guide/docs/latest/using-virtual-machines/graphical-and-console-access.html
 |  |
 |  | for a usage example
 | /
/
WARNING: You installed plugin "virt" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
```
## 部署 CDI
Containerized Data Importer（CDI）项目提供了用于使 PVC 作为 KubeVirt VM 磁盘的功能。建议同时部署 CDI：
下载地址：[https://github.com/kubevirt/containerized-data-importer/releases](https://github.com/kubevirt/containerized-data-importer/releases)
```bash

$ kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-operator.yaml
$ kubectl create -f https://github.com/kubevirt/containerized-data-importer/releases/download/$VERSION/cdi-cr.yaml
```

查看

```bash
kubectl get all  -n cdi
Warning: kubevirt.io/v1 VirtualMachineInstancePresets is now deprecated and will be removed in v2.
NAME                                   READY   STATUS    RESTARTS   AGE
pod/cdi-apiserver-6fd47b9fbc-kr4c6     1/1     Running   0          73s
pod/cdi-deployment-6bbbb86599-p5qqn    1/1     Running   0          87s
pod/cdi-operator-686f8954f9-xk6qx      1/1     Running   0          2m9s
pod/cdi-uploadproxy-7cdf94cb7f-v4xqw   1/1     Running   0          82s

NAME                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/cdi-api                  ClusterIP   10.255.133.175   <none>        443/TCP    73s
service/cdi-prometheus-metrics   ClusterIP   10.255.74.224    <none>        8080/TCP   87s
service/cdi-uploadproxy          ClusterIP   10.255.104.130   <none>        443/TCP    82s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cdi-apiserver     1/1     1            1           73s
deployment.apps/cdi-deployment    1/1     1            1           87s
deployment.apps/cdi-operator      1/1     1            1           2m10s
deployment.apps/cdi-uploadproxy   1/1     1            1           82s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/cdi-apiserver-6fd47b9fbc     1         1         1       73s
replicaset.apps/cdi-deployment-6bbbb86599    1         1         1       87s
replicaset.apps/cdi-operator-686f8954f9      1         1         1       2m10s
replicaset.apps/cdi-uploadproxy-7cdf94cb7f   1         1         1       82s

```


## 创建 pvc

```bash
$ kubectl -n cdi get svc -l cdi.kubevirt.io=cdi-uploadproxy
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
cdi-uploadproxy   ClusterIP   10.96.209.203   <none>        443/TCP   108m


$ virtctl image-upload \
    pvc iso-win2019 \
  --force-bind \
  --storage-class="openebs-hostpath" \
  --access-mode=ReadWriteOnce \
  --image-path='SW_DVD9_Win_Server_STD_CORE_2019_1809.19_64Bit_ChnSimp_DC_STD_MLF_X23-31940.ISO' \
  --size=10G \
  --uploadproxy-url=https://10.43.100.62  \
  --wait-secs=240
```


##  创建 win 持久卷

```bash
$ cat winhd.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: winhd
  annotations:
    volume.beta.kubernetes.io/storage-provisioner: openebs.io/local
    volume.kubernetes.io/storage-provisioner: openebs.io/local
    volume.kubernetes.io/selected-node: rke-master01
    pv.kubernetes.io/bound-by-controller: "yes"
  finalizers:
  - kubernetes.io/pvc-protection
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 15Gi
  storageClassName: openebs-hostpath
  volumeMode: Filesystem

```
创建

```bash
[root@rke-master01 kubevirt]# kubectl get pvc,pv
NAME                                                                   STATUS   VOLUME                                     
persistentvolumeclaim/winhd                                            Bound    pvc-90f57608-6525-420e-88b2-3adeb16d3389   15Gi       RWO            openebs-hostpath   7h23m

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                    STORAGECLASS       REASON   AGE
persistentvolume/pvc-90f57608-6525-420e-88b2-3adeb16d3389   15Gi       RWO            Delete           Bound    default/winhd                                            openebs-hostpath            7h22m

```

##  创建 vms

```bash
[root@rke-master01 kubevirt]# cat windows.yaml 
apiVersion: kubevirt.io/v1alpha3
kind: VirtualMachine
metadata:
  name: win2019
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/domain: win2019
    spec:
      domain:
        cpu:
          cores: 4
        devices:
          disks:
          - bootOrder: 1
            cdrom:
              bus: sata
            name: cdromiso
          - disk:
              bus: virtio
            name: harddrive
          - cdrom:
              bus: sata
            name: virtiocontainerdisk
          interfaces:
          - masquerade: {}
            model: e1000
            name: default
        machine:
          type: q35
        resources:
          requests:
            memory: 4G
      networks:
      - name: default
        pod: {}
      volumes:
      - name: cdromiso
        persistentVolumeClaim:
          claimName: iso-win2019-scratch
      - name: harddrive
        persistentVolumeClaim:
          claimName: winhd
      - containerDisk:
          image: kubevirt/virtio-container-disk
        name: virtiocontainerdisk

```

```bash
$ kubectl get vms
NAME      AGE     STATUS               READY
win2019   7h23m   ErrorUnschedulable   False

$ virtctl start win2019 
$ kubectl virt start win2019 
```


参考：

- [https://github.com/kubevirt/containerized-data-importer/releases](https://github.com/kubevirt/containerized-data-importer/releases)
- [https://kubevirt.io/labs/kubernetes/lab1](https://kubevirt.io/labs/kubernetes/lab1)
- [https://kubevirt.io/labs/kubernetes/lab2](https://kubevirt.io/labs/kubernetes/lab2)
- [https://kubevirt.io/labs/kubernetes/lab3](https://kubevirt.io/labs/kubernetes/lab3)
- [https://kubevirt.io/quickstart_kind/](https://kubevirt.io/quickstart_kind/)
