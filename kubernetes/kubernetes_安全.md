#  kubernetes Security Overview
tags: 安全




{% youtube %}
https://www.youtube.com/watch?v=oBf5lrmquYI
{% endyoutube %}




##  1. 前言
保护 [Kubernetes](https://kubernetes.io/) 似乎是一项神秘的任务。作为一个由一系列不同组件组成的高度复杂的系统，Kubernetes 不是您可以通过简单地启用安全模块或安装安全工具来保护的东西。

相反，Kubernetes 安全要求团队解决可能影响 Kubernetes 集群内各个层和服务的每种类型的安全风险。例如，团队必须了解如何保护 Kubernetes [节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)、[网络](https://kubernetes.io/zh-cn/docs/tasks/network/)、[Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/)、数据等。

此外，Kubernetes 管理员需要了解 Kubernetes 原生提供哪些工具来解决安全问题，以及他们需要将哪些类型的第三方安全工具与其集群集成以填补空白。这也是一个复杂的话题，因为尽管 Kubernetes 不是一个安全平台，但它确实提供了某些类型的原生安全工具，例如[基于角色的访问控制 (RBAC)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)。

如果您是 Kubernetes 新手，并且仍然试图了解整个事情的工作原理，更不用说如何保证它的安全，那么上述所有内容都会让人感到不知所措。但是，如果将它们分解成可消化的部分，这些概念实际上就足够简单了。为此，本文将介绍 Kubernetes 安全性的各个方面，并解释每个方面的基础知识，以及 Kubernetes 安全性在各个层级和服务级别的最佳实践。

## 2. 节点安全
节点是组成 Kubernetes 集群的服务器。在大多数情况下，节点运行某些版本的 Linux，尽管工作节点可能运行 Windows。节点可以是虚拟机或裸机服务器，但从安全角度来看，区别并不重要。

您应该采用与保护任何类型服务器相同的安全策略来保护 Kubernetes 节点。这些包括：

 - 删除无关的应用程序、库和操作系统的其他组件，以最小化您的攻击面。使用极简 Linux 发行版（例如Alpine
   Linux）供应节点是最佳实践。
 - 消除不必要的用户帐户。
 - 确保除非绝对必要，否则不会以 root 身份运行。
 - 在可用的情况下，部署操作系统强化框架，如 AppArmor 或 SELinux。
 - 收集和分析操作系统日志以检测可能的违规行为。

如果您在任何类型的环境中都有在操作系统级别保护服务器的经验，那么您可能已经知道如何处理 Kubernetes 节点安全性。在节点级别，当您处理运行 Kubernetes 的节点时，安全考虑与处理任何类型的服务器并没有什么不同。

保护主节点和保护工作节点之间也没有根本区别。主节点的安全性更为重要，因为主节点上的漏洞可能会对您的集群造成更大的损害，但在主节点上保护操作系统的过程与工作节点相同。

## 3. Kubernetes API 安全
Kubernetes API 将集群的各个部分绑定在一起。因此，它是 Kubernetes 中最重要的保护资源之一。

Kubernetes API 默认设计为安全的。它只会响应它可以正确验证和授权的请求。

也就是说，API 身份验证和授权由您配置的 [RBAC 策略](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/)管理。因此，API 仅与您的 RBAC 策略一样安全。因此，创建安全的 RBAC 策略来执行最小权限原则并在粒度基础上分配权限是确保 [Kubernetes API](https://kubernetes.io/zh-cn/docs/concepts/overview/kubernetes-api/) 安全性的基本最佳实践。

此外，您可以利用准入控制器进一步增强 API 安全性。在 API 服务器已经对请求进行身份验证和授权后，准入控制器会评估请求。通过这种方式，准入控制器为不应该被允许的请求提供了一个可选的第二层防御。通过启用和配置准入控制器，您可以强制执行与 API 请求相关的各种安全规则。此处记录了可用的规则。

最后，通过配置安全证书并要求 API 服务器在安全端口而不是本地主机上服务请求，可以在网络级别保护 API 请求。

## 4. Kubernetes 网络安全
[Kubernetes 网络安全](https://sysdig.com/learn-cloud-native/kubernetes-security/network-security/)类似于 pod 安全，因为它首先遵循您将用于保护任何网络的最佳实践。

您应该确保尽可能创建一个网络架构，将工作负载与公共 Internet 隔离，除非它们需要与之交互。您应该在网关级别部署防火墙以阻止来自违规主机的流量。您应该监控网络流量是否有违规迹象。这些都是您可以使用 Kubernetes 外部工具（例如服务网格）执行的所有步骤。

然而，Kubernetes 也提供了数量有限的原生工具来帮助保护网络资源。该工具以网络策略的形式出现。虽然网络策略本身并不是一项安全功能，但管理员可以使用它们来控制 Kubernetes 集群内的流量流动方式。

因此，您可以创建网络策略来执行诸如在网络级别隔离 pod 或过滤传入流量之类的事情。

网络策略不能替代保护 Kubernetes 之外的网络配置；相反，请将它们视为补充您构建到整个网络架构中的安全规则的附加资源。

## 5. Kubernetes Pod 安全性
在 Kubernetes 中，[Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 是一个容器或一组容器，用于运行应用程序。为了保护您的应用程序，您需要保护您的 pod。

pod 安全性的某些方面需要 Kubernetes 外部的实践。您应该在部署之前对应用程序执行安全测试，并在运行容器映像之前对其进行扫描。您应该从 pod 中收集日志并对其进行分析，以检测潜在的违规或滥用行为。

但是，Kubernetes 提供了一些原生工具，可以在 Pod 已经运行后加强 Pod 的安全性。这些包括：

 - [RBAC policies](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/rbac/)，可用于管理集群内的用户和服 务对 Pod 的访问。
 - [Security contexts](https://sysdig.com/learn-cloud-native/kubernetes-security/security-contexts/)，它定义了 pod 运行的特权级别。
 - [网络策略](https://kubernetes.io/docs/concepts/services-networking/network-policies/)，（如上所述）可用于在网络级别隔离 pod。
 - [准入控制器](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)，它可以强制执行附加规则，这些规则基本上扩展了您基于 RBAC 建立的规则。

您使用的 Pod 安全工具的类型以及您配置它们的方式将取决于您的工作负载的性质。Pod 安全性没有一种万能的方法。例如，一些 pod 可以在网络级别完全相互隔离，而另一些则需要能够通信。

但是，无论您的具体要求是什么，您都应该评估可用于保护 Kubernetes pod 的资源，并确保充分利用它们。

## 6. Kubernetes 数据安全
Kubernetes 不存储任何数据，除了存在于运行中的 pod 中的非持久性数据和存储在节点上的日志数据。通常，您的集群创建和/或访问的数据将存在于某种外部存储系统中，该系统通过存储插件与 Kubernetes 交互。

因此，为了保护与 Kubernetes 相关的数据，您应该遵循用于保护任何大型存储系统内的数据的最佳实践。尽可能加密静态数据。使用访问控制工具来限制谁可以访问数据。确保管理存储池的服务器已正确锁定。备份数据以帮助保护自己免受数据盗窃或勒索软件攻击。

至于原生存在于 Kubernetes pod 和节点中的相对少量数据，Kubernetes 没有提供任何特殊工具来保护这些数据。但是，您可以通过使用上述最佳实践保护您的 pod 和节点来保护它。

## 7. 其他 Kubernetes 安全资源
除了上述特定于组件的安全实践之外，管理员还应该了解 Kubernetes 的其他安全资源。

- [审核日志](https://sysdig.com/learn-cloud-native/kubernetes-security/kubernetes-audit-log/)

Kubernetes 可以选择详细记录集群中执行了哪些操作、谁执行了这些操作以及结果如何。使用这些审计日志，您可以全面审计您的集群，实时检测潜在的安全问题以及事后研究安全事件。

要使用审计日志，您必须首先创建一个审计策略，该策略定义 Kubernetes 将如何记录事件。Kubernetes 文档包含有关建立审计策略的完整详细信息。

此外，由于 Kubernetes 不提供帮助您大规模分析审计日志的工具，您可能希望将审计日志流式传输到外部监控或可观察性平台，以帮助您检测异常并提醒您注意违规行为。否则，您只能手动监控审计事件，这在规模上是不切实际的。

- [命名空间](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/cluster-resources/namespace-v1/)

在 Kubernetes 中，命名空间可用于将不同的工作负载相互隔离。

如果您愿意，您可以在单个命名空间中运行所有内容，但从安全角度来看，为集群中的每个团队和/或工作负载类型创建不同的命名空间是最佳实践。例如，您可能希望使用单独的命名空间将您的开发/测试环境与生产环境分开。

管理多个命名空间在一定程度上增加了 Kubernetes 的管理复杂性，因为在许多（但不是全部）情况下，您需要为每个命名空间创建不同的 RBAC 策略。但是，额外的努力是值得的，因为它可以最大限度地减少违规的潜在影响。

- 在 Kubernetes 中使用外部安全工具

尽管 Kubernetes 提供了某些类型的工具来帮助强化集群内运行的资源，但 Kubernetes 并非旨在帮助您检测或管理安全事件。

要大规模管理 Kubernetes 安全性，您很可能需要利用外部安全工具。这些工具可以执行几个重要的安全功能，包括：

 - 扫描您的 RBAC 策略、安全上下文和其他配置数据，以识别可能导致安全问题的错误配置。
 - 提供应用程序和容器映像扫描功能，您可以使用它来构建一个自动安全管道，以馈送到您的 Kubernetes 集群中。
 - 收集、汇总和分析应用程序日志和审核日志，以帮助您检测可能预示违规的异常情况。

Kubernetes 有多种外部安全工具——当然包括 [Sysdig](https://sysdig.com/)，它是专门为帮助 DevOps 团队保护 Kubernetes 和其他云原生环境的所有层而构建的。

参考：

 - [Kubernetes Security 101: Fundamentals and Best Practices](https://sysdig.com/learn-cloud-native/kubernetes-security/kubernetes-security-101/)

