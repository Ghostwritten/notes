

---
## 1. 简介



了解 Kube-apparmor-manager 如何帮助您管理 Kubernetes 上的 AppArmor 配置文件，以**减少集群的攻击面**。

[AppArmor](https://apparmor.net/)是一个 Linux 内核安全模块，它补充了标准的 Linux 用户和基于组的权限，以将程序限制在一组有限的资源中。

AppArmor 可以针对任何应用程序进行配置，以减少其潜在的攻击面并提供更深入的防御。您可以通过配置文件对其进行配置并将其调整为将特定程序或容器所需的访问列入白名单，例如 Linux 功能、网络访问、文件权限等。

在本博客中，我们将首先给出 AppArmor 配置文件的快速示例，以及 Kubernetes 工作负载如何使用它来减少攻击面。然后，我们将推出一个新的开源工具，[KUBE-AppArmor-manager](https://github.com/sysdiglabs/kube-apparmor-manager)，并告诉你如何能帮助到Kubernetes集群内轻松管理AppArmor配置文件。最后但并非最不重要的一点是，我们将演示如何从图像配置文件构建 AppArmor 配置文件以[防止反向 shell 攻击](https://sysdig.com/blog/reverse-shell-falco-sysdig-secure/)。

## 2. 准备
### 2.1 Kubernetes & docker版本
Kubernetes 版本至少是 `v1.4` -- `AppArmor` 在 `Kubernetes v1.4` 版本中才添加了对 AppArmor 的支持。早于 v1.4 版本的 Kubernetes 组件不知道新的 AppArmor 注释，并且将会 默认忽略 提供的任何 AppArmor 设置。为了确保您的 Pods 能够得到预期的保护，必须验证节点的 Kubelet 版本：

```bash
$ kubectl get nodes -o=jsonpath=$'{range .items[*]}{@.metadata.name}: {@.status.nodeInfo.kubeletVersion}\n{end}'
master: v1.20.1
node1: v1.20.1
node2: v1.20.1

$ kubectl get nodes -o=jsonpath=$'{range .items[*]}{@.metadata.name}: {@.status.nodeInfo.containerRuntimeVersion}\n{end}'
master: docker://19.3.4
node1: docker://19.3.4
node2: docker://19.3.4

```
### 2.2 内核模块
AppArmor 内核模块已启用 -- 要使 Linux 内核强制执行 AppArmor 配置文件，必须安装并且启动 AppArmor 内核模块。默认情况下，有几个发行版支持该模块，如 Ubuntu 和 SUSE，还有许多发行版提供可选支持。要检查模块是否已启用，请检查 `/sys/module/apparmor/parameters/enabled` 文件：

```bash
 cat /sys/module/apparmor/parameters/enabled
 Y
```
###  2.3 节点配置文件加载
配置文件已加载 -- 通过指定每个容器都应使用 AppArmor 配置文件，AppArmor 应用于 Pod。如果指定的任何配置文件尚未加载到内核， Kubelet (>=v1.4) 将拒绝 Pod。通过检查 /sys/kernel/security/apparmor/profiles 文件，可以查看节点加载了哪些配置文件。例如:

```bash
$ ssh root@192.168.211.41 "sudo cat /sys/kernel/security/apparmor/profiles | sort"
docker-default (enforce)
docker-nginx (enforce)
/sbin/dhclient (enforce)
/usr/bin/curl (enforce)
/usr/lib/connman/scripts/dhclient-script (enforce)
/usr/lib/NetworkManager/nm-dhcp-client.action (enforce)
/usr/lib/NetworkManager/nm-dhcp-helper (enforce)
/usr/sbin/tcpdump (enforce)
```
###  2.4 kubelet版本
只要 Kubelet 版本包含 AppArmor 支持(>=v1.4)，如果不满足任何先决条件，Kubelet 将拒绝带有 AppArmor 选项的 Pod。您还可以通过检查节点就绪状况消息来验证节点上的 AppArmor 支持(尽管这可能会在以后的版本中删除)：

```bash
$ kubectl get nodes -o=jsonpath=$'{range .items[*]}{@.metadata.name}: {.status.conditions[?(@.reason=="KubeletReady")].message}\n{end}'
master: kubelet is posting ready status. AppArmor enabled
node1: kubelet is posting ready status. AppArmor enabled
node2: kubelet is posting ready status. AppArmor enabled
```

## 3. AppArmor 配置文件
AppArmor 配置文件被指定为 `per-container`。要指定要用其运行 Pod 容器的 AppArmor 配置文件，请向 Pod 的元数据添加注释：

```bash
container.apparmor.security.beta.kubernetes.io/<container_name>: <profile_ref>
```

`<container_name>` 的名称是容器的简称，用以描述简介，并且简称为 `<profile_ref>` 。`<profile_ref>` 可以作为其中之一：

 - `runtime/default` 应用运行时的默认配置
 - `localhost/<profile_name>` 应用在名为 `<profile_name>` 的主机上加载的配置文件
 - `unconfined` 表示不加载配置文件

`apparmor` 配置文件定义了目标受限应用程序可以访问系统上的哪些资源（如网络、系统功能或文件）。

下面是一个简单的 AppArmor 配置文件示例：

```bash
profile k8s-apparmor-example-deny-write flags=(attach_disconnected) {
  file,
  # Deny all file writes.
  deny /** w,
}
```
在此示例中，配置文件授予应用程序所有类型的访问权限，但写入整个文件系统除外。它包含两个规则：

 - `file`: 允许对整个文件系统进行各种访问
 - `deny` /** w: 拒绝在根目录下写入任何文件/。该表达式/**转换为根目录下的任何文件，以及其子目录下的文件。

通过以下步骤设置 Kubernetes 集群以便容器可以使用 apparmor 配置文件：

 - 在所有集群节点上安装并启用 AppArmor。
 - 将要使用的 apparmor 配置文件复制到每个节点，并将其解析为强制（enforce）模式或抱怨（complain）模式。
 - 使用 AppArmor 配置文件名称注释容器工作负载。

以下是在 Pod 中使用配置文件的方法：

```bash
apiVersion: v1
kind: Pod
metadata:
  name: hello-apparmor
  annotations:
    # Tell Kubernetes to apply the AppArmor profile "k8s-apparmor-example-deny-write".
    container.apparmor.security.beta.kubernetes.io/hello: localhost/k8s-apparmor-example-deny-write
spec:
  containers:
  - name: hello
    image: busybox
    command: [ "sh", "-c", "echo 'Hello AppArmor!' && sleep 1h" ]
```
在上面的 `pod yaml` 中，名为的容器hello正在使用名为的 `AppArmor` 配置文件`k8s-apparmor-example-deny-write`。如果 AppArmor 配置文件不存在，则 Pod 将无法创建。

每个配置文件都可以在强制模式（阻止访问不允许的资源）或抱怨模式（仅报告违规）下运行。构建 AppArmor 配置文件后，最好先将其应用到抱怨模式，然后让工作负载运行一段时间。通过分析 AppArmor 日志，您可以检测并修复任何误报活动。一旦您有足够的信心，您就可以将配置文件转为强制模式。

如果之前的配置文件在强制模式下运行，它将阻止任何文件写入活动：

```bash
$ kubectl exec hello-apparmor touch /tmp/test
touch: /tmp/test: Permission denied
error: error executing remote command: command terminated with non-zero exit code: Error executing in Docker Container:
```

这是一个简化的例子。


## 4. Practice - AppArmor for curl

```c
root@master:~/cks/runtime-security# aa-status 
apparmor module is loaded.
6 profiles are loaded.
6 profiles are in enforce mode.
   /sbin/dhclient
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/sbin/tcpdump
   docker-default
0 profiles are in complain mode.
10 processes have profiles defined.
10 processes are in enforce mode.
   docker-default (26146) 
   docker-default (26164) 
   docker-default (26184) 
   docker-default (26480) 
   docker-default (27226) 
   docker-default (32926) 
   docker-default (47085) 
   docker-default (47820) 
   docker-default (47906) 
   docker-default (48662) 
0 processes are in complain mode.
0 processes are unconfined but have a profile defined.


root@master:~/cks/runtime-security# apt-get install apparmor-utils


root@master:~/cks/runtime-security# aa-
aa-audit           aa-complain        aa-enabled         aa-genprof         aa-remove-unknown  aa-update-browser  
aa-autodep         aa-decode          aa-enforce         aa-logprof         aa-status          
aa-cleanprof       aa-disable         aa-exec            aa-mergeprof       aa-unconfined   


root@master:~/cks/runtime-security# aa-genprof curl  



root@master:~/cks/runtime-security# aa-status 
apparmor module is loaded.
7 profiles are loaded.
7 profiles are in enforce mode.
   /sbin/dhclient
   /usr/bin/curl
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/sbin/tcpdump
   docker-default


root@master:~/cks/runtime-security# cd /etc/apparmor.d/
root@master:/etc/apparmor.d# ls
abstractions  cache  disable  force-complain  local  sbin.dhclient  tunables  usr.bin.curl  usr.sbin.rsyslogd  usr.sbin.tcpdump
root@master:/etc/apparmor.d# cat usr.bin.curl 
# Last Modified: Mon May 24 23:11:35 2021
#include <tunables/global>

/usr/bin/curl {
  #include <abstractions/base>

  /usr/bin/curl mr,

}



root@master:/etc/apparmor.d# aa-logprof 
Reading log entries from /var/log/syslog.
Updating AppArmor profiles in /etc/apparmor.d.
Enforce-mode changes:

Profile:        /usr/bin/curl
Network Family: inet
Socket Type:    dgram

 [1 - #include <abstractions/nameservice>]
  2 - network inet dgram, 
(A)llow / [(D)eny] / (I)gnore / Audi(t) / Abo(r)t / (F)inish
Adding #include <abstractions/nameservice> to profile.

Profile:  /usr/bin/curl
Path:     /etc/ssl/openssl.cnf
Mode:     r
Severity: 2

  1 - #include <abstractions/openssl> 
  2 - #include <abstractions/ssl_keys> 
 [3 - /etc/ssl/openssl.cnf]
(A)llow / [(D)eny] / (I)gnore / (G)lob / Glob with (E)xtension / (N)ew / Abo(r)t / (F)inish / (M)ore
Adding /etc/ssl/openssl.cnf r to profile

= Changed Local Profiles =

The following local profiles were changed. Would you like to save them?

 [1 - /usr/bin/curl]
(S)ave Changes / Save Selec(t)ed Profile / [(V)iew Changes] / View Changes b/w (C)lean profiles / Abo(r)t
Writing updated profile for /usr/bin/curl.



root@master:/etc/apparmor.d# cat usr.bin.curl 
# Last Modified: Mon May 24 23:18:29 2021
#include <tunables/global>

/usr/bin/curl {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  /etc/ssl/openssl.cnf r,
  /usr/bin/curl mr,

}



root@master:/etc/apparmor.d# curl killer.sh -v
* Rebuilt URL to: killer.sh/
*   Trying 35.227.196.29...
* TCP_NODELAY set
* Connected to killer.sh (35.227.196.29) port 80 (#0)
> GET / HTTP/1.1
> Host: killer.sh
> User-Agent: curl/7.58.0
> Accept: */*
> 
< HTTP/1.1 301 Moved Permanently
< Cache-Control: private
< Content-Type: text/html; charset=UTF-8
< Referrer-Policy: no-referrer
< Location: https://killer.sh/
< Content-Length: 215
< Date: Tue, 25 May 2021 06:19:36 GMT
< 
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://killer.sh/">here</A>.
</BODY></HTML>
* Connection #0 to host killer.sh left intact

```
##  5. Practice - AppArmor for Docker Nginx
k8s网站： [https://v1-18.docs.kubernetes.io/zh/docs/tutorials/clusters/apparmor/](https://v1-18.docs.kubernetes.io/zh/docs/tutorials/clusters/apparmor/)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b369b84b77e76413a0148a58efed60d4.png)

```c
root@master:~/cks/apparmor# cat /etc/apparmor.d/docker-nginx
#include <tunables/global>


profile docker-nginx flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>

  network inet tcp,
  network inet udp,
  network inet icmp,

  deny network raw,

  deny network packet,

  file,
  umount,

  deny /bin/** wl,
  deny /boot/** wl,
  deny /dev/** wl,
  deny /etc/** wl,
  deny /home/** wl,
  deny /lib/** wl,
  deny /lib64/** wl,
  deny /media/** wl,
  deny /mnt/** wl,
  deny /opt/** wl,
  deny /proc/** wl,
  deny /root/** wl,
  deny /sbin/** wl,
  deny /srv/** wl,
  deny /tmp/** wl,
  deny /sys/** wl,
  deny /usr/** wl,

  audit /** w,

  /var/run/nginx.pid w,

  /usr/sbin/nginx ix,

  deny /bin/dash mrwklx,
  deny /bin/sh mrwklx,
  deny /usr/bin/top mrwklx,


  capability chown,
  capability dac_override,
  capability setuid,
  capability setgid,
  capability net_bind_service,

  deny @{PROC}/* w,   # deny write for all files directly in /proc (not in a subdir)
  # deny write to files not in /proc/<number>/** or /proc/sys/**
  deny @{PROC}/{[^1-9],[^1-9][^0-9],[^1-9s][^0-9y][^0-9s],[^1-9][^0-9][^0-9][^0-9]*}/** w,
  deny @{PROC}/sys/[^k]** w,  # deny /proc/sys except /proc/sys/k* (effectively /proc/sys/kernel)
  deny @{PROC}/sys/kernel/{?,??,[^s][^h][^m]**} w,  # deny everything except shm* in /proc/sys/kernel/
  deny @{PROC}/sysrq-trigger rwklx,
  deny @{PROC}/mem rwklx,
  deny @{PROC}/kmem rwklx,
  deny @{PROC}/kcore rwklx,

  deny mount,

  deny /sys/[^f]*/** wklx,
  deny /sys/f[^s]*/** wklx,
  deny /sys/fs/[^c]*/** wklx,
  deny /sys/fs/c[^g]*/** wklx,
  deny /sys/fs/cg[^r]*/** wklx,
  deny /sys/firmware/** rwklx,
  deny /sys/kernel/security/** rwklx,
}



root@master:~/cks/apparmor# apparmor_parser /etc/apparmor.d/docker-nginx 
root@master:~/cks/apparmor# aa-status 
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


```
```c
root@master:~/cks/apparmor# docker run nginx
Status: Downloaded newer image for nginx:latest
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
^C

root@master:~/cks/apparmor# docker run --security-opt apparmor=docker-default nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
^C

root@master:~/cks/apparmor# docker run --security-opt apparmor=docker-nginx nginx
/docker-entrypoint.sh: 13: /docker-entrypoint.sh: cannot create /dev/null: Permission denied
/docker-entrypoint.sh: No files found in /docker-entrypoint.d/, skipping configuration
^C


root@master:~/cks/apparmor# docker run --security-opt apparmor=docker-nginx -d  nginx
f608a4a126e2e2b145dcf094b41c29bea1f7b8beeb38871178e0ea0ae8eab061
root@master:~/cks/apparmor# docker exec -ti f608a4a126e2e2b145dcf094b41c29bea1f7b8beeb38871178e0ea0ae8eab061 bash
root@f608a4a126e2:/# touch /root/test
touch: cannot touch '/root/test': Permission denied
root@f608a4a126e2:/# sh
bash: /bin/sh: Permission denied
root@f608a4a126e2:/# touch /test
root@f608a4a126e2:/# exit
exit



root@master:~/cks/apparmor# docker run --security-opt apparmor=docker-default -d  nginx
3f067ecff95e3ac8a70995a8bb23c6c58feba96d4450fd1bbb59f2cd2d142ec2
root@master:~/cks/apparmor# docker exec -ti 3f067ecff95e3ac8a70995a8bb23c6c58feba96d4450fd1bbb59f2cd2d142ec2 bash
root@3f067ecff95e:/# sh
# touch /root/test

```

## 6. Practice - AppArmor for Kubernetes Nginx

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c82a89b86d8224a120550223eb3cf1f4.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0996b61379e955cee023d36df4de58f3.png)
 
[**AppArmor Pod annotation**](https://v1-18.docs.kubernetes.io/zh/docs/tutorials/clusters/apparmor/#pod-%E6%B3%A8%E9%87%8A)


```bash
root@master:~/cks/apparmor# scp /etc/apparmor.d/docker-nginx root@192.168.211.41:/etc/apparmor.d/                                                                                100% 1644     1.6KB/s   00:00    
           
root@master:~/cks/apparmor# scp /etc/apparmor.d/docker-nginx root@192.168.211.42:/etc/apparmor.d/                                                                                100% 1644     1.6KB/s   00:00    

root@node1:/etc/apparmor.d# apparmor_parser /etc/apparmor.d/docker-nginx 
root@node1:/etc/apparmor.d# aa-status 
apparmor module is loaded.
7 profiles are loaded.
7 profiles are in enforce mode.
   /sbin/dhclient
   /usr/lib/NetworkManager/nm-dhcp-client.action
   /usr/lib/NetworkManager/nm-dhcp-helper
   /usr/lib/connman/scripts/dhclient-script
   /usr/sbin/tcpdump
   docker-default
   docker-nginx



root@master:~/cks/apparmor# k run secure --image=nginx -oyaml --dry-run=client > pod.yaml
root@master:~/cks/apparmor# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  annotations:    #添加此行
    container.apparmor.security.beta.kubernetes.io/secure: localhost/hello  #添加此行
  labels:
    run: secure
  name: secure
spec:
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

root@master:~/cks/apparmor# k create  -f pod.yaml 
pod/secure created

root@master:~/cks/apparmor# k get pods secure
NAME     READY   STATUS    RESTARTS   AGE
secure   0/1     Blocked   0          6s



root@master:~/cks/apparmor# k describe pod secure
Name:         secure
Namespace:    default
Priority:     0
Node:         node2/192.168.211.42
Start Time:   Mon, 24 May 2021 23:50:37 -0700
Labels:       run=secure
Annotations:  container.apparmor.security.beta.kubernetes.io/secure: localhost/hello
Status:       Pending
Reason:       AppArmor
Message:      Cannot enforce AppArmor: profile "hello" is not loaded
IP:           
IPs:          <none>
Containers:
  secure:
    Container ID:   
    Image:          nginx
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       Blocked
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4lh26 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-4lh26:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-4lh26
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  12s   default-scheduler  Successfully assigned default/secure to node2
root@master:~/cks/apparmor# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  annotations: 
    container.apparmor.security.beta.kubernetes.io/secure: localhost/hello
  labels:
    run: secure
  name: secure
spec:
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


#修改pod.yaml  annotations
root@master:~/cks/apparmor# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  annotations: 
    container.apparmor.security.beta.kubernetes.io/secure: localhost/docker-nginx  #修改此行
  labels:
    run: secure
  name: secure
spec:
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/apparmor# k -f pod.yaml delete --force --grace-period 0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "secure" force deleted
root@master:~/cks/apparmor# k create -f pod.yaml 
pod/secure created


root@master:~/cks/apparmor# k get pod secure
NAME     READY   STATUS    RESTARTS   AGE
secure   1/1     Running   0          10s

```







