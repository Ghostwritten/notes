


**相关阅读**：

 1. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【1】动手实践感受区别所在](https://ghostwritten.blog.csdn.net/article/details/119712220)
 2. [KubernetesImagePolicyWebhook与ValidatingAdmissionWebhook【2】Image_Policy.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119811128)
 3. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【3】validating_admission.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119979444)
 4. [Kubernetes ImagePolicyWebhook与ValidatingAdmissionWebhook【4】main.go源码解析](https://ghostwritten.blog.csdn.net/article/details/119978143)
 5. [kubernetes 快速学习手册](https://ghostwritten.blog.csdn.net/article/details/108562082)

----
## 1. 介绍
**准入控制器**（Admission Control）在授权后对请求做进一步的验证或添加默认参数。不同于授权和认证只关心请求的用户和操作，准入控制还处理请求的内容，并且仅对创建、更新、删除或连接（如代理）等有效，而对读操作无效。



##  2. 对比：ImagePolicyWebhook 和 ValidatingAdmissionWebhook
[ImagePolicyWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook)是准入控制器，仅评估镜像，你需要解析请求做逻辑和以允许或集群中否认镜像的响应。

ImagePolicyWebhook优势：

 - 如果无法连接Webhook端点，可以指示API服务器拒绝镜像，这非常方便，但是它也会带来问题，例如核心Pod无法调度。

ImagePolicyWebhook劣势：

 - 配置涉及更多内容，并且需要访问主节点或apiserver配置，文档尚不明确，可能难以进行更改，更新等。
 - 部署并不是那么简单，你需要使用systemd进行部署或将其作为docker容器在主机中运行，更新dns等

[ValidatingAdmissionWebhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)使用 Webhook 验证请求，这些 Webhook 并行调用，并且任何一个调用拒绝都会导致请求失败 。
ValidatingAdmissionWebhook优势：

 - 由于该服务作为Pod运行，因此部署更容易。
 - 一切都可以成为kubernetes资源。
 - 需要较少的人工干预和访问主机。
 - 如果pod或服务不可用，则将允许所有镜像，这在某些情况下会带来安全风险，因此，如果要使用此路径，请确保使其高度可用，这实际上可以通过指定failurePolicytoFail来配置的Ignore（Fail是默认设置）。

ValidatingAdmissionWebhook劣势：
 - **具有RBAC权限的任何人都可以更新/更改配置，因为它只是kubernetes的一个资源**。




##  3. ImagePolicyWebhook配置说明
ImagePolicyWebhook: 通过 webhook 决定 image 策略，需要同时配置 `–admission-control-config-file`



```bash
imagePolicy:
  kubeConfigFile: /path/to/kubeconfig/for/backend
  # time in s to cache approval
  allowTTL: 50
  # time in s to cache denial
  denyTTL: 50
  # time in ms to wait between retries
  retryBackoff: 500
  # determines behavior if the webhook backend fails
  defaultAllow: true
```
如果不手动去实践这两个准入控制的玩法，我们根本无法真正体会领悟到`ImagePolicyWebhook` 或 `ValidatingAdmissionWebhook`的运用到底是有怎么样的区别，各自又都适合什么样的场景处置。

那么跟我一起操作吧。

## 4. 示例
`ImagePolicyWebhook` 或 `ValidatingAdmissionWebhook`实现准入控制器将拒绝所有使用带有latest标签的镜像的Pod。


### 4.1  下载介质
构建
如果你打算将其用作普通服务：
```bash
 $ go get github.com/kainlite/kube-image-bouncer
 $ tree -L 2
.
├── Dockerfile
├── Gopkg.lock
├── Gopkg.toml
├── handlers
│   ├── image_policy.go
│   └── validating_admission.go
├── kubernetes
│   ├── image-bouncer-webhook.yaml
│   ├── nginx-latest.yml
│   ├── nginx-versioned.yml
│   └── validating-webhook-configuration.yaml
├── LICENSE
├── main.go
├── README.md
├── rules
│   ├── from_whitelisted_registry.go
│   ├── from_whitelisted_registry_test.go
│   ├── not_latest.go
│   └── not_latest_test.go
└── vendor
    ├── github.com
    ├── golang.org
    ├── gopkg.in
    └── k8s.io

```
下载webhook镜像

```c
$ docker pull kainlite/kube-image-bouncer
```
### 4.2 下载安装证书管理工具cfssl
webhook 端点必须由 tls 保护以供 kubernetes 使用。该证书也可以是自签名证书。
如果你想了解更多信息，我们可以依靠kubernetes CA生成我们需要的证书。具体可以参考 [管理群集中的TLS证书](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/) 。

下载[cfssl与cfssljson地址](https://github.com/cloudflare/cfssl/releases)

```bash
wget wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssl_1.6.0_linux_amd64

chmod 755 cfssl_1.6.0_linux_amd64
mv cfssl_1.6.0_linux_amd64 /usr/local/bin/cfssl
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.0/cfssljson_1.6.0_linux_amd64
chmod 755 cfssljson_1.6.0_linux_amd64
mv cfssljson_1.6.0_linux_amd64 /usr/local/bin/cfssljson
```


### 4.3 创建证书签名请求
即，通过运行以下命令生成私钥和证书签名请求（或 CSR）:

```bash
$ cat <<EOF | cfssl  genkey - | cfssljson -bare server
{
  "hosts": [
    "image-bouncer-webhook.default.svc",
    "image-bouncer-webhook.default.svc.cluster.local",
    "image-bouncer-webhook.default.pod.cluster.local",
    "192.168.211.40",
    "172.22.0.20"
  ],
  "CN": "system:node:image-bouncer-webhook.default.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
  "names": [
    {
      "O": "system:nodes"
    }
  ]
}
EOF
```
```bash
#output
2021/08/15 03:49:12 [INFO] generate received request
2021/08/15 03:49:12 [INFO] received CSR
2021/08/15 03:49:12 [INFO] generating key: ecdsa-256
2021/08/15 03:49:12 [INFO] encoded CSR

$ ls
server.csr  server-key.pem
```
其中 `192.168.211.40` 是服务的集群 IP，`image-bouncer-webhook.svc.cluster.local` 是服务的 DNS 名称，`172.22.0.20` 是 Pod 的 IP，而 `image-bouncer-webhook.pod.cluster.local` 是 Pod 的 DNS 名称。 

此命令生成两个文件：**它生成包含 PEM 编码 pkcs#10 证书请求的 server.csr， 以及 PEM 编码密钥的 server-key.pem，用于待生成的证书**

### 4.4 创建证书签名请求对象发送到 Kubernetes API
使用以下命令创建 CSR YAML 文件，并发送到 API 服务器：

```bash
$ cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: image-bouncer-webhook.default
spec:
  request: $(cat server.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kubelet-serving
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
```
在上个步骤 中创建csr生成的 `server.csr` 文件是 `base64 编码`并存储在 `.spec.request` 字段中的。我们还要求提供 “`digital signature`（数字签名）”， “`密钥加密`（key encipherment）” 和 “`服务器身份验证`（server auth）” 密钥用途， 由 `kubernetes.io/kubelet-serving` 签名程序签名的证书。 你也可以要求使用特定的 `signerName`。更多信息可参阅 [支持的签署者名称](https://kubernetes.io/zh/docs/reference/access-authn-authz/certificate-signing-requests/#signers)。

在 API server 中可以看到这些 CSR 处于 Pending 状态
```bash
$ k get csr
NAME                            AGE   SIGNERNAME                      REQUESTOR          CONDITION
image-bouncer-webhook.default   10s   kubernetes.io/kubelet-serving   kubernetes-admin   Pending
```
### 4.5 批准证书签名请求
批准证书签名请求是通过自动批准过程完成的，或由集群管理员一次性完成。 有关这方面涉及的更多信息，请参见下文。

如果一直处于pending状态，可以手动批准证书签名请求

```bash
$ kubectl get csr|grep 'Pending'| awk 'NR>0{print $1}' |xargs kubectl certificate approve
certificatesigningrequest.certificates.k8s.io/image-bouncer-webhook.default approved

$ kubectl  get csr
NAME                            AGE   SIGNERNAME                      REQUESTOR          CONDITION
image-bouncer-webhook.default   45s   kubernetes.io/kubelet-serving   kubernetes-admin   Approved,Issued

```




### 4.6 下载证书

```bash
$ kubectl get csr image-bouncer-webhook.default -o jsonpath='{.status.certificate}' | base64 --decode > server.crt
```
###  4.7 ImagePolicyWebhook 部署
有两种方法来部署webhook控制器，要使其正常工作，你将需要按照以下说明创建证书。但是首先我们需要注意一个细节，将此证书添加到主服务器中的主机文件中运行：
我们使用的名称必须与证书中的名称匹配，因为它可以在kuberntes外部运行，甚至可以在外部使用，我们只是使用主机条目来伪造它

```bash
$ echo "127.0.0.1 image-bouncer-webhook.default.svc" >> /etc/hosts
```
另外，在apiserver中，你需要在`/etc/kubernetes/manifests/kube-apiserver.yaml`使用以下设置进行更新：

 - 如果你的api-server组件是本地运行，配置如下：

```bash
.....
--admission-control-config-file=/etc/kubernetes/kube-image-bouncer/admission_configuration.json
--enable-admission-plugins=ImagePolicyWebhook
.....
```

 - 如果你的api-server组件是pod运行，配置如下：

```bash
.....
--admission-control-config-file=/etc/kubernetes/kube-image-bouncer/admission_configuration.json
--enable-admission-plugins=ImagePolicyWebhook
......
    - mountPath: /etc/kubernetes/kube-image-bouncer  #添加此行
      name: k8s-admission    #添加此行
      readOnly: true    #添加此行
.......
  - hostPath:    #添加此行
      path: /etc/kubernetes/kube-image-bouncer   #添加此行
      type: DirectoryOrCreate    #添加此行
    name: k8s-admission    #添加此行
```
api-server组件会自动重启，生效修改后的配置文件


创建一个名为`/etc/kubernetes/kube-image-bouncer/admission_configuration.json`准入控制配置文件，其内容如下：

```bash
{
  "imagePolicy": {
     "kubeConfigFile": "/etc/kubernetes/kube-image-bouncer/kube-image-bouncer.yml",
     "allowTTL": 50,
     "denyTTL": 50,
     "retryBackoff": 500,
     "defaultAllow": false
  }
}
```
或者
创建一个名为`/etc/kubernetes/kube-image-bouncer/admission_configuration.yaml`格式

```bash
imagePolicy:
  kubeConfigFile: /etc/kubernetes/kube-image-bouncer/kube-image-bouncer.yml
  # time in s to cache approval
  allowTTL: 50
  # time in s to cache denial
  denyTTL: 50
  # time in ms to wait between retries
  retryBackoff: 500
  # determines behavior if the webhook backend fails
  defaultAllow: false
```
如果想要允许默认镜像，请调整`defaultAllow`为`true`。

创建具有以下内容的kubeconfig文件`/etc/kubernetes/kube-image-bouncer/kube-image-bouncer.yml`：

```bash
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/kube-image-bouncer/pki/server.crt
    server: https://image-bouncer-webhook.default.svc:1323/image_policy
  name: bouncer_webhook
contexts:
- context:
    cluster: bouncer_webhook
    user: api-server
  name: bouncer_validator
current-context: bouncer_validator
preferences: {}
users:
- name: api-server
  user:
    client-certificate: /etc/kubernetes/pki/apiserver.crt
    client-key:  /etc/kubernetes/pki/apiserver.key
```
此配置文件指示API服务器连接地址为`https://image-bouncer-webhook.default.svc:1323`的Webhook服务器并使用其`/image_policy`端点，我们可以重用apiserver的证书以及已经生成的kube-image-bouncer证书。

请注意，你需要与证书放在同一个文件夹中，这样才能起作用：

运行webhook
```bash
$ pwd
/etc/kubernetes/kube-image-bouncer/pki
$  ls -l
total 12
-rwxr-xr-x 1 root root 1155 Aug 15 06:43 server.crt
-rwxr-xr-x 1 root root  704 Aug 15 06:43 server.csr
-rwxr-xr-x 1 root root  227 Aug 15 06:43 server-key.pem

$ docker run --rm -v `pwd`/server-key.pem:/certs/server-key.pem:ro -v `pwd`/server.crt:/certs/server.crt:ro -p 1323:1323 --network host kainlite/kube-image-bouncer -k /certs/server-key.pem -c /certs/server.crt

WARNING: Published ports are discarded when using host network mode
WARN: accepting images from ALL registries

   ____    __
  / __/___/ /  ___
 / _// __/ _ \/ _ \
/___/\__/_//_/\___/ v3.3.5
High performance, minimalist Go web framework
https://echo.labstack.com
____________________________________O/_______
                                    O\
⇨ https server started on [::]:1323

```

测试创建一个带有`latest`标签的pod

```bash
$ cat nginx-latest.yaml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-latest
spec:
  replicas: 1
  selector:
    app: nginx-latest
  template:
    metadata:
      name: nginx-latest
      labels:
        app: nginx-latest
    spec:
      containers:
      - name: nginx-latest
        image: nginx:latest
        ports:
        - containerPort: 80
```

```bash
$ kubectl apply -f nginx-latest.yaml 
replicationcontroller/nginx-latest created
$ kubectl  get pods -A |grep nginx #没有运行的pod
$ kubectl describe rc nginx-latest  #警告不允许有latest标签的镜像
  Warning  FailedCreate  8s (x6 over 87s)  replication-controller  (combined from similar events): Error creating: pods "nginx-latest-f7667" is forbidden: image policy webhook backend denied one or more images: Images using latest tag are not allowed
```


测试创建一个不带有latest标签的pod

```bash
$ cat nginx-versioned.yaml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-versioned
spec:
  replicas: 1
  selector:
    app: nginx-versioned
  template:
    metadata:
      name: nginx-versioned
      labels:
        app: nginx-versioned
    spec:
      containers:
      - name: nginx-versioned
        image: nginx:1.13.8
        ports:
        - containerPort: 80
```

```bash
$ kubectl apply -f nginx-versioned.yaml 
$ kubectl  get rc
NAME              DESIRED   CURRENT   READY   AGE
nginx-latest      1         0         0       6m46s
nginx-versioned   1         1         1       4s
$ kubectl  get pods -n default |grep nginx
nginx-versioned-sqndl   1/1     Running   0          88s
```
带确定版本的镜像创建pod成功。

以上我们完成了利用`ImagePolicyWebhook`准入控制的方法，对镜像的标签规则进行了访问控制。其中，有几个部署关键点需要我们掌握：

 1. 创建证书签名请求
 2. 创建证书签名请求对象发送到 Kubernetes API
 3. 批准证书签名请求
 4. 下载证书为webhook访问api-server使用
 5. ImagePolicyWebhook在api-server的配置
 6. ImagePolicyWebhook与api-server通过证书实现的访问控制的实现
 7. webhoook的部署
 
 当然，如何编写webhook我会放到后面详细介绍。

下面我们用`ValidatingAdmissionWebhook`准入控制实现拒绝带有`latest`镜像标签的任务的访问。

在开始之前，我们要删除关于`ImagePolicyWebook`的资源，以及相关配置。
```bash
kubectl delete -f nginx-latest.yaml
kubectl delete -f nginx-versioned.yaml
rm -rf /etc/kubernetes/kube-image-bouncer/
```
`apiserver`改回原来的配置

```bash
--enable-admission-plugins=NodeRestriction
```
r
###  4.8 ValidatingAdmissionWebhook 部署
修改配置`/etc/kubernetes/manifests/kube-apiserver.yaml`，启动`ValidatingAdmissionWebhook`

```bash
......
- --enable-admission-plugins=NodeRestriction,ValidatingAdmissionWebhook
......
```
确认修改后已重启。

如果你要使用`ValidatingAdmissionWebhook` ，我们实现与API的访问，当然也需要证书，因为`ValidatingAdmissionWebhook`是kubernetes的资源对象，那么证书的存储与使用方式当然kubernetes识别的资源对象为首要条件，那么就是`secret`了，首先你必须创建一个名为tls的secret文件(用来保存webhook证书和密钥) 。
```bash
$ pwd
/etc/kubernetes/kube-image-bouncer/pki

$ ls
server.crt  server.csr  server-key.pem

$ kubectl create secret tls tls-image-bouncer-webhook --key server-key.pem --cert server.crt
secret/tls-image-bouncer-webhook created
```
然后为创建一个名为`image-bouncer-webhook的 deployment` 文件：

```bash
apiVersion: v1
kind: Service
metadata:
  labels:
    app: image-bouncer-webhook
  name: image-bouncer-webhook
spec:
  ports:
    - name: https
      port: 443
      targetPort: 1323
      protocol: "TCP"
  selector:
    app: image-bouncer-webhook
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: image-bouncer-webhook
spec:
  selector:
    matchLabels:
      app: image-bouncer-webhook
  template:
    metadata:
      labels:
        app: image-bouncer-webhook
    spec:
      containers:
        - name: image-bouncer-webhook
          imagePullPolicy: Always
          image: "kainlite/kube-image-bouncer:latest"
          args:
            - "--cert=/etc/admission-controller/tls/tls.crt"
            - "--key=/etc/admission-controller/tls/tls.key"
            - "--debug"
          volumeMounts:
            - name: tls
              mountPath: /etc/admission-controller/tls
      volumes:
        - name: tls
          secret:
            secretName: tls-image-bouncer-webhook
```
注意: 我们要遵循`image-bouncer-webhook.default`的域名规则方式。


```bash
$ kubectl apply -f kubernetes/image-bouncer-webhook.yaml
service/image-bouncer-webhook created
deployment.apps/image-bouncer-webhook created

kubectl  get pods
NAME                                     READY   STATUS    RESTARTS   AGE
image-bouncer-webhook-57f5ff98f4-7rdwv   1/1     Running   0          43s

```
最后创建`ValidatingWebhookConfiguration`，确保使用我们的webhook端点 ：

```bash
$ cat <<EOF | kubectl apply -f -
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: image-bouncer-webook
webhooks:
  - name: image-bouncer-webhook.default.svc
    rules:
      - apiGroups:
          - ""
        apiVersions:
          - v1
        operations:
          - CREATE
        resources:
          - pods
    failurePolicy: Ignore
    sideEffects: None
    admissionReviewVersions: ["v1", "v1beta1"]
    clientConfig:
      caBundle: $(kubectl get csr image-bouncer-webhook.default -o jsonpath='{.status.certificate}')
      service:
        name: image-bouncer-webhook
        namespace: default
EOF
```



更改可能需要一点时间才能体现出来，因此请等待几秒钟，然后尝试一下。推出 helm chart后，这将实现自动化。

测试创建一个带latest标签的pod
```bash
$ kubectl apply -f nginx-latest.yaml 
replicationcontroller/nginx-latest created
$ kubectl  get pods -A |grep nginx #没有运行的pod
$ kubectl describe rc nginx-latest  #警告不允许有latest标签的镜像
.....
Warning  FailedCreate  11s (x14 over 53s)  replication-controller  Error creating: admission webhook "image-bouncer-webhook.default.svc" denied the request: Images using latest tag are not allowed
```
测试创建一个不带有latest标签的pod
```bash
$ kubectl apply -f nginx-versioned.yaml 
$ kubectl  get rc
NAME              DESIRED   CURRENT   READY   AGE
nginx-latest      1         0         0       6m46s
nginx-versioned   1         1         1       4s
$ kubectl  get pods -n default |grep nginx
nginx-versioned-sqndl   1/1     Running   0          88s
```


以上我们完成了利用`ValidatingWebhookConfiguration`准入控制的方法，对镜像的标签规则进行了访问控制。其中，有几个部署关键点需要我们掌握：
 1. 创建证书签名请求
 2. 创建证书签名请求对象发送到 Kubernetes API
 3. 批准证书签名请求
 4. 创建tls的secrets
 5. 通过deployment部署webhook
 6. 创建ValidatingWebhookConfiguration对象，实现api对ValidatingWebhookConfiguration的访问


代码详细分析，请看下回。敬请期待。

参考链接：
[https://techsquad.rocks/blog/kubernetes_image_policy_webhook_explained/](https://techsquad.rocks/blog/kubernetes_image_policy_webhook_explained/)
