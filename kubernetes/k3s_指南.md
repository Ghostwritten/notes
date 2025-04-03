#  k3s 指南
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e7f223839799fe80ddb6fe6ab56363b.png)






##  简介
轻量级Kubernetes
k3s是经CNCF一致性认证的Kubernetes发行版，专为物联网及边缘计算设计。易于安装，全部在不到 100 MB 的二进制文件中。

非常适合：

- 边缘（Edge）
- 物联网（IoT）
- CI
- Development
- ARM
- 嵌入 K8s


## 什么是 K3s?

K3s 是一个完全兼容的 Kubernetes 发行版，具有以下增强功能：

- 打包为单个二进制文件。
- 基于 `sqlite3` 作为默认存储机制的轻量级存储后端。`etcd3`、`MySQL`、`Postgres` 也仍然可用。
- 包裹在简单的启动器中，可以处理 TLS 和选项的许多复杂性。
- 默认情况下安全，轻量级环境的合理默认值。
- 添加了简单但强大的“包含电池”功能，例如：本地存储提供程序、服务负载均衡器、- Helm 控制器和 Traefik 入口控制器。
- 所有 Kubernetes 控制平面组件的操作都封装在一个二进制文件和进程中。这允许 K3s 自动化和管理复杂的集群操作，例如分发证书。
- 外部依赖性已最小化（只需要现代内核和 cgroup 挂载）。K3s 打包了所需的依赖，包括：
  - containerd
  - Flannel
  - CoreDNS
  - CNII
  - 主机实用程序（iptables、socat 等）
  - Ingress controller (traefik)
  - 嵌入式服务负载均衡器（Embedded service loadbalancer）
  - 嵌入式网络策略控制器（Embedded network policy controller）


##  技术亮点
- 单进程架构简化部署
- 移除各种非必需代码，减少资源占用
- TLS证书管理
- 内置Containerd
- 内置运行rootfs
- 内置Helm Charts管理机制
- 内置L4/L7 LB支持

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/46befa9f5eabd1fa0d53c54e9391773c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82cf4d25e1eeb1dcaae4309c3c7eed42.png)


##  架构
本页介绍了高可用 K3s 服务器集群的架构以及它与单节点服务器集群的区别。它还描述了代理节点如何注册到 K3s 服务器。

`k3s server`服务器节点定义为运行命令的机器（裸机或虚拟机） 。工作节点定义为运行`k3s agent`命令的机器。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc6b8e75cb850762067b122e0613a2af.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b54a73e4f556f12d7de3200587a6b9cd.png)

##  发展趋势
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c59c873a85c9923387017ee16a28553c.png)
##  云边缘
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c66eddb625e8f6cefb7c1676f3da3d42.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e2e5d920b064cce74b809b4edb9e7e04.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c60c6f94ae0e50e8856f35f4180c00d7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2c4cfed73a3837b5704fcf5a4d40bdb6.png)

##  k3s 周边
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d161a265853951d56a2331e8b37af6c4.png)



###  单节点
下图显示了具有单节点 K3s 服务器和嵌入式 SQLite 数据库的集群示例。

在此配置中，每个代理节点都注册到同一个服务器节点。K3s 用户可以通过调用服务器节点上的 K3s API 来操作 Kubernetes 资源。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7ab92a918b10429d1add443a53f88ed6.png)
###  高可用
单服务器集群可以满足各种用例，但对于 Kubernetes 控制平面的正常运行时间至关重要的环境，您可以在 HA 配置中运行 K3s。一个 HA K3s 集群包括：

- 两个或多个服务于 Kubernetes API 并运行其他控制平面服务的服务器节点
- 外部数据存储（与单服务器设置中使用的嵌入式 SQLite 数据存储相对）

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/287f7488b67f4d6595ca162f198f0d4d.png)

##  代理注册
在高可用服务器配置中，每个节点还必须使用固定的注册地址向 Kubernetes API 注册，如下图所示。

注册后，代理节点直接与其中一个服务器节点建立连接。


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b54f9397c7919629c0e4ac86e2a356f8.png)

代理节点注册到由`k3s agent`进程启动的 `websocket` 连接，连接由作为代理进程一部分运行的客户端负载均衡器维护。

代理将使用节点集群密码以及随机生成的节点密码向服务器注册，存储在`/etc/rancher/node/password`. 服务器会将各个节点的密码存储为 `Kubernetes` 机密，任何后续尝试都必须使用相同的密码。节点密码秘密存储在`kube-system`命名空间中，名称使用模板`<host>.node-password.k3s`。

> 注意：在 `K3s v1.20.2`之前，服务器将密码存储在磁盘上的`/var/lib/rancher/k3s/server/cred/node-passwd`.

如果`/etc/rancher/node`删除了代理的目录，则应为代理重新创建密码文件，或者从服务器中删除条目。

`--with-node-id`通过使用该标志启动 K3s 服务器或代理，可以将唯一节点 ID 附加到主机名。


##  部署清单
位于目录路径的[清单](https://github.com/k3s-io/k3s/tree/master/manifests)`/var/lib/rancher/k3s/server/manifests`在构建时被捆绑到 `K3s` 二进制文件中。这些将由[rancher/helm-controller](https://github.com/k3s-io/helm-controller#helm-controller)在运行时安装。


##  集群要求
K3s 非常轻量级，但有一些最低要求，如下所述。

无论您是将 K3s 集群配置为在 Docker 还是 Kubernetes 设置中运行，运行 K3s 的每个节点都应满足以下最低要求。您可能需要更多资源来满足您的需求。

- 两个节点不能具有相同的主机名。
如果您的所有节点都具有相同的主机名，请使用该`--with-node-id`选项为每个节点附加一个随机后缀，或者设计一个唯一名称以与您添加到集群的每个节点一起传递`--node-name`或`$K3S_NODE_NAME`为每个节点传递。
- K3s 有望在大多数现代 Linux 系统上运行。
一些操作系统有特定的要求：

  - 如果您使用的是`(Red Hat/CentOS) Enterprise Linux`，请按照以下[步骤](https://docs.k3s.io/advanced#additional-preparation-for-red-hatcentos-enterprise-linux)进行其他设置。
  - 如果您使用的是`Raspberry Pi OS`，请按照以下[步骤](https://docs.k3s.io/advanced#additional-preparation-for-raspberry-pi-os)切换到旧版 iptables。
  
- 硬件要求会根据您的部署规模进行扩展。此处概述了最低建议。
   - RAM：至少 512MB（我们建议至少 1GB）
  - CPU：至少 1 个

[K3s Resource Profiling](https://docs.k3s.io/reference/resource-profiling)捕获测试结果以确定 K3s 代理、具有工作负载的 K3s 服务器和具有一个代理的 K3s 服务器的最低资源需求。它还包含对 K3s 服务器和代理利用率影响最大的分析，以及如何保护集群数据存储免受代理和工作负载的干扰。

-  K3s 的性能取决于数据库的性能。为确保最佳速度，我们建议尽可能使用 `SSD`。使用 SD 卡或 eMMC 的 ARM 设备的磁盘性能会有所不同。

- K3s 服务器需要 `6443` 端口才能被所有节点访问。

- 当使用 `Flannel VXLAN` 时，节点需要能够通过 `UDP` 端口 `8472` 访问其他节点，或者当使用 `Flannel Wireguard` 后端时，节点需要能够通过 `UDP` 端口 `51820` 和 `51821`（使用 IPv6 时）访问其他节点。该节点不应侦听任何其他端口。K3s 使用反向隧道，以便节点与服务器建立出站连接，并且所有 kubelet 流量都通过该隧道运行。但是，如果您不使用 Flannel 并提供自己的自定义 CNI，那么 `K3s` 不需要 `Flannel` 所需的端口。

如果您希望使用指标服务器，则需要在每个节点上打开端口 10250。如果您计划使用嵌入式 etcd 实现高可用性，则服务器节点必须可以在端口 2379 和 2380 上相互访问。


>**重要提示**：节点上的 `VXLAN` 端口不应暴露给外界，因为它会打开您的集群网络以供任何人访问。在禁止访问端口 `8472` 的防火墙/安全组后面运行您的节点。 警告： `Flannel` 依赖于`Bridge CNI` 插件来创建交换流量的 `L2` 网络。具有 `NET_RAW` 功能的流氓 `pod` 可以滥用该 L2 网络来发起攻击，例如`ARP` 欺骗。因此，如kubernetes 文档中所述，请设置一个受限配置文件，在不可信任的 pod 上禁用 `NET_RAW`。

K3s 服务器节点的入站规则
|协议	|港口|	资源|描述
|--|--|--|--|
|TCP|	6443	|K3s代理节点| 	Kubernetes API 服务器
|UDP|	8472	|K3s 服务器和代理节点	| 仅 Flannel VXLAN 需要
|UDP|	51820|	K3s 服务器和代理节点| 	仅 Flannel Wireguard 后端需要
|UDP|	51821	|K3s 服务器和代理节点| 	只有使用 IPv6 的 Flannel Wireguard 后端才需要
|TCP|	10250|	K3s 服务器和代理节点	| Kubelet 指标
|TCP|	2379-2380|	K3s 服务器节点	| 仅适用于具有嵌入式 etcd 的 HA

通常允许所有出站流量。

###  大型集群
硬件要求取决于您的 K3s 集群的大小。对于生产和大型集群，我们建议使用具有外部数据库的高可用性设置。对于生产中的外部数据库，建议使用以下选项：

- MySQL
- PostgreSQL

#### cpu 与 内存
以下是高可用性 K3s 服务器中节点的最低 CPU 和内存要求：
|Deployment Size|	Nodes	|VCPUS	|RAM|
|--|--|--|--|
|Small	|Up to 10|	2|	4 GB|
|Medium|	Up to 100|	4|	8 GB|
|Large|	Up to 250	|8|	16 GB|
|X-Large	|Up to 500|	16|	32 GB|
|XX-Large|	500+	|32|	64 GB|

集群性能取决于数据库性能。为确保最佳速度，我们建议始终使用 `SSD` 磁盘来支持您的 `K3s` 集群。在云提供商上，您还需要使用允许最大 `IOPS` 的最小大小。

您应该考虑增加集群 `CIDR` 的子网大小，这样您就不会用完 `Pod` 的 `IP`。您可以通过`--cluster-cidr`在启动时将选项传递给 K3s 服务器来做到这一点。

###  数据库
K3s 支持不同的数据库，包括 `MySQL`、`PostgreSQL`、`MariaDB` 和 `etcd`，以下是运行大型集群所需的数据库资源的大小指南：
|Deployment Size|	Nodes|	VCPUS	|RAM|
|--|--|--|--|
|Small|	Up to 10|	1	|2 GB|
|Medium	|Up to 100|	2|	8 GB|
|Large	|Up to 250|	4	|16 GB|
|X-Large	|Up to 500|	8	|32 GB|
|XX-Large|	500+|	16|	64 GB|


##  配置选项
###  使用安装脚本
如[快速入门指南](https://docs.k3s.io/quick-start)中所述，您可以使用`https://get.k3s.io`上提供的安装脚本将 K3s 作为服务安装在基于 `systemd` 和 `openrc` 的系统上。

您可以结合使用`INSTALL_K3S_EXEC`、`K3S_`环境变量和命令标志来配置安装。

为了说明这一点，以下命令都会导致在没有 `flannel` 和使用令牌的情况下注册服务器的相同行为：

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--flannel-backend none --token 12345" sh -s -
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --flannel-backend none" K3S_TOKEN=12345 sh -s -
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - --flannel-backend none
curl -sfL https://get.k3s.io | K3S_TOKEN=12345 sh -s - server --flannel-backend none
curl -sfL https://get.k3s.io | sh -s - --flannel-backend none --token 12345
```
有关所有环境变量的详细信息，请参阅[环境变量](https://docs.k3s.io/reference/env-variables)。

###  二进制配置
如前所述，安装脚本主要关注将 K3s 配置为作为服务运行。
如果您选择不使用该脚本，您只需从我们的[发布页面](https://github.com/k3s-io/k3s/releases/tag/v1.25.2+k3s1)下载二进制文件，将其放在您的路径上，然后执行即可运行 K3s。或者您可以安装 K3s 而不将其作为服务启用：

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_SKIP_ENABLE=true sh -
```
`K3S_`您可以通过环境变量以这种方式配置K3s ：

```bash
K3S_KUBECONFIG_MODE="644" k3s server
```
或命令标志：

```bash
k3s server --write-kubeconfig-mode 644
```
k3s 代理也可以这样配置：

```bash
k3s agent --server https://k3s.example.com --token mypassword
```
- 有关配置 K3s 服务器的详细信息，请参阅[服务器配置](https://docs.k3s.io/reference/server-config)。
- 有关配置 K3s 代理的详细信息，请参阅[代理配置](https://docs.k3s.io/reference/agent-config)。

您还可以使用该`--help`标志查看所有可用选项的列表。

###  配置文件
> [从v1.19.1+k3s1 开始可用](https://github.com/k3s-io/k3s/releases/tag/v1.19.1+k3s1)

除了使用环境变量和 CLI 参数配置 K3s 之外，K3s 还可以使用配置文件。

默认情况下，位于 的 YAML 文件中的值`/etc/rancher/k3s/config.yaml`将在安装时使用。

下面是一个基本`server`配置文件的示例：

```bash
write-kubeconfig-mode: "0644"
tls-san:
  - "foo.local"
node-label:
  - "foo=bar"
  - "something=amazing"
```
通常，CLI 参数映射到它们各自的 YAML 键，可重复的 CLI 参数表示为 YAML 列表。

下面显示了仅使用 CLI 参数的相同配置来演示这一点：

```bash
k3s server \
  --write-kubeconfig-mode "0644"    \
  --tls-san "foo.local"             \
  --node-label "foo=bar"            \
  --node-label "something=amazing"
```
也可以同时使用配置文件和 `CLI` 参数。在这些情况下，将从两个来源加载值，但 CLI 参数将优先。对于诸如 的可重复参数`--node-label`，CLI 参数将覆盖列表中的所有值。

最后，可以通过 CLI 参数-`-config FILE`, `-c FILE`或环境变量`$K3S_CONFIG_FILE`更改配置文件的位置。

上述所有选项都可以组合成一个示例。

在以下位置创建一个`config.yaml`文件`/etc/rancher/k3s/config.yaml`：

```bash
token: "secret"
debug: true
```
然后使用环境变量和标志的组合运行安装脚本：

```bash
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="server" sh -s - --flannel-backend none
```

或者，如果您已经安装了 K3s 二进制文件：

```bash
K3S_KUBECONFIG_MODE="644" k3s server --flannel-backend none
```

##  网络选项
以下网络选项：
- Flannel options
- Custom CNI
- Dual-stack installation
- IPv6-only-installation
- Distributed hybrid or multicloud cluster

###  Flannel options 
`flannel` 的默认后端是 `VXLAN`。要启用加密，请传递下面的 IPSec（Internet 协议安全）或 WireGuard 选项。

如果你想使用 WireGuard 作为你的 flannel 后端，它可能需要额外的内核模块。有关详细信息，请参阅WireGuard 安装指南。WireGuard 安装步骤将确保为您的操作系统安装适当的内核模块。在尝试利用 WireGuard flannel 后端选项之前，您需要在每个节点上安装 WireGuard，包括服务器和代理。wireguard后端将从 v1.26 中删除，以支持 Flannel 的本地wireguard-native后端。

我们建议用户尽快迁移到新的后端。当节点提出新配置时，迁移需要很短的停机时间。您应该遵循以下两个步骤：

1 - 更新所有控制平面节点中的 K3s 配置。配置文件/etc/rancher/k3s/config.yaml应该包括flannel-backend: wireguard-native而不是flannel-backend: wireguard.

2 - 重新启动所有节点。


参考：

 - [https://k3s.io/](https://k3s.io/)
 - [https://docs.rancher.cn/k3s/](https://docs.rancher.cn/k3s/)
 - [https://github.com/k3s-io/k3s](https://github.com/k3s-io/k3s)
 - [k3s进阶之路](https://space.bilibili.com/430496045/channel/seriesdetail?sid=801855)
 - [k3s从入门到进阶全攻略](https://mp.weixin.qq.com/mp/homepage?__biz=MzIxMDA5MzA4MQ==&hid=1&sn=59d1e8fc0fe66404c2b4d0486efecd5e&scene=1&devicetype=iOS13.7&version=18001d30&lang=zh_CN&nettype=3G%20&ascene=7&session_us=gh_342ccd4fb9db&fontScale=100&wx_header=3)
 - [k3s 问题讨论文档](https://shimo.im/docs/HjYY8c836jPgPJK9/read)

