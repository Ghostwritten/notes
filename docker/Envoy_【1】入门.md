

##  1. 简介
[Envoy](https://www.envoyproxy.io/)是一个开源的边缘和服务代理，专为云原生应用程序而设计。

以下场景演示了如何将 Envoy 配置为代理，允许您将流量转发到不同的目的地。

你将学到如何：

 - 配置 Envoy 代理以将流量转发到外部网站。
 - 配置 Envoy 代理以将流量转发到 Docker 容器。
 - 执行基于路径的路由以控制流量目的地。

一旦 Envoy 代理就位，它可以扩展以支持负载平衡、健康检查和指标。这些将在更高级的场景中讨论。

##  2. 配置
Envoy 使用 YAML 定义文件`/etc/envoy/envoy.yaml`进行配置，以控制代理的行为。在这一步中，我们将使用静态配置 API 构建配置。这意味着所有设置都是在配置中预先定义的。

Envoy 还支持动态配置。这允许通过外部源发现设置。

### 1.1 **Resources**
Envoy 配置的第一行定义了正在使用的 API 配置。在这种情况下，我们正在配置静态 API，所以第一行应该是`static_resources`。将代码片段复制到编辑器。

```bash
static_resources:
```
### 1.2 **Listeners**
配置的开头定义了`Listeners`。侦听器是 `Envoy` 侦听请求的网络配置，例如 IP 地址和端口。Envoy 在 Docker 容器内部运行，因此它需要侦听 IP 地址`0.0.0.0`。在这种情况下，Envoy 将侦听端口`10000`。

以下是定义此设置的配置。将代码片段复制到编辑器

```bash
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 10000 }
```
###  1.3 Filter Chains and Filters
随着 Envoy 侦听传入流量，下一阶段是定义如何处理请求。每个 `Listener` 都有一组过滤器，不同的 Listener 可以有一组不同的过滤器。

在此示例中，我们将所有流量代理到 Google.com（感谢 Google！）。结果：我们应该能够请求 Envoy 端点并看到 Google 主页出现，而不需要更改 URL。

过滤是使用`filter_chains`定义的。每个过滤器的目的是找到传入请求的匹配项，将其匹配到目标目的地。将代码片段复制到编辑器。

```bash
filter_chains:
 - filters:
 - name: envoy.http_connection_manager
    config:
      stat_prefix: ingress_http
      route_config:
        name: local_route
        virtual_hosts:
        - name: local_service
          domains: ["*"]
          routes:
          - match: { prefix: "/" }
            route: { host_rewrite: www.google.com, cluster: service_google }
      http_filters:
      - name: envoy.router
```
过滤器使用`envoy.http_connection_manager`，一个专为 HTTP 连接设计的内置过滤器。详细情况如下：
 - `stat_prefix`：为连接管理器发出统计信息时使用的人类可读前缀。
 - `route_config`：路由的配置。如果虚拟主机匹配，则检查路由。在这个例子中， route_config匹配所有传入的 HTTP
   请求，不管请求的主机域。
 - `routes`：如果 URL 前缀匹配，则一组路由规则定义接下来应该发生什么。在这种情况下，“/”表示匹配请求的根
 - `host_rewrite`：更改 HTTP 请求的入站主机标头。
 - `cluster`：将处理请求的集群的名称。实现定义如下。
 - `http_filters`：过滤器允许 Envoy 在处理请求时调整和修改请求。

### 1.4 **Clusters**
当请求与过滤器匹配时，请求被传递到集群上。下面显示的集群定义了主机是通过 HTTPS 运行的 `google.com`。如果定义了多个主机，那么 Envoy 将执行循环策略。

复制集群实现完成配置：

```bash
  clusters:
  - name: service_google
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    hosts: [{ socket_address: { address: google.com, port_value: 443 }}]
    tls_context: { sni: www.google.com }
```
### 1.5 **Admin**

最后，需要一个管理部分。在以下步骤中更详细地解释了管理部分。

```bash
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }
```
这个结构定义了 Envoy 静态配置的样板。监听器定义了 Envoy 的端口和 IP 地址。侦听器有一组过滤器来匹配传入的请求。一旦请求匹配，它将被转发到集群。

你可以在[Github](https://github.com/envoyproxy/envoy/blob/6a578630a8f6189f86bc1e6b4b4d7ebffabadadd/configs/google_com_proxy.v2.yaml)上查看完整的配置。
**/etc/envoy/envoy.yaml**
```bash
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 127.0.0.1, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 10000 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { host_rewrite: www.google.com, cluster: service_google }
          http_filters:
          - name: envoy.router
  clusters:
  - name: service_google
    connect_timeout: 0.25s
    type: LOGICAL_DNS
    # Comment out the following line to test on v6 networks
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    hosts: [{ socket_address: { address: google.com, port_value: 443 }}]
    tls_context: { sni: www.google.com }
```

##  3. 部署代理
  
  配置到位后，可以启动容器并将其提供给 Envoy。使用 Docker，这是通过将 Volume Mount 挂载到`/etc/envoy/envoy.yaml` 来完成的。
启动代理，绑定到端口 80：

```bash
docker run --name=proxy -d \
  -p 80:10000 \
  -v $(pwd)/envoy/envoy.yaml:/etc/envoy/envoy.yaml \
  envoyproxy/envoy:latest

#启动后，您应该能够将 HTTP 请求发送到 80 端口
curl localhost
```
您可以通过本地浏览器查看此 URL正如您将看到的，请求被代理到 Google.com，您应该看到没有 URL 的 Google 主页改变。

## 4. 部署界面访问
Envoy 提供了一个管理视图，允许您查看配置、统计信息、日志和其他 Envoy 内部数据。

可以通过添加额外的资源定义来定义 admin，其中定义了 admin 视图的端口。该端口不应与其他侦听器配置冲突。

```bash
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }
```
启动admin
这个 Docker 容器还向外界公开了管理端口。上面的资源配置将使管理视图对公众可用，仅用于演示目的，请参阅有关如何保护管理门户的文档。

要公开管理门户，请运行以下命令：

```bash
docker run --name=proxy-with-admin -d \
    -p 9901:9901 \
    -p 10000:10000 \
    -v $(pwd)/envoy/envoy.yaml:/etc/envoy/envoy.yaml \
    envoyproxy/envoy:latest
```
通过端口`http://ip:9901`访问界面
“当前形式的管理界面既允许执行破坏性操作（例如，关闭服务器），也可能暴露私人信息（例如，统计信息、集群名称、证书信息等）。访问权限至关重要仅允许通过安全网络访问管理界面” [Envoy 文档](https://www.envoyproxy.io/docs/envoy/latest/operations/admin)

##  5. 路由到 Docker 应用容器
最后一个示例使用 Envoy 根据请求的 URL 路径将流量代理到不同的 Python 服务。

### 5.1 配置
应用程序的配置被定义为一个 Docker Compose 文件。我们使用 Docker Compose 文件是因为我们想同时运行多个容器——一个用于代理，一个用于每个单独的服务。

```bash
cat examples/front-proxy/docker-compose.yml
```

```bash
version: '2'
services:

  front-envoy:
    build:
      context: .
      dockerfile: Dockerfile-frontenvoy
    volumes:
      - ./front-envoy.yaml:/etc/front-envoy.yaml
    networks:
      - envoymesh
    expose:
      - "80"
      - "8001"
    ports:
      - "8000:80"
      - "8001:8001"

  service1:
    build:
      context: .
      dockerfile: Dockerfile-service
    volumes:
      - ./service-envoy.yaml:/etc/service-envoy.yaml
    networks:
      envoymesh:
        aliases:
          - service1
    environment:
      - SERVICE_NAME=1
    expose:
      - "80"

  service2:
    build:
      context: .
      dockerfile: Dockerfile-service
    volumes:
      - ./service-envoy.yaml:/etc/service-envoy.yaml
    networks:
      envoymesh:
        aliases:
          - service2
    environment:
      - SERVICE_NAME=2
    expose:
      - "80"

networks:
  envoymesh: {}
```
###  5.2 应用程序
该服务是一个 Python Web 应用程序，它还使用容器内的 Envoy 将流量转发到 Python 应用程序。没有必要在应用程序前面放置 Envoy。

```bash
cat examples/front-proxy/service.py
```

```bash
from flask import Flask
from flask import request
import socket
import os
import sys
import requests

app = Flask(__name__)

TRACE_HEADERS_TO_PROPAGATE = [
    'X-Ot-Span-Context',
    'X-Request-Id',

    # Zipkin headers
    'X-B3-TraceId',
    'X-B3-SpanId',
    'X-B3-ParentSpanId',
    'X-B3-Sampled',
    'X-B3-Flags',

    # Jaeger header (for native client)
    "uber-trace-id"
]

@app.route('/service/<service_number>')
def hello(service_number):
    return ('Hello from behind Envoy (service {})! hostname: {} resolved'
            'hostname: {}\n'.format(os.environ['SERVICE_NAME'], 
                                    socket.gethostname(),
                                    socket.gethostbyname(socket.gethostname())))

@app.route('/trace/<service_number>')
def trace(service_number):
    headers = {}
    # call service 2 from service 1
    if int(os.environ['SERVICE_NAME']) == 1 :
        for header in TRACE_HEADERS_TO_PROPAGATE:
            if header in request.headers:
                headers[header] = request.headers[header]
        ret = requests.get("http://localhost:9000/trace/2", headers=headers)
    return ('Hello from behind Envoy (service {})! hostname: {} resolved'
            'hostname: {}\n'.format(os.environ['SERVICE_NAME'], 
                                    socket.gethostname(),
                                    socket.gethostbyname(socket.gethostname())))

if __name__ == "__main__":
    app.run(host='127.0.0.1', port=8080, debug=True)
```

###  5.3  Envoy Frontend Proxy
Envoy 代理配置定义在：

```bash
cat examples/front-proxy/front-envoy.yaml
```

```bash
static_resources:
  listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 80
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
                  prefix: "/service/1"
                route:
                  cluster: service1
              - match:
                  prefix: "/service/2"
                route:
                  cluster: service2
          http_filters:
          - name: envoy.router
            config: {}
  clusters:
  - name: service1
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service1
        port_value: 80
  - name: service2
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    hosts:
    - socket_address:
        address: service2
        port_value: 80
admin:
  access_log_path: "/dev/null"
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8001
```

如第一步所述，配置从定义一组`static_resources` 开始。路由根据请求的 URL 进行匹配。

###  5.4 部署
使用下面的 Docker Compose 命令启动示例：

```bash
$ $ ls envoy/examples/front-proxy/
deprecated_v1/         Dockerfile-frontenvoy  front-envoy.yaml       service-envoy.yaml     start_service.sh
docker-compose.yml     Dockerfile-service     README.md              service.py        


$ docker-compose -f ~/envoy/examples/front-proxy/docker-compose.yml up -d
Creating network "frontproxy_envoymesh" with the default driver
Creating frontproxy_service1_1 ... 
Creating frontproxy_front-envoy_1 ... 
Creating frontproxy_service2_1 ... 
Creating frontproxy_front-envoy_1
Creating frontproxy_service1_1
Creating frontproxy_front-envoy_1 ... done
```
###  5.5 admin访问
**访问界面**
```bash
$ curl http://localhost:8001

<head>
  <title>Envoy Admin</title>
  <link rel='shortcut icon' type='image/png' href='data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAAEnQAABJ0Ad5mH3gAAAH9SURBVEhL7ZRdTttAFIUrUFaAX5w9gIhgUfzshFRK+gIbaVbAzwaqCly1dSpKk5A485/YCdXpHTB4BsdgVe0bD0cZ3Xsm38yZ8byTUuJ/6g3wqqoBrBhPTzmmLfptMbAzttJTpTKAF2MWC7ADCdNIwXZpvMMwayiIwwS874CcOc9VuQPR1dBBChPMITpFXXU45hukIIH6kHhzVqkEYB8F5HYGvZ5B7EvwmHt9K/59CrU3QbY2RNYaQPYmJc+jPIBICNCcg20ZsAsCPfbcrFlRF+cJZpvXSJt9yMTxO/IAzJrCOfhJXiOgFEX/SbZmezTWxyNk4Q9anHMmjnzAhEyhAW8LCE6wl26J7ZFHH1FMYQxh567weQBOO1AW8D7P/UXAQySq/QvL8Fu9HfCEw4SKALm5BkC3bwjwhSKrA5hYAMXTJnPNiMyRBVzVjcgCyHiSm+8P+WGlnmwtP2RzbCMiQJ0d2KtmmmPorRHEhfMROVfTG5/fYrF5iWXzE80tfy9WPsCqx5Buj7FYH0LvDyHiqd+3otpsr4/fa5+xbEVQPfrYnntylQG5VGeMLBhgEfyE7o6e6qYzwHIjwl0QwXSvvTmrVAY4D5ddvT64wV0jRrr7FekO/XEjwuwwhuw7Ef7NY+dlfXpLb06EtHUJdVbsxvNUqBrwj/QGeEUSfwBAkmWHn5Bb/gAAAABJRU5ErkJggg=='/>
  <style>
    .home-table {
      font-family: sans-serif;
      font-size: medium;
      border-collapse: collapse;
    }

    .home-row:nth-child(even) {
      background-color: #dddddd;
    }

    .home-data {
      border: 1px solid #dddddd;
      text-align: left;
      padding: 8px;
    }

    .home-form {
      margin-bottom: 0;
    }
  </style>
</head>
<body>
  <table class='home-table'>
    <thead>
      <th class='home-data'>Command</th>
      <th class='home-data'>Description</th>
     </thead>
     <tbody>
<tr class='home-row'><td class='home-data'><a href='certs'>certs</a></td><td class='home-data'>print certs on machine</td></tr>
<tr class='home-row'><td class='home-data'><a href='clusters'>clusters</a></td><td class='home-data'>upstream cluster status</td></tr>
<tr class='home-row'><td class='home-data'><a href='config_dump'>config_dump</a></td><td class='home-data'>dump current Envoy configs (experimental)</td></tr>
<tr class='home-row'><td class='home-data'><form action='cpuprofiler' method='post' class='home-form'><button>cpuprofiler</button></form></td><td class='home-data'>enable/disable the CPU profiler</td></tr>
<tr class='home-row'><td class='home-data'><form action='healthcheck/fail' method='post' class='home-form'><button>healthcheck/fail</button></form></td><td class='home-data'>cause the server to fail health checks</td></tr>
<tr class='home-row'><td class='home-data'><form action='healthcheck/ok' method='post' class='home-form'><button>healthcheck/ok</button></form></td><td class='home-data'>cause the server to pass health checks</td></tr>
<tr class='home-row'><td class='home-data'><a href='help'>help</a></td><td class='home-data'>print out list of admin commands</td></tr>
<tr class='home-row'><td class='home-data'><a href='hot_restart_version'>hot_restart_version</a></td><td class='home-data'>print the hot restart compatibility version</td></tr>
<tr class='home-row'><td class='home-data'><a href='listeners'>listeners</a></td><td class='home-data'>print listener addresses</td></tr>
<tr class='home-row'><td class='home-data'><form action='logging' method='post' class='home-form'><button>logging</button></form></td><td class='home-data'>query/change logging levels</td></tr>
<tr class='home-row'><td class='home-data'><a href='memory'>memory</a></td><td class='home-data'>print current allocation/heap usage</td></tr>
<tr class='home-row'><td class='home-data'><form action='quitquitquit' method='post' class='home-form'><button>quitquitquit</button></form></td><td class='home-data'>exit the server</td></tr>
<tr class='home-row'><td class='home-data'><form action='reset_counters' method='post' class='home-form'><button>reset_counters</button></form></td><td class='home-data'>reset all counters to zero</td></tr>
<tr class='home-row'><td class='home-data'><a href='runtime'>runtime</a></td><td class='home-data'>print runtime values</td></tr>
<tr class='home-row'><td class='home-data'><form action='runtime_modify' method='post' class='home-form'><button>runtime_modify</button></form></td><td class='home-data'>modify runtime values</td></tr>
<tr class='home-row'><td class='home-data'><a href='server_info'>server_info</a></td><td class='home-data'>print server version/status information</td></tr>
<tr class='home-row'><td class='home-data'><a href='stats'>stats</a></td><td class='home-data'>print server stats</td></tr>
<tr class='home-row'><td class='home-data'><a href='stats/prometheus'>stats/prometheus</a></td><td class='home-data'>print server stats in prometheus format</td></tr>

    </tbody>
  </table>
</body>
```
**应用路由**

```bash
$ curl localhost:8000/service/1
Hello from behind Envoy (service 1)! hostname: 270af01a022e resolvedhostname: 172.19.0.2
$ curl localhost:8000/service/2
Hello from behind Envoy (service 2)! hostname: b39f03aae28d resolvedhostname: 172.19.0.3
```

