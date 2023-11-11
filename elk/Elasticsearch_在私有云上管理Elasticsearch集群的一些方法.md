

----
## 1. 管理单个集群
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312151145573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
集群容量不够时，需手工增加节点
● 有节点丢失时，手工修复或更换节点
○ 确保 Rack Awareness
● 集群版本升级；数据备份；滚动升级
○ 完全手动，管理成本高
○ 无法统一管理，例如整合变

## 2. ECE，帮助你管理多个 Elasticsearch 集群
ECE – Elastic Cloud Enterprise
○ https://www.elastic.co/cn/products/ece
● 通过单个控制台，管理多个集群
○ 支持不同方式的集群部署（支持各类部署） /
跨数据中心 / 部署 Anti Affinity
○ 统一监控所有集群的状态
○ 图形化操作
■ 增加删除节点
■ 升级集群 / 滚动更新 / 自动数据备份

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312151248843.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 3. 基于 Kubernetes 的方案
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312151828140.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
● 基于容器技术，使用 Operator 模式进行编排管理
● 配置，管理监控多个集群
● 支持 Hot & Warm
● 数据快照和恢


## 4. Kubernetes CRD
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312151917304.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 5. 构建自己的管理系统
● 基于虚拟机的编排管理方式
○ `Puppet Infrastructure` （Puppet / Elasticsearch Puppet Module / Foreman）
○ `Workflow based Provision` & `Management`
● 基于 Kubernetes 的容器化编排管理方式
○ 基于 `Operator` 模式
○ Kubernetes - `Customer Resource Definition`

## 6. 将 Elasticsearch 部署在 Kubernetes
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312152241614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 7. 什么是 Kubernetes Operator 模式
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031215234917.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 8. Operator SDK
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210312152445638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

