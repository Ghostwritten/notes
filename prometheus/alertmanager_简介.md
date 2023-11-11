#  alertmanager 简介
tags: alertmanager


![在这里插入图片描述](https://img-blog.csdnimg.cn/e4a0e351ffc64c74bfcc35ff70f10a90.png)


##  1. 前言
Prometheus的报警功能主要是利用Alertmanager这个组件。当Alertmanager接收到 Prometheus 端发送过来的 Alerts 时，Alertmanager 会对 Alerts 进行去重复，分组，按标签内容发送不同报警组，包括：邮件，微信，webhook。

使用prometheus进行告警分为两部分：Prometheus Server中的告警规则会向Alertmanager发送。然后，Alertmanager管理这些告警，包括进行重复数据删除，分组和路由，以及告警的静默和抑制。

##  2. 架构
![Alertmanager 告警图](https://img-blog.csdnimg.cn/8ccf20670f684e018bf9fe903c4131f6.png)

![Alertmanager 逻辑图](https://img-blog.csdnimg.cn/093be54b4e574396a6058c596fde0b16.png)
##  3. 特性
Alertmanager除了提供基本的告警通知能力以外，还主要提供了如：分组、抑制以及静默等告警特性：
![Alertmanager特性](https://img-blog.csdnimg.cn/cbde8666fcab4917bb4ff07ea5741d2c.png)
###  3.1 group（分组）
分组机制可以将详细的告警信息合并成一个通知。在某些情况下，比如由于系统宕机导致大量的告警被同时触发，在这种情况下分组机制可以将这些被触发的告警合并为一个告警通知，避免一次性接受大量的告警通知，而无法对问题进行快速定位。
例如，当集群中有数百个正在运行的服务实例，并且为每一个实例设置了告警规则。假如此时发生了网络故障，可能导致大量的服务实例无法连接到数据库，结果就会有数百个告警被发送到Alertmanager。
而作为用户，可能只希望能够在一个通知中中就能查看哪些服务实例收到影响。这时可以按照服务所在集群或者告警名称对告警进行分组，而将这些告警内聚在一起成为一个通知。
告警分组，告警时间，以及告警的接受方式可以通过Alertmanager的配置文件进行配置。

分组有三个参数：

 - `group by`：指定分组依据哪个label，可以是多个以逗号隔开。
 - `group_wait`：分组聚合时间窗口，当第一个新分组开始到发送告警的等待时间，系统会将这段时间的同组告警合并为一条。
 - `group_interval` :同一分组的告警发送间隔，如果分组1已经成功发送了，后来的告警也还属于分组1，则等待这个间隔时间后再发送。


###  3.2 Inhibition（抑制）
抑制是指当某一告警发出后，可以停止重复发送由此告警引发的其它告警的机制。
例如，当集群不可访问时触发了一次告警，通过配置Alertmanager可以忽略与该集群有关的其它所有告警。这样可以避免接收到大量与实际问题无关的告警通知。
抑制机制同样通过Alertmanager的配置文件进行设置。

###  3.3 Silences（静默）
静默提供了一个简单的机制可以快速根据标签对告警进行静默处理。如果接收到的告警符合静默的配置，Alertmanager则不会发送告警通知。
静默设置需要在Alertmanager的Werb页面上进行设置。


参考：

 - [官方 ALERTMANAGER](https://prometheus.io/docs/alerting/latest/alertmanager/)
 - [Prometheus告警简介](https://yunlzheng.gitbook.io/prometheus-book/parti-prometheus-ji-chu/alert/prometheus-alert-manager-overview)
 - [Promethus之AlertManager介绍](https://developer.aliyun.com/article/704660)

 
