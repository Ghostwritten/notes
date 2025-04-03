#  kubernetes  Auditing 实战
tags:   Auditing
<!-- catalg:~实战~ -->



[![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3e0d22c2d4fe194c5da6412852c92c3.gif#pic_center)](https://www.behance.net/gallery/87137241/Terminator-x-Game-for-peace)



上一篇是：[kuberentes Auditing 入门](https://ghostwritten.blog.csdn.net/article/details/126265666)

##  1. minikube 安装 kuberentes

```bash
minikube start
```

 - [Minikube 在 Centos 7 部署 Kubernetes](https://blog.csdn.net/xixihahalelehehe/article/details/123796854)
 - [Ubuntu 18.04 通过 Minikube 安装 Kubernetes v1.20](https://blog.csdn.net/xixihahalelehehe/article/details/113527867)


##  2. 配置 audit-policy.yaml
创建一个`audit-policy.yaml`文件

```bash
vim /etc/kubernetes/audit-policy.yaml
```
输入：

```bash
apiVersion: audit.k8s.io/v1 # This is required.
kind: Policy

# Prevent requests in the RequestReceived stage from generating audit events.
omitStages:
  - "RequestReceived"

rules:
  # Log pod changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["pods"]
  # Log "pods/log", "pods/status" at Metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["pods/log", "pods/status"]
  # Exclude logging requests to a configmap called "controller-config"
  - level: None
    resources:
    - group: ""
      resources: ["configmaps"]
      resourceNames: ["controller-config"]
  # Don't log watch requests by the "system:kube-proxy" on endpoints or services
  - level: None
    users: ["system:kube-proxy"]
    verbs: ["watch"]
    resources:
    - group: "" # core API group
      resources: ["endpoints", "services"]
  # Log deployment changes at RequestResponse level
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["deployments"]
  # Log service changes at metadata level
  - level: Metadata
    resources:
    - group: ""
      resources: ["services"]
  # Log the request body of configmap changes in the kube-system namespace.
  - level: Request
    resources:
    - group: "" # core API group
      resources: ["configmaps"]
    # You can use an empty string [""] to select resources not associated with a namespace.
    namespaces: ["kube-system"]
  # Log configmap and secret changes in all other namespaces at the Metadata level.
  - level: Metadata
    resources:
    - group: "" # core API group
      resources: ["secrets", "configmaps"]
  # Log all other resources in core and extensions at the Request level.
  - level: Request
    resources:
    - group: "" # core API group
    - group: "extensions" # Version of group should NOT be included.
  # A wild-card rule to log all other requests at the Metadata level.
  - level: Metadata
    # Long-running requests like watches that fall under this rule will not
    # generate an audit event in RequestReceived.
    omitStages:
      - "RequestReceived"
```
##  3. 配置存储 Auditing 日志
在以下目录中创建`audit.log` 。这是 Kubernetes 将保存您的审计日志的地方。

```bash
mkdir /var/log/kubernetes/ && cd /var/log/kubernetes/
mkdir audit/ && sudo touch audit.log
```
编辑`kube-apiserver`配置文件。

```bash
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
更新参数

```bash
- --audit-policy-file=/etc/kubernetes/audit-policy.yaml
- --audit-log-path=/var/log/kubernetes/audit/audit.log
```

更新配置文件的卷挂载部分。

```bash
# ...
volumeMounts:
    - mountPath: /etc/kubernetes/audit-policy.yaml
      name: audit
      readOnly: true
    - mountPath: /var/log/kubernetes/audit/audit.log
      name: audit-log
 # ...
```
更新volume部分。

```bash
 # ...
 volumes:

 - hostPath:
      path: /etc/kubernetes/audit-policy.yaml
      type: File
   name: audit
 - hostPath:
      path: /var/log/kubernetes/audit/audit.log
      type: FileOrCreate
   name: audit-log
 # ...
```
##  4. 测试 Auditing 功能
给 Kubernetes 几秒钟的时间来应用您对`kube-apiserver`配置文件所做的更改，然后您可以测试您的审计以确保其正常工作。

```bash
kubectl get pods -n kube-system -w
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82a07458eb82264085d7aba025e1434d.png)
创建一个新的 pod，看看 Kubernetes 是否会将请求记录在日志文件中。

```bash
kubectl run nginx --image=nginx
```
返回`minikube ssh`终端并通过运行以下命令打开日志文件以查看其中是否有日志。

```bash
sudo cat /var/log/kubernetes/audit/audit.log | grep -i "nginx"
```
如果您在日志文件中看到类似于下图的内容，则说明您的 Kubernetes 集群上现已启用并配置了审计。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4eefc95461c4b604eab0e5c8c16e44d8.png)
##  5. ContainIQ 监控 Kubernetes 审计日志
您可以在 Kubernetes 集群中安装多种工具，以在收集和查看日志时提供更丰富的体验。一个这样的工具是 ContainIQ，一个[K8s 监控平台](https://www.containiq.com/feature/logs)。ContainIQ 让您和您的团队可以轻松监控集群内的 Kubernetes 指标、日志和事件。它提供了改进的查看和过滤选项，以帮助您查看[日志中真正重要的内容](https://www.containiq.com/post/kubernetes-logging)。

安装`metrics server`（如果尚未安装）。

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
下载 ContainIQ 部署文件。

```bash
curl -L -o deployment.yaml https://raw.githubusercontent.com/containiq/containiq-deployment/master/deployment.yaml
```
通过[创建帐户](https://app.containiq.com/register)获取您的 ContainIQ API 密钥，然后打开部署文件，滚动到底部，并将您的 API 密钥添加到密钥对象。
完成后，应用部署。

```bash
kubectl apply -f deployment.yaml
```
在集群上实施 ContainIQ 可使您的日志记录更上一层楼。虽然 Kubernetes 审计日志提供了有价值的信息，但它们仅以原始数据日志的形式提供访问，这些日志难以阅读且筛选耗时。但是，使用 ContainIQ，您拥有一个用户界面，允许您根据 pod、时间戳和搜索查询轻松过滤日志，从而更轻松地提取您要查找的信息。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/72488431acbb1ab13964fe97d0ebf886.png)


##  6. demo

###  6.1 configure apiserver to store Audit Logs in json format
```bash
mkdir /etc/kubernetes/auditing
mkidr /etc/kubernetes/audit/logs
```

```bash
$ cat /etc/kubernetes/audit/policy.yaml 
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```

```bash
$ vim /etc/kubernetes/manifests/kube-apiserver.yaml


.....
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml       # add
    - --audit-log-path=/etc/kubernetes/audit/logs/audit.log       # add
    - --audit-log-maxsize=500                                     # add
    - --audit-log-maxbackup=5  
.........
# ...
volumeMounts:
    - mountPath: /etc/kubernetes/audit/policy.yaml
      name: audit
      readOnly: true
    - mountPath: /var/log/kubernetes/audit/audit.log
      name: audit-log
 # ...

 volumes:
 - hostPath:
      path: /etc/kubernetes/audit/policy.yaml
      type: File
   name: audit
 - hostPath:
      path: /var/log/kubernetes/audit/audit.log
      type: FileOrCreate
   name: audit-log
 # ...

```
  
查看kube-apiserver 启动是否正常

```bash
$ k get pods -n kube-system |grep api
kube-apiserver-master                      1/1     Running   3          5m
```
查看audit.log日志

```bash
tail /etc/kubernetes/audit/logs/audit.log 
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"3ea2f430-108b-4b17-b967-6e26619fda99","stage":"RequestReceived","requestURI":"/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-controller-manager?timeout=10s","verb":"update","user":{"username":"system:kube-controller-manager","groups":["system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kube-controller-manager/v1.20.7 (linux/amd64) kubernetes/132a687/leader-election","objectRef":{"resource":"leases","namespace":"kube-system","name":"kube-controller-manager","apiGroup":"coordination.k8s.io","apiVersion":"v1"},"requestReceivedTimestamp":"2021-05-24T08:28:50.431353Z","stageTimestamp":"2021-05-24T08:28:50.431353Z"}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"f6a3fc3b-96eb-4e2e-9ba2-66d2e345fb8a","stage":"RequestReceived","requestURI":"/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-scheduler?timeout=10s","verb":"update","user":{"username":"system:kube-scheduler","groups":["system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kube-scheduler/v1.20.7 (linux/amd64) kubernetes/132a687/leader-election","objectRef":{"resource":"leases","namespace":"kube-system","name":"kube-scheduler","apiGroup":"coordination.k8s.io","apiVersion":"v1"},"requestReceivedTimestamp":"2021-05-24T08:28:50.435266Z","stageTimestamp":"2021-05-24T08:28:50.435266Z"}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"3ea2f430-108b-4b17-b967-6e26619fda99","stage":"ResponseComplete","requestURI":"/apis/coordination.k8s.io/v1/namespaces/kube-system/leases/kube-controller-manager?timeout=10s","verb":"update","user":{"username":"system:kube-controller-manager","groups":["system:authenticated"]},"sourceIPs":["192.168.211.40"],"userAgent":"kube-controller-manager/v1.20.7 (linux/amd64) kubernetes/132a687/leader-election","objectRef":{"resource":"leases","namespace":"kube-system","name":"kube-controller-manager","uid":"bcb35dd3-5cb0-4460-99af-dcedb37a6bfa","apiGroup":"coordination.k8s.io","apiVersion":"v1","resourceVersion":"30280"},"responseStatus":{"metadata":{},"code":200},"requestReceivedTimestamp":"2021-05-24T08:28:50.431353Z","stageTimestamp":"2021-05-24T08:28:50.438912Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by ClusterRoleBinding \"system:kube-controller-manager\" of ClusterRole \"system:kube-controller-manager\" to User \"system:kube-controller-manager\""}}
```

###  6.2 create a secret and investigate the json audit log

```bash
$ k create secret generic very-secure --from-literal=user=admin
secret/very-secure created
```
查看日志

```bash
$ cat /etc/kubernetes/audit/logs/audit.log |grep very-secure | jq .
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

###  6.3 restrict amount of Audit logs to collect
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6c2d8e49fb7ec307fc8a4b3667897e1f.png)

```bash
$ cat policy.yaml 
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
```
重启kube-apiserver
```bash
mv /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/
ps aux |grep api
mv /etc/kubernetes/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml
ps aux |grep api

```
查看audit.log

```bash
tail /etc/kubernetes/audit/logs/audit.log | jq .
```
###  6.4 use Audit Logs to investigate access history
创建 servicemount
```bash
k create sa very-crazy-sa
k get sa
k get secret
cat /etc/kubernetes/audit/logs/audit.log |grep very-crazy-sa
```
创建 pod

```bash
$ k run accessor --image=nginx --dry-run=client -oyaml > pod.yaml

$ vim pod3.yaml 
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



k create -f pod.yaml
k get pod accessor -w


$ cat /etc/kubernetes/audit/logs/audit.log |grep accessor
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"RequestResponse","auditID":"1025990a-e51c-4c51-9010-6436999a88cb","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/default/pods/accessor/status","verb":"patch","user":{"username":"system:node:node2","groups":["system:nodes","system:authenticated"]},"sourceIPs":["192.168.211.42"],"userAgent":"kubelet/v1.20.1 (linux/amd64) kubernetes/c4d7527","objectRef":{"resource":"pods","namespace":"default","name":"accessor","apiVersion":"v1","subresource":"status"},"responseStatus":{"metadata":{},"code":200},"requestObject":{"metadata":{"uid":"d67a9290-1dc8-4f14-ac84-88e9ae82d2a2"},"status":{"$setElementOrder/conditions":[{"type":"Initialized"},{"type":"Ready"},{"type":"ContainersReady"},{"type":"PodScheduled"}],"conditions":[{"lastTransitionTime":"2021-05-24T09:29:44Z","message":null,"reason":null,"status":"True","type":"Ready"},{"lastTransitionTime":"2021-05-24T09:29:44Z","message":null,"reason":null,"status":"True","type":"ContainersReady"}],"containerStatuses":[{"containerID":"docker://d80411bf54156c9f65c4887c4255dc545b8ebdf518d4d9470f23b0ad3f984b39","image":"nginx:latest","imageID":"docker-pullable://nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2","lastState":{},"name":"accessor","ready":true,"restartCount":0,"started":true,"state":{"running":{"startedAt":"2021-05-24T09:29:43Z"}}}],"phase":"Running","podIP":"10.244.104.11","podIPs":[{"ip":"10.244.104.11"}]}},"responseObject":{"kind":"Pod","apiVersion":"v1","metadata":{"name":"accessor","namespace":"default","uid":"d67a9290-1dc8-4f14-ac84-88e9ae82d2a2","resourceVersion":"35224","creationTimestamp":"2021-05-24T09:29:24Z","labels":{"run":"accessor"},"annotations":{"cni.projectcalico.org/podIP":"10.244.104.11/32","cni.projectcalico.org/podIPs":"10.244.104.11/32"},"managedFields":[{"manager":"kubectl-create","operation":"Update","apiVersion":"v1","time":"2021-05-24T09:29:24Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:labels":{".":{},"f:run":{}}},"f:spec":{"f:containers":{"k:{\"name\":\"accessor\"}":{".":{},"f:image":{},"f:imagePullPolicy":{},"f:name":{},"f:resources":{},"f:terminationMessagePath":{},"f:terminationMessagePolicy":{}}},"f:dnsPolicy":{},"f:enableServiceLinks":{},"f:restartPolicy":{},"f:schedulerName":{},"f:securityContext":{},"f:serviceAccount":{},"f:serviceAccountName":{},"f:terminationGracePeriodSeconds":{}}}},{"manager":"calico","operation":"Update","apiVersion":"v1","time":"2021-05-24T09:29:26Z","fieldsType":"FieldsV1","fieldsV1":{"f:metadata":{"f:annotations":{".":{},"f:cni.projectcalico.org/podIP":{},"f:cni.projectcalico.org/podIPs":{}}}}},{"manager":"kubelet","operation":"Update","apiVersion":"v1","time":"2021-05-24T09:29:44Z","fieldsType":"FieldsV1","fieldsV1":{"f:status":{"f:conditions":{"k:{\"type\":\"ContainersReady\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Initialized\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}},"k:{\"type\":\"Ready\"}":{".":{},"f:lastProbeTime":{},"f:lastTransitionTime":{},"f:status":{},"f:type":{}}},"f:containerStatuses":{},"f:hostIP":{},"f:phase":{},"f:podIP":{},"f:podIPs":{".":{},"k:{\"ip\":\"10.244.104.11\"}":{".":{},"f:ip":{}}},"f:startTime":{}}}}]},"spec":{"volumes":[{"name":"very-crazy-sa-token-fr7sw","secret":{"secretName":"very-crazy-sa-token-fr7sw","defaultMode":420}}],"containers":[{"name":"accessor","image":"nginx","resources":{},"volumeMounts":[{"name":"very-crazy-sa-token-fr7sw","readOnly":true,"mountPath":"/var/run/secrets/kubernetes.io/serviceaccount"}],"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","imagePullPolicy":"Always"}],"restartPolicy":"Always","terminationGracePeriodSeconds":30,"dnsPolicy":"ClusterFirst","serviceAccountName":"very-crazy-sa","serviceAccount":"very-crazy-sa","nodeName":"node2","securityContext":{},"schedulerName":"default-scheduler","tolerations":[{"key":"node.kubernetes.io/not-ready","operator":"Exists","effect":"NoExecute","tolerationSeconds":300},{"key":"node.kubernetes.io/unreachable","operator":"Exists","effect":"NoExecute","tolerationSeconds":300}],"priority":0,"enableServiceLinks":true,"preemptionPolicy":"PreemptLowerPriority"},"status":{"phase":"Running","conditions":[{"type":"Initialized","status":"True","lastProbeTime":null,"lastTransitionTime":"2021-05-24T09:29:24Z"},{"type":"Ready","status":"True","lastProbeTime":null,"lastTransitionTime":"2021-05-24T09:29:44Z"},{"type":"ContainersReady","status":"True","lastProbeTime":null,"lastTransitionTime":"2021-05-24T09:29:44Z"},{"type":"PodScheduled","status":"True","lastProbeTime":null,"lastTransitionTime":"2021-05-24T09:29:24Z"}],"hostIP":"192.168.211.42","podIP":"10.244.104.11","podIPs":[{"ip":"10.244.104.11"}],"startTime":"2021-05-24T09:29:24Z","containerStatuses":[{"name":"accessor","state":{"running":{"startedAt":"2021-05-24T09:29:43Z"}},"lastState":{},"ready":true,"restartCount":0,"image":"nginx:latest","imageID":"docker-pullable://nginx@sha256:df13abe416e37eb3db4722840dd479b00ba193ac6606e7902331dcea50f4f1f2","containerID":"docker://d80411bf54156c9f65c4887c4255dc545b8ebdf518d4d9470f23b0ad3f984b39","started":true}],"qosClass":"BestEffort"}},"requestReceivedTimestamp":"2021-05-24T09:29:44.384315Z","stageTimestamp":"2021-05-24T09:29:44.426816Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":""}}
```
参考：

 - [Kubernetes Auditing](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/)
 - [Google Kubernetes Engine (GKE)  audit-policy](https://cloud.google.com/kubernetes-engine/docs/concepts/audit-policy)
 - [6 Best Practices for Kubernetes Audit Logging](https://goteleport.com/blog/kubernetes-audit-logging/)
 - [Kubernetes Audit Logs | Use Cases & Best Practices](https://www.containiq.com/post/kubernetes-audit-logs)

