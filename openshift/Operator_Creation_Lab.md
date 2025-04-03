


---


##  1. 简介
`Operator`提供了一种在`OpenShift`®和`Kubernetes`中创建自定义控制器的方法。他们使用自定义资源定义(crd)的概念来扩展`Kubernetes API`。Operator监视要创建的自定义资源对象——当它看到创建的自定义资源时，它会根据自定义资源对象中的参数创建应用程序。
操作符通常用Go编程语言编写，就像Kubernetes或OpenShift一样。这意味着Kubernetes API用于直接在集群中创建和监视对象。
因为这不是一个对开发人员非常友好的体验，所以Red Hat提供了一个软件开发工具包(SDK)，它大大简化了operator的创建。SDK处理所有的核心逻辑，只留下业务逻辑供开发人员填写。
Red Hat还在SDK中提供了另外两个功能，可以从`Ansible Roles`或`playbook`和`Helm Charts`创建操作员。
在本实验室中，您使用Ansible®Operator SDK来创建一个Operator，使用两个Ansible角色和一个playbook来设置`PostgreSQL`数据库和一个使用该数据库的Gitea服务器。不像在模板中，这个操作员可以完全配置Gitea服务器，因为它可以在添加到Gitea配置之前查询已经创建的路由。
有关更多信息，请[参阅Ansible操作符文档](https://github.com/operator-framework/operator-sdk/blob/v0.17.0/doc/ansible/user-guide.md)。
有关[Helm Charts](https://github.com/operator-framework/operator-sdk/blob/v0.17.0/doc/helm/user-guide.md)的更多信息，请参helm操作员工具包。请注意，Helm图表需要满足常规的OpenShift应用程序同样的要求——例如，pods不以root身份运行——才能在OpenShift上成功运行。

##  2. 目标

 - 使用Ansible Operator SDK从Ansible Roles创建一个Operator
 - Deploy the Operator to OpenShift
 - Create a Gitea server using a custom resource
 - Create 第二个 Gitea server
 - 检查创建的对象
 - Delete the Gitea servers
 - Delete the Operator

##  3. 环境准备

> If you previously set up an environment for this class, you can skip this section and go directly to the Install Operator SDK section.

在本节中，您将提供实验室环境，以提供对执行实验室所需的所有组件的访问。实验室环境是一个基于云的环境，因此您可以从任何地方通过Internet访问它。但是，不要期望性能与专用环境相匹配。

1. Go to the [OPENTLC lab portal](https://labs.opentlc.com/) and use your OPENTLC credentials to log in.

> 如果您不记得自己的密码，请进入[OPENTLC帐户管理页面重置密码](https://account.opentlc.com/account/)。

2. Navigate to `Services` → `Catalogs` → `All Services` → `OPENTLC OpenShift 4 Labs`.

3. On the left, select OpenShift 4 Operators Lab.

4. On the right, click Order.

5. If you are not in North America, select your region from the Region list.

6. At the bottom right, click Submit.

> Do not select `App Control` → `Start` after ordering the lab.


几分钟后，你将收到三封电子邮件，其中包括如何连接到环境的说明:

 - 第一封电子邮件通知您配置已经开始。
 - 当请求虚拟机成功时，触发第二封邮件。
 - 最后一封电子邮件声明配置已经完成，并提供了启动和使用实验室环境的重要信息。

等待直到您收到最后一封电子邮件，然后再尝试连接到您的环境。
记下您的GUID——您环境中唯一的4个字符的标识符。


###  3.1 关机后启动环境(Reference)
为了节省资源，实验室环境在8小时后自动关闭。本节提供在本课程的实验环境自动关闭后重新启动的说明。
Go to the OPENTLC lab portal and use your OPENTLC credentials to log in.
Expect to see a list of your Active Services when you first log in—if not, select Services → My Services.
在您的服务列表中，选择您的实验室环境。
选择App Control→Start启动您的实验室环境。
在“Are you sure?”提示。
At the bottom right, click Submit.
几分钟后，您将收到一封电子邮件，通知您实验室环境已经启动

### 3.2 Access Bastion Host
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

##   4. Install Operator SDK
构建Operator的第一步是将Operator SDK安装到您的客户端VM中
Download the `Operator SDK` binary into `/usr/local/bin`:
本实验室使用`v0.17.0`版本，因为Operator SDK更改非常频繁，您需要使用定义良好的存储库快照，以便实验室顺利运行。
有一些更新的版本，它们的工作方式略有不同。使用Operator SDK时，请使用与您的OpenShift / Kubernetes版本匹配的版本，并参考最新的文档。

```bash
cd $HOME

sudo wget https://github.com/operator-framework/operator-sdk/releases/download/v0.17.0/operator-sdk-v0.17.0-x86_64-linux-gnu -O /usr/local/bin/operator-sdk -O /usr/local/bin/operator-sdk

sudo chmod +x /usr/local/bin/operator-sdk
```
检查`operator-sdk`二进制文件是否正确安装:

```bash
operator-sdk version
```
Sample Output

```bash
operator-sdk version: "v0.17.0", commit: "2fd7019f856cdb6f6618e2c3c80d15c3c79d1b6c", kubernetes version: "unknown", go version: "go1.13.10 linux/amd64"
```
##  5. Create Operator from Ansible Roles Using Operator SDK
有一个第三方存储库，包括`Ansible Roles`来设置`PostgreSQL`数据库和一个基于`OpenShift`的`Gitea`服务器。存储库还包含一个`Ansible Playbook`，它调用这些角色来创建一个由OpenShift项目中的PostgreSQL数据库支持的Gitea服务器。
在本节中，您将使用`Ansible Operator`将这些role转换为一个Operator，该Operator可以通过简单地请求一个自定义Gitea资源来创建Gitea安装。

###   5.1 Clone Repository
1.克隆包含角色的存储库:

```bash
cd $HOME

git clone https://github.com/redhat-gpte-devopsautomation/ansible-operator-roles

cd ansible-operator-roles

git checkout v0.17.0

cd $HOME
```
Examine the playbook:

```bash
cat $HOME/ansible-operator-roles/playbooks/gitea.yaml
```
可以选择，检查这两个角色及其不同变量的默认值:

```bash
# postgresql-ocp Tasks File
cat $HOME/ansible-operator-roles/roles/postgresql-ocp/tasks/main.yml

# postgresql-ocp Variable defaults
cat $HOME/ansible-operator-roles/roles/postgresql-ocp/defaults/main.yml

# gitea-ocp Tasks File
cat $HOME/ansible-operator-roles/roles/gitea-ocp/tasks/main.yml

# gitea-ocp Variable defaults
cat $HOME/ansible-operator-roles/roles/gitea-ocp/defaults/main.yml
```
###  5.2 Create Ansible Operator 骨架
[参考Ansible操作用户指南](https://github.com/operator-framework/operator-sdk/blob/v0.17.0/doc/ansible/user-guide.md)。

在你的主目录中，使用`operator-sdk`二进制文件来创建一个新的`Ansible Operator`框架:
使用以下值来确保你的操作符在共享集群中工作:

 - `gitea-operator` as the Operator name
 - `gpte.opentlc.com/v1alpha1` as the API version
 - `Gitea` as the kind
将`ansible`设置为`Operator`类型，并包含使用剧本而不仅仅是角色的选项。

```bash
cd $HOME

operator-sdk new gitea-operator --api-version=gpte.opentlc.com/v1alpha1 --kind=Gitea --type=ansible --generate-playbook
```
Sample Output

```bash
INFO[0000] Creating new Ansible operator 'gitea-operator'.
INFO[0000] Created deploy/service_account.yaml
INFO[0000] Created deploy/role.yaml
INFO[0000] Created deploy/role_binding.yaml
INFO[0000] Created deploy/crds/gpte.opentlc.com_v1alpha1_gitea_cr.yaml
INFO[0000] Created build/Dockerfile
INFO[0000] Created roles/gitea/README.md
INFO[0000] Created roles/gitea/meta/main.yml
INFO[0000] Created roles/gitea/files/.placeholder
INFO[0000] Created roles/gitea/templates/.placeholder
INFO[0000] Created roles/gitea/vars/main.yml
INFO[0000] Created molecule/test-local/converge.yml
INFO[0000] Created roles/gitea/defaults/main.yml
INFO[0000] Created roles/gitea/tasks/main.yml
INFO[0000] Created molecule/default/molecule.yml
INFO[0000] Created molecule/default/prepare.yml
INFO[0000] Created molecule/default/converge.yml
INFO[0000] Created molecule/default/verify.yml
INFO[0000] Created roles/gitea/handlers/main.yml
INFO[0000] Created watches.yaml
INFO[0000] Created deploy/operator.yaml
INFO[0000] Created .travis.yml
INFO[0000] Created requirements.yml
INFO[0000] Created molecule/test-local/molecule.yml
INFO[0000] Created molecule/test-local/prepare.yml
INFO[0000] Created molecule/test-local/verify.yml
INFO[0000] Created molecule/cluster/molecule.yml
INFO[0000] Created molecule/cluster/create.yml
INFO[0000] Created molecule/cluster/prepare.yml
INFO[0000] Created molecule/cluster/converge.yml
INFO[0000] Created molecule/cluster/verify.yml
INFO[0000] Created molecule/cluster/destroy.yml
INFO[0000] Created molecule/templates/operator.yaml.j2
INFO[0000] Generated CustomResourceDefinition manifests.
INFO[0000] Generating Ansible playbook.
INFO[0000] Created playbook.yml
INFO[0000] Project creation complete.
```

> 如果在创建过程中遇到问题，可以删除`$HOME/gitea-operator`目录，然后重试。

###  5.3 更新框架
`operator-sdk`创建了一个框架playbook和一个框架roles目录。但是您已经从前面克隆的存储库中获得了剧本和角色。在本节中，您将更新骨架。

1.切换到`gitea-operator`目录，并将playbook和roles目录替换为您下载的目录

```bash
cd $HOME/gitea-operator

rm -rf roles playbook.yml

mkdir roles

cp -R $HOME/ansible-operator-roles/roles/postgresql-ocp ./roles

cp -R $HOME/ansible-operator-roles/roles/gitea-ocp ./roles

cp $HOME/ansible-operator-roles/playbooks/gitea.yaml ./playbook.yml
```
2.检查watches.yaml file:

```bash
cat ./watches.yaml
```
Sample Output
```bash
---
- version: v1alpha1
  group: gpte.opentlc.com
  kind: Gitea
  playbook: playbook.yml
```
这是`Ansible  Operator`.的主要配置。group, version, and kind 来自用于创建框架的命令。`playbook`就是`playbook.yml`文件名。在下一步中，您将看到文件如何在/opt/ansible位置结束。

3.在构建目录中检查`Dockerfile`:

```bash
cat build/Dockerfile
```
Sample Output

```bash
FROM quay.io/operator-framework/ansible-operator:v0.17.0

COPY requirements.yml ${HOME}/requirements.yml
RUN ansible-galaxy collection install -r ${HOME}/requirements.yml \
 && chmod -R ug+rwx ${HOME}/.ansible

COPY watches.yaml ${HOME}/watches.yaml

COPY roles/ ${HOME}/roles/
COPY playbook.yml ${HOME}/playbook.yml
```
请注意正在构建的容器映像的源映像，并观察Dockerfile所做的一切就是将roles, playbook, and watches复制到容器映像中.

`ansible-operator`容器映像已经包含了Operator需要的逻辑，并且只使用剧本来设置应用程序。

 playbook需要是幂等的(意味着它在第二次运行时一定不会失败)，因为Operator会定期运行 playbook，以确认所有必要的对象都就位了。
虽然本例中的角色支持删除应用程序(通过设置`state=absent`)，但这对于`Ansible Operators`中的roles来说是不必要的。当`Ansible operator`创建Kubernetes对象时，它会向所有创建的对象注入`ownerReference`字段。因此，删除CR(本例中为Gitea)将删除所有关联的对象。

###  5.4 Build Operator
构建您的Operator容器映像:
为了构建容器映像并将其推送到注册中心，您需要注册中心的用户ID——可以是Docker Hub或Quay。

> 出于本实验的目的，如果您还没有用户ID，建议您在[https://quay.io](https://quay.io)注册一个用户ID。

**创建帐户**链接有点隐藏。单击右上角的**Sign In**查看登录屏幕，该屏幕也有**Create** 

注册用户ID后，在Quay中创建一个名为`gitea-operator`的公共存储库。

> 确保选择`Create New Repository`页面上的`Public`单选按钮

这是您推送容器图像的地方。

Log in to Quay:

```bash
export QUAY_ID=<your quay id>
podman login -u ${QUAY_ID} quay.io
```
Sample Output

```bash
Password:
Login Succeeded!
```
使用`Operator SDK`构建container image 并将其标记为tag。`quay.io/<your quay id>/gitea-operator:v0.0.1`，确保替换`<your quay id>`与你的码头。io用户ID:

> 您可以使用`podman`来构建这个`operator container image`。因为这个环境正在运行Red Hat®Enterprise
> Linux®8，并且您正在使用`podman`，所以没有必要将映像构建为根目录。

```bash
cd $HOME/gitea-operator
operator-sdk build quay.io/${QUAY_ID}/gitea-operator:v0.0.1 --image-builder podman
```
Sample Output

```bash
INFO[0000] Building OCI image quay.io/wkulhanek/gitea-operator:v0.0.1
STEP 1: FROM quay.io/operator-framework/ansible-operator:v0.17.0
Getting image source signatures
Copying blob 51a84b8e6e04 done
Copying blob a7518fc4b4b8 done
Copying blob 9d05b535e611 done
Copying blob 1730fea3a412 done
Copying blob ee2244abc66f done
Copying blob befb03b11956 done
Copying blob 96917d26f5e1 done
Copying blob a5236a9afd95 done
Copying blob 45b98e5f36b9 done
Copying config 5d34c0760f done
Writing manifest to image destination
Storing signatures
STEP 2: COPY requirements.yml ${HOME}/requirements.yml
8dba84ada95d0261faae69a19097e25ff0861e0735095f99875b4feef1889966
STEP 3: RUN ansible-galaxy collection install -r ${HOME}/requirements.yml  && chmod -R ug+rwx ${HOME}/.ansible
Process install dependency map
Starting collection install process
Installing 'community.kubernetes:0.11.0' to '/opt/ansible/.ansible/collections/ansible_collections/community/kubernetes'
Installing 'operator_sdk.util:0.0.0' to '/opt/ansible/.ansible/collections/ansible_collections/operator_sdk/util'
eb1530aeaee13eb0fab5859aae71f3b83cbeb9b8ebd77fd5bc87060acd52f993
STEP 4: COPY watches.yaml ${HOME}/watches.yaml
33796acc6461231ea0f1dab95f99f39e50c7e8cf3b4caf2e49fcafbac3934928
STEP 5: COPY roles/ ${HOME}/roles/
62176192051771f71883cae3d19ee22dcc2f21a390351a46c3d0d56eb835b0fb
STEP 6: COPY playbook.yml ${HOME}/playbook.yml
STEP 7: COMMIT quay.io/wkulhanek/gitea-operator:v0.0.1
4bf51414d98651a92c0f73897055f2cd4cb8ffcf3a1d45aa97a0c03a3a022bda
INFO[0066] Operator build complete.
```
Verify the built container:

```bash
podman images
```
Sample Output

```bash
REPOSITORY                                    TAG       IMAGE ID       CREATED          SIZE
quay.io/wkulhanek/gitea-operator              v0.0.1    4bf51414d986   28 seconds ago   500 MB
quay.io/operator-framework/ansible-operator   v0.17.0   5d34c0760fd5   3 weeks ago      500 MB
```
将容器图像推送到Quay:

```bash
podman push quay.io/${QUAY_ID}/gitea-operator:v0.0.1
```

> 如果看到错误消息，请确保使用`podman`登录`podman login -u ${QUAY_ID} quay.io`再次登录到Quay。

在文本编辑器中打开Operator部署定义:

```bash
vim ./deploy/operator.yaml
```
修改如下:

 - 要使用您刚刚推送的 image，请将占位符`{{REPLACE_IMAGE}}`替换为实际的图像定义，确保将`<your quay id>`替换为您的`quay.io`用户ID。
 - ：确保`imagePullPolicy`被设置为`Always`


确认最后的文件如下所示:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitea-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      name: gitea-operator
  template:
    metadata:
      labels:
        name: gitea-operator
    spec:
      serviceAccountName: gitea-operator
      containers:
        - name: gitea-operator
          # Replace this with the built image name
          image: quay.io/<your quay id>/gitea-operator:v0.0.1
          imagePullPolicy: "Always"
          volumeMounts:
          - mountPath: /tmp/ansible-operator/runner
            name: runner
          env:
            - name: WATCH_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: OPERATOR_NAME
              value: "gitea-operator"
            - name: ANSIBLE_GATHERING
              value: explicit
      volumes:
        - name: runner
          emptyDir: {}
```
##  6. Deploy Gitea Operator
现在一切都已经构建好了，您可以部署Gitea Operator了
###  6.1 Create Custom Resource Definition
1.在您的bastion主机上，确保您以root身份登录，以`system:admin`身份对OpenShift集群进行身份验证
检查由 Operator 	SDK创建的CRD:

```bash
cat ./deploy/crds/gpte.opentlc.com_giteas_crd.yaml
```
Sample Output

```bash
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: giteas.gpte.opentlc.com
spec:
  group: gpte.opentlc.com
  names:
    kind: Gitea
    listKind: GiteaList
    plural: giteas
    singular: gitea
  scope: Namespaced
  subresources:
    status: {}
  validation:
    openAPIV3Schema:
      type: object
      x-kubernetes-preserve-unknown-fields: true
  versions:
  - name: v1alpha1
    served: true
    storage: true
```
Create the CRD for your Operator:

```bash
oc create -f ./deploy/crds/gpte.opentlc.com_giteas_crd.yaml
```
Sample Output

```bash
W0319 19:46:47.952583   38757 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/giteas.gpte.opentlc.com created
```
Create the `gitea-admin-rules` ClusterRole:

```bash
echo '---
apiVersion: authorization.openshift.io/v1
kind: ClusterRole
metadata:
  labels:
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
  name: gitea-admin-rules
rules:
- apiGroups:
   - apps
  resources:
  - deployments/finalizers
  verbs:
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - ""
  resources:
  - events
  - services/finalizers
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - monitoring.coreos.com
  resources:
  - servicemonitors
  verbs:
  - get
  - create
- apiGroups:
  - gpte.opentlc.com
  resources:
  - giteas
  - giteas/status
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch' | oc create -f -
```
标签`rbac.authorization.k8s.io/aggregate-to-admin="true"`修改默认的项目`admin`角色，自动将这个集群角色授予任何项目所有者，然后谁自动获得项目的管理权限.

确认CRD已部署到集群:

```bash
oc get crd | grep gitea
```
Sample Output

```bash
giteas.gpte.opentlc.com                                     2021-03-19T19:46:47Z
```
###  6.2 Deploy Operator
Operators可以部署在集群范围的基础上，也可以部署在`namespace/project` 的基础上。在这个实验室中，您可以在您自己的项目中部署您的Operators。

1.以普通用户身份登录集群，使用`andrew`作为用户ID和`r3dh4t1!`的密码:

```bash
oc login -u andrew -p r3dh4t1! $(oc whoami --show-server)
```
2.创建Operator项目:

```bash
oc new-project gitea-operator --display-name="Gitea"
```
3.在deploy子目录中依次部署这些构件，从你的Operator 将要使用的service_account开始:

```bash
oc create -f ./deploy/service_account.yaml
```
4.使用以下提示创建向Operator授予正确权限的`role`和`RoleBinding`

 - Operator SDK为您创建一个框架角色文件。但是，根据Operator需要创建或操作的内容，您可能需要自定义角色
 - Gitea Operator部署了一些特定于openshift的对象，比如`Route`。路由是在`route.openshift.io`API组中定义的，因此需要将路由添加到Operator需要权限的资源列表中。
 - Gitea Operator还需要创建和操作`serviceaccounts`对象。
 - 使用`vim  ./deploy/role.yaml`编辑role定义。命令，将`route`作为一个对象包含在Operator需要访问的对象中。
 - 为`route.openshift.io`添加另一个`apiGroup`对象。
 - 在""(空)API组下添加一个`serviceaccounts`资源。

5.确认最后的文件如下所示:

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: gitea-operator
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - services/finalizers
  - endpoints
  - persistentvolumeclaims
  - events
  - configmaps
  - secrets
  - serviceaccounts
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - apps
  resources:
  - deployments
  - daemonsets
  - replicasets
  - statefulsets
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - monitoring.coreos.com
  resources:
  - servicemonitors
  verbs:
  - get
  - create
- apiGroups:
  - apps
  resourceNames:
  - gitea-operator
  resources:
  - deployments/finalizers
  verbs:
  - update
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - apps
  resources:
  - replicasets
  - deployments
  verbs:
  - get
- apiGroups:
  - "route.openshift.io"
  resources:
  - routes
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
- apiGroups:
  - gpte.opentlc.com
  resources:
  - 'giteas'
  - 'giteas/status'
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
```
6.创建`role`和`RoleBinding`(从而将角色中的权限授予Operator使用的服务帐户):

```bash
oc create -f ./deploy/role.yaml

oc create -f ./deploy/role_binding.yaml
```

> 现在，启动`Gitea Operator`的所有先决条件都已经就绪。

7.创建指向Operator映像的部署

```bash
oc create -f ./deploy/operator.yaml
```
8.等待Operator运行，第一次可能会花费更长的时间，因为容器映像需要从Quay中提取，而且它很大。
9.观察pod，然后按`Ctrl+C`退出一旦操作员豆荚运行

```c
watch oc get pod
```
Sample Output	

```bash
Every 2.0s: oc get pod

NAME                             READY   STATUS    RESTARTS   AGE
gitea-operator-7796fc5fb-cjl8v   1/1     Running   0          45s
```
一旦`Operator pod`开始运行，检查日志，确保将Operator pod 的名称替换为您特定的Operator pod 的名称:

```bash
oc logs gitea-operator-7796fc5fb-cjl8v
```
Sample Output

```bash
{"level":"info","ts":1589396455.1062052,"logger":"cmd","msg":"Go Version: go1.13.10"}
{"level":"info","ts":1589396455.1062434,"logger":"cmd","msg":"Go OS/Arch: linux/amd64"}
{"level":"info","ts":1589396455.1062486,"logger":"cmd","msg":"Version of operator-sdk: v0.17.0"}
{"level":"info","ts":1589396455.1067598,"logger":"cmd","msg":"Watching single namespace.","Namespace":"gitea-operator"}
{"level":"info","ts":1589396457.3138108,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":"0.0.0.0:8383"}
{"level":"info","ts":1589396457.3144138,"logger":"watches","msg":"Environment variable not set; using default value","envVar":"WORKER_GITEA_GPTE_OPENTLC_COM","default":1}
{"level":"info","ts":1589396457.3144422,"logger":"watches","msg":"Environment variable not set; using default value","envVar":"ANSIBLE_VERBOSITY_GITEA_GPTE_OPENTLC_COM","default":2}
{"level":"info","ts":1589396457.314486,"logger":"cmd","msg":"Environment variable not set; using default value","Namespace":"gitea-operator","envVar":"ANSIBLE_DEBUG_LOGS","ANSIBLE_DEBUG_LOGS":false}
{"level":"info","ts":1589396457.3145196,"logger":"ansible-controller","msg":"Watching resource","Options.Group":"gpte.opentlc.com","Options.Version":"v1alpha1","Options.Kind":"Gitea"}
{"level":"info","ts":1589396457.31465,"logger":"leader","msg":"Trying to become the leader."}
{"level":"info","ts":1589396459.5430717,"logger":"leader","msg":"No pre-existing lock was found."}
{"level":"info","ts":1589396459.551477,"logger":"leader","msg":"Became the leader."}
{"level":"info","ts":1589396464.020476,"logger":"metrics","msg":"Metrics Service object updated","Service.Name":"gitea-operator-metrics","Service.Namespace":"gitea-operator"}
{"level":"info","ts":1589396466.3479352,"logger":"cmd","msg":"Could not create ServiceMonitor object","Namespace":"gitea-operator","error":"servicemonitors.monitoring.coreos.com \"gitea-operator-metrics\" already exists"}
{"level":"info","ts":1589396466.3496406,"logger":"proxy","msg":"Starting to serve","Address":"127.0.0.1:8888"}
{"level":"info","ts":1589396466.34987,"logger":"controller-runtime.manager","msg":"starting metrics server","path":"/metrics"}
{"level":"info","ts":1589396466.3500016,"logger":"controller-runtime.controller","msg":"Starting EventSource","controller":"gitea-controller","source":"kind source: gpte.opentlc.com/v1alpha1, Kind=Gitea"}
{"level":"info","ts":1589396466.450473,"logger":"controller-runtime.controller","msg":"Starting Controller","controller":"gitea-controller"}
{"level":"info","ts":1589396466.450538,"logger":"controller-runtime.controller","msg":"Starting workers","controller":"gitea-controller","worker count":1}
```

> 希望在日志中看不到任何错误，并且在启动完成后，希望日志输出类似于此。

您的Operator已经准备好部署Gitea服务器。


##  7. Use Operator to Create Gitea Server
要使用Operator部署应用程序，您需要创建一个自定义资源，它是一个Kubernetes对象。当在Operator监视的项目中创建自定义资源时，Operator根据自定义资源中的配置选项创建组成该应用程序的对象。

Gitea操作员理解以下配置选项(及其他):
|Name	|Default	|Required|
|--|--|--|--|
|postgresqlVolumeSize|1Gi|No|
|giteaVolumeSize|1Gi|No|
|giteaSsl|False|No|


在Kubernetes中，变量名的约定是使用`CamelCase`，而在Ansible中是使用`snake_case`。因此，自定义资源需要变量名在CamelCase中，Ansible Playbooks和角色希望它们在snake_case中。Ansible操作符基本图像自动处理此转换。

`deploy/crds/gpte.opentlc.com_giteas_crd.yaml`文件包含一个自定义资源框架，但是Operator SDK生成器不能知道Ansible Operator 中的剧本理解哪些变量。因此，需要对自定义资源进行定制。

在编写Operator时，需要考虑到在一个项目中可能有多个由Operator创建的对象实例。在Gitea的例子中，这意味着可能有一个以上的PostgreSQL数据库和一个以上的Gitea服务器。

所有创建的对象都需要具有唯一的名称。这通常是通过使用自定义资源的名称作为创建对象的前缀来实现的。您正在使用的角色是为满足这些需求而构建的。

1.创建一个自定义资源来部署Gitea服务器在你的项目:

```bash
echo "apiVersion: gpte.opentlc.com/v1alpha1
kind: Gitea
metadata:
  name: repository
spec:
  postgresqlVolumeSize: 4Gi
  giteaVolumeSize: 4Gi
  giteaSsl: True" > $HOME/gitea-operator/gitea-server.yaml
```
2.Create the Gitea server:

```bash
oc create -f $HOME/gitea-operator/gitea-server.yaml
```
3.在终端窗口中，观看pod出现:

```bash
watch oc get pod
```
4.在另一个终端窗口中，当Ansible Playbook执行时，跟踪`Gitea Operator pod`的日志

```bash
oc logs -f gitea-operator-7796fc5fb-cjl8v
```

> 请注意，Gitea operator 一遍又一遍地执行Ansible
> Playbook，以确保所有对象实际上仍然存在。您可以通过更改话务员的协调时间来配置运行频率。
> 
> 注意，Ansible operator将所有者引用添加到所有创建的对象。您不使用操作符删除已部署的应用程序，而是删除自定义资源


5.在你的项目中检索一个Gitea对象列表:

```bash
oc get giteas
```
Sample Output

```bash
NAME         AGE
repository   4m35s
```
6.检查自定义资源:

```bash
oc describe gitea repository
```
Sample Output

```bash
Name:         repository
Namespace:    gitea-operator
Labels:       <none>
Annotations:  <none>
API Version:  gpte.opentlc.com/v1alpha1
Kind:         Gitea
Metadata:
  Creation Timestamp:  2020-05-13T19:02:26Z
  Generation:          1
  Resource Version:    63883
  Self Link:           /apis/gpte.opentlc.com/v1alpha1/namespaces/gitea-operator/giteas/repository
  UID:                 9d490633-f50f-442f-a318-4b0126d50f24
Spec:
  Gitea Ssl:               true
  Gitea Volume Size:       4Gi
  Postgresql Volume Size:  4Gi
Status:
  Conditions:
    Ansible Result:
      Changed:             0
      Completion:          2020-05-13T19:04:33.786151
      Failures:            0
      Ok:                  7
      Skipped:             0
    Last Transition Time:  2020-05-13T19:02:26Z
    Message:               Awaiting next reconciliation
    Reason:                Successful
    Status:                True
    Type:                  Running
Events:                    <none>
```
在Status部分中，operator保存各种状态消息和状态状态。

7.获取你的Gitea服务器的路由:

```bash
oc get route
```
Sample Output

```bash
NAME         HOST/PORT                                                   PATH   SERVICES     PORT    TERMINATION     WILDCARD
repository   repository-gitea-operator.apps.cluster-$GUID.$BASE_DOMAIN          repository   <all>   edge/Redirect   None
```
回想一下，自定义资源中的`giteaSsl`参数被设置为`true`。这意味着操作员为**Gitea存储库**设置了一条安全路由。

8.使用web浏览器中的路由访问您的Gitea存储库的主页，它已经完全配置好了。

> 请注意，如果没有使用`https://`，作为路由的前缀，该路由将被配置为自动将不安全的请求重定向到安全端点。

9.如果出现提示，如果您的集群使用自签名证书，请接受不安全的证书。
10.通过创建另一个名为`another- Gitea`的自定义资源来部署Gitea服务器的第二个副本，这一次，将`giteaSsl`设置为`False`:

```bash
echo "apiVersion: gpte.opentlc.com/v1alpha1
kind: Gitea
metadata:
  name: another-gitea
spec:
  postgresqlVolumeSize: 4Gi
  giteaVolumeSize: 4Gi
  giteaSsl: False" > $HOME/gitea-operator/gitea-server-2.yaml

oc create -f $HOME/gitea-operator/gitea-server-2.yaml
```
11.观察pod上来并确认你有两个Gitea服务器，每个都有自己的PostgreSQL数据库运行在你的项目中:

```bash
watch oc get pod
```
Sample Output

```bash
Every 2.0s: oc get pod

NAME                                        READY   STATUS    RESTARTS   AGE
another-gitea-6d5ff9b95f-8h4r8              1/1     Running   0          2m6s
gitea-operator-7796fc5fb-cjl8v              1/1     Running   0          9m13s
postgresql-another-gitea-54cc66c49d-fr4d2   1/1     Running   0          2m36s
postgresql-repository-6789b9dfb4-q9fv8      1/1     Running   0          7m27s
repository-74d8468fd7-qhv7h                 1/1     Running   0          6m45s
```
12.按“`Ctrl+C`”退出watch命令。
13.获取，检索路由，并连接到第二个Gitea服务器在您的web浏览器。

> 注意，这个路由是不安全的，但是两个Gitea实例都是完全配置的。

##  8. Delete Gitea Servers
删除由Operator 创建的对象与删除自定义资源一样简单。**遵循最佳实践的Operator在所有创建的对象中设置一个`ownerReference`字段，以支持级联删除**。`Ansible Operator`监视`Kubernetes API`调用来创建资源，并在成功创建资源后向每个资源添加`ownerReference`字段。在本节中，您将检查所创建的pod的所有者引用。
1.首先，检索部署列表:

```bash
oc get deployments
```
Sample Output

```bash
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
another-gitea              1/1     1            1           2m28s
gitea-operator             1/1     1            1           14m
postgresql-another-gitea   1/1     1            1           2m58s
postgresql-repository      1/1     1            1           7m49s
repository                 1/1     1            1           7m7s
```
2.获取第一个`PostgreSQL`部署的YAML表示(在上面的例子中是`PostgreSQL-repository`):

```bash
oc get deployment postgresql-repository -o yaml
```
Sample Output

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2020-05-13T19:02:37Z"
  generation: 1
  name: postgresql-repository
  namespace: gitea-operator
  ownerReferences:
  - apiVersion: gpte.opentlc.com/v1alpha1
    kind: Gitea
    name: repository
    uid: 9d490633-f50f-442f-a318-4b0126d50f24
  resourceVersion: "63395"
  selfLink: /apis/apps/v1/namespaces/gitea-operator/deployments/postgresql-repository
  uid: 6016c079-ee99-44ff-8abf-dcf0130032b0
spec:
  progressDeadlineSeconds: 600

[...]
```

> 注意，`ownerReference`指向一个具有名称`repository`的Gitea类对象。此`repository`对象是您创建的自定义资源。

3.检查这个对象，看看那里是否有另一个`ownerReference`:

```bash
oc get gitea repository -o yaml
```
Sample Output

```bash
apiVersion: gpte.opentlc.com/v1alpha1
kind: Gitea
metadata:
  creationTimestamp: "2020-05-13T19:02:26Z"
  generation: 1
  name: repository
  namespace: gitea-operator
  resourceVersion: "63883"
  selfLink: /apis/gpte.opentlc.com/v1alpha1/namespaces/gitea-operator/giteas/repository
  uid: 9d490633-f50f-442f-a318-4b0126d50f24
spec:
  giteaSsl: true
[...]
```

 - `Gitea`对象是顶级对象，因此它不包含另一个`ownerReference`。
 - Secrets, PVCs, ConfigMaps, services, and routes也有相同的`ownerReference`集。

4.通过删除自定义资源来删除第一个Gitea服务器

```bash
oc delete gitea repository
```
5.使用watch oc get pod查看属于第一个Gitea服务器的pod被删除。
6.删除另一个giea服务器:

```bash
oc delete gitea another-gitea
```
##  9. Delete Gitea Operator
1.通过删除deployment 、role、RoleBinding和 service account的部署来删除Gitea Operator

```bash
oc delete deployment gitea-operator
oc delete rolebinding gitea-operator
oc delete role gitea-operator
oc delete sa gitea-operator
```
2.Delete the project:

```bash
oc delete project gitea-operator
```

> 注意，这个命令还删除了在第1步中删除的所有项。

3.以system身份登录集群:admin:

```bash
oc login -u system:admin
```
4.Delete the cluster role and CRD:

```bash
oc delete clusterrole gitea-admin-rules
oc delete crd giteas.gpte.opentlc.com
```
这就结束了operator creation的实验。
