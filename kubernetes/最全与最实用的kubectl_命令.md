# kubectl 命令
tags: kubectl,命令








---


## kubectl 安装

- [Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

## kubectl 插件
- [Krew—Kubectl 插件的包管理器](https://blog.csdn.net/xixihahalelehehe/article/details/129838856)
## Kubectl 自动补全
BASH

```bash
source <(kubectl completion bash) # 在 bash 中设置当前 shell 的自动补全，要先安装 bash-completion 包。
echo "source <(kubectl completion bash)" >> ~/.bashrc # 在您的 bash shell 中永久的添加自动补全
```

您还可以为 kubectl 使用一个速记别名，该别名也可以与 completion 一起使用：

```bash
alias k='kubectl'
alias kg='k get'
alias kd='k describe'
alias kl='k logs'
alias ke='k explain'
alias kr='k replace'
alias kc='k create'
alias kgp='k get po'
alias kgn='k get no'
alias kge='k get ev'
alias kex='k exec -it'
alias kgc='k config get-contexts'
alias ksn='k config set-context --current --namespace'
alias kuc='k config use-context'
alias krun='k run'
export do='--dry-run=client -oyaml'
export force='--grace-period=0 --force'

source <(kubectl completion bash)
source <(kubectl completion bash | sed 's/kubectl/k/g' )
complete -F __start_kubectl k


alias krp='k run test --image=busybox --restart=Never'
alias kuc='k config use-context'
```
ZSH 

```bash
source <(kubectl completion zsh)  # 在 zsh 中设置当前 shell 的自动补全
echo "if [ $commands[kubectl] ]; then source <(kubectl completion zsh); fi" >> ~/.zshrc # 在您的 zsh shel
```

## kubectl run
创建并运行一个或多个容器镜像。
创建一个deployment 或job 来管理容器。
语法

```bash
$ kubectl run NAME --image=image [--env="key=value"] [--port=port] [--replicas=replicas] [--dry-run=bool] [--overrides=inline-json] [--command] -- [COMMAND] [args...]
```

示例：

```bash
启动nginx实例。
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --restart=Never -n mynamespace
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n mynamespace -f -
kubectl run busybox --image=busybox --command --restart=Never -it -- env

启动带API组的nginx实例。将来被弃用
kubectl run nginx --image=nginx  --generator=run-pod/v1  

带有标签function=mantou的pod
kubectl run nginx2 --image=nginx   --labels function=mantou
多个标签
kubectl run nginx2 --image=nginx   --labels function=mantou，disk=ssd

# 创建nginx-app的deployment,并记录升级。
kubectl run nginx-app --image=nginx:1.11.0-alpine --record

使用默认命令启动 nginx 容器，但对该命令使用自定义参数（arg1 .. argN）
kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

启动hazelcast实例，暴露容器端口 5701。
kubectl run hazelcast --image=hazelcast --port=5701

启动hazelcast实例，在容器中设置环境变量“DNS_DOMAIN = cluster”和“POD_NAMESPACE = default”。
kubectl run hazelcast --image=hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"

启动nginx实例，设置副本数5。
kubectl run nginx --image=nginx --replicas=5

配置cpu与内存的pod
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'

运行 Dry  打印相应的API对象而不创建它们。
kubectl run nginx --image=nginx --dry-run

在特定的命令空间的一个pod运行多个容器
kubectl run test --image=nginx --image=redis --image=memcached --image=consul --restart=Nerver -n kube-public

启动一个单一的 nginx 实例，但是使用从 JSON 分析的一部分值来重载部署规格.
kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'

启动一个 busybox 的 pod 并将其保留在前台，如果它退出，请不要重新启动它.
kubectl run -i -t busybox --image=busybox --restart=Never

启动 cron 作业计算 π 后2000位，每5分钟打印一次.
kubectl run pi --schedule="0/5 * * * ?" --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```
注意：
Flag `--generator` has been deprecated, has no effect and will be removed in the future
```bash
--schedule=<schedule>	CronJob	
--restart=Always	Deployment	
--restart=OnFailure	Job	
--restart=Never	Pod
```

如果不指定生成器，kubectl 将按以下顺序考虑其他参数：

```bash
--schedule
--restart
```

## kubectl create 
### kubectl create namespace
```bash
#创建一个命名空间
kubectl create namespace mynamespace
kubectl create namespace myns -o yaml --dry-run
```
### kubectl create pod 
```c
#通过pod.json文件创建一个pod。
kubectl create -f ./pod.json

通过stdin的JSON创建一个pod。

cat pod.json | kubectl create -f -

API版本为v1的JSON格式的docker-registry.yaml文件创建资源。
kubectl create -f docker-registry.yaml --edit --output-version=v1 -o json
```
### kubectl create deployment
```c
#创建一个deployment
kubectl create deployment nginx --image=nginx --restart=always
```
### kubectl create configmap
```c

#创建一个configmap
echo -e "foo3=lili\nfoo4=lele" > config.txt
kubectl create configmap db-config --from-env-file=config.txt
kubectl get cm db-config -o yaml

kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
kubectl describe cm config

echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
kubectl create cm configmap3 --from-env-file=config.env
kubectl get cm configmap3 -o yaml

echo -e "var3=val3\nvar4=val4" > config4.txt
kubectl create cm configmap4 --from-file=special=config4.txt
kubectl describe cm configmap4

```
### kubectl create quota
```c
#创建一个quota
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml
```


### kubectl create secret
```c
#创建一个secret
kubectl create secret generic mysecret --from-literal=password=mypass

echo -n admin > username
kubectl create secret generic mysecret2 --from-file=username
kubectl get secret mysecret2 -o yaml
echo YWRtaW4K | base64 -d # on MAC it is -D, which decodes the value and shows 'admin'
kubectl get secret mysecret2 -o jsonpath='{.data.username}{"\n"}' | base64 -d

kc secret generic my-secret --from-literal=APP_SECRET=sdcdcsdcsdcsdc
kc secret generic my-secret --from-file=secret.txt
kc secret generic my-secret --from-env-file=secret.env
```
### kubectl create job/cronjob
```c
#创建一个job
kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml

job.spec.activeDeadlineSeconds=30 #如果执行超过30秒则停止job，
job.spec.completions=5 #运行5次
job.spec.parallelism=5 #并行运行5次

#创建一个cronjob，定时任务
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'

kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
 
```
### kubectl create serviceaccount
```c

#创建一个serviceaccount
kubectl create sa myuser
kubectl get sa -A
kubectl get sa default -o yaml > sa.yaml
kubectl run nginx --image=nginx --restart=Never --serviceaccount=myuser -o yaml --dry-run > pod.yaml
kubectl create -f pod.yaml
kubectl describe pod nginx # will see that a new secret called myuser-token-***** has been mounted

```

### kubectl create rolebinding/role/clusterrolebinding/clusterrole

```c
root@master:~/k8slib# k -n red create rolebinding secret-manager --role=secret-manager --user=jane
rolebinding.rbac.authorization.k8s.io/secret-manager created
root@master:~/k8slib# k -n blue create role secret-manager --verb=get --verb=list --resource=secrets 
role.rbac.authorization.k8s.io/secret-manager created
root@master:~/k8slib# k -n blue create rolebinding secret-manager --role=secret-manager --user=jane
rolebinding.rbac.authorization.k8s.io/secret-manager created
#测试
root@master:~/k8slib# k -n red auth can-i get secrets --as jane
yes
root@master:~/k8slib# k -n red auth can-i get secrets --as tom
no
root@master:~/k8slib# k -n red auth can-i delete secrets --as jane
no
root@master:~/k8slib# k -n red auth can-i list secrets --as jane
no
root@master:~/k8slib# k -n blue auth can-i list secrets --as jane
yes
root@master:~/k8slib# k -n blue auth can-i get secrets --as jane
yes
root@master:~/k8slib# k -n blue auth can-i get pods --as jane
no

root@master:~/k8slib# k create clusterrole deploy-deleter --verb delete --resource deployments
clusterrole.rbac.authorization.k8s.io/deploy-deleter created
root@master:~/k8slib# k create clusterrolebinding deploy-deleter --user jane --clusterrole deploy-deleter
clusterrolebinding.rbac.authorization.k8s.io/deploy-deleter created
root@master:~/k8slib# k -n red create rolebinding deploy-deleter  --user jim --clusterrole deploy-deleter
rolebinding.rbac.authorization.k8s.io/deploy-deleter created
root@master:~/k8slib# k auth can-i delete deployments --as jane
yes
root@master:~/k8slib# k auth can-i delete deployments --as jane -n default
yes
root@master:~/k8slib# k auth can-i delete deployments --as jane -n red
yes
root@master:~/k8slib# k auth can-i delete pods --as jane -n red
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -n default
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -A
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -n red
yes
```

## kubectl exec

```bash
kubectl exec -it $(kubectl get pods -n kube-system| grep kube-apiserver|awk '{print $1}') -n kube-system -- /usr/local/bin/kube-apiserver -h |grep  enable-admission-plugins
```

## kubectl apply

```bash
# 将pod.json中的配置应用到pod
kubectl apply -f ./pod.json

# 将控制台输入的JSON配置应用到Pod
cat pod.json | kubectl apply -f -
```
选项

```bash
  -f, --filename=[]: 包含配置信息的文件名，目录名或者URL。
      --include-extended-apis[=true]: If true, include definitions of new APIs via calls to the API server. [default true]
  -o, --output="": 输出模式。"-o name"为快捷输出(资源/name).
      --record[=false]: 在资源注释中记录当前 kubectl 命令。
  -R, --recursive[=false]: Process the directory used in -f, --filename recursively. Useful when you want to manage related manifests organized within the same directory.
      --schema-cache-dir="~/.kube/schema": 非空则将API schema缓存为指定文件，默认缓存到'$HOME/.kube/schema'
      --validate[=true]: 如果为true，在发送到服务端前先使用schema来验证输入。
```
继承自父命令的选项

```bash
      --alsologtostderr[=false]: 同时输出日志到标准错误控制台和文件。
      --certificate-authority="": 用以进行认证授权的.cert文件路径。
      --client-certificate="": TLS使用的客户端证书路径。
      --client-key="": TLS使用的客户端密钥路径。
      --cluster="": 指定使用的kubeconfig配置文件中的集群名。
      --context="": 指定使用的kubeconfig配置文件中的环境名。
      --insecure-skip-tls-verify[=false]: 如果为true，将不会检查服务器凭证的有效性，这会导致你的HTTPS链接变得不安全。
      --kubeconfig="": 命令行请求使用的配置文件路径。
      --log-backtrace-at=:0: 当日志长度超过定义的行数时，忽略堆栈信息。
      --log-dir="": 如果不为空，将日志文件写入此目录。
      --log-flush-frequency=5s: 刷新日志的最大时间间隔。
      --logtostderr[=true]: 输出日志到标准错误控制台，不输出到文件。
      --match-server-version[=false]: 要求服务端和客户端版本匹配。
      --namespace="": 如果不为空，命令将使用此namespace。
      --password="": API Server进行简单认证使用的密码。
  -s, --server="": Kubernetes API Server的地址和端口号。
      --stderrthreshold=2: 高于此级别的日志将被输出到错误控制台。
      --token="": 认证到API Server使用的令牌。
      --user="": 指定使用的kubeconfig配置文件中的用户名。
      --username="": API Server进行简单认证使用的用户名。
      --v=0: 指定输出日志的级别。
      --vmodule=: 指定输出日志的模块，格式如下：pattern=N，使用逗号分隔。
```



## kubectl replace
使用配置文件或stdin来替换资源。
语法：

```bash
$ kubectl get TYPE NAME -o yaml
```
示例：
```bash
使用pod.json中的数据替换pod。
kubectl replace -f ./pod.json

基于 stdin 输入的 JSON 替换 pod
$ cat pod.json | kubectl replace -f -                              

更新镜像版本(tag)到v4
kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -

强制替换，删除原有资源，然后重新创建资源
kubectl replace --force -f ./pod.json
```

## kubectl expose
将资源暴露为新的Kubernetes Service。

指定deployment、service、replica set、replication controller或pod ，并使用该资源的选择器作为指定端口上新服务的选择器。deployment 或 replica set只有当其选择器可转换为service支持的选择器时，即当选择器仅包含matchLabels组件时才会作为暴露新的Service。

资源包括(不区分大小写)：

pod（po），service（svc），replication controller（rc），deployment（deploy），replica set（rs）


语法
$ kubectl  expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type]
示例

```bash
为RC的nginx创建service，并通过Service的80端口转发至容器的8000端口上。
kubectl expose rc nginx --port=80 --target-port=8000

创建 type NodePort 类型 svc
kubectl expose pod audit-pod --type NodePort --port 5678

为使用副本集RS的复制的 nginx 创建一个服务,该服务使用80端口，并连接到容器的8000端口上
kubectl expose rs nginx --port=80 --target-port=8000

由“nginx-controller.yaml”中指定的type和name标识的RC创建Service，并通过Service的80端口转发至容器的8000端口上。
kubectl expose -f nginx-controller.yaml --port=80 --target-port=8000

为一个 pod 的有效端口创建一个服务，该服务在444的端口使用名为“frontend”
kubectl expose pod valid-pod --port=444 --name=frontend

基于上述服务创建第二个服务，将容器端口8443对外暴露为端口443，名称为 “nginx-https”
kubectl expose service nginx --port=443 --target-port=8443 --name=nginx-https

为端口4100上的复制流应用创建一个服务，平衡UDP流量并命名为“video-stream”.
kubectl expose rc streamer --port=4100 --protocol=udp --name=video-stream

为一个 nginx deployment 创建服务，该服务在端口80上运行，并连接到容器的8000端口.
kubectl expose deployment nginx --port=80 --target-port=8000

```
##  kubectl explain
打印指定的定义资源
能的资源类型包括:`pods (po)、services (svc)、replicationcontrollers (rc)、nodes (no)、events (ev)、componentstatus (cs)、limitranges (limits)、persistentvolume (pv)、persistentvolume eclures (pvc)、resourcequotas (quota)、namespaces (ns)、horizontalpodautoscalers (hpa)、endpoints (ep)`。

`--recursive`展示完整的`spec`递归字段
```bash
kubectl explain deployment.spec --recursive
```

```bash
$ kubectl explain deployment.spec 
KIND:     Deployment
VERSION:  apps/v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the Deployment.

     DeploymentSpec is the specification of the desired behavior of the
     Deployment.

FIELDS:
   minReadySeconds	<integer>
     Minimum number of seconds for which a newly created pod should be ready
     without any of its container crashing, for it to be considered available.
     Defaults to 0 (pod will be considered available as soon as it is ready)

   paused	<boolean>
     Indicates that the deployment is paused.

   progressDeadlineSeconds	<integer>
     The maximum time in seconds for a deployment to make progress before it is
     considered to be failed. The deployment controller will continue to process
     failed deployments and a condition with a ProgressDeadlineExceeded reason
     will be surfaced in the deployment status. Note that progress will not be
     estimated during the time a deployment is paused. Defaults to 600s.

   replicas	<integer>
     Number of desired pods. This is a pointer to distinguish between explicit
     zero and not specified. Defaults to 1.

   revisionHistoryLimit	<integer>
     The number of old ReplicaSets to retain to allow rollback. This is a
     pointer to distinguish between explicit zero and not specified. Defaults to
     10.

   selector	<Object> -required-
     Label selector for pods. Existing ReplicaSets whose pods are selected by
     this will be the ones affected by this deployment. It must match the pod
     template's labels.

   strategy	<Object>
     The deployment strategy to use to replace existing pods with new ones.

   template	<Object> -required-
     Template describes the pods that will be created.
```

```bash
kubectl explain deployments.spec
# or
kubectl explain deployment.spec
# or
kubectl explain deploy.spec
```

## kubectl annotate
更新注释
```bash
使用注释 'description' 和值 'my frontend' 更新 pod'foo'. ＃如果相同的注释多次设置，则只会应用最后一个值
kubectl annotate pods foo description='my frontend'

在 “pod.json” 中更新由类型和名称标识的 pod
kubectl annotate -f pod.json description='my frontend'

使用注释 'description' 和值 'my frontend running nginx' 更新 pod'foo'，并覆盖任何现有值.
kubectl annotate --overwrite pods foo description='my frontend running nginx'

更新命名空间中的所有 pod
kubectl annotate pods --all description='my frontend running nginx'

仅当资源与版本1没有变化时才更新 pod'foo'.
kubectl annotate pods foo description='my frontend running nginx' --resource-version=1

通过删除名为 “description” 的注释（如果存在）更新 pod'foo'. ＃不需要 --overwrite 标志.
kubectl annotate pods foo description-

```
## kubectl get
```bash
kubectl get namespaces
kubectl get ns
kubectl get event
```
### kubectl get pods

```bash
kubectl get pods pod1 pod2 #获取多个pod
kubectl get pods -n logging

kubectl get pods -n logging -o wide
kubectl get pods -A
kubectl get po -A
kubectl get po -A -o wide
---
kubectl get po nginx -o yaml
# or
kubectl get po nginx -oyaml
# or
kubectl get po nginx --output yaml
# or
kubectl get po nginx --output=yaml
---
kubectl get pods pods zongxun-test-1 -o yaml -n <namespace>  #查看完整创建配置信息
kubectl get pods pods zongxun-test-1 -o yaml -n <namespace> | more #查看部分创建配置信息
kubectl get pods pods zongxun-test-1 -o json -n <namespace>  #查看创建配置信息
kubectl get pods -A --show-labels #显示pod并带label
kubectl get pods -A -L controller-revision-hash #获取标为controller-revision-hash的pod
kubectl get pods -A --watch  #动态查看pod状态，如何杀掉pod，状态由running变为terminting

#Get only the 'app=v2' pods
kubectl get po -l app=v2
# or
kubectl get po -l 'app in (v2)'
# or
kubectl get po --selector=app=v2
```

###  --selector

```bash
# 获取包含 app=cassandra 标签的所有 Pods 的 version 标签
kubectl get pods --selector=app=cassandra -o \
  jsonpath='{.items[*].metadata.labels.version}'

```

#### --field-selector
参考资料：[https://blog.csdn.net/fly910905/article/details/102572878](https://blog.csdn.net/fly910905/article/details/102572878)
 - metadata.name=my-service
 - metadata.namespace!=default
 - status.phase=Pending

```bash
kubectl get pods --field-selector status.phase=Running -n kube-system #查看正在运行的pod
kubectl get pods --field-selector status.phase=Runnin
kubectl get ingress --field-selector foo.bar=baz
kubectl get services --field-selector metadata.namespace!=default
链式选择器
kubectl get pods --field-selector=status.phase!=Running,spec.restartPolicy=Always

kubectl get pod --field-selector=status.phase==Running
NAME                                 READY   STATUS    RESTARTS   AGE
hello-world-flask-79498b6444-6lk59   1/1     Running   0          89s
hello-world-flask-79498b6444-vtjn5   1/1     Running   0          85s

```
#### --sort-by排序

```bash
# 列出当前名字空间下所有 Services，按名称排序
kubectl get services --sort-by=.metadata.name

# 列出 Pods，按重启次数排序
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# 列举所有 PV 持久卷，按容量排序
kubectl get pv --sort-by=.spec.capacity.storage
# 列出事件（Events），按时间戳排序
kubectl get events --sort-by=.metadata.creationTimestamp

# 根据重启次数排序列出 pod
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

kubectl get pv --sort-by=.spec.capacity.storage
```
###  -o=custom-columns

```bash
# 集群中运行着的所有镜像
root@master:~# k get pods -A -o=custom-columns='DATA:spec.containers[*].image'
DATA
nginx
nginx
calico/node:v3.16.6
calico/node:v3.16.6
calico/node:v3.16.6
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/kube-apiserver:v1.20.7
k8s.gcr.io/kube-controller-manager:v1.20.7
k8s.gcr.io/kube-proxy:v1.20.7
k8s.gcr.io/kube-proxy:v1.20.7
k8s.gcr.io/kube-proxy:v1.20.7
k8s.gcr.io/kube-scheduler:v1.20.7
kubernetesui/metrics-scraper:v1.0.6

# 列举 default 名字空间中运行的所有镜像，按 Pod 分组
kubectl  get pods --namespace default --output=custom-columns="NAME:.metadata.name,IMAGE:.spec.containers[*].image"
NAME       IMAGE
frontend   nginx

# 除 "k8s.gcr.io/coredns:1.6.2" 之外的所有镜像
root@master:~# kubectl get pods -A -o=custom-columns='DATA:spec.containers[?(@.image!="calico/node:v3.16.6")].image'
DATA
nginx
nginx
<none>
<none>
<none>
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/kube-apiserver:v1.20.7
k8s.gcr.io/kube-controller-manager:v1.20.7
k8s.gcr.io/kube-proxy:v1.20.7
k8s.gcr.io/kube-proxy:v1.20.7
k8s.gcr.io/kube-proxy:v1.20.7
k8s.gcr.io/kube-scheduler:v1.20.7
kubernetesui/metrics-scraper:v1.0.6

# 输出 metadata 下面的所有字段，无论 Pod 名字为何
kubectl get pods -A -o=custom-columns='DATA:metadata.*'
kubectl get pods -o custom-columns='NAME:metadata.name'
kubectl get pods \
  -o custom-columns='NAME:metadata.name,NODE:spec.nodeName'
# Select all elements of a list
kubectl get pods -o custom-columns='DATA:spec.containers[*].image'

# Select a specific element of a list
kubectl get pods -o custom-columns='DATA:spec.containers[0].image'

# Select those elements of a list that match a filter expression
kubectl get pods -o custom-columns='DATA:spec.containers[?(@.image!="nginx")].image'

# Select all fields under a specific location, regardless of their name
kubectl get pods -o custom-columns='DATA:metadata.*'

# Select all fields with a specific name, regardless of their location
kubectl get pods -o custom-columns='DATA:..image'
```


###  jsonpath

```bash
# 获取包含 app=cassandra 标签的所有 Pods 的 version 标签
kubectl get pods --selector=app=cassandra -o jsonpath='{.items[*].metadata.labels.version}'

# 获取指定 pod 内的容器名字
kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec.containers[*].name}" | tr -s '[[:space:]]' '\n' |sort |uniq

# 获取全部节点的 ExternalIP 地址
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'


# 假设你的 Pods 有默认的容器和默认的名字空间，并且支持 'env' 命令，可以使用以下脚本为所有 Pods 生成 ENV 变量。
# 该脚本也可用于在所有的 Pods 里运行任何受支持的命令，而不仅仅是 'env'。 
for pod in $(kubectl get po --output=jsonpath={.items..metadata.name}); do echo $pod && kubectl exec -it $pod env; done

#获取k8s集群节点和k8s版本
$ kubectl get nodes -o=jsonpath=$'{range .items[*]}{@.metadata.name}: {@.status.nodeInfo.kubeletVersion}\n{end}'
master: v1.20.1
node1: v1.20.1
node2: v1.20.1

#获取k8s集群节点和docker版本
$ kubectl get nodes -o=jsonpath=$'{range .items[*]}{@.metadata.name}: {@.status.nodeInfo.containerRuntimeVersion}\n{end}'
master: docker://19.3.4
node1: docker://19.3.4
node2: docker://19.3.4

#获取node节点apparmor是否开启
kubectl get nodes -o=jsonpath=$'{range .items[*]}{@.metadata.name}: {.status.conditions[?(@.reason=="KubeletReady")].message}\n{end}'
master: kubelet is posting ready status. AppArmor enabled
node1: kubelet is posting ready status. AppArmor enabled
node2: kubelet is posting ready status. AppArmor enabled

```
###  go-template

```bash
kubectl get myslclusters -A -o go-template --template='{{ range.items }}{{ printf "%s|%s|%s\n" .metadata.namespace .metadata.name status.clusterStatus }} {{ end }}' 
```




## kubectl logs 

```bash
kubectl logs my-pod                                 # 获取 pod 日志（标准输出）
kubectl logs my-pod |grep -i error                  # 获取 pod 日志关键词error（标准输出）
kubectl logs -l name=myLabel                        # 获取含 name=myLabel 标签的 Pods 的日志（标准输出）
kubectl logs my-pod --previous                      # 获取上个容器实例的 pod 日志（标准输出）
kubectl logs my-pod -c my-container                 # 获取 Pod 容器的日志（标准输出, 多容器场景）
kubectl logs -l name=myLabel -c my-container        # 获取含 name=myLabel 标签的 Pod 容器日志（标准输出, 多容器场景）
kubectl logs my-pod -c my-container --previous      # 获取 Pod 中某容器的上个实例的日志（标准输出, 多容器场景）
kubectl logs -f my-pod                              # 流式输出 Pod 的日志（标准输出）
kubectl logs -f my-pod -c my-container              # 流式输出 Pod 容器的日志（标准输出, 多容器场景）
kubectl logs -f -l name=myLabel --all-containers    # 流式输出含 name=myLabel 标签的 Pod 的所有日志（标准输出）

# 返回pod ruby中已经停止的容器web-1的日志快照
$ kubectl logs -p -c ruby web-1

# 持续输出pod ruby中的容器web-1的日志
$ kubectl logs -f -c ruby web-1

# 仅输出pod nginx中最近的20条日志
$ kubectl logs --tail=20 nginx

# 输出pod nginx中最近一小时内产生的所有日志
$ kubectl logs --since=1h nginx
```







```bash
kubectl describe pods/zongxun-test-1 -n <namespace> #查看监控信息
kubectl describe pods zongxun-test-1 -n <namespace> 
kubeclt exec -n <namespace> -ti <pod> bash
kubectl exec -n <namespace> -ti <pod> -c <container>  bash
kubectl logs -f -n kube-system <pod>
kubectl get cm -n kube-system
kubectl edit cm -n kube-system coredns
kubectl edit deployment -n kube-system coredns
kubectl scale deployment -n kube-system coredns --replicas=0
kubectl delete pods -n kube-system <pod> <pod>
```


允许master节点部署pod，使用命令如下: 

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## kubectl cordon
使node1不能调度pod
```bash
kubectl cordon <node-name>
```
Which will cause the node to be in the status: Ready,SchedulingDisabled.
使node1恢复调度pod
```bash
kubectl uncordon node1
```
## kubectl drain
节点foo不可调度

```bash
kubectl cordon foo
```

使node1节点不可调度，并重新分配该节点上的pod

```bash
kubectl drain node node1  --ignore-daemonsets --delete-local-data
```
## kubectl rollout 

```bash
查看deployment的历史记录
kubectl rollout history deployment/abc

查看daemonset修订版3的详细信息
kubectl rollout history daemonset/abc --revision=3

查看deployment的状态
kubectl rollout status deployment/nginx

将deployment标记为暂停。＃只要deployment在暂停中，使用deployment更新将不会生效。
kubectl rollout pause deployment/nginx

恢复已暂停的 deployment
kubectl rollout resume deployment/nginx

回滚到之前的deployment版本
kubectl rollout undo deployment/abc
kubectl rollout undo --dry-run=true deployment/abc

回滚到daemonset 修订3版本
kubectl rollout undo daemonset/abc --to-revision=3

重启deployment
kubectl rollout restart deployment xxx
```
## kubectl delete

```bash
使用 pod.json中指定的资源类型和名称删除pod。
kubectl delete -f ./pod.json

根据传入stdin的JSON所指定的类型和名称删除pod。
cat pod.json | kubectl delete -f -

删除名为“baz”和“foo”的Pod和Service。
kubectl delete pod,service baz foo

删除 Label name = myLabel的pod和Service。
kubectl delete pods,services -l name=myLabel

强制删除dead node上的pod
kubectl delete pod foo --grace-period=0 --force

删除所有pod
kubectl delete pods --all

优雅下线时间也可以设置成零，零的意思是立即从 Kubernetes 中删除此 Pod
$ kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted


等等删除资源
kubectl delete service audit-pod --wait
kubectl delete pod audit-pod --wait --now
```

##  kubectl describe

```bash
# 描述一个node
$ kubectl describe nodes kubernetes-minion-emt8.c.myproject.internal

# 描述一个pod
$ kubectl describe pods/nginx

# 描述pod.json中的资源类型和名称指定的pod
$ kubectl describe -f pod.json

# 描述所有的pod
$ kubectl describe pods

# 描述所有包含label name=myLabel的pod
$ kubectl describe po -l name=myLabel

# 描述所有被replication controller “frontend”管理的pod（rc创建的pod都以rc的名字作为前缀）
$ kubectl describe pods frontend
```
选项

```bash
  -f, --filename=[]: 用来指定待描述资源的文件名，目录名或者URL。
  -l, --selector="": 用于过滤资源的Label。
```

继承自父命令的选项

```bash
      --alsologtostderr[=false]: 同时输出日志到标准错误控制台和文件。
      --api-version="": 和服务端交互使用的API版本。
      --certificate-authority="": 用以进行认证授权的.cert文件路径。
      --client-certificate="": TLS使用的客户端证书路径。
      --client-key="": TLS使用的客户端密钥路径。
      --cluster="": 指定使用的kubeconfig配置文件中的集群名。
      --context="": 指定使用的kubeconfig配置文件中的环境名。
      --insecure-skip-tls-verify[=false]: 如果为true，将不会检查服务器凭证的有效性，这会导致你的HTTPS链接变得不安全。
      --kubeconfig="": 命令行请求使用的配置文件路径。
      --log-backtrace-at=:0: 当日志长度超过定义的行数时，忽略堆栈信息。
      --log-dir="": 如果不为空，将日志文件写入此目录。
      --log-flush-frequency=5s: 刷新日志的最大时间间隔。
      --logtostderr[=true]: 输出日志到标准错误控制台，不输出到文件。
      --match-server-version[=false]: 要求服务端和客户端版本匹配。
      --namespace="": 如果不为空，命令将使用此namespace。
      --password="": API Server进行简单认证使用的密码。
  -s, --server="": Kubernetes API Server的地址和端口号。
      --stderrthreshold=2: 高于此级别的日志将被输出到错误控制台。
      --token="": 认证到API Server使用的令牌。
      --user="": 指定使用的kubeconfig配置文件中的用户名。
      --username="": API Server进行简单认证使用的用户名。
      --v=0: 指定输出日志的级别。
      --vmodule=: 指定输出日志的模块，格式如下：pattern=N，使用逗号分隔。
```

##  kubectl label

```bash
给名为foo的Pod添加label unhealthy=true。
kubectl label pods foo unhealthy=true

给名为foo的Pod修改label 为 'status' / value 'unhealthy'，且覆盖现有的value。
kubectl label --overwrite pods foo status=unhealthy

给 namespace 中的所有 pod 添加 label
kubectl label pods --all status=unhealthy

仅当resource-version=1时才更新 名为foo的Pod上的label。
kubectl label pods foo status=unhealthy --resource-version=1

删除名为“app”的label 。（使用“ - ”减号相连）
kubectl label po nginx1 nginx2 nginx3 app-
# or
kubectl label po nginx{1..3} app-
# or
kubectl label po -l app app-

给node设置标签
docker label node node1 name=ek8s-node-1 #设置
docker get node --show-labels #查看

删除一个Label，只需在命令行最后指定Label的key名并与一个减号相连即可：
$ kubectl label nodes 1.1.1.1 role-

修改一个Label的值，需要加上--overwrite参数：
$ kubectl label nodes 1.1.1.1 role=apache --overwrite
```
## kubectl set

```bash
升级镜像
kubectl set image pod/nginx nginx=nginx:1.7.1
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```
## kubectl scale

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx

#Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80



kubectl autoscale deployment hello-world-flask --cpu-percent=50 --min=2 --max=10
```
##  kubectl api-resources

```bash
kubectl api-resources
```
## kubectl plugin list

```bash
kubectl plugin list
```
## kubectl diff

```bash
kubectl diff -f pod.json
cat service.yaml | kubectl diff -f -
kubectl diff -Rf /deployment/ | grep -v "kubectl\|recursive\|generation" | tee diff.log
```

## kubectl auth 

```c
root@master:~/k8slib# k auth can-i delete deployments --as jane
yes
root@master:~/k8slib# k auth can-i delete deployments --as jane -n default
yes
root@master:~/k8slib# k auth can-i delete deployments --as jane -n red
yes
root@master:~/k8slib# k auth can-i delete pods --as jane -n red
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -n default
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -A
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -n red
yes
```
## kubectl certificate

```c
root@master:~/cks/RBAC# k get csr
NAME   AGE   SIGNERNAME                            REQUESTOR          CONDITION
jane   7s    kubernetes.io/kube-apiserver-client   kubernetes-admin   Pending
手动批准（或拒绝）证书签名请求
root@master:~/cks/RBAC# k certificate approve jane
certificatesigningrequest.certificates.k8s.io/jane approved
root@master:~/cks/RBAC# k get csr
NAME   AGE   SIGNERNAME                            REQUESTOR          CONDITION
jane   73s   kubernetes.io/kube-apiserver-client   kubernetes-admin   Approved,Issued

拒绝证书签名请求
root@master:~/cks/RBAC# k certificate deny jane
```

## kubectl config

```bash
k8s@terminal:~$ k config get-contexts
CURRENT   NAME                    CLUSTER                 AUTHINFO                NAMESPACE
          gianna@infra-prod       gianna@infra-prod       gianna@infra-prod       
          infra-prod              infra-prod              infra-prod              
          restricted@infra-prod   restricted@infra-prod   restricted@infra-prod   
*         workload-prod           workload-prod           workload-prod           
          workload-stage          workload-stage          workload-stage          

k8s@terminal:~$ k config get-contexts -o name
gianna@infra-prod
infra-prod
restricted@infra-prod
workload-prod
workload-stage


k8s@terminal:~$ k config view -o jsonpath='{.contexts[*].name}'
gianna@infra-prod infra-prod restricted@infra-prod workload-prod workload-stage

k8s@terminal:~$ k config view -o jsonpath="{.contexts[*].name}" | tr " " "\n"
gianna@infra-prod
infra-prod
restricted@infra-prod
workload-prod

k8s@terminal:~$ k config view --raw
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1ERXlNakU0TWpBeE5Gb1hEVE14TURFeU1ERTRNakF4TkZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTG5GCmsyL0oyeWRQWVJhMlAwZ0xHRmloTnliNHUvM0N3VjVLOWgzKzd5MkNLbTNPenpIbG9GSFgzL0FjQW1VamxRWCsKSkQ5TU51VU1NR3pQaXFvUWsyVFBwZkVIa3ljaVk0Q0xQK0hpd3hKSExHdlhVazNlZGxGMEtGL3pTUkFWT3IvTAp5dnllOGtOazd4K3M5UUpoK1RsRElyUjY1RXFaNDJNbzJJbE5jYTAyMHZlR1ZEdUpHM1JyMHRoTEhsTkh6MU5sCmZNa1pBRnNtK3ZGYWxGdzJHRE5HdXFqZjgvMEVjQVJYQkhnM3kzMndxN2lwRis4OVlDNnJwcG1MSXVITXdkVVQKNzJpbENabm1XSStVMlh2aDVRT1drblhGdWpGa2xNRGk0cU9XREJXbFJ3NGZLRGNQUnVoRWlMUmZJOUlTOUlFcQovZ3N4Wmk1TTB4aURacVNXUmIwQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZQMS8xMTRQZU5LU3J3cFA3OUwzbDlYdWdCNlVNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFCaGViODl5aWZWU1hndWtHazJHUkFaZXBDbzRMTDhhaXdMT2ZzT1VwSVplMEFZWFJjQwpycVNyM2dWemJaekRZMmZKMjZTNEtNdzlOZUdWNlZsd2tZNTlvOTZWOGZOMUhlMUZNb2g5TStuUWV3RGFBT1V5CllaMTkzcjhubUxGZGVnR2RJSCtCQXkzYnJkcDJXZVZ0eW56OHRRZW0rSlVBdUFQR0k1OEpRYVNoYTdadXJiT0EKUVlXTHNsc1V6bVJBbkpTQXB1OUNZdU5jTXlUNkxmNTFaaWcyV0NxSHozZEh0SHVXR2NTK0J0NllmVFAvakZZQgpmNHVFTG1DQld0Mjg4bFJiczhhVjdjZXRKckQxOGsrN01nUTBrdWtaYjBMeG5iRndrelhDN0kyUkg0SjN0WTNuClNjSGVHMXQxVjlyLytwMHdDMmdVRUtFbGlQd0wyVGFzbmhHcQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.100.21:6443
  name: gianna@infra-prod
```

```bash
创建 kubelet bootstrapping kubeconfig 文件 设置集群参数
[root@k8smaster ssl]# kubectl config set-cluster kubernetes
–certificate-authority=/opt/kubernetes/ssl/ca.pem
–embed-certs=true
–server=https://192.168.137.171:6443
–kubeconfig=bootstrap.kubeconfig
Cluster “kubernetes” set.

设置客户端认证参数
[root@k8smaster ssl]# kubectl config set-credentials kubelet-bootstrap
–token=ad6d5bb607a186796d8861557df0d17f
–kubeconfig=bootstrap.kubeconfig
User “kubelet-bootstrap” set.

设置上下文参数
[root@k8smaster ssl]# kubectl config set-context default
–cluster=kubernetes
–user=kubelet-bootstrap
–kubeconfig=bootstrap.kubeconfig
Context “default” created.

选择默认上下文
[root@k8smaster ~]# kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
Switched to context “default”.
```
`kubectl config` 设置别名
```bash
# Get current context
alias krc='kubectl config current-context'
# List all contexts
alias klc='kubectl config get-contexts -o name | sed "s/^/  /;\|^  $(krc)$|s/ /*/"'
# Change current context
alias kcc='kubectl config use-context "$(klc | fzf -e | sed "s/^..//")"'

# Get current namespace
alias krn='kubectl config get-contexts --no-headers "$(krc)" | awk "{print \$5}" | sed "s/^$/default/"'
# List all namespaces
alias kln='kubectl get -o name ns | sed "s|^.*/|  |;\|^  $(krn)$|s/ /*/"'
# Change current namespace
alias kcn='kubectl config set-context --current --namespace "$(kln | fzf -e | sed "s/^..//")"'
```

###  kubectl update

```bash
kubectl patch deployment hello-world-flask --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/resources", "value": {"requests": {"memory": "100Mi", "cpu": "100m"}}}]'deployment.apps/hello-world-flask patched
```

### kubectl wait

```bash
$ kubectl wait deployment -n kube-system metrics-server --for condition=Available=True --timeout=90s
deployment.apps/metrics-server condition met
```

## kubectl token

打印 kubeadm 加入集群的命令
```bash
kubectl token create --print-join-command
```

##  综合

###  Helpful commands for debugging

```bash
# Run busybox container
k run busybox --image=busybox:1.28 --rm --restart=Never -it sh
# Connect to a specific container in a Pod
k exec -it busybox -c busybox2 -- /bin/sh
# adding limits and requests in command
kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
# Create a Pod with a service
kubectl run nginx --image=nginx --restart=Never --port=80 --expose
# Check port
nc -z -v -w 2 <service-name> <port-name>
# NSLookup
nslookup <service-name>
nslookup 10-32-0-10.default.pod
```

###  Rolling updates and rollouts

```bash
k set image deploy/nginx nginx=nginx:1.17.0 --record
k rollout status deploy/nginx
k rollout history deploy/nginx
# Rollback to previous version
k rollout undo deploy/nginx
# Rollback to revision number
k rollout undo deploy/nginx --to-revision=2
k rollout pause deploy/nginx
k rollout resume deploy/nginx
k rollout restart deploy/nginx
kubectl run nginx-deploy --image=nginx:1.16 --replias=1 --record
```

### 切换命名空间

```bash
kubectl config set-context $(kubectl config current-context) --namespace=default
```


✈<font color=	#FF4500 size=4 style="font-family:Courier New">更多阅读：</font>



 - [https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/](https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/)
 - [kubectl 常用](https://kubernetes.io/zh/docs/reference/kubectl/cheatsheet/)
 - [https://learnk8s.io/blog/kubectl-productivity](https://learnk8s.io/blog/kubectl-productivity)
 - [Be fast with Kubectl 1.19 CKAD/CKA](https://faun.pub/be-fast-with-kubectl-1-18-ckad-cka-31be00acc443)
 - [Kubectl Cheatsheet](https://collabnix.com/kubectl-cheatsheet/)


✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>



 - [docker 命令](https://blog.csdn.net/xixihahalelehehe/article/details/123378401?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681086016780271517687%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681086016780271517687&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-123378401.nonecase&utm_term=podman&spm=1018.2226.3001.4450)
 - [podman 命令](https://blog.csdn.net/xixihahalelehehe/article/details/121611523?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681086016780271517687%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681086016780271517687&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-121611523.nonecase&utm_term=podman&spm=1018.2226.3001.4450)
 -  [crictl 命令](https://blog.csdn.net/xixihahalelehehe/article/details/116591151?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681092916780271596159%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681092916780271596159&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-13-116591151.nonecase&utm_term=%E5%91%BD%E4%BB%A4&spm=1018.2226.3001.4450)
 - [kubectl 命令](https://blog.csdn.net/xixihahalelehehe/article/details/107714611?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164731349216781683936376%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164731349216781683936376&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-107714611.nonecase&utm_term=kubectl&spm=1018.2226.3001.4450)
 - [operator-sdk 命令](https://blog.csdn.net/xixihahalelehehe/article/details/112024963?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681502916780255218754%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681502916780255218754&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-112024963.nonecase&utm_term=operator-sdk&spm=1018.2226.3001.4450)
