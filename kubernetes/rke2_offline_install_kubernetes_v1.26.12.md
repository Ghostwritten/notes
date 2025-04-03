![](https://img-blog.csdnimg.cn/direct/6b09495582264c4dabbfb20335317f1b.png)




## 1. 准备

7 台主机

|主机名  |  ip|cpu|内存|disk |os|角色|user|密码|
|--|--| --|--|--|--|--|--|--|
|  kube-master01| 192.168.10.131 |8|16 |50|redhat 8.8|master|root|root|
|  kube-master02| 192.168.10.132 |8|16 |50|redhat 8.8|master|root|root|
|  kube-master03| 192.168.10.133 |8|16 |50|redhat 8.8|master|root|root|
|  kube-node01| 192.168.10.134 |8|16 |50|redhat 8.8|master|root|root|
|  kube-node02| 192.168.10.135 |8|16 |50|redhat 8.8|master|root|root|
|  kube-node03 | 192.168.10.136 | 8|16 |50|redhat 8.8|master|root|root|
|  bastion01 | 192.168.10.139 | 8|16 |50|redhat 8.8|master|root|root|


## 2. 安装 ansible
bastion01

```bash
yum  -y install epel-release
yum -y install ansible
vim /etc/ansible/hosts
[all]
kube-master01 ansible_host=10.80.10.131
kube-master02 ansible_host=10.80.10.132
kube-master03 ansible_host=10.80.10.133
kube-node01 ansible_host=10.80.10.134
kube-node02 ansible_host=10.80.10.135
kube-node03 ansible_host=10.80.10.136

[bastion]
bastion01 ansible_host=10.80.10.139 ansible_user=root

[kube_control_plane]
kube-master01
kube-master02
kube-master03

[etcd]
kube-master01
kube-master02
kube-master03

[kube_node]
kube-node01
kube-node02
kube-node03
```
## 3. 基础配置
### 3.1 配置 hosts

```bash
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.80.10.131 kube-master01
10.80.10.132 kube-master02
10.80.10.133 kube-master03
10.80.10.134 kube-node01
10.80.10.135 kube-node02
10.80.10.136 kube-node03

$ ansible all -m copy -a "src=/etc/hosts dest=/etc/hosts"
```

### 3.2 安装软件包

```bash
yum -y install lrzsz vim gcc glibc openssl openssl-devel net-tools http-tools wget curl  yum-utils telnet
```

### 3.3  内核参数

```bash
$ vim  sysctl.sh 
echo "
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward=1
net.ipv4.conf.all.forwarding=1
net.ipv4.neigh.default.gc_thresh1=4096
net.ipv4.neigh.default.gc_thresh2=6144
net.ipv4.neigh.default.gc_thresh3=8192
net.ipv4.neigh.default.gc_interval=60
net.ipv4.neigh.default.gc_stale_time=120

# 参考 https://github.com/prometheus/node_exporter#disabled-by-default
kernel.perf_event_paranoid=-1

#sysctls for k8s node config
net.ipv4.tcp_slow_start_after_idle=0
net.core.rmem_max=16777216
fs.inotify.max_user_watches=524288
kernel.softlockup_all_cpu_backtrace=1

kernel.softlockup_panic=0

kernel.watchdog_thresh=30
fs.file-max=2097152
fs.inotify.max_user_instances=8192
fs.inotify.max_queued_events=16384
vm.max_map_count=262144
net.core.netdev_max_backlog=16384
net.ipv4.tcp_wmem=4096 12582912 16777216
net.core.wmem_max=16777216
net.core.somaxconn=32768
net.ipv4.ip_forward=1
net.ipv4.tcp_max_syn_backlog=8096
net.ipv4.tcp_rmem=4096 12582912 16777216

net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1

kernel.yama.ptrace_scope=0
vm.swappiness=0

# 可以控制core文件的文件名中是否添加pid作为扩展。
kernel.core_uses_pid=1

# Do not accept source routing
net.ipv4.conf.default.accept_source_route=0
net.ipv4.conf.all.accept_source_route=0

# Promote secondary addresses when the primary address is removed
net.ipv4.conf.default.promote_secondaries=1
net.ipv4.conf.all.promote_secondaries=1

# Enable hard and soft link protection
fs.protected_hardlinks=1
fs.protected_symlinks=1

# 源路由验证
# see details in https://help.aliyun.com/knowledge_detail/39428.html
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_announce=2
net.ipv4.conf.all.arp_announce=2

# see details in https://help.aliyun.com/knowledge_detail/41334.html
net.ipv4.tcp_max_tw_buckets=5000
net.ipv4.tcp_syncookies=1
net.ipv4.tcp_fin_timeout=30
net.ipv4.tcp_synack_retries=2
kernel.sysrq=1

" >> /etc/sysctl.conf
modprobe br_netfilter
sysctl -p

$ ansible all -m script -a "sysctl.sh"
```

### 3.4 连接数限制

```bash
ansible all -m lineinfile -a "path=/etc/security/limits.conf line='* soft nofile 655360\n* hard nofile 131072\n* soft nproc 655350\n* hard nproc 655350\n* soft memlock unlimited\n* hard memlock unlimited'" -b

```

### 3.5 关闭swap 、selinux、防火墙

```bash
ansible all -i hosts -s -m systemd -a "name=firewalld state=stopped enabled=no"
ansible all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'" -b
ansible all -m shell -a "getenforce 0"
ansible all -m shell -a "sed -i '/.*swap.*/s/^/#/' /etc/fstab" -b
ansible all -m shell -a " swapoff -a && sysctl -w vm.swappiness=0"

```

### 3.6 时间同步

定义自己的时间服务器
```bash
yum -y install chrony
mv /etc/chrony.conf /etc/chrony.conf_bak
cat > /etc/chrony.conf <<EOF
pool ntp.aliyun.com iburst
pool ntp1.aliyun.com iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony
EOF

systemctl enable chronyd
systemctl restart chronyd
timedatectl status
chronyc sources -v

```

## 4. RKE2 安装
设置一个 HA 集群需要以下步骤：

配置一个固定的注册地址
启动第一个 server 节点
加入其他 server 节点
加入 agent 节点
参考：https://docs.rancher.cn/docs/rke2/install/ha/_index/

注意：由于主机有限，我们就把第一个启动的节点设置为注册地址，下面只进行2、3步骤。

### 4.1 下载安装
rke2版本信息：https://github.com/rancher/rke2/releases

```bash
sudo mkdir /root/rke2-artifacts && cd /root/rke2-artifacts/
wget https://github.com/rancher/rke2/releases/download/v1.26.12%2Brke2r1/rke2-images.linux-amd64.tar.zst
wget https://github.com/rancher/rke2/releases/download/v1.26.12%2Brke2r1/rke2.linux-amd64.tar.gz
wget https://github.com/rancher/rke2/releases/download/v1.26.12%2Brke2r1/sha256sum-amd64.txt
curl -sfL https://get.rke2.io --output install.sh
```

安装

```bash
INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh
```
输出：

```bash
$ INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh
[INFO]  staging local checksums from /root/rke2-artifacts/sha256sum-amd64.txt
[INFO]  staging zst airgap image tarball from /root/rke2-artifacts/rke2-images.linux-amd64.tar.zst
[INFO]  staging tarball from /root/rke2-artifacts/rke2.linux-amd64.tar.gz
[INFO]  verifying airgap tarball
[INFO]  installing airgap tarball to /var/lib/rancher/rke2/agent/images
[INFO]  verifying tarball
[INFO]  unpacking tarball file to /usr/local
```
启用 rke2-server 服务

```bash
systemctl enable rke2-server.service && systemctl start rke2-server.service
```

 如有需要，可以查看日志

```bash
journalctl -u rke2-server -f
```
启动的过程可能需要3-8分钟，请耐心等候！

启动完成之后，你通过以下命令设置 kubectl 进行交互

设置环境变量
```bash
cat >>/root/.bashrc<< EOF
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/var/lib/rancher/rke2/bin
EOF
source /root/.bashrc
```
查看集群

```bash
$ kubectl get node
NAME            STATUS   ROLES                       AGE     VERSION
rke2-master01   Ready    control-plane,etcd,master   3m30s   v1.26.12+rke2r1

$ kubectl get po -A
NAMESPACE     NAME                                                    READY   STATUS      RESTARTS   AGE
kube-system   cloud-controller-manager-rke2-master01                  1/1     Running     0          29m
kube-system   etcd-rke2-master01                                      1/1     Running     0          29m
kube-system   helm-install-rke2-canal-6v6qr                           0/1     Completed   0          29m
kube-system   helm-install-rke2-coredns-b5ttn                         0/1     Completed   0          29m
kube-system   helm-install-rke2-ingress-nginx-45cqw                   0/1     Completed   0          29m
kube-system   helm-install-rke2-metrics-server-mq6qh                  0/1     Completed   0          29m
kube-system   helm-install-rke2-snapshot-controller-crd-jn4zf         0/1     Completed   0          29m
kube-system   helm-install-rke2-snapshot-controller-zt8f5             0/1     Completed   2          29m
kube-system   helm-install-rke2-snapshot-validation-webhook-kgjbt     0/1     Completed   0          29m
kube-system   kube-apiserver-rke2-master01                            1/1     Running     0          29m
kube-system   kube-controller-manager-rke2-master01                   1/1     Running     0          29m
kube-system   kube-proxy-rke2-master01                                1/1     Running     0          29m
kube-system   kube-scheduler-rke2-master01                            1/1     Running     0          29m
kube-system   rke2-canal-ssvcb                                        2/2     Running     0          29m
kube-system   rke2-coredns-rke2-coredns-565dfc7d75-6dbr9              1/1     Running     0          29m
kube-system   rke2-coredns-rke2-coredns-autoscaler-6c48c95bf9-lb2xt   1/1     Running     0          29m
kube-system   rke2-ingress-nginx-controller-8lp6v                     1/1     Running     0          28m
kube-system   rke2-metrics-server-c9c78bd66-szclt                     1/1     Running     0          28m
kube-system   rke2-snapshot-controller-6f7bbb497d-b426h               1/1     Running     0          28m
kube-system   rke2-snapshot-validation-webhook-65b5675d5c-2b98t       1/1     Running     0          28m
```
查看镜像

```bash
crictl --runtime-endpoint  /run/k3s/containerd/containerd.sock images
I0105 03:04:04.797054   38955 util_unix.go:103] "Using this endpoint is deprecated, please consider using full URL format" endpoint="/run/k3s/containerd/containerd.sock" URL="unix:///run/k3s/containerd/containerd.sock"
IMAGE                                                                TAG                                        IMAGE ID            SIZE
docker.io/rancher/hardened-calico                                    v3.26.3-build20231109                      116d7534875a5       550MB
docker.io/rancher/hardened-cluster-autoscaler                        v1.8.6-build20230609                       4b341204b793f       158MB
docker.io/rancher/hardened-coredns                                   v1.10.1-build20230607                      e9693e4a055c6       178MB
docker.io/rancher/hardened-dns-node-cache                            1.22.20-build20230607                      b8c68fd62f6ec       185MB
docker.io/rancher/hardened-etcd                                      v3.5.9-k3s1-build20230802                  c6b7a4f2f79b2       168MB
docker.io/rancher/hardened-flannel                                   v0.23.0-build20231109                      c776826db2fda       222MB
docker.io/rancher/hardened-k8s-metrics-server                        v0.6.3-build20230607                       c32586d7f004e       172MB
docker.io/rancher/hardened-kubernetes                                v1.26.12-rke2r1-build20231220              f3833faba37f6       741MB
docker.io/rancher/klipper-helm                                       v0.8.2-build20230815                       5f89cb8137ccb       256MB
docker.io/rancher/klipper-lb                                         v0.4.4                                     af74bd845c4a8       12.5MB
docker.io/rancher/mirrored-ingress-nginx-kube-webhook-certgen        v20230312-helm-chart-4.5.2-28-g66a760794   5a86b03a88d23       48.5MB
docker.io/rancher/mirrored-sig-storage-snapshot-controller           v6.2.1                                     1ef6c138bd5f2       58.4MB
docker.io/rancher/mirrored-sig-storage-snapshot-validation-webhook   v6.2.2                                     ff52c2bcf9f88       49MB
docker.io/rancher/nginx-ingress-controller                           nginx-1.9.3-hardened1                      bfdece8fa3f14       800MB
docker.io/rancher/pause                                              3.6                                        6270bb605e12e       686kB
docker.io/rancher/rke2-cloud-provider                                v1.26.3-build20230406                      f906d1e7a5774       175MB
docker.io/rancher/rke2-runtime                                       v1.26.12-rke2r1                            b41c0bf12eaed       348MB

#正是打包的18个镜像
$ crictl --runtime-endpoint  /run/k3s/containerd/containerd.sock images | wc -l
I0105 04:44:47.837366   27749 util_unix.go:103] "Using this endpoint is deprecated, please consider using full URL format" endpoint="/run/k3s/containerd/containerd.sock" URL="unix:///run/k3s/containerd/containerd.sock"
18

$ ctr --address /run/k3s/containerd/containerd.sock ns ls
NAME   LABELS 
k8s.io        
$ ctr --address /run/k3s/containerd/containerd.sock -n k8s.io i ls
REF                                                                                                    TYPE                                                 DIGEST                                                                  SIZE      PLATFORMS   LABELS                          
docker.io/rancher/hardened-calico:v3.26.3-build20231109                                                application/vnd.docker.distribution.manifest.v2+json sha256:a04597f6c764a8a6b6efeea49c0b07192b5592356ecd2e9df93afd1cbd5b0040 524.3 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/hardened-cluster-autoscaler:v1.8.6-build20230609                                     application/vnd.docker.distribution.manifest.v2+json sha256:4482a289e12fe12b67be83ae9bd873632cf6aa831d18a79bf9956665ac5dc67b 150.5 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/hardened-coredns:v1.10.1-build20230607                                               application/vnd.docker.distribution.manifest.v2+json sha256:ff06feb91cd772ca1d11392bfb01c4403923980d0c479ee9b0c0b9cbd6a1037e 170.2 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/hardened-dns-node-cache:1.22.20-build20230607                                        application/vnd.docker.distribution.manifest.v2+json sha256:b668f8ab563d548467d92c51686f62291c55ab2ef891dc5f0936cfdf04933374 176.3 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/hardened-etcd:v3.5.9-k3s1-build20230802                                              application/vnd.docker.distribution.manifest.v2+json sha256:c3152682e39151efb3d56be9b9cec0a4c289430755250319d0590e372c2ae833 160.1 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/hardened-flannel:v0.23.0-build20231109                                               application/vnd.docker.distribution.manifest.v2+json sha256:ace90ebb20a719162a93455fada9361ebaa3de7c74543525172184cd8552f99e 212.2 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/hardened-k8s-metrics-server:v0.6.3-build20230607                                     application/vnd.docker.distribution.manifest.v2+json sha256:a62b2b9fdffe0a503508219b0ad85ff19266038a71471e83b80860a3007fe0b9 163.7 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/hardened-kubernetes:v1.26.12-rke2r1-build20231220                                    application/vnd.docker.distribution.manifest.v2+json sha256:406825324934b223aa163329d984dc0fd7f11ed7efa93cdbb12956aa9c6f8026 706.6 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/klipper-helm:v0.8.2-build20230815                                                    application/vnd.docker.distribution.manifest.v2+json sha256:9f6b0a352533fe34763f81f014952f0595b9bd2ad531b179767c81ef77172668 244.5 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/klipper-lb:v0.4.4                                                                    application/vnd.docker.distribution.manifest.v2+json sha256:1068256da90ae89e55b6b59cfd170f56285acfd8193abcaf0aeebce100fd1d6e 11.9 MiB  linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/mirrored-ingress-nginx-kube-webhook-certgen:v20230312-helm-chart-4.5.2-28-g66a760794 application/vnd.docker.distribution.manifest.v2+json sha256:57182383859f52f92a14a8f1a52a8c83c01314c9866c2aa94f3269c34ce8043e 46.2 MiB  linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/mirrored-sig-storage-snapshot-controller:v6.2.1                                      application/vnd.docker.distribution.manifest.v2+json sha256:ef36c4cf203caac19b894e7b03534e212c675c19f5e82bbc903ccc080818c69a 55.7 MiB  linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/mirrored-sig-storage-snapshot-validation-webhook:v6.2.2                              application/vnd.docker.distribution.manifest.v2+json sha256:e5edbd113f9d9310e4001baf92b1a70db0070755da55fe31181550eb4074cadd 46.7 MiB  linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/nginx-ingress-controller:nginx-1.9.3-hardened1                                       application/vnd.docker.distribution.manifest.v2+json sha256:bfd22a6fb7a6614c2c1c6efd645af9dac02c8a2eefeed8cefce9aaaf7dffeac8 763.1 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/pause:3.6                                                                            application/vnd.docker.distribution.manifest.v2+json sha256:79b611631c0d19e9a975fb0a8511e5153789b4c26610d1842e9f735c57cc8b13 669.8 KiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/rke2-cloud-provider:v1.26.3-build20230406                                            application/vnd.docker.distribution.manifest.v2+json sha256:fb39ba6b718d9444d92598ecefb94623c3c64af50d56b76e095bb7b28ebc67d2 167.3 MiB linux/amd64 io.cri-containerd.image=managed 
docker.io/rancher/rke2-runtime:v1.26.12-rke2r1                                                         application/vnd.docker.distribution.manifest.v2+json sha256:ac979e425e203f6374f32a97453af6072afe172786cef96375cf2db72eedaa75 332.0 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:116d7534875a5767406cd0b844e8bb4c88193831c72d78ccf00abb00dc1bf652                                application/vnd.docker.distribution.manifest.v2+json sha256:a04597f6c764a8a6b6efeea49c0b07192b5592356ecd2e9df93afd1cbd5b0040 524.3 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:1ef6c138bd5f2ac45f7b4ee54db0e513efad8576909ae9829ba649fb4b067388                                application/vnd.docker.distribution.manifest.v2+json sha256:ef36c4cf203caac19b894e7b03534e212c675c19f5e82bbc903ccc080818c69a 55.7 MiB  linux/amd64 io.cri-containerd.image=managed 
sha256:4b341204b793f4135593707e7af9b74d17948ec78cf930c5555365d7ab8630e6                                application/vnd.docker.distribution.manifest.v2+json sha256:4482a289e12fe12b67be83ae9bd873632cf6aa831d18a79bf9956665ac5dc67b 150.5 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:5a86b03a88d2316e2317c2576449a95ddbd105d69b2fe7b01d667b0ebab37422                                application/vnd.docker.distribution.manifest.v2+json sha256:57182383859f52f92a14a8f1a52a8c83c01314c9866c2aa94f3269c34ce8043e 46.2 MiB  linux/amd64 io.cri-containerd.image=managed 
sha256:5f89cb8137ccbd39377d91b9d75faf4ec4ee0a2d2a3a63635535b10c69c935fa                                application/vnd.docker.distribution.manifest.v2+json sha256:9f6b0a352533fe34763f81f014952f0595b9bd2ad531b179767c81ef77172668 244.5 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:6270bb605e12e581514ada5fd5b3216f727db55dc87d5889c790e4c760683fee                                application/vnd.docker.distribution.manifest.v2+json sha256:79b611631c0d19e9a975fb0a8511e5153789b4c26610d1842e9f735c57cc8b13 669.8 KiB linux/amd64 io.cri-containerd.image=managed 
sha256:af74bd845c4a83b7e4fa48e0c5a91dcda8843f586794fbb8b7f4bb7ed9e8cc56                                application/vnd.docker.distribution.manifest.v2+json sha256:1068256da90ae89e55b6b59cfd170f56285acfd8193abcaf0aeebce100fd1d6e 11.9 MiB  linux/amd64 io.cri-containerd.image=managed 
sha256:b41c0bf12eaed3b9c891524491271f9bbc69f7d64d329c19a2fc03081e665e35                                application/vnd.docker.distribution.manifest.v2+json sha256:ac979e425e203f6374f32a97453af6072afe172786cef96375cf2db72eedaa75 332.0 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:b8c68fd62f6eca96605fb7c008ac85d6f04c03f35871e99e6d02b5aa0b7af209                                application/vnd.docker.distribution.manifest.v2+json sha256:b668f8ab563d548467d92c51686f62291c55ab2ef891dc5f0936cfdf04933374 176.3 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:bfdece8fa3f1449a2b25c12f3e375c57258a6cd4d925f7983177f5f652afc885                                application/vnd.docker.distribution.manifest.v2+json sha256:bfd22a6fb7a6614c2c1c6efd645af9dac02c8a2eefeed8cefce9aaaf7dffeac8 763.1 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:c32586d7f004ede455a89444586801f9d30669c671e48ddad7be05c54dce9d3b                                application/vnd.docker.distribution.manifest.v2+json sha256:a62b2b9fdffe0a503508219b0ad85ff19266038a71471e83b80860a3007fe0b9 163.7 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:c6b7a4f2f79b24f9310e769ce7c1e0caba47fbf2d03a2025b19bee2090dae94d                                application/vnd.docker.distribution.manifest.v2+json sha256:c3152682e39151efb3d56be9b9cec0a4c289430755250319d0590e372c2ae833 160.1 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:c776826db2fda39152c467ecee8dd0d8f0414b1443423a2c819174f5d3bef7c1                                application/vnd.docker.distribution.manifest.v2+json sha256:ace90ebb20a719162a93455fada9361ebaa3de7c74543525172184cd8552f99e 212.2 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:e9693e4a055c697c4914cd6ac0eec06f5900f4d5f1d448f52b13c467b3599462                                application/vnd.docker.distribution.manifest.v2+json sha256:ff06feb91cd772ca1d11392bfb01c4403923980d0c479ee9b0c0b9cbd6a1037e 170.2 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:f3833faba37f6afbf70b2d11bc4871936a9c6c99927b0a1c01e4702d95af75fe                                application/vnd.docker.distribution.manifest.v2+json sha256:406825324934b223aa163329d984dc0fd7f11ed7efa93cdbb12956aa9c6f8026 706.6 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:f906d1e7a5774a6e36dddaadcefa240b1813bc921b50303fd0b0874519ccf889                                application/vnd.docker.distribution.manifest.v2+json sha256:fb39ba6b718d9444d92598ecefb94623c3c64af50d56b76e095bb7b28ebc67d2 167.3 MiB linux/amd64 io.cri-containerd.image=managed 
sha256:ff52c2bcf9f8893ac479bade578b25e9f4315173bcba6f605ca94a4c7ab84235                                application/vnd.docker.distribution.manifest.v2+json sha256:e5edbd113f9d9310e4001baf92b1a70db0070755da55fe31181550eb4074cadd 46.7 MiB  linux/amd64 io.cri-containerd.image=managed 
```

### 4.2 配置其他管理节点
第一服务器节点建立秘密令牌，当连接到集群时，其他服务器或代理节点将向该秘密令牌注册。
要将自己的预共享密钥指定为令牌，请在启动时设置令牌参数。

如果您没有指定预共享密钥，RKE 2将生成一个并将其放置在`/var/lib/rancher/rke 2/server/node-token`中.
在rke2-master01 查看
```bash
$ cat /var/lib/rancher/rke2/server/node-token 
K10280f64f7fcf7d94dfa45b6867fd55ef18597e966e5b817552970a24bf15ec6d1::server:417c78df294d6fb88640ef7c9304c070
```

传递介质

```bash
$ tree 
.
├── install.sh
├── rke2-images-all.linux-amd64.txt
├── rke2-images.linux-amd64.tar.zst
├── rke2.linux-amd64.tar.gz
└── sha256sum-amd64.txt
$ scp -r rke2-artifacts root@192.168.23.92:/root
```

rke2-master02 配置

```bash
$ mkdir -p /etc/rancher/rke2/
$ vim /etc/rancher/rke2/config.yaml
server: https://192.168.23.91:9345
token: K10280f64f7fcf7d94dfa45b6867fd55ef18597e966e5b817552970a24bf15ec6d1::server:417c78df294d6fb88640ef7c9304c070
```
安装 rke2-server

```bash
INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts sh install.sh
cat >>/root/.bashrc<< EOF
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/var/lib/rancher/rke2/bin
EOF
source /root/.bashrc
systemctl enable rke2-server.service && systemctl start rke2-server.service
```
查看日志

```bash
journalctl -u rke2-server -f
```
在rke2-master01 查看集群

```bash
$ kubectl get node
NAME            STATUS   ROLES                       AGE   VERSION
rke2-master01   Ready    control-plane,etcd,master   80m   v1.26.12+rke2r1
rke2-master02   Ready    control-plane,etcd,master   65s   v1.26.12+rke2r1

$ kubectl get po -A
NAMESPACE     NAME                                                    READY   STATUS      RESTARTS   AGE
kube-system   cloud-controller-manager-rke2-master01                  1/1     Running     0          90m
kube-system   cloud-controller-manager-rke2-master02                  1/1     Running     0          10m
kube-system   etcd-rke2-master01                                      1/1     Running     0          89m
kube-system   etcd-rke2-master02                                      1/1     Running     0          10m
kube-system   helm-install-rke2-canal-6v6qr                           0/1     Completed   0          90m
kube-system   helm-install-rke2-coredns-b5ttn                         0/1     Completed   0          90m
kube-system   helm-install-rke2-ingress-nginx-45cqw                   0/1     Completed   0          90m
kube-system   helm-install-rke2-metrics-server-mq6qh                  0/1     Completed   0          90m
kube-system   helm-install-rke2-snapshot-controller-crd-jn4zf         0/1     Completed   0          90m
kube-system   helm-install-rke2-snapshot-controller-zt8f5             0/1     Completed   2          90m
kube-system   helm-install-rke2-snapshot-validation-webhook-kgjbt     0/1     Completed   0          90m
kube-system   kube-apiserver-rke2-master01                            1/1     Running     0          90m
kube-system   kube-apiserver-rke2-master02                            1/1     Running     0          10m
kube-system   kube-controller-manager-rke2-master01                   1/1     Running     0          90m
kube-system   kube-controller-manager-rke2-master02                   1/1     Running     0          10m
kube-system   kube-proxy-rke2-master01                                1/1     Running     0          90m
kube-system   kube-proxy-rke2-master02                                1/1     Running     0          10m
kube-system   kube-scheduler-rke2-master01                            1/1     Running     0          90m
kube-system   kube-scheduler-rke2-master02                            1/1     Running     0          10m
kube-system   rke2-canal-kzvc9                                        2/2     Running     0          11m
kube-system   rke2-canal-ssvcb                                        2/2     Running     0          89m
kube-system   rke2-coredns-rke2-coredns-565dfc7d75-6dbr9              1/1     Running     0          89m
kube-system   rke2-coredns-rke2-coredns-565dfc7d75-tvf2f              1/1     Running     0          11m
kube-system   rke2-coredns-rke2-coredns-autoscaler-6c48c95bf9-lb2xt   1/1     Running     0          89m
kube-system   rke2-ingress-nginx-controller-8lp6v                     1/1     Running     0          88m
kube-system   rke2-ingress-nginx-controller-x2p78                     1/1     Running     0          10m
kube-system   rke2-metrics-server-c9c78bd66-szclt                     1/1     Running     0          89m
kube-system   rke2-snapshot-controller-6f7bbb497d-b426h               1/1     Running     0          88m
kube-system   rke2-snapshot-validation-webhook-65b5675d5c-2b98t       1/1     Running     0          89m
```


### 4.3 新增 worker 节点
传递介质

```bash
$ tree 
.
├── install.sh
├── rke2-images-all.linux-amd64.txt
├── rke2-images.linux-amd64.tar.zst
├── rke2.linux-amd64.tar.gz
└── sha256sum-amd64.txt
$ scp -r rke2-artifacts root@192.168.23.92:/root
```

```bash
$ mkdir -p /etc/rancher/rke2/
$ vim /etc/rancher/rke2/config.yaml
server: https://192.168.23.91:9345
token: K10280f64f7fcf7d94dfa45b6867fd55ef18597e966e5b817552970a24bf15ec6d1::server:417c78df294d6fb88640ef7c9304c070
```

安装 rke2-server

```bash
INSTALL_RKE2_ARTIFACT_PATH=/root/rke2-artifacts INSTALL_RKE2_TYPE="agent" sh install.sh
cat >>/root/.bashrc<< EOF
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/var/lib/rancher/rke2/bin
EOF
source /root/.bashrc
systemctl enable rke2-agent.service && systemctl start rke2-agent.service
```

rke2-master01查看集群

```bash
$ kubectl get node
NAME            STATUS   ROLES                       AGE    VERSION
rke2-master01   Ready    control-plane,etcd,master   132m   v1.26.12+rke2r1
rke2-master02   Ready    control-plane,etcd,master   53m    v1.26.12+rke2r1
rke2-node01     Ready    <none>                      58s    v1.26.12+rke2r1
```


参考：

- [https://docs.rke2.io/zh/install/airgap](https://docs.rke2.io/zh/install/airgap)
- [https://docs.rke2.io/zh/install/quickstart#2-%E5%90%AF%E7%94%A8-rke2-server-%E6%9C%8D%E5%8A%A1](https://docs.rke2.io/zh/install/quickstart#2-%E5%90%AF%E7%94%A8-rke2-server-%E6%9C%8D%E5%8A%A1)
- [https://hackmd.io/@yansheng133/BkpQk1m7j](https://hackmd.io/@yansheng133/BkpQk1m7j)
- [https://github.com/rancher/rke2/releases/tag/v1.26.12%2Brke2r1](https://github.com/rancher/rke2/releases/tag/v1.26.12+rke2r1)
