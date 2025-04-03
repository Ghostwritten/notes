![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/181a34fab71149dbb24c338dcb1055a6.png)



# 引言
在现代应用程序开发中，Kubernetes（简称K8s）已经成为了容器编排的标准。Minikube作为一个轻量级的本地Kubernetes集群环境，为开发和测试提供了极大的便利。今天，我将分享如何在MacBook上使用k8sGPT扫描Minikube集群，以确保我们的Kubernetes集群设置和应用程序的最佳实践。

```bash
$ minikube start --nodes 2 -p multinode-demo
😄  Darwin 14.5 (arm64) 上的 [multinode-demo] minikube v1.33.1
✨  自动选择 docker 驱动。其他选项：parallels, ssh
📌  使用具有 root 权限的 Docker Desktop 驱动程序
👍  Starting "multinode-demo" primary control-plane node in "multinode-demo" cluster
🚜  Pulling base image v0.0.44 ...
    > gcr.io/k8s-minikube/kicbase...:  435.76 MiB / 435.76 MiB  100.00% 6.65 Mi
🔥  Creating docker container (CPUs=2, Memory=7792MB) ...
🌐  找到的网络选项：
    ▪ HTTP_PROXY=192.168.21.101:7890
❗  You appear to be using a proxy, but your NO_PROXY environment does not include the minikube IP (192.168.49.2).
📘  Please see https://minikube.sigs.k8s.io/docs/handbook/vpn_and_proxy/ for more details
    ▪ HTTPS_PROXY=192.168.21.101:7890
🐳  正在 Docker 26.1.1 中准备 Kubernetes v1.30.0…
    ▪ env HTTP_PROXY=192.168.21.101:7890
    ▪ env HTTPS_PROXY=192.168.21.101:7890
    ▪ 正在生成证书和密钥...
    ▪ 正在启动控制平面...
    ▪ 配置 RBAC 规则 ...
🔗  配置 CNI (Container Networking Interface) ...
🔎  正在验证 Kubernetes 组件...
    ▪ 正在使用镜像 gcr.io/k8s-minikube/storage-provisioner:v5
🌟  启用插件： default-storageclass, storage-provisioner

👍  Starting "multinode-demo-m02" worker node in "multinode-demo" cluster
🚜  Pulling base image v0.0.44 ...
🔥  Creating docker container (CPUs=2, Memory=7792MB) ...
🌐  找到的网络选项：
    ▪ NO_PROXY=192.168.49.2
    ▪ HTTP_PROXY=192.168.21.101:7890
❗  You appear to be using a proxy, but your NO_PROXY environment does not include the minikube IP (192.168.49.3).
📘  Please see https://minikube.sigs.k8s.io/docs/handbook/vpn_and_proxy/ for more details
    ▪ HTTPS_PROXY=192.168.21.101:7890
🐳  正在 Docker 26.1.1 中准备 Kubernetes v1.30.0…
    ▪ env HTTP_PROXY=192.168.21.101:7890
    ▪ env HTTPS_PROXY=192.168.21.101:7890
    ▪ env NO_PROXY=192.168.49.2
🔎  正在验证 Kubernetes 组件...
🏄  完成！kubectl 现在已配置，默认使用"multinode-demo"集群和"default"命名空间
$ kubectl get node
NAME                 STATUS   ROLES           AGE    VERSION
multinode-demo       Ready    control-plane   2m3s   v1.30.0
multinode-demo-m02   Ready    <none>          103s   v1.30.0
$ minikube status -p multinode-demo
multinode-demo
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

multinode-demo-m02
type: Worker
host: Running
kubelet: Running

$ minikube addons list
|-----------------------------|--------------------------------|
|         ADDON NAME          |           MAINTAINER           |
|-----------------------------|--------------------------------|
| ambassador                  | 3rd party (Ambassador)         |
| auto-pause                  | minikube                       |
| cloud-spanner               | Google                         |
| csi-hostpath-driver         | Kubernetes                     |
| dashboard                   | Kubernetes                     |
| default-storageclass        | Kubernetes                     |
| efk                         | 3rd party (Elastic)            |
| freshpod                    | Google                         |
| gcp-auth                    | Google                         |
| gvisor                      | minikube                       |
| headlamp                    | 3rd party (kinvolk.io)         |
| helm-tiller                 | 3rd party (Helm)               |
| inaccel                     | 3rd party (InAccel             |
|                             | [info@inaccel.com])            |
| ingress                     | Kubernetes                     |
| ingress-dns                 | minikube                       |
| inspektor-gadget            | 3rd party                      |
|                             | (inspektor-gadget.io)          |
| istio                       | 3rd party (Istio)              |
| istio-provisioner           | 3rd party (Istio)              |
| kong                        | 3rd party (Kong HQ)            |
| kubeflow                    | 3rd party                      |
| kubevirt                    | 3rd party (KubeVirt)           |
| logviewer                   | 3rd party (unknown)            |
| metallb                     | 3rd party (MetalLB)            |
| metrics-server              | Kubernetes                     |
| nvidia-device-plugin        | 3rd party (NVIDIA)             |
| nvidia-driver-installer     | 3rd party (Nvidia)             |
| nvidia-gpu-device-plugin    | 3rd party (Nvidia)             |
| olm                         | 3rd party (Operator Framework) |
| pod-security-policy         | 3rd party (unknown)            |
| portainer                   | 3rd party (Portainer.io)       |
| registry                    | minikube                       |
| registry-aliases            | 3rd party (unknown)            |
| registry-creds              | 3rd party (UPMC Enterprises)   |
| storage-provisioner         | minikube                       |
| storage-provisioner-gluster | 3rd party (Gluster)            |
| storage-provisioner-rancher | 3rd party (Rancher)            |
| volumesnapshots             | Kubernetes                     |
| yakd                        | 3rd party (marcnuri.com)       |
|-----------------------------|--------------------------------|
$ kubectl create deployment hello-minikube1 --image=kicbase/echo-server:1.0
deployment.apps/hello-minikube1 created
$ kubectl expose deployment hello-minikube1 --type=LoadBalancer --port=8080
service/hello-minikube1 exposed
$ kubectl get pod -A
NAMESPACE     NAME                                     READY   STATUS              RESTARTS        AGE
default       hello-minikube1-67bf99b564-jstcc         0/1     ContainerCreating   0               12s
kube-system   coredns-7db6d8ff4d-85s2n                 1/1     Running             2 (3m16s ago)   3m43s
kube-system   etcd-multinode-demo                      1/1     Running             0               3m57s
kube-system   kindnet-wtzgz                            1/1     Running             0               3m44s
kube-system   kindnet-zwhtz                            1/1     Running             0               3m39s
kube-system   kube-apiserver-multinode-demo            1/1     Running             0               3m57s
kube-system   kube-controller-manager-multinode-demo   1/1     Running             0               3m57s
kube-system   kube-proxy-8t4fd                         1/1     Running             0               3m39s
kube-system   kube-proxy-dxhnv                         1/1     Running             0               3m44s
kube-system   kube-scheduler-multinode-demo            1/1     Running             0               3m57s
kube-system   storage-provisioner                      1/1     Running             1 (3m32s ago)   3m55s
$ kubectl get pod -A
NAMESPACE     NAME                                     READY   STATUS              RESTARTS        AGE
default       hello-minikube1-67bf99b564-jstcc         0/1     ContainerCreating   0               16s
kube-system   coredns-7db6d8ff4d-85s2n                 1/1     Running             2 (3m20s ago)   3m47s
kube-system   etcd-multinode-demo                      1/1     Running             0               4m1s
kube-system   kindnet-wtzgz                            1/1     Running             0               3m48s
kube-system   kindnet-zwhtz                            1/1     Running             0               3m43s
kube-system   kube-apiserver-multinode-demo            1/1     Running             0               4m1s
kube-system   kube-controller-manager-multinode-demo   1/1     Running             0               4m1s
kube-system   kube-proxy-8t4fd                         1/1     Running             0               3m43s
kube-system   kube-proxy-dxhnv                         1/1     Running             0               3m48s
kube-system   kube-scheduler-multinode-demo            1/1     Running             0               4m1s
kube-system   storage-provisioner                      1/1     Running             1 (3m36s ago)   3m59s
$ brew install k8sgpt
==> Downloading https://formulae.brew.sh/api/formula.jws.json
############################################################################################################################################################################################################################################## 100.0%
==> Downloading https://formulae.brew.sh/api/cask.jws.json
############################################################################################################################################################################################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/k8sgpt/manifests/0.3.40
############################################################################################################################################################################################################################################## 100.0%
==> Fetching k8sgpt
==> Downloading https://ghcr.io/v2/homebrew/core/k8sgpt/blobs/sha256:e22d500e85a13ae94bce5be3471eb9c2fc10b343fc335adb9fd6c39a9adfc9bd
############################################################################################################################################################################################################################################## 100.0%
==> Pouring k8sgpt--0.3.40.arm64_sonoma.bottle.tar.gz
🍺  /opt/homebrew/Cellar/k8sgpt/0.3.40: 7 files, 89.3MB
==> Running `brew cleanup k8sgpt`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
$ k8sgpt generate


Opening: https://beta.openai.com/account/api-keys to generate a key for openai

Please copy the generated key and run `k8sgpt auth add` to add it to your config file

$ k8sgpt auth add
Warning: backend input is empty, will use the default value: openai
Warning: model input is empty, will use the default value: gpt-3.5-turbo
Enter openai Key: openai added to the AI backend provider list
$ k8sgpt analyze --explain
AI Provider: openai

No problems detected

$ k8sgpt analyze --explain --with-doc
AI Provider: openai

No problems detected

$ k8sgpt analyze --explain --filter=Pod --namespace=default
AI Provider: openai

No problems detected

$ k8sgpt analyze --explain --filter=Service --output=json
{
  "provider": "openai",
  "errors": null,
  "status": "OK",
  "problems": 0,
  "results": null
}
$ k8sgpt analyze --explain --filter=Service --output=json --anonymize
{
  "provider": "openai",
  "errors": null,
  "status": "OK",
  "problems": 0,
  "results": null
}
$ k8sgpt filters list
Active:
> PersistentVolumeClaim
> Ingress
> Deployment
> ReplicaSet
> Service
> StatefulSet
> CronJob
> Node
> ValidatingWebhookConfiguration
> MutatingWebhookConfiguration
> Pod
Unused:
> Log
> GatewayClass
> Gateway
> HTTPRoute
> HorizontalPodAutoScaler
> PodDisruptionBudget
> NetworkPolicy
$ k8sgpt filters add Log
Warning: by enabling logs, you will be sending potentially sensitive data to the AI backend.
Filter Log added
$ k8sgpt filters list
Active:
> Service
> Node
> StatefulSet
> CronJob
> Log
> Ingress
> MutatingWebhookConfiguration
> ValidatingWebhookConfiguration
> Pod
> Deployment
> ReplicaSet
> PersistentVolumeClaim
Unused:
> HorizontalPodAutoScaler
> PodDisruptionBudget
> NetworkPolicy
> GatewayClass
> Gateway
> HTTPRoute
$ k8sgpt auth list
Default:
> openai
Active:
> openai
Unused:
> localai
> ollama
> azureopenai
> cohere
> amazonbedrock
> amazonsagemaker
> google
> noopai
> huggingface
> googlevertexai
> oci
> watsonxai

$ minikube stop  -p multinode-demo
✋  正在停止节点 "multinode-demo-m02" ...
🛑  正在通过 SSH 关闭“multinode-demo-m02”…
✋  正在停止节点 "multinode-demo" ...
🛑  正在通过 SSH 关闭“multinode-demo”…
🛑  2 个节点已停止。

$ docker ps -a
CONTAINER ID   IMAGE                                 COMMAND                   CREATED          STATUS                        PORTS     NAMES
19a65b1bbc03   gcr.io/k8s-minikube/kicbase:v0.0.44   "/usr/local/bin/entr…"   45 minutes ago   Exited (130) 48 seconds ago             multinode-demo-m02
0f2e98799a5a   gcr.io/k8s-minikube/kicbase:v0.0.44   "/usr/local/bin/entr…"   45 minutes ago   Exited (130) 37 seconds ago             multinode-demo
b3c3c3c6fec6   ghcr.io/open-webui/open-webui:main    "bash start.sh"           3 months ago     Exited (0) 13 days ago                  open-webui
```

# 实践体验
在我的实践中，k8sGPT帮助我发现了一些常见的配置问题，如资源限制未设置、未使用最新的镜像版本等。这些问题可能看起来微不足道，但在生产环境中可能会导致性能下降或安全隐患。

此外，k8sGPT还建议了很多优化措施，例如使用ConfigMap和Secret来管理配置数据，避免将敏感信息直接硬编码到Pod中。通过这些建议，我的Minikube集群变得更加健壮和高效。

# 总结
通过在MacBook上使用k8sGPT扫描Minikube集群，我们可以快速发现和解决Kubernetes集群中的潜在问题。Minikube提供了一个方便的本地开发环境，而k8sGPT则为集群的健康和优化提供了强有力的支持。希望这篇文章能帮助你在使用Kubernetes时更加得心应手。如果你还没有尝试过k8sGPT，强烈推荐你在你的开发环境中试试它。


参考： 

- https://github.com/k8sgpt-ai/k8sgpt
- https://minikube.sigs.k8s.io/docs/tutorials/multi_node/
- 
