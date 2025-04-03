![](https://i-blog.csdnimg.cn/blog_migrate/182c2eb93fa2ddf8208fd39a91d70989.png)





## 1. 背景
手工部署 Kubernetes 二进制集群相对于使用自动化工具或发行版进行部署有一些优势：
1.	定制性：手工部署允许您对 Kubernetes 集群的各个组件进行定制。您可以选择特定的版本、配置选项和插件，以满足您的需求。这种灵活性使您能够根据具体的要求进行精细调整和配置。
2.	理解和掌控：通过手动部署，您能够更加深入地理解 Kubernetes 的各个组件和内部工作原理。这有助于您对集群的运行方式和行为有更全面的了解，并能更好地进行故障排除和性能优化。
3.	教育和学习：手动部署对于学习和教育目的是有益的。通过手工部署，您将了解到 Kubernetes 的各个方面，包括网络、存储、调度和安全等。这对于深入学习和理解 Kubernetes 的工作原理非常有帮助。
4.	灵活性：手动部署使您能够选择更适合您环境和需求的硬件和网络设置。您可以根据自己的资源和约束进行灵活的部署，以满足特定的性能、可用性和安全性要求。
5.	版本控制：手工部署允许您更好地控制和管理 Kubernetes 的版本更新和升级。您可以选择何时升级集群，并可以在升级之前进行必要的测试和验证。
需要注意的是，手工部署 Kubernetes 集群需要更多的时间、资源和技术知识。它对于有经验的操作员和对 Kubernetes 有深入了解的人来说可能更合适。

## 准备条件

2.2.3.2	测试环境K8S网段规划
POD网段：10.0.0.0/16
Service网段：10.255.0.0/16

- kube-master01: 192.168.23.51
os: kylinos
cpu: 4
mem: 8G
disk: 50G

- kube-master01: 192.168.23.51
os: kylinos
cpu: 4
mem: 8G
disk: 50G


## 基础配置
### 配置root远程登录

```bash
sed -i 's/PermitRootLogin no/PermitRootLogin yes/g' /etc/ssh/sshd_config
sed -i 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config 
systemctl restart sshd
```

### 配置主机名
根据所在节点配置主机名

```bash
hostnamectl set-hostname  kube-master01
```

### 安装 ansible
注意：这里ansible 选择性安装。步骤包含单节点执行或批量执行。

```bash
yum -y install epel-release
yum -y install ansible

配置 vim /etc/ansible/hosts

```bash
[all]
kube-master01 ansible_host=192.168.23.51
kube-node01 ansible_host=192.168.23.52


[kube_node]
kube-node01
```





3.1.4	 配置互信

```bash
ssh-keygen
for i in `cat /etc/ansible/hosts |grep 192 | awk '{print $2}' | awk -F '=' '{print $2}'`;do ssh-copy-id root@$i;done
```

测试 ansible

```bash
ansible all -m ping
```

### 配置hosts文件

```bash
cat > /etc/hosts << EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.23.51    kube-master01	       
192.168.23.52    kube-node01    
192.168.23.50  harbor01    
EOF
```

批量:

```bash
ansible all -m copy -a "src=/etc/hosts dest=/etc/hosts force=yes"
ansible all -m shell -a "cat /etc/hosts"
```

### 关闭防firewalld火墙

```bash
systemctl status firewalld|grep Active
systemctl stop firewalld ; systemctl disable firewalld
```

批量：

```bash
ansible all -m systemd -a "name=firewalld state=stopped enabled=no"
```

### 关闭 selinux

```bash
grep ‘SELINUX=’ /etc/selinux/config |grep -v ‘#’
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config 
getenforce
reboot
ansible all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'" -b
```

## 关闭交换分区swap

```bash
sed -ri 's/.*swap.*/#&/' /etc/fstab 
swapoff -a && sysctl -w vm.swappiness=0
free -h
ansible all -m shell -a "sed -i '/.*swap.*/s/^/#/' /etc/fstab" -b
ansible all -m shell -a " swapoff -a && sysctl -w vm.swappiness=0"  
ansible all -m shell -a " free -h"
```

## 修改内核参数

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


ansible all -m shell -a " modprobe bridge && modprobe br_netfilter && modprobe ip_conntrack"
ansible all -m file -a "path=/etc/sysctl.d/k8s.conf state=touch mode=0644"
ansible all -m blockinfile -a "path=/etc/sysctl.d/k8s.conf block='net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
kernel.pid_max = 99999
vm.max_map_count = 262144'"
ansible all -m shell -a " sysctl -p /etc/sysctl.d/k8s.conf"
```

## 安装iptables
在所有master节点与node节点上面安装iptables

```bash
yum install  iptables-services -y
service iptables stop   && systemctl disable iptables
iptables -F
ansible all  -m yum -a "name=iptables-services  state=present"
ansible all -m systemd -a "name=iptables state=stopped enabled=no"
ansible all -m shell -a "iptables -F"
```

## 开启 ipvs
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


ansible all -m copy -a "src=/etc/sysconfig/modules/ipvs.modules dest=/etc/sysconfig/modules/ipvs.modules"

ansible all -m shell -a "chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep ip_vs"
```

### 配置limits参数

```bash
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf
echo "* soft nproc 65536"  >> /etc/security/limits.conf
echo "* hard nproc 65536"  >> /etc/security/limits.conf
echo "* soft  memlock  unlimited"  >> /etc/security/limits.conf
echo "* hard memlock  unlimited"  >> /etc/security/limits.conf

ansible all -m lineinfile -a "path=/etc/security/limits.conf line='* soft nofile 65536\n* hard nofile 65536\n* soft nproc 65536\n* hard nproc 65536\n* soft memlock unlimited\n* hard memlock unlimited'" -b
ansible all -m shell -a "tail -n 7 /etc/security/limits.conf"
```

### 配置时钟同步
注意：时间配置内部NTP服务器，需要提供NTP地址。

```bash
yum install -y htop tree wget jq git net-tools ntpdate
timedatectl set-timezone Asia/Shanghai && date && echo 'Asia/Shanghai' > /etc/timezone
date && ntpdate -u  ntpser01.bsg.com.cn && date
echo '0,10,20,30,40,50 * * * * /usr/sbin/ntpdate -u ntpser01.bsg.com.cn ' >> /var/spool/cron/root && crontab -l
service crond restart && service crond status
cat /var/spool/cron/root && service crond status |grep -I active
ansible all -m shell -a "yum install -y htop tree wget jq git net-tools ntpdate"
ansible all -m shell -a "timedatectl set-timezone Asia/Shanghai && date && echo 'Asia/Shanghai' > /etc/timezone"
ansible all -m shell -a "date && ntpdate -u  ntpser01.bsg.com.cn && date"
ansible all -m shell -a "echo '0,10,20,30,40,50 * * * * /usr/sbin/ntpdate -u ntpser01.bsg.com.cn ' >> /var/spool/cron/root && crontab -l"
ansible all  -m systemd -a "name=crond state=restarted"
```

### 配置journal进行持久化

```bash
sed -i 's/#Storage=auto/Storage=auto/g' /etc/systemd/journald.conf && mkdir -p /var/log/journal && systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald.service
ls -al /var/log/journal



ansible all -m shell -a "sed -i 's/#Storage=auto/Storage=auto/g' /etc/systemd/journald.conf && mkdir -p /var/log/journal && systemd-tmpfiles --create --prefix /var/log/journal"
ansible all  -m systemd -a "name=systemd-journald.service state=restarted"
```

### 配置history命令

```bash
echo 'export HISTTIMEFORMAT="%Y-%m-%d %T"' >> ~/.bashrc && source ~/.bashrc



ansible all -m shell -a "echo 'export HISTTIMEFORMAT=\"%Y-%m-%d %T\"' >> ~/.bashrc && source ~/.bashrc"
```

### 依赖包安装
yum -y install openssl-devel libnl libnl-3 libnl-devel.x86_64  gcc gcc-c++ autoconf automake make  zlib  zlib-devel  unzip conntrack ipvsadm  nfs-utils -y


### Docker软件安装
在所有主机上面安装docker软件

```bash
wget https://download.docker.com/linux/static/stable/x86_64/docker-19.03.15.tgz
```

#将软件包传送到其他节点

```bash
scp docker-19.03.15.tgz  192.168.23.52:/root/
```

#解压安装包

```bash
tar -xf docker-20.10.24.tgz
cp docker/* /usr/bin
mkdir /etc/docker
cat > /usr/lib/systemd/system/docker.service << EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
EOF
```


# 在配置文件中新增本地镜像仓库下载地址

```bash
cat << EOF > /etc/docker/daemon.json
{
"insecure-registries": ["harbor.bsgchina.com"],
"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF



systemctl daemon-reload;systemctl start docker;systemctl enable docker && systemctl status docker
docker login -u admin -p Bsgchina@2023 harbor.bsgchina.com
docker pull harbor.bsgchina.com/k8s-public/busybox:1.28
```


Figure 2: 
1.	测试同步
通过docker push image同步到一台harbor，检查复制管理，看看是否有新的同步出现。
批量：

```bash
ansible all -m copy -a "src=docker dest=/root/"
ansible all -m shell -a "chmod -R 755 /root/docker"
ansible all -m shell -a "cp -a /root/docker/* /usr/bin/"
ansible all -m shell -a "mkdir /etc/docker"
ansible all -m copy -a "src=/usr/lib/systemd/system/docker.service dest=/usr/lib/systemd/system/"
ansible all -m copy -a "src=/etc/docker/daemon.json dest=/etc/docker/"
ansible all -m shell -a "systemctl daemon-reload;systemctl start docker;systemctl enable docker;systemctl status docker"
ansible all -m shell -a "docker pull harbor.bsgchina.com/library/busybox:1.28"
```

注意：harbor 已安装好
### Openssl 升级
下载:
https://www.openssl.org/source/openssl-1.1.1v.tar.gz

```bash
cat /etc/redhat-release
openssl version 
mv /usr/bin/openssl /usr/bin/openssl.bak
mv /usr/include/openssl /usr/include/openssl.bak
tar zxvf openssl-1.1.1v.tar.gz
cd openssl-1.1.1v/
./config --prefix=/usr/local/openssl
make && make install
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
ldconfig -v
openssl version
```


（2）测试harbor高可用拉取推送情况
#分别登陆harbor服务器

```bash
docker login harbor.bsgchina.com #这个是157和158的vip
docker pull httpd
docker tag docker.io/library/httpd:latest harbor.bsgchina.com/library/httpd:latest
docker push  harbor.bsgchina.com/hy/httpd:latest
#换一台机器去拉取刚刚推送上去的镜像
[root@a-t-k8s-node02 docker]# docker pull  harbor.bsgchina.com/library/httpd:latest
latest: Pulling from library/httpd
33847f680f63: Already exists 
d74938eee980: Pull complete 
#拉取成功，harbor的高可用搭建完毕，还可以停止一台harbor服务器再进行测试一下
```

3.1.19	镜像准备

将K8S、监控所需要镜像上传到镜像仓库

准备好介质：

```bash
.
├── docker.io_bats_bats_v1.4.1.tar
├── docker.io_coredns_coredns_1.9.1.tar
├── docker.io_library_busybox_1.31.1.tar
├── images.sh
├── images.txt
├── kubernetesui_dashboard_v2.7.0.tar
├── kubernetesui_metrics-scraper_v1.0.8.tar
├── registry.k8s.io_ingress-nginx_controller_v1.5.1.tar
├── registry.k8s.io_ingress-nginx_kube-webhook-certgen_v20220916-gd32f8c343.tar
├── registry.k8s.io_ingress-nginx_kube-webhook-certgen_v20221220-controller-v1.5.1-58-g787ea74b6.tar
├── registry.k8s.io_kube-state-metrics_kube-state-metrics_v2.9.2.tar
├── registry.k8s.io_pause_3.9.tar
└── siriuszg_addon-resizer_1.8.4.tar

$ cat images.txt 
quay.io/calico/node:v3.24.5
quay.io/calico/pod2daemon-flexvol:v3.24.5
quay.io/calico/cni:v3.24.5
quay.io/calico/kube-controllers:v3.24.5
quay.io/calico/node:v3.24.5
quay.io/calico/pod2daemon-flexvol:v3.24.5
quay.io/calico/cni:v3.24.5
registry.k8s.io/ingress-nginx/controller:v1.5.1
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20220916-gd32f8c343
kubernetesui/dashboard:v2.7.0
kubernetesui/metrics-scraper:v1.0.8
coredns/coredns:1.9.1
registry.k8s.io/pause:3.9
registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.9.2
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20221220-controller-v1.5.1-58-g787ea74b6
docker.io/library/busybox:1.31.1

#镜像解压入库。
docker login -u admin -p ‘Harbor2021#@!’ harbor.bsgchina.com 
./images.sh harbor.bsgchina.com 

harbor.bsgchina.com/k8s-public/calico/node:v3.24.5
harbor.bsgchina.com/k8s-public/calico/pod2daemon-flexvol:v3.24.5
harbor.bsgchina.com/k8s-public/calico/cni:v3.24.5
harbor.bsgchina.com/k8s-public/calico/kube-controllers:v3.24.5
harbor.bsgchina.com/k8s-public/ingress/ingress-nginx_controller:v0.48.1
harbor.bsgchina.com/k8s-public/kubernetesui/dashboard:v2.7.0
harbor.bsgchina.com/k8s-public/kubernetesui/metrics-scraper:v1.0.8 
harbor.bsgchina.com/k8s-public/coredns/coredns:v1.9.1
harbor.bsgchina.com/k8s-public/ingress/kube-webhook-certgen:v1.5.1
harbor.bsgchina.com/k8s-public/kubernetesui/metrics-scraper:v1.0.6
harbor.bsgchina.com/k8s-public/ingress/nginx-ingress-controller:0.24.1
harbor.bsgchina.com/k8s-public/busybox:v1.28.4
harbor.bsgchina.com/k8s-public/kubernetesui/pause-amd64:v3.0
```

## etcd 安装
注意：etcd集群搭建在主机kube-master01上操作。
### 配置etcd证书
#### 配置etcd工作目录

```bash
mkdir -p  /etc/etcd/ssl/
mkdir -p /etc/etcd/cfg
```

#### 安装签发证书工具cfssl

```bash
mkdir cfssl && cd cfssl
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssl_1.6.4_linux_amd64
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssljson_1.6.4_linux_amd64
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.4/cfssl-certinfo_1.6.4_linux_amd64
for i in `ls cfssl*` ;  do mv $i  ${i%%_*} ; done
cp -a  cfssl* /usr/local/bin/
chmod +x /usr/local/bin/cfssl*
echo "export PATH=/usr/local/bin:$PATH" >>/etc/profile
```

#### 配置etcd组件CA证书
1、	自签CA

```bash
cd /etc/etcd/ssl
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "438000h"
    },
    "profiles": {
      "www": {
         "expiry": "438000h",
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
            "L": "Guangzhou",
            "ST": "Guangdong"
        }
    ]
}
EOF
```

2、	生成证书

```bash
cfssl gencert -initca ca-csr.json | cfssljson -bare ca 


$ ls *pem
ca-key.pem  ca.pem
```

#### 使用自签CA签发Etcd HTTPS证书
1、创建证书申请文件
#文件hosts字段中IP为所有etcd节点的集群内部通信IP，不要漏了！为了方便后期扩容可以多写几个ip预留扩容

```bash
cat > etcd-csr.json << EOF
{
    "CN": "etcd",
    "hosts": [
    "192.168.23.51",
    "192.168.23.52",
    "192.168.23.53"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Guangzhou",
            "ST": "Guangdong"
        }
    ]
}
EOF
```

2、生成证书

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www etcd-csr.json | cfssljson -bare etcd

#生成了一个证书和秘钥
ls etcd*pem
etcd-key.pem  etcd.pem
```

### 部署etcd集群
以下在etcd节点1上操作就行，然后把生成的文件拷贝到其他etcd集群主机。
下载地址：[https://github.com/etcd-io/etcd/releases/download/v3.5.3/etcd-v3.5.3-linux-amd64.tar.gz](https://github.com/etcd-io/etcd/releases/download/v3.5.3/etcd-v3.5.3-linux-amd64.tar.gz)
#### 安装etcd工具

```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.5.3/etcd-v3.5.3-linux-amd64.tar.gz
tar zxvf etcd-v3.5.3-linux-amd64.tar.gz
cp -a  etcd-v3.5.3-linux-amd64/{etcd,etcdctl} /usr/local/bin/
```

#### 创建etcd配置文件

```bash
cat > /etc/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.23.51:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.23.51:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.23.51:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.23.51:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.23.51:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

#### 创建启动服务

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
systemctl daemon-reload; systemctl start etcd; systemctl enable etcd;systemctl status etcd
```





#### 检查etcd状态

```bash
ETCDCTL_API=3 /usr/local/bin/etcdctl --cacert=/etc/etcd/ssl/ca.pem --cert=/etc/etcd/ssl/etcd.pem --key=/etc/etcd/ssl/etcd-key.pem --endpoints="https://192.168.23.51:2379" endpoint health
#下面为输出信息 successfully成功
https://192.168.23.51:2379 is healthy: successfully committed proposal: took = 34.533591ms
```

至此etcd就安装完成


### 部署K8S master节点组件
注意：master组件搭建在主机kube-master01上操作。
#### 下载kubernets组件
下载二进制软件包
Github地址：
[https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG)

```bash
wget https://dl.k8s.io/v1.18.5/kubernetes-server-linux-amd64.tar.gz
mkdir -p /etc/kubernetes/bin 
tar zxvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin
cp kubectl kube-apiserver kube-scheduler kube-controller-manager /usr/local/bin/
```

### 部署apiserver组件
#### 创建工作目录
将生成的证书和配置文件临时存放在/data/work路径
mkdir -p /etc/kubernetes/ssl
mkdir /var/log/kubernetes
#### 生成kube-apiserver证书
1、自签证书颁发机构（CA）

```bash
cd /etc/kubernetes/ssl
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "438000h"
    },
    "profiles": {
      "kubernetes": {
         "expiry": "438000h",
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
            "L": "Guangzhou",
            "ST": "Guangdong",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```

2、生成证书

```bash
cd /etc/kubernetes/ssl
cfssl gencert -initca ca-csr.json | cfssljson -bare ca 

ls *pem
ca-key.pem  ca.pem
```

2、使用自签CA签发kube-apiserver HTTPS证书
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
        "192.168.23.50",
        "192.168.23.51",
        "192.168.23.52",
        "192.168.23.53",
        "192.168.23.54",
        "192.168.23.55",
        "192.168.23.56",
        "192.168.23.57",
        "192.168.23.58",
        "192.168.23.59",
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
            "L": "Guangzhou",
            "ST": "Guangdong",
            "O": "k8s",
            "OU": "system"
        }
    ]
}
EOF
```

4、生成证书

```bash
cd /etc/kubernetes/ssl 
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-apiserver-csr.json | cfssljson -bare kube-apiserver

ls kube-apiserver*pem
kube-apiserver-key.pem  kube-apiserver.pem
```

#### 创建token.csv文件

```bash
mkdir -p /etc/kubernetes/cfg/  &&  cd /etc/kubernetes/cfg/
cat > token.csv << EOF
$(head -c 16 /dev/urandom | od -An -t x | tr -d ' '),kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF
```

#### 创建api-server的配置文件

```bash
cat > /etc/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/var/log/kubernetes \\
--etcd-servers=https://192.168.23.51:2379 \\
--bind-address=192.168.23.51 \\
--secure-port=6443 \\
--advertise-address=192.168.23.51 \\
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

####	创建服务启动文件

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

####	设置开机自启动

```bash
systemctl daemon-reload;systemctl start kube-apiserver;systemctl enable kube-apiserver; systemctl status kube-apiserver
```

### 部署kubectl 组件
#### 创建 csr 请求文件

> 注意：O 字段作为 Group； "O": "system:masters", 必须是 system:masters，否则后面 kubectl create clusterrolebinding 报错。证书 O 配置为 system:masters 在集群内部  cluster-admin 的 clusterrolebinding 将system:masters 组和cluster-admin clusterrole 绑定在一起

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
      "ST": "Guangzhou",
      "L": "Guangzhou",
      "O": "system:masters",             
      "OU": "system"
    }
  ]
}
EOF
```

####	生成客户端的证书

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
```

####	配置安全上下文

```bash
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.23.51:6443 --kubeconfig=kube.config
kubectl config set-credentials admin --client-certificate=admin.pem --client-key=admin-key.pem --embed-certs=true --kubeconfig=kube.config
kubectl config set-context kubernetes --cluster=kubernetes --user=admin --kubeconfig=kube.config
kubectl config use-context kubernetes --kubeconfig=kube.config
mkdir ~/.kube -p
cp kube.config ~/.kube/config
cp kube.config /etc/kubernetes/cfg/admin.conf
kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user Kubernetes
```


####	查看集群组件状态

```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.23.51:6443

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


###	部署kube-controller-manager 组件
####	创建 kube-controller-manager csr 请求文件

> 注意：节点hostsip根据所需设置即可。注意：hosts 列表包含所有 kube-controller-manager 节点 IP； CN为 system:kube- controller-manager O 为 system:kube-controller-manager，kubernetes 内置的 ClusterRoleBindings system:kube-controller-manager 赋予 kube-controller-manager 工作所需的权限。

```bash
cd /etc/kuberentes/ssl
cat > kube-controller-manager-csr.json << EOF
{
    "CN": "system:kube-controller-manager",
    "hosts": [
        "127.0.0.1",
        "10.0.0.1",
        "10.255.0.1",
        "192.168.118.50",
        "192.168.118.51",
        "192.168.118.52",
        "192.168.118.53",
        "192.168.118.54",
        "192.168.118.55",
        "192.168.118.56",
        "192.168.118.57",
        "192.168.118.58",
        "192.168.118.59",
        "192.168.118.60",
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
            "L": "Guangzhou",
            "ST": "Guangdong",
            "O": "system:kube-controller-manager",
            "OU": "system"
        }
    ]
}
EOF
```

#### 	生成 kube-controller-manager证书

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

####	创建 kube-controller-manager 的 kubeconfig

```bash
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.23.51:6443 --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-credentials system:kube-controller-manager --client-certificate=kube-controller-manager.pem --client-key=kube-controller-manager-key.pem --embed-certs=true --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-context system:kube-controller-manager --cluster=kubernetes --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig

kubectl config use-context system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
```

####	创建kube-controller-manager配置文件

```bash
mv /etc/kubernetes/ssl/kube-controller-manager.kubeconfig /etc/kubernetes/cfg/
cat >/etc/kubernetes/cfg/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--port=10252 \
  --bind-address=127.0.0.1 \
  --kubeconfig=/etc/kubernetes/cfg/kube-controller-manager.kubeconfig \
  --service-cluster-ip-range=10.255.0.0/16 \
  --cluster-name=kubernetes \
  --allocate-node-cidrs=true \
  --cluster-cidr=10.0.0.0/16 \
  --leader-elect=true \
  --feature-gates=RotateKubeletServerCertificate=true \
  --controllers=*,bootstrapsigner,tokencleaner \
  --horizontal-pod-autoscaler-sync-period=10s \
  --use-service-account-credentials=true \
  --alsologtostderr=true \
  --logtostderr=false \
  --log-dir=/var/log/kubernetes \
  --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem \
  --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem  \
  --root-ca-file=/etc/kubernetes/ssl/ca.pem \
  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --experimental-cluster-signing-duration=438000h0m0s  \
  --v=2"
EOF
```

####	创建kube-controller-manager服务启动文件

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

####	设置kube-controller-manager开机自启动

```bash
systemctl daemon-reload;systemctl start kube-controller-manager;systemctl enable kube-controller-manager; systemctl status kube-controller-manager
```

###	部署kube-scheduler组件
####	创建kube-scheduler的csr 请求
>注意：节点hostsip根据所需设置即可。hosts 列表包含所有 kube-scheduler 节点 IP； CN 为 system:kube-scheduler、O 为 system:kube-scheduler，kubernetes 内置的 ClusterRoleBindings system:kube-scheduler 将赋予 kube-scheduler 工作所需的权限。

```bash
cd /etc/kubernetes/ssl
cat > kube-scheduler-csr.json << EOF
{
    "CN": "system:kube-scheduler",
    "hosts": [
        "127.0.0.1",
        "10.0.0.1",
        "10.255.0.1",
        "192.168.23.50",
        "192.168.23.51",
        "192.168.23.52",
        "192.168.23.53",
        "192.168.23.54",
        "192.168.23.55",
        "192.168.23.56",
        "192.168.23.57",
        "192.168.23.58",
        "192.168.23.59",
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
            "L": "Guangzhou",
            "ST": "Guangdong",
            "O": "system:kube-scheduler",
            "OU": "system"
        }
    ]
}
EOF
```

####	生成kube-scheduler证书

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```

####	创建 kube-scheduler 的 kubeconfig文件

```bash
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.23.51:6443 --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-credentials system:kube-scheduler --client-certificate=kube-scheduler.pem --client-key=kube-scheduler-key.pem --embed-certs=true --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-context system:kube-scheduler --cluster=kubernetes --user=system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
kubectl config use-context system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
```

#### 创建kube-scheduler配置文件

```bash
mv /etc/kubernetes/ssl/kube-scheduler.kubeconfig /etc/kubernetes/cfg/
cat >/etc/kubernetes/cfg/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--logtostderr=false \\
--v=2 \\
--kubeconfig=/etc/kubernetes/cfg/kube-scheduler.kubeconfig \\
--log-dir=/var/log/kubernetes \\
--leader-elect \\
--bind-address=127.0.0.1"
EOF
```

####	创建kube-scheduler服务启动文件

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

####	设置kube-scheduler开机自启动

```bash
systemctl daemon-reload;systemctl start kube-scheduler;systemctl enable kube-scheduler;systemctl status kube-scheduler
```

####	检查集群状态
master所有组件都已经启动成功，通过kubectl工具查看当前集群组件状态：

```bash
kubectl get  cs

NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-1               Healthy   {"health":"true"}   
etcd-2               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"}
```

##	部署k8s-Worker 节点组件
下面还是在Master节点上操作，即同时作为Worker Node(master节点也是能工作的只是默认打了不可调度污点)
###	创建工作目录并拷贝二进制文件
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
scp kubelet kube-proxy root@192.168.23.52:/usr/local/bin/
```

### 部署kubelet
以下操作在master1上面操作
####	创建kubelet配置文件
#参数说明
- –hostname-override：显示名称，集群中唯一
- –network-plugin：启用CNI
- –kubeconfig：空路径，会自动生成，后面用于连接apiserver
- –bootstrap-kubeconfig：首次启动向apiserver申请证书
- –config：配置参数文件
- –cert-dir：kubelet证书生成目录
- –pod-infra-container-image：管理Pod网络容器的镜像
- -cgroup-driver：启用systemd

```bash
cat > /etc/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/var/log/kubernetes \\
--hostname-override=kube-master01 \\
--network-plugin=cni \\
--kubeconfig=/etc/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/etc/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/etc/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/etc/kubernetes/ssl \\
--cgroup-driver=systemd \\
--pod-infra-container-image=harbor.bsgchina.com/library/pause:3.9"
EOF
```

####	配置kubelet参数文件

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

###	生成bootstrap.kubeconfig文件

```bash
export KUBE_APISERVER="https://192.168.23.51:6443" 
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

####	配置kubelet启动文件

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

####	设置开机启动

```bash
systemctl daemon-reload;systemctl start kubelet;systemctl enable kubelet; systemctl status kubelet
```

###	批准kubelet证书申请并加入集群

```bash
kubectl get csr
###下面为输出结果
NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-X2Ez6ppownEMadnJQIegR2Pdo6L6HQIK3zih83Hk_tc   25s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending

# 批准申请
kubectl certificate approve node-csr-X2Ez6ppownEMadnJQIegR2Pdo6L6HQIK3zih83Hk_tc

# 查看节点 因为还没有部署网络组件和插件所以还没有就绪
kubectl get node
```

###	部署kube-proxy
####	创建kube-proxy配置文件

```bash
cat > /etc/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/var/log/kubernetes/ \\
--config=/etc/kubernetes/cfg/kube-proxy-config.yml"
EOF
```

####	配置参数文件

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

####	生成kube-proxy.kubeconfig文件
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
      "L": "Guangzhou",
      "ST": "Guangdong",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy

ls kube-proxy*pem
```

2、生成kubeconfig文件：

```bash
cd /etc/kubernetes/cfg/
KUBE_APISERVER="https://192.168.23.51:6443"

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

####	创建kube-proxy启动服务文件

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

####	设置开机自启动

```bash
systemctl daemon-reload;systemctl start kube-proxy;systemctl enable kube-proxy; sleep 2;systemctl status kube-proxy
```

3.4.4.6	配置kubectl命令自动补全
在所有master节点执行

```bash
yum install -y bash-completion 
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

####	部署CNI网络
3.4.5.1	下载cni-plugins插件
先准备好CNI二进制文件：
下载地址： 
[https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz](https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz)
解压二进制包并移动到默认工作目录：

```bash
mkdir -p /opt/cni/bin
tar zxvf cni-plugins-linux-amd64-v1.2.0.tgz -C /opt/cni/bin
```

####	下载calico插件

```bash
#下载calico
wget https://github.com/projectcalico/calico/releases/download/v3.24.5/release-v3.24.5.tgz
tar -zxvf release-v3.24.5.tgz

cd release-v3.24.5/manifests/
```

3.4.5.3	修改calico配置文件
下载wget https://github.com/projectcalico/calico/archive/v3.24.5.tar.gz
在calico-3.24.5/manifests/目录编辑calico-etcd.yaml
Cp calico-etcd.yaml calico-etcd.yaml_bak

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
  etcd_endpoints: "https://192.168.23.51:2379,https://192.168.118.44:2379,https://192.168.118.45:2379"
  # If you're using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  #这是上面三个文件在容器内的挂载路径，去掉注释使用默认的就行
  etcd_ca:  "/calico-secrets/etcd-ca"
  etcd_cert: "/calico-secrets/etcd-cert"
  etcd_key:  "/calico-secrets/etcd-key"
...
#将以下2行去掉注释，将ip修改为/etc/kubernetes/kube-controller-manager.conf中--cluster-cidr=10.0.0.0/16
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

#查看该yaml中的image，将其镜像替换成新的镜像地址
harbor.bsgchina.com/library/calico/cni:v3.24.5
harbor.bsgchina.com/library/calico/node:v3.24.5
harbor.bsgchina.com/library/calico/kube-controllers:v3.24.5
#将文件中的镜像替换成新的
各个节点登陆镜像仓库验证

```bash
docker login -u admin -p 'Harbor2021#@!' 192.168.128.156
kubectl apply -f calico-etcd.yaml
kubectl get pods -n kube-system
#pod 启动异常，但node已经Ready，继续下一步。

kubectl get node
NAME           STATUS   ROLES    AGE         VERSION
kube-master01   Ready    <none>   <invalid>   v1.23.17
```

####	授权apiserver访问kubelet

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

###	新增Worker节点
####	拷贝已部署好的Node相关文件到新节点
在master节点将Worker Node涉及文件拷贝到新节点node节点（192.168.118.46、192.168.118.47）

```bash
  scp -r /etc/kubernetes/ root@192.168.23.52:/etc/ ;
  scp -r /usr/lib/systemd/system/{kubelet,kube-proxy}.service root@192.168.23.52:/usr/lib/systemd/system; 
  scp -r /opt/cni/ root@192.168.23.52:/opt/; 
```

#### kubelet证书和kubeconfig文件
在所有node节点上操作

```bash
rm -rf /etc/kubernetes/cfg/kubelet.kubeconfig 
rm -f /etc/kubernetes/ssl/kubelet*
rm -f /etc/kubernetes/cfg/{kube-apiserver,kube-controller-manager,kube-scheduler}.kubeconfig
rm -rf /etc/kubernetes/cfg/{kube-controller-manager.conf,kube-scheduler.conf,kube-apiserver.conf}
mkdir -p /var/log/kubernetes
```

3.4.6.3	修改主机名
改成各节点对应主机名

```bash
vi /etc/kubernetes/cfg/kubelet.conf
--hostname-override=a-t-wms-k8s-node01

vi /etc/kubernetes/cfg/kube-proxy-config.yml
hostnameOverride: a-t-wms-k8s-node01
```

####	设置开机自启动

```bash
systemctl daemon-reload;systemctl start kubelet;systemctl enable kubelet;systemctl start kube-proxy;systemctl enable kube-proxy;systemctl status kubelet;systemctl status kube-proxy
```

####	在Master上批准新Node kubelet证书申请

```bash
kubectl get csr
#跟上面一样
NAME                                                   AGE     SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-8tQMJx_zBLGfmPbbkm6eusU9LYpm95LdFBZAsFfQPxM   41m     kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Approved,Issued
node-csr-LPeOESRPGxxFrrM6uUhHFp22Ick-bjJ3oIYsvlYnhzs   3m48s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Approved,Issued

kubectl certificate approve node-csr-LPeOESRPGxxFrrM6uUhHFp22Ick-bjJ3oIYsvlYnhzs
```

####	查看Node节点状态

```bash
kubectl get node
```

###	新增master节点
新Master 与已部署的Master1所有操作一致。所以我们只需将Master1所有K8s文件拷贝过来，再修改下服务器IP和主机名启动即可。
####	拷贝文件（Master1操作）
拷贝Master1上所有K8s文件和etcd证书到Master2~master3节点：

```bash
scp -r /etc/kubernetes root@192.168.23.52:/etc
scp -r /opt/cni/ root@192.168.23.52:/opt
scp /usr/lib/systemd/system/kube* root@192.168.23.52:/usr/lib/systemd/system
scp /usr/local/bin/kube*  root@192.168.23.52:/usr/local/bin/
```

####	删除证书文件
在新增的master节点上删除kubelet证书和kubeconfig文件：

```bash
rm -f /etc/Kubernetes/cfg/kubelet.kubeconfig
rm -f /etc/kubernetes/ssl/kubelet*
mkdir -p /var/log/kubernetes
```

####	修改配置文件IP和主机名
修改master2~master3两个控制节点的apiserver、kubelet和kube-proxy配置文件为本地IP地址和主机名：

```bash
vi /etc/kubernetes/cfg/kube-apiserver.conf
...
--bind-address=192.168.118.$i \
--advertise-address=192.168.118.$i \
...

vi /etc/kubernetes/cfg/kubelet.conf
--hostname-override=a-t-wms-k8s-master02

vi /etc/kubernetes/cfg/kube-proxy-config.yml
hostnameOverride: a-t-wms-k8s-master02
```

####	设置开机自启动

```bash
systemctl daemon-reload;systemctl start kube-apiserver;systemctl start kube-controller-manager;systemctl start kube-scheduler
systemctl start kubelet;systemctl start kube-proxy;systemctl enable kube-apiserver;systemctl enable kube-controller-manager;systemctl enable kube-scheduler;systemctl enable kubelet;systemctl enable kube-proxy
systemctl status kubelet;systemctl status kube-proxy;systemctl status kube-apiserver;systemctl status kube-controller-manager;systemctl status kube-scheduler
```

####	查看集群状态

```bash
kubectl get cs

NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-1               Healthy   {"health":"true"}   
etcd-2               Healthy   {"health":"true"}   
etcd-0               Healthy   {"health":"true"}
```

3.4.7.6	批准kubelet证书申请

```bash
kubectl get csr

NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-JDJFNav36F0SfcRl8weU_tuebqj9OV3yIHSJkVRxnq4   79s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending

kubectl certificate approve node-csr-JDJFNav36F0SfcRl8weU_tuebqj9OV3yIHSJkVRxnq4

kubectl get node
NAME          STATUS   ROLES    AGE   VERSION
k8s-master    Ready    <none>   34h   v1.23.17
k8s-master2   Ready    <none>   83m   v1.23.17
k8s-node1     Ready    <none>   33h   v1.23.17
k8s-node2     Ready    <none>   33h   v1.23.17
```

至此，k8s节点部署完毕


