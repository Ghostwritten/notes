

## 1. 概述  
随着云计算的兴起，开发、部署和设计应用程序的需求和可能性发生了显著变化。  
 
云提供商提供各种随需应变的服务，从简单(虚拟)服务器、网络、存储、数据库等等开始。 部署和管理这些服务非常方便，无论是交互式的，还是通过使用应用程序编程接口(api)。  
 
在本章中，您将学习现代应用程序架构的原则，通常被称为云本地架构。 我们将发现是什么使这些应用程序原生于云系统，以及它们与传统方法的区别。  

##  2. 学习目标  
在本章结束时，你应该能够:  
 

 - 讨论云本地架构的特征。
 - 解释自动伸缩和无服务器模式的好处。
 - 理解在云本地环境中开放标准和社区项目的重要性。

##  3. Cloud Native Architecture Fundamentals
云本地架构的核心思想是通过结合使用文化、技术和架构设计模式来优化您的软件，以实现成本效益、可靠性和更快的上市时间。  
 
“云原生”一词可以在各种定义中找到，其中一些侧重于技术，而另一些可能侧重于文化方面。  
 
云本地计算基金会定义如下:  
 
“云本地技术使组织能够在现代动态环境(如公共、私有和混合云)中构建和运行可伸缩的应用程序。 容器、服务网格、微服务、不可变基础设施和声明性api都是这种方法的例证。  
 
这些技术支持具有弹性、可管理和可观察性的松散耦合系统。 结合强大的自动化，它们允许工程师以最小的工作量频繁地、可预测地进行高影响的更改。 […]”  
 
传统或遗留应用程序通常在设计时考虑了整体方法，这意味着它们是自包含的，包括完成一项任务所需的所有功能和组件。 一个单一的应用程序通常有一个单一的代码库，并作为一个可以在服务器上运行的单一二进制文件提供。  
 
如果您考虑一个在线商店的电子商务软件，那么一个单一的应用程序将包括来自图形用户界面、列出产品、购物车、结帐、处理订单等等的所有功能。  
 
虽然以这种格式开发和部署应用程序非常容易，但在管理复杂性、跨多个团队扩展开发、快速实现更改以及在应用程序面临大量负载时有效地扩展应用程序等方面也同样困难。  
 
云本地架构可以为日益复杂的应用程序和用户日益增长的需求提供解决方案。 其基本思想是将应用程序分解为更小的部分，从而使它们更易于管理。 不是在一个应用程序中提供所有功能，而是有多个解耦的应用程序在网络中彼此通信。 如果我们坚持之前的例子，你可以为你的用户界面，你的结帐和其他一切有一个应用程序。 这些小型的、独立的、具有明确定义的功能范围的应用程序通常被称为微服务。  
 
这使得拥有多个团队成为可能，每个团队都拥有应用程序的不同功能，但也可以单独操作和扩展它们。 例如，如果有很多人试图购买产品，你就能够扩展具有大量负载的服务，比如购物车和结帐。
  ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/661dd23a202deca9f5e9a51c2845ef58.png)
Monolithic vs Microservices Architecture
云本地架构可以有很多优点，但是集成起来也很复杂，因此必须满足一些需求才能高效地工作。

##  4. 云本地架构的特性
**自动化程度高**  
为了管理您的云本地应用程序的所有移动部分，从开发到部署的每个步骤都建议自动化。 这可以通过使用现代自动化工具和持续集成/持续交付(CI/CD)管道来实现(我们将在稍后的课程中更多地讨论管道，但现在， 请知道，CI/CD管道是一个概念，它可以用于交付软件的新版本时所需要的多个步骤)，这些步骤由版本控制系统(如git)支持。 以最小的人力参与构建、测试和部署应用程序以及基础设施，允许对生产进行快速、频繁和增量的更改。 如果必须重建整个系统，可靠的自动化系统还允许更容易的灾难恢复。  

**自我疗愈**  
应用程序和基础设施不时会出现故障。 这是预料之中的，因此云本地应用程序框架和基础设施组件包括健康检查，这些检查有助于从内部监视应用程序，并在必要时自动重新启动它们。 此外，由于应用程序已经被划分了，有可能只有部分应用程序停止工作或变慢，而其他部分不会。  

**可伸缩的**  
扩展应用程序描述的是在处理更多负载的同时仍然提供愉快的用户体验的过程。 一种扩展方法是启动同一个应用程序的多个副本，并在它们之间分发负载。 基于应用程序指标(如CPU或内存)自动化此行为还可以确保服务的可用性和性能。  

**效率(成本)**  
就像在高流量情况下扩展应用程序一样，在流量较低时，缩小应用程序和云提供商的基于使用的定价模型可以节省成本。 为了优化基础设施的使用，像Kubernetes这样的编排系统可以帮助更高效、更密集地放置应用程序。  

**易于维护**  
使用Microservices可以将应用程序分解成更小的部分，使它们更具可移植性、更容易测试和跨多个团队分发。

  **缺省安全**  
云环境通常在多个客户或团队之间共享，这需要不同的安全模型。 在过去，许多系统被划分在不同的安全区域，拒绝来自不同网络或人的访问。 一旦你进入一个区域，你可以进入里面的每个系统。 像零信任计算这样的模式通过要求每个用户和进程进行身份验证来缓解这种情况。  

[十二个因素的应用程序](https://ghostwritten.blog.csdn.net/article/details/121496200)是你的云原生旅程的一个良好的基线和起点。十二个因素的应用程序是开发云原生应用程序的指南，它从简单的事情开始，比如代码库的版本控制， 支持环境的配置和更复杂的模式，比如应用程序的无状态性和并发性。  
 
虽然这些模式和技术在云中运行时提供了充分的优势，但应用于本地系统时也可以提供很多好处。 最后但并非最不重要的是，如果您将应用程序和基础设施迁移到云，它们允许更平稳的转换。  


##  5. Autoscaling
自动缩放模式描述了基于当前需求的资源动态调整。CPU和内存是决定何时在负载增加或减少时伸缩应用程序的明显指标，但也可以考虑基于时间或业务指标的其他方法来提高或降低服务的规模。
通常，当我们谈论自动伸缩时，我们谈论的是水平伸缩（[horizontal scaling](https://blog.csdn.net/xixihahalelehehe/article/details/120778634?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164811337616780264040376%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164811337616780264040376&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-120778634.nonecase&utm_term=horizontal%20scaling&spm=1018.2226.3001.4450)），它描述了生成新的计算资源的过程，这些资源可以是应用程序进程、虚拟机的新副本，或者——以一种不那么直接的方式——甚至是服务器和其他硬件的新机架。
另一方面，垂直扩展（[Vertical scaling](https://blog.csdn.net/xixihahalelehehe/article/details/120778634?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164811337616780264040376%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164811337616780264040376&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-120778634.nonecase&utm_term=horizontal%20scaling&spm=1018.2226.3001.4450)）描述底层硬件大小的变化，它只适用于裸金属的某些硬件限制，但也适用于虚拟机。通过允许虚拟机和进程消耗更多的CPU和内存，可以很容易地扩展它们，上限是由底层硬件的计算和内存容量定义的。硬件本身可以扩展，例如，通过添加更多的RAM，但只有在所有RAM插槽被占用时才可以。
为了说明垂直和水平缩放之间的区别，想象你必须携带一个你拿不起来的重物。你可以锻炼肌肉自己扛，但你的身体有力量的上限。这是垂直扩展。你也可以打电话给你的朋友，请他们帮助你，分享工作。这是水平扩展。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/93855db649bb120fd0c156d4e9a81877.png)
水平vs垂直缩放
在各种环境中配置自动伸缩需要配置实例(虚拟机或容器)的最小和最大限制，以及触发伸缩的度量。为了配置正确的伸缩，您经常需要运行许多(接近生产环境的)负载测试，并在伸缩应用程序时分析行为和负载平衡。
依赖于基于使用情况的按需定价模型的云环境为自动伸缩提供了非常有效的平台，能够在几秒钟内提供大量资源，甚至在资源暂时不需要的情况下将其扩展到零。

##  6. Serverless
与术语“serverless”所暗示的相反，服务器当然仍然需要作为应用程序的基础。云提供商建议部署应用程序非常容易，但需要您准备和配置多个资源，如**网络**、**虚拟机**、**操作系统**和**负载平衡器**，以运行一个简单的web应用程序。serverless计算的思想是将开发人员从这些复杂的任务中解放出来。简而言之，您可以只提供应用程序代码，而云提供商选择正确的环境来运行您的应用程序。
所有流行的云供应商都有一个或多个私有的无serverless运行时和一个serverless子集(也称为功能即服务([FaaS](https://www.youtube.com/watch?v=EOIja7yFScs)))的商业产品。云提供商抽象底层基础设施，这样开发人员就可以通过上传他们的代码(例如。zip文件)或提供容器映像来部署软件。
与其他云计算模型相比，serverless计算更加关注应用程序的按需供应和扩展。**自动扩展**是无服务器的一个重要核心概念，可以包括基于传入请求等事件的扩展和配置。这允许更精确的计费，可以基于所提到的事件，而不是广泛使用的基于时间的计费。
FaaS系统不是完全替代容器编制平台或更传统的虚拟机，而是经常与现有平台结合使用，或者作为现有平台的扩展使用，因为它们允许非常快速的部署，并有利于出色的测试和沙箱环境。

虽然容器映像是为FaaS或无服务器系统打包软件的一种很好的标准化方法，但构建在Kubernetes之上的Knative等系统允许扩展具有无服务器计算能力的现有平台。对于私有云和本地环境中的无服务器操作，这是一种可行的解决方案。请记住，这可以简化开发人员的工作，但会增加操作云平台的复杂性。
尽管无服务器技术有许多优势，但它最初在标准化方面遇到了困难。许多云提供商都有自己的专有产品，这使得它们很难在不同的平台之间切换。为了解决这些问题，创建了[clouddevents](https://cloudevents.io/)项目，并提供了事件数据应该如何结构化的规范。事件是伸缩无服务器工作负载或触发相应函数的基础。采用这种标准的供应商和工具越多，在多个平台上使用无服务器和事件驱动架构就越容易。
为无服务器平台编写的应用程序对云本地架构有更严格的要求，但同时可以从中获益最多。编写小型的、无状态的应用程序使它们非常适合于事件或数据流、计划任务、业务逻辑或批处理。

## 7. 开放标准
许多云本地技术严重依赖开源软件，这可以使其更容易启动，并防止厂商锁定。每个人都可以在开源项目中协作，这使得实现行业标准变得很容易，甚至可以在商业产品和云提供商产品中找到他们的方法。
一个常见的问题是如何构建和分发软件包，因为应用程序对底层操作系统和应用程序运行时有很多需求和依赖关系。为了克服这个问题，容器发展成为一种打包和运输现代软件的标准方法。虽然Docker经常被用作容器技术的同义词，但社区已经致力于开放容器倡议([open container Initiative](https://opencontainers.org/), OCI)的开放行业标准。
在Linux基金会的庇护下，Open Container Initiative提供了两个标准，它们定义了如何构建和运行容器的方式。[image-spec](https://github.com/opencontainers/image-spec)定义了如何构建和打包容器映像。而[runtime-spec](https://github.com/opencontainers/runtime-spec)则指定容器的配置、执行环境和生命周期。OCI项目最近增加的一个内容分发规范([distribution - spec](https://github.com/opencontainers/distribution-spec))为内容的分发提供了一个标准，特别是容器图像的分发。
像这样的开放标准有助于并补充Kubernetes这样的其他系统，Kubernetes实际上是用于编排容器的标准平台。以下几章将介绍一些标准:

 - [OCI规范](https://opencontainers.org/):关于如何运行、构建分发容器的映像、运行时和分发规范
 - [容器网络接口(CNI)](https://github.com/containernetworking/cni):关于如何实现容器网络的规范。
 - [容器运行时接口(CRI)](https://github.com/kubernetes/cri-api):关于如何在容器编排系统中实现容器运行时的规范。
 - [容器存储接口(CSI)](https://github.com/container-storage-interface/spec):关于如何在容器编排系统中实现存储的规范。
 - [服务网格接口(SMI)](https://smi-spec.io/):关于如何在容器编排系统中实现服务网格的Ap规范，重点关注Kubernetes。

按照这种方法，其他系统(如Prometheus或OpenTelemetry)在这个生态系统中发展壮大，并为监视和观测提供了额外的标准。

##  8. Cloud Native Roles & Site Reliability Engineering

当然，我们在过去二十年中看到的技术和文化变革导致了任务和工作描述的调整。之前的职位包括系统、网络或数据库管理员、软件开发人员或测试经理。
云计算领域的工作更难以描述，而且转换也更顺利，因为职责通常由来自不同领域、拥有不同技能的多个人员共同承担。

 - `云计算架构师(Cloud Architect)`
负责采用云技术，设计应用程序环境和基础设施，重点关注安全性、可伸缩性和部署机制。
 - `DevOps工程师`
通常被描述为开发人员和管理员的简单组合，但这样做并不公平。DevOps工程师使用工具和流程来平衡软件开发和操作。从在整个部署生命周期中编写、构建和测试软件的方法开始。
 - `安全工程师(Security Engineer)`
也许这是最容易理解的角色。尽管如此，安全工程师的角色已经发生了重大变化。云技术创造了新的攻击载体，如今，这个角色必须更具包容性，并成为团队的一个组成部分。
 - `DevSecOps工程师`
为了使安全成为现代IT环境中不可或缺的一部分，DevSecOps工程师将前两者的角色结合起来。这个角色通常用于在更传统的开发团队和安全团队之间建立桥梁。
 - `数据工程师(Data Engineer)`
数据工程师面临的挑战是收集、存储和分析正在或可能在大型系统中收集的大量数据。这包括提供和管理专门的基础设施，以及使用这些数据。
 - `开发人员(Full-Stack Developer)`
精通前端和后端开发以及基础设施的全才。
 - `站点可靠性工程师(SRE)`
具有更强定义的角色是站点可靠性工程师([SRE](https://en.wikipedia.org/wiki/Site_reliability_engineering))。SRE于2003年左右在谷歌成立，并成为许多组织的重要工作。SRE的首要目标是创建和维护可靠且可扩展的软件。为了实现这一点，软件工程方法被用来解决操作问题和自动化操作任务。
为了衡量性能和可靠性，**SREs使用了三个主要指标:**
-- **服务级别目标(SLO)**:“为您的服务的可靠性指定一个目标级别。-设定的目标，例如使服务延迟小于100ms。
-- **服务水平指示器(SLI)**:“对所提供的服务水平的某些方面仔细定义的定量度量”——例如，一个请求实际需要回答多长时间。
-- **服务水平协议(SLA)**:与用户签订的一项明确或隐含的协议，其中包括满足(或不满足)其所包含的服务水平协议的后果。其后果最容易在财务方面体现出来——回扣或罚款——但它们也可以采取其他形式。——回答了如果不满足SLOs会发生什么。
围绕这些指标，SREs可能定义一个错误预算。错误预算定义应用程序在执行操作(如停止部署到生产环境)之前可能出现的错误数量(或时间)。

##  9. 社区和治理
许多被许多人视为行业标准的开源项目都是由云本地计算基金会([CNCF](https://www.cncf.io/projects/))托管和支持的。项目根据成熟度进行分类，并在毕业前经历一个沙盒和孵化阶段。CNCF强大的社区可以支持项目的整个生命周期，从在CNCF景观中的公众可见性和分类开始，直到项目成熟生产使用。
CNCF有一个技术监督委员会(TOC)，负责定义和维护技术愿景，批准新项目，接受来自最终用户委员会的反馈，并定义在CNCF项目中应该实施的通用实践。
同时，TOC并不控制项目，但鼓励它们自治和社区拥有，并实践“最小可行治理”的原则。这些项目的指导方针包括如何维护、审查和发布项目，其用户和工作组是什么，等等。由于开源项目依赖于它们各自的社区，因此治理与传统方法有很大的不同。云本地技术的巨大自由迫使人们将规则强加于自己，并强制执行它们。

##   10. 额外的资源
了解更多关于……
原生云架构

 - [采用云原生架构，第1部分:架构演化与成熟](https://www.infoq.com/articles/cloud-native-architecture-adoption-part1/)，作者:Srini Penchikala,Marcio Esteves, Richard serter (2019)
 - [云原生架构的5个原则](https://cloud.google.com/blog/products/application-development/5-principles-for-cloud-native-architecture-what-it-is-and-how-to-master-it)——它是什么以及如何掌握它，Tom Grey (2019)
 - [什么是原生云，什么是原生云应用程序?](https://tanzu.vmware.com/cloud-native)
 - [CNCF云原生交互景观](https://landscape.cncf.io/)

良好的框架

 - [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)
 - [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
 - [Microsoft Azure Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/)

Microservices

 - [microservices是什么?](https://microservices.io/)
 - [《微服务》](https://martinfowler.com/articles/microservices.html)，詹姆斯·刘易斯和马丁·福勒著
 - [在Netflix采用微服务:架构设计的教训](https://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/)

Serverless

 - [CNCF采取措施实现无服务器计算](https://www.cncf.io/blog/2018/02/14/cncf-takes-first-step-towards-serverless-computing/)，Kristen Evans (2018)
 - [CNCF无服务器白皮书v1.0](https://github.com/cncf/wg-serverless/tree/master/whitepapers/serverless-overview) (2019)
 - [Serverless架构](https://cloud.google.com/serverless/whitepaper)

网站可靠性工程

 - [SRE Book](https://sre.google/sre-book/introduction/)，本杰明·特雷纳·斯洛斯(2017)
 - [DevOps、SRE和平台工程](https://iximiuz.com/en/posts/devops-sre-and-platform-engineering/)，Ivan
   Velicho著(2021)

---

✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>
 - [KCNA考试 第一章：cloud foundry云原生工程师考试](https://ghostwritten.blog.csdn.net/article/details/121482847)
 - [KCNA考试 第二章：Cloud Native Architecture](https://ghostwritten.blog.csdn.net/article/details/121492840)
 - [KCNA考试 第三章：容器编排](https://ghostwritten.blog.csdn.net/article/details/121527922)
 - [KCNA考试 第四章：kubernetes需要掌握的基础知识](https://ghostwritten.blog.csdn.net/article/details/123477851)
 - [KCNA考试 第五章：kubernetes实践](https://ghostwritten.blog.csdn.net/article/details/123496572)
 - [KCNA考试 第六章：持续交付](https://ghostwritten.blog.csdn.net/article/details/123553700)
 -  - [KCNA考试 第七章：监控与探测](https://ghostwritten.blog.csdn.net/article/details/123573993)
 -  [十二个因素的应用程序](https://ghostwritten.blog.csdn.net/article/details/121496200)
 - [KCNA（云原生入门）测试题](https://ghostwritten.blog.csdn.net/article/details/123631209)
----
