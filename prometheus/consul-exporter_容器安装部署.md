#  consul-exporter 容器安装部署
tags: exporters
<!-- catalog: ~consul-exporter~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/54c022fdd0fb48abaa50a967d723fe60.png)





## 1. 简介
Consul有多个组件，但总体而言，它是基础架构中的一款服务发现和配置的工具。 它提供了几个关键功能：
服务发现：Consul client 可以提供服务，例如api或mysql，也可以使用Consul client来发现指定服务的提供者。 使用DNS或HTTP，应用程序可以轻松找到他们所依赖的服务。
健康检查：Consul client 可以提供任何数量的健康检查，或者与给定的服务（“Web服务器是否返回200 OK”），或与本地节点（“内存利用率是否低于90％”）相关联。 可以使用此信息来监控集群运行状况，服务发现组件使用此信息将流量从有问题的主机中移除出去。
KV Store：应用程序可以使用Consul的分层键/值存储，包括动态配置，功能标记，协调，leader选举等等。 简单的HTTP API使其易于使用。
多数据中心：Consul支持多个数据中心。 这意味着Consul的用户不必担心构建额外的抽象层以扩展到多个区域

## 2. 部署

```bash
docker run -tid --restart=always  -p 9107:9107 --name consul-expoter prom/consul-exporter:latest --consul.server=192.168.1.190:8500
```

> 注意： The consul_exporter supports all environment variables provided by
> the official consul/api package, including C`ONSUL_HTTP_TOKEN` to set
> the ACL token.

## 3. 进程启动参数

```bash
./consul_exporter --help
consul.allow_stale: Allows any Consul server (non-leader) to service a read.
consul.ca-file: File path to a PEM-encoded certificate authority used to validate the authenticity of a server certificate.
consul.cert-file: File path to a PEM-encoded certificate used with the private key to verify the exporter's authenticity.
consul.health-summary: Collects information about each registered service and exports consul_catalog_service_node_healthy. This requires n+1 Consul API queries to gather all information about each service. Health check information are available via consul_health_service_status as well, but only for services which have a health check configured. Defaults to true.
consul.key-file: File path to a PEM-encoded private key used with the certificate to verify the exporter's authenticity.
consul.require_consistent: Forces the read to be fully consistent.
consul.server: Address (host and port) of the Consul instance we should connect to. This could be a local agent (localhost:8500, for instance), or the address of a Consul server.
consul.server-name: When provided, this overrides the hostname for the TLS certificate. It can be used to ensure that the certificate name matches the hostname we declare.
consul.timeout: Timeout on HTTP requests to consul.
log.format: Set the log target and format. Example: logger:syslog?appname=bob&local=7 or logger:stdout?json=true
log.level: Logging level. info by default.
version: Show application version.
web.listen-address: Address to listen on for web interface and telemetry.
web.telemetry-path: Path under which to expose metrics.
```
## 4. metrics说明

```bash
Metric                            Meaning Labels
consul_up                         Was the last query of Consul successful 
consul_raft_peers                 How many peers (servers) are in the Raft cluster 
consul_serf_lan_members           How many members are in the cluster 
consul_catalog_services           How many services are in the cluster 
consul_catalog_service_node_healthy Is this service healthy on this node service, node
consul_health_node_status         Status of health checks associated with a node check, node, status
consul_health_service_status      Status of health checks associated with a service check, node, service, status
consul_catalog_kv                 The values for selected keys in Consul's key/value catalog. Keys with non-numeric values are omitted key
```

