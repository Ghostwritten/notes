#  cadvisor 容器安装部署
tags: exporters
<!-- catalog: ~cadvisor~ -->



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d8349b028ad5372c2d381fcad02735b0.png)


##  1. 介绍
[cAdvisor](https://github.com/google/cadvisor) 是谷歌公司用来分析运行中的Docker容器的资源占用以及性能特性的工具.

cAdvisor部署为一个运行中的daemon，它会收集、聚集、处理并导出运行中容器的信息。这些信息能够包含容器级别的资源隔离参数、资源的历史使用状况、反映资源使用和网络统计数据完整历史状况。对docker的监控能力非常强大。

cAdvior功能已经被集成到了`kubelet`组件中，也就是说，安装好kubernetes后，cAdvisor就已经安装到了每一个计算节点上。

##  2. 原理
使用[Grafana](https://grafana.com/docs/grafana/latest/?utm_source=grafana_gettingstarted)、[Prometheus](https://prometheus.io/docs/introduction/first_steps/) 和 [cAdvisor](https://github.com/google/cadvisor)来建立仪表板监控系统。仪表板用于研究服务器实例、它们的正常运行时间、异常、错误和上下文场景等。

所有数据都显示在自定义 Grafana 仪表板上，从仪表板触发查询，该仪表板命中 Prometheus，该仪表板作为数据源插入 Grafana。

容器信息从 cAdvisor 流入 Prometheus。

下图显示了这些开源工具之间的数据流，一种基于 Grafana、Prometheus、cAdvisor 的仪表板监控架构流。有关如何[部署 Grafana 的更多信息，请阅读此处](https://scaleyourapp.com/what-is-grafana-why-use-it-everything-you-should-know-about-it/)。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3df83f51d2cd60d91a6cc3c0003d9c04.png)



## 3. 部署

###  3.1 容器部署
#存储对接influxdb
```bash
#存储对接influxdb
$ docker run -v /:/rootfs:ro -v /var/run:/var/run:rw -v /sys:/sys:ro -v /var/lib/docker/:/var/lib/docker:ro -p 8083:8080 -p 8082:8082 --detach=true  --name cadvisor google/cadvisor:canary -storage_driver=influxdb -storage_driver_index="cadvisor" -storage_driver_host=influxsrv:8086


$ docker run \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:ro \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/dev/disk/:/dev/disk:ro \
--publish=9101:8080 \
--detach=true \
--name=cadvisor \
--restart=always \
local.harbor.io/google-containers/cadvisor:latest
```
**注意**:
服务起不来如何解决？

```bash
$ mount -o remount,rw '/sys/fs/cgroup'
$ ln -s /sys/fs/cgroup/cpu,cpuacct /sys/fs/cgroup/cpuacct,cpu
```
它也可以与 [Canary](https://hub.docker.com/r/google/cadvisor-canary/) 一起运行。以下是详细信息。

此外，该工具是一个没有外部依赖的静态 Go 二进制文件。如果我们愿意，我们可以独立运行它。运行时行为可以通过[一系列标志](https://github.com/google/cadvisor/blob/master/docs/runtime_options.md)来控制。

cAdvisor 还定期执行一些内务管理。使用这些标志，我们可以控制工具执行任务的方式和时间。


###  3.2  docker-compose 部署
创建一个`docker-compose.yml`文件
```bash
version: '3.2'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    volumes:
    - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
    - cadvisor
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
    - redis
  redis:
    image: redis:latest
    container_name: redis
    ports:
    - 6379:6379
```
此配置指示 Docker Compose 运行三个服务，每个服务对应一个Docker容器：

 - 该`prometheus`服务使用本地`prometheus.yml`配置文件（通过参数导入到容器中volumes）。
 - 该`cadvisor`服务公开端口 8080（cAdvisor 指标的默认端口）并依赖于各种本地卷（/、、/var/run等）。
 - 该`redis`服务是一个标准的 Redis 服务器。cAdvisor 将自动从该容器收集容器指标，即无需任何进一步配置。

运行安装：

```bash
$ docker-compose up

#查看容器状态
$ docker-compose ps
 Name                 Command               State           Ports
----------------------------------------------------------------------------
cadvisor     /usr/bin/cadvisor -logtostderr   Up      8080/tcp
prometheus   /bin/prometheus --config.f ...   Up      0.0.0.0:9090->9090/tcp
redis        docker-entrypoint.sh redis ...   Up      0.0.0.0:6379->6379/tcp
```

##  4. 探索 cAdvisor Web UI
您可以访问 [cAdvisor Web UI](https://github.com/google/cadvisor/blob/master/docs/web.md) ，网址为`http://localhost:8080`。您可以在我们的安装中探索特定 Docker 容器的统计信息和图表`http://localhost:8080/docker/<container>`。例如，Redis 容器的指标可以在`http://localhost:8080/docker/redis`、Prometheus：`http://localhost:8080/docker/prometheus`等处访问。cAdvisor 收集的数据可以在其端口公开的基于 Web 的 UI的帮助下进行查看。

它还通过[REST API公开其数据](https://github.com/google/cadvisor/blob/master/docs/api.md)

通常，返回的容器信息是容器名称、子容器列表、容器规格、容器最近 N 秒的详细使用统计、资源使用直方图等。

返回的机器信息是可调度逻辑核心 CPU 的数量、内存容量（以字节为单位）、支持的最大 CPU 频率、可用文件系统、网络设备、机器拓扑 - 节点、核心、线程等。




## 5. promethues 配置 cadvisor

```bash
  - job_name: "cadvisor"
    static_configs:
    - targets:
      - '192.168.1.120:9101'
```
或者：

```bash
scrape_configs:
- job_name: cadvisor
  scrape_interval: 5s
  static_configs:
  - targets:
    - cadvisor:8080
```
##  6. cadvisor metrics 
`container_start_time_seconds`指标开始，该指标记录容器的启动时间（以秒为单位）。`name="<container_name>"`您可以使用表达式按名称选择特定容器。容器名称对应`container_name`，Docker Compose 配置中的参数。`container_start_time_seconds{name="redis"}`例如，表达式显示容器的开始时间redis。

cadvisor metrics 列表：
```bash
process_virtual_memory_bytes 1.42579712e+09
process_start_time_seconds 1.66521371663e+09
process_resident_memory_bytes 6.3102976e+07
process_open_fds 14
process_max_fds 1.048576e+06
process_cpu_seconds_total 53.33
machine_memory_bytes 8.181809152e+09
machine_cpu_cores 4
go_threads 19
go_memstats_sys_bytes 7.2349944e+07
go_memstats_stack_sys_bytes 1.671168e+06
go_memstats_stack_inuse_bytes 1.671168e+06
go_memstats_other_sys_bytes 727622
go_memstats_next_gc_bytes 2.8742496e+07
go_memstats_mspan_sys_bytes 458752
go_memstats_mspan_inuse_bytes 423016
go_memstats_mcache_sys_bytes 16384
go_memstats_mcache_inuse_bytes 6912
go_memstats_mallocs_total 5.109018e+06
go_memstats_lookups_total 0
go_memstats_last_gc_time_seconds 1.6652145196693025e+09
go_memstats_heap_sys_bytes 6.5437696e+07
go_memstats_heap_released_bytes 0
go_memstats_heap_objects 182977
go_memstats_heap_inuse_bytes 2.0832256e+07
go_memstats_heap_idle_bytes 4.460544e+07
go_memstats_heap_alloc_bytes 1.5082544e+07
go_memstats_gc_sys_bytes 2.482176e+06
go_memstats_gc_cpu_fraction 0.0004565680065549785
go_memstats_frees_total 4.926041e+06
go_memstats_buck_hash_sys_bytes 1.556146e+06
go_memstats_alloc_bytes_total 9.62135304e+08
go_memstats_alloc_bytes 1.5082544e+07
go_goroutines 91
go_gc_duration_seconds_sum 0.097979934
go_gc_duration_seconds_count 115
go_gc_duration_seconds
container_tasks_state
container_start_time_seconds
container_spec_memory_swap_limit_bytes
container_spec_memory_reservation_limit_bytes
container_spec_memory_limit_bytes
container_spec_cpu_shares
container_spec_cpu_period
container_scrape_error 0
container_network_transmit_packets_total
container_network_transmit_packets_dropped_total
container_network_transmit_errors_total
container_network_transmit_bytes_total
container_network_receive_packets_total
container_network_receive_packets_dropped_total
container_network_receive_errors_total
container_network_receive_bytes_total
container_memory_working_set_bytes
container_memory_usage_bytes
container_memory_swap
container_memory_rss
container_memory_max_usage_bytes
container_memory_mapped_file
container_memory_failures_total
container_memory_failcnt
container_memory_cache
container_last_seen
container_fs_writes_total
container_fs_writes_merged_total
container_fs_write_seconds_total
container_fs_writes_bytes_total
container_fs_usage_bytes
container_fs_sector_writes_total
container_fs_sector_reads_total
container_fs_reads_total
container_fs_reads_merged_total
container_fs_read_seconds_total
container_fs_reads_bytes_total
container_fs_limit_bytes
container_fs_io_time_weighted_seconds_total
container_fs_io_time_seconds_total
container_fs_io_current
container_fs_inodes_total
container_fs_inodes_free
container_cpu_user_seconds_total
container_cpu_usage_seconds_total
container_cpu_system_seconds_total
container_cpu_load_average_10s
cadvisor_version_info

```


promSQL表达式
|Expression|	Description	|For
|--|--|--|
|rate(container_cpu_usage_seconds_total{name="redis"}[1m])	|cgroup在最后一分钟的 CPU使用率	|redis容器_
|container_memory_usage_bytes{name="redis"}	|cgroup 的总内存使用量（以字节为单位）|	redis容器_
|rate(container_network_transmit_bytes_total[1m])	|容器在最后一分钟每秒通过网络传输的字节数|	所有容器
|rate(container_network_receive_bytes_total[1m])	|容器在最后一分钟每秒通过网络接收的字节数	|所有容器

参考：

 - [https://github.com/google/cadvisor](https://github.com/google/cadvisor)
 - [What is cAdvisor? How Does it Work? Explained…](https://scaleyourapp.com/what-is-cadvisor-how-does-it-work-explained/)
 - [MONITORING DOCKER CONTAINER METRICS USING CADVISOR](https://prometheus.io/docs/guides/cadvisor/)

