

---
##  1. 简介
此场景演示了如何使用 Envoy 代理保护 HTTP 流量。保护 HTTP 流量对于保护用户隐私和数据至关重要。

在这种情况下，您将学习如何：

 - 应用 SSL 证书以保护 HTTP 流量。
 - 将 HTTP 流量重定向到 HTTPS。

在本场景结束时，您将了解如何使用 TLS 证书通过 Envoy 保护 HTTP 流量。

##  2. SSL 证书
出于测试目的，以下命令将为域example.com生成自签名证书。这种自签名会导致有关证书的警告消息，但非常适合在本地测试配置。部署到生产环境时，您需要从让我们加密等服务为您的站点生成证书。

生成证书
下面的命令在名为 certs/ 的目录中创建一个新的证书和密钥。它将域设置为`example.com`。

```bash
mkdir certs; cd certs;
openssl req -nodes -new -x509 \
  -keyout example-com.key -out example-com.crt \
  -days 365 \
  -subj '/CN=example.com/O=My Company Name LTD./C=US';
cd -
```
##  3. 保护流量
为了保护 HTTP 流量，需要添加 `atls_context`作为过滤器。TLS 上下文提供了为 Envoy 代理中配置的域指定证书集合的能力。处理 HTTPS 请求时，将使用匹配的证书。

在这种情况下，证书是我们在第一步中生成的自签名证书。

### 3.1 将 TLS 上下文添加到 HTTPS 侦听器
打开`envoy.yaml`配置文件。它包含所需 HTTPS 支持的概要。它配置了两个侦听器，一个在端口 `8080` 上用于 HTTP 流量，另一个在 `8443` 上用于 HTTPS 流量。

HTTPS 侦听器定义了 HTTP 连接管理器，它将代理传入请求`/service/1`和`/service/2`端点。这需要扩展以包括所需的内容`tls_context` ，如下所示。

```bash
static_resources:
  listeners:
  - name: listener_http
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            virtual_hosts:
            - name: backend
              domains:
              - "example.com"
              routes:
              - match:
                  prefix: "/"
                redirect:
                  path_redirect: "/"
                  https_redirect: true
          http_filters:
          - name: envoy.router
            config: {}
  - name: listener_https
    address:
      socket_address: { address: 0.0.0.0, port_value: 8443 }
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
              - "example.com"
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
      tls_context:
        common_tls_context:
          tls_certificates:
            - certificate_chain:
                filename: "/etc/envoy/certs/example-com.crt"
              private_key:
                filename: "/etc/envoy/certs/example-com.key"
  clusters:
  - name: service1
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 172.18.0.3
        port_value: 80
  - name: service2
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 172.18.0.4
        port_value: 80

admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8001
```
在上下文中，定义了生成的证书和密钥。如果我们有多个域，每个域都有自己的证书，那么将定义多个证书链。

##  4. 重定向 HTTP 流量
随着TLS上下文定义，网站将能够通过HTTPS流量。如果用户碰巧访问了该网站的 HTTP 版本，我们希望他们将他们重定向到 HTTPS 版本以确保他们是安全的。

### 4.1 编辑 HTTP 过滤器
在我们的 HTTP 配置中，作为域的过滤器匹配的一部分，`https_redirect: true`需要向过滤器配置添加一个标志。

我们的标准 Envoy 代理配置如下所示。

```bash
route_config:
  virtual_hosts:
  - name: backend
  domains:
  - "example.com"
  routes:
  - match:
      prefix: "/"
```
这需要扩展以包括字段 HTTPS 重定向。

```bash
redirect:
                  path_redirect: "/"
                  https_redirect: true
```
当用户访问该站点的 HTTP 版本时，Envoy 代理会根据过滤器配置匹配域和路径。这会导致用户被重定向到站点的 HTTPS 版本。

这可以在完成的示例中看到`envoy-completed.yaml`。

```bash
static_resources:
  listeners:
  - name: listener_http
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            virtual_hosts:
            - name: backend
              domains:
              - "example.com"
              routes:
              - match:
                  prefix: "/"
                redirect:
                  path_redirect: "/"
                  https_redirect: true
          http_filters:
          - name: envoy.router
            config: {}
  - name: listener_https
    address:
      socket_address: { address: 0.0.0.0, port_value: 8443 }
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
              - "example.com"
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
      tls_context:
        common_tls_context:
          tls_certificates:
            - certificate_chain:
                filename: "/etc/envoy/certs/example-com.crt"
              private_key:
                filename: "/etc/envoy/certs/example-com.key"
  clusters:
  - name: service1
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 172.18.0.3
        port_value: 80
  - name: service2
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 172.18.0.4
        port_value: 80

admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8001
```
##  5. 启动代理
有了额外的配置，Envoy 就可以启动了。

在这种情况下，代理在用于 HTTP 流量的端口 `80` 和用于 HTTPS 的端口 `443` 上公开。它还公开了 `8001` 上的管理仪表板，允许您查看有关证书的仪表板信息。

###  5.1 Start Envoy

```bash
docker run -it --name proxy1 -p 80:8080 -p 443:8443 -p 8001:8001 -v /root/:/etc/envoy/ envoyproxy/envoy
```
所有 HTTPS 和 TLS 终止都是通过 Envoy 代理处理的，这意味着不需要修改应用程序。要启动一系列 HTTP 服务器来处理传入的请求，请运行以下命令：

```bash
docker run -d katacoda/docker-http-server; docker run -d katacoda/docker-http-server;
```
###  5.2 测试配置
代理启动后，可以测试配置。

首先，如果您发出 HTTP 请求，由于您的配置标志，代理应返回到 HTTPS 版本的重定向响应。

```bash
$ curl -H "Host: example.com" http://localhost -i
HTTP/1.1 301 Moved Permanently
location: https://example.com/
date: Wed, 17 Nov 2021 07:21:36 GMT
server: envoy
content-length: 0
```
您可以看到指示您配置的重定向的响应。

```bash
HTTP/1.1 301 Moved Permanently
location: https://example.com/
```

HTTPS 请求将根据您的配置进行处理。尝试以下请求：

```bash
$ curl -k -H "Host: example.com" https://localhost/service/1 -i
HTTP/1.1 200 OK
date: Wed, 17 Nov 2021 07:22:21 GMT
content-length: 58
content-type: text/html; charset=utf-8
x-envoy-upstream-service-time: 0
server: envoy

<h1>This request was processed by host: d8044fe383d8</h1>
$ curl -k -H "Host: example.com" https://localhost/service/2 -i
HTTP/1.1 200 OK
date: Wed, 17 Nov 2021 07:22:24 GMT
content-length: 58
content-type: text/html; charset=utf-8
x-envoy-upstream-service-time: 0
server: envoy

<h1>This request was processed by host: fe21146f781e</h1>
```
请注意，如果没有-k参数，cURL 将因自签名证书而响应错误：

```bash
$ curl -H "Host: example.com" https://localhost/service/2 -i
curl: (60) server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none
More details here: http://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
```
###  5.3 验证证书
使用 OpenSSL CLI 可以查看从服务器返回的证书。这将允许我们验证从 Envoy 返回的正确证书：

```bash
$ echo | openssl s_client -showcerts -servername example.com -connect localhost:443 2>/dev/null | openssl x509 -inform pem -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 16688607145593841019 (0xe799d7c7652aa57b)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=example.com, O=My Company Name LTD., C=US
        Validity
            Not Before: Nov 17 07:09:40 2021 GMT
            Not After : Nov 17 07:09:40 2022 GMT
        Subject: CN=example.com, O=My Company Name LTD., C=US
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:d4:33:e2:7f:38:aa:79:eb:1f:7e:bd:2f:23:58:
                    87:16:f2:73:a7:c0:15:d8:5b:e7:8b:c7:6c:cc:09:
                    91:f1:94:80:f2:8c:a6:e2:79:be:a2:6b:71:ef:fa:
                    d4:c3:ba:47:66:9c:64:b7:3a:b4:cb:c4:5c:da:cf:
                    27:81:46:cc:a5:d5:5d:0b:d4:87:39:39:13:3a:e4:
                    d0:ff:a5:3d:14:c8:5d:4f:e7:8e:86:d9:7e:1a:36:
                    22:fa:9f:d8:85:23:61:4e:fb:53:11:a1:e1:42:21:
                    e5:69:2b:9b:2c:a5:13:d2:31:66:87:d2:cc:60:4c:
                    5d:c1:f7:4f:e8:7c:2c:9d:28:1b:8c:d4:41:8e:86:
                    92:08:77:7b:bd:5d:ef:03:c4:26:e2:43:d9:25:af:
                    36:d8:75:93:d1:6e:6c:ec:64:f2:b4:f8:f3:f8:53:
                    7b:d8:d7:b8:71:5e:00:c1:34:1f:0b:c6:e8:22:b5:
                    eb:e5:01:5b:97:aa:12:98:9e:28:a7:1b:9d:17:03:
                    a9:64:9b:08:ff:88:39:04:fb:65:94:cb:3e:0b:f1:
                    27:20:be:02:d7:1d:fb:39:82:5c:26:a1:0f:85:eb:
                    b9:5d:69:ac:81:a9:78:04:d1:9a:df:61:ae:d5:89:
                    08:2a:b0:59:9b:04:d0:b4:fb:56:6a:3b:2b:81:be:
                    37:8d
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                26:B4:FC:FC:C1:FF:A8:3C:9C:32:15:E9:C9:05:39:00:FB:82:09:C3
            X509v3 Authority Key Identifier: 
                keyid:26:B4:FC:FC:C1:FF:A8:3C:9C:32:15:E9:C9:05:39:00:FB:82:09:C3

            X509v3 Basic Constraints: 
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         5d:0e:26:d5:d4:52:12:3a:d0:f8:00:51:ab:48:13:64:59:99:
         fc:48:4b:88:33:3a:85:42:b8:f2:65:2e:58:aa:9c:a9:24:9f:
         c2:35:0d:9a:22:22:da:46:45:b1:39:67:7d:83:b7:42:54:cd:
         f3:00:5a:c3:b9:94:66:45:1f:8c:64:10:b1:47:e3:6e:36:c6:
         07:c6:88:68:2f:35:46:94:4b:c8:c6:d8:b1:b9:0c:b4:8e:9f:
         d4:c3:63:31:08:49:e8:44:37:bc:0d:91:aa:27:e4:b8:0f:87:
         1a:73:d8:35:5b:6d:c6:74:4a:67:c4:35:75:89:b8:2f:ab:5b:
         d2:f9:c7:14:01:81:5d:de:a1:ce:b0:80:65:a0:a6:b7:49:80:
         e5:4f:34:e4:66:6f:c1:92:29:eb:4f:41:02:86:ad:4d:e0:11:
         c7:49:87:3f:49:0a:3b:ae:c0:2a:05:86:b5:84:88:32:e7:1f:
         58:d1:6a:55:d3:e0:df:c6:db:e8:c8:d8:c0:58:e9:fc:2d:53:
         f1:c7:23:96:f3:b2:29:23:c7:44:d4:c9:d2:2c:38:7f:81:f2:
         74:27:3b:14:dc:c5:f0:38:c9:3b:ff:b1:f7:67:48:60:1c:2a:
         30:f6:28:54:f8:7c:fa:64:47:1a:63:db:ab:aa:7a:42:f8:d6:
         25:93:8d:f9
```
###  5.4 Dashboard
界面返回有关定义的证书及其年龄的信息。https://ip/certs 找到
