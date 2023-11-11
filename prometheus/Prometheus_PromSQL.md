# Prometheus PromSQL
tags: prometheus
<!-- catalog: ~config~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/6fe0e6735eda4463997ab871cb423b07.png)




##  1. 简介
通过PromQL用户可以非常方便地对监控样本数据进行统计分析，PromQL支持常见的运算操作符，同时PromQL中还提供了大量的内置函数可以实现对数据的高级处理。当然在学习PromQL之前，用户还需要了解Prometheus的样本数据模型。PromQL作为Prometheus的核心能力除了实现数据的对外查询和展现，同时告警监控也是依赖PromQL实现的。

## 2. 目标
PromQL 的学习你将能够有效地构建、分享和理解 PromQL 查询，可以帮助我们从容应对报警规则、仪表盘可视化等需求，还能够避免一些在使用 PromQL 表达式的时候遇到的一些陷进。

## 3. 架构
![在这里插入图片描述](https://img-blog.csdnimg.cn/fd4cba17683f4a1da97b4c2dbf31511c.png)
当 `Prometheus` 从系统和服务收集指标数据时，它会把数据存储在内置的时序数据库（TSDB）中，要对收集到的数据进行任何处理，我们都可以使用 PromQL 从 TSDB 中读取数据，同时可以对所选的数据执行过滤、聚合以及其他转换操作。

PromQL 的执行可以通过两种方式来触发：

 - 在 Prometheus 服务器中，记录规则和警报规则会定期运行，并执行查询操作来计算规则结果（例如触发报警）。该执行在 Prometheus 服务内部进行，并在配置规则时自动发生。
 - 外部用户和 UI 界面可以使用 Prometheus 服务提供的 HTTP API 来执行 PromQL 查询。这就是仪表盘软件（例如 `Grafana`、`PromLens` 以及 Prometheus 内置 Web UI）访问 PromQL 的方式。

## 4. 场景
PromQL 可以用于许多监控场景，下面简单介绍几个相关案例。

### 4.1 临时查询

我们可以用 PromQL 来对收集的数据进行实时查询，这有助于我们去调试和诊断遇到的一些问题，我们一般也是直接使用内置的表达式查询界面来执行这类查询：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a0c9797ff8a341bb84ab4a096faa4a27.png)
### 4.2 仪表盘

同样我们也可以基于 `PromQL` 查询来创建可视化的图形、表格等面板，当然一般我们都会使用 Grafana：
![在这里插入图片描述](https://img-blog.csdnimg.cn/fcd6ae52c5b847b69bc59e1d88342813.png)
Grafana 原生支持 Prometheus 作为数据源，并内置支持了 PromQL 表达式的查询。

### 4.3 报警

Prometheus 可以直接使用基于 PromQL 对收集的数据进行的查询结果来生成报警，一个完整的报警规则如下所示：

```bash
groups:
  - name: demo-service-alerts
    rules:
      - alert: Many5xxErrors
        expr: |
          (
            sum by(path, instance, job) (
              rate(demo_api_request_duration_seconds_count{status=~"5..",job="demo"}[1m])
            )
          /
            sum by(path, instance, job) (
              rate(demo_api_request_duration_seconds_count{job="demo"}[1m])
            ) * 100 > 0.5
          )
        for: 30s
        labels:
          severity: critical
        annotations:
          title: "{{$labels.instance}} high 5xx rate on {{$labels.path}}"
          description: "The 5xx error rate for path {{$labels.path}} on {{$labels.instance}} is {{$value}}%."
```
除了构成报警规则核心的 PromQL 表达式（上面 YAML 文件中的 expr 属性），报警规则还包含其他的一些元数据字段。

然后，Prometheus 可以通过一个名为 Alertmanager 的组件来发送报警通知，可以配置一些接收器来接收这些报警，比如用钉钉来接收报警：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2da94635d339485b9f235e04ce0591d2.png)
### 4.4 自动化

此外我们还可以构建自动化流程，针对 PromQL 执行的查询结果来做出决策，比如 Kubernetes 中基于自定义指标的 HPA。

## 5. 数据模型
在开始学习 PromQL 的知识之前，我们先重新来熟悉下 Prometheus 的数据模型
![在这里插入图片描述](https://img-blog.csdnimg.cn/623cb1de04be484daa8a58186e1c35cb.png)
Prometheus 会将所有采集到的样本数据以时间序列的方式保存在**内存数据库**中，并且定时保存到硬盘上。时间序列是按照时间戳和值的序列顺序存放的，我们称之为**向量**(vector)，每条时间序列通过指标名称(metrics name)和一组标签集(labelset)命名。如下所示，可以将时间序列理解为一个以时间为 X 轴的数字矩阵：

```bash
  ^
  │   . . . . . . . . . . . . . . . . .   . .   node_cpu_seconds_total{cpu="cpu0",mode="idle"}
  │     . . . . . . . . . . . . . . . . . . .   node_cpu_seconds_total{cpu="cpu0",mode="system"}
  │     . . . . . . . . . .   . . . . . . . .   node_load1{}
  │     . . . . . . . . . . . . . . . .   . .
  v
    <------------------ 时间 ---------------->

```
在时间序列中的每一个点称为一个样本（sample），样本由以下三部分组成：

 - **指标(metric)**：指标名和描述当前样本特征的标签集合
 - **时间戳(timestamp)**：一个精确到毫秒的时间戳
 - **样本值(value)**： 一个 float64 的浮点型数据表示当前样本的值

如下所示：


```bash
<--------------- metric ---------------------><-timestamp -><-value->
http_request_total{status="200", method="GET"}@1434417560938 => 94355
http_request_total{status="200", method="GET"}@1434417561287 => 94334

http_request_total{status="404", method="GET"}@1434417560938 => 38473
http_request_total{status="404", method="GET"}@1434417561287 => 38544

http_request_total{status="200", method="POST"}@1434417560938 => 4748
http_request_total{status="200", method="POST"}@1434417561287 => 4785
```

在形式上，所有的指标都通过如下格式表示：

```bash
<metric name>{<label name> = <label value>, ...}
```

 - **指标的名称**可以反映被监控样本的含义（比如，`http*request_total` 表示当前系统接收到的 HTTP 请求总量）。指标名称只能由 ASCII 字符、数字、下划线以及冒号组成并必须符合正则表达式`[a-zA-Z*:]a-zA-Z0-9\*:]\*`。
 - **标签(label)**反映了当前样本的特征维度，通过这些维度 Prometheus 可以对样本数据进行过滤，聚合等。标签的名称只能由 ASCII 字符、数字以及下划线组成并满足正则表达式 `[a-zA-Z_][a-zA-Z0-9_]*`。

> 注意：在 TSDB 内部，指标名称也只是一个特殊的标签，标签名为 `__name__`，由于这个标签在 PromQL 中随时都会使用，所以在使用 PromQL 查询的时候就被单独写在了标签列表前面了。另外像 `method=""` 这样的空标签在 Prometheus 种相当于一个不存在的标签，在 Prometheus 代码里面是明确地剥离了空标签的，并不会存储它们。

每个不同的 `metric_name`和 `label` 组合都称为时间序列，在 Prometheus 的表达式语言中，表达式或子表达式包括以下四种类型之一：

 - **瞬时向量（Instant vector）**：一组时间序列，每个时间序列包含单个样本，它们共享相同的时间戳。也就是说，表达式的返回值中只会包含该时间序列中的最新的一个样本值。而相应的这样的表达式称之为瞬时向量表达式。
 - **区间向量（Range vector）**：一组时间序列，每个时间序列包含一段时间范围内的样本数据，这些是通过将时间选择器附加到方括号中的瞬时向量（例如[5m]5 分钟）而生成的。
 - **标量（Scalar）**：一个简单的数字浮点值。
 - **字符串（String）**：一个简单的字符串值。

所有这些指标都是 Prometheus 定期从 metrics 接口那里采集过来的。采集的间隔时间的设置由 `prometheus.yml` 配置中的 `scrape_interval` 指定。最多抓取间隔为 `30` 秒，这意味着至少每 30 秒就会有一个带有新时间戳记录的新数据点，这个值可能会更改，也可能不会更改，但是每隔 `scrape_interval` 都会产生一个新的数据点。

## 6. 指标类型
从存储上来讲所有的监控指标都是相同的，但是在不同的场景下这些指标又有一些细微的差异。 例如，在 `Node Exporter` 返回的样本中指标 `node_load1` 反应的是当前系统的负载状态，随着时间的变化这个指标返回的样本数据是在不断变化的。而指标 `node_cpu_seconds_total` 所获取到的样本数据却不同，它是一个持续增大的值，因为其反应的是 CPU 的累计使用时间，从理论上讲只要系统不关机，这个值是会一直变大。

为了能够帮助用户理解和区分这些不同监控指标之间的差异，Prometheus 定义了 4 种不同的指标类型：`Counter（计数器）`、`Gauge（仪表盘）`、`Histogram（直方图）`、`Summary（摘要）`。

在 node-exporter（后面会详细讲解）返回的样本数据中，其注释中也包含了该样本的类型。例如：


```bash
# HELP node_cpu_seconds_total Seconds the cpus spent in each mode.
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="cpu0",mode="idle"} 362812.7890625
```
### 6.1 Counter
Counter (只增不减的计数器) 类型的指标其工作方式和计数器一样，**只增不减**，所以它对于存储诸如服务的 HTTP 请求数量或使用的 CPU 时间之类的信息非常有用。常见的监控指标，如 `http_requests_total`、`node_cpu_seconds_total` 都是 `Counter` 类型的监控指标。
![在这里插入图片描述](https://img-blog.csdnimg.cn/8eea2c5f065048f887c738a8072ce589.png)
可能你会觉得一直增加的数据没什么用处，了解服务从开始有多少请求有什么价值吗？但是需要记住，每个指标都存储了时间戳的，所有你的 HTTP 请求数现在可能是 1000 万，但是 Prometheus 也会记录之前某个时间点的值，我们可以去查询过去一个小时内的请求数，当然更多的时候我们想要看到的是请求数增加或减少的速度有多快，因此通常情况对于 Counter 指标我们都是去查看变化率而不是本身的数字。PromQL 内置的聚合操作和函数可以让用户对这些数据进行进一步的分析，例如，通过 `rate()` 函数**获取 HTTP 请求的增长率**：

```bash
rate(http_requests_total[5m])
```

### 6.2 Gauge
与 `Counter` 不同，`Gauge`（可增可减的仪表盘）类型的指标侧重于反应系统的当前状态，因此这类指标的样本数据可增可减。常见指标如 `node_memory_MemFree_bytes`（当前主机空闲的内存大小）、`node_memory_MemAvailable_bytes`（可用内存大小）都是 Gauge 类型的监控指标。由于 Gauge 指标仍然带有时间戳存储，所有我们可以看到随时间变化的值，通常可以直接把它们绘制出来，这样就可以看到值本身而不是变化率了，通过 Gauge 指标，用户可以直接查看系统的当前状态。

![在这里插入图片描述](https://img-blog.csdnimg.cn/ed2b8503ad8d4ca9a681cd24962c6f4c.png)
这些简单的指标类型都只是为每个样本获取一个数字，但 Prometheus 的强大之处在于如何让你跟踪它们，比如我们绘制了两张图，一个是 **HTTP 请求的变化率**，另一个是**分配的 gauge 类型的实际内存**，直接从图上就可以看出这两个之间有一种关联性，当请求出现峰值的时候，内存的使用也会出现峰值，但是我们仔细观察也会发现在峰值过后，内存使用量并没有恢复到峰值前的水平，整体上它在逐渐增加，这表明很可能应用程序中存在内存泄露的问题，通过这些简单的指标就可以帮助我们找到这些可能存在的问题。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3050c9d40b73493fb97e679c7b4a52d2.png)
对于 Gauge 类型的监控指标，通过 PromQL 内置函数 `delta()` 可以获取样本在一段时间范围内的变化情况。例如，计算 CPU 温度在两个小时内的差异：

```bash
delta(cpu_temp_celsius{host="zeus"}[2h])
```
还可以直接使用 `predict_linear()` 对数据的变化趋势进行预测。例如，预测系统磁盘空间在 4 个小时之后的剩余情况：

```bash
predict_linear(node_filesystem_free_bytes[1h], 4 * 3600)
```

### 6.3 Histogram 和 Summary
除了 `Counter` 和 `Gauge` 类型的监控指标以外，`Prometheus` 还定义了 `Histogram` 和 `Summary` 的指标类型。`Histogram` 和 `Summary` 主用用于统计和分析样本的分布情况。

在大多数情况下人们都倾向于使用某些量化指标的平均值，例如 CPU 的平均使用率、页面的平均响应时间，这种方式也有很明显的问题，以系统 API 调用的平均响应时间为例：如果大多数 API 请求都维持在 `100ms` 的响应时间范围内，而个别请求的响应时间需要 `5s`，那么就会导致某些 WEB 页面的响应时间落到中位数上，而这种现象被称为**长尾问题**。

为了区分是平均的慢还是长尾的慢，最简单的方式就是按照请求延迟的范围进行分组。例如，统计延迟在 `0~10ms` 之间的请求数有多少而 `10~20ms` 之间的请求数又有多少。通过这种方式可以快速分析系统慢的原因。`Histogram` 和 `Summary` 都是为了能够解决这样的问题存在的，通过 `Histogram` 和`Summary` 类型的监控指标，我们可以快速了解监控样本的分布情况。

#### 6.3.1 Summary

摘要用于记录某些东西的平均大小，可能是计算所需的时间或处理的文件大小，摘要显示两个相关的信息：`count（事件发生的次数）`和 `sum（所有事件的总大小）`，如下图计算摘要指标可以返回次数为 3 和总和 15，也就意味着 3 次计算总共需要 15s 来处理，平均每次计算需要花费 5s。下一个样本的次数为 10，总和为 113，那么平均值为 11.3，因为两组指标都记录有时间戳，所以我们可以使用摘要来构建一个图表，显示平均值的变化率，比如图上的语句表示的是 5 分钟时间段内的平均速率。

![在这里插入图片描述](https://img-blog.csdnimg.cn/fe0fd2246843488a8b67a57c0bd2d411.png)
例如，指标 `prometheus_tsdb_wal_fsync_duration_seconds` 的指标类型为 `Summary`，它记录了 `Prometheus Server` 中 `wal_fsync` 的处理时间，通过访问 `Prometheus Server` 的 `/metrics` 地址，可以获取到以下监控样本数据：


```bash
# HELP prometheus_tsdb_wal_fsync_duration_seconds Duration of WAL fsync.
# TYPE prometheus_tsdb_wal_fsync_duration_seconds summary
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.5"} 0.012352463
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.9"} 0.014458005
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.99"} 0.017316173
prometheus_tsdb_wal_fsync_duration_seconds_sum 2.888716127000002
prometheus_tsdb_wal_fsync_duration_seconds_count 216
```

从上面的样本中可以得知当前 `Prometheus Server` 进行 `wal_fsync` 操作的总次数为 `216` 次，耗时 `2.888716127000002s`。其中中位数（quantile=0.5）的耗时为 `0.012352463`，9 分位数（quantile=0.9）的耗时为 `0.014458005s`。

#### 6.3.2 Histogram

摘要非常有用，但是平均值会隐藏一些细节，上图中 10 与 113 的总和包含非常广的范围，如果我们想查看时间花在什么地方了，那么我们就需要直方图了。直方图以 bucket 桶的形式记录数据，所以我们可能有一个桶用于需要 1s 或更少的计算，另一个桶用于 5 秒或更少、10 秒或更少、20 秒或更少、60 秒或更少。该指标返回每个存储桶的计数，其中 3 个在 5 秒或更短的时间内完成，6 个在 10 秒或更短的时间内完成。Prometheus 中的直方图是累积的，因此所有 10 次计算都属于 60 秒或更少的时间段，而在这 10 次中，有 9 次的处理时间为 20 秒或更少，这显示了数据的分布。所以可以看到我们的大部分计算都在 10 秒以下，只有一个超过 20 秒，这对于计算百分位数很有用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/766f0bfd94fc44279ebdd49e0398a444.png)
在 Prometheus Server 自身返回的样本数据中，我们也能找到类型为 Histogram 的监控指标`prometheus_tsdb_compaction_chunk_range_seconds_bucket`：

```bash
# HELP prometheus_tsdb_compaction_chunk_range_seconds Final time range of chunks on their first compaction
# TYPE prometheus_tsdb_compaction_chunk_range_seconds histogram
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="100"} 71
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="400"} 71
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="1600"} 71
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="6400"} 71
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="25600"} 405
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="102400"} 25690
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="409600"} 71863
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="1.6384e+06"} 115928
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="6.5536e+06"} 2.5687892e+07
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="2.62144e+07"} 2.5687896e+07
prometheus_tsdb_compaction_chunk_range_seconds_bucket{le="+Inf"} 2.5687896e+07
prometheus_tsdb_compaction_chunk_range_seconds_sum 4.7728699529576e+13
prometheus_tsdb_compaction_chunk_range_seconds_count 2.5687896e+07
```
与 `Summary` 类型的指标相似之处在于 `Histogram` 类型的样本同样会反应当前指标的记录的总数(以 `_count` 作为后缀)以及其值的总量（以 `_sum` 作为后缀）。不同在于 `Histogram` 指标直接反应了在不同区间内样本的个数，区间通过标签 `le` 进行定义。

## 7. 演示服务
为了尽可能详细地给大家演示 `PromQL` 指标查询，这里我们将 `Fork` 一个开源的 `Prometheus` 演示服务来进行查询，这样可以让我们更加灵活地对指标数据进行控制，项目仓库地址：[https://github.com/cnych/prometheus_demo_service](https://github.com/cnych/prometheus_demo_service)，这是一个 Go 语言开发的服务，我们可以自己构建应用。

首先准备 `golang` 环境：

```bash
☸ ➜ wget https://golang.org/dl/go1.16.3.linux-amd64.tar.gz
☸ ➜ rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.3.linux-amd64.tar.gz
# 配置环境变量，可以将下面命令添加到 /etc/profile 中
☸ ➜ export PATH=$PATH:/usr/local/go/bin
# 执行go命令验证
☸ ➜ go version
go version go1.16.3 linux/amd64
```
然后 clone 代码构建：

```bash
# 首先clone代码(建议使用ssh模式，你懂的~)
☸ ➜ git clone https://github.com/cnych/prometheus_demo_service
☸ ➜ cd prometheus_demo_service
# 配置 GOPROXY 代理
☸ ➜ export GOPROXY=https://goproxy.cn
# 构建
☸ ➜ env GOOS=linux GOARCH=amd64 go build -o prometheus_demo_service
```
构建完成后启动 3 个服务，分别监听 `10000`、`10001`、`10002` 端口：

```bash
☸ ➜ ps -aux |grep demo
root      15224  2.9  0.1 834120 14836 pts/0    Sl   10:39   0:00 ./prometheus_demo_service --listen-address=:10000
root      15333  3.0  0.2 899656 16888 pts/0    Sl   10:39   0:00 ./prometheus_demo_service --listen-address=:10001
root      15353  2.7  0.1 907596 14896 pts/0    Sl   10:39   0:00 ./prometheus_demo_service --listen-address=:10002

```
上面 3 个服务都在 `/metrics` 端点暴露了一些指标数据，我们可以把这 3 个服务配置到 Prometheus 抓取任务中，这样后续就可以使用这几个服务来进行 PromQL 查询说明了。

完整的 `prometheus.yml` 配置文件如下所示：

```bash
global:
  scrape_interval: 5s # 抓取频率

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  # 配置demo抓取任务
  - job_name: demo
    scrape_interval: 15s
    scrape_timeout: 10s
    static_configs:
      - targets:
          - demo-service-0:10000
          - demo-service-1:10001
          - demo-service-2:10002

```
这里我们将 3 个服务配置到名为 `demo` 的抓取任务中，为了看上去更加清晰，这里我们使用 `demo-service-<index>` 来代替服务地址，直接在 `Prometheus` 所在节点的 `/etc/hosts` 文件中添加上对应服务的映射：


```bash
☸ ➜ cat /etc/hosts
......
192.168.31.46 demo-service-0
192.168.31.46 demo-service-1
192.168.31.46 demo-service-2
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1421278f05004c4086220029f70d37bb.png)
配置完成后直接启动 Prometheus 服务即可：

```bash
☸ ➜ ./prometheus
```
启动后可以在 `/targets` 页面查看是否在正确抓取监控指标：
![在这里插入图片描述](https://img-blog.csdnimg.cn/6b7bf1e4fa59445989a53cc579208ef3.png)
该演示服务模拟了一些用于我们测试的监控指标，包括：

 - 暴露请求计数和响应时间（以 path、method 和响应状态码为标签 key）的 HTTP API 服务
 - 一个定期的批处理任务，它暴露了最后一次成功运行的时间戳和处理的字节数
 - 有关 CPU 数量及其使用情况的综合指标
 - 有关内存使用情况的综合指标
 - 有关磁盘总大小及其使用情况的综合指标
 - 其他指标......


## 8. PromQL 基础
### 8.1 嵌套结构
与 SQL 查询语言（SELECT * FROM ...）不同，PromQL 是一种嵌套的函数式语言，就是我们要把需要查找的数据描述成一组嵌套的表达式，每个表达式都会评估为一个中间值，每个中间值都会被用作它上层表达式中的参数，而查询的最外层表达式表示你可以在表格、图形中看到的最终返回值。比如下面的查询语句：

```bash
histogram_quantile(  # 查询的根，最终结果表示一个近似分位数。
  0.9,  # histogram_quantile() 的第一个参数，分位数的目标值
  # histogram_quantile() 的第二个参数，聚合的直方图
  sum by(le, method, path) (
    # sum() 的参数，直方图过去5分钟每秒增量。
    rate(
      # rate() 的参数，过去5分钟的原始直方图序列
      demo_api_request_duration_seconds_bucket{job="demo"}[5m]
    )
  )
)

```
`PromQL` 表达式不仅仅是整个查询，而是查询的任何嵌套部分（比如上面的r`ate(...)`部分），你可以把它作为一个查询本身来运行。在上面的例子中，每行注释代表一个表达式。


### 8.2  结果类型
在查询 `Prometheus` 时，有两个 类型 的概念经常出现，区分它们很重要。

 - 抓取目标报告的指标类型：`counter`、`gauge`、`histogram`、`summary`。
 - PromQL 表达式的结果数据类型：`字符串`、`标量`、`瞬时向量`或`区间向量`。

PromQL 实际上没有直接的指标类型的概念，只关注表达式的结果类型。每个 PromQL 表达式都有一个类型，每个函数、运算符或其他类型的操作都要求其参数是某种表达式类型。例如，`rate()` 函数要求它的参数是一个区间向量，但是 `rate()` 本身评估为一个瞬时向量输出，所以 `rate()` 的结果只能用在期望是瞬时向量的地方。

PromQL 中可能的表达式类型包括：

 - `string(字符串)`：字符串只会作为某些函数（如 `label_join()` 和 `label_replace()`）的参数出现。
 - `scalar(标量)`：一个单一的数字值，如 `1.234`，这些数字可以作为某些函数的参数，如 `histogram_quantile(0.9, ...)` 或 `topk(3, ...)`，也会出现在**算术运算**中。
 - `instant vector(瞬时向量)`：一组标记的时间序列，每个序列有一个样本，都在同一个时间戳，瞬时向量可以由 TSDB 时间序列选择器直接产生，如`node_cpu_seconds_total`，也可以由任何函数或其他转换来获取。

```bash
node_cpu_seconds_total{cpu="0", mode="idle"}   → 19165078.75 @ timestamp_1
node_cpu_seconds_total{cpu="0", mode="system"} →   381598.72 @ timestamp_1
node_cpu_seconds_total{cpu="0", mode="
user"}   → 23211630.97 @ timestamp_1
```
- `range vector(区间向量)`：一组标记的时间序列，每个序列都有一个随时间变化的样本范围。在 PromQL 中只有两种方法可以生成区间向量：在查询中使用字面区间向量选择器（如 `node_cpu_seconds_total[5m]`），或使用子查询表达式（如 `<expression>[5m:10s]`），当想要在指定的时间窗口内聚合一个序列的行为时，区间向量非常有用，就像 `rate(node_cpu_seconds_total[5m])` 计算每秒增加率一样，在 `node_cpu_seconds_total` 指标的最后 5 分钟内求平均值。

```bash
node_cpu_seconds_total{cpu="0", mode="idle"}   → 19165078.75 @ timestamp_1,  19165136.3 @ timestamp_2, 19165167.72 @ timestamp_3
node_cpu_seconds_total{cpu="0", mode="system"} → 381598.72   @ timestamp_1,   381599.98 @ timestamp_2,   381600.58 @ timestamp_3
node_cpu_seconds_total{cpu="0", mode="user"}   → 23211630.97 @ timestamp_1, 23211711.34 @ timestamp_2, 23211748.64 @ timestamp_3
```

> 注意：但是指标类型呢？如果你已经使用过 PromQL，你可能知道某些函数仅适用于特定类型的指标！例如，`histogram_quantile()` 函数仅适用于直方图指标， `rate()` 仅适用于计数器指标，`deriv()` 仅适用于 `Gauge`。但是 PromQL 实际上并没有检查你是否传入了正确类型的指标——这些函数通常会运行并为错误类型的输入指标返回一些无意义的数据，这取决于用户是否传入了遵守某些假设的时间序列（比如在直方图的情况下有一个有意义的 le 标签，或者在计数器的情况下单调递增）。

### 8.3 查询类型和评估时间
PromQL 查询中对时间的引用只有相对引用，比如 `[5m]`，表示过去 5 分钟，那么如何指定一个绝对的时间范围，或在一个表格中显示查询结果的时间戳？在 PromQL 中，这样的时间参数是与表达式分开发送到 Prometheus 查询 API 的，确切的时间参数取决于你发送的查询类型，Prometheus 有两种类型的 PromQL 查询：**瞬时查询**和**区间查询**。

#### 8.3.1 瞬时查询
瞬时查询用于类似表格的视图，你想在一个时间点上显示 PromQL 查询的结果。一个瞬时查询有以下参数：

 - PromQL 表达式
 - 一个评估的时间戳

在查询的时候可以选择查询过去的数据，比如 `foo[1h]` 表示查询 `foo` 序列最近 1 个小时的数据，访问过去的数据，对于计算一段时间内的比率或平均数等聚合会非常有用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5e013001ec2344d491096d8b15cdef54.png)
在 `Prometheus` 的 WebUI 界面中表格视图中的查询就是瞬时查询，API 接口 `/api/v1/query?query=xxxx&time=xxxx` 中的 `query` 参数就是 PromQL 表达式，`time` 参数就是评估的时间戳。瞬时查询可以返回任何有效的 PromQL 表达式类型（字符串、标量、即时和范围向量）。

下面来看一个瞬时查询的示例，看看它是如何进行评估工作的。比如 `http_requests_total` 在指定的时间戳来评估表达式，`http_requests_total` 是一个瞬时向量选择器，它可以选择该时间序列的最新样本，最新意味着查询最近 5 分钟的样本数据。

如果我们在一个有最近样本的时间戳上运行此查询，结果将包含两个序列，每个序列都有一个样本：
![在这里插入图片描述](https://img-blog.csdnimg.cn/98768da6ed7945a087f955fca32ef04e.png)
注意每个返回的样本输出时间戳不再是原始样本被采集的时间戳，而会被设置为评估的时间戳。

如果在时间戳之前有一个 `>5m` 的间隙，这个时候如果我们执行相同的查询：

![在这里插入图片描述](https://img-blog.csdnimg.cn/3a451aeebd74470b82d88b9c985069bb.png)
这个情况下查询的结果将返回为空，因为很显然在最近 5 分钟内没有能够匹配的样本。

####  8.3.2 区间查询
区间查询主要用于图形，想在一个指定的时间范围内显示一个 PromQL 表达式，范围查询的工作方式与即时查询完全相同，这些查询在指定时间范围的评估步长中进行评估。当然，这在后台是高度优化的，在这种情况下，Prometheus 实际上并没有运行许多独立的即时查询。

区间查询包括以下一些参数：

 - PromQL 表达式
 - 开始时间
 - 结束时间
 - 评估步长

在开始时间和结束时间之间的每个评估步长上评估表达式后，单独评估的时间片被拼接到一个单一的区间向量中。区间查询允许传入瞬时向量类型或标量类型的表达式，但始终返回一个范围向量（标量或瞬时向量在一个时间范围内被评估的结果）。

在 Prometheus 的 WebUI 界面中图形视图中的查询就是区间查询，API 接口 `/api/v1/query_range?query=xxx&start=xxxxxx&end=xxxx&step=14` 中的 `query` 参数就是 PromQL 表达式，`start` 为开始时间，`end` 为结束时间，`step` 为评估的步长。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1080b937a9d14355b668fadb731ac391.png)
比如把上面的 `http_requests_total` 表达式作为一个范围查询来进行评估，它的评估结果如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/97912557a0134416b6711b2261eea20d.png)
注意每个评估步骤的行为与独立的瞬时查询完全一样，而且每个独立的瞬时查询都没有查询的总体范围的概念，在我们这个示例中最终的结果将是一个区间向量，其中包含两个选定序列在一定时间范围内的样本，但也将包含某些时间步长的序列数据的间隙。

## 9. 选择时间序列
###  9.1 过滤指标名称
最简单的 PromQL 查询就是直接选择具有指定指标名称的序列，例如，以下查询将返回所有具有指标名称 `demo_api_request_duration_seconds_count` 的序列：


```bash
demo_api_request_duration_seconds_count
```

该查询将返回许多具有相同指标名称的序列，但有不同的标签组合 `instance`、`job`、`method`、`path` 和 `status` 等。输出结果如下所示：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/243f269b59a24ac59977eac455df90f7.png)
###  9.2 根据标签过滤
如果我们只查询 `demo_api_request_duration_seconds_count` 中具有 `method="GET"` 标签的那些指标序列，则可以在指标名称后用大括号加上这个过滤条件：


```bash
demo_api_request_duration_seconds_count{method="GET"}
```

此外我们还可以使用逗号来组合多个标签匹配器：


```bash
demo_api_request_duration_seconds_count{instance="demo-service-0:10000",method="GET",job="demo"}
```

上面将得到 demo 任务下面 `demo-service-0:10000` 这个实例且 `method="GET"` 的指标序列数据：
![在这里插入图片描述](https://img-blog.csdnimg.cn/5863ab01167342b3af53b2cef6970eaf.png)
需要注意的是组合使用多个匹配条件的时候，是过滤所有条件都满足的时间序列。

除了相等匹配之外，Prometheus 还支持其他几种匹配器类型：

 - `!=`：不等于
 - `=~`：正则表达式匹配
 - `!~`：正则表达式不匹配

甚至我们还可以完全省略指标名称，比如直接查询所有 `path` 标签以 `/api` 开头的所有序列：

```bash
{path=~"/api.*"}
```

该查询会得到一些具有不同指标名称的序列：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9aec23b9f9c149f4908423e909e850f9.png)

> 注意： Prometheus 中的正则表达式总是针对完整的字符串而不是部分字符串匹配。因此，在匹配任何以 /api 开通的路径时，不需要以 ^ 开头，但需要在结尾处添加 .*，这样可以匹配 path="/api" 这样的序列。

前面我们说过在 `Prometheus` 内部，指标名称本质上是一个名为 `__name__` 的特性标签，所以查询 `demo_api_request_duration_seconds_count` 实际上和下面的查询方式是等效的：


```bash
{__name__="demo_api_request_duration_seconds_count"}
```

按上面的方法编写的选择器，可以得到一个瞬时向量，其中包含所有选定序列的单个最新值。事实上有些函数要求你不是传递一个单一的值，而是传递一个序列在一段时间范围内的值，也就是前面我们说的区间向量。这个时候我们可以通过附加一个`[<数字><单位>]`形式的持续时间指定符，将即时向量选择器改变为范围向量选择器（例如[5m]表示 5 分钟）。

比如要查询最近 5 分钟的可用内存，可以执行下面的查询语句：


```bash
demo_memory_usage_bytes{type="free"}[5m]
```

将得到如下所示的查询结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9dcdada64b404c8e8b4a4539e92e4c8b.png)
可以使用的有效的时间单位为：

 - ms -毫秒
 - s -秒
 - m - 分钟
 - h - 小时
 - d - 天
 - y - 年

有时我们还需要以时移方式访问过去的数据，通常用来与当前数据进行比较。要将过去的数据时移到当前位置，可以使用 `offset <duration>` 修饰符添加到任何范围或即时序列选择器进行查询（例如 `my_metric offset 5m` 或 `my_metric[1m] offset 7d`）。

例如，要选择一个小时前的可用内存，可以使用下面的查询语句：


```bash
demo_memory_usage_bytes{type="free"} offset 1h
```

这个时候查询的值则是一个小时之前的数据：
![在这里插入图片描述](https://img-blog.csdnimg.cn/734be5b3244d453a9e3690bfe28c1ed9.png)
练习：

1.构建一个查询，选择所有时间序列。
```bash
{job!=""}
```
或者：

```bash
{__name__=~".+"}
```

2.构建一个查询，查询所有指标名称为 `demo_api_request_duration_seconds_count` 并且 `method` 标签不为 `POST` 的序列。
```bash
demo_api_request_duration_seconds_count{method!="POST"}
```
3.使用 `demo_memory_usage_bytes` 指标查询一小时前的 1 分钟时间范围的的可用空闲内存。

```bash
demo_memory_usage_bytes{type="free"}[1m] offset 1h
```

##  10. 变化率
通常来说直接绘制一个原始的 `Counter` 类型的指标数据用处不大，因为它们会一直增加，一般来说是不会去直接关心这个数值的，因为 Counter 一旦重置，总计数就没有意义了，比如我们直接执行下面的查询语句：
```bash
demo_api_request_duration_seconds_count{job="demo"}
```
可以得到下图所示的图形：
![在这里插入图片描述](https://img-blog.csdnimg.cn/3ff3556ef98848f8be7926a1ed6363f4.png)
可以看到所有的都是不断增长的，一般来说我们更想要知道的是 Counter 指标的变化率，PromQL 提供了不同的函数来计算变化率。

###  10.1 rate
用于计算变化率的最常见函数是 `rate()`，`rate()` 函数用于计算在指定时间范围内计数器平均每秒的增加量。因为是计算一个时间范围内的平均值，所以我们需要在序列选择器之后添加一个范围选择器。

例如我们要计算 `demo_api_request_duration_seconds_count` 在最近五分钟内的每秒平均变化率，则可以使用下面的查询语句：


```bash
rate(demo_api_request_duration_seconds_count[5m])
```

可以得到如下所示的图形：
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2952f6573ed413f9238118a8d04a097.png)
现在绘制的图形看起来显然更加有意义了，进行 rate 计算的时候是选择指定时间范围下的第一和最后一个样本进行计算，下图是表示瞬时计算的计算方式：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f3f7c2d9ad204bcb8b91ef1e00c2795e.png)
往往我们需要的是绘制一个图形，那么就需要进行区间查询，指定一个时间范围内进行多次计算，将结果串联起来形成一个图形：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ae84bc4b3d941c7970efb96596570e3.png)
对于 `rate()` 和相关函数有几个需要说明的：

- 当被抓取指标进的程重启时，Counter 指标可能会重置为 0，但 `rate()` 函数会自动处理这个问题，它会假设 Counter 指标的值只要是减少了就认为是被重置了，然后它可以调整后续的样本，例如，如果时间序列的值为[5,10,4,6]，则将其视为[5,10,14,16]。
- 变化率是从指定的时间范围下包含的样本进行计算的，需要注意的是这个时间窗口的边界并不一定就是一个样本数据，可能会不完全对齐，所以，即使对于每次都是增加整数的 Counter，也可能计算结果是非整数。

![在这里插入图片描述](https://img-blog.csdnimg.cn/ae9dc5ba704d40d19036b6d2a5369981.png)
另外我们需要注意当把 `rate()` 与一个聚合运算符（例如 `sum()`）或一个随时间聚合的函数（任何以 `_over_time` 结尾的函数）结合起来使用时，总是先取用 `rate()` 函数，然后再进行聚合，否则，当你的目标重新启动时，`rate()` 函数无法检测到 `Counter` 的重置。

> 注意：`rate()` 函数需要在指定窗口下至少有两个样本才能计算输出。一般来说，比较好的做法是选择范围窗口大小至少是抓取间隔的`4`倍，这样即使在遇到窗口对齐或抓取故障时也有可以使用的样本进行计算，例如，对于 1 分钟的抓取间隔，你可以使用 4 分钟的 `Rate` 计算，但是通常将其四舍五入为 5 分钟。所以如果使用 `query_range` 区间查询，例如在绘图中，那么范围应该至少是步长的大小，否则会丢失一些数据。


### 10.2 irate [increase, predict_linear, delta, deriv]
由于使用 `rate` 或者 `increase` 函数去计算样本的平均增长速率，容易陷入长尾问题当中，其无法反应在时间窗口内样本数据的突发变化。

例如，**对于主机而言在 2 分钟的时间窗口内，可能在某一个由于访问量或者其它问题导致 CPU 占用 100%的情况，但是通过计算在时间窗口内的平均增长率却无法反应出该问题**。

为了解决该问题，PromQL 提供了另外一个灵敏度更高的函数`irate(v range-vector)`。irate 同样用于计算区间向量的计算率，但是其反应出的是瞬时增长率。

`irate` 函数是通过区间向量中最后两个样本数据来计算区间向量的增长速率。这种方式可以避免在时间窗口范围内的长尾问题，并且体现出更好的灵敏度，通过 `irate` 函数绘制的图标能够更好的反应样本数据的瞬时变化状态。那既然是使用最后两个点计算，那为什么还要指定类似于 `[1m]` 的时间范围呢？这个 `[1m]` 不是用来计算的，irate 在计算的时候会最多向前在 `[1m]` 范围内找点，如果超过 [1m] 没有找到数据点，这个点的计算就放弃了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/41b3d167aaae49b1b259546cf7be9042.png)
由于 `rate()` 提供了更平滑的结果，因此在长期趋势分析或者告警中更推荐使用 `rate` 函数，因为当速率只出现一个短暂的峰值时，不应该触发该报警。

使用 `irate()` 函数上面的表达式会出现一些短暂下降的图形：

![在这里插入图片描述](https://img-blog.csdnimg.cn/15b3ec5716f34ba4b1fc564c0f76f151.png)
除了计算每秒速率，你还可以使用 `increase()` 函数查询指定时间范围内的总增量，它基本上相当于**速率乘以时间范围选择器中的秒数**：


```bash
increase(demo_api_request_duration_seconds_count{job="demo"}[1h])
```

比如上面表达式的结果和使用 `rate()` 函数计算的结果整体图形趋势都是一样的，只是 Y 轴的数据不一样而已，一个表示数量，一个表示百分比。`rate()`、`irate()` 和 `increase()` 函数只能输出非负值的结果，对于跟踪一个可以上升或下降的值的指标（如温度、内存或磁盘空间），可以使用 `delta()` 和 `deriv()` 函数来代替。

`deriv()` 函数可以计算一个区间向量中各个时间序列二阶导数，使用简单线性回归，`deriv(v range-vector)` 的参数是一个区间向量，返回一个瞬时向量，这个函数一般只用在 `Gauge` 类型的时间序列上。例如，要计算在 15 分钟的窗口下，每秒钟磁盘使用量上升或下降了多少：

![在这里插入图片描述](https://img-blog.csdnimg.cn/659d53d84a044a9181ecdce079045d3e.png)
还有另外一个 `predict_linear()` 函数可以**预测一个 Gauge 类型的指标在未来指定一段时间内的值**，例如我们可以根据过去 15 分钟的变化情况，来预测一个小时后的磁盘使用量是多少，可以用如下所示的表达式来查询：


```bash
predict_linear(demo_disk_usage_bytes{job="demo"}[15m], 3600)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/36cd6acc93b544de8571024f3a54fa8d.png)
这个函数可以用于报警，告诉我们磁盘是否会在几个小时候内用完。

## 11. 聚合
我们知道 Prometheus 的时间序列数据是多维数据模型，我们经常就有根据各个维度进行汇总的需求。
### 11.1 基于标签聚合 [sum,max,min,avg,count,topk,group.... ]
例如我们想知道我们的 `demo` 服务每秒处理的请求数，那么可以将单个的速率相加就可以。


```bash
sum(rate(demo_api_request_duration_seconds_count{job="demo"}[5m]))
```

可以得到如下所示的结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/4dea16e2df5c4d4c843e0c70bbe33d55.png)
但是我们可以看到绘制出来的图形没有保留任何标签维度，一般来说可能我们希望保留一些维度，例如，我们可能更希望计算每个 `instance` 和 `path` 的变化率，但并不关心单个 `method` 或者 `status` 的结果，这个时候我们可以在 `sum()` 聚合器中添加一个 `without()` 的修饰符：


```bash
sum without(method, status) (rate(demo_api_request_duration_seconds_count{job="demo"}[5m]))
```

上面的查询语句相当于用 `by()` 修饰符来保留需要的标签的取反操作：


```bash
sum by(instance, path, job) (rate(demo_api_request_duration_seconds_count{job="demo"}[5m]))
```

现在得到的 `sum` 结果是就是按照 `instance`、`path`、`job` 来进行分组去聚合的了：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b1ff9ac17b2e41b58fa02763424a5f66.png)
这里的分组概念和 SQL 语句中的分组去聚合就非常类似了。

除了 `sum()` 之外，`Prometheus` 还支持下面的这些聚合器：

 - `sum()`：对聚合分组中的所有值进行求和
 - `min()`：获取一个聚合分组中最小值
 - `max()`：获取一个聚合分组中最大值
 - `avg()`：计算聚合分组中所有值的平均值
 - `stddev()`：计算聚合分组中所有数值的标准差
 - `stdvar()`：计算聚合分组中所有数值的标准方差
 - `count()`：计算聚合分组中所有序列的总数
 - `count_values()`：计算具有相同样本值的元素数量
 - `bottomk(k, ...)`：计算按样本值计算的最小的 k 个元素
 - `topk(k，...)`：计算最大的 k 个元素的样本值
 - `quantile(φ，...)`：计算维度上的 φ-分位数(0≤φ≤1)
 - `group(...)`：只是按标签分组，并将样本值设为 1。

练习：

1.按 `job` 分组聚合，计算我们正在监控的所有进程的总内存使用量（`process_resident_memory_bytes` 指标）：

```bash
sum by(job) (process_resident_memory_bytes)
```
2.计算 `demo_cpu_usage_seconds_total` 指标有多少不同的 CPU 模式：

```bash
count (group by(mode) (demo_cpu_usage_seconds_total))
```

3.计算每个 `job` 任务和指标名称的时间序列数量：

```bash
count by (job, __name__) ({__name__ != ""})
```

### 11.2 基于时间聚合 [avg_over_time()...... ]
前面我们已经学习了如何使用 `sum()`、`avg()` 和相关的聚合运算符从标签维度进行聚合，这些运算符在一个时间内对多个序列进行聚合，但是有时候我们可能想在每个序列中按时间进行聚合，例如，使尖锐的曲线更平滑，或深入了解一个序列在一段时间内的最大值。

为了基于时间来计算这些聚合，`PromQL` 提供了一些与标签聚合运算符类似的函数，但是在这些函数名前面附加了 `_over_time()`：

 - `avg_over_time(range-vector)`：区间向量内每个指标的**平均值**。
 - `min_over_time(range-vector)`：区间向量内每个指标的**最小值**。
 - `max_over_time(range-vector)`：区间向量内每个指标的**最大值**。
 - `sum_over_time(range-vector)`：区间向量内每个指标的**求和**。
 - `count_over_time(range-vector)`：区间向量内每个指标的**样本数据个数**。
 - `quantile_over_time(scalar, range-vector)`：区间向量内每个指标的**样本数据值分位数**。
 - `stddev_over_time(range-vector)`：区间向量内每个指标的**总体标准差**。
 - `stdvar_over_time(range-vector)`：区间向量内每个指标的**总体标准方差**。

例如，我们查询 `demo` 实例中使用的 `goroutine` 的原始数量，可以使用查询语句 `go_goroutines{job="demo"}`，这会产生一些尖锐的峰值图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/37812dcc1990412e9d0acc3e7962552e.png)
我们可以通过对图中的每一个点来计算 10 分钟内的 `goroutines` 数量进行平均来使图形更加平滑：
```bash
avg_over_time(go_goroutines{job="demo"}[10m])
```
这个查询结果生成的图表看起来就平滑很多了：
![在这里插入图片描述](https://img-blog.csdnimg.cn/76b5e6cc32b947c595be63421866f48f.png)
比如要**查询 1 小时内内存的使用率**则可以用下面的查询语句：

```bash
100 * (1 - ((avg_over_time(node_memory_MemFree_bytes[1h]) + avg_over_time(node_memory_Cached_bytes[1h]) + avg_over_time(node_memory_Buffers_bytes[1h])) / avg_over_time(node_memory_MemTotal_bytes[1h])))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c45541c753944b28b7d6854bd3019bc4.png)
### 11.3 子查询 [max_over_time()]
上面所有的 `_over_time()` 函数都需要一个范围向量作为输入，通常情况下只能由一个区间向量选择器来产生，比如 `my_metric[5m]`。但是如果现在我们想使用例如 `max_over_time()` 函数来找出过去一天中 `demo` 服务的最大请求率应该怎么办呢？

请求率 `rate` 并不是一个我们可以直接选择时间的原始值，而是一个计算后得到的值，比如：
```bash
rate(demo_api_request_duration_seconds_count{job="demo"}[5m])
```
如果我们直接将表达式传入 `max_over_time()` 并附加一天的持续时间查询的话就会产生错误：


```bash
# ERROR!
max_over_time(
  rate(
    demo_api_request_duration_seconds_count{job="demo"}[5m]
  )[1d]
)
```

实际上 Prometheus 是支持子查询的，它允许我们首先以指定的步长在一段时间内执行内部查询，然后根据子查询的结果计算外部查询。子查询的表示方式类似于区间向量的持续时间，但需要冒号后添加了一个额外的步长参数：`[<duration>:<resolution>]`。

这样我们可以重写上面的查询语句，告诉 Prometheus 在一天的范围内评估内部表达式，步长分辨率为 15s：


```bash
max_over_time(
  rate(
    demo_api_request_duration_seconds_count{job="demo"}[5m]
  )[1d:15s] # 在1天内明确地评估内部查询，步长为15秒
)
```

也可以省略冒号后的步长，在这种情况下，Prometheus 会使用配置的全局 `evaluation_interval` 参数进行评估内部表达式：


```bash
max_over_time(
  rate(
    demo_api_request_duration_seconds_count{job="demo"}[5m]
  )[1d:]
)
```

这样就可以得到过去一天中 demo 服务最大的 5 分钟请求率，不过冒号仍然是需要的，以明确表示运行子查询。子查询还允许添加一个偏移修饰符 `offset` 来对内部查询进行时间偏移，类似于瞬时和区间向量选择器。

但是也需要注意长时间计算子查询代价也是非常昂贵的，我们可以使用记录规则（后续会讲解）预先记录中间的表达式，而不是每次运行外部查询时都实时计算它。

练习：

输出过去一小时内 demo 服务的最大 95 分位数延迟值（1 分钟内平均），按 path 划分：

```bash
max_over_time(
   histogram_quantile(0.95, sum by(le, path) (
     rate(demo_api_request_duration_seconds_bucket[1m])
    )
  )[1h:]
)
```

## 12. 运算
Prometheus 的查询语言支持基本的逻辑运算和算术运算。
### 12.1 算术运算符
在 Prometheus 系统中支持下面的二元算术运算符：

加法

 - `-` 减法
-  `*` 乘法
- `/` 除法
- `%` 模
- `^` 幂等

最简单的我们可以将一个数字计算当做一个 PromQL 语句，用于标量与标量之间计算，比如：


```bash
(2 + 3 / 6) * 2^2
```

可以得到如下所示的结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/871f537665b94c1c99fca1985c29306c.png)
图形中返回的是一个值为 10 的标量（scalar）类型的数据。

二元运算同样适用于向量和标量之间，例如我们可以将一个字节数除以两次 1024 来转换为 MiB，如下查询语句：

```bash
demo_batch_last_run_processed_bytes{job="demo"} / 1024 / 1024
```

最后计算的结果就是 `MiB` 单位的了：

![在这里插入图片描述](https://img-blog.csdnimg.cn/e401cc82005c43daab47542df008549a.png)

另外 PromQL 的一个强大功能就是可以让我们在向量与向量之间进行二元运算。

例如 `demo_api_request_duration_seconds_sum` 的数据包含了在 `path`、`method`、`status` 等不同维度上花费的总时间，指标 `demo_api_request_duration_seconds_count` 包含了上面同维度下的请求总次数。则我们可以用下面的语句来查询过去 5 分钟的平均请求持续时间：

```bash
rate(demo_api_request_duration_seconds_sum{job="demo"}[5m])
/
rate(demo_api_request_duration_seconds_count{job="demo"}[5m])
```
PromQL 会通过相同的标签集自动匹配操作符左边和右边的元素，并将二元运算应用到它们身上。由于上面两个指标的标签集合都是一致的，所有可以得到相同标签集的平均请求延迟结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c448c825d12241cf841fa472312f2d7f.png)
###  12.2 向量匹配
#### 12.2.1  一对一
上面的示例其实就是一对一的向量匹配，但是一对一向量匹配也有两种情况，就是是否按照所有标签匹配进行计算，下图是匹配所有标签的情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/7813be7b3d654590ab085104338092c2.png)
图中我们两个指标 `foo` 和 `bar`，分别生成了 3 个序列：

```bash
# TYPE foo gauge
foo{color="red", size="small"} 4
foo{color="green", size="medium"} 8
foo{color="blue", size="large"} 16
# TYPE bar gauge
bar{color="green", size="xlarge"} 2
bar{color="blue", size="large"} 7
bar{color="red", size="small"} 5

```
当我们执行查询语句 `foo{} + bar{}` 的时候，对于向量左边的每一个元素，操作符都会尝试在右边里面找到一个匹配的元素，匹配是通过比较所有的标签来完成的，没有匹配的元素会被丢弃，我们可以看到其中的 `foo{color="green", size="medium"}` 与 `bar{color="green", size="xlarge"}` 两个序列的标签是不匹配的，其余两个序列标签匹配，所以计算结果会抛弃掉不匹配的序列，得到的结果为其余序列的值相加。

上面例子中其中不匹配的标签主要是因为第二个 size 标签不一致造成的，那么如果我们在计算的时候忽略掉这个标签可以吗？如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/76b7d90d003249cd8f040e2ec8fa9f3c.png)
同样针对上面的两个指标，我们在进行计算的时候可以使用 `on` 或者 `ignoring` 修饰符来指定用于匹配的标签进行计算，由于示例中两边的标签都具有 color 标签，所以在进行计算的时候我们可以基于该标签（on (color)）或者忽略其他的标签（ignoring (size)）进行计算，这样得到的结果就是所以匹配的标签序列相加的结果，要注意结果中的标签也是匹配的标签。

#### 12.2.2 一对多与多对一 [by, on, group_left]
上面讲解的一对一的向量计算是最直接的方式，在多数情况下，`on` 或者 `ignoring` 修饰符有助于是查询返回合理的结果，但通常情况用于计算的两个向量之间并不是一对一的关系，更多的是一对多或者多对一的关系，对于这种场景我们就不能简单使用上面的方式进行处理了。

多对一和一对多两种匹配模式指的是一侧的每一个向量元素可以与多侧的多个元素匹配的情况，在这种情况下，必须使用 `group` 修饰符：`group_left` 或者 `group_right` 来确定哪一个向量具有更高的基数（充当多的角色）。多对一和一对多两种模式一定是出现在操作符两侧表达式返回的向量标签不一致的情况，因此同样需要使用 ignoring 和 on 修饰符来排除或者限定匹配的标签列表。

例如 `demo_num_cpus` 指标告诉我们每个实例的 CPU 核心数量，只有 `instance` 和 `job` 这两个标签维度。
![在这里插入图片描述](https://img-blog.csdnimg.cn/e69ead419c4a403e926814041c7a6ab2.png)

而 `demo_cpu_usage_seconds_total` 指标则多了一个 `mode` 标签的维度，将每个 `mode` 模式（`idle`、`system`、`user`）的 CPU 使用情况分开进行了统计。

![在这里插入图片描述](https://img-blog.csdnimg.cn/0613a2e6667040b0a32044adbb94386e.png)
如果要计算每个模式的 CPU 使用量除以核心数，我们需要告诉除法运算符按照 `demo_cpu_usage_seconds_total` 指标上额外的 `mode` 标签维度对结果进行分组，我们可以使用 `group_left`（表示左边的向量具有更高的基数）修饰符来实现。同时，我们还需要通过 `on()` 修饰符明确将所考虑的标签集减少到需要匹配的标签列表：


```bash
rate(demo_cpu_usage_seconds_total{job="demo"}[5m])
/ on(job, instance) group_left
demo_num_cpus{job="demo"}
```

上面的表达式可以正常得到结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/66af8271b0eb46ac96a7566408557d67.png)
除了 `on()` 之外，还可以使用相反的 `ignoring()` 修饰符，可以用来将一些标签维度从二元运算操作匹配中忽略掉，如果在操作符的右侧有额外的维度，则应该使用 group_right（表示右边的向量具有更高的基数）修饰符。

比如上面的查询语句同样可以用 ignoring 关键字来完成：


```bash
rate(demo_cpu_usage_seconds_total{job="demo"}[5m])
/ ignoring(mode) group_left
demo_num_cpus{job="demo"}
```

得到的结果和前面用 on() 查询的结果是一致的。

到这里我们就知道了如何在 PromQL 中进行标量和向量之间的运算了。不过我们在使用 PromQL 查询数据的时候还行要避免使用关联查询，先想想能不能通过 `Relabel`（后续会详细介绍）的方式给原始数据多加个 Label，一条语句能查出来的何必用 `Join` 呢？时序数据库不是关系数据库。

**练习：**

1.计算过去 5 分钟所有 POST 请求平均数的总和相对于所有请求平均数总和的百分比。


```bash
sum(rate(demo_api_request_duration_seconds_count{method="POST"}[5m]))
/
sum(rate(demo_api_request_duration_seconds_count[5m])) * 100
```

2.计算过去 5 分钟内每个实例的 user 和 system 的模式（demo_cpu_usage_seconds_total 指标）下 CPU 使用量平均值总和。


```bash
sum by(instance, job) (rate(demo_cpu_usage_seconds_total{mode=~"user|system"}[5m]))
```

或者


```bash
sum without(mode) (rate(demo_cpu_usage_seconds_total{mode=~"user|system"}[5m]))
```

或者


```bash
rate(demo_cpu_usage_seconds_total{mode="user"}[5m]) + ignoring(mode)
rate(demo_cpu_usage_seconds_total{mode="system"}[5m])
```

##  13. 阈值 [>、<、==, ignoring, predict_linear]
`PromQL` 通过提供一组过滤的**二元运算符**（>、<、== 等），允许根据其样本值过滤一组序列，这种过滤最常见的场景就是在报警规则中使用的阈值。比如我们想查找在过去 `15` 分钟内的 `status="500"` 错误率大于 `20%` 的所有 `HTTP` 路径，我们在 `rate` 表达式后面添加一个 `>0.2` 的过滤运算符：


```bash
rate(demo_api_request_duration_seconds_count{status="500",job="demo"}[15m]) > 0.2
```

这个查询只会将错误率大于 20% 的数据过滤出来。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7ecefec9fd7743aea2544c9c5d65cf7f.png)

> 注意：由于在图形中的每个步长都是完全独立评估表达式的，因此根据每个步骤的过滤条件，某些比率会出现或消失（因此存在间隙）。一般来说，二元过滤运算符在图形中并不常见，大多数在报警条件中出现，用来表示阈值。

这种过滤方式不仅适用于单个数字，PromQL 还允许你用一组时间序列过滤另一组序列。与上面的二元运算一样，比较运算符会自动应用于比较左侧和右侧具有相同标签集的序列之间。 `on() / ignoring()` 和 `group_left() / group_right()` 修饰符的作用也与我们前面学习的二元算术运算符一样。

以下示例是选择所有具有 500 错误率且至少比同一路径的总请求率大 50 倍的路径：


```bash
  rate(demo_api_request_duration_seconds_count{status="500",job="demo"}[5m]) * 50
> ignoring(status)
  sum without(status) (rate(demo_api_request_duration_seconds_count{
  job="demo"}[5m]))
```

不过需要注意的是我们必须忽略匹配中的 `status` 标签，因为在左边一直有这个标签，而右边没有这个标签。

![在这里插入图片描述](https://img-blog.csdnimg.cn/fb4aafb5416f4b63bfddaa9ff134bd17.png)
比如我们还可以计算 demo 演示服务实例在一小时内的预测磁盘使用量，但要过滤只有那些预测磁盘已满的实例。


```bash
predict_linear(demo_disk_usage_bytes{job="demo"}[1h], 3600) >= demo_disk_total_bytes{job="demo"}
```
Prometheus 支持以下过滤操作：

 - `==`
 - `!=`
 - `<`
 - `<=`
 - `>`
 - `>=`

有时你可能想知道比较运算符的结果而不实际删除任何输出系列。要实现这一点，我们可以向运算符添加一个 `bool` 修饰符来保留所有的序列，但是把输出样本值设置为 1（比较为真）或 0（比较为假）。

例如，要简单地显示一组数据中哪些请求率高于或低于 0.2/s，我们可以这样查询：


```bash
rate(demo_api_request_duration_seconds_count{job="demo"}[5m]) > bool 0.2
```

我们可以看到输入序列的结果为 0 或 1，把数字条件转换为了布尔输出值。

![在这里插入图片描述](https://img-blog.csdnimg.cn/017e5e4e394244eba704628faf31e93f.png)
**练习：**

1.构建一个查询，显示使用少于 20MB 内存的目标（process_resident_memory_bytes 指标）。


```bash
process_resident_memory_bytes / 1024^2 < 20
```

2.构建一个查询，显示 Prometheus 服务内部所有在过去 5 分钟内没有收到任何查询的 HTTP 处理器。


```bash
rate(prometheus_http_requests_total[5m]) == 0
```

##  13. 集合操作 [and, or, unless]
有的时候我们需要过滤或将一组时间序列与另一组时间序列进行合并，Prometheus 提供了 3 个在瞬时向量之间操作的集合运算符。

 - `and`（集合交集）：比如对较高错误率触发报警，但是只有当对应的总错误率超过某个阈值的时候才会触发报警
 - `or`（集合并集）：对序列进行并集计算
 - `unless`（除非）：比如要对磁盘空间不足进行告警，除非它是只读文件系统。

![在这里插入图片描述](https://img-blog.csdnimg.cn/61232d9c90fd47aea16e909d627c2029.png)

与算术和过滤二元运算符类似，这些集合运算符会尝试根据相同的标签集在左侧和右侧之间查找来匹配序列，除非你提供 `on()` 或 `ignoring()` 修饰符来指定应该如何找到匹配。

> 注意：与算术和过滤二进制运算符相比，集合运算符没有 `group_left()` 或 `group_right()`修饰符，因为集合运算符总是进行多对多的匹配，也就是说，它们总是允许任何一边的匹配序列与另一边的多个序列相匹配。

对于 `and` 运算符，如果找到一个匹配的，左边的序列就会成为输出结果的一部分，如果右边没有匹配的序列，则不会输出任何结果。

例如我们想筛选出第 90 个百分位延迟高于 50ms 的所有 HTTP 端点，但只针对每秒收到多个请求的维度组合，查询方式如下所示：

```bash
 histogram_quantile(0.9, rate(demo_api_request_duration_seconds_bucket{job="demo"}[5m])) > 0.05
and
  rate(demo_api_request_duration_seconds_count{job="demo"}[5m]) > 1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/94b7ab36483447c48a8eb403fc2d88b8.png)
有的时候我们也需要对两组时间序列进行合并操作，而不是交集，这个时候我们可以使用 or 集合运算符，产生的结果是运算符左侧的序列，加上来自右侧但左侧没有匹配标签集的时间序列。比如我们要列出所有低于 10 或者高于 30 的请求率，则可以用下面的表达式来查询：


```bash
  rate(demo_api_request_duration_seconds_count{job="demo"}[5m]) < 10
or
  rate(demo_api_request_duration_seconds_count{job="demo"}[5m]) > 30
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0630a114944b4b13a9c52b24f7963727.png)
们可以看到在图中使用值过滤器和集合操作会导致时间序列在图中有断点现象，这取决于他们在图中的时间间隔下是否能够与过滤器进行匹配，所以一般情况下，我们建议只在告警规则中使用这种过滤操作。

还有一个 `unless` 操作符，它只会保留左边的时间序列，如果右边不存在相等的标签集合的话。

**练习：**

1.构建一个查询，显示按 `path`、`method`、`status`（5 分钟内平均）划分的 demo API 请求的第 95 个百分位延迟，除非这个维度组合每秒收到的请求少于 1 个请求（5 分钟内平均）。


```bash
 histogram_quantile(0.95, sum by(path, method, status, le) (rate(demo_api_request_duration_seconds_bucket[5m])))
unless
 sum by(path, method, status) (rate(demo_api_request_duration_seconds_count[5m])) < 1
```
##  14. 排序 [sort(), sort_desc(), topk(), bottomk()]
本节我们将学习如何对查询结果进行排序，或者只选择一组序列中最大或最小的值。

我们可以使用 `sort()`（升序） 或者 `sort_desc()`（降序）函数来实现对输出结果进行排序，例如，要显示按值排序的每个路径请求率，从最高到最低，我们可以用下面的语句进行查询：

```bash
sort_desc(sum by(path) (rate(demo_api_request_duration_seconds_count{job="demo"}[5m])))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a7372b2e99304260b2c063385556ab59.png)
有的时候我们并不是对所有的时间序列感兴趣，只对最大或最小的几个序列感兴趣，我们可以使用 `topk()` 和 `bottomk()` 这两个运算符来操作，可以返回 K 个最大或最小的序列，比如只显示每个 `path` 和 `method` 的前三的请求率，我们可以使用下面的语句来查询。


```bash
topk(3, sum by(path, method) (rate(demo_api_request_duration_seconds_count{job="demo"}[5m])))
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/398ad7c5808f4b46943de3acd9e92e51.png)
**练习：**


1.构建一个查询以升序的方式显示所有 3 个 demo 服务的磁盘使用情况。
```bash
sort(demo_disk_usage_bytes)
```

2.构建一个查询，按 method、path 和 status 维度显示 3 个最低流量的 demo API 请求比率。

```bash
bottomk(3, sum by(method, path, status) (rate(demo_api_request_duration_seconds_count[5m])))
```

##  15. 直方图 [histogram_quantile()]
在这一节中，我们将学习直方图指标，了解如何根据这些指标来计算**分位数**。Prometheus 中的直方图指标允许一个服务记录一系列数值的分布。直方图通常用于**跟踪请求的延迟或响应大小**等指标值，当然理论上它是可以跟踪任何根据某种分布而产生波动数值的大小。Prometheus 直方图是在客户端对数据进行的采样，它们使用的一些可配置的（例如延迟）bucket 桶对观察到的值进行计数，然后将这些 bucket 作为单独的时间序列暴露出来。

下图是一个非累积直方图的例子：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ecd5956e6244144981605d297c38aed.png)
在 Prometheus 内部，直方图被实现为一组时间序列，每个序列代表指定**桶的计数**（例如10ms以下的请求数、25ms以下的请求数、50ms以下的请求数等）。 在 Prometheus 中每个 bucket 桶的计数器是累加的，这意味着较大值的桶也包括所有低数值的桶的计数。在作为直方图一部分的每个时间序列上，相应的桶由特殊的 le 标签表示。le 代表的是小于或等于。

与上面相同的直方图在 Prometheus 中的累积直方图如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9bd22bf6b9a547fc95fcddd2dcb5b710.png)
可以看到在 Prometheus 中直方图的计数是累计的，这是很奇怪的，因为通常情况下非累积的直方图更容易理解。Prometheus 为什么要这么做呢？想象一下，如果直方图指标中加入了额外的标签，或者划分了更多的 bucket，那么样本数据的分析就会变得越来越复杂，如果直方图是累积的，在抓取指标时就可以根据需要丢弃某些 `bucket`，这样可以在降低 Prometheus 维护成本的同时，还可以粗略计算样本值的**分位数**。通过这种方法，用户不需要修改应用代码，便可以动态减少抓取到的样本数量。另外直方图还提供了 `_sum` 指标和 `_count` 指标，所以即使你丢弃了所有的 bucket，仍然可以通过这两个指标值来计算请求的平均响应时间。通过累积直方图的方式，还可以很轻松地计算某个 bucket 的样本数占所有样本数的比例。

我们在演示的 `demo` 服务中暴露了一个直方图指标 `demo_api_request_duration_seconds_bucket`，用于跟踪 API 请求时长的分布，由于这个直方图为每个跟踪的维度导出了 `26` 个 bucket，因此这个指标有很多时间序列。我们可以先来看下来自**一个服务实例的一个请求维度组合的直方图**，查询语句如下所示：


```bash
demo_api_request_duration_seconds_bucket{instance="demo-service-0:10000", method="POST", path="/api/bar", status="200", job="demo"}
```

正常我们可以看到 26 个序列，每个序列代表一个 bucket，由 le 标签标识：
![在这里插入图片描述](https://img-blog.csdnimg.cn/83b03cff14aa4139aed378238e6214a8.png)
直方图可以帮助我们了解这样的问题，比如"**我有多少个请求超过了100ms的时间？**" (当然需要直方图中配置了一个以 100ms 为边界的桶)，又比如"**我99%的请求是在多少延迟下完成的？**"，这类数值被称为**百分位**数或**分位数**。在 Prometheus 中这两个术语几乎是可以通用，只是百分位数指定在 0-100 范围内，而分位数表示在 0 和 1 之间，所以第 99 个百分位数相当于目标分位数 0.99。

如果你的直方图桶粒度足够小，那么我们可以使用 `histogram_quantile(φ scalar, b instant-vector)` 函数用于计算历史数据指标一段时间内的分位数。该函数将目标分位数 `(0 ≤ φ ≤ 1)` 和直方图指标作为输入，就是大家平时讲的 pxx，`p50` 就是中位数，参数 `b` 一定是包含 `le` 这个标签的瞬时向量，不包含就无从计算分位数了，但是计算的分位数是一个预估值，并不完全准确，因为这个函数是假定每个区间内的样本分布是线性分布来计算结果值的，预估的准确度取决于 `bucket` 区间划分的粒度，粒度越大，准确度越低。

回到我们的演示服务，我们可以尝试计算所有维度在所有时间内的第 `90` 个百分位数，也就是 **90% 的请求的持续时间**。

```bash
# BAD!
histogram_quantile(0.9, demo_api_request_duration_seconds_bucket{job="demo"})
```

但是这个查询方式是有一点问题的，当单个服务实例重新启动时，bucket 的 Counter 计数器会被重置，而且我们常常想看看现在的延迟是多少（比如在过去 5 分钟内），而不是整个时间内的指标。我们可以使用 `rate()` 函数应用于底层直方图计数器来实现这一点，该函数会自动处理 Counter 重置，又可以只计算每个桶在指定时间窗口内的平均增长。

我们可以这样去**计算过去 5 分钟内第 90 个百分位数的 API 延迟**：


```bash
# GOOD!
histogram_quantile(0.9, rate(demo_api_request_duration_seconds_bucket{job="demo"}[5m]))
```

这个查询就好很多了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d74a3cdd29f54e81bb5de8a6655e25f9.png)

这个查询会显示每个维度（`job`、`instance`、`path`、`method` 和 `status`）的第 90 个百分点，但是我们可能对单独的这些维度并不感兴趣，想把他们中的一些指标聚合起来，这个时候我们可以在查询的时候使用 Prometheus 的 `sum` 运算符与 `histogram_quantile()` 函数结合起来，**计算出聚合的百分位**，假设在我们想要聚合的维度之间，直方图桶的配置方式相同（桶的数量相同，上限相同），**我们可以将不同维度之间具有相同 le 标签值的桶加在一起，得到一个聚合直方图**。然后，我们可以使用该聚合直方图作为 `histogram_quantile()` 函数的输入。

> 注意：这是假设直方图的桶在你要聚合的所有维度之间的配置是相同的，桶的配置也应该是相对静态的配置，不会一直变化，因为这会破坏你使用 histogram_quantile() 查看的时间范围内的结果。

下面的查询计算了第 90 个百分位数的延迟，但只按 job、instance 和 path 维度进行聚合结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2dedd277401f44ec810c6217106c1e71.png)
 **练习：**

1.构建一个查询，计算在 `0.0001` 秒内完成的 `demo` 服务 API 请求的总百分比，与过去 5 分钟内所有请求总数的平均值。


```bash
sum(rate(demo_api_request_duration_seconds_bucket{le="0.0001"}[5m]))
/
sum(rate(demo_api_request_duration_seconds_bucket{le="+Inf"}[5m])) * 100
```

或者可以使用下面的语句查询


```bash
sum(rate(demo_api_request_duration_seconds_bucket{le="0.0001"}[5m]))
/
 sum(rate(demo_api_request_duration_seconds_count[5m])) * 100
```

2.构建一个查询，计算 demo 服务 API 请求的第 50 个百分位延迟，按 `status code` 和 `method` 进行划分，在过去一分钟的平均值。


```bash
histogram_quantile(0.5, sum by(status, method, le) (rate(demo_api_request_duration_seconds_bucket[1m])))
```

##  16. 数据对比 [offset, unless]
有的时候我们可能需要去访问过去的数据，并和当前数据进行对比。例如，**我们可能想比较今天的请求率和一周前的请求率之间的差异。**我们可以在任何区间向量或瞬时向量选择器上附加一个偏移量 `offset<duration>` 的修饰符（比如 `my_metric offset 5m` 或者 `my_metric[1m] offset 7d`）。

让我们来看一个示例，在我们的 demo 服务中暴露了一个 Counter 指标 `demo_items_shipped_total`，该指标追踪物品的运输情况，用 5 分钟来模拟"每日"流量周期，所以我们不必等待一整天才能查看该时段的数据。

我们只使用第一个演示服务实例来测试即可，首先我们来看看它的速率：


```bash
rate(demo_items_shipped_total{instance="demo-service-0:10000"}[1m])
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/075f0713a9bd437eb5ff776e9ecd8389.png)
该服务还暴露了一个 0 或 1 的布尔指标，告诉我们现在是否是假期：

![在这里插入图片描述](https://img-blog.csdnimg.cn/b8f83da0c0244e9e8f4c69d4e01b22b5.png)
将假期与发货商品率进行比较，注意到节假日时它会减少!我们可以尝试将当前的发货速度与 7"天"（7 * 5 分钟）前的速度进行比较，看看是否有什么不正常的情况。


```bash
  rate(demo_items_shipped_total{instance="demo-service-0:10000"}[1m])
/
  rate(demo_items_shipped_total{instance="demo-service-0:10000"}[1m] offset 35m)
```

通常情况下，该比率约为 1，但当当天或前一天是假期时，我们得到的比率比正常情况下要略低或高。

![在这里插入图片描述](https://img-blog.csdnimg.cn/684fbd39184b4f79bec8f9843a011095.png)
但是，如果原因只是假期，我们想忽略这个较低或较高的比率。我们可以在过去或现在是假期的时候过滤掉这个比率，方法是附加一个 `unless` 集合操作符。


```bash
(
    rate(demo_items_shipped_total{instance="demo-service-0:10000"}[1m])
  /
    rate(demo_items_shipped_total{instance="demo-service-0:10000"}[1m] offset 35m)
)
unless
  (
    demo_is_holiday == 1  # Is it currently a holiday?
  or
    demo_is_holiday offset 35m == 1  # Was it a holiday 7 "days" ago?
  )
```

或者另外一种方法，我们只需要比较今天和一周前是否有相同的节日：


```bash
(
    rate(demo_items_shipped_total{instance="demo-service-0:10000"}[1m])
  /
    rate(demo_items_shipped_total{instance="demo-service-0:10000"}[1m] offset 35m)
)
unless
  (
      demo_is_holiday
    !=
      demo_is_holiday offset 35m
  )
```

这样我们就可以过滤掉当前时间有假期或过去有假期的结果。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7c3d6a0ab24f46bd91a1f65d89c9aeda.png)
 **练习：**

1.构建一个查询，计算每个 `path` 路径的总请求率和 `35` 分钟前的差异。
```bash
 sum by(path) (rate(demo_api_request_duration_seconds_count[5m])) -
  sum by(path) (rate(demo_api_request_duration_seconds_count[5m] offset 35m))
```

##  17. 检测
本节我们将学习如何来检查我们的实例数据抓取健康状况。

### 17.1 检查抓取实例  [up]
每当 `Prometheus` 抓取一个目标时，它都会存储一个合成的样本，其中包含指标名称 `up` 和被抓取实例的 job 和 instance 标签，如果抓取成功，则样本的值被设置为 1，如果抓取失败，则设置为 0，所以我们可以通过如下所示的查询来获取当前哪些实例处于正常或挂掉的状态：


```bash
up{job="demo"}
```

正常三个演示服务实例都处于正常状态，所以应该都为1。如果我们将第一个实例停掉，重新查询则第一个实例结果为0：
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa44d25f4fda424abb727426db0d5db5.png)

如果只希望显示 down 掉的实例，可以通过过滤0值来获取：

![在这里插入图片描述](https://img-blog.csdnimg.cn/5eb6d9ca03904d8fadc12d5bb2afd369.png)
一般情况下这种类型的查询会用于指标抓取健康状态报警。

> 注意：因为 `count()` 是一个聚合运算符，它期望有一组维度的时间序列作为其输入，并且可以根据 `by` 或 `without` 子句将输出序列分组。任何输出组只能基于现有的输入序列，如果根本没有输入序列，就不会产生输出。

###  17.2 检查序列数据 [absent(),absent_over_time()]
在某些情况下，只查看序列的样本值是不够的，有时还需要检测是否存在某些序列，上面我们用 `up{job="demo"} == 0` 语句来查询所有无法抓取的演示服务实例，但是只有已经被抓取的目标才会被加上 `up` 指标，如果 Prometheus 都没有抓取到任何的演示服务目标应该怎么办呢？比如它的抓取配置出问题了，服务发现可能返回也为空，或者由于 Prometheus 自身出了某些问题。

在这种情况下，`absent()` 函数就非常有用了，`absent()` 将一个瞬时向量作为其输入，当输入包含序列时，将返回一个空结果，不包含时将返回单个输出序列，而且样本值为1。

例如，查询语句 `absent(up{job="demo"})` 将得到一个空的输出结果，如果测试一个没有被抓取的 job 是否存在的时候，将得到样本值1。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7add0454764d48d1ba224901bd28295e.png)
这可以帮助我们检测序列是否存在的情况。此外还有一个 `absent()` 的变种，叫做 `absent_over_time()`，它接受一个区间向量，告诉你在该输入向量的整个时间范围内是否有样本。

**练习：**

1.构建一个查询，检测指标 `demo_api_request_duration_seconds_count` 是否具有 `PUT` 的 `method` 标签的序列。


```bash
 absent(demo_api_request_duration_seconds_count{method="PUT"})
```

2.构建一个查询，当过去一小时内任务 `non-existent` 没有记录 up 指标时，该查询输出一个系列。


```bash
absent_over_time(up{job="non-existent"}[1h])
```

转载：

 - [阳明 Prometheus PromQL](https://p8s.io/docs/promql/intro/)

