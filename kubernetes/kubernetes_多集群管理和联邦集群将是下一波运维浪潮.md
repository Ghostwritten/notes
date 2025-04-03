![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d885c70c6f8a1c8002d20fdc768e5294.png)

## 问题
调研一下国内外K8s平台软件，哪个具有创建标准的K8s集群的功能？

##  背景

随着云原生技术在越来越多的企业和组织中的大规模落地，如何高效、可靠地管理大规模资源池以应对不断增长的业务挑战成为了当下云原生技术的关键挑战。在过去的很长一段时间内，不同厂商尝试通过定制Kubernetes原生组件的方式扩展单集群的规模，这在提高规模的同时也引入了复杂的单集群运维、不清晰的集群升级路径等问题。而多集群技术能在不侵入修改Kubernetes单集群的基础上横向扩展资源池的规模，在扩展资源池的同时降低了企业的运维管理等成本。




##  对比

| 工具名称    | 公司                                      | 开源  | 创建 k8s | 注册 k8s | 管理 k8s |
|---------|-----------------------------------------|-----|--------|--------|--------|
| VMware  | TKGm （tanzu）                            | yes | yes    | yes    | yes    |
| redhat  | openshift & acm                         | yes | yes    | yes    | yes    |
| Suse    | rancher                                 | yes | yes    | yes    | yes    |
| 青云科技    | KubeSphere                              | yes | yes    | yes    | yes    |
| 飞致云     | kubeoperate                             | yes | yes    | yes    | yes    |
| 华为      | Karmada                                 | yes | no     | yes    | yes    |
| 亚马逊     | Amazon Elastic Kubernetes Service (EKS) | NO  | yes    | yes    | yes    |
| 微软      | Azure Kubernetes Service (AKS)          | NO  | yes    | yes    | yes    |
| 谷歌      | Google Kubernetes Engine (GKE)          | NO  | yes    | yes    | yes    |
| 腾讯      | Tencent Kubernetes Engine（TKE）          | NO  | yes    | yes    | yes    |
| 阿里      | 分布式云容器平台ACK One                         | NO  | yes    | yes    | yes    |
| 华为      | Ubiquitous Cloud Native Service（UCS）    | NO  | yes    | yes    | yes    |


##  多集群和联邦集群

多集群和联邦集群是两种不同的架构模式，用于管理和操作多个 Kubernetes 集群。它们有以下区别：

架构设计：多集群（Multi-Cluster）是一种分布式架构，其中每个 Kubernetes 集群是独立的实体，具有自己的控制平面和工作节点。每个集群拥有自己的 API 服务器、调度器和控制器。而联邦集群（Federation）是一种集中式架构，它在多个 Kubernetes 集群之上提供了一个统一的控制平面，称为联邦控制平面（Federation Control Plane），用于集中管理和协调多个集群。

集群自治性：在多集群架构中，每个集群是自治的，它们可以独立管理和操作自己的资源和工作负载。集群之间的通信和资源共享需要通过定义跨集群的网络和策略来实现。而在联邦集群中，联邦控制平面具有全局视图和控制权，它可以协调多个集群之间的资源和工作负载，实现全局的调度和资源管理。

跨集群通信：在多集群架构中，集群之间的通信是通过网络连接和网络策略来实现的。集群之间可以通过网络隧道、虚拟专用网络（VPN）或其他连接方式进行通信。而在联邦集群中，联邦控制平面负责管理集群之间的通信和数据同步，并提供透明的跨集群服务发现和路由。

部署和管理复杂性：多集群架构相对较为简单，每个集群都是独立的实体，可以独立进行部署、管理和升级。但是，多集群架构需要额外的管理工作，例如跨集群配置同步和策略管理。而联邦集群架构提供了更高级别的抽象和集中化管理，减少了管理复杂性，但也需要额外的联邦控制平面来维护和操作。

总的来说，多集群适用于需要独立管理和操作多个 Kubernetes 集群的场景，而联邦集群适用于需要集中管理和协调多个集群的场景，具有更高级别的抽象和集中化控制。选择合适的架构取决于你的具体需求和环境






##  方法
本次调研以具备创建标准的 kubernetes 集群为前提，探究国内外关于kubernetes多集群管理平台的治理能力。并根据平台的发布时间、工具成熟度、迭代频率、以参与度等作为评估参考，选出适合客户企业多kubernetes集群管理的解决方案。

关于 Kubernetes 多集群操作包括如下： 

- 跨不同环境（数据中心；私有云、混合云和公共云；以及边缘）创建、更新和删除 Kubernetes 集群 
- 更新控制平面和计算节点 
- 跨混合环境管理应用程序生命周期 
- 扩展、保护和升级集群（甚至可能独立于提供商） 
- 维护和更新多个节点 
- 搜索、查找和修改任何 Kubernetes 资源 
- 在集群上实施基于角色的访问控制 (RBAC)。（例如，管理员可以访问所有集群，但开发人员可能只需要访问开发集群。） 
- 定义集群间的资源配额 
- 创建 Pod 策略
- 创建网络和治理策略
- 定义集群上的污点和容忍度 
- 扫描风险和漏洞  

##  调查分析
此次调研选择6个公司的产品作为参考，分别为：

- VMware Tanzu
- redhat acm
- suse rancher
### VMware Tanzu
[2019年 8月26日在 VMworld 2019 大会](https://news.vmware.com/releases/vmware-announces-vmware-tanzu-portfolio-to-transform-the-way-enterprises-build-run-and-manage-software-on-kubernetes)上，VMware 宣布推出新的产品 [VMware Tanzu](https://tanzu.vmware.com/)。，VMware Tanzu Mission Control 将利用 Cluster API 进行生命周期管理，利用 Velero 进行备份/恢复，利用 Sonobuoy 进行配置控制，利用 Contour 进行入口控制。VMware 现在是 Kubernetes 的前三名贡献者。 


## Red Hat Advanced Cluster Management （acm）
 Red Hat 于2020 年 7 月 30 日推出了适用于 Kubernetes 的高级集群管理。Red Hat Advanced Cluster Management for Kubernetes 作为 Red Hat OpenShift 集群的附加组件进行安装，并使用此集群作为其所有操作的中央控制器。此集群称为集线器集群，它会为用户提供一个管理平面以连接到高级集群管理。通过高级集群管理控制台导入或创建的所有其他 OpenShift 集群均由集线器集群管理，称为受管集群。

Red Hat Advanced Cluster Management （acm）生命周期如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bfa3ebe1c3bf76975ee35ffb37907059.png)


使用 Red Hat Advanced Cluster Management for Kubernetes 可以执行以下任务：

- 跨数据中心和公有云创建，导入和管理多个集群。

- 从一个控制台在多个集群上部署和管理应用程序或工作负载。

- 监控和分析不同集群资源的运行状况和状态

- 监控并强制实施多个集群的安全合规性。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c6b4dec955af020225f2012039ce38ae.png)

## Suse rancher

## KubeSphere

##  kubeoperate
##  Google Kubernetes Engine (GKE)

## 其他
不具备创建kubernetes 集群但具备管理能力的工具：

### Karmada

[Karmada](https://karmada.io/)（Kubernetes Armada）是华为开源的一个项目，于2020年11月正式发布。是一个 Kubernetes 管理系统，使您能够在多个 Kubernetes 集群和云中运行云原生应用程序，而无需更改应用程序。通过使用 Kubernetes 原生 API 并提供先进的调度功能，Karmada 实现了真正的开放式、多云 Kubernetes。Karmada 旨在为多云和混合云场景下的多集群应用程序管理提供即插即用的自动化，具有集中式多云管理、高可用性、故障恢复和流量调度等关键功能。同时，Karmada 是[Cloud Native Computing Foundation（CNCF）](https://github.com/karmada-io/karmada)的沙箱项目。


### 阿里的 ocm
