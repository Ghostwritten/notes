#  kuberenetes pod SecurityContext
tags: Pod,对象





----
## 1. 简介
**安全上下文**（Security Context）定义 Pod 或 Container 的特权与访问控制设置。安全上下文包括但不限于：

 - `自主访问控制（Discretionary Access Control）`：基于 用户 ID（UID）和组 ID（GID）.来判定对对象（例如文件）的访问权限
 - `安全性增强的 Linux（SELinux）`：为对象赋予安全性标签。
 - 以特权模式或者非特权模式运行。
 - `Linux 权能`: 为进程赋予 root 用户的部分特权而非全部特权。
 - `AppArmor`：使用程序文件来限制单个程序的权限。
 - `Seccomp`：限制一个进程访问文件描述符的权限。
 - `AllowPrivilegeEscalation`：控制进程是否可以获得超出其父进程的特权。此布尔值直接控制是否为容器进程设置no_new_privs 标志。当容器以特权模式运行或者具有 CAP_SYS_ADMIN权能时，AllowPrivilegeEscalation 总是为 true。
 - `readOnlyRootFilesystem`：以只读方式加载容器的根文件系统。


## 2. 功能
### 2.1 配置pod的安全上下文

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
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```
 - 在配置文件中，该`runAsUser`字段指定对于Pod中的任何容器，所有进程都以用户ID 1000运行。
 - 该`runAsGroup`字段为Pod中的任何容器中的所有进程指定主组ID3000。如果省略此字段，则容器的主要组ID将为root（0）。runAsGroup指定时，用户1000和组3000也将拥有所有创建的文件。
 - 由于`fsGroup`指定了字段，因此容器的所有进程也是补充组ID2000的一部分。卷的所有者/data/demo和在该卷中创建的任何文件都将是组ID 2000。


```bash
$ kubectl apply -f https://k8s.io/examples/pods/security/security-context.yaml
$ kubectl get pod security-context-demo
$ kubectl exec -it security-context-demo -- sh
$ ps 
PID   USER     TIME  COMMAND
    1 1000      0:00 sleep 1h
    6 1000      0:00 sh
...

$ cd /data
$ ls -l
drwxrwsrwx 2 root 2000 4096 Jun  6 20:08 demo
$ cd demo
$ echo hello > testfile
$ ls -l
-rw-r--r-- 1 1000 2000 6 Jun  6 20:08 testfile
$ id
uid=1000 gid=3000 groups=2000
```
### 2.2 为 Pod 配置卷访问权限和属主变更策略

```bash
FEATURE STATE: Kubernetes v1.18 [alpha]
```
默认情况下，Kubernetes 在挂载一个卷时，会递归地更改每个卷中的内容的属主和访问权限，使之与 Pod 的 securityContext 中指定的 fsGroup 匹配。对于较大的数据卷，检查和变更属主与访问权限可能会花费很长时间，降低 Pod 启动速度。你可以在 securityContext 中使用 fsGroupChangePolicy 字段来控制 Kubernetes 检查和管理卷属主和访问权限的方式。

fsGroupChangePolicy - fsGroupChangePolicy 定义在卷被暴露给 Pod 内部之前对其 内容的属主和访问许可进行变更的行为。此字段仅适用于那些支持使用 fsGroup 来 控制属主与访问权限的卷类型。此字段的取值可以是：

 - `OnRootMismatch`：只有根目录的属主与访问权限与卷所期望的权限不一致时，才改变其中内容的属主和访问权限。这一设置有助于缩短更改卷的属主与访问权限所需要的时间。
 - `Always`：在挂载卷时总是更改卷中内容的属主和访问权限。

```bash
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  fsGroupChangePolicy: "OnRootMismatch"
```
这是一个Alpha功能。要使用它，请为`kube-api-server`，`kube-controller-manager`和`kubelet` 启用功能门 `ConfigurableFSGroupPolicy`。

### 2.3 对启动的容器使用sysctl修改内核参数，比如es这类容器镜像。

```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sc-demo
spec:
  template:
    containers:
    - name: sc-demo
      image: xxxxxx
      command: [ "sh", "-c", "sleep 1h" ]
      securityContext:
        privileged: true
```
给予 `privileged: true` 是一个比较粗的权限，一般不建议如此，可以为权限定义更细粒度的权限，类似需要在容器中使用 perf 命令，则可以进行如下定义：

```bash
...
securityContext:
    capabilities:
        add: ["SYS_ADMIN"]
```
具体 capabilities 的使用规则可参考：在 Kubernetes 中配置 `Container Capabilities`

### 2.4 对启动的容器赋予SELinux标签

```bash
...
securityContext:
  seLinuxOptions:
    level: "s0:c123,c456"
```
要指定 SELinux，需要在宿主操作系统中装载 SELinux 安全性模块。`seLinuxOptions` 字段的取值是一个 SELinuxOptions 对象

##  3. 实例

![在这里插入图片描述](https://img-blog.csdnimg.cn/202105141409585.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210514141050142.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210514141138804.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
参考链接：
[https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#before-you-begin](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#before-you-begin)

### 3.1 Set Container User and Group

```c
root@master:~/cks/securitytext# k run pod --image=busybox --command -oyaml --dry-run=client > pod.yaml -- sh -c 'sleep 1d'
root@master:~/cks/securitytext# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/securitytext# k create -f pod.yaml 
root@master:~/cks/security# k get pods
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          83s


root@master:~/cks/securitytext# k exec -ti pod -- sh
/ $ id
uid=1000 gid=3000

/ $ touch test
touch: test: Permission denied
/ $ cd /tmp
/tmp $ touch test
/tmp $ ls -lh
total 0      
-rw-r--r--    1 1000     3000           0 May 15 15:00 test

```

### 3.2 Force Container Non-Root

```c
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      runAsNonRoot: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}




root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted
root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created
root@master:~/cks/securitytext# k get pod 
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          68s



#未指定非root用户
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      runAsNonRoot: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted
root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created


oot@master:~/cks/securitytext# k get pod 
NAME   READY   STATUS                       RESTARTS   AGE
pod    0/1     CreateContainerConfigError   0          22s

#报错
root@master:~/cks/securitytext# k describe pod pod 
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  50s                default-scheduler  Successfully assigned default/pod to node2
  Normal   Pulled     33s                kubelet            Successfully pulled image "busybox" in 16.355541429s
  Warning  Failed     29s (x2 over 33s)  kubelet            Error: container has runAsNonRoot and image will run as root (pod: "pod_default(99f9dcab-cebe-4432-ad70-08c86d3ed9be)", container: pod)
  Normal   Pulled     29s                kubelet            Successfully pulled image "busybox" in 2.345508306s
  Normal   Pulling    16s (x3 over 49s)  kubelet            Pulling image "busybox"

```


### 3.3 Privileged Containers
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515231135395.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515231243387.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 3.4 Create Privileged Containers
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515231407815.png)

```c
#非root用户测试执行sysctl命令
root@master:~/cks/securitytext# k get pods -w
NAME   READY   STATUS              RESTARTS   AGE
pod    0/1     ContainerCreating   0          3s
pod    1/1     Running             0          23s
root@master:~/cks/securitytext# k exec -ti pod -- sh
/ $ sysctl kernel.hostname=attacker
sysctl: error setting key 'kernel.hostname': Read-only file system   #无权限
/ $ exit
command terminated with exit code 1

#默认用户测试执行sysctl命令
root@master:~/cks/securitytext# vim pod.yaml  
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
#    securityContext:
#      runAsNonRoot: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted
root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created

root@master:~/cks/securitytext# k get pods
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          32s
root@master:~/cks/securitytext# k exec -ti pod -- sh
/ # sysctl kernel.hostname=attacker
sysctl: error setting key 'kernel.hostname': Read-only file system  #无权限
/ # id
uid=0(root) gid=0(root) groups=10(wheel)


#添加privileged: true特权
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      privileged: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted
root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created
root@master:~/cks/securitytext# k get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          36s
root@master:~/cks/securitytext# k exec -ti pod -- sh
/ # id
uid=0(root) gid=0(root) groups=10(wheel)
/ # sysctl kernel.hostname=attacker  #命令生效
kernel.hostname = attacker


```


### 3.5 PrivilegeEscalation
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515232409858.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515232446237.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

### 3.6 Practice - Disable PriviledgeEscalation

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210515232455714.png)

```c
root@master:~/cks/securitytext# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
#  securityContext:
#    runAsUser: 1000
#    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true 
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/securitytext# kubectl -f pod.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "pod" force deleted

root@master:~/cks/securitytext# k create -f pod.yaml 
pod/pod created
root@master:~/cks/securitytext# k get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          29s


root@master:~/cks/securitytext# k exec -ti pod -- sh
/ # cat /proc/1/status
Name:	sleep
State:	S (sleeping)
Tgid:	1
Ngid:	0
Pid:	1
PPid:	0
TracerPid:	0
Uid:	0	0	0	0
Gid:	0	0	0	0
FDSize:	64
Groups:	10 
NStgid:	1
NSpid:	1
NSpgid:	1
NSsid:	1
VmPeak:	    1308 kB
VmSize:	    1308 kB
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	       4 kB
VmRSS:	       4 kB
VmData:	      48 kB
VmStk:	     132 kB
VmExe:	     892 kB
VmLib:	18446744073709551612 kB
VmPTE:	      12 kB
VmPMD:	       8 kB
VmSwap:	       0 kB
HugetlbPages:	       0 kB
Threads:	1
SigQ:	0/7778
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000000000004
SigCgt:	0000000000000000
CapInh:	00000000a80425fb
CapPrm:	00000000a80425fb
CapEff:	00000000a80425fb
CapBnd:	00000000a80425fb
CapAmb:	0000000000000000
Seccomp:	0
Speculation_Store_Bypass:	thread vulnerable
Cpus_allowed:	00000000,00000000,00000000,00000003
Cpus_allowed_list:	0-1
Mems_allowed:	00000000,00000001
Mems_allowed_list:	0
voluntary_ctxt_switches:	47
nonvoluntary_ctxt_switches:	10

```

参考：

 - [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
 -  [linux selinux策略管理与标签](https://ghostwritten.blog.csdn.net/article/details/123700519)
 - [您应该了解10 个 Kubernetes安全上下文设置（待翻译）](https://snyk.io/blog/10-kubernetes-security-context-settings-you-should-understand/)


![在这里插入图片描述](https://img-blog.csdnimg.cn/00ff14d4d3cd4cdfab8a0c79e8d485b0.png)
