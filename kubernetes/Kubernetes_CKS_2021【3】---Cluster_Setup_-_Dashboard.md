#  Kubernetes  访问 Dashboard
tags: Dashboard
<!--  catalog: ~Dashboard~ -->




![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0d9e544aaf871293a53b54cc36197092.jpeg#pic_center)



## 1. 简介
[Dashboard](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/) 是基于网页的 Kubernetes 用户界面。 你可以使用 Dashboard 将容器应用部署到 Kubernetes 集群中，也可以对容器应用排错，还能管理集群资源。 你可以使用 Dashboard 获取运行在集群中的应用的概览信息，也可以创建或者修改 Kubernetes 资源 （如 Deployment，Job，DaemonSet 等等）。 例如，你可以对 Deployment 实现弹性伸缩、发起滚动升级、重启 Pod 或者使用向导创建新的应用。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6c0a6ec8662e99f8dfaf5baab47c6697.png)

##  2. 默认 dashboard 权限

 - `get`,`update`以及命名空间中的`delete Secrets`的权限。`kubernetes-dashboard-key-holder`、`kubernetes-dashboard-certs`、`kubernetes-dashboard-csrf`、`kubernetes-dashboard`
 - `get`和命名空间中命名update的配置映射的权限。`kubernetes-dashboard-settings`、`kubernetes-dashboard`
 - `get` 权限`services/proxy`，以便允许收集指标所需的命名空间中的`heapster`服务`dashboard-metrics-scraper`。kubernetes-dashboard
 - `get`，`list`以及API 的`watch`权限，`metrics.k8s.io`以便允许从metrics-server访问`dashboard-metrics-scraper`.


##  3. 验证
Kubernetes Dashboard 支持几种不同的用户身份验证方式：

 - [授权标头（Authorization header ）](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/README.md#authorization-header)在每个请求中传递给仪表板。从 1.6 版开始支持。具有最高优先级。如果存在，将跳过登录视图。
 - 可在仪表板登录视图上使用的不记名令牌。
 - 可在仪表板登录视图中使用的用户名/密码。
 - 可用于 Dashboard登录视图的Kubeconfig文件。

###  3.1 登录视图
如果您使用的是推荐的最新安装，则默认情况下将启用登录功能。`--tls-cert-file`在任何其他情况下，如果您更喜欢手动配置证书，则需要将`--tls-cert-key`标志传递给 `Dashboard`。`HTTPS` 端点将暴露在Dashboard 容器的端口上`8443`。您可以通过提供`--port`标志来更改它。
使用`Skip`选项将使 `Dashboard` 使用 Dashboard 使用的 `Service Account` 的权限。Skip自 1.10.1 起，按钮默认禁用。使用--enable-skip-login仪表板标志来显示它。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21f92fd1719698bbe39173ee16610863.png)
###  3.2 Authorization header
通过 HTTP 访问 Dashboard 时，使用授权标头是使 Dashboard 充当用户的唯一方法。请注意，由于[普通 HTTP 流量容易受到MITM 攻击](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)，因此存在一些风险。

要使 Dashboard 使用授权标头，您只需将`Authorization: Bearer <token>`每个请求传递给 Dashboard。这可以通过在仪表板前面配置反向代理来实现。代理将负责与身份提供者进行身份验证，并将请求标头中生成的令牌传递给仪表板。请注意，Kubernetes API 服务器需要正确配置才能接受这些令牌。

要快速测试它，请查看允许手动修改请求标头的[Requestly Chrome 浏览器插件](https://chrome.google.com/webstore/detail/redirect-url-modify-heade/mdnleldcmiljblolnjhpnblkcekpdkpa)。

 **重要提示**：如果通过 API 服务器代理访问 Dashboard，授权标头将不起作用。访问仪表板指南中描述的访问仪表`kubectl proxy`板和访问仪表板的方式都将不起作用。这是因为一旦请求到达 API 服务器，所有额外的标头都会被丢弃。

### 3.3 Bearer Token
建议先熟悉[Kubernetes认证文档](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)，了解如何获取token，可以用来登录。例如，每个服务帐户都有一个带有有效承载令牌的秘密，可用于登录仪表板。

如何创建服务帐户并授予其权限：

 - [Service Account Tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens)
 - [Role and ClusterRole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole)
 - [Service Account Permissions](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#service-account-permissions)

###  3.4 默认
默认情况下禁用基本身份验证。原因是 Kubernetes API 服务器需要配置授权模式 `ABAC` 和`--basic-auth-file`提供的标志。如果没有该 API 服务器会自动回退到[匿名用户](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests)，并且无法检查提供的凭据是否有效。

为了在仪表板-`-authentication-mode=basic`标志中启用基本身份验证，必须提供。默认情况下，它设置为--`authentication-mode=token`。

> 注意：基本身份验证`--basic-auth-file`自 `Kubernetes v1.19`起已被弃用。对于要--basic-auth-file标记的类似功能，请`--token-auth-file` 与[Static Token File](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-token-file)一起使用。

###  3.5 Kubeconfig
提供这种登录方法是为了方便。  kubeconfig 文件中仅支持 flag 指定`--authentication-mode`的身份验证选项。如果它被配置为使用任何其他方式，错误将显示在仪表板中。目前不支持外部身份提供者或基于证书的身份验证。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b781d93d0dab3a1ae97a91a9fb0f46c2.png)

##  4. 管理员权限
您可以通过在下面创建来授予 Dashboard 的服务帐户完整的管理员权限`ClusterRoleBinding`。根据选择的安装方法复制 YAML 文件并另存为，即`dashboard-admin.yaml`. 用于`kubectl create -f dashboard-admin.yaml`部署它。之后，您可以使用Skip登录页面上的选项来访问仪表板。
**正式发布**

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
```
**开发版本**

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-head
  namespace: kubernetes-dashboard-head
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard-head
    namespace: kubernetes-dashboard-head
```

## 5. Dashboard 参数
参数列表：

| Argument name               | Default value      | Description                                                                                                                                                                                                                                                                                               |
|-----------------------------|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| insecure-port	              | 9090               | The port to listen to for incoming HTTP requests.                                                                                                                                                                                                                                                         |
| port                        | 8443               | The secure port to listen to for incoming HTTPS requests.                                                                                                                                                                                                                                                 |
| insecure-bind-address       | 127.0.0.1          | The IP address on which to serve the `--insecure-port` (set to 127.0.0.1 for loopback only).                                                                                                                                                                                                              |
| bind-address                | 0.0.0.0            | The IP address on which to serve the `--port` (set to 0.0.0.0 for all interfaces).                                                                                                                                                                                                                        |
| default-cert-dir            | /certs             | Directory path containing `--tls-cert-file` and `--tls-key-file` files. Used also when auto-generating certificates flag is set. Relative to the container, not the host.                                                                                                                                 |
| tls-cert-file               | -                  | File containing the default x509 Certificate for HTTPS.                                                                                                                                                                                                                                                   |
| tls-key-file                | -                  | File containing the default x509 private key matching --tls-cert-file.                                                                                                                                                                                                                                    |
| auto-generate-certificates  | false              | When set to true, Dashboard will automatically generate certificates used to serve HTTPS.                                                                                                                                                                                                                 |
| apiserver-host              | -                  | The address of the Kubernetes Apiserver to connect to in the format of protocol://address:port, e.g., http://localhost:8080. If not specified, the assumption is that the binary runs inside a Kubernetes cluster and local discovery is attempted.                                                       |
| api-log-level               | INFO               | Level of API request logging. Should be one of `INFO\|NONE\|DEBUG`. |
| heapster-host               | -                  | The address of the Heapster Apiserver to connect to in the format of protocol://address:port, e.g., http://localhost:8082. If not specified, the assumption is that the binary runs inside a Kubernetes cluster and service proxy will be used.                                                           |
| sidecar-host                | -                  | The address of the Sidecar Apiserver to connect to in the format of protocol://address:port, e.g., http://localhost:8000. If not specified, the assumption is that the binary runs inside a Kubernetes cluster and service proxy will be used.                                                            |
| metrics-provider            | sidecar            | Select provider type for metrics. 'none' will not check metrics.                                                                                                                                                                                                                                          |
| metric-client-check-period  | 30                 | Time in seconds that defines how often configured metric client health check should be run.                                                                                                                                                                                                               |
| kubeconfig                  | -                  | Path to kubeconfig file with authorization and master location information.                                                                                                                                                                                                                               |
| namespace                   | kube-system        | When non-default namespace is used, create encryption key in the specified namespace.                                                                                                                                                                                                                     |
| token-ttl                   | 900                | Expiration time (in seconds) of JWE tokens generated by dashboard. '0' never expires.                                                                                                                                                                                                                     |
| authentication-mode         | token              | Enables authentication options that will be reflected on the login screen in the same order as provided. Multiple options can be used at once. Supported values: token, basic. Note that basic option should only be used if apiserver has '--authorization-mode=ABAC' and '--basic-auth-file' flags set. |
| enable-insecure-login       | false              | When enabled, Dashboard login view will also be shown when Dashboard is not served over HTTPS.                                                                                                                                                                                                            |
| enable-skip-login           | false              | When enabled, the skip button on the login page will be shown.                                                                                                                                                                                                                                            |
| disable-settings-authorizer | false              | When enabled, Dashboard settings page will not require user to be logged in and authorized to access settings page.                                                                                                                                                                                       |
| locale-config               | ./locale_conf.json | File containing the configuration of locales.                                                                                                                                                                                                                                                             |
| system-banner               | -                  | When non-empty displays message to Dashboard users. Accepts simple HTML tags.                                                                                                                                                                                                                             |
| system-banner-severity      | INFO               | Severity of system banner. Should be one of `INFO\|WARNING\|ERROR`. |



## 6 访问
###  6.1 NodePort 访问
[https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)

```C

$  wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.1.0/aio/deploy/recommended.yaml

$ vim recommended.yaml
//跳转到40行左右，修改其对应的service，类型配置为Nodeport
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort    //添加类型为NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 31010      //映射到宿主机的端口为31010
  selector:
    k8s-app: kubernetes-dashboard

$ kubectl apply -f recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created


$ k get pod,svc  -n kubernetes-dashboard 
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-79c5968bdc-zq4tc   1/1     Running   0          117s
pod/kubernetes-dashboard-7448ffc97b-rwnmp        1/1     Running   0          117s

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
service/dashboard-metrics-scraper   ClusterIP   10.100.200.128   <none>        8000/TCP        117s
service/kubernetes-dashboard        NodePort    10.96.43.126     <none>        443:31010/TCP   117s

```



访问:`http://192.168.211.40:31010/`

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/289eeb50864120c6b390fda463bad705.png)


```bash
//创建dashboard的管理用户
$ kubectl create serviceaccount dashboard-admin -n kube-system
$ kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

$ kubectl get secrets -n kube-system | grep dashboard-admin
dashboard-admin-token-lx8th                      kubernetes.io/service-account-token   3      33s

$ kubectl describe secrets dashboard-admin-token-lx8th -n kube-system
Name:         dashboard-admin-token-lx8th
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: d6a78702-0099-47bc-949c-946f246403a0

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjNROWFvUGhIWk9sYzBHT3JvOUJDb2wtX29Tc1Z0blRVUmxpeEhmaUpIUFEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tbHg4dGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZDZhNzg3MDItMDA5OS00N2JjLTk0OWMtOTQ2ZjI0NjQwM2EwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.g4B9UQki-7PLBwNsNiE7ppPb8uu0tiAqNBoThREYY4MRoYkxc7Fu9_CtbJZaOjD02iGa5EacsKbG7vmVyrNT_iB1PzOC2ueU5K1QV9M41MftSCw4CBHCgISso3tQDYkTfaRNnWl6weFZqJz3vAQnNX-iOhC7_8KaHzY0qKIahZFvV4j_9-s-a2cxObFp8x_UdKS5sFrZeiu5vz93R8frzuL_3A-UsB1x_k9Ptb9kQieQBV_YCO5qvcdKb3wQUnFYQyZkt05v5yIlLouhNNeQFFJN1f_ycdmoT5yzg2cSM_j1Ls6H92N8SY7R0gG_A14ZQc2hFvxlBRc7F67TIfaWbQ


或者执行
$ kubectl get secret -n kube-system  $(kubectl get serviceaccount  dashboard-admin   -n kube-system -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
```
获取 token：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cad945da67e8b0c7e30323f99b170871.png)


访问 
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/381484bec2a113c3ea692286cfa7edd6.png)

###  6.2 kubeconfig 访问

```bash
//查看刚才创建的token
$ kubectl get secrets -n kube-system | grep dashboard
dashboard-admin-token-lx8th                      kubernetes.io/service-account-token   3      13m

//查看token的详细信息，会获取token
$ kubectl describe secrets -n kube-system dashboard-admin-token-lx8th
Name:         dashboard-admin-token-lx8th
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: d6a78702-0099-47bc-949c-946f246403a0

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjNROWFvUGhIWk9sYzBHT3JvOUJDb2wtX29Tc1Z0blRVUmxpeEhmaUpIUFEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tbHg4dGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZDZhNzg3MDItMDA5OS00N2JjLTk0OWMtOTQ2ZjI0NjQwM2EwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.g4B9UQki-7PLBwNsNiE7ppPb8uu0tiAqNBoThREYY4MRoYkxc7Fu9_CtbJZaOjD02iGa5EacsKbG7vmVyrNT_iB1PzOC2ueU5K1QV9M41MftSCw4CBHCgISso3tQDYkTfaRNnWl6weFZqJz3vAQnNX-iOhC7_8KaHzY0qKIahZFvV4j_9-s-a2cxObFp8x_UdKS5sFrZeiu5vz93R8frzuL_3A-UsB1x_k9Ptb9kQieQBV_YCO5qvcdKb3wQUnFYQyZkt05v5yIlLouhNNeQFFJN1f_ycdmoT5yzg2cSM_j1Ls6H92N8SY7R0gG_A14ZQc2hFvxlBRc7F67TIfaWbQ
ca.crt:     1066 bytes


//将token的信息生成一个变量
$ DASH_TOKEN=$(kubectl get secrets -n kube-system dashboard-admin-token-lx8th -o jsonpath={.data.token} | base64 -d)


//将k8s集群的配置信息写入到一个文件中，文件可自定义
kubectl config set-cluster kubernetes --server=192.168.211.40:6443 --kubeconfig=/root/.dashboard-admin.conf

$ cat /root/.dashboard-admin.conf
apiVersion: v1
clusters:
- cluster:
    server: 192.168.211.40:6443
  name: kubernetes
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null

$ //将token的信息也写入到文件中（同一个文件）
$ kubectl config set-credentials dashboard-admin --token=${DASH_TOKEN} --kubeconfig=/root/.dashboard-admin.conf
User "dashboard-admin" set.

$ cat /root/.dashboard-admin.conf
apiVersion: v1
clusters:
- cluster:
    server: 192.168.211.40:6443
  name: kubernetes
contexts: null
current-context: ""
kind: Config
preferences: {}
users:
- name: dashboard-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IjNROWFvUGhIWk9sYzBHT3JvOUJDb2wtX29Tc1Z0blRVUmxpeEhmaUpIUFEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tbHg4dGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZDZhNzg3MDItMDA5OS00N2JjLTk0OWMtOTQ2ZjI0NjQwM2EwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.g4B9UQki-7PLBwNsNiE7ppPb8uu0tiAqNBoThREYY4MRoYkxc7Fu9_CtbJZaOjD02iGa5EacsKbG7vmVyrNT_iB1PzOC2ueU5K1QV9M41MftSCw4CBHCgISso3tQDYkTfaRNnWl6weFZqJz3vAQnNX-iOhC7_8KaHzY0qKIahZFvV4j_9-s-a2cxObFp8x_UdKS5sFrZeiu5vz93R8frzuL_3A-UsB1x_k9Ptb9kQieQBV_YCO5qvcdKb3wQUnFYQyZkt05v5yIlLouhNNeQFFJN1f_ycdmoT5yzg2cSM_j1Ls6H92N8SY7R0gG_A14ZQc2hFvxlBRc7F67TIfaWbQ


//用户信息也写入文件中（同一个文件）
$ kubectl config set-context dashboard-admin@kubernetes --cluster=kubernetes --user=dashboard-admin --kubeconfig=/root/.dashboard-admin.conf

$ cat /root/.dashboard-admin.conf 
apiVersion: v1
clusters:
- cluster:
    server: 192.168.211.40:6443
  name: kubernetes
contexts:
- context:                          #添加
    cluster: kubernetes             #添加
    user: dashboard-admin           #添加
  name: dashboard-admin@kubernetes  #添加
current-context: ""
kind: Config
preferences: {}
users:
- name: dashboard-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IjNROWFvUGhIWk9sYzBHT3JvOUJDb2wtX29Tc1Z0blRVUmxpeEhmaUpIUFEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tbHg4dGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZDZhNzg3MDItMDA5OS00N2JjLTk0OWMtOTQ2ZjI0NjQwM2EwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.g4B9UQki-7PLBwNsNiE7ppPb8uu0tiAqNBoThREYY4MRoYkxc7Fu9_CtbJZaOjD02iGa5EacsKbG7vmVyrNT_iB1PzOC2ueU5K1QV9M41MftSCw4CBHCgISso3tQDYkTfaRNnWl6weFZqJz3vAQnNX-iOhC7_8KaHzY0qKIahZFvV4j_9-s-a2cxObFp8x_UdKS5sFrZeiu5vz93R8frzuL_3A-UsB1x_k9Ptb9kQieQBV_YCO5qvcdKb3wQUnFYQyZkt05v5yIlLouhNNeQFFJN1f_ycdmoT5yzg2cSM_j1Ls6H92N8SY7R0gG_A14ZQc2hFvxlBRc7F67TIfaWbQ


//将上下文的配置信息也写入文件中（同一个文件）
$ kubectl config use-context dashboard-admin@kubernetes --kubeconfig=/root/.dashboard-admin.conf

$ cat /root/.dashboard-admin.conf 
apiVersion: v1
clusters:
- cluster:
    server: 192.168.211.40:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: dashboard-admin
  name: dashboard-admin@kubernetes
current-context: dashboard-admin@kubernetes  #添加dashboard-admin@kubernetes
kind: Config
preferences: {}
users:
- name: dashboard-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IjNROWFvUGhIWk9sYzBHT3JvOUJDb2wtX29Tc1Z0blRVUmxpeEhmaUpIUFEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tbHg4dGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZDZhNzg3MDItMDA5OS00N2JjLTk0OWMtOTQ2ZjI0NjQwM2EwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.g4B9UQki-7PLBwNsNiE7ppPb8uu0tiAqNBoThREYY4MRoYkxc7Fu9_CtbJZaOjD02iGa5EacsKbG7vmVyrNT_iB1PzOC2ueU5K1QV9M41MftSCw4CBHCgISso3tQDYkTfaRNnWl6weFZqJz3vAQnNX-iOhC7_8KaHzY0qKIahZFvV4j_9-s-a2cxObFp8x_UdKS5sFrZeiu5vz93R8frzuL_3A-UsB1x_k9Ptb9kQieQBV_YCO5qvcdKb3wQUnFYQyZkt05v5yIlLouhNNeQFFJN1f_ycdmoT5yzg2cSM_j1Ls6H92N8SY7R0gG_A14ZQc2hFvxlBRc7F67TIfaWbQ
//最后将配置信息导入到客户端本地
$ sz /root/.dashboard-admin.conf
```
访问：`https://192.168.211.40:31010/`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2fed2706644f9ef691cff2204ee07a97.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e10f6e2aa3b21e5e05cfbdc7e5dcba7c.png)


###  6.3 proxy 访问
出于[安全原因](https://www.containiq.com/post/kubernetes-security-best-practices)，推荐的配置为 Dashboard ServiceAccount 提供对 Kubernetes 资源的有限访问权限。这可以防止[secrets](https://smoothies.com.cn/kubernetes-docs/%E5%AF%B9%E8%B1%A1/kubernetes-Secrets.html)或证书等敏感集群数据被意外暴露。

也就是说，要利用所有 Web UI 功能，您需要一个具有管理（超级用户）权限的服务帐户，即具有cluster -admin集群角色的服务帐户。

使用您喜欢的文本编辑器，创建一个`admin-user.yml`文件：

```bash
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

```

```bash
kubectl apply -f admin-user.yml
```

Accessing the Kubernetes Dashboard

访问 Kubernetes Dashboard 的推荐方式是通过 Bearer Tokens。这为管理权限提供了极大的灵活性。

要获取 ServiceAccount `admin-user`的 `Bearer Token`，请运行以下命令。

```bash
kubectl get secret -n kubernetes-dashboard $(kubectl get serviceaccount admin-user -n kubernetes-dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
```
输出：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99440ea73ce22a471880ae1ab0151ed1.png)
赋值保存，执行公开 Kubernetes Dashboard 命令：

```bash
kubectl proxy
```
如果您已按照本教程中描述的过程进行操作，终端中将显示一条消息，指示 WebUI 在`127.0.0.1:8001`可用。出于安全原因，仪表板只能从运行`kubectl proxy`命令的机器访问。请记住这一点，特别是如果 Kubernetes 安装在远程服务器上。

代理激活后，您可以访问 Web UI，将您喜欢的浏览器指向以下地址：
[http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4166dfc8b62452bbbe04af6a7f0da17c.png)




参考：

 - [部署和访问 Kubernetes 仪表板（Dashboard）](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/)
 - [https://github.com/kubernetes/dashboard](https://github.com/kubernetes/dashboard)
 - [Tutorial: Deploy the Kubernetes Dashboard (web UI)](https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html)
 - [Kubernetes Dashboard | Installation, Tips, and Examples](https://www.containiq.com/post/intro-to-kubernetes-dashboards)


