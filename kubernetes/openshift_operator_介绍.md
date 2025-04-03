

## 1. 介绍

 - Operator概述
 - Operator部署
 - Operator创建

假设和前提条件
你想:

 - 了解操作人员如何在Red Hat®OpenShift®容器平台上工作
 - 了解如何部署Operator
 - 学习如何创建Operator

##  2. Operator概述
OpenShift API对象由资源和控制器组成

 - 资源:Pod、ConfigMap、secret、service、route、PersistentVolumeClaim等。
 - 控制器:Deployment, ReplicaSet, StatefulSet, DaemonSet, et等等

Control loop: Observe → Analyze → Act
Operator:管理资源的控制器类型

 - 在软件中代表人类的操作知识来管理应用程序
 - 是否可以使用stock controllers或管理资源本身


Domain-/Application-Specific特定的知识

 - Install and uninstall
 - Upgrade
 - Scale properly
 - Back up
 - Restore
 - Self-heal

数据库常用维护任务
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18375afc83f9e73e79d650a24a2d1299.png)
与OpenShift互动
Operators 利用自定义资源定义(crd)

 - crds允许扩展Kubernetes/OpenShift API,然后API知道新的资源
 - crd允许创建定制资源(CRs)
 - Operators  watches CR的创建，通过创建应用程序进行反应
 - CRs的管理方式与普通的OpenShift对象相同:创建、获取、描述、删除等。

例子:

```bash
oc get tomcats
```


```bash
CRD
 +
Custom controller
 +
Domain-specific knowledge
 =
Operator
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/28f245c4bc419cab05d12dc897adcf92.png)
[https://github.com/operator-framework](https://github.com/operator-framework)

##  3. Operator Components
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e3b17453ba7edac926c3e5dd07c3d255.png)

 - Operator Deletion
 - Custom Resource Creation Using an Operator


##   4. Operator Installation
可以为以下任何一种安装Operators :

 - 整个集群
 - 单项目

需要不同的RBAC对象
Operator监视一个项目或整个集群，以进行CR创建事件
是否可以自动向项目管理员授予发放CRs的权限
某些对象需要集群管理员权限（cluster-admin）

Cluster-Wide Operator 的安装步骤
所有步骤都需要以集群管理员权限执行

 - 创建 CRD
 - 为 Operator `创建项目`以在其中运行
 - 为 Operator pod 创建服务帐户（`service account`）以运行
 - 创建 RBAC `ClusterRole`
 - 为运行 Operator 的服务帐户创建 RBAC `ClusterRoleBinding`
 - 创建指向包含 Operator 代码的容器映像的 Operator 部署
 - 将 `WATCH_NAMESPACE` 环境变量设置为“”


命名空间运算符的安装步骤
除 CRD 之外的所有步骤都可以在项目管理员权限下执行

 - 创建 `CRD`
 - - 需要集群管理员（cluster-admin）权限
 - 为 Operator 创建项目以在其中运行
 - 为 Operator pod 创建`service account`以运行
 - 创建 RBAC `Role`
 - 为运行 Operator 的服务帐户创建 `RBAC RoleBinding`
 - 创建指向包含 Operator 代码的容器映像的 Operator 部署
 - 将 `WATCH_NAMESPACE` 环境变量设置为当前项目名称
 - 要自动设置，请使用 downwardAPI `metadata.namespace` 字段


##  5. Sample Operator Components
### 5.1 Custom Resource Definition

```bash
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: tomcats.apache.org
spec:
  group: apache.org
  names:
    kind: Tomcat
    listKind: TomcatList
    plural: tomcats
    singular: tomcat
    shortNames:
    - tc
  scope: Namespaced
  version: v1alpha1
```
### 5.2 RBAC Role

```bash
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tomcat-operator
rules:
- apiGroups:
  - tomcats.apache.org
  resources:
  - "*"
  verbs:
  - "*"
- apiGroups:
  - ""
  resources:
  - "*"
  verbs:
  - "*"
```
如果不使用集群管理权限，OpenShift需要明确的`resources` 和 `verbs`列表

### 5.3 RBAC RoleBinding

```bash
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: default-account-tomcat-operator
subjects:
- kind: ServiceAccount
  name: tomcat-operator
roleRef:
  kind: Role
  name: tomcat-operator
  apiGroup: rbac.authorization.k8s.io
```
### 5.4 添加权限项目管理员（admin）
为每个项目管理员添加权限，以创建从项目访问CRD的角色:

```bash
apiVersion: authorization.openshift.io/v1
kind: ClusterRole
metadata:
  labels:
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
  name: gogs-admin-rules
rules:
- apiGroups:
  - gpte.opentlc.com
  resources:
  - gogs
  verbs:
  - create
  - update
  - delete
  - get
  - list
  - watch
  - patch
```
###  5.5 Operator Deployment

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      name: tomcat-operator
  template:
    metadata:
      labels:
        name: tomcat-operator
    spec:
      serviceAccountName: tomcat-operator
      containers:
        - name: tomcat-operator
          image: quay.io/wkulhanek/tomcat-operator:v0.1.0
          imagePullPolicy: Always
          env:
            - name: WATCH_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
```
To deploy a cluster-wide Operator, replace valueFrom: with value: "".



## 6 Custom Resource 管理
运行操作员将在以下两种方式中创建CR:

 - 整个OpenShift集群
 - 运行者正在运行的项目

当创建CR时，Operator接收事件
然后Operator创建所有OpenShift资源组成的应用程序

Example: Custom Resource Creation
to create CR instance:
Create YAML file with definition of CR:

```bash
apiVersion: apache.org/v1alpha1
kind: Tomcat
metadata:
  name: mytomcat
spec:
  replicaCount: 2
```
Create resource in OpenShift:

```bash
oc create -f mytomcat.yaml
```
Custom Resource Management
要操作和检查CR，请使用oc命令:

```bash
oc get tomcats
oc describe tomcat mytomcat
oc scale tomcats --replicas=2 # only if Operator supports scaling
```
不要直接扩展ReplicaSets, deployment, StatefulSets, etc

 - Use Operator to scale
 - Operator继续监视已创建的资源，并将其设置回初始状态

删除所有创建的OpenShift API对象，删除CR:

```bash
oc delete tomcat mytomcat
```
Summary

 - Operator Components
 - Operator Installation
 - Sample Operator Components
 - Operator Deletion
 - Custom Resource Creation Using an Operator

