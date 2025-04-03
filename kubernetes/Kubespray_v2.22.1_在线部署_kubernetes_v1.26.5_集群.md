![](https://i-blog.csdnimg.cn/blog_migrate/b86014b8483e0b9ecbc26272d77855fc.png)




## 1. 介绍
[kubespray​](https://kubespray.io/#/)  是一个用于部署和管理 Kubernetes 集群的开源工具。它使用 Ansible 作为配置管理工具，可以根据用户需求灵活地配置和部署 Kubernetes 集群。它还支持各种功能，如高可用性、网络插件、存储插件、日志和监控等。

具有以下几个特点：
- 可以部署在 Ubuntu、CentOS、Red Hat、Google Cloud Platform、Amazon Web Services 和 Microsoft Azure。.
- 部署 High Available Kubernetes 集群.
- 可组合性 (Composable)，可自行选择 Network Plugin (flannel, calico, canal, weave) 来部署.
- 支持多种 Linux distributions(CoreOS, Debian Jessie, Ubuntu 16.04, CentOS/RHEL7).

Kubespray 项目的发展历程可以分为以下几个阶段：

- Kubernetes CoreOS 阶段（2015-2016）：Kubespray 项目最初是由 Kevin Corcoran 在 2015 年创建的，当时它的名字叫做 Kubernetes CoreOS。这个阶段的目标是帮助使用 CoreOS 操作系统的用户在其上快速部署 Kubernetes 集群。

- Kargo 阶段（2016-2017）：随着 Kubernetes 的普及和用户需求的变化，项目逐渐演变成了一个用于在多个操作系统和云平台上部署和管理 Kubernetes 的工具。在 2016 年，项目改名为 Kargo，并在 GitHub 上进行了开源发布。

- Kubespray 阶段（2017-至今）：由于名称的一些问题，项目在 2017 年再次更名为 Kubespray，这个名字更好地反映了项目的功能和定位。从这个阶段开始，Kubespray 逐渐成为一个成熟的项目，并在 Kubernetes 社区中得到广泛的关注和使用。



本篇将说明如何通过 Kubespray 部署 Kubernetes 至裸机节点，安装版本如下所示：
- [kubespray](https://github.com/kubernetes-sigs/kubespray) v2.22.1
- [Kubernetes](https://kubernetes.io/) v1.26.5
- [Etcd](https://etcd.io/) 3.5.6
- [calico](https://www.calicolabs.com/) v3.25.1
- [containerd](https://containerd.io/) v1.7.1

## 2. 预备条件

通过 vSphere client 创建虚拟机，vSphere client 如何创建虚拟机请看这里。

需求：

系统： Rocky Linux 8.7

CPU: 4

MEM: 8G

DISK1: 30G
DISK2: 50G（openEBS专用）


配置地址与主机名分别为：

```bash
192.168.10.40  bastion01
192.168.10.41  kube-master01
192.168.10.42  kube-master02
192.168.10.43  kube-master03
192.168.10.44  kube-node01
192.168.10.45  kube-node02
192.168.10.46  kube-node03
192.168.10.47  kube-node04
```

注意：以下操作内容均为：192.168.10.40(bastion01) 节点

## 3. 配置 hostname

```bash
192.168.10.40:
hostnamctl set-hostname bastion01
192.168.10.41:
hostnamctl set-hostname kube-master01
192.168.10.42:
hostnamctl set-hostname kube-master02
192.168.10.43:
hostnamctl set-hostname kube-master03
192.168.10.44:
hostnamctl set-hostname kube-node01
192.168.10.45:
hostnamctl set-hostname kube-node02
192.168.10.46:
hostnamctl set-hostname kube-node03
192.168.10.47:
hostnamctl set-hostname kube-node04
```



## 4. yum 
(bastion01操作)

```bash
dnf -y update
dnf -y install wget vim socat wget bash-completion net-tools zip bzip2 bind-utils

```





## 5. 下载 kubespray
默认最快安装
```bash
dnf -y install git
```
或者编译安装指定版本 git
- [git 安装](https://ghostwritten.blog.csdn.net/article/details/125107061)

```bash
git clone -b v2.22.1 https://github.com/kubernetes-sigs/kubespray.git
```

或者
从 [https://github.com/kubernetes-sigs/kubespray/releases](https://github.com/kubernetes-sigs/kubespray/releases) 指定版本下载

```bash
wget https://github.com/kubernetes-sigs/kubespray/archive/refs/tags/v2.22.1.zip
unzip v2.22.1.zip
cd kubespray-v2.22.1
```

## 6. 编写 inventory.ini
(bastion01操作)


```bash
$ vim inventory/sample/inventory.ini
[all]
kube-master01 ansible_host=192.168.10.41 
kube-master02 ansible_host=192.168.10.42
kube-master03 ansible_host=192.168.10.43 
kube-node01 ansible_host=192.168.10.44
kube-node02 ansible_host=192.168.10.45
kube-node03 ansible_host=192.168.10.46
kube-node04 ansible_host=192.168.10.47

[bastion]
bastion ansible_host=192.168.10.40 ansible_user=root

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
kube-node04

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```
## 7. 配置互信
(bastion01操作)

```bash
ssh-keygen
for i in `cat inventory/sample/inventory.ini |grep host| awk '{print $2}' | awk -F '=' '{print $2}'`;do ssh-copy-id root@$i;done
```

## 8. 安装 ansible
(bastion01操作)

```bash
sudo dnf -y install epel-release
sudo dnf -y install ansible

```
## 9. 关闭防火墙
(bastion01操作)

```bash
ansible all -i inventory/sample/inventory.ini -s -m ping
ansible all -i inventory/sample/inventory.ini -s -m systemd -a "name=firewalld state=stopped enabled=no"
```

## 10. 安装 docker
(bastion01操作)

```bash
sudo dnf check-update
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf -y install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl enable docker

docker pull quay.io/kubespray/kubespray:v2.22.1
```
## 11. 配置内核参数
(bastion01操作)

```bash
modprobe bridge
modprobe br_netfilter
cat <<EOF>> /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl -p /etc/sysctl.conf
```

## 12. 启动容器 kubespray
(bastion01操作)

```bash
docker run --rm -it --net=host --mount type=bind,source="$(pwd)"/inventory/sample,dst=/inventory \
  --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa \
  quay.io/kubespray/kubespray:v2.22.1 bash
```
查看默认的集群配置如下，你可以配置指定的 kubernetes 版本、网络插件、容器运行时等等，[查看支持组件版本](https://github.com/kubernetes-sigs/kubespray/blob/master/roles/download/defaults/main/checksums.yml)。

```bash
root@38261b1472a7:/kubespray# cat /inventory/group_vars/k8s_cluster/k8s-cluster.yml |grep -v "#" |grep -v "^$"
---
kube_config_dir: /etc/kubernetes
kube_script_dir: "{{ bin_dir }}/kubernetes-scripts"
kube_manifest_dir: "{{ kube_config_dir }}/manifests"
kube_cert_dir: "{{ kube_config_dir }}/ssl"
kube_token_dir: "{{ kube_config_dir }}/tokens"
kube_api_anonymous_auth: true
kube_version: v1.26.5
local_release_dir: "/tmp/releases"
retry_stagger: 5
kube_owner: kube
kube_cert_group: kube-cert
kube_log_level: 2
credentials_dir: "{{ inventory_dir }}/credentials"
kube_network_plugin: calico
kube_network_plugin_multus: false
kube_service_addresses: 10.233.0.0/18
kube_pods_subnet: 10.233.64.0/18
kube_network_node_prefix: 24
enable_dual_stack_networks: false
kube_service_addresses_ipv6: fd85:ee78:d8a6:8607::1000/116
kube_pods_subnet_ipv6: fd85:ee78:d8a6:8607::1:0000/112
kube_network_node_prefix_ipv6: 120
kube_apiserver_ip: "{{ kube_service_addresses|ipaddr('net')|ipaddr(1)|ipaddr('address') }}"
kube_proxy_mode: ipvs
kube_proxy_strict_arp: false
kube_proxy_nodeport_addresses: >-
  {%- if kube_proxy_nodeport_addresses_cidr is defined -%}
  [{{ kube_proxy_nodeport_addresses_cidr }}]
  {%- else -%}
  []
  {%- endif -%}
kube_encrypt_secret_data: false
cluster_name: cluster.local
ndots: 2
dns_mode: coredns
enable_nodelocaldns: true
enable_nodelocaldns_secondary: false
nodelocaldns_ip: 169.254.25.10
nodelocaldns_health_port: 9254
nodelocaldns_second_health_port: 9256
nodelocaldns_bind_metrics_host_ip: false
nodelocaldns_secondary_skew_seconds: 5
enable_coredns_k8s_external: false
coredns_k8s_external_zone: k8s_external.local
enable_coredns_k8s_endpoint_pod_names: false
resolvconf_mode: host_resolvconf
deploy_netchecker: false
skydns_server: "{{ kube_service_addresses|ipaddr('net')|ipaddr(3)|ipaddr('address') }}"
skydns_server_secondary: "{{ kube_service_addresses|ipaddr('net')|ipaddr(4)|ipaddr('address') }}"
dns_domain: "{{ cluster_name }}"
container_manager: containerd
kata_containers_enabled: false
kubeadm_certificate_key: "{{ lookup('password', credentials_dir + '/kubeadm_certificate_key.creds length=64 chars=hexdigits') | lower }}"
k8s_image_pull_policy: IfNotPresent
kubernetes_audit: false
default_kubelet_config_dir: "{{ kube_config_dir }}/dynamic_kubelet_dir"
podsecuritypolicy_enabled: false
volume_cross_zone_attachment: false
persistent_volumes_enabled: false
event_ttl_duration: "1h0m0s"
auto_renew_certificates: false
kubeadm_patches:
  enabled: false
  source_dir: "{{ inventory_dir }}/patches"
  dest_dir: "{{ kube_config_dir }}/patches"
```
配置代理，否则 `registry.k8s.io` 镜像无法下载
```bash
$ cat /inventory/group_vars/all/all.yml 
...
http_proxy: "http://192.168.10.105:7890"
https_proxy: "http://192.168.10.105:7890"
no_proxy: "localhost,127.0.0.0/8,169.0.0.0/8,10.0.0.0/8,192.168.0.0/16,*.coding.net,*.tencentyun.com,*.myqcloud.com"
...
```

## 13. 部署

```bash
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa cluster.yml
```
输出：

```bash
PLAY RECAP ***************************************************************************************************************************************************************************************
bastion01                  : ok=7    changed=1    unreachable=0    failed=0    skipped=13   rescued=0    ignored=0   
kube-master01              : ok=715  changed=66   unreachable=0    failed=0    skipped=1267 rescued=0    ignored=7   
kube-master02              : ok=618  changed=56   unreachable=0    failed=0    skipped=1107 rescued=0    ignored=2   
kube-master03              : ok=620  changed=57   unreachable=0    failed=0    skipped=1105 rescued=0    ignored=2   
kube-node01                : ok=496  changed=32   unreachable=0    failed=0    skipped=772  rescued=0    ignored=1   
kube-node02                : ok=496  changed=32   unreachable=0    failed=0    skipped=771  rescued=0    ignored=1   
kube-node03                : ok=496  changed=32   unreachable=0    failed=0    skipped=771  rescued=0    ignored=1   
kube-node04                : ok=496  changed=32   unreachable=0    failed=0    skipped=771  rescued=0    ignored=1   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Sunday 16 July 2023  13:10:35 +0000 (0:00:00.475)       0:45:56.422 *********** 
=============================================================================== 
etcd : Gen_certs | Write etcd member/admin and kube_control_plane client certs to other etcd nodes --------------------------------------------------------------------------------------- 58.21s
kubernetes/control-plane : Joining control plane node to the cluster. -------------------------------------------------------------------------------------------------------------------- 45.75s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ------------------------------------------------------------------------------------------------------------------- 30.58s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources ------------------------------------------------------------------------------------------------------------------------------ 30.37s
kubernetes/control-plane : kubeadm | Initialize first master ----------------------------------------------------------------------------------------------------------------------------- 23.72s
container-engine/containerd : containerd | Unpack containerd archive --------------------------------------------------------------------------------------------------------------------- 20.86s
kubernetes/preinstall : Ensure kube-bench parameters are set ----------------------------------------------------------------------------------------------------------------------------- 20.26s
container-engine/validate-container-engine : Populate service facts ---------------------------------------------------------------------------------------------------------------------- 20.00s
network_plugin/calico : Calico | Create calico manifests --------------------------------------------------------------------------------------------------------------------------------- 19.06s
container-engine/crictl : extract_file | Unpacking archive ------------------------------------------------------------------------------------------------------------------------------- 18.65s
network_plugin/cni : CNI | Copy cni plugins ---------------------------------------------------------------------------------------------------------------------------------------------- 17.99s
container-engine/crictl : download_file | Validate mirrors ------------------------------------------------------------------------------------------------------------------------------- 17.95s
container-engine/containerd : download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------- 17.91s
container-engine/runc : download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------------- 17.35s
kubernetes/kubeadm : Join to cluster ----------------------------------------------------------------------------------------------------------------------------------------------------- 16.92s
container-engine/containerd : containerd | Remove orphaned binary ------------------------------------------------------------------------------------------------------------------------ 16.90s
network_plugin/calico : Start Calico resources ------------------------------------------------------------------------------------------------------------------------------------------- 16.07s
container-engine/nerdctl : download_file | Validate mirrors ------------------------------------------------------------------------------------------------------------------------------ 16.03s
container-engine/nerdctl : extract_file | Unpacking archive ------------------------------------------------------------------------------------------------------------------------------ 15.75s
etcd : Gen_certs | Gather etcd member/admin and kube_control_plane client certs from first etcd node ------------------------------------------------------------------------------------- 15.07s

```
exit 退出部署容器

## 14. 配置连接集群
(bastion01操作)

安装 kubectl
```bash
dnf -y install golang
curl -LO https://dl.k8s.io/release/v1.26.5/bin/linux/amd64/kubectl
chmod 755 kubectl
mv kubectl /usr/local/bin
```
bastion01 配置 `kubeconfig`
```bash
mkdir /root/.kube
```
从`kube-master01` 节点 `/etc/kubernetes/admin.conf`  拷贝至bastion01节点 `/root/.kube/config`
并修改`https://127.0.0.1:6443` 为 `https://192.168.10.41:6443`
```bash
$ vim /root/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1EY3hOakV5TlRjeE1sb1hEVE16TURjeE16RXlOVGN4TWxvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTTBWCkhjRVVoT2twa0xyM3F5V3BrZnJyalpNQ1UwdGFPZi83VHYyMWduRFF6d2IzZnpiU1piTlhrc3A5YW9IbUtZKzYKaGxSOFRXUVZTdm8yeGNlUGJhRVd1WktIOEJpUVJENTkwNytaT1Y5SzFOT1VNSUlMVTRFKzJlZmJ2dXdtZjhSOAorVXpZc0Z6RHkzVnR2aW9UbUY4elZaL28yMElXbVh5bWxDRUlySGpWT1NxSU9EdDdTVUxsdHV4dnNyKy9PMUNjCkwwTDZPOHQyanVxWHpibDg2MytES0RwUVREcWFZS3QzM1ZXOFlyV3NPODdWbXA0aEtka2s1TVZwOG43bnQ5SXoKVVp2MW9oMkFoUGxYaXcxRWxuSkc1Rm5OOVErSDlHV0ZraW14dU5KQkpLSGxERVAwRXdSU05xMkVDM3prNWhrNAoyTEJhbHBDa2Y5K25YS0g0Ym1FQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZFRUxuc3RvZ0NJTHROV0NCYVJjM2dCTCtaemRNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBQWp2dXRzZk9YU1JBKzZtTjNzaApMd3RqMWVwQU0ranZVR3BWSlB2R2g1OTg4Ym02bTJNVkd1dDE4c3ZSRk0ySDJwVWNubzhFRDhVaHZnaElMZEh5CnB2YTErYUZ4WHN4bEQ4VzB3Nlk5d0tKMkorN2tmQmJaMUdhdS9WNVQrZmRhMTRJTEt4TGJxRzVLYU4wN1ZGZ2EKVUNvZGhzTkRuRzhxT3Z0dzRpczRwRlBBallTRTU1WTZnMjBWUFdXUDI0SEFpZmxBUmE4ZjlVYVlhWjJxb3BXagpOVFc3cDFVWmRUc283NUIrTEkxcDUrWlRLMllTVzRHdVQ2ZU10eGxWWnZVNG50VmVlaUt6WWNpTXhOTDlzdTdCCm5waTZheEFoc3dQTHdhZVJBMU4vQ24zdS9KdFE5c3ZDWUM2NmxNa2FBRHFSR1U3MTdJSzc2QnMwdCtubGE3NWcKa21FPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.10.41:6443
  name: cluster.local
contexts:
- context:
    cluster: cluster.local
    user: kubernetes-admin
  name: kubernetes-admin@cluster.local
current-context: kubernetes-admin@cluster.local
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJYkwzWTJmaFBreUF3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TXpBM01UWXhNalUzTVRKYUZ3MHlOREEzTVRVeE1qVTNNVFphTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXZvbzVxd1ZvL1hxbnNrVWUKZlRObHl5TjRMdCtGV2c3VldZMTJJSEt2ZXVRa1JnRW1aNGF0eFMvQjJIdEIrczJqZmRPRlkrK1hxZmlZeTdJZApSN3JEdEgxV2pjSFBjeTZqWHVrVHdTT09HS2VnbzlsbkZCVm8xWmtFQk9icHlvM0Exdk1KaEFqek8zdk0zMXA3ClR2aFp3Q0dnYysvMEtqV204SnY5UVZyQ2t5U2l3aG9zTlNPcDRsRW5ieGcwd0xPY28xNTV0VEFjRXBWSE1MREgKdy9GSTRkVlBJYUZPSXpBazhIdldHdmhMWTMyN3lvOUtKdG54cEZNV1NvNmtHQmJDaDJQRmRjbVU4UTNvaUI5MQpubjQ2eFFBL1JqWlNBNk1OM25lSStNa0FWcGJaMjFnRGVsMEhUWEw3Q0dYSEJqOW5TQjBRYVRRWVR2ckpBT1hQCjFlR3B1UUlEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JSQkM1N0xhSUFpQzdUVmdnV2tYTjRBUy9tYwozVEFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBaE85UXI4N0pFQVVsdnZiMGY4blY3OHBpTnArNGgvQTl6S2JnCm1wNnNiOFF5Uy9UalFlTkxGNWRvcWVCKzZpeHc1NkwybGlLWTJZWTNTQnVJc2h6Y042OW9FUW12anExTk1lbzEKRkNxTFJNQjRaaWJkdmQzSWwxVVdpTWZRQ2pwajNzRXFDbWhHY082M2FRbU1wZFpvYkNVMU9HYzEreGhuNkJkSApaVWlJeUNnSXBkVVNzOWpGdStJUENkUWpvMkNpeEdNU1VNNWdGR0YwWUN0K1ZWSTBFNnMxSG9oT1kzV0NqaFFzCnQvT0ZnTGR3YkI5b3lENml2NElQbTR5U2grdUs5WjRWcE0wYUZxTHl2aVRYTnJuR1pXa211RVVHWmQxNmdDVWoKRjlVMCtsekZXZHZCeGRRcitWOStIRlhKWmp6RXdEbGxDdGtJdG1MRmhrendRZnlLaGc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcFFJQkFBS0NBUUVBdm9vNXF3Vm8vWHFuc2tVZWZUTmx5eU40THQrRldnN1ZXWTEySUhLdmV1UWtSZ0VtClo0YXR4Uy9CMkh0QitzMmpmZE9GWSsrWHFmaVl5N0lkUjdyRHRIMVdqY0hQY3k2alh1a1R3U09PR0tlZ285bG4KRkJWbzFaa0VCT2JweW8zQTF2TUpoQWp6TzN2TTMxcDdUdmhad0NHZ2MrLzBLaldtOEp2OVFWckNreVNpd2hvcwpOU09wNGxFbmJ4ZzB3TE9jbzE1NXRUQWNFcFZITUxESHcvRkk0ZFZQSWFGT0l6QWs4SHZXR3ZoTFkzMjd5bzlLCkp0bnhwRk1XU282a0dCYkNoMlBGZGNtVThRM29pQjkxbm40NnhRQS9SalpTQTZNTjNuZUkrTWtBVnBiWjIxZ0QKZWwwSFRYTDdDR1hIQmo5blNCMFFhVFFZVHZySkFPWFAxZUdwdVFJREFRQUJBb0lCQVFDazlJUENaSGVsWXFlRgp0VU1VL3djMFd5dXo0THpRMzZDaTI4NFZmMVFlSHg2c0lGakFMWitJNDdSOUZ4Qmk4ZDZGa3phYTh4U3BDTmczCkdLY3lyeVM0di8zTDBhc29PNHNpSXNTQVk2aWovWk1iNXAzUGpFMXJCZ0t0djc5TkpYVjZZWU91ZEJVblBTRjcKaUJqU29EMExFZEdZTFhlRGgxbFVXcWRoQ2hNRFVLYVVTVXlMMWhoZzhScW40NjNTK2IyOWJ4VGpaQitVTnVjdApKc0ZKQUN3QVcycDMrMDROTi9oNm9IVWxGVnpnRGNmR0tMLzRyUXVST0EwT0ZaSlpWWkI4NkE4M2hobllsczBNCjFEdFlEd3NZUkdLOW9vYk1xaVQ1eHBIY0ZZTENOSWRkVGFTSUdEYWVmWnhtSDR0dmhOUHA1NVpKUHhlbXA0dFUKdnlEMlA5VUJBb0dCQVBZYXFZaytacisrUkdNbXhSRVVoU3NzQVhIWmtNYzBYQWxZZkhvRTVCbURXdUFOQit1WgpBV1NlbkFzclgraGhyV05NTkVTY3hYaEFabmVWZzRQVDB3QUxwblNRSllqVlpxMVhrVFZ0Yk83VWxMdWYvanJ6CkNyN0xRNWFSZjZEUHFhcEhsOXZJOGlNTlM5UWFkbEFwUXhESFZEbFNBc2lLUGpHQ25UMlVBMTdaQW9HQkFNWXoKbVNhb0JKUVhab0kyYi8vcnNlaFNZaXNnWGkxOWFsdWpLVGN6WVFoSzF3YkNtU1p2Mi9jd0tSUVM0YlpUSFF2MwpDOElIT0xXV3YrMUlyTUFIbGtwVWR0VXdCNHkvZEtjZGI5V1NHTzRwODI5ajNrVjJHa3hPRXRwSW00ZDZYa0ZTCjQ5Q0RNRElZQVBMMS9ETUZFY3k4NmhEb29aQVErZEgvNituS0Q1WGhBb0dBSmJwNnFTYWUyK0JRWFo3dzhTaGoKTGZZbUZvMFRDK2IwQVI3R25uSW5nZDNJVGJiUnN3V1cyQlVVdVFXaVExN09GUDMydVZvTFQ5OFhsbGVlZk5RNQpjYlZYaEdFZ3ovUmZORTNMWGhSemNiMjNPM2hRb2pybU44K3pnZDYyWVRIVXdkME40OHpQaWg0Y3ROeUZyTTVXCmtManVLWWR3RTh4VnNvTmlsYkVlUHlrQ2dZRUFxOFQ1L0p0dVpGMm5WRUFUYm9yNGd5d3FzYzk2YnhnYS9kSDQKblVOazI0Zm90STRmcGtVWk1DL0gyZ0xISkhrQldtWS9CV2UyeVFFZDBtbkNkU1hlSlFyd2RiQUxTdnArQVhxcwplajRFWnh0cVF1WWRNcnU0N05wWTBsNU1rK3dFRmI3ZGVzN0hEUkxxZDZXaGJTSCtuQjQ1Q0hCajNIUXAzY3BhCnpTRjF3bUVDZ1lFQW8xMFBHWmREcTNCaHJodjd5NHhiUkJyQldCZTJqNGV2S3pRVUtYeHNmV3pjZ2g2ZGF5NE4KK2VqWFhHaEtaTWRoSzZ0VkVTcFpuRVFrU3VMRFJab2Rra1B6cjBEWDd1bE5iY2JqeXpRR1VWdDV2L0MreWlkYgplcDRzelNuWFVpUDZiaExpL3V0VlRZdXZVT3BXazRXRS80NitDWWJzck1mUWR6SFQ4eHNwaE1NPQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```
测试

```bash
[root@bastion01 ~]# kubectl get pod -A 
NAMESPACE     NAME                                      READY   STATUS    RESTARTS      AGE
kube-system   calico-kube-controllers-6dfcdfb99-lx6mt   1/1     Running   0             18m
kube-system   calico-node-2n7gm                         1/1     Running   0             20m
kube-system   calico-node-5774s                         1/1     Running   0             20m
kube-system   calico-node-bdr75                         1/1     Running   0             20m
kube-system   calico-node-gfw9h                         1/1     Running   0             20m
kube-system   calico-node-mx9wq                         1/1     Running   0             20m
kube-system   calico-node-q84fm                         1/1     Running   0             20m
kube-system   calico-node-xmn2q                         1/1     Running   0             20m
kube-system   coredns-645b46f4b6-6nr7r                  1/1     Running   0             17m
kube-system   coredns-645b46f4b6-qtdqc                  1/1     Running   0             17m
kube-system   dns-autoscaler-659b8c48cb-85rz5           1/1     Running   0             17m
kube-system   kube-apiserver-kube-master01              1/1     Running   1             25m
kube-system   kube-apiserver-kube-master02              1/1     Running   1             24m
kube-system   kube-apiserver-kube-master03              1/1     Running   1             23m
kube-system   kube-controller-manager-kube-master01     1/1     Running   2             25m
kube-system   kube-controller-manager-kube-master02     1/1     Running   3             24m
kube-system   kube-controller-manager-kube-master03     1/1     Running   3             24m
kube-system   kube-proxy-2b9n6                          1/1     Running   0             22m
kube-system   kube-proxy-5tdcr                          1/1     Running   0             22m
kube-system   kube-proxy-6mm8v                          1/1     Running   0             22m
kube-system   kube-proxy-8s982                          1/1     Running   0             22m
kube-system   kube-proxy-lfl5x                          1/1     Running   0             22m
kube-system   kube-proxy-pmn67                          1/1     Running   0             22m
kube-system   kube-proxy-v7gjq                          1/1     Running   0             22m
kube-system   kube-scheduler-kube-master01              1/1     Running   2 (12m ago)   25m
kube-system   kube-scheduler-kube-master02              1/1     Running   1             23m
kube-system   kube-scheduler-kube-master03              1/1     Running   1             24m
kube-system   nginx-proxy-kube-node01                   1/1     Running   0             21m
kube-system   nginx-proxy-kube-node02                   1/1     Running   0             21m
kube-system   nginx-proxy-kube-node03                   1/1     Running   0             22m
kube-system   nginx-proxy-kube-node04                   1/1     Running   0             22m
kube-system   nodelocaldns-4r9vj                        1/1     Running   0             17m
kube-system   nodelocaldns-gcfds                        1/1     Running   0             17m
kube-system   nodelocaldns-k5sgg                        1/1     Running   0             17m
kube-system   nodelocaldns-l2vxf                        1/1     Running   0             17m
kube-system   nodelocaldns-t2t89                        1/1     Running   0             17m
kube-system   nodelocaldns-x2fbm                        1/1     Running   0             17m
kube-system   nodelocaldns-zfl2s                        1/1     Running   0             17m

[root@bastion01 ~]# kubectl get node
NAME            STATUS   ROLES           AGE   VERSION
kube-master01   Ready    control-plane   26m   v1.26.5
kube-master02   Ready    control-plane   25m   v1.26.5
kube-master03   Ready    control-plane   25m   v1.26.5
kube-node01     Ready    <none>          23m   v1.26.5
kube-node02     Ready    <none>          23m   v1.26.5
kube-node03     Ready    <none>          23m   v1.26.5
kube-node04     Ready    <none>          23m   v1.26.5


[root@bastion01 ~]# kubectl version
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5", GitCommit:"890a139214b4de1f01543d15003b5bda71aae9c7", GitTreeState:"clean", BuildDate:"2023-05-17T14:14:46Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.7
Server Version: version.Info{Major:"1", Minor:"26", GitVersion:"v1.26.5", GitCommit:"890a139214b4de1f01543d15003b5bda71aae9c7", GitTreeState:"clean", BuildDate:"2023-05-17T14:08:49Z", GoVersion:"go1.19.9", Compiler:"gc", Platform:"linux/amd64"}
[root@bastion01 ~]# kubectl version --short
Flag --short has been deprecated, and will be removed in the future. The --short output will become the default.
Client Version: v1.26.5
Kustomize Version: v4.5.7
Server Version: v1.26.5

```
kube-master01

```bash
[root@kube-master01 ~]#  /usr/local/bin/etcd --version
etcd Version: 3.5.6
Git SHA: cecbe35ce
Go Version: go1.16.15
Go OS/Arch: linux/amd64

```

## 15. 扩容节点
配置 inventory.ini

```bash
cat /inventory/inventory.ini 
[all]
kube-master01 ansible_host=192.168.10.41 
kube-master02 ansible_host=192.168.10.42
kube-master03 ansible_host=192.168.10.43 
kube-node01 ansible_host=192.168.10.44
kube-node02 ansible_host=192.168.10.45
kube-node03 ansible_host=192.168.10.46
kube-node04 ansible_host=192.168.10.47
kube-node05 ansible_host=192.168.10.48  #添加


[bastion]
bastion ansible_host=192.168.10.40 ansible_user=root

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
kube-node04
kube-node05  #添加

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr

```
执行：
```bash
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa scale.yml
```

##  16. 缩容节点

```bash
ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa -e "node=kube-node05" remove-node.yml
```


## 17. 清理集群

```bash
cd kubespray-2.22.1
docker run --rm -it --net=host --mount type=bind,source="$(pwd)"/inventory/sample,dst=/inventory \
  --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa \
  quay.io/kubespray/kubespray:v2.22.1 bash

ansible-playbook -i /inventory/inventory.ini --private-key /root/.ssh/id_rsa reset.yml
```
输出：

```bashPLAY RECAP *****************************************************************************************************************************************************************************************
bastion01                  : ok=7    changed=1    unreachable=0    failed=0    skipped=13   rescued=0    ignored=0   
kube-master01              : ok=144  changed=16   unreachable=0    failed=0    skipped=155  rescued=0    ignored=0   
kube-master02              : ok=116  changed=15   unreachable=0    failed=0    skipped=133  rescued=0    ignored=0   
kube-master03              : ok=116  changed=15   unreachable=0    failed=0    skipped=133  rescued=0    ignored=0   
kube-node01                : ok=114  changed=13   unreachable=0    failed=0    skipped=135  rescued=0    ignored=0   
kube-node02                : ok=114  changed=13   unreachable=0    failed=0    skipped=135  rescued=0    ignored=0   
kube-node03                : ok=114  changed=13   unreachable=0    failed=0    skipped=135  rescued=0    ignored=0   
kube-node04                : ok=114  changed=13   unreachable=0    failed=0    skipped=135  rescued=0    ignored=0   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Saturday 29 July 2023  04:40:09 +0000 (0:00:05.781)       0:15:42.418 ********* 
=============================================================================== 
reset : reset | delete some files and directories ----------------------------------------------------------------------------------------------------------------------------------------- 255.59s
Gather necessary facts (hardware) ---------------------------------------------------------------------------------------------------------------------------------------------------------- 34.26s
reset : reset | remove services ------------------------------------------------------------------------------------------------------------------------------------------------------------ 25.83s
container-engine/docker : Docker | Remove docker configuration files ----------------------------------------------------------------------------------------------------------------------- 22.12s
Gather information about installed services ------------------------------------------------------------------------------------------------------------------------------------------------ 22.06s
kubespray-defaults : Gather ansible_default_ipv4 from all hosts ---------------------------------------------------------------------------------------------------------------------------- 21.65s
kubernetes/preinstall : Ensure kube-bench parameters are set ------------------------------------------------------------------------------------------------------------------------------- 20.32s
reset : reset | stop etcd services --------------------------------------------------------------------------------------------------------------------------------------------------------- 19.92s
kubernetes/preinstall : Create kubernetes directories -------------------------------------------------------------------------------------------------------------------------------------- 14.55s
container-engine/docker : Docker | Get package facts --------------------------------------------------------------------------------------------------------------------------------------- 14.30s
reset : reset | stop services -------------------------------------------------------------------------------------------------------------------------------------------------------------- 14.14s
reset : flush iptables --------------------------------------------------------------------------------------------------------------------------------------------------------------------- 13.76s
bootstrap-os : Install libselinux python package ------------------------------------------------------------------------------------------------------------------------------------------- 13.67s
reset : reset | unmount kubelet dirs ------------------------------------------------------------------------------------------------------------------------------------------------------- 13.35s
kubernetes/preinstall : Create cni directories --------------------------------------------------------------------------------------------------------------------------------------------- 10.54s
kubernetes/preinstall : Install packages requirements --------------------------------------------------------------------------------------------------------------------------------------- 9.82s
kubernetes/preinstall : Remove swapfile from /etc/fstab ------------------------------------------------------------------------------------------------------------------------------------- 7.48s
kubernetes/preinstall : Create other directories of root owner ------------------------------------------------------------------------------------------------------------------------------ 7.18s
reset : reset | remove dns settings from dhclient.conf -------------------------------------------------------------------------------------------------------------------------------------- 7.13s
kubernetes/preinstall : Hosts | Update (if necessary) hosts file ---------------------------------------------------------------------------------------------------------------------------- 7.08s

在这里插入代码片
```

参考：

- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- [https://kubespray.io/#/](https://kubespray.io/#/)
