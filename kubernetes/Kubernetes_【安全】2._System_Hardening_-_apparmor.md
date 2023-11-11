

现在，想想在生产中实施 AppArmor的挑战。

首先，您必须为每个容器构建强大的配置文件，以在不阻塞日常任务的情况下防止攻击。

然后，您将必须跨集群中的所有节点管理多个配置文件。

我们将介绍[kube-apparmor-manager](https://github.com/sysdiglabs/kube-apparmor-manager)如何帮助管理部分，以及[Sysdig Secure](https://sysdig.com/blog/sysdig-secure-2-4/) 中的[图像分析功能](https://sysdig.com/blog/sysdig-secure-2-4/)如何帮助构建这些配置文件。

##  Kube-apparmor-manager
有一些工具，比如[apparmor-loader](https://github.com/kubernetes/kubernetes/tree/master/test/images/apparmor-loader)，可以帮助管理 Kubernetes 集群中的 `AppArmor` 配置文件。Apparmor-loader作为特权守护进程运行，轮询包含 AppArmor 配置文件的配置映射，最后将配置文件解析为强制模式或投诉模式。然而，这引入了远非理想的特权工作负载。这就是为什么我们可以想出另一种方法。

Kube-apparmor-manager方法不同：

 - 它使用自定义资源(apparmorprofiles.crd.security.sysdig.com)将配置文件表示为Kubernetes 对象。
 - 一个kubectl插件转换`AppArmorProfile`对象，存储在`ETCD`，到实际`AppArmor`配置文件，并同步他们的节点之间。

让我们详细看看它们是如何工作的。

###  AppArmorProfile 自定义资源定义
[AppArmorProfile CRD](https://github.com/sysdiglabs/kube-apparmor-manager/blob/master/crd/crd.security.sysdig.com_apparmorprofiles.yaml)定义了一个架构来将 AppArmor 配置文件表示为 Kubernetes 对象。

这就是我们的示例 AppArmor 配置文件在这种格式下的样子：

```bash
apiVersion: crd.security.sysdig.com/v1alpha1
kind: AppArmorProfile
metadata:
  name: k8s-apparmor-example-deny-write
spec:
  # Add fields here
  enforced: true
  rules: |
    # read only file paths
    file,
    deny /** w,
```
该`enforced`字段指示配置文件处于强制模式还是投诉模式。该字段rules包含带有白名单或黑名单规则列表的 AppArmor 配置文件正文。

请注意，这是一个集群级别的对象。


###   Apparmor-manager 插件
在 Kubernetes 集群中安装 CRD 后，您可以开始使用 kubectl 与 AppArmorProfile 对象进行交互。但是，您仍然需要将 AppArmorProfile 对象中的内容转换为实际的 AppArmor 配置文件，并将它们分发到所有节点。

这就是apparmor-managerkubectl 插件所做的。

您可以使用[krew](https://github.com/kubernetes-sigs/krew)安装它：

```bash
$ wget https://github.com/kubernetes-sigs/krew/releases/download/v0.4.2/krew-linux_amd64.tar.gz
$ tar xzvf krew-linux_amd64.tar.gz
$ mv krew-linux_amd64 /usr/local/bin/krew
$ krew install apparmor-manager
$ krew install krew
WARNING: To be able to run kubectl plugins, you need to add
the following to your ~/.bash_profile or ~/.bashrc:

    export PATH="${PATH}:${HOME}/.krew/bin"

and restart your shell.

Updated the local copy of plugin index.
Installing plugin: krew
Installed plugin: krew
\
 | Use this plugin:
 | 	kubectl krew
 | Documentation:
 | 	https://krew.sigs.k8s.io/
 | Caveats:
 | \
 |  | krew is now installed! To start using kubectl plugins, you need to add
 |  | krew's installation directory to your PATH:
 |  | 
 |  |   * macOS/Linux:
 |  |     - Add the following to your ~/.bashrc or ~/.zshrc:
 |  |         export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
 |  |     - Restart your shell.
 |  | 
 |  |   * Windows: Add %USERPROFILE%\.krew\bin to your PATH environment variable
 |  | 
 |  | To list krew commands and to get help, run:
 |  |   $ kubectl krew
 |  | For a full list of available plugins, run:
 |  |   $ kubectl krew search
 |  | 
 |  | You can find documentation at
 |  |   https://krew.sigs.k8s.io/docs/user-guide/quickstart/.
 | /
/


$ echo 'export PATH="${PATH}:${HOME}/.krew/bin"' >> /root/.bashrc 

$ kubectl krew install apparmor-manager

```

当apparmor-manager通过 SSH 与工作节点通信时，您需要设置一些环境变量：

 - `SSH_USERNAME`: SSH 用户名以访问工作节点。默认为`admin`.
 - `SSH_PERM_FILE`：用于访问工作节点的 SSH 私钥。默认为`$HOME/.ssh/id_rsa`.
 - `SSH_PASSPHRASE`: SSH 密码（仅当私钥受密码保护时才适用）。

如果节点上没有安装 AppArmor，apparmor-manager 可以帮助您使用以下命令在工作节点上启用 AppArmor：

```bash
$ kubectl apparmor-manager init
```
该init命令还将为您安装 CRD。

配置完所有内容后，您可以使用以下命令检查节点上 AppArmor 的状态：

```bash
$ kubectl apparmor-manager status
+-------------------------------+---------------+----------------+--------+------------------+
|           NODE NAME           |  INTERNAL IP  |  EXTERNAL IP   |  ROLE  | APPARMOR ENABLED |
+-------------------------------+---------------+----------------+--------+------------------+
| ip-172-20-45-132.ec2.internal | 172.20.45.132 | 54.91.xxx.xx   | master | false            |
| ip-172-20-54-2.ec2.internal   | 172.20.54.2   | 54.82.xx.xx    | node   | true             |
| ip-172-20-58-7.ec2.internal   | 172.20.58.7   | 18.212.xxx.xxx | node   | true             |
+-------------------------------+---------------+----------------+--------+------------------+
```
您还可以使用 kubectl 创建您的第一个 AppArmorProfile 对象：

```bash
$ kubectl apply -f deny-write.yaml
apparmorprofile.crd.security.sysdig.com/k8s-apparmor-example-deny-write created
$ kubectl get aap
NAME                              AGE
k8s-apparmor-example-deny-write   5s
```
创建后，您需要将 AppArmorProfiles 同步到工作节点：

```bash
$ kubectl apparmor-manager enforced
+-------------------------------+--------+---------------------------------------------------------------+
|           NODE NAME           |  ROLE  |                       ENFORCED PROFILES                       |
+-------------------------------+--------+---------------------------------------------------------------+
| ip-172-20-48-62.ec2.internal  | node   | /usr/sbin/ntpd,docker-default,k8s-apparmor-example-deny-write |
| ip-172-20-77-231.ec2.internal | node   | /usr/sbin/ntpd,docker-default,k8s-apparmor-example-deny-write |
| ip-172-20-80-19.ec2.internal  | master |                                                               |
| ip-172-20-97-60.ec2.internal  | node   | /usr/sbin/ntpd,docker-default,k8s-apparmor-example-deny-write |
+-------------------------------+--------+---------------------------------------------------------------+
```
这`k8s-apparmor-example-deny-write`是我们刚刚创建并同步的一个，而另外两个默认安装在 AppArmor 中。

最后一步是配置 Pod 以使用此配置文件，使用`annotations`我们之前看到的。

接下来，我们来谈谈如何使用 Sysdig Secure 构建健壮的 AppArmor 配置文件。

##  Sysdig Secure 构建强大的 Apparmor Profile

借助图像分析，Sysdig Secure 将分析容器 24 小时，了解预期的进程、文件系统活动、网络行为和系统调用。有了这些知识，您可以生成学习的映像配置文件，并使用它来创建运行时策略，以保护容器免受生产中的异常行为的影响。

