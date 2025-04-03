![](https://i-blog.csdnimg.cn/blog_migrate/b8fa25491bef0852db2b454dd51ea438.png)



## 1. 系统兼容
| Operating System | Containerd 1.4 - 1.7 | 
|--|--|
|Ubuntu 20.04 LTS   |  Yes | 
|Ubuntu 22.04 LTS    |  Yes | 
|CentOS 7.9   |  Yes |  No|
|Red Hat Core OS (RHCOS)   |  No | 
|Red Hat Enterprise Linux 8.6, 8.7, 8.8   |  Yes | 


## 2. 下载

软件版本
| 软件         | 版本     |
|------------|--------|
| runc       | 1.1.9  |
| containerd | 1.7.5  |
| nerdctl    | 1.5.0  |
| crictl     | 1.28.0 |
| cni-plugin | 1.3.0  |



此脚本包含下载与安装。

- sh install-containerd.sh d    #下载
- sh install-containerd.sh download    #下载
- sh install-containerd.sh i    #安装
- sh install-containerd.sh install    #安装
```bash
#!/bin/bash


name=`basename $0 .sh`
ENABLE_DOWNLOAD=${ENABLE_DOWNLOAD:-true}
BASE_DIR="$( dirname "$( readlink -f "${0}" )" )"

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
    CRICTL_VERSION=1.28.0
    CNI_VERSION=1.3.0

    download https://github.com/opencontainers/runc/releases/download/v${RUNC_VERSION}/runc.amd64 runc/v${RUNC_VERSION}
    download https://github.com/containerd/containerd/releases/download/v${CONTAINERD_VERSION}/containerd-${CONTAINERD_VERSION}-linux-amd64.tar.gz
    download https://github.com/containerd/nerdctl/releases/download/v${NERDCTL_VERSION}/nerdctl-${NERDCTL_VERSION}-linux-amd64.tar.gz
    download https://github.com/kubernetes-sigs/cri-tools/releases/download/v${CRICTL_VERSION}/crictl-v${CRICTL_VERSION}-linux-amd64.tar.gz
    download https://github.com/containernetworking/plugins/releases/download/v${CNI_VERSION}/cni-plugins-linux-amd64-v${CNI_VERSION}.tgz kubernetes/cni

else
    FILES_DIR=./files
fi

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


echo ""
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

cat > /etc/systemd/system/containerd.service <<EOF
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



cat > /etc/containerd/config.toml <<EOF
version = 2
root = "/var/lib/containerd"
state = "/run/containerd"
oom_score = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  uid = 0
  gid = 0

[debug]
  address = "/run/containerd/debug.sock"
  uid = 0
  gid = 0
  level = "info"

[metrics]
  address = ""
  grpc_histogram = false

[cgroup]
  path = ""

[plugins]
  [plugins."io.containerd.grpc.v1.cri".containerd]
    default_runtime_name = "runc"
    snapshotter = "overlayfs"
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    runtime_type = "io.containerd.runc.v2"
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    systemdCgroup = true
EOF


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
下载介质

```bash
sh install-containerd.sh d
```
输出：

```bash
==> download https://github.com/opencontainers/runc/releases/download/v1.1.9/runc.amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 10.1M  100 10.1M    0     0  1777k      0  0:00:05  0:00:05 --:--:-- 2389k
==> download https://github.com/containerd/containerd/releases/download/v1.7.5/containerd-1.7.5-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 44.6M  100 44.6M    0     0  12.2M      0  0:00:03  0:00:03 --:--:-- 29.5M
==> download https://github.com/containerd/nerdctl/releases/download/v1.5.0/nerdctl-1.5.0-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 9026k  100 9026k    0     0  3508k      0  0:00:02  0:00:02 --:--:-- 6374k
==> download https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.28.0/crictl-v1.28.0-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 22.7M  100 22.7M    0     0  8202k      0  0:00:02  0:00:02 --:--:-- 15.1M
==> download https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-amd64-v1.3.0.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 43.2M  100 43.2M    0     0  11.8M      0  0:00:03  0:00:03 --:--:-- 21.2M
```

介质结构

```bash
$ tree
.
├── files
│   ├── containerd-1.7.5-linux-amd64.tar.gz
│   ├── crictl-v1.27.1-linux-amd64.tar.gz
│   ├── crictl-v1.28.0-linux-amd64.tar.gz
│   ├── kubernetes
│   │   └── cni
│   │       └── cni-plugins-linux-amd64-v1.3.0.tgz
│   ├── nerdctl-1.5.0-linux-amd64.tar.gz
│   └── runc
│       └── v1.1.9
│           └── runc.amd64
└── install-containerd.sh

5 directories, 7 files
```

## 3. 安装
拷贝离线机器节点安装 

```bash
sh install-containerd.sh i
```
输出：

```bash
==> Install nerdctl
nerdctl
containerd-rootless-setuptool.sh
containerd-rootless.sh
==> Install crictl plugins
crictl
==> Install runc
==> Install containerd

overlay
br_netfilter
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
* Applying /usr/lib/sysctl.d/00-system.conf ...
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-call-arptables = 0
* Applying /usr/lib/sysctl.d/10-default-yama-scope.conf ...
kernel.yama.ptrace_scope = 0
* Applying /usr/lib/sysctl.d/50-default.conf ...
kernel.sysrq = 16
kernel.core_uses_pid = 1
kernel.kptr_restrict = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.all.promote_secondaries = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/99-kubernetes-cri.conf ...
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
* Applying /etc/sysctl.d/99-sysctl.conf ...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
kernel.pid_max = 99999
vm.max_map_count = 262144
* Applying /etc/sysctl.conf ...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
kernel.pid_max = 99999
vm.max_map_count = 262144
bin/ctr
bin/containerd-stress
bin/containerd
bin/containerd-shim
bin/containerd-shim-runc-v1
bin/containerd-shim-runc-v2
==> Start containerd
Created symlink from /etc/systemd/system/multi-user.target.wants/containerd.service to /etc/systemd/system/containerd.service.
   Active: active (running) since Thu 2024-02-29 11:12:33 CST; 175ms ago
==> Install CNI plugins
./
./loopback
./bandwidth
./ptp
./vlan
./host-device
./tuning
./vrf
./sbr
./tap
./dhcp
./static
./firewall
./macvlan
./dummy
./bridge
./ipvlan
./portmap
./host-local
[root@kube-master01 nerdctl]# cat download.sh 
#!/bin/bash
```

## 4. 检查
检查 containerd 服务状态

```bash
$ systemctl status containerd.service 
● containerd.service - containerd container runtime
   Loaded: loaded (/etc/systemd/system/containerd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2023-12-07 15:29:13 CST; 1h 1min ago
     Docs: https://containerd.io
  Process: 2069 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
 Main PID: 2073 (containerd)
    Tasks: 23
   Memory: 304.1M
   CGroup: /system.slice/containerd.service
           ├─2073 /usr/local/bin/containerd
           └─2733 /usr/local/bin/containerd-shim-runc-v2 -namespace default -id nginx1 -address /run/containerd/containerd.sock

Dec 07 16:05:16 localhost.localdomain containerd[2073]: time="2023-12-07T16:05:16.070075409+08:00" level=info msg="loading plugin \"io.containerd.ttrpc.v1.pause\"..." runtime=i...rd.ttrpc.v1
Dec 07 16:05:16 localhost.localdomain containerd[2073]: time="2023-12-07T16:05:16.070225781+08:00" level=info msg="loading plugin \"io.containerd.event.v1.publisher\"..." runti...rd.event.v1
Dec 07 16:05:16 localhost.localdomain containerd[2073]: time="2023-12-07T16:05:16.070487356+08:00" level=info msg="loading plugin \"io.containerd.ttrpc.v1.task\"..." runtime=io...rd.ttrpc.v1
Dec 07 16:05:25 localhost.localdomain containerd[2073]: time="2023-12-07T16:05:25.314451770+08:00" level=info msg="shim disconnected" id=nginx1 namespace=default
Dec 07 16:05:25 localhost.localdomain containerd[2073]: time="2023-12-07T16:05:25.314619808+08:00" level=warning msg="cleaning up after shim disconnected" id=nginx1 namespace=default
Dec 07 16:05:25 localhost.localdomain containerd[2073]: time="2023-12-07T16:05:25.314660589+08:00" level=info msg="cleaning up dead shim" namespace=default
Dec 07 16:07:07 localhost.localdomain containerd[2073]: time="2023-12-07T16:07:07.682100420+08:00" level=info msg="loading plugin \"io.containerd.internal.v1.shutdown\"..." run...internal.v1
Dec 07 16:07:07 localhost.localdomain containerd[2073]: time="2023-12-07T16:07:07.682320004+08:00" level=info msg="loading plugin \"io.containerd.ttrpc.v1.pause\"..." runtime=i...rd.ttrpc.v1
Dec 07 16:07:07 localhost.localdomain containerd[2073]: time="2023-12-07T16:07:07.682429251+08:00" level=info msg="loading plugin \"io.containerd.event.v1.publisher\"..." runti...rd.event.v1
Dec 07 16:07:07 localhost.localdomain containerd[2073]: time="2023-12-07T16:07:07.682477794+08:00" level=info msg="loading plugin \"io.containerd.ttrpc.v1.task\"..." runtime=io...rd.ttrpc.v1
Hint: Some lines were ellipsized, use -l to show in full.
```

检查ctr & crictl $ nerdctl & containerd命令是否安装成功。

```bash
$ containerd --version
containerd github.com/containerd/containerd v1.7.5 fe457eb99ac0e27b3ce638175ef8e68a7d2bc373

$ crictl version
Version:  0.1.0
RuntimeName:  containerd
RuntimeVersion:  v1.7.5
RuntimeApiVersion:  v1

$ nerdctl version
WARN[0000] unable to determine buildctl version: exec: "buildctl": executable file not found in $PATH 
Client:
 Version:       v1.5.0
 OS/Arch:       linux/amd64
 Git commit:    b33a58f288bc42351404a016e694190b897cd252
 buildctl:
  Version:

Server:
 containerd:
  Version:      v1.7.5
  GitCommit:    fe457eb99ac0e27b3ce638175ef8e68a7d2bc373
 runc:
  Version:      1.1.9
  GitCommit:    v1.1.9-0-gccaecfcb

$ ctr version
Client:
  Version:  v1.7.5
  Revision: fe457eb99ac0e27b3ce638175ef8e68a7d2bc373
  Go version: go1.20.7

Server:
  Version:  v1.7.5
  Revision: fe457eb99ac0e27b3ce638175ef8e68a7d2bc373
  UUID: 86b0073d-e329-4184-b319-e4c965369e7a
```

## 5. 测试

创建镜像仓库容器
```bash
$ vim  start-registry.sh
#!/bin/bash


REGISTRY_IMAGE=${REGISTRY_IMAGE:-registry:2.8.2}
REGISTRY_DIR=${REGISTRY_DIR:-/var/lib/registry}
REGISTRY_PORT=${REGISTRY_PORT:-35000}

if [ ! -e $REGISTRY_DIR ]; then
    sudo mkdir $REGISTRY_DIR
fi

echo "===> Start registry"
sudo /usr/local/bin/nerdctl --insecure-registry=true  run -d \
    --network host \
    -e REGISTRY_HTTP_ADDR=0.0.0.0:${REGISTRY_PORT} \
    --restart always \
    --name registry \
    -v $REGISTRY_DIR:/var/lib/registry \
    $REGISTRY_IMAGE


$ sh start-registry.sh 
```

配置 `/etc/hosts`

```bash
$ cat /etc/hosts
10.70.0.73 registry.demo
```

推送镜像入库
```bash
nerdctl pull  registry:2.8.2
nerdctl tag registry:2.8.2 registry.demo:35000/registry:2.8.2
nerdctl --insecure-registry=true push  registry.demo:35000/registry:2.8.2
nerdctl --insecure-registry=true pull  registry.demo:35000/registry:2.8.2
```


参考：

-  [https://github.com/opencontainers/runc/releases](https://github.com/opencontainers/runc/releases)
- [https://github.com/containerd/containerd](https://github.com/containerd/containerd)
- [https://github.com/containerd/nerdctl](https://github.com/containerd/nerdctl)
- [https://github.com/kubernetes-sigs/cri-tools](https://github.com/kubernetes-sigs/cri-tools)
- [https://github.com/containernetworking](https://github.com/containernetworking)
