#  Kubernetes Admission Webhook Validating 与 mutating 实践
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d261b0f7293afabd681ad010299791d1.png)




## 1. k8s 的配置
启用 MutatingAdmissionWebhook 和 ValidatingAdmissionWebhook
MutatingAdmissionWebhook 和 ValidatingAdmissionWebhook 默认是不启用的，apiserver 想调用 webhook，还得 enable 相关的能力

```bash
[root@master ~]# kubectl get pods kube-apiserver-master.lihao04.virtual -n kube-system -o yaml|grep enable
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
  enableServiceLinks: true
```

因为 enable-admission-plugins 缺失 feature，我们要 enable

```bash
# 修改 /etc/kubernetes/manifests/kube-apiserver.yaml
- --enable-admission-plugins=NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook
修改配置文件后立刻生效
```

```bash
[root@master manifests]# kubectl get pods kube-apiserver-master.lihao04.virtual -n kube-system -o yaml|grep enable
    - --enable-admission-plugins=NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook
    - --enable-bootstrap-token-auth=true
  enableServiceLinks: true
```

```bash
# 确实重启过
[root@master manifests]# kubectl get pod -n kube-system|grep api
kube-apiserver-master.lihao04.virtual            1/1     Running            0          54s
```



## 2. 构建
建项目文件夹：

```bash
$ mkdir admission-webhook && cd admission-webhook
# 创建go项目代码目录，设置当前目录为GOPATH路径
$ mkdir src && export GOPATH=$pwd
$ mkdir -p src/github.com/cnych/ && cd src/github.com/cnych/
```

进入到上面的src/github.com/cnych/目录下面，将项目代码 clone 下面：

```bash
$ git clone https://github.com/cnych/admission-webhook-example.git
```

我们可以看到代码根目录下面有一个build的脚本，只需要提供我们自己的 docker 镜像用户名然后直接构建即可：
注意：192.168.211.41:5000是我的镜像仓库名，方便其他节点拉取镜像

```bash
#!/bin/bash

export DOCKER_USER=192.168.211.41:5000
export GO111MODULE=on 
export GOPROXY=https://goproxy.cn
# build webhook
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o admission-webhook-example 
# build docker image
docker build --no-cache -t ${DOCKER_USER}/admission-webhook-example:v1 .
rm -rf admission-webhook-example

docker push ${DOCKER_USER}/admission-webhook-example:v1

```
```bash
$ ./build
```
## 3. 部署服务
为了部署 webhook server，我们需要在我们的 Kubernetes 集群中创建一个 service 和 deployment 资源对象，部署是非常简单的，只是需要配置下服务的 TLS 配置。我们可以在代码根目录下面的 deployment 文件夹下面查看deployment.yaml文件中关于证书的配置声明，会发现从命令行参数中读取的证书和私钥文件是通过一个 secret 对象挂载进来的：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: admission-webhook-example-deployment
  labels:
    app: admission-webhook-example
spec:
  replicas: 1
  selector:
    matchLabels:
      app: admission-webhook-example
  template:
    metadata:
      labels:
        app: admission-webhook-example
    spec:
      serviceAccount: admission-webhook-example-sa
      containers:
        - name: admission-webhook-example
          image: 192.168.211.41:5000/admission-webhook-example:v1
#          imagePullPolicy: Always
          args:
            - -tlsCertFile=/etc/webhook/certs/cert.pem
            - -tlsKeyFile=/etc/webhook/certs/key.pem
            - -alsologtostderr
            - -v=4
            - 2>&1
          volumeMounts:
            - name: webhook-certs
              mountPath: /etc/webhook/certs
              readOnly: true
      volumes:
        - name: webhook-certs
          secret:
            secretName: admission-webhook-example-certs

```
在生产环境中，对于 TLS 证书（特别是私钥）的处理是非常重要的，我们可以使用类似于cert-manager之类的工具来自动处理 TLS 证书，或者将私钥密钥存储在Vault中，而不是直接存在 secret 资源对象中。

我们可以使用任何类型的证书，但是需要注意的是我们这里设置的 CA 证书是需要让 apiserver 能够验证的，我们这里可以重用 Istio 项目中的生成的[证书签名请求脚本](https://github.com/istio/istio/blob/release-0.7/install/kubernetes/webhook-create-signed-cert.sh)。通过发送请求到 apiserver，获取认证信息，然后使用获得的结果来创建需要的 secret 对象。

首先，运行[该脚本](https://github.com/cnych/admission-webhook-example/blob/blog/deployment/webhook-create-signed-cert.sh)检查 secret 对象中是否有证书和私钥信息：

```bash
$ ./deployment/webhook-create-signed-cert.sh
creating certs in tmpdir /var/folders/x3/wjy_1z155pdf8jg_jgpmf6kc0000gn/T/tmp.IboFfX97 
Generating RSA private key, 2048 bit long modulus (2 primes)
..................+++++
........+++++
e is 65537 (0x010001)
certificatesigningrequest.certificates.k8s.io/admission-webhook-example-svc.default created
NAME                                    AGE   REQUESTOR          CONDITION
admission-webhook-example-svc.default   1s    kubernetes-admin   Pending
certificatesigningrequest.certificates.k8s.io/admission-webhook-example-svc.default approved
secret/admission-webhook-example-certs created

$ kubectl get secret admission-webhook-example-certs
NAME                              TYPE     DATA   AGE
admission-webhook-example-certs   Opaque   2      28s
```
一旦 secret 对象创建成功，我们就可以直接创建 deployment 和 service 对象。

```bash
$ kubectl create serviceaccount admission-webhook-example-sa
$ kubectl create -f deployment/deployment.yaml
deployment.apps "admission-webhook-example-deployment" created

$ kubectl create -f deployment/service.yaml
service "admission-webhook-example-svc" created
```
在我们的 webhook 服务运行起来了，它可以接收来自 apiserver 的请求。但是我们还需要在 kubernetes 上创建一些配置资源。首先来配置 `validating` 这个 webhook，查看 webhook 配置，我们会注意到它里面包含一个`CA_BUNDLE`的占位符：

```bash
clientConfig:
  service:
	name: admission-webhook-example-svc
	namespace: default
	path: "/validate"
  caBundle: ${CA_BUNDLE}
```
CA 证书应提供给 admission webhook 配置，这样 apiserver 才可以信任 webhook server 提供的 TLS 证书。因为我们上面已经使用 Kubernetes API 签署了证书，所以我们可以使用我们的 kubeconfig 中的 CA 证书来简化操作。代码仓库中也提供了一个小脚本`webhook-patch-ca-bundle.sh`用来替换 CA_BUNDLE 这个占位符，

```bash
#!/bin/bash

ROOT=$(cd $(dirname $0)/../../; pwd)

set -o errexit
set -o nounset
set -o pipefail


export CA_BUNDLE=$(kubectl config view --raw --flatten -o json | jq -r '.clusters[] | .cluster."certificate-authority-data"')

if command -v envsubst >/dev/null 2>&1; then
    envsubst
else
    sed -e "s|\${CA_BUNDLE}|${CA_BUNDLE}|g"
fi
```

创建 validating webhook 之前运行该命令即可：

```bash
$ cat ./deployment/validatingwebhook.yaml | ./deployment/webhook-patch-ca-bundle.sh > ./deployment/validatingwebhook-ca-bundle.yaml

$ cat deployment/validatingwebhook-ca-bundle.yaml 
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingWebhookConfiguration
metadata:
  name: validation-webhook-example-cfg
  labels:
    app: admission-webhook-example
webhooks:
  - name: required-labels.qikqiak.com
    clientConfig:
      service:
        name: admission-webhook-example-svc
        namespace: default
        path: "/validate"
      caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1ERXdOekE0TXpJd05Wb1hEVE14TURFd05UQTRNekl3TlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTWtVClIybHV4NnVaSWRnQURPdmFvb2YwRkpsT0VkK3JOSHZ2U3hZQ3NaL3FzcGQrN2NFS0Q4TTNRNERQWmJoc2hlR2cKUWg3bEVSL2V3Mm5kL1pFSXIwZEpzakhxMjlwMkpaSzZmZ1hvYWsyV29QVWEwYTRleU50L09TOGpCQUVZdmtrVQpVVlJraVNkbk95WGpZSXgvdzJuOEdFdUZSU3ZNeWx2WExLbkppWkVTRDhTRDA0eE51aUxIRndEUG9FZEEyNGlQCnV6UXdnRjFlTFNYdnNxM1ZrZDNDM1hGL2JKUitWVW9LQjAycGhYc1o4WDRXWkp1T2ZVaHRrOXErbnUxckhhR2IKUjYyQTVoVHlhWHZkblV6T0Z4THg5Yll6MHIwdVJxakRJZ3hKZUNoMGlscFhUYUxJNmVSS29tREViSEkwTlBhTAp2QVJHMERvNTEvTmFXbzhPVFZFQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZOTGswU0hiNjBleU9VVEwzZkVPQ2E1Njd5TkZNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBcSt2c3F5WldYeFZTell3SWlqU0k0Qm1IelpCRWQ0ZGM1Sm1TbDlHK3hCelRwUmJoRQpTZTNwQ3FOeGRYWk9ZZHNnZjFic1VsVFcxMm9HbmhtdVlDTFIwYmcyWDI0YWViam96OGRvNlR2WW1STWRTeTFtCkpnRGlRRDRzdXBYUkQvOVVLdHkrY0FQY3hLdDlWSm9lNE9OeWFEcGNXaWV3UEVZSUNxRWRGVklacnYvT1BZQWQKblJOQzgweVdtZk5jNFlOSFdIZG5SZ0xrS2c1eTBEMVpkR1BMOEFUcnFsVk1hNXd2d3RoZ1V6MHRWN2VFbE1xdgpjNWNid0N2QnI0U291RkgzaHcyR0NvQ0pRNkUzazkycTQ4andmNlJNWGVIMElNMSs4SjltNS9UOVl2Tk1GN1c1Cjc4MnJYZy9Pa1oyY0VETWQ0Ym1GZjFzQUt2MGNiek50SFZuMgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    rules:
      - operations: [ "CREATE" ]
        apiGroups: ["apps", ""]
        apiVersions: ["v1"]
        resources: ["deployments","services"]
    namespaceSelector:
      matchLabels:
        admission-webhook-example: enabled

```
执行完成后可以查看`validatingwebhook-ca-bundle.yaml`文件中的`CA_BUNDLE`占位符的值是否已经被替换掉了。需要注意的是 clientConfig 里面的 path 路径是`/validate`，因为我们代码在是将 validate 和 mutate 集成在一个服务中的。
然后就是需要配置一些 RBAC 规则，我们想在 `deployment` 或 `service` 创建时拦截 API 请求，所以`apiGroups`和`apiVersions`对应的值分别为`apps/v1`对应 `deployment`，`v1`对应 `service`。对于 RBAC 的配置方法可以查看我们前面的文章:[Kubernetes RBAC 详解](https://www.qikqiak.com/post/use-rbac-in-k8s/)

webhook 的最后一部分是配置一个namespaceSelector，我们可以为 webhook 工作的命名空间定义一个 selector，这个配置不是必须的，比如我们这里添加了下面的配置：

```bash
namespaceSelector:
  matchLabels:
	admission-webhook-example: enabled
```
则我们的 webhook 会只适用于设置了admission-webhook-example=enabled标签的 namespace， 您可以在Kubernetes参考文档中查看此资源配置的完整布局。

所以，首先需要在default这个 namespace 中添加该标签：

```bash
$ kubectl label namespace default admission-webhook-example=enabled
namespace "default" labeled
```
最后，创建这个 validating webhook 配置对象，这会动态地将 webhook 添加到 webhook 链上，所以一旦创建资源，就会拦截请求然后调用我们的 webhook 服务：

```bash
$ kubectl create -f deployment/validatingwebhook-ca-bundle.yaml
validatingwebhookconfiguration.admissionregistration.k8s.io "validation-webhook-example-cfg" created
```
## 4. 测试Validating webhook
现在让我们创建一个 deployment 资源来验证下是否有效，代码仓库下有一个sleep.yaml的资源清单文件，直接创建即可：

```bash
$ kubectl create -f sleep.yaml 
Error from server (required labels are not set): error when creating "sleep.yaml": admission webhook "required-labels.qikqiak.com" denied the request: required labels are not set
```
正常情况下创建的时候会出现上面的错误信息，然后部署另外一个`sleep-with-labels.yaml`的资源清单：

```bash
$ kubectl create -f deployment/sleep-with-labels.yaml
deployment.apps "sleep" created
```

可以看到可以正常部署，先我们将上面的 deployment 删除，然后部署另外一个sleep-no-validation.yaml资源清单，该清单中不存在所需的标签，但是配置了`admission-webhook-example.qikqiak.com/validate=false`这样的 annotation，正常也是可以正常创建的：

```bash
$ kubectl delete deployment sleep
$ kubectl create -f deployment/sleep-no-validation.yaml
deployment.apps "sleep" created
```
## 5. 测试 mutating webhook
首先，我们将上面的 validating webhook 删除，防止对 mutating 产生干扰，然后部署新的配置。 `mutating webhook` 与 `validating webhook` 配置基本相同，但是 webook server 的路径是/mutate，同样的我们也需要先填充上CA_BUNDLE这个占位符。

```bash
$ kubectl delete validatingwebhookconfiguration validation-webhook-example-cfg
validatingwebhookconfiguration.admissionregistration.k8s.io "validation-webhook-example-cfg" deleted

$ cat ./deployment/mutatingwebhook.yaml | ./deployment/webhook-patch-ca-bundle.sh > ./deployment/mutatingwebhook-ca-bundle.yaml

$ kubectl create -f deployment/mutatingwebhook-ca-bundle.yaml
mutatingwebhookconfiguration.admissionregistration.k8s.io "mutating-webhook-example-cfg" created
```
现在我们可以再次部署上面的sleep应用程序，然后查看是否正确添加 label 标签：

```bash
$ kubectl create -f deployment/sleep.yaml
deployment.apps "sleep" created

$ kubectl get  deploy sleep -o yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    admission-webhook-example.qikqiak.com/status: mutated
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: 2018-09-24T11:35:50Z
  generation: 1
  labels:
    app.kubernetes.io/component: not_available
    app.kubernetes.io/instance: not_available
    app.kubernetes.io/managed-by: not_available
    app.kubernetes.io/name: not_available
    app.kubernetes.io/part-of: not_available
    app.kubernetes.io/version: not_available
...
```

最后，我们重新创建 validating webhook，来一起测试。现在，尝试再次创建 sleep 应用。正常是可以创建成功的，我们可以查看下[admission-controllers 的文档](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-are-they)

> 准入控制分两个阶段进行，第一阶段，运行 mutating admission 控制器，第二阶段运行 validating admission控制器。

所以 mutating webhook 在第一阶段添加上缺失的 labels 标签，然后 `validating webhook` 在第二阶段就不会拒绝这个 deployment 了，因为标签已经存在了，用`not_available`设置他们的值。

```bash
$ kubectl create -f deployment/validatingwebhook-ca-bundle.yaml
validatingwebhookconfiguration.admissionregistration.k8s.io "validation-webhook-example-cfg" created

$ kubectl create -f deployment/sleep.yaml
deployment.apps "sleep" created
```
参考：

 - [https://banzaicloud.com/blog/k8s-admission-webhooks/](https://banzaicloud.com/blog/k8s-admission-webhooks/)
 - [https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-are-they](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#what-are-they)



相关阅读：

 - [k8s准入控制器【1】--介绍](https://ghostwritten.blog.csdn.net/article/details/112242163)
 - [k8s准入控制器【2】--动态准入控制](https://ghostwritten.blog.csdn.net/article/details/112258669)
 - [k8s 准入控制器【3】--编写和部署准入控制器Webhook--根据标签才可创建pod](https://ghostwritten.blog.csdn.net/article/details/112280103)
 - [k8s 准入控制器【4】--编写和部署准入控制器 Webhook--以非root运行pod](https://ghostwritten.blog.csdn.net/article/details/112335665)
 - [kubeadm部署安装k8sv1.18.1详解](https://ghostwritten.blog.csdn.net/article/details/105567076)
 - [go语言入门安装与hello world](https://ghostwritten.blog.csdn.net/article/details/104502695)

