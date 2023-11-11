

![](https://img-blog.csdnimg.cn/0e79c51c19484d74af61c3691cc34b50.png)





- [ansible-playbook 分步执行可节约排错时间](https://ghostwritten.blog.csdn.net/article/details/130096634)


## 1. TASK [kubernetes/preinstall : Hosts | create list from inventory] 

遇到以下报错：
```bash
TASK [kubernetes/preinstall : Hosts | create list from inventory] ******************************************************************************************************************
fatal: [kube-control-plan01 -> localhost]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'dict object' has no attribute 'address'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/kubernetes/preinstall/tasks/0090-etchosts.yml': line 2, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n---\n- name: Hosts | create list from inventory\n  ^ here\n"}

```
修改以下内容：
该文件修改第6行添加：`and 'address' in hostvars[item]['ansible_default_ipv4']`，实例：

```bash
$ vi /root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/kubernetes/preinstall/tasks/0090-etchosts.yml

     1  ---
     2  - name: Hosts | create list from inventory
     3    set_fact:
     4      etc_hosts_inventory_block: |-
     5        {% for item in (groups['k8s_cluster'] + groups['etcd']|default([]) + groups['calico_rr']|default([]))|unique -%}
     6        {% if 'access_ip' in hostvars[item] or 'ip' in hostvars[item] or 'ansible_default_ipv4' in hostvars[item] and 'address' in hostvars[item]['ansible_default_ipv4'] -%}
```

## 2: TASK [container-engine/containerd : containerd Create registry directories] 
报错内容：
```bash
fatal: [kube-control-plan01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 114, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd Create registry directories\n  ^ here\n"}
```
修改 此文件`with_items` 改为 `with_dict`
```bash
$ vi /root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml


   114  - name: containerd ｜ Create registry directories
   115    file:
   116      path: "{{ containerd_cfg_dir }}/certs.d/{{ item.key }}"
   117      state: directory
   118      mode: 0755
   119      recurse: true
   120    with_dict: "{{ containerd_insecure_registries }}"
   121    when: containerd_insecure_registries is defined
   122
   123  - name: containerd ｜ Write hosts.toml file
   124    blockinfile:
   125      path: "{{ containerd_cfg_dir }}/certs.d/{{ item.key }}/hosts.toml"
   126      owner: "root"
   127      mode: 0640
   128      create: true
   129      block: |
   130        server = "{{ item.value }}"
   131        [host."{{ item.value }}"]
   132          capabilities = ["pull", "resolve", "push"]
   133          skip_verify = true
   134    with_dict: "{{ containerd_insecure_registries }}"
   135    when: containerd_insecure_registries is defined
```




##  3. TASK [kubernetes/control-plane : kubeadm | Initialize first master] 

```bash
TASK [kubernetes/control-plane : kubeadm | Initialize first master] ****************************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"attempts": 3, "changed": true, "cmd": ["timeout", "-k", "300s", "300s", "/usr/local/bin/kubeadm", "init", "--config=/etc/kubernetes/kubeadm-config.yaml", "--ignore-preflight-errors=all", "--skip-phases=addon/coredns", "--upload-certs"], "delta": "0:01:56.126009", "end": "2023-04-10 21:41:40.621046", "failed_when_result": true, "msg": "non-zero return code", "rc": 1, "start": "2023-04-10 21:39:44.495037", "stderr": "W0410 21:39:44.514437  222884 utils.go:69] The recommended value for \"clusterDNS\" in \"KubeletConfiguration\" is: [10.233.0.10]; the provided value is: [169.254.25.10]\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml already 
```
由于containerd 配置导致 kubelet 启动失败：`unknown service runtime.v1alpha2.ImageService`

原因：
- [https://github.com/containerd/containerd/issues/4581](https://github.com/containerd/containerd/issues/4581)

因此，我们对`containerd` 配置模板进行修改删掉仓库相关配置，内容如下：

```bash
修改 关于 config.toml 的ansible-playbook 模板 

```bash
cat roles/container-engine/containerd/templates/config.toml.j2
version = 2
root = "{{ containerd_storage_dir }}"
state = "{{ containerd_state_dir }}"
oom_score = {{ containerd_oom_score }}

[grpc]
  max_recv_message_size = {{ containerd_grpc_max_recv_message_size | default(16777216) }}
  max_send_message_size = {{ containerd_grpc_max_send_message_size | default(16777216) }}

[debug]
  level = "{{ containerd_debug_level | default('info') }}"

[metrics]
  address = "{{ containerd_metrics_address | default('') }}"
  grpc_histogram = {{ containerd_metrics_grpc_histogram | default(false) | lower }}

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    sandbox_image = "{{ pod_infra_image_repo }}:{{ pod_infra_image_tag }}"
    max_container_log_line_size = {{ containerd_max_container_log_line_size }}
    enable_unprivileged_ports = {{ containerd_enable_unprivileged_ports | default(false) | lower }}
    enable_unprivileged_icmp = {{ containerd_enable_unprivileged_icmp | default(false) | lower }}
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "{{ containerd_default_runtime | default('runc') }}"
      snapshotter = "{{ containerd_snapshotter | default('overlayfs') }}"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
{% for runtime in [containerd_runc_runtime] + containerd_additional_runtimes %}
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.{{ runtime.name }}]
          runtime_type = "{{ runtime.type }}"
          runtime_engine = "{{ runtime.engine }}"
          runtime_root = "{{ runtime.root }}"
{% if runtime.base_runtime_spec is defined %}
          base_runtime_spec = "{{ containerd_cfg_dir }}/{{ runtime.base_runtime_spec }}"
{% endif %}

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.{{ runtime.name }}.options]
{% for key, value in runtime.options.items() %}
            {{ key }} = {{ value }}
{% endfor %}
{% endfor %}
{% if kata_containers_enabled %}
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata-qemu]
          runtime_type = "io.containerd.kata-qemu.v2"
{% endif %}
{% if gvisor_enabled %}
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
          runtime_type = "io.containerd.runsc.v1"
{% endif %}
    [plugins."io.containerd.grpc.v1.cri".registry]
{% if containerd_insecure_registries is defined and containerd_insecure_registries|length>0 %}
      config_path = "{{ containerd_cfg_dir }}/certs.d"
{% endif %}

{% if containerd_extra_args is defined %}
{{ containerd_extra_args }}
{% endif %}
```
然后执行重新部署：
```bash
ansible-playbook -i inventory/local/inventory.ini --become --become-user=root reset.yml
ansible-playbook -i inventory/local/inventory.ini --become --become-user=root cluster.yml 
```

## 4. reslov.conf 权限无法修改


```bash
K [kubernetes/preinstall : Add domain/search/nameservers/options to resolv.conf] *********************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: OSError: [Errno 1] Operation not permitted
fatal: [kube-control-plan01]: FAILED! => {"changed": false, "msg": "Unable to make /root/.ansible/tmp/ansible-tmp-1681283032.2323463-3779-108831243697784/tmpCSlNgC into to /etc/resolv.conf, failed final rename from /etc/.ansible_tmpRVelRDresolv.conf: [Errno 1] Operation not permitted"}
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: OSError: [Errno 1] Operation not permitted
```
执行：

```bash
t@c925b21ac60e:/kubespray/outputs/kubespray-2.21.0# ansible -i inventory/local/inventory.ini all -m shell -a "lsattr /etc/resolv.conf"
[WARNING]: Skipping callback plugin 'ara_default', unable to load
kube-node01 | CHANGED | rc=0 >>
----i--------e-- /etc/resolv.conf
kube-control-plan01 | CHANGED | rc=0 >>
-------------e-- /etc/resolv.conf
kube-node02 | CHANGED | rc=0 >>
----i--------e-- /etc/resolv.conf
kube-node03 | CHANGED | rc=0 >>
----i--------e-- /etc/resolv.conf
dbscale-control-plan01 | CHANGED | rc=0 >>
----i--------e-- /etc/resolv.conf
root@c925b21ac60e:/kubespray/outputs/kubespray-2.21.0# ansible -i inventory/local/inventory.ini all -m shell -a "chattr -i /etc/resolv.conf"
[WARNING]: Skipping callback plugin 'ara_default', unable to load
kube-control-plan01 | CHANGED | rc=0 >>

kube-node02 | CHANGED | rc=0 >>

kube-node03 | CHANGED | rc=0 >>

kube-node01 | CHANGED | rc=0 >>

dbscale-control-plan01 | CHANGED | rc=0 >>

root@c925b21ac60e:/kubespray/outputs/kubespray-2.21.0# ansible -i inventory/local/inventory.ini all -m shell -a "lsattr /etc/resolv.conf"
[WARNING]: Skipping callback plugin 'ara_default', unable to load
kube-node03 | CHANGED | rc=0 >>
-------------e-- /etc/resolv.conf
kube-control-plan01 | CHANGED | rc=0 >>
-------------e-- /etc/resolv.conf
kube-node01 | CHANGED | rc=0 >>
-------------e-- /etc/resolv.conf
kube-node02 | CHANGED | rc=0 >>
-------------e-- /etc/resolv.conf
dbscale-control-plan01 | CHANGED | rc=0 >>
-------------e-- /etc/resolv.conf


```

## 5. install package failed

```bash
TASK [kubernetes/preinstall : Install packages requirements] ********************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"attempts": 4, "changed": false, "changes": {"installed": ["conntrack", "container-selinux", "socat", "ipvsadm"]}, "msg": "libnetfilter_queue-1.0.2-2.el7_2.x86_64 was supposed to be installed but is not!\nlibselinux-2.5-15.el7.x86_64 was supposed to be installed but is not!\n2:container-selinux-2.119.2-1.911c772.el7_8.noarch was supposed to be installed but is not!\npolicycoreutils-python-2.5-34.el7.x86_64 was supposed to be installed but is not!\nipvsadm-1.27-8.el7.x86_64 was supposed to be installed but is not!\nselinux-policy-targeted-3.13.1-268.el7_9.2.noarch was supposed to be installed but is not!\nconntrack-tools-1.4.4-7.el7.x86_64 was supposed to be installed but is not!\npolicycoreutils-2.5-34.el7.x86_64 was supposed to be installed but is not!\nlibselinux-utils-2.5-15.el7.x86_64 was supposed to be installed but is not!\nlibnetfilter_cttimeout-1.0.0-7.el7.x86_64 was supposed to be installed but is not!\nsetools-libs-3.3.8-4.el7.x86_64 was supposed to be installed but is not!\nlibsemanage-python-2.5-14.el7.x86_64 was supposed to be installed but is not!\nlibsemanage-2.5-14.el7.x86_64 was supposed to be installed but is not!\nlibselinux-python-2.5-15.el7.x86_64 was supposed to be installed but is not!\nlibsepol-2.5-10.el7.x86_64 was supposed to be installed but is not!\nselinux-policy-3.13.1-268.el7_9.2.noarch was supposed to be installed but is not!\nlibnetfilter_cthelper-1.0.0-11.el7.x86_64 was supposed to be installed but is not!\nsocat-1.7.3.2-2.el7.x86_64 was supposed to be installed but is not!\nlibsemanage-python-2.5-11.el7.x86_64 was supposed to be removed but is not!\nlibsemanage-2.5-11.el7.x86_64 was supposed to be removed but is not!\nlibselinux-python-2.5-12.el7.x86_64 was supposed to be removed but is not!\nsetools-libs-3.3.8-2.el7.x86_64 was supposed to be removed but is not!\npolicycoreutils-2.5-22.el7.x86_64 was supposed to be removed but is not!\nlibsepol-2.5-8.1.el7.x86_64 was supposed to be removed but is not!\npolicycoreutils-python-2.5-22.el7.x86_64 was supposed to be removed but is 
```
解决方法：
我们可以利用镜像自带的 yum源提供支持，我们需要以下配置：

```bash
mkdir /root/kubespray-offline-2.21.0-0/outputs/rpms/cdrom
mount -t iso9660 /dev/cdrom /root/kubespray-offline-2.21.0-0/outputs/rpms/cdrom
```
查看

```bash
$ df -Th |grep cdrom
/dev/sr0            iso9660    10G   10G     0 100% /mnt/cdrom
$ ls /mnt/cdrom/
AppStream  BaseOS  EFI  images  isolinux  LICENSE  media.repo  TRANS.TBL
```

持久挂载
```bash
$ vim /etc/fstab
/dev/sr0 /mnt/cdrom iso9660 defaults 0 0
```
本地配置yum源，在`offline.repo`添加：

```bash
$ cat /etc/yum.repos.d/offline.repo
[offline-repo]
name=Offline repo
baseurl=http://localhost/rpms/local/
enabled=1
gpgcheck=0

[BaseOS]
name=BaseOS
baseurl=http://localhost/rpms/cdrom/BaseOS
enable=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://localhost/rpms/cdrom/AppStream
enabled=1
gpgcheck=0
```
其他集群节点 yum 源配置：

```bash
 cat /etc/yum.repos.d/offline.repo
[offline-repo]
async = 1
baseurl = http://100.168.110.199/rpms/local
enabled = 1
gpgcheck = 0
name = Offline repo for kubespray


[BaseOS]
name=BaseOS
baseurl=http://100.168.110.199/rpms/cdrom/BaseOS
enable=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://100.168.110.199/rpms/cdrom/AppStream
enabled=1
gpgcheck=0
```
验证：

```bash
$ yum clean all
$ yum repolist --verbose
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, needs-restarting, playground, repoclosure, repodiff, repograph, repomanage, reposync, versionlock
YUM version: 4.7.0
cachedir: /var/cache/dnf
Last metadata expiration check: 1:36:06 ago on Wed 19 Apr 2023 01:59:27 PM CST.
Repo-id            : AppStream
Repo-name          : AppStream
Repo-revision      : 8.5
Repo-distro-tags      : [cpe:/o:rocky:rocky:8]:  ,  , 8, L, R, c, i, k, n, o, u, x, y
Repo-updated       : Sun 14 Nov 2021 05:25:39 PM CST
Repo-pkgs          : 6,163
Repo-available-pkgs: 5,279
Repo-size          : 8.0 G
Repo-baseurl       : http://100.168.110.199/rpms/cdrom/AppStream
Repo-expire        : 172,800 second(s) (last: Tue 18 Apr 2023 10:24:35 PM CST)
Repo-filename      : /etc/yum.repos.d/offline.repo

Repo-id            : BaseOS
Repo-name          : BaseOS
Repo-revision      : 8.5
Repo-distro-tags      : [cpe:/o:rocky:rocky:8]:  ,  , 8, L, R, c, i, k, n, o, u, x, y
Repo-updated       : Sun 14 Nov 2021 05:23:13 PM CST
Repo-pkgs          : 1,708
Repo-available-pkgs: 1,706
Repo-size          : 1.2 G
Repo-baseurl       : http://100.168.110.199/rpms/cdrom/BaseOS
Repo-expire        : 172,800 second(s) (last: Tue 18 Apr 2023 10:24:35 PM CST)
Repo-filename      : /etc/yum.repos.d/offline.repo

Repo-id            : offline-repo
Repo-name          : Offline repo for kubespray
Repo-revision      : 1681789393
Repo-updated       : Tue 18 Apr 2023 11:43:14 AM CST
Repo-pkgs          : 265
Repo-available-pkgs: 262
Repo-size          : 182 M
Repo-baseurl       : http://100.168.110.199/rpms/local
Repo-expire        : 172,800 second(s) (last: Wed 19 Apr 2023 01:59:27 PM CST)
Repo-filename      : /etc/yum.repos.d/offline.repo
Total packages: 8,136
```


##  6. TASK [container-engine/containerd : containerd ｜ Write hosts.toml file] 

```bash
TASK [container-engine/containerd : containerd ｜ Write hosts.toml file] ***********************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'ansible.utils.unsafe_proxy.AnsibleUnsafeText object' has no attribute 'key'\n\nThe error appears to be in '/root/kubespray-offline-2.21.0/outputs/kubespray-2.21.0/roles/container-engine/containerd/tasks/main.yml': line 123, column 3, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n- name: containerd ｜ Write hosts.toml file\n  ^ here\n"}

```

解决方法：
修改 `with_items` 为 `with_dict`
```bash
$ vi roles/container-engine/containerd/tasks/main.yml
- name: containerd ｜ Create registry directories
  file:
    path: "{{ containerd_cfg_dir }}/certs.d/{{ item.key }}"
    state: directory
    mode: 0755
    recurse: true
  with_dict: "{{ containerd_insecure_registries }}"
  when: containerd_insecure_registries is defined
```


或者手动将分发脚本，并将上面 `ansible-playbook` 代码内容注释。
```bash
$ cat host.toml
server = "100.168.110.199:35000"
[host."100.168.110.199:35000"]
  capabilities = ["pull", "resolve", "push"]
  skip_verify = true
```
执行：
```bash
ansible -i inventory/local/inventory.ini all -m copy -a "src=./host.toml dest=/etc/containerd/certs.d/100.168.110.199:35000/"
```


##  7. TASK [kubernetes/node : Modprobe nf_conntrack_ipv4] 

```bash
TASK [kubernetes/node : Modprobe nf_conntrack_ipv4] ********************************************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-425.13.1.el8_7.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring

```

手动批量执行：

```bash
ansible -i inventory/local/inventory.ini all -m shell -a "lsmod | grep conntrack"
ansible -i inventory/local/inventory.ini all -m shell -a "modprobe ip_conntrack"
ansible -i inventory/local/inventory.ini all -m shell -a "modprobe br_netfilter"
ansible -i inventory/local/inventory.ini all -m shell -a "sysctl -p"
```

> 说明：`nf_conntrack_ipv4` havs been rename to `nf_conntrack` since Linux kernel `4.18+`


## 8 可忽略报错

### 8.1 TASK [network_plugin/calico : Calico | Get existing calico network pool] 

```bash

TASK [network_plugin/calico : Calico | Get existing calico network pool] ***********************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"changed": false, "cmd": ["/usr/local/bin/calicoctl.sh", "get", "ippool", "default-pool", "-o", "json"], "delta": "0:00:00.022302", "end": "2023-04-18 18:22:15.399574", "msg": "non-zero return code", "rc": 1, "start": "2023-04-18 18:22:15.377272", "stderr": "resource does not exist: IPPool(default-pool) with error: ippools.crd.projectcalico.org \"default-pool\" not found", "stderr_lines": ["resource does not exist: IPPool(default-pool) with error: ippools.crd.projectcalico.org \"default-pool\" not found"], "stdout": "null", "stdout_lines": ["null"]}
...ignoring
Tuesday 18 April 2023  06:22:15 -0400 (0:00:00.295)       0:18:10.590 *********

TASK [network_plugin/calico : Calico | Set kubespray calico network pool] **********************************************************************************************************
ok: [kube-control-plan01]
Tuesday 18 April 2023  06:22:15 -0400 (0:00:00.066)       0:18:10.656 *********
Tuesday 18 April 2023  06:22:15 -0400 (0:00:00.069)       0:18:10.726 *********

TASK [network_plugin/calico : Calico | Configure calico network pool] **************************************************************************************************************
ok: [kube-control-plan01]
Tuesday 18 April 2023  06:22:15 -0400 (0:00:00.333)       0:18:11.060 *********
Tuesday 18 April 2023  06:22:15 -0400 (0:00:00.065)       0:18:11.125 *********
Tuesday 18 April 2023  06:22:15 -0400 (0:00:00.072)       0:18:11.197 *********
Tuesday 18 April 2023  06:22:15 -0400 (0:00:00.067)       0:18:11.264 *********
Tuesday 18 April 2023  06:22:16 -0400 (0:00:00.066)       0:18:11.331 *********
Tuesday 18 April 2023  06:22:16 -0400 (0:00:00.018)       0:18:11.350 *********
Tuesday 18 April 2023  06:22:16 -0400 (0:00:00.015)       0:18:11.366 *********
Tuesday 18 April 2023  06:22:16 -0400 (0:00:00.023)       0:18:11.389 *********

TASK [network_plugin/calico : Calico | Get existing BGP Configuration] *************************************************************************************************************
fatal: [kube-control-plan01]: FAILED! => {"changed": false, "cmd": ["/usr/local/bin/calicoctl.sh", "get", "bgpconfig", "default", "-o", "json"], "delta": "0:00:00.024893", "end": "2023-04-18 18:22:16.520670", "msg": "non-zero return code", "rc": 1, "start": "2023-04-18 18:22:16.495777", "stderr": "resource does not exist: BGPConfiguration(default) with error: bgpconfigurations.crd.projectcalico.org \"default\" not found", "stderr_lines": ["resource does not exist: BGPConfiguration(default) with error: bgpconfigurations.crd.projectcalico.org \"default\" not found"], "stdout": "null", "stdout_lines": ["null"]}
...ignoring
```


### 8.2 TASK [kubernetes-apps/ansible : Kubernetes Apps | Register coredns deployment annotation `createdby`] 

```bash
TASK [kubernetes-apps/ansible : Kubernetes Apps | Register coredns deployment annotation `createdby`] ******************************************************************************fatal: [kube-control-plan01]: FAILED! => {"changed": false, "cmd": ["/usr/local/bin/kubectl", "--kubeconfig", "/etc/kubernetes/admin.conf", "get", "deploy", "-n", "kube-system", "coredns", "-o", "jsonpath={ .spec.template.metadata.annotations.createdby }"], "delta": "0:00:00.036241", "end": "2023-04-18 18:23:08.117748", "msg": "non-zero return code", "rc": 1, "start": "2023-04-18 18:23:08.081507", "stderr": "Error from server (NotFound): deployments.apps \"coredns\" not found", "stderr_lines": ["Error from server (NotFound): deployments.apps \"coredns\" not found"], "stdout": "", "stdout_lines": []}
...ignoring
Tuesday 18 April 2023  06:23:07 -0400 (0:00:00.323)       0:19:03.310 *********

TASK [kubernetes-apps/ansible : Kubernetes Apps | Register coredns service annotation `createdby`] *********************************************************************************fatal: [kube-control-plan01]: FAILED! => {"changed": false, "cmd": ["/usr/local/bin/kubectl", "--kubeconfig", "/etc/kubernetes/admin.conf", "get", "svc", "-n", "kube-system", "coredns", "-o", "jsonpath={ .metadata.annotations.createdby }"], "delta": "0:00:00.038043", "end": "2023-04-18 18:23:08.443334", "msg": "non-zero return code", "rc": 1, "start": "2023-04-18 18:23:08.405291", "stderr": "Error from server (NotFound): services \"coredns\" not found", "stderr_lines": ["Error from server (NotFound): services \"coredns\" not found"], "stdout": "", "stdout_lines": []}
...ignoring
```

参考：
- github：[https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- 官网：[https://kubespray.io/#/](https://kubespray.io/#/)
- 网友kubespray 学习：[https://github.com/wenwenxiong/book/tree/master/k8s/kubespray](https://github.com/wenwenxiong/book/tree/master/k8s/kubespray)
