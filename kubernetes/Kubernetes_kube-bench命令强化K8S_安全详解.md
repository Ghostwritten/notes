



----
## 1. 简介
Kube-Bench是一款针对Kubernete的安全检测工具，从本质上来说，Kube-Bench是一个基于Go开发的应用程序，它可以帮助研究人员对部署的Kubernete进行安全检测，安全检测原则遵循CIS Kubernetes Benchmark。

测试规则需要通过YAML文件进行配置，因此我们可以轻松更新该工具的测试规则。

## 2. 注意

 - Kube-Bench尽可能地实现了`CIS Kubernetes Benchmark`，如果kube-bench没有正确执行安全基准测试，请点击【https://www.cisecurity.org/】提交问题。
 - Kubernete版本和CIS基准测试版本之间没有一对一的映射。请参阅CIS Kubernetes基准测试支持，以查看基准测试的不同版本包含哪些Kubernetes版本。
 - Kube-Bench无法检查受管集群的主节点，例如GKE、EKS和AKS，因为Kube-Bench不能访问这些节点。不过，Kube-Bench在这些环境中仍然可以检查worker节点配置。


## 3. CIS Kubernetes Benchmark支持
Kube-Bench支持的Kubernete测试规则定义在CIS Kubernetes Benchmark之中：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210615141333238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
默认配置下，Kube-Bench将根据目标设备上运行的Kubernete版本来确定要运行的测试集，但请注意，Kube-Bench不会自动检测OpenShift和GKE。

## 4. 工具下载

```bash
git clone https://github.com/aquasecurity/kube-bench.git
```

 - 可以选择在容器中运行Kube-Bench（跟主机共享PID命名空间）；
 - 在主机中运行安装了Kube-Bench的容器，然后直接在主机中运行Kube-Bench；
 - 访问项目Releases页面下载并安装最新版本的源码，别忘了下载配置文件以及cfg目录下的测试文件；
 - 从源码编译；


## 5.安装
### 5.1 容器中安装

```bash
docker run --rm -v `pwd`:/host aquasec/kube-bench:latest install

./kube-bench [master|node]
```
### 5.2  源码安装
首先在设备上安装并配置好Go环境，然后运行下列命令即可：

```c
go get github.com/aquasecurity/kube-bench

cd $GOPATH/src/github.com/aquasecurity/kube-bench

go build -o kube-bench .

# See all supported options

./kube-bench --help

# Run all checks

./kube-bench
```

## 6. 运行
### 6.1 容器中运行
你可以直接通过主机PID命名空间来在一个容器中安装并运行Kube-Bench，并加载配置文件所在目录，比如说“/etc”或“/var”：

```bash
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest [master|node] --version 1.13
```
如果你不想使用“`/opt/kube-bench/cfg/`”下的默认配置，你还可以通过加载自定义的配置文件来使用它们：

```bash
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t -v path/to/my-config.yaml:/opt/kube-bench/cfg/config.yam -v $(which kubectl):/usr/local/mount-from-host/bin/kubectl -v ~/.kube:/.kube -e KUBECONFIG=/.kube/config aquasec/kube-bench:latest [master|node]
```

### 6.2 二进制运行
如果你想直接通过命令行工具运行Kube-Bench，你还需要root/sudo权限来访问所有的配置文件。

Kube-Bench将会根据检测到的节点类型以及Kubernete运行的集群版本来自动选择使用哪一个“controls”。这种行为可以通过定义master或node子命令以及“`—version`”命令行参数来进行修改。

Kube-Bench版本还可以通过“KUBE_BENCH_VERSION”环境变量来设置，“—version”参数将优先于“KUBE_BENCH_VERSION”环境变量生效。

比如说，我们可以使用Kube-Bench对一个master执行版本自动检测：

```bash
kube-bench master
```

或者，使用Kube-Bench针对Kubernete v1.13执行worker节点测试：

```bash
kube-bench node --version 1.13
```

kube-bench将会根据对应的CIS Benchmark版本来映射“—version”。不如说，如果你指定的是“—version 1.13”，此时映射的CIS Benchmark版本为“cis-1.14”。

或者说，你还可以指定“—benchmark”来运行指定的CIS Benchmark版本：

```bash
kube-bench node --benchmark cis-1.4
```

如果你想要指定CIS Benchmark的target，比如说master、node、etcd等，你可以运行“run —targets”子命令：

```bash
kube-bench --benchmark cis-1.4 run --targets master,node
```

或

```bash
kube-bench --benchmark cis-1.5 run --targets master,node,etcd,policies
```
下表中显示的是不同CIS Benchmark版本对应的有效目标：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021061514203112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 7. 实战
### 7.1 检查master

```c
root@master:~/cks/metadata# kubectl get nodes
NAME     STATUS   ROLES                  AGE   VERSION
master   Ready    control-plane,master   93d   v1.20.1
node1    Ready    <none>                 93d   v1.20.1
node2    Ready    <none>                 93d   v1.20.1
root@master:~/cks/metadata# docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest master --version 1.20
Unable to find image 'aquasec/kube-bench:latest' locally
latest: Pulling from aquasec/kube-bench
801bfaa63ef2: Pull complete 
b6310d2303a3: Pulling fs layer 
4fb7d152903c: Download complete 
c8ea24332a01: Downloading 
c02463f95f5e: Download complete 
763038f5a72b: Download complete 
64bce6c8fa58: Download complete 
latest: Pulling from aquasec/kube-bench
801bfaa63ef2: Pull complete 
b6310d2303a3: Pull complete 
4fb7d152903c: Pull complete 
c8ea24332a01: Pull complete 
c02463f95f5e: Pull complete 
763038f5a72b: Pull complete 
64bce6c8fa58: Pull complete 
Digest: sha256:bf9394a3ab84ebe9065a3f0256196f4257e73dad620ccab2f844fbc2f5e7969c
Status: Downloaded newer image for aquasec/kube-bench:latest
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 Master Node Configuration Files
[PASS] 1.1.1 Ensure that the API server pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.2 Ensure that the API server pod specification file ownership is set to root:root (Automated)
[PASS] 1.1.3 Ensure that the controller manager pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.4 Ensure that the controller manager pod specification file ownership is set to root:root (Automated)
[PASS] 1.1.5 Ensure that the scheduler pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.6 Ensure that the scheduler pod specification file ownership is set to root:root (Automated)
[PASS] 1.1.7 Ensure that the etcd pod specification file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.8 Ensure that the etcd pod specification file ownership is set to root:root (Automated)
[WARN] 1.1.9 Ensure that the Container Network Interface file permissions are set to 644 or more restrictive (Manual)
[WARN] 1.1.10 Ensure that the Container Network Interface file ownership is set to root:root (Manual)
[PASS] 1.1.11 Ensure that the etcd data directory permissions are set to 700 or more restrictive (Automated)
[FAIL] 1.1.12 Ensure that the etcd data directory ownership is set to etcd:etcd (Automated)
[PASS] 1.1.13 Ensure that the admin.conf file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.14 Ensure that the admin.conf file ownership is set to root:root (Automated)
[PASS] 1.1.15 Ensure that the scheduler.conf file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.16 Ensure that the scheduler.conf file ownership is set to root:root (Automated)
[PASS] 1.1.17 Ensure that the controller-manager.conf file permissions are set to 644 or more restrictive (Automated)
[PASS] 1.1.18 Ensure that the controller-manager.conf file ownership is set to root:root (Automated)
[PASS] 1.1.19 Ensure that the Kubernetes PKI directory and file ownership is set to root:root (Automated)
[PASS] 1.1.20 Ensure that the Kubernetes PKI certificate file permissions are set to 644 or more restrictive (Manual)
[PASS] 1.1.21 Ensure that the Kubernetes PKI key file permissions are set to 600 (Manual)
[INFO] 1.2 API Server
[WARN] 1.2.1 Ensure that the --anonymous-auth argument is set to false (Manual)
[PASS] 1.2.2 Ensure that the --basic-auth-file argument is not set (Automated)
[PASS] 1.2.3 Ensure that the --token-auth-file parameter is not set (Automated)
[PASS] 1.2.4 Ensure that the --kubelet-https argument is set to true (Automated)
[PASS] 1.2.5 Ensure that the --kubelet-client-certificate and --kubelet-client-key arguments are set as appropriate (Automated)
[FAIL] 1.2.6 Ensure that the --kubelet-certificate-authority argument is set as appropriate (Automated)
[PASS] 1.2.7 Ensure that the --authorization-mode argument is not set to AlwaysAllow (Automated)
[PASS] 1.2.8 Ensure that the --authorization-mode argument includes Node (Automated)
[PASS] 1.2.9 Ensure that the --authorization-mode argument includes RBAC (Automated)
[WARN] 1.2.10 Ensure that the admission control plugin EventRateLimit is set (Manual)
[PASS] 1.2.11 Ensure that the admission control plugin AlwaysAdmit is not set (Automated)
[WARN] 1.2.12 Ensure that the admission control plugin AlwaysPullImages is set (Manual)
[WARN] 1.2.13 Ensure that the admission control plugin SecurityContextDeny is set if PodSecurityPolicy is not used (Manual)
[PASS] 1.2.14 Ensure that the admission control plugin ServiceAccount is set (Automated)
[PASS] 1.2.15 Ensure that the admission control plugin NamespaceLifecycle is set (Automated)
[FAIL] 1.2.16 Ensure that the admission control plugin PodSecurityPolicy is set (Automated)
[PASS] 1.2.17 Ensure that the admission control plugin NodeRestriction is set (Automated)
[PASS] 1.2.18 Ensure that the --insecure-bind-address argument is not set (Automated)
[PASS] 1.2.19 Ensure that the --insecure-port argument is set to 0 (Automated)
[PASS] 1.2.20 Ensure that the --secure-port argument is not set to 0 (Automated)
[FAIL] 1.2.21 Ensure that the --profiling argument is set to false (Automated)
[FAIL] 1.2.22 Ensure that the --audit-log-path argument is set (Automated)
[FAIL] 1.2.23 Ensure that the --audit-log-maxage argument is set to 30 or as appropriate (Automated)
[FAIL] 1.2.24 Ensure that the --audit-log-maxbackup argument is set to 10 or as appropriate (Automated)
[FAIL] 1.2.25 Ensure that the --audit-log-maxsize argument is set to 100 or as appropriate (Automated)
[PASS] 1.2.26 Ensure that the --request-timeout argument is set as appropriate (Automated)
[PASS] 1.2.27 Ensure that the --service-account-lookup argument is set to true (Automated)
[PASS] 1.2.28 Ensure that the --service-account-key-file argument is set as appropriate (Automated)
[PASS] 1.2.29 Ensure that the --etcd-certfile and --etcd-keyfile arguments are set as appropriate (Automated)
[PASS] 1.2.30 Ensure that the --tls-cert-file and --tls-private-key-file arguments are set as appropriate (Automated)
[PASS] 1.2.31 Ensure that the --client-ca-file argument is set as appropriate (Automated)
[PASS] 1.2.32 Ensure that the --etcd-cafile argument is set as appropriate (Automated)
[WARN] 1.2.33 Ensure that the --encryption-provider-config argument is set as appropriate (Manual)
[WARN] 1.2.34 Ensure that encryption providers are appropriately configured (Manual)
[WARN] 1.2.35 Ensure that the API Server only makes use of Strong Cryptographic Ciphers (Manual)
[INFO] 1.3 Controller Manager
[WARN] 1.3.1 Ensure that the --terminated-pod-gc-threshold argument is set as appropriate (Manual)
[FAIL] 1.3.2 Ensure that the --profiling argument is set to false (Automated)
[PASS] 1.3.3 Ensure that the --use-service-account-credentials argument is set to true (Automated)
[PASS] 1.3.4 Ensure that the --service-account-private-key-file argument is set as appropriate (Automated)
[PASS] 1.3.5 Ensure that the --root-ca-file argument is set as appropriate (Automated)
[PASS] 1.3.6 Ensure that the RotateKubeletServerCertificate argument is set to true (Automated)
[PASS] 1.3.7 Ensure that the --bind-address argument is set to 127.0.0.1 (Automated)
[INFO] 1.4 Scheduler
[FAIL] 1.4.1 Ensure that the --profiling argument is set to false (Automated)
[PASS] 1.4.2 Ensure that the --bind-address argument is set to 127.0.0.1 (Automated)

== Remediations master ==
1.1.9 Run the below command (based on the file location on your system) on the master node.
For example,
chmod 644 <path/to/cni/files>

1.1.10 Run the below command (based on the file location on your system) on the master node.
For example,
chown root:root <path/to/cni/files>

1.1.12 On the etcd server node, get the etcd data directory, passed as an argument --data-dir,
from the below command:
ps -ef | grep etcd
Run the below command (based on the etcd data directory found above).
For example, chown etcd:etcd /var/lib/etcd

1.2.1 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the below parameter.
--anonymous-auth=false

1.2.6 Follow the Kubernetes documentation and setup the TLS connection between
the apiserver and kubelets. Then, edit the API server pod specification file
/etc/kubernetes/manifests/kube-apiserver.yaml on the master node and set the
--kubelet-certificate-authority parameter to the path to the cert file for the certificate authority.
--kubelet-certificate-authority=<ca-string>

1.2.10 Follow the Kubernetes documentation and set the desired limits in a configuration file.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
and set the below parameters.
--enable-admission-plugins=...,EventRateLimit,...
--admission-control-config-file=<path/to/configuration/file>

1.2.12 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --enable-admission-plugins parameter to include
AlwaysPullImages.
--enable-admission-plugins=...,AlwaysPullImages,...

1.2.13 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --enable-admission-plugins parameter to include
SecurityContextDeny, unless PodSecurityPolicy is already in place.
--enable-admission-plugins=...,SecurityContextDeny,...

1.2.16 Follow the documentation and create Pod Security Policy objects as per your environment.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --enable-admission-plugins parameter to a
value that includes PodSecurityPolicy:
--enable-admission-plugins=...,PodSecurityPolicy,...
Then restart the API Server.

1.2.21 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the below parameter.
--profiling=false

1.2.22 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --audit-log-path parameter to a suitable path and
file where you would like audit logs to be written, for example:
--audit-log-path=/var/log/apiserver/audit.log

1.2.23 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --audit-log-maxage parameter to 30 or as an appropriate number of days:
--audit-log-maxage=30

1.2.24 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --audit-log-maxbackup parameter to 10 or to an appropriate
value.
--audit-log-maxbackup=10

1.2.25 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --audit-log-maxsize parameter to an appropriate size in MB.
For example, to set it as 100 MB:
--audit-log-maxsize=100

1.2.33 Follow the Kubernetes documentation and configure a EncryptionConfig file.
Then, edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the --encryption-provider-config parameter to the path of that file: --encryption-provider-config=</path/to/EncryptionConfig/File>

1.2.34 Follow the Kubernetes documentation and configure a EncryptionConfig file.
In this file, choose aescbc, kms or secretbox as the encryption provider.

1.2.35 Edit the API server pod specification file /etc/kubernetes/manifests/kube-apiserver.yaml
on the master node and set the below parameter.
--tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM
_SHA256,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_AES_256_GCM
_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_AES_256_GCM
_SHA384

1.3.1 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the --terminated-pod-gc-threshold to an appropriate threshold,
for example:
--terminated-pod-gc-threshold=10

1.3.2 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the below parameter.
--profiling=false

1.4.1 Edit the Scheduler pod specification file /etc/kubernetes/manifests/kube-scheduler.yaml file
on the master node and set the below parameter.
--profiling=false


== Summary master ==
45 checks PASS
10 checks FAIL   #10个错误
10 checks WARN
0 checks INFO

== Summary total ==
45 checks PASS
10 checks FAIL
10 checks WARN
0 checks INFO
```
修改其中一个错误，借助CIS_Kubernetes_Benchmark_v1.6.0.pdf搜索
链接：[https://pan.baidu.com/s/1C6vQSfDG0Twiy97QousbSQ](%E4%BF%AE%E6%94%B9%E5%85%B6%E4%B8%AD%E4%B8%80%E4%B8%AA%E9%94%99%E8%AF%AF%EF%BC%8C%E5%80%9F%E5%8A%A9CIS_Kubernetes_Benchmark_v1.6.0.pdf%E6%90%9C%E7%B4%A2%20%E9%93%BE%E6%8E%A5%EF%BC%9Ahttps://pan.baidu.com/s/1C6vQSfDG0Twiy97QousbSQ%20%E6%8F%90%E5%8F%96%E7%A0%81%EF%BC%9Ar6vn)
提取码：r6vn
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210615142303379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:~/cks/metadata# useradd etcd
root@master:~/cks/metadata# chown etcd:etcd /var/lib/etcd
```
再次检查运行

```bash
root@master:~/cks/metadata# docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest master --version 1.20

............#检查通过
[PASS] 1.1.12 Ensure that the etcd data directory ownership is set to etcd:etcd (Automated)
...........

== Summary total ==
46 checks PASS
9 checks FAIL  #比上次少了一个检查失败的指标
10 checks WARN
0 checks INFO
```
### 7.2 检查node
远程到node主机

```bash
root@node:~/cks/metadata# docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest node --version 1.20
```


参考链接：
[https://cloud.tencent.com/developer/article/1680433](https://cloud.tencent.com/developer/article/1680433)
