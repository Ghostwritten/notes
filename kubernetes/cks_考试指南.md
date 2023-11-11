视频：[Kubernetes 安全专家(CKS)](https://space.bilibili.com/400114617/channel/collectiondetail?sid=666460)


![在这里插入图片描述](https://img-blog.csdnimg.cn/b240f3186993451a88a7437dcb1d22fb.png#pic_center)


##  1. CIS benchmarking

 - [https://www.cisecurity.org/benchmark/kubernetes](https://www.cisecurity.org/benchmark/kubernetes)
 - [https://github.com/aquasecurity/kube-bench](https://github.com/aquasecurity/kube-bench)

### 1.1 本地安装

 - [https://github.com/aquasecurity/kube-bench/blob/main/docs/installation.md](https://github.com/aquasecurity/kube-bench/blob/main/docs/installation.md)

```bash
curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.6.9/kube-bench_0.6.9_linux_amd64.deb -o kube-bench_0.6.9_linux_amd64.deb

apt install ./kube-bench_0.6.9_linux_amd64.deb -f

kube-bench
```
###  1.2 kubernetes 安装

master
```bash
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-master.yaml
```
node:

```bash
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job-node.yaml
```

```bash
kubectl get job
kubectl get po
kubectl log xxxx
```

##  2. ingress-nginx with TLS
![在这里插入图片描述](https://img-blog.csdnimg.cn/2d53388372ad4785a4bc637fb3377a9d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/38e2dced251d4d48b4e2ebaeac350f8f.png)

### 2.1 create svc & pod
```bash
k get no
k run web --image caddy
k expose po web --name caddy-svc --port 80

```
### 2.2 create ingress resource
caddy-ingress.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-ing
spec:
  rules:
    - host: learnwithgvr.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: caddy-svc
                port:
                  number: 80

```

```bash
k apply -f caddy-ingress.yaml
curl learnwithgvr.com
```
### 2.3 create TLS certificate key-pair

```bash
 openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls.key -out tls.crt -subj "/CN=learnwithgvr.com" -days 365 

```
create secret with TLS data

```bash
k create secret tls sec-learnwithgvr --cert=tls.crt --key=tls.key
```

### 2.4 update ingress with TLS

caddy-ingress.yaml 
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: simple-ing
spec:
  tls:
  - hosts:
      - learnwithgvr.com
    secretName: sec-learnwithgvr
  rules:
    - host: learnwithgvr.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: caddy-svc
                port:
                  number: 80 

```
执行：

```bash
k apply -f caddy-ingress.yaml
k describe ing
```
###  2.5 verify/test
```bash
curl --cacert tls.crt https://learnwithgvr.com
```




## 3. network policy
![在这里插入图片描述](https://img-blog.csdnimg.cn/9323e1d8caeb4f68b307bc47ce4189a8.png)

 - 限制控制平面端口（6443、2379、2380、10250、10251、10252）
 - 限制工作节点端口（10250、30000-32767）
 - 对于云，使用 Kubernetes 网络策略来限制 pod 对云元数据的访问

示例假设 AWS 云，元数据 IP 地址为 169.254.169.254 应被阻止，而所有其他外部地址则不被阻止。

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-only-cloud-metadata-access
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  egress:
  - from:
    - ipBlock:
      cidr: 0.0.0.0/0
      except:
      - 169.254.169.254/32
```


##  4. Minimize use of, and access to, GUI elements

```bash
#steps to create serviceaccount & use
kubectl create serviceaccount simple-user -n kube-system
kubectl create clusterrole simple-reader --verb=get,list,watch --resource=pods,deployments,services,configmaps
kubectl create clusterrolebinding cluster-simple-reader --clusterrole=simple-reader --serviceaccount=kube-system:simple-user

SEC_NAME=$(kubectl get serviceAccount simple-user -o jsonpath='{.secrets[0].name}')
USER_TOKEN=$(kubectl get secret $SEC_NAME -o json | jq -r '.data["token"]' | base64 -d)

cluster_name=$(kubectl config get-contexts $(kubectl config current-context) | awk '{print $3}' | tail -n 1)
kubectl config set-credentials simple-user --token="${USER_TOKEN}"
kubectl config set-context simple-reader --cluster=$cluster_name --user simple-user
kubectl config set-context simple-reader
```

 - [How to Access Dashboard?](https://blog.csdn.net/xixihahalelehehe/article/details/115913408)
 - use kubectl proxy to access to the Dashboard [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)
 - Ref: [https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#accessing-the-dashboard-ui)
 - Ref: [https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)

##  5. Verify platform binaries before deploying

 - CKS Preparation Guide Github[https://github.com/ramanagali/Interview_Guide/blob/main/CKS_Preparation_Guide.md](https://github.com/ramanagali/Interview_Guide/blob/main/CKS_Preparation_Guide.md)
 - [Verify Platform](https://ghostwritten.blog.csdn.net/article/details/116019776)

Example 1

```bash
curl -LO https://dl.k8s.io/v1.23.1/kubernetes-client-darwin-arm64.tar.gz -o kubernetes-client-darwin-arm64.tar.gz

# Print SHA Checksums - mac
shasum -a 512 kubernetes-client-darwin-arm64.tar.gz
# Print SHA Checksums - linux
sha512sum kubernetes-client-darwin-arm64.tar.gz
sha512sum kube-apiserver
sha512sum kube-proxy
```

Example 2

```bash
kubectl version --short --client

#download checksum for kubectl for linux - (change version)
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

#download old version
curl -LO "https://dl.k8s.io/v1.22.1/bin/linux/amd64/kubectl.sha256"

#download checksum for kubectl for mac - (change version)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl.sha256"

#insall coreutils (for mac)
brew install coreutils

#verify kubectl binary (for linux)
echo "$(<kubectl.sha256) /usr/bin/kubectl" | sha256sum --check
#verify kubectl binary (for mac)
echo "$(<kubectl.sha256) /usr/local/bin/kubectl" | sha256sum -c
```

 - Ref:   [https://github.com/kubernetes/kubernetes/releases](https://github.com/kubernetes/kubernetes/releases)
 - Ref:[https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG#changelogs](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG#changelogs)
 - Ref: [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

##  6. Restrict access to Kubernetes API 

```bash
$ k get pod -A
$ k proxy
Starting to serve on 127.0.0.1:8001
```

```bash
 $ curl 127.0.0.1:8001
 {
  "paths": [
    "/.well-known/openid-configuration",
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apiregistration.k8s.io/v1beta1",
    "/apis/apps",
    "/apis/apps/v1",
    "/apis/authentication.k8s.io",
    "/apis/authentication.k8s.io/v1",
    "/apis/authentication.k8s.io/v1beta1",
    "/apis/authorization.k8s.io",
    "/apis/authorization.k8s.io/v1",
    "/apis/authorization.k8s.io/v1beta1",
    "/apis/autoscaling",
    "/apis/autoscaling/v1",
    "/apis/autoscaling/v2beta1",
    "/apis/autoscaling/v2beta2",
    "/apis/batch",
    "/apis/batch/v1",
    "/apis/batch/v1beta1",
    "/apis/certificates.k8s.io",
    "/apis/certificates.k8s.io/v1",
    "/apis/certificates.k8s.io/v1beta1",
    "/apis/config.gatekeeper.sh",
    "/apis/coordination.k8s.io",
    "/apis/coordination.k8s.io/v1",
    "/apis/coordination.k8s.io/v1beta1",
    "/apis/crd.projectcalico.org",
    "/apis/crd.projectcalico.org/v1",
    "/apis/discovery.k8s.io",
    "/apis/discovery.k8s.io/v1beta1",
    "/apis/events.k8s.io",
    "/apis/events.k8s.io/v1",
    "/apis/events.k8s.io/v1beta1",
    "/apis/extensions",
    "/apis/extensions/v1beta1",
    "/apis/flowcontrol.apiserver.k8s.io",
    "/apis/flowcontrol.apiserver.k8s.io/v1beta1",
    "/apis/networking.k8s.io",
    "/apis/networking.k8s.io/v1",
    "/apis/networking.k8s.io/v1beta1",
    "/apis/node.k8s.io",
    "/apis/node.k8s.io/v1",
    "/apis/node.k8s.io/v1beta1",
    "/apis/policy",
    "/apis/policy/v1beta1",
    "/apis/rbac.authorization.k8s.io",
    "/apis/rbac.authorization.k8s.io/v1",
    "/apis/rbac.authorization.k8s.io/v1beta1",
    "/apis/scheduling.k8s.io",
    "/apis/scheduling.k8s.io/v1",
    "/apis/scheduling.k8s.io/v1beta1",
    "/apis/status.gatekeeper.sh",
    "/apis/storage.k8s.io",
    "/apis/storage.k8s.io/v1",
    "/apis/storage.k8s.io/v1beta1",
    "/apis/templates.gatekeeper.sh",
    "/healthz",
    "/healthz/autoregister-completion",
    "/healthz/etcd",
    "/healthz/log",
    "/healthz/ping",
    "/healthz/poststarthook/aggregator-reload-proxy-client-cert",
    "/healthz/poststarthook/apiservice-openapi-controller",
    "/healthz/poststarthook/apiservice-registration-controller",
    "/healthz/poststarthook/apiservice-status-available-controller",
    "/healthz/poststarthook/bootstrap-controller",
    "/healthz/poststarthook/crd-informer-synced",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/kube-apiserver-autoregistration",
    "/healthz/poststarthook/priority-and-fairness-config-consumer",
    "/healthz/poststarthook/priority-and-fairness-config-producer",
    "/healthz/poststarthook/priority-and-fairness-filter",
    "/healthz/poststarthook/rbac/bootstrap-roles",
    "/healthz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/healthz/poststarthook/start-cluster-authentication-info-controller",
    "/healthz/poststarthook/start-kube-aggregator-informers",
    "/healthz/poststarthook/start-kube-apiserver-admission-initializer",
    "/livez",
    "/livez/autoregister-completion",
    "/livez/etcd",
    "/livez/log",
    "/livez/ping",
    "/livez/poststarthook/aggregator-reload-proxy-client-cert",
    "/livez/poststarthook/apiservice-openapi-controller",
    "/livez/poststarthook/apiservice-registration-controller",
    "/livez/poststarthook/apiservice-status-available-controller",
    "/livez/poststarthook/bootstrap-controller",
    "/livez/poststarthook/crd-informer-synced",
    "/livez/poststarthook/generic-apiserver-start-informers",
    "/livez/poststarthook/kube-apiserver-autoregistration",
    "/livez/poststarthook/priority-and-fairness-config-consumer",
    "/livez/poststarthook/priority-and-fairness-config-producer",
    "/livez/poststarthook/priority-and-fairness-filter",
    "/livez/poststarthook/rbac/bootstrap-roles",
    "/livez/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/livez/poststarthook/start-apiextensions-controllers",
    "/livez/poststarthook/start-apiextensions-informers",
    "/livez/poststarthook/start-cluster-authentication-info-controller",
    "/livez/poststarthook/start-kube-aggregator-informers",
    "/livez/poststarthook/start-kube-apiserver-admission-initializer",
    "/logs",
    "/metrics",
    "/openapi/v2",
    "/openid/v1/jwks",
    "/readyz",
    "/readyz/autoregister-completion",
    "/readyz/etcd",
    "/readyz/informer-sync",
    "/readyz/log",
    "/readyz/ping",
    "/readyz/poststarthook/aggregator-reload-proxy-client-cert",
    "/readyz/poststarthook/apiservice-openapi-controller",
    "/readyz/poststarthook/apiservice-registration-controller",
    "/readyz/poststarthook/apiservice-status-available-controller",
    "/readyz/poststarthook/bootstrap-controller",
    "/readyz/poststarthook/crd-informer-synced",
    "/readyz/poststarthook/generic-apiserver-start-informers",
    "/readyz/poststarthook/kube-apiserver-autoregistration",
    "/readyz/poststarthook/priority-and-fairness-config-consumer",
    "/readyz/poststarthook/priority-and-fairness-config-producer",
    "/readyz/poststarthook/priority-and-fairness-filter",
    "/readyz/poststarthook/rbac/bootstrap-roles",
    "/readyz/poststarthook/scheduling/bootstrap-system-priority-classes",
    "/readyz/poststarthook/start-apiextensions-controllers",
    "/readyz/poststarthook/start-apiextensions-informers",
    "/readyz/poststarthook/start-cluster-authentication-info-controller",
    "/readyz/poststarthook/start-kube-aggregator-informers",
    "/readyz/poststarthook/start-kube-apiserver-admission-initializer",
    "/readyz/shutdown",
    "/version"
  ]
}
```

```bash
$ curl 127.0.0.1:8001/api/v1
{
  "kind": "APIResourceList",
  "groupVersion": "v1",
  "resources": [
    {
      "name": "bindings",
      "singularName": "",
      "namespaced": true,
      "kind": "Binding",
      "verbs": [
        "create"
      ]
    },
    {
      "name": "componentstatuses",
      "singularName": "",
      "namespaced": false,
      "kind": "ComponentStatus",
      "verbs": [
        "get",
        "list"
      ],
      "shortNames": [
        "cs"
      ]
    },
    {
      "name": "configmaps",
      "singularName": "",
      "namespaced": true,
      "kind": "ConfigMap",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "cm"
      ],
      "storageVersionHash": "qFsyl6wFWjQ="
    },
    {
      "name": "endpoints",
      "singularName": "",
      "namespaced": true,
      "kind": "Endpoints",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "ep"
      ],
      "storageVersionHash": "fWeeMqaN/OA="
    },
    {
      "name": "events",
      "singularName": "",
      "namespaced": true,
      "kind": "Event",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "ev"
      ],
      "storageVersionHash": "r2yiGXH7wu8="
    },
    {
      "name": "limitranges",
      "singularName": "",
      "namespaced": true,
      "kind": "LimitRange",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "limits"
      ],
      "storageVersionHash": "EBKMFVe6cwo="
    },
    {
      "name": "namespaces",
      "singularName": "",
      "namespaced": false,
      "kind": "Namespace",
      "verbs": [
        "create",
        "delete",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "ns"
      ],
      "storageVersionHash": "Q3oi5N2YM8M="
    },
    {
      "name": "namespaces/finalize",
      "singularName": "",
      "namespaced": false,
      "kind": "Namespace",
      "verbs": [
        "update"
      ]
    },
    {
      "name": "namespaces/status",
      "singularName": "",
      "namespaced": false,
      "kind": "Namespace",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "nodes",
      "singularName": "",
      "namespaced": false,
      "kind": "Node",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "no"
      ],
      "storageVersionHash": "XwShjMxG9Fs="
    },
    {
      "name": "nodes/proxy",
      "singularName": "",
      "namespaced": false,
      "kind": "NodeProxyOptions",
      "verbs": [
        "create",
        "delete",
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "nodes/status",
      "singularName": "",
      "namespaced": false,
      "kind": "Node",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "persistentvolumeclaims",
      "singularName": "",
      "namespaced": true,
      "kind": "PersistentVolumeClaim",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "pvc"
      ],
      "storageVersionHash": "QWTyNDq0dC4="
    },
    {
      "name": "persistentvolumeclaims/status",
      "singularName": "",
      "namespaced": true,
      "kind": "PersistentVolumeClaim",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "persistentvolumes",
      "singularName": "",
      "namespaced": false,
      "kind": "PersistentVolume",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "pv"
      ],
      "storageVersionHash": "HN/zwEC+JgM="
    },
    {
      "name": "persistentvolumes/status",
      "singularName": "",
      "namespaced": false,
      "kind": "PersistentVolume",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "pods",
      "singularName": "",
      "namespaced": true,
      "kind": "Pod",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "po"
      ],
      "categories": [
        "all"
      ],
      "storageVersionHash": "xPOwRZ+Yhw8="
    },
    {
      "name": "pods/attach",
      "singularName": "",
      "namespaced": true,
      "kind": "PodAttachOptions",
      "verbs": [
        "create",
        "get"
      ]
    },
    {
      "name": "pods/binding",
      "singularName": "",
      "namespaced": true,
      "kind": "Binding",
      "verbs": [
        "create"
      ]
    },
    {
      "name": "pods/eviction",
      "singularName": "",
      "namespaced": true,
      "group": "policy",
      "version": "v1beta1",
      "kind": "Eviction",
      "verbs": [
        "create"
      ]
    },
    {
      "name": "pods/exec",
      "singularName": "",
      "namespaced": true,
      "kind": "PodExecOptions",
      "verbs": [
        "create",
        "get"
      ]
    },
    {
      "name": "pods/log",
      "singularName": "",
      "namespaced": true,
      "kind": "Pod",
      "verbs": [
        "get"
      ]
    },
    {
      "name": "pods/portforward",
      "singularName": "",
      "namespaced": true,
      "kind": "PodPortForwardOptions",
      "verbs": [
        "create",
        "get"
      ]
    },
    {
      "name": "pods/proxy",
      "singularName": "",
      "namespaced": true,
      "kind": "PodProxyOptions",
      "verbs": [
        "create",
        "delete",
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "pods/status",
      "singularName": "",
      "namespaced": true,
      "kind": "Pod",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "podtemplates",
      "singularName": "",
      "namespaced": true,
      "kind": "PodTemplate",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "storageVersionHash": "LIXB2x4IFpk="
    },
    {
      "name": "replicationcontrollers",
      "singularName": "",
      "namespaced": true,
      "kind": "ReplicationController",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "rc"
      ],
      "categories": [
        "all"
      ],
      "storageVersionHash": "Jond2If31h0="
    },
    {
      "name": "replicationcontrollers/scale",
      "singularName": "",
      "namespaced": true,
      "group": "autoscaling",
      "version": "v1",
      "kind": "Scale",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "replicationcontrollers/status",
      "singularName": "",
      "namespaced": true,
      "kind": "ReplicationController",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "resourcequotas",
      "singularName": "",
      "namespaced": true,
      "kind": "ResourceQuota",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "quota"
      ],
      "storageVersionHash": "8uhSgffRX6w="
    },
    {
      "name": "resourcequotas/status",
      "singularName": "",
      "namespaced": true,
      "kind": "ResourceQuota",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "secrets",
      "singularName": "",
      "namespaced": true,
      "kind": "Secret",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "storageVersionHash": "S6u1pOWzb84="
    },
    {
      "name": "serviceaccounts",
      "singularName": "",
      "namespaced": true,
      "kind": "ServiceAccount",
      "verbs": [
        "create",
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "sa"
      ],
      "storageVersionHash": "pbx9ZvyFpBE="
    },
    {
      "name": "serviceaccounts/token",
      "singularName": "",
      "namespaced": true,
      "group": "authentication.k8s.io",
      "version": "v1",
      "kind": "TokenRequest",
      "verbs": [
        "create"
      ]
    },
    {
      "name": "services",
      "singularName": "",
      "namespaced": true,
      "kind": "Service",
      "verbs": [
        "create",
        "delete",
        "get",
        "list",
        "patch",
        "update",
        "watch"
      ],
      "shortNames": [
        "svc"
      ],
      "categories": [
        "all"
      ],
      "storageVersionHash": "0/CO1lhkEBI="
    },
    {
      "name": "services/proxy",
      "singularName": "",
      "namespaced": true,
      "kind": "ServiceProxyOptions",
      "verbs": [
        "create",
        "delete",
        "get",
        "patch",
        "update"
      ]
    },
    {
      "name": "services/status",
      "singularName": "",
      "namespaced": true,
      "kind": "Service",
      "verbs": [
        "get",
        "patch",
        "update"
      ]
    }
  ]
}
```

```bash
k create ns test
k run web --image nginx -n test
k get po -n test
```

```bash
$ curl 127.0.0.1:8001/api/v1/namespaces/test/pods
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "222842"
  },
  "items": [
    {
      "metadata": {
        "name": "web",
        "namespace": "test",
        "uid": "9f0e679b-6f92-48fe-a398-fcda6b36347f",
        "resourceVersion": "222793",
        "creationTimestamp": "2022-08-30T06:09:15Z",
        "labels": {
          "run": "web"
        },
        "annotations": {
          "cni.projectcalico.org/podIP": "192.168.166.145/32",
          "cni.projectcalico.org/podIPs": "192.168.166.145/32"
        },
        "managedFields": [
          {
            "manager": "kubectl-run",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-08-30T06:09:15Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {"f:metadata":{"f:labels":{".":{},"f:run":{}}},"f:spec":{"f:containers":{"k:{\"name\":\"web\"}":{".":{},"f:image":{},"f:imagePullPolicy":{},"f:name":{},"f:resources":{},"f:terminationMessagePath":{},"f:terminationMessagePolicy":{}}},"f:dnsPolicy":{},"f:enableServiceLinks":{},"f:restartPolicy":{},"f:schedulerName":{},"f:securityContext":{},"f:terminationGracePeriodSeconds":{}}}
          },
          {
            "manager": "calico",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-08-30T06:09:18Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {"f:metadata":{"f:annotations":{".":{},"f:cni.projectcalico.org/podIP":{},"f:cni.projectcalico.org/podIPs":{}}}}
          },
          {
            "manager": "kubelet",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2022-08-30T06:09:48Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {"f:status":{"f:conditions":{"k:{\"type\":\"ContainersReady\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Initialized\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Ready\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}}},"f:containerStatuses":{},"f:hostIP":{},"f:phase":{},"f:podIP":{},"f:podIPs":{".":{},"k:{\"ip\":\"192.168.166.145\"}":{".":{},"f:ip":{}}},"f:startTime":{}}}
          }
        ]
      },
      "spec": {
        "volumes": [
          {
            "name": "default-token-th4sb",
            "secret": {
              "secretName": "default-token-th4sb",
              "defaultMode": 420
            }
          }
        ],
        "containers": [
          {
            "name": "web",
            "image": "nginx",
            "resources": {
              
            },
            "volumeMounts": [
              {
                "name": "default-token-th4sb",
                "readOnly": true,
                "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount"
              }
            ],
            "terminationMessagePath": "/dev/termination-log",
            "terminationMessagePolicy": "File",
            "imagePullPolicy": "Always"
          }
        ],
        "restartPolicy": "Always",
        "terminationGracePeriodSeconds": 30,
        "dnsPolicy": "ClusterFirst",
        "serviceAccountName": "default",
        "serviceAccount": "default",
        "nodeName": "node1",
        "securityContext": {
          
        },
        "schedulerName": "default-scheduler",
        "tolerations": [
          {
            "key": "node.kubernetes.io/not-ready",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          },
          {
            "key": "node.kubernetes.io/unreachable",
            "operator": "Exists",
            "effect": "NoExecute",
            "tolerationSeconds": 300
          }
        ],
        "priority": 0,
        "enableServiceLinks": true,
        "preemptionPolicy": "PreemptLowerPriority"
      },
      "status": {
        "phase": "Running",
        "conditions": [
          {
            "type": "Initialized",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-08-30T06:09:15Z"
          },
          {
            "type": "Ready",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-08-30T06:09:48Z"
          },
          {
            "type": "ContainersReady",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-08-30T06:09:48Z"
          },
          {
            "type": "PodScheduled",
            "status": "True",
            "lastProbeTime": null,
            "lastTransitionTime": "2022-08-30T06:09:15Z"
          }
        ],
        "hostIP": "192.168.211.41",
        "podIP": "192.168.166.145",
        "podIPs": [
          {
            "ip": "192.168.166.145"
          }
        ],
        "startTime": "2022-08-30T06:09:15Z",
        "containerStatuses": [
          {
            "name": "web",
            "state": {
              "running": {
                "startedAt": "2022-08-30T06:09:47Z"
              }
            },
            "lastState": {
              
            },
            "ready": true,
            "restartCount": 0,
            "image": "nginx:latest",
            "imageID": "docker-pullable://nginx@sha256:b95a99feebf7797479e0c5eb5ec0bdfa5d9f504bc94da550c2f58e839ea6914f",
            "containerID": "docker://5d62f6cac401ab23a48b3824c4e06a92189e8e4005d1e6b78bfe26395259efeb",
            "started": true
          }
        ],
        "qosClass": "BestEffort"
      }
    }
  ]
}
```

```bash
k config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.211.40:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/1aa958b037084d7f8d6cdbdaa5cb9120.png)

```bash
curl -k  https://192.168.211.40:6443
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {
    
  },
  "code": 403
}
```

```bash
$ k cluster-info
Kubernetes control plane is running at https://192.168.211.40:6443
KubeDNS is running at https://192.168.211.40:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy


$  cat /root/.kube/config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1EY3lOekE0TVRreU9Wb1hEVE15TURjeU5EQTRNVGt5T1Zvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTkh5CmNiNy9JRFFVOE0vd0ZsRU84VFpKY2pQNC9jYWEwbXY3RlJtbXl0dGVDaFJUTWNWcmRibFYyNzFXRVpQR2plK0wKUC9PU2U5K2ovQzFqWGdjQ1VvU2x2RHIxK1IvYnNyN3BsZVkvcTR3TXlFSGV4bHkyK3Q2a1VoL0Z5bW9iZEZqSQpGM01JbjE5QllzN1E0dFVuR3E0UGJxZGN5b01EdlpkL3RERCtMYWk0aDZLSlBGcHZwRnZTQ0RwcDdrUlROM29ICnVJU2x2d3kwUEVhVFZBano3UVBPYmhVY1NpTUQrZWtBdWErNXFoU0NvOHNQbnlZVWRxK1lUU01PT05ObWZGVnMKRTBDZlRTbm5YMVd4RmwxQ3AyaHphSFV0b1E5Yld1SjNRU004RXZkTS80S05xQ1krMDlaYVZ0NFF2NGNHUWNRSgp6MXFVUVJkZnRCT0JnMmhHSG44Q0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZCTE1KeWc2czlFbEdLRmpjdmQ2SlZvdDZzZEVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCMGlvRGRGaUs1T0ovVHJkWWpyalhmczQweTJ1QXMwWmc1K010TUxUelhrQjJjczRZYQp0eWR1V3htVmdWdzNYSDhrYWdmRVlUVTQ2STU5STVRQVlHekt6blAvWndZRlVVcS81ZlpNbDJ1cDh4ZUl6NUxJClN4MFFnV1oxanNhaHAwRnZLNk0wMEQrNVRQS3M5MW8vV3BqR1BYWVNhMHJ5YXd0K3dZNlNMblpDTGp4ZFR4aS8KaExXazR3WTA1WjVFZnNWNDVZYTgxVEl5V3pzekdTaWNXM3ZWU1hQRFB3Z1REOTN2WWtEbU9ZSVFVYkdtekhmTgpndmlLNlhKZ3VZR2pkRERxd2xwT1I4NVEvRFZzVWY2bS9wbk1kVThRcHFNM2Y2T0Q4eDI5WEZ3dTEra0MxSmlMCnZmNUNNdWN3S3NUY2JGUnY4RWZOaEl5c2VxTDhXYXBsUXlxdgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.211.40:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJVlRJdExqbmRtQmt3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TWpBM01qY3dPREU1TWpsYUZ3MHlNekEzTWpjd09ERTVNekphTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXlzc1ZxRmhpalllVzFNVGIKcFFYaEwrSlVHK2FuSEJTUUFDdVV5YWFrSUl3TjlPd2V3aUxLNVIvRVYrVC95OHpTcGs1cEx4cldmM3VSTTlFdgord0NwbG4xQzVGekF1cGdXSWUybnNkZ2R2am16Z3FCbGlCWTI1SW9nbXlybjMrb0RBVlRkUEV1Y1p2YndhbzhFCm9HNFVacGVwMGtZb2toT1lLMTVzWHBWOFJwNCtLSDRkcG9SaWNpSy9CTVU3MmlwcWhrZDE0bE1XVVIxVlhCa0QKY0F3cUJKTEJ0KzJKY0ZqZGYxSGdJaXhLSGRycTNHb2doY0w3eDFUMENFa3AzaGhiVDE1UzNTUC8zSE1ibktLcQplN0RNNk1UOU9DQWRybXgzdUFxMmV5akJPclliZG5JRzVJQTU2OHlhRW9KOHFVR0tRQmFrdVJockhoMXhWL3dYCkx1WWNsd0lEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVVFc3duS0RxejBTVVlvV055OTNvbFdpM3F4MFF3RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFBaG5pNkNzK3NLOVF4U0VPVFpRQWdReGtCYVpnaGF5OGJYdmxRVndQaXoreUtoZlFUeERsckRNCmNsSmoxZ0FBeVM2aDFvZ1VEcU5va29CeVM2R0tUNUVBZ24rQ1JhR1Zld0NKbnNEQ2g2Z01qYWZORG1aNEhTbloKTmE5ZFp0bSt2TklWNWhDNTBMUEJyaUQwVkZRTEFVRXhEakJ3bjVpN0FkTm8zeGRQWHNib25Kd3ZxbDFTMmEwUgpPTzdJRURhUXNFOGFFd1UvbGhocWVNczhhWkNRMU01RllQeDdhell3RmJBRmJiRVJQUk9Ld1g1WUMyaTU4OHdWCmNMREYrNnNiVzVjSk5KekJGZFp3djhhZDNseFVzODJiNFhOdXdacTQyd09IREQybDd1ODZGMFl3K2Q2aEIvbEwKaWIvRVpRZy9oZHlhQkZSQWxSa3BJL05oRDdGR1JGST0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBeXNzVnFGaGlqWWVXMU1UYnBRWGhMK0pVRythbkhCU1FBQ3VVeWFha0lJd045T3dlCndpTEs1Ui9FVitUL3k4elNwazVwTHhyV2YzdVJNOUV2K3dDcGxuMUM1RnpBdXBnV0llMm5zZGdkdmptemdxQmwKaUJZMjVJb2dteXJuMytvREFWVGRQRXVjWnZid2FvOEVvRzRVWnBlcDBrWW9raE9ZSzE1c1hwVjhScDQrS0g0ZApwb1JpY2lLL0JNVTcyaXBxaGtkMTRsTVdVUjFWWEJrRGNBd3FCSkxCdCsySmNGamRmMUhnSWl4S0hkcnEzR29nCmhjTDd4MVQwQ0VrcDNoaGJUMTVTM1NQLzNITWJuS0txZTdETTZNVDlPQ0Fkcm14M3VBcTJleWpCT3JZYmRuSUcKNUlBNTY4eWFFb0o4cVVHS1FCYWt1UmhySGgxeFYvd1hMdVljbHdJREFRQUJBb0lCQUNlVkxFMEhzM1RjbWx3OQpjSUh0ZTk3VTFvWDdwM0tic04vWG9kc2FZNzdXbDRMTzg5SUE2SW1BZ2RxR0lFZXZXdzZMRDR6YU9EUDU4b1dpCnR6TFBGa3NCZUNVSzFiT1dLL3ZEWDVBZkZ1OGlaQitESDA1SXg3NGtGK2t4bnNEZDlHZzJJRmk4aVhLdmtJMjgKRExNanlXZWRBdERBVVByeVNDbHU3TWdwZFhCeTZUZFJZTEhENjBLQkM2WlhDWUdwSlQ2aG5ZWGF2M0xtN1htawpXK0ZBeFMyUWsxR3ExTm9wMnRNbVlTd3I5ZjE0T0lFVzdaNHI5V2xtVlhsQXhWdzN5RUgwUC9UL0dGQk5VeUNBCjNITUFZSVRFak51Mlp4U2JNY2JrdGk0Q3NZMC9CMHk1UzNPNS9iUWxYTy9LYnNzZVhKVGl1djY0Q1hROVhja1gKSFRDRUFBRUNnWUVBLzNVVFR4dkJtL0FWTklRSmZPT2l1ZW5yWDlTRzVwTytma1hDeTJ4aFZFZFdPK3k2MG83Wgo1dGpTekRLYkJvOFAyOW9vS0JiZkRGaXhLQlBqYi93bHArc0hPN2svSFJBK1RBeFV4dWpITGlEVEhrU2pkWUJSCjFxQWFNOHlWamdSWTJjbzBpK21oQlN1Sk5TK2dmR1Q4Q0tISG9CSHFUQ3d5MXR3cDFYeDlFQUVDZ1lFQXl6bGUKZXBZeUdzQ1EzQ3FMdUJDU3dqbnJ4ZHE0WTFiSFlNTU1UVnI5S0dKWU04Y1NrRnRicEVJLzZYVk84Ry91QUVEZQpRUVE3Y0RTYjBISWNiWk04YXRUMTN4U0xINlhVWEtmQ1Y3cXVVQkdtUldqdlpJN0NTVkVnSG1laUdSNEVlRm9ZCjBGOTNSY0pJZnVFaVk3emlRQktPR1pqZTNpM1JSR0YzZzZwaHJKY0NnWUVBMXdNNmlsWXBZay96K1N5OU02SUIKb0F1ME1nZVd0OUpZL3IxRzFLTlhWSEZxc3F0eEg3SmUwMzlpQmI3K1hzbmhKa0g3bEtxVGVEZmFmSW9vMzJQUwphZ0JYS1R5bFU1Z05aMExseERtL0ZDTktydXBFenF4L3RXOHlQckVPbStjcXhiejg5MXBnVGhLenZORm1lZTBoCmVUNTU0RS9UN2VNeHMwakI2VStMa0FFQ2dZQUJhZnpHVFpVN3FtdFhuTlFzQzdGNXVIMXpldm9kZHRVY1R6OGUKcXF0b1JJYm9sVklEdng3OEhabmtQZlMycDVDNFg3c3NLS05oUEh4NUR0SXowUHB5bzlpeUhLcDdKZVE4WU01eApYZE1vcTNiRXRONDFqT2k5S2R0WFd0RTk2MytNZHRRRlh5U3RUNVRCalQ5NEFqQncwYkE3YlZ6Zm51SDkzOCs5CkVzcHJNUUtCZ1FDZEFYNWFjQUN5L1RudVg0bGNNRUJtbk43bGFyZCtValNHYTlsU1M1TzhFSSsvakh6OGlSckoKSlYrMllNd2lnYnBNdjM2M1p5WDBBcnRyT1FoMElXaFl0eDdlcDQ4azJLTzhHaUViNUJsZ0pmaDJLdmF0cjFwSApjQVhocCs4RUFJbUZ0c1NoRGtXdjhMaVNMa0FYOVE2dUo4bEdscFJCTnUzVTByRnBMNXFmSVE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=


```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0bd281732c594b06b0cdbeb500bab56f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/450f669ff7c44ca390cb058ae0561ec0.png)

```bash
$ kubectl config current-context
kubernetes-admin@kubernetes

$ k auth can-i --list
Resources                                       Non-Resource URLs   Resource Names   Verbs
*.*                                             []                  []               [*]
                                                [*]                 []               [*]
selfsubjectaccessreviews.authorization.k8s.io   []                  []               [create]
selfsubjectrulesreviews.authorization.k8s.io    []                  []               [create]
                                                [/api/*]            []               [get]
                                                [/api]              []               [get]
                                                [/apis/*]           []               [get]
                                                [/apis]             []               [get]
                                                [/healthz]          []               [get]
                                                [/healthz]          []               [get]
                                                [/livez]            []               [get]
                                                [/livez]            []               [get]
                                                [/openapi/*]        []               [get]
                                                [/openapi]          []               [get]
                                                [/readyz]           []               [get]
                                                [/readyz]           []               [get]
                                                [/version/]         []               [get]
                                                [/version/]         []               [get]
                                                [/version]          []               [get]
                                                [/version]          []               [get]

```


 1. localhost

    - port 8080
    - no TLS
    - default IP is localhost, change with `--insecure-bind-address`

 1. secure port

    - default is 6443, change with `--secure-port`
     - set TLS certificate with `--tls-cert-file`
     - set TLS certificate key with `--tls-private-key-file` flag

Control anonymous requests to Kube-apiserver by using `--anonymous-auth=false`

example for adding anonymous access

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: anonymous-review-access
rules:
- apiGroups:
  - authorization.k8s.io
  resources:
  - selfsubjectaccessreviews
  - selfsubjectrulesreviews
  verbs:
  - create
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: anonymous-review-access
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: anonymous-review-access
subjects:
- kind: User
  name: system:anonymous
  namespace: default

```



验证
```bash
#check using anonymous
kubectl auth can-i --list --as=system:anonymous -n default
#check using yourown account
kubectl auth can-i --list
```

 - Ref: [https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips](https://kubernetes.io/docs/concepts/security/controlling-access/#api-server-ports-and-ips)
 - Ref: [https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#anonymous-requests)
 - Ref: [https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#without-kubectl-proxy](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#without-kubectl-proxy)
 - Ref: [https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-authentication-authorization/#kubelet-authentication](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-authentication-authorization/#kubelet-authentication)

##  7. Disable Automount Service Account Token

```bash
$ k creat ns dev
$ k get sa -n dev
NAME      SECRETS   AGE
default   1         66s

$ k get secrets -n dev
NAME                  TYPE                                  DATA   AGE
default-token-7xv8n   kubernetes.io/service-account-token   3      74s

$ k run web --image nginx -n dev

$ k get po -n dev 
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          58s

```

```bash
$ k get po -n dev web -oyaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.166.146/32
    cni.projectcalico.org/podIPs: 192.168.166.146/32
  creationTimestamp: "2022-08-30T06:59:27Z"
  labels:
    run: web
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:run: {}
      f:spec:
        f:containers:
          k:{"name":"web"}:
            .: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:name: {}
            f:resources: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
    manager: kubectl-run
    operation: Update
    time: "2022-08-30T06:59:27Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:cni.projectcalico.org/podIP: {}
          f:cni.projectcalico.org/podIPs: {}
    manager: calico
    operation: Update
    time: "2022-08-30T06:59:29Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"192.168.166.146"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2022-08-30T06:59:38Z"
  name: web
  namespace: dev
  resourceVersion: "227448"
  uid: 4098d08f-ae18-4612-bf82-fe854537ca14
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: web
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-7xv8n
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: node1
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-7xv8n
    secret:
      defaultMode: 420
      secretName: default-token-7xv8n
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-08-30T06:59:27Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-08-30T06:59:38Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-08-30T06:59:38Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2022-08-30T06:59:27Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://ef8ec3aea71713fe9a4b904387ed938db8c0d90fedb7be586db41a35ce829201
    image: nginx:latest
    imageID: docker-pullable://nginx@sha256:b95a99feebf7797479e0c5eb5ec0bdfa5d9f504bc94da550c2f58e839ea6914f
    lastState: {}
    name: web
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2022-08-30T06:59:38Z"
  hostIP: 192.168.211.41
  phase: Running
  podIP: 192.168.166.146
  podIPs:
  - ip: 192.168.166.146
  qosClass: BestEffort
  startTime: "2022-08-30T06:59:27Z"

```

```bash
k exec -it web -n dev -- sh
# cat /var/run/secrets/kubernetes.io/serviceaccount/token
eyJhbGciOiJSUzI1NiIsImtpZCI6IjNROWFvUGhIWk9sYzBHT3JvOUJDb2wtX29Tc1Z0blRVUmxpeEhmaUpIUFEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZXYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiZGVmYXVsdC10b2tlbi03eHY4biIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMzA4ODQ2N2MtMjRmNC00OWJmLTkyMTItYWJkMzJkYWE4OTQwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRldjpkZWZhdWx0In0.GfHynulFu_d9F1rq_hApxwuInPlFmJMkt8BLqfRsk8gPtxLrQXE8aJG1PzZXlYxW5oxU-QkqE3StqDZ2gfpvJfLgBMmRT3u7kt5N2zQ7L9slxJt_sazdqrpxV6848yHR_gMM3hGpKOfShyvI_vmZ9IbIIE3mpuXFbUNk6T2i5OuZgrqCX462OMp_gVMIe1DRAmxHRkU6qjPb7KSCbZsE1bu8Ov5Ns6_L5Peg9di4nge5wGj60ggiZ927Pkr93FlFyDp8xGNIjgXnGLPjiN5Lyz8sfWm-Gci-HDChMhN1NMMRjNGGaC3MJ-VbXlsibBhC0rsG9CWmnlQs9I-5OOrR6A
```

```bash
k get secrets -n dev
NAME                  TYPE                                  DATA   AGE
default-token-7xv8n   kubernetes.io/service-account-token   3      22m
root@master:~# k describe secrets -n dev default-token-7xv8n
Name:         default-token-7xv8n
Namespace:    dev
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 3088467c-24f4-49bf-9212-abd32daa8940

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjNROWFvUGhIWk9sYzBHT3JvOUJDb2wtX29Tc1Z0blRVUmxpeEhmaUpIUFEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZXYiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiZGVmYXVsdC10b2tlbi03eHY4biIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMzA4ODQ2N2MtMjRmNC00OWJmLTkyMTItYWJkMzJkYWE4OTQwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRldjpkZWZhdWx0In0.GfHynulFu_d9F1rq_hApxwuInPlFmJMkt8BLqfRsk8gPtxLrQXE8aJG1PzZXlYxW5oxU-QkqE3StqDZ2gfpvJfLgBMmRT3u7kt5N2zQ7L9slxJt_sazdqrpxV6848yHR_gMM3hGpKOfShyvI_vmZ9IbIIE3mpuXFbUNk6T2i5OuZgrqCX462OMp_gVMIe1DRAmxHRkU6qjPb7KSCbZsE1bu8Ov5Ns6_L5Peg9di4nge5wGj60ggiZ927Pkr93FlFyDp8xGNIjgXnGLPjiN5Lyz8sfWm-Gci-HDChMhN1NMMRjNGGaC3MJ-VbXlsibBhC0rsG9CWmnlQs9I-5OOrR6A
ca.crt:     1066 bytes
namespace:  3 bytes

```
token 一致。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5e05e4310e7441ef8c1876908f8b3e6d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8eb6a60cb396468687dee45d8210a058.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/df0a1b1501b4420c8e065e644c5902ab.png)
### 7.1 创建服务账户：test-dev 
```bash
$ k create sa test-svc -n dev --dry-run=client -o yaml > svc.yaml

$ vim svc.yaml 
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: test-svc
  namespace: dev
automountServiceAccountToken: false    # 添加


$ k get sa -n dev
NAME       SECRETS   AGE
default    1         34m
test-svc   1         2m48s

$ k get secrets -n dev
NAME                   TYPE                                  DATA   AGE
default-token-7xv8n    kubernetes.io/service-account-token   3      34m
test-svc-token-6qs7z   kubernetes.io/service-account-token   3      3m

```
### 7.2 创建无secret token pod：web2

```bash
k run web2 --image nginx -n dev --dry-run=client -o yaml > web2.yaml


$ cat web2.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web2
  name: web2
  namespace: dev
spec:
  serviceAccountName: test-svc   # 添加
  automountServiceAccountToken: false    # 添加
  containers:
  - image: nginx
    name: web2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


$ k apply -f web2.yaml

k get pod -n dev web2
NAME   READY   STATUS    RESTARTS   AGE
web2   1/1     Running   0          47s

# 无挂载
$ k describe  pod -n dev web2
```
### 7.3 服务test-svc RBAC角色权限

```bash
kubectl create role test-role -n dev --verb=get,list,watch,create --resource=pods --resource=pods,deployments,services,configmaps

kubectl create rolebinding cluster-test-binding -n dev --role=test-role --serviceaccount=dev:test-svc
```


##  8. Minimize host OS footprint Reduce Attack Surface
![在这里插入图片描述](https://img-blog.csdnimg.cn/7b34ac31836a46c5b3f55155b1392e41.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3974710d12564ba3bed664bf6b1ec952.png)

```bash
 k explain Pod.spec.hostIPC
KIND:     Pod
VERSION:  v1

FIELD:    hostIPC <boolean>

DESCRIPTION:
     Use the host's ipc namespace. Optional: Default to false.
root@master:~# k explain Pod.spec.hostNetwork
KIND:     Pod
VERSION:  v1

FIELD:    hostNetwork <boolean>

DESCRIPTION:
     Host networking requested for this pod. Use the host's network namespace.
     If this option is set, the ports that will be used must be specified.
     Default to false.

```

###  8.1 limit node access to users
![在这里插入图片描述](https://img-blog.csdnimg.cn/c556d1bfb427451f9a27a1fbeb6411f6.png)

```bash
userdel user1
groupdel group1
usermod -s /usr/sbin/nologin user2
useradd -d /opt/sam -s /bin/bash -G admin -u 2328 sam
```


###  8.2 remove obsolete/unnecessary software
![在这里插入图片描述](https://img-blog.csdnimg.cn/e7915a13842d4a26944cab297aaf91d5.png)

```bash
systemctl list-units --type service
systemctl stop squid
systemctl disable squid
apt remove squid
```

###  8.3 ssh hardening
![在这里插入图片描述](https://img-blog.csdnimg.cn/ba39144816734a7f9c4c072d1a786aa1.png)

```bash
ssh-keygen -t rsa
cat /home/mark/.ssh/authorized_keys

PermiRootLogin no
PasswordAuthentication no

systemctl restart sshd
```


###  8.4 ldentify & fix open ports
![在这里插入图片描述](https://img-blog.csdnimg.cn/59b00622dbc440ec83b656730f81f516.png)

```bash
apt list --installed
systemctl list -units --type service
lsmod
systemctl list-units --all | grep -i nginx
systemctl stop nginx
rm /lib/systemd/system/nginx.service
apt remove nginx -y 
netstat -atnlp | grep -i 9090 | grep -w -i listen
cat /etc/services | grep -i ssh
netstat -an | grep 22 | grep -w -i listen
netstat -natulp |grep -i light
```

###  8.5 Restrict Obsolete kernel modules
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac860a464bc54edfaced5ffa2acf75ce.png)

```bash
lsmod
vim /letc/modprobe.d/blacklist.conf
blacklist sctp
blacklist dccp
shutdown -r now
```


###  8.6 restrict allowed hostPath using PSP
![在这里插入图片描述](https://img-blog.csdnimg.cn/401e76df0b95495eaf7fbbd7cb0a723a.png)

### 8.7 UFW

```bash
systemctl enable ufw
systemctl start ufw

ufw default allow outgoing
ufw default deny incoming

ufw allow 1000:2000/tcp
ufw allow from 173.1.3.0/25 to any port 80 proto tcp

ufw delete deny 80
ufw delete 5
```


##  9. Restrict Container Access with AppArmor
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9f117d739f546f3b110886f5521dcc1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/18a77ea208ca4b26a48e8b62fd1113fd.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3f6cb2a216294de4af8130621784b965.png)

 - Kernel Security Module to granular access control for programs on Host OS
 - AppArmor Profile - Set of Rules, to be enabled in nodes
 - AppArmor Profile loaded in 2 modes

    - Complain Mode - Discover the program
    - Enforce Mode - prevent the program
 - create AppArmor Profile

```bash
$ sudo vi /etc/apparmor.d/deny-write

#include <tunables/global>
profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>
  file,
  # Deny all file writes.
  deny /** w,
}
```
copy deny-write profile to worker `node01` `scp deny-write node01:/tmp`

load the profile on all our nodes default directory /etc/apparmor.d `sudo apparmor_parser /etc/apparmor.d/deny-write`

apply to pod
```bash

apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
  annotations:
    container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write
spec:
  containers:
  - name: hello
    image: busybox
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
```

useful commands:

```bash
#check status            
systemctl status apparmor

#check enabled in nodes  
cat /sys/module/apparmor/parameters/enabled

#check profiles          
cat /sys/kernel/security/apparmor/profiles

#install                 
apt-get install apparmor-utils

#default Profile file directory  is /etc/apparmor.d/

#create apparmor profile   
aa-genprof /root/add_data.sh

#apparmor module status    
aa-status

#load profile file           
apparmor_parser -q /etc/apparmor.d/usr.sbin.nginx
```

```bash
$ aa-status
apparmor module is loaded.
8 profiles are loaded.
8 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/curl
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/sbin/tcpdump
   docker-default
   docker-nginx
0 profiles are in complain mode.
10 processes have profiles defined.
10 processes are in enforce mode.
   docker-default (2136) 
   docker-default (2154) 
   docker-default (2165) 
   docker-default (2181) 
   docker-default (2343) 
   docker-default (2399) 
   docker-default (2461) 
   docker-default (2507) 
   docker-default (2927) 
   docker-default (2959) 
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.

```

##  10. Restrict a Container's Syscalls with seccomp
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa7524504c204904a9b3658706c8b157.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/451e81fb522941a2ab8295f2456341e3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6b454ca667684d268c5f86b78db3eb55.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c6f589eeb9a4216a4c17713c1ab8822.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/61329487bfad4f0bab10af82a846a56a.png)
k create deployment web --image nginx --replicas 1
k get po
k 


## 11. Pod Security Policies (PodSecurityPolicy)
 ![](https://img-blog.csdnimg.cn/48181d8e5eee465e97585a86c2a4b4d4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/fbf1ee13e4e048a0bcd84b016eb6007a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/741d04cda7334e94a713c0df904ae6f4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/004f698387c14eccb12d17e9a573e7e4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/64c4d20af1714a2986f4ce7206324655.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/19933086571243708393a005081c33e4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/986cc87c645345f796d3e8a5865067e5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0556c8d4645642aaa21cdef116486032.png)

```bash
k create ns dev
k create sa psp-test-sa -n dev
```
创建[PodSecurityPolicy](https://kubernetes.io/zh-cn/docs/concepts/security/pod-security-policy/#create-a-policy-and-a-pod)

```bash
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: dev-psp
spec:
  privileged: false  # 不允许提权的 Pod！
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'

```

```bash
$ k apply -f psp.yaml 

$ k get psp -A
NAME      PRIV    CAPS   SELINUX    RUNASUSER   FSGROUP    SUPGROUP   READONLYROOTFS   VOLUMES
dev-psp   false          RunAsAny   RunAsAny    RunAsAny   RunAsAny   false            *


$ k describe psp dev-psp
Name:         dev-psp
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  policy/v1beta1
Kind:         PodSecurityPolicy
Metadata:
  Creation Timestamp:  2022-09-01T08:40:58Z
  Managed Fields:
    API Version:  policy/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        f:allowPrivilegeEscalation:
        f:fsGroup:
          f:rule:
        f:runAsUser:
          f:rule:
        f:seLinux:
          f:rule:
        f:supplementalGroups:
          f:rule:
        f:volumes:
    Manager:         kubectl-client-side-apply
    Operation:       Update
    Time:            2022-09-01T08:40:58Z
  Resource Version:  308516
  UID:               fc6c4ea9-ec94-4217-aec5-b662d89b986b
Spec:
  Allow Privilege Escalation:  true
  Fs Group:
    Rule:  RunAsAny
  Run As User:
    Rule:  RunAsAny
  Se Linux:
    Rule:  RunAsAny
  Supplemental Groups:
    Rule:  RunAsAny
  Volumes:
    *
Events:  <none>

```

```bash
k create clusterrole dev-cr --verb=use --resource=PodSecurityPolicy --resource-name=dev-psp

k create rolebinding dev-binding -n dev --role=dev-cr --serviceaccount=dev:dev-sa
```

##  12. Open Policy Agent - OPA Gatekeeper
![在这里插入图片描述](https://img-blog.csdnimg.cn/30d22f0841a444e091cd1fc233ab452f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1fd718c6e0f54775b35593f8dd328ab3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/38c5657df432424b94873c054ec9cfc0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5f05eb253f83463e8a53a744584b336e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf0d475cf3a947bd8842f904c291061d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f0d3843464da4445aaf9bb5fb4987671.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2db0d6dd6d74545b18611e42b4e557b.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/11a68239f2054539bbc1d4042b5f125b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/80335c5625f94377918e92e269e72f46.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/687ba6f3e2dd4a67b84a11fae7b586c3.png)

```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.7/deploy/gatekeeper.yaml

helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts --force-update
helm install gatekeeper/gatekeeper --name-template=gatekeeper --namespace gatekeeper-system --create-namespace
```

```bash
$ k apply -f gatekeeper.yaml 
namespace/gatekeeper-system created
resourcequota/gatekeeper-critical-pods created
customresourcedefinition.apiextensions.k8s.io/configs.config.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constraintpodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplatepodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplates.templates.gatekeeper.sh created
serviceaccount/gatekeeper-admin created
podsecuritypolicy.policy/gatekeeper-admin created
role.rbac.authorization.k8s.io/gatekeeper-manager-role created
clusterrole.rbac.authorization.k8s.io/gatekeeper-manager-role created
rolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
secret/gatekeeper-webhook-server-cert created
service/gatekeeper-webhook-service created
deployment.apps/gatekeeper-audit created
deployment.apps/gatekeeper-controller-manager created
poddisruptionbudget.policy/gatekeeper-controller-manager created
validatingwebhookconfiguration.admissionregistration.k8s.io/gatekeeper-validating-webhook-configuration created

```

```bash
$ k get all -n gatekeeper-system
NAME                                                READY   STATUS    RESTARTS   AGE
pod/gatekeeper-audit-85cb65679d-dxksb               1/1     Running   0          87s
pod/gatekeeper-controller-manager-57f4675cb-js5wh   1/1     Running   0          87s
pod/gatekeeper-controller-manager-57f4675cb-rh2lw   1/1     Running   0          87s
pod/gatekeeper-controller-manager-57f4675cb-z994p   1/1     Running   0          87s

NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/gatekeeper-webhook-service   ClusterIP   10.102.147.19   <none>        443/TCP   87s

NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/gatekeeper-audit                1/1     1            1           87s
deployment.apps/gatekeeper-controller-manager   3/3     3            3           87s

NAME                                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/gatekeeper-audit-85cb65679d               1         1         1       87s
replicaset.apps/gatekeeper-controller-manager-57f4675cb   3         3         3       87s

```
k8srequiredlabels_template.yaml
```bash
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          type: object
          properties:
            labels:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }

```

```bash
k apply -f k8srequiredlabels_template.yaml
```

```bash
$ k api-resources |grep crd
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
bgpconfigurations                              crd.projectcalico.org/v1               false        BGPConfiguration
bgppeers                                       crd.projectcalico.org/v1               false        BGPPeer
blockaffinities                                crd.projectcalico.org/v1               false        BlockAffinity
clusterinformations                            crd.projectcalico.org/v1               false        ClusterInformation
felixconfigurations                            crd.projectcalico.org/v1               false        FelixConfiguration
globalnetworkpolicies                          crd.projectcalico.org/v1               false        GlobalNetworkPolicy
globalnetworksets                              crd.projectcalico.org/v1               false        GlobalNetworkSet
hostendpoints                                  crd.projectcalico.org/v1               false        HostEndpoint
ipamblocks                                     crd.projectcalico.org/v1               false        IPAMBlock
ipamconfigs                                    crd.projectcalico.org/v1               false        IPAMConfig
ipamhandles                                    crd.projectcalico.org/v1               false        IPAMHandle
ippools                                        crd.projectcalico.org/v1               false        IPPool
kubecontrollersconfigurations                  crd.projectcalico.org/v1               false        KubeControllersConfiguration
networkpolicies                                crd.projectcalico.org/v1               true         NetworkPolicy
networksets                                    crd.projectcalico.org/v1               true         NetworkSet


$  k api-resources
...........
constrainttemplates                            templates.gatekeeper.sh/v1beta1        false        ConstraintTemplate


$ k get crd
NAME                                                  CREATED AT
bgpconfigurations.crd.projectcalico.org               2022-07-27T08:23:44Z
bgppeers.crd.projectcalico.org                        2022-07-27T08:23:44Z
blockaffinities.crd.projectcalico.org                 2022-07-27T08:23:44Z
clusterinformations.crd.projectcalico.org             2022-07-27T08:23:44Z
configs.config.gatekeeper.sh                          2022-09-01T09:14:37Z
constraintpodstatuses.status.gatekeeper.sh            2022-09-01T09:14:37Z
constrainttemplatepodstatuses.status.gatekeeper.sh    2022-09-01T09:14:37Z
constrainttemplates.templates.gatekeeper.sh           2022-09-01T09:14:37Z
felixconfigurations.crd.projectcalico.org             2022-07-27T08:23:44Z
globalnetworkpolicies.crd.projectcalico.org           2022-07-27T08:23:44Z
globalnetworksets.crd.projectcalico.org               2022-07-27T08:23:44Z
hostendpoints.crd.projectcalico.org                   2022-07-27T08:23:44Z
ipamblocks.crd.projectcalico.org                      2022-07-27T08:23:44Z
ipamconfigs.crd.projectcalico.org                     2022-07-27T08:23:44Z
ipamhandles.crd.projectcalico.org                     2022-07-27T08:23:44Z
ippools.crd.projectcalico.org                         2022-07-27T08:23:44Z
k8srequiredlabels.constraints.gatekeeper.sh           2022-09-01T09:24:54Z
kubecontrollersconfigurations.crd.projectcalico.org   2022-07-27T08:23:44Z
networkpolicies.crd.projectcalico.org                 2022-07-27T08:23:44Z
networksets.crd.projectcalico.org                     2022-07-27T08:23:44Z



$ k get constrainttemplates
NAME                AGE
k8srequiredlabels   4m18s

```

```bash
$ k describe constrainttemplates k8srequiredlabels 
Name:         k8srequiredlabels
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  templates.gatekeeper.sh/v1beta1
Kind:         ConstraintTemplate
Metadata:
  Creation Timestamp:  2022-09-01T09:24:54Z
  Generation:          1
  Managed Fields:
    API Version:  templates.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:crd:
          .:
          f:spec:
            .:
            f:names:
              .:
              f:kind:
            f:validation:
              .:
              f:openAPIV3Schema:
                .:
                f:properties:
                f:type:
        f:targets:
    Manager:      kubectl-client-side-apply
    Operation:    Update
    Time:         2022-09-01T09:24:54Z
    API Version:  templates.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        .:
        f:byPod:
        f:created:
    Manager:         gatekeeper
    Operation:       Update
    Time:            2022-09-01T09:24:55Z
  Resource Version:  312752
  UID:               6b85c61b-a1d9-403d-851c-bbe8b3e8ef7c
Spec:
  Crd:
    Spec:
      Names:
        Kind:  K8sRequiredLabels
      Validation:
        openAPIV3Schema:
          Properties:
            Labels:
              Items:
                Type:  string
              Type:    array
          Type:        object
  Targets:
    Rego:  package k8srequiredlabels

violation[{"msg": msg, "details": {"missing_labels": missing}}] {
  provided := {label | input.review.object.metadata.labels[label]}
  required := {label | label := input.parameters.labels[_]}
  missing := required - provided
  count(missing) > 0
  msg := sprintf("you must provide labels: %v", [missing])
}

    Target:  admission.k8s.gatekeeper.sh
Status:
  By Pod:
    Id:                   gatekeeper-audit-85cb65679d-dxksb
    Observed Generation:  1
    Operations:
      audit
      status
    Template UID:         6b85c61b-a1d9-403d-851c-bbe8b3e8ef7c
    Id:                   gatekeeper-controller-manager-57f4675cb-js5wh
    Observed Generation:  1
    Operations:
      webhook
    Template UID:         6b85c61b-a1d9-403d-851c-bbe8b3e8ef7c
    Id:                   gatekeeper-controller-manager-57f4675cb-rh2lw
    Observed Generation:  1
    Operations:
      webhook
    Template UID:         6b85c61b-a1d9-403d-851c-bbe8b3e8ef7c
    Id:                   gatekeeper-controller-manager-57f4675cb-z994p
    Observed Generation:  1
    Operations:
      webhook
    Template UID:  6b85c61b-a1d9-403d-851c-bbe8b3e8ef7c
  Created:         true
Events:            <none>

```
constraint.yaml  ,要求`Deployment`部署必须有`labels: env`
```bash
$ cat constraint_2.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: deploy-must-have-env
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    labels: ["env"]

```

```bash
k apply -f constraint.yaml
```
参考：[Deployments](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)

deployement.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
```

没有配置`env`标签就会报错
```bash
k apply -f deployment.yaml 
Error from server ([deploy-must-have-env] you must provide labels: {"env"}): error when creating "deployment.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [deploy-must-have-env] you must provide labels: {"env"}

```
修改deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    env: demo
spec:
  replicas: 1
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

```
执行：
```bash
$ k apply -f deployment.yaml 
deployment.apps/nginx-deployment created

$ k get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           39s

```
如果创建namespace必须有`labels: env`
k8srequiredlabels_template_ns.yaml
```bash
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-gk
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["gatekeeper"]
```

```bash
$ k apply -f k8srequiredlabels_template_ns.yaml
$ k create ns test
Error from server ([ns-must-have-gk] you must provide labels: {"gatekeeper"}): admission webhook "validation.gatekeeper.sh" denied the request: [ns-must-have-gk] you must provide labels: {"gatekeeper"}
```
如何正确创建

```bash
$ k create ns test --dry-run=client -oyaml > test_ns.yaml
$  vim test_ns.yaml 
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: test
  labels:   #添加
    gatekeeper: test   #添加
spec: {}
status: {}

#创建ns成功
$ k apply -f test_ns.yaml 
namespace/test created

$ k get ns |grep test
test                   Active   47s

```

##  13. Security Context for a Pod or Container
![在这里插入图片描述](https://img-blog.csdnimg.cn/78c225b1e9e044adaf5c79142b540093.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b20738f2ad37468aa4ef6d7d86b4d411.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/591069f783d64e2e80ca1e2951356177.png)

```bash
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false

```

```bash
kubectl apply -f https://k8s.io/examples/pods/security/security-context.yaml
kubectl get pod security-context-demo
kubectl exec -it security-context-demo -- sh
# id
uid=1000 gid=3000 groups=2000
```

##  14.  Manage Kubernetes secrets
![在这里插入图片描述](https://img-blog.csdnimg.cn/f65cdd08e8d04d46bb675daadb52d7ac.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4e62aebf95204f5d9b10b9825d505b69.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d919f01b35a4bf9b33ba196a3aeaf64.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/90ed140a93f648b697001f6c89465bd9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/94552cfde4f84614bd4e66cd3cb41efe.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/578d79eb90824abfa0a5ebf690aeaebe.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1926be344c2e4f33a9fa15f5e4a7df49.png)


 - Opaque(Generic) secrets -
 - Service account token Secrets
 - Docker config Secrets
    - `kubectl create secret docker-registry my-secret --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL`
 - Basic authentication Secret -
 - SSH authentication secrets -
 - TLS secrets -
     - `kubectl create secret tls tls-secret --cert=path/to/tls.cert --key=path/to/tls.key`
 - Bootstrap token Secrets -


Secret as Data to a Container Using a Volume

```bash
kubectl create secret generic mysecret --from-literal=username=devuser --from-literal=password='S!B\*d$zDsb='
```
Decode Secret

```bash
kubectl get secrets/mysecret --template={{.data.password}} | base64 -d
```

```bash
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secret-volume"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: mysecret
```
Secret as Data to a Container Using Environment Variables

```bash
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    # refer all secret data
    envFrom:
      - secretRef:
        name: mysecret-2
    # refer specific variable
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
```
Ref: [https://kubernetes.io/docs/concepts/configuration/secret](https://kubernetes.io/docs/concepts/configuration/secret)

Ref: [https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data)

Ref: [https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/)


##  15. Container  Runtime Sandboxes gVisor runsc kata containers
![在这里插入图片描述](https://img-blog.csdnimg.cn/d526247c0b7441e0ac8a624b7adcc0ae.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/635c18128b894cc58ee1a15f47769f8f.png)



 - 沙盒是容器与主机隔离的概念
 - docker 使用默认的 SecComp 配置文件来限制特权。白名单或黑名单
 - AppArmor 可以访问该容器的细粒度控制。白名单或黑名单
 - 如果容器中有大量应用程序，那么 SecComp/AppArmor 就不是这样了

###  15.1gvisor
gVisor 位于容器和 Linux 内核之间。每个容器都有自己的 `gVisior`

gVisor 有 2 个不同的组件

 - `Sentry` : 作为容器的内核
 - `Gofer`：是访问系统文件的文件代理。容器和操作系统之间的中间人

gVisor 在 hostOS 中使用 runsc 来运行沙箱（OCI 兼容）

```bash
apiVersion: node.k8s.io/v1  # RuntimeClass is defined in the node.k8s.io API group
kind: RuntimeClass
metadata:
  name: myclass  # The name the RuntimeClass will be referenced by
handler: runsc  # non-namespaced, The name of the corresponding CRI configuration
```
### 15.2 kata 容器
Kata 在 VM 中安装轻量级容器，为容器（如 VM）提供自己的内核

Kata 容器提供硬件虚拟化支持（不能在云端运行，GCP 支持）

Kata 容器使用 kata 运行时。

```bash
apiVersion: node.k8s.io/v1  # RuntimeClass is defined in the node.k8s.io API group
kind: RuntimeClass
metadata:
  name: myclass  # The name the RuntimeClass will be referenced by
handler: kata  # non-namespaced, The name of the corresponding CRI configuration
```
### 15.3 安装 gVisor

```bash
curl -fsSL https://gvisor.dev/archive.key | sudo gpg --dearmor -o /usr/share/keyrings/gvisor-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/gvisor-archive-keyring.gpg] https://storage.googleapis.com/gvisor/releases release main" | sudo tee /etc/apt/sources.list.d/gvisor.list > /dev/null

sudo add-apt-repository "deb [arch=amd64,arm64] https://storage.googleapis.com/gvisor/releases release main"

sudo apt-get update && sudo apt-get install -y runsc

cat <<EOF | sudo tee /etc/containerd/config.toml
version = 2
[plugins."io.containerd.runtime.v1.linux"]
  shim_debug = true
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  runtime_type = "io.containerd.runc.v2"
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
  runtime_type = "io.containerd.runsc.v1"
EOF

sudo systemctl restart containerd

echo 0 > /proc/sys/kernel/dmesg_restrict
echo "kernel.dmesg_restrict=0" >> /etc/sysctl.conf

{
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.13.0/crictl-v1.13.0-linux-amd64.tar.gz
tar xf crictl-v1.13.0-linux-amd64.tar.gz
sudo mv crictl /usr/local/bin
}

cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
EOF
```

##  16. mTLS实现pod到pod加密
![在这里插入图片描述](https://img-blog.csdnimg.cn/c1810796a3a744df98c37a4a1efff554.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/70076e56c2424a3cbdacd0323eca2e40.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d22cbc0e28f84532a0627668ad8f5f97.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7fdf0f7830904b4099f1644abf0b2743.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b3893dc819c9426eb345c45253e7294d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ff20fbc47093482ea5e5d4958c75329c.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6bd28f3d144540c09dbabd53986b38ab.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3203a05b210046288a46183549963ffe.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/821cba15ab1344078a44ea05c491db17.png)




mTLS：Pod 之间的安全通信
使用服务网格 Istio 和 Linkerd，mTLS 更容易、更易于管理
mTLS 可以强制或严格
Istio：[https ://istio.io/latest/docs/tasks/security/authentication/mtls-migration/](https://istio.io/latest/docs/tasks/security/authentication/mtls-migration/)
 Linkerd： [https ://linkerd.io/2.11/features/automatic-mtls/](https://linkerd.io/2.11/features/automatic-mtls/)
文章：[https ://itnext.io/how-to-secure-kubernetes-in-cluster-communication-5a9927be415b](https://itnext.io/how-to-secure-kubernetes-in-cluster-communication-5a9927be415b)

通过 DNS 访问的 Kubernetes 服务的 TLS 证书步骤

 1. 下载并安装 CFSSL
 2. 使用生成私钥cfssl genkey
 3. 创建 CertificateSigningRequest

```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: my-svc.my-namespace
spec:
  request: $(cat server.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kubelet-serving
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
```
获得 CSR 批准（由 k8s 管理员）

```bash
kubectl certificate approve my-svc.my-namespace
```

一旦获得批准，然后从 status.certificate 中检索

```bash
kubectl get csr my-svc.my-namespace -o jsonpath='{.status.certificate}' | base64 --decode > server.crt
```

下载并使用它

```bash
kubectl get csr
```
参考：[https ://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

##  17. 最小化基本镜像(images)占用空间

![在这里插入图片描述](https://img-blog.csdnimg.cn/f803861163c94474a263740b8ff6ecfb.png)
使用Slim/Minimal图像而不是基本图像
使用 Docker多阶段构建实现精益
使用无发行版：
Distroless Images 将只有您的应用程序和运行时依赖项
没有包管理器、shell、n/w 工具、文本编辑器等
Distroless 镜像非常小
使用trivy图像扫描仪检测容器漏洞、filesys、git 等
参考：[https ://github.com/GoogleContainerTools/distroless](https://github.com/GoogleContainerTools/distroless)
参考：[https ://github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)


## 18. 允许Registries使用ImagePolicyWebhook

![在这里插入图片描述](https://img-blog.csdnimg.cn/d6fa17391c104f86a5231eb677ac0df8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ebaa96921a84a5cb21d28b1cda0ccef.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c194e75cf52e4311aa058ed1cf708bb0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8df3fd50c2d48e5b6b74724e1c027e8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/56eaca32a95245a498eea3feea537ab3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/57aff4d8c5a749c28cafd7bfaa1ddd9c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2071db2013843ce992787ab2a9181ab.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3983be4e6d7249eab5b9c10ad4c8571f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a1462abc23e14a1faa758aa553e6e913.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/244b4eb88d7944a7a97781c4d8623a0b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0aa1b155a52943b48bfa29af53775da1.png)
###  18.1 using ImagePolicyWebhook Admission Controller

```bash
#cfssl
sudo apt update
sudo apt install golang-cfssl
```

```bash
#create CSR to send to KubeAPI
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "image-bouncer-webhook",
    "image-bouncer-webhook.default.svc",
    "image-bouncer-webhook.default.svc.cluster.local",
    "192.168.56.10",
    "10.96.0.0"
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

#create csr request
cat <<EOF | kubectl apply -f -
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

#approver cert
kubectl certificate approve image-bouncer-webhook.default

# download signed server.crt
kubectl get csr image-bouncer-webhook.default -o jsonpath='{.status.certificate}' | base64 --decode > server.crt

mkdir -p /etc/kubernetes/pki/webhook/

#copy to /etc/kubernetes/pki/webhook
cp server.crt /etc/kubernetes/pki/webhook/server.crt

# create secret with signed server.crt
kubectl create secret tls tls-image-bouncer-webhook --key server-key.pem --cert server.crt
```
###  18.2 Using ImagePolicyWebhook Admission webhook server (deployment & service)

```bash
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
            - "--registry-whitelist=docker.io,gcr.io"
          volumeMounts:
            - name: tls
              mountPath: /etc/admission-controller/tls
      volumes:
        - name: tls
          secret:
            secretName: tls-image-bouncer-webhook
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: image-bouncer-webhook
  name: image-bouncer-webhook
spec:
  type: NodePort
  ports:
    - name: https
      port: 443
      targetPort: 1323
      protocol: "TCP"
      nodePort: 30020
  selector:
    app: image-bouncer-webhook
```
dd this to resolve service `echo "127.0.0.1 image-bouncer-webhook" >> /etc/hosts`

check service using `telnet image-bouncer-webhook 30020` or `netstat -na | grep 30020`

Create custom kubeconfig with above service, its client certificate

/etc/kubernetes/pki/webhook/admission_kube_config.yaml

```bash
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/webhook/server.crt
    server: https://image-bouncer-webhook:30020/image_policy
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
创建 `ImagePolicyWebhook AdmissionConfiguration` 文件（json/yaml），更新自定义 `kubeconfig` 文件在
/etc/kubernetes/pki/admission_config.json

```bash
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: /etc/kubernetes/pki/webhook/admission_kube_config.yaml
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: false
```
在 kubeapi 服务器配置的 `enable-admission-plugins` 中启用 `ImagePolicyWebhook`

更新 kube api 服务器 `admission-control-config-file` 中的 `admin-config` 文件

 - `/etc/kubernetes/manifests/kube-apiserver.yaml`

```bash
- --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
- --admission-control-config-file=/etc/kubernetes/pki/admission_config.json
```

```bash
apiVersion: v1
kind: Pod
metadata:
  name: dh-busybox
spec:
  restartPolicy: Never
  containers:
  - name: busybox
    image: docker.io/library/busybox
    #image: gcr.io/google-containers/busybox:1.27
    command: ['sh', '-c', 'sleep 3600']
```
参考：[https ://stackoverflow.com/questions/54463125/how-to-reject-docker-registries-in-kubernetes](https://stackoverflow.com/questions/54463125/how-to-reject-docker-registries-in-kubernetes)
参考：[https ://github.com/containerd/cri/blob/master/docs/registry.md](https://github.com/containerd/cri/blob/master/docs/registry.md)
参考：[https ://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#imagepolicywebhook)
参考：[https ://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/](https://kubernetes.io/blog/2019/03/21/a-guide-to-kubernetes-admission-controllers/)
参考：[https ://computingforgeeks.com/how-to-install-cloudflare-cfssl-on-linux-macos/](https://computingforgeeks.com/how-to-install-cloudflare-cfssl-on-linux-macos/)


![在这里插入图片描述](https://img-blog.csdnimg.cn/562de71e6daf4a17b0bda0cae1ee2c4f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/642ae4e77f8d4250a67efd0e79e946fa.png)
##  19. 对镜像(images)进行签名和验证
![在这里插入图片描述](https://img-blog.csdnimg.cn/5363cb6954a042bda03709bb13847417.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5d9456701804432b94efba4c05193ae8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b5b2f3db3c7a44caa5df57e4b4faf1c8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f7af8c9b23224492a97f889613ccbc50.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2bb82f96e04c4f129b1d53a5ea080e4e.png)

##  20. Static analysis using Kubesec, Kube-score, KubeLinter, Checkov & KubeHunter

![在这里插入图片描述](https://img-blog.csdnimg.cn/813d35094e4946f38fbed4461484122d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ac2716e9762d45ddb860a35b79527a5c.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/56caf1f643f745fa8d3ca978d69ba4a2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/dbbfd5ee18ef45d0bbd9b493e20ba94f.png)

###  20.1 kubesec
![在这里插入图片描述](https://img-blog.csdnimg.cn/97c6d5d2bab844498dcb01e79e011cb7.png)

 - [Docker container image](https://hub.docker.com/r/kubesec/kubesec/tags) at `docker.io/kubesec/kubesec:v2`
 - Linux/MacOS/Win binary ([get the latest release](https://github.com/controlplaneio/kubesec/releases))
 - [Kubernetes Admission Controller](https://github.com/controlplaneio/kubesec-webhook)
 - [Kubectl plugin](https://github.com/controlplaneio/kubectl-kubesec)

本地静态分析
```bash
$ cat <<EOF > kubesec-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubesec-demo
spec:
  containers:
  - name: kubesec-demo
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      readOnlyRootFilesystem: true
EOF
$ kubesec scan kubesec-test.yaml
```
docker 方法：

```bash
docker run -i kubesec/kubesec:512c5e0 scan /dev/stdin < kubesec-test.yaml
```
Kubesec HTTP Server

```bash
$ kubesec http 8080 &
[1] 12345
{"severity":"info","timestamp":"2019-05-12T11:58:34.662+0100","caller":"server/server.go:69","message":"Starting HTTP server on port 8080"}
```

```bash
$ curl -sSX POST --data-binary @test/asset/score-0-cap-sys-admin.yml http://localhost:8080/scan
[
  {
    "object": "Pod/security-context-demo.default",
    "valid": true,
    "message": "Failed with a score of -30 points",
    "score": -30,
    "scoring": {
      "critical": [
        {
          "selector": "containers[] .securityContext .capabilities .add == SYS_ADMIN",
          "reason": "CAP_SYS_ADMIN is the most privileged capability and should always be avoided"
        },
        {
          "selector": "containers[] .securityContext .runAsNonRoot == true",
          "reason": "Force the running image to run as a non-root user to ensure least privilege"
        },
  // ...
```
### 20.2 kube-score

```bash
brew install kube-score
kube-score score pod.yaml
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/573cfeb58b85401bb295e685db09029e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2dfe6f33a18745f985afc85ba09553a1.png)

###  20.3 checkov
![在这里插入图片描述](https://img-blog.csdnimg.cn/e54d6589cf8644e5acb27f4067f19940.png)

###  20.4 kube-linter
![在这里插入图片描述](https://img-blog.csdnimg.cn/d190dabfaf6c49e1a01cf6b337d93d69.png)

### 20.5 kube-hunter
![在这里插入图片描述](https://img-blog.csdnimg.cn/aa71681650c84075acd51be0377528a6.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/8d83b21d8099491e86b128b46475beb2.png)


##  21. 扫描镜像查找已知漏洞
### 21.1 Trivy 
![在这里插入图片描述](https://img-blog.csdnimg.cn/6fa517ce98134944ae624c0b262bac63.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/63861593037a4563972b004bc5690637.png)
###  21.2 Anchore CLI
![在这里插入图片描述](https://img-blog.csdnimg.cn/99cceae39bae4e3c8cb73a607a4e07f4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/49f32b3c7ca04d699a58740966d60bb6.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7972ccb0f12b47a197c9fc21770f2236.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8109444aeb8d412381e7f50fe19697ce.png)

Anchore CLI in mac using Docker Compose command
```bash
curl -O https://engine.anchore.io/docs/quickstart/docker-compose.yaml
docker-compose up -d
docker-compose ps
docker-compose exec api anchore-cli system status
docker-compose exec api anchore-cli system feeds list
docker-compose exec api anchore-cli system wait 
docker-compose exec api anchore-cli image add nginx:1.19.0
docker-compose exec api anchore-cli image wait nginx:1.19.0
docker-compose exec api anchore-cli image content nginx:1.19.0 os
docker-compose exec api anchore-cli image vuln  nginx:1.19.0 all
docker-compose exec api anchore-cli evaluate check nginx:1.19.0
```
Anchore CLI in mac using Docker Compose with Anchore CLI commadn

```bash
curl https://engine.anchore.io/docs/quickstart/docker-compose.yaml | docker-compose -p anchore -f - up -d
docker run --rm -e ANCHORE_CLI_URL=http://anchore_engine-api_1:8228/v1/ --network anchore_default -it anchore/engine-cli
anchore-cli --version
anchore-cli image add nginx:1.19.0 && anchore-cli image wait nginx:1.19.0
anchore-cli image vuln nginx:1.19.0 all
anchore-cli image evaluate check nginx:1.19.0 all
```
### 21.3 Clair

```bash
docker run -p 5432:5432 -d --name db arminc/clair-db:latest
docker run -p 6060:6060 --link db:postgres -d --name clair arminc/clair-local-scan:latest
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
Ref: [https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#10-scan-images-and-run-ids](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/#10-scan-images-and-run-ids)
Ref: [https://github.com/aquasecurity/trivy](https://github.com/aquasecurity/trivy)
Ref: [https://github.com/anchore/anchore-cli#command-line-examples](https://github.com/anchore/anchore-cli#command-line-examples)
Ref: [https://github.com/linuxacademy/content-cks-trivy-k8s-webhook](https://github.com/linuxacademy/content-cks-trivy-k8s-webhook)
Ref: https://project-serum.github.io/anchor/getting-started/installation.html#install-yarn
Ref: [https://docs.anchore.com/3.0/docs/installation/](https://docs.anchore.com/3.0/docs/installation/)
[https://githubhelp.com/anchore/anchore-engine](https://githubhelp.com/anchore/anchore-engine)


##  22. 用Falco & Slack警报检测系统调用等文件恶意活动
![在这里插入图片描述](https://img-blog.csdnimg.cn/610d9e0ba47d43198b4f06f0165f3f10.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/386aeaa0e5ed4f3e8e6170c28429577c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f3a317e0eec549f0894f35b9f593d9a1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ef2bfbf5250c48ee8251418941195ead.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/22e57dc3b77e4c7eba0ed330a3aa4b66.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b99f99bb2a034c8bb7b92772e4e4c300.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/3f9c0f2332054e689d22bec4b283fb3e.png)



alco 可以检测任何涉及进行 Linux 系统调用的行为并发出警报

Falco在用户空间和内核空间运行，主要组件...

 - Driver
 - Policy Engine
 - Libraries
 - Falco Rules


##  23. Ensure immutability of containers at runtime
![在这里插入图片描述](https://img-blog.csdnimg.cn/6fff952e6b5346ae90f6ddc317e5edad.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/305c6611ee794138bfb126c49e8ac77b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1c59907fe0fe40a89bf75d0ff5bb814d.png)

##  24. 使用 Audit Logs 监控访问情况
![在这里插入图片描述](https://img-blog.csdnimg.cn/9149711961e74e689a76db490a3cccfb.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/03c46e63a618411d828cb30461f96606.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fc0bc4e098b7459bb65daea493d9e725.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8ab69b9cbd349a2b779d20fa769aa5c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ca9c3cda8b8b429fab612cf342659e58.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f1230dd980945e0bee8c581b353b95a.png)


##  25. Pod安全准入控制- PodSecurityPolicy

![在这里插入图片描述](https://img-blog.csdnimg.cn/e1b34ba6a6c341e48af9ded4d9826674.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/672a9900b1d1477188bc9a15fee7dcf2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/774a9306c9f744fe94192776ba380c67.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b8d4015662f443e2ab8661ac93ca6335.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6ee24b1847714cdb9cb8d94af90bde71.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e3f5ec4e4e8340cb9bea854a597a8d9e.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2b682eab6b014a6ea0ee7895ac392d2c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/52409f051e124789a2734dabf45d7bc3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fdf316e187584f4c96cb1c088ebb3da6.png)
###  enforce
![在这里插入图片描述](https://img-blog.csdnimg.cn/d3af3813ba8246aa85e63831e9c2f1b8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0a042fe163c49d68054ad04abbbb5ef.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/d578500a27b74597958f797b8a1be025.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b4be3f7b30ab43829d65ae9e129a6757.png)
###  warn
![在这里插入图片描述](https://img-blog.csdnimg.cn/036603ca5dbf440bbaf99250304bdef1.png)
###  audit
![在这里插入图片描述](https://img-blog.csdnimg.cn/10175e62f95045b88cd14c20612b1a2a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0c32f6d14ae04e829c8c530abd82024f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/25cc636d9c7544fc908d2d5cfd47cee2.png)

##  26.  CKS Exam Experience, Topics to Focus & Tips
![在这里插入图片描述](https://img-blog.csdnimg.cn/83bdbdc5290640be89c5b71405d517a2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7b25773ee09a4ef6bd7fee34c3d29ab2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b63a4a31556e4f7185853edeb2b3646a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/56589b173a1a4b378c1ad106b099989e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5dcdfc5e741d48169e3243c9c41e5206.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d523fc3fed714fd7aa133f9c5e20c499.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d34cee908ad84bec8baff5274c50704d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2e7615cb1bc84e9cb8ad66e9954ef20e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9905b4ab3bb4c92982e4ac89ec628f1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f7aadeb332084af89ed353444101622b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f758e2aad31e4142bc8293d4b6d3d569.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f4c8eee8665f4db1b5cbadb04ec28735.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9e5e6ab28fb472ab097b4cf8df3f5fb.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9930f90c1cbd41bcbbc8ff3d945154c3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f4a02f17cd9341cebd95ebad5475859c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/de563503328a44678576e0aab0f506a4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5babf904514e47e88d8a227c0ac41efb.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0c84f59fa004575a5fb64f162fa45e1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/74cf16f47c5841eb9212ff496a3c96e0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a103c6c722d145a992c1a7a878271bfc.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9452f5611794ccbac2903c7a97b7ea3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/76b2029effb444d6acaf31b99cc183b9.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/54098d72ee134df8952d80279fb54da8.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4eb31ebd26774336ae1af4b1da9ec034.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9a0605901f994ce780945acb68c04770.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/aaef1fdb6a674d91b36e8e3a85a33cb0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8cc9db44f0354ce6aee57078dc647819.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2b2d03b0f26840b1b1978b97d23eff0d.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/691cc27068ec479683b4da3b0b324828.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/058080a8f66b4c96914751bd7740f37b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6ea35b12715742c9be8340e3896eb3f8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0fb762039a2b4b8d9a9bcfbf99e19b02.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2910657ffeec4465aa0d48884a7c2d34.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2f679710cbeb487ba0de65d5b91009c5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/754ce264cece417792c360e331e1eee3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8360fbcd075e49b9a0251f957a2551be.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/550218898eb245a5b7aaee57e3ecafa5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/606671f1082c4b81876ca0f472e722bc.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5bbc3f23e09341528953bda3f63fef31.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/95c253b536324db8ae41a0680fe15b28.png)

```bash
echo "set ts=2 sw=2 sts=2 expandtab" > ~/.vimrc
```

```bash
export oy="-o yaml"
export now="--force --grace-period=0"
```
使用日志对 api 服务器进行故障排除...

```bash
cd /var/log/pods/
ls -l 
cd kube-system_kube-apiserver-controlplane_xxx
tail -f kube-apiserver/0.log
```
使用 docker 对 api 服务器进行故障排除 `docker ps -a | grep -i kube-api`

命令式命令，打开 yaml，保存/应用

```bash
kubectl run web --image nginx --dry-run=client -oyaml | vim - 
:wq file1.yaml
:wq kubectl apply -f -
```
查找进程并删除

```bash
netstat -atnlp | grep -i 9090 | grep -w -i listen
apt list --installed | grep -i nginx
systemctl list-units --all | grep -i nginx
```
使用查找 kubelet 配置路径

```bash
ps aux | grep -i kubelet | grep -w config
```
使用查找 pod 不变性问题

```bash
k get po --output=custom-columns="NAME:.metadata.name,SEC:spec.securityContext,SEC_CXT:.spec.containers[*].securityContext"
```
使用查找 Falco 规则

```bash
journalctl -fu falco | grep "xyz error"
cat /etc/falco/falco_rules.yaml | grep "xyz error" -A5 -B5
cat /var/log/syslog | grep falco | grep xyz | grep error

# if falco installed as Helm/ds
kubectl logs --selector app=falco | grep Error
```

