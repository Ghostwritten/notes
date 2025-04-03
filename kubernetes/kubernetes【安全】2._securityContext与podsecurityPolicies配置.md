

----

 - [Kubernetes【安全】1. SecurityContext安全上下文](https://blog.csdn.net/xixihahalelehehe/article/details/108539153)
 - [kubernetes【安全】2.securityContext与podsecurityPolicies配置](https://ghostwritten.blog.csdn.net/article/details/116789828)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)

----

## 1. 介绍
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4bc523e5abc030ff2deca2500756562e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6349eb45fcd2566a6f4ab317f716c9f4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/53e3670cce7d0cf7cbcd41ca415d61a0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8c8d4a4f31cb7bfd3965f2b9e83651d.png)
参考链接：
[https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#before-you-begin](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#before-you-begin)

## 2. Set Container User and Group

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

## 3. Force Container Non-Root

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


## 4. Privileged Containers
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a5343780192a0a63097983897a0f9514.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d262d165cf209d19fea38901b2b74317.png)
## 5. Create Privileged Containers
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad0a7f46c629b153814bff6136da51e3.png)

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


## 6. PrivilegeEscalation
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b00acedbd363bfe982cb26ebfb84d5af.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4d6793331629c9642c309607cbdd3cfc.png)

## 7. Practice - Disable PriviledgeEscalation

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/47d0decf46c1803013b82acd91b524f7.png)

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
## 8. PodSecurityPolicies
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1e70523c738da2eb07db582fd8cae91a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/77855a84dfb28eae41950f55167db785.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5e097a0f04b647045c142df562fcff0e.png)

##  9. Create and enable PodSecurityPolicy
[pod-security-policy](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8433068411e788457a61c1bd4c6203f5.png)

```c
root@master:~/cks/securitytext# vim /etc/kubernetes/manifests/kube-apiserver.yaml
---
    - --enable-admission-plugins=NodeRestriction,PodSecurityPolicy
---


root@master:~/cks/securitytext# cat psp.yaml 
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: default
spec:
  allowPrivilegeEscalation: false
  privileged: false  # Don't allow privileged pods!
  # The rest fills in some required fields.
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


root@master:~/cks/securitytext# k create -f psp.yaml 
podsecuritypolicy.policy/default created


root@master:~/cks/securitytext# k create deploy nginx --image=nginx
deployment.apps/nginx created
root@master:~/cks/securitytext# k get deploy nginx -w
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/1     0            0           22s
^Croot@master:~/cks/securitytext#  k run nginx --image=nginx
pod/nginx created
root@master:~/cks/securitytext# k get pod nginx
NAME    READY   STATUS              RESTARTS   AGE
nginx   1/1     Running             0          44s

root@master:~/cks/securitytext#  k create role psp-access --verb=use --resource=podsecuritypolicies
role.rbac.authorization.k8s.io/psp-access created
root@master:~/cks/securitytext# k create rolebinding psp-access --role=psp-access --serviceaccount=default:default
rolebinding.rbac.authorization.k8s.io/psp-access created
root@master:~/cks/securitytext# k get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/1     0            0           3m26s
root@master:~/cks/securitytext# k delete deploy nginx
deployment.apps "nginx" deleted
root@master:~/cks/securitytext# k create deploy nginx --image=nginx
deployment.apps/nginx created
^Croot@master:~/cks/securitytext# k get deploy nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           20s

```

allowPrivilegeEscalation设置为rue

```bash
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

root@master:~/cks/securitytext# k -f pod.yaml create
Error from server (Forbidden): error when creating "pod.yaml": pods "pod" is forbidden: PodSecurityPolicy: unable to admit pod: [spec.containers[0].securityContext.allowPrivilegeEscalation: Invalid value: true: Allowing privilege escalation for containers is not allowed]
```

[更多细节参考](https://www.cnblogs.com/sunsky303/p/11090540.html)


总结：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a33a6ab206a3653d5c1658d867863898.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c71208150f7ea2301be01b8f61043a00.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d54b4543d97b2c83175b3049b36c75ac.png)

