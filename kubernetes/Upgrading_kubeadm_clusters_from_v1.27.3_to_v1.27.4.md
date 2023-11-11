![](https://img-blog.csdnimg.cn/f7df35a91711478ebf7ff319c81cb574.png)




刚刚 [kubernetes release](https://github.com/kubernetes/kubernetes/releases) 升级到 `v1.27.4` 了。相关 v1.27.4 升级内容请参考在[这里](https://github.com/kubernetes/kubernetes/releases)，迫不及待想升级一下。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d3edd02160744d10ad750e1b72261ded.png)


##  1. Before you begin
- 快照
- 业务应用备份数据


准备要升级的`kubernetes` 集群信息：
```bash
$ kubectl get node
NAME            STATUS     ROLES           AGE   VERSION
kube-master01   Ready   control-plane   42h   v1.27.3
kube-node01     Ready   <none>          42h   v1.27.3
kube-prom01     Ready   <none>          42h   v1.27.3

```

## 2. Notes
- 如果您正在为任何kubelet执行次要版本升级，则必须首先清空您正在升级的节点（或多个节点）。在控制平面节点，它们可能正在运行CoreDNS Pod或其他关键工作负载。有关详细信息，请参见 [Draining nodes](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)。
- 升级后所有容器都将重新启动，因为容器规范散列值已更改。
- 要验证kubelet服务在kubelet升级后是否成功重启，可以执行`systemctl status kubelet`，或者使用`journalctl -xeu kubelet`查看服务日志。
- 建议不要使用`kubeadm upgrade`的`--config`标志和[kubeadm configuration API](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/)类型来重新配置集群，这可能会导致意外结果。请按照[重新配置kubeadm集群](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-reconfigure/)中的步骤操作 

## 3. Master
### 3.1 Login into the first node and upgrade the kubeadm tool only

```bash
$dnf list kubeadm kubelet kubectl --showduplicates | sort -r |grep 1.27.4
kubelet.x86_64                       1.27.4-0                        kubernetes 
kubectl.x86_64                       1.27.4-0                        kubernetes 
kubeadm.x86_64                       1.27.4-0                        kubernetes 

$ dnf check-update kubeadm kubectl kubelet
Last metadata expiration check: 2:04:02 ago on Thu 20 Jul 2023 09:34:52 AM CST.

kubeadm.x86_64                                                                                  1.27.4-0                                                                                  kubernetes
kubectl.x86_64                                                                                  1.27.4-0                                                                                  kubernetes
kubelet.x86_64                                                                                  1.27.4-0                                                                                  kubernetes

$ dnf update -y kubeadm

```
输出：

```bash
[root@kube-prom01 ~]# dnf update -y kubeadm-1.27.4-0
Last metadata expiration check: 1:16:03 ago on Thu 20 Jul 2023 11:14:46 AM CST.
Dependencies resolved.
====================================================================================================================================================================================================
 Package                                       Architecture                                 Version                                          Repository                                        Size
====================================================================================================================================================================================================
Upgrading:
 kubeadm                                       x86_64                                       1.27.4-0                                         kubernetes                                        11 M

Transaction Summary
====================================================================================================================================================================================================
Upgrade  1 Package

Total download size: 11 M
Downloading Packages:
e9bba51c897d8e465298724f44da6e457097f87aaac71b18fd6539b9e3503995-kubeadm-1.27.4-0.x86_64.rpm                                                                        9.2 MB/s |  11 MB     00:01    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                               9.2 MB/s |  11 MB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                            1/1 
  Running scriptlet: kubeadm-1.27.4-0.x86_64                                                                                                                                                    1/1 
  Upgrading        : kubeadm-1.27.4-0.x86_64                                                                                                                                                    1/2 
  Cleanup          : kubeadm-1.27.3-0.x86_64                                                                                                                                                    2/2 
  Running scriptlet: kubeadm-1.27.3-0.x86_64                                                                                                                                                    2/2 
  Verifying        : kubeadm-1.27.4-0.x86_64                                                                                                                                                    1/2 
  Verifying        : kubeadm-1.27.3-0.x86_64                                                                                                                                                    2/2 

Upgraded:
  kubeadm-1.27.4-0.x86_64                                                                                                                                                                           

Complete!

$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4", GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean", BuildDate:"2023-07-19T12:19:40Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}

```


### 3.2 Verify the upgrade plan
此命令检查您的群集是否可以升级，并获取您可以升级到的版本。它还显示了一个包含组件配置版本状态的表。
```bash
[root@kube-master01 ~]# kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.27.3
[upgrade/versions] kubeadm version: v1.27.4
[upgrade/versions] Target version: v1.27.4
[upgrade/versions] Latest version in the v1.27 series: v1.27.4

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       TARGET
kubelet     3 x v1.27.3   v1.27.4

Upgrade to the latest version in the v1.27 series:

COMPONENT                 CURRENT   TARGET
kube-apiserver            v1.27.3   v1.27.4
kube-controller-manager   v1.27.3   v1.27.4
kube-scheduler            v1.27.3   v1.27.4
kube-proxy                v1.27.3   v1.27.4
CoreDNS                   v1.10.1   v1.10.1
etcd                      3.5.7-0   3.5.7-0

You can now apply the upgrade by executing the following command:

	kubeadm upgrade apply v1.27.4

_____________________________________________________________________


The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________

```


### 3.3 Drain the control plane node
```bash

$ kubectl drain kube-master01 --ignore-daemonsets --delete-emptydir-data
node/kube-master01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-q87zt
evicting pod kube-system/coredns-5d78c9869d-lz7h6
evicting pod kube-system/coredns-5d78c9869d-lwvp5
pod/coredns-5d78c9869d-lwvp5 evicted
pod/coredns-5d78c9869d-lz7h6 evicted
node/kube-master01 drained
```

### 3.4 kubeadm upgrade

> 注意：`kubeadm upgrade`也会自动更新它在此节点上管理的证书。要选择退出证书更新，可以使用标志--certificate-renewal=false。有关详细信息，请参阅[证书管理指南](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)。

```bash
[root@kube-master01 ~]# kubeadm upgrade  apply v1.27.4
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.27.4"
[upgrade/versions] Cluster version: v1.27.3
[upgrade/versions] kubeadm version: v1.27.4
[upgrade] Are you sure you want to proceed? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.27.4" (timeout: 5m0s)...
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Current and new manifests of etcd are equal, skipping upgrade
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests4157307796"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-07-20-12-04-33/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-apiserver
[upgrade/staticpods] Component "kube-apiserver" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Renewing controller-manager.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-controller-manager.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-07-20-12-04-33/kube-controller-manager.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-controller-manager
[upgrade/staticpods] Component "kube-controller-manager" upgraded successfully!
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Renewing scheduler.conf certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-scheduler.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2023-07-20-12-04-33/kube-scheduler.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
[apiclient] Found 1 Pods for label selector component=kube-scheduler
[upgrade/staticpods] Component "kube-scheduler" upgraded successfully!
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config1926833644/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.27.4". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.

```
### 3.5 Uncordon the control plane node

```bash
$ kubectl uncordon kube-master01
node/kube-master01 uncordoned
```

###  3.6 Upgrade kubelet and kubectl

```bash
#centos、rocky、rhel：
dnf update -y kubelet-1.27.4-0 kubectl-1.27.4-0 --disableexcludes=kubernetes

#ubuntu：
$ apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.27.4-0 kubectl=1.27.4-0 && \
apt-mark hold kubelet kubectl
```

输出：

```bash
[root@kube-master01 ~]# dnf update -y kubelet-1.27.4-0 kubectl-1.27.4-0 --disableexcludes=kubernetes
Last metadata expiration check: 2:44:01 ago on Thu 20 Jul 2023 09:34:52 AM CST.
Dependencies resolved.
====================================================================================================================================================================================================
 Package                                       Architecture                                 Version                                          Repository                                        Size
====================================================================================================================================================================================================
Upgrading:
 kubectl                                       x86_64                                       1.27.4-0                                         kubernetes                                        11 M
 kubelet                                       x86_64                                       1.27.4-0                                         kubernetes                                        20 M

Transaction Summary
====================================================================================================================================================================================================
Upgrade  2 Packages

Total download size: 31 M
Downloading Packages:
(1/2): 28f442261f1306377aa2704f9f87117d27850ca00f5c26130080a57ccdb38c9d-kubectl-1.27.4-0.x86_64.rpm                                                                 1.3 MB/s |  11 MB     00:08    
(2/2): 49e46174a716325c333a575df9c990b0e237616e7c78537580d7e14204eca1d0-kubelet-1.27.4-0.x86_64.rpm                                                                 1.9 MB/s |  20 MB     00:10    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                               3.0 MB/s |  31 MB     00:10     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                            1/1 
  Running scriptlet: kubelet-1.27.4-0.x86_64                                                                                                                                                    1/1 
  Upgrading        : kubelet-1.27.4-0.x86_64                                                                                                                                                    1/4 
  Upgrading        : kubectl-1.27.4-0.x86_64                                                                                                                                                    2/4 
  Cleanup          : kubectl-1.27.3-0.x86_64                                                                                                                                                    3/4 
  Cleanup          : kubelet-1.27.3-0.x86_64                                                                                                                                                    4/4 
  Running scriptlet: kubelet-1.27.3-0.x86_64                                                                                                                                                    4/4 
  Verifying        : kubectl-1.27.4-0.x86_64                                                                                                                                                    1/4 
  Verifying        : kubectl-1.27.3-0.x86_64                                                                                                                                                    2/4 
  Verifying        : kubelet-1.27.4-0.x86_64                                                                                                                                                    3/4 
  Verifying        : kubelet-1.27.3-0.x86_64                                                                                                                                                    4/4 

Upgraded:
  kubectl-1.27.4-0.x86_64                                                                          kubelet-1.27.4-0.x86_64                                                                         

Complete!

```
重启 kubelet

```bash
sudo systemctl daemon-reload && sudo systemctl restart kubelet && systemctl status kubelet
```






> 注意：如果kubeadm升级计划显示任何需要手动升级的组件配置，用户必须通过`--config`命令行标志提供一个包含替换配置的配置文件，以便`kubeadm upgrade apply`。如果不这样做，将导致`kubeadm upgrade apply`退出并返回错误，并且不执行升级。

>  手动升级您的CNI提供程序插件：
您的容器网络接口（CNI）提供商可能有自己的升级说明。检查[插件](https://kubernetes.io/docs/concepts/cluster-administration/addons/)页面以找到您的CNI提供商，并查看是否需要其他升级步骤。
如果CNI提供程序作为DaemonSet运行，则在其他控制平面节点上不需要此步骤。


### 3.7 Apply the upgrade plan to the other master nodes
假如有三个 `control-plane` node

```bash
$ kubectl drain kube-master02 --ignore-daemonsets --delete-emptydir-data
$ ssh root@x.x.x.x
$ dnf update -y kubeadm-1.27.4-0  --disableexcludes=kubernetes
$ kubeadm upgrade node experimental-control-plane
$ kubectl uncordon kube-master02

#centos、rocky、rhel：
dnf update -y kubelet-1.27.4 kubectl-1.27.4 --disableexcludes=kubernetes

#ubuntu：
$ apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.27.4-0 kubectl=1.27.4-0 && \
apt-mark hold kubelet kubectl

#重启 kubelet
sudo systemctl daemon-reload && sudo systemctl restart kubelet && systemctl status kubelet
```


##  4. Worker
### 4.1 Upgrade kubeadm on first worker node

```bash
$ ssh worker@x.x.x.x

#ubuntu：
$ apt-mark unhold kubeadm && apt-get update && apt-get install -y kubeadm=1.27.4-00 && apt-mark hold kubeadm

#centos、rocky、rhel：
dnf list kubeadm kubelet kubectl --showduplicates | sort -r |grep 1.27.4
dnf update -y kubeadm-1.27.4-0
```


###  4.2 Login to a master node and drain first worker node

```bash
$ ssh root@x.x.x.x
$ kubectl drain <node-to-drain> --ignore-daemonsets
$ kubectl drain kube-prom01 --ignore-daemonsets
```
输出：

```bash
[root@kube-master01 ~]# kubectl drain kube-prom01 --ignore-daemonsets
node/kube-prom01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/kube-proxy-rb7k2
node/kube-prom01 drained
```

###  4.3 Upgrade kubelet config on worker node

```bash
$ ssh root@x.x.x.x
$ kubeadm upgrade node
或者
$ kubeadm upgrade node config --kubelet-version v1.27.4
```
输出：

```bash
[root@kube-prom01 ~]# kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config3537245876/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.

```
### 4.4 Upgrade kubelet and kubectl

```bash
```bash

#centos、rocky、rhel：
dnf update -y kubelet-1.27.4 kubectl-1.27.4 --disableexcludes=kubernetes

#ubuntu：
$ apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.27.4-0 kubectl=1.27.4-0 && \
apt-mark hold kubelet kubectl

sudo systemctl daemon-reload && sudo systemctl restart kubelet && systemctl status kubelet
```

### 4.5 Uncordon the worker node
通过将节点标记为可调度，使其重新联机：
(master操作)
```bash
kubectl uncordon <node-to-uncordon>
kubectl uncordon kube-prom01
```
> 注意：其他 worker node 升级步骤同上

## 4.6  Check cluster

```bash
[root@kube-master01 ~]# kubectl get nodes
NAME            STATUS   ROLES           AGE   VERSION
kube-master01   Ready    control-plane   43h   v1.27.4
kube-node01     Ready    <none>          43h   v1.27.4
kube-prom01     Ready    <none>          43h   v1.27.4
[root@kube-master01 ~]# kubectl get pod -A
NAMESPACE     NAME                                    READY   STATUS    RESTARTS       AGE
kube-system   coredns-5d78c9869d-d6tjw                1/1     Running   0              46m
kube-system   coredns-5d78c9869d-l6qsw                1/1     Running   0              46m
kube-system   etcd-kube-master01                      1/1     Running   1 (138m ago)   43h
kube-system   kube-apiserver-kube-master01            1/1     Running   0              43m
kube-system   kube-controller-manager-kube-master01   1/1     Running   0              43m
kube-system   kube-proxy-862db                        1/1     Running   0              42m
kube-system   kube-proxy-gnpjp                        1/1     Running   0              7m50s
kube-system   kube-proxy-k7vpf                        1/1     Running   0              48s
kube-system   kube-scheduler-kube-master01            1/1     Running   0              42m
[root@kube-master01 ~]# kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4", GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean", BuildDate:"2023-07-19T12:20:54Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1
Server Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4", GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean", BuildDate:"2023-07-19T12:14:49Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}
[root@kube-master01 ~]# kubelet --version
Kubernetes v1.27.4
[root@kube-master01 ~]# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.4", GitCommit:"fa3d7990104d7c1f16943a67f11b154b71f6a132", GitTreeState:"clean", BuildDate:"2023-07-19T12:19:40Z", GoVersion:"go1.20.6", Compiler:"gc", Platform:"linux/amd64"}
```


参考：
- [Kubernetes Upgrade: The Definitive Guide to Do-It-Yourself](https://platform9.com/blog/kubernetes-upgrade-the-definitive-guide-to-do-it-yourself/)
- [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
