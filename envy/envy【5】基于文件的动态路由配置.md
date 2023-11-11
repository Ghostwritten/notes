##  简介
在此场景中，您将学习如何将 Envoy 静态配置转换为动态配置，从而允许在 Envoy 内进行更改并自动更新。

你将学习：

 - 可用的动态配置 API。
 - 如何配置 Envoy 以使用基于动态文件的上游配置。
 - 如何更改配置并在 Envoy 中查看结果。

## Envoy 动态配置

在前面的场景中，我们已经定义了静态配置。然而，这使得在需要更改时重新加载配置变得困难。为了解决这个问题，静态配置可以定义为动态配置。使用动态配置，当进行更改时，Envoy 将自动重新加载更改并将其应用于配置和流量路由。

Envoy 支持动态配置的不同部分。可用的 API 有：

 - `EDS`：端点发现服务 (EDS) API 提供了一种 Envoy 可以发现上游集群成员的方式。这允许您动态添加和删除处理流量的服务器。
 - `CDS`：集群发现服务 (CDS) API 在一种机制上分层，Envoy 可以通过该机制发现路由期间使用的上游集群。
 - `RDS`：路由发现服务 (RDS) API 在一种机制上分层，Envoy 可以通过该机制在运行时发现 HTTP连接管理器过滤器的整个路由配置。这将实现诸如动态变化的交通转移和蓝/绿释放等概念。
 - `LDS`：侦听器发现服务 (LDS) 建立在一种机制之上，Envoy 可以通过该机制在运行时发现整个侦听器。
 - `SDS`：秘密发现服务 (SDS) 在一种机制上分层，通过该机制 Envoy 可以为其侦听器发现加密秘密（证书加私钥、TLS会话票证密钥），以及对等证书验证逻辑的配置（受信任的根证书、撤销等）。

配置值可以来自文件系统、`REST-JSON` 或 `gRPC` 端点。

可以在[xDS 配置 API 概述](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/dynamic_configuration)中找到更多信息

在接下来的步骤中，我们将更改我们的配置以使用端点发现服务 (EDS)，允许根据来自文件系统的数据动态添加节点。

##  集群 ID
所需的 Envoy 配置的初始大纲可在 envoy.yaml

需要的第一个更改是添加一个Node。这允许识别 Envoy 节点，可能允许应用独特的配置。
将以下代码段添加到`envoy.yaml`文件顶部。

```bash
node:
  id: id_1
  cluster: test
```
该API还具有用于附加元数据，例如支持[局部](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/core/v3/base.proto.html#core-locality)性用于提供区域和基于区域的信息。

```bash
node:
  id: id_1
  cluster: test

node:
  id: id_1
  cluster: test

admin:
  access_log_path: "/dev/null"
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 9901
      
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 10000 }
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
                  cluster: targetCluster
          http_filters:
          - name: envoy.router
```

##  EDS 配置
EDS 配置被定义为允许动态控制上游集群。

在静态配置中，这被定义为：

