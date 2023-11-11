

----
## 1. 介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516000835175.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516000819495.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516000921543.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051600102447.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516001205963.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051600122529.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. Create sidecar proxy
任务
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021051622183375.png)


```c
root@master:~/cks/mTLS# k run app --image=bash --command -oyaml --dry-run=client > app.yaml -- sh -c 'ping baidu.com'
root@master:~/cks/mTLS# cat app.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - command:
    - sh
    - -c
    - ping baidu.com
    image: bash
    name: app
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
root@master:~/cks/mTLS# k -f app.yaml  create
pod/app created

root@master:~/cks/mTLS# k get pods -w
NAME   READY   STATUS              RESTARTS   AGE
app    0/1     ContainerCreating   0          14s
app    1/1     Running             0          23s
^Croot@master:~/cks/mTLS# k logs -f app
PING baidu.com (39.156.69.79): 56 data bytes
64 bytes from 39.156.69.79: seq=0 ttl=127 time=7.785 ms
64 bytes from 39.156.69.79: seq=1 ttl=127 time=7.526 ms
64 bytes from 39.156.69.79: seq=2 ttl=127 time=8.031 ms
64 bytes from 39.156.69.79: seq=3 ttl=127 time=8.429 ms
64 bytes from 39.156.69.79: seq=4 ttl=127 time=8.007 ms
64 bytes from 39.156.69.79: seq=5 ttl=127 time=7.250 ms
64 bytes from 39.156.69.79: seq=6 ttl=127 time=8.438 ms
64 bytes from 39.156.69.79: seq=7 ttl=127 time=7.412 ms
64 bytes from 39.156.69.79: seq=8 ttl=127 time=7.328 ms
```

[set-capabilities-for-a-container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container)

```c
root@master:~/cks/securitytext# cat app.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
  - name: proxy
    image: ubuntu
    command:
    - sh
    - -c
    - 'apt-get update && apt-get install iptables -y && iptables -L && sleep 1d'
    securityContext:
      capabilities:
        add: ["NET_ADMIN"]      
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



root@master:~/cks/securitytext# kubectl -f app.yaml delete --force --grace-period=0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "app" force deleted
root@master:~/cks/securitytext# k create -f app.yaml 
pod/app created

root@master:~/cks/securitytext# k get pods
NAME   READY   STATUS    RESTARTS   AGE
app    2/2     Running   0          45s

root@master:~/cks/securitytext# k logs app -c proxy
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [109 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:3 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [21.7 kB]
Get:4 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [700 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:9 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [267 kB]
Get:10 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [817 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
root@master:~/cks/securitytext# k logs app -c proxy -f
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [109 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:3 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [21.7 kB]
Get:4 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [700 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease [101 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:9 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [267 kB]
Get:10 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [817 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [969 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [1238 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [299 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [29.7 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [4305 B]
Fetched 17.8 MB in 1min 27s (205 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  libip4tc2 libip6tc2 libmnl0 libnetfilter-conntrack3 libnfnetlink0 libnftnl11
  libxtables12 netbase
Suggested packages:
  firewalld kmod nftables
The following NEW packages will be installed:
  iptables libip4tc2 libip6tc2 libmnl0 libnetfilter-conntrack3 libnfnetlink0
  libnftnl11 libxtables12 netbase
0 upgraded, 9 newly installed, 0 to remove and 0 not upgraded.
Need to get 595 kB of archives.
After this operation, 3490 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal/main amd64 libip4tc2 amd64 1.8.4-3ubuntu2 [18.8 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal/main amd64 libmnl0 amd64 1.0.4-2 [12.3 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal/main amd64 libxtables12 amd64 1.8.4-3ubuntu2 [28.4 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal/main amd64 netbase all 6.1 [13.1 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal/main amd64 libip6tc2 amd64 1.8.4-3ubuntu2 [19.2 kB]
Get:6 http://archive.ubuntu.com/ubuntu focal/main amd64 libnfnetlink0 amd64 1.0.1-3build1 [13.8 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal/main amd64 libnetfilter-conntrack3 amd64 1.0.7-2 [41.4 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal/main amd64 libnftnl11 amd64 1.1.5-1 [57.8 kB]
Get:9 http://archive.ubuntu.com/ubuntu focal/main amd64 iptables amd64 1.8.4-3ubuntu2 [390 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 595 kB in 3s (182 kB/s)
Selecting previously unselected package libip4tc2:amd64.
(Reading database ... 4121 files and directories currently installed.)
Preparing to unpack .../0-libip4tc2_1.8.4-3ubuntu2_amd64.deb ...
Unpacking libip4tc2:amd64 (1.8.4-3ubuntu2) ...
Selecting previously unselected package libmnl0:amd64.
Preparing to unpack .../1-libmnl0_1.0.4-2_amd64.deb ...
Unpacking libmnl0:amd64 (1.0.4-2) ...
Selecting previously unselected package libxtables12:amd64.
Preparing to unpack .../2-libxtables12_1.8.4-3ubuntu2_amd64.deb ...
Unpacking libxtables12:amd64 (1.8.4-3ubuntu2) ...
Selecting previously unselected package netbase.
Preparing to unpack .../3-netbase_6.1_all.deb ...
Unpacking netbase (6.1) ...
Selecting previously unselected package libip6tc2:amd64.
Preparing to unpack .../4-libip6tc2_1.8.4-3ubuntu2_amd64.deb ...
Unpacking libip6tc2:amd64 (1.8.4-3ubuntu2) ...
Selecting previously unselected package libnfnetlink0:amd64.
Preparing to unpack .../5-libnfnetlink0_1.0.1-3build1_amd64.deb ...
Unpacking libnfnetlink0:amd64 (1.0.1-3build1) ...
Selecting previously unselected package libnetfilter-conntrack3:amd64.
Preparing to unpack .../6-libnetfilter-conntrack3_1.0.7-2_amd64.deb ...
Unpacking libnetfilter-conntrack3:amd64 (1.0.7-2) ...
Selecting previously unselected package libnftnl11:amd64.
Preparing to unpack .../7-libnftnl11_1.1.5-1_amd64.deb ...
Unpacking libnftnl11:amd64 (1.1.5-1) ...
Selecting previously unselected package iptables.
Preparing to unpack .../8-iptables_1.8.4-3ubuntu2_amd64.deb ...
Unpacking iptables (1.8.4-3ubuntu2) ...
Setting up libip4tc2:amd64 (1.8.4-3ubuntu2) ...
Setting up libip6tc2:amd64 (1.8.4-3ubuntu2) ...
Setting up libmnl0:amd64 (1.0.4-2) ...
Setting up libxtables12:amd64 (1.8.4-3ubuntu2) ...
Setting up libnfnetlink0:amd64 (1.0.1-3build1) ...
Setting up netbase (6.1) ...
Setting up libnftnl11:amd64 (1.1.5-1) ...
Setting up libnetfilter-conntrack3:amd64 (1.0.7-2) ...
Setting up iptables (1.8.4-3ubuntu2) ...
update-alternatives: using /usr/sbin/iptables-legacy to provide /usr/sbin/iptables (iptables) in auto mode
update-alternatives: using /usr/sbin/ip6tables-legacy to provide /usr/sbin/ip6tables (ip6tables) in auto mode
update-alternatives: using /usr/sbin/arptables-nft to provide /usr/sbin/arptables (arptables) in auto mode
update-alternatives: using /usr/sbin/ebtables-nft to provide /usr/sbin/ebtables (ebtables) in auto mode
Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination     
```


