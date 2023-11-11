#  kubernetes kubelet 配置
tags: kubelet
<!-- catalog: ~配置~ -->



[
![在这里插入图片描述](https://img-blog.csdnimg.cn/60087a6b79ca40e8a87cd2d0dd85a9d4.jpeg#pic_center)](https://www.callofduty.com/cn/zh)




##  1. 背景
kubeadm CLI 工具的生命周期与 kubelet 解耦， kubelet是运行在 Kubernetes 集群内每个节点上的守护进程。kubeadm CLI 工具由用户在 Kubernetes 初始化或升级时执行，而 kubelet 始终在后台运行。

由于 kubelet 是一个守护进程，它需要由某种 init 系统或服务管理器来维护。当使用 DEB 或 RPM 安装 kubelet 时，systemd 被配置为管理 kubelet。您可以改用其他服务管理器，但需要手动配置。

一些 kubelet 配置细节需要在集群中涉及的所有 kubelet 中相同，而其他配置方面需要基于每个 kubelet 进行设置，以适应给定机器的不同特征（例如操作系统、存储和网络） . 您可以手动管理 kubelet 的配置，但 kubeadm 现在提供了一种[Kubelet ConfigurationAPI](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/) 类型来 集中管理您的 kubelet 配置


##  2. 配置模式

###  2.1 kubeadm 配置 kubelet
将集群级别的配置传播到每个 kubelet
您可以为 kubelet 提供`kubeadm init`和`kubeadm join` 命令使用的默认值。有趣的示例包括使用不同的容器运行时或设置服务使用的默认子网。

如果您希望您的服务使用子网`10.96.0.0/12`作为服务的默认子网，您可以将-`-service-cidr`参数传递给 `kubeadm`：

```bash
kubeadm init --service-cidr 10.96.0.0/12
```
现在从该子网分配服务的虚拟 IP。您还需要使用标志设置 kubelet 使用的 DNS 地址`--cluster-dns`。对于集群中每个管理器和节点上的每个 kubelet，此设置需要相同。kubelet 提供了一个版本化、结构化的 API 对象，可以在 kubelet 中配置大部分参数，并将此配置推送到集群中每个正在运行的 kubelet。这个对象被称为 [Kubelet Configuration](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/)。允许用户指定标志，Kubelet Configuration例如集群 DNS IP 地址，以骆驼大小写键的值列表表示，如下例所示：

```bash
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
clusterDNS:
- 10.96.0.10
```


通过调用`kubeadm config print init-defaults --component-configs KubeletConfiguration`，您可以看到此结构的所有默认值。

#### 2.1.1 使用时的工作流程kubeadm init 
当您调用`kubeadm init`时，kubelet 配置会被编组到磁盘`/var/lib/kubelet/config.yaml`，同时也会上传到 集群命名空间中的ConfigMap名字为`kubelet-config` 。 集群中所有 kubelet 的基线集群范围配置`kube-system`也会写入 kubelet 配置文件`/etc/kubernetes/kubelet.conf`此配置文件指向允许 kubelet 与 API 服务器通信的客户端证书。这解决了 将集群级配置传播到每个 kubelet的需要。

为了解决 提供特定于实例的配置详细信息的第二种模式，kubeadm 将环境文件写入到`/var/lib/kubelet/kubeadm-flags.env`，其中包含一个标志列表，以便在 kubelet 启动时传递给它。标志在文件中显示如下：

```bash
KUBELET_KUBEADM_ARGS="--flag1=value1 --flag2=value2 ..."
```

除了启动 kubelet 时使用的标志外，该文件还包含动态参数，例如 cgroup 驱动程序以及是否使用不同的容器运行时套接字（--cri-socket）。

将这两个文件编组到磁盘后，如果您使用的是 systemd，kubeadm 会尝试运行以下两个命令：

```bash
systemctl daemon-reload && systemctl restart kubelet
```

如果重新加载和重新启动成功，则`kubeadm init`继续正常的工作流程。
####  2.1.2 使用时的工作流程kubeadm join 
当你运行时`kubeadm join`，kubeadm 使用 `Bootstrap Token` 凭证来执行 TLS 引导，它会获取下载 kubelet-configConfigMap 所需的凭证并将其写入`/var/lib/kubelet/config.yaml`. 动态环境文件的生成方式与`kubeadm init`.

接下来，kubeadm运行以下两条命令将新配置加载到 kubelet 中：

```bash
systemctl daemon-reload && systemctl restart kubelet
```
kubelet 加载新配置后，kubeadm 写入 KubeConfig 文件`/etc/kubernetes/bootstrap-kubelet.conf`，其中包含 `CA` 证书和 `Bootstrap Token`。kubelet 使用这些来执行 TLS Bootstrap 并获得一个唯一的凭证，该凭证存储在`/etc/kubernetes/kubelet.conf`.

当`/etc/kubernetes/kubelet.conf`文件被写入时，kubelet 已经完成了 `TLS Bootstrap`。Kubeadm在完成 TLS Bootstrap 后删除该文件`/etc/kubernetes/bootstrap-kubelet.conf`。

`/etc/kubernetes/kubelet.conf`内容示例：
```bash
在这里插入代码片[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generate at runtime, populating
the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably,
# the user should use the .NodeRegistration.KubeletExtraArgs object in the configuration files instead.
# KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```
此文件为 kubelet 的 kubeadm 管理的所有文件指定默认位置。

 - 用于 TLS Bootstrap 的 KubeConfig 文件是`/etc/kubernetes/bootstrap-kubelet.conf`，但仅在`/etc/kubernetes/kubelet.conf`不存在时使用。
 - 具有唯一 kubelet 身份的 KubeConfig 文件是`/etc/kubernetes/kubelet.conf`.
 - 包含 kubelet 的 Component Config 的文件是`/var/lib/kubelet/config.yaml`.
 - 包含的动态环境文件`KUBELET_KUBEADM_ARGS`来自`/var/lib/kubelet/kubeadm-flags.env`.
 - 可以包含用户指定的标志覆盖的文件`KUBELET_EXTRA_ARGS`来自 `/etc/default/kubelet`（对于 DEB）或`/etc/sysconfig/kubelet`（对于 RPM）。`KUBELET_EXTRA_ARGS` 在标志链中是最后一个，并且在设置冲突的情况下具有最高优先级。

### 2.2  systemd 配置 kubelet drop-in 文件
kubeadm附带有关 systemd 应如何运行 kubelet 的配置。请注意，kubeadm CLI 命令永远不会触及这个插入文件。

这个由kubeadm [DEB](https://github.com/kubernetes/release/blob/master/cmd/kubepkg/templates/latest/deb/kubeadm/10-kubeadm.conf)或 [RPM](https://github.com/kubernetes/release/blob/master/cmd/kubepkg/templates/latest/rpm/kubeadm/10-kubeadm.conf) 包安装的配置文件被写入 `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` 并被 systemd 使用。它增强了 [kubelet.service](https://github.com/kubernetes/release/blob/master/cmd/kubepkg/templates/latest/rpm/kubelet/kubelet.service) [RPM](https://github.com/kubernetes/release/blob/master/cmd/kubepkg/templates/latest/rpm/kubelet/kubelet.service)或 [kubelet.service](https://github.com/kubernetes/release/blob/master/cmd/kubepkg/templates/latest/deb/kubelet/lib/systemd/system/kubelet.service) [DEB](https://github.com/kubernetes/release/blob/master/cmd/kubepkg/templates/latest/deb/kubelet/lib/systemd/system/kubelet.service)的基础：

##  3. 配置注意
由于硬件、操作系统、网络或其他主机特定参数的差异，一些主机需要特定的 kubelet 配置：
1. 由 kubelet 配置标志指定的 DNS 解析文件的路径`--resolv-conf`可能因操作系统而异，或者取决于您是否使用 `systemd-resolved`. 如果此路径错误，则 kubelet 配置错误的 Node 上的 DNS 解析将失败。

2. Node API 对象`.metadata.name`默认设置为机器的主机名，除非您使用云提供商。`--hostname-override`如果您需要指定与机器主机名不同的节点名称，您可以使用该标志来覆盖默认行为。

3. 目前，`kubelet` 无法自动检测容器运行时使用的 cgroup 驱动，但 的值`--cgroup-driver`必须与容器运行时使用的 cgroup 驱动相匹配，以保证 kubelet 的健康。

4. `--container-runtime-endpoint=<path>`要指定容器运行时，您必须使用标志设置其端点 。

您可以通过在服务管理器中配置单个 kubelet 的配置来指定这些标志，例如 systemd。

参考：

 - [Configuring each kubelet in your cluster using kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/)

