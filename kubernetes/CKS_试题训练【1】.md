

-----
考试准备
22道题、120分钟

```bash
alias k=kubectl
​
export do="-o yaml --dry-run=client"
```

## 1. 考点：Use Role Based Access Controls to minimize exposure
### 试题 1：RBAC授权问题
Use context: `kubectl config use-context infra-prod`

You have `admin` access to `cluster2`. There is also context `gianna@infra-prod` which authenticates as user `gianna` with the same cluster.

There are existing cluster-level `RBAC` resources in place to, among other things, ensure that user `gianna` can never read **Secret contents cluster-wide**. Confirm this is correct or restrict the existing RBAC resources to ensure this.

I addition, create more RBAC resources to allow user gianna to create `Pods` and `Deployments` in Namespaces `security`, `restricted` and `internal`. It's likely the user will receive these exact permissions as well for other Namespaces in the future.

中文：
 您有“admin”权限访问“`cluster2`”。还有上下文‘`gianna@infra-prod`’，它以相同集群的用户‘`gianna`’身份验证。
有现有的集群级别的“RBAC”资源，以确保用户“`gianna`”永远不能读取集群范围内的`Secret`内容。确认这是正确的，或者限制现有的RBAC资源来确保这一点。
此外，创建更多的RBAC资源，允许用户gianna在名称空间`security`, `restricted` and `internal`中创建`Pods` and `Deployments`。在将来，用户很可能也会获得其他名称空间的这些完全相同的权限。

Answer:
#### 第1步：检查当前 RBAC rules
我们可能应该首先看看用户的现有RBAC资源`gianna`。我们不知道资源名称，但是我们知道它们是集群级别的，因此我们可以搜索ClusterRoleBinding：:

```bash
k get clusterrolebinding -oyaml | grep gianna -A10 -B20
k edit clusterrolebinding gianna
```
查看名字为`gianna`的ClusterRoleBinding

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: "2020-09-26T13:57:58Z"
  name: gianna
  resourceVersion: "3049"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/gianna
  uid: 72b64a3b-5958-4cf8-8078-e5be2c55b25d
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: gianna
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: gianna
```

查看名字为`gianna`的ClusterRole:

```bash
$ k edit clusterrole gianna
```

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: "2020-09-26T13:57:55Z"
  name: gianna
  resourceVersion: "3038"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/gianna
  uid: b713c1cf-87e5-4313-808e-1a51f392adc0
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  - configmaps
  - pods
  - namespaces
  verbs:
  - list
```

根据任务，用户永远都不能读取Secrets内容。仅有list权限可能表明这是正确的,但是我们需要登录确认:

```bash
➜ k auth can-i list secrets --as gianna
yes
​
➜ k auth can-i get secrets --as gianna
no
```

我们以gianna登录切换查看验证它的权限:

```bash
➜ k config use-context gianna@infra-prod
Switched to context "gianna@infra-prod".
​
➜ k -n security get secrets
NAME                  TYPE                                  DATA   AGE
default-token-gn455   kubernetes.io/service-account-token   3      20m
kubeadmin-token       Opaque                                1      20m
mysql-admin           Opaque                                1      20m
postgres001           Opaque                                1      20m
postgres002           Opaque                                1      20m
vault-token           Opaque                                1      20m
​
➜ k -n security get secret kubeadmin-token
Error from server (Forbidden): secrets "kubeadmin-token" is forbidden: User "gianna" cannot get resource "secrets" in API group "" in the namespace "security"
Still all expected, but being able to list resources also allows to specify the format:

➜ k -n security get secrets -oyaml | grep password
    password: ekhHYW5lQUVTaVVxCg==
        {"apiVersion":"v1","data":{"password":"ekhHYW5lQUVTaVVxCg=="},"kind":"Secret","metadata":{"annotations":{},"name":"kubeadmin-token","namespace":"security"},"type":"Opaque"}
          f:password: {}
    password: bWdFVlBSdEpEWHBFCg==
...
```

用户`gianna`实际上能够阅读`secrets`内容。为了防止这种情况发生，我们应该删除列出它们的功能:

```bash
$ k config use-context infra-prod # back to admin context
​
$ k edit clusterrole gianna
# kubectl edit clusterrole gianna
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: "2020-09-26T13:57:55Z"
  name: gianna
  resourceVersion: "4496"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/gianna
  uid: b713c1cf-87e5-4313-808e-1a51f392adc0
rules:
- apiGroups:
  - ""
  resources:
#  - secrets       # remove
  - configmaps
  - pods
  - namespaces
  verbs:
  - list
 
```
#### 第2步：创建其他RBAC rules
让我们谈谈RBAC资源：

一个`ClusterRole` | 角色定义了一组权限及其在整个群集中或仅在单个命名空间中可用的位置。

一个`ClusterRoleBinding` | RoleBinding将一组权限与一个帐户相关联，并定义在整个群集中还是仅在单个命名空间中应用该权限的位置。

因此，有4种不同的RBAC组合和3种有效的：

 - Role+ RoleBinding（在单一可用的命名空间，在单施加命名空间）
 - ClusterRole + ClusterRoleBinding（在群集范围内可用，在群集范围内应用）
 - ClusterRole + RoleBinding （在整个群集中可用，在单个Namespace中应用）
 - Role+ ClusterRoleBinding（不可能：仅在单个命名空间中可用，在整个群集范围内应用）

 

用户gianna应该能够在三个命名空间中创建Pod和Deployments。我们可以使用上面列表中的数字1或3。但是因为该任务说：“用户将来也可能会获得其他命名空间的确切权限”，所以我们选择数字3，因为它仅需要创建一个ClusterRole而不是三个Roles。

这将创建一个`ClusterRole`，如下所示：

```bash
$ kubectl create clusterrole gianna-additional --verb=create --resource=pods --resource=deployments

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: gianna-additional
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - create
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - create
```
接下来是三个绑定:

```bash
k -n security create rolebinding gianna-additional \
--clusterrole=gianna-additional --user=gianna
​
k -n restricted create rolebinding gianna-additional \
--clusterrole=gianna-additional --user=gianna
​
k -n internal create rolebinding gianna-additional \
--clusterrole=gianna-additional --user=gianna
```
这将创建RoleBindings，例如：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: gianna-additional
  namespace: security
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: gianna-additional
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: gianna
```
测试验证

```bash
➜ k -n default auth can-i create pods --as gianna
no
​
➜ k -n security auth can-i create pods --as gianna
yes
​
➜ k -n restricted auth can-i create pods --as gianna
yes
​
➜ k -n internal auth can-i create pods --as gianna
yes
```
您也可以通过上下文`gianna@infra-prod`以用户`gianna`的身份实际创建Pods和部署来验证这一点。


##  2. 考点：Setup appropriate OS level security domains e.g. using PSP, OPA, security contexts
### 试题1：使用OPA策略提高安全性
Use context: `kubectl config use-context infra-prod`

There is an existing `Open Policy Agent` + `Gatekeeper` policy to enforce that all Namespaces need to have label security-level set. Extend the policy constraint and template so that all Namespaces also need to set label management-team. Any new Namespace creation without these two labels should be prevented.

Write the names of all existing Namespaces which violate the updated policy into /opt/course/p2/fix-namespaces.
中文：
现有的`Open Policy Agent` + `Gatekeeper`策略可以强制所有命名空间都需要`security-level`设置标签。扩展策略约束和模板等所有命名空间也需要设置label `management-team`。**没有这两个标签的任何新的命名空间创建都应被阻止。**

将违反更新策略的所有现有命名空间的名称写入`/opt/course/p2/fix-namespaces`。

 

回答：
我们看一下现有的`OPA`约束，这些约束是`Gatekeeper`使用`CRD`实现的：

```bash
➜ k get crd
NAME                                                 CREATED AT
blacklistimages.constraints.gatekeeper.sh            2020-09-14T19:29:31Z
configs.config.gatekeeper.sh                         2020-09-14T19:29:04Z
constraintpodstatuses.status.gatekeeper.sh           2020-09-14T19:29:05Z
constrainttemplatepodstatuses.status.gatekeeper.sh   2020-09-14T19:29:05Z
constrainttemplates.templates.gatekeeper.sh          2020-09-14T19:29:05Z
requiredlabels.constraints.gatekeeper.sh             2020-09-14T19:29:31Z
```
所以我们可以这样做:

```bash
➜ k get constraint
NAME                                                           AGE
blacklistimages.constraints.gatekeeper.sh/pod-trusted-images   10m
​
NAME                                                                  AGE
requiredlabels.constraints.gatekeeper.sh/namespace-mandatory-labels   10m
```
并检查`namespace-mandatory-label`的违规情况，

```bash
➜ k describe requiredlabels namespace-mandatory-labels
Name:         namespace-mandatory-labels
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  constraints.gatekeeper.sh/v1beta1
Kind:         RequiredLabels
...
Status:
...
  Total Violations:  1
  Violations:
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"security-level"}
    Name:                sidecar-injector
Events:                  
```
我们看到了名称空间“`sidecar-injector`”的一个冲突。让我们来了解一下所有的名称空间:

```bash
➜ k get ns --show-labels
NAME                STATUS   AGE   LABELS
default             Active   21m   management-team=green,security-level=high
gatekeeper-system   Active   14m   admission.gatekeeper.sh/ignore=no-self-managing,control-plane=controller-manager,gatekeeper.sh/system=yes,management-team=green,security-level=high
jeffs-playground    Active   14m   security-level=high
kube-node-lease     Active   21m   management-team=green,security-level=high
kube-public         Active   21m   management-team=red,security-level=low
kube-system         Active   21m   management-team=green,security-level=high
restricted          Active   14m   management-team=blue,security-level=medium
security            Active   14m   management-team=blue,security-level=medium
sidecar-injector    Active   14m   <none>
```
当我们尝试创建一个没有必要标签的命名空间时，我们会得到一个OPA错误:

```bash
➜ k create ns test
Error from server ([denied by namespace-mandatory-labels] you must provide labels: {"security-level"}): error when creating "ns.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [denied by namespace-mandatory-labels] you must provide labels: {"security-level"}
```
接下来我们编辑约束以添加另一个必需的标签:

```bash
$ k edit requiredlabels namespace-mandatory-labels

apiVersion: constraints.gatekeeper.sh/v1beta1
kind: RequiredLabels
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"constraints.gatekeeper.sh/v1beta1","kind":"RequiredLabels","metadata":{"annotations":{},"name":"namespace-mandatory-labels"},"spec":{"match":{"kinds":[{"apiGroups":[""],"kinds":["Namespace"]}]},"parameters":{"labels":["security-level"]}}}
  creationTimestamp: "2020-09-14T19:29:53Z"
  generation: 1
  name: namespace-mandatory-labels
  resourceVersion: "3081"
  selfLink: /apis/constraints.gatekeeper.sh/v1beta1/requiredlabels/namespace-mandatory-labels
  uid: 2a51a291-e07f-4bab-b33c-9b8c90e5125b
spec:
  match:
    kinds:
    - apiGroups:
      - ""
      kinds:
      - Namespace
  parameters:
    labels:
    - security-level
    - management-team # add
```
正如我们所看到的约束是使用`kind: RequiredLabels`作为模板，它是`Gatekeeper`创建的`CRD`。让我们应用更改，看看会发生什么(给OPA一分钟时间在内部应用更改):

```bash
➜ k describe requiredlabels namespace-mandatory-labels
...
  Violations:
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"management-team"}
    Name:                jeffs-playground
```
更改之后，我们可以看到另一个名称空间`jeffs-playground`遇到了麻烦。因为它只指定了一个必需的标签。但是，早先违反名称空间`sidecar-injector`的情况如何呢?

```bash
➜ k get ns --show-labels
NAME                STATUS   AGE   LABELS
default             Active   21m   management-team=green,security-level=high
gatekeeper-system   Active   17m   admission.gatekeeper.sh/ignore=no-self-managing,control-plane=controller-manager,gatekeeper.sh/system=yes,management-team=green,security-level=high
jeffs-playground    Active   17m   security-level=high
kube-node-lease     Active   21m   management-team=green,security-level=high
kube-public         Active   21m   management-team=red,security-level=low
kube-system         Active   21m   management-team=green,security-level=high
restricted          Active   17m   management-team=blue,security-level=medium
security            Active   17m   management-team=blue,security-level=medium
sidecar-injector    Active   17m   <none>
```
命名空间`sidecar-injector`也应该有麻烦，但现在不会了。这似乎是不对的，这意味着我们仍然可以创建没有任何标签的名称空间，就像使用 `k create ns test`.

所以我们检查template:

```bash
➜ k get constrainttemplates
NAME              AGE
blacklistimages   20m
requiredlabels    20m
​
➜ k edit constrainttemplates requiredlabels

apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
...
spec:
  crd:
    spec:
      names:
        kind: RequiredLabels
      validation:
        openAPIV3Schema:
          properties:
            labels:
              items: string
              type: array
  targets:
  - rego: |
      package k8srequiredlabels
​
      violation[{"msg": msg, "details": {"missing_labels": missing}}] {
        provided := {label | input.review.object.metadata.labels[label]}
        required := {label | label := input.parameters.labels[_]}
        missing := required - provided
        # count(missing) == 1 # WRONG
        count(missing) > 0
        msg := sprintf("you must provide labels: %v", [missing])
      }
    target: admission.k8s.gatekeeper.sh
```
在rego脚本中，我们需要将`count(missing) == 1`更改为`count(missing) > 0`。如果我们不这样做，那么策略只会在缺少一个标签时报错，但是可能会有多个标签缺失。
等了一会儿之后，我们再次检查约束:

```bash
➜ k describe requiredlabels namespace-mandatory-labels
...
  Total Violations:  2
  Violations:
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"management-team"}
    Name:                jeffs-playground
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"security-level", "management-team"}
    Name:                sidecar-injector
Events:                  <none>
```
这看起来更好。最后，我们将有冲突的命名空间名称写入所需的位置

```bash
$  vim /opt/course/p2/fix-namespaces
sidecar-injector
jeffs-playground
```
----

## 3. 考点：Detect threats within physical infrastructure, apps, networks, data, users and workloads
### 试题1：杀死并清理威胁进程
Use context: `kubectl config use-context workload-stage`
A security scan result shows that there is an unknown `miner` process running on one of the `Nodes` in `cluster3`. The report states that the process is listening on port `6666`. Kill the process and delete the binary.

中文：
安全扫描结果显示，在cluster3中的一个节点上运行着一个未知的miner进程。报告指出进程正在监听端口6666。终止进程并删除二进制文件。

回答
Answer:
We have a look at existing Nodes:

```bash
➜ k get node
NAME               STATUS   ROLES    AGE   VERSION
cluster3-master1   Ready    master   24m   v1.19.6
cluster3-worker1   Ready    <none>   22m   v1.19.6
```

First we check the master:

```bash
➜ ssh cluster3-master1
​
➜ root@cluster3-master1:~# netstat -plnt | grep 6666
​
➜ root@cluster3-master1:~# 
```

Doesn't look like any process listening on this port. So we check the worker:

```bash
➜ ssh cluster3-worker1
​
➜ root@cluster3-worker1:~# netstat -plnt | grep 6666
tcp6       0      0 :::6666                 :::*        LISTEN      9591/system-atm     
```

There we go! We could also use lsof:

```bash
➜ root@cluster3-worker1:~# lsof -i :6666
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
system-at 9591 root    3u  IPv6  47760      0t0  TCP *:6666 (LISTEN)
```

Before we kill the process we can check the magic /proc directory for the full process path:

```bash
➜ root@cluster3-worker1:~# ls -lh /proc/9591/exe
lrwxrwxrwx 1 root root 0 Sep 26 16:10 /proc/9591/exe -> /bin/system-atm
```

So we finish it:

```bash
➜ root@cluster3-worker1:~# kill -9 9591
​
➜ root@cluster3-worker1:~# rm /bin/system-atm
```

