# falco 入门
tags： 安全




---

[![印度电影已经在《宿敌》中将教材政治渗透问题得到体现](https://img-blog.csdnimg.cn/7906f1d234cc4c858152ea10bca15be6.png)](https://movie.douban.com/subject/35882880/)
*印度电影已经在《宿敌》中将教材政治渗透问题得到体现。*



## 1. 简介
[Falco](https://falco.org/docs/#what-is-falco) 项目是最初由[Sysdig](https://sysdig.com/), Inc构建的开源运行时安全工具。Falco 被捐赠给 CNCF，现在是 [CNCF 孵化项目](https://www.cncf.io/blog/2020/01/08/toc-votes-to-move-falco-into-cncf-incubator/)。

Falco 是一个 Linux 安全工具，它使用系统调用来保护和监控系统。Falco 可用于 Kubernetes 运行时安全性。运行 Falco 最安全的方法是将 Falco 直接安装在主机系统上，这样 Falco 与 Kubernetes 隔离，以防万一。然后可以通过在 Kubernetes 中运行的只读代理来使用 Falco 警报。

您还可以使用 `Helm` 在 Kubernetes 中将 Falco 作为守护程序集直接运行。

Falco 通过以下方式使用系统调用来保护和监控系统：

 - 在运行时从内核解析 Linux 系统调用
 - 针对强大的规则引擎断言流
 - 违反规则时发出警报

{% youtube %}
https://www.youtube.com/watch?v=u409G5PsO1w
{% endyoutube %}


## 2. 特点
Falco 的主要特点：

 - 加强安全性——创建由上下文丰富且灵活的引擎驱动的安全规则，以定义意外的应用程序行为。
 - 降低风险 – 通过将 Falco 插入您当前的安全响应工作流程和流程，立即响应违反政策的警报。
 - 利用最新规则 - 使用来自社区的恶意活动和 CVE 漏洞检测发出警报。

## 3. 检测
Falco 附带了一组默认规则，用于检查内核是否存在异常行为，例如：

 - 使用特权容器提权
 - 使用诸如setns
 - 读取/写入知名目录，例如/etc, /usr/bin,/usr/sbin等
 - 创建符号链接
 - 所有权和模式更改
 - 意外的网络连接或套接字突变
 - 产生的进程使用execve
 - 执行 shell 二进制文件，例如sh, bash, csh,zsh等
 - 执行 SSH 二进制文件，例如ssh, scp,sftp等
 - 变异 Linux coreutils可执行文件
 - 变异登录二进制文件
 - 变异shadowutil或passwd可执行文件，例如shadowconfig, pwck, chpasswd, getpasswd, change, useradd, etc, 等。

## 4. 规则
规则是 Falco 反对的项目。它们在 Falco 配置文件中定义，代表您可以在系统上检查的事件。请参阅 Falco规则。

## 5. 警报
警报是可配置的下游操作，可以像日志记录一样简单，也可以像STDOUT向客户端传递 gRPC 调用一样复杂。有关配置、理解和开发警报的更多信息，请参阅Falco 警报。Falco 可以将警报发送到：

 - 标准输出
 - 一份文件
 - 系统日志
 - 一个衍生的程序
 - HTTP[s] 端点
 - 通过 gRPC API 的客户端


## 6. 组件
Falco 由四个主要组件组成：

 - `Userspace program` - 是falco可用于与 Falco 交互的 CLI 工具。用户空间程序处理信号，解析来自 Falco驱动程序的信息，并发送警报。
 - `Configuration` - 定义 Falco 的运行方式、断言的规则以及如何执行警报。有关详细信息，请参阅配置。
 - `Driver` - 是一种遵循 Falco 驱动程序规范并发送系统调用信息流的软件。不安装驱动程序就无法运行 Falco。目前，Falco 支持以下驱动程序：
   - （默认）基于C++ 库构建libscap的内核模块libsinsp
   - 从相同模块构建的 BPF 探针
   - 用户空间检测

有关详细信息，请参阅Falco 驱动程序。

- `Plugins` - 允许用户通过添加新的事件源和可以从事件中提取信息的新字段来扩展 falco 库/falco 可执行文件的功能。有关更多信息，请参阅插件。

##  7. 架构
Falco 可以检测任何涉及进行 Linux 系统调用的行为并发出警报。Falco 警报是根据调用进程的特定系统调用、参数和属性触发的。Falco 在用户空间和内核空间运行。系统调用由 Falco 内核模块解释。然后使用用户空间中的库分析系统调用。然后使用配置了 Falco 规则的规则引擎过滤事件。然后向配置为 Syslog、文件、标准输出等的输出警告可疑事件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a513d409570e4128991465974d02078b.png)

##  8. 下载
两种下载和运行 Falco 的方式：

 - 直接在 Linux 主机上运行 Falco
 - 在容器中运行 Falco 用户空间程序，并在底层主机上安装驱动程序。

|| development | stable  |
|-|-------------|---------|
| rpm         | [![rpm-dev](https://img-blog.csdnimg.cn/7eeacbe2c432462eaa6f34d6523f65fe.png) ](https://download.falco.org/?prefix=packages/rpm-dev/)|[![在这里插入图片描述](https://img-blog.csdnimg.cn/7733df4c62404b95a2646afa364f3b87.png)](https://download.falco.org/?prefix=packages/rpm/)
| deb         | [![deb-dev](https://img-blog.csdnimg.cn/025cb1bb5a5c44e2b8c2d4f4168e36ae.png)](https://download.falco.org/?prefix=packages/deb-dev/stable/) |[![在这里插入图片描述](https://img-blog.csdnimg.cn/a6312eced7174ff086d8e349df3346db.png)](https://download.falco.org/?prefix=packages/deb/stable/)
|
| binary      | [![bin-dev](https://img-blog.csdnimg.cn/4727d3779a834bc2ba1d5c73ee1d2d16.png)](https://download.falco.org/?prefix=packages/bin-dev/x86_64/)|[![在这里插入图片描述](https://img-blog.csdnimg.cn/1e89a9d6da2440a88bc0434672f8e1db.png)](https://download.falco.org/?prefix=packages/bin/x86_64/)
|


下载容器镜像
| 标签  | 拉命令                                                     | 描述                                        |
|-----|---------------------------------------------------------|-------------------------------------------|
| 最新的 | docker pull falcosecurity/falco-no-driver:latest        | 最新版本                                      |
| 版本  | docker pull falcosecurity/falco-no-driver:<version>     | 特定版本的 Falco，例如0.32.1                      |
| 最新的 | docker pull falcosecurity/falco-driver-loader:latest    | falco-driver-loader带有构建工具链的最新版本           |
| 版本  | docker pull falcosecurity/falco-driver-loader:<version> | 特定版本，falco-driver-loader例如0.32.1带有构建工具链的  |
| 最新的 | docker pull falcosecurity/falco:latest                  | falco-driver-loader包含的最新版本                |
| 版本  | docker pull falcosecurity/falco:<version>               | 特定版本的 Falco，例如0.32.1包含falco-driver-loader |

##  9. 安装
###  9.1 Debian/Ubuntu
配置信任 falcosecurity GPG 密钥，配置 apt 存储库，并更新软件包列表：

```bash
curl -s https://falco.org/repo/falcosecurity-3672BA8F.asc | apt-key add -
echo "deb https://download.falco.org/packages/deb stable main" | tee -a /etc/apt/sources.list.d/falcosecurity.list
apt-get update -y
```
Install kernel headers:

```bash
apt-get -y install linux-headers-$(uname -r)
```
安装 Falco：

```bash
apt-get install -y falco
```
现在安装了 Falco、内核模块驱动程序和默认配置。Falco 作为一个系统单元运行。

有关如何使用 Falco 管理、运行和调试的信息，请参阅运行。

###  9.2 CentOS/RHEL/Fedora/Amazon Linux
配置信任 falcosecurity GPG 密钥并配置 yum 存储库：

```bash
rpm --import https://falco.org/repo/falcosecurity-3672BA8F.asc
curl -s -o /etc/yum.repos.d/falcosecurity.repo https://falco.org/repo/falcosecurity-rpm.repo
```
Install kernel headers:

```bash
yum -y install kernel-devel-$(uname -r)
```
安装 Falco：

```bash
yum -y install falco
```
卸载 Falco：

```bash
yum erase falco
```
### 9.3 openSUSE
配置信任 falcosecurity GPG 密钥并配置 zypper 存储库：

```bash
rpm --import https://falco.org/repo/falcosecurity-3672BA8F.asc
curl -s -o /etc/zypp/repos.d/falcosecurity.repo https://falco.org/repo/falcosecurity-rpm.repo
```
Install kernel headers:

```bash
zypper -n install kernel-default-devel-$(uname -r | sed s/\-default//g)
```

> 注意— 如果上述命令未找到该包，您可能需要运行zypper -n dist-upgrade以修复它。可能需要重新启动系统。

安装 Falco：

```bash
zypper -n install falco
```
现在安装了 Falco、内核模块驱动程序和默认配置。Falco 作为一个系统单元运行。

有关如何使用 Falco 管理、运行和调试的信息，请参阅运行。
卸载 Falco：

```bash
zypper rm falco
```
###  9.4 Linux 通用（二进制包） 
下载最新的二进制文件：

```bash
curl -L -O https://download.falco.org/packages/bin/x86_64/falco-0.32.1-x86_64.tar.gz
```
安装 Falco：

```bash
tar -xvf falco-0.32.1-x86_64.tar.gz
cp -R falco-0.32.1-x86_64/* /
```
安装以下依赖项：

您的发行版的内核头文件
安装驱动程序后，您可以手动运行falco.

安装驱动程序
安装驱动程序的最简单方法是使用`falco-driver-loader`脚本。

默认情况下，它首先尝试使用`dkms`. 如果不可能，那么它会尝试将预建的下载到`~/.falco/`. 如果找到内核模块，则将其插入。

如果要安装 eBPF 探针驱动程序，请运行`falco-driver-loader bpf`. 它首先尝试在本地构建 eBPF 探针，否则将预构建下载到~/.falco/.

如果您使用的是 eBPF 探针，为了确保性能不会下降，请确保

 - 您的内核已`CONFIG_BPF_JIT`启用
 - `net.core.bpf_jit_enable`设置为 1（启用 BPF JIT 编译器）
 - 这可以通过验证`sysctl -n net.core.bpf_jit_enable`

可配置选项：

`DRIVERS_REPO`- 设置此环境变量以覆盖预构建内核模块和 eBPF 探针的默认存储库 URL，不带斜杠。
即，`https://myhost.mydomain.com`或者如果服务器具有子目录结构`https://myhost.mydomain.com/drivers`。
驱动程序需要使用以下结构托管： `/${driver_version}/falco_${target}_${kernelrelease}_${kernelversion}`.`[ko|o]whereko`和分别o代表内核模块和eBPF探针。
例如，`/a259b4bf49c3330d9ad6c3eed9eb1a31954259a6/falco_amazonlinux2_4.14.128-112.105.amzn2.x86_64_1.ko`.
该`falco-driver-loader`脚本使用上述格式获取驱动程序。

###  9.5 minikube 安装  falco
在本地环境中在 Kubernetes 上使用 Falco 的最简单方法是在[Minikube](https://minikube.sigs.k8s.io/docs/start/)上。

当minikube使用默认`--driver`参数运行时，Minikube 会创建一个运行各种 Kubernetes 服务的 VM 和一个运行 Pod 等的容器框架。通常，不可能直接在 minikube VM 上构建 Falco 内核模块，因为 VM 没有包括正在运行的内核的内核头文件。

[为了解决这个问题，从 `Falco 0.13.1` 开始，可以在https://s3.amazonaws.com/download.draios.com](https://s3.amazonaws.com/download.draios.com)获得最后 10 个 minikube 版本的预构建内核模块。这允许下载后备步骤通过可加载的内核模块成功。Falco 现在在每个新的 Falco 版本中都支持 10 个最新版本的 minikube。Falco 目前保留以前构建的内核模块供下载，并继续提供有限的历史支持。

1. 使用 VM 驱动程序使用 Minikube 创建集群，在本例中为 Virtualbox：

```bash
minikube start --driver=virtualbox
```
2. 检查所有 pod 是否正在运行：

```bash
kubectl get pods --all-namespaces
```
3. 将稳定图表添加到 Helm 存储库：

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
```
4. 使用 Helm 安装 Falco：

```bash
helm install falco falcosecurity/falco
```
输出：

```bash
AME: falco
LAST DEPLOYED: Wed Jan 20 18:24:08 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Falco agents are spinning up on each node in your cluster. After a few
seconds, they are going to start monitoring your containers looking for
security issues.


No further action should be required.


Tip:
You can easily forward Falco events to Slack, Kafka, AWS Lambda and more with falcosidekick.
Full list of outputs: https://github.com/falcosecurity/charts/falcosidekick.
You can enable its deployment with `--set falcosidekick.enabled=true` or in your values.yaml.
See: https://github.com/falcosecurity/charts/blob/master/falcosidekick/values.yaml for configuration values.
```
5. 检查日志以确保 Falco 正在运行：

```bash
kubectl logs -l app=falco -f
```
输出：

```bash
* Trying to dkms install falco module with GCC /usr/bin/gcc-5
DIRECTIVE: MAKE="'/tmp/falco-dkms-make'"
* Running dkms build failed, couldn't find /var/lib/dkms/falco/5c0b863ddade7a45568c0ac97d037422c9efb750/build/make.log (with GCC /usr/bin/gcc-5)
* Trying to load a system falco driver, if present
* Success: falco module found and loaded with modprobe
Wed Jan 20 12:55:47 2021: Falco version 0.27.0 (driver version 5c0b863ddade7a45568c0ac97d037422c9efb750)
Wed Jan 20 12:55:47 2021: Falco initialized with configuration file /etc/falco/falco.yaml
Wed Jan 20 12:55:47 2021: Loading rules from file /etc/falco/falco_rules.yaml:
Wed Jan 20 12:55:48 2021: Loading rules from file /etc/falco/falco_rules.local.yaml:
Wed Jan 20 12:55:49 2021: Starting internal webserver, listening on port 8765
```

### 9.6 kind 安装 falco
[kind](https://kind.sigs.k8s.io/docs/)允许您在本地计算机上运行 Kubernetes。此工具要求您 安装和配置Docker 。目前不能直接在带有 Linuxkit 的 Mac 上运行，但这些说明适用于运行kind.

在kind集群上运行 Falco 如下：

1. 创建一个配置文件。例如：kind-config.yaml

2. 将以下内容添加到文件中：

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraMounts:
    # allow Falco to use devices provided by the kernel module
  - hostPath: /dev
    containerPath: /dev
    # allow Falco to use the Docker unix socket
  - hostPath: /var/run/docker.sock
    containerPath: /var/run/docker.sock

```
3. 通过指定配置文件创建集群：

```bash
kind create cluster --config=./kind-config.yaml
```
4. 在 kind 集群中的一个节点上安装Falco。要将 Falco 安装为 Kubernetes 集群上的`daemonset`，请使用 Helm，方法同上。请参阅[https://github.com/falcosecurity/charts/tree/master/falco](https://github.com/falcosecurity/charts/tree/master/falco)。



##  10. 升级
根据您选择的安装方法，在将 Falco 升级到最新版本之前，您首先必须删除活动内核模块：

```bash
rmmod falco
```
###  10.1 Debian/Ubuntu
如果您apt按照 Falco 0.27.0 或更早版本的说明配置了存储库，则可能需要更新存储库 URL：

```bash
sed -i 's,https://dl.bintray.com/falcosecurity/deb,https://download.falco.org/packages/deb,' /etc/apt/sources.list.d/falcosecurity.list
apt-get clean
apt-get -y update
```
检查存在的`apt-get update`日志`https://download.falco.org/packages/deb`。

如果您按照提供的说明安装了 Falco ：

```bash
apt-get --only-upgrade install falco
```
###  10.2 CentOS/RHEL/Fedora/Amazon Linux
如果您yum按照 Falco 0.27.0 或更早版本的说明配置了存储库，则可能需要更新存储库 URL

```bash
sed -i 's,https://dl.bintray.com/falcosecurity/rpm,https://download.falco.org/packages/rpm,' /etc/yum.repos.d/falcosecurity.repo
yum clean all
```
然后检查`falcosecurity-rpm`存储库是否指向[https://download.falco.org/packages/rpm/](https://download.falco.org/packages/rpm/)：

```bash
yum repolist -v falcosecurity-rpm
```
如果您按照提供的说明安装了 Falco ：

检查更新：

```bash
yum check-update
```
如果有更新的 Falco 版本可用

```bash
yum update falco
```
###  10.3 openSUSE 
如果您zypper按照 `Falco 0.27.0` 或更早版本的说明配置了存储库，则可能需要更新存储库 URL：

```bash
sed -i 's,https://dl.bintray.com/falcosecurity/rpm,https://download.falco.org/packages/rpm,' /etc/zypp/repos.d/falcosecurity.repo
zypper refresh
```
然后检查`falcosecurity-rpm`存储库是否指向[https://download.falco.org/packages/rpm/](https://download.falco.org/packages/rpm/)：

```bash
zypper lr falcosecurity-rpm
```
如果您按照提供的说明安装了 Falco ：

```bash
zypper update falco
```

##  11. 部署
###  11.1 Kubernetes 
Falco 可以作为[DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)部署在 Kubernetes 中，以监控集群每个节点中的系统事件。

###  11.2 helm
在 Kubernetes 中安装 Falco 的最简单方法之一是使用[Helm](https://v3.helm.sh/docs/intro/install/)。Falco 社区支持官方 [helm chart](https://github.com/falcosecurity/charts/tree/master/falco)。

###  11.3 DaemonSet
Falco 也可以手动安装在 Kubernetes 中。在这种情况下，您负责提供 DaemonSet 对象 YAML 定义并将其部署到您的集群中。有关更多详细信息，您可以在此处找到[示例](https://github.com/falcosecurity/deploy-kubernetes/tree/main/kubernetes/falco/templates)。



##  12. 运行
### 12.1 将 Falco 作为 service 运行

```bash
systemctl enable falco
systemctl start falco
```

您还可以使用 . 查看 Falco 日志journalctl。

```bash
journalctl -fu falco
```

###   12.2 Docker 中运行
即使使用容器镜像，Falco 也需要在主机上安装内核头文件作为正确构建驱动程序（内核模块或eBPF 探针）的先决条件。当预建驱动程序已经可用时，不需要此步骤。

Falco 提供了一组官方docker 镜像。图像可以通过以下两种方式使用：

 - 最低特权（推荐）
 - 完全特权

#### 12.2.1 最低特权（推荐）
a. 安装内核模块：

 - 可以直接在主机上使用官方的安装方式
 - 或者，您可以临时使用特权容器在主机上安装驱动程序：

```bash
docker pull falcosecurity/falco-driver-loader:latest
docker run --rm -i -t \
    --privileged \
    -v /root/.falco:/root/.falco \
    -v /proc:/host/proc:ro \
    -v /boot:/host/boot:ro \
    -v /lib/modules:/host/lib/modules \
    -v /usr:/host/usr:ro \
    -v /etc:/host/etc:ro \
    falcosecurity/falco-driver-loader:latest
```
`falcosecurity/falco-driver-loader`图像只是包装了脚本`falco-driver-loader`。

b. 使用 Docker 以最小权限原则在容器中运行 Falco ：

```bash
docker pull falcosecurity/falco-no-driver:latest
docker run --rm -i -t \
    -e HOST_ROOT=/ \
    --cap-add SYS_PTRACE --pid=host $(ls /dev/falco* | xargs -I {} echo --device {}) \
    -v /var/run/docker.sock:/var/run/docker.sock \
    falcosecurity/falco-no-driver:latest
```

如果您在启用了 AppArmor LSM 的系统（例如 Ubuntu）上运行 Falco，您还需要传递--`security-opt apparmor:unconfine`d给`docker run`上面的命令。

您可以使用以下命令验证您是否启用了 AppArmor：

```bash
docker info | grep -i apparmor
```
请注意，每个 CPU `s /dev/falco* | xargs -I {} echo --device {}`输出一个`--device /dev/falcoX`选项（即，仅由 Falco 的内核模块创建的设备）。此外，`-e HOST_ROOT=/`这是必要的，因为`--device`无法将设备重新映射到`/host/dev/`.

要使用 `eBPF` 驱动程序以最低权限模式运行 `Falco`，我们列出了所有必需的功能：

 - 在小于 5.8 的内核上，Falco 需要`CAP_SYS_ADMIN`，`CAP_SYS_RESOURCE`、`CAP_SYS_PTRACE`
 - 在`kernels >=5.8` 上，`CAP_BPF`并且`CAP_PERFMON`被分离出来`CAP_SYS_ADMIN`，因此所需的功能是`CAP_BPF`, `CAP_PERFMON`, `CAP_SYS_RESOURCE`, `CAP_SYS_PTRACE`. 不幸的是，Docker 还不支持通过该`--cap-add`选项添加两个新引入的功能。出于这个原因，我们继续使用`CAP_SYS_ADMIN`，因为它仍然允许执行`CAP_BPF`和授予的相同操作`CAP_PERFMON`。在不久的将来，Docker 将支持添加这两个功能，我们将能够替换`CAP_SYS_ADMIN`.


安装 eBPF 探针

```bash
docker pull falcosecurity/falco-driver-loader:latest
docker run --rm -i -t \
    --privileged \
    -v /root/.falco:/root/.falco \
    -v /proc:/host/proc:ro \
    -v /boot:/host/boot:ro \
    -v /lib/modules:/host/lib/modules:ro \
    -v /usr:/host/usr:ro \
    -v /etc:/host/etc:ro \
    falcosecurity/falco-driver-loader:latest bpf
```
然后，运行 `Falco`

```bash
docker pull falcosecurity/falco-no-driver:latest
docker run --rm -i -t \
    --cap-drop all \
    --cap-add sys_admin \
    --cap-add sys_resource \
    --cap-add sys_ptrace \
    -v /var/run/docker.sock:/host/var/run/docker.sock \
    -e FALCO_BPF_PROBE="" \
    -v /root/.falco:/root/.falco \
    -v /etc:/host/etc \
    -v /proc:/host/proc:ro \
    falcosecurity/falco-no-driver:latest

```
同样，如果您的系统启用了 `AppArmor LSM`，您将需要添加`--security-opt apparmor:unconfined`到最后一个命令。

####  12.2.2 完全特权
要使用具有完全权限的 Docker 在容器中运行 Falco，请使用以下命令。

如果您想将 Falco 与内核模块驱动程序一起使用：

```bash
docker pull falcosecurity/falco:latest
docker run --rm -i -t \
    --privileged \
    -v /var/run/docker.sock:/host/var/run/docker.sock \
    -v /dev:/host/dev \
    -v /proc:/host/proc:ro \
    -v /boot:/host/boot:ro \
    -v /lib/modules:/host/lib/modules:ro \
    -v /usr:/host/usr:ro \
    -v /etc:/host/etc:ro \
    falcosecurity/falco:latest
```
或者，您可以使用 eBPF 探针驱动程序：

```bash
docker pull falcosecurity/falco:latest
docker run --rm -i -t \
    --privileged \
    -e FALCO_BPF_PROBE="" \
    -v /var/run/docker.sock:/host/var/run/docker.sock \
    -v /proc:/host/proc:ro \
    -v /boot:/host/boot:ro \
    -v /lib/modules:/host/lib/modules:ro \
    -v /usr:/host/usr:ro \
    -v /etc:/host/etc:ro \
    falcosecurity/falco:latest

```
也可以在完全特权模式下使用`falco-no-driver`镜像。`falco-driver-loader`这在由于空间、资源、安全或策略限制而不允许完整 Falco 映像的环境中可能是可取的。您可以将 eBPF 探针或内核模块加载到其自己的临时容器中，如下所示：

```bash
docker pull falcosecurity/falco-driver-loader:latest
docker run --rm -i -t \
    --privileged \
    -v /var/run/docker.sock:/host/var/run/docker.sock \
    -v /dev:/host/dev \
    -v /proc:/host/proc:ro \
    -v /boot:/host/boot:ro \
    -v /lib/modules:/host/lib/modules:ro \
    -v /usr:/host/usr:ro \
    -v /etc:/host/etc:ro \
    falcosecurity/falco-driver-loader:latest
```
完成此操作后，或者如果您通过`falco-driver-loader`脚本而不是 Docker 映像在主机上永久安装了驱动程序，那么您可以简单地falco-no-driver以特权模式加载映像：

```bash
docker pull falcosecurity/falco-no-driver:latest
docker run --rm -i -t \
    --privileged \
    -v /var/run/docker.sock:/host/var/run/docker.sock \
    -v /dev:/host/dev \
    -v /proc:/host/proc:ro \
    falcosecurity/falco-no-driver:latest
```
要使用eBPF 探针`falco-no-driver`，`falco-driver-loader`您必须删除`-v /dev:/host/dev`（仅内核模块需要）并添加：

```bash
 -e FALCO_BPF_PROBE="" -v /root/.falco:/root/.falco \
```
其他可配置选项：

 - `DRIVER_REPO`- 请参阅安装驱动程序部分。
 - `SKIP_DRIVER_LOADER`-设置此环境变量以避免镜像启动`falco-driver-loader`时运行。`falcosecurity/falco`当驱动程序已经通过其他方式安装在主机上时很有用

###  12.3 Hot Reload 
这将重新加载 Falco 配置并重新启动引擎，而不会终止 pid。这对于在不杀死守护进程的情况下传播新的配置更改很有用。

```bash
kill -1 $(cat /var/run/falco.pid)
```
参考：

 - [falco getting started](https://falco.org/docs/getting-started/)
 - [sysdig falco](https://sysdig.com/opensource/falco/)
 - [Falco case studies](https://www.cncf.io/projects/falco/)

