
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d892a291f5a0f4e6dff296be077a6bbd.png#pic_center)

---

## 1. 要求
考试模式：线上考试

考试时间：2小时

认证有效期：2年

软件版本：Kubernetes v1.19

系统：Ubuntu 18.4

有效期：考试资格自购买之日起12个月内有效

重考政策：可接受1次重考

经验水平：中级

**考试报名**：[https://training.linuxfoundation.cn/certificates/16](https://training.linuxfoundation.cn/certificates/16)

----
## 2. 考点内容
### 2.1 集群安装：10%

 - 使用网络安全策略来限制集群级别的访问
 - 使用CIS基准检查Kubernetes组件(etcd, kubelet, kubedns, kubeapi)的安全配置
 - 正确设置带有安全控制的Ingress对象
 - 保护节点元数据和端点
 - 最小化GUI元素的使用和访问
 - 在部署之前验证平台二进制文件

课程：

 - [Kubernetes CKS 2021—Network Policies](https://ghostwritten.blog.csdn.net/article/details/115913233)
 - [Kubernetes CKS 2021—Cluster Setup - GUI Elements](https://ghostwritten.blog.csdn.net/article/details/115913408)
 - [Kubernetes CKS 2021---Cluster Setup - Secure Ingress](https://ghostwritten.blog.csdn.net/article/details/115949825)
 - [Kubernetes CKS 2021---Cluster Setup - Node Metadata](https://ghostwritten.blog.csdn.net/article/details/115971711)
 - [Kubernetes CKS 2021---Cluster Setup - CIS Benchmarks](https://ghostwritten.blog.csdn.net/article/details/116006742)
 - [Kubernetes CKS 2021---Cluster Setup - Verify Platform](https://ghostwritten.blog.csdn.net/article/details/116019776)

### 2.2 集群强化：15%

 - 限制访问Kubernetes API
 - 使用基于角色的访问控制来最小化暴露
 - 谨慎使用服务帐户，例如禁用默认设置，减少新创建帐户的权限
 - 经常更新Kubernetes

课程：

 - [Kubernetes CKS 2021---Cluster Hardening - Restrict API Access](https://ghostwritten.blog.csdn.net/article/details/116135832)
 - [Kubernetes CKS 2021---Exercise caution in using ServiceAccounts](https://ghostwritten.blog.csdn.net/article/details/116133701)
 - [Kubernetes CKS 2021---Cluster Hardening -RBAC](https://ghostwritten.blog.csdn.net/article/details/116131268)
 - [Kubernetes CKS 2021---Cluster Hardening - Upgrade Kubernetes](https://ghostwritten.blog.csdn.net/article/details/116159731)

### 2.3 系统强化：15%

 - 最小化主机操作系统的大小(减少攻击面)
 - 最小化IAM角色
 - 最小化对网络的外部访问
 - 适当使用内核强化工具，如AppArmor, seccomp

课程：

 - [k8s CKS 2021---系统加固减少攻击面](https://ghostwritten.blog.csdn.net/article/details/117263565)
 - [Kubernetes CKS 2021---OS Level Security Domains](https://ghostwritten.blog.csdn.net/article/details/116789828)
 - [Kubernetes CKS 2021---System Hardening - Kernel Hardening Tools](https://ghostwritten.blog.csdn.net/article/details/117249927)

### 2.4 微服务漏洞最小化：20%

 - 设置适当的OS级安全域，例如使用PSP, OPA，安全上下文
 - 管理Kubernetes机密
 - 在多租户环境中使用容器运行时 (例如gvisor, kata容器)
 - 使用mTLS实现Pod对Pod加密

课程：

 - [Kubernetes CKS 2021---Open Policy Agent (OPA)](https://ghostwritten.blog.csdn.net/article/details/116904422)
 - [Kubernetes CKS 2021---Microservice Vulnerabilities-Manage secrets](https://ghostwritten.blog.csdn.net/article/details/116200609)
 - [Kubernetes CKS 2021---Microservice Vulnerabilities - Container Runtime Sandboxes](https://ghostwritten.blog.csdn.net/article/details/116230013)
 - [Kubernetes CKS 2021---Microservice Vulnerabilities - mTLS](https://ghostwritten.blog.csdn.net/article/details/116868179)

### 2.5 供应链安全：20%

 - 最小化基本镜像大小
 - 保护您的供应链：将允许的注册表列入白名单，对镜像进行签名和验证
 - 使用用户工作负载的静态分析(例如kubernetes资源，Docker文件)
 - 扫描镜像，找出已知的漏洞

课程：

 - [Kubernetes CKS 2021---Supply Chain Security - Image Footprint](https://ghostwritten.blog.csdn.net/article/details/117081884)
 - [Kubernetes CKS 2021---Supply Chain Security - Secure Supply Chain](https://ghostwritten.blog.csdn.net/article/details/117126383)
 - [Kubernetes CKS 2021---Supply Chain Security - Static Analysis](https://ghostwritten.blog.csdn.net/article/details/117111168)
 - [Kubernetes CKS 2021---Supply Chain Security - Image Vulnerability Scanning](https://ghostwritten.blog.csdn.net/article/details/117113680)
 

### 2.6 监控、日志记录和运行时安全：20%

 - 在主机和容器级别执行系统调用进程和文件活动的行为分析，以检测恶意活动
 - 检测物理基础架构，应用程序，网络，数据，用户和工作负载中的威胁
 - 检测攻击的所有阶段，无论它发生在哪里，如何扩散
 - 对环境中的不良行为者进行深入的分析调查和识别
 - 确保容器在运行时不变
 - 使用审计日志来监视访问

课程：

 - [Kubernetes CKS 2021--Runtime Security - Behavioral Analytics at host and container level](https://ghostwritten.blog.csdn.net/article/details/117220390)
 - [Kubernetes CKS 2021---Runtime Security - Immutability of containers at runtime](https://ghostwritten.blog.csdn.net/article/details/117224333)
 - [Kubernetes CKS 2021---Runtime Security - Auditing](https://ghostwritten.blog.csdn.net/article/details/117225475)


-----
## 3. 需要掌握内容
### 3.1 群集设置 10%
1.使用网络安全策略限制群集级别的访问

 - [使用网络策略控制流量](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
 - [保护Kubernetes集群安全](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/)
 - [声明网络策略以控制Pod的通信方式](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)
 - [强化集群网络策略](https://kubernetes.io/blog/2017/10/enforcing-network-policies-in-kubernetes/)

2.使用CIS基准来检查Kubernetes组件（etcd，kubelet，kubedns，kubeapi）的安全配置

 - [了解什么是Center for Internet Security（CIS）基准](https://docs.microsoft.com/en-us/compliance/regulatory/offering-CIS-Benchmark#:~:text=CIS%20benchmarks%20are%20configuration%20baselines,organizations%20improve%20their%20cyberdefense%20capabilities.)
 - [Kubernetes CIS Benchmark测试的工具：kube-bench的安装](https://github.com/aquasecurity/kube-bench#running-kube-bench)
 - [etcd和kubelet的CIS基准](https://cloud.google.com/kubernetes-engine/docs/concepts/cis-benchmarks#default-valueshttps://cloud.google.com/kubernetes-engine/docs/concepts/cis-benchmarks#default-values)
 - 用kube-bench检测master及worker上隐患配置

3.配置ingress的安全设置

 - [了解什么是Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
 - [什么是输入控制器（Ingress Controllers?）?](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
 - [在Minikube入口控制器上设置入口](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)
 - 创建自签名证书替换ingress自带的证书

4.保护节点元数据 

 - [限制通过API访问元数据](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/#restricting-cloud-metadata-api-access)
 - [在Kubernetes中设置安全端点](https://blog.cloud66.com/setting-up-secure-endpoints-in-kubernetes/)
 - [保护集群元数据(GKE)](https://cloud.google.com/kubernetes-engine/docs/how-to/protecting-cluster-metadata)
 - 通过配置文件设置 Kubelet 参数

5.最大限度地减少对dashboard的使用和访问

 - [基于web的Kubernetes用户界面](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
 - [设置Kubernetes dashboard的安全](https://blog.heptio.com/on-securing-the-kubernetes-dashboard-16b09b1b7aca?gi=ed673f28dbc8)

6.部署前验证kubernetes二进制文件

 - [通过sha512sum验证kubernetes二进制文件](https://github.com/kubernetes/kubernetes/releases)

### 3.2 群集强化 15%
1.限制对Kubernetes API的访问

 - [加强集群的安全性](https://cloud.google.com/anthos/gke/docs/on-prem/how-to/hardening-your-cluster)
 - 了解访问kubernetes api的流程
 - [控制对Kubernetes API的访问](https://kubernetes.io/docs/concepts/security/controlling-access/)

2.使用RBAC最大程度的减少资源暴露

 - [了解kubernetes api server的授权模块](https://kubernetes.io/docs/reference/access-authn-authz/authorization/#authorization-modules)
 - [使用RBAC授权](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
 - [理解RBAC授权](https://www.youtube.com/watch?v=G3R24JSlGjY)

3.SA的安全设置，例如禁用默认值，最小化对新创建sa的权限

 - [Kubernetes访问控制:探索服务帐户SA](https://thenewstack.io/kubernetes-access-control-exploring-service-accounts/)
 - [Kubernetes:创建服务帐户和Kubeconfigs](https://docs.armory.io/docs/spinnaker-install-admin-guides/manual-service-account/)
 - [置Pods的服务帐号SA](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
 - [在Kubernetes中部署时禁用默认服务帐户](https://stackoverflow.com/questions/52583497/how-to-disable-the-use-of-a-default-service-account-by-a-statefulset-deployments)
 - [Kubernetes不应该挂载默认的服务帐户](https://github.com/kubernetes/kubernetes/issues/57601)
 - [通过消除危险的权限来保护Kubernetes集群](https://www.cyberark.com/resources/threat-research-blog/securing-kubernetes-clusters-by-eliminating-risky-permissions)
 - 了解默认情况下SA带来的安全隐患及演示
 - 如何有效解决sa的权限问题

4.更新Kubernetes

 - [用kubeadm升级集群](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
 - [kubeadm升级](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)

### 3.3 系统强化 15%
1.服务器的安全设置

 - [最小化主机操作系统占用空间(减少攻击面)](https://blog.sonatype.com/kubesecops-kubernetes-security-practices-you-should-follow#:~:text=Reduce%20Kubernetes%20Attack%20Surfaces)
 - 去除系统不需要的内核模块

2.最小化IAM角色

 - [了解什么是最低特权原则（POLP）](https://digitalguardian.com/blog/what-principle-least-privilege-polp-best-practice-information-security-and-compliance)

3.最小化外部网络访问

 - [使用操作系统级防火墙保护主机](https://help.replicated.com/community/t/managing-firewalls-with-ufw-on-kubernetes/230)
 - [使用安全组来保护网络(Azure)](https://docs.microsoft.com/en-us/azure/aks/concepts-security#azure-network-security-groups)
 - [亚马逊重视安全组的考虑](https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html)
 - 使resourcequota及limitrange限制对资源的访问

4.适当使用内核强化工具，例如AppArmor，seccomp

 - [Kubernetes强化了最佳实践](https://www.sumologic.com/blog/kubernetes-security/#security-best-practices)
 - [使用AppArmor限制容器对资源的访问](https://kubernetes.io/docs/tutorials/clusters/apparmor/)
 - [使用Seccomp限制容器的syscall](https://kubernetes.io/docs/tutorials/clusters/seccomp/)

### 3.4 最小化微服务漏洞 20%
1.使用PSP，OPA，安全上下文提高安全性

 - [了解并配置Pod安全策略(PSP)](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)
 - [了解什么是 Open Policy Agent(OPA)](https://www.youtube.com/watch?v=Yup1FUc2Qn0&feature=youtu.be)
 - [OPA Gatekeeper的配置](https://kubernetes.io/blog/2019/08/06/opa-gatekeeper-policy-and-governance-for-kubernetes/)
 - [为Pod或容器配置安全上下文(securityContext)](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

2.管理Kubernetes secret

 - [使用secret存储敏感信息](https://kubernetes.io/docs/concepts/configuration/secret/)
 - [静态加密 Secret 数据](https://www.weave.works/blog/managing-secrets-in-kubernetes)

3.在多租户环境中使用沙箱运行容器（例如gvisor，kata容器）

 - 了解为什么要部署沙箱
 - [什么是gVisor？安装gvisor](https://gvisor.dev/docs/)
 - [使用谷歌的gVisor实现安全容器](https://thenewstack.io/how-to-implement-secure-containers-using-googles-gvisor/)
 - [使用gVisor运行Pod](https://gvisor.dev/docs/user_guide/quick_start/kubernetes/)
 - [Kata容器和Kubernetes:它们如何组合在一起?](https://platform9.com/blog/kata-containers-docker-and-kubernetes-how-they-all-fit-together/)
 - [如何使用Kata容器与Kubernetes?](https://github.com/kata-containers/documentation/blob/master/how-to/how-to-use-k8s-with-cri-containerd-and-kata.md)


4.使用mTLS实施Pod到Pod的加密

 - [了解什么是mTLS(mutual TLS)](https://codeburst.io/mutual-tls-authentication-mtls-de-mystified-11fa2a52e9cf?gi=9e3ace049450)
 - [使用mTLS进行流量加密](https://www.istioworkshop.io/11-security/01-mtls/)
 - [使用Istio提高端到端安全性](https://istio.io/latest/blog/2017/0.1-auth/)

### 3.5 供应链安全 20%
1.减小image的大小

 - [为什么要在Kubernetes中构建小容器映像](https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-how-and-why-to-build-small-container-images)
 - [如何创建比较小的镜像](https://cloud.google.com/solutions/best-practices-for-building-containers#build-the-smallest-image-possible)

2.保护供应链：将允许的镜像仓库列入白名单，对镜像进行签名和验证

 - [了解准入控制器Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)
 - [如何拒绝docker镜像仓库（黑名单）在Kubernetes?](https://stackoverflow.com/questions/54463125/how-to-reject-docker-registries-in-kubernetes)
 - [了解并配置ImagePolicyWebhook，确保只运行来自批准来源的映像](https://github.com/kubernetes/kubernetes/issues/22888)
 - [配置kubernetes所使用镜像仓库的白名单及黑名单](https://www.openpolicyagent.org/docs/latest/kubernetes-primer/)
 - [对镜像进行签名和验证](https://medium.com/sse-blog/container-image-signatures-in-kubernetes-19264ac5d8ce)

3.分析文件及镜像安全隐患（例如Kubernetes的yaml文件，Dockerfile）

 - [使用Kube-score进行静态分析](https://kube-score.com/)
 - [Kubernetes静态代码分析与Checkov](https://bridgecrew.io/blog/kubernetes-static-code-analysis-with-checkov/)
 - [使用Clair进行静态分析](https://github.com/quay/clair)
 
4.扫描图像，找出已知的漏洞
 - [扫描你的Docker镜像的漏洞](https://medium.com/better-programming/scan-your-docker-images-for-vulnerabilities-81d37ae32cb3)
 - [用Clair扫描Docker容器的漏洞](https://github.com/leahnp/clair-klar-kubernetes-demo)


### 3.6 监控、审计和runtime 20%
1.分析容器系统调用，以检测恶意进程

 - [如何使用Falco检测Kubernetes漏洞](https://sysdig.com/blog/how-to-detect-kubernetes-vulnerability-cve-2019-11246-using-falco/)
 - [用sysdig 对kubernetes进行安全监控](https://medium.com/@SkyscannerEng/kubernetes-security-monitoring-at-scale-with-sysdig-falco-a60cfdb0f67a)

2.检测物理基础设施、应用程序、网络、数据、用户和工作负载中的威胁

 - [Common Kubernetes配置安全威胁](https://www.cncf.io/blog/2020/08/07/common-kubernetes-config-security-threats/)
 - [Kubernetes威胁建模指南](https://www.trendmicro.com/vinfo/us/security/news/virtualization-and-cloud/guidance-on-kubernetes-threat-modeling)

3.检测攻击的所有阶段，无论它发生在哪里，如何传播

 - [在威胁堆栈中调查Kubernetes的攻击场景](https://www.threatstack.com/blog/kubernetes-attack-scenarios-part-1)
 - [Kubernetes攻击剖析——不可信的Docker镜像是如何让我们失望的](https://www.optiv.com/explore-optiv-insights/source-zero/anatomy-kubernetes-attack-how-untrusted-docker-images-fail-us)
 4.对环境中的不良行为进行深入的分析调查和识别
 [Kubernetes安全101:风险和最佳实践](https://www.stackrox.com/post/2020/05/kubernetes-security-101/)
5.确保容器在运行时的不变性
 - [利用Kubernetes确保容器是不可变的](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/container_security_guide/keeping_containers_fresh_and_updateable#leveraging_kubernetes_and_openshift_to_ensure_that_containers_are_immutable)
 - [为什么我们要使用不可变的Docker图像?](https://medium.com/sroze/why-i-think-we-should-all-use-immutable-docker-images-9f4fdcb5212f)
 - [有了不可变的基础设施，您的系统可以起死回生](https://techbeacon.com/enterprise-it/immutable-infrastructure-your-systems-can-rise-dead)

6.Kubernetes审计

 - [kubernetes审计](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)
 - [如何监控Kubernetes审计日志](https://www.datadoghq.com/blog/monitor-kubernetes-audit-logs/)
 - [分析Kubernetes审计日志](https://docs.sysdig.com/en/kubernetes-audit-logging.html)

## 4. 考试资料

 - [https://github.com/Evalle/CKS](https://github.com/Evalle/CKS)
 - [kubernetes-security书籍](https://www.stackrox.com/post/2020/04/enhancing-kubernetes-security-with-open-policy-agent-opa-part-1/)





相关阅读：

 - [CKA、CKAD考试经验](https://ghostwritten.blog.csdn.net/article/details/104956536)
 - [CKAD考试预备动员](https://ghostwritten.blog.csdn.net/article/details/108313252)
 - [ubuntu16.0安装kubernetes集群为练习CKA准备](https://ghostwritten.blog.csdn.net/article/details/107133651)
 - [CKA原英文考试2019年12月答案](https://ghostwritten.blog.csdn.net/article/details/107884669)


##  5. 考试命令

### NetworkPolicy
```bash
k run frontend --image=nginx
k run backend --image=nginx
k expose pod frontend --port 80
k expose pod backend --port 80
k get pods,svc
k exec frontend -- curl backend
k exec backend -- curl frontend

vim default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  - Ingress


vim frontend.yaml
# allows frontend pods to communicate with backend pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          run: backend


vim backend.yaml
# allows backend pods to have incoming traffic from frontend pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: frontend


k exec frontend -- curl 192.168.104.27
k exec backend -- curl 192.168.166.179 


kubectl create ns cassandra
kubectl edit ns cassandra
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-04-20T07:19:22Z"
  name: cassandra
  resourceVersion: "533198"
  uid: 766ae069-4dc9-4acd-a4db-ce852c293cc6
  labels:  #添加
    ns: cassandra #添加
spec:
  finalizers:
  - kubernetes
status:
  phase: Active


k  -n cassandra run cassandra --image=nginx
k -n cassandra get pod -owide
k exec backend -- curl 192.168.104.26
vim backend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            run: frontend
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            ns: cassandra

k exec backend -- curl 192.168.104.26

cat cassandra-deny.yaml
# deny all incoming and outgoing traffic from all pods in namespace cassandra
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cassandra-deny
  namespace: cassandra
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress

k exec backend -- curl 192.168.104.26
（通）

cat cassandra-deny.yaml
# deny all incoming and outgoing traffic from all pods in namespace cassandra
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cassandra-deny
  namespace: cassandra
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress

k exec backend -- curl 192.168.104.26  
(拒绝)

vim  cassandra.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cassandra
  namespace: cassandra
spec:
  podSelector:
    matchLabels:
      run: cassandra
  policyTypes:
    - Ingress
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            ns: default


k edit ns default
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-01-19T03:27:58Z"
  labels: #添加
    ns: default   #添加
  name: default
  resourceVersion: "541475"
  uid: 2d566715-f0a4-49b3-b590-dfa7df30d0ba
spec:
  finalizers:
  - kubernetes
status:
  phase: Active


k exec backend -- curl 192.168.104.26
```
###  Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.1.0/aio/deploy/recommended.yaml
namespace/kubernetes-dashboard created

k -n kubernetes-dashboard get pod,svc

k -n kubernetes-dashboard edit deploy kubernetes-dashboard
.....
      containers:
      - args:
        - --auto-generate-certificates
        - --namespace=kubernetes-dashboard
        image: kubernetesui/dashboard:v2.1.0
        imagePullPolicy: Always
......
改为
    spec:
      containers:
      - args:
        - --namespace=kubernetes-dashboard
        - --insecure-port=9090
        image: kubernetesui/dashboard:v2.1.0


k -n kubernetes-dashboard get pod,svc

k -n kubernetes-dashboard edit svc kubernetes-dashboard
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":"kubernetes-dashboard"},"spec":{"ports":[{"port":443,"targetPort":8443}],"selector":{"k8s-app":"kubernetes-dashboard"}}}
  creationTimestamp: "2021-04-21T02:55:03Z"
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
  resourceVersion: "557996"
  uid: bd515d85-4dc6-4ac0-9890-ca2a711a7b26
spec:
  clusterIP: 10.99.150.161
  clusterIPs:
  - 10.99.150.161
  ports:
  - port: 9090             #443改为9090
    protocol: TCP
    targetPort: 9090        #8443改为9090
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort           #ClusterIP改为NodePort
status:
  loadBalancer: {}

k -n kubernetes-dashboard get svc

#RBAC for the Dashboard

k -n kubernetes-dashboard get sa
k get clusterroles  |grep view
k -n kubernets-dashboard create rolebinding insecure --serviceaccount kubernetes-dashboard:kubernetes-dashboard --clusterrole view 

k -n kubernetes-dashboard create clusterrolebinding insecure --serviceaccount kubernetes-dashboard:kubernetes-dashboard --clusterrole view


```
###  Secure Ingress

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.40.2/deploy/static/provider/baremetal/deploy.yaml


k get pod,svc -n ingress-nginx

cat secure-ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80


k create -f secure-ingress.yaml 
k get ing
k run pod1 --image=nginx
k run pod2 --image=httpd
k expose pod pod1 --port 80 --name service1
k expose pod pod2 --port 80 --name service2
curl  http://192.168.211.40:31459/service1
curl  http://192.168.211.40:31459/service2


curl  https://192.168.211.40:32300/service1 -kv
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
k create secret tls secure-ingress --cert=cert.pem --key=key.pem
k get sec
k get secret

vim secure-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
      - secure-ingress.com
    secretName: secure-ingress
  rules:
  - host: secure-ingress.com  
    http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80

      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80

 k apply -f secure-ingress.yaml 
 curl  https://secure-ingress.com:32300/service2 -kv --resolv secure-ingress.com:32300:192.168.211.41
```

###  Node Metadata

```clike
curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"
curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/0/" -H "Metadata-Flavor: Google"
k run nginx --image=nginx
k get pods
k exec -ti nginx bash
curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"
cat deny.yaml
# all pods in namespace cannot access metadata endpoint
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32

 k create -f deny.yaml 
 k exec -ti nginx bash
 curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"         ## 卡住

cat allow.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-allow
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: metadata-accessor
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 169.254.169.254/32


k create -f allow.yaml 
k label pod nginx role=metadata-accessor
k get pods nginx --show-labels
k exec -ti nginx bash
curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"  #正常访问


k edit pod nginx
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.104.31/32
  creationTimestamp: "2021-04-22T03:17:45Z"
  labels:
    role: metadata-accessor   #删除
    run: nginx
  name: nginx
  namespace: default


 k exec -ti nginx bash
 curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"  #卡住无法访问

```
###   CIS Benchmarks

```clike
kubectl get nodes
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest master --version 1.20

useradd etcd
chown etcd:etcd /var/lib/etcd
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest master --version 1.20

docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest node --version 1.20


./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml master
kube-bench --config-dir /data/software/kube-bench/cfg --config /data/software/kube-bench/cfg/config.yaml node
```
###  Verify

```csharp
sha512sum kubernetes-server-linux-arm64.tar.gz  > compare
sha512sum kubernetes/server/bin/kube-apiserver
k -n kube-system get pod | grep api
k -n kube-system get pod kube-apiserver-master -o yaml | grep image
docker cp 0fb5321dfd57:/ container-fs
find container-fs/ | grep kube-apiserver
sha512sum container-fs/usr/local/bin/kube-apiserver
```

###  Restrict API Access

```csharp
curl https://localhost:6443
curl https://localhost:6443 -k
vim /etc/kubernetes/manifests/kube-apiserver.yaml
...
    - kube-apiserver
    - --advertise-address=192.168.211.40
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=8080  #0改成8080
  .....

curl http://localhost:8080
curl https://192.168.211.40:6443 --cacert ca --cert  ca.crt --key ca.key

```

###  ServiceAccounts

```bash
k get sa,secrets
k describe sa default
k create sa accessor
k describe secret accessor-token-bnd4s
k run accessor --image=nginx --dry-run=client -oyaml > accessor.yaml
cat accessor.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: accessor
  name: accessor
spec:
  serviceAccountName: accessor  #添加此行
  containers:
  - image: nginx
    name: accessor
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


k create -f accessor.yaml
k exec -ti accessor -- bash
mount |grep sec
cd /run/secrets/kubernetes.io/serviceaccount
cat token 
curl https://kubernetes
curl https://kubernetes -k


cat accessor.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: accessor
  name: accessor
spec:
  serviceAccountName: accessor
  automountServiceAccountToken: false   #添加此行
  containers:
  - image: nginx
    name: accessor
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

k -f accessor.yaml replace --force
k exec -ti accessor -- bash
mount |grep ser
k get pod
k auth can-i delete secrets --as system:serviceaccount:default:accessor
k create clusterrolebinding accessor --clusterrole edit --serviceaccount default:accessor
k auth can-i delete secrets --as system:serviceaccount:default:accessor
```
### RBAC

```bash
k create ns red
k create ns blue
k -n red create role secret-manager -verb=get --resource=secrets -oyaml --dry-run=client
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: secret-manager
  namespace: red
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get


k -n red create rolebinding secret-manager --role=secret-manager --user=jane

k -n red auth can-i get secrets --as jane
k -n red auth can-i get secrets --as tom
k -n red auth can-i delete secrets --as jane
k -n red auth can-i list secrets --as jane
k -n blue auth can-i list secrets --as jane
k -n blue auth can-i get secrets --as jane
k -n blue auth can-i get pods --as jane

k create clusterrole deploy-deleter --verb delete --resource deployments

k create clusterrolebinding deploy-deleter --user jane --clusterrole deploy-deleter

k -n red create rolebinding deploy-deleter  --user jim --clusterrole deploy-deleter
k auth can-i delete deployments --as jane
k auth can-i delete deployments --as jane -n default
k auth can-i delete deployments --as jane -n red
k auth can-i delete pods --as jane -n red
k auth can-i delete deployments --as jim -n default
k auth can-i delete deployments --as jim -A
k auth can-i delete deployments --as jim -n red
```
### Upgrade Kubernetes

```bash
k drain master --ignore-daemonsets
k get nodes
apt-cache show kubeadm |grep 1.20
apt-get install kubeadm=1.20.2-00 kubectl=1.20.2-00 kubelet=1.20.2-00
kubeadm upgrade plan
kubeadm upgrade apply v1.20.6
k get nodes

k drain node1 --ignore-daemonsets
k uncordon node1 
kubeadm version
apt-cache show kubeadm  |grep -e '1.20'
apt-get install kubeadm=1.20.2-00 kubectl=1.20.2-00 kubelet=1.20.2-00
kubeadm version
kubectl version
kubelet version
k uncordon node1 

```

###  securityContext与podsecurityPolicies

```bash
k run pod --image=busybox --command -oyaml --dry-run=client > pod.yaml -- sh -c 'sleep 1d'
cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

k exec -ti pod -- sh
/ $ id
uid=1000 gid=3000

/ $ touch test
touch: test: Permission denied
/ $ cd /tmp
/tmp $ touch test
/tmp $ ls -lh
total 0      
-rw-r--r--    1 1000     3000           0 May 15 15:00 test

```
###  seccomp and apparmor

```bash
$ cat /etc/apparmor.d/docker-nginx
$ apparmor_parser /etc/apparmor.d/docker-nginx 
$ aa-status 
$ docker run nginx
$ docker run --security-opt apparmor=docker-default nginx
$ docker run --security-opt apparmor=docker-nginx nginx
/docker-entrypoint.sh: 13: /docker-entrypoint.sh: cannot create /dev/null: Permission denied
/docker-entrypoint.sh: No files found in /docker-entrypoint.d/, skipping configuration

$ docker run --security-opt apparmor=docker-nginx -d  nginx
$ docker exec -ti f608a4a126e2e2b145dcf094b41c29bea1f7b8beeb38871178e0ea0ae8eab061 bash
$ touch /root/test
touch: cannot touch '/root/test': Permission denied
$ sh
bash: /bin/sh: Permission denied
$ touch /test


$ apparmor_parser /etc/apparmor.d/docker-nginx 
$ aa-status 
$ k run secure --image=nginx -oyaml --dry-run=client > pod.yaml
$ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  annotations:    #添加此行
    container.apparmor.security.beta.kubernetes.io/secure: localhost/hello  #添加此行
  labels:
    run: secure
  name: secure
spec:
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

$ k create  -f pod.yaml 
$ k get pods secure
NAME     READY   STATUS    RESTARTS   AGE
secure   0/1     Blocked   0          6s
$ k describe pod secure
nnotations:  container.apparmor.security.beta.kubernetes.io/secure: localhost/hello
Status:       Pending
Reason:       AppArmor
Message:      Cannot enforce AppArmor: profile "hello" is not loaded



$ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  annotations: 
    container.apparmor.security.beta.kubernetes.io/secure: localhost/docker-nginx  #修改此行
  labels:
    run: secure
  name: secure
spec:
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


$ k create -f pod.yaml 
$ k get pod secure
NAME     READY   STATUS    RESTARTS   AGE
secure   1/1     Running   0          10s

```

参考：

 - [Everything you need to know about the CKS Kubernetes Security Specialist certification (except the answers)](https://bargenqua.st/posts/cks/)

