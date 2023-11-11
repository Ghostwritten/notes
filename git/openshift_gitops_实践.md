


---

 1. [Red Hat OpenShift 4.8 环境集群搭建](https://ghostwritten.blog.csdn.net/article/details/123207497)
 2. [openshift 如何输出json日志](https://ghostwritten.blog.csdn.net/article/details/123335781)
 3. [openshfit Vertical Pod Autoscaler 实践](https://ghostwritten.blog.csdn.net/article/details/123335420)
 4. [openshift Certified Helm Charts 实践](https://ghostwritten.blog.csdn.net/article/details/123335635)
 5. [openshift 创建一个Serverless应用程序](https://ghostwritten.blog.csdn.net/article/details/123335299)
 6. [openshift gitops 实践](https://ghostwritten.blog.csdn.net/article/details/123336100)
 7. [openshift Tekton pipeline 实践](https://ghostwritten.blog.csdn.net/article/details/123375339)

---

##  1. 介绍
[GitOps](https://www.gitops.tech/)是一种为云本地应用程序实现持续部署的声明性方法。您可以使用GitOps创建可重复的过程，管理Red Hat®OpenShift®容器平台集群和跨多集群Kubernetes环境的应用程序。GitOps可以快速地处理和自动化复杂的部署，从而在部署和发布周期中节省时间。


在本实验室中，您将部署 GitOps operator并使用它将 `Coffee Shop` 应用程序部署到 `prod-coffeeshop` 命名空间中。 Coffee Shop 应用程序作为 YAML 清单和 Kustomize 文件存储在 Git 存储库中。您可以检查该 Git 存储库以了解有关 Kustomize 工作原理的详细信息。

首先，部署 [Argo CD](https://argo-cd.readthedocs.io/en/stable/) 并访问 Web 界面。然后，您将 Coffee Shop 应用程序前端的 Argo CD 应用程序定义复制并粘贴到界面中并观察其部署。您在部署该应用程序后将其删除。

然后，您将探索能够将复杂的应用程序集部署到多个集群的应用程序集，但您将只部署到本地集群。您使用应用程序集不仅可以部署咖啡店前端组件，还可以部署咖啡师 Knative 应用程序，所有这些都在一个应用程序对象中。

最后，您对集群上的应用程序进行一些手动更改，并观察 Argo CD 如何将应用程序返回到 GitOps 存储库中描述的所需状态。

目标：

 1. 使用 Argo CD 部署单个应用程序
 2. 使用 Argo CD 应用程序集来部署 Kubernetes 部署和 Knative 服务
 3. 体验 Argo CD 修复


##  2. 安装和验证GitOps operator

 1. 浏览到您的 OpenShift Container Platform Web 控制台，并以`admin`身份登录。
 2. 这方面的说明和凭据在您收到的配置电子邮件中。
 3. 在 Administrator 透视图中，选择 `Operators → Operator Hub`。
 4. 要轻松找到安装程序卡，请在 `Filter by keyword...` 字段中键入 `openshift gitops` 并单击该卡。
 5. 单击安装，然后再次安装，保留“更新通道：稳定”和“更新批准：自动”的所有默认值。


##  3. 通过Web UI访问Argo CD

> Argo CD还没有自动与OpenShift容器平台认证系统集成。

 1. 获取 Argo CD 管理员密码：
a. 从项目下拉列表中选择项目：`openshift-gitops`。
b. 从导航菜单中，选择`Workloads → Secrets`。
c. 使用 Search by name.. 并输入 `openshift-gitops-cluster`，然后单击 `openshift-gitops-cluster` 链接。
d.导航到 `Secrets → Secret details` 并向下滚动到 Data 部分。
e. 找到密码字段并单击⎘（复制）将密码值复制到剪贴板。
 2. 现在找到 Argo CD Web UI 的链接：
a. 在导航菜单上，选择`Networking → Routes`.
b. 单击第三列中的 `openshift-gitops-server Location` 链接。
 3. 使用用户名 admin 和剪贴板中的密码登录 Argo CD Web UI

##  4. 检查 Argo CD Projects, Applications and Application Sets

Argo CD 项目不同于 OpenShift 项目。

Argo CD 将项目定义为

 - 来源：应用程序配置 YAML 清单存储库，例如 Git 或 Helm 存储库。
 - 目标：部署源的 OpenShift 服务器、命名空间及其用户。
 - SyncPolicy：定义 Argo CD 如何保持源和目标同步
 - 工具：Kustomize、脚本等项目。将应用程序配置设置（例如 YAML 或
   Helm）转换为特定目标详细信息的工具。示例包括：主机名、端口、机密和容器映像注册表。

Argo CD 定义了一个应用程序以包括以下内容

 - 命名空间
 - 部署
 - 服务
 - 路线

Argo CD 应用程序集添加应用程序自动化

 - Argo CD 中的多集群支持和集群多租户支持。
 - 应用程序可以从多个来源进行模板化，包括 Git 或 Argo CD 自己定义的集群列表。


##  5. 准备咖啡店项目
在本练习中，您将为您的Coffee Shop应用程序创建一个新的专用项目。Argo CD的用户界面可能有点挑剔，所以请仔细遵循说明。如果您错过了某个步骤，一些看似合理的默认项实际上可能并不适用。这是一个安全特性——确保您意识到您正在“允许所有”。

1.从Argo CD web控制台，点击齿轮图标进入项目管理界面:
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc0eca4d8c23408eae09bb0e8c1db6ef.png)
2. Select PROJECTS → NEW PROJECT.

3. Enter `coffee-shop` as the new project name and click `CREATE`.

4. Scroll down to `SOURCE REPOSITORIES` and click `EDIT`.

5. Make sure there is an asterisk * in the field and click SAVE.
星号允许使用此项目的应用程序使用任何存储库。您必须这样做才能知道该项目中的应用程序现在可以从任何地方的任何存储库中提取配置。

6. 向下滚动到目的地并点击编辑。

7. Click `ADD DESTINATION` and under `Namespace` replace the asterisk `*` with `prod-coffeeshop.`

> 您不需要输入服务器名称，因为您使用的服务器是`OpenShift GitOps operator`的本地服务器。您特别选择了`prod-coffeeshop`名称空间，因为您不希望Argo CD管理OpenShift容器平台环境—只是`prod-coffeeshop`上的所有应用程序。

8. Click SAVE.

9. 点击叠纸图标进入应用管理界面:
![在这里插入图片描述](https://img-blog.csdnimg.cn/e81ad07cd34541e7856764c492b55b49.png)
 现在你有一个项目，你可以与新的Argo CD应用程序相关联，以指导OpenShift容器平台上的应用程序管理。

## 6. 部署 Coffee Shop 应用程序
在本练习中，您将把Coffee Shop应用程序部署到生产名称空间中。Coffee Shop应用程序有三个组件:数据库、Coffee Shop前端和订单管理系统，以及管理每个订单从准备（`PREPARING` ）到收集状态（`COLLECTED` ）的Barista服务。

已经为您部署了数据库。
首先只部署Coffee Shop应用程序前端组件，这样就可以了解什么是`Argo CD`“应用程序”。
您的Argo CD应用程序管理界面应该显示“没有应用程序”。Argo CD应用程序YAML内容提供给您复制和粘贴到Argo CD接口。

 - Click `CREATE APPLICATION` and then click `EDIT` AS YAML.
 - Copy and paste the following manifest:

```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: coffee-shop
  namespace: openshift-gitops
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: prod-coffeeshop
  project: coffee-shop
  source:
    path: ./coffee-shop-kustomize/coffee-shop/overlays/production
    repoURL: http://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps.git
    targetRevision: HEAD
```

> 如果这个应用程序定义有任何问题，你可以在这里找到一个工作副本:[https://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps/blob/main/coffee-shop-argocd/coffee-shop.yaml.](https://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps/blob/main/coffee-shop-argocd/coffee-shop.yaml)

 1. Click `SAVE`.
观察定义应用程序的字段现在已被填充。
 2. Click `CREATE`.
 3. In the pop-up window that appears, click `SYNC` → `SYNCHRONIZE`.
 4. 单击应用程序名称可以查看应用程序的滚出，并查看所有应用程序部分的表示  
![在这里插入图片描述](https://img-blog.csdnimg.cn/e8f7cc940d144cb6900791cf4833f116.png)
 5. 成功部署应用程序组件后，单击咖啡店应用程序上的`DELETE`。 您可以删除它，以便可以使用一个对象(Application集)部署两个组件

## 7. 部署 Barista Component as Knative Service with Argo CD Application Set

在这个练习中，您将使用`OpenShift GitOps Argo CD`的一个新特性:`Application Sets`。
应用程序集使得跨多个集群部署多个应用程序变得很容易，这些集群拥有不同的所有者。
在本例中，您对现有的Coffee Shop应用程序组件(Coffee - Shop和barista)进行了简单的部署，它们都在同一台服务器上和同一名称空间中。

1.返回OpenShift容器平台web控制台。
应用程序集还没有用户界面，所以你需要OpenShift容器平台web控制台来完成这项工作。（不要关闭你的Argo CD网络控制台。）
2.从您的web控制台顶部的工具栏，单击ocp_web_console_add_icon(添加)导入您的应用程序的Argo CD应用程序集的YAML清单如下:

```bash
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: coffee-shop-set
  namespace: openshift-gitops
spec:
  generators:
  - git:
      repoURL: https://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps.git
      revision: HEAD
      directories:
      - path: coffee-shop-kustomize/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      destination:
        namespace: prod-coffeeshop
        server: 'https://kubernetes.default.svc'
      project: coffee-shop
      source:
        path: '{{path}}/overlays/production/'
        repoURL: >-
          https://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps.git
        targetRevision: HEAD
      syncPolicy:
        automated:
          allowEmpty: false
          prune: true
          selfHeal: true
        retry:
          backoff:
            duration: 5s
            factor: 2
            maxDuration: 3m
          limit: 5
        syncOptions:
          - Validate=false
          - CreateNamespace=true
          - PrunePropagationPolicy=foreground
          - PruneLast=true
```

> 如果这个应用程序定义有任何问题，你可以在这里找到一个工作副本:[https://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps/blob/main/coffee-shop-argocd/coffee-shop-set.yaml](https://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps/blob/main/coffee-shop-argocd/coffee-shop-set.yaml).

3.Click `Create` to create your application set.

###  7.1 Sync Applications

 1. 更改您的Argo CD web控制台，并注意，`barista`应用程序已添加。
 2. On the barista application, click `SYNC`.
 3. If a pop-up menu appears, click `SYNCHRONIZE`.
 4. Repeat the `SYNC → SYNCHRONIZE` steps on the coffee-shop
    application.
Expect the `coffee-shop` application to sync.


##  8. 更改应用程序设置和观察Argo CD Argo CD 修复
Argo CD监视您的源和目的地的变化。当同步时，Argo CD将目的地返回到源文件中定义的状态，根据您的Argo CD应用程序配置，可以是自动的，也可以是批准的。Argo CD默认是保守的，所以它的同步自动化策略没有打开。但在这个实验室环境中，政策被开启了。
对于本练习，您将手动扩大生产咖啡店应用程序，并观察Argo CD将其缩小到一个副本。

 1. Go back to the OpenShift Container Platform web console.
 2. From the `Administrator` perspective, select `Project: prod-coffeeshop`.
 3. 从导航菜单, select `Workloads → Deployments`.
 4. Locate the `coffee-shop` deployment, click options_menu_icon(Options), and select `Edit` Pod count.
 5. Click ⊕ (Plus) four times to scale up the deployment to five pods.
 6. Return to the Argo CD web console.
 7. From the application management interface, click the `coffee-shop`  application card.

> You can see the single replica configuration here:
> [https://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps/blob/main/coffee-shop-kustomize/coffee-shop/base/deployment.yaml#L14.](https://github.com/redhat-gpte-devopsautomation/ocp48_hands_on_apps/blob/main/coffee-shop-kustomize/coffee-shop/base/deployment.yaml#L14)

您可以单击`REFRESH`来刷新实际部署的OpenShift容器平台资源的Argo CD视图。


##  9. 发现 New Logging Data

您已经将几个新的应用程序组件部署到`prod-coffeeshop`名称空间中。在本练习中，您将了解日志是如何更改的。


###  9.1 Add `prod-coffeeshop` Namespace to `ClusterLogForwarder`

 1. 转到OpenShift容器平台web控制台，选择管理员透视图.
 2. On the navigation menu, select `Home → Search`.
 3. From the `Project`: drop-down list, select the `openshift-logging` namespace.
 4. From the `Resources` drop-down list, select `ClusterLogForwarder`.
 5. Click the `CLF instance`.
 6. Click the `YAML` tab.
 7. Scroll to line 52, - `dev-coffeeshop`.
 8. 紧跟在第52行之后，将- prod- coffeshop作为新数组元素插入到该YAML数组中。
 
 Expect it to look like this:
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/f1ecfc7c336b44ea88b014dd9b0dfc01.png)
 9. Click `Save`.

###  9.2 验证 New Logs

 1. Go to the Kibana web console.
 2. Create a new index pattern to capture all of the Coffee Shop projects—both `dev-coffeeshop` and `prod-coffeeshop`:
a. On the navigation menu, click `Management`.
b. Click `Create index pattern` and add an index pattern named `*-coffeeshop-*`.
c. Click `Next`.
d. In the Time Filter field select `@timestamp`.
 3. From the navigation menu, click `Discover`.
 4. Select the `*-coffeeshop-*` index pattern you just built, and also the `Available Fields kubernetes.namespace_name` and `structured.message`.
  Expect to see results from both the dev-coffeeshop and prod-coffeeshop namespaces.
 5. Observe that new orders are also being processed by the `Create Order` cron job running in your cluster.


您观察到Argo CD在您的第一个生产咖啡店应用程序中单独部署应用程序。然后，您看到Argo CD部署了一个ApplicationSet，该ApplicationSet自动创建了两个应用程序:coffee-shop和barista。咖啡师恰好是Knative提供的服务。使用Argo CD来部署Kubernetes部署或Knative服务并不需要特别的活动。
您还看到了如何从prod- coffeshop名称空间获得日志结果，并且可以查询结构化的。*字段以获得更详细的分析。
在下一个实验中，您将为developmen创建一个简单的管道
