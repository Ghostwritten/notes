# grafana 简介
tags: grafana


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7642cd534b8f5af960e3bc91d4019aa.png)


##  1. Grafana OSS

[Grafana 开源软件](https://grafana.com/oss/)使您能够查询、可视化、警告和探索存储在任何位置的指标、日志和跟踪。Grafana OSS 为您提供了将时间序列数据库 (TSDB) 数据转换为富有洞察力的图表和可视化的工具。

[安装Grafana](https://grafana.com/docs/grafana/latest/setup-grafana/installation/)并使用[Grafana 入门](https://grafana.com/docs/grafana/latest/getting-started/build-first-dashboard/)中的说明设置您的第一个仪表板后，您将有许多选项可供选择，具体取决于您的要求。例如，如果您想查看有关您的智能家居的天气数据和统计数据，那么您可以创建一个[播放列表](https://grafana.com/docs/grafana/latest/dashboards/create-manage-playlists/)。如果您是企业的管理员并且正在为多个团队管理 Grafana，那么您可以设置[配置](https://grafana.com/docs/grafana/latest/administration/provisioning/)和[身份验证](https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/)。


## 2. 功能
###  2.1 Explore metrics, logs, and traces
通过即席查询和动态钻取探索您的数据。拆分视图并并排比较不同的时间范围、查询和数据源。有关详细信息，请参阅探索。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e62fc3e468f7c00c575c19484b97df27.png)
###  2.2 Alerts（警报）
如果您使用 Grafana Alerting，则可以通过许多不同的[警报通知器](https://grafana.com/docs/grafana/latest/alerting/contact-points/#list-of-notifiers-supported-by-grafana)发送警报，包括 PagerDuty、SMS、电子邮件、VictorOps、OpsGenie 或 Slack。

如果您更喜欢其他一些通信渠道，警报挂钩允许您使用一些代码创建不同的通知器。为您最重要的指标直观地[定义警报规则](https://grafana.com/docs/grafana/latest/alerting/alerting-rules/)。

###  2.3 Annotations（注释）
使用来自不同数据源的丰富事件注释图表。将鼠标悬停在事件上以查看完整的事件元数据和标签。

此功能在 Grafana 中显示为图形标记，可用于在出现问题时关联数据。您可以手动创建注释——只需控制单击图表并输入一些文本——或者您可以从任何数据源获取数据。有关详细信息，请参阅[注释](https://grafana.com/docs/grafana/latest/dashboards/annotations/)。

###  2.4 Dashboard variables
[模板变量](https://grafana.com/docs/grafana/latest/dashboards/variables/)允许您创建可用于许多不同用例的仪表板。这些模板不会对值进行硬编码，因此，例如，如果您有一个生产服务器和一个测试服务器，则可以为两者使用相同的仪表板。

模板允许您深入了解您的数据，例如，从所有数据到北美数据，再到德克萨斯数据，等等。您还可以在组织内的团队之间共享这些仪表板，或者如果您为流行的数据源创建了一个出色的仪表板模板，您可以将其贡献给整个社区以进行自定义和使用。

###  2.5 配置 Grafana
如果您是 Grafana 管理员，那么您需要彻底熟悉[Grafana 配置选项](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/)和[Grafana CLI](https://grafana.com/docs/grafana/latest/cli/)。

配置包括配置文件和环境变量。您可以设置默认端口、日志记录级别、电子邮件 IP 地址、安全性等。

### 2.6 导入 dashboards and plugins
在官方库中发现数百个[仪表板](https://grafana.com/grafana/dashboards/)和插件。由于社区成员的热情和动力，每周都会增加新的。

###  2.7 Authentication（验证）
Grafana 支持不同的身份验证方法，例如 LDAP 和 OAuth，并允许您将用户映射到组织。有关详细信息，请参阅[用户身份验证概述](https://grafana.com/docs/grafana/latest/setup-grafana/configure-security/configure-authentication/)。

在 Grafana Enterprise 中，您还可以将用户映射到团队：如果您的公司有自己的身份验证系统，Grafana 允许您将内部系统中的团队映射到 Grafana 中的团队。这样，您可以自动授予人们访问为其团队指定的仪表板的权限。有关详细信息，请参阅[Grafana Enterprise](https://grafana.com/docs/grafana/latest/enterprise/)。

### 2.8  Provisioning（供应）
虽然单击、拖放以创建单个仪表板很容易，但需要许多仪表板的高级用户将希望使用脚本自动设置。您可以在 Grafana 中编写任何脚本。

例如，如果您正在启动一个新的 Kubernetes 集群，您还可以使用脚本自动启动 Grafana，该脚本将预先设置并锁定正确的服务器、IP 地址和数据源，以便用户无法更改它们。这也是控制大量仪表板的一种方式。有关详细信息，请参阅[配置](https://grafana.com/docs/grafana/latest/administration/provisioning/)。

### 2.9 Permissions（权限）
当组织拥有一个 Grafana 和多个团队时，他们通常希望能够将事物分开并共享仪表板。您可以创建一个用户团队，然后[设置文件夹和仪表板的权限](https://grafana.com/docs/grafana/latest/administration/user-management/manage-dashboard-permissions/)，如果您使用的是[Grafana Enterprise](https://grafana.com/docs/grafana/latest/enterprise/) ，则可以设置到[数据源级别](https://grafana.com/docs/grafana/latest/administration/data-source-management/#data-source-permissions)。

###  2.10 其他 Grafana Labs OSS 项目
除了 Grafana，Grafana Labs 还提供以下开源项目：

- Grafana Loki： [Grafana Loki](https://grafana.com/docs/loki/latest/) 是一个开源的组件集，可以组成一个功能齐全的日志堆栈。有关详细信息，请参阅[Grafana Loki 文档](https://grafana.com/docs/loki/latest/)。

- Grafana Tempo： Grafana Tempo 是一个开源、易于使用和大容量的分布式跟踪后端。有关详细信息，请参阅[Grafana Tempo 文档](https://grafana.com/docs/tempo/latest/?pg=oss-tempo&plcmt=hero-txt/)。

- Grafana Mimir： Grafana Mimir 是一个开源软件项目，为 Prometheus 提供可扩展的长期存储。有关 Grafana Mimir 的更多信息，请参阅[Grafana Mimir 文档](https://grafana.com/docs/mimir/latest/)。

参考：

 - [Grafana OSS](https://grafana.com/docs/grafana/latest/introduction/)

