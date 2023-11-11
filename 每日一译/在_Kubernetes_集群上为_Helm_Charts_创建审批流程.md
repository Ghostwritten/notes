![在这里插入图片描述](https://img-blog.csdnimg.cn/74b2ebac471146f68882f43cd541b3be.png)


##  1. 前言
Kubernetes 采用声明式方法来部署和管理云基础设施以及应用程序。您声明您想要的状态，而不指定如何实现它的精确操作或步骤，Kubernetes 可以实现它。 

单个应用程序可能会受到多个声明性配置的影响，并在多个 YAML 清单文件中进行管理。如果您有许多应用程序，维护所有独立资源及其 YAML 清单文件可能会变得很耗时。

为了解决这个问题并使维护更容易，许多组织已经转向[Helm](https://helm.sh/)，这是一种支持应用程序管理的开源 Kubernetes 部署工具。通过使用 Helm Charts，DevOps 团队可以为其集群上运行的应用程序和服务创建和部署可重用的定义文件包，从而大大简化了维护工作。

对于任何可重复使用的组件，在该组件被大规模合并之前，对质量保证和安全风险进行额外的审查是至关重要的。 

在本指南中，我们将概述使用 Helm 图表的基础知识，以及为什么您的组织应该在将 Helm 图表安装到 Kubernetes 集群之前对其进行审批。

##  2. 什么是 Helm Charts？
Helm 是一种部署工具，可帮助团队自动创建、打包、配置和部署应用程序和服务到 Kubernetes 集群。

Helm Charts 是 YAML 文件和模板的包，以目录格式构建，用作运行应用程序或服务所需的 Kubernetes 清单文件。以下是图表“SampleChart”的图表结构示例：

```bash
SampleChart/
Chart.yaml          # A YAML file containing information about the chart
LICENSE             # OPTIONAL: A plain text file containing the license for the chart 
README.md           # OPTIONAL: A human-readable README file 
values.yaml         # The default configuration values for this chart 
values.schema.json  # OPTIONAL: A JSON Schema for imposing a structure on the values.yaml file 
charts/             # A directory containing any charts upon which this chart depends. 
crds/               # Custom Resource Definitions 
templates/          # A directory of templates that, when combined with values,                     
                    # will generate valid Kubernetes manifest files. 
templates/NOTES.txt # OPTIONAL: A plain text file containing short usage notes
```

> 资料来源：[Helm 文档 - 图表文件结构](https://helm.sh/docs/topics/charts/)

您可以将一个图表多次安装到同一个集群中，每次安装称为“发布”。这使您可以将图表用于特定于部署的配置，而不是手动编写单个 YAML 文件的过程。例如，您不需要单独的 Helm Chart 来在生产、暂存和开发环境中部署容器化应用程序。

## 3.  使用内部服务 Helm 图表 VS 第三方 Helm 图表

内部服务图表是从头开始创建的 Helm 图表。您可以通过配置镜像拉取策略并在 Kubernetes 资源的values.yaml文件中添加规范来创建 Helm Chart 。这种创建图表的方法确实需要深入了解 Kubernetes 资源。

如果您的组织有特定用途的标准图表，您可以使用以下 Helm 命令搜索本地 Helm 客户端中添加的内部存储库：

```bash
helm search repo
```

您还可以从公开可用的存储库中安装 Helm Charts。ArtifactHub中有数千个存储库，您可以利用它们来查找有用的图表。

您可以使用以下命令从 ArtifactHub 搜索图表：

```bash
helm search hub
```

helm 搜索中心将显示 ArtifactHub 和您的中心中所有公开可用的 Helm 图表。您可以轻松访问数千个图表，但在生产中的集群上安装图表之前需要考虑一些事项。

## 4.  为第 3 方 Helm 图表构建审批工作流
许多组织都有一个批准流程来限制预配置 Helm Charts 的下载。虽然 Helm 通过打包依赖项和默认安全设置来轻松扩展部署服务，但大多数 Helm Charts 默认情况下没有安全性，尤其是从存储库下载的那些。 

根据 Bridgecrew 的一份报告， 618 个被扫描的 Helm 存储库中有71%包含错误配置和安全问题，例如运行 root 容器、允许权限提升以及未设置资源限制。这就是为什么组织通常有一个批准流程来限制预配置 Helm Charts 的安装，以便在大规模部署之前检查它们是否存在安全漏洞。

如果您没有适当的批准流程，请考虑以下一些步骤： 

### 4.1 提交新图表
首先，您需要一种标准的方式来提交 Helm 图表以供批准。这可能是 Jira 票证或其他一些自动化流程，用于标记来自安全团队的合适人员并列出请求所处的阶段，例如“已收到”或“审核中”。

### 4.2 解决安全和质量差距
当您的团队审查新图表时，有一个标准的安全问题清单很有用。如果图表已经具有强大的安全设置，那么选中这些框可能会加快批准速度。

加快审查速度的另一种方法是利用自动安全扫描。您可以使用像[Snyk CLI这样的商业工具或像Bridgecrew这样的开源工具来运行 Helm 图表安全扫描](https://docs.snyk.io/products/snyk-infrastructure-as-code/scan-kubernetes-configuration-files/scan-and-fix-security-issues-in-helm-charts)。

如果第 3 方图表存在安全问题，您或您的安全团队可以尝试通过发布补丁来解决问题，因为 Helm 图表是高度可配置的。如果出现任何问题，Helm 会提供一个简单的解决方案，包括回滚和版本控制。

如果图表存在重大安全问题或在某些方面不符合组织标准，则可以拒绝该图表并建议替代方案。

### 4.3 将图表添加到已批准的图表列表
对于成功的批准流程，重要的是及时清楚地传达工单的当前状态和结果的原因。在内部推广批准的图表和参考用例也很有用，它们非常适合防止返工或不必要的批准提交。


## 5. 如何在集群上安装 Helm Chart
要在 Kubernetes 集群上安装批准的 Helm Chart，您必须拥有最新版本的 Helm和kubectl的本地配置副本。您可以使用 helm install 命令安装 Helm Chart：

```bash
helm install <release name> <chart name>
```

此安装命令将图表部署到 Kubernetes 集群，并使用发布名称对其进行标记。唯一的版本名称允许图表多次安装在同一个集群中。在安装过程中，helm 客户端将显示有关创建的资源、发布状态以及您可以采取的额外配置步骤的信息。

您可以通过覆盖 YAML 文件中的默认设置来配置设置，并将它们包含在安装中。如果不需要更改，您还可以将图表回滚到以前的版本。

## 6. 使用 Blink 自动化 Helm Chart 审批流程
Helm 图表简化了您管理 Kubernetes 应用程序的方式，但设置审批流程可能意味着需要处理更多的支持票。

[创建 Blink 帐户](https://app.blinkops.com/signup)时，您可以使用无代码/低代码步骤快速构建新 Helm 图表的智能自助审批流程。 

当从业者提交新的 Helm 图表以供批准时，您可以自动开始扫描安全漏洞和基本资格检查。然后，如果您希望安全团队中的某个人手动批准每个图表，您可以将安全扫描的输出发送到 Slack 频道，并像单击按钮一样简单地进行批准或拒绝。

通过自动化的审批流程，您可以减少团队的瓶颈，同时还可以轻松确保安全实践。

立即[创建您的免费 Blink 帐户](https://app.blinkops.com/signup)并简化您的审批流程。
