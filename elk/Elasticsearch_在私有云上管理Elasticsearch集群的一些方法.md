

----
## 1. 管理单个集群
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4133431c138e40cb2ef5e387c7a3a8b8.png)
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

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b649f426ea66ed4fb632ec649d753903.png)
## 3. 基于 Kubernetes 的方案
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5c45ce818c027b2eaf6394ecc9b77485.png)
● 基于容器技术，使用 Operator 模式进行编排管理
● 配置，管理监控多个集群
● 支持 Hot & Warm
● 数据快照和恢


## 4. Kubernetes CRD
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b131a3421dee9fd6bc9a4699ebb2eca.png)
## 5. 构建自己的管理系统
● 基于虚拟机的编排管理方式
○ `Puppet Infrastructure` （Puppet / Elasticsearch Puppet Module / Foreman）
○ `Workflow based Provision` & `Management`
● 基于 Kubernetes 的容器化编排管理方式
○ 基于 `Operator` 模式
○ Kubernetes - `Customer Resource Definition`

## 6. 将 Elasticsearch 部署在 Kubernetes
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/525c667c0148d36cbb4ba8012bb2e022.png)
## 7. 什么是 Kubernetes Operator 模式
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/09feeebb06c34524ecdcfd6b2eb949c0.png)
## 8. Operator SDK
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4648a0116d04810b0ff11f6a057d1ed.png)

