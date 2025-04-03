
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2e9623e62ebf4907bdc44499c617d0f3.jpeg)



## 1. 简介

提供备份和恢复在任何Kubernetes集群上运行的Rancher应用程序的能力
此图表支持捕获Rancher应用程序的备份并从这些备份恢复。此图表可用于将Rancher从一个Kubernetes集群迁移到另一个Kubernetes集群。
有关如何使用该功能的更多信息，请参阅我们的[文档](https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/backup-restore-and-disaster-recovery)。
此图表安装以下组件：

- backup-restore-operator：该操作器负责备份 Rancher 从本地集群创建和管理的所有 Kubernetes 资源和 CRD。它通过查询 Kubernetes API 服务器来收集这些资源，将所有资源打包以创建 tarball 文件并将其保存在配置的备份存储位置。
该操作器可以配置为将备份存储在与 S3 兼容的对象存储（例如 AWS S3 和 MinIO）和持久卷中。在部署期间，您可以创建默认存储位置，但始终可以选择在每次备份时覆盖默认存储位置，但仅限于使用与 S3 兼容的对象存储。
它保留所有资源的所有者引用，从而保持对象之间的依赖关系。
该操作器提供加密支持，在将用户指定的资源保存到备份文件之前对其进行加密。它使用与启用 Kubernetes 静态加密相同的加密配置。


Backup - 备份是一种 CRD（备份），它定义何时进行备份、在何处存储备份以及使用哪种加密（可选）。备份可以临时进行，也可以安排在间隔内进行。
Restore - 恢复是一种 CRD（恢复），它定义使用哪个备份来将 Rancher 应用程序恢复到。

## 2. 注意

从Kubernetes v1.25开始，Pod安全策略已从Kubernetes API中删除。
​
因此，在升级到Kubernetes v1.25之前（或在Kubernetes v1.25+集群中进行全新安装时），用户需要对该图表执行就地升级，如果global.cattle.psp.enabled之前设置为true，则将其设置为false。



在这个图表版本中，以前与任何PSP资源相关的任何字段都被删除了，取而代之的是一个全局字段：global.cott.psp.enabled

如果在通过Helm升级删除PSP之前将集群升级到Kubernetes v1.25+（即使您手动清理资源），它将使Helm版本在集群内处于中断状态，这样进一步的Helm操作将无法工作（Helm卸载，Helm升级等）。
如果你的图表卡在这个状态，请咨询牧场主文档如何清理你的头盔释放秘密。
​
在将global.cattle.psp.enabled设置为false时，图表将从集群中删除所有为其部署的PSP资源。这是此图表的默认设置。
​
作为PSP的替代品，应使用Pod Security Admission。有关如何配置图表发布名称空间以使用新的Pod Security Admission并应用Pod Security Standards的更多详细信息，请参阅Rancher文档。

## 3. 在线安装


```bash
helm repo add rancher-chart https://charts.rancher.io
helm repo update
helm install rancher-backup-crd rancher-chart/rancher-backup-crd -n cattle-resources-system --create-namespace
helm install rancher-backup rancher-chart/rancher-backup -n cattle-resources-system
```

##  4. 下载介质

```bash
helm fetch rancher-chart/rancher-backup-crd
helm fetch rancher-chart/rancher-backup
```


```bash
$ ls
rancher-backup-103.0.3+up4.0.3.tgz  rancher-backup-crd-103.0.3+up4.0.3.tgz
```

获取镜像

```bash
helm template rancher-backup-crd rancher-chart/rancher-backup-crd | grep 'image:' | awk '{print $2}' | sed 's/\"//g' 
helm template rancher-backup rancher-chart/rancher-backup | grep 'image:' | awk '{print $2}' | sed 's/\"//g' 
```
所需镜像
```bash
rancher/backup-restore-operator:v4.0.3
rancher/kubectl:v1.27.16
```

## 5. 离线安装

```bash
helm install rancher-backup-crd rancher-backup-crd-103.0.3+up4.0.3.tgz -n cattle-resources-system --create-namespace
helm install rancher-backup rancher-backup-103.0.3+up4.0.3.tgz -n cattle-resources-system --set image.repository=harbor.bsgchina.com/rancher/backup-restore-operator --set global.kubectl.repository=harbor.bsgchina.com/rancher/kubectl --set global.kubectl.tag=v1.27.16
```

```bash
$ kubectl get pod   -n cattle-resources-system 
NAME                              READY   STATUS    RESTARTS   AGE
rancher-backup-787bd4b98b-dwff8   1/1     Running   0          41s
```

## 6. 卸载

```bash
kubectl delete job.batch/rancher-backup-patch-sa   -n cattle-resources-system
helm uninstall rancher-backup                    -n      cattle-resources-system
```

