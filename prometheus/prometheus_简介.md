#  prometheus 简介
tags: prometheus




![在这里插入图片描述](https://img-blog.csdnimg.cn/fee2265a3657479cb18042f065cb5607.png)


##  1. 前言
[Prometheus](https://prometheus.io/)受启发于Google的Brogmon监控系统（相似的Kubernetes是从Google的Brog系统演变而来），从2012年开始由前Google工程师在Soundcloud以开源软件的形式进行研发，并且于2015年早期对外发布早期版本。2016年5月继Kubernetes之后成为第二个正式加入CNCF基金会的项目，同年6月正式发布1.0版本。2017年底发布了基于全新存储层的2.0版本，能更好地与容器平台、云平台配合。Prometheus作为新一代的云原生监控系统，目前已经有超过650+位贡献者参与到Prometheus的研发工作上，并且超过120+项的第三方集成。

Prometheus是一种用于记录和处理任何纯数字时间序列的监控解决方案。它收集、组织和存储指标以及唯一标识符和时间戳。它通过“抓取”指标 HTTP 端点从目标收集指标。支持的“目标”包括基础设施平台（例如[Kubernetes](https://kubernetes.io/)）、应用程序和服务（例如数据库管理系统）。Prometheus 与其配套的 [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) 服务一起是一个灵活的指标收集和警报工具。


##  2. 监控
在[《SRE: Google运维解密》](https://www.aliyundrive.com/s/nq4BtP9bHuQ)一书中指出，监控系统需要能够有效的支持白盒监控和黑盒监控。通过白盒能够了解其内部的实际运行状态，通过对监控指标的观察能够预判可能出现的问题，从而对潜在的不确定因素进行优化。而黑盒监控，常见的如HTTP探针，TCP探针等，可以在系统或者服务在发生故障时能够快速通知相关的人员进行处理。通过建立完善的监控体系，从而达到以下目的：

 - **长期趋势分析**：通过对监控样本数据的持续收集和统计，对监控指标进行长期趋势分析。例如，通过对磁盘空间增长率的判断，我们可以提前预测在未来什么时间节点上需要对资源进行扩容。
 - **对照分析**：两个版本的系统运行资源使用情况的差异如何？在不同容量情况下系统的并发和负载变化如何？通过监控能够方便的对系统进行跟踪和比较。
 - **告警**：当系统出现或者即将出现故障时，监控系统需要迅速反应并通知管理员，从而能够对问题进行快速的处理或者提前预防问题的发生，避免出现对业务的影响。
 - **故障分析与定位**：当问题发生后，需要对问题进行调查和处理。通过对不同监控监控以及历史数据的分析，能够找到并解决根源问题。
 - **数据可视化**：通过可视化仪表盘能够直接获取系统的运行状态、资源使用情况、以及服务运行状态等直观的信息。



##  3. 架构
![Prometheus 组件和架构](https://img-blog.csdnimg.cn/b41476d794e74dda87b96feac016f014.png)
Prometheus 从检测作业中直接或通过中间推送网关从短期作业中抓取指标。它在本地存储所有抓取的样本，并对这些数据运行规则，以从现有数据聚合和记录新的时间序列或生成警报。Grafana或其他 API 使用者可用于可视化收集的数据。


##  4. 组件
Prometheus 生态系统由多个组件组成，其中有许多组件是可选的：

 - `Prometheus Server` 作为服务端，用来存储时间序列数据。
 - 客户端库（`client`）用来检测应用程序代码。
 - 用于支持临时任务的推送网关（`pushgateway`）。
 - `Exporter` 用来监控 HAProxy，StatsD，Graphite 等特殊的监控目标，并向 Prometheus
   提供标准格式的监控样本数据。
 - `alartmanager` 用来处理告警。
 - 其他各种周边工具。

其中大多数组件都是用 Go 编写的，因此很容易构建和部署为静态二进制文件。

##  5. 特征
普罗米修斯的主要特点是：

 - 具有由度量名称和键/值对标识的时间序列数据的多维[数据模型](https://prometheus.io/docs/concepts/data_model/)
 - PromQL，一种 利用这种维度的[灵活查询语言](https://prometheus.io/docs/prometheus/latest/querying/basics/)
 - 不依赖分布式存储；单个服务器节点是自治的
 - 时间序列收集通过 HTTP 上的拉模型进行
 - 通过中间网关支持推送[时间序列](https://prometheus.io/docs/instrumenting/pushing/)
 - 通过服务发现或静态配置发现目标
 - 多种图形模式和仪表板支持

##  6. 指标（metrics）
用外行的话来说，度量是数字度量。时间序列意味着随着时间的推移记录变化。用户想要测量的内容因应用程序而异。对于 Web 服务器，它可能是请求时间，对于数据库，它可能是活动连接数或活动查询数等。

指标在理解为什么您的应用程序以某种方式工作方面起着重要作用。假设您正在运行一个 Web 应用程序并发现该应用程序很慢。您将需要一些信息来了解您的应用程序发生了什么。例如，当请求数量很高时，应用程序可能会变慢。如果您有请求计数指标，您可以找出原因并增加服务器数量来处理负载。

###  6.1 Counter
计数器的值将始终增加。它永远不会减少，但可以重置为零。因此，如果抓取失败，则仅意味着丢失了数据点。累积增长将在下一次读取时可用。例子：

 - 收到的 HTTP 请求总数；
 - 异常的数量。

###  6.2 Gauge
仪表是任何给定时间点的快照。它既可以增加也可以减少。如果数据获取失败，您将丢失一个样本；下一次提取可能会显示不同的值：示例磁盘空间、内存使用情况。

###  6.3  Histogram
直方图对观察结果进行采样并在可配置的桶中对其进行计数。它们用于请求持续时间或响应大小等。例如，您可以测量特定 HTTP 请求的请求持续时间。直方图将有一组桶，比如 1ms、10ms 和 25ms。Prometheus 不会存储每个请求的每个持续时间，而是存储落入特定存储桶的请求的频率。

###   6.4 Summary
与直方图样本观察类似，通常要求持续时间或响应大小。它将提供观察的总数和所有观察值的总和，允许您计算观察值的平均值。例如，在一分钟内，您有三个请求需要 2、3、4 秒。总和为 9，计数为 3。延迟为 3 秒。


##  7. 比较
### 7.1 Prometheus vs Zabbix
Zabbix 使用的是 C 和 PHP, Prometheus 使用 Golang, 整体而言 Prometheus 运行速度更快一点。
Zabbix 属于传统主机监控，主要用于物理主机，交换机，网络等监控，Prometheus 不仅适用主机监控，还适用于 Cloud, SaaS, Openstack，Container 监控。
Zabbix 在传统主机监控方面，有更丰富的插件。
Zabbix 可以在 WebGui 中配置很多事情，但是 Prometheus 需要手动修改文件配置。
### 7.2 Prometheus vs Graphite
Graphite 功能较少，它专注于两件事，存储时序数据， 可视化数据，其他功能需要安装相关插件，而 Prometheus 属于一站式，提供告警和趋势分析的常见功能，它提供更强的数据存储和查询能力。
在水平扩展方案以及数据存储周期上，Graphite 做的更好。
### 7.3 Prometheus vs InfluxDB
InfluxDB 是一个开源的时序数据库，主要用于存储数据，如果想搭建监控告警系统， 需要依赖其他系统。
InfluxDB 在存储水平扩展以及高可用方面做的更好, 毕竟核心是数据库。
### 7.4 Prometheus vs OpenTSDB
OpenTSDB 是一个分布式时序数据库，它依赖 Hadoop 和 HBase，能存储更长久数据， 如果你系统已经运行了 Hadoop 和 HBase, 它是个不错的选择。
如果想搭建监控告警系统，OpenTSDB 需要依赖其他系统。


### 7.5 Prometheus vs Sensu
Sensu 广义上讲是 Nagios 的升级版本，它解决了很多 Nagios 的问题，如果你对 Nagios 很熟悉，使用 Sensu 是个不错的选择。
Sensu 依赖 RabbitMQ 和 Redis，数据存储上扩展性更好

### 7.6 Prometheus vs Nagios
Nagios 数据不支持自定义 Labels, 不支持查询，告警也不支持去噪，分组, 没有数据存储，如果想查询历史状态，需要安装插件。
Nagios 是上世纪 90 年代的监控系统，比较适合小集群或静态系统的监控，显然 Nagios 太古老了，很多特性都没有，相比之下Prometheus 要优秀很多。

对于常用的监控系统，如[Nagios](https://www.nagios.org/)、[Zabbix](https://www.zabbix.com/)的用户而言，往往并不能很好的解决上述问题。这里以Nagios为例，如下图所示是Nagios监控系统的基本架构：
![Nagios监控系统](https://img-blog.csdnimg.cn/8bdd8a1bed7f47bead6ade361f624fd6.png)
Nagios的主要功能是监控服务和主机。Nagios软件需要安装在一台独立的服务器上运行，该服务器称为监控中心。每一台被监控的硬件主机或者服务都需要运行一个与监控中心服务器进行通信的Nagios软件后台程序，可以理解为Agent或者插件。
![Nagios主机监控页面](https://img-blog.csdnimg.cn/d077f58c09a24a739c9b14e8fffc4a7a.png)
首先对于Nagios而言，大部分的监控能力都是围绕系统的一些边缘性的问题，主要针对系统服务和资源的状态以及应用程序的可用性。 例如：Nagios通过`check_disk`插件可以用于检查磁盘空间，`check_load`用于检查CPU负载等。这些插件会返回4种Nagios可识别的状态，0(OK)表示正常，1(WARNING)表示警告，2(CRITTCAL)表示错误，3(UNKNOWN)表示未知错误，并通过Web UI显示出来。

对于Nagios这类系统而言，其核心是采用了测试和告警(check&alert)的监控系统模型。 对于基于这类模型的监控系统而言往往存在以下问题：

 - **与业务脱离的监控**：监控系统获取到的监控指标与业务本身也是一种分离的关系。好比客户可能关注的是服务的可用性、服务的SLA等级，而监控系统却只能根据系统负载去产生告警；
 - **运维管理难度大**：Nagios这一类监控系统本身运维管理难度就比较大，需要有专业的人员进行安装，配置和管理，而且过程并不简单；
 - **可扩展性低**： 监控系统自身难以扩展，以适应监控规模的变化；
 - **问题定位难度大**：当问题产生之后（比如主机负载异常增加）对于用户而言，他们看到的依然是一个黑盒，他们无法了解主机上服务真正的运行情况，因此当故障发生后，这些告警信息并不能有效的支持用户对于故障根源问题的分析和定位。


##  8. 优势
Prometheus是一个开源的完整监控解决方案，其对传统监控系统的测试和告警模型进行了彻底的颠覆，形成了基于中央化的规则计算、统一分析和告警的新模型。 相比于传统监控系统Prometheus具有以下优点：
### 8.1 易于管理
Prometheus核心部分只有一个单独的二进制文件，不存在任何的第三方依赖(数据库，缓存等等)。唯一需要的就是本地磁盘，因此不会有潜在级联故障的风险。
Prometheus基于`Pull`模型的架构方式，可以在任何地方（本地电脑，开发环境，测试环境）搭建我们的监控系统。对于一些复杂的情况，还可以使用Prometheus服务发现(Service Discovery)的能力动态管理监控目标。


###  8.2 强大的数据模型
所有采集的监控数据均以指标(metric)的形式保存在内置的时间序列数据库当中(TSDB)。所有的样本除了基本的指标名称以外，还包含一组用于描述该样本特征的标签。
如下所示：

```bash
http_request_status{code='200',content_path='/api/path', environment='produment'} => [value1@timestamp1,value2@timestamp2...]

http_request_status{code='200',content_path='/api/path2', environment='produment'} => [value1@timestamp1,value2@timestamp2...]
```
每一条时间序列由指标名称(Metrics Name)以及一组标签(Labels)唯一标识。每条时间序列按照时间的先后顺序存储一系列的样本值。
表示维度的标签可能来源于你的监控对象的状态，比如`code=404`或者`content_path=/api/path`。也可能来源于的你的环境定义，比如`environment=produment`。基于这些`Labels`我们可以方便地对监控数据进行聚合，过滤，裁剪。

###  8.3 强大的查询语言PromQL
Prometheus 内置了一个强大的数据查询语言PromQL。PromQL 允许对收集的时间序列数据进行切片和切块，以生成临时图形、表格和警报。同时PromQL也被应用于数据可视化(如Grafana)以及告警当中。
通过PromQL可以轻松回答类似于以下问题：

 - 在过去一段时间中95%应用延迟时间的分布范围？
 - 预测在4小时后，磁盘空间占用大致会是什么情况？
 - CPU占用率前5位的服务有哪些？(过滤)

###  8.4 高效
对于监控系统而言，大量的监控任务必然导致有大量的数据产生。而Prometheus可以高效地处理这些数据，对于单一Prometheus Server实例而言它可以处理：

 - 数以百万的监控指标;
 - 每秒处理数十万的数据点。

###  8.5 可扩展
Prometheus 是如此简单，因此你可以在每个数据中心、每个团队运行独立的`Prometheus Sevrer`。Prometheus对于联邦集群的支持，可以让多个Prometheus实例产生一个逻辑集群，当单实例Prometheus Server处理的任务量过大时，通过使用功能分区(sharding)+联邦集群(federation)可以对其进行扩展。

###  8.6 易于集成
使用Prometheus可以快速搭建监控服务，并且可以非常方便地在应用程序中进行集成。目前支持： Java， JMX， Python， Go，Ruby， .Net， Node.js等等语言的客户端SDK，基于这些SDK可以快速让应用程序纳入到Prometheus的监控当中，或者开发自己的监控数据收集程序。同时这些客户端收集的监控数据，不仅仅支持Prometheus，还能支持Graphite这些其他的监控工具。
同时Prometheus还支持与其他的监控系统进行集成：Graphite， Statsd， Collected， Scollector， muini， Nagios等。
Prometheus社区还提供了大量第三方实现的监控数据采集支持：JMX， CloudWatch， EC2， MySQL， PostgresSQL， Haskell， Bash， SNMP， Consul， Haproxy， Mesos， Bind， CouchDB， Django， Memcached， RabbitMQ， Redis， RethinkDB， Rsyslog等等。


###  8.7 可视化
`Prometheus Server`中自带了一个`Prometheus UI`，通过这个UI可以方便地直接对数据进行查询，并且支持直接以图形化的形式展示数据。同时Prometheus还提供了一个独立的基于Ruby On Rails的Dashboard解决方案Promdash。最新的Grafana可视化工具也已经提供了完整的Prometheus支持，基于Grafana可以创建更加精美的监控图标。基于Prometheus提供的API还可以实现自己的监控可视化UI。


### 8.8 高效存储
Prometheus 以高效的自定义格式将时间序列存储在内存和本地磁盘上。扩展是通过功能分片和联合来实现的。

### 8.9 操作简单
每台服务器的可靠性都是独立的，仅依赖于本地存储。用 Go 编写，所有二进制文件都是静态链接的，易于部署。

### 8.10 精确警报
警报基于 Prometheus 灵活的 PromQL 定义并维护维度信息。警报管理器处理通知和静音。

参考：

 - [官方 prometheus](https://prometheus.io/docs/introduction/overview/)
 - [prometheus book](https://yunlzheng.gitbook.io/prometheus-book/)
 - [An introduction to monitoring with Prometheus](https://opensource.com/article/19/11/introduction-monitoring-prometheus)
 - [songjiayang.gitbooks.io prometheus](https://songjiayang.gitbooks.io/prometheus/content/)


