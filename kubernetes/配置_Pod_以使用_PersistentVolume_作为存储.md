任务：

 1. 集群管理员创建由物理存储支持的 PersistentVolume。管理员不将卷与任何 Pod 关联。
 2. 群集用户创建一个 PersistentVolumeClaim，它将自动绑定到合适PersistentVolume。
 3. 用户创建一个使用 PersistentVolumeClaim 作为存储的 Pod。


准备：一个挂载的本地目录并且目录下有个文件

```bash
mkdir /mnt/data
echo 'Hello from Kubernetes storage' > /mnt/data/index.html
```
## 1. 创建 一个hostpath类型的PersistentVolume 

```bash
$ vim pods/storage/pv-volume.yaml 
```

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

```bash
#创建pv
$ kubectl create -f pods/storage/pv-volume.yaml 
#查看pv
$ kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
task-pv-volume   10Gi       RWO            Retain           Available           manual                  8s
```
输出结果显示该 PersistentVolume 的状态（STATUS） 为 Available。 这意味着它还没有被绑定给 `PersistentVolumeClaim`。

## 2. 创建 PersistentVolumeClaim 

```bash
$ vim pods/storage/pv-claim.yaml 
```

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

```bash
#创建pvs
kubectl create -f pods/storage/pv-claim.yaml 
```

创建 PersistentVolumeClaim 之后，Kubernetes 控制平面将查找满足申领要求的 PersistentVolume。 如果控制平面找到具有相同 StorageClass 的适当的 PersistentVolume，则将 PersistentVolumeClaim 绑定到该 PersistentVolume 上。

查看 PersistentVolume 信息：

```bash
$ kubectl get pv task-pv-volume
NAME            STATUS    VOLUME           CAPACITY   ACCESSMODES   STORAGECLASS   AGE
task-pv-claim   Bound     task-pv-volume   10Gi       RWO           manual         30s
```
输出结果表明该 PersistentVolumeClaim 绑定了你的 PersistentVolume task-pv-volume。

## 3. 创建 Pod
$ vim pods/storage/pv-pod.yaml 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```
**注意**: Pod 的配置文件指定了 PersistentVolumeClaim，但没有指定 PersistentVolume。对 Pod 而言，PersistentVolumeClaim 就是一个存储卷.

```bash
#创建pod
kubectl create -f pods/storage/pv-pod.yaml
# 查看是否正常
kubectl get pod task-pv-pod
#进入pod
kubectl exec -it task-pv-pod -- /bin/bash
#验证是nginx服务否正常
root@task-pv-pod:/# apt-get update
root@task-pv-pod:/# apt-get install curl
root@task-pv-pod:/# curl localhost
Hello from Kubernetes storage
```

## 4. 访问控制 
使用 group ID（GID）配置的存储仅允许 Pod 使用相同的 GID 进行写入。 GID 不匹配或缺少将会导致许可被拒绝的错误。 为了减少与用户的协调，管理员可以使用 GID 对 PersistentVolume 进行注解。 这样 GID 就能自动的添加到使用 PersistentVolume 的任何 Pod 中。

使用 `pv.beta.kubernetes.io/gid` 注解的方法如下所示：

```bash
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv1
  annotations:
    pv.beta.kubernetes.io/gid: "1234"
```
当 Pod 使用带有 GID 注解的 PersistentVolume 时，注解的 GID 会被应用于 Pod 中的所有容器，应用的方法与 Pod 的安全上下文中指定的 GID 相同。 每个 GID，无论是来自 PersistentVolume 注解还是来自 Pod 的规范，都应用于每个容器中运行的第一个进程。

