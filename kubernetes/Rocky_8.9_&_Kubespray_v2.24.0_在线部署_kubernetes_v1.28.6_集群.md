![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f61133758aba8bc76a05010a36a9a64.png)




## 1. 简介

kubespray​ 是一个用于部署和管理 Kubernetes 集群的开源工具。它使用 Ansible 作为配置管理工具，可以根据用户需求灵活地配置和部署 Kubernetes 集群。它还支持各种功能，如高可用性、网络插件、存储插件、日志和监控等。

具有以下几个特点：

- 可以部署在 Ubuntu、CentOS、Red Hat、Google Cloud Platform、Amazon Web Services 和 Microsoft Azure。.
- 部署 High Available Kubernetes 集群.
- 可组合性 (Composable)，可自行选择 Network Plugin (flannel, calico, canal, weave) 来部署.
- 支持多种 Linux distributions(CoreOS, Debian Jessie, Ubuntu 16.04, CentOS/RHEL7).


本次部署说明：[Kubespray v2.24.0](https://github.com/kubernetes-sigs/kubespray/releases/tag/v2.24.0) 在线部署 kubernetes v1.28.6，软件版本支持如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/825e5ede6024fa3a0253bcebc65c3eaa.png)

## 2. 预备条件
6 个节点：

| 角色 | ip  | cpu| mem| disk|ios|kernel|
|--|--|--|--|--|--|--|
| bastion01 |192.168.23.30  |4  |8G|100G| Rocky 8.8 |4.18+|
| kube-master01 |192.168.23.31 |4  |8G|100G|Rocky 8.8|4.18+|
| kube-node01 |192.168.23.32 |16  |32G|100G,200G |Rocky 8.8|4.18+|
| kube-node02 |192.168.23.33 |16 |32G|100G,200G|Rocky 8.8|4.18+|
| kube-node03 |192.168.23.34 |16  |32G|100G,200G|Rocky 8.8|4.18+|
| kube-node04 |192.168.23.35 |16  |32G|100G,200G|Rocky 8.8|4.18+|


注意：机器一定要检查：
- 是否有时间同步服务器：ntp or chrony
- dns
- yum


## 3. 基础配置
### 3.1 配置hostname
对应节点分别执行：
```bash
hostnamctl set-hostname bastion01
hostnamctl set-hostname kube-master01
hostnamctl set-hostname kube-node01
hostnamctl set-hostname kube-node02
hostnamctl set-hostname kube-node03
hostnamctl set-hostname kube-node04

```



### 3.2 配置互信
(bastion01操作)

```bash
ssh-keygen
for i in `cat inventory/sample/inventory.ini |grep host| awk '{print $2}' | awk -F '=' '{print $2}'`;do ssh-copy-id root@$i;done
```




## 4. 配置部署环境

(bastion01操作)

### 4.1 在线安装docker

```bash
sudo dnf check-update
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf  list docker-ce --showduplicates | sort -r
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker && systemctl enable docker  && systemctl status docker
docker pull quay.io/kubespray/kubespray:v2.24.0
```

### 4.2 启动容器 kubespray
(bastion01操作)

```bash
docker run -it --net=host --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa \
  quay.io/kubespray/kubespray:v2.24.0 bash
```

### 4.3 编写 inventory.ini
(bastion01操作)
```bash
$ vim inventory/sample/inventory.ini
[all]
kube-master01 ansible_host=192.168.23.31 ip=192.168.23.31 etcd_member_name=etcd1
kube-master02 ansible_host=192.168.23.32 ip=192.168.23.32 etcd_member_name=etcd2
kube-master03 ansible_host=192.168.23.33 ip=192.168.23.33 etcd_member_name=etcd3
kube-node01 ansible_host=192.168.23.34 ip=192.168.23.34
kube-node02 ansible_host=192.168.23.35 ip=192.168.23.35

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

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```


查看默认的集群配置如下，你可以配置指定的 kubernetes 版本、网络插件、容器运行时等等，[查看支持组件版本](https://github.com/kubernetes-sigs/kubespray/blob/master/roles/download/defaults/main/checksums.yml)。

```bash
root@38261b1472a7:/kubespray# cat inventory/sample/group_vars/k8s_cluster/k8s-cluster.yml |grep -v "#" |grep -v "^$"
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
$ vim  inventory/sample/group_vars/all/all.yml
...
http_proxy: "http://192.168.21.101:7890"
https_proxy: "http://192.168.21.101:7890"
no_proxy: "localhost,127.0.0.0/8,169.0.0.0/8,10.0.0.0/8,192.168.0.0/16,*.coding.net,*.tencentyun.com,*.myqcloud.com"
...
```

###  4.4 关闭防火墙、swap、selinux
(bastion01操作)

```bash
ansible -i inventory/sample/inventory.ini all -m ping
ansible -i inventory/sample/inventory.ini all -m systemd -a "name=firewalld state=stopped enabled=no"
ansible -i inventory/sample/inventory.ini all -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'" -b
ansible -i inventory/sample/inventory.ini all -m shell -a "getenforce 0"
ansible -i inventory/sample/inventory.ini all -m shell -a "sed -i '/.*swap.*/s/^/#/' /etc/fstab" -b
ansible -i inventory/sample/inventory.ini all -m shell -a " swapoff -a && sysctl -w vm.swappiness=0"
```

### 4.5 配置内核模块

```bash
ansible -i inventory/sample/inventory.ini all -m shell -a " modprobe bridge && modprobe br_netfilter && modprobe ip_conntrack"
ansible -i inventory/sample/inventory.ini all -m lineinfile -a "path=/etc/rc.local line='modprobe br_netfilter\nmodprobe ip_conntrack'"
```

## 5. 部署

```bash
ansible-playbook -i inventory/sample/inventory.ini cluster.yml
或者
ansible-playbook -i inventory/sample/inventory.ini --private-key /root/.ssh/id_rsa cluster.yml
或者
ansible-playbook -i inventory/sample/inventory.ini --private-key /root/.ssh/id_rsa --become --become-user=root cluster.yml
```
执行两次，第一次拉取镜像失败了，第二次成功。
输出

```bash
PLAY RECAP ************************************************************************************kube-master01              : ok=732  changed=88   unreachable=0    failed=0    skipped=1140 rescued=0    ignored=6   
kube-master02              : ok=642  changed=77   unreachable=0    failed=0    skipped=1014 rescued=0    ignored=3   
kube-master03              : ok=644  changed=78   unreachable=0    failed=0    skipped=1012 rescued=0    ignored=3   
kube-node01                : ok=510  changed=38   unreachable=0    failed=0    skipped=708  rescued=0    ignored=1   
kube-node02                : ok=510  changed=38   unreachable=0    failed=0    skipped=704  rescued=0    ignored=1   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Tuesday 20 February 2024  12:28:50 +0000 (0:00:00.953)       0:39:11.876 ****** 
=============================================================================== 
download : Download_file | Download item ---------------------------------------------- 67.35s
download : Download_container | Download image if required ---------------------------- 65.96s
download : Download_container | Download image if required ---------------------------- 55.70s
download : Download_container | Download image if required ---------------------------- 52.75s
download : Download_file | Download item ---------------------------------------------- 52.66s
download : Download_container | Download image if required ---------------------------- 47.75s
etcd : Gen_certs | Write etcd member/admin and kube_control_plane client certs to other etcd nodes -- 42.64s
download : Download_file | Download item ---------------------------------------------- 39.98s
download : Download_container | Download image if required ---------------------------- 29.49s
download : Download_container | Download image if required ---------------------------- 29.04s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ---------------- 27.46s
download : Download_container | Download image if required ---------------------------- 26.89s
download : Download_container | Download image if required ---------------------------- 25.15s
kubernetes/control-plane : Kubeadm | Initialize first master -------------------------- 24.86s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources --------------------------- 22.03s
download : Download_container | Download image if required ---------------------------- 20.59s
download : Download_container | Download image if required ---------------------------- 19.24s
kubernetes/control-plane : Joining control plane node to the cluster. ----------------- 18.58s
container-engine/containerd : Download_file | Download item --------------------------- 17.74s
kubespray-defaults : Gather ansible_default_ipv4 from all hosts ----------------------- 16.87s
```

## 6. 集群检查
登陆 kube-master01 节点，检查集群状态

```bash
$ kubectl get node
NAME            STATUS   ROLES           AGE     VERSION
kube-master01   Ready    control-plane   4h12m   v1.28.6
kube-master02   Ready    control-plane   4h11m   v1.28.6
kube-master03   Ready    control-plane   4h11m   v1.28.6
kube-node01     Ready    <none>          4h10m   v1.28.6
kube-node02     Ready    <none>          4h10m   v1.28.6
$ kubectl get pod -A
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-648dffd99-rst7q   1/1     Running   0          4h11m
kube-system   calico-node-h672c                         1/1     Running   0          4h12m
kube-system   calico-node-kh6x6                         1/1     Running   0          4h12m
kube-system   calico-node-ppbck                         1/1     Running   0          4h12m
kube-system   calico-node-sq7j6                         1/1     Running   0          4h12m
kube-system   calico-node-twqxr                         1/1     Running   0          4h12m
kube-system   coredns-77f7cc69db-9xn9l                  1/1     Running   0          4h10m
kube-system   coredns-77f7cc69db-pc9sv                  1/1     Running   0          4h9m
kube-system   dns-autoscaler-8576bb9f5b-lxspw           1/1     Running   0          4h9m
kube-system   kube-apiserver-kube-master01              1/1     Running   1          4h15m
kube-system   kube-apiserver-kube-master02              1/1     Running   1          4h15m
kube-system   kube-apiserver-kube-master03              1/1     Running   2          4h15m
kube-system   kube-controller-manager-kube-master01     1/1     Running   2          4h15m
kube-system   kube-controller-manager-kube-master02     1/1     Running   2          4h15m
kube-system   kube-controller-manager-kube-master03     1/1     Running   4          4h15m
kube-system   kube-proxy-g5ps5                          1/1     Running   0          4h13m
kube-system   kube-proxy-kq8bz                          1/1     Running   0          4h13m
kube-system   kube-proxy-nhsbt                          1/1     Running   0          4h13m
kube-system   kube-proxy-vznb9                          1/1     Running   0          4h13m
kube-system   kube-proxy-xt862                          1/1     Running   0          4h13m
kube-system   kube-scheduler-kube-master01              1/1     Running   1          4h15m
kube-system   kube-scheduler-kube-master02              1/1     Running   1          4h15m
kube-system   kube-scheduler-kube-master03              1/1     Running   1          4h15m
kube-system   nginx-proxy-kube-node01                   1/1     Running   0          4h14m
kube-system   nginx-proxy-kube-node02                   1/1     Running   0          4h14m
kube-system   nodelocaldns-98twv                        1/1     Running   0          4h9m
kube-system   nodelocaldns-cgks9                        1/1     Running   0          4h9m
kube-system   nodelocaldns-jwn7s                        1/1     Running   0          4h9m
kube-system   nodelocaldns-rfpt8                        1/1     Running   0          4h9m
kube-system   nodelocaldns-w4zg2                        1/1     Running   0          4h9m
```

