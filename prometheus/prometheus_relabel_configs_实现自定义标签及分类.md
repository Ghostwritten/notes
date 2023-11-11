## 1. 介绍
配`relabel_configs` 的功能， Prometheus 允许用户在采集任务设置中，通过 relabel_configs 来添加自定义的 Relabeling 的额过程，来对标签进行指定规则的重写。 Prometheus 加载 Targets 后，这些 Targets 会自动包含一些默认的标签，Target 以 __ 作为前置的标签是在系统内部使用的，这些标签不会被写入到样本数据中。眼尖的会发现，每次增加 Target 时会自动增加一个 instance 标签，而 instance 标签的内容刚好对应 Target 实例的 __address__ 值，这是因为实际上 Prometheus 内部做了一次标签重写处理，默认 __address__ 标签设置为 <host>:<port> 地址，经过标签重写后，默认会自动将该值设置为 instance 标签，所以我们能够在页面看到该标签。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210128190317823.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
 详细 relabel_configs 配置及说明可以参考  [relabel_config 官网说明](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config)，这里我简单列举一下里面每个 `relabel_action` 的作用，方便下边演示。

 - `replace`: 根据 `regex` 的配置匹配 `source_labels` 标签的值（注意：多个 source_label 的值会按照`separator` 进行拼接），并且将匹配到的值写入到 `target_label` 当中，如果有多个匹配组，则可以使用 ${1}, ${2}确定写入的内容。如果没匹配到任何内容则不对 target_label 进行重新， 默认为 `replace`。
 - `keep`: 丢弃 source_labels 的值中没有匹配到 regex 正则表达式内容的 Target 实例
 - `drop`: 丢弃 source_labels 的值中匹配到 regex 正则表达式内容的 Target 实例
 - `hashmod`:  将 target_label 设置为关联的 source_label 的哈希模块
 - `labelmap`: 根据 regex 去匹配 Target 实例所有标签的名称（注意是名称），并且将捕获到的内容作为为新的标签名称，regex 匹配到标签的的值作为新标签的值
 - `labeldrop`:  对 Target 标签进行过滤，会移除匹配过滤条件的所有标签
 - `labelkeep`: 对 Target 标签进行过滤，会移除不匹配过滤条件的所有标签
## 2. 示例
### 2.1 kubernetes-nodes
```bash
- job_name: 'kubernetes-nodes'

  kubernetes_sd_configs:
  - role: node
```
输出
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210128194444684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
端口不对，修改端口，如何修改，通过利用`source_labels`匹配[__address__]的值，利用regex正则表达式匹配端口，然后利用`replacement`想要的值，最后利用`target_label`把新的值用想要的label表示。

```bash
- job_name: 'kubernetes-nodes'

  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - source_labels: [__address__]
    regex: '(.*):10250'
    replacement: '${1}:9100'
    target_label: __address__
    action: replace
```
![ZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)](https://img-blog.csdnimg.cn/20210128194856509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

9100端口变过来了，labels少，如何变多，利用`labelmap`功能

```bash
- job_name: 'kubernetes-nodes'

  kubernetes_sd_configs:
  - role: node
  relabel_configs:
  - source_labels: [__address__]
    regex: '(.*):10250'
    replacement: '${1}:9100'
    target_label: __address__
    action: replace
  - action: labelmap
    regex: __meta_kubernetes_node_label_(.+)
```
如下label变多了
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021012819505922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 2.2 kubernetes-service-endpoints

```bash
- job_name: 'kubernetes-service-endpoints'

  kubernetes_sd_configs:
  - role: endpoints
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210128200628590.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

发现有一些endpoints并不是我们想要的，通过label可能匹配一类有共同特点并且是我们想要展示的endpoints。我们利用`source_labels`选出一个label，并通过`regex`过滤我们想要的值（true），然后通过`keep`舍弃没有这个label并且等于这个label却值不等于true的endpoints。

```bash
  kubernetes_sd_configs:
  - role: endpoints
  relabel_configs:
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
    action: keep
    regex: true
```

输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210128201253395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
发现endpoints数量立马变少了。
当然获取有的endpoint的采集并非是http，而是https，所有，我们需要一个支持http与https的办法，利用regex正则匹配http与https两种情况，并且利用source_labels、replace、target_label组合将相对长的label变短。

```bash
- job_name: 'kubernetes-service-endpoints'

  kubernetes_sd_configs:
  - role: endpoints
  relabel_configs:
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
    action: keep
    regex: true
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
    action: replace
    target_label: __scheme__
    regex: (https?)
```
然而我的集群中并没有https的endpoints，暂时图略。
当然，metrics的路径也有可能各不相同，因此，采用正则匹配路径的各种形式。

```bash
- job_name: 'kubernetes-service-endpoints'

  kubernetes_sd_configs:
  - role: endpoints
  relabel_configs:
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
    action: keep
    regex: true
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
    action: replace
    target_label: __scheme__
    regex: (https?)
  - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
    action: replace
    target_label: __metrics_path__
    regex: (.+)
```
然而这并没有结束，因为有的endpoints是不通的，我们需要去除多余的。如图。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129163549395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)





![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129163408548.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

labmap可以显示更多标签，source_labels、replace、target_label组合方法可以修改较长的label。
```bash
- job_name: 'kubernetes-service-endpoints'
    
      kubernetes_sd_configs:
      - role: endpoints
    
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131155755523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2.3 kube-state-metrics

```bash
- job_name: 'kube-state-metrics'
  kubernetes_sd_configs:
  - role: endpoints
  relabel_configs:
  - source_labels: [__address__]
    regex: '(.*):10250'
    replacement: '${1}:8080'
    target_label: __address__
    action: replace
  - source_labels: [__meta_kubernetes_service_name]
    action: keep
    regex: 'kube-state-metrics'
```
输出
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131163725507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

