
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c46b95d40b864122a3661f2c6145a43a.jpeg)



## 1. 概述
Milvus Operator是一个解决方案，可帮助您部署和管理完整的Milvus服务堆栈，以目标Kubernetes（K8）集群。该堆栈包括所有Milvus组件和相关依赖项，如etcd，Pulsar和MinIO。

## 2. 预备条件

- Create a K8s cluster：[Kubespray v2.25.0 Online Install Iubernetes v1.29.5](https://ghostwritten.blog.csdn.net/article/details/141222163)

- Install a StorageClass. You can check the installed StorageClass as follows：[Helm Install OpenEBS LocalPV v1.6](https://ghostwritten.blog.csdn.net/article/details/141226519)

```bash
$ kubectl get sc

NAME                  PROVISIONER                  RECLAIMPOLICY    VOLUMEBIINDINGMODE    ALLOWVOLUMEEXPANSION     AGE
standard (default)    k8s.io/minikube-hostpath     Delete           Immediate             false 

```

- [在安装前检查硬件和软件要求](https://milvus.io/docs/prerequisite-helm.md)。
- 在安装Milvus之前，建议使用[Milvus调整工具](https://milvus.io/tools/sizing)根据您的数据大小估计硬件要求。这有助于确保Milvus安装的最佳性能和资源分配。






##  3. Install cert-manager


Milvus Operator使用cert-manager为webhook服务器提供证书。

>如果您选择使用Helm部署Milvus Operator，则可以安全地跳过此步骤。
Milvus操作员需要证书管理器1.1.3或更高版本。


运行以下命令安装cert-manager。

```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.15.2/cert-manager.yaml
```

检查

```bash
$ kubectl get pod -A
NAMESPACE      NAME                                          READY   STATUS    RESTARTS        AGE
cert-manager   cert-manager-6fd987499c-xxqpw                 1/1     Running   0               99m
cert-manager   cert-manager-cainjector-5b94bd6f-6hhhp        1/1     Running   0               99m
cert-manager   cert-manager-webhook-575479ff47-9fh8b         1/1     Running   0               99m
```

## 4. Install Milvus Operator

Milvus Operator在Kubernetes自定义资源之上定义Milvus集群自定义资源。定义自定义资源后，您可以以声明方式使用K8s API并管理Milvus部署堆栈，以确保其可扩展性和高可用性。

您可以通过以下任一方式安装Milvus Operator：

- With Helm
- With kubectl

### 4.1 Helm 安装 Milvus Operator 
运行以下命令安装Milvus Operator with Helm。

```bash
helm install milvus-operator \
  -n milvus-operator --create-namespace \
  --wait --wait-for-jobs \
  https://github.com/zilliztech/milvus-operator/releases/download/v1.0.1/milvus-operator-1.0.1.tgz

```
安装过程结束后，您将看到类似于以下内容的输出。

```bash
NAME: milvus-operator
LAST DEPLOYED: Thu Jul  7 13:18:40 2022
NAMESPACE: milvus-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Milvus Operator Is Starting, use `kubectl get -n milvus-operator deploy/milvus-operator` to check if its successfully installed
If Operator not started successfully, check the checker's log with `kubectl -n milvus-operator logs job/milvus-operator-checker`
Full Installation doc can be found in https://github.com/zilliztech/milvus-operator/blob/main/docs/installation/installation.md
Quick start with `kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_minimum.yaml`
More samples can be found in https://github.com/zilliztech/milvus-operator/tree/main/config/samples
CRD Documentation can be found in https://github.com/zilliztech/milvus-operator/tree/main/docs/CRD

```

### 4.2  kubectl 安装 milvus-operator
运行以下命令，使用kubectl安装Milvus Operator。

```bash
kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/deploy/manifests/deployment.yaml
```
安装过程结束后，您将看到类似于以下内容的输出。

```bash
namespace/milvus-operator created
customresourcedefinition.apiextensions.k8s.io/milvusclusters.milvus.io created
serviceaccount/milvus-operator-controller-manager created
role.rbac.authorization.k8s.io/milvus-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/milvus-operator-manager-role created
clusterrole.rbac.authorization.k8s.io/milvus-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/milvus-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/milvus-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/milvus-operator-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/milvus-operator-proxy-rolebinding created
configmap/milvus-operator-manager-config created
service/milvus-operator-controller-manager-metrics-service created
service/milvus-operator-webhook-service created
deployment.apps/milvus-operator-controller-manager created
certificate.cert-manager.io/milvus-operator-serving-cert created
issuer.cert-manager.io/milvus-operator-selfsigned-issuer created
mutatingwebhookconfiguration.admissionregistration.k8s.io/milvus-operator-mutating-webhook-configuration created
validatingwebhookconfiguration.admissionregistration.k8s.io/milvus-operator-validating-webhook-configuration created
```
您可以检查Milvus Operator pod是否正在运行，如下所示：

```bash
$ 


NAME                               READY   STATUS    RESTARTS   AGE
milvus-operator-5fd77b87dc-msrk4   1/1     Running   0          46s

```

## 5. 安装 Milvus
### 5.1 Kubectl 安装 Milvus
一旦Milvus Operator pod运行，您就可以按如下方式部署Milvus集群。

```bash
kubectl apply -f https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_cluster_default.yaml

```

上面的命令使用默认配置将Milvus集群及其组件和依赖项部署在单独的pod中。要自定义这些设置，我们建议您使用[Milvus调整工具](https://milvus.io/tools/sizing)根据您的实际数据大小调整配置，然后下载相应的YAML文件。要了解有关配置参数的更多信息，请参阅[Milvus系统配置检查表](https://milvus.io/docs/system_configuration.md)。


>发布名称只能包含字母、数字和破折号。版本名称中不允许使用点。
您还可以在独立模式下部署Milvus实例，其中所有组件都包含在单个Pod中。为此，请将上述命令中的配置文件URL更改为https://raw.githubusercontent.com/zilliztech/milvus-operator/main/config/samples/milvus_default.yaml


### 5.2 Helm 安装 milvus
```bash
helm repo add zilliztech https://zilliztech.github.io/milvus-helm/
helm repo update
```

```bash
helm install my-release zilliztech/milvus \
  --set namespace=milvus \
  --create-namespace \
  --set cluster.enabled=false \
  --set etcd.replicaCount=1 \
  --set minio.mode=standalone \
  --set pulsar.enabled=false \
  --set etcd.persistence.size=10Gi \
  --set etcd.persistence.storageClass="openebs-lvmsc-hdd" \
  --set minio.persistence.size=50Gi \
  --set minio.persistence.storageClass="openebs-lvmsc-hdd" \
  --set milvus.service.type=NodePort \
  --set minio.service.type=NodePort \
  --set milvus.service.nodePort=30001 \
  --set minio.service.nodePort=30002 \
  --set milvus.persistence.persistentVolumeClaim.storageClass="openebs-lvmsc-hdd" \
  --set milvus.persistence.persistentVolumeClaim.size=50Gi
```


## 6. 检查Milvus群集状态
运行以下命令检查Milvus集群状态

```bash
kubectl get milvus my-release -o yaml
```
一旦你的Milvus集群准备好了，上面命令的输出应该类似于下面。如果status.status字段保持Unhealthy，则您的Milvus集群仍在创建中。

```bash
apiVersion: milvus.io/v1alpha1
kind: Milvus
metadata:
...
status:
  conditions:
  - lastTransitionTime: "2021-11-02T05:59:41Z"
    reason: StorageReady
    status: "True"
    type: StorageReady
  - lastTransitionTime: "2021-11-02T06:06:23Z"
    message: Pulsar is ready
    reason: PulsarReady
    status: "True"
    type: PulsarReady
  - lastTransitionTime: "2021-11-02T05:59:41Z"
    message: Etcd endpoints is healthy
    reason: EtcdReady
    status: "True"
    type: EtcdReady
  - lastTransitionTime: "2021-11-02T06:12:36Z"
    message: All Milvus components are healthy
    reason: MilvusClusterHealthy
    status: "True"
    type: MilvusReady
  endpoint: my-release-milvus.default:19530
  status: Healthy

```
Milvus Operator创建Milvus依赖项，如etcd、Pulsar和MinIO，然后创建Milvus组件，如代理、协调器和节点。
一旦您的Milvus集群准备就绪，Milvus集群中所有Pod的状态应类似于以下内容。

```bash
$ kubectl get pods

NAME                                            READY   STATUS      RESTARTS   AGE
my-release-etcd-0                               1/1     Running     0          14m
my-release-etcd-1                               1/1     Running     0          14m
my-release-etcd-2                               1/1     Running     0          14m
my-release-milvus-datanode-5c686bd65-wxtmf      1/1     Running     0          6m
my-release-milvus-indexnode-5b9787b54-xclbx     1/1     Running     0          6m
my-release-milvus-proxy-84f67cdb7f-pg6wf        1/1     Running     0          6m
my-release-milvus-querynode-5bcb59f6-nhqqw      1/1     Running     0          6m
my-release-milvus-mixcoord-fdcccfc84-9964g      1/1     Running     0          6m
my-release-minio-0                              1/1     Running     0          14m
my-release-minio-1                              1/1     Running     0          14m
my-release-minio-2                              1/1     Running     0          14m
my-release-minio-3                              1/1     Running     0          14m
my-release-pulsar-bookie-0                      1/1     Running     0          14m
my-release-pulsar-bookie-1                      1/1     Running     0          14m
my-release-pulsar-bookie-init-h6tfz             0/1     Completed   0          14m
my-release-pulsar-broker-0                      1/1     Running     0          14m
my-release-pulsar-broker-1                      1/1     Running     0          14m
my-release-pulsar-proxy-0                       1/1     Running     0          14m
my-release-pulsar-proxy-1                       1/1     Running     0          14m
my-release-pulsar-pulsar-init-d2t56             0/1     Completed   0          14m
my-release-pulsar-recovery-0                    1/1     Running     0          14m
my-release-pulsar-toolset-0                     1/1     Running     0          14m
my-release-pulsar-zookeeper-0                   1/1     Running     0          14m
my-release-pulsar-zookeeper-1                   1/1     Running     0          13m
my-release-pulsar-zookeeper-2                   1/1     Running     0          13m

```

## 7. 将本地端口转发到Milvus
运行以下命令获取Milvus集群服务的端口。

```bash
$ kubectl get pod my-release-milvus-proxy-84f67cdb7f-pg6wf --template
='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'
19530

```

输出显示Milvus实例服务于默认端口19530。

然后，运行以下命令，将本地端口转发到Milvus服务的端口。

```bash
$ kubectl port-forward service/my-release-milvus 27017:19530
Forwarding from 127.0.0.1:27017 -> 19530

```
可选地，您可以在上面的命令中使用：用途：19530而不是27017：19530来让kubectl为您分配本地端口，这样您就不必管理端口冲突。
默认情况下，kubectl的port-forwarding只监听localhost。如果您希望Milvus监听选定的或所有的IP地址，请使用地址标志。以下命令使port-forward侦听主机上的所有IP地址。

```bash
$ kubectl port-forward --address 0.0.0.0 service/my-release-milvus 27017:19530
Forwarding from 0.0.0.0:27017 -> 19530

```

## 8. 卸载 Milvus

执行以下命令卸载Milvus集群。

```bash
helm delete my-release
helm delete my-release -n milvus
for i in `kubectl get pvc |grep -v NAME  | awk '{print $1}'`;do kubectl delete pvc $i;done
for i in `kubectl get pvc -n milvus |grep -v NAME  | awk '{print $1}'`;do kubectl delete pvc $i -n milvus;done
```


使用默认配置删除Milvus集群时，etcd、Pulsar和MinIO等依赖项不会被删除。因此，下次安装相同的Milvus集群实例时，这些依赖项将再次使用。
要删除依赖项和私有虚拟云（PVC）沿着Milvus集群，请参阅配置文件。





