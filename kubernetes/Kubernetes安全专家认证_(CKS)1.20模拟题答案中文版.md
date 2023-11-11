

-----
## 问题 1 | 上下文
任务权重：1%

 

您可以通过kubectl上下文从主终端访问多个集群。将所有上下文名称写入`/opt/course/1/contexts`，每行一个。

从 kubeconfig 中提取用户的证书`restricted@infra-prod`并将其解码为`/opt/course/1/cert`.

 

回答：
也许最快的方法就是运行：
```bash
k config get-contexts -o name > /opt/course/1/contexts

```bash
# cat /opt/course/1/contexts
gianna@infra-prod
infra-prod
restricted@infra-prod
workload-prod
workload-stage


k config view --raw
And copy it manually. Or we do:

k config view --raw -ojsonpath="{.users[2].user.client-certificate-data}" | base64 -d > /opt/course/1/cert
```

-----
## 问题 2 | Falco 的运行时安全性
任务权重：4%

 

使用上下文： `kubectl config use-context workload-prod`

 

Falco 使用默认配置安装在 node `cluster1-worker1`。使用`ssh cluster1-worker1`. 用它来：

 1. 找到一个Pod运行镜像nginx，它会在其容器内创建不需要的包管理进程。
 2. 找到一个Pod运行镜像httpd，它修改了`/etc/passwd.`

`/opt/course/2/falco.log`以格式保存案例 1 的 Falco 日志`time,container-id,container-name,user-name`。任何行都不应包含其他信息。收集日志至少 30 秒。

然后通过将控制有问题的Pod的部署的副本缩放到 0 来删除线程（1 和 2）。

 

回答：
Falco是开源云原生运行时安全项目，是事实上的 Kubernetes 威胁检测引擎。

> 注意： 您可能必须熟悉的其他工具是sysdig或tracee

使用 Falco 作为服务

首先我们可以稍微调查一下 Falco 配置：

```c
➜ ssh cluster1-worker1
​
➜ root@cluster1-worker1:~# service falco status
● falco.service - LSB: Falco syscall activity monitoring agent
   Loaded: loaded (/etc/init.d/falco; generated)
   Active: active (running) since Sat 2020-10-10 06:36:15 UTC; 2h 1min ago
...
​
➜ root@cluster1-worker1:~# cd /etc/falco
​
➜ root@cluster1-worker1:/etc/falco# ls
falco.yaml  falco_rules.local.yaml  falco_rules.yaml  k8s_audit_rules.yaml  rules.available  rules.d

➜ root@cluster1-worker1:~# cat /var/log/syslog | grep falco
Oct  9 21:46:55 ubuntu-bionic falco: Falco version 0.26.1 (driver version 2aa88dcf6243982697811df4c1b484bcbe9488a2)
Oct  9 21:46:55 ubuntu-bionic falco: Falco initialized with configuration file /etc/falco/falco.yaml
...


➜ root@cluster1-worker1:~# cat /var/log/syslog | grep falco | grep nginx | grep process
Oct  9 23:14:49 ubuntu-bionic falco: 23:14:49.070029616: Error Package management process launched in container (user=root user_loginuid=-1 command=apk container_id=b51765aeafcb container_name=k8s_nginx_webapi-6b797d5b65-7mxz7_team-blue_a0221829-d3ee-4c72-bfac-496a16e1c3fc_0 image=nginx:1.19.2-alpine)
...


➜ root@cluster1-worker1:~# cat /var/log/syslog | grep falco | grep httpd | grep passwd
Oct  9 23:26:19 ubuntu-bionic falco: 23:26:19.950348409: Error File below /etc opened for writing (user=root user_loginuid=-1 command=sed -i $d /etc/passwd parent=sh pcmdline=sh -c echo hacker >> /etc/passwd; sed -i '$d' /etc/passwd; true file=/etc/passwdJPAmlk program=sed gparent=<NA> ggparent=<NA> gggparent=<NA> container_id=d9ce1e213996 image=httpd)
​
➜ root@cluster1-worker1:~# docker ps | grep d9ce1e213996
d9ce1e213996        35ab485181ce                           "httpd-foreground"       3 minutes ago       Up 3 minutes                            k8s_httpd_rating-service-5c54c948c9-fnvn2_team-purple_02083909-c6f5-4f87-bab3-fae9de2fc55a_0


➜ root@cluster1-worker1:~# service falco stop
​
➜ root@cluster1-worker1:~# falco
Sat Dec  5 19:58:29 2020: Falco version 0.26.1 (driver version 2aa88dcf6243982697811df4c1b484bcbe9488a2)
Sat Dec  5 19:58:29 2020: Falco initialized with configuration file /etc/falco/falco.yaml
Sat Dec  5 19:58:29 2020: Loading rules from file /etc/falco/falco_rules.yaml:
Sat Dec  5 19:58:29 2020: Loading rules from file /etc/falco/falco_rules.local.yaml:
Sat Dec  5 19:58:30 2020: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Sat Dec  5 19:58:30 2020: Starting internal webserver, listening on port 8765
​
19:58:34.436913858: Error Package management process launched in container (user=root user_loginuid=-1 command=apk container_id=fd6a98d42973 container_name=k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0 image=nginx:1.19.2-alpine)
...


➜ root@cluster1-worker1:~# service falco stop
​
➜ root@cluster1-worker1:~# falco
Sat Dec  5 19:58:29 2020: Falco version 0.26.1 (driver version 2aa88dcf6243982697811df4c1b484bcbe9488a2)
Sat Dec  5 19:58:29 2020: Falco initialized with configuration file /etc/falco/falco.yaml
Sat Dec  5 19:58:29 2020: Loading rules from file /etc/falco/falco_rules.yaml:
Sat Dec  5 19:58:29 2020: Loading rules from file /etc/falco/falco_rules.local.yaml:
Sat Dec  5 19:58:30 2020: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Sat Dec  5 19:58:30 2020: Starting internal webserver, listening on port 8765
​
19:58:34.436913858: Error Package management process launched in container (user=root user_loginuid=-1 command=apk container_id=fd6a98d42973 container_name=k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0 image=nginx:1.19.2-alpine)
...

➜ root@cluster1-worker1:~# cd /etc/falco/
​
➜ root@cluster1-worker1:/etc/falco# grep -r "Package management process launched" .
./falco_rules.yaml:    Package management process launched in container (user=%user.name user_loginuid=%user.loginuid
​
➜ root@cluster1-worker1:/etc/falco# cp falco_rules.yaml falco_rules.yaml_ori
​
➜ root@cluster1-worker1:/etc/falco# vim falco_rules.yaml
# Container is supposed to be immutable. Package management should be done in building the image.
- rule: Launch Package Management Process in Container
  desc: Package management process ran inside container
  condition: >
    spawned_process
    and container
    and user.name != "_apt"
    and package_mgmt_procs
    and not package_mgmt_ancestor_procs
    and not user_known_package_manager_in_container
  output: >
    Package management process launched in container (user=%user.name user_loginuid=%user.loginuid
    command=%proc.cmdline container_id=%container.id container_name=%container.name image=%container.image.repository:%container.image.tag)
  priority: ERROR
  tags: [process, mitre_persistence]
```

And change it into the required format:

```bash
# Container is supposed to be immutable. Package management should be done in building the image.
- rule: Launch Package Management Process in Container
  desc: Package management process ran inside container
  condition: >
    spawned_process
    and container
    and user.name != "_apt"
    and package_mgmt_procs
    and not package_mgmt_ancestor_procs
    and not user_known_package_manager_in_container
  output: >
    Package management process launched in container %evt.time,%container.id,%container.name,%user.name
  priority: ERROR
  tags: [process, mitre_persistence]

➜ root@cluster1-worker1:/etc/falco# falco | grep "Package management"
​
20:23:14.395725592: Error Package management process launched in container 20:23:14.395725592,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root
20:23:19.566382518: Error Package management process launched in container 20:23:19.566382518,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root
20:23:24.502379334: Error Package management process launched in container 20:23:24.502379334,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root
20:23:29.802281942: Error Package management process launched in container 20:23:29.802281942,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root
20:23:34.560697766: Error Package management process launched in container 20:23:34.560697766,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root
20:23:39.840817808: Error Package management process launched in container 20:23:39.840817808,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root
20:23:44.409411640: Error Package management process launched in container 20:23:44.409411640,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root
20:23:49.588266119: Error Package management process launched in container 20:23:49.588266119,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root
20:23:54.487145960: Error Package management process launched in container 20:23:54.487145960,fd6a98d42973,k8s_nginx_webapi-5fcb69b746-gtx8q_team-blue_d5e9178c-60fb-43e5-af89-3b8a579614ef_0,root



➜ k get pod -A | grep webapi
team-blue              webapi-6b797d5b65-7mxz7                      1/1     Running
​
➜ k -n team-blue scale deploy webapi --replicas 0
deployment.apps/webapi scaled
​
➜ k get pod -A | grep rating-service
team-purple            rating-service-5c54c948c9-fnvn2              1/1     Running
​
➜ k -n team-purple scale deploy rating-service --replicas 0
```





-----
## 问题 3 | apiserver 安全
任务权重：3%

 

使用上下文： `kubectl config use-context workload-prod`

 

您从 DevSecOps 团队收到了一份清单，该团队对 k8s 集群1 ( )进行了安全调查workload-prod。该列表说明了有关 apiserver 设置的以下内容：

 1. 可通过 NodePort服务访问

更改 apiserver 设置，以便：

 1. 只能通过 ClusterIP服务访问

 

回答：
为了修改 apiserver 的参数，我们首先 ssh 进入主节点并检查 apiserver 进程正在运行的参数：

```c
➜ ssh cluster1-master1
​
➜ root@cluster1-master1:~# ps aux | grep kube-apiserver
root     13534  8.6 18.1 1099208 370684 ?      Ssl  19:55   8:40 kube-apiserver --advertise-address=192.168.100.11 --allow-privileged=true --anonymous-auth=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --insecure-port=0 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --kubernetes-service-node-port=31000 --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-
...


➜ root@cluster1-master1:~# kubectl get svc
NAME         TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kubernetes   NodePort   10.96.0.1    <none>        443:31000/TCP   5d2h


➜ root@cluster1-master1:~# cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/3_kube-apiserver.yaml
​
➜ root@cluster1-master1:~# vim /etc/kubernetes/manifests/kube-apiserver.yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.100.11:6443
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
    - --advertise-address=192.168.100.11
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
#    - --kubernetes-service-node-port=31000   # delete or set to 0
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
...


➜ root@cluster1-master1:~# kubectl -n kube-system get pod | grep apiserver
kube-apiserver-cluster1-master1            1/1     Running        0          38s
​
➜ root@cluster1-master1:~# ps aux | grep kube-apiserver | grep node-port

➜ root@cluster1-master1:~# kubectl get svc
NAME         TYPE       CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
kubernetes   NodePort   10.96.0.1    <none>        443:31000/TCP   5d3h

➜ root@cluster1-master1:~# kubectl delete svc kubernetes
service "kubernetes" deleted

➜ root@cluster1-master1:~# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6s
```



-----
## 问题 4 | Pod 安全策略
任务权重：8%

 
使用上下文： `kubectl config use-context workload-prod`

 

命名空间 `team-red` 中有部署 `docker-log-hacker`，它作为主机路径卷安装在运行它的节点上。这意味着Pod可以例如读取在同一个Node上运行的所有 Docker 容器日志`/var/lib/docker`

您需要通过以下方式禁止这种行为：

 1. 在 apiserver 中启用准入插件`PodSecurityPolicy`
 2. 创建一个名为`psp-mount`的`PodSecurityPolicy`，它只允许目录的 hostPath 卷`/tmp`
 3. 创建一个名为`psp-mount`的ClusterRole，允许使用新的PSP
 4. 创建`RoleBinding`命名`psp-mount`在命名空间 team-red，结合新`ClusterRole`所有`ServiceAccounts`在命名空间`team-red`
  

重新启动吊舱的部署docker-log-hacker事后验证，防止新的创造。
注意： PSP会影响整个集群。如果您遇到问题，您可以随时再次禁用准入插件。
 

回答：
调查
首先，让我们检查一下Pod of Deployment的功能： docker-log-hacker

```c
➜ k -n team-red get pod | grep hacker
docker-log-hacker-79cd6c58d5-2g4zv      1/1     Running   0          88s
​
➜ k -n team-red describe pod docker-log-hacker-79cd6c58d5-2g4zv
Name:         docker-log-hacker-79cd6c58d5-2g4zv
Namespace:    team-red
Priority:     0
Node:         cluster1-worker1/192.168.100.12
Start Time:   Mon, 28 Sep 2020 09:17:44 +0000
Labels:       app=docker-log-hacker
              pod-template-hash=79cd6c58d5
Annotations:  <none>
Status:       Running
IP:           10.44.0.19
IPs:
  IP:           10.44.0.19
Controlled By:  ReplicaSet/docker-log-hacker-79cd6c58d5
Containers:
  bash:
...
    Command:
      sh
      -c
      while true; do sleep 1d; done
...
    Mounts:
      /dockerlogs from dockerlogs (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-9c2wf (ro)
...
Volumes:
  dockerlogs:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/docker
...


➜ k -n team-red exec -it docker-log-hacker-79cd6c58d5-2g4zv -- sh
​
➜ # cd /dockerlogs/containers/
​
➜ /dockerlogs/containers # ls
025c21a3e9f466550d15d06620318dc2a4dc5bd09562b3e30169fde56162f6ba
092cb84e4cb17f537aaf50a78f6e3d0737b90d78ff49c03aa88547daf66359dd
17f00b3e25313b05c1f642831f25e388852664699d8a5be26315cb14642016de
...
​
➜ /dockerlogs/containers # cd 47ac9c0a75aff011e865ebfb7b1695bddc891fccf59e6eafddb06032d44c6d5b/
​
➜ /dockerlogs/containers/47ac9c0a75aff011e865ebfb7b1695bddc891fccf59e6eafddb06032d44c6d5b # head 47ac9c0a75aff011e865e
bfb7b1695bddc891fccf59e6eafddb06032d44c6d5b-json.log 
​
{"log":"Mon Sep 28 08:55:14 UTC 2020\n","stream":"stdout","time":"2020-09-28T08:55:14.431789056Z"}
{"log":"uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)\n","stream":"stdout","time":"2020-09-28T08:55:14.441553845Z"}
{"log":"\n","stream":"stdout","time":"2020-09-28T08:55:14.442354173Z"}
{"log":"Mon Sep 28 08:55:15 UTC 2020\n","stream":"stdout","time":"2020-09-28T08:55:15.446719832Z"}
...
```
我们可以看到这个Pod可以从运行在同一个Node上的所有容器访问 Docker 日志。除非必要，否则应该阻止的事情。


为PodSecurityPolicy启用准入插件
我们启用准入插件并创建一个配置备份，以防我们配置错误：

```c
➜ ssh cluster1-master1
​
➜ root@cluster1-master1:~# cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/4_kube-apiserver.yaml
​
➜ root@cluster1-master1:~# vim /etc/kubernetes/manifests/kube-apiserver.yaml 
# /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.100.11:6443
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
        - --advertise-address=192.168.100.11
        - --allow-privileged=true
        - --anonymous-auth=true
        - --authorization-mode=Node,RBAC
        - --client-ca-file=/etc/kubernetes/pki/ca.crt
        - --enable-admission-plugins=NodeRestriction,PodSecurityPolicy      # change
        - --enable-bootstrap-token-auth=true
        - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
...
 
```

现有PodSecurityPolicy
在未授权任何策略的情况下启用PSP准入插件将阻止在集群中创建任何Pod。这就是为什么已经有一个现有的PSP允许一切和所有命名空间，除了通过RoleBinding使用它： `default-allow-allteam-red`

```c
➜ k get psp
NAME                PRIV   CAPS   SELINUX    RUNASUSER   ...
default-allow-all   true   *      RunAsAny   RunAsAny    ...
​
➜ k get rolebinding -A | grep psp-access
default                psp-access ...  ClusterRole/psp-access                         
kube-public            psp-access ...  ClusterRole/psp-access                            
kube-system            psp-access ...  ClusterRole/psp-access                               
kubernetes-dashboard   psp-access ...  ClusterRole/psp-access                               
team-blue              psp-access ...  ClusterRole/psp-access                               
team-green             psp-access ...  ClusterRole/psp-access                               
team-purple            psp-access ...  ClusterRole/psp-access                               
team-yellow            psp-access ...  ClusterRole/psp-access       
```

创建新的PodSecurityPolicy
接下来，我们通过从 k8s 文档中复制一个示例并对其进行更改来创建具有任务要求的新PSP：
```c
vim 4_psp.yaml
# 4_psp.yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp-mount
spec:
  privileged: true
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
  allowedHostPaths:             # task requirement
    - pathPrefix: "/tmp"        # task requirement
```

```c
k -f 4_psp.yaml create
```
到目前为止，PSP没有任何影响，因为我们还没有授予任何Pods - ServiceAccounts使用它的RBAC 权限。所以我们这样做：

```c
k -n team-red create clusterrole psp-mount --verb=use --resource=podsecuritypolicies --resource-name=psp-mount

k -n team-red create rolebinding psp-mount --clusterrole=psp-mount --group system:serviceaccounts
```
测试新的 PSP
我们重新启动部署并检查状态：

```c
➜ k -n team-red rollout restart deploy docker-log-hacker
deployment.apps/docker-log-hacker restarted
​
➜ k -n team-red describe deploy docker-log-hacker
Name:                   docker-log-hacker
Namespace:              team-red
...
Replicas:               1 desired | 0 updated | 0 total | 0 available | 2 unavailable
...
Pod Template:
  Labels:       app=docker-log-hacker
  Annotations:  kubectl.kubernetes.io/restartedAt: 2020-09-28T11:08:18Z
  Containers:
   bash:
...
  Volumes:
   dockerlogs:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/docker
    HostPathType:  
Conditions:
  Type             Status  Reason
  ----             ------  ------
  Available        False   MinimumReplicasUnavailable
  ReplicaFailure   True    FailedCreate
```

我们看到 FailedCreate 和检查事件显示了有关原因的更多信息：

```c
➜ k -n team-red get events --sort-by='{.metadata.creationTimestamp}'
​
docker-log-hacker-6bdfbf8546-" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.volumes[0].hostPath.pathPrefix: Invalid value: "/var/lib/docker": is not allowed to be used]
```

漂亮，PSP似乎工作。为了进一步验证，我们可以更改部署：

```c
k -n team-red edit deploy docker-log-hacker
# kubectl -n team-red edit deploy docker-log-hacker
apiVersion: apps/v1
kind: Deployment
metadata:
...
spec:
...
  template:
    metadata:
...
    spec:
      containers:
      - command:
        - sh
        - -c
        - while true; do sleep 1d; done
        image: bash
...
        volumeMounts:
        - mountPath: /dockerlogs
          name: dockerlogs
...
      volumes:
      - hostPath:
          path: /tmp                         # change
          type: ""
```
我们应该看到它正在运行：

```c
➜ k -n team-red get pod -l app=docker-log-hacker
NAME                                 READY   STATUS    RESTARTS   AGE
docker-log-hacker-5674dbccc9-5lc6q   1/1     Running   0          20s
```
当PSP允许创建Pod 时，这将通过注释显示：

```c
➜ k -n team-red describe pod -l app=docker-log-hacker
...
Annotations:  kubernetes.io/psp: psp-mount
...
```

-----
## 问题 5 | CIS Benchmark
任务权重：3%

相关阅读：

 - [Kubernetes CKS 2021【6】---Cluster Setup - CIS Benchmarks](https://ghostwritten.blog.csdn.net/article/details/116006742)

 

使用上下文： `kubectl config use-context infra-prod`

 

您需要根据 CIS 基准建议评估 `cluster2` 的特定设置。使用已安装在节点上的工具`kube-bench`。

使用`ssh cluster2-master1`和连接`ssh cluster2-worker1`。

在主节点上确保（必要时更正）为以下各项设置 CIS 建议：

 1. `kube-controller-manager`的`--profiling`论点
 2. 目录所有权 `/var/lib/etcd`

在工作节点上确保（必要时更正）为以下各项设置 CIS 建议：

 1. kubelet 配置的权限 `/var/lib/kubelet/config.yaml`
 2. kubelet的`--client-ca-file`论证

 

回答：
1号
首先，我们通过 ssh 进入主节点，针对主组件运行kube-bench：
```c
➜ ssh cluster2-master1
​
➜ root@cluster2-master1:~# kube-bench master
...
== Summary ==
41 checks PASS
13 checks FAIL
11 checks WARN
0 checks INFO
```
我们看到一些通过、失败和警告。让我们检查控制器管理器所需的任务（1）：
```c
➜ root@cluster2-master1:~# kube-bench master | grep kube-controller -A 3
1.3.1 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml

on the master node and set the --terminated-pod-gc-threshold to an appropriate threshold,
for example:
--terminated-pod-gc-threshold=10
--
1.3.2 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the below parameter.
--profiling=false
​
1.3.6 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the --feature-gates parameter to include RotateKubeletServerCertificate=true.
--feature-gates=RotateKubeletServerCertificate=true
```
在那里我们看到 1.3.2 建议设置--profiling=false，所以我们遵守：

```c
➜ root@cluster2-master1:~# vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```

Edit the corresponding line:

```c
# /etc/kubernetes/manifests/kube-controller-manager.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.244.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --node-cidr-mask-size=24
    - --port=0
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --use-service-account-credentials=true
    - --profiling=false            # add
...
```

我们等待Pod重新启动，然后再次运行kube-bench以检查问题是否已解决：
```c
➜ root@cluster2-master1:~# kube-bench master | grep kube-controller -A 3
1.3.1 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the --terminated-pod-gc-threshold to an appropriate threshold,
for example:
--terminated-pod-gc-threshold=10
--
1.3.6 Edit the Controller Manager pod specification file /etc/kubernetes/manifests/kube-controller-manager.yaml
on the master node and set the --feature-gates parameter to include RotateKubeletServerCertificate=true.
--feature-gates=RotateKubeletServerCertificate=true
```

问题已解决，1.3.2 已通过：

```c
root@cluster2-master1:~# kube-bench master | grep 1.3.2
[PASS] 1.3.2 Ensure that the --profiling argument is set to false (Scored)
```
2号
接下来的任务（2）是检查 directory 的所有权/var/lib/etcd，所以我们先看看：
```c
➜ root@cluster2-master1:~# ls -lh /var/lib | grep etcd
drwx------  3 root      root      4.0K Sep 11 20:08 etcd

➜ root@cluster2-master1:~# stat -c %U:%G /var/lib/etcd
root:root


➜ root@cluster2-master1:~# kube-bench master | grep "/var/lib/etcd" -B5
​
1.1.12 On the etcd server node, get the etcd data directory, passed as an argument --data-dir,
from the below command:
ps -ef | grep etcd
Run the below command (based on the etcd data directory found above).
For example, chown etcd:etcd /var/lib/etcd
```
为了遵守，我们运行以下命令：


```c
➜ root@cluster2-master1:~# chown etcd:etcd /var/lib/etcd
​
➜ root@cluster2-master1:~# ls -lh /var/lib | grep etcd
drwx------  3 etcd      etcd      4.0K Sep 11 20:08 etcd

➜ root@cluster2-master1:~# kube-bench master | grep 1.1.12
[PASS] 1.1.12 Ensure that the etcd data directory ownership is set to etcd:etcd (Scored)
Done.
```
3号
继续第 (3) 项，我们将前往工作节点并确保 kubelet 配置文件具有建议的最低必要权限：

```c
➜ ssh cluster2-worker1
​
➜ root@cluster2-worker1:~# kube-bench node
...
== Summary ==
13 checks PASS
10 checks FAIL
2 checks WARN
0 checks INFO


➜ root@cluster2-worker1:~# stat -c %a /var/lib/kubelet/config.yaml
777


➜ root@cluster2-worker1:~# kube-bench node | grep /var/lib/kubelet/config.yaml -B2
​
2.2.10 Run the following command (using the config file location identified in the Audit step)
chmod 644 /var/lib/kubelet/config.yaml
We obey and set the recommended permissions:

➜ root@cluster2-worker1:~# chmod 644 /var/lib/kubelet/config.yaml
​
➜ root@cluster2-worker1:~# stat -c %a /var/lib/kubelet/config.yaml
644

➜ root@cluster2-worker1:~# kube-bench node | grep 2.2.10
[PASS] 2.2.10 Ensure that the kubelet configuration file has permissions set to 644 or more restrictive (Scored)
```

4号
最后，对于数字 (4)，让我们检查是否 --client-ca-file根据建议正确设置了 kubelet 的参数kube-bench：
```c
➜ root@cluster2-worker1:~# kube-bench node | grep client-ca-file
[PASS] 2.1.4 Ensure that the --client-ca-file argument is set as appropriate (Scored)
2.2.7 Run the following command to modify the file permissions of the --client-ca-file
2.2.8 Run the following command to modify the ownership of the --client-ca-file .


➜ root@cluster2-worker1:~# ps -ef | grep kubelet
root      5157     1  2 20:28 ?        00:03:22 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
root     19940 11901  0 22:38 pts/0    00:00:00 grep --color=auto kubelet
​
➜ root@croot@cluster2-worker1:~# vim /var/lib/kubelet/config.yaml
# /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
...
```


----
## 问题 6 | 验证平台二进制文件
任务权重：2%

相关阅读：

 - [Kubernetes CKS 2021【7】---Cluster Setup - Verify Platform](https://ghostwritten.blog.csdn.net/article/details/116019776)

 

（可以在任何 kubectl 上下文中解决）

 

有四个 Kubernetes 服务器二进制文件位于`/opt/course/6/binaries`. 您将获得以下经过验证的 sha512 值：

kube-apiserver f417c0555bc0167355589dd1afe23be9bf909bf98312b1025f12015d1b58a1c62c9908c0067a7764fa35efdac7016a9efa8711a44425dd6692906a7c283f032c

kube-controller-manager 60100cc725e91fe1a949e1b2d0474237844b5862556e25c2c655a33boa8225855ec5ee22fa4927e6c46a60d43a7c4403a27268f96fbb726307d1608b44f38a60

kube代理 52f9d8ad045f8eee1d689619ef8ceef2d86d50c75a6a332653240d7ba5b2a114aca056d9e513984ade24358c9662714973c1960c62a5cb37dd375631c8a614c6

kubelet 4be40f2440619e990897cf956c32800dc96c2c983bf64519854a3309fa5aa21827991559f9c44595098e27e6f2ee4d64a3fdec6baba8a177881f20e3ec61e26c

删除那些与上面的 sha512 值不匹配的二进制文件。

 

回答：
我们检查目录：
```c
➜ cd /opt/course/6/binaries
​
➜ ls
kube-apiserver  kube-controller-manager  kube-proxy  kubelet
To generate the sha512 sum of a binary we do:

➜ sha512sum kube-apiserver 
f417c0555bc0167355589dd1afe23be9bf909bf98312b1025f12015d1b58a1c62c9908c0067a7764fa35efdac7016a9efa8711a44425dd6692906a7c283f032c  kube-apiserver
Looking good, next:

➜ sha512sum kube-controller-manager
60100cc725e91fe1a949e1b2d0474237844b5862556e25c2c655a33b0a8225855ec5ee22fa4927e6c46a60d43a7c4403a27268f96fbb726307d1608b44f38a60  kube-controller-manager
Okay, next:

➜ sha512sum kube-proxy
52f9d8ad045f8eee1d689619ef8ceef2d86d50c75a6a332653240d7ba5b2a114aca056d9e513984ade24358c9662714973c1960c62a5cb37dd375631c8a614c6  kube-proxy
Also good, and finally:

➜ sha512sum kubelet
7b720598e6a3483b45c537b57d759e3e82bc5c53b3274f681792f62e941019cde3d51a7f9b55158abf3810d506146bc0aa7cf97b36f27f341028a54431b335be  kubelet
```

但我们之前真的正确比较过所有东西吗？让我们再仔细看看kube-controller-manager：


```c
➜ sha512sum kube-controller-manager > compare
​
➜ vim compare 
```

Edit to only have the provided hash and the generated one in one line each:

```c
# ./compare
60100cc725e91fe1a949e1b2d0474237844b5862556e25c2c655a33b0a8225855ec5ee22fa4927e6c46a60d43a7c4403a27268f96fbb726307d1608b44f38a60  
60100cc725e91fe1a949e1b2d0474237844b5862556e25c2c655a33boa8225855ec5ee22fa4927e6c46a60d43a7c4403a27268f96fbb726307d1608b44f38a60


➜ cat compare | uniq
60100cc725e91fe1a949e1b2d0474237844b5862556e25c2c655a33b0a8225855ec5ee22fa4927e6c46a60d43a7c4403a27268f96fbb726307d1608b44f38a60
60100cc725e91fe1a949e1b2d0474237844b5862556e25c2c655a33boa8225855ec5ee22fa4927e6c46a60d43a7c4403a27268f96fbb726307d1608b44f38a60
```
为了完成任务，我们这样做：

```bash
rm kubelet kube-controller-manager
```



-----
## 问题 7 | 开放策略代理
任务权重：6%

 

使用上下文： `kubectl config use-context infra-prod`

 

已安装 `Open Policy Agent` 和 `Gatekeeper` 以强制将某些映像注册表列入黑名单。更改现有约束和/或模板以将`very-bad-registry.com`.

通过在`default` Namespace 中使用 image `very-bad-registry.com/image` 创建单个Pod来测试它，它应该不起作用。

您也可以通过查看现有的验证更改部署在命名空间，它使用来自新不可信源的图像。OPA 约束应该为该约束抛出违规消息。 untrusted default

 

回答：
我们查看现有的 OPA 约束，这些约束是由 Gatekeeper 使用 CRD 实现的：

```c
➜ k get crd
NAME                                                 CREATED AT
blacklistimages.constraints.gatekeeper.sh            2020-09-14T19:29:31Z
configs.config.gatekeeper.sh                         2020-09-14T19:29:04Z
constraintpodstatuses.status.gatekeeper.sh           2020-09-14T19:29:05Z
constrainttemplatepodstatuses.status.gatekeeper.sh   2020-09-14T19:29:05Z
constrainttemplates.templates.gatekeeper.sh          2020-09-14T19:29:05Z
requiredlabels.constraints.gatekeeper.sh             2020-09-14T19:29:31Z


➜ k get constraint
NAME                                                           AGE
blacklistimages.constraints.gatekeeper.sh/pod-trusted-images   10m
​
NAME                                                                  AGE
requiredlabels.constraints.gatekeeper.sh/namespace-mandatory-labels   10m


k edit blacklistimages pod-trusted-images
# kubectl edit blacklistimages pod-trusted-images
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: BlacklistImages
metadata:
...
spec:
  match:
    kinds:
    - apiGroups:
      - ""
      kinds:
      - Pod
```
看起来这个约束只是将模板应用于所有Pods，没有传递参数。所以我们编辑模板

```c
k edit constrainttemplates blacklistimages
# kubectl edit constrainttemplates blacklistimages
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
...
spec:
  crd:
    spec:
      names:
        kind: BlacklistImages
  targets:
  - rego: |
      package k8strustedimages
​
      images {
        image := input.review.object.spec.containers[_].image
        not startswith(image, "docker-fake.io/")
        not startswith(image, "google-gcr-fake.com/")
        not startswith(image, "very-bad-registry.com/") # ADD THIS LINE
      }
​
      violation[{"msg": msg}] {
        not images
        msg := "not trusted image!"
      }
    target: admission.k8s.gatekeeper.sh
```


我们只需要添加另一行。编辑后，我们尝试创建一个坏图像的Pod：
```c
➜ k run opa-test --image=very-bad-registry.com/image
Error from server ([denied by pod-trusted-images] not trusted image!): admission webhook 
```
好的！一段时间后，我们还可以看到，荚内的现有部署“不可信”，将被列为违法：
```c
➜ k describe blacklistimages pod-trusted-images
...
  Total Violations:  1
  Violations:
    Enforcement Action:  deny
    Kind:                Pod
    Message:             not trusted image!
    Name:                untrusted-68c4944d48-2hgt9
    Namespace:           default
Events:                  <none>
```
太好了，OPA 与不良注册表作斗争！

----

## 问题 8 | 安全 Kubernetes dashboard
任务权重：3%

相关阅读：

 - [Kubernetes CKS 2021【3】---Cluster Setup - Dashboard](https://ghostwritten.blog.csdn.net/article/details/115913408)

使用上下文： kubectl config use-context workload-prod

 

Kubernetes Dashboard 安装在Namespace 中并配置为： `kubernetes-dashboard`

 1. 允许用户“跳过登录”
 2. 允许不安全访问（无身份验证的 HTTP）
 3. 允许基本身份验证
 4. 允许从集群外部访问


要求您通过以下方式使其更安全：

 1. 拒绝用户“跳过登录”
 2. 拒绝不安全的访问，强制使用 HTTPS（自签名证书目前还可以）
 3. 添加`--auto-generate-certificates`参数
 4. 使用令牌强制身份验证（可以使用 RBAC）
 5. 只允许集群内部访问

 

回答：
前往[https://github.com/kubernetes/dashboard/tree/master/docs](https://github.com/kubernetes/dashboard/tree/master/docs)查找有关仪表板的文档。

首先我们看一下Namespace： `kubernetes-dashboard`

```c
➜ k -n kubernetes-dashboard get pod,svc
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-7b59f7d4df-fbpd9   1/1     Running   0          24m
pod/kubernetes-dashboard-6d8cd5dd84-w7wr2        1/1     Running   0          24m
​
NAME                                TYPE        ...   PORT(S)                        AGE
service/dashboard-metrics-scraper   ClusterIP   ...   8000/TCP                       24m
service/kubernetes-dashboard        NodePort    ...   9090:32520/TCP,443:31206/TCP   24m
```
我们可以看到一个正在运行的Pod和一个暴露它的 NodePort服务。让我们尝试通过 NodePort 连接到它，我们可以使用任何节点的IP ：

（您的端口可能不同）


```c
➜ k get node -o wide
NAME               STATUS   ROLES    AGE   VERSION   INTERNAL-IP    ...
cluster1-master1   Ready    master   37m   v1.19.1   192.168.100.11 ...
cluster1-worker1   Ready    <none>   36m   v1.19.1   192.168.100.12 ...
cluster1-worker2   Ready    <none>   34m   v1.19.1   192.168.100.13 ...
​
➜ curl http://192.168.100.11:32520
<!--
Copyright 2017 The Kubernetes Authors.
​
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
​
    http://www.apache.org/licenses/LICENSE-2.0
```

仪表板不安全，因为它允许未经身份验证的不安全 HTTP 访问，并且暴露在外部。它加载了一些参数，使其不安全，让我们解决这个问题。

首先，我们创建一个备份，以防我们需要撤消某些操作：

```c
k -n kubernetes-dashboard get deploy kubernetes-dashboard -oyaml > 8_deploy_kubernetes-dashboard.yaml

k -n kubernetes-dashboard edit deploy kubernetes-dashboard
```
The changes to make are :

```c
  template:
    spec:
      containers:
      - args:
        - --namespace=kubernetes-dashboard  
        - --authentication-mode=token        # change or delete, "token" is default
        - --auto-generate-certificates       # add
        #- --enable-skip-login=true          # delete or set to false
        #- --enable-insecure-login           # delete
        image: kubernetesui/dashboard:v2.0.3
        imagePullPolicy: Always
        name: kubernetes-dashboard
​
```
接下来，我们将不得不处理 NodePort服务：


```c
k -n kubernetes-dashboard get svc kubernetes-dashboard -o yaml > 8_svc_kubernetes-dashboard.yaml # backup
​
k -n kubernetes-dashboard edit svc kubernetes-dashboard
And make the following changes:

 spec:
  clusterIP: 10.107.176.19
  externalTrafficPolicy: Cluster
  ports:
  - name: http
    nodePort: 32513  # delete
    port: 9090
    protocol: TCP
    targetPort: 9090
  - name: https
    nodePort: 32441  # delete
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: ClusterIP          # change or delete
status:
  loadBalancer: {}
```
让我们确认更改，即使没有浏览器我们也可以做到：

```c
➜ k run tmp --image=nginx:1.19.2 --restart=Never --rm -it -- bash
If you don't see a command prompt, try pressing enter.
root@tmp:/# curl http://kubernetes-dashboard.kubernetes-dashboard:9090
curl: (7) Failed to connect to kubernetes-dashboard.kubernetes-dashboard port 9090: Connection refused
​
➜ root@tmp:/# curl https://kubernetes-dashboard.kubernetes-dashboard
curl: (60) SSL certificate problem: self signed certificate
More details here: https://curl.haxx.se/docs/sslcerts.html
​
curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
​
➜ root@tmp:/# curl https://kubernetes-dashboard.kubernetes-dashboard -k
<!--
Copyright 2017 The Kubernetes Authors.
```

我们看到不安全访问被禁用并且 HTTPS 有效（现在使用自签名证书）。让我们也检查一下远程访问：

（您的端口可能不同）

```c

➜ curl http://192.168.100.11:32520
curl: (7) Failed to connect to 192.168.100.11 port 32520: Connection refused
​
➜ k -n kubernetes-dashboard get svc
NAME                        TYPE        CLUSTER-IP       ...   PORT(S)
dashboard-metrics-scraper   ClusterIP   10.111.171.247   ...   8000/TCP
kubernetes-dashboard        ClusterIP   10.100.118.128   ...   9090/TCP,443/TCP
```
Much better.


-----
## 问题 9 | AppArmor 配置文件
任务权重：3%

相关阅读：

 - [Kubernetes CKS 2021 Course【24】---System Hardening - Kernel Hardening Tools](https://ghostwritten.blog.csdn.net/article/details/117249927)

 

使用上下文： `kubectl config use-context workload-prod`

 

一些容器需要运行更安全和更受限制。为此，有一个现有的 `AppArmor` 配置文件`/opt/course/9/profile`。

 1. 在Node `cluster1-worker1`上安装 AppArmor 配置文件。使用 `ssh cluster1-worker1`
 2. security=apparmor给节点添加标签
 3. Create a Deployment named `apparmor` in Namespace `default` with:
One replica of image `nginx:1.19.2`
NodeSelector for `security=apparmor`
Single container named `c1` with the `AppArmor` profile enabled
该吊舱与配置文件中启用可能无法正常运行。写的日志荚成`/opt/course/9/logs`这样其他球队可以让应用程序运行工作。

回答：
[https://kubernetes.io/docs/tutorials/clusters/apparmor](https://kubernetes.io/docs/tutorials/clusters/apparmor)

 

第1部分
首先我们看一下提供的配置文件：

```c
vim /opt/course/9/profile
```
```c
# /opt/course/9/profile 
​
#include <tunables/global>
  
profile very-secure flags=(attach_disconnected) {
  #include <abstractions/base>
​
  file,
​
  # Deny all file writes.
  deny /** w,
}
```
非常简单的配置文件very-secure，它拒绝所有文件写入。接下来我们将它复制到Node 上：

```c
➜ scp /opt/course/9/profile cluster1-worker1:~/
Warning: Permanently added the ECDSA host key for IP address '192.168.100.12' to the list of known hosts.
profile                                                                           100%  161   329.9KB/s   00:00
​
➜ ssh cluster1-worker1
​
➜ root@cluster1-worker1:~# ls
profile
```

And install it:

```c
➜ root@cluster1-worker1:~# apparmor_parser -q ./profile
Verify it has been installed:

➜ root@cluster1-worker1:~# apparmor_status
apparmor module is loaded.
17 profiles are loaded.
17 profiles are in enforce mode.
   /sbin/dhclient
...
   man_filter
   man_groff
   very-secure
0 profiles are in complain mode.
56 processes have profiles defined.
56 processes are in enforce mode.
...
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.
```
在那里，我们看到了许多其他的very-secure，它是 中指定的配置文件的名称/opt/course/9/profile。
第2部分
我们标记节点：
```c
k label -h # show examples
​
k label node cluster1-worker1 security=apparmor
```
第 3 部分
现在我们可以继续创建使用配置文件的部署。
```c
k create deploy apparmor --image=nginx:1.19.2 $do > 9_deploy.yaml
​
vim 9_deploy.yaml
# 9_deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: apparmor
  name: apparmor
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apparmor
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: apparmor
      annotations:                                                                 # add
        container.apparmor.security.beta.kubernetes.io/c1: localhost/very-secure   # add
    spec:
      nodeSelector:                    # add
        security: apparmor             # add
      containers:
      - image: nginx:1.19.2
        name: c1                       # change
        resources: {}
```

```c
k -f 9_deploy.yaml create
```
What the damage?
```c
➜ k get pod -owide | grep apparmor
apparmor-85c65645dc-w852p            0/1     CrashLoopBackOff   1          118s   10.44.0.18   cluster1-worker1   <none>           <none>
​
➜ k logs apparmor-85c65645dc-w852p
/docker-entrypoint.sh: No files found in /docker-entrypoint.d/, skipping configuration
/docker-entrypoint.sh: 13: /docker-entrypoint.sh: cannot create /dev/null: Permission denied
2020/09/26 18:14:11 [emerg] 1#1: mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
This looks alright, the Pod is running on cluster1-worker1 because of the nodeSelector. The AppArmor profile simply denies all filesystem writes, but Nginx needs to write into some locations to run, hence the errors.
```
这看起来没问题，由于 nodeSelector ，Pod正在运行cluster1-worker1。AppArmor 配置文件只是拒绝所有文件系统写入，但 Nginx 需要写入某些位置才能运行，因此会出现错误。

看起来我们的配置文件正在运行，但我们也可以通过检查 Docker 容器来确认这一点：

```c
➜ ssh cluster1-worker1
​
➜ root@cluster1-worker1:~# docker ps -a | grep apparmor
41f014a9e7a8        7e4d58f0e5f3                           "/docker-entrypoint.…"   About a minute ago   Exited (1) About a minute ago                       k8s_c1_apparmor-85c65645dc-w852p_default_3e209d37-74ee-43c5-8a5e-b61a683f068c_6
c79fe47d5a78        k8s.gcr.io/pause:3.2                   "/pause"                 7 minutes ago        Up 7 minutes                                        k8s_POD_apparmor-85c65645dc-w852p_default_3e209d37-74ee-43c5-8a5e-b61a683f068c_0
​
➜ root@cluster1-worker1:~# docker inspect 41f014a9e7a8 | grep -i profile
        "AppArmorProfile": "very-secure",
```
我们还需要使用docker ps -a来显示已停止的容器。然后docker inspect显示容器正在使用我们的 AppArmor 配置文件。另行通知之间的快速ps和inspect为K8S将重新启动波德定期时错误状态。

为了完成任务，我们将日志写入所需的位置：
```c
k logs apparmor-85c65645dc-w852p > /opt/course/9/logs
```



-----
## 问题 10 | 容器运行时沙盒 gVisor
任务权重：4%

相关阅读：

 - [Kubernetes CKS 2021 Course【13】---Microservice Vulnerabilities - Container Runtim Sandboxes](https://ghostwritten.blog.csdn.net/article/details/116230013)

使用上下文： `kubectl config use-context workload-prod`


紫色团队希望更安全地运行他们的一些工作负载。Worker 节点`cluster1-worker2`已经安装了容器引擎 `containerd` 并将其配置为支持 `runc/gvisor` 运行时。

该`cluster1-worker2`kubelet用途containerd代替泊坞窗。将 kubelet 配置为使用 containerd 的两个参数写入`/opt/course/10/arguments`.

创建一个以handler命名的`RuntimeClass`with handler `runsc`.

Create a Pod that uses the RuntimeClass. The Pod should be in Namespace `team-purple`, named `gvisor-test` and of image `nginx:1.19.2`. Make sure the Pod runs on `cluster1-worker2`.

dmesg将成功启动的Pod的输出写入`/opt/course/10/gvisor-test-dmesg`.

 

回答：
我们检查节点，可以看到 worker2 使用 containerd 运行：
```bash
➜ k get node -o wide
NAME               STATUS   ROLES    AGE   VERSION ... CONTAINER-RUNTIME
cluster1-master1   Ready    master   9h    v1.19.1 ... docker://19.3.6
cluster1-worker1   Ready    <none>   9h    v1.19.1 ... docker://19.3.6
cluster1-worker2   Ready    <none>   9h    v1.19.1 ... containerd://1.3.3
```

首先，我们通过 ssh 进入工作节点，并可选择检查是否按照任务中的描述安装和配置了 containerd 和 runc：

```c
➜ ssh cluster1-worker2
​
➜ root@cluster1-worker2:~# runsc --version
runsc version release-20201130.0
spec: 1.0.1-dev
​
➜ root@cluster1-worker2:~# service containerd status
● containerd.service - containerd container runtime
   Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-09-03 15:58:22 UTC; 2min 36s ago
   ...
​
➜ root@cluster1-worker2:~# cat /etc/containerd/config.toml
disabled_plugins = ["restart"]
[plugins.linux]
  shim_debug = true
[plugins.cri.containerd.runtimes.runsc]
  runtime_type = "io.containerd.runsc.v1"
```
看起来不错。接下来我们检查 kubelet 的参数。

```c
# defines how the kubelet is started
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
We see that it references file /etc/default/kubelet.

# /etc/default/kubelet
KUBELET_EXTRA_ARGS="--container-runtime remote --container-runtime-endpoint unix:///run/containerd/containerd.sock"
```
我们将这些写入主终端上所需的位置：
```c
# /opt/course/10/arguments
--container-runtime remote
--container-runtime-endpoint unix:///run/containerd/containerd.sock
 
```
对于下一个要求，最好前往RuntimeClasses [https://kubernetes.io/docs/concepts/containers/runtime-class](https://kubernetes.io/docs/concepts/containers/runtime-class/)的 k8s 文档，窃取一个示例并创建 gvisor 一个：

```c
vim 10_rtc.yaml
```
```c
# 10_rtc.yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc


k -f 10_rtc.yaml create

```

```c
k -n team-purple run gvisor-test --image=nginx:1.19.2 --dry-run=client -o yaml > 10_pod.yaml

```c
vim 10_pod.yaml
# 10_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: gvisor-test
  name: gvisor-test
  namespace: team-purple
spec:
  nodeName: cluster1-worker2 # add
  runtimeClassName: gvisor   # add
  containers:
  - image: nginx:1.19.2
    name: gvisor-test
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```c
k -f 10_pod.yaml create

```
创建 pod 后，我们应该检查它是否正在运行以及它是否使用 gvisor 沙箱：
```c
➜ k -n team-purple get pod gvisor-test
NAME          READY   STATUS    RESTARTS   AGE
gvisor-test   1/1     Running   0          30s
​
➜ k -n team-purple exec gvisor-test -- dmesg
[    0.000000] Starting gVisor...
[    0.417740] Checking naughty and nice process list...
[    0.623721] Waiting for children...
[    0.902192] Gathering forks...
[    1.258087] Committing treasure map to memory...
[    1.653149] Generating random numbers by fair dice roll...
[    1.918386] Creating cloned children...
[    2.137450] Digging up root...
[    2.369841] Forking spaghetti code...
[    2.840216] Rewriting operating system in Javascript...
[    2.956226] Creating bureaucratic processes...
[    3.329981] Ready!
```

看起来不错。根据需要，我们最终将dmesg输出写入文件：
```c
k -n team-purple exec gvisor-test > /opt/course/10/gvisor-test-dmesg -- dmesg
```

-----
## 问题 11 | ETCD 中的秘密
任务权重：7%

相关阅读：

 - [Kubernetes CKS 2021【12】---Microservice 漏洞-Manage secrets](https://ghostwritten.blog.csdn.net/article/details/116200609)

使用上下文： `kubectl config use-context workload-prod`
 
有一个现有的秘密被称为`database-access`在命名空间`team-green`

直接从 ETCD读取完整的Secret内容（使用etcdctl）并将其存储到`/opt/course/11/etcd-secret-content`. 将密钥“pass”的明文和解码的Secret值写入`/opt/course/11/database-password`.

 

回答：
让我们尝试直接从 ETCD获取Secret值，这将起作用，因为它没有加密。

首先，我们通过 ssh 进入在此设置中运行 ETCD 的主节点，并检查是否etcdctl已安装并列出其选项：
```bash
➜ ssh cluster1-master1
​
➜ root@cluster1-master1:~# etcdctl


NAME:
   etcdctl - A simple command line client for etcd.
​
WARNING:
   Environment variable ETCDCTL_API is not set; defaults to etcdctl v2.
   Set environment variable ETCDCTL_API=3 to use v3 API or ETCDCTL_API=2 to use v2 API.
​
USAGE:
   etcdctl [global options] command [command options] [arguments...]
...
   --cert-file value   identify HTTPS client using this SSL certificate file
   --key-file value    identify HTTPS client using this SSL key file
   --ca-file value     verify certificates of HTTPS-enabled servers using this CA bundle
...
```
其中，我们看到了证明自己身份的论据。apiserver 连接到 ETCD，因此我们可以运行以下命令来获取必要的 .crt 和 .key 文件的路径：

```c
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
```

The output is as follows :

    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379 # optional since we're on same node


有了这些信息，我们查询 ETCD 的秘密值：
```c
➜ root@cluster1-master1:~# ETCDCTL_API=3 etcdctl \
--cert /etc/kubernetes/pki/apiserver-etcd-client.crt \
--key /etc/kubernetes/pki/apiserver-etcd-client.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt get /registry/secrets/team-green/database-access
```
Kubernetes 中的 ETCD 将数据存储在/registry/{type}/{namespace}/{name}. 我们就是这样来寻找的/registry/secrets/team-green/database-access。在 k8s 文档的页面上还有一个示例，您可以将其保存为书签以便在考试期间快速访问。

这些任务要求我们将输出存储在我们的终端上。为此，我们可以简单地将内容复制并粘贴到终端上的新文件中：

```c
# /opt/course/11/etcd-secret-content
/registry/secrets/team-green/database-access
k8s
​
​
v1Secret
​
database-access
team-green"*$3e0acd78-709d-4f07-bdac-d5193d0f2aa32bB
0kubectl.kubernetes.io/last-applied-configuration{"apiVersion":"v1","data":{"pass":"Y29uZmlkZW50aWFs"},"kind":"Secret","metadata":{"annotations":{},"name":"database-access","namespace":"team-green"}}
z
kubectl-client-side-applyUpdatevFieldsV1:
{"f:data":{".":{},"f:pass":{}},"f:metadata":{"f:annotations":{".":{},"f:kubectl.kubernetes.io/last-applied-configuration":{}}},"f:type":{}}
pass
    confidentialOpaque"
```
我们还需要存储明文和“解密”的数据库密码。为此，我们可以从 ETCD 输出中复制 base64 编码的值并在我们的终端上运行：

```c
➜ echo Y29uZmlkZW50aWFs | base64 -d > /opt/course/11/database-password
​
➜ cat /opt/course/11/database-password
confidential
```

-----
## 问题 12 | 黑客秘密
任务权重：8%

使用上下文： `kubectl config use-context restricted@infra-prod`

 

您需要调查Namespace  `restricted`中可能的权限逃逸。上下文以用户身份进行身份验证，该用户 `restricted`仅具有有限的权限并且不应能够读取Secret值。

试图找到的密码键值秘密，并在命名空间。将解码后的明文值写入文件,和. secret1  secret2    secret3 restricted/opt/course/12/secret1    /opt/course/12/secret2     /opt/course/12/secret3

 

回答：
首先我们应该探索边界，我们可以尝试：
```c
➜ k -n restricted get role,rolebinding,clusterrole,clusterrolebinding
Error from server (Forbidden): roles.rbac.authorization.k8s.io is forbidden: User "restricted" cannot list resource "roles" in API group "rbac.authorization.k8s.io" in the namespace "restricted"
Error from server (Forbidden): rolebindings.rbac.authorization.k8s.io is forbidden: User "restricted" cannot list resource "rolebindings" in API group "rbac.authorization.k8s.io" in the namespace "restricted"
Error from server (Forbidden): clusterroles.rbac.authorization.k8s.io is forbidden: User "restricted" cannot list resource "clusterroles" in API group "rbac.authorization.k8s.io" at the cluster scope
Error from server (Forbidden): clusterrolebindings.rbac.authorization.k8s.io is forbidden: User "restricted" cannot list resource "clusterrolebindings" in API group "rbac.authorization.k8s.io" at the cluster scope
```
但没有查看 RBAC 资源的权限。所以我们尝试显而易见的：

```c
➜ k -n restricted get secret
Error from server (Forbidden): secrets is forbidden: User "restricted" cannot list resource "secrets" in API group "" in the namespace "restricted"
​
➜ k -n restricted get secret -o yaml
apiVersion: v1
items: []
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
Error from server (Forbidden): secrets is forbidden: User "restricted" cannot list resource "secrets" in API group "" in the namespace "restricted"
```

我们不允许获取或列出任何Secrets。我们能看到什么？
```c
➜ k -n restricted get all
NAME                    READY   STATUS    RESTARTS   AGE
pod1-fd5d64b9c-pcx6q    1/1     Running   0          37s
pod2-6494f7699b-4hks5   1/1     Running   0          37s
pod3-748b48594-24s76    1/1     Running   0          37s
Error from server (Forbidden): replicationcontrollers is forbidden: User "restricted" cannot list resource "replicationcontrollers" in API group "" in the namespace "restricted"
Error from server (Forbidden): services is forbidden: User "restricted" cannot list resource "services" in API group "" in the namespace "restricted"
...
```
有一些Pods，让我们看看这些关于Secret访问：

```bash
k -n restricted get pod -o yaml | grep -i secret
```

此输出为我们提供了足够的信息来执行以下操作：

```c
➜ k -n restricted exec pod1-fd5d64b9c-pcx6q -- cat /etc/secret-volume/password
you-are

➜ echo you-are > /opt/course/12/secret1
And for the second Secret:

➜ k -n restricted exec pod2-6494f7699b-4hks5 -- env | grep PASS
PASSWORD=an-amazing
​
➜ echo an-amazing > /opt/course/12/secret2
```
不过，似乎没有一个Pod挂载secret3。我们可以创建或编辑现有的Pod来挂载secret3吗？
```c
➜ k -n restricted run test --image=nginx
Error from server (Forbidden): pods is forbidden: User "restricted" cannot create resource "pods" in API group "" in the namespace "restricted"
​
➜ k -n restricted delete pod pod1
Error from server (Forbidden): pods "pod1" is forbidden: User "restricted" cannot delete resource "pods" in API group "" in the namespace "restricted"
```

看起来不像

但是Pods似乎可以访问Secrets，我们可以尝试使用Pod的ServiceAccount来访问第三个Secret。我们实际上可以看到（如使用）只有Pod挂载了ServiceAccount令牌： `k -n restricted get pod -o yaml | grep automountServiceAccountToken pod3-*`

```c
➜ k -n restricted exec -it pod3-748b48594-24s76 -- sh
​
/ # mount | grep serviceaccount
tmpfs on /run/secrets/kubernetes.io/serviceaccount type tmpfs (ro,relatime)
​
/ # ls /run/secrets/kubernetes.io/serviceaccount
ca.crt     namespace  token
```

> 注意： 您应该了解ServiceAccounts以及它们如何与Pods一起工作，如文档中所述

 

我们可以看到手动联系 apiserver 的所有必要信息：

```c
/ # curl https://kubernetes.default/api/v1/namespaces/restricted/secrets -H "Authorization: Bearer $(cat /run/secrets/kubernetes.io/serviceaccount/token)" -k
...
    {
      "metadata": {
        "name": "secret3",
        "namespace": "restricted",
...
          }
        ]
      },
      "data": {
        "password": "cEVuRXRSYVRpT24tdEVzVGVSCg=="
      },
      "type": "Opaque"
    }
...
```

让我们对其进行编码并将其写入请求的位置：

```bash
➜ echo cEVuRXRSYVRpT24tdEVzVGVSCg== | base64 -d
pEnEtRaTiOn-tEsTeR
​
➜ echo cEVuRXRSYVRpT24tdEVzVGVSCg== | base64 -d > /opt/course/12/secret3
```

这会给我们：
```c
# /opt/course/12/secret1
you-are
# /opt/course/12/secret2
an-amazing
# /opt/course/12/secret3
pEnEtRaTiOn-tEsTeR
```

我们破解了所有的秘密！获得正确和安全的 RBAC 可能很棘手。

需要考虑的一件事是，授予“列出” Secrets的权限，也将允许用户读取Secret值，`kubectl get secrets -o yaml`即使没有“获取”权限集也可以使用。

----

## 问题 13 | 限制对元数据服务器的访问
任务权重：7%

使用上下文： `kubectl config use-context infra-prod`

 

有可用的元数据服务在`http://192.168.100.21:32000`哪个节点能够达到的敏感数据，像初始化云凭据。默认情况下，集群中的所有Pod也可以访问此端点。DevSecOps 团队要求您限制对该元数据服务器的访问。

在命名空间中： `metadata-access`

 - 创建一个名为NetworkPolicy 的网络策略，以`metadata-deny`阻止192.168.100.21所有Pod的出口，但仍允许访问其他所有内容
 - 创建一个名为NetworkPolicy 的网络策略，允许具有标签的Pod访问端点`metadata-allow`role:
   metadata-accessor192.168.100.21

目标命名空间中有现有的Pod，您可以使用它们来测试您的策略，但不要更改它们的标签。

 

回答：
Spotify有一个著名的黑客攻击，它基于通过节点元数据显示的信息。

 

检查豆荚的命名空间和其标签： `metadata-access`

```c
➜ k -n metadata-access get pods --show-labels
NAME                    ...   LABELS
pod1-7d67b4ff9-xrcd7    ...   app=pod1,pod-template-hash=7d67b4ff9
pod2-7b6fc66944-2hc7n   ...   app=pod2,pod-template-hash=7b6fc66944
pod3-7dc879bd59-hkgrr   ...   app=pod3,role=metadata-accessor,pod-template-hash=7dc879bd59
```
命名空间中有三个Pod，其中一个带有标签。role=metadata-accessor

检查从Pod对元数据服务器的访问：
```bash
➜ k exec -it -n metadata-access pod1-7d67b4ff9-xrcd7 -- curl http://192.168.100.21:32000
metadata server
​
➜ k exec -it -n metadata-access pod2-7b6fc66944-2hc7n -- curl http://192.168.100.21:32000
metadata server
​
➜ k exec -it -n metadata-access pod3-7dc879bd59-hkgrr -- curl http://192.168.100.21:32000
metadata server
```
所有三个都能够访问元数据服务器。

为了限制访问，我们创建了一个NetworkPolicy来拒绝对特定 IP 的访问。

```c
vim 13_metadata-deny.yaml
# 13_metadata-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: metadata-deny
  namespace: metadata-access
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 192.168.100.21/32
```

```bash
k -f 13_metadata-deny.yaml apply
```
注意： 您应该了解一般默认拒绝 K8s NetworkPolcies。

验证对元数据服务器的访问是否已被阻止，但其他端点仍可访问：

```c
➜ k exec -it -n metadata-access pod1-7d67b4ff9-xrcd7 -- curl http://192.168.100.21:32000
curl: (28) Failed to connect to 192.168.100.21 port 32000: Operation timed out
command terminated with exit code 28
​
➜ kubectl exec -it -n metadata-access pod1-7d67b4ff9-xrcd7 -- curl -I https://kubernetes.io
HTTP/2 200
cache-control: public, max-age=0, must-revalidate
content-type: text/html; charset=UTF-8
date: Mon, 14 Sep 2020 15:39:39 GMT
etag: "b46e429397e5f1fecf48c10a533f5cd8-ssl"
strict-transport-security: max-age=31536000
age: 13
content-length: 22252
server: Netlify
x-nf-request-id: 1d94a1d1-6bac-4a98-b065-346f661f1db1-393998290
```
同样，验证其他两个Pods。

现在创建另一个NetworkPolicy，允许从带有标签的Pod访问元数据服务器。`role=metadata-accessor`


```c
vim 13_metadata-allow.yaml
# 13_metadata-allow.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: metadata-allow
  namespace: metadata-access
spec:
  podSelector:
    matchLabels:
      role: metadata-accessor
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.100.21/32
```
```bash
k -f 13_metadata-allow.yaml apply
```
验证所需的Pod是否可以访问元数据端点，而其他Pod没有：
```bash
➜ k -n metadata-access exec pod3-7dc879bd59-hkgrr -- curl http://192.168.100.21:32000
metadata server
​
➜ k -n metadata-access exec pod2-7b6fc66944-9ngzr  -- curl http://192.168.100.21:32000
^Ccommand terminated with exit code 130
```
它仅适用于具有标签的Pod。有了这个，我们实施了所需的安全限制。

如果Pod没有匹配的NetworkPolicy，则允许所有流量进出它。一旦Pod具有匹配的NP，则包含的规则是可加的。这意味着对于具有标签的Pod，`metadata-accessor`规则将组合为：
```bash
# merged policies into one for pods with label metadata-accessor
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to: # first rule
    - ipBlock: # condition 1
        cidr: 0.0.0.0/0
        except:
        - 192.168.100.21/32
  - to: # second rule
    - ipBlock: # condition 1
        cidr: 192.168.100.21/32
```

我们可以看到合并后的NP包含两个单独的规则，每个规则都有一个条件。我们可以这样读：
(destination is 0.0.0.0/0 but not 192.168.100.21/32) OR (destination is 192.168.100.21/32)
因此，它允许带有标签的Pod访问所有内容。metadata-accessor

----
## 第 14 题 | 系统调用活动
任务权重：4%


使用上下文： `kubectl config use-context workload-prod`

 

有pod的命名空间。一个安全的调查发现，在这些运行某些程序荚使用的系统调用，它是由一组黄色的内部政策禁止的 Syscall kill，
找到有问题的Pod并通过将父部署的副本减少到 0 来删除它们。

 

回答：
在用户空间中运行的进程使用系统调用与 Linux 内核进行通信。有许多可用的系统调用： [https://man7.org/linux/man-pages/man2/syscalls.2.html](https://man7.org/linux/man-pages/man2/syscalls.2.html)。为容器进程限制这些是有意义的，Docker 默认已经限制了一些，比如rebootSyscall。限制更多是可能的，例如使用 `Seccomp` 或 `AppArmor`。

但是对于这个任务，我们应该简单地找出哪个二进制进程执行特定的 Syscall。容器中的进程只是在同一个 Linux 操作系统上运行，但相互隔离。这就是我们首先检查Pod正在运行哪些节点的原因：

```bash
➜ k -n team-yellow get pod -owide
NAME                          ...  NODE               NOMINATED NODE   READINESS GATES
collector1-59ddbd6c7f-ffjjv   ...  cluster1-worker1   <none>           <none>
collector1-59ddbd6c7f-p9qqz   ...  cluster1-worker1   <none>           <none>
collector2-7b6868b5dc-h6zxx   ...  cluster1-worker1   <none>           <none>
collector3-77b7c5bf47-5hgcb   ...  cluster1-worker1   <none>           <none>
collector3-77b7c5bf47-rswrl   ...  cluster1-worker1   <none>           <none>
```

一切cluster1-worker1就绪，因此我们 ssh 进入它并找到第一个Deployment的进程。 collector1

```bash
➜ ssh cluster1-worker1
​
➜ root@cluster1-worker1:~# docker ps | grep collector1
3e07aee08a48        registry.killer.sh:5000/collector1     "./collector1-process"   14 seconds ago       Up 13 seconds                           k8s_c_collector1-59ddbd6c7f-vvwgt_team-yellow_099822ad-2dfc-4963-bfc7-d806e00c0daf_0
...
```
部署有两个副本，我们可以看到进程执行。我们可以找到进程PID： collector1./collector1-process

```bash
➜ root@cluster1-worker1:~# ps aux | grep collector1-process
root     10991  0.0  0.0   2412   760 ?        Ssl  22:41   0:00 ./collector1-process
root     11150  0.0  0.0   2412   756 ?        Ssl  22:41   0:00 ./collector1-process
```
使用我们可以调用的 PIDstrace来查找 Sycall：

```bash
➜ root@cluster1-worker1:~# strace -p 10991
strace: Process 10991 attached
restart_syscall(<... resuming interrupted futex ...>) = -1 ETIMEDOUT (Connection timed out)
futex(0x4ad5d0, FUTEX_WAKE, 1)          = 1
kill(666, SIGTERM)                      = -1 ESRCH (No such process)
futex(0xc420030948, FUTEX_WAKE, 1)      = 1
futex(0x4afe80, FUTEX_WAIT, 0, {tv_sec=0, tv_nsec=999998945}) = -1 ETIMEDOUT (Connection timed out)
...
```

第一次尝试，已经成功了！我们看到它通过调用kill(666, SIGTERM).

接下来让我们检查部署过程： collector2

```c
➜ root@cluster1-worker1:~# docker ps | grep collector2
60531737cb83        registry.killer.sh:5000/collector2     "./collector2-process"   4 minutes ago       Up 4 minutes                            k8s_c_collector2-7b6868b5dc-
...
​
➜ root@cluster1-worker1:~# ps aux | grep collector2-process
root     10438  0.0  0.0   2420   764 ?        Ssl  20:19   0:00 ./collector2-process
root     26442  0.0  0.0  14856  1000 pts/0    S+   21:13   0:00 grep --color=auto collector2-process
​
➜ root@cluster1-worker1:~# strace -p 10438
strace: Process 11080 attached
restart_syscall(<... resuming interrupted futex ...>) = -1 ETIMEDOUT (Connection timed out)
futex(0x4af3d0, FUTEX_WAKE, 1)          = 1
futex(0x4af2f0, FUTEX_WAKE, 1)          = 1
futex(0xc420030548, FUTEX_WAKE, 1)      = 1
futex(0x4b1c80, FUTEX_WAIT, 0, {tv_sec=0, tv_nsec=999999578}) = -1 ETIMEDOUT (Connection timed out)
...
```

看起来没问题。怎么样collector3：
```bash
➜ root@cluster1-worker1:~# docker ps | grep collector3
4db52d9aa69e        registry.killer.sh:5000/collector3     "./collector3-process"   6 minutes ago       Up 6 minutes                            k8s_c_collector3-77b7c5bf47-
...
​
➜ root@cluster1-worker1:~# ps aux | grep collector3-process
root     10915  0.0  0.0   2428   756 ?        Ssl  22:41   0:00 ./collector3-process
root     11135  0.0  0.0   2428   760 ?        Ssl  22:41   0:00 ./collector3-process
root     13374  0.0  0.1  14856  1104 pts/0    S+   22:49   0:00 grep --color=auto collector3-process
​
➜ root@cluster1-worker1:~# strace -p 10915
strace: Process 10915 attached
restart_syscall(<... resuming interrupted futex ...>) = -1 ETIMEDOUT (Connection timed out)
futex(0x4b13d0, FUTEX_WAKE, 1)          = 1
futex(0x4b12f0, FUTEX_WAKE, 1)          = 1
futex(0xc420030548, FUTEX_WAKE, 1)      = 1
futex(0x4b3c80, FUTEX_WAIT, 0, {tv_sec=0, tv_nsec=999999504}) = -1 ETIMEDOUT (Connection timed out)
```

也没有关于禁止的系统调用。所以我们完成了任务：
```bash
k -n team-yellow scale deploy collector1 --replicas 0
```

----------------

## 问题 15 | 在 Ingress 上配置 TLS
任务权重：4%

相关阅读：

 - [Kubernetes CKS 2021【4】---Cluster Setup - Secure Ingress](https://ghostwritten.blog.csdn.net/article/details/115949825)

 

使用上下文： `kubectl config use-context workload-prod`

 

在Namespace team-pink`中有一个现有的 Nginx Ingress资源，它被命名为接受两条路径并指向不同的 ClusterIP服务。 team-pinksecure/app/api

您可以从主终端连接到它，例如：

 - HTTP： curl -v http://secure-ingress.test:31080/app
 - HTTPS： curl -kv https://secure-ingress.test:31443/app

现在它使用 Nginx 入口控制器默认生成的 TLS 证书。

你被要求改用在所提供的密钥和证书`/opt/course/15/tls.key`以及`/opt/course/15/tls.crt`。由于它是一个自签名证书，因此您在连接到它时需要使用`curl -k`它。


回答：
调查
我们可以获取Ingress的 IP 地址，我们看到它与指向的 IP 地址相同`secure-ingress.test`：

```bash
➜ k -n team-pink get ing secure 
NAME     CLASS    HOSTS                 ADDRESS          PORTS   AGE
secure   <none>   secure-ingress.test   192.168.100.12   80      7m11s
​
➜ ping secure-ingress.test
PING cluster1-worker1 (192.168.100.12) 56(84) bytes of data.
64 bytes from cluster1-worker1 (192.168.100.12): icmp_seq=1 ttl=64 time=0.316 ms
```
现在，让我们尝试访问的路径`/app`，并`/api`通过HTTP：
```bash
➜ curl http://secure-ingress.test:31080/app
This is the backend APP!
​
➜ curl http://secure-ingress.test:31080/api
This is the API Server!
```
HTTPS 呢？
```bash
➜ curl https://secure-ingress.test:31443/api
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html
​
curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
​
➜ curl -k https://secure-ingress.test:31443/api
This is the API Server!
```
如果我们接受使用-k. 但是服务器使用什么样的证书呢？

```c
➜ curl -kv https://secure-ingress.test:31443/api
...
* Server certificate:
*  subject: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  start date: Sep 28 12:28:35 2020 GMT
*  expire date: Sep 28 12:28:35 2021 GMT
*  issuer: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
...
```
它似乎是“Kubernetes Ingress Controller Fake Certificate”。

实现自己的 TLS 证书
首先，让我们使用提供的密钥和证书生成一个Secret：
```c
➜ cd /opt/course/15
​
➜ :/opt/course/15$ ls
tls.crt  tls.key
​
➜ :/opt/course/15$ k -n team-pink create secret tls tls-secret --key tls.key --cert tls.crt
secret/tls-secret created
```
现在，我们配置Ingress以使用此Secret：
```c
➜ k -n team-pink get ing secure -oyaml > 15_ing_bak.yaml
​
➜ k -n team-pink edit ing secure
# kubectl -n team-pink edit ing secure
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
...
  generation: 1
  name: secure
  namespace: team-pink
...
spec:
  tls:                            # add
    - hosts:                      # add
      - secure-ingress.test       # add
      secretName: tls-secret      # add
  rules:
  - host: secure-ingress.test
    http:
      paths:
      - backend:
          service:
            name: secure-app
            port: 80
        path: /app
        pathType: ImplementationSpecific
      - backend:
          service:
            name: secure-api
            port: 80
        path: /api
        pathType: ImplementationSpecific
...
```
添加更改后，我们再次检查Ingress资源：
```bash
➜ k -n team-pink get ing
NAME     CLASS    HOSTS                 ADDRESS          PORTS     AGE
secure   <none>   secure-ingress.test   192.168.100.12   80, 443   25m
```
它现在实际上列出了 HTTPS 的 443 端口。确认：
```c
➜ curl -k https://secure-ingress.test:31443/api
This is the API Server!
​
➜ curl -kv https://secure-ingress.test:31443/api
...
* Server certificate:
*  subject: CN=secure-ingress.test; O=secure-ingress.test
*  start date: Sep 25 18:22:10 2020 GMT
*  expire date: Sep 20 18:22:10 2040 GMT
*  issuer: CN=secure-ingress.test; O=secure-ingress.test
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
...
```
我们可以看到提供的证书现在被Ingress用于 TLS 终止。

----------------

## 问题 16 | Docker 镜像攻击面
任务权重：7%

使用上下文： `kubectl config use-context workload-prod`


有一个部署在命名空间它运行图像。DevSecOps 要求您通过以下方式改进此图像： `image-verify` `team-blueregistry.killer.sh:5000/image-verify:v1`

 1. 将基本图像更改为 alpine:3.12
 2. 未安装 curl
 3. 更新nginx以使用版本约束 >=1.18.0
 4. 以用户身份运行主进程 myuser

不要在 Dockerfile 中添加任何新行，只需编辑现有的行。该文件位于`/opt/course/16/image/Dockerfile.`

将您的版本标记为v2. 您可以使用以下方法构建、标记和推送：

```bash
cd /opt/course/16/image
sudo docker build -t registry.killer.sh:5000/image-verify:v2 .
sudo docker run registry.killer.sh:5000/image-verify:v2 # to test your changes
sudo docker push registry.killer.sh:5000/image-verify:v2
```
使部署使用您更新的图像标签v2。

 

回答：
我们首先应该看看Docker Image：
```bash
cd /opt/course/16/image
​
cp Dockerfile Dockerfile.bak
​
vim Dockerfile
# /opt/course/16/image/Dockerfile
FROM alpine:3.4
RUN apk update && apk add vim curl nginx=1.10.3-r0
RUN addgroup -S myuser && adduser -S myuser -G myuser
COPY ./run.sh run.sh
RUN ["chmod", "+x", "./run.sh"]
USER root
ENTRYPOINT ["/bin/sh", "./run.sh"]
```
非常简单的 Dockerfile，它似乎执行了一个脚本 run.sh：
```bash
# /opt/course/16/image/run.sh
while true; do date; id; echo; sleep 1; done
```
所以它只在循环中输出当前日期和凭证信息。我们可以在现有部署中看到输出 image-verify：
```bash
➜ k -n team-blue logs -f -l id=image-verify
Fri Sep 25 20:59:12 UTC 2020
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```
我们看到它以 root 身份运行。

接下来我们根据要求更新Dockerfile：

```c
# /opt/course/16/image/Dockerfile
​
# change
FROM alpine:3.12
​
# change
RUN apk update && apk add vim nginx>=1.18.0
​
RUN addgroup -S myuser && adduser -S myuser -G myuser
COPY ./run.sh run.sh
RUN ["chmod", "+x", "./run.sh"]
​
# change
USER myuser
​
ENTRYPOINT ["/bin/sh", "./run.sh"]
```
然后我们构建新镜像：
```c
➜ :/opt/course/16/image$ sudo docker build -t registry.killer.sh:5000/image-verify:v2 .
Sending build context to Docker daemon  3.072kB
...
Successfully built a5df16d42c5b
Successfully tagged registry.killer.sh:5000/image-verify:v2
We can then test our changes by running the container locally:

➜ :/opt/course/16/image$ sudo docker run registry.killer.sh:5000/image-verify:v2 
Fri Sep 25 21:02:09 UTC 2020
uid=101(myuser) gid=102(myuser) groups=102(myuser)
Looking good, so we push:

➜ :/opt/course/16/image$ sudo docker push registry.killer.sh:5000/image-verify:v2
The push refers to repository [registry.killer.sh:5000/image-verify]
bf21d6611c7c: Layer already exists 
82eb465441ab: Layer already exists 
f88b13f57e3a: Pushed 
32099b2fa646: Pushed 
50644c29ef5a: Pushed 
v2: digest: sha256:867c1fded95faeec9e73404e822f6ed001b83163bd1e86f8945e8c00a758fdae size: 1362
```

我们更新部署以使用新镜像：
```c
k -n team-blue edit deploy image-verify
# kubectl -n team-blue edit deploy image-verify
apiVersion: apps/v1
kind: Deployment
metadata:
...
spec:
...
  template:
...
    spec:
      containers:
      - image: registry.killer.sh:5000/image-verify:v2 # change
```

之后我们可以通过查看Pod日志来验证我们的更改：


```bash
➜ k -n team-blue logs -f -l id=image-verify
Fri Sep 25 21:06:55 UTC 2020
uid=101(myuser) gid=102(myuser) groups=102(myuser)
Also to verify our changes even further:

➜ k -n team-blue exec image-verify-55fbcd4c9b-x2flc -- curl
OCI runtime exec failed: exec failed: container_linux.go:349: starting container process caused "exec: \"curl\": executable file not found in $PATH": unknown
command terminated with exit code 126
​
➜ k -n team-blue exec image-verify-55fbcd4c9b-x2flc -- nginx -v
nginx version: nginx/1.18.0
```


----
## 问题 17 | 审计日志策略
任务权重：7%
相关阅读：

 - [https://kubernetes.io/docs/tasks/debug-application-cluster/audit/](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)
 - [Kubernetes CKS 2021 Course【23】---Runtime Security - Auditing](https://ghostwritten.blog.csdn.net/article/details/117225475)

使用上下文： `kubectl config use-context infra-prod`

审计日志记录功能在集群中被启用了审计策略位于`/etc/kubernetes/audit/policy.yaml上cluster2-master1`。

更改配置，以便仅存储一份日志备份。

以仅存储日志的方式更改策略：

 1. From `Secret resources`, `level Metadata`
 2. From "`system:nodes`" userGroups, level `RequestResponse`
更改策略后，请确保清空日志文件，使其仅包含根据您所做更改的条目，例如使用`truncate -s 0 /etc/kubernetes/audit/logs/audit.log`.

回答：
首先，我们检查 apiserver 配置并根据要求进行更改：
```c
➜ ssh cluster2-master1
​
➜ root@cluster2-master1:~# cp /etc/kubernetes/manifests/kube-apiserver.yaml ~/17_kube-apiserver.yaml # backup
​
➜ root@cluster2-master1:~# vim /etc/kubernetes/manifests/kube-apiserver.yaml 
# /etc/kubernetes/manifests/kube-apiserver.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.100.21:6443
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
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml
    - --audit-log-path=/etc/kubernetes/audit/logs/audit.log
    - --audit-log-maxsize=5
    - --audit-log-maxbackup=1                                    # CHANGE
    - --advertise-address=192.168.100.21
    - --allow-privileged=true
...
 
```

> 注意： 您应该知道如何按照文档中的描述完全自己启用审核日志记录。请随意在此环境中的另一个集群中尝试此操作。


现在我们看看现有的Policy：
```c
➜ root@cluster2-master1:~# vim /etc/kubernetes/audit/policy.yaml
# /etc/kubernetes/audit/policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```
我们可以看到这个简单的策略记录了元数据级别的所有内容。所以我们把它改成需求：

```c
# /etc/kubernetes/audit/policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
​
# log Secret resources audits, level Metadata
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]
​
# log node related audits, level RequestResponse
- level: RequestResponse
  userGroups: ["system:nodes"]
​
# for everything else don't log anything
- level: None
```

保存更改后，我们必须重新启动 apiserver：

```c
➜ root@cluster2-master1:~# cd /etc/kubernetes/manifests/
​
➜ root@cluster2-master1:/etc/kubernetes/manifests# mv kube-apiserver.yaml ..
​
➜ root@cluster2-master1:/etc/kubernetes/manifests# truncate -s 0 /etc/kubernetes/audit/logs/audit.log
​
➜ root@cluster2-master1:/etc/kubernetes/manifests# mv ../kube-apiserver.yaml .
Once the apiserver is running again we can check the new logs and scroll through some entries:

{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "Metadata",
  "auditID": "e598dc9e-fc8b-4213-aee3-0719499ab1bd",
  "stage": "RequestReceived",
  "requestURI": "...",
  "verb": "watch",
  "user": {
    "username": "system:serviceaccount:gatekeeper-system:gatekeeper-admin",
    "uid": "79870838-75a8-479b-ad42-4b7b75bd17a3",
    "groups": [
      "system:serviceaccounts",
      "system:serviceaccounts:gatekeeper-system",
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.102.21"
  ],
  "userAgent": "manager/v0.0.0 (linux/amd64) kubernetes/$Format",
  "objectRef": {
    "resource": "secrets",
    "apiVersion": "v1"
  },
  "requestReceivedTimestamp": "2020-09-27T20:01:36.238911Z",
  "stageTimestamp": "2020-09-27T20:01:36.238911Z",
  "annotations": {
    "authentication.k8s.io/legacy-token": "..."
  }
}
Above we logged a watch action by OPA Gatekeeper for Secrets, level Metadata.

{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "RequestResponse",
  "auditID": "c90e53ed-b0cf-4cc4-889a-f1204dd39267",
  "stage": "ResponseComplete",
  "requestURI": "...",
  "verb": "list",
  "user": {
    "username": "system:node:cluster2-master1",
    "groups": [
      "system:nodes",
      "system:authenticated"
    ]
  },
  "sourceIPs": [
    "192.168.100.21"
  ],
  "userAgent": "kubelet/v1.19.1 (linux/amd64) kubernetes/206bcad",
  "objectRef": {
    "resource": "configmaps",
    "namespace": "kube-system",
    "name": "kube-proxy",
    "apiVersion": "v1"
  },
  "responseStatus": {
    "metadata": {},
    "code": 200
  },
  "responseObject": {
    "kind": "ConfigMapList",
    "apiVersion": "v1",
    "metadata": {
      "selfLink": "/api/v1/namespaces/kube-system/configmaps",
      "resourceVersion": "83409"
    },
    "items": [
      {
        "metadata": {
          "name": "kube-proxy",
          "namespace": "kube-system",
          "selfLink": "/api/v1/namespaces/kube-system/configmaps/kube-proxy",
          "uid": "0f1c3950-430a-4543-83e4-3f9c87a478b8",
          "resourceVersion": "232",
          "creationTimestamp": "2020-09-26T20:59:50Z",
          "labels": {
            "app": "kube-proxy"
          },
          "annotations": {
            "kubeadm.kubernetes.io/component-config.hash": "..."
          },
          "managedFields": [
            {
...
            }
          ]
        },
...
      }
    ]
  },
  "requestReceivedTimestamp": "2020-09-27T20:01:36.223781Z",
  "stageTimestamp": "2020-09-27T20:01:36.225470Z",
  "annotations": {
    "authorization.k8s.io/decision": "allow",
    "authorization.k8s.io/reason": ""
  }
}
```
在上面的一个中，我们通过 system:nodes 为ConfigMaps记录了一个列表操作，级别为 RequestResponse。

因为所有 JSON 条目都写在文件的一行中，我们还可以对我们的Policy运行一些简单的验证：

```c
# shows Secret entries
cat audit.log | grep '"resource":"secrets"' | wc -l
​
# confirms Secret entries are only of level Metadata
cat audit.log | grep '"resource":"secrets"' | grep -v '"level":"Metadata"' | wc -l
​
# shows RequestResponse level entries
cat audit.log | grep -v '"level":"RequestResponse"' | wc -l
​
# shows RequestResponse level entries are only for system:nodes
cat audit.log | grep '"level":"RequestResponse"' | grep -v "system:nodes" | wc -l
```


---
## 问题 18 | 通过审计日志调查
相关阅读：

 - [Kubernetes CKS 2021 Course【23】---Runtime Security - Auditing](https://ghostwritten.blog.csdn.net/article/details/117225475)

任务权重：4%

使用上下文： kubectl config use-context infra-prod


命名空间包含五个Opaque 类型的机密，可以认为是高度机密的。最新的事件预防调查显示，ServiceAccount在一段时间内对集群的访问过于广泛。此SA不应该访问该Namespace 中的任何Secrets。 security p.auster

找出其中的秘密在命名空间这个SA通过查看审计日志下做访问。 security

```bash
/opt/course/18/audit.log
```

将密码更改为仅由该SA访问的那些Secrets 的任何新字符串。

 

回答：
首先我们看一下这是关于的秘密：
```bash
➜ k -n security get secret | grep Opaque
kubeadmin-token       Opaque                                1      37m
mysql-admin           Opaque                                1      37m
postgres001           Opaque                                1      37m
postgres002           Opaque                                1      37m
vault-token           Opaque                                1      37m
```

Next we investigate the Audit Log file:

```bash
➜ cd /opt/course/18
​
➜ :/opt/course/18$ ls -lh
total 7.1M
-rw-r--r-- 1 k8s k8s 7.5M Sep 24 21:31 audit.log
​
➜ :/opt/course/18$ cat audit.log | wc -l
4451
```

审计日志可能很大，通过创建审计策略来限制数量并在 Elasticsearch 等系统中传输数据是很常见的。在本例中，我们有一个简单的 JSON 导出，但它已经包含 4451 行。

我们应该尝试将文件过滤到相关信息：
```bash
➜ :/opt/course/18$ cat audit.log | grep "p.auster" | wc -l
28
```
还不错，ServiceAccount只有 28 个日志。 p.auster
```bash
➜ :/opt/course/18$ cat audit.log | grep "p.auster" | grep Secret | wc -l
2
```

并且只有 2 个与Secrets相关的日志......
```bash
➜ :/opt/course/18$ cat audit.log | grep "p.auster" | grep Secret | grep list | wc -l
0
➜ :/opt/course/18$ cat audit.log | grep "p.auster" | grep Secret | grep get | wc -l
2
```

没有列出操作，这很好，但是有 2 个获取操作，所以我们检查一下：
```c
cat audit.log | grep "p.auster" | grep Secret | grep get | vim -
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "RequestResponse",
  "auditID": "74fd9e03-abea-4df1-b3d0-9cfeff9ad97a",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/security/secrets/vault-token",
  "verb": "get",
  "user": {
    "username": "system:serviceaccount:security:p.auster",
    "uid": "29ecb107-c0e8-4f2d-816a-b16f4391999c",
    "groups": [
      "system:serviceaccounts",
      "system:serviceaccounts:security",
      "system:authenticated"
    ]
  },
...
  "userAgent": "curl/7.64.0",
  "objectRef": {
    "resource": "secrets",
    "namespace": "security",
    "name": "vault-token",
    "apiVersion": "v1"
  },
 ...
}
{
  "kind": "Event",
  "apiVersion": "audit.k8s.io/v1",
  "level": "RequestResponse",
  "auditID": "aed6caf9-5af0-4872-8f09-ad55974bb5e0",
  "stage": "ResponseComplete",
  "requestURI": "/api/v1/namespaces/security/secrets/mysql-admin",
  "verb": "get",
  "user": {
    "username": "system:serviceaccount:security:p.auster",
    "uid": "29ecb107-c0e8-4f2d-816a-b16f4391999c",
    "groups": [
      "system:serviceaccounts",
      "system:serviceaccounts:security",
      "system:authenticated"
    ]
  },
...
  "userAgent": "curl/7.64.0",
  "objectRef": {
    "resource": "secrets",
    "namespace": "security",
    "name": "mysql-admin",
    "apiVersion": "v1"
  },
...
}
```
在那里，我们看到的秘密，并通过被访问。因此，我们更改了这些密码。 vault-tokenmysql-adminp.auster

```bash
➜ echo new-vault-pass | base64
bmV3LXZhdWx0LXBhc3MK
​
➜ k -n security edit secret vault-token
​
➜ echo new-mysql-pass | base64
bmV3LW15c3FsLXBhc3MK
​
➜ k -n security edit secret mysql-admin

```
审计日志ftw。

通过运行cat audit.log | grep "p.auster" | grep Secret | grep password我们可以看到密码存储在 Audit Logs 中，因为它们存储了Secrets的完整内容。在日志中显示密码从来都不是一个好主意。在这种情况下，仅存储可以通过审计策略控制的Secrets 的元数据级别信息可能就足够了。

----------
## 第 19 题 | 不可变的根文件系统
相关阅读：


任务权重：2%

使用上下文： `kubectl config use-context workload-prod`

将部署在命名空间应该运行一成不变的，它是从文件创建。即使在成功闯入之后，攻击者也不应该修改正在运行的容器的文件系统。 `immutable-deployment` `team-purple` `/opt/course/19/immutable-deployment.yaml`

以容器内没有进程可以修改本地文件系统的方式修改部署，只有`/tmp`目录应该是可写的。不要修改 Docker 镜像。

将更新的 YAML 保存在`/opt/course/19/immutable-deployment-new.yaml`并更新正在运行的Deployment 下。
 

回答：
默认情况下，容器中的进程可以写入本地文件系统。当非恶意进程被劫持时，这会增加攻击面。阻止应用程序写入磁盘或仅允许某些目录可以降低风险。例如，如果 Nginx 中存在允许攻击者覆盖容器内任何文件的错误，那么这只有在 Nginx 进程本身可以首先写入文件系统时才有效。

将根文件系统设为只读可以在 Docker 镜像本身或Pod声明中完成。

让我们先检查部署的命名空间： immutable-deployment team-purple

```c
➜ k -n team-purple edit deploy -o yaml
# kubectl -n team-purple edit deploy -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: team-purple
  name: immutable-deployment
  labels:
    app: immutable-deployment
  ...
spec:
  replicas: 1
  selector:
    matchLabels:
      app: immutable-deployment
  template:
    metadata:
      labels:
        app: immutable-deployment
    spec:
      containers:
      - image: busybox:1.32.0
        command: ['sh', '-c', 'tail -f /dev/null']
        imagePullPolicy: IfNotPresent
        name: busybox
      restartPolicy: Always
...
```

容器具有对根文件系统的写访问权限，因为现有 SecurityContext没有为Pod或容器定义任何限制。根据任务，我们不允许更改 Docker 映像。

因此，我们修改 YAML 清单以包含所需的更改：

```bash
cp /opt/course/19/immutable-deployment.yaml /opt/course/19/immutable-deployment-new.yaml
​
vim /opt/course/19/immutable-deployment-new.yaml

# /opt/course/19/immutable-deployment-new.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: team-purple
  name: immutable-deployment
  labels:
    app: immutable-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: immutable-deployment
  template:
    metadata:
      labels:
        app: immutable-deployment
    spec:
      containers:
      - image: busybox:1.32.0
        command: ['sh', '-c', 'tail -f /dev/null']
        imagePullPolicy: IfNotPresent
        name: busybox
        securityContext:                  # add
          readOnlyRootFilesystem: true    # add
        volumeMounts:                     # add
        - mountPath: /tmp                 # add
          name: temp-vol                  # add
      volumes:                            # add
      - name: temp-vol                    # add
        emptyDir: {}                      # add
      restartPolicy: Always
```
SecurityContexts 可以在Pod或容器级别设置，这里要求后者。强制readOnlyRootFilesystem: true将呈现根文件系统只读。然后，我们可以通过使用 emptyDir 卷来允许某些目录可写。

进行更改后，让我们更新部署：
```bash
➜ k delete -f /opt/course/19/immutable-deployment-new.yaml
deployment.apps "immutable-deployment" deleted
​
➜ k create -f /opt/course/19/immutable-deployment-new.yaml
deployment.apps/immutable-deployment created
We can verify if the required changes are propagated:

➜ k -n team-purple exec immutable-deployment-5b7ff8d464-j2nrj -- touch /abc.txt
touch: /abc.txt: Read-only file system
command terminated with exit code 1
​
➜ k -n team-purple exec immutable-deployment-5b7ff8d464-j2nrj -- touch /var/abc.txt
touch: /var/abc.txt: Read-only file system
command terminated with exit code 1
​
➜ k -n team-purple exec immutable-deployment-5b7ff8d464-j2nrj -- touch /etc/abc.txt
touch: /etc/abc.txt: Read-only file system
command terminated with exit code 1
​
➜ k -n team-purple exec immutable-deployment-5b7ff8d464-j2nrj -- touch /tmp/abc.txt
​
➜ k -n team-purple exec immutable-deployment-5b7ff8d464-j2nrj -- ls /tmp
abc.txt
```
该部署已使容器的文件系统是只读的更新

-----

## 第 20 题 | 更新 Kubernetes
相关阅读：

 - [Kubernetes CKS 2021【11】---Cluster Hardening - Upgrade Kubernetes](https://ghostwritten.blog.csdn.net/article/details/116159731)

任务权重：8%

使用上下文： kubectl config use-context workload-stage

集群正在运行 Kubernetes 1.19.6。1.20.1通过包管理器将其更新为可用apt。

使用ssh cluster3-master1和ssh cluster3-worker1连接到实例。

回答：
让我们来看看当前的版本：
```bash
➜ k get node
NAME               STATUS   ROLES    AGE   VERSION
cluster3-master1   Ready    master   13m   v1.19.6
cluster3-worker1   Ready    <none>   11m   v1.19.6
```
控制平面主组件
首先，我们应该更新主节点上运行的控制平面组件，因此我们将其排空：
```bash
➜ k drain cluster3-master1 --ignore-daemonsets
Next we ssh into it and check versions:

➜ ssh cluster3-master1
​
➜ root@cluster3-master1:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.4", GitCommit:"d360454c9bcd1634cf4cc52d1867af5491dc9c5f", GitTreeState:"clean", BuildDate:"2020-11-11T13:15:05Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}
​
➜ root@cluster3-master1:~# kubelet --version
Kubernetes v1.18.6
```
安装想要的kubeadm版本：
```bash
root@cluster3-master1:~# apt-get install kubeadm=1.20.1-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
kubeadm is already the newest version (1.20.1-00).
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
```
我们可以看到它kubeadm已经安装在想要的版本中，否则我们将不得不安装它。

检查 kubeadm 有哪些可用的升级计划：

```bash
➜ root@cluster3-master1:~# kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
...
```
我们使用以下方法应用它：
```c
➜ root@cluster3-master1:~# kubeadm upgrade apply v1.20.1
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.20.1"
[upgrade/versions] Cluster version: v1.19.6
[upgrade/versions] kubeadm version: v1.20.1
...
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.20.1". Enjoy!
​
[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
After this finished we verify we're up to date by showing upgrade plans again:

➜ root@cluster3-master1:~# kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.20.1
[upgrade/versions] kubeadm version: v1.20.1
[upgrade/versions] Latest stable version: v1.20.1
[upgrade/versions] Latest stable version: v1.20.1
[upgrade/versions] Latest version in the v1.20 series: v1.20.1
[upgrade/versions] Latest version in the v1.20 series: v1.20.1
 

Control Plane kubelet and kubectl
➜ root@cluster3-master1:~# apt-get update
​
➜ root@cluster3-master1:~# apt-get install kubelet=1.20.1-00 kubectl=1.20.1-00
...
Preparing to unpack .../kubectl_1.20.1-00_amd64.deb ...
Unpacking kubectl (1.20.1-00) over (1.19.6-00) ...
Preparing to unpack .../kubelet_1.20.1-00_amd64.deb ...
Unpacking kubelet (1.20.1-00) over (1.19.6-00) ...
Setting up kubelet (1.20.1-00) ...
Setting up kubectl (1.20.1-00) ...
➜ root@cluster3-master1:~# systemctl daemon-reload && systemctl restart kubelet
​
➜ root@cluster3-master1:~# kubectl get node
NAME               STATUS                     ROLES                  AGE   VERSION
cluster3-master1   Ready,SchedulingDisabled   control-plane,master   21h   v1.20.1
cluster3-worker1   Ready                      <none>                 21h   v1.19.6
Done, and uncordon:

➜ k uncordon cluster3-master1
node/cluster3-master1 uncordoned
 

Data Plane
➜ k get node
NAME               STATUS   ROLES                  AGE   VERSION
cluster3-master1   Ready    control-plane,master   21h   v1.20.1
cluster3-worker1   Ready    <none>                 21h   v1.19.6
```

我们的数据平面由一个工作节点组成，所以让我们更新它。第一件事是我们应该排空它：
```c
k drain cluster3-worker1 --ignore-daemonsets
Next we ssh into it and upgrade kubeadm to the wanted version, or check if already done:

➜ ssh cluster3-worker1
​
➜ root@cluster3-worker1:~# apt-get update
...
​
➜ root@cluster3-worker1:~# apt-get install kubeadm=1.20.1-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
kubeadm is already the newest version (1.20.1-00).
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
​
➜ root@cluster3-worker1:~# kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
Now we follow that kubeadm told us in the last line and upgrade kubelet (and kubectl):

➜ root@cluster3-worker1:~# apt-get install kubelet=1.20.1-00 kubectl=1.20.1-00
...
Preparing to unpack .../kubectl_1.20.1-00_amd64.deb ...
Unpacking kubectl (1.20.1-00) over (1.19.6-00) ...
Preparing to unpack .../kubelet_1.20.1-00_amd64.deb ...
Unpacking kubelet (1.20.1-00) over (1.19.6-00) ...
Setting up kubelet (1.20.1-00) ...
Setting up kubectl (1.20.1-00) ...
​
➜ root@cluster3-worker1:~# systemctl daemon-reload && systemctl restart kubelet
Looking good, what does the node status say?

➜ k get node
NAME               STATUS                     ROLES                  AGE   VERSION
cluster3-master1   Ready                      control-plane,master   21h   v1.20.1
cluster3-worker1   Ready,SchedulingDisabled   <none>                 21h   v1.20.1
Beautiful, let's make it schedulable again:

➜ k uncordon cluster3-worker1
node/cluster3-worker1 uncordoned
​
➜ k get node
NAME               STATUS   ROLES                  AGE   VERSION
cluster3-master1   Ready    control-plane,master   21h   v1.20.1
cluster3-worker1   Ready    <none>                 21h   v1.20.1
We're up to date.

 
```





------

## 第 21 题 | 图像漏洞扫描

相关知识点：

 - [Kubernetes CKS 2021 Course【19】---Supply Chain Security - Image Vulnerability Scanning](https://ghostwritten.blog.csdn.net/article/details/117113680)

任务权重：2%

（可以在任何 kubectl 上下文中解决）

漏洞扫描器trivy安装在您的主终端上。使用它来扫描以下图像以查找已知的 CVE：

 - nginx:1.16.1-alpine
 - k8s.gcr.io/kube-apiserver:v1.18.0
 - k8s.gcr.io/kube-controller-manager:v1.18.0
 - docker.io/weaveworks/weave-kube:2.7.0

写不包含漏洞的所有图像`CVE-2020-10878`或`CVE-2020-1967`成`/opt/course/21/good-images`。

 

回答：
 

该工具trivy使用起来非常简单，它将图像与公共数据库进行比较。
```c
➜ trivy nginx:1.16.1-alpine
2020-10-09T20:59:39.198Z        INFO    Need to update DB
2020-10-09T20:59:39.198Z        INFO    Downloading DB...
18.81 MiB / 18.81 MiB [-------------------------------------
2020-10-09T20:59:45.499Z        INFO    Detecting Alpine vulnerabilities...
​
nginx:1.16.1-alpine (alpine 3.10.4)
===================================
Total: 7 (UNKNOWN: 0, LOW: 0, MEDIUM: 7, HIGH: 0, CRITICAL: 0)
​
+---------------+------------------+----------+-------------------
|    LIBRARY    | VULNERABILITY ID | SEVERITY | INSTALLED VERSION
+---------------+------------------+----------+-------------------
| libcrypto1.1  | CVE-2020-1967    | MEDIUM   | 1.1.1d-r2         
... 
To solve the task we can run:

➜ trivy nginx:1.16.1-alpine | grep -E 'CVE-2020-10878|CVE-2020-1967'
| libcrypto1.1  | CVE-2020-1967    | MEDIUM   
| libssl1.1     | CVE-2020-1967    |          
​
➜ trivy k8s.gcr.io/kube-apiserver:v1.18.0 | grep -E 'CVE-2020-10878|CVE-2020-1967'
| perl-base     | CVE-2020-10878      | HIGH
​
➜ trivy k8s.gcr.io/kube-controller-manager:v1.18.0 | grep -E 'CVE-2020-10878|CVE-2020-1967'
| perl-base     | CVE-2020-10878      | HIGH   
​
➜ trivy docker.io/weaveworks/weave-kube:2.7.0 | grep -E 'CVE-2020-10878|CVE-2020-1967'
➜
```
唯一没有两个 CVE 中任何一个的图像是`docker.io/weaveworks/weave-kube:2.7.0`，因此我们的答案是：

```c
# /opt/course/21/good-images
docker.io/weaveworks/weave-kube:2.7.0
```


--------------------

## Question 22 | 手动静态安全分析

Task weight: 3%

(can be solved in any kubectl context)

发布工程团队已与您共享了一些 YAML 清单和 Dockerfile 以供查看。这些文件位于`/opt/course/22/files`.

作为容器安全专家，您需要执行手动静态分析并找出与不需要的凭据暴露相关的可能安全问题。以 root 身份运行进程在此任务中无关紧要。

将有问题的文件名写入`/opt/course/22/security-issues.`

 

注意：在 Dockerfile 和 YAML 清单中，假设存在引用的文件、文件夹、机密和卷挂载。忽略语法或逻辑错误。

 
回答：
我们检查位置 /opt/course/22/files并列出文件。

```bash
➜ ls -la /opt/course/22/files
共 48 个
drwxr-xr-x 2 k8s k8s 4096 9 月 16 日 19:08。
drwxr-xr-x 3 k8s k8s 4096 Sep 16 19:08 ..
-rw-r--r-- 1 k8s k8s 692 Sep 16 19:08 Dockerfile-go
-rw-r--r-- 1 k8s k8s 897 Sep 16 19:08 Dockerfile-mysql
-rw-r--r-- 1 k8s k8s 743 Sep 16 19:08 Dockerfile-py
-rw-r--r-- 1 k8s k8s 341 Sep 16 19:08 deployment-nginx.yaml
-rw-r--r-- 1 k8s k8s 705 Sep 16 19:08 deployment-redis.yaml
-rw-r--r-- 1 k8s k8s 392 Sep 16 19:08 pod-nginx.yaml
-rw-r--r-- 1 k8s k8s 228 Sep 16 19:08 pv-manual.yaml
-rw-r--r-- 1 k8s k8s 188 Sep 16 19:08 pvc-manual.yaml
-rw-r--r-- 1 k8s k8s 211 Sep 16 19:08 sc-local.yaml
-rw-r--r-- 1 k8s k8s 902 Sep 16 19:08 statefulset-nginx.yaml
```

我们有 3 个 Dockerfile 和 7 个 Kubernetes 资源 YAML 清单。接下来，我们应该仔细检查每一个，以发现凭据使用方式的安全问题。

 

注意：您应该熟悉Docker 最佳实践和Kubernetes 配置最佳实践。

 

在浏览文件时，我们可能会注意到：

 

1号
乍一看，文件Dockerfile-mysql可能看起来很无辜。它复制一个文件secret-token，使用它并在之后删除它。但是由于 Docker 的工作方式，每个RUN,COPY和ADD命令都会创建一个新层，并且每个层都在映像中持久化。

这意味着即使文件secret-token在 Z 层被删除，它仍然包含在 X 和 Y 层的图像中。在这种情况下，最好使用例如传递给 Docker 的变量。

```c
# /opt/course/22/files/Dockerfile-mysql
FROM ubuntu
​
# Add MySQL configuration
COPY my.cnf /etc/mysql/conf.d/my.cnf
COPY mysqld_charset.cnf /etc/mysql/conf.d/mysqld_charset.cnf
​
RUN apt-get update && \
    apt-get -yq install mysql-server-5.6 &&
​
# Add MySQL scripts
COPY import_sql.sh /import_sql.sh
COPY run.sh /run.sh
​
# Configure credentials
COPY secret-token .                                       # LAYER X
RUN /etc/register.sh ./secret-token                       # LAYER Y
RUN rm ./secret-token # delete secret token again         # LATER Z
​
EXPOSE 3306
CMD ["/run.sh"]
```
So we do:

```bash
echo Dockerfile-mysql >> /opt/course/22/security-issues
```
2号
该文件 `deployment-redis.yaml`从名为Secret 的 Secret 中获取凭证，并将这些凭证写入环境变量。到目前为止一切顺利，但在容器的命令中，它正在回显这些，任何有权访问日志的用户都可以直接读取这些内容。mysecret

```c
# /opt/course/22/files/deployment-redis.yaml
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
      - name: mycontainer
        image: redis
        command: ["/bin/sh"]
        args:
        - "-c"
        - "echo $SECRET_USERNAME && echo $SECRET_PASSWORD && docker-entrypoint.sh" # NOT GOOD
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
```

Credentials in logs is never a good idea, hence we do:

```bash
echo deployment-redis.yaml >> /opt/course/22/security-issues
```

3号
在文件中statefulset-nginx.yaml，密码直接暴露在容器的环境变量定义中。

```c
# /opt/course/22/files/statefulset-nginx.yaml
...
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
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
        image: k8s.gcr.io/nginx-slim:0.8
        env:
        - name: Username
          value: Administrator
        - name: Password
          value: MyDiReCtP@sSw0rd               # NOT GOOD
        ports:
        - containerPort: 80
          name: web
..
```

This should better be injected via a Secret. So we do:

```bash
echo statefulset-nginx.yaml >> /opt/course/22/security-issues
```

```bash
➜ cat /opt/course/22/security-issues
Dockerfile-mysql
deployment-redis.yaml
statefulset-nginx.yaml
```

 


