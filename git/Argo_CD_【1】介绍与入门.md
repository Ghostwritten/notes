

---

 - [GitOps【1】理论认识](https://blog.csdn.net/xixihahalelehehe/article/details/122193489?spm=1001.2014.3001.5501)
 - [Argo CD【1】介绍与入门](https://blog.csdn.net/xixihahalelehehe/article/details/122238344?spm=1001.2014.3001.5501)
 - [Argo CD 【2】动手实践](https://blog.csdn.net/xixihahalelehehe/article/details/122242023?spm=1001.2014.3001.5501)

---

##  1. Argo 概览
2020 年 4 月 7 日，CNCF Technical Oversight Committee (TOC) 投票通过 [Argo](https://argoproj.github.io/) 项目进入 [CNCF](https://www.cncf.io/) 孵化。

其实 [Argo](https://argoproj.github.io/) 是在 2017 年由 `Applatix` 公司创建的（这家公司的主要产品和服务目前主要与 AWS 进行深度融合，当然也包括一些多云的服务），2018 年初被 Intuit 收购。之后，`BlackRock` 为 Argo 项目贡献了 `Argo Events` 子项目。两家公司都积极参与项目和社区的开发和培育中。

Argo 及其子项目提供了一种简单的方法管理 Workflow，事件和应用程序。所有 Argo 工具都通过 CRD 的方式实现。他们可以使用或集成其他 CNCF 项目，如 [gRPC](https://grpc.io/)、[Prometheus](https://prometheus.io/)、[NATS](https://nats.io/)、[Helm](https://helm.sh/) 和 [CloudEvents](https://cloudevents.io/)。

Argo 生态目前主要由四个子项目组成，包括：

 - `Argo Workflows` -- 第一个 Argo 项目，是 Kubernetes 的原生工作流引擎，支持 `DAG` 和`step-based` 的工作流；
 - `Argo Events` -- Kubernetes 上的基于事件的依赖管理器，用于触发 Kubernetes 中的 Argo工作流和其他操作。
 - `Argo CD` -- 是 Argo 社区和 Intuit 带来的开源项目，支持基于 GitOps 的声明性部署 Kubernetes 资源。
 - `Argo Rollouts` -- 支持声明式渐进式交付策略，例如 canary 、blue-green 和更多形式。

## 2. Argo CD 介绍

Argo CD 旨在提供一个声明式持续交付 (CD) 工具。**Argo CD 支持多种配置管理工具，包括 ksonnet /jsonnet，[kustomize](https://blog.csdn.net/xixihahalelehehe/article/details/111223923?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164085068916780366534627%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164085068916780366534627&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-111223923.nonecase&utm_term=kustomize&spm=1018.2226.3001.4450) 和 [Helm](https://helm.sh/) 等。Argo CD 扩展了声明式和基于 Git 的配置管理的优势，以在不影响安全性和合规性的情况下加速应用程序的部署和生命周期管理**。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ae8ba276c57741bdbdbd01af2061797a.png)


##  3. Argo CD 适用场景

 - 应用程序及其部署环境的配置是声明性的，并且是版本可控的；
 - 应用程序部署和生命周期管理简单、可自动和可审计（企业友好）；
 - 应用程序部署快速、可靠和幂等；
 - 需要检测并纠正与版本控制配置的任何偏差；
 - 回滚简单；

##  4. Argo CD 特征

 - 将应用程序自动部署到指定的目标环境
 - 支持多种配置管理/模板工具（Kustomize、Helm、Ksonnet、Jsonnet、plain-YAML）
 - 能够管理和部署到多个集群
 - SSO 集成（OIDC、OAuth2、LDAP、SAML 2.0、GitHub、GitLab、Microsoft、LinkedIn）
 - 用于授权的多租户和 RBAC 策略
 - 回滚/随处回滚到 Git 存储库中提交的任何应用程序配置
 - 应用资源健康状况分析
 - 自动配置漂移检测和可视化
 - 自动或手动将应用程序同步到所需状态
 - 提供应用程序活动实时视图的 Web UI
 - 用于自动化和 CI 集成的 CLI
 - Webhook 集成（GitHub、BitBucket、GitLab）
 - 自动化的访问令牌
 - PreSync、Sync、PostSync 挂钩以支持复杂的应用程序部署（例如蓝色/绿色和金丝雀升级）
 - 应用程序事件和 API 调用的审计跟踪
 - 普罗米修斯指标
 - 用于覆盖 Git 中的 ksonnet/helm 参数的参数覆盖

## 5.  Argo CD 架构
![在这里插入图片描述](https://img-blog.csdnimg.cn/8b781df8963a41589b18301c09a63201.png)
## 6.  Argo CD 组成
从整体上看，Argo CD 有三个主要的组成部分：`API Server` 、`Repository Server` 、`Application Controller`。




###  6.1 API Server
Argo CD 的 API Server 是一个 `gRPC/REST server`，它公开 `Web UI`、`CLI` 以及一些其他场景需要用到的 API。

它主要进行以下几个内容：

 - 应用程序管理和状态报告；
 - 调用应用程序操作（例如：同步、回滚、用户定义的操作）；
 - repository 和集群 credential 管理（存 K8s secrets）；
 - 身份验证和授权委托给外部身份认证组件；
 - RBAC（Role-based access control 基于角色的访问控制）；
 - Git webhook 事件的 listener/forwarder；

###  6.2 Repository Server
`Repository Server` 是一个内部服务，它负责保存应用程序 Git 仓库的本地缓存，并负责生成和返回可供 Kubernetes 使用的 `manifests`，它接受的输入信息主要有以下内容：	

 - 仓库地址（URL）
 - revision（commit, tag, branch）
 - 应用程序路径
 - 模板的特定设置：参数、ksonnet 环境、helm values.yaml

###  6.3 Application Controller
`Application Controller` 是一个 `Kubernetes controller`，它持续监听正在运行的应用程序并将当前的实时状态与所需的目标状态（如 repo 中指定的）进行比较。它检测 `OutOfSync` 应用程序状态并有选择地采取纠正措施。它负责为生命周期事件（PreSync、Sync、PostSync）调用任何用户定义的 hooks。


## 7. Argo CD 核心概念
假设您熟悉核心 Git、Docker、Kubernetes、持续交付和 GitOps 概念。

 - `Application`清单定义的一组 Kubernetes 资源。这是自定义资源定义 (CRD)。
 - `Application source type`哪些工具来构建应用程序。
 - `Target state`应用程序所需的状态，由 Git 存储库中的文件表示。
 - `Live state`该应用程序的实时状态。部署了哪些 Pod 等。
 - `Sync status`实时状态是否与目标状态匹配。部署的应用程序是否与 Git 所说的一样？
 - `Sync`使应用程序移动到其目标状态的过程。例如，通过对 Kubernetes 集群应用更改。
 - `Sync operation status` 同步是否成功。
 - `Refresh`将 Git 中的最新代码与实时状态进行比较。弄清楚有什么不同。
 - `Health`应用程序的健康状况，是否正常运行？它可以服务请求吗？
 - 工具一种从文件目录创建清单的工具。例如 Kustomize 或 Ksonnet。请参阅应用程序源类型。

## 8. Argo CD 基础必备

 - [容器、VM 和 Docker 的初学者友好介绍](https://www.freecodecamp.org/news/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b/)
 - [Kubernetes 简介](https://www.edx.org/course/introduction-to-kubernetes)
 - [教程](https://kubernetes.io/docs/tutorials/)
 - [动手实验室](https://katacoda.com/courses/kubernetes/)
 - [Kustomize](https://kustomize.io/)
 - [Helm](https://helm.sh/)
 - [Jenkins User Guide](https://www.jenkins.io/)





##  9. Argo CD 版本
自2019年3月17号发布了 `v1.0.0` 版本开始，到现在，`Argo CD`已经全面进入了 `v2.x` 时代，当前最新的版本是 `v2.1.5`。

在 2.0 中引入了`Pods View`功能、重写了日志可视化、新增了通知横幅功能、还有很多后台操作及自定义操作，并且致力于打造 `Argo CD Core` （轻量级 `Argo CD` 发行版，仅打包核心 GitOps 功能，依赖`Kubernetes API/RBAC` 为 UI 和 CLI 提供支持）。

 - `Pods View` ：对于拥有数百个 Pod 的应用程序特别有用。它没有可视化应用程序的所有 Kubernetes 资源，而是仅显示Kubernetes pod 和密切相关的资源。
 - 新日志可视化：支持分页、过滤、禁用/启用日志流的能力，甚至为终端爱好者提供暗模式。支持查看多个部署 Pod 的聚合日志，Argo CD CLI 也支持日志流。
 - UI通知横幅功能：使用`ConfigMap` 中的`ui.bannercontent`和`ui.bannerurl`属性指定通知消息和可选 URL argocd-cm。
 - 后台操作：资源的`deletion/pruning`、仅同步更改的资源、`Prune Last`、引入了对`Sealed-secrets`、`kubernetes-external-secrets` 和 `strimzi CRD` 的健康检查。

##  10. Argo CD 部署简要
Argo CD 有四种安装方式：多租户（multi-tenant）、 core 、自定义 、Helm。

### 10.1 多租户（multi-tenant）
多租户是 `Argo CD` 使用最多的部署方式。用户可以使用 `Web UI` 或 `argocd` CLI 通过 API Server访问 Argo CD 。

该 argocd CLI 必须先执行 `argocd login <server-host>`命令。[更多细节请参考](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd_login/)

```bash
argocd login SERVER [flags]

# Login to Argo CD using a username and password
argocd login cd.argoproj.io

# Login to Argo CD using SSO
argocd login cd.argoproj.io --sso

# Configure direct access using Kubernetes API server
argocd login cd.argoproj.io --core
```
在项目的 GitHub 仓库中默认提供了两种部署方式（manifests）：
###  10.2 非高可用
这种类型的安装通常用于演示和测试。（不推荐用于生产）

`install.yaml` - 具有集群管理员权限的标准 Argo CD 安装。可使用 Argo CD 在其运行的同一集群中部署应用程序，当然也可以使用输入的 `credentials` 部署到外部集群。

`namespace-install.yaml` - 只需要命名空间级别的权限（不需要集群管理员权限）。但这种安装的话 Argo CD 不能在其所运行的同一集群中部署应用程序，并且将仅依赖于输入的外部集群 credentials 。

> 注意：Argo CD CRD 不包含在 namespace-install.yaml 中 必须单独安装。CRD manifests位于manifests/crds目录中。使用以下命令安装它们：

```bash
kubectl apply -k https://github.com/argoproj/argo-cd/manifests/crds\?ref\=stable
```
### 10.3 高可用
强烈建议生产环境使用高可用方式安装。

bundle 包含相同的组件，但针对高可用性和弹性进行了调整。

 - `ha/install.yaml` - 与上文中提到的 install.yaml 相同，但配置了相关组件的多个副本；
 - `ha/namespace-install.yaml` - 与上文提到的 namespace-install.yaml
   相同，但配置了相关组件的多个副本；


###  10.4 core
core 安装最适合独立使用 Argo CD 且不需要多租户功能的集群管理员。

此安装包括较少的组件，并且更易于设置。bundle 不包括 API Server或 UI，并只安装每个组件的轻量级（非 HA）版本。

用户需要 Kubernetes 访问权限来管理 Argo CD。该 argocd CLI 必须使用下面的命令进行配置：

```bash
kubectl config set-context --current --namespace=argocd
argocd login --core
```
Web UI 也可以使用，可以使用以下命令启动。

```bash
argocd admin dashboard
```
具体的 `manifests` 对应于仓库中的 `core-install.yaml` 。

###  10.5 kustomize
Argo CD manifests 也可以使用自定义安装。建议将 manifests 作为远程资源包含在内，并使用 Kustomize 补丁应用其他自定义。

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: argocd
resources:
- https://raw.githubusercontent.com/argoproj/argo-cd/v2.0.4/manifests/ha/install.yaml
```
###  10.6 Helm
Argo CD 可以使用Helm安装。Helm chart 目前由社区维护，访问 [https://github.com/argoproj/argo-helm/tree/master/charts/argo-cd](https://github.com/argoproj/argo-helm/tree/master/charts/argo-cd) 获取即可，此处不再添加示例。


