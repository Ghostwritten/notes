

# Operator Deployment 实验1
##  1. 介绍
在这个实验室中，你将`CoreOS etcd Operator`安装到你的Red Hat®OpenShift®容器平台集群中。然后使用Operator在项目中创建etcd集群，并验证etcd集群是否正常工作。

Goals

 - Install the CoreOS etcd Operator into your OpenShift Container Platform cluster
 - Use the Operator to deploy an etcd cluster into your OpenShift Container Platform cluster
 - Update the etcd cluster configuration
 - Verify that the etcd cluster is working properly
 - Delete the etcd cluster
 - Delete the etcd Operator

## 2. 环境准备
如果您之前为这个类设置了环境，您可以跳过这一节，直接进入安装CoreOS etcd操作符一节。

在本节中，您将提供实验室环境，以提供对执行实验室所需的所有组件的访问。实验室环境是一个基于云的环境，因此您可以从任何地方通过Internet访问它。但是，不要期望性能与专用环境相匹配。

 1. Go to the OPENTLC lab portal and use your OPENTLC credentials to log
    in.

>     如果您不记得自己的密码，请进入[OPENTLC帐户管理页面重置密码](https://account.opentlc.com/account/)。

 - Navigate to `Services` → `Catalogs` → `All Services` → `OPENTLC OpenShift 4 Labs`.
 - On the left, select `OpenShift 4 Operators Lab`.
 - On the right, click `Order`.
 - If you are not in `North America`, select your region from the Region list.
 - At the bottom right, click `Submit`.

> Do not select `App Control` → `Start` after ordering the lab.

几分钟后，你将收到三封电子邮件，其中包括如何连接到环境的说明:

 - 第一封电子邮件通知您配置已经开始
 - 当请求虚拟机成功时，触发第二封邮件
 - 最后一封电子邮件声明配置已经完成，并提供了启动和使用实验室环境的重要信息。
 - 等待直到您收到最后一封电子邮件，然后再尝试连接到您的环境。
 - 记下您的`guide`——**您环境中唯一的4个字符的标识符**。

##   3. Start Environment After Shutdown (Reference)
为了节省资源，实验室环境在8小时后自动关闭。本节提供在本课程的实验环境自动关闭后重新启动的说明。

 - Go to the [OPENTLC lab](https://labs.opentlc.com/) portal and use your OPENTLC credentials to log in.
 - Expect to see a list of your `Active Services` when you first log in—if not, select `Services` → `My Services`.
 - 在您的服务列表中，选择您的实验室环境。
 - 选择App Control→Start启动您的实验室环境。
 - 在“Are you sure?”提示。
 - At the bottom right, click Submit.

> 几分钟后，您将收到一封电子邮件，通知您实验室环境已经启动

##  4. Access Bastion Host
该堡垒管理主机作为进入环境的接入点，而不是Red Hat®OpenShift®容器平台环境的一部分。
使用邮件中提供的OPENTLC帐户名、主机名和密码连接到您的bastion主机:

```bash
ssh opentlc-username_@bastion.GUID.BASEDOMAIN
```
确认您当前的身份是`system:admin`:

```bash
oc whoami
```
Sample Output

```bash
system:admin
```
##  5. Install CoreOS etcd Operator
安装Operator的第一步是从 repository中检索Operator代码。在这种情况下，您使用`CoreOS GitHub`存储库。这个 repository通常包含设置`Operator`所需的所有YAML文件。
本实验室的重点是手动安装一个能在所有OpenShift容器平台版本中工作的Operator，从OpenShift容器平台3.9开始。如果一个运营商已经被释放到运营商中心，它可以很容易地从OpenShift容器平台web控制台安装代替。

1.Clone the `etcd-operator` repository and change to the repository directory:

```bash
cd $HOME
git clone https://github.com/coreos/etcd-operator.git
cd $HOME/etcd-operator
```
2.Create a new `etcd` project in which to deploy your Operator:

```bash
oc new-project etcd
```
Create the `ClusterRole` and `ClusterRoleBinding` to enable the Operator to work:

> 在本练习中，使用提供的shell脚本从`./example/rbac`目录中的模板文件创建`ClusterRole`和`ClusterRoleBinding`。这个脚本需要您部署的名称空间的名称来设置正确的绑定。在安装其他操作符时，您可能需要编辑提供的YAML文件，然后使用`oc create -f`来创建角色和角色绑定。

```bash
./example/rbac/create_role.sh --namespace=etcd
```
Sample Output

```bash
Creating role with ROLE_NAME=etcd-operator, NAMESPACE=etcd
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRole is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRole
clusterrole.rbac.authorization.k8s.io/etcd-operator created
Creating role binding with ROLE_NAME=etcd-operator, ROLE_BINDING_NAME=etcd-operator, NAMESPACE=etcd
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
clusterrolebinding.rbac.authorization.k8s.io/etcd-operator created
```
4.查看 the created ClusterRole:

```bash
oc describe clusterrole etcd-operator
```
Sample Output

```bash
Name:         etcd-operator
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources                                       Non-Resource URLs  Resource Names  Verbs
  ---------                                       -----------------  --------------  -----
  endpoints                                       []                 []              [*]
  events                                          []                 []              [*]
  persistentvolumeclaims                          []                 []              [*]
  pods                                            []                 []              [*]
  services                                        []                 []              [*]
  customresourcedefinitions.apiextensions.k8s.io  []                 []              [*]
  deployments.apps                                []                 []              [*]
  etcdbackups.etcd.database.coreos.com            []                 []              [*]
  etcdclusters.etcd.database.coreos.com           []                 []              [*]
  etcdrestores.etcd.database.coreos.com           []                 []              [*]
  secrets                                         []                 []              [get]
```
注意以下几点：

 - 资源列表包括一个新的API组，`etc .database.coreos.com`。这是操作员的自定义资源定义(CRD)的API组。
 - 这个API组中的资源是`etcdbackups`、`etcdclusters`和`etcddrestores`
 - 操作员还需要权限来操作`apiextensions.k8s`中的`customresourcedefinition`。在启动时创建CRD，以及它可能需要操作的各种其他资源的权限

5.查看创建的ClusterRoleBinding:

```bash
oc describe clusterrolebinding etcd-operator
```
Sample Output

```bash
Name:         etcd-operator
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  etcd-operator
Subjects:
  Kind            Name     Namespace
  ----            ----     ---------
  ServiceAccount  default  etcd
```
注意以下几点

 - `etcd -operator`集群角色绑定到`etcd`项目中的`default`服务帐户(查找ServiceAccount字段)。
 - 其他运营商设置自己的服务帐号，而不是使用`default`的service account.。

这个Operator是为OpenShift容器平台的早期版本编写的。因此，部署仍然使用旧版本的deployment定义。将该文件替换为以下更新版本:

```bash
cat << EOF >./example/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: etcd-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      name: etcd-operator
  template:
    metadata:
      labels:
        name: etcd-operator
    spec:
      containers:
      - name: etcd-operator
        image: quay.io/coreos/etcd-operator:v0.9.4
        command:
        - etcd-operator
        # Uncomment to act for resources in all namespaces. More information in doc/user/clusterwide.md
        #- -cluster-wide
        env:
        - name: MY_POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: MY_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
EOF
```
现在，所有的先决条件都已经就绪，创建etcd Operator

> You can examine the `./example/deployment`.yaml file to learn how the deployment is set up.:

```bash
oc create -f ./example/deployment.yaml
```
Wait until the Operator pod is running:

```bash
oc get pod
```
Sample Output

```bash
NAME                             READY   STATUS    RESTARTS   AGE
etcd-operator-55978c4587-dflrd   1/1     Running   0          15s
```
一旦Operator开始运行，检查操作员日志:

```bash
oc logs etcd-operator-55978c4587-dflrd
```
Sample Output

```bash
time="2020-05-13T17:39:28Z" level=info msg="etcd-operator Version: 0.9.4"
time="2020-05-13T17:39:28Z" level=info msg="Git SHA: c8a1c64"
time="2020-05-13T17:39:28Z" level=info msg="Go Version: go1.11.5"
time="2020-05-13T17:39:28Z" level=info msg="Go OS/Arch: linux/amd64"
time="2020-05-13T17:39:28Z" level=info msg="Event(v1.ObjectReference{Kind:\"Endpoints\", Namespace:\"etcd\", Name:\"etcd-operator\", UID:\"049748ad-e60a-4e16-9f38-4a5ab9cf45a3\", APIVersion:\"v1\", ResourceVersion:\"37261\", FieldPath:\"\"}): type: 'Normal' reason: 'LeaderElection' etcd-operator-55978c4587-dflrd became leader"
```
如果正确地设置了RBAC，预计不会看到任何错误。

##  6. Confirm CRD
当第一次启动时，etcd Operator自动创建一个CRD。
1.验证`etcdclusters.etd.database.coreos.com` CRD被创建:

```bash
oc get crds | grep etcd
```
Sample Output

```bash
etcdclusters.etcd.database.coreos.com                       2020-05-13T17:39:28Z
etcds.operator.openshift.io                                 2020-05-13T16:31:17Z
```
另外，还可以看到内置（built-in）的`etcd .operator.openshift.io` CRD。
2.检查CRD:

```bash
oc describe crd etcdclusters.etcd.database.coreos.com
```
Sample Output

```bash
Name:         etcdclusters.etcd.database.coreos.com
[...]
  Group:      etcd.database.coreos.com
  Names:
    Kind:       EtcdCluster
    List Kind:  EtcdClusterList
    Plural:     etcdclusters
    Short Names:
      etcd
    Singular:  etcdcluster
  Scope:       Namespaced
[...]
```
注意以下几点

 - 这个CRD服务于类对象:EtcdCluster, `EtcdCluster`的单数形式和EtcdCluster的复数形式。
 - 这个特定 的作用域（scope）是命名空间的，这意味着它只在运行该Operato的同一个命名空间中创建etcd集群。

3.执行oc命令查看etcd集群。

```bash
oc get etcdclusters
```

Sample Output

```bash
No resources found in etcd namespace.
```
该命令返回空，因为您还没有创建任何etcd集群。

##  7. Create etcd Clusters
在本节中，您将创建一个新的etcd集群

1.首先，查看使用CRD创建etcd集群的文件:

```bash
cat ./example/example-etcd-cluster.yaml
```
Sample Output

```bash
apiVersion: "etcd.database.coreos.com/v1beta2"
kind: "EtcdCluster"
metadata:
  name: "example-etcd-cluster"
  ## Adding this annotation make this cluster managed by clusterwide operators
  ## namespaced operators ignore it
  # annotations:
  #   etcd.database.coreos.com/scope: clusterwide
spec:
  size: 3
  version: "3.2.13"
```
注意以下几点:

 - 该文件使用`EtcdCluster`作为要创建的对象类型。
 - 集群大小为3份，etcd使用的版本为3.2.13。
 - 您部署的`etcd Operator` 使用的是`namespaced` 作用域而不是`clusterwide` 作用域，这意味着您的etcd集群需要使用注释中解释的相同定义。

2.创建新的etcd集群

```clike
oc create -f ./example/example-etcd-cluster.yaml
```
Sample Output

```clike
etcdcluster.etcd.database.coreos.com/example-etcd-cluster created
```
期待看到etcd pods一个接一个地启动。
3.Confirm that there are three pods running:

```clike
watch oc get pods
```
Sample Output

```clike
NAME                              READY   STATUS    RESTARTS   AGE
etcd-operator-55978c4587-dflrd    1/1     Running   0          3m19s
example-etcd-cluster-bqpblk2t44   1/1     Running   0          69s
example-etcd-cluster-tr2nq5hmv2   1/1     Running   0          52s
example-etcd-cluster-vjwfxhgdnx   1/1     Running   0          28s
```
Press `Ctrl+C` to exit the watch.
确认你的项目中有`EtcdCluster`:

```clike
oc get etcdclusters
```
Sample Output

```clike
NAME                   AGE
example-etcd-cluster   80s
```
## 8.  Update etcd Cluster
1.更新etcd集群文件以请求5个etcd副本 (edit the file and change size to 5):

> Etcd集群必须有奇数个成员，以便在各个集群参与者之间建立quorum。

```clike
vi ./example/example-etcd-cluster.yaml
```
Sample Output

```clike
apiVersion: "etcd.database.coreos.com/v1beta2"
kind: "EtcdCluster"
metadata:
  name: "example-etcd-cluster"
  ## Adding this annotation make this cluster managed by clusterwide operators
  ## namespaced operators ignore it
  # annotations:
  #   etcd.database.coreos.com/scope: clusterwide
spec:
  size: 5
  version: "3.2.13"
```
2.更新etcd集群，用更新后的文件替换配置:

```clike
oc apply -f ./example/example-etcd-cluster.yaml
```
Sample Output

```clike
Warning: oc apply should be used on resource created by either oc create --save-config or oc apply
etcdcluster.etcd.database.coreos.com/example-etcd-cl uster configured
```

> 注意，如果使用oc edit来编辑etcd集群，这将不起作用。

3.观察你的pod，看到两个额外的pod出现:

```go
watch oc get pods
```
Sample Output

```clike
NAME                              READY   STATUS    RESTARTS   AGE
etcd-operator-55978c4587-dflrd    1/1     Running   0          4m38s
example-etcd-cluster-bqpblk2t44   1/1     Running   0          2m28s
example-etcd-cluster-dbp4vbcd7b   1/1     Running   0          16s
example-etcd-cluster-dk7s9524mt   1/1     Running   0          32s
example-etcd-cluster-tr2nq5hmv2   1/1     Running   0          2m11s
example-etcd-cluster-vjwfxhgdnx   1/1     Running   0          107s
```
新的pod可能需要几秒钟的时间才能出现，在这之后，您可以期待看到一个有5个成员的etcd集群。

4.Press `Ctrl+C` to exit.

##  9. etcd Cluster
操作人员可以向OpenShift事件日志中event log。etcd operator在事件日志中使用`EtcdCluster`作为它所记录的事件类型。etcd集群还为客户端建立连接的服务。

1.检查 the etcd Operator events:

```clike
oc get events | grep etcdcluster
```
Sample Output

```css
29s         Normal    New Member Added         etcdcluster/example-etcd-cluster      New member example-etcd-cluster-dbp4vbcd7b added to cluster
2m24s       Normal    New Member Added         etcdcluster/example-etcd-cluster      New member example-etcd-cluster-tr2nq5hmv2 added to cluster
2m          Normal    New Member Added         etcdcluster/example-etcd-cluster      New member example-etcd-cluster-vjwfxhgdnx added to cluster
2m41s       Normal    New Member Added         etcdcluster/example-etcd-cluster      New member example-etcd-cluster-bqpblk2t44 added to cluster
45s         Normal    New Member Added         etcdcluster/example-etcd-cluster      New member example-etcd-cluster-dk7s9524mt added to cluster
```
检查项目中的服务:

```clike
oc get services
```
Sample Output

```clike
NAME                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
example-etcd-cluster          ClusterIP   None           <none>        2379/TCP,2380/TCP   2m55s
example-etcd-cluster-client   ClusterIP   172.30.76.12   <none>        2379/TCP            2m55s
```

> 请注意`example-etc -cluster-client`的IP地址。

使用`etcdctl`命令确认您的集群是健康的:
**因为这是一个服务，所以在集群之外不可用。这意味着您不能从bastion主机连接到etcd集群，因为bastion主机不是OpenShift网络的一部分。**


Find one of the etcd pods:

```css
oc get pod
```
Sample Output

```go
NAME                              READY   STATUS    RESTARTS   AGE
etcd-operator-55978c4587-dflrd    1/1     Running   0          5m20s
example-etcd-cluster-bqpblk2t44   1/1     Running   0          3m10s
example-etcd-cluster-dbp4vbcd7b   1/1     Running   0          58s
example-etcd-cluster-dk7s9524mt   1/1     Running   0          74s
example-etcd-cluster-tr2nq5hmv2   1/1     Running   0          2m53s
example-etcd-cluster-vjwfxhgdnx   1/1     Running   0          2m29s
```
使用列表中的最后一个pod (example-etc -cluster-vjwfxhgdnx)，在你的pod中打开一个shell:

```clike
oc rsh example-etcd-cluster-vjwfxhgdnx
```
使用`etcdctl cluster-health`命令来验证你的`etcd cluster`实际上是健康的:

> 注意，您可以使用服务名称来寻址etcd集群。

```go
etcdctl --endpoints http://example-etcd-cluster:2379 cluster-health
```
Sample Output

```go
member 1f248dae4d0254de is healthy: got healthy result from http://example-etcd-cluster-vjwfxhgdnx.example-etcd-cluster.etcd.svc:2379
member ba84d4a63ded1e07 is healthy: got healthy result from http://example-etcd-cluster-tr2nq5hmv2.example-etcd-cluster.etcd.svc:2379
member cba4428fb3572c06 is healthy: got healthy result from http://example-etcd-cluster-dk7s9524mt.example-etcd-cluster.etcd.svc:2379
member cdc0d0e10d8344d1 is healthy: got healthy result from http://example-etcd-cluster-bqpblk2t44.example-etcd-cluster.etcd.svc:2379
member dfc973fe7fc06992 is healthy: got healthy result from http://example-etcd-cluster-dbp4vbcd7b.example-etcd-cluster.etcd.svc:2379
cluster is healthy
```
4.尝试访问数据库并在其中存储一些东西:

```go
etcdctl --endpoints http://example-etcd-cluster:2379 set foo 'Hello world!'
# Output: Hello world!

etcdctl --endpoints http://example-etcd-cluster:2379 get foo
# Output: Hello world!
```
5.您可以看到etcd数据库正在工作。
Disconnect from the etcd pod:

```go
exit
```
通过删除项目中的`example-etcd-cluster`对象来删除整个etcd集群:

```go
oc delete etcdcluster example-etcd-cluster
```
操作员检测到并删除与该集群关联的pod。

Sample Output

```go
etcdcluster.etcd.database.coreos.com "example-etcd-cluster" deleted
```
或者，看着pod消失，然后按`Ctrl+C`退出:

```go
watch oc get pods
```
##  10. Clean Up Environment
要从你的集群中删除Operator，反转你用来设置Operator的命令:

```go
cd $HOME/etcd-operator
oc delete -f example/deployment.yaml
oc delete clusterrole etcd-operator
oc delete clusterrolebinding etcd-operator
oc delete crd etcdclusters.etcd.database.coreos.com
```
Delete your etcd project:

```go
oc delete project etcd
```

