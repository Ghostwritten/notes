#  kubeadm 命令



---
## 1. kubeadm 概述
Kubeadm 是一个工具，它提供了 `kubeadm init` 以及 `kubeadm join` 这两个命令作为快速创建 kubernetes 集群的最佳实践。

## 2. 安装kubeadm
官方参考：
[kubeadm安装](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

## 3. kubeadm任务

 1. kubeadm init 启动引导一个 Kubernetes 主节点
 2. kubeadm join 启动引导一个 Kubernetes 工作节点并且将其加入到集群
 3. kubeadm upgrade 更新 Kubernetes 集群到新版本
 4. kubeadm config 如果你使用 kubeadm v1.7.x 或者更低版本，你需要对你的集群做一些配置以便使用 kubeadmupgrade 命令
 5. kubeadm token 使用 kubeadm join 来管理令牌
 6. kubeadm reset 还原之前使用 kubeadm init 或者 kubeadm join 对节点所作改变
 7. kubeadm version 打印出 kubeadm 版本
 8. kubeadm alpha 预览一组可用的新功能以便从社区搜集反馈

## 4. kubeadm init 流程
"init" 命令执行以下阶段：

```bash
preflight                    Run pre-flight checks
kubelet-start                Write kubelet settings and (re)start the kubelet
certs                        Certificate generation
  /ca                          Generate the self-signed Kubernetes CA to provision identities for other Kubernetes components
  /apiserver                   Generate the certificate for serving the Kubernetes API
  /apiserver-kubelet-client    Generate the certificate for the API server to connect to kubelet
  /front-proxy-ca              Generate the self-signed CA to provision identities for front proxy
  /front-proxy-client          Generate the certificate for the front proxy client
  /etcd-ca                     Generate the self-signed CA to provision identities for etcd
  /etcd-server                 Generate the certificate for serving etcd
  /etcd-peer                   Generate the certificate for etcd nodes to communicate with each other
  /etcd-healthcheck-client     Generate the certificate for liveness probes to healthcheck etcd
  /apiserver-etcd-client       Generate the certificate the apiserver uses to access etcd
  /sa                          Generate a private key for signing service account tokens along with its public key
kubeconfig                   Generate all kubeconfig files necessary to establish the control plane and the admin kubeconfig file
  /admin                       Generate a kubeconfig file for the admin to use and for kubeadm itself
  /kubelet                     Generate a kubeconfig file for the kubelet to use *only* for cluster bootstrapping purposes
  /controller-manager          Generate a kubeconfig file for the controller manager to use
  /scheduler                   Generate a kubeconfig file for the scheduler to use
control-plane                Generate all static Pod manifest files necessary to establish the control plane
  /apiserver                   Generates the kube-apiserver static Pod manifest
  /controller-manager          Generates the kube-controller-manager static Pod manifest
  /scheduler                   Generates the kube-scheduler static Pod manifest
etcd                         Generate static Pod manifest file for local etcd
  /local                       Generate the static Pod manifest file for a local, single-node local etcd instance
upload-config                Upload the kubeadm and kubelet configuration to a ConfigMap
  /kubeadm                     Upload the kubeadm ClusterConfiguration to a ConfigMap
  /kubelet                     Upload the kubelet component config to a ConfigMap
upload-certs                 Upload certificates to kubeadm-certs
mark-control-plane           Mark a node as a control-plane
bootstrap-token              Generates bootstrap tokens used to join a node to a cluster
kubelet-finalize             Updates settings relevant to the kubelet after TLS bootstrap
  /experimental-cert-rotation  Enable kubelet client certificate rotation
addon                        Install required addons for passing Conformance tests
  /coredns                     Install the CoreDNS addon to a Kubernetes cluster
  /kube-proxy                  Install the kube-proxy addon to a Kubernetes cluster
```
描述：

 1. 在进行更改之前，kubeadm 运行一系列检查以验证系统状态。一些检查只会触发警告，有些检查会被视为错误并会退出kubeadm，直到问题得到解决或用户指定了 `--skip-preflight-checks。`
 2. kubeadm 将生成一个 token，以便其它 node 可以用来注册到 master 中。用户也可以选择自己提供一个 token。
 3. kubeadm 将生成一个自签名 CA 来为每个组件设置身份（包括node）。它也生成客户端证书以便各种组件可以使用。如果用户已经提供了自己的 CA 并将其放入 cert 目（通过--cert-dir 配置，默认路径为 `/etc/kubernetes/pki`），则跳过此步骤。
 4. 输出一个 kubeconfig 文件以便 kubelet 能够使用这个文件来连接到 API server，以及一个额外的kubeconfig 文件以作管理用途。
 5. kubeadm 将会为 API server、controller manager 和 scheduler 生成 Kubernetes 的静态 Pod manifest 文件，并将这些文件放入 `/etc/kubernetes/manifests` 中。Kubelet将会监控这个目录，以便在启动时创建 pod。这些都是 Kubernetes 的关键组件，一旦它们启动并正常运行后，kubeadm就能启动和管理其它额外的组件了。
 6. kubeadm 将会给 master 节点 “taint” 标签，以让控制平面组件只运行在这个节点上。它还建立了 RBAC授权系统，并创建一个特殊的 ConfigMap 用来引导与 kubelet 的互信连接。
 7. kubeadm 通过 API server 安装插件组件。目前这些组件有内部的 DNS server 和 kube-proxy DaemonSet。

## 5. kubeadm init 初始化参数

```bash
kubeadm init [flags]
```
参数说明：

```bash
--apiserver-advertise-address string
API Server将要广播的监听地址。如指定为 `0.0.0.0` 将使用缺省的网卡地址。

--apiserver-bind-port int32     缺省值: 6443
API Server绑定的端口

--apiserver-cert-extra-sans stringSlice
可选的额外提供的证书主题别名（SANs）用于指定API Server的服务器证书。可以是IP地址也可以是DNS名称。

--cert-dir string     缺省值: "/etc/kubernetes/pki"
证书的存储路径。

--config string
kubeadm配置文件的路径。警告：配置文件的功能是实验性的。

--cri-socket string     缺省值: "/var/run/dockershim.sock"
指明要连接的CRI socket文件

--dry-run
不会应用任何改变；只会输出将要执行的操作。


--feature-gates string
键值对的集合，用来控制各种功能的开关。可选项有:
Auditing=true|false (当前为ALPHA状态 - 缺省值=false)
CoreDNS=true|false (缺省值=true)
DynamicKubeletConfig=true|false (当前为BETA状态 - 缺省值=false)

-h, --help
获取init命令的帮助信息

--ignore-preflight-errors stringSlice
忽视检查项错误列表，列表中的每一个检查项如发生错误将被展示输出为警告，而非错误。 例如: 'IsPrivilegedUser,Swap'. 如填写为 'all' 则将忽视所有的检查项错误。

--kubernetes-version string     缺省值: "stable-1"
为control plane选择一个特定的Kubernetes版本。

--node-name string
指定节点的名称。

--pod-network-cidr string
指明pod网络可以使用的IP地址段。 如果设置了这个参数，control plane将会为每一个节点自动分配CIDRs。

--service-cidr string     缺省值: "10.96.0.0/12"
为service的虚拟IP地址另外指定IP地址段

--service-dns-domain string     缺省值: "cluster.local"
为services另外指定域名, 例如： "myorg.internal".

--skip-token-print
不打印出由 `kubeadm init` 命令生成的默认令牌。

--token string
这个令牌用于建立主从节点间的双向受信链接。格式为 [a-z0-9]{6}\.[a-z0-9]{16} - 示例： abcdef.0123456789abcdef

--token-ttl duration     缺省值: 24h0m0s
令牌被自动删除前的可用时长 (示例： 1s, 2m, 3h). 如果设置为 '0', 令牌将永不过期。
```

### 5.1 kubeadm init phase分段执行
--skip-phases 可用于跳过某些阶段
```bash
sudo kubeadm init phase control-plane all --config=configfile.yaml
sudo kubeadm init phase etcd local --config=configfile.yaml
sudo kubeadm init --skip-phases=control-plane,etcd --config=configfile.yaml
```

### 5.2 Kubeadm自定义组件配置参数
kubeadm ClusterConfiguration对象公开了extraArgs可以覆盖传递给控制平面组件（如APIServer，ControllerManager和Scheduler）的默认标志的字段。使用以下字段定义组件：

 - apiServer
 - controllerManager
 - scheduler

该extraArgs字段由key: value对组成。覆盖控制平面组件的标志：
#### 5.2.1 APIServer配置

```bash
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
apiServer:
  extraArgs:
    advertise-address: 192.168.0.103
    anonymous-auth: "false"
    enable-admission-plugins: AlwaysPullImages,DefaultStorageClass
    audit-log-path: /home/johndoe/audit.log
```
#### 5.2.2 ControllerManager配置

```bash
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
controllerManager:
  extraArgs:
    cluster-signing-key-file: /home/johndoe/keys/ca.key
    bind-address: 0.0.0.0
    deployment-controller-sync-period: "50"
```
#### 5.2.3 Scheduler配置

```bash
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
scheduler:
  extraArgs:
    address: 0.0.0.0
    config: /home/johndoe/schedconfig.yaml
    kubeconfig: /home/johndoe/kubeconfig.yaml
```
覆盖默认配置
```bash
kubeadm init --config=component.yaml
```
### 5.3 kubeadm自定义镜像
默认情况下，kubeadm从中提取图像`k8s.gcr.io`。如果请求的Kubernetes版本是CI标签（例如ci/latest） `gcr.io/kubernetes-ci-images`

允许的自定义为：

 - 提供替代`imageRepository`的方法 k8s.gcr.io。
 - 设置`useHyperKubeImage`为true使用HyperKube图像。
 - 为了提供一个具体的`imageRepository`和imageTag为ETCD或DNS插件。

请注意，配置字段kubernetesVersion或命令行标志 `--kubernetes-version`会影响图像的版本

### 5.4 kubeadm将证书上载到集群
此机密将在2小时后自动过期。证书使用32字节密钥加密，可以使用进行指定--certificate-key。相同的密钥可以被用来下载时附加的控制平面节点通过使接合，证书 `--control-plane`和-`-certificate-key`通过`kubeadm join`上传。
在到期后重新上传证书：

```bash
kubeadm init phase upload-certs --upload-certs --certificate-key=SOME_VALUE --config=SOME_YAML_FILE
```
如果该标志--certificate-key未传递到kubeadm init， kubeadm init phase upload-certs则会自动生成一个新密钥。

以下命令可用于按需生成新密钥：

```bash
kubeadm alpha certs certificate-key
```
## 6. kubeadmin join
### 6.1 流程详解
[请参考](https://blog.csdn.net/shida_csdn/article/details/83269238)
### 6.2 参数

```bash
--apiserver-advertise-address string    #如果该节点应托管一个新的控制平面实例，则API Server的IP地址将通告其正在侦听的地址。如果未设置，将使用默认网络接口。
--apiserver-bind-port int32     Default: 6443  #如果该节点应承载一个新的控制平面实例，则该API服务器要绑定到的端口。
--certificate-key string ##使用此密钥可以解密由init上传的证书secret
--config string   #kubeadm配置文件的路径。
--control-plane  #在此节点上创建一个新的控制平面实例
--cri-socket string   #要连接的CRI套接字的路径。如果为空，则kubeadm将尝试自动检测此值；仅当您安装了多个CRI或具有非标准CRI插槽时，才使用此选项。
--discovery-file string  #从中加载集群信息的文件或URL
--discovery-token string  #发现token,从中加载集群信息的文件或URL
--discovery-token-ca-cert-hash stringSlice #对于基于令牌的发现，请验证根CA公共密钥是否与此哈希匹配（格式：“ <类型>：<值>”）。
--discovery-token-unsafe-skip-ca-verification  #对于基于令牌的发现，允许加入时不使用--discovery-token-ca-cert-hash固定
-k, --experimental-kustomize string  #kustomize静态pod清单的补丁的存储路径。
--ignore-preflight-errors stringSlice  #检查清单，其错误将显示为警告。例如：“ IsPrivilegedUser，Swap”。值“ all”忽略所有检查的错误。
--node-name string  #指定节点名称。
--skip-phases stringSlice  #要跳过的阶段列表
--tls-bootstrap-token string  #指定用于在加入节点时临时通过Kubernetes控制平面进行身份验证的令牌。
--token string  #如果未提供这些值，则将它们用于发现令牌和tls-bootstrap令牌。
```

```bash
$ kubeadm join --skip-phases=preflight --config=config.yaml
$ kubeadm join 192.168.211.40:6443 --token 5zw7z4.qjzipdh89aguvzn5     --discovery-token-ca-cert-hash sha256:167d0176ccd1c90b7373917940620fb7a48b245913eb25a05726345902f6213c 
```


##  7. kubeadm config

```bash
$ kubeadm config upload from-file
$ kubeadm config view  #查看集群中 kubeadm 配置所在的 ConfigMap
$ kubeadm config print init-defaults #打印初始化配置
$ kubeadm config print join-defaults #打印join配置


$ kubeadm config images pull #根据配置文件拉取镜像


$ kubeadm config images list #显示需要拉取的镜像
k8s.gcr.io/kube-apiserver:v1.20.9
k8s.gcr.io/kube-controller-manager:v1.20.9

k8s.gcr.io/kube-scheduler:v1.20.9
k8s.gcr.io/kube-proxy:v1.20.9
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/coredns:1.7.0

$ kubeadm config print init-defaults > kubeadm.conf

```

## 8. kubeadm token
kubeadm init或输出的命令中返回的kubeadm join..

```bash
kubeadm token create --print-join-command
```


参考：

 - [github kubeadm](https://github.com/kubernetes/kubeadm)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)
