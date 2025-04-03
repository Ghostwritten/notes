



----
## 1. 介绍
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d33ff661b05a9b1f0599237ccbf73443.png)
**audit logs introduction**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/189432627f98be5e56097a059c367a58.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/20e009d2dbbe3c566c0a1a862e830229.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/70dbbb6ba10cfc1e59d27dc22a0e87fc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f17880a317319660d2364fedaf6336c6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6ca7a584c5719c98216ebdec03e834f8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/db0625be8466ddc5bc311760462ee8d3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a13cbb6f749c7cdef30e7ab4a20d508e.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0f654d2089f3cdd82cc0865920784385.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/20a4ac16ef70ecdddb0e8dbeba1706a5.png)
##  2. Apiserver启用“Audit Logging”
官方链接：
[https://kubernetes.io/docs/tasks/debug-application-cluster/audit/](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a60b283cd05f7101116210f749926dab.png)

```c
root@master:/etc/kubernetes/manifests# mkdir /etc/kubernetes/auditing
root@master:/etc/kubernetes/manifests# mkidr /etc/kubernetes/audit/logs
root@master:/etc/kubernetes/manifests# cat /etc/kubernetes/audit/policy.yaml 
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata


root@master:/etc/kubernetes/manifests# cat kube-apiserver.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.211.40:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml       # add
    - --audit-log-path=/etc/kubernetes/audit/logs/audit.log       # add
    - --audit-log-maxsize=500                                     # add
    - --audit-log-maxbackup=5  
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
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.20.7
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.211.40
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 192.168.211.40
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 192.168.211.40
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/audit      # add
      name: audit                           # add
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:                               # add
      path: /etc/kubernetes/audit           # add
      type: DirectoryOrCreate               # add
    name: audit                             # add
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}



root@master:/etc/kubernetes/manifests# k get pods -n kube-system |grep api
kube-apiserver-master                      1/1     Running   3          5m



root@master:/etc/kubernetes/manifests# tail /etc/kubernetes/audit/logs/audit.log 
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"3ea2f430-108b-4b17-b967-6e26619fda99","stage":"RequestReceived","requestURI":"/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-controller-manager?timeout=10s","verb":"update","user":{"username":"system:kube-controller-manager","groups":["system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kube-controller-manager/v1.20.7 (linux/amd64) kubernetes/132a687/leader-election","objectRef":{"resource":"leases","namespace":"kube-system","name":"kube-controller-manager","apiGroup":"coordination.k8s.io","apiVersion":"v1"},"requestReceivedTimestamp":"2021-05-24T08:28:50.431353Z","stageTimestamp":"2021-05-24T08:28:50.431353Z"}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"f6a3fc3b-96eb-4e2e-9ba2-66d2e345fb8a","stage":"RequestReceived","requestURI":"/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-scheduler?timeout=10s","verb":"update","user":{"username":"system:kube-scheduler","groups":["system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kube-scheduler/v1.20.7 (linux/amd64) kubernetes/132a687/leader-election","objectRef":{"resource":"leases","namespace":"kube-system","name":"kube-scheduler","apiGroup":"coordination.k8s.io","apiVersion":"v1"},"requestReceivedTimestamp":"2021-05-24T08:28:50.435266Z","stageTimestamp":"2021-05-24T08:28:50.435266Z"}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"3ea2f430-108b-4b17-b967-6e26619fda99","stage":"ResponseComplete","requestURI":"/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-controller-manager?timeout=10s","verb":"update","user":{"username":"system:kube-controller-manager","groups":["system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kube-controller-manager/v1.20.7 (linux/amd64) kubernetes/132a687/leader-election","objectRef":{"resource":"leases","namespace":"kube-system","name":"kube-controller-manager","uid":"bcb35dd3-5cb0-4460-99af-dcedb37a6bfa","apiGroup":"coordination.k8s.io","apiVersion":"v1","resourceVersion":"30280"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2021-05-24T08:28:50.431353Z","stageTimestamp":"2021-05-24T08:28:50.438912Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by ClusterRoleBinding \"system:kube-controller-manager\" of ClusterRole \"system:kube-controller-manager\" to User \"system:kube-controller-manager\""}}

```
##  3. 创建Secret 审查 Audit Logs
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88d21b51ceed9ddcd85492f912c8a734.png)

```c
root@master:/etc/kubernetes/manifests# k create secret generic very-secure --from-literal=user=admin
secret/very-secure created
root@master:/etc/kubernetes/manifests# cat /etc/kubernetes/audit/logs/audit.log |grep very-secure | jq .
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "Metadata",
  "auditID": "12831143-4615-4f4c-a443-6b80c946a0b1",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/default/secrets?fieldManager=kubectl-create",
  "verb": "create",
  "user": {
    "username": "kubernetes-admin",
    "groups": [
      "system:masters",
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.211.40"
  ],
  "userAgent": "kubectl/v1.20.2 (linux/amd64) kubernetes/faecb19",
  "objectRef": {
    "resource": "secrets",
    "namespace": "default",
    "name": "very-secure",
    "apiVersion": "v1"
  },
  "responseStatus": {
    "metadata": {},
    "code": 201
  },
  "requestReceivedTimestamp": "2021-05-24T08:30:54.109005Z",
  "stageTimestamp": "2021-05-24T08:30:54.114724Z",
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": ""
  }
}

```
##   4. 创建高级审计（Audit）策略
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d2def023c553d7b8d09daa89b31e635.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c9111815775c92afe24aeac7de055d78.png)

```c
root@master:/etc/kubernetes/audit# cat policy.yaml 
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
- level: Metadata

- level: None
  verbs: ["get","list","watch"]

- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]

- level: RequestResponse

#重启kube-apiserver
root@master:~/imagev1.20.7# mv /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/
root@master:~/imagev1.20.7# ps aux |grep api
root@master:~/imagev1.20.7# mv /etc/kubernetes/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml


root@master:~/imagev1.20.7# ps aux |grep api
root      25311  102 19.6 1165724 399712 ?      Ssl  01:49   0:08 kube-apiserver --audit-policy-file=/etc/kubernetes/audit/policy.yaml --audit-log-path=/etc/kubernetes/audit/logs/audit.log --audit-log-maxsize=500 --audit-log-maxbackup=5 --advertise-address=192.168.211.40 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --insecure-port=0 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key



root@master:/etc/kubernetes/manifests# tail /etc/kubernetes/audit/logs/audit.log | jq .
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "RequestResponse",
  "auditID": "af001aec-e743-40fb-9530-33d78b0b837a",
  "stage": "ResponseComplete",
  "requestURI": "/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-controller-manager?timeout=10s",
  "verb": "update",
  "user": {
    "username": "system:kube-controller-manager",
    "groups": [
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.211.40"
  ],
  "userAgent": "kube-controller-manager/v1.20.7 (linux/amd64) kubernetes/132a687/leader-election",
  "objectRef": {
    "resource": "leases",
    "namespace": "kube-system",
    "name": "kube-controller-manager",
    "uid": "bcb35dd3-5cb0-4460-99af-dcedb37a6bfa",
    "apiGroup": "coordination.k8s.io",
    "apiVersion": "v1",
    "resourceVersion": "33178"
  },
  "responseStatus": {
    "metadata": {},
    "code": 200
  },
  "requestObject": {
    "kind": "Lease",
    "apiVersion": "coordination.k8s.io/v1",
    "metadata": {
      "name": "kube-controller-manager",
      "namespace": "kube-system",
      "uid": "bcb35dd3-5cb0-4460-99af-dcedb37a6bfa",
      "resourceVersion": "33178",
      "creationTimestamp": "2021-05-14T08:37:45Z",
      "managedFields": [
        {
          "manager": "kube-controller-manager",
          "operation": "Update",
          "apiVersion": "coordination.k8s.io/v1",
          "time": "2021-05-14T08:37:45Z",
          "fieldsType": "FieldsV1",
          "fieldsV1": {
            "f:spec": {
              "f:acquireTime": {},
              "f:holderIdentity": {},
              "f:leaseDurationSeconds": {},
              "f:leaseTransitions": {},
              "f:renewTime": {}
            }
          }
        }
      ]
    },
    "spec": {
      "holderIdentity": "master_0b1b6fb7-55d4-4a8b-b560-bc283491de73",
      "leaseDurationSeconds": 15,
      "acquireTime": "2021-05-24T09:05:09.579371Z",
      "renewTime": "2021-05-24T09:05:54.355135Z",
      "leaseTransitions": 7
    }
  },
  "responseObject": {
    "kind": "Lease",
    "apiVersion": "coordination.k8s.io/v1",
    "metadata": {
      "name": "kube-controller-manager",
      "namespace": "kube-system",
      "uid": "bcb35dd3-5cb0-4460-99af-dcedb37a6bfa",
      "resourceVersion": "33179",
      "creationTimestamp": "2021-05-14T08:37:45Z",
      "managedFields": [
        {
          "manager": "kube-controller-manager",
          "operation": "Update",
          "apiVersion": "coordination.k8s.io/v1",
          "time": "2021-05-14T08:37:45Z",
          "fieldsType": "FieldsV1",
          "fieldsV1": {
            "f:spec": {
              "f:acquireTime": {},
              "f:holderIdentity": {},
              "f:leaseDurationSeconds": {},
              "f:leaseTransitions": {},
              "f:renewTime": {}
            }
          }
        }
      ]
    },
    "spec": {
      "holderIdentity": "master_0b1b6fb7-55d4-4a8b-b560-bc283491de73",
      "leaseDurationSeconds": 15,
      "acquireTime": "2021-05-24T09:05:09.579371Z",
      "renewTime": "2021-05-24T09:05:54.355135Z",
      "leaseTransitions": 7
    }
  },
  "requestReceivedTimestamp": "2021-05-24T09:05:54.382234Z",
  "stageTimestamp": "2021-05-24T09:05:54.397800Z",
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": "RBAC: allowed by ClusterRoleBinding \"system:kube-controller-manager\" of ClusterRole \"system:kube-controller-manager\" to User \"system:kube-controller-manager\""
  }
}



```


##  5. 审查API access 历史

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f3ab216c07824d88d11a65d758a098e.png)

```c
root@master:/etc/kubernetes/manifests# k create sa very-crazy-sa
serviceaccount/very-crazy-sa created
root@master:/etc/kubernetes/manifests# k get sa
NAME            SECRETS   AGE
default         1         10d
very-crazy-sa   1         5s
root@master:/etc/kubernetes/manifests# k get secret
NAME                        TYPE                                  DATA   AGE
default-token-4lh26         kubernetes.io/service-account-token   3      10d
very-crazy-sa-token-fr7sw   kubernetes.io/service-account-token   3      46s
very-secure                 Opaque                                1      52m
root@master:/etc/kubernetes/manifests# cat /etc/kubernetes/audit/logs/audit.log |grep very-crazy-sa
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"RequestResponse","auditID":"1332e3c4-63d1-408e-b2d0-95b21774cd2d","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/default/serviceaccounts?fieldManager=kubectl-create","verb":"create","user":{"username":"kubernetes-admin","groups":["system:masters","system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kubectl/v1.20.2 (linux/amd64) kubernetes/faecb19","objectRef":{"resource":"serviceaccounts","namespace":"default","name":"very-crazy-sa","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestObject":{"kind":"ServiceAccount","apiVersion":"v1","metadata":{"name":"very-crazy-sa","creationTimestamp":null}},"responseObject":{"kind":"ServiceAccount","apiVersion":"v1","metadata":{"name":"very-crazy-sa","namespace":"default","uid":"26cc66f7-cf7d-443a-8bb0-b1b9af6bd30a","resourceVersion":"34576","creationTimestamp":"2021-05-24T09:22:24Z"}},"requestReceivedTimestamp":"2021-05-24T09:22:24.643249Z","stageTimestamp":"2021-05-24T09:22:24.652927Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"96c8fea3-0874-46a7-baed-e32c07316529","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/default/secrets","verb":"create","user":{"username":"system:kube-controller-manager","groups":["system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kube-controller-manager/v1.20.7 (linux/amd64) kubernetes/132a687/tokens-controller","objectRef":{"resource":"secrets","namespace":"default","name":"very-crazy-sa-token-fr7sw","apiVersion":"v1"},"responseStatus":{"metadata":{},"code":201},"requestReceivedTimestamp":"2021-05-24T09:22:24.676670Z","stageTimestamp":"2021-05-24T09:22:24.686139Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by ClusterRoleBinding \"system:kube-controller-manager\" of ClusterRole \"system:kube-controller-manager\" to User \"system:kube-controller-manager\""}}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"RequestResponse","auditID":"27737c46-ab58-4b5b-a12c-b3974e7ecbff","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/default/serviceaccounts/very-crazy-sa","verb":"update","user":{"username":"system:kube-controller-manager","groups":["system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kube-controller-manager/v1.20.7 (linux/amd64) kubernetes/132a687/tokens-controller","objectRef":{"resource":"serviceaccounts","namespace":"default","name":"very-crazy-sa","uid":"26cc66f7-cf7d-443a-8bb0-b1b9af6bd30a","apiVersion":"v1","resourceVersion":"34576"},"responseStatus":{"metadata":{},"code":200},"requestObject":{"kind":"ServiceAccount","apiVersion":"v1","metadata":{"name":"very-crazy-sa","namespace":"default","uid":"26cc66f7-cf7d-443a-8bb0-b1b9af6bd30a","resourceVersion":"34576","creationTimestamp":"2021-05-24T09:22:24Z"},"secrets":[{"name":"very-crazy-sa-token-fr7sw"}]},"responseObject":{"kind":"ServiceAccount","apiVersion":"v1","metadata":{"name":"very-crazy-sa","namespace":"default","uid":"26cc66f7-cf7d-443a-8bb0-b1b9af6bd30a","resourceVersion":"34578","creationTimestamp":"2021-05-24T09:22:24Z"},"secrets":[{"name":very-crazy-sa-token-fr7sw"}]},"requestReceivedTimestamp":"2021-05-24T09:22:24.688390Z","stageTimestamp":"2021-05-24T09:22:24.691198Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by ClusterRoleBinding \"system:kube-controller-manager\" of ClusterRole \"system:kube-controller-manager\" to User \"system:kube-controller-manager\""}}
```



```c
root@master:/cks/runtime-security# k run accessor --image=nginx --dry-run=client -oyaml > pod.yaml

root@master:~/cks/runtime-security# vim pod3.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: accessor
  name: accessor
spec:
  serviceAccountName: very-crazy-sa   #添加此行
  containers:
  - image: nginx
    name: accessor
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@master:~/cks/runtime-security# k create -f pod.yaml 
pod/accessor created
root@master:~/cks/runtime-security# k get pod accessor -w
NAME       READY   STATUS              RESTARTS   AGE
accessor   0/1     ContainerCreating   0          11s
accessor   1/1     Running             0          20s


root@master:~/cks/runtime-security# cat /etc/kubernetes/audit/logs/audit.log |grep accessor
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"RequestResponse","auditID":"1025990a-e51c-4c51-9010-6436999a88cb","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/default/pods/accessor/status","verb":"patch","user":{"username":"system:node:node2","groups":["system:nodes","system:authenticated"]},"sourceIPs":["192.168.211.42"],"userAgent":"kubelet/v1.20.1 (linux/amd64) kubernetes/c4d7527","objectRef":{"resource":"pods","namespace":"default","name":"accessor","apiVersion":"v1","subresource":"status"},"responseStatus":{"metadata":{},"code":200},"requestObject":{"metadata":{"uid":"d67a9290-1dc8-4f14-ac84-88e9ae82d2a2"},"status":{"$setElementOrder/conditions":[{"type":"Initialized"},{"type":"Ready"},{"type":"ContainersReady"},{"type":"PodScheduled"}],"conditions":[{"lastTransitionTime":"2021-05-24T09:29:44Z","message":null,"reason":null,"status":"True","type":"Ready"},{"lastTransitionTime":"2021-05-24T09:29:44Z","message":null,"reason":null,"status":"True","type":"ContainersReady"}],"containerStatuses":[{"containerID":"docker://d80411bf54156c9f65c4887c4255dc545b8ebdf518d4d9470f23b0ad3f984b39","image":"nginx:latest","imageID":"docker-pullable://nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2","lastState":{},"name":"accessor","ready":true,"restartCount":0,"started":true,"state":{"running":{"startedAt":"2021-05-24T09:29:43Z"}}}],"phase":"Running","podIP":"10.244.104.11","podIPs":[{"ip":"10.244.104.11"}]}},"responseObject":{"kind":"Pod","apiVersion":"v1","metadata":{"name":"accessor","namespace":"default","uid":"d67a9290-1dc8-4f14-ac84-88e9ae82d2a2","resourceVersion":"35224","creationTimestamp":"2021-05-24T09:29:24Z","labels":{"run":"accessor"},"annotations":{"cni.projectcalico.org/podIP":"10.244.104.11/32","cni.projectcalico.org/podIPs":"10.244.104.11/32"},"managedFields":[{"manager":"kubectl-create","operation":"Update","apiVersion":"v1","time":"2021-05-24T09:29:24Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:labels":{".":{},"f:run":{}}},"f:spec":{"f:containers":{"k:{\"name\":\"accessor\"}":{".":{},"f:image":{},"f:imagePullPolicy":{},"f:name":{},"f:resources":{},"f:terminationMessagePath":{},"f:terminationMessagePolicy":{}}},"f:dnsPolicy":{},"f:enableServiceLinks":{},"f:restartPolicy":{},"f:schedulerName":{},"f:securityContext":{},"f:serviceAccount":{},"f:serviceAccountName":{},"f:terminationGracePeriodSeconds":{}}}},{"manager":"calico","operation":"Update","apiVersion":"v1","time":"2021-05-24T09:29:26Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:annotations":{".":{},"f:cni.projectcalico.org/podIP":{},"f:cni.projectcalico.org/podIPs":{}}}}},{"manager":"kubelet","operation":"Update","apiVersion":"v1","time":"2021-05-24T09:29:44Z","fieldsType":"FieldsV1","fieldsV1":{"f:status":{"f:conditions":{"k:{\"type\":\"ContainersReady\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Initialized\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Ready\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}}},"f:containerStatuses":{},"f:hostIP":{},"f:phase":{},"f:podIP":{},"f:podIPs":{".":{},"k:{\"ip\":\"10.244.104.11\"}":{".":{},"f:ip":{}}},"f:startTime":{}}}}]},"spec":{"volumes":[{"name":"very-crazy-sa-token-fr7sw","secret":{"secretName":"very-crazy-sa-token-fr7sw","defaultMode":420}}],"containers":[{"name":"accessor","image":"nginx","resources":{},"volumeMounts":[{"name":"very-crazy-sa-token-fr7sw","readOnly":true,"mountPath":"/var/run/secrets/kubernetes.io/serviceaccount"}],"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","imagePullPolicy":"Always"}],"restartPolicy":"Always","terminationGracePeriodSeconds":30,"dnsPolicy":"ClusterFirst","serviceAccountName":"very-crazy-sa","serviceAccount":"very-crazy-sa","nodeName":"node2","securityContext":{},"schedulerName":"default-scheduler","tolerations":[{"key":"node.kubernetes.io/not-ready","operator":"Exists","effect":"NoExecute","tolerationSeconds":300},{"key":"node.kubernetes.io/unreachable","operator":"Exists","effect":"NoExecute","tolerationSeconds":300}],"priority":0,"enableServiceLinks":true,"preemptionPolicy":"PreemptLowerPriority"},"status":{"phase":"Running","conditions":[{"type":"Initialized","status":"True","lastProbeTime":null,"lastTransitionTime":"2021-05-24T09:29:24Z"},{"type":"Ready","status":"True","lastProbeTime":null,"lastTransitionTime":"2021-05-24T09:29:44Z"},{"type":"ContainersReady","status":"True","lastProbeTime":null,"lastTransitionTime":"2021-05-24T09:29:44Z"},{"type":"PodScheduled","status":"True","lastProbeTime":null,"lastTransitionTime":"2021-05-24T09:29:24Z"}],"hostIP":"192.168.211.42","podIP":"10.244.104.11","podIPs":[{"ip":"10.244.104.11"}],"startTime":"2021-05-24T09:29:24Z","containerStatuses":[{"name":"accessor","state":{"running":{"startedAt":"2021-05-24T09:29:43Z"}},"lastState":{},"ready":true,"restartCount":0,"image":"nginx:latest","imageID":"docker-pullable://nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2","containerID":"docker://d80411bf54156c9f65c4887c4255dc545b8ebdf518d4d9470f23b0ad3f984b39","started":true}],"qosClass":"BestEffort"}},"requestReceivedTimestamp":"2021-05-24T09:29:44.384315Z","stageTimestamp":"2021-05-24T09:29:44.426816Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}

```

