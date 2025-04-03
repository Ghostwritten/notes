
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bc9dab89a3af475b9ef93a88ab0468be.png)

# 准备条件
# 1.1 ocp 集群
确保集群状态正常并包含 Master 和 Worker 节点。



```plain
$ oc get node
NAME      STATUS   ROLES                  AGE   VERSION
master1   Ready    control-plane,master   20h   v1.27.16+03a907c
master2   Ready    control-plane,master   21h   v1.27.16+03a907c
master3   Ready    control-plane,master   21h   v1.27.16+03a907c
worker1   Ready    worker                 20h   v1.27.16+03a907c
worker2   Ready    worker                 20h   v1.27.16+03a907c
worker3   Ready    worker                 20h   v1.27.16+03a907c
```

## 1.2 create data vgs
在每个 Worker 节点上，将独立磁盘配置为数据卷组：

```plain
[root@worker1 ~]# fdisk -l
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: Virtual disk    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: Virtual disk    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 3D2E62C9-9F50-4990-BAA7-04EA5FAEF718

Device       Start       End   Sectors  Size Type
/dev/sda1     2048      4095      2048    1M BIOS boot
/dev/sda2     4096    264191    260096  127M EFI System
/dev/sda3   264192   1050623    786432  384M Linux filesystem
/dev/sda4  1050624 209715166 208664543 99.5G Linux filesystem
[root@worker1 ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@worker1 ~]# vgcreate data-vg /dev/sdb
  Volume group "data-vg" successfully created
[root@worker1 ~]# vgs
  VG      #PV #LV #SN Attr   VSize    VFree   
  data-vg   1   0   0 wz--n- <100.00g <100.00g
```

# install openebs

通过网盘分享的文件：openEBS_1.6.0.tar.gz
链接: https://pan.baidu.com/s/1TSRAwmi_Hbwk0l7sTp-ZvQ?pwd=bydu 提取码: bydu 
--来自百度网盘超级会员v5的分享

介质下载：[https://download.csdn.net/download/xixihahalelehehe/90151265](https://download.csdn.net/download/xixihahalelehehe/90151265)
## 2.1 解压介质
```plain
$ tar zxvf openEBS_1.6.0.tar.gz
$ cd openEBS
$ ls
helm  images  images.sh  lvm-localpv  lvm-localpv-1.6.0.tgz  ocp-openebs-install.sh  openebs-images.txt  openebs-mirror.yaml  openebs_vars.sh
```



## 2.2 推送镜像
+ 修改镜像仓库名：registry.ocp.local:8443
+ 修改项目名：openebs
+ 修改镜像列表名：openebs-images.txt

```plain
$ vim images.sh
head images.sh 
#!/bin/bash

action=$1
registry_name='registry.ocp.local:8443'
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"
project='openebs'

images_list='openebs-images.txt'
images_dir="${BASE_DIR}/images"
```



执行解压并推送

```plain
sh images.sh load
sh images.sh push
```



## 2.3 映射镜像仓库源


更新或创建 imagetagmirrorset 会导致所有节点crio服务重启。

```plain
# 更新前检查mcp是否正常。
$ oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master   rendered-master-b7c777b236e6ac9c002c99137b4a8847   True      False      False      3              3                   3                     0                      155m
worker   rendered-worker-e1591d33fa7854335f42067f277014d2   True      False      False      3              3                   3                     0                      155m

$ oc apply -f openebs-mirror.yaml 
imagetagmirrorset.config.openshift.io/openebs-mirror created

#跟踪 mcp 状态。
$ oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master   rendered-master-b7c777b236e6ac9c002c99137b4a8847   False     True       False      3              0                   0                     0                      156m
worker   rendered-worker-e1591d33fa7854335f42067f277014d2   False     True       False      3              0                   0 0                      156m

$ watch oc get node                                                                           registry.ocp.local: Tue Dec 17 19:17:56 2024

NAME      STATUS                     ROLES                  AGE    VERSION
master1   Ready                      control-plane,master   148m   v1.27.16+03a907c
master2   Ready                      control-plane,master   166m   v1.27.16+03a907c
master3   Ready,SchedulingDisabled   control-plane,master   167m   v1.27.16+03a907c
worker1   Ready                      worker                 150m   v1.27.16+03a907c
worker2   Ready,SchedulingDisabled   worker                 150m   v1.27.16+03a907c
worker3   Ready                      worker                 149m   v1.27.16+03a907c
```

等待所有节点Ready ，没有SchedulingDisabled，并且oc get mcp更新完毕，如下：

```plain
$ oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master   rendered-master-b7c777b236e6ac9c002c99137b4a8847   True      False      False      3              3                   3                     0                      155m
worker   rendered-worker-e1591d33fa7854335f42067f277014d2   True      False      False      3              3                   3                     0                      155m
[root@registry openebs]# oc get node
```

跟踪更新状态。

```plain
$ oc describe co/machine-config |grep -A 2  Extension
  Extension:
    Master:  1 (ready 1) out of 3 nodes are updating to latest configuration rendered-master-ee47d033f70a27fd37bb3ded4ab3b36c
    Worker:  2 (ready 2) out of 3 nodes are updating to latest configuration rendered-worker-c31a696b0a23a0ffe31236ba38526198
```



结束状态。

```plain
$ oc describe co/machine-config |grep -A 2  Extension
  Extension:
    Master:  all 3 nodes are at latest configuration rendered-master-18624ec71717d62727713f00642866fc
    Worker:  all 3 nodes are at latest configuration rendered-worker-8eff47b3935dbe6762a38a174e8e7ddb
```

## 2.4 配置权限
```plain
$ oc get crd 
....
volumesnapshotclasses.snapshot.storage.k8s.io                     2024-08-23T02:46:00Z
volumesnapshotcontents.snapshot.storage.k8s.io                    2024-08-23T02:46:00Z
volumesnapshots.snapshot.storage.k8s.io                           2024-08-23T02:46:00Z


$ oc edit crd volumesnapshotclasses.snapshot.storage.k8s.io 
$ oc edit crd volumesnapshotcontents.snapshot.storage.k8s.io 
$ oc edit crd volumesnapshots.snapshot.storage.k8s.io     
在编辑器中，找到 metadata 部分并添加如下内容：
metadata:
  labels:
    app.kubernetes.io/managed-by: "Helm"
  annotations:
    meta.helm.sh/release-name: "openebs-lvmlocalpv"
    meta.helm.sh/release-namespace: "openebs"

$ oc edit scc restricted
现在设置allowHostDirVolumePlugin：true并保存更改。文件应该如下所示：

 allowHostDirVolumePlugin: true
 allowHostIPC: false
 allowHostNetwork: false
 allowHostPID: false
 allowHostPorts: false
 allowPrivilegedContainer: false
 allowedCapabilities: []
 allowedFlexVolumes: []
 ....
```





## 2.5 安装 [**<font style="color:rgb(31, 35, 40);">lvm-localpv</font>**](https://github.com/openebs/lvm-localpv)
定制环境变量

```plain
$ vim openebs_vars.sh 
export OFFLINE_INSTALL="true"
export OPENEBS_CHART_DIR="lvm-localpv"
export OPENEBS_STORAGECLASS_YAML="storageclass.yaml"
export OPENEBS_CONTROLLER_NODE_NAMES="worker1"
export OPENEBS_DATA_NODE_NAMES="worker1,worker2,worker3"
export OPENEBS_STORAGECLASS_NAME="openebs-lvmsc-hdd"
export OPENEBS_VG_NAME="data-vg"
export OPENEBS_CREATE_STORAGECLASS="true"
export OPENEBS_KUBE_NAMESPACE="openebs"
```



声明变量

```plain
$ source openebs_vars.sh 
```

  
安装[**<font style="color:rgb(31, 35, 40);">lvm-localpv</font>**](https://github.com/openebs/lvm-localpv)

```plain
$ sh ocp-openebs-install.sh
```



打开另一个终端跟踪部署流程。

```plain
$ oc get pod -n openebso
NAME                                                        READY   STATUS    RESTARTS   AGE
openebs-lvmlocalpv-lvm-localpv-controller-f9b96d86c-nzjpc   5/5     Running   0          146m
openebs-lvmlocalpv-lvm-localpv-node-c9w9q                   2/2     Running   0          108m
openebs-lvmlocalpv-lvm-localpv-node-dk4ls                   2/2     Running   0          100m
openebs-lvmlocalpv-lvm-localpv-node-phfx2                   2/2     Running   0          99m
```

pod运行正常后检查storageclass

```plain
$ oc get sc
NAME                PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-lvmsc-hdd   local.csi.openebs.io   Delete          WaitForFirstConsumer   true                   18h
```

