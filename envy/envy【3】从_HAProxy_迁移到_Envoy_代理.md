
## 1.简介
这个场景旨在支持你从 HAProxy 迁移到 Envoy Proxy。它将帮助您将以前对 HAProxy 的任何经验和理解应用到 Envoy。

你将学到如何：

 - 配置 `Envoy` 代理服务器配置和设置。
 - 配置 `Envoy` 代理以将流量代理到外部服务。
 - 配置访问和错误日​​志。

在场景结束时，您将了解核心 Envoy Proxy 功能，以及如何将现有 HAProxy 脚本迁移到平台。

##  2. HAProxy 示例
可以通过打开在编辑器中查看示例 HA 代理配置`haproxy.cfg`。

```bash
global
        log /dev/log    local0
        log /dev/log    local1 notice
        user haproxy
        group haproxy
        daemon

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull

frontend localnodes
    bind *:8080
    mode http
    default_backend nodes

backend nodes
    mode http
    balance roundrobin
    option forwardfor
    server web01 172.18.0.3:80 check
    server web02 172.18.0.4:80 check
```

HA 代理配置通常有四个关键要素：

 - 配置 HA 代理服务器、日志结构和进程范围的安全性和性能调整，这些调整会在低级别影响 HAProxy。这是在所有实例中全局定义的。请参阅中的global部分`haproxy.cfg`。
 - 配置默认设置适用于它之后的所有`frontend`和`backend`部分。请参阅 中的defaults部分`haproxy.cfg`。
 - 配置 HA 代理以接受传入请求。本节定义客户端可以连接到的 IP 地址和端口。请参阅 中的frontend部分`haproxy.cfg`。
 - 配置如何处理流量的位置。它定义了一组将被负载平衡并分配来处理请求的服务器。请参阅 中的backend部分`haproxy.cfg`。

并非所有配置都适用于 Envoy 代理，由于不同的架构和决策，某些方面不需要。`Envoy Proxy` 有四种关键类型，支持 `HA Proxy` 提供的核心基础设施。核心是：

 - `Listeners`: 它们定义了 Envoy 代理如何接受传入的请求。目前，Envoy Proxy 仅支持基于 TCP的侦听器。建立连接后，它会传递给一组过滤器进行处理。
 - `Filters`：它们是可以处理入站和出站数据的管道架构的一部分。此功能启用过滤器，例如 Gzip，它在将数据发送到客户端之前压缩数据。
 - `Routers`：它们将流量转发到所需的目的地，定义为集群。
 - `Clusters`：它们定义流量的目标端点和配置设置。

我们将使用这四个组件来创建 Envoy 代理配置以匹配定义的 HA 代理配置。Envoy 一直专注于 API 和动态配置。在这种情况下，配置将使用 HA 代理定义的静态、硬编码资源。

##  3. Frontend Configuration
在 HTTP 配置块中，HA 代理配置侦听端口 `8080`，所有流量都由后端节点处理。

```bash
frontend localnodes
    bind *:8080
    mode http
    default_backend nodes
```
在 Envoy Proxy 中，这个概念由 `Listeners` 处理。
###  3.1 Envoy Listeners
配置的 Envoy 绑定被定义为 `Listeners`。每个侦听器都可以定义一个端口和一系列在该端口上响应的`filters`, `routes` and `clusters`。在这种情况下，定义了一个绑定到端口 8080 的侦听器（listener）。
Envoy Proxy 使用 YAML 表示法进行配置。如果您不熟悉此表示法，可以查看此[链接](https://yaml.org/spec/1.2.2/)。

```bash
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
```
在下一步中，您将找到将处理流量的路由和集群的配置。

##  4. Backend Configuration
后端配置定义负载平衡器配置以处理传入流量。在此配置示例中，以循环方式定义了两个节点。

```bash
backend nodes
    mode http
    balance roundrobin
    option forwardfor
    server web01 172.18.0.3:80 check
    server web02 172.18.0.4:80 check
```
在 Envoy 中，这个功能是通过创建过滤器和集群来处理的。
###  4.1 Envoy Filters and Clusters
对于静态配置，过滤器定义了如何处理传入请求。在这种情况下，我们定义了与所有流量匹配的过滤器。当发出与定义的域和路由匹配的请求时，流量将转发到集群。这相当于上游配置：

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
            - "*"
          routes:
          - match:
              prefix: "/"
            route:
              cluster: nodes
      http_filters:
      - name: envoy.router
```
名称`envoy.http_connection_manager`是 Envoy Proxy 中的内置过滤器。其他过滤器包括Redis、Mongo、TCP。您可以在文档中找到完整列表。

过滤器控制 Envoy 如何匹配传入的 HTTP 请求以及哪个集群应该处理它们。集群控制哪些服务器正在处理流量和负载平衡配置，例如循环。

```bash
clusters:
- name: nodes
  connect_timeout: 0.25s
  type: STRICT_DNS
  dns_lookup_family: V4_ONLY
  lb_policy: ROUND_ROBIN
  hosts: [
    { socket_address: { address: 172.18.0.3, port_value: 80 }},
    { socket_address: { address: 172.18.0.4, port_value: 80 }}
  ]
```
有关其他负载平衡策略的更多信息，请访问[Envoy 文档](https://www.envoyproxy.io/docs/envoy/v1.8.0/intro/arch_overview/load_balancing)。

##  5. 记录访问和错误
HAProxy 的最后一部分配置存储日志文件的位置以及拥有该进程的用户。

```bash
global
        log /dev/log    local0
        log /dev/log    local1 notice
        user haproxy
        group haproxy
        daemon

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
```
`Envoy Proxy` 的日志遵循云原生方法，其中应用程序日志输出到stdout和stderr。

用户访问日志默认处于禁用状态，需要进行配置。要为 HTTP 请求启用访问日志，请包括`access_log`HTTP 连接管理器的配置。路径可以是设备（例如 stdout），也可以是磁盘上的文件，具体取决于您的要求。

```bash
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
```
结果应如下所示：

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
默认情况下，Envoy 有一个格式字符串，其中包含 HTTP 请求的详细信息。

```bash
[%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%"
%RESPONSE_CODE% %RESPONSE_FLAGS% %BYTES_RECEIVED% %BYTES_SENT% %DURATION%
%RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-FORWARDED-FOR)%" "%REQ(USER-AGENT)%"
"%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n
```
此格式字符串的结果是：

```bash
[2018-11-23T04:51:00.281Z] "GET / HTTP/1.1" 200 - 0 58 4 1 "-" "curl/7.47.0" "f21ebd42-6770-4aa5-88d4-e56118165a7d" "one.example.com" "172.18.0.4:80"
```
可以通过设置格式字段来自定义输出的内容，例如：

```bash
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
    format: "[%START_TIME%] "%REQ(:METHOD)% %REQ(X-ENVOY-ORIGINAL-PATH?:PATH)% %PROTOCOL%" %RESPONSE_CODE% %RESP(X-ENVOY-UPSTREAM-SERVICE-TIME)% "%REQ(X-REQUEST-ID)%" "%REQ(:AUTHORITY)%" "%UPSTREAM_HOST%"\n"
```
也可以通过设置`json_format`字段将日志行输出为 JSON ，例如：

```bash
access_log:
- name: envoy.file_access_log
  config:
    path: "/dev/stdout"
    json_format: {"protocol": "%PROTOCOL%", "duration": "%DURATION%", "request_method": "%REQ(:METHOD)%"}
```
可以在此处找到有关 Envoy 日志记录方法的更多信息。

日志记录并不是使用 Envoy Proxy 获得生产可见性的唯一方法。该平台内置了高级跟踪和度量功能。

##  6. 启动
Envoy 代理配置现在已经从 HA 代理转换而来。最后一部分是启动 Envoy Proxy 实例进行测试。

在配置的顶部，该行`user haproxy`指示以低权限用户身份运行 HA 代理。

Envoy Proxy 采用云原生方法来管理流程所有者。当我们通过容器启动 Envoy 代理时，我们可以指定一个低权限用户。

下面将通过主机上的 Docker 容器启动 Envoy 代理。该命令公开 Envoy 以侦听端口 80 上的传入请求。但是，正如侦听器配置中所指定的，Envoy 代理正在侦听端口 8080 上的传入流量。这允许进程以低权限用户身份运行。

```bash
docker run --name proxy1 -p 80:8080 --user 1000:1000 -v /root/envoy.yaml:/etc/envoy/envoy.yaml envoyproxy/envoy
```

```bash
$ curl localhost -IL
HTTP/1.1 503 Service Unavailable
content-length: 57
content-type: text/plain
date: Wed, 17 Nov 2021 06:57:20 GMT
server: envoy
```
HTTP 请求将导致503错误，因为上游连接未运行且不可用。因此，Envoy 代理没有可用于请求的目标目的地。以下命令将启动一系列与为 Envoy 定义的配置相匹配的 HTTP 服务。

```bash
docker run -d katacoda/docker-http-server; docker run -d katacoda/docker-http-server;
```

```bash
$ curl localhost -i
HTTP/1.1 200 OK
date: Wed, 17 Nov 2021 07:01:31 GMT
content-length: 58
content-type: text/html; charset=utf-8
x-envoy-upstream-service-time: 0
server: envoy

<h1>This request was processed by host: 9e0d200c3327</h1>
```

有了可用的服务，Envoy 就可以成功地将流量代理到目标目的地。
您应该会看到一个响应，指示哪个 Docker 容器处理了请求。在 Envoy 代理日志中，您还应该看到输出的访问行。
