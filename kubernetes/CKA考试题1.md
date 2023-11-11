1. 列出环境内所有的pv 并以 name字段排序（使用kubectl自带排序功能）

```bash
kubectl get pv --sort-by=.metadata.name
```
考点：[kubectl get](https://blog.csdn.net/xixihahalelehehe/article/details/107714611)
2. 列出指定pod的日志中状态为Error的行，并记录在指定的文件上

```bash
kubectl logs <podname> | grep bash > /opt/KUCC000xxx/KUCC000xxx.txt
```

3.列出k8s可用的节点，不包含不可调度的 和 NoReachable的节点，并把数字写入到文件里
#笨方法，人工数

```bash
kubectl get nodes
```

参考：kubernetes备忘：[https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

4.创建一个pod名称为nginx，并将其调度到节点为 disk=stat上

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

参考：将pod分配给节点，[https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/)

5. 提供一个pod的yaml，要求添加Init Container，Init Container的作用是创建一个空文件，pod的Containers判断文件是否存在，不存在则退出

```bash
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name:  nginx
    image:  busybox:1.28
    ports:
    - containerPort: 80
    command：['sh', '-c', 'if [ ! -e "/opt/myfile"]; then exit;fi;']
    volumeMounts:
    - name: workdir
      mountPath: /opt/
  # These containers are run during pod initialization
  initContainers:
  - name: install
    image: busybox
    command: ['sh', '-c', 'touch -p /opt/myfile']
    volumeMounts:
    - name: workdir
      mountPath: /opt/
  volumes:
  - name: workdir
    emptyDir: {}
```

参考：Init Container [https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/#creating-a-pod-that-has-an-init-container](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/#creating-a-pod-that-has-an-init-container)

  [https://kubernetes.io/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

6. 指定在命名空间内创建一个pod名称为test，内含四个指定的镜像nginx、redis、memcached、busybox
第一种方式
```bash
kubectl run test --image=nginx --image=redis --image=memcached --image=buxybox --restart=Never -n <namespace>
```
第二种方式
cat pod4.yaml 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: kucc
spec:
  containers:
  - name: nginx
    image: nginx
  - name: redis
    image: redis
  - name: memcached
    image: memcached
  - name: consul
    image: consul
```

```bash
kubectl apply -f pod4.yaml
```

必须自己写 yaml
7. 创建一个pod名称为test，镜像为nginx，Volume名称cache-volume为挂在在/data目录下，且Volume是non-Persistent的

```bash
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: test
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

参考：volume ： [https://kubernetes.io/docs/concepts/storage/volumes/#local](https://kubernetes.io/docs/concepts/storage/volumes/#local)


8. 列出Service名为test下的pod 并找出使用CPU使用率最高的一个，将pod名称写入文件中
#使用-o wide 获取service test的SELECTOR

```bash
kubectl get svc test -o wide
```

##获取结果我就随便造了

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE SELECTOR

test ClusterIP None <none> 3306/TCP 50d app=wordpress,tier=mysql

#获取对应SELECTOR的pod使用率，找到最大那个写入文件中

```bash
kubectl top pods -l 'app=wordpress,tier=mysql'
```

9.创建一个Pod名称为nginx-app，镜像为nginx，并根据pod创建名为nginx-app的Service，type为NodePort

```bash
kubectl run nginx-app --image=nginx
```

之后创建service

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-app
spec:
  selector:
    run: nginx-app
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  - name: https
    protocol: TCP
    port: 443
    targetPort: 9377
  type: NodePort
```

参考：service  [https://kubernetes.io/docs/concepts/services-networking/service/](https://kubernetes.io/docs/concepts/services-networking/service/)

10.创建一个nginx的Workload，保证其在每个节点上运行，注意不要覆盖节点原有的Tolerations
 这道题直接复制文档的yaml太长了，由于damonSet的格式和Deployment格式差不多，我用旁门左道的方法 先创建Deploy，再修改，这样速度会快一点

```bash
kubectl run nginx --image=nginx:1.17.1 -oyaml > nginx-daemonset.yaml
```

# 修改yaml文件

```bash
vi nginx-daemonset.yaml
#修改apiVersion和kind
#apiVersion: extensions/v1beta1
#kind: Deployment
apiVersion:apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
#去掉replicas
# replicas: 1
  selector:
    matchLabels:
      run: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

11. 将deployment为nginx-app的副本数从1变成4。
#方法1

```bash
kubectl scale  --replicas=4 deployment nginx-app
```

#方法2，使用edit命令将replicas改成4

```bash
kubectl edit deploy nginx-app
```

参考： [https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/#scaling-the-application-by-increasing-the-replica-count](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/#scaling-the-application-by-increasing-the-replica-count)

[https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

12. 创建nginx-app的deployment ，使用镜像为nginx:1.11.0-alpine ,修改镜像为1.11.3-alpine，并记录升级，再使用回滚，将镜像回滚至nginx:1.11.0-alpine

```bash
# 创建nginx-app的deployment
kubectl run nginx-app --image=nginx:1.11.0-alpine --record
# 修改镜像，nginx-app为container的名字
kubectl set image deployment nginx-app nginx-app=nginx:1.11.3-alipne
# 回滚
kubectl rollout undo deployment nginx-app
```

参考：[https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

13. 根据已有的一个nginx的pod、创建名为nginx的svc、并使用nslookup查找出service dns记录，pod的dns记录并分别写入到指定的文件中

```bash
#创建一个服务
kubectl create svc nodeport nginx --tcp=80:80
#创建一个指定版本的busybox，用于执行nslookup
kubectl create -f https://k8s.io/examples/admin/dns/busybox.yaml
#将svc的dns记录写入文件中
kubectl exec -ti busybox -- nslookup nginx > 指定文件
#获取pod的ip地址
kubectl get pod nginx -o yaml
#将获取的pod ip地址使用nslookup查找dns记录
kubectl exec -ti busybox -- nslookup <Pod ip>
```

考点：网络相关，DNS解析

参考：https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/

14. 创建Secret 名为mysecret，内含有password字段，值为bob，然后 在pod1里 使用ENV进行调用，Pod2里使用Volume挂载在/data 下
#将密码值使用base64加密,记录在Notepad里

```bash
echo -n 'bob' | base64
```

secret.yaml

```bash
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: Ym9i
```

pod1.yaml   使用env进行调用

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
```

pod2.yaml   挂载到data目录下

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: mypod
    image: nginx
    volumeMounts:
    - name: mysecret
      mountPath: "/data"
      readOnly: true
  volumes:
  - name: mysecret
    secret:
      secretName: mysecret
```

参考：[https://kubernetes.io/docs/concepts/configuration/secret/](https://kubernetes.io/docs/concepts/configuration/secret/)


15. 使node1节点不可调度，并重新分配该节点上的pod
#直接drain会出错，需要添加--ignore-daemonsets --delete-local-data参数

```bash
kubectl drain node node1  --ignore-daemonsets --delete-local-data
```

参考：[https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)

16. 使用etcd 备份功能备份etcd（提供enpoints，ca、cert、key）
ETCDCTL_API=3 

```bash
etcdctl --endpoints https://127.0.0.1:2379 \
--cacert=ca.pem --cert=cert.pem --key=key.pem \
snapshot save snapshotdb
```

参考： [https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)


 17. 给出一个失联节点的集群，排查节点故障，要保证改动是永久的。
#查看集群状态

```bash
kubectl get nodes
```

#查看故障节点信息

```bash
kubectl describe node node1
```

#Message显示kubelet无法访问（记不清了）
#进入故障节点

```bash
ssh node1
```

#查看节点中的kubelet进程

```bash
ps -aux | grep kubelete
```

#没找到kubelet进程，查看kubelet服务状态

```bash
systemctl status kubelet.service
```

#kubelet服务没启动，启动服务并观察

```bash
systemctl start kubelet.service
```

#启动正常，enable服务

```bash
systemctl enable kubelet.service
```

#回到考试节点并查看状态
exit

```bash
kubectl get nodes #正常
```

参考：[https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

18. 创建一个pv，类型是hostPath，位于/data中，大小1G，模式ReadOnlyMany

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-host
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  hostPath:
    path: /data
```

参考： [https://kubernetes.io/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)


19. 给出一个集群，将节点node1添加到集群中，并使用TLS bootstrapping
 参考：[https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#kube-controller-manager-configuration](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/#kube-controller-manager-configuration)
 [https://blog.fanfengqiang.com/2019/03/11/kubernetes-TLS-Bootstrapping%E9%85%8D%E7%BD%AE/](https://blog.fanfengqiang.com/2019/03/11/kubernetes-TLS-Bootstrapping%E9%85%8D%E7%BD%AE/)




