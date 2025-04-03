视频：[Kubernetes 安全专家(CKS)](https://space.bilibili.com/400114617/channel/collectiondetail?sid=666460)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ea9393cbaf4e03acae777be9d610a092.png#pic_center)


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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37565d34ee420a9f8b8a89dbb804f9f8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1f80a3ac2a7ada05a4886246d54e0540.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a849b268f65a60064a62c5cd1118547b.png)

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

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cd29dd73a63d43bc4f8bbe4b4a5c5570.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76153b3653ddb933d8ada15ef66a483a.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/070ca623a5d6ffe39656bfe237cabcd1.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c658cd9303a0df51923b0f0050d8f64b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8acf8e0298d76167a1b3ec3b2b2f691.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a806479aad1fbc6d6cc2fc389391d3b.png)
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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/925d3d1deeb2c6fcfe66a6ab498f040a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fd4e1084a538245368e093d89d133ef2.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0bba55d800c083d8d282fef166a8ef23.png)

```bash
userdel user1
groupdel group1
usermod -s /usr/sbin/nologin user2
useradd -d /opt/sam -s /bin/bash -G admin -u 2328 sam
```


###  8.2 remove obsolete/unnecessary software
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/90fd85250334bbc82b49a1bb9483e555.png)

```bash
systemctl list-units --type service
systemctl stop squid
systemctl disable squid
apt remove squid
```

###  8.3 ssh hardening
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/03c68771d4c83dc34b583dfb55f84123.png)

```bash
ssh-keygen -t rsa
cat /home/mark/.ssh/authorized_keys

PermiRootLogin no
PasswordAuthentication no

systemctl restart sshd
```


###  8.4 ldentify & fix open ports
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/81fd25aab261b5a5a474c1594bf9f189.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/98066022faff3ecde9694a9763895c9a.png)

```bash
lsmod
vim /letc/modprobe.d/blacklist.conf
blacklist sctp
blacklist dccp
shutdown -r now
```


###  8.6 restrict allowed hostPath using PSP
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6fff3fb16a5cd22b9b6ad9981fd918f9.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f46e845891d21bd58c86b26d282e4a6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8b58be98d8b6480f837aa85079d01a9a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/575b19cff289ba119b61bb533fe8918c.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/533d5c4c0789496c201f84e26aea9cb7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/06e8a304217ca531f462632d333c2577.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e681a5c2b0715be292dfed984cb5ea86.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fcf5e8037a4b26b48da36d3339498c25.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/91b77255cd527f9059d948da04ed5c51.png)
k create deployment web --image nginx --replicas 1
k get po
k 


## 11. Pod Security Policies (PodSecurityPolicy)
 ![](https://i-blog.csdnimg.cn/blog_migrate/b3d3e7d838530084c25ba47e915315d2.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cee5c0df8620fe14e986099605af15de.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f50f56baa200c6e45727a9beb599464b.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/add0332200d697f0cc4bf9a067a9bd4d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/012a8b696a434df0180df0d31299f420.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/79c8945917a168a718fcac283eb30f6f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3edc040a96903ac38e325a59544aaa07.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/91d7e82825b210b95b5e447de0558fe3.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec5c22c6245160fd68e5300a802c6f18.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c5c78a06fdb0891a146bc734f1772b6a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/404f40ec83e77969798828043792b89a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/41b89f99632ec5c6ac2493aeebf89d80.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5ba458dd46ad8ac1d6ad0a983e142107.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/829bbbff917bcbff3880ff2a55b73f02.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9df195014aa4d29f0ff68cb2f287eee1.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/74197be695916fab2af032bdb28fa31d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f2ed4b676e7c9a4e2296ac936f472f60.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d30e5835dc200ef3661a37866254542f.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/69e8d084a33559d31733dc3b44cc9871.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48450432b20f47bfc5f530ec75277585.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7f65a1bde6afdc2daea184fe2d261b93.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0953ef3976ad8990d8139a3f87839a09.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7ee0860e58fd2d3d218a86e9180e383c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9ddd50efa9c60d09fc21e8f1e234f099.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/92751f2b68e3e0d0bff70f439511d8a3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d65c938b024b0a0c6767be01818e716f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb2f9ec54454c88039b2997e8129d898.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7e97b0daf76354ac906163e8d28d1bec.png)


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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/35b092c3150bf0881dda829afe7603c7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/095309b348b51aec16d6d3d0db6cb794.png)



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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16741d5bfa25f45105a1f684719eea78.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8940d141f04a00f7f8d7054fc3c9e7b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c350c35f11addb3aa9f039f442182a14.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61a2e3722233dd48a8c9da19269d28f6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6ab1e0632ee0ec5197d12bc5d2122569.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f7ed105a63339183fe08e83373656e3.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/35ec026bd58431e8d1cdb14c3d77130d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b875e9068db253872a7f394f46ba6ad3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d528aed5013bf34de828633ab9092dd3.png)




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

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/45a27ed84b3bfa9448b43a0673c58310.png)
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

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ebf62f43aabb6755ac511c78b7f1752f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/09b071f5e89fd329a5c20693c8b45578.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5c1cb9e692d065cdd558c6fb5e9b5c4f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/979169afd9acad8622698c7da8891f4a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a2d3d2b2f50afd363ac9fc21da37f160.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b5e7b6b36d936d214dbc341201f862c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/414bd3e6a83bb6fdc25ef58f41fa17a3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9994a815a0a779d55776ad81a92f6f6a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/597fae28239b336e2004f5f6949debc4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8dc787ebc064b2229422bf8ca2bc9a40.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71de3077688358fc88513e9ab3118655.png)
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


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/776401a17a8c4cb24707890a9a2f953f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/449d4f4695788e4d9111581a720892f8.png)
##  19. 对镜像(images)进行签名和验证
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6216c1296694c2be02090dd810c78198.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fb62dc7d2aa07dd75ba69603e1f596a3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/da5cdd5134b938cdf246f068b2ad1be0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27cdfd394a5ea94194f6ae4e73e7be5c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1c4db3dd769da9e84ef351888aa68999.png)

##  20. Static analysis using Kubesec, Kube-score, KubeLinter, Checkov & KubeHunter

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3ea2ff5f99802876590427df63a8abf1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/10c1474cb5fd52cfa9ee72bc6a6fa9a1.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/819798c357b735084f6c17559aa67480.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0cd18216222ab6e8ad875c4f2c815f0f.png)

###  20.1 kubesec
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ecb825d6b5521967da7fe750345c9e30.png)

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

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f73810a7af8b88c35126c227cc609ad.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5c32ae37cf74dfafb497142df5e8b442.png)

###  20.3 checkov
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c46da22f820d96e2d5705b7d0eeeaee5.png)

###  20.4 kube-linter
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d0225a58e12c26e5523b4034f769a1c9.png)

### 20.5 kube-hunter
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d814e66c87265c1d18c48b3b37af7c3.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1fbbbe47fdc591aec334bb9b1e961f99.png)


##  21. 扫描镜像查找已知漏洞
### 21.1 Trivy 
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a76b8636c9aa7c99b59c34fd3ad58892.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/029f07bcd101d8306afc422646a0e6df.png)
###  21.2 Anchore CLI
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71bf534f67b7daee639c909512255457.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/409e9e8eab5f6c4797fdf4069d0861e0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/97210a31f121133edb0750f504e2a2a8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4ae0a9ba7da85016255ee5ee006bf47b.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7671598b72bba0649a38d0ab6aac6035.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/795b9100df827582ad4925ca5fb4618e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/597f2056f11dba73fbffd309a123f5ea.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a8274839ae44ac1755ff4105eef7241.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e4cb1831fdf75854a146076bf329e2b8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b5fc1fa331bc42c6289292cc332719f.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4982fcc1b7f8099a10638da313f60f25.png)



alco 可以检测任何涉及进行 Linux 系统调用的行为并发出警报

Falco在用户空间和内核空间运行，主要组件...

 - Driver
 - Policy Engine
 - Libraries
 - Falco Rules


##  23. Ensure immutability of containers at runtime
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5759812cb4c77b4c2fa6a57d69673705.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/999968747878f70b38e76e2142a83e57.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/205db680e828bf1b9a33ffeee3cf6a46.png)

##  24. 使用 Audit Logs 监控访问情况
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/92c4d793eebb0ece0a129032ef4d036c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/034e00f67229e58a3d0c33ea577c8adf.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a26eb6ed360d3bd5d3ec197bf574d76.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/386c131e16a782e66cacfd955ca96aa5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a59b3885e7e3482c6a1c6658d013de5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de3feaf8a51579afa890352f33d69e0c.png)


##  25. Pod安全准入控制- PodSecurityPolicy

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1bfce9d8ed79448ee265ec6b90812460.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3fbbbceb0a9d4b467ae7e70a2d5c00d4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c57f9c6fda82f79ddbbde8029c65f05f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c448dfc56b33898935572de7e677da2e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0f03187a4ccd6acfe5d02247efab7a02.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4d01481a07b4ef42e15e371ddfde484.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6d6df9e80dddb87b721280893147b7ef.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bee594b05e5e00f24979e5850b77d175.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4c1b667654963abb372a6dcc6bba791f.png)
###  enforce
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c037ad1a52b19e29c0609c44bb2ab0db.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8a15cc2520bc3cdb23c7a80ecc17b25.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4c16ed02218f0e8b38ee6445ce699f8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1b49e905cfebaec4ed3e72b9d188c787.png)
###  warn
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/976d042e340163c2a01dba90f1f533b3.png)
###  audit
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ade902a13bd907c20e57693ba73ccdfe.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e9fbcf222ee9ca837eda4fe6001d0ff4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3765b16111c2779d175e3bec943f34f0.png)

##  26.  CKS Exam Experience, Topics to Focus & Tips
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75ed65a1fe57d6ae7fbd322d394243e5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e29f497e3d27e659b9b11c9585ac9c95.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/40c869214721c291136dc934e3625e79.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/086c0bef75339458ed6baf67b0bee92f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/263346c9c31a7151f65110f3fb13e712.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/efdc14178cb4ad5413c96057a67b7706.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/68c98203bbd19a33045b53ecb09031d3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ccc20b208e6e614c8641169e97bc2ea5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bbd05152114b2eea7d1898f81fc9df4a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b0dba5181d045cc6083bd0b5b6a976a2.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7f1d588b4732bd63603cbe8fb38a0a77.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44b5515fe552a9e46913bbf267056adc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18039fe618b90d8e5087fc3a47e7d243.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc914623500ea3fdb8d3b21a6d7750e3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c69f3285078faab1a9ed27c3d8cf9fa1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75a5dea3e548b9a75823c16f1af8ad1f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/85bb17aed7af43481c0357adc18766ab.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4e6032a849da3e73eed19c022fd32ea6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/299363863112d6494bb4d8e92e8773d7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18537d3b2c80fa8258b6c9668bdffda9.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/defa6d50e87b544fe65830edbf586204.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3fee94b3648776e0813bf46ad098b273.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8302076d042601d9a2ac19accf056eca.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37391a86fc0b710010524b648cd563f3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a271a1ab0972f00c5fe5f88b03b8e1a5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e6e396b949860dbae900b2a9ca639b85.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e40dfc8fb40ab228fe348de660cee383.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fd61df3f17935e8c8a383f6f2d9937de.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d0455dcea253258a8197cf1d0e0718eb.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4de353e2eb4b041347cba77b2890355e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5a31a99e2d26394aa10584a7322b79da.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4a9fcdc42ac65462d5b145864cf490a6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8a756fab448ba6a61e20e1c44b14db92.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88e70e043d162a6ce8bf64e618efb809.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e43b3404be6fb211eaf3e45bb0cfa39.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e6d882409f66b02d8f539e562be45ecb.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71d93d7ebd548c31c1e8b7fce63fc95f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/daa78dd74096d386c6b27afc1770e044.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f52dd1a2760fead1ea73639383ece311.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2bbc2071d1eb63b2a909480c609f48a2.png)

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

