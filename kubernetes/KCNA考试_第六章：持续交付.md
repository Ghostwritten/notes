


---
##  1. 简介
这些年来，在任何平台上部署应用程序都有了很大的进步。一开始,应用程序可能会在同一台机器上执行他们写,后经由物理媒介(软盘、u盘、CD),现在我们在代码中检查服务器,构建和应用程序,把它放在一个容器,直接将其部署到一个平台像Kubernetes。
我们交付应用程序的方式深受DevOps运动的影响，DevOps运动在2000年代后期取得了突破。DevOps运动是一场文化变革，带来了许多新方法

##  2. 学习目标
在本章结束时，你应该能够:

 1. 讨论自动化在集成和交付应用程序中的重要性。
 2. 理解对Git和版本控制系统的需求。
 3. 解释什么是CI/CD管道。
 4. 讨论作为代码的基础设施(IaS)的概念。
 5. 讨论GitOps的原理以及它是如何与Kubernetes整合的。

## 3. 应用程序交付
每个应用程序的生命周期都是从编写的代码开始的。源代码不仅是应用程序的基础，而且是知识产权，因此是大多数公司或个人的资本。我们很久以前就发现，管理源代码的最佳方式是版本控制系统。

在2005年，Linus Torvalds创建了Git，这是现在几乎所有人都在使用的标准版本控制系统。Git是一个分散的系统，可以用来跟踪源代码中的更改。本质上，Git可以在您的更改合并回主分支之前，使用代码副本(在分支或分支中调用)。

请务必查看这个网页以了解更多关于[git](https://git-scm.com/)的信息，因为它是一个强大的行业标准工具，几乎所有的开发人员和管理员每天都在使用它。
在检查源代码之后，交付应用程序之前的下一步就是构建它，这也可能包括容器映像的构建，如容器编排一章所述。
为了确保你的应用的高质量，下一步应该是广泛和自动测试应用程序，以确保所有功能仍然在有人作出更改后。

最后一步是将应用程序交付到它应该运行的平台。如果您的目标平台是Kubernetes，那么您可以编写一个YAML文件来部署您的应用程序，同时您新构建的容器映像可以推送到容器注册中心，Kubernetes会在那里为您下载它。

今天，源代码并不是版本控制系统中管理的唯一内容。为了充分利用云资源，基础设施作为代码(IaC)的原则开始流行起来。您可以在文件中描述基础设施，并使用云供应商的API来设置基础设施，而不是手动安装基础设施。这允许开发人员更多地参与基础设施的设置。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d2f42384f16b14855cbcb5764d245f77.gif#pic_center)


## 4. CI / CD
随着服务越来越小，部署越来越频繁，一个合乎逻辑且重要的步骤就是部署过程的自动化。DevOps运动强调了频繁和快速部署的重要性。在传统的设置中，部署将包括开发人员和管理员，许多容易出错的手动步骤，以及对某些东西会损坏的持续担心。

自动化是克服这些障碍的关键，今天我们知道并使用持续集成/持续交付(CI/CD)的原则，它描述了应用程序部署、配置甚至基础设施的不同步骤。

 - 持续集成是这个过程的第一部分，它描述了编写代码的永久构建和测试。高度自动化和版 本控制的使用允许多个开发人员和团队在相同的代码库上工作。
 - 持续交付是过程的第二部分，它将预构建软件的部署自动化。在云环境中，您经常会看到软件在发布和交付到生产系统之前被部署到开发或交付环境中。

要自动化整个工作流程，您可以使用CI/CD管道，它实际上只不过是所有相关步骤的脚本形式，在服务器上甚至容器中运行。管道应该与管理代码库更改的版本控制系统集成。每当您的代码有新的修订准备部署时，管道就开始执行脚本，这些脚本构建代码、运行测试、将它们部署到服务器，甚至执行安全性和遵从性检查。
除了管道步骤的通用脚本之外，现代CI/CD工具还有更多的功能，比如直接交互和来自Kubernetes这样的系统的反馈。

流行的CI/CD工具包括:

 - [Spinnaker](https://spinnaker.io/)
 - [GitLab](https://about.gitlab.com/)
 - [Jenkins](https://www.jenkins.io/)
 - [Jenkins X](https://jenkins-x.io/)
 - [Tekton CD](https://github.com/tektoncd/pipeline)
 - [Argo.](https://argoproj.github.io/)

为了更深入地了解DevOps、站点可靠性工程和基础设施作为代码，我们强烈建议您学习DevOps和[站点可靠性工程概论(LFS162)](https://training.linuxfoundation.org/training/introduction-to-devops)，这是edX上的一门免费课程。

##  5. GitOps
作为代码的基础设施在提高提供基础设施的质量和速度方面是一场真正的革命，它工作得如此之好，以至于今天，配置、网络、策略或安全都可以被描述为代码，甚至通常与软件位于同一个存储库中。

GitOps进一步将Git的概念作为真理的单一来源，并将基础设施的配置和更改过程与版本控制操作集成在一起。
如果代码是分支的，并且应该合并回主分支，您可以创建一个合并或拉取请求，在实际合并之前，其他开发人员可以对其进行审查。这在软件开发中已经是一个很长时间的最佳实践了，并且还包括为每一个应该进行的更改运行CI管道。在GitOps中，这些合并请求用于管理基础设施更改。

CI/CD管道有两种不同的方法来实现你想要的更改:

 - Push-based
管道启动并运行在平台中进行更改的工具。变更可以由提交或合并请求触发。

 - Pull-based
代理会监视git存储库的变化，并将存储库中的定义与实际运行状态进行比较。如果检测到更改，则代理将更改应用到基础设施。

使用基于拉的方法的两个流行的GitOps框架的例子是[Flux](https://fluxcd.io/)和[ArgoCD](https://argo-cd.readthedocs.io/en/stable/)。ArgoCD是作为Kubernetes控制器实现的，而Flux是用GitOps工具包构建的，这是一组api和控制器，可以用来扩展Flux，甚至构建一个定制的交付平台。
ArgoCD架构很好地概述了GitOps是如何实现的。

[ArgoCD架构，检索自ArgoCD文档](https://argo-cd.readthedocs.io/en/stable/operator-manual/architecture/)
Kubernetes特别适合GitOps，因为它提供了一个API，并且从一开始就为声明性的资源配置和更改而设计。您可能会注意到，Kubernetes使用了类似于基于拉的方法的思想:监视数据库的变化，如果变化不匹配所需的状态，则将其应用到运行状态。
要了解更多关于GitOps的运行情况以及ArgoCD和Flux的使用，可以考虑报名[免费课程《GitOps入门》(LFS169)](https://training.linuxfoundation.org/training/introduction-to-gitops-lfs169/)。

##  6. 其它资源


10 Deploys Per Day - Start of the DevOps movement at Flickr

 - [Velocity 09: John Allspaw and Paul Hammond, "10+ Deploys Per Day"](https://www.youtube.com/watch?v=LdOe18KhtT4)
 - [10+ Deploys Per Day: Dev and Ops Cooperation at Flickr](https://www.slideshare.net/jallspaw/10-deploys-per-day-dev-and-ops-cooperation-at-flickr), by John Allspaw and Paul Hammond

Learn git in a playful way

 - [Oh My Git! An open source game about learning Git!](https://ohmygit.org/)
 - [Learn Git Branching](https://learngitbranching.js.org/)

Infrastructure as Code

 - [Delivering Cloud Native Infrastructure as Code](https://www.pulumi.com/whitepapers/delivering-cloud-native-infrastructure-as-code/)
 - [Unlocking the Cloud Operating Model: Provisioning](https://www.hashicorp.com/resources/unlocking-the-cloud-operating-model-provisioning)

Beginners guide to CI/CD

 - [GitLab's guide to CI/CD for beginners](https://about.gitlab.com/blog/2020/07/06/beginner-guide-ci-cd/)



---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>
 - [KCNA考试 第一章：cloud foundry云原生工程师考试](https://ghostwritten.blog.csdn.net/article/details/121482847)
 - [KCNA考试 第二章：Cloud Native Architecture](https://ghostwritten.blog.csdn.net/article/details/121492840)
 - [KCNA考试 第三章：容器编排](https://ghostwritten.blog.csdn.net/article/details/121527922)
 - [KCNA考试 第四章：kubernetes需要掌握的基础知识](https://ghostwritten.blog.csdn.net/article/details/123477851)
 - [KCNA考试 第五章：kubernetes实践](https://ghostwritten.blog.csdn.net/article/details/123496572)
 - [KCNA考试 第六章：持续交付](https://ghostwritten.blog.csdn.net/article/details/123553700)
 - [KCNA考试 第七章：监控与探测](https://ghostwritten.blog.csdn.net/article/details/123573993)
 -  [十二个因素的应用程序](https://ghostwritten.blog.csdn.net/article/details/121496200)
 -  [KCNA（云原生入门）测试题](https://ghostwritten.blog.csdn.net/article/details/123631209)

----
