

---

 - [GitOps【1】理论认识](https://blog.csdn.net/xixihahalelehehe/article/details/122193489?spm=1001.2014.3001.5501)
 - [Argo CD【1】介绍与入门](https://blog.csdn.net/xixihahalelehehe/article/details/122238344?spm=1001.2014.3001.5501)
 - [Argo CD 【2】动手实践](https://blog.csdn.net/xixihahalelehehe/article/details/122242023?spm=1001.2014.3001.5501)

---


> 您是否听说过 `GitOps` 并且很想知道它的全部内容？在本片中，我们将描述 `GitOps` 工作流的原则和模式，以及如何实施它们以在生产中和大规模运行 Kubernetes。我们还将描述 GitOps 和基础设施即代码 (IAC) 配置管理工具之间的区别，当然还会向您展示如何将 GitOps 最佳实践作为您自己开发环境的一部分。

---


##  1. 什么是GitOps
GitOps 最早是在2017年由 Weaveworks 创立提出，它是一种进行 Kubernetes 集群管理和应用程序交付的方式。GitOps 使用 Git 作为声明性基础设施和应用程序的单一事实来源。GitOps 的核心思想是拥有一个 Git repository，包含目标环境中当前所需基础设施的声明性描述，以及使目标环境与 Git repository 中描述的状态相匹配的自动化过程。借助 GitOps，可以针对 Git  repository 与集群中运行的内容之间的任何差异发出警报，如果存在差异，Kubernetes reconcilers会根据情况自动更新或回滚集群。以 Git 作为 pipeline 的中心，开发人员可以使用自己熟悉的工具发出PR，以加速和简化 Kubernetes 中应用程序部署和操作任务。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9a67a64d30b49f8a71614e6c524fed0.png)

构建云原生应用的运营模式
GitOps 可以概括为以下两点：

 1. Kubernetes 和其他云原生技术的运营模型，提供了一组最佳实践，可统一针对容器化集群和应用程序的 Git 部署、管理和监控。
 2. 通往管理应用程序的开发人员体验的途径；其中端到端 CICD 管道和 Git 工作流应用于操作和开发。


##   2. GitOps 管理集群条件

 1. 整个系统以声明方式描述。
对于 Gitops，Kubernetes 只是许多现代云原生工具的一个例子，这些工具是“声明性的”并且可以被视为代码。声明性意味着配置由一组事实而不是一组指令来保证。在 Git 中对应用程序的声明进行版本控制后，您就有了单一的事实来源。然后，您的应用程序可以轻松部署到 Kubernetes 以及从 Kubernetes 回滚。更重要的是，当灾难发生时，您的集群基础设施也可以可靠且快速地重现。
 2. 在 Git 中版本化的规范所需系统状态。
将您的系统声明存储在版本控制系统中，并作为您的标准事实来源，您就有了一个单一的地方，可以从中派生和驱动一切。这使回滚变得微不足道；您可以在其中使用 `Git revert` 返回到之前的应用程序状态。借助 Git 出色的安全保证，您还可以使用 SSH 密钥对提交进行签名，从而对代码的作者身份和出处实施强大的安全保证。  
 3. 可自动应用于系统的已批准更改。
一旦您在 Git 中保存了声明的状态，下一步就是允许对该状态的任何更改自动应用于您的系统。重要的是，您不需要集群凭据即可对系统进行更改。对于 GitOps，有一个隔离的环境，状态定义位于其外部。这使您可以将您做什么和将如何做分开。
 4. 软件代理以确保正确性并在分歧时发出警报。
一旦您的系统状态被声明并保持在版本控制之下，软件代理就会在现实与您的期望不符时通知您。代理的使用还可以确保您的整个系统能够自我修复。通过自我修复，我们不仅仅指节点或 Pod 出现故障时（由 Kubernetes 处理），而是指更广泛的意义上，例如人为错误的情况。在这种情况下，软件代理充当您操作的反馈和控制回路。

##  3. GitOps 的主要优势

 1. 提高生产力
具有集成反馈控制回路的持续部署自动化加快了平均部署时间。您的团队每天可以交付 30-100 倍的变更，从而使整体开发产出增加 2-3 倍。
 2. 增强的开发人员体验
推送代码而不是容器。开发人员可以使用熟悉的工具（如 Git）更快速地管理 Kubernetes 的更新和功能，而无需了解 Kubernetes 的内部。新入职的开发人员可以在几天而不是几个月内快速上手并提高工作效率。
 3. 提高稳定性
当您使用 Git 工作流管理集群时，您会自动获得 Kubernetes 之外的所有集群更改的便捷审计日志。谁做了什么以及何时对您的集群进行审计跟踪，可用于满足 SOC 2 合规性并确保稳定性。  
 4. 更高的可靠性
借助 Git 的恢复/回滚和分叉功能，您可以获得稳定且可重现的回滚。因为您的整个系统是在 Git 中描述的，所以您还有一个单一的事实来源，可以在崩溃后从中恢复，将恢复时间 (MTTR) 从几小时缩短到几分钟。
 5. 一致性和标准化
由于 GitOps 提供了一种用于进行基础架构、应用程序和 Kubernetes 附加组件更改的模型，因此您在整个组织中拥有一致的端到端工作流。不仅您的持续集成和持续部署管道都由拉取请求驱动，而且您的运营任务也可以通过 Git 完全重现。  
 6. 更强大的安全保证
Git 强大的正确性和安全性保证，以用于跟踪和管理更改的强大加密技术为后盾，以及签署更改以证明作者和来源的能力是安全定义集群所需状态的关键。

##  4. GitOps 是持续交付与云原生的结合
GitOps 构建并迭代了从 DevOps 和站点可靠性工程中汲取的想法，这些想法始于 [Martin Fowler 在 2006 年的全面持续集成概述](https://martinfowler.com/articles/continuousIntegration.html)。 

作为 CI/CD 管道的工作流，GitOps 被 描述为[开发过程的圣杯](https://thenewstack.io/the-best-ci-cd-tool-for-kubernetes-doesnt-exist/)。  因为没有一个工具可以完成 CICD 管道中所需的一切，所以 GitOps 使您可以自由地为不同部分选择最佳工具。您可以从开源生态系统或闭源中选择一组工具，或者根据您的用例，您甚至可以将它们组合起来。创建 CICD 管道最困难的部分是将所有部分粘合在一起。

无论您为交付管道选择什么，将 GitOps 最佳实践与 Git（或任何版本控制）一起应用都应该是您流程的一个组成部分。这样做将使向持续交付的过渡更容易。这不仅从技术角度来看，而且从文化角度来看也是如此。

##  5. IAC 工具与 GitOps
以按需提供服务器的基础设施即代码工具已经存在了很长一段时间。这些工具起源于保持基础设施配置版本化、备份和可从源代码控制重现的概念。 

管理和比较基础架构和应用程序的当前状态的能力，以便您可以使用来自 Git 的完整审计跟踪来测试、部署、回滚、前滚，这就是 GitOps 哲学及其最佳实践的组成部分。这是可能的，因为 Kubernetes 几乎完全通过声明性配置进行管理，并且容器是不可变的。我们使用 [Terraform](https://www.terraform.io/) 和 [Ansible](https://www.ansible.com/) 来配置服务器。我们还会在 Git 中备份这些配置文件并对其进行版本控制。IAC 工具及其相关配置文件构成了我们 GitOps 工作流程的核心部分。[Chef](https://www.chef.io/)、[Puppet](https://puppet.com/) 和 [Ansible](https://www.ansible.com) 等 IAC 工具支持“差异警报”等功能。这些有助于操作员了解何时可能需要采取措施将实时系统“融合”到预期状态（由配置脚本定义）。最近，最佳实践是部署不可变的镜像（例如容器），以便不太可能出现分歧。

在“GitOps”模型中，我们使用 Git 来解决发散和收敛问题，并借助一组“差异”和“同步”工具（[kubediff](https://github.com/weaveworks/kubediff)， 以及 [terradiff](https://github.com/jml/terradiff) 和 ansiblediff）将预期状态与实际状态进行比较。

##  6. GitOps 工作流
开发人员工作流程：

 - 新功能的拉取请求被推送到 GitHub 以供审查。
 - 代码由同事审查和批准。修改代码并重新批准后，将其合并到 Git。
 - Git 合并触发 CI 和构建管道，运行一系列测试，然后最终构建一个新映像并将新映像存入镜像仓库。
 - Weave Cloud 'Deployment Automator'监视镜像仓库，从镜像仓库中提取新镜像并在配置存储库中更新其 YAML。
 - Weave Cloud 'Deployment Synchronizer'（安装到集群）检测到集群已过期。它从配置存储库中提取更改的清单并将新功能部署到生产中。

启用 GitOps 的 CICD 管道：
![在这里插入图片描述](https://img-blog.csdnimg.cn/bc2a8565a7f448b18b8de99478782964.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ae978a5d586e4120a5828d6268c276a8.png)
GitOps 是一个面向发布的操作和功能模型。您向客户交付新功能的速度在一定程度上取决于您的团队在此周期中各个阶段的完成速度。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e2a87d0d9b8f49fb8fc3e0500d22ca3a.png)
##  7. GitOps 是如何工作的呢？
###  7.1 把环境配置作为 Git repository
GitOps 以代码库为核心来组织部署。我们需要至少有两个仓库：应用程序库和环境配置库。应用程序库包含应用程序的源代码和部署应用程序的 manifests。环境配置库包含部署环境当前所需基础架构的所有部署manifests。它描述了哪些应用程序和基础设施服务（消息代理、服务网格、监控工具等）应该在部署环境中以何种配置和版本运行等内容。

两种部署类型之间的区别在于如何确保部署环境与所需的基础架构相同。这里推荐，首选基于 pull 的方法，实现 GitOps 更安全、也有很多已有的最佳实践来借鉴。

###  7.2 基于 pull 的部署
传统的 CI/CD pipeline由外部事件触发，比如新代码被推送到应用程序库时，就触发了。

而基于 Pull 的部署方法，引入了operator。它通过不断将环境配置库中的期望状态与部署环境中的实际状态进行比较来接管pipeline的角色。当发现差异时，operator会更新部署环境中的状态以匹配环境配置库。此外，它还可以监控 `image registry` ，以查找要部署的新镜像版本。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5189ef7743af4296827eaa9fed807fea.png)
基于 pull 模型的部署不仅能做到环境配置库更改时更新环境；

`operator`也能做到当实际环境与环境配置库中存在差异时进行还原。

这就确保了所有更改都可以在 Git 日志中进行跟踪，因为任何人都不允许对集群进行直接更改。

那么，这种方式的监控点就集中在 operator 及各个组件上了（比如，镜像仓库是否能正常拉取到镜像等等）。

为了避免基于 Push 场景中的上帝模式权限问题，operator 应该始终与要部署的应用程序在同一环境或集群中。（k8s RBAC授权：Kubernetes 从 1.6 开始支持基于角色的访问控制机制（Role-Based Access，RBAC），集群管理员可以对用户或 service account 的角色进行更精确的资源访问控制。在 RBAC 中，权限与角色相关联，用户通过成为适当角色的成员而得到这些角色的权限。）




### 7.3 基于 push 的部署
基于 Push 的部署策略可以利用流行的 CI/CD 工具来实现，例如 [Jenkins](https://www.jenkins.io/)、[CircleCI](https://circleci.com/) 或 [Travis CI](https://www.lambdatest.com/automate-selenium-tests-with-travisci?utm_source=google&utm_medium=cpc&utm_campaign=%5BHOP%5D%20Test%20Automation%20APAC&utm_term=&utm_id=12741819968&gclid=CjwKCAiAiKuOBhBQEiwAId_sK3RqSKJRWIj7xBaHMcN4lR24_fTIsAdwY-xkQ-RFWzw4-zDOag0M4xoCnVMQAvD_BwE)。应用程序的源代码与部署的应用程序所需的 Kubernetes YAML 一起存在于应用程序库中。每当更新应用程序代码时，都会触发构建pipeline，构建容器镜像，最后使用新的部署manifest，更新环境配置库。

也可以将 YAML 的模板存储在应用程序库中。构建新版本时，可以使用模板在环境配置库中生成 YAML。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b16508116ec44c6e80359bdc96e3cf41.png)
对环境配置库的更改会触发部署`pipeline`。pipeline负责将环境配置库中的所有manifests应用到基础设施。这就需要我们更关注部署权限细分及控制。同时，这种方式无法自动注意到环境及其所需状态的任何偏差。我们需要额外的监控报警方式，来保障环境与环境存储库中描述的内容一致。

对于大多数应用程序来说，只使用一个应用程序库和一个环境配置库是不现实的。GitOps 也能应对。可以设置多个构建 pipeline 来更新环境配置库。然后就像上两个描述过程一样，自动化 GitOps 工作流程开始并部署应用程序。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f56f02de9f0646918f4d4a05da8f6223.png)
我们需要在环境配置库中使用独立的分支，来管理多个环境。选择设置 operator 或者构建pipeline，自动化 GitOps 工作流程开始并部署应用程序。

---
相关阅读：

 - [亲自尝试 GitOps](https://www.weave.works/product/gitops-core/)
 - [GitOps：通过拉取请求操作](https://www.weave.works/blog/gitops-operations-by-pull-request)
 - [GitOps 管道 - 第 2 部分](https://www.weave.works/blog/the-gitops-pipeline)
 - [GitOps：“Git Push”所有东西（新堆栈）](https://thenewstack.io/gitops-git-push-all-the-things/)
 - [Kubernetes 的最佳 CI/CD 工具不存在](https://thenewstack.io/the-best-ci-cd-tool-for-kubernetes-doesnt-exist/)
 - [GitOps 2017-2020 的完整历史](https://assets.contentstack.io/v3/assets/blt300387d93dabf50e/blt326ac6188d55a84a/60643ace4f68540feec1038c/History_of_GitOps_Timeline.pdf)
 - [Kubernetes 的 GitOps：专注于声明式基础设施的 DevOps 迭代](https://thenewstack.io/gitops-kubernetes-devops-iteration-focused-declarative-infrastructure/)
 - [为什么我们使用 Terraform 而不是 Chef、Puppet、Ansible、SaltStack 或 CloudFormation](https://blog.gruntwork.io/why-we-use-terraform-and-not-chef-puppet-ansible-saltstack-or-cloudformation-7989dad2865c?gi=9f77c3185034)
 - [GitOps 到底是什么？](https://www.weave.works/blog/what-is-gitops-really)
 - [Weaveworks 和 AWS；我们如何管理 Kubernetes 集群](https://www.weave.works/technologies/kubernetes-on-aws/)
 - [生产就绪 Kubernetes 集群的配置和生命周期](https://www.weave.works/blog/provisioning-lifecycle-production-ready-kubernetes-cluster/)
 - [GitOps 常见问题](https://www.weave.works/technologies/gitops-frequently-asked-questions/)
 - [将 Kubernetes Operator 模式与替代方案进行比较](https://cloudark.medium.com/why-to-write-kubernetes-operators-9b1e32a24814)
 - [操作员介绍：将操作知识融入软件](https://cloud.redhat.com/blog)
 - [Kubernetes 的 CI/CD：你需要知道的](https://www.weave.works/technologies/ci-cd-for-kubernetes/)
 - [您的管道有多安全？](https://www.weave.works/blog/how-secure-is-your-cicd-pipeline)
 - [GitOps 可观察性](https://www.weave.works/blog/gitops-part-3-observability)
 - [使用 Prometheus 监控 Kubernetes](https://www.weave.works/technologies/monitoring-kubernetes-with-prometheus/#deploy-new-features)
 - [GitOps：Kubernetes 的高速 CICD](https://www.weave.works/blog/gitops-high-velocity-cicd-for-kubernetes)
 - [使用 Wea​​ve Cloud 的 GitOps 工作流程 - 教程](https://www.weave.works/blog/gitops-workflows-for-istio-canary-deployments)
 - [如何使用 GitOps 提高业务绩效：事实](https://go.weave.works/DORA_GitOps_WP.html?LeadSource=Web%20Content&CampaignID=7014M000001z7lD&LSD=Website)
 - [使用 Kubernetes 和 GitOps 的多云策略](https://go.weave.works/2021_multi_cloud_strategies_wp.html?LeadSource=WebContent&LSD=Website&CampaignID=7014M000001z9sV)


参考链接：
[GitOps 应用实践系列 - 综述（一）](https://mp.weixin.qq.com/s?__biz=MzI2ODAwMzUwNA==&mid=2649296484&idx=1&sn=03e9d37f5919bf6702f3463644b3291c&chksm=f2eb9fbbc59c16ad41151faeacdccf5d9e9d52bde12e77d48234c87a837e07b90240153f4701&scene=21#wechat_redirect)
[https://www.weave.works/technologies/gitops/](https://www.weave.works/technologies/gitops/)
