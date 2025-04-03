
[https://www.cncf.io/online-programs/kubernetes-runtime-security-with-falco-and-sysdig/](https://www.cncf.io/online-programs/kubernetes-runtime-security-with-falco-and-sysdig/)

----

## sysdig agent
Sysdig 代理是一个轻量级的主机组件，它处理系统调用、创建捕获文件并执行审计和合规性。它是一个支持对云原生环境进行监控、保护和故障排除的平台。

Sysdig 代理易于部署和升级，并且开箱即用，它们将收集和报告各种预定义的指标。
Sysdig还执行以下操作：

 - 指标处理、分析、聚合和传输
 - 通过与 Sysdig 收集器的双向通信进行策略监控和警报通知
 - 与第三方软件集成以整合客户生态系统数据
 - 完全融入容器化和编排环境

数据处理流程：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/113529a94ef98eeb2c3476c6d22dc959.png)

 1. 以下代理组件收集相关数据：系统调用、统计数据、JMX、Promscrape
 2. 存储指标以供分析。
 3. 每秒发送一次指标以进行分析和聚合。
 4. 构建 10 秒汇总数据。
 5. 每 10 秒发送一次数据进行序列化
 6. 将序列化的指标发送到 Sysdig 收集器

[适合各种linux安装](https://docs.sysdig.com/en/docs/installation/sysdig-agent/agent-installation/host-requirements-for-agent-installation/#linux-distributions-and-kernels)
[适合各种平台编排](https://docs.sysdig.com/en/docs/installation/sysdig-agent/agent-installation/host-requirements-for-agent-installation/#orchestration-platforms)
新版本是v12.0.0

