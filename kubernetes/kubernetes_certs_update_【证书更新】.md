![](https://img-blog.csdnimg.cn/926789aec5eb4217aa03394d44ff765a.png)





## 1. 预备条件
## 1.1 kubelet是否支持证书自动轮换
 查看 kubelet是否支持证书自动轮换，默认轮换的证书位于目录 `/var/lib/kubelet/pki`

```bash
[root@kube-master01 ~]# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Tue 2023-07-25 14:57:05 CST; 18min ago
     Docs: https://kubernetes.io/docs/
 Main PID: 2983 (kubelet)
    Tasks: 13 (limit: 49016)
   Memory: 55.3M
   CGroup: /system.slice/kubelet.service
           └─2983 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/

[root@kube-master01 ~]# cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf 
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS

 [root@kube-master01 ~]# cat /var/lib/kubelet/config.yaml |grep rotate
rotateCertificates: true

```
`rotateCertificates`为`true`，代表支持自动轮询替换kubelet证书。

## 1.2 查看集群指定证书位置
```bash

[root@kube-master01 ~]# kubeadm config print init-defaults
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.k8s.io
kind: ClusterConfiguration
kubernetesVersion: 1.27.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
[root@kube-master01 ~]# kubeadm config print join-defaults
apiVersion: kubeadm.k8s.io/v1beta3
caCertPath: /etc/kubernetes/pki/ca.crt
discovery:
  bootstrapToken:
    apiServerEndpoint: kube-apiserver:6443
    token: abcdef.0123456789abcdef
    unsafeSkipCAVerification: true
  timeout: 5m0s
  tlsBootstrapToken: abcdef.0123456789abcdef
kind: JoinConfiguration
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: kube-master01
  taints: null

```

## 1.3 查看证书是否过期
> 注意：如果是比较低的kubernetes版本执行：`kubeadm alpha certs check-expiration`，比较新的kubernetes版本：`kubeadm  certs check-expiration`

```bash
[root@kube-master01 ~]# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jul 19, 2024 04:05 UTC   359d            ca                      no      
apiserver                  Jul 19, 2024 04:04 UTC   359d            ca                      no      
apiserver-etcd-client      Jul 19, 2024 04:04 UTC   359d            etcd-ca                 no      
apiserver-kubelet-client   Jul 19, 2024 04:04 UTC   359d            ca                      no      
controller-manager.conf    Jul 19, 2024 04:05 UTC   359d            ca                      no      
etcd-healthcheck-client    Jul 17, 2024 09:08 UTC   358d            etcd-ca                 no      
etcd-peer                  Jul 17, 2024 09:08 UTC   358d            etcd-ca                 no      
etcd-server                Jul 17, 2024 09:08 UTC   358d            etcd-ca                 no      
front-proxy-client         Jul 19, 2024 04:04 UTC   359d            front-proxy-ca          no      
scheduler.conf             Jul 19, 2024 04:05 UTC   359d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Jul 15, 2033 09:08 UTC   9y              no      
etcd-ca                 Jul 15, 2033 09:08 UTC   9y              no      
front-proxy-ca          Jul 15, 2033 09:08 UTC   9y      

```



## 2. 证书过期临近更新操作


### 2.1 备份证书

```bash
mkdir /tmp/certs-old
cp -rp /etc/kubernetes /tmp/certs-old
```
查看

```bash
[root@kube-master01 ~]# ls -l certs-old/
total 0
drwxr-xr-x 5 root root 136 Jul 20 12:04 kubernetes
[root@kube-master01 ~]# ls -l certs-old/.kube/
total 8
drwxr-x--- 4 root root   35 Jul 18 17:11 cache
-rw------- 1 root root 5641 Jul 18 17:10 config
[root@kube-master01 ~]# ls -l certs-old/kubernetes/
total 32
-rw------- 1 root root 5637 Jul 20 12:05 admin.conf
-rw------- 1 root root 5669 Jul 20 12:05 controller-manager.conf
-rw------- 1 root root 1989 Jul 18 17:09 kubelet.conf
drwxr-xr-x 2 root root  113 Jul 20 02:57 manifests
drwxr-xr-x 3 root root 4096 Jul 18 17:08 pki
-rw------- 1 root root 5621 Jul 20 12:05 scheduler.conf
drwx------ 5 root root  145 Jul 20 12:05 tmp
[root@kube-master01 ~]# ls -l certs-old/kubernetes/pki/
total 56
-rw-r--r-- 1 root root 1294 Jul 20 12:04 apiserver.crt
-rw-r--r-- 1 root root 1155 Jul 20 12:04 apiserver-etcd-client.crt
-rw------- 1 root root 1679 Jul 20 12:04 apiserver-etcd-client.key
-rw------- 1 root root 1679 Jul 20 12:04 apiserver.key
-rw-r--r-- 1 root root 1164 Jul 20 12:04 apiserver-kubelet-client.crt
-rw------- 1 root root 1675 Jul 20 12:04 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1099 Jul 18 17:08 ca.crt
-rw------- 1 root root 1679 Jul 18 17:08 ca.key
drwxr-xr-x 2 root root  162 Jul 18 17:08 etcd
-rw-r--r-- 1 root root 1115 Jul 18 17:08 front-proxy-ca.crt
-rw------- 1 root root 1679 Jul 18 17:08 front-proxy-ca.key
-rw-r--r-- 1 root root 1119 Jul 20 12:04 front-proxy-client.crt
-rw------- 1 root root 1675 Jul 20 12:04 front-proxy-client.key
-rw------- 1 root root 1679 Jul 18 17:08 sa.key
-rw------- 1 root root  451 Jul 18 17:08 sa.pub

$ ls certs-old/kubernetes/pki/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

```



### 2.2 更新证书

```bash
[root@kube-master01 ~]# kubeadm  certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.

[root@kube-master01 ~]#kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jul 24, 2024 06:52 UTC   364d            ca                      no      
apiserver                  Jul 24, 2024 06:52 UTC   364d            ca                      no      
apiserver-etcd-client      Jul 24, 2024 06:52 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Jul 24, 2024 06:52 UTC   364d            ca                      no      
controller-manager.conf    Jul 24, 2024 06:52 UTC   364d            ca                      no      
etcd-healthcheck-client    Jul 24, 2024 06:52 UTC   364d            etcd-ca                 no      
etcd-peer                  Jul 24, 2024 06:52 UTC   364d            etcd-ca                 no      
etcd-server                Jul 24, 2024 06:52 UTC   364d            etcd-ca                 no      
front-proxy-client         Jul 24, 2024 06:52 UTC   364d            front-proxy-ca          no      
scheduler.conf             Jul 24, 2024 06:52 UTC   364d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Jul 15, 2033 09:08 UTC   9y              no      
etcd-ca                 Jul 15, 2033 09:08 UTC   9y              no      
front-proxy-ca          Jul 15, 2033 09:08 UTC   9y              no      

[root@kube-master01 ~]# ls -l /etc/kubernetes/
total 32
-rw------- 1 root root 5641 Jul 25 14:52 admin.conf
-rw------- 1 root root 5673 Jul 25 14:52 controller-manager.conf
-rw------- 1 root root 1989 Jul 18 17:09 kubelet.conf
drwxr-xr-x 2 root root  113 Jul 20 02:57 manifests
drwxr-xr-x 3 root root 4096 Jul 18 17:08 pki
-rw------- 1 root root 5617 Jul 25 14:52 scheduler.conf
drwx------ 5 root root  145 Jul 20 12:05 tmp
[root@kube-master01 ~]# ls -l /etc/kubernetes/pki/
total 56
-rw-r--r-- 1 root root 1294 Jul 25 14:52 apiserver.crt
-rw-r--r-- 1 root root 1155 Jul 25 14:52 apiserver-etcd-client.crt
-rw------- 1 root root 1675 Jul 25 14:52 apiserver-etcd-client.key
-rw------- 1 root root 1679 Jul 25 14:52 apiserver.key
-rw-r--r-- 1 root root 1164 Jul 25 14:52 apiserver-kubelet-client.crt
-rw------- 1 root root 1679 Jul 25 14:52 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1099 Jul 18 17:08 ca.crt
-rw------- 1 root root 1679 Jul 18 17:08 ca.key
drwxr-xr-x 2 root root  162 Jul 18 17:08 etcd
-rw-r--r-- 1 root root 1115 Jul 18 17:08 front-proxy-ca.crt
-rw------- 1 root root 1679 Jul 18 17:08 front-proxy-ca.key
-rw-r--r-- 1 root root 1119 Jul 25 14:52 front-proxy-client.crt
-rw------- 1 root root 1679 Jul 25 14:52 front-proxy-client.key
-rw------- 1 root root 1679 Jul 18 17:08 sa.key
-rw------- 1 root root  451 Jul 18 17:08 sa.pub


```
### 2.3 重启平台组件

```bash
[root@kube-master01 manifests]# mv /etc/kubernetes/manifests/* /root/certs-old/etc-kubernetes-manifests/
[root@kube-master01 manifests]# crictl ps 
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD
5ede0cf8bb2d2       ead0a4a53df89       2 hours ago         Running             coredns             2                   f5f9fc235b4f7       coredns-5d78c9869d-l6qsw
64591e9c84957       ead0a4a53df89       2 hours ago         Running             coredns             2                   f7fd104b43a70       coredns-5d78c9869d-d6tjw
ae281e1b09574       6848d7eda0341       2 hours ago         Running             kube-proxy          2                   ee26bf2b947f1       kube-proxy-862db

#等待20秒
[root@kube-master01 manifests]# mv  /root/certs-old/etc-kubernetes-manifests/* .
[root@kube-master01 manifests]# crictl ps 
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
c79467f61b16f       e7972205b6614       1 second ago        Running             kube-apiserver            0                   054aa5b7aa334       kube-apiserver-kube-master01
1575c51f03ec5       f466468864b7a       1 second ago        Running             kube-controller-manager   0                   dd7da3afef78a       kube-controller-manager-kube-master01
e900e9e36238d       98ef2570f3cde       1 second ago        Running             kube-scheduler            0                   28e22a70ffe19       kube-scheduler-kube-master01
9102a026ffc03       86b6af7dd652c       1 second ago        Running             etcd                      0                   cf60901f178f9       etcd-kube-master01
5ede0cf8bb2d2       ead0a4a53df89       2 hours ago         Running             coredns                   2                   f5f9fc235b4f7       coredns-5d78c9869d-l6qsw
64591e9c84957       ead0a4a53df89       2 hours ago         Running             coredns                   2                   f7fd104b43a70       coredns-5d78c9869d-d6tjw
ae281e1b09574       6848d7eda0341       2 hours ago         Running             kube-proxy                2                   ee26bf2b947f1       kube-proxy-862db
[root@kube-master01 manifests]# kubectl get pod -A
NAMESPACE     NAME                                    READY   STATUS    RESTARTS       AGE
kube-system   coredns-5d78c9869d-d6tjw                1/1     Running   2 (140m ago)   5d5h
kube-system   coredns-5d78c9869d-l6qsw                1/1     Running   2 (140m ago)   5d5h
kube-system   etcd-kube-master01                      1/1     Running   0              127m
kube-system   kube-apiserver-kube-master01            1/1     Running   0              126m
kube-system   kube-controller-manager-kube-master01   1/1     Running   0              20m
kube-system   kube-proxy-862db                        1/1     Running   2 (140m ago)   5d4h
kube-system   kube-proxy-gnpjp                        1/1     Running   1 (5h5m ago)   5d4h
kube-system   kube-proxy-k7vpf                        1/1     Running   1 (5h5m ago)   5d4h
kube-system   kube-scheduler-kube-master01            1/1     Running   0              5d4h
```


## 3. 证书过期更新操作

### 3.1 备份证书

```bash
mkdir certs-old
cp -rp /etc/kubernetes certs-old
```
查看

```bash
[root@kube-master01 ~]# ls -l certs-old/
total 0
drwxr-xr-x 5 root root 136 Jul 20 12:04 kubernetes
[root@kube-master01 ~]# ls -l certs-old/.kube/
total 8
drwxr-x--- 4 root root   35 Jul 18 17:11 cache
-rw------- 1 root root 5641 Jul 18 17:10 config
[root@kube-master01 ~]# ls -l certs-old/kubernetes/
total 32
-rw------- 1 root root 5637 Jul 20 12:05 admin.conf
-rw------- 1 root root 5669 Jul 20 12:05 controller-manager.conf
-rw------- 1 root root 1989 Jul 18 17:09 kubelet.conf
drwxr-xr-x 2 root root  113 Jul 20 02:57 manifests
drwxr-xr-x 3 root root 4096 Jul 18 17:08 pki
-rw------- 1 root root 5621 Jul 20 12:05 scheduler.conf
drwx------ 5 root root  145 Jul 20 12:05 tmp
[root@kube-master01 ~]# ls -l certs-old/kubernetes/pki/
total 56
-rw-r--r-- 1 root root 1294 Jul 20 12:04 apiserver.crt
-rw-r--r-- 1 root root 1155 Jul 20 12:04 apiserver-etcd-client.crt
-rw------- 1 root root 1679 Jul 20 12:04 apiserver-etcd-client.key
-rw------- 1 root root 1679 Jul 20 12:04 apiserver.key
-rw-r--r-- 1 root root 1164 Jul 20 12:04 apiserver-kubelet-client.crt
-rw------- 1 root root 1675 Jul 20 12:04 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1099 Jul 18 17:08 ca.crt
-rw------- 1 root root 1679 Jul 18 17:08 ca.key
drwxr-xr-x 2 root root  162 Jul 18 17:08 etcd
-rw-r--r-- 1 root root 1115 Jul 18 17:08 front-proxy-ca.crt
-rw------- 1 root root 1679 Jul 18 17:08 front-proxy-ca.key
-rw-r--r-- 1 root root 1119 Jul 20 12:04 front-proxy-client.crt
-rw------- 1 root root 1675 Jul 20 12:04 front-proxy-client.key
-rw------- 1 root root 1679 Jul 18 17:08 sa.key
-rw------- 1 root root  451 Jul 18 17:08 sa.pub

$ ls certs-old/kubernetes/pki/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml

```



### 3.2 更新证书

```bash
[root@kube-master01 ~]# kubeadm  certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed

Done renewing certificates. You must restart the kube-apiserver, kube-controller-manager, kube-scheduler and etcd, so that they can use the new certificates.

#查看日期是否更新
[root@kube-master01 ~]#kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jul 24, 2024 06:52 UTC   364d            ca                      no      
apiserver                  Jul 24, 2024 06:52 UTC   364d            ca                      no      
apiserver-etcd-client      Jul 24, 2024 06:52 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Jul 24, 2024 06:52 UTC   364d            ca                      no      
controller-manager.conf    Jul 24, 2024 06:52 UTC   364d            ca                      no      
etcd-healthcheck-client    Jul 24, 2024 06:52 UTC   364d            etcd-ca                 no      
etcd-peer                  Jul 24, 2024 06:52 UTC   364d            etcd-ca                 no      
etcd-server                Jul 24, 2024 06:52 UTC   364d            etcd-ca                 no      
front-proxy-client         Jul 24, 2024 06:52 UTC   364d            front-proxy-ca          no      
scheduler.conf             Jul 24, 2024 06:52 UTC   364d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Jul 15, 2033 09:08 UTC   9y              no      
etcd-ca                 Jul 15, 2033 09:08 UTC   9y              no      
front-proxy-ca          Jul 15, 2033 09:08 UTC   9y              no      

[root@kube-master01 ~]# ls -l /etc/kubernetes/
total 32
-rw------- 1 root root 5641 Jul 25 14:52 admin.conf
-rw------- 1 root root 5673 Jul 25 14:52 controller-manager.conf
-rw------- 1 root root 1989 Jul 18 17:09 kubelet.conf
drwxr-xr-x 2 root root  113 Jul 20 02:57 manifests
drwxr-xr-x 3 root root 4096 Jul 18 17:08 pki
-rw------- 1 root root 5617 Jul 25 14:52 scheduler.conf
drwx------ 5 root root  145 Jul 20 12:05 tmp
[root@kube-master01 ~]# ls -l /etc/kubernetes/pki/
total 56
-rw-r--r-- 1 root root 1294 Jul 25 14:52 apiserver.crt
-rw-r--r-- 1 root root 1155 Jul 25 14:52 apiserver-etcd-client.crt
-rw------- 1 root root 1675 Jul 25 14:52 apiserver-etcd-client.key
-rw------- 1 root root 1679 Jul 25 14:52 apiserver.key
-rw-r--r-- 1 root root 1164 Jul 25 14:52 apiserver-kubelet-client.crt
-rw------- 1 root root 1679 Jul 25 14:52 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1099 Jul 18 17:08 ca.crt
-rw------- 1 root root 1679 Jul 18 17:08 ca.key
drwxr-xr-x 2 root root  162 Jul 18 17:08 etcd
-rw-r--r-- 1 root root 1115 Jul 18 17:08 front-proxy-ca.crt
-rw------- 1 root root 1679 Jul 18 17:08 front-proxy-ca.key
-rw-r--r-- 1 root root 1119 Jul 25 14:52 front-proxy-client.crt
-rw------- 1 root root 1679 Jul 25 14:52 front-proxy-client.key
-rw------- 1 root root 1679 Jul 18 17:08 sa.key
-rw------- 1 root root  451 Jul 18 17:08 sa.pub


```
### 3.3 重启平台组件

```bash
[root@kube-master01 manifests]# mv /etc/kubernetes/manifests/* /root/certs-old/etc-kubernetes-manifests/
[root@kube-master01 manifests]# crictl ps 
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD
5ede0cf8bb2d2       ead0a4a53df89       2 hours ago         Running             coredns             2                   f5f9fc235b4f7       coredns-5d78c9869d-l6qsw
64591e9c84957       ead0a4a53df89       2 hours ago         Running             coredns             2                   f7fd104b43a70       coredns-5d78c9869d-d6tjw
ae281e1b09574       6848d7eda0341       2 hours ago         Running             kube-proxy          2                   ee26bf2b947f1       kube-proxy-862db

#等待20秒
[root@kube-master01 manifests]# mv  /root/certs-old/etc-kubernetes-manifests/* .
[root@kube-master01 manifests]# crictl ps 
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
c79467f61b16f       e7972205b6614       1 second ago        Running             kube-apiserver            0                   054aa5b7aa334       kube-apiserver-kube-master01
1575c51f03ec5       f466468864b7a       1 second ago        Running             kube-controller-manager   0                   dd7da3afef78a       kube-controller-manager-kube-master01
e900e9e36238d       98ef2570f3cde       1 second ago        Running             kube-scheduler            0                   28e22a70ffe19       kube-scheduler-kube-master01
9102a026ffc03       86b6af7dd652c       1 second ago        Running             etcd                      0                   cf60901f178f9       etcd-kube-master01
5ede0cf8bb2d2       ead0a4a53df89       2 hours ago         Running             coredns                   2                   f5f9fc235b4f7       coredns-5d78c9869d-l6qsw
64591e9c84957       ead0a4a53df89       2 hours ago         Running             coredns                   2                   f7fd104b43a70       coredns-5d78c9869d-d6tjw
ae281e1b09574       6848d7eda0341       2 hours ago         Running             kube-proxy                2                   ee26bf2b947f1       kube-proxy-862db
[root@kube-master01 manifests]# kubectl get pod -A
NAMESPACE     NAME                                    READY   STATUS    RESTARTS       AGE
kube-system   coredns-5d78c9869d-d6tjw                1/1     Running   2 (140m ago)   5d5h
kube-system   coredns-5d78c9869d-l6qsw                1/1     Running   2 (140m ago)   5d5h
kube-system   etcd-kube-master01                      1/1     Running   0              127m
kube-system   kube-apiserver-kube-master01            1/1     Running   0              126m
kube-system   kube-controller-manager-kube-master01   1/1     Running   0              20m
kube-system   kube-proxy-862db                        1/1     Running   2 (140m ago)   5d4h
kube-system   kube-proxy-gnpjp                        1/1     Running   1 (5h5m ago)   5d4h
kube-system   kube-proxy-k7vpf                        1/1     Running   1 (5h5m ago)   5d4h
kube-system   kube-scheduler-kube-master01            1/1     Running   0              5d4h
```


### 3.4  更新 config

```bash
mkdir /tmp/kube_bak && cp -r /root/.kube /tmp/kube_bak
cp /etc/kubernetes/admin.conf /root/.kube /
```

### 3.5 更新库表



参考：
- [https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
- [https://kubernetes.io/zh-cn/docs/tasks/tls/certificate-rotation/](https://kubernetes.io/zh-cn/docs/tasks/tls/certificate-rotation/)


