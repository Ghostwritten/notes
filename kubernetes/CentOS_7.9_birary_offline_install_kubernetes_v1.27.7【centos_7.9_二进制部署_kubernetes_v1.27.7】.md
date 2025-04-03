![](https://i-blog.csdnimg.cn/blog_migrate/159da9b17df44941faad0ce866ff4b15.png)





## 1. 预备条件
系统： centos linux 7.9


192.168.10.121
cpu：4
内存：8
磁盘：60G（系统盘）（Thin Provision）
虚拟机名称：192.168.10.121-centos-7.9-kube-master01
主机名：kube-master01

192.168.10.122
cpu：4
内存：8
磁盘：60G（系统盘）（Thin Provision）
虚拟机名称：192.168.10.122-centos-7.9-kube-master02
主机名：kube-master02

192.168.10.123
cpu：4
内存：8
磁盘：60G（系统盘）（Thin Provision）
虚拟机名称：192.168.10.122-centos-7.9-kube-master03
主机名：kube-master03

192.168.10.124
cpu：4
内存：8
磁盘：60G（系统盘）（Thin Provision）
虚拟机名称：192.168.10.122-centos-7.9-kube-node01
主机名：kube-node04

192.168.10.125
cpu：4
内存：8
磁盘：60G（系统盘）（Thin Provision）
虚拟机名称：192.168.10.122-centos-7.9-kube-node02
主机名：kube-node03


192.168.10.126
cpu：4
内存：8
磁盘：60G（系统盘）（Thin Provision）
虚拟机名称：192.168.10.122-centos-7.9-kube-node03
主机名：kube-node03


192.168.10.12
cpu：4
内存：8
磁盘：60G（系统盘）（Thin Provision）
虚拟机名称：192.168.10.122-centos-7.9-kube-node05
主机名：kube-node05


 软件版本


```bash
kubernetes v1.27.7
containerd_version: 1.7.5
etcd v3.5.9
calico v3.25.2
coredns v1.10.1
nerdctl v1.5.0
runc: v1.1.9
calico: "v3.25.2"
cni: "v1.3.0"
helm: "v3.12.3"
krew: "v0.4.4"
nodelocaldns: "1.22.20"
metrics_server: "v0.6.4"
ingress_nginx: "v1.9.4"
cert_manager: "v1.13.2"
pause: "3.9"
```
## 2. 基础配置
### 2.1 配置root远程登录

```bash
sed -i 's/PermitRootLogin no/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config 
systemctl restart sshd
```

###  2.2 配置主机名

```bash
hostnamectl set-hostname  kube-master01
hostnamectl set-hostname  kube-master02
hostnamectl set-hostname  kube-master03
hostnamectl set-hostname  kube-node01
hostnamectl set-hostname  kube-node02
hostnamectl set-hostname  kube-node03
```

### 2.3 安装 ansible
> 注意：这里ansible 选择性安装。步骤包含单节点执行或批量执行。

```bash
yum -y install epel-release
yum -y install ansible

cat <<EOF> /etc/ansible/hosts
[all]
kube-master01 ansible_host=192.168.10.121
kube-master02 ansible_host=192.168.10.122
kube-master03 ansible_host=192.168.10.123
kube-node01 ansible_host=192.168.10.124
kube-node02 ansible_host=192.168.10.125
kube-node03 ansible_host=192.168.10.126


[k8s:children]
master
node

[master]
kube-master01
kube-master02
kube-master03

[node]
kube-node01
kube-node02
kube-node03
EOF
```
### 2.4 配置互信

```bash
ssh-keygen
for i in `cat /etc/ansible/hosts |grep '192.168' | awk '{print $2}' | awk -F '=' '{print $2}'`;do ssh-copy-id root@$i;done

ansible all -m ping

```
### 2.5 配置hosts文件

```bash
cat > /etc/hosts << EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.10.121   kube-master01
192.168.10.122   kube-master02
192.168.10.123   kube-master03
192.168.10.124   kube-node01      
192.168.10.125   kube-node02      
192.168.10.126   kube-node03 
192.168.10.22 registry01.ghostwritten.com
EOF

批量:
ansible all -m copy -a "src=/etc/hosts dest=/etc/hosts force=yes"
ansible all -m shell -a "cat /etc/hosts"
```

### 2.6 关闭防firewalld火墙

```bash
systemctl stop firewalld ; systemctl disable firewalld

批量：
ansible all -m systemd -a "name=firewalld state=stopped enabled=no"
```

### 2.7 关闭 selinux

```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config 
getenforce
reboot
批量：
ansible all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'" -b

```

###  2.8 关闭交换分区swap

```bash
sed -ri 's/.*swap.*/#&/' /etc/fstab 
swapoff -a && sysctl -w vm.swappiness=0
free -h

批量：
ansible all -m shell -a "sed -i '/.*swap.*/s/^/#/' /etc/fstab" -b
ansible all -m shell -a " swapoff -a && sysctl -w vm.swappiness=0"  
ansible all -m shell -a " free -h"
```

### 2.9 修改内核参数

```bash
modprobe bridge && modprobe br_netfilter && modprobe ip_conntrack
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
kernel.pid_max = 99999
vm.max_map_count = 262144
EOF
sysctl -p /etc/sysctl.d/k8s.conf

批量：
ansible all -m shell -a " modprobe bridge && modprobe br_netfilter && modprobe ip_conntrack"
ansible all -m file -a "path=/etc/sysctl.d/k8s.conf state=touch mode=0644"
ansible all -m blockinfile -a "path=/etc/sysctl.d/k8s.conf block='net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
kernel.pid_max = 99999
vm.max_map_count = 262144'"
ansible all -m shell -a " sysctl -p /etc/sysctl.d/k8s.conf"
```

新改

```bash
cat <<EOF > /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720
 
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
 
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
net.ipv6.conf.all.forwarding = 1
EOF
 
sysctl --system
```

```bash
ansible all -m file -a "path=/etc/sysctl.d/k8s.conf state=touch mode=0644"
ansible all -m copy -a "src=/etc/sysctl.d/k8s.conf dest=/etc/sysctl.d/"
ansible all -m shell -a "sysctl --system"
```

### 2.10 安装iptables

```bash
yum install  iptables-services -y
service iptables stop   && systemctl disable iptables
iptables -F

批量：
ansible all  -m yum -a "name=iptables-services  state=present"
ansible all -m systemd -a "name=iptables state=stopped enabled=no"
ansible all -m shell -a "iptables -F"
```

### 2.11 开启ipvs
在所有master节点与node节点上面需要开启ipvs

```bash
cat > /etc/sysconfig/modules/ipvs.modules << EOF
#!/bin/bash
ipvs_modules="ip_vs ip_vs_lc ip_vs_wlc ip_vs_rr ip_vs_wrr ip_vs_lblc ip_vs_lblcr ip_vs_dh ip_vs_sh ip_vs_nq ip_vs_sed ip_vs_ftp nf_conntrack"
for kernel_module in \${ipvs_modules}; do
 /sbin/modinfo -F filename \${kernel_module} > /dev/null 2>&1
 if [ 0 -eq 0 ]; then
 /sbin/modprobe \${kernel_module}
 fi
done
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep ip_vs


批量：
ansible all -m copy -a "src=/etc/sysconfig/modules/ipvs.modules dest=/etc/sysconfig/modules/ipvs.modules"

ansible all -m shell -a "chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep ip_vs"

```

###  2.12	配置limits参数

```bash
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf
echo "* soft nproc 65536"  >> /etc/security/limits.conf
echo "* hard nproc 65536"  >> /etc/security/limits.conf
echo "* soft  memlock  unlimited"  >> /etc/security/limits.conf
echo "* hard memlock  unlimited"  >> /etc/security/limits.conf


批量：

ansible all -m lineinfile -a "path=/etc/security/limits.conf line='* soft nofile 65536\n* hard nofile 65536\n* soft nproc 65536\n* hard nproc 65536\n* soft memlock unlimited\n* hard memlock unlimited'" -b
```

### 2.13 配置 yum

```bash
ansible all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible all -m shell -a "mkdir -p /etc/yum.repos.d"
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
ansible all -m copy -a " src=/etc/yum.repos.d/CentOS-Base.repo dest=/etc/yum.repos.d/"
ansible all -m shell -a  "yum clean all && yum repolist"
```

###  2.14	配置时钟同步
> 注意：时间配置内部NTP服务器，需要提供NTP地址。

```bash
yum install -y htop tree wget jq git net-tools ntpdate
timedatectl set-timezone Asia/Shanghai && date && echo 'Asia/Shanghai' > /etc/timezone
date && ntpdate -u ntp01.ghostwritten.com.cn && date
echo '0,10,20,30,40,50 * * * * /usr/sbin/ntpdate -u ntp01.ghostwritten.com.cn ' >> /var/spool/cron/root && crontab -l
service crond restart && service crond status

批量：
ansible all -m shell -a "yum install -y htop tree wget jq git net-tools ntpdate"
ansible all -m shell -a "timedatectl set-timezone Asia/Shanghai && date && echo 'Asia/Shanghai' > /etc/timezone"
ansible all -m shell -a "date && ntpdate -u  ntp01.ghostwritten.com.cn && date"
ansible all -m shell -a "echo '0,10,20,30,40,50 * * * * /usr/sbin/ntpdate -u ntp01.ghostwritten.com.cn ' >> /var/spool/cron/root && crontab -l"
ansible all  -m systemd -a "name=crond state=restarted"

```

###  2.15	journal 持久化

```bash
sed -i 's/#Storage=auto/Storage=auto/g' /etc/systemd/journald.conf && mkdir -p /var/log/journal && systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald.service
ls -al /var/log/journal

批量：
ansible all -m shell -a "sed -i 's/#Storage=auto/Storage=auto/g' /etc/systemd/journald.conf && mkdir -p /var/log/journal && systemd-tmpfiles --create --prefix /var/log/journal"
ansible all  -m systemd -a "name=systemd-journald.service state=restarted"
```

###  2.16	配置 history 

```bash
echo 'export HISTTIMEFORMAT="%Y-%m-%d %T"' >> ~/.bashrc && source ~/.bashrc

批量：
ansible all -m shell -a "echo 'export HISTTIMEFORMAT=\"%Y-%m-%d %T\"' >> ~/.bashrc && source ~/.bashrc"
```

###  2.17	依赖包安装

```bash
yum -y install openssl-devel libnl libnl-3 libnl-devel.x86_64  gcc gcc-c++ autoconf automake make  zlib  zlib-devel  unzip conntrack ipvsadm  nfs-utils -y
```

### 2.18 内核升级

- [Linux CentOS7.x 升级内核的方法](https://ghostwritten.blog.csdn.net/article/details/121618058)




## 3. 下载介质与安装 containerd 

下载介质与安装 containerd 软件脚本如下：
```bash
#!/bin/bash


name=`basename $0 .sh`
ENABLE_DOWNLOAD=${ENABLE_DOWNLOAD:-true}
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"
REGISTRY='registry01.ghostwritten.com'

if [ ! -e files ]; then
    mkdir -p files
fi

FILES_DIR=./files
IMAGES_DIR=./images

# download files, if not found
download() {
    url=$1
    dir=$2

    filename=$(basename $1)
    mkdir -p ${FILES_DIR}/$dir

    if [ ! -e ${FILES_DIR}/$dir/$filename ]; then
        echo "==> download $url"
        (cd ${FILES_DIR}/$dir && curl -SLO $1)
    fi
}

download_files() {

if $ENABLE_DOWNLOAD; then
    # TODO: These version must be same as kubespray. Refer `roles/downloads/defaults/main.yml` of kubespray.
    RUNC_VERSION=1.1.9
    CONTAINERD_VERSION=1.7.5
    NERDCTL_VERSION=1.5.0
    CRICTL_VERSION=1.27.1
    CNI_VERSION=1.3.0
    DASHBOARD_VERSION=3.0.0-alpha0
    METRICS_VERSION=0.6.4
    HELM_VERSION=3.13.2
    ETCD_VERSION=3.5.9
    KUBERNETES_VERSION=2.17.7
    CFSSL_VERSION=1.6.4
    CALICO_VERSION=3.25.2

    download https://github.com/opencontainers/runc/releases/download/v${RUNC_VERSION}/runc.amd64 runc/v${RUNC_VERSION}
    download https://github.com/containerd/containerd/releases/download/v${CONTAINERD_VERSION}/containerd-${CONTAINERD_VERSION}-linux-amd64.tar.gz
    download https://github.com/containerd/nerdctl/releases/download/v${NERDCTL_VERSION}/nerdctl-${NERDCTL_VERSION}-linux-amd64.tar.gz
    download https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz
    download https://github.com/containernetworking/plugins/releases/download/v${CNI_VERSION}/cni-plugins-linux-amd64-v${CNI_VERSION}.tgz kubernetes/cni
    download https://raw.githubusercontent.com/kubernetes/dashboard/v${DASHBOARD_VERSION}/charts/kubernetes-dashboard.yaml kubernetes/dashboard
    download https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml kubernetes/ingress-nginx
    download https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed kubernetes/coredns
    download https://github.com/kubernetes-sigs/metrics-server/releases/download/v${METRICS_VERSION}/components.yaml kubernetes/metrics-server
    download https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz 
    download https://github.com/etcd-io/etcd/releases/download/v${ETCD_VERSION}/etcd-v${ETCD_VERSION}-linux-amd64.tar.gz 
    download https://dl.k8s.io/v${KUBERNETES_VERSION}/kubernetes-server-linux-amd64.tar.gz kubernetes
    download https://github.com/cloudflare/cfssl/releases/download/v${CFSSL_VERSION}/cfssl_${CFSSL_VERSION}_linux_amd64 cfssl
    download https://github.com/cloudflare/cfssl/releases/download/v${CFSSL_VERSION}/cfssljson_${CFSSL_VERSION}_linux_amd64 cfssl
    download https://github.com/cloudflare/cfssl/releases/download/v${CFSSL_VERSION}/cfssl-certinfo_${CFSSL_VERSION}_linux_amd64 cfssl
    download https://github.com/projectcalico/calico/releases/download/v${CALICO_VERSION}/calicoctl-linux-amd64 kubernetes/calico
    download https://github.com/projectcalico/calico/archive/v${CALICO_VERSION}.tar.gz kubernetes/calico




    
else
    FILES_DIR=./files
fi

}

download_images() {

    mkdir -p ${IMAGES_DIR}
    cp ${BASE_DIR}/images.sh  ${IMAGES_DIR}
    cp ${BASE_DIR}/images.txt ${IMAGES_DIR}

    for image in `cat ${BASE_DIR}/images.txt`;do
       image_name=`echo $image | awk -F ':' '{print $1}' | sed 's/\//\_/g'`
       image_version=`echo $image | awk -F ':' '{print $2}'`
       if !  $docker pull $image;then
         $docker pull $image
       fi
       $docker save -o ${RELEASE_DIR}/images/${image_name}_${image_version}.tar $image
    done

}

select_latest() {
    local latest=$(ls $* | tail -1)
    if [ -z "$latest" ]; then
        echo "No such file: $*"
        exit 1
    fi
    echo $latest
}




install_runc() {

# Install runc
echo "==> Install runc"
sudo cp $(select_latest "${FILES_DIR}/runc/v*/runc.amd64") /usr/local/bin/runc
sudo chmod 755 /usr/local/bin/runc

}

install_nerdctl() {
# Install nerdctl
echo "==> Install nerdctl"
tar xvf $(select_latest "${FILES_DIR}/nerdctl-*-linux-amd64.tar.gz") -C /tmp
sudo cp /tmp/nerdctl /usr/local/bin

}

install_crictl () {
# Install crictl plugins
echo "==> Install crictl plugins"
sudo tar xvzf $(select_latest "${FILES_DIR}/crictl-v*-linux-amd64.tar.gz") -C /usr/local/bin

cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF

}

install_containerd() {
# Install containerd

echo "==> Install containerd"

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
systemctl restart systemd-modules-load.service
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl --system


sudo tar xvf $(select_latest "${FILES_DIR}/containerd-*-linux-amd64.tar.gz") --strip-components=1 -C /usr/local/bin

cat <<EOF> /etc/systemd/system/containerd.service
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
EOF

sudo mkdir -p \
     /etc/systemd/system/containerd.service.d \
     /etc/containerd \
     /var/lib/containerd \
     /run/containerd


containerd config default | tee /etc/containerd/config.toml
 
sed -i "s#SystemdCgroup\ \=\ false#SystemdCgroup\ \=\ true#g" /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep SystemdCgroup
 
sed -i "s#registry.k8s.io/pause:3.8#${REGISTRY}/library/pause:3.9#g" /etc/containerd/config.toml
cat /etc/containerd/config.toml | grep sandbox_image


local config='/etc/containerd/config.toml'
if grep -q "registry.mirrors]" $config;then
    registry_mirrors=$(grep "registry.mirrors]" $config | awk '{print $1}' |  sed 's/\[/\\\[/g;s/\]/\\\]/g;s/\./\\\./g;s/\"/\\\"/g' )
    sudo sed -i "/${registry_mirrors}/ a\ \ \ \ \ \ \ \ \[plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors.\"${REGISTRY}\"]\n\ \ \ \ \ \ \ \ \ \ endpoint = [\"http://${REGISTRY}\"]" $config
else
    sudo sed -i '/\[plugins\.\"io\.containerd\.grpc\.v1\.cri\"\.registry\]/ a\ \ \ \ \ \ [plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors]\n\ \ \ \ \ \ \ \ [plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors.\"${REGISTRY}\"]\n\ \ \ \ \ \ \ \ \ \ endpoint = [\"http://${REGISTRY}\"]' $config
fi

echo "==> Start containerd"
sudo systemctl daemon-reload && sudo systemctl enable --now containerd && sudo systemctl restart containerd && sudo systemctl status containerd | grep Active
}

install_cni() {
# Install cni plugins
echo "==> Install CNI plugins"
sudo mkdir -p /opt/cni/bin
sudo tar xvzf $(select_latest "${FILES_DIR}/kubernetes/cni/cni-plugins-linux-amd64-v*.tgz") -C /opt/cni/bin

}

action=$1

case $action in
  d )
    download_files
    #download_images
    ;;
  i|install)
    install_nerdctl
    install_crictl
    install_runc
    install_containerd
    install_cni
    ;;
   *)
    echo "Usage: $name [d|i]"
    echo "sh $name d: it is download packages."
    echo "sh$name i: it is install packages."
    ;;
esac
exit 0
```

下载介质,会下载所有安装所需介质
> 注意：下载需要网络出海功能

```bash
sh download-containerd.sh d
```

```bash
[root@kube-master01 containerd]# sh download.sh d
==> download https://raw.githubusercontent.com/kubernetes/dashboard/v3.0.0-alpha0/charts/kubernetes-dashboard.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11486  100 11486    0     0   7228      0  0:00:01  0:00:01 --:--:--  7232
==> download https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 16232  100 16232    0     0  11893      0  0:00:01  0:00:01 --:--:-- 11891
==> download https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4490  100  4490    0     0   3675      0  0:00:01  0:00:01 --:--:--  3677
==> download https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.4/components.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100  4186  100  4186    0     0   1953      0  0:00:02  0:00:02 --:--:-- 11824
==> download https://get.helm.sh/helm-v3.13.2-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 15.4M  100 15.4M    0     0   850k      0  0:00:18  0:00:18 --:--:-- 1169k
==> download https://github.com/etcd-io/etcd/releases/download/v3.5.9/etcd-v3.5.9-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 18.4M  100 18.4M    0     0   764k      0  0:00:24  0:00:24 --:--:--  794k
==> download https://dl.k8s.io/v2.17.7/kubernetes-server-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   138  100   138    0     0    158      0 --:--:-- --:--:-- --:--:--   158
100   233  100   233    0     0    132      0  0:00:01  0:00:01 --:--:--   568
==> download https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssl_1.6.4_linux_amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 11.4M  100 11.4M    0     0   717k      0  0:00:16  0:00:16 --:--:-- 1113k
==> download https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssljson_1.6.4_linux_amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 7464k  100 7464k    0     0   652k      0  0:00:11  0:00:11 --:--:--  652k
==> download https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssl-certinfo_1.6.4_linux_amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 9336k  100 9336k    0     0   684k      0  0:00:13  0:00:13 --:--:-- 1024k
==> download https://github.com/projectcalico/calico/releases/download/v3.25.2/calicoctl-linux-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 60.8M  100 60.8M    0     0   685k      0  0:01:30  0:01:30 --:--:--  864k
==> download https://github.com/projectcalico/calico/archive/v3.25.2.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
  0     0    0 22.7M    0     0   755k      0 --:--:--  0:00:30 --:--:--  815k
```


下载后，在每台节点安装 containerd以及镜像管理的工具。直接执行

```bash
sh download.sh i
```





## 4. 安装 nfs
共享介质其他节点，方便其他节点安装
kube-master01
```bash
yum -y install nfs-utils
cat <<EOF> /etc/exports
/root/k8s-offline-binary-install *(rw,no_root_squash)
EOF
systemctl start nfs-server && systemctl enable nfs-server
exportfs -rv
ansible all -m shell -a "yum -y install nfs-utils" --limit 'all:!kube-master01'
ansible all -m shell -a "showmount -e 192.168.10.121" --limit 'all:!kube-master01'
ansible all -m shell -a "mkdir /opt/k8s" --limit 'all:!kube-master01'
ansible all -m shell -a "mount -t nfs 192.168.10.121:/root/k8s-offline-binary-install /opt/k8s" --limit 'all:!kube-master01'
ansible all -m shell -a "df -Th" --limit 'all:!kube-master01'
```

## 5. 安装镜像仓库
如果已有镜像仓库，例如harbor 则无需再安装

- [Centos 7.9 Install Docker Insecure Registry](https://ghostwritten.blog.csdn.net/article/details/134633564)
- [centos 7.9 部署 harbor 镜像仓库实践](https://blog.csdn.net/xixihahalelehehe/article/details/127920005)

推送镜像入库

当前目录

```bash
$ ls
download.sh  files  images  images.sh  images.txt
```
- download.sh: 下载介质（包、镜像）并安装 containerd
- file： 存放介质包
- images： 存放镜像包
- images.sh 解压镜像包并打标签推送镜像入私有仓库
- images.txt 镜像列表

镜像推送入库脚本如下

```bash
 cat images.sh
#!/bin/bash

type=$1
registry_name=${2:-'registry.infini:5000'}
registry_project=${3:-''}
docker=/usr/bin/docker
BASE_DIR="$(dirname "$(readlink -f "${0}")")"
IMAGES_DIR=${BASE_DIR}/images

load_images() {

 
 for i in `ls ${BASE_DIR}/*.tar`
 do 
   sudo $docker load -i ${IMAGES_DIR}/$i
 done


}


save_images() {

    mkdir -p ${IMAGES_DIR}
    cp ${BASE_DIR}/images.sh  ${IMAGES_DIR}
    cp ${BASE_DIR}/images.txt ${IMAGES_DIR}

    for image in `cat ${BASE_DIR}/images.txt`;do
       image_name=`echo $image | awk -F ':' '{print $1}' | sed 's/\//\_/g'`
       image_version=`echo $image | awk -F ':' '{print $2}'`
       if [[ "$(docker images -q $image 2> /dev/null)" == "" ]];then
         $docker pull $image
       fi
       if [[ "$(ls ${IMAGES_DIR} | grep ${image_name} | grep ${image_version})" == "" ]] ;then
          $docker save -o ${IMAGES_DIR}/${image_name}_${image_version}.tar $image
       fi
    done

}

push_images() {

while read -r line
do
    image_repo=`echo $line | awk -F '/' '{print $1}'`
    image_name=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $1}'`
    pro_name=`echo $line | awk -F'/' '{split($(NF-1), arr, ":"); print arr[1]}'`
    image_tag=`echo $line | awk -F '/' '{print $NF}' | awk -F ':' '{print $2}'`
    #echo "docker tag $line ${registry_name}/k8s-public/${pro_name}/${image_name}:${image_tag}"
    if [[ ${registry_project} == '' ]]; then
      image_new="${registry_name}/${pro_name}/${image_name}:${image_tag}"
    else
      image_new="${registry_name}/${registry_project}/${pro_name}/${image_name}:${image_tag}"
    fi
    docker tag $line ${image_new}
    docker push  ${image_new}
   # sudo docker push ${registry_name}/${image_name}:${image_tag}

done < ${BASE_DIR}/images.txt


}

case $type in 
     s|save)
       save_images
       ;;
     p|push)
       load_images
       push_images
       ;;
     *)
     echo "Usage: $name [s|p]"
     echo "info: [$name s] is pull images and save images "
     echo "info: [$name p] is load images and push images "
     exit 1
     ;;
esac
exit 0
```

推送入库，推送前要确认执行命令类型，第一个参数是push，第二个参数是镜像仓库地址，例如：registry01.ghostwritten.com;第三个参数是项目名。

执行示例如下：

```bash
sh images.sh push registry01.ghostwritten.com library
```

## 6. 安装etcd
> 注意：etcd集群搭建在主机kube-master01、kube-master02、kube-master03上操作。另外，下面etcd 证书配置、安装操作，仅在kube-master01执行。
### 6.1 配置 etcd 目录

```bash
mkdir -p  /etc/etcd/ssl/
mkdir -p /etc/etcd/cfg
```
### 6.2 安装签发证书工具c fssl

```bash
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssl_1.6.4_linux_amd64
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssljson_1.6.4_linux_amd64
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssl-certinfo_1.6.4_linux_amd64
for i in `ls cfssl*` ;  do mv $i  ${i%%_*} ; done
cp -a  cfssl* /usr/local/bin/
chmod +x /usr/local/bin/cfssl*
echo "export PATH=/usr/local/bin:$PATH" >>/etc/profile

```
### 6.3 配置 etcd CA证书

```bash
cd /etc/etcd/ssl
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "www": {
         "expiry": "876000h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF


cat > ca-csr.json << EOF
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai"
        }
    ]
}
EOF

```

### 6.4 生成证书

```bash
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca 
2023/09/09 19:01:08 [INFO] generating a new CA key and certificate from CSR
2023/09/09 19:01:08 [INFO] generate received request
2023/09/09 19:01:08 [INFO] received CSR
2023/09/09 19:01:08 [INFO] generating key: rsa-2048
2023/09/09 19:01:09 [INFO] encoded CSR
2023/09/09 19:01:09 [INFO] signed certificate with serial number 138414421117420972973835238652637837605026477450
$ ls
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem

```

### 6.5 自签CA签发Etcd HTTPS证书

创建证书申请文件

文件hosts字段中IP为所有etcd节点的集群内部通信IP，不要漏了！为了方便后期扩容可以多写几个ip预留扩容

```bash
cat > etcd-csr.json << EOF
{
    "CN": "etcd",
    "hosts": [
    "192.168.10.121",
    "192.168.10.122",
    "192.168.10.123"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai"
        }
    ]
}
EOF
```
生成证书

```bash
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www etcd-csr.json | cfssljson -bare etcd

#生成了一个证书和秘钥
$ ls etcd*pem
etcd-key.pem  etcd.pem
```



### 6.6 安装 etcd
以下在etcd节点1上操作就行，然后把生成的文件拷贝到其他etcd集群主机。

```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.5.9/etcd-v3.5.9-linux-amd64.tar.gz
tar zxvf etcd-v3.5.9-linux-amd64.tar.gz
cp -a  etcd-v3.5.9-linux-amd64/{etcd,etcdctl} /usr/local/bin/
```

### 6.7 创建 etcd 配置文件

```bash
cat > /etc/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.10.121:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.10.121:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.10.121:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.10.121:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.10.121:2380,etcd-2=https://192.168.10.122:2380,etcd-3=https://192.168.10.123:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

### 6.8 创建启动服务

```bash
cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
EnvironmentFile=/etc/etcd/cfg/etcd.conf
ExecStart=/usr/local/bin/etcd \
--cert-file=/etc/etcd/ssl/etcd.pem \
--key-file=/etc/etcd/ssl/etcd-key.pem \
--peer-cert-file=/etc/etcd/ssl/etcd.pem \
--peer-key-file=/etc/etcd/ssl/etcd-key.pem \
--trusted-ca-file=/etc/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \
--logger=zap
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

```
### 6.9 同步文件至 kube-master02 与 kube-master03
将master01节点上面的证书与启动配置文件同步到master01~master03节点。

```bash
scp -r /etc/etcd root@192.168.10.122:/etc/
scp -r /etc/etcd root@192.168.10.123:/etc/
scp /usr/lib/systemd/system/etcd.service root@192.168.10.122:/usr/lib/systemd/system/
scp /usr/lib/systemd/system/etcd.service root@192.168.10.123:/usr/lib/systemd/system/
scp /usr/local/bin/etcd  root@192.168.10.122:/usr/local/bin/ 
scp /usr/local/bin/etcd root@192.168.10.123:/usr/local/bin/ 
scp /usr/local/bin/etcdctl  root@192.168.10.122:/usr/local/bin/
scp /usr/local/bin/etcdctl  root@192.168.10.123:/usr/local/bin/
```

然后在节点master02和master03分别修改etcd.conf配置文件中的节点名称和当前服务器IP：
Master02节点etcd配置

```bash
cat <<EOF> /etc/etcd/cfg/etcd.conf
ETCD_NAME="etcd-2"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.10.122:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.10.122:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.10.122:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.10.122:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.10.121:2380,etcd-2=https://192.168.10.122:2380,etcd-3=https://192.168.10.123:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

Master03节点etcd配置

```bash
cat <<EOF> /etc/etcd/cfg/etcd.conf
ETCD_NAME="etcd-3"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.10.123:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.10.123:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.10.123:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.10.123:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.10.121:2380,etcd-2=https://192.168.10.122:2380,etcd-3=https://192.168.10.123:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

然后在master01节点执行如下操作,#可能会卡主不动，手动到对应服务器执行即可

```bash
systemctl daemon-reload; systemctl start etcd; systemctl enable etcd;systemctl status etcd
```
批量：

```bash
ansible master -m shell -a "systemctl daemon-reload;systemctl start  etcd;systemctl enable  etcd && systemctl status  etcd"
```

### 6.10	检查 etcd 状态

```bash
ETCDCTL_API=3 /usr/local/bin/etcdctl --cacert=/etc/etcd/ssl/ca.pem --cert=/etc/etcd/ssl/etcd.pem --key=/etc/etcd/ssl/etcd-key.pem --endpoints="https://192.168.10.121:2379,https://192.168.10.122:2379,https://192.168.10.123:2379" endpoint health
#下面为输出信息 successfully成功
https://192.168.10.123:2379 is healthy: successfully committed proposal: took = 17.306403ms
https://192.168.10.121:2379 is healthy: successfully committed proposal: took = 19.633604ms
https://192.168.10.122:2379 is healthy: successfully committed proposal: took = 26.415757ms

```

至此etcd就安装完成


##  7. 部署 K8S master节点组件
> 注意：master组件搭建在主机kube-master01、kube-master02、kube-master03上操作。另外，下面证书配置、安装仅在 kube-master01 执行。

### 7.1 安装 kubernets 二进制命令

如果遗漏了该介质下载内容，可以手动执行如下命令
```bash
wget https://dl.k8s.io/v1.27.7/kubernetes-server-linux-amd64.tar.gz
```

安装kubernetes 二进制命令
```bash
mkdir -p /etc/kubernetes/bin 
tar zxvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin
cp kubectl kube-apiserver kube-scheduler kube-controller-manager /usr/local/bin/
```

### 7.2	部署 apiserver 组件
#### 7.2.1	创建工作目录

```bash
mkdir -p /etc/kubernetes/ssl
mkdir /var/log/kubernetes
```

#### 7.2.2	生成kube-apiserver证书
1.自签证书颁发机构（CA）

```bash
cd /etc/kubernetes/ssl
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "kubernetes": {
         "expiry": "876000h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF

cat > ca-csr.json << EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```

2.生成证书

```bash
cd /etc/kubernetes/ssl
cfssl gencert -initca ca-csr.json | cfssljson -bare ca 
ls *pem
ca-key.pem  ca.pem
```

3.使用自签CA签发kube-apiserver HTTPS证书
#hosts字段中IP为所有集群成员的ip集群内部ip，一个都不能少！为了方便后期扩容可以多写几个预留的IP

```bash
cd /etc/kubernetes/ssl
cat > kube-apiserver-csr.json << EOF
{
    "CN": "kubernetes",
"hosts": [
        "127.0.0.1",
        "10.0.0.1",
        "10.255.0.1",
        "192.168.10.120",
        "192.168.10.121",
        "192.168.10.122",
        "192.168.10.123",
        "192.168.10.124",
        "192.168.10.125",
        "192.168.10.126",
        "192.168.10.127",
        "192.168.10.128",
        "192.168.10.129",
        "192.168.10.130",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai",
            "O": "k8s",
            "OU": "system"
        }
    ]
}
EOF

```
生成证书

```bash
cd /etc/kubernetes/ssl 
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-apiserver-csr.json | cfssljson -bare kube-apiserver
ls kube-apiserver*pem
kube-apiserver-key.pem  kube-apiserver.pem
```

#### 7.2.3	创建token.csv文件

```bash
mkdir -p /etc/kubernetes/cfg/  &&  cd /etc/kubernetes/cfg/
cat > token.csv << EOF
$(head -c 16 /dev/urandom | od -An -t x | tr -d ' '),kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF
```

#### 7.2.4	创建api-server的配置文件

```bash
cat > /etc/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--v=2 \\
--etcd-servers=https://192.168.10.121:2379,https://192.168.10.122:2379,https://192.168.10.123:2379 \\
--bind-address=192.168.10.121 \\
--secure-port=6443 \\
--advertise-address=192.168.10.121 \\
--allow-privileged=true \\
--service-cluster-ip-range=10.255.0.0/16 \\
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \\
--authorization-mode=RBAC,Node \\
--enable-bootstrap-token-auth=true \\
--token-auth-file=/etc/kubernetes/cfg/token.csv \\
--service-node-port-range=30000-61000 \\
--kubelet-client-certificate=/etc/kubernetes/ssl/kube-apiserver.pem \\
--kubelet-client-key=/etc/kubernetes/ssl/kube-apiserver-key.pem \\
--tls-cert-file=/etc/kubernetes/ssl/kube-apiserver.pem  \\
--tls-private-key-file=/etc/kubernetes/ssl/kube-apiserver-key.pem \\
--client-ca-file=/etc/kubernetes/ssl/ca.pem \\
--service-account-signing-key-file=/etc/kubernetes/ssl/ca-key.pem \\
--service-account-key-file=/etc/kubernetes/ssl/ca-key.pem \\
--service-account-issuer=https://kubernetes.default.svc.cluster.local \\
--etcd-cafile=/etc/etcd/ssl/ca.pem \\
--etcd-certfile=/etc/etcd/ssl/etcd.pem \\
--etcd-keyfile=/etc/etcd/ssl/etcd-key.pem \\
--requestheader-client-ca-file=/etc/kubernetes/ssl/ca.pem \\
--proxy-client-cert-file=/etc/kubernetes/ssl/kube-apiserver.pem \\
--proxy-client-key-file=/etc/kubernetes/ssl/kube-apiserver-key.pem \\
--requestheader-allowed-names=kubernetes \\
--requestheader-extra-headers-prefix=X-Remote-Extra- \\
--requestheader-group-headers=X-Remote-Group \\
--requestheader-username-headers=X-Remote-User \\
--enable-aggregator-routing=true \\
--audit-log-maxage=30 \\
--audit-log-maxbackup=3 \\
--audit-log-maxsize=100 \\
--audit-log-path=/var/log/kubernetes/k8s-audit.log"
EOF
```

#### 7.2.5	创建服务启动文件

```bash
cat >/usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kube-apiserver.conf
ExecStart=/usr/local/bin/kube-apiserver \$KUBE_APISERVER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

#### 7.2.6	设置开机自启动

```bash
systemctl daemon-reload;systemctl start kube-apiserver;systemctl enable kube-apiserver; systemctl status kube-apiserver
```


### 7.3 部署kubectl 组件
#### 7.3.1	创建 csr 请求文件
> 注意：O 字段作为 Group； "O": "system:masters", 必须是 system:masters，否则后面 kubectl create clusterrolebinding 报错。证书 O 配置为 system:masters 在集群内部 cluster-admin 的 clusterrolebinding 将system:masters 组和cluster-admin clusterrole 绑定在一起


```bash
cd /etc/kubernetes/ssl/
cat > admin-csr.json << EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "system:masters",             
      "OU": "system"
    }
  ]
}
EOF
```

####  7.3.2	生成客户端的证书

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
```
#### 7.3.3	配置安全上下文

```bash
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.10.121:6443 --kubeconfig=kube.config
kubectl config set-credentials admin --client-certificate=admin.pem --client-key=admin-key.pem --embed-certs=true --kubeconfig=kube.config
kubectl config set-context kubernetes --cluster=kubernetes --user=admin --kubeconfig=kube.config
kubectl config use-context kubernetes --kubeconfig=kube.config
mkdir ~/.kube -p
cp kube.config ~/.kube/config
cp kube.config /etc/kubernetes/cfg/admin.conf
kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user Kubernetes
```


####  7.3.4	查看集群组件状态

```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.118.43:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
$ kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Unhealthy   …connect: connection refused                              
controller-manager   Unhealthy   …. connect: connection refused                              
etcd-1               Healthy   {"health":"true","reason":""}   
etcd-2               Healthy   {"health":"true","reason":""}   
etcd-0               Healthy   {"health":"true","reason":""}   
$ kubectl get all --all-namespaces
NAMESPACE   NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
default     service/kubernetes   ClusterIP   10.255.0.1   <none>        443/TCP   2m
```

### 7.4	部署kube-controller-manager 组件
#### 7.4.1	创建 kube-controller-manager csr 请求文件
> 注意：节点hosts ip根据所需设置即可。hosts 列表包含所有 kube-controller-manager 节点 IP； CN 为 system:kube- controller-manager O 为 system:kube-controller-manager， kubernetes 内置的 ClusterRoleBindings system:kube-controller-manager 赋予 kube-controller-manager 工作所需的权限。

```bash
cd /etc/kubernetes/ssl
cat > kube-controller-manager-csr.json << EOF
{
    "CN": "system:kube-controller-manager",
    "hosts": [
        "127.0.0.1",
        "10.0.0.1",
        "10.255.0.1",
        "192.168.10.120",
        "192.168.10.121",
        "192.168.10.122",
        "192.168.10.123",
        "192.168.10.124",
        "192.168.10.125",
        "192.168.10.126",
        "192.168.10.127",
        "192.168.10.128",
        "192.168.10.129",
        "192.168.10.130",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai",
            "O": "system:kube-controller-manager",
            "OU": "system"
        }
    ]
}
EOF
```
#### 7.4.2	生成 kube-controller-manager证书

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

#### 7.4.3	创建 kube-controller-manager 的 kubeconfig

```bash
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.10.121:6443 --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-credentials system:kube-controller-manager --client-certificate=kube-controller-manager.pem --client-key=kube-controller-manager-key.pem --embed-certs=true --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-context system:kube-controller-manager --cluster=kubernetes --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
kubectl config use-context system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
```


#### 7.4.4	创建kube-controller-manager配置文件

```bash
mv /etc/kubernetes/ssl/kube-controller-manager.kubeconfig /etc/kubernetes/cfg/
cat >/etc/kubernetes/cfg/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--bind-address=127.0.0.1 \\
  --kubeconfig=/etc/kubernetes/cfg/kube-controller-manager.kubeconfig \\
  --service-cluster-ip-range=10.255.0.0/16 \\
  --cluster-name=kubernetes \\
  --allocate-node-cidrs=true \\
  --cluster-cidr=10.0.0.0/16 \\
  --leader-elect=true \\
  --feature-gates=RotateKubeletServerCertificate=true \\
  --controllers=*,bootstrapsigner,tokencleaner \\
  --horizontal-pod-autoscaler-sync-period=10s \\
  --use-service-account-credentials=true \\
  --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem \\
  --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem  \\
  --root-ca-file=/etc/kubernetes/ssl/ca.pem \\
  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem \\
  --cluster-signing-duration=876000h0m0s  \\
  --v=2"
EOF
```

#### 7.4.5	创建kube-controller-manager服务启动文件

```bash
cat >/usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kube-controller-manager.conf
ExecStart=/usr/local/bin/kube-controller-manager \$KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

#### 7.4.6 设置kube-controller-manager开机自启动

```bash
systemctl daemon-reload;systemctl restart kube-controller-manager;systemctl enable kube-controller-manager; systemctl status kube-controller-manager
```
### 7.5 部署 kube-scheduler 组件
#### 7.5.1 创建 kube-scheduler 的csr 请求
> 注意：节点hostsip根据所需设置即可。hosts 列表包含所有 kube-scheduler 节点 IP； CN 为 system:kube-scheduler、O 为 system:kube-scheduler，kubernetes 内置的 ClusterRoleBindings system:kube-scheduler 将赋予 kube-scheduler 工作所需的权限。

```bash
cd /etc/kubernetes/ssl
cat > kube-scheduler-csr.json << EOF
{
    "CN": "system:kube-scheduler",
    "hosts": [
        "127.0.0.1",
        "10.0.0.1",
        "10.255.0.1",
        "192.168.10.120",
        "192.168.10.121",
        "192.168.10.122",
        "192.168.10.123",
        "192.168.10.124",
        "192.168.10.125",
        "192.168.10.126",
        "192.168.10.127",
        "192.168.10.128",
        "192.168.10.129",
        "192.168.10.130",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Shanghai",
            "ST": "Shanghai",
            "O": "system:kube-scheduler",
            "OU": "system"
        }
    ]
}
EOF
```

####  7.5.2	生成 kube-scheduler 证书

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```

#### 7.5.3 创建 kube-scheduler 的 kubeconfig 文件

```bash
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.10.121:6443 --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-credentials system:kube-scheduler --client-certificate=kube-scheduler.pem --client-key=kube-scheduler-key.pem --embed-certs=true --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-context system:kube-scheduler --cluster=kubernetes --user=system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
kubectl config use-context system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
```

#### 7.5.4 创建 kube-scheduler 配置文件

```bash
mv /etc/kubernetes/ssl/kube-scheduler.kubeconfig /etc/kubernetes/cfg/
cat >/etc/kubernetes/cfg/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--v=2 \\
--kubeconfig=/etc/kubernetes/cfg/kube-scheduler.kubeconfig \\
--leader-elect \\
--bind-address=127.0.0.1"
EOF
```

#### 7.5.5 创建 kube-scheduler 服务启动文件
```bash
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kube-scheduler.conf
ExecStart=/usr/local/bin/kube-scheduler \$KUBE_SCHEDULER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

#### 7.5.6 设置 kube-scheduler 开机自启动

```bash
systemctl daemon-reload;systemctl restart kube-scheduler;systemctl enable kube-scheduler;systemctl status kube-scheduler
```

#### 7.5.7 检查集群状态
master所有组件都已经启动成功，通过kubectl工具查看当前集群组件状态：

```bash
$ kubectl get  cs

NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-1               Healthy   {"health":"true"}   
etcd-2               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"}
```


##  8.	部署 k8s-Worker 节点组件
下面还是在Master节点上操作，即同时作为Worker Node(master节点也是能工作的只是默认打了不可调度污点)
### 8.1	创建工作目录
在所有worker node创建工作目录：

```bash
mkdir -p /etc/kubernetes/{bin,ssl,cfg} 
mkdir -p /var/log/kubernetes 
mkdir -p /var/lib/kubelet
```

从master节点拷贝：
#还是在master上操作

```bash
cd kubernetes/server/bin
cp  kubelet kube-proxy  /usr/local/bin
for i in {122..126};do  scp kubelet kube-proxy root@192.168.10.$i:/usr/local/bin/;done
```

### 8.2 部署 kubelet
以下操作在master1上面操作
#### 8.2.1	创建 kubelet 配置文件
参数说明

```bash
–hostname-override：显示名称，集群中唯一
–kubeconfig：空路径，会自动生成，后面用于连接apiserver
–bootstrap-kubeconfig：首次启动向apiserver申请证书
–config：配置参数文件
–cert-dir：kubelet证书生成目录
–pod-infra-container-image：管理Pod网络容器的镜像
-cgroup-driver：启用systemd
```
配置：

```bash
cat > /etc/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--v=2 \\
--hostname-override=kube-master01 \\
--kubeconfig=/etc/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/etc/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/etc/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/etc/kubernetes/ssl \\
--runtime-request-timeout=15m  \\
--container-runtime-endpoint=unix:///run/containerd/containerd.sock \\
--cgroup-driver=systemd \\
--node-labels=node.kubernetes.io/node=''"
EOF
```

#### 8.2.2	配置 kubelet 参数文件

```bash
cat > /etc/kubernetes/cfg/kubelet-config.yml << EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: systemd
clusterDNS:
- 10.255.0.2
clusterDomain: cluster.local 
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/ssl/ca.pem 
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
EOF
```

#### 8.2.3 生成 bootstrap.kubeconfig 文件

```bash
export KUBE_APISERVER="https://192.168.10.121:6443" 
export TOKEN=$(awk -F "," '{print $1}' /etc/kubernetes/cfg/token.csv)

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=bootstrap.kubeconfig
  
kubectl config set-credentials "kubelet-bootstrap" \
  --token=${TOKEN} \
  --kubeconfig=bootstrap.kubeconfig
  
kubectl config set-context default \
  --cluster=kubernetes \
  --user="kubelet-bootstrap" \
  --kubeconfig=bootstrap.kubeconfig
  
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig

kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap
mv bootstrap.kubeconfig /etc/kubernetes/cfg/
```

#### 8.2.4 配置 kubelet 启动文件

```bash
cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
After=docker.service
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kubelet.conf
ExecStart=/usr/local/bin/kubelet \$KUBELET_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

```

#### 8.2.5 设置开机启动

```bash
systemctl daemon-reload;systemctl restart kubelet;systemctl enable kubelet; systemctl status kubelet
```
### 8.2.6 批准 kubelet 证书申请并加入集群

```bash
$ kubectl get csr
###下面为输出结果
NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-X2Ez6ppownEMadnJQIegR2Pdo6L6HQIK3zih83Hk_tc   25s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending

# 批准申请
$ kubectl certificate approve node-csr-X2Ez6ppownEMadnJQIegR2Pdo6L6HQIK3zih83Hk_tc

# 查看节点 因为还没有部署网络组件和插件所以还没有就绪
$ kubectl get node
NAME            STATUS     ROLES    AGE   VERSION
kube-master01   NotReady   <none>   2s    v1.23.17

```
### 8.3	部署 kube-proxy
#### 8.3.1 创建 kube-proxy 配置文件

```bash
cat > /etc/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--v=2 \\
--config=/etc/kubernetes/cfg/kube-proxy-config.yml"
EOF
```

#### 8.3.2 配置参数文件

```bash
cat > /etc/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /etc/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: kube-master01
clusterCIDR: 10.0.0.0/16
mode: "ipvs"
EOF
```

#### 8.3.3 生成kube-proxy.kubeconfig文件
1、生成kube-proxy证书：

```bash
cd /etc/kubernetes/ssl
cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "Shanghai",
      "ST": "Shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy

ls kube-proxy*pem
```

#### 8.3.4 生成 kubeconfig 文件：

```bash
cd /etc/kubernetes/cfg/
export KUBE_APISERVER="https://192.168.10.121:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials kube-proxy \
  --client-certificate=/etc/kubernetes/ssl/kube-proxy.pem \
  --client-key=/etc/kubernetes/ssl/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

#### 8.3.5 创建 kube-proxy 启动服务文件

```bash
cat > /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Proxy
After=network.target
[Service]
EnvironmentFile=/etc/kubernetes/cfg/kube-proxy.conf
ExecStart=/usr/local/bin/kube-proxy \$KUBE_PROXY_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

#### 8.3.6 设置开机自启动

```bash
systemctl daemon-reload;systemctl start kube-proxy;systemctl enable kube-proxy; sleep 2;systemctl status kube-proxy
```

#### 8.3.7	配置kubectl命令自动补全
在所有master节点执行

```bash
yum install -y bash-completion 
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```
## 9. 部署 CNI 网络
### 9.1 下载 cni-plugins 插件
先准备好CNI二进制文件：

```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz
mkdir -p /opt/cni/bin
tar zxvf cni-plugins-linux-amd64-v1.3.0.tgz -C /opt/cni/bin
```

### 9.2 下载 calico 插件

```bash
wget https://github.com/projectcalico/calico/releases/download/v3.25.2/release-v3.25.2.tgz
tar -zxvf release-v3.25.2.tgz 
cd release-v3.25.2/manifests/
```

### 9.3 修改 calico 配置文件
下载wget https://github.com/projectcalico/calico/archive/v3.25.2.tar.gz
在calico-3.25.2/manifests/目录编辑calico-etcd.yaml

```bash
vi calico-etcd.yaml
...
data:
  # Populate the following with etcd TLS configuration if desired, but leave blank if
  # not using TLS for etcd.
  # The keys below should be uncommented and the values populated with the base64
  # encoded contents of each file that would be associated with the TLS data.
  # Example command for encoding a file contents: cat <file> | base64 -w 0
  #将以下三行注释取消，将null替换成指定值，获取方式cat <file> | base64 -w 0，file可以查看/etc/kubernetes/kube-apiserver.conf 中指定的ectd指定的文件路径
cat  /etc/etcd/ssl/etcd-key.pem | base64 -w 0
cat  /etc/etcd/ssl/etcd.pem | base64 -w 0
cat  /etc/etcd/ssl/ca.pem | base64 -w 0

  etcd-key: null
  etcd-cert: null
  etcd-ca: null
...
data:
  # Configure this with the location of your etcd cluster.
  #同样查看/etc/kubernetes/kube-apiserver.conf，将etcd-server的地址填些进去
  etcd_endpoints: "https://192.168.10.121:2379,https://192.168.10.122:2379,https://192.168.10.123:2379"
  # If you're using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  #这是上面三个文件在容器内的挂载路径，去掉注释使用默认的就行
  etcd_ca:  "/calico-secrets/etcd-ca"
  etcd_cert: "/calico-secrets/etcd-cert"
  etcd_key:  "/calico-secrets/etcd-key"
...
#将以下2行去掉注释，将ip修改为/etc/kubernetes/cfg/kube-controller-manager.conf中--cluster-cidr=10.0.0.0/16
            - name: CALICO_IPV4POOL_CIDR
              value: "10.0.0.0/16" #默认值是"192.168.0.0/16"
             #这一行下发插入下面2行，指定服务器使用的网卡，可以用.*通配匹配，也可以是具体网卡
            - name: IP_AUTODETECTION_METHOD
              value: "interface=ens.*"
...::
#默认开启的是IPIP模式，需要将其关闭，就会自动启用BGP模式
#BGP模式网络效率更高，但是node节点需要在同一网段，如需跨网段部署k8s集群，建议使用默认IPIP模式
            # Enable IPIP
            - name: CALICO_IPV4POOL_IPIP
              value: "Never" #将Always修改成Never
...
```

查看该yaml中的image，将其镜像替换成新的镜像地址
- registry01.ghostwritten.com/library/calico/cni:v3.25.2
- registry01.ghostwritten.com/library/calico/node:v3.25.2
- registry01.ghostwritten.com/library/calico/kube-controllers:v3.25.2

将文件中的镜像替换成新的，各个节点登陆镜像仓库验证

```bash
docker login -u admin -p Harbor12345  registry01.ghostwritten.com
kubectl apply -f calico-etcd.yaml
kubectl get pods -n kube-system
```

#pod 启动异常，但node已经Ready，继续下一步。

$ kubectl get node
NAME           STATUS   ROLES    AGE         VERSION
kube-master01   Ready    <none>   <invalid>   v1.23.17

### 9.4 授权 apiserver 访问 kubelet

```bash
cat > apiserver-to-kubelet-rbac.yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
      - pods/log
    verbs:
      - "*"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF

kubectl apply -f apiserver-to-kubelet-rbac.yaml
```
再次查看

```bash
$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5454b755c5-5ssg8   1/1     Running   0          45s
calico-node-fztkj                          1/1     Running   0          46s

$ kubectl get node
NAME            STATUS   ROLES    AGE   VERSION
kube-master01   Ready    <none>   35m   v1.23.17

```

## 10. 新增 Worker 节点
### 10.1 拷贝已部署好的 Node 相关文件到新节点
在master节点将Worker Node涉及文件拷贝到新节点node节点（192.168.23.44、192.168.23.45、192.168.23.46）

```bash
for i in 124 125 126 
do 
  scp -r /etc/kubernetes/ root@192.168.10.$i:/etc/ ;
  scp -r /usr/lib/systemd/system/{kubelet,kube-proxy}.service root@192.168.10.$i:/usr/lib/systemd/system; 
  scp -r /opt/cni/ root@192.168.10.$i:/opt/; 
done
```

### 10.2 kubelet 证书和 kubeconfig 文件
在所有node节点上操作

```bash
rm -rf /etc/kubernetes/cfg/kubelet.kubeconfig 
rm -f /etc/kubernetes/ssl/kubelet*
rm -f /etc/kubernetes/cfg/{kube-apiserver,kube-controller-manager,kube-scheduler}.kubeconfig
rm -rf /etc/kubernetes/cfg/{kube-controller-manager.conf,kube-scheduler.conf,kube-apiserver.conf}
mkdir -p /var/log/kubernetes
```

### 10.3 修改主机名
改成各节点对应主机名

```bash
vi /etc/kubernetes/cfg/kubelet.conf
--hostname-override=kube-node01

vi /etc/kubernetes/cfg/kube-proxy-config.yml
hostnameOverride: kube-node01
```

### 10.4 设置开机自启动

```bash
systemctl daemon-reload;systemctl start kubelet;systemctl enable kubelet;systemctl start kube-proxy;systemctl enable kube-proxy;systemctl status kubelet;systemctl status kube-proxy
```

### 10.5 Master 批准新 Node kubelet 证书申请

```bash
$ kubectl get csr
#跟上面一样
NAME                                                   AGE     SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-8tQMJx_zBLGfmPbbkm6eusU9LYpm95LdFBZAsFfQPxM   41m     kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Approved,Issued
node-csr-LPeOESRPGxxFrrM6uUhHFp22Ick-bjJ3oIYsvlYnhzs   3m48s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Approved,Issued

kubectl certificate approve node-csr-LPeOESRPGxxFrrM6uUhHFp22Ick-bjJ3oIYsvlYnhzs

$ kubectl get pod -n kube-system 
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5454b755c5-5ssg8   1/1     Running   0          22m
calico-node-787r8                          1/1     Running   0          2m8s
calico-node-fztkj                          1/1     Running   0          22m

$ ubectl get node
NAME            STATUS     ROLES    AGE     VERSION
kube-master01   Ready      <none>   2d20h   v1.27.7
kube-node01     Ready      <none>   22m     v1.27.7
```
## 11. 新增 master 节点
新Master 与已部署的Master1所有操作一致。所以我们只需将Master1所有K8s文件拷贝过来，再修改下服务器IP和主机名启动即可。
### 11.1 拷贝文件（Master1操作）
拷贝Master1上所有K8s文件和etcd证书到Master2~master3节点：

```bash
for i in 122 123; 
do
    scp -r /etc/kubernetes root@192.168.10.$i:/etc; 
    scp -r /opt/cni/ root@192.168.10.$i:/opt; 
    scp /usr/lib/systemd/system/kube* root@192.168.10.$i:/usr/lib/systemd/system; 
    scp /usr/local/bin/kube*  root@192.168.10.$i:/usr/local/bin/; 
done
```

### 11.2 删除证书文件
在新增的master节点上删除kubelet证书和kubeconfig文件：

```bash
rm -f /etc/Kubernetes/cfg/kubelet.kubeconfig
rm -f /etc/kubernetes/ssl/kubelet*
mkdir -p /var/log/kubernetes
```

### 11.3 修改配置文件IP和主机名
修改master2~master3两个控制节点的apiserver、kubelet和kube-proxy配置文件为本地IP地址和主机名：

```bash
vi /etc/kubernetes/cfg/kube-apiserver.conf
...
--bind-address=192.168.23.42 \
--advertise-address=192.168.23.42 \
...

vi /etc/kubernetes/cfg/kubelet.conf
--hostname-override=kube-master02

vi /etc/kubernetes/cfg/kube-proxy-config.yml
hostnameOverride: kube-master02
```

### 11.4 设置开机自启动

```bash
systemctl daemon-reload;systemctl start kube-apiserver;systemctl start kube-controller-manager;systemctl start kube-scheduler
systemctl start kubelet;systemctl start kube-proxy;systemctl enable kube-apiserver;systemctl enable kube-controller-manager;systemctl enable kube-scheduler;systemctl enable kubelet;systemctl enable kube-proxy
systemctl status kubelet;systemctl status kube-proxy;systemctl status kube-apiserver;systemctl status kube-controller-manager;systemctl status kube-scheduler
```



### 11.5 批准 kubelet 证书申请

```bash
kubectl get csr

NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-JDJFNav36F0SfcRl8weU_tuebqj9OV3yIHSJkVRxnq4   79s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending

kubectl certificate approve node-csr-JDJFNav36F0SfcRl8weU_tuebqj9OV3yIHSJkVRxnq4

$ kubectget node
NAME            STATUS   ROLES    AGE    VERSION
kube-master01   Ready    <none>   106m   v1.23.17
kube-master02   Ready    <none>   17m    v1.23.17
kube-master03   Ready    <none>   85s    v1.23.17
kube-node01     Ready    <none>   52m    v1.23.17
kube-node02     Ready    <none>   47m    v1.23.17
kube-node03     Ready    <none>   41m    v1.23.17

至此，k8s节点部署完毕
```


## 12. 集群管理
### 12.1 设置节点角色

```bash
kubectl label nodes kube-master01 node-role.kubernetes.io/master=
kubectl label nodes kube-master02 node-role.kubernetes.io/master=
kubectl label nodes kube-master03 node-role.kubernetes.io/master=
kubectl label nodes kube-node01 node-role.kubernetes.io/worker=
kubectl label nodes kube-node02 node-role.kubernetes.io/worker=
kubectl label nodes kube-node03 node-role.kubernetes.io/worker=
```

#### 12.2. master 节点设置不可调度

```bash
kubectl taint nodes kube-master01  node-role.kubernetes.io/master=:NoSchedule
kubectl taint nodes kube-master02  node-role.kubernetes.io/master=:NoSchedule
kubectl taint nodes kube-master03  node-role.kubernetes.io/master=:NoSchedule
kubectl cordon kube-master01
kubectl cordon kube-master02
kubectl cordon kube-master03
kubectl describe node kube-master01 |grep Taints
```

取消不可调度的命令，不需要执行（不需要执行）

```bash
kubectl taint node  kube-master01 node-role.kubernetes.io/master-
```




##  13.	部署 CoreDNS v1.10.1
CoreDNS用于集群内部Service名称解析,版本`v1.10.1`

```bash
wget https://raw.githubusercontent.com/coredns/deployment/master/kubernetes/coredns.yaml.sed

#修改corefile.yaml
  Corefile: |
    .:53 {
        log
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa { #修改为
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
            ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf #修改为本机的resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }#去掉这个地方的后缀
#修改文件，参考/etc/kubernetes/kubelet-config.yml 配置文件中的clusterIP
spec:
  selector:
    k8s-app: kube-dns
  clusterIP: 10.255.0.2
#将文件中的镜像地址替换成新地址
sed -i 's/coredns\/coredns:1.9.4/registry01.ghostwritten.com\/library\/coredns\/coredns:1.10.1/g' coredns.yaml
registry01.ghostwritten.com/library/coredns/coredns:v1.10.1
```
执行:

```bash
kubectl apply -f coredns.yaml
```
查看

```bash
$ kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5454b755c5-5ssg8   1/1     Running   0          112m
calico-node-787r8                          1/1     Running   0          92m
calico-node-b5vxm                          1/1     Running   0          87m
calico-node-fztkj                          1/1     Running   0          112m
calico-node-rdvkw                          1/1     Running   0          81m
calico-node-rfpbc                          1/1     Running   0          40m
calico-node-zh6bq                          1/1     Running   0          57m
coredns-7d87bc76f7-nlqj4                   1/1     Running   0          30s

```
DNS解析测试：

```bash
$ kubectl run busybox --image registry01.ghostwritten.com/library/busybox:1.31.1 --restart=Never --rm -it busybox -- sh

/ # nslookup kubernetes
Server:    10.255.0.2
Address 1: 10.255.0.2 kube-dns.kube-system.svc.cluster.local
Name:      kubernetes
Address 1: 10.255.0.1 kubernetes.default.svc.cluster.local
```




## 14.	部署 ingress v1.9.4
确认版本：v1.9.4

```bash
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml
sed -i 's/registry.k8s.io/registry01.ghostwritten.com/g' deploy.yaml
```
修改 deploy.yaml

```bash

spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/component: controller
  revisionHistoryLimit: 10
  minReadySeconds: 0
  replicas: 3  #将ingress设为多副本
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/component: controller
    spec:
      dnsPolicy: ClusterFirst
      hostNetwork: true #在此位置添加此项
      containers:
        - name: controller
          image: registry01.ghostwritten.com/library/ingress-nginx/controller:v1.9.4
          imagePullPolicy: IfNotPresent
#并将文件中的镜像地址替换成新镜像地址
registry.k8s.io/ingress-nginx/controller:v1.9.4
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
registry01.ghostwritten.com/library/ingress-nginx/controller:v1.9.4
registry01.ghostwritten.com/library/ingress-nginx/kube-webhook-certgen:v20231011-8b53cabe0
d
```
执行：

```bash
$ kubectl apply -f deploy.yaml
#查看ingress的service
$ kubectl get svc -n ingress-nginx
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.255.81.111   <none>        80:31769/TCP,443:56884/TCP   8s
ingress-nginx-controller-admission   ClusterIP   10.255.36.43    <none>        443/TCP                      8s

$ kubectl get pod -n ingress-nginx
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-njjwd       0/1     Completed   0          61s
ingress-nginx-admission-patch-rqf89        0/1     Completed   2          61s
ingress-nginx-controller-dc9bb5fbc-9qx52   1/1     Running     0          61s


#访问验证
$ curl 192.168.23.41:31769
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
$ curl 192.168.23.41:56884
<html>
<head><title>400 The plain HTTP request was sent to HTTPS port</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<center>The plain HTTP request was sent to HTTPS port</center>
<hr><center>nginx</center>
</body>
</html>

#出现404.说明配置成功
```

测试 ingress 应用
下载介质中会包含次镜像
docker pull nginx:1.25.3

创建一个测试pod

```bash
$ vi nginx-demo.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  selector: 
    app: nginx
  ports:
  - name: http
    targetPort: 80
    port: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: default
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry01.ghostwritten.com/library/nginx:1.25.3
        ports: 
        - name: http
          containerPort: 80 
#创建ingress规则
---
apiVersion: networking.k8s.io/v1
kind: Ingress 
metadata:
  name: ingress-nginx
  namespace: default 
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules: 
  - host: "nginx.demo.com"
    http:
      paths: 
      - pathType: Prefix
        path: "/" 
        backend:
          service:
            name: nginx
            port:
              number: 80
```

部署ingress应用

```bash
kubectl apply -f nginx-demo.yaml

#查看ingress
$ kubectl get ingress
NAME            CLASS    HOSTS            ADDRESS         PORTS   AGE
ingress-nginx   <none>   nginx.demo.com   192.168.23.45   80      2m22s
#配置本地解析
vim /etc/hosts
192.168.23.45 nginx.demo.com

#访问
curl nginx.demo.com
```

##  11. 部署 cert-manager  v1.13.2

```bash
wget https://github.com/cert-manager/cert-manager/releases/download/v1.13.2/cert-manager.yaml
quay.io/jetstack/cert-manager-cainjector:v1.13.2
quay.io/jetstack/cert-manager-controller:v1.13.2
quay.io/jetstack/cert-manager-webhook:v1.13.2
sed -i 's/quay.io/registry01.ghostwritten.com\/library/g' cert-manager.yaml
kubectl apply -f cert-manager.yaml
```
查看

```bash
$ kubectl get pod -n cert-manager
NAME                                       READY   STATUS    RESTARTS      AGE
cert-manager-7bf89fc7df-7mnsq              1/1     Running   1 (44m ago)   129m
cert-manager-cainjector-75f5f7c6fc-v2ng5   1/1     Running   1 (59m ago)   129m
cert-manager-webhook-6bbb999589-xp686      1/1     Running   0             129m
```

##  11.	部署 Dashboard v3.0.0-alpha0

```bash
wget kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v3.0.0-alpha0/charts/kubernetes-dashboard.yaml

vim recommended.yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30001  #添加
  type: NodePort
  selector:
    k8s-app: kubernetes-dashboard
##修改了type:NodePort   和定义了nodePort: 30001
# 修改镜像
sed -i 's/docker.io/registry01.ghostwritten.com/g' kubernetes-dashboard.yaml

执行：

```bash
kubectl apply -f recommended.yaml
kubectl get pods,svc -n kubernetes-dashboard

kubectl create serviceaccount dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

cat <<EOF> secret.yaml 
apiVersion: v1
kind: Secret
metadata:
  name: dashboard-admin
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: dashboard-admin
type: kubernetes.io/service-account-token
EOF

#获取token，用于登陆
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```
修改 kubernetes-dashboard 的 ingress 域名，修改为自定义的域名：kubernetes.dashboard01

```bash
$ kubectl get ingress -n kubernetes-dashboard
NAME                   CLASS   HOSTS                    ADDRESS          PORTS     AGE
kubernetes-dashboard   nginx   kubernetes-dashboard01   192.168.10.126   80, 443   173m

$ kubectl edit ingress -n kubernetes-dashboard kubernetes-dashboard 
  spec:
    ingressClassName: nginx
    rules:
    - host: kubernetes-dashboard01 #修改次行
      http:

$ kubectl get ingress -n kubernetes-dashboard
NAME                   CLASS   HOSTS                    ADDRESS          PORTS     AGE
kubernetes-dashboard   nginx   kubernetes-dashboard01   192.168.10.126   80, 443   173m
```
本地主机mac 添加域名解析 在`/etc/hosts`

```bash
192.168.10.126 kubernetes-dashboard01
```

#登陆到dashboard: `http://kubernetes-dashboard01`, 用刚获取token登陆验证应用状态
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/acdb665279b7479aec8694cf6c60de52.png)

```bash
$ kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
Name:         dashboard-admin
Namespace:    kube-system
Labels:       kubernetes.io/legacy-token-last-used=2023-12-05
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: 6f6b0e4c-41fb-43bf-9330-fa0b0a82d7e2

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1314 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlZwZWtxYm9odHN4Y2xobnVxRG10a2FRa1VHNGJfOUlwTzZ2WG1hMzFzQUUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNmY2YjBlNGMtNDFmYi00M2JmLTkzMzAtZmEwYjBhODJkN2UyIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.2yPU97VIZZuLMAf31aFAz8Me4LERmDwB8HwU3Qky_4vNeUpgmp79rVr3DyCK2GWRzl0RjngNrvgFGCysOBp6cZa9PtN7cACLjr1LV9KM5w457glo0DbKgklQrfQkB6uvJMpc2eirEeJP2fRL20TU5aWIwtqBb42dBihHMYiZSX7XwMBKD9MGwQESHSMi-lTjigGgQteXtzgfVPr5RT2uJ79EJOzqLB2LaM44zgQsJK6WIAssGpKtRsn43Vw-chUS1C-ZtD7ZKFhGLg4kRXIfcCaAFzwcIeP6J59ouaoKDWJU-L__U5JDDQ6tTkKsXGdHxwycqD2T7yDi16tij5hwVw
```

![](https://i-blog.csdnimg.cn/blog_migrate/7607eec79c362e31213aa88746ae9970.png)


##  14. 部署 metrics-server v0.6.4
metrics_server: "v0.6.4"
```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
#修改镜像 和添加 --kubelet-insecure-tls 
命令参数：
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: registry01.ghostwritten.com/library/metrics-server/metrics-server:v0.6.4
        imagePullPolicy: IfNotPresent
```

部署：

```bash
Kubectl apply -f components.yaml
```

查看

```bash
$ kubectl get deploy -n kube-system metrics-server
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            1           8m9s
$ kubectl get pod  -n kube-system |grep metrics-server 
metrics-server-8646bb957d-q4nzt            1/1     Running   0          2m27s

$ kubectl top node
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
kube-master01   470m         11%    3027Mi          38%       
kube-master02   731m         18%    1327Mi          16%       
kube-master03   410m         10%    1279Mi          16%       
kube-node01     199m         4%     804Mi           10%       
kube-node02     222m         5%     727Mi           9%        
kube-node03     166m         4%     867Mi           11%    
```

## 15. 部署 prometheus
(略)

