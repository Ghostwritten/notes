#  alertmanger 安装
tags: alertmanager



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dbd9153d8fc6c30686a816108aa6652f.png)


##  1. 简介

`Alertmanager`和`Prometheus Server`一样均采用Golang实现，并且没有第三方依赖。一般来说我们可以通过以下几种方式来部署Alertmanager：二进制包、容器以及源码方式安装。

##  2. 安装

###  2.1 二进制包部署 AlertManager

####  2.1.1 下载
Alertmanager最新版本的下载地址可以从Prometheus官方网站[https://prometheus.io/download/](https://prometheus.io/download/)获取。

```bash
export VERSION=0.24.0
curl -LO https://github.com/prometheus/alertmanager/releases/download/v$VERSION/alertmanager-$VERSION.darwin-amd64.tar.gz
tar xvf alertmanager-$VERSION.darwin-amd64.tar.gz
```
#### 2.1.2 创建 alertmanager 配置文件
Alertmanager解压后会包含一个默认的`alertmanager.yml`配置文件，内容如下所示：

```bash
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```
Alertmanager的配置主要包含两个部分：路由`(route`)以及接收器(`receivers`)。所有的告警信息都会从配置中的顶级路由(route)进入路由树，根据路由规则将告警信息发送给相应的接收器。

在Alertmanager中可以定义一组接收器，比如可以按照角色(比如系统运维，数据库管理员)来划分多个接收器。接收器可以关联邮件，Slack以及其它方式接收告警信息。
当前配置文件中定义了一个默认的接收者`default-receiver`由于这里没有设置接收方式，目前只相当于一个占位符。

在配置文件中使用route定义了顶级的路由，路由是一个基于标签匹配规则的树状结构。所有的告警信息从顶级路由开始，根据标签匹配规则进入到不同的子路由，并且根据子路由设置的接收器发送告警。目前配置文件中只设置了一个顶级路由route并且定义的接收器为default-receiver。因此，所有的告警都会发送给default-receiver。

#### 2.1.3 启动

Alermanager会将数据保存到本地中，默认的存储路径为`data/`。因此，在启动Alertmanager之前需要创建相应的目录：


```bash
./alertmanager
```

用户也在启动Alertmanager时使用参数修改相关配置。`--config.file`用于指定alertmanager配置文件路径，`--storage.path`用于指定数据存储路径。

#### 2.1.4 查看状态
Alertmanager启动后可以通过9093端口访问，`http://192.168.211.50:9093`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b277610ee5e8b1c397253725b20cddeb.png)




###  2.2 docker 安装
```bash
$ vim /root/alertmanager/config.yml
global:
  # resolve_timeout：解析超时时间
  resolve_timeout: 5m
  # smtp_smarthost: 使用email打开服务配置
  smtp_smarthost: 'smtp.126.com:465'
  # smtp_from：指定通知报警的邮箱
  smtp_from: 'xiangsikai@126.com'
  # smtp_auth_username：邮箱用户名
  smtp_auth_username: 'xiangsikai@126.com'
  # smtp_auth_password：授权密码
  smtp_auth_password: 'xsk123'
  # smtp_require_tls：是否启用tls
  smtp_require_tls: false

# route标记：告警如何发送分配
route:
  # group_by：采用哪个标签作为分组的依据
  group_by: ['alertname']
  # group_wait：分组等待的时间
  group_wait: 10s
  # group_interval：上下两组发送告警的间隔时间
  group_interval: 10s
  # repeat_interval：重复发送告警时间。默认1h
  repeat_interval: 1m
  # receiver 定义谁来通知报警
  receiver: 'mail'

# receiver标记：告警接受者 
receivers:
# name：报警来源自定义名称
- name: 'mail'
  # email_configs：通过邮箱发送报警
  email_configs:
    # to：指定接收端email
    - to: 'xiangsikai@126.com'

# inhibit_rules标记：降低告警收敛，减少报警，发送关键报警
#inhibit_rules:
#  - source_match:
#      severity: 'critical'
#    target_match:
#      severity: 'warning'
#    equal: ['alertname', 'dev', 'instance']
```



svEdlmtIA-6joL79FCaL7ZYO27lYt_D6DbDgm6P_128

运行容器
```bash

$ docker run -d -p 9093:9093 --name alertmanager docker.io/prom/alertmanager:latest
$ docker run -d -p 9093:9093 -v /root/alertmanager/config.yml:/etc/alertmanager/config.yml --name alertmanager docker.io/prom/alertmanager:latest #将配置挂载出来
```

###  2.3  关联 Prometheus 与 Alertmanager
在Prometheus的架构中被划分成两个独立的部分。Prometheus负责产生告警，而Alertmanager负责告警产生后的后续处理。因此Alertmanager部署完成后，需要在Prometheus中设置Alertmanager相关的信息。
编辑Prometheus配置文件`prometheus.yml`,并添加以下内容

```bash
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```
重启Prometheus服务，成功后，可以从`http://192.168.211.50:9090/config`查看alerting配置是否生效。
此时，再次尝试手动拉高系统CPU使用率：

```bash
cat /dev/zero>/dev/null
```

等待Prometheus告警进行触发状态：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1aefd4952bc5360aa65c8e036aaf909c.png)
查看Alertmanager UI此时可以看到Alertmanager接收到的告警信息。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/351b76efa4d1a894b44077ce3e2d4981.png)



## 2. 配置文件说明
Alertmanager主要负责对Prometheus产生的告警进行统一处理，因此在Alertmanager配置中一般会包含以下几个主要部分：
全局配置（global）：用于定义一些全局的公共参数，如全局的SMTP配置，Slack配置等内容；

 - 模板（`templates`）：用于定义告警通知时的模板，如HTML模板，邮件模板等；
 - 告警路由（`route`）：根据标签匹配，确定当前告警应该如何处理；
 - 接收人（`receivers`）：接收人是一个抽象的概念，它可以是一个邮箱也可以是微信，
 - Slack或者Webhook等，接收人一般配合告警路由使用；
 - 抑制规则（`inhibit_rules`）：合理设置抑制规则可以减少垃圾告警的产生

global：

 - smtp_smarthost、smtp_from、smtp_auth_username、smtp_auth_password用于设置
 - smtp邮件的地址及用户信息
 - hipchat_auth_token与安全性认证有关
 - **resolve_timeout**，该参数定义了当Alertmanager持续多长时间未接收到告警后标记告警状态为resolved（已解决）。该参数的定义可能会影响到告警恢复通知的接收时间，读者可根据自己的实际场景进行定义，其默认值为5分钟。

templates：

 - 指定告警信息展示的模版

route：可存在多个Route，一个Route有多个Group，一个Group有多个Alert

 - `group_by`：指定所指定的维度对告警进行分组
 - `group_wait`:指定每组告警发送等待的时间，根据`resolve_timeout`判断Alert是否解决，然后发送通知
 - `group_interval`:指定告警调度的时间间隔，判断Alert是否解决，当上次发送通知到现在的间隔大于repeat_interval或者Group有更新时会发送通知
 - repeat_interval:在连续告警触发的情况下，重复发送告警的时间间隔

routes

 - match_re:定义告警接收者的匹配方式
 - service:定义匹配的方式，纬度service值以foo1或foo2或baz开始/结束时表示匹配成功
 - receiver：定义了匹配成功的的情况下的接受者

receiver

 - 指定告警默认的接受者

receivers

 - 定义了具体的接收者，也就是告警具体的方式方式

inhibit_rules

 - 定义告警的抑制条件，过滤不必要的告警

## 3. 配置示例：
### 3.1 receivers接收
#### 3.1.1 微信接收
```bash
receivers:
- name: 'wechat'
  wechat_configs:
  - send_resolved: true
    api_secret: 'svEdlmtIA-6joL79FCaL7ZYO27lYt_D6DbDgm6P_128'
    corp_id: 'ww906049af7aae6271'
    api_url: "https://qyapi.weixin.qq.com/cgi-bin/"
    to_party: "1"
    to_user: '@all'
    agent_id: "1000002"
```

#### 3.1.2 邮箱接收

```bash
templates:
  [ - <filepath> ... ]
receivers:
- name: 'mengyuan'
  email_configs:
  - to: 'xxx@xxx.com'
    html: '{{ template "email.mengyuan.html" . }}'
    headers: { Subject: "[WARN] 报警邮件test" }
```

### 3.2 inhibit_rules抑制
当触发`'物理节点夯机或宕机'`告警的时候，`'监控客户端状态异常告警'`会被抑制。
```bash
inhibit_rules:
  - source_match:
      alertname: '物理节点宕机'
    target_match:
      alertname: '监控客户端状态异常告警'
    equal: ['ip']

  - source_match:
      alertname: '物理节点夯机或宕机'
    target_match:
      alertname: '服务组件端口状态异常'
    equal: ['ip']
```
**alertname**是从何而来？
来自prometheus配置告警略文件`blackbox.rules.yaml`

```bash
  - alert: 物理节点夯机或宕机
    expr: probe_success{group=~"test"} == 0
    for: 2m
    labels:
      severity: "警告"
    annotations:
      info: |
        {{ with query "time()+8*3600" }}{{ . | first | value | humanizeTimestamp | reReplaceAll "(.+)\\.(.*)" "$1" }} {{ end }} {{$labels.group}}的{{$labels.ip}}服务器夯机或宕机
      summary: "请联系管理员登陆主机进行检测，判断是人为操作原因，还是主机故障。"
```



## 4 告警模板
### 4.1 微信模板
test1.tmpl
```bash
{{ define "wechat.default.message" }}
{{- if gt (len .Alerts.Firing) 0 -}}
{{- range $index, $alert := .Alerts -}}
{{- if eq $index 0 -}}
告警类型: {{ $alert.Labels.alertname }}
告警级别: {{ $alert.Labels.severity }}
=====================
{{- end }}
===告警详情===
告警详情: {{ $alert.Annotations.message }}
故障时间: {{ $alert.StartsAt.Format "2006-01-02 15:04:05" }}
===参考信息===
{{ if gt (len $alert.Labels.instance) 0 -}}故障实例ip: {{ $alert.Labels.instance }};{{- end -}}
{{- if gt (len $alert.Labels.namespace) 0 -}}故障实例所在namespace: {{ $alert.Labels.namespace }};{{- end -}}
{{- if gt (len $alert.Labels.node) 0 -}}故障物理机ip: {{ $alert.Labels.node }};{{- end -}}
{{- if gt (len $alert.Labels.pod_name) 0 -}}故障pod名称: {{ $alert.Labels.pod_name }}{{- end }}
=====================
{{- end }}
{{- end }}
{{- if gt (len .Alerts.Resolved) 0 -}}
{{- range $index, $alert := .Alerts -}}
{{- if eq $index 0 -}}
告警类型: {{ $alert.Labels.alertname }}
告警级别: {{ $alert.Labels.severity }}
=====================
{{- end }}
===告警详情===
告警详情: {{ $alert.Annotations.message }}
故障时间: {{ $alert.StartsAt.Format "2006-01-02 15:04:05" }}
恢复时间: {{ $alert.EndsAt.Format "2006-01-02 15:04:05" }}
===参考信息===
{{ if gt (len $alert.Labels.instance) 0 -}}故障实例ip: {{ $alert.Labels.instance }};{{- end -}}
{{- if gt (len $alert.Labels.namespace) 0 -}}故障实例所在namespace: {{ $alert.Labels.namespace }};{{- end -}}
{{- if gt (len $alert.Labels.node) 0 -}}故障物理机ip: {{ $alert.Labels.node }};{{- end -}}
{{- if gt (len $alert.Labels.pod_name) 0 -}}故障pod名称: {{ $alert.Labels.pod_name }};{{- end }}
=====================
{{- end }}
{{- end }}
{{- end }}
```
test2.tmpl
```bash
{{ define "wechat.default.message" }}
{{ range .Alerts }}
告警级别：{{ .Labels.severity }}
告警类型：{{ .Labels.alertname }}
故障主机: {{ .Labels.instance }}
告警主题: {{ .Annotations.summary }}
告警详情: {{ .Annotations.description }}
触发时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }}
{{ end }}
{{ end }}
```

### 4.2 邮箱模板
第一个
```bash
模板文件email.tmpl
{{ define "email.mengyuan.html" }}
<table>
    <tr><td>报警名</td><td>开始时间</td></tr>
    {{ range $i, $alert := .Alerts }}
        <tr><td>{{ index $alert.Labels "alertname" }}</td><td>{{ $alert.StartsAt }}</td></tr>
    {{ end }}
</table>
{{ end }}
```
第二个

```bash
{{ define "test.html" }}
<table border="1">
        <tr>
                <td>报警项</td>
                <td>实例</td>
                <td>报警阀值</td>
                <td>开始时间</td>
        </tr>
        {{ range $i, $alert := .Alerts }}
                <tr>
                        <td>{{ index $alert.Labels "alertname" }}</td>
                        <td>{{ index $alert.Labels "instance" }}</td>
                        <td>{{ index $alert.Annotations "value" }}</td>
                        <td>{{ $alert.StartsAt }}</td>
                </tr>
        {{ end }}
</table>
{{ end }}
```

## 5. alertmanger API

```bash
/api/v1/alerts
/api/v1/status
/api/v1/silences
/silence/{silenceID}  get delete
/api/v2/receivers
 /alerts/groups
```

```bash
#查询已有的告警，只要客户仍然处于活动状态（通常约为30秒到3分钟），客户就会不断重新发送警报
curl http://180.76.154.185:9093/api/v1/alerts
[
  {
    "labels": {
      "<labelname>": "<labelvalue>",
      ...
    },
    "annotations": {
      "<labelname>": "<labelvalue>",
    },
    "startsAt": "<rfc3339>",
    "endsAt": "<rfc3339>"  #endsAt只有在警报的结束时间已知时才被设置。否则，它将被设置为自上次收到警报以来的可配置超时时间。
    "generatorURL": "<generator_url>" #该generatorURL字段是唯一的后向链接，用于标识客户端中此警报的原因实体。
  },
  ...
]

#生成一个告警
$ curl -H "Content-Type: application/json" -d ‘[{"labels":{"alertname":"TestAlert1"}}]’ http://180.76.154.185:9093/api/v1/alerts

#抑制一个告警
$ curl -H "Content-Type: application/json" -X POST -d '{"comment": "test1","createdBy": "test1","endsAt": "2019-02-20T18:00:59.46418637Z","matchers": [{"isRegex": false,"name": "severity","value": "critical"},{"isRegex": false,"name": "job","value": "prometheus"},{"isRegex": false,"name": "instance","value": "localhost:9090"},{"isRegex": false,"name": "alertname","value": "InstanceDown"}]}' http://alerts.example.org:9093/api/v1/silences

删除silences
$ curl -X DELETE http://alerts.example.org:9093/api/v1/silence/<silenceId>
```

## 6. amtool 工具
命令行可以对告警进行抑制、查询等操作。

```bash
amtool check-config /monitor/alertmanager/config/alertmanager/alertmanager.yml
#查看当前已触发的报警
amtool alert --alertmanager.url="http://127.0.0.1:9093"

#查看所有沉默
amtool silence query --alertmanager.url="http://127.0.0.1:9093"

#查看匹配的沉默
amtool silence query alertname=PodMemory  --alertmanager.url="http://127.0.0.1:9093"
#沉默添加，如果comment_required设置为true，不写comment的话会提交不上
#alertname就是Prometheus那里设置的alert
#--start 生效时间
#--end   结束时间
amtool silence add alertname="PodMemory" container_name="elasticsearch" --start="2019-01-02T15:04:05+08:00" --end="2019-01-02T15:05:05+08:00" --comment="test"  -a "wuchen" --alertmanager.url="http://127.0.0.1:9093"

#更新
amtool silence update --end="2019-02-02T15:05:05+08:00"  6cac2b37-4ce2-4af5-8305-9f47850d72a4  --alertmanager.url="http://127.0.0.1:9093"


#docker官方镜像，默认数据源保存在/alertmanager，所以要挂载数据盘，不然容器重启一切都白做

#因为查看沉默列表的时候，生效时间输出的不完整，所以可以
amtool silence query -o json --alertmanager.url="http://127.0.0.1:9093"
#还可以导出json数据
amtool silence query -o json PodMemory > 1.json --alertmanager.url="http://127.0.0.1:9093"

#沉默解除
amtool silence expire 8188b168-1332-4398-83a5-a9df4263c60d --alertmanager.url="http://127.0.0.1:9093"

#解除所有沉默
amtool silence expire $(amtool silence query -q --alertmanager.url="http://127.0.0.1:9093") --alertmanager.url="http://127.0.0.1:9093"
```
参考：

 - [Prometheus book 告警处理](https://yunlzheng.gitbook.io/prometheus-book/parti-prometheus-ji-chu/alert)
 - [prometheus+grafana+alertmanager 安装配置文档](https://blog.51cto.com/mageedu/2568334)
 - [Alertmanager 安装与配置](https://blog.csdn.net/qiancool/article/details/122320582)

