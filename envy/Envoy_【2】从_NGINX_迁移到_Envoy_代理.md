

## 1. 简介
此场景旨在支持您从 NGINX 迁移到 Envoy 代理。它将帮助您将以前对 NGINX 的任何经验和理解应用到 Envoy。

你将学到如何：

 - 配置 `Envoy` 代理服务器配置和设置。
 - 配置 `Envoy` 代理以将流量代理到外部服务。
 - 配置访问和错误日​​志。

在场景结束时，您将了解核心 Envoy 代理功能，以及如何将现有 NGINX 脚本迁移到平台。

##  2. NGINX 示例
这个场景使用了一个基于NGINX Wiki的完整示例精心制作的`nginx.conf`。您可以通过打开在编辑器中查看配置`nginx.conf`

```bash
user  www www;
pid /var/run/nginx.pid;
worker_processes  2;

events {
  worker_connections   2000;
}

http {
  gzip on;
  gzip_min_length  1100;
  gzip_buffers     4 8k;
  gzip_types       text/plain;

  log_format main      '$remote_addr - $remote_user [$time_local]  '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$gzip_ratio"';

  log_format download  '$remote_addr - $remote_user [$time_local]  '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$http_range" "$sent_http_content_range"';


  upstream targetCluster {
    172.18.0.3:80;
    172.18.0.4:80;
  }

  server {
    listen        8080;
    server_name   one.example.com  www.one.example.com;

    access_log   /var/log/nginx.access_log  main;
    error_log  /var/log/nginx.error_log  info;

    location / {
      proxy_pass         http://targetCluster/;
      proxy_redirect     off;

      proxy_set_header   Host             $host;
      proxy_set_header   X-Real-IP        $remote_addr;
    }
  }
}
```
NGINX 配置通常具有三个关键要素：

 - 1 配置 NGINX 服务器、日志结构和 `Gzip` 功能。这是在所有实例中全局定义的。
 - 2 配置 NGINX 以在端口 8080 上接受`one.example.com`主机的请求。
 - 3 配置目标位置如何处理到URL不同部分的流量。

并非所有配置都适用于 Envoy 代理，您不需要配置某些方面。Envoy Proxy 有四种关键类型，支持 NGINX 提供的核心基础设施。核心是：

 - `Listeners`: 它们定义了 Envoy 代理如何接受传入的请求。目前，`Envoy Proxy` 仅支持基于 TCP的侦听器。建立连接后，它会传递给一组过滤器进行处理。
 - `Filters`:它们是可以处理入站和出站数据的管道架构的一部分。此功能启用过滤器，例如 Gzip，它在将数据发送到客户端之前压缩数据。
 - `Routers`:这些将流量转发到所需的目的地，定义为集群。
 - `Clusters`:它们定义流量的目标端点和配置设置。

我们将使用这四个组件来创建 Envoy 代理配置以匹配定义的 NGINX 配置。Envoy 一直专注于 API 和动态配置。在这种情况下，配置将使用 NGINX 定义的静态硬编码资源。


##  3. envy与nginx 配置对比
第一部分`nginx.conf`定义了一些应该配置的 NGINX 内部结构。

###  3.1 Worker Connections
下面的配置侧重于定义工作进程和连接的数量。这表明 NGINX 将如何扩展以处理需求。

```bash
worker_processes  2;

events {
  worker_connections   2000;
}
```
Envoy Proxy 以不同的方式管理工作进程和连接。

Envoy 为系统中的每个硬件线程生成一个工作线程。每个工作线程运行一个非阻塞事件循环，负责

 1. 倾听每一位听众（listener）
 2. 接受新连接
 3. 实例化连接的过滤器堆栈
 4. 处理连接生命周期内的所有 IO。

连接的所有进一步处理都完全在工作线程内处理，包括任何转发行为。

Envoy 中的所有连接池（pool）都是每个工作线程。尽管 HTTP/2 连接池一次只与每个上游主机建立一个连接，但如果有四个工作人员，则每个上游主机在稳定状态下将有四个 HTTP/2 连接。通过将所有内容保留在单个工作线程中，几乎所有代码都可以在没有锁的情况下编写，就像是单线程一样。拥有过多的工作人员会浪费内存，创建更多的空闲连接，并导致连接池命中率较低。

有关更多信息，请访问[Envoy 代理博客](https://blog.envoyproxy.io/envoy-threading-model-a8d44b922310?gi=20c704e902f6)。


###  3.2 HTTP Configuration
NGINX 配置的下一个块定义了 HTTP 设置，例如：

 - 支持哪些 `mime` 类型
 - 默认超时
 - Gzip 配置

您可以通过 Envoy Proxy 中的过滤器配置这些方面

> [nginx中的MIME.types的作用](https://blog.csdn.net/Debug_zhang/article/details/50749646)


###  3.3 Server Configuration
在 HTTP 配置块中，NGINX 配置指定侦听端口 8080 并响应域`one.example.com`和`www.one.example.com` 的传入请求。

```bash
 server {
    listen        80;
    server_name   one.example.com  www.one.example.com;
```
在 Envoy 中，这由 `Listeners` 管理。

###  3.4 Envoy Listeners
从 Envoy Proxy 开始，最重要的方面是定义列表器 （listers）。您需要创建一个配置文件来描述您希望如何运行 Envoy 实例。

下面的代码片段将创建一个新的侦听器并将其绑定到端口 8080。该配置向 Envoy 代理指示它应该为传入请求绑定到哪些端口。

Envoy Proxy 使用 YAML 表示法进行配置。如果您不熟悉此表示法，可以查看此[链接](https://yaml.org/spec/1.2.2/)。

```bash
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
```
无需定义`server_name`，因为 Envoy 代理过滤器将处理此问题。

###  3.5 Location Configuration
当请求进入 NGINX 时，位置块定义了如何处理以及将流量转发到哪里。在以下代码段中，该站点的所有流量都被代理到名为 的上游集群`targetCluster`。上游集群定义了应该处理请求的节点。我们将在下一步中讨论这个问题。

```bash
location / {
    proxy_pass         http://targetCluster/;
    proxy_redirect     off;

    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
}
```
在 Envoy 中，这由过滤器管理。

###  3.6 Envoy Filters
对于静态配置，过滤器（filters）定义了如何处理传入请求。在这种情况下，我们正在设置与上一步中的`server_names`匹配的过滤器。当收到与定义的域和路由匹配的传入请求时，流量将转发到集群。这相当于上游 NGINX 配置。

```bash
filter_chains:
- filters:
  - name: envoy.http_connection_manager
    config:
      codec_type: auto
      stat_prefix: ingress_http
      route_config:
        name: local_route
        virtual_hosts:
        - name: backend
          domains:
            - "one.example.com"
            - "www.one.example.com"
          routes:
          - match:
              prefix: "/"
            route:
              cluster: targetCluster
      http_filters:
      - name: envoy.router
```
名称`envoy.http_connection_manager`是 Envoy Proxy 中的内置过滤器。其他过滤器包括`Redis`、`Mongo`、`TCP`。您可以在文档中找到完整列表。

有关其他负载平衡策略的更多信息，请访问[Envoy 文档](https://www.envoyproxy.io/docs/envoy/v1.8.0/intro/arch_overview/load_balancing)。

###  3.7 Proxy and Upstream Configuration
在 NGINX 中，上游配置定义了一组将处理流量的目标服务器。在这种情况下，已经分配了两个集群。

```bash
  upstream targetCluster {
    172.18.0.3:80;
    172.18.0.4:80;
  }
```
在 Envoy 中，这是由集群管理的。

###  3.8 Envoy Clusters
上游的等价物被定义为集群。在这种情况下，已经定义了将为流量提供服务的主机。访问主机的方式（例如超时）被定义为集群配置。这允许对超时和负载平衡等方面进行更精细的控制。

```bash
 clusters:
  - name: targetCluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    hosts: [
      { socket_address: { address: 172.18.0.3, port_value: 80 }},
      { socket_address: { address: 172.18.0.4, port_value: 80 }}
    ]
```
当使用`STRICT_DNS`服务发现时，Envoy 将持续异步地解析指定的 DNS 目标。DNS 结果中返回的每个 IP 地址都将被视为上游集群中的显式主机。这意味着如果查询返回两个 IP 地址，Envoy 将假设集群有两个主机，并且都应该进行负载均衡。如果主机从结果中删除，Envoy 会假设它不再存在，并将从任何现有连接池中排出流量。

有关更多信息，请参阅[Envoy 代理文档](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/service_discovery#strict-dns)。

###  3.9 Logging Access and Errors
最后的配置是日志记录。Envoy Proxy 没有将错误日志通过管道传输到磁盘，而是遵循云原生方法。所有应用程序日志都输出到`stdout`和`stderr`。

当用户提出请求时，访问日志是可选的，默认情况下是禁用的。要启用 HTTP 请求的访问日志，包括`access_log`为HTTP 连接管理器的配置。路径可以是设备（例如stdout）或磁盘上的文件，具体取决于您的要求。

以下配置会将所有访问日志通过管道传输到stdout. 将代码片段复制到连接管理器的配置部分：

```bash
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
```
默认情况下，Envoy 有一个格式字符串，其中包含 HTTP 请求的详细信息：

```bash
- name: envoy.http_connection_manager
  config:
    codec_type: auto
    stat_prefix: ingress_http
    access_log:
    - name: envoy.file_access_log
      config:
        path: "/dev/stdout"
    route_config:
```
此格式字符串的结果是：

```bash
[2018-11-23T04:51:00.281Z] "GET / HTTP/1.1" 200 - 0 58 4 1 "-" "curl/7.47.0" "f21ebd42-6770-4aa5-88d4-e56118165a7d" "one.example.com" "172.18.0.4:80"
```
可以通过设置格式字段来自定义输出的内容。例如：

```bash
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
    format: "[%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%" %RESPONSE_CODE% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n"
```
通过设置json_format字段，日志行也可以输出为 JSON 。例如：

```bash
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
    json_format: {"protocol": "%PROTOCOL%", "duration": "%DURATION%", "request_method": "%REQ(:METHOD)%"}
```
有关 Envoy 日志记录方法的更多信息，请访问[https://www.envoyproxy.io/docs/envoy/latest/configuration/access_log#config-access-log-format-dictionaries](https://www.envoyproxy.io/docs/envoy/latest/configuration/access_log#config-access-log-format-dictionaries)

日志记录并不是使用 Envoy Proxy 获得生产可见性的唯一方法。它内置了高级跟踪和度量功能。您可以在[跟踪文档](https://www.envoyproxy.io/docs/envoy/latest/)中或通过[交互式跟踪方案](https://www.envoyproxy.io/try/implementing-metrics-tracing)找到更多信息。

##  4. 启动
您现在已将配置从 NGINX 转换为 Envoy 代理。最后一步是启动 Envoy Proxy 实例来测试它。

###  4.1 以用户身份运行
在 NGINX 配置的顶部，该行`user  www www;`指示以低权限用户身份运行 NGINX 以提高安全性。

Envoy Proxy 采用云原生方法来管理流程所有者。当我们通过容器启动 Envoy 代理时，我们可以指定一个低权限用户。

###  4.2 启动 Envoy 代理
下面的命令将通过主机上的 Docker 容器启动 Envoy 代理。该命令公开 Envoy 以侦听端口 80 上的传入请求。但是，正如侦听器配置中指定的那样，Envoy 代理正在侦听端口 8080 上的传入流量。这允许进程以低特权用户身份运行。

```bash
$ docker run --name proxy1 -p 80:8080 --user 1000:1000 -v /root/envoy.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy
[2021-11-15 08:27:31.097][000011][info][main] [source/server/server.cc:202] initializing epoch 0 (hot restart version=10.200.16384.127.options=capacity=16384, num_slots=8209 hash=228984379728933363 size=2654312)
[2021-11-15 08:27:31.116][000011][info][main] [source/server/server.cc:204] statically linked extensions:
[2021-11-15 08:27:31.116][000011][info][main] [source/server/server.cc:206]   access_loggers: envoy.file_access_log,envoy.http_grpc_access_log
[2021-11-15 08:27:31.116][000011][info][main] [source/server/server.cc:209]   filters.http: envoy.buffer,envoy.cors,envoy.ext_authz,envoy.fault,envoy.filters.http.header_to_metadata,envoy.filters.http.jwt_authn,envoy.filters.http.rbac,envoy.grpc_http1_bridge,envoy.grpc_json_transcoder,envoy.grpc_web,envoy.gzip,envoy.health_check,envoy.http_dynamo_filter,envoy.ip_tagging,envoy.lua,envoy.rate_limit,envoy.router,envoy.squash
[2021-11-15 08:27:31.116][000011][info][main] [source/server/server.cc:212]   filters.listener: envoy.listener.original_dst,envoy.listener.proxy_protocol,envoy.listener.tls_inspector
[2021-11-15 08:27:31.116][000011][info][main] [source/server/server.cc:215]   filters.network: envoy.client_ssl_auth,envoy.echo,envoy.ext_authz,envoy.filters.network.rbac,envoy.filters.network.sni_cluster,envoy.filters.network.thrift_proxy,envoy.http_connection_manager,envoy.mongo_proxy,envoy.ratelimit,envoy.redis_proxy,envoy.tcp_proxy
[2021-11-15 08:27:31.116][000011][info][main] [source/server/server.cc:217]   stat_sinks: envoy.dog_statsd,envoy.metrics_service,envoy.stat_sinks.hystrix,envoy.statsd
[2021-11-15 08:27:31.116][000011][info][main] [source/server/server.cc:219]   tracers: envoy.dynamic.ot,envoy.lightstep,envoy.zipkin
[2021-11-15 08:27:31.117][000011][info][main] [source/server/server.cc:222]   transport_sockets.downstream: envoy.transport_sockets.alts,envoy.transport_sockets.capture,raw_buffer,tls
[2021-11-15 08:27:31.117][000011][info][main] [source/server/server.cc:225]   transport_sockets.upstream: envoy.transport_sockets.alts,envoy.transport_sockets.capture,raw_buffer,tls
[2021-11-15 08:27:31.121][000011][critical][main] [source/server/server.cc:80] error initializing configuration '/etc/envoy/envoy.yaml': Unable to parse JSON as proto (INVALID_ARGUMENT:filter_chains: Cannot find field.): {"filter_chains":[{"filters":[{"config":{"http_filters":[{"name":"envoy.router"}],"route_config":{"virtual_hosts":[{"routes":[{"route":{"cluster":"targetCluster"},"match":{"prefix":"/"}}],"domains":["one.example.com","www.one.example.com"],"name":"backend"}],"name":"local_route"},"stat_prefix":"ingress_http","codec_type":"auto"},"name":"envoy.http_connection_manager"}]}]}
```
###  4.3 测试
启动代理后，现在可以进行和处理测试。以下 cURL 命令使用代理配置中定义的主机标头发出请求。

```bash
$ curl -H "Host: one.example.com" localhost -i
HTTP/1.1 503 Service Unavailable
content-length: 57
content-type: text/plain
date: Mon, 15 Nov 2021 08:34:48 GMT
server: envoy

```
HTTP 请求将导致503错误。这是因为上游连接未运行且不可用。因此，Envoy 代理没有可用于请求的目标目的地。以下命令将启动一系列与为 Envoy 定义的配置相匹配的 HTTP 服务。

```bash
$ docker run -d katacoda/docker-http-server; docker run -d katacoda/docker-http-server;
Unable to find image 'katacoda/docker-http-server:latest' locally
latest: Pulling from katacoda/docker-http-server
f139eb4721ae: Pull complete 
Digest: sha256:76dc8a47fd019f80f2a3163aba789faf55b41b2fb06397653610c754cb12d3ee
Status: Downloaded newer image for katacoda/docker-http-server:latest
8a94122a8b00f3ba11112f8e454d0d82eea73269ca1a32766c8ef13452032c85
f7b79df5ff888e921fd6bbf1353a3a62cdd559e77d335a867620d783a29af94b
```
有了可用的服务，Envoy 就可以成功地将流量代理到目标目的地

```bash
$ curl -H "Host: one.example.com" localhost -i
HTTP/1.1 200 OK
date: Mon, 15 Nov 2021 08:35:17 GMT
content-length: 58
content-type: text/html; charset=utf-8
x-envoy-upstream-service-time: 0
server: envoy

<h1>This request was processed by host: 0cf13596f52e</h1>
```
您应该会看到一个响应，指示哪个 docker 容器处理了请求。在 Envoy Proxy 日志中，您还应该看到输出的访问行。

###  4.4 额外的 HTTP 响应头
在有效请求的响应标头中，您将看到额外的 HTTP 标头。标头显示上游主机处理请求所花费的时间。它以毫秒表示。如果客户端想要确定与网络延迟相比的服务时间，这将非常有用。

```bash
x-envoy-upstream-service-time: 0
server: envoy
```

