#  kubernetes pod podsecurityPolicies
tags: 资源对象，pod



[
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fa6d8cdcf85bcfc60478dd9a86344dcf.jpeg#pic_center)](https://www.rottentomatoes.com/tv/devs)

*美剧《开发者》（Devs）颠覆感藏在最后。*


##  1. 简介
[Pod Security Policies](https://kubernetes.io/docs/concepts/security/pod-security-policy/)（PSP）是集群级的 Pod 安全策略，自动为集群内的 Pod 和 Volume 设置 Security Context。

使用 PSP 需要 API Server 开启 extensions/v1beta1/podsecuritypolicy，并且配置 PodSecurityPolicy admission 控制器。

> 注意： PodSecurityPolicy 自 Kubernetes v1.21 起已弃用，并将在 v1.25 中删除。我们建议迁移到[Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/)或 3rd party admission 插件。有关迁移指南，请参阅[从 PodSecurityPolicy 迁移到内置 PodSecurity 准入控制器](https://kubernetes.io/docs/tasks/configure-pod-container/migrate-from-psp/)。有关弃用的更多信息，请参阅[PodSecurityPolicy 弃用：过去、现在和未来](https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/)。

##  2. API 版本对照表
| Kubernetes 版本 | Extension 版本       |
|---------------|--------------------|
| v1.5-v1.15    | extensions/v1beta1 |
| v1.10+        | policy/v1beta1     |


##  3. 支持的控制项
| 控制项                             | 说明                    |
|---------------------------------|-----------------------|
| privileged                      | 运行特权容器                |
| defaultAddCapabilities          | 可添加到容器的 Capabilities  |
| requiredDropCapabilities        | 会从容器中删除的 Capabilities |
| allowedCapabilities             | 允许使用的 Capabilities 列表 |
| volumes                         | 控制容器可以使用哪些 volume     |
| hostNetwork                     | 允许使用 host 网络          |
| hostPorts                       | 允许的 host 端口列表         |
| hostPID                         | 使用 host PID namespace |
| hostIPC                         | 使用 host IPC namespace |
| seLinux                         | SELinux Context       |
| runAsUser                       | user ID               |
| supplementalGroups              | 允许的补充用户组              |
| fsGroup                         | volume FSGroup        |
| readOnlyRootFilesystem          | 只读根文件系统               |
| allowedHostPaths                | 允许 hostPath 插件使用的路径列表 |
| allowedFlexVolumes              | 允许使用的 flexVolume 插件列表 |
| allowPrivilegeEscalation        | 允许容器进程设置 no_new_privs |
| defaultAllowPrivilegeEscalation | 默认是否允许特权升级            |


##  4. 实例
###   4.1 控制是否允许超出父进程特权
`allowPrivilegeEscalation`：控制进程是否可以获得超出其父进程的特权。 此布尔值直接控制是否为容器进程设置 `no_new_privs`标志。 当容器满足一下条件之一时，`allowPrivilegeEscalation` 总是为 true： 以特权模式运行，或者 具有 `CAP_SYS_ADMIN` 权能 `readOnlyRootFilesystem`：以只读方式加载容器的根文件系统。
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

###  4.2  限制端口
限制容器的 host 端口范围为 8000-8080

```bash
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: permissive
spec:
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  hostPorts:
  - min: 8000
    max: 8080
  volumes:
  - '*'
```

###  4.3  限制只允许使用 lvm 和 cifs 等 flexVolume 插件

```bash
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: allow-flex-volumes
spec:
  fsGroup:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
    - flexVolume
  allowedFlexVolumes:
    - driver: example/lvm
    - driver: example/cifs
```

参考：

 - [Pod Security Policies](https://kubernetes.io/docs/concepts/security/pod-security-policy/)

