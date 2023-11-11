

----
## 1. consul对外暴露了4种通讯接口
### 1.1 RPC
主要用于内部通讯Gossip/日志分发/选主等

###  1.2 HTTP API
服务发现/健康检查/KV存储等几乎所有功能
默认端口为8500

### 1.3 Consul Commands (CLI)
consul命令行工具可以与consul agent进行连接，提供一部分consul的功能。
实时上Consul CLI默认就是调用的HTTP API来与consul集群进行通讯。
可以通过配置CONSUL_HTTP_ADDR修改Consul CLI连接的目标地址

```bash
CONSUL_HTTP_ADDR=http://127.0.0.1:8500
```
###  1.4 DNS
仅用于服务查询

## 2 服务发现
### 2.1 DNS方式

```bash
$ dig @127.0.0.1 -p 8600 web.service.consul

;; QUESTION SECTION:
;web.service.consul.        IN  A

;; ANSWER SECTION:
web.service.consul. 0   IN  A   127.0.0.1
```

我们可以通过cosul提供的DNS接口来获取当前的service“web” 对应的可用节点(详细用法见参考资料4)
DNS方式要求使用方主动进行DNS解析，是主动请求的过程。它对线上服务节点的变化，反应是延迟的。

### 2.2 Watch方式

 - watch采用HTTP长轮训(long polling)实现的。
### 2.3 服务注册
服务注册可以通过 服务注册接口 /agent/service/register 很容易做到
## 3 服务注销
节点和服务的注销可以使用HTTP API:

注销任意节点和服务：`/catalog/deregister`
注销当前节点的服务：`/agent/service/deregister/:service_id`

**注意**：

如果注销的服务还在运行，则会再次同步到catalog中，因此应该只在agent不可用时才使用catalog的注销API。

节点在宕机时状态会变为failed，默认情况下72小时后会被从集群移除。

如果某个节点不继续使用了，也可以在本机使用consul leave命令，或者在其它节点使用consul force-leave 节点Id，则节点上的服务和健康检查全部注销。
