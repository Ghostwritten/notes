


---

 - [kubernetes【工具】kind【1】入门实践](https://ghostwritten.blog.csdn.net/article/details/121968488)
 - [kubernetes【工具】kind【2】集群配置](https://ghostwritten.blog.csdn.net/article/details/122171462)

 - [https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/)

---
## 1. 简介
[Kind](https://github.com/kubernetes-sigs/kind)（Kubernetes in Docker） 是一个 Kubernetes 孵化项目，Kind 是一套开箱即用的 Kubernetes 环境搭建方案。顾名思义，就是将 Kubernetes 所需要的所有组件，全部部署在一个 Docker 容器中，可以很方便的搭建 Kubernetes 集群。

Kind 已经广泛的应用于 Kubernetes 上游及相关项目的 CI 环境中，官方文档中也把 Kind 作为一种本地集群搭建的工具推荐给大家。

##  2. Kind 可以做什么？

 1. 快速创建一个或多个 Kubernetes 集群
 2. 支持部署高可用的 Kubernetes 集群
 3. 支持从源码构建并部署一个 Kubernetes 集群
 4. 可以快速低成本体验一个最新的 Kubernetes 集群，并支持 Kubernetes 的绝大部分功能
 5. 支持本地离线运行一个多节点集群

##  3. Kind 有哪些优势？

 1. 最小的安装依赖，仅需要安装 Docker 即可
 2. 使用方法简单，只需 Kind Cli 工具即可快速创建集群
 3. 使用容器来模似 Kubernetes 节点
 4. 内部使用 Kubeadm 的官方主流部署工具
 5. 通过了 CNCF 官方的 K8S Conformance 测试


##  4. Kind 是如何工作的？
`Kind` 使用容器来模拟每一个 Kubernetes 节点，并在容器里面运行 `Systemd`。 容器里的 `Systemd` 托管了 Kubelet 和 `Containerd`，然后容器内部的 Kubelet 把其它 Kubernetes 组件：Kube-Apiserver、Etcd、CNI 等等组件运行起来。

Kind 内部使用了 Kubeadm 这个工具来做集群的部署，包括高可用集群也是借助 Kubeadm 提供的特性来完成的。在高用集群下还会额外部署了一个 Nginx 来提供负载均衡 VIP。

##  5. 准备
### 5.1 安装docker
[https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

```bash
systemctl start docker && systemctl enable docker
```

###  5.2 安装kubectl
官方：[https://kubernetes.io/docs/tasks/tools/#install-kubectl](https://kubernetes.io/docs/tasks/tools/#install-kubectl)

```bash
 yum install kubectl
```
###  5.3 安装 Kind
Kind 使用 Golang 进行开发，原生支持良好的跨平台特性，通常只需要直接下载构建好的二进制文件就可使用。

**Linux**

```bash
$ curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.11.0/kind-linux-amd64
$ chmod +x ./kind
$ mv ./kind /usr/local/bin/kind
```
**Windows**

```bash
$ curl.exe -Lo kind-windows-amd64.exe https://github.com/kubernetes-sigs/kind/releases/download/v0.5.1/kind-windows-amd64
$ mv .\kind-windows-amd64.exe c:\kind.exe
```
更多平台的安装方法可参考[官方文档](https://kind.sigs.k8s.io/docs/user/quick-start/)

源码
如果本地环境已经配置好 `Golang` (1.11+) 的开发环境，你也可以直接通过源码进行安装。

```bash
$ go get sigs.k8s.io/kind@v0.5.1
```
## 6. kind命令

```bash
$ kind
kind creates and manages local Kubernetes clusters using Docker container 'nodes'

Usage:
  kind [command]

Available Commands:
  build       Build one of [base-image, node-image]
  create      Creates one of [cluster]
  delete      Deletes one of [cluster]
  export      exports one of [logs]
  get         Gets one of [clusters, nodes, kubeconfig-path]
  help        Help about any command
  load        Loads images into nodes
  version     prints the kind CLI version

Flags:
  -h, --help              help for kind
      --loglevel string   logrus log level [panic, fatal, error, warning, info, debug] (default "warning")
      --version           version for kind

Use "kind [command] --help" for more information about a command.
```
简单说下几个比较常用选项的含义：

 - build：用来从 Kubernetes 源代码构建一个新的镜像。
 - create：创建一个 Kubernetes 集群。
 - delete：删除一个 Kubernetes 集群。
 - get： 可用来查看当前集群、节点信息以及 Kubectl 配置文件的地址。
 - load：从宿主机向 Kubernetes 节点内导入镜像。


##  7. 创建集群

```bash
#默认集群名字是kind
kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

$ kind get  clusters 
kind


#创建第二个集群
$ kind create cluster --name kind-2
Creating cluster "kind-2" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind-2"
You can now use your cluster with:

kubectl cluster-info --context kind-kind-2

Have a nice day! 👋



$ kind get clusters
kind
kind-2

#为了与特定的集群交互，你只需要在kubectl中指定集群名作为上下文:
$ kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:34804
CoreDNS is running at https://127.0.0.1:34804/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.


$ kubectl cluster-info --context kind-kind-2
Kubernetes control plane is running at https://127.0.0.1:39972
CoreDNS is running at https://127.0.0.1:39972/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.


$ kubectl get node --context kind-kind
NAME                 STATUS   ROLES                  AGE   VERSION
kind-control-plane   Ready    control-plane,master   10m   v1.21.1

$ kubectl get node --context kind-kind-2
NAME                   STATUS   ROLES                  AGE     VERSION
kind-2-control-plane   Ready    control-plane,master   7m10s   v1.21.1


$ kubectl get pod -n kube-system --context kind-kind-2
NAME                                           READY   STATUS    RESTARTS   AGE
coredns-558bd4d5db-25zcm                       1/1     Running   0          8m22s
coredns-558bd4d5db-w5bnx                       1/1     Running   0          8m22s
etcd-kind-2-control-plane                      1/1     Running   0          8m24s
kindnet-2wbfz                                  1/1     Running   0          8m22s
kube-apiserver-kind-2-control-plane            1/1     Running   0          8m24s
kube-controller-manager-kind-2-control-plane   1/1     Running   0          8m24s
kube-proxy-b67sv                               1/1     Running   0          8m22s
kube-scheduler-kind-2-control-plane            1/1     Running   0          8m24s


$ kubectl get pod -n kube-system --context kind-kind
NAME                                         READY   STATUS    RESTARTS   AGE
coredns-558bd4d5db-fvpxx                     1/1     Running   0          12m
coredns-558bd4d5db-v9mwb                     1/1     Running   0          12m
etcd-kind-control-plane                      1/1     Running   0          12m
kindnet-78zdc                                1/1     Running   0          12m
kube-apiserver-kind-control-plane            1/1     Running   0          12m
kube-controller-manager-kind-control-plane   1/1     Running   0          12m
kube-proxy-28lm4                             1/1     Running   0          12m
kube-scheduler-kind-control-plane            1/1     Running   0          12m



#修改默认集群
#当前
$ kubectl get node
NAME                 STATUS   ROLES                  AGE   VERSION
kind-2-control-plane   Ready    control-plane,master   46m   v1.21.1


$ cat /root/.kube/config 
······················
contexts:
- context:
    cluster: kind-kind
    user: kind-kind
  name: kind-kind
- context:
    cluster: kind-kind-2
    user: kind-kind-2
  name: kind-kind-2
current-context: kind-kind-2   #修改kind-kind即可切换到kind集群
kind: Config
preferences: {}
users:
·············


$ kubectl get nodes
NAME                 STATUS   ROLES                  AGE   VERSION
kind-control-plane   Ready    control-plane,master   46m   v1.21.1



$ kind get kubeconfig --name=kind
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1USXlOREUxTXpJek0xb1hEVE14TVRJeU1qRTFNekl6TTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS3FYCjlHOHBYQ21BbVI2c2dmZ1hSQW9rSXZ2Yi94YXUvRVlLVm1rTmJCUWlrb3B0bnFHcTdMdTBQajRRb0RSU0MxRDYKbjhsc0tsMnlZYi9oVXp6UjRRUG1Ld1ZSWk1lZExpQ3pHMWIrN3Awd0tzcG5aMlh5dFZCMHBHWVJlTUR4OWkvSwpqZi8zQjBOMUwyNHMyWDRMZi9obVZSd3RXcGNCMytmTE95bmVYSjVWM0VDallTTFUvMnZ3TFFTcVpseUIwUzZZCm5TQTlzcWpmMzIyRjJJUHZvTTlwa3prZzhDZ3NWZVhIdFNXZ2FqbVA3OVdsM1NIeGZaVTBJNWljT2VvaStucXUKanVoTHlqbmdEYllxYzRZc1FXVE5JRTVvNURZU1JFM2xJNm81YWVoK1ZQYWVsNGFUNFg3YldCTXMzblVwcFFFYgpMUVRSZW80RFJQbERPc20zbHA4Q0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZKUkVHYXpJNmRWRG1hcEY3YlNMdHQybGFaNy9NQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBOEVsTzRmVUtpazZCMDNqdTI2c1NZM1dSeTVEczVZd1A3dTdIbXRoa1dpZ2JocE9hWApQaEt4WWZPbjhPQUdSRXVndXVmSUVzOVNDclJXRVJtbU9uckJRMVEzMlFuSDdRUFA5OGxhbEhBK0lrZDE2cWFWCno0S0Z0MVo2SjFTQTZwRDhlSEVMRUd0SmtlMUF5TVhEdVVZRkcwWlFkcjJwL2xxM2x4L2Z6ZzIrWXpNTzUrMGQKc3NINkk5N0R6azlqdGh2T0RIYmdxbWx1cHdQd21WRjFmQnVybDZKTXFBcm5xeVNEYjFRdnN2YUs1MmpWaEtCcwoyVTRDSXdoSWhXcWx5c1BTS1d0WlczcGtKNVRTbUxpUXBQYnNZLzFqVmk3d0l3ZXFrTk5EQWZ4NkRualN5Mm9VCkJPUFJ6M1BtSjRRRWtUdXNsM1h0RHoyd3JiTGI3TFJjUUFtSwotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://127.0.0.1:34804
  name: kind-kind
contexts:
- context:
    cluster: kind-kind
    user: kind-kind
  name: kind-kind
current-context: kind-kind
kind: Config
preferences: {}
users:
- name: kind-kind
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJY2pOclVNc3g2RlF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TVRFeU1qUXhOVE15TXpOYUZ3MHlNakV5TWpReE5UTXlNelJhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXhETHVOelRJNjUyUldVenMKYjlRbFIrZGxHOUErZTNSNWtxajZSOHdSUEdMWmd1YnVvRHkyd2dqSWlKUGhWNDFlRjBaeVEwenMyS0hpYkJzRwozemJLd05LQTdXYk5GYnBDWlNReVBad1MzUUQ3NklzR3FCY3RDOXhMcE9ocE1EQnlwanYrdnNyU2pDQnk4NVEyCkxIYWkvZVE1OXFWQkRuS2hhMkxITU1pV09MYk5RZUNsQThnUE9wMzRTSUdBdFZHaDlraXp0SjNxdENKU204bFoKcXBseEh0ZTYyRk02Ty80VVdaV1BoQ0piSmJISkRXVEtPaDNTL2VYRE9JQzl2VlE3Ny9iQVNIY2hjNVFocElsUgp3LzBiS1o4bGVUaVhJZmZMOFljT29WUVgzejIrNHhsMXAwQkFaTlBMWFlCd1ZFOXoxeTYwNXh6cXZlMVRpQ3BCCkx3QmE0UUlEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JTVVJCbXN5T25WUTVtcVJlMjBpN2JkcFdtZQovekFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBRkhNVnlxZFlDMk5hdSswMTlZOTZVVDNwUmZCc0tJNE5HY2xiCno5a004ZkRmMTg3aGpneFF5KzIyYXdoOUpxMmIxOWZhK1ovb0E0YW81VjRTcy9ZdjQxMFJFYXhWblFFdkt4OHIKblZzNXYwbGdXd1VoTGVlK09DUzh1V3FKN1NvTWVZYmNhTUU2NFJ4ZnVCQTZEaUUzVEh0SCtKcmNpbm90ZWI4aApVT2hiNHVER1lOTyt2SVk5OW1xSGZzTlM4S3JBQmJieFFGZHBlTUQ1ZmtKMWE4alhqOGdyTFNYYjUzM2JTclhXCjhiSm5yU2VrRkI0ck9wdnV2RVk2bTNVbXo4TVZ3WWRIYVBPYlRMSXJQU0wzalVwWUx1RGFZY0VNaUtSeUpacVcKU3A4R3FoTVR5bFlQaHdYUkZVaUNyVWwvODNFdDMvbGczWndQUzFVb0Z3U2U2aHhUeVE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBeERMdU56VEk2NTJSV1V6c2I5UWxSK2RsRzlBK2UzUjVrcWo2Ujh3UlBHTFpndWJ1Cm9EeTJ3Z2pJaUpQaFY0MWVGMFp5UTB6czJLSGliQnNHM3piS3dOS0E3V2JORmJwQ1pTUXlQWndTM1FENzZJc0cKcUJjdEM5eExwT2hwTURCeXBqdit2c3JTakNCeTg1UTJMSGFpL2VRNTlxVkJEbktoYTJMSE1NaVdPTGJOUWVDbApBOGdQT3AzNFNJR0F0VkdoOWtpenRKM3F0Q0pTbThsWnFwbHhIdGU2MkZNNk8vNFVXWldQaENKYkpiSEpEV1RLCk9oM1MvZVhET0lDOXZWUTc3L2JBU0hjaGM1UWhwSWxSdy8wYktaOGxlVGlYSWZmTDhZY09vVlFYM3oyKzR4bDEKcDBCQVpOUExYWUJ3VkU5ejF5NjA1eHpxdmUxVGlDcEJMd0JhNFFJREFRQUJBb0lCQUV5L0wzZmc2Z2RncDQ2cgpESUhpRm9NOS9Nc1lkcGlNUTFJZlQyZnVaMytibTBJZFc1TEtyU0xSbEwvNE9ObXFydmVqMHVhSW5NMVE1ZVVyCjNWQkxlcHhhdTV3aDdtOWxZTHQzb1QrQVlkQ1pwZkNkRVltSEoxUFFaTGFwUXh4YWx6NTNrWHJJay92RVpiTHEKY3hhSmdkQ1hDaVYxRnpHem5Ya0lOcXJhakFpNnFTenRjL0ZVWklwVGw4a1ZJdEc5RUh1NnN6eFh3NU1NdG1ZeQpWbGZnSitlallIVUoyTHBVVk8zNDBVc25NUTZham1nQ0tEUEFZd05xd1hoNE5xcUhMMXAzL3A2M2RVbVRUUFVQCmhBbHgxeWtRR2FJRlBXREFLVFA2N0dEeEdjMGJGVGl2T0gveFloUmxaOUhuVUlOeE9mcDd3SkVvVmRhVWJJWXoKSlRsSFRKRUNnWUVBemJDNmZPSGovbGpNZVZFRHVPOXUrRHcyWDlibWJrZysxTEk2N1F3ZDQzcFI1a3FPT1J6YQozUVZ5ZmhNdDNDeTNLVDhZUER1dmZFZ3huV3Z4cEh5YWVYMW9YdjVVeHpZN0lGeWNWWXJLdTcyZHVxUHRtVXFQCnBOWEdDaTduRW9OUUJNRFc1NDhxQUtUd3dtR2hMZnhFSFl3MjdxZzN5b1FUUU9GZ0FaQ1E4Z2NDZ1lFQTlDL20KUnlRVm5oaW83cEFsdnNIRnVMS21jeHlaSzJNaldQbWxkMnYzK3dMTUlXYk5JcUg3bzZJaFBQQzdBOFNUbVZscQo4eWNNU2ZnN2dpZ3NYYnFuMkk5U2ZrMDZvMmdLc0pjLzI1Um80OWk5Y2lsaXpkbFpSaTdzL2UvK2xydUxtZyt1Cmd1c05LVno5dHlWWHFtajJqVmFkT3pFdmJuSmp4S2VzRG9ZUWNkY0NnWUVBdlFTNEMwVUdZRmkzNXBCRmJIQlQKT0xrVWVyUWdZNTN1WjBVMkUxbzhLU3ZpRVUvWUxMSFFpcVdUMWpuSHZmbzFneWpoRzVENXJhc21OUFRhUlg3Zgo4ZDhGeDYzT3VKYWtkUlBGOG5JdDVhTFZUSXVTTDNrdVVackZkOXdzS240VFRacnNvalNVczZ6Zk5ySEREV0F5Ck5Ea0N6Z1ExNk52QVdiSUNxTTF4OVljQ2dZRUFsMWtJOVpjYi92MXgxMHRvMmE3b2llM1ExUkFvcjRlbTVRTDIKMStvSHJZQ3lYUkdHbTZ5aWQyMktCR2VBd25rWXNyZUZYbWdaYWM5OXN0S0xqUnlmNDg0UlowOGV4U0U3WHZDZwpGODBJcGhBMGU0bkRQNnN6ZGhpbnMwMEpFd3Z6SHU0UlQvdTRFS2NlYW1HdTBHUjJUR3dlMEExUVJMaUp0ZDNtCitxbUZqOGtDZ1lFQW9ORUNKSVFDZzJKYko0c1dLT2F6d1pzZG8xdm5GQkNScXpKcGlqMGZDTkV2MnFjNmxYY2oKMzFNQmk1d2lRYzZnem9SR3p1TDhCby9CT1p2SVA5ZVBJeDZscmpHaWdodFpNenh0OGlVb3JKa29wYi9nb01ZRQpJU0VMZi9jeTExdkw4aTdGMjBpd095ZWFWa1NkWmRuZnJiZHBZeWpueUdTZ2dwUkxhNnJPNnY4PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

```

>  --name 是可选参数。如果不指定，默认创建出来的集群名字为 kind。

用默认安装的方式时，我们没有指定任何配置文件。从安装过程的输出来看，一共分为 4 步：

 - 检查本地环境是否存在一个基础的安装镜像，默认是 `kindest/node:1.21.1`，该镜像里面包含了所有需要安装的东西，包括：`kubectl`、`kubeadm`、`kubelet`
   的二进制文件，以及安装对应版本 `Kubernetes` 所需要的镜像。
 - 准备 Kubernetes 节点，主要就是启动容器、解压镜像这类的操作。
 - 建立对应的 `kubeadm` 的配置，完成之后就通过 kubeadm 进行安装。安装完成后还会做一些清理操作，比如：删掉主节点上的污点，否则对于没有容忍的 Pod 无法完成部署。
 - 上面所有操作都完成后，就成功启动了一个 Kubernetes 集群并输出一些操作集群的提示信息。

默认情况下，Kind 会先下载 `kindest/node:v1.21.1` 镜像。如果你想指定不同版本，可以使用 `--image` 参数，类似这样：`kind create cluster --image kindest/node:v1.21.1`

`kindest/node` 这个镜像目前托管于 `Docker Hub` 上，下载时可能会较慢。同样的问题 Kind 进行集群的创建也是存在的，Kind 实际使用 Kubeadm 进行集群的创建。对 Kubeadm 有所了解的同学都知道它默认使用的镜像在国内是不能访问的，所以一样需要自行解决网络问题。

如果你存在上面说的网络问题，最好配置一个国内的加速器或者镜像源。如果你还不知道如何配置加速器和镜像源可以参考：「[Docker / Kubernetes 镜像源不可用，教你几招搞定它！](https://mp.weixin.qq.com/s?__biz=MzI3MTI2NzkxMA==&mid=2247488553&idx=1&sn=14cbe47bc50df50f536345efb4d10b5e&chksm=eac53500ddb2bc16be6bbfe69917895d0feed7c4e85d40450a5adb931ff01257d9f546c58538&token=687022088&lang=zh_CN#rd)」和 「 [Docker 下使用 DaoCloud / 阿里云镜像加速](https://mp.weixin.qq.com/s?__biz=MzI3MTI2NzkxMA==&mid=2247483698&idx=1&sn=dfb6edca74539a9a4d8228495d1c17a0&chksm=eac5201bddb2a90d5f8e6d4733ed2d0ff9466474471933e4ea5d5fc1ef6be15b86470f50eccb&token=21948731&lang=zh_CN#rd)」两篇文章。


##  8. 载入镜像

```bash
$ kind load docker-image busybox alpine --name kind
ERROR: image: "busybox" not present locally

$ docker pull busybox
Using default tag: latest
Trying to pull repository docker.io/library/busybox ... 
latest: Pulling from docker.io/library/busybox
3cb635b06aa2: Pull complete 
Digest: sha256:b5cfd4befc119a590ca1a81d6bb0fa1fb19f1fbebd0397f25fae164abe1e8a6a
Status: Downloaded newer image for docker.io/busybox:latest


$ docker pull alpine
Using default tag: latest
Trying to pull repository docker.io/library/alpine ... 
latest: Pulling from docker.io/library/alpine
59bf1c3509f3: Pull complete 
Digest: sha256:21a3deaa0d32a8057914f36584b5288d2e5ecc984380bc0118285c70fa8c9300
Status: Downloaded newer image for docker.io/alpine:latest

#一定不要用“latest”，因为它会在创建实例的时候失败。
$ docker tag docker.io/busybox:latest busybox:my-latest
$ docker tag docker.io/alpine:latest alpine:my-latest

$ kind load docker-image busybox alpine --name kind
Image: "busybox" with ID "sha256:ffe9d497c32414b1c5cdad8178a85602ee72453082da2463f1dede592ac7d5af" not yet present on node "kind-control-plane", loading...
Image: "alpine" with ID "sha256:c059bfaa849c4d8e4aecaeb3a10c2d9b3d85f5165c66ad3a4d937758128c4d18" not yet present on node "kind-control-plane", loading...

```
工作流workflow like：

```bash
docker build -t my-custom-image:unique-tag ./my-image-dir
kind load docker-image my-custom-image:unique-tag
kubectl apply -f my-manifest-using-my-image:unique-tag
```
查看各个集群镜像：

```bash
$ docker ps
CONTAINER ID        IMAGE                                                                                          COMMAND                  CREATED             STATUS              PORTS                       NAMES
894613da2a0a        kindest/node:v1.21.1@sha256:fae9a58f17f18f06aeac9772ca8b5ac680ebbed985e266f711d936e91d113bad   "/usr/local/bin/en..."   36 minutes ago      Up 36 minutes       127.0.0.1:39972->6443/tcp   kind-2-control-plane
a9171fe66dc1        kindest/node:v1.21.1@sha256:fae9a58f17f18f06aeac9772ca8b5ac680ebbed985e266f711d936e91d113bad   "/usr/local/bin/en..."   38 minutes ago      Up 38 minutes       127.0.0.1:34804->6443/tcp   kind-control-plane



$ docker exec -ti kind-control-plane crictl images
IMAGE                                      TAG                  IMAGE ID            SIZE
docker.io/kindest/kindnetd                 v20210326-1e038dc5   6de166512aa22       54MB
docker.io/library/alpine                   latest               c059bfaa849c4       5.87MB
docker.io/library/busybox                  latest               ffe9d497c3241       1.46MB
docker.io/rancher/local-path-provisioner   v0.0.14              e422121c9c5f9       13.4MB
k8s.gcr.io/build-image/debian-base         v2.1.0               c7c6c86897b63       21.1MB
k8s.gcr.io/coredns/coredns                 v1.8.0               296a6d5035e2d       12.9MB
k8s.gcr.io/etcd                            3.4.13-0             0369cf4303ffd       86.7MB
k8s.gcr.io/kube-apiserver                  v1.21.1              6401e478dcc01       127MB
k8s.gcr.io/kube-controller-manager         v1.21.1              d0d10a483067a       121MB
k8s.gcr.io/kube-proxy                      v1.21.1              ebd41ad8710f9       133MB
k8s.gcr.io/kube-scheduler                  v1.21.1              7813cf876a0d4       51.9MB
k8s.gcr.io/pause                           3.4.1                0f8457a4c2eca       301kB


$ docker exec -ti kind-2-control-plane crictl images
IMAGE                                      TAG                  IMAGE ID            SIZE
docker.io/kindest/kindnetd                 v20210326-1e038dc5   6de166512aa22       54MB
docker.io/rancher/local-path-provisioner   v0.0.14              e422121c9c5f9       13.4MB
k8s.gcr.io/build-image/debian-base         v2.1.0               c7c6c86897b63       21.1MB
k8s.gcr.io/coredns/coredns                 v1.8.0               296a6d5035e2d       12.9MB
k8s.gcr.io/etcd                            3.4.13-0             0369cf4303ffd       86.7MB
k8s.gcr.io/kube-apiserver                  v1.21.1              6401e478dcc01       127MB
k8s.gcr.io/kube-controller-manager         v1.21.1              d0d10a483067a       121MB
k8s.gcr.io/kube-proxy                      v1.21.1              ebd41ad8710f9       133MB
k8s.gcr.io/kube-scheduler                  v1.21.1              7813cf876a0d4       51.9MB
k8s.gcr.io/pause                           3.4.1                0f8457a4c2eca       301kB
```




##  9. 部署一个服务


```bash
$ docker pull nginx:1.14.2
$ kind load docker-image nginx:1.14.2 --name kind
Image: "nginx" with ID "sha256:f6987c8d6ed59543e9f34327c23e12141c9bad1916421278d720047ccc8e1bee" not yet present on node "kind-control-plane", loading...

$ vim nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80

kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/2     2            0           39s


kubectl get pods         
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-66b6c48dd5-2s5cb   1/1     Running   0          84s
nginx-deployment-66b6c48dd5-8wf8b   1/1     Running   0          84s


```
由于我们没有做服务暴露，所以是不能直接访问对应的服务的，我们可以用 kubectl 提供的端口转发功能来讲流量从本地转发给 k8s 集群。


## 10. 删除集群

```bash
$ kind delete clusters my-cluster
Deleted clusters: ["my-cluster"]
```


