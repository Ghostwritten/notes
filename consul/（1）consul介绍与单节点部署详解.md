


---
## 1. consul 介绍
Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其它分布式服务注册与发现的方案，Consul 的方案更“一站式”，内置了服务注册与发现框 架、分布一致性协议实现、健康检查、Key/Value 存储、多数据中心方案，不再需要依赖其它工具（比如 ZooKeeper 等）。使用起来也较 为简单。Consul 使用 Go 语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与 Docker 等轻量级容器可无缝配合。

### 1.1 Consul 的优势：

 - 使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 采用的是 Paxos, 而etcd 使用的则是 Raft。
 - 支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟,分片等情况等。 zookeeper 和 etcd 均不提供多数据中心功能的支持。
 - 支持健康检查。 etcd 不提供此功能。
 - 支持 http 和 dns 协议接口。 zookeeper 的集成较为复杂, etcd 只支持 http 协议。
 - 官方提供 web 管理界面, etcd 无此功能。
 - 综合比较, Consul 作为服务注册和配置管理的新星, 比较值得关注和研究。

### 1.2 特性：

 - 服务发现
 - 健康检查
 - Key/Value 存储
 - 多数据中心
 
### 1.3 Consul 角色
 - client: 客户端, 无状态, 将 HTTP 和 DNS 接口请求转发给局域网内的服务端集群。
 - server: 服务端, 保存配置信息, 高可用集群, 在局域网内与本地客户端通讯, 通过广域网与其它数据中心通讯。 每个数据中心的server 数量推荐为 3 个或是 5 个。

Consul 客户端、服务端还支持夸中心的使用，更加提高了它的高可用性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200510154030725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
Consul 工作原理：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200510154050732.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

### 1.4 需要的端口
|use  |	Default Ports  |
|--|--|
| DNS : The DNS server (TCP and UDP)| 8600 |
| HTTP: The HTTP API (TCP Only) | 8500  |
| HTTPS: The HTTPs API  | disabled (8501)* |
| gRPC: The gRPC API	disabled |  8301 |
|Wan Serf: The Serf WAN port TCP and UDP)  | 8302 |
| server: Server RPC address (TCP Only) |  8300 |
|Sidecar Proxy Min: Inclusive min port number to use for automatically assigned sidecar service registrations.  |  21000|
|Sidecar Proxy Max: Inclusive max port number to use for automatically assigned sidecar service registrations.  |  21255 |




##  2. Consul 安装
实验环境：

```bash
CentOS Linux 7 (Core)  192.168.1.153
CentOS Linux 7 (Core)   192.168.1.154
CentOS Linux 7 (Core) 192.168.1.155
```

Consul 不同于 Eureka 需要单独安装，访问Consul 官网下载 Consul 的最新版本，我这里是 consul_1.7.3。

根据不同的系统类型选择不同的安装包，从下图也可以看出 Consul 支持所有主流系统。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200510154443945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 2.1 linux安装

```bash
$ wget https://releases.hashicorp.com/consul/1.7.3/consul_1.7.3_linux_amd64.zip
$ mv consul /usr/local/bin/
$ consul -v
Consul v1.7.3
```
### 2.2 docker 安装

```bash
docker run -d --net=host --name=dev-consul consul agent -dev -client 0.0.0.0
```

## 3. 运行一个开发模式的单节点consul

```bash
$ consul agent -dev -ui -client 0.0.0.0
```
 - -ui: 开启ui页面
 - -client: 让consul server拥有client的功能，允许接受服务注册；0.0.0.0表示允许使用任意IP连接Consul，如果不指定，默认是127.0.0.1。
 - -dev: 表示以开发模式运行Consul

 http://192.168.1.153:8500
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200510160159194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

**查看当前节点数量**
```bash
$ consul members
Node         Address         Status  Type    Build  Protocol  DC   Segment
rds-consul1  127.0.0.1:8301  alive   server  1.7.3  2         dc1  <all>
```
用户还可以通过HTTP API的方式查看当前节点信息：

```bash
$  curl localhost:8500/v1/catalog/nodes
[
    {
        "ID": "daf44993-e58e-91b3-aeef-6e07294fce21",
        "Node": "rds-consul1",
        "Address": "127.0.0.1",
        "Datacenter": "dc1",
        "TaggedAddresses": {
            "lan": "127.0.0.1",
            "lan_ipv4": "127.0.0.1",
            "wan": "127.0.0.1",
            "wan_ipv4": "127.0.0.1"
        },
        "Meta": {
            "consul-network-segment": ""
        },
        "CreateIndex": 9,
        "ModifyIndex": 9
    }
]
```
**Consul还提供了内置的DNS服务，可以通过Consul的DNS服务的方式访问其中的节点：**

```bash
$ dig @127.0.0.1 -p 8600 localhost.node.consul

; <<>> DiG 9.9.4-RedHat-9.9.4-50.el7 <<>> @127.0.0.1 -p 8600 localhost.node.consul
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 35141
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;localhost.node.consul.		IN	A

;; AUTHORITY SECTION:
consul.			0	IN	SOA	ns.consul. hostmaster.consul. 1589122613 3600 600 86400 0

;; Query time: 0 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Sun May 10 22:56:53 CST 2020
;; MSG SIZE  rcvd: 100
```

## 4. 发现一个服务
### 4.1 启动一个web服
使被consul服务发现

```python
$ cat web.py 
#!/usr/bin/python
from bottle import run, get
 
@get('/hello')
def hello():
    return "Hello World!"
 
run(host='0.0.0.0', port=8080, debug=True)
```

```bash
$ python web.py
Bottle v0.12.18 server starting up (using WSGIRefServer())...
Listening on http://0.0.0.0:8080/
Hit Ctrl-C to quit.
```

curl -i "127.0.0.1:8080/hello"  检查web是否正常


### 4.2 运行一个开发模式的单节点consul去发现web服务


```bash
$ mkdir /etc/consul.d  #创建配置文件存放目录
#内容包含了服务名，IP地址，端口，检测脚本。
$ vim /etc/consul.d/web.json #探测发现服务的配置文件【注册服务】
{
  "services": [
    {
      "id": "hello1",
      "name": "hello",
      "tags": [
        "primary"
      ],
      "address": "192.168.1.153",
      "port": 8080,
      "checks": [
        {
        "http": "http://192.168.1.153:8080/hello",
        "tls_skip_verify": false,
        "method": "Get",
        "interval": "10s",
        "timeout": "1s"
        }
      ]
    }
  ]
}

$ mkdir /etc/consul/data #存储数据目录
$ consul agent -dev -ui -client 0.0.0.0 -config-dir /etc/consul.d -data-dir=/etc/consul/data
或者
$ consul agent -dev -ui -client 0.0.0.0 -config-file /etc/consul.d/web.json -data-dir=/etc/consul/data
```

 - `-config-dir`：指定service的配置文件和检查定义所在的位置•通常会指定为"某一个路径/consul.d"（**通常情况下，.d表示一系列配置文件存放的目录**）
 - `-data-dir`：提供一个目录用来存放agent的状态，所有的agent都需要该目录，该目录必须是稳定的，系统重启后都继续存在
 - `-config-file`：明确的指定要加载哪个配置文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200510225019825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020051023022844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

一旦服务注册成功之后，用户就可以通过DNS或HTTP API的方式查询服务信息。默认情况下，所有的服务都可以使用`NAME.service.consul`域名的方式进行访问。
例如，可以使用`hello.service.consul`域名查询`hello`服务的信息：

```bash
$ dig @127.0.0.1 -p 8600 hello.service.consul
.....
;hello.service.consul.		IN	A

;; ANSWER SECTION:
hello.service.consul.	0	IN	A	192.168.1.153
......
```
如上所示DNS记录会返回当前可用的node_exporter服务实例的IP地址信息。

除了使用DNS的方式以外，Consul还支持用户使用HTTP API的形式获取服务列表：

```bash
$ curl http://localhost:8500/v1/catalog/service/hello
[
    {
        "ID": "daf44993-e58e-91b3-aeef-6e07294fce21",
        "Node": "rds-consul1",
        "Address": "127.0.0.1",
        "Datacenter": "dc1",
        "TaggedAddresses": {
            "lan": "127.0.0.1",
            "lan_ipv4": "127.0.0.1",
            "wan": "127.0.0.1",
            "wan_ipv4": "127.0.0.1"
        },
        "NodeMeta": {
            "consul-network-segment": ""
        },
        "ServiceKind": "",
        "ServiceID": "hello1",
        "ServiceName": "hello",
        "ServiceTags": [
            "primary"
        ],
        "ServiceAddress": "192.168.1.153",
        "ServiceTaggedAddresses": {
            "lan_ipv4": {
                "Address": "192.168.1.153",
                "Port": 8080
            },
            "wan_ipv4": {
                "Address": "192.168.1.153",
                "Port": 8080
            }
        },
        "ServiceWeights": {
            "Passing": 1,
            "Warning": 1
        },
        "ServiceMeta": {},
        "ServicePort": 8080,
        "ServiceEnableTagOverride": false,
        "ServiceProxy": {
            "MeshGateway": {},
            "Expose": {}
        },
        "ServiceConnect": {},
        "CreateIndex": 11,
        "ModifyIndex": 11
    }
]
```
参考连接：
[prometheus-book](https://yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/sd/service-discovery-with-consul)
[https://www.jianshu.com/p/f8746b81d65d](https://www.jianshu.com/p/f8746b81d65d)
[运维之美](https://www.hi-linux.com/posts/28048.html)
[纯洁的微笑](http://www.ityouknow.com/springcloud/2018/07/20/spring-cloud-consul.html)
