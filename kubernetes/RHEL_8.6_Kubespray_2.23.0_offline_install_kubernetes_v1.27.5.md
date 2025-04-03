![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/378bc94195b603d673515575f028215b.png)



## 1. 预备条件
9 个节点：

| 角色 | ip  | cpu| mem| disk|ios|kernel|
|--|--|--|--|--|--|--|
| download |10.70.0.70、192.168.23.70   |4  |8G|100G|rhel 8.6|4.18+|
| bastion01 |10.70.0.78  |4  |8G|100G|rhel 8.6|4.18+|
| harbor01 |10.70.0.77  |4  |8G|100G|rhel 8.6|4.18+|
| kube-master01 |10.70.0.71  |4  |8G|100G|rhel 8.6|4.18+|
| kube-master02 |10.70.0.72  |4  |8G|100G|rhel 8.6|4.18+|
| kube-master03 |10.70.0.73  |4  |8G|100G|rhel 8.6|4.18+|
| kube-node01 |10.70.0.74  |4  |8G|100G|rhel 8.6|4.18+|
| kube-node02 |10.70.0.75  |4  |8G|100G|rhel 8.6|4.18+|
| kube-node03 |10.70.0.76  |4  |8G|100G|rhel 8.6|4.18+|


rhel-8.6-x86_64-dvd.iso
百度云：[https://pan.baidu.com/s/1uEdrMtFhEcJTNZpFP-lRyg?pwd=g56s](https://pan.baidu.com/s/1uEdrMtFhEcJTNZpFP-lRyg?pwd=g56s) ，提取码：g56s 
## 2. download01 节点 安装 docker
（download）

- 配置非安全私有仓库：[https://williamlieurance.com/insecure-podman-registry/](https://williamlieurance.com/insecure-podman-registry/)
- 配置代理：[https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-ConfiguringNetworkingforPodman.html#podman-install-proxy](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-ConfiguringNetworkingforPodman.html#podman-install-proxy)

```bash
yum -y install docker
systemctl start podman && systemctl enable podman
```



## 3. download01 节点 介质下载
（download）
介质列表：

- rhel-8.6-x86_64-dvd.iso （用于安装系统和配置yum源）
- kubespray-offline-2.23.0-0.tar.gz（离线安装 kubernetes）


下载最新版本 2.23.0-0： [https://github.com/kubespray-offline/kubespray-offline/releases](https://github.com/kubespray-offline/kubespray-offline/releases)

```bash
wget https://github.com/kubespray-offline/kubespray-offline/archive/refs/tags/v2.23.0-0.zip
 unzip v2.23.0-0.zip
```
在下载介质之前需要安装镜像下载工具：
- run `install-docker.sh` to install `Docker CE`.
- run `install-containerd.sh` to install `containerd` and `nerdctl.`
- Set docker environment variable to `/usr/local/bin/nerdctl` in config.sh.

```bash
./install-docker.sh
./install-containerd.sh
```
我们也已经通过yum 安装 docker（podman），因为是rhel 8.7，仅支持podman或者nerdctl。这里选择podman，同样也可以选择 [nerdctl install](https://blog.csdn.net/xixihahalelehehe/article/details/134264754) ，这取决于习惯。

在下载所有介质之前需要配置代理否则，是否无法下载成功的。
临时配置主机代理：

```bash
export https_proxy=http://192.168.21.101:7890
export http_proxy=http://192.168.21.101:7890
export no_proxy=192.168.21.2,10.0.0.0/8,192.168.0.0/16,localhost,127.0.0.0/8,.coding.net,.tencentyun.com,.myqcloud.com
```
永久配置主机代理

```bash
$ vim /root/.bashrc
ENV_PROXY="http://192.168.21.101:7890"

enable_proxy() {
    export HTTP_PROXY="$ENV_PROXY"
    export HTTPS_PROXY="$ENV_PROXY"
    export ALL_PROXY="$ENV_PROXY"
    export NO_PROXY="proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

    export http_proxy="$ENV_PROXY"
    export https_proxy="$ENV_PROXY"
    export all_proxy="$ENV_PROXY"
    export no_proxy="proxyhost,localhost,*.vsphere.local,*.vm.demo,*.tanzu.demo,192.168.21.101,127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"

    #git config --global http.proxy $ENV_PROXY
    #git config --global http.proxy $ENV_PROXY
}



disable_proxy() {
    unset HTTP_PROXY
    unset HTTPS_PROXY
    unset ALL_PROXY
    unset NO_PROXY

    unset http_proxy
    unset https_proxy
    unset all_proxy
    unset no_proxy

    #git config --global --unset http.proxy
    #git config --global --unset https.proxy
}

#source <(kubectl completion bash)

enable_proxy
#disable_proxy

$ source /root/.bashrc
```


 容器代理：
 

```bash
cat /etc/systemd/system/podman.service.d/http-proxy.conf 
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890"
Environment="HTTPS_PROXY=http://192.168.21.101:7890"
Environment="NO_PROXY=localhost,127.0.0.1,.coding.net,.tencentyun.com,.myqcloud.com"

```
重启podman
```bash
systemctl daemon-reload && systemctl restart podman
```

安装成功后，下载介质执行：

```bash
./download-all.sh
```
所有工件都存储在./outputs 目录。
此脚本调用以下所有脚本：
- `prepare-pkgs.sh`
Setup python, etc.
- `prepare-py.sh`
Setup python venv, install required python packages.
- `get-kubespray.sh`
Download and extract kubespray, if KUBESPRAY_DIR does not exist.
- `pypi-mirror.sh`
Download PyPI mirror files
- `download-kubespray-files.sh`
Download kubespray offline files (containers, files, etc)
- `download-additional-containers.sh`
Download additional containers.
You can add any container image repoTag to imagelists/*.txt.
- `create-repo.sh`
Download RPM or DEB repositories.
- `copy-target-scripts.sh`
Copy scripts for target node.


用时 30分钟 下载完成介质。也许一次会下载失败，需要调试反复执行，或者根据环境改动一些内容。
> 注意：如果你喜欢通过容器部署集群，需要下载 kubespray 镜像，默认 `quay.io/kubespray/kubespray:v2.21.0`  ，存在一个 pip 包依赖（jmespath 1.0.1），下面镜像已重新编译。

                              
```bash
nerdctl pull ghostwritten/kubespray:v2.21-a94b893e2
nerdctl save -o kubespray:v2.21-a94b893e2.tar ghostwritten/kubespray:v2.21-a94b893e2
```
或者

```bash
docker pull quay.io/kubespray/kubespray:v2.23.0
docker save -o quay-io-kubespray-image-v2.23.0.tar quay.io/kubespray/kubespray:v2.23.0
```

打包下载的介质：

```bash
tar zcvf kubespray-offline-2.23.0-0.tar.gz kubespray-offline-2.23.0-0
```
将介质传输至离线的 bastion01
## 4. bastion01节点配置 yum 源

（bastion01离线状态）



参考：[https://www.redhat.com/sysadmin/apache-yum-dnf-repo](https://www.redhat.com/sysadmin/apache-yum-dnf-repo)

```bash
mkdir /mnt/cdrom
mount -o loop /root/kubernetes-offline/rhel-8.6-x86_64-dvd.iso /mnt/cdrom
```



本地配置yum源

```bash
cat local.repo 
[BaseOS]
name=BaseOS
baseurl=file:///mnt/cdrom/BaseOS
enable=1
gpgcheck=0

[AppStream]
name=AppStream
baseurl=file:///mnt/cdrom/AppStream
enabled=1
gpgcheck=0

```
测试

```bash
$ yum repolist --verbose
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, kpatch, needs-restarting, playground, product-id, repoclosure, repodiff, repograph, repomanage, reposync, subscription-manager, uploadprofile
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered with an entitlement server. You can use subscription-manager to register.

YUM version: 4.7.0
cachedir: /var/cache/dnf
Last metadata expiration check: 0:35:00 ago on Thu 19 Oct 2023 05:45:22 PM CST.
Repo-id            : AppStream
Repo-name          : AppStream
Repo-revision      : 1656402327
Repo-updated       : Tue 28 Jun 2022 03:45:28 PM CST
Repo-pkgs          : 6,609
Repo-available-pkgs: 5,345
Repo-size          : 8.6 G
Repo-baseurl       : file:///mnt/cdrom/AppStream
Repo-expire        : 172,800 second(s) (last: Thu 19 Oct 2023 05:45:22 PM CST)
Repo-filename      : /etc/yum.repos.d/local.repo

Repo-id            : BaseOS
Repo-name          : BaseOS
Repo-revision      : 1656402376
Repo-updated       : Tue 28 Jun 2022 03:46:17 PM CST
Repo-pkgs          : 1,714
Repo-available-pkgs: 1,712
Repo-size          : 1.3 G
Repo-baseurl       : file:///mnt/cdrom/BaseOS
Repo-expire        : 172,800 second(s) (last: Thu 19 Oct 2023 05:45:21 PM CST)
Repo-filename      : /etc/yum.repos.d/local.repo
Total packages: 8,323

```

```bash

yum clean all
yum install ntp
```
其他节点使用源

```bash
$ yum -y install httpd

$ vim /etc/httpd/conf/httpd.conf


listen 81

# vim  /etc/httpd/conf.d/define.conf
<VirtualHost *:81>
        ServerName  www.XXX-ym.com
        DocumentRoot /mnt/cdrom
</VirtualHost>

# vim  /etc/httpd/conf.d/permission.conf
<Directory /mnt/cdrom>
	Require all granted      
</Directory>


$ systemctl  restart  httpd
$ chcon -R --reference=/var/www  /mnt/cdrom  //调整SELinux属性

$ systemctl  restart  httpd

```
远程节点配置yum源

```bash
$ cat http_iso.repo 
[BaseOS] 
name=http_iso 
baseurl=http://10.70.0.254/BaseOS
gpgcheck=0 
enabled=1
[AppStream] 
name=http_iso 
baseurl=http://10.70.0.254/AppStream
gpgcheck=0 
enabled=1

```



## 5. 安装 docker insecure  registry
（bastion01）
- 配置非安全私有仓库：[https://williamlieurance.com/insecure-podman-registry/](https://williamlieurance.com/insecure-podman-registry/)
- 配置代理：[https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-ConfiguringNetworkingforPodman.html#podman-install-proxy](https://docs.oracle.com/en/operating-systems/oracle-linux/podman/podman-ConfiguringNetworkingforPodman.html#podman-install-proxy)

```bash
yum -y install docker
```


打包下载的介质传输到离线环境：

```bash
cd ../
tar zcvf kubespray-offline-2.23.0-0.tar.gz kubespray-offline-2.23.0-0
```
登陆部署节点（registry01），然后在输出目录中运行以下脚本：

```bash
mkdir /root/kubernetes-offline && cd kubespray-offline-2.23.0-0.tar.gz
tar zxvf kubespray-offline-2.23.0-0.tar.gz
cd /root/kubernetes-offline/kubespray-offline-2.23.0-0/outputs
docker load -i images/docker.io_library_registry-2.8.2.tar.gz
```

```bash
mkdir /var/lib/registry
docker run -d --restart=always --name registry -p 35000:5000 -p 443:443 -v /var/lib/registry:/var/lib/registry docker.io/library/registry:2.8.2
docker ps
```
配置域名

```bash
echo "10.60.0.30 registry.demo" >>/etc/hosts
```
配置 docker 

```bash
cat <<EOF> /etc/containers/registries.conf.d/myregistry.conf
[[registry]]
location = "registry.demo:35000"
insecure = true
EOF
docker tag docker.io/library/registry:2.8.2 registry.demo:35000/registry:2.8.2
docker push registry.demo:35000/registry:2.8.2
```

##  6. bastion01 部署 nginx 与 镜像入库

该配置部署的目的主要完成 nginx部署，为了其他节点拉取各类文件、工具包；并且完成镜像推送入库步骤。

```bash
cd /root/kubernetes-offline/kubespray-offline-2.23.0-0/outputs
```

### 6.1 配置 config.sh
修改 `KUBESPRAY_VERSION:-2.22.1`，并且添加`LOCAL_REGISTRY='registry.demo:35000'`。

```bash
$ cat config.sh 
#!/bin/bash
KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-2.23.0}
#KUBESPRAY_VERSION=${KUBESPRAY_VERSION:-master}

LOCAL_REGISTRY='registry.demo:35000'
REGISTRY_PORT=${REGISTRY_PORT:-35000}
```
### 6.2 配置 setup-docker.sh

```bash
$ cp setup-container.sh setup-docker.sh
$ cat setup-container.sh 
#!/bin/bash

# install containerd
#./install-containerd.sh  #注释掉

# Load images
echo "==> Load registry, nginx images"
#NERDCTL=/usr/local/bin/nerdctl  #注释
NERDCTL=/usr/bin/docker  #添加
cd ./images

for f in docker.io_library_registry-*.tar.gz docker.io_library_nginx-*.tar.gz; do
    sudo $NERDCTL load -i $f
done

if [ -f kubespray-offline-container.tar.gz ]; then
    sudo $NERDCTL load -i kubespray-offline-container.tar.gz
fi
```

### 6.3 配置 start-nginx.sh
部署改为 `docker` 运行容器。

```bash
cat start-nginx.sh 
#!/bin/bash

source ./config.sh

BASEDIR="."
if [ ! -d images ] && [ -d ../outputs ]; then
    BASEDIR="../outputs"  # for tests
fi
BASEDIR=$(cd $BASEDIR; pwd)

NGINX_IMAGE=nginx:1.25.2

echo "===> Start nginx"
sudo /usr/bin/docker run -d \
    --network host \
    --restart always \
    --name nginx \
    -v ${BASEDIR}:/usr/share/nginx/html \
    ${NGINX_IMAGE}

```
### 6.4 配置 load-push-all-images.sh 
将`NERDCTL`变量改为`/usr/bin/docker`。

```bash
NERDCTL=/usr/bin/docker
```
### 6.5 配置 set-all.sh
注释掉`run ./setup-container.sh` 与 `run ./start-registry.sh`

```bash
 cat setup-all.sh
#!/bin/bash

run() {
    echo "=> Running: $*"
    $* || {
        echo "Failed in : $*"
        exit 1
    }
}

# prepare
#run ./setup-container.sh
run ./setup-docker.sh

# start web server
run ./start-nginx.sh

# setup local repositories
run ./setup-offline.sh

# setup python
run ./setup-py.sh

# start private registry
#run ./start-registry.sh

# load and push all images to registry
run ./load-push-all-images.sh

```
### 6.6 执行

```bash
set-all.sh
```
输出：

```bash
$ docker ps
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
CONTAINER ID  IMAGE                             COMMAND               CREATED         STATUS             PORTS                    NAMES
63e5ffe05ebb  docker.io/library/registry:2.8.2  /etc/docker/regis...  2 hours ago     Up 2 hours ago     0.0.0.0:35000->5000/tcp  registry
5c2458ea4933  docker.io/library/nginx:1.25.2    nginx -g daemon o...  42 minutes ago  Up 42 minutes ago                           nginx


$ docker images |grep 35000
Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
registry.bsg:35000/nginx                                    1.25.2          bc649bab30d1  8 days ago     191 MB
registry.bsg:35000/registry                                 2.8.2           fb59138d7af1  3 weeks ago    24.6 MB
registry.bsg:35000/library/nginx                            1.25.2-alpine   d571254277f6  3 weeks ago    44.4 MB
registry.bsg:35000/calico/typha                             v3.25.2         11995f5f3ea7  6 weeks ago    64.9 MB
registry.bsg:35000/calico/kube-controllers                  v3.25.2         cb60adca8b5e  6 weeks ago    70.4 MB
registry.bsg:35000/calico/apiserver                         v3.25.2         3c95ef610728  6 weeks ago    82.2 MB
registry.bsg:35000/calico/cni                               v3.25.2         8ea297882254  6 weeks ago    202 MB
registry.bsg:35000/calico/pod2daemon-flexvol                v3.25.2         2ea0861c9c7a  6 weeks ago    14.7 MB
registry.bsg:35000/calico/node                              v3.25.2         ed304c33a22f  6 weeks ago    245 MB
registry.bsg:35000/kube-apiserver                           v1.27.5         b58f4bc39345  8 weeks ago    122 MB
registry.bsg:35000/kube-proxy                               v1.27.5         f249729a2355  8 weeks ago    72.7 MB
registry.bsg:35000/kube-scheduler                           v1.27.5         96c06904875e  8 weeks ago    59.8 MB
registry.bsg:35000/kube-controller-manager                  v1.27.5         ae819fd2a0d7  8 weeks ago    114 MB
registry.bsg:35000/library/haproxy                          2.8.2-alpine    a3c8e99e9327  2 months ago   24.6 MB
registry.bsg:35000/metrics-server/metrics-server            v0.6.4          a608c686bac9  2 months ago   70.3 MB
registry.bsg:35000/ingress-nginx/controller                 v1.8.1          825aff16c20c  3 months ago   286 MB
registry.bsg:35000/cilium/cilium                            v1.13.4         d00a7abfa71a  4 months ago   487 MB
registry.bsg:35000/cilium/operator                          v1.13.4         9dd45d0a36c9  4 months ago   126 MB
registry.bsg:35000/cilium/hubble-relay                      v1.13.4         e901ba48d58c  4 months ago   48.8 MB
registry.bsg:35000/flannel/flannel                          v0.22.0         38c11b8f4aa1  4 months ago   70.9 MB
registry.bsg:35000/kubeovn/kube-ovn                         v1.11.5         212bc9523e78  5 months ago   432 MB
registry.bsg:35000/library/registry                         2.8.1           f5352b75f67e  5 months ago   26.5 MB
registry.bsg:35000/ghcr.io/kube-vip/kube-vip                v0.5.12         1227540e2a61  6 months ago   41.7 MB
registry.bsg:35000/jetstack/cert-manager-controller         v1.11.1         b23ed1d12779  6 months ago   63.6 MB
registry.bsg:35000/jetstack/cert-manager-cainjector         v1.11.1         4b822c353dc8  6 months ago   41.1 MB
registry.bsg:35000/jetstack/cert-manager-webhook            v1.11.1         b22616882946  6 months ago   48.6 MB
registry.bsg:35000/cpa/cluster-proportional-autoscaler      v1.8.8          b6d1a4be0743  6 months ago   39.4 MB
registry.bsg:35000/rancher/local-path-provisioner           v0.0.24         b29384aeb4b1  7 months ago   40.4 MB
registry.bsg:35000/cilium/hubble-ui                         v0.11.0         b555a2c7b3de  7 months ago   53.2 MB
registry.bsg:35000/cilium/hubble-ui-backend                 v0.11.0         0631ce248fa6  7 months ago   48.6 MB
registry.bsg:35000/dns/k8s-dns-node-cache                   1.22.20         ff71cd4ea5ae  7 months ago   69.4 MB
registry.bsg:35000/metallb/controller                       v0.13.9         26952499c302  8 months ago   64.3 MB
registry.bsg:35000/metallb/speaker                          v0.13.9         697605b35935  8 months ago   114 MB
registry.bsg:35000/coredns/coredns                          v1.10.1         ead0a4a53df8  8 months ago   53.6 MB
registry.bsg:35000/coreos/etcd                              v3.5.7          27c4d6f345ac  9 months ago   57.5 MB
registry.bsg:35000/flannel/flannel-cni-plugin               v1.1.2          7a2dcab94698  10 months ago  8.25 MB
registry.bsg:35000/pause                                    3.9             e6f181688397  12 months ago  748 kB
registry.bsg:35000/kubernetesui/dashboard                   v2.7.0          07655ddf2eeb  13 months ago  249 MB
registry.bsg:35000/envoyproxy/envoy                         v1.22.5         e9c4ee2ce720  14 months ago  133 MB
registry.bsg:35000/cloudnativelabs/kube-router              v1.5.1          6080a224cdd1  14 months ago  104 MB
registry.bsg:35000/sig-storage/local-volume-provisioner     v2.5.0          84fe61c6a33a  15 months ago  134 MB
registry.bsg:35000/kubernetesui/metrics-scraper             v1.0.8          115053965e86  16 months ago  43.8 MB
registry.bsg:35000/cilium/certgen                           v0.1.8          a283370c8d83  20 months ago  44.4 MB
registry.bsg:35000/sig-storage/csi-snapshotter              v5.0.0          c5bdb516176e  21 months ago  56.5 MB
registry.bsg:35000/sig-storage/csi-node-driver-registrar    v2.4.0          f45c8a305a0b  23 months ago  21.2 MB
registry.bsg:35000/ghcr.io/k8snetworkplumbingwg/multus-cni  v3.8            c65d3833b509  2 years ago    300 MB
registry.bsg:35000/sig-storage/snapshot-controller          v4.2.1          223fc919ab6a  2 years ago    52.7 MB
registry.bsg:35000/sig-storage/csi-resizer                  v1.3.0          1df30f0e2555  2 years ago    55.4 MB
registry.bsg:35000/k8scloudprovider/cinder-csi-plugin       v1.22.0         e4c74b94269d  2 years ago    219 MB
registry.bsg:35000/sig-storage/csi-provisioner              v3.0.0          fe0f921f3c92  2 years ago    58 MB
registry.bsg:35000/sig-storage/csi-attacher                 v3.3.0          37f46af926da  2 years ago    55 MB
registry.bsg:35000/weaveworks/weave-npc                     2.8.1           7f92d556d4ff  2 years ago    39.7 MB
registry.bsg:35000/weaveworks/weave-kube                    2.8.1           df29c0a4002c  2 years ago    89.8 MB
registry.bsg:35000/amazon/aws-alb-ingress-controller        v1.1.9          4b1d22ffb3c0  3 years ago    40.6 MB
registry.bsg:35000/amazon/aws-ebs-csi-driver                v0.5.0          187fd7ffef67  3 years ago    444 MB
registry.bsg:35000/external_storage/rbd-provisioner         v2.1.1-k8s1.11  9fb54e49f9bf  5 years ago    417 MB
registry.bsg:35000/external_storage/cephfs-provisioner      v2.1.0-k8s1.11  cac658c0a096  5 years ago    413 MB
registry.bsg:35000/mirantis/k8s-netchecker-server           v1.2.2          3fe402881a14  5 years ago    124 MB
registry.bsg:35000/mirantis/k8s-netchecker-agent            v1.2.2          bf9a79a05945  5 years ago    5.86 MB

```
##  7. bastion01 配置互信

```bash
ssh-keygen
for i in {71..78};do ssh-copy-id root@10.70.0.$i;done
```
##  8. 启动容器部署环境

```bash
docker  run --name kubespray-offline-ansible --network=host --rm -itd -v "$(pwd)"/kubespray-offline-2.23.0-0:/kubespray -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.23.0  bash
```

## 9. 部署前准备
检查批量通信、镜像仓库地址域名解析、可能存在缺失的内核加载模块。

```bash
```bash
$ yum -y install ansible*

$ vim /etc/ansible/hosts
[all]
kube-master01 ansible_host=10.70.0.71
kube-master02 ansible_host=10.70.0.72
kube-master03 ansible_host=10.70.0.73
kube-node01 ansible_host=10.70.0.74
kube-node02 ansible_host=10.70.0.75
kube-node03 ansible_host=10.70.0.76


$ ansible all -m ping
$ ansible all -m lineinfile -a "path=/etc/hosts line='10.70.0.78 registry.demo'"
$ ansible all -m lineinfile -a "path=/etc/rc.local line='modprobe br_netfilter\nmodprobe ip_conntrack'"
$ ansible all -m copy -a "src=/etc/sysctl.conf dest=/etc/"
$ ansible all -m shell  -a "cat /etc/sysctl.conf"
```

###  9.1 配置 `extract-kubespray.sh`
保持 `kubespray-v2.22.1` 内容不变，注释掉 `apply patches`内容。

```bash
cd /root/kubernetes-offline/kubespray-offline-2.23.0-0/outputs
vim extract-kubespray.sh
```

```bash
#!/bin/bash

cd $(dirname $0)
CURRENT_DIR=$(pwd)
source ./config.sh

KUBESPRAY_TARBALL=files/kubespray-${KUBESPRAY_VERSION}.tar.gz
DIR=kubespray-${KUBESPRAY_VERSION}

if [ -d $DIR ]; then
    echo "${DIR} already exists."
    exit 0
fi

if [ ! -e $KUBESPRAY_TARBALL ]; then
    echo "$KUBESPRAY_TARBALL does not exist."
    exit 1
fi

tar xvzf $KUBESPRAY_TARBALL || exit 1

# apply patches
#sleep 1 # avoid annoying patch error in shared folders.
#if [ -d $CURRENT_DIR/patches/${KUBESPRAY_VERSION} ]; then
#    for patch in $CURRENT_DIR/patches/${KUBESPRAY_VERSION}/*.patch; do
#        if [[ -f "${patch}" ]]; then
#          echo "===> Apply patch: $patch"
#          (cd $DIR && patch -p1 < $patch)
#        fi
#    done
#fi

```
执行：

```bash
./extract-kubespray.sh
```
### 9.2 编写 inventory.ini
```bash
vim kubespray-2.23.0/inventory/sample/inventory.ini
```

```bash
[all]
kube-master01 ansible_host=10.70.0.71
kube-master02 ansible_host=10.70.0.72
kube-master03 ansible_host=10.70.0.73
kube-node01 ansible_host=10.70.0.74
kube-node02 ansible_host=10.70.0.75
kube-node03 ansible_host=10.70.0.76

[bastion]
bastion01 ansible_host=10.70.0.78 ansible_user=root

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

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

### 9.3 编写 offline.yml

```bash
vim  outputs/kubespray-2.22.1/inventry/sample/group_vars/all/offline.yml
```


```bash
http_server: "http://10.60.0.30"
registry_host: "registry.demo:35000"

containerd_insecure_registries: # Kubespray #8340
  "registry.demo:35000": "http://registry.demo:35000"

files_repo: "{{ http_server }}/files"
yum_repo: "{{ http_server }}/rpms"
ubuntu_repo: "{{ http_server }}/debs"

# Registry overrides
kube_image_repo: "{{ registry_host }}"
gcr_image_repo: "{{ registry_host }}"
docker_image_repo: "{{ registry_host }}"
quay_image_repo: "{{ registry_host }}"

# Download URLs: See roles/download/defaults/main.yml of kubespray.
kubeadm_download_url: "{{ files_repo }}/kubernetes/{{ kube_version }}/kubeadm"
kubectl_download_url: "{{ files_repo }}/kubernetes/{{ kube_version }}/kubectl"
kubelet_download_url: "{{ files_repo }}/kubernetes/{{ kube_version }}/kubelet"
# etcd is optional if you **DON'T** use etcd_deployment=host
etcd_download_url: "{{ files_repo }}/kubernetes/etcd/etcd-{{ etcd_version }}-linux-amd64.tar.gz"
cni_download_url: "{{ files_repo }}/kubernetes/cni/cni-plugins-linux-{{ image_arch }}-{{ cni_version }}.tgz"
crictl_download_url: "{{ files_repo }}/kubernetes/cri-tools/crictl-{{ crictl_version }}-{{ ansible_system | lower }}-{{ image_arch }}.tar.gz"
# If using Calico
calicoctl_download_url: "{{ files_repo }}/kubernetes/calico/{{ calico_ctl_version }}/calicoctl-linux-{{ image_arch }}"
# If using Calico with kdd
calico_crds_download_url: "{{ files_repo }}/kubernetes/calico/{{ calico_version }}.tar.gz"

runc_download_url: "{{ files_repo }}/runc/{{ runc_version }}/runc.{{ image_arch }}"
nerdctl_download_url: "{{ files_repo }}/nerdctl-{{ nerdctl_version }}-{{ ansible_system | lower }}-{{ image_arch }}.tar.gz"
containerd_download_url: "{{ files_repo }}/containerd-{{ containerd_version }}-linux-{{ image_arch }}.tar.gz"

#containerd_insecure_registries:
#    "{{ registry_addr }}"："{{ registry_host }}"

# CentOS/Redhat/AlmaLinux/Rocky Linux
## Docker / Containerd
docker_rh_repo_base_url: "{{ yum_repo }}/docker-ce/$releasever/$basearch"
docker_rh_repo_gpgkey: "{{ yum_repo }}/docker-ce/gpg"

# Fedora
## Docker
docker_fedora_repo_base_url: "{{ yum_repo }}/docker-ce/{{ ansible_distribution_major_version }}/{{ ansible_architecture }}"
docker_fedora_repo_gpgkey: "{{ yum_repo }}/docker-ce/gpg"
## Containerd
containerd_fedora_repo_base_url: "{{ yum_repo }}/containerd"
containerd_fedora_repo_gpgkey: "{{ yum_repo }}/docker-ce/gpg"
```


### 9.4 配置 containerd.yml

```bash
$ vim  inventory/sample/group_vars/all/containerd.yml
......
containerd_registries_mirrors:
 - prefix: registry.bsg:35000
   mirrors:
    - host: http://registry.bsg:35000
      capabilities: ["pull", "resolve"]
      skip_verify: true
....
```

### 9.5 配置 nerdctl
解决镜像拉取失败

```bash
vim roles/container-engine/nerdctl/templates/nerdctl.toml.j2 
...
insecure_registry = true #添加
```

### 9.6 配置 nf_conntrack

123-148行 nf_conntrack_ipv4修改为 nf_conntrack
```bash
$ vim roles/kubernetes/node/tasks/main.yml
- name: Modprobe nf_conntrack
  community.general.modprobe:
    name: nf_conntrack
    state: present
  register: modprobe_nf_conntrack
  ignore_errors: true  # noqa ignore-errors
  when:
    - kube_proxy_mode == 'ipvs'
  tags:
    - kube-proxy

- name: Persist ip_vs modules
  copy:
    dest: /etc/modules-load.d/kube_proxy-ipvs.conf
    mode: 0644
    content: |
      ip_vs
      ip_vs_rr
      ip_vs_wrr
      ip_vs_sh
      {% if modprobe_nf_conntrack is success -%}
      nf_conntrack
      {%-   endif -%}
  when: kube_proxy_mode == 'ipvs'
  tags:
    - kube-proxy

```

### 9.7 配置 resolve.conf
```bash
echo "nameserver 8.8.8.8" >  /etc/resolv.conf
ansible all -m copy -a "src=/etc/resolv.conf dest=/etc/"
```

### 9.8 配置开源插件  metrics-server

```bash
$ cat inventory/sample/group_vars/k8s_cluster/addons.yml |grep metric
metrics_server_enabled: true #false 改为 true
```

## 10. 部署 offline repo
Deploy offline repo configurations which use your yum_repo/ubuntu_repo to all target nodes using ansible.

First, copy offline setup playbook to kubespray directory.

###  10.1 添加系统yum源

```bash
ansible all -m shell -a  "mv /etc/yum.repos.d /tmp"
ansible all -m shell -a  "mkdir /etc/yum.repos.d"
ansible all -m copy -a "src=/etc/yum.repos.d/http_iso.repo dest=/etc/yum.repos.d/"
```

###  10.2 添加 其他工具yum源
```bash
$ docker exec -ti kubespray-offline-ansible bash
$ cd outputs/kubespray-2.23.0
$ ls playbooks
ansible_version.yml  cluster.yml  facts.yml  legacy_groups.yml  recover_control_plane.yml  remove_node.yml  reset.yml  scale.yml  upgrade_cluster.yml
$ ls ../playbook/
offline-repo.yml  roles

cp -r ../playbook .
```

```bash

#ansible -i inventory/sample/inventory.ini all -m copy -a "src=/tmp/yum.repos.d/offline.repo dest=/etc/yum.repos.d/"

ansible-playbook -i  inventory/sample/inventory.ini playbook/offline-repo.yml 
ansible -i inventory/sample/inventory.ini all  -m shell -a "yum repolist --verbose"
```
或者登到批量对象节点检查 `offline.repo`是否正常

```bash
[root@kube-master01 ~]# yum repolist -v
This system is not registered with an entitlement server. You can use subscription-manager to register.

YUM version: 4.7.0
cachedir: /var/cache/dnf
Repo-id            : AppStream
Repo-name          : http_iso
Repo-revision      : 1656402327
Repo-updated       : Tue Jun 28 15:45:28 2022
Repo-pkgs          : 6609
Repo-available-pkgs: 5334
Repo-size          : 8.6 G
Repo-baseurl       : http://10.70.0.254/AppStream
Repo-expire        : 172800 second(s) (last: Fri Oct 20 16:54:08 2023)
Repo-filename      : /etc/yum.repos.d/http_iso.repo

Repo-id            : BaseOS
Repo-name          : http_iso
Repo-revision      : 1656402376
Repo-updated       : Tue Jun 28 15:46:17 2022
Repo-pkgs          : 1714
Repo-available-pkgs: 1558
Repo-size          : 1.3 G
Repo-baseurl       : http://10.70.0.254/BaseOS
Repo-expire        : 172800 second(s) (last: Fri Oct 20 16:54:07 2023)
Repo-filename      : /etc/yum.repos.d/http_iso.repo

Repo-id            : offline-repo
Repo-name          : Offline repo for kubespray
Repo-revision      : 1697770474
Repo-updated       : Fri Oct 20 10:54:34 2023
Repo-pkgs          : 299
Repo-available-pkgs: 299
Repo-size          : 213 M
Repo-baseurl       : http://10.70.0.78/rpms/local
Repo-expire        : 172800 second(s) (last: Fri Oct 20 16:47:16 2023)
Repo-filename      : /etc/yum.repos.d/offline.repo
Total packages: 8622http_iso                                         67 MB/s | 2.4 MB     00:00    
http_iso                                         71 MB/s | 7.5 MB     00:00    

```



##  11. 开始部署
执行部署：
```bash
ansible-playbook -i inventory/sample/inventory.ini --become --become-user=root cluster.yml
```


输出：

```bash
Saturday 21 October 2023  16:12:13 +0000 (0:00:00.364)       0:36:05.955 ****** 

TASK [network_plugin/calico : Check if inventory match current cluster configuration] ***************************************************************************************
task path: /kubespray/outputs/kubespray-2.23.0/roles/network_plugin/calico/tasks/check.yml:168
ok: [kube-master01] => {
    "changed": false,
    "msg": "All assertions passed"
}
Saturday 21 October 2023  16:12:14 +0000 (0:00:00.511)       0:36:06.466 ****** 
Saturday 21 October 2023  16:12:14 +0000 (0:00:00.177)       0:36:06.643 ****** 
Saturday 21 October 2023  16:12:14 +0000 (0:00:00.193)       0:36:06.836 ****** 

PLAY RECAP ******************************************************************************************************************************************************************
bastion01                  : ok=6    changed=0    unreachable=0    failed=0    skipped=14   rescued=0    ignored=0   
kube-master01              : ok=728  changed=54   unreachable=0    failed=0    skipped=1278 rescued=0    ignored=7   
kube-master02              : ok=141  changed=6    unreachable=0    failed=0    skipped=369  rescued=0    ignored=1   
kube-master03              : ok=141  changed=6    unreachable=0    failed=0    skipped=369  rescued=0    ignored=1   
kube-node01                : ok=505  changed=21   unreachable=0    failed=0    skipped=774  rescued=0    ignored=1   
kube-node02                : ok=505  changed=21   unreachable=0    failed=0    skipped=773  rescued=0    ignored=1   
kube-node03                : ok=505  changed=21   unreachable=0    failed=0    skipped=773  rescued=0    ignored=1   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Saturday 21 October 2023  16:12:15 +0000 (0:00:00.863)       0:36:07.700 ****** 
=============================================================================== 
etcd : Gen_certs | Write etcd member/admin and kube_control_plane client certs to other etcd nodes ------------------------------------------------------------------ 31.69s
/kubespray/outputs/kubespray-2.23.0/roles/etcd/tasks/gen_certs_script.yml:102 ----------------------------------------------------------------------------------------------
container-engine/nerdctl : Download_file | Download item ------------------------------------------------------------------------------------------------------------ 22.60s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:88 ----------------------------------------------------------------------------------------------
container-engine/runc : Download_file | Download item --------------------------------------------------------------------------------------------------------------- 22.52s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:88 ----------------------------------------------------------------------------------------------
etcdctl_etcdutl : Download_file | Download item --------------------------------------------------------------------------------------------------------------------- 22.32s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:88 ----------------------------------------------------------------------------------------------
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ---------------------------------------------------------------------------------------------- 22.14s
/kubespray/outputs/kubespray-2.23.0/roles/kubernetes-apps/ansible/tasks/coredns.yml:2 --------------------------------------------------------------------------------------
download : Download_file | Download item ---------------------------------------------------------------------------------------------------------------------------- 22.08s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:88 ----------------------------------------------------------------------------------------------
container-engine/crictl : Download_file | Download item ------------------------------------------------------------------------------------------------------------- 21.82s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:88 ----------------------------------------------------------------------------------------------
bootstrap-os : Check RHEL subscription-manager status --------------------------------------------------------------------------------------------------------------- 21.79s
/kubespray/outputs/kubespray-2.23.0/roles/bootstrap-os/tasks/bootstrap-redhat.yml:26 ---------------------------------------------------------------------------------------
container-engine/containerd : Download_file | Download item --------------------------------------------------------------------------------------------------------- 21.77s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:88 ----------------------------------------------------------------------------------------------
kubernetes-apps/ansible : Kubernetes Apps | Start Resources --------------------------------------------------------------------------------------------------------- 19.64s
/kubespray/outputs/kubespray-2.23.0/roles/kubernetes-apps/ansible/tasks/main.yml:39 ----------------------------------------------------------------------------------------
kubernetes/control-plane : Kubeadm | Initialize first master -------------------------------------------------------------------------------------------------------- 19.22s
/kubespray/outputs/kubespray-2.23.0/roles/kubernetes/control-plane/tasks/kubeadm-setup.yml:169 -----------------------------------------------------------------------------
download : Download | Download files / images ----------------------------------------------------------------------------------------------------------------------- 17.91s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/main.yml:19 -------------------------------------------------------------------------------------------------------
etcdctl_etcdutl : Extract_file | Unpacking archive ------------------------------------------------------------------------------------------------------------------ 17.88s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/extract_file.yml:2 ------------------------------------------------------------------------------------------------
container-engine/runc : Download_file | Validate mirrors ------------------------------------------------------------------------------------------------------------ 17.71s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:58 ----------------------------------------------------------------------------------------------
container-engine/nerdctl : Download_file | Validate mirrors --------------------------------------------------------------------------------------------------------- 17.71s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:58 ----------------------------------------------------------------------------------------------
container-engine/crictl : Extract_file | Unpacking archive ---------------------------------------------------------------------------------------------------------- 17.66s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/extract_file.yml:2 ------------------------------------------------------------------------------------------------
container-engine/crictl : Download_file | Validate mirrors ---------------------------------------------------------------------------------------------------------- 17.64s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:58 ----------------------------------------------------------------------------------------------
container-engine/containerd : Download_file | Validate mirrors ------------------------------------------------------------------------------------------------------ 17.56s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/download_file.yml:58 ----------------------------------------------------------------------------------------------
container-engine/nerdctl : Extract_file | Unpacking archive --------------------------------------------------------------------------------------------------------- 16.62s
/kubespray/outputs/kubespray-2.23.0/roles/download/tasks/extract_file.yml:2 ------------------------------------------------------------------------------------------------
network_plugin/calico : Wait for calico kubeconfig to be created ---------------------------------------------------------------------------------------------------- 15.16s
/kubespray/outputs/kubespray-2.23.0/roles/network_plugin/calico/tasks/install.yml:440 --------------------------------------------------------------------------------------

```

## 12. 检查

```bash
$ kubectl get node
NAME            STATUS   ROLES           AGE     VERSION
kube-master01   Ready    control-plane   10m     v1.27.5
kube-node01     Ready    <none>          8m32s   v1.27.5
kube-node02     Ready    <none>          8m32s   v1.27.5
kube-node03     Ready    <none>          8m32s   v1.27.5

$ kubectl get pod -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-84658d8c67-s6qdc   1/1     Running   0          50m
kube-system   calico-node-99pmm                          1/1     Running   0          52m
kube-system   calico-node-hw94j                          1/1     Running   0          52m
kube-system   calico-node-mgbmj                          1/1     Running   0          52m
kube-system   calico-node-rgttm                          1/1     Running   0          52m
kube-system   coredns-76df6bcdf4-5lvhd                   1/1     Running   0          49m
kube-system   coredns-76df6bcdf4-fg8cp                   1/1     Running   0          49m
kube-system   dns-autoscaler-6cdd8566bb-4qcr6            1/1     Running   0          49m
kube-system   kube-apiserver-kube-master01               1/1     Running   1          55m
kube-system   kube-controller-manager-kube-master01      1/1     Running   2          55m
kube-system   kube-proxy-f7ljn                           1/1     Running   0          53m
kube-system   kube-proxy-pjctr                           1/1     Running   0          53m
kube-system   kube-proxy-sprx6                           1/1     Running   0          53m
kube-system   kube-proxy-wsh7r                           1/1     Running   0          53m
kube-system   kube-scheduler-kube-master01               1/1     Running   1          55m
kube-system   nginx-proxy-kube-node01                    1/1     Running   0          53m
kube-system   nginx-proxy-kube-node02                    1/1     Running   0          53m
kube-system   nginx-proxy-kube-node03                    1/1     Running   0          53m
kube-system   nodelocaldns-42zzm                         1/1     Running   0          49m
kube-system   nodelocaldns-8q7cs                         1/1     Running   0          49m
kube-system   nodelocaldns-bcnm2                         1/1     Running   0          49m
kube-system   nodelocaldns-srv8d                         1/1     Running   0          49m


$ crictl images
IMAGE                                                    TAG                 IMAGE ID            SIZE
registry.bsg:35000/calico/cni                            v3.25.2             8ea2978822546       202MB
registry.bsg:35000/calico/kube-controllers               v3.25.2             cb60adca8b5ed       70.4MB
registry.bsg:35000/calico/node                           v3.25.2             ed304c33a22f1       245MB
registry.bsg:35000/calico/pod2daemon-flexvol             v3.25.2             2ea0861c9c7a0       14.7MB
registry.bsg:35000/coredns/coredns                       v1.10.1             ead0a4a53df89       53.6MB
registry.bsg:35000/cpa/cluster-proportional-autoscaler   v1.8.8              b6d1a4be0743f       39.4MB
registry.bsg:35000/dns/k8s-dns-node-cache                1.22.20             ff71cd4ea5ae5       69.4MB
registry.bsg:35000/kube-apiserver                        v1.27.5             b58f4bc393450       122MB
registry.bsg:35000/kube-controller-manager               v1.27.5             ae819fd2a0d75       114MB
registry.bsg:35000/kube-proxy                            v1.27.5             f249729a23555       72.7MB
registry.bsg:35000/kube-scheduler                        v1.27.5             96c06904875e1       59.8MB
registry.bsg:35000/pause                                 3.9                 e6f1816883972       747kB

```


## 清理集群
##  报错



###  镜像拉取失败
```bash
SK [download : Download_container | Download image if required] *********************************************************************************************************************
fatal: [kube-master01]: FAILED! => {"attempts": 4, "changed": true, "cmd": ["/usr/local/bin/nerdctl", "-n", "k8s.io", "pull", "--quiet", "registry.bsg:35000/calico/node:v3.25.2"], "0.082595", "end": "2023-10-20 17:59:27.145736", "msg": "non-zero return code", "rc": 1, "start": "2023-10-20 17:59:27.063141", "stderr": "time=\"2023-10-20T17:59:27+08:00\" level=in next host\" error=\"failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\" host=\"registrtime=\"2023-10-20T17:59:27+08:00\" level=error msg=\"server \\\"registry.bsg:35000\\\" does not seem to support HTTPS\" error=\"failed to resolve reference \\\"registry.bsg:35000/ca.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\"\ntime=\"2023-10-20T17:59:27info msg=\"Hint: you may want to try --insecure-registry to allow plain HTTP (if you are in a trusted network)\"\ntime=\"2023-10-20T17:59:27+08:00\" level=fatal msg=\"failed to reso\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTstderr_lines": ["time=\"2023-10-20T17:59:27+08:00\" level=info msg=\"trying next host\" error=\"failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3: server gave HTTP response to HTTPS client\" host=\"registry.bsg:35000\"", "time=\"2023-10-20T17:59:27+08:00\" level=error msg=\"server \\\"registry.bsg:35000\\\" does not seem to  error=\"failed to resolve reference \\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": ve HTTP response to HTTPS client\"", "time=\"2023-10-20T17:59:27+08:00\" level=info msg=\"Hint: you may want to try --insecure-registry to allow plain HTTP (if you are in a trusted ime=\"2023-10-20T17:59:27+08:00\" level=fatal msg=\"failed to resolve reference \\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:3node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\""], "stdout": "", "stdout_lines": []}
fatal: [kube-node02]: FAILED! => {"attempts": 4, "changed": true, "cmd": ["/usr/local/bin/nerdctl", "-n", "k8s.io", "pull", "--quiet", "registry.bsg:35000/calico/node:v3.25.2"], "de076080", "end": "2023-10-20 17:59:26.653303", "msg": "non-zero return code", "rc": 1, "start": "2023-10-20 17:59:26.577223", "stderr": "time=\"2023-10-20T17:59:26+08:00\" level=infoext host\" error=\"failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\" host=\"registry.me=\"2023-10-20T17:59:26+08:00\" level=error msg=\"server \\\"registry.bsg:35000\\\" does not seem to support HTTPS\" error=\"failed to resolve reference \\\"registry.bsg:35000/cali\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\"\ntime=\"2023-10-20T17:59:26+0fo msg=\"Hint: you may want to try --insecure-registry to allow plain HTTP (if you are in a trusted network)\"\ntime=\"2023-10-20T17:59:26+08:00\" level=fatal msg=\"failed to resolv"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPSderr_lines": ["time=\"2023-10-20T17:59:26+08:00\" level=info msg=\"trying next host\" error=\"failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.2server gave HTTP response to HTTPS client\" host=\"registry.bsg:35000\"", "time=\"2023-10-20T17:59:26+08:00\" level=error msg=\"server \\\"registry.bsg:35000\\\" does not seem to surror=\"failed to resolve reference \\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": ht HTTP response to HTTPS client\"", "time=\"2023-10-20T17:59:26+08:00\" level=info msg=\"Hint: you may want to try --insecure-registry to allow plain HTTP (if you are in a trusted nee=\"2023-10-20T17:59:26+08:00\" level=fatal msg=\"failed to resolve reference \\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:350de/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\""], "stdout": "", "stdout_lines": []}
FAILED - RETRYING: [kube-node01]: Download_container | Download image if required (1 retries left).
FAILED - RETRYING: [kube-node03]: Download_container | Download image if required (1 retries left).
fatal: [kube-node01]: FAILED! => {"attempts": 4, "changed": true, "cmd": ["/usr/local/bin/nerdctl", "-n", "k8s.io", "pull", "--quiet", "registry.bsg:35000/calico/node:v3.25.2"], "de084238", "end": "2023-10-20 17:59:35.557300", "msg": "non-zero return code", "rc": 1, "start": "2023-10-20 17:59:35.473062", "stderr": "time=\"2023-10-20T17:59:35+08:00\" level=infoext host\" error=\"failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\" host=\"registry.me=\"2023-10-20T17:59:35+08:00\" level=error msg=\"server \\\"registry.bsg:35000\\\" does not seem to support HTTPS\" error=\"failed to resolve reference \\\"registry.bsg:35000/cali\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\"\ntime=\"2023-10-20T17:59:35+0fo msg=\"Hint: you may want to try --insecure-registry to allow plain HTTP (if you are in a trusted network)\"\ntime=\"2023-10-20T17:59:35+08:00\" level=fatal msg=\"failed to resolv"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPSderr_lines": ["time=\"2023-10-20T17:59:35+08:00\" level=info msg=\"trying next host\" error=\"failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.2server gave HTTP response to HTTPS client\" host=\"registry.bsg:35000\"", "time=\"2023-10-20T17:59:35+08:00\" level=error msg=\"server \\\"registry.bsg:35000\\\" does not seem to surror=\"failed to resolve reference \\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": ht HTTP response to HTTPS client\"", "time=\"2023-10-20T17:59:35+08:00\" level=info msg=\"Hint: you may want to try --insecure-registry to allow plain HTTP (if you are in a trusted nee=\"2023-10-20T17:59:35+08:00\" level=fatal msg=\"failed to resolve reference \\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:350de/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\""], "stdout": "", "stdout_lines": []}
fatal: [kube-node03]: FAILED! => {"attempts": 4, "changed": true, "cmd": ["/usr/local/bin/nerdctl", "-n", "k8s.io", "pull", "--quiet", "registry.bsg:35000/calico/node:v3.25.2"], "de076280", "end": "2023-10-20 17:59:36.900722", "msg": "non-zero return code", "rc": 1, "start": "2023-10-20 17:59:35.824442", "stderr": "time=\"2023-10-20T17:59:35+08:00\" level=infoext host\" error=\"failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\" host=\"registry.me=\"2023-10-20T17:59:35+08:00\" level=error msg=\"server \\\"registry.bsg:35000\\\" does not seem to support HTTPS\" error=\"failed to resolve reference \\\"registry.bsg:35000/cali\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\"\ntime=\"2023-10-20T17:59:35+0fo msg=\"Hint: you may want to try --insecure-registry to allow plain HTTP (if you are in a trusted network)\"\ntime=\"2023-10-20T17:59:35+08:00\" level=fatal msg=\"failed to resolv"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPSderr_lines": ["time=\"2023-10-20T17:59:35+08:00\" level=info msg=\"trying next host\" error=\"failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.2server gave HTTP response to HTTPS client\" host=\"registry.bsg:35000\"", "time=\"2023-10-20T17:59:35+08:00\" level=error msg=\"server \\\"registry.bsg:35000\\\" does not seem to surror=\"failed to resolve reference \\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:35000/v2/calico/node/manifests/v3.25.2\\\": ht HTTP response to HTTPS client\"", "time=\"2023-10-20T17:59:35+08:00\" level=info msg=\"Hint: you may want to try --insecure-registry to allow plain HTTP (if you are in a trusted nee=\"2023-10-20T17:59:35+08:00\" level=fatal msg=\"failed to resolve reference \\\"registry.bsg:35000/calico/node:v3.25.2\\\": failed to do request: Head \\\"https://registry.bsg:350de/manifests/v3.25.2\\\": http: server gave HTTP response to HTTPS client\""], "stdout": "", "stdout_lines": []}

NO MORE HOSTS LEFT ******************************************************************************************************************************************************************

PLAY RECAP **************************************************************************************************************************************************************************
bastion01                  : ok=6    changed=1    unreachable=0    failed=0    skipped=14   rescued=0    ignored=0   
kube-master01              : ok=302  changed=2    unreachable=0    failed=1    skipped=400  rescued=0    ignored=1   
kube-master02              : ok=85   changed=1    unreachable=0    failed=0    skipped=330  rescued=0    ignored=1   
kube-master03              : ok=85   changed=1    unreachable=0    failed=0    skipped=330  rescued=0    ignored=1   
kube-node01                : ok=256  changed=2    unreachable=0    failed=1    skipped=344  rescued=0    ignored=1   
kube-node02                : ok=256  changed=2    unreachable=0    failed=1    skipped=344  rescued=0    ignored=1   
kube-node03                : ok=256  changed=2    unreachable=0    failed=1    skipped=344  rescued=0    ignored=1   
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```




### 内核模块启动

```bash
TASK [kubernetes/node : Modprobe nf_conntrack_ipv4] ********************************************************************************************************************************
fatal: [kube-master01]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [kube-node01]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [kube-node02]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
fatal: [kube-node03]: FAILED! => {"changed": false, "msg": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64\n", "name": "nf_conntrack_ipv4", "params": "", "rc": 1, "state": "present", "stderr": "modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64\n", "stderr_lines": ["modprobe: FATAL: Module nf_conntrack_ipv4 not found in directory /lib/modules/4.18.0-372.9.1.el8.x86_64"], "stdout": "", "stdout_lines": []}
...ignoring
```

解决方法：

```bash
 ansible all -m shell -a "modprobe br_netfilter"

修改
$  cat roles/kubernetes/node/tasks/main.yml
- name: Modprobe nf_conntrack_ipv4
  community.general.modprobe:
    name: nf_conntrack_ipv4
    state: present
  register: modprobe_nf_conntrack_ipv4
  ignore_errors: true  # noqa ignore-errors
  when:
    - kube_proxy_mode == 'ipvs'
  tags:
    - kube-proxy
修改为
- name: Modprobe nf_conntrack
  community.general.modprobe:
    name: nf_conntrack
    state: present
  register: modprobe_nf_conntrack
  ignore_errors: true  # noqa ignore-errors
  when:
    - kube_proxy_mode == 'ipvs'
  tags:
    - kube-proxy
```

###  集群初始化失败

```bash
TASK [kubernetes/control-plane : Kubeadm | Initialize first master] ****************************************************************************************************************
fatal: [kube-master01]: FAILED! => {"attempts": 3, "changed": true, "cmd": ["timeout", "-k", "300s", "300s", "/usr/local/bin/kubeadm", "init", "--config=/etc/kubernetes/kubeadm-config.yaml", "--ignore-preflight-errors=all", "--skip-phases=addon/coredns", "--upload-certs"], "delta": "0:05:00.012324", "end": "2023-10-20 22:05:51.670418", "failed_when_result": true, "msg": "non-zero return code", "rc": 124, "start": "2023-10-20 22:00:51.658094", "stderr": "W1020 22:00:51.756218   81472 utils.go:69] The recommended value for \"clusterDNS\" in \"KubeletConfiguration\" is: [10.233.0.10]; the provided value is: [169.254.25.10]\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml already exists\n\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml]: /etc/kubernetes/manifests/kube-scheduler.yaml already exists\n\t[WARNING FileExisting-tc]: tc not found in system path\n\t[WARNING Port-10250]: Port 10250 is in use", "stderr_lines": ["W1020 22:00:51.756218   81472 utils.go:69] The recommended value for \"clusterDNS\" in \"KubeletConfiguration\" is: [10.233.0.10]; the provided value is: [169.254.25.10]", "\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists", "\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml already exists", "\t[WARNING FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml]: /etc/kubernetes/manifests/kube-scheduler.yaml already exists", "\t[WARNING FileExisting-tc]: tc not found in system path", "\t[WARNING Port-10250]: Port 10250 is in use"], "stdout": "[init] Using Kubernetes version: v1.27.5\n[preflight] Running pre-flight checks\n[preflight] Pulling images required for setting up a Kubernetes cluster\n[preflight] This might take a minute or two, depending on the speed of your internet connection\n[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'\n[certs] Using certificateDir folder \"/etc/kubernetes/ssl\"\n[certs] Using existing ca certificate authority\n[certs] Using existing apiserver certificate and key on disk\n[certs] Using existing apiserver-kubelet-client certificate and key on disk\n[certs] Using existing front-proxy-ca certificate authority\n[certs] Using existing front-proxy-client certificate and key on disk\n[certs] External etcd mode: Skipping etcd/ca certificate authority generation\n[certs] External etcd mode: Skipping etcd/server certificate generation\n[certs] External etcd mode: Skipping etcd/peer certificate generation\n[certs] External etcd mode: Skipping etcd/healthcheck-client certificate generation\n[certs] External etcd mode: Skipping apiserver-etcd-client certificate generation\n[certs] Using the existing \"sa\" key\n[kubeconfig] Using kubeconfig folder \"/etc/kubernetes\"\n[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/admin.conf\"\n[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/kubelet.conf\"\n[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/controller-manager.conf\"\n[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/scheduler.conf\"\n[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"\n[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"\n[kubelet-start] Starting the kubelet\n[control-plane] Using manifest folder \"/etc/kubernetes/manifests\"\n[control-plane] Creating static Pod manifest for \"kube-apiserver\"\n[control-plane] Creating static Pod manifest for \"kube-controller-manager\"\n[control-plane] Creating static Pod manifest for \"kube-scheduler\"\n[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory \"/etc/kubernetes/manifests\". This can take up to 5m0s\n[kubelet-check] Initial timeout of 40s passed.", "stdout_lines": ["[init] Using Kubernetes version: v1.27.5", "[preflight] Running pre-flight checks", "[preflight] Pulling images required for setting up a Kubernetes cluster", "[preflight] This might take a minute or two, depending on the speed of your internet connection", "[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'", "[certs] Using certificateDir folder \"/etc/kubernetes/ssl\"", "[certs] Using existing ca certificate authority", "[certs] Using existing apiserver certificate and key on disk", "[certs] Using existing apiserver-kubelet-client certificate and key on disk", "[certs] Using existing front-proxy-ca certificate authority", "[certs] Using existing front-proxy-client certificate and key on disk", "[certs] External etcd mode: Skipping etcd/ca certificate authority generation", "[certs] External etcd mode: Skipping etcd/server certificate generation", "[certs] External etcd mode: Skipping etcd/peer certificate generation", "[certs] External etcd mode: Skipping etcd/healthcheck-client certificate generation", "[certs] External etcd mode: Skipping apiserver-etcd-client certificate generation", "[certs] Using the existing \"sa\" key", "[kubeconfig] Using kubeconfig folder \"/etc/kubernetes\"", "[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/admin.conf\"", "[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/kubelet.conf\"", "[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/controller-manager.conf\"", "[kubeconfig] Using existing kubeconfig file: \"/etc/kubernetes/scheduler.conf\"", "[kubelet-start] Writing kubelet environment file with flags to file \"/var/lib/kubelet/kubeadm-flags.env\"", "[kubelet-start] Writing kubelet configuration to file \"/var/lib/kubelet/config.yaml\"", "[kubelet-start] Starting the kubelet", "[control-plane] Using manifest folder \"/etc/kubernetes/manifests\"", "[control-plane] Creating static Pod manifest for \"kube-apiserver\"", "[control-plane] Creating static Pod manifest for \"kube-controller-manager\"", "[control-plane] Creating static Pod manifest for \"kube-scheduler\"", "[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory \"/etc/kubernetes/manifests\". This can take up to 5m0s", "[kubelet-check] Initial timeout of 40s passed."]}

NO MORE HOSTS LEFT *****************************************************************************************************************************************************************
```

发现 kube-master01 message 报错：

```bash
Oct 21 22:03:22 kube-master01 kubelet[122472]: E1021 22:03:22.826020  122472 dns.go:292] "Could not open resolv conf file." err="open /etc/resolv.conf: no such file or directory"
Oct 21 22:03:22 kube-master01 kubelet[122472]: I1021 22:03:22.827596  122472 util.go:30] "No sandbox for pod can be found. Need to start a new one" pod="kube-system/kube-scheduler-kube-master01"
Oct 21 22:03:22 kube-master01 kubelet[122472]: E1021 22:03:22.827626  122472 kuberuntime_sandbox.go:45] "Failed to generate sandbox config for pod" err="open /etc/resolv.conf: no such file or directory" pod="kube-system/kube-apiserver-kube-master01"
Oct 21 22:03:22 kube-master01 kubelet[122472]: E1021 22:03:22.827722  122472 kuberuntime_manager.go:1122] "CreatePodSandbox for pod failed" err="open /etc/resolv.conf: no such file or directory" pod="kube-system/kube-apiserver-kube-master01"
Oct 21 22:03:22 kube-master01 kubelet[122472]: E1021 22:03:22.827742  122472 dns.go:292] "Could not open resolv conf file." err="open /etc/resolv.conf: no such file or directory"
Oct 21 22:03:22 kube-master01 kubelet[122472]: E1021 22:03:22.827819  122472 kuberuntime_sandbox.go:45] "Failed to generate sandbox config for pod" err="open /etc/resolv.conf: no such file or directory" pod="kube-system/kube-scheduler-kube-master01"
Oct 21 22:03:22 kube-master01 kubelet[122472]: E1021 22:03:22.827870  122472 kuberuntime_manager.go:1122] "CreatePodSandbox for pod failed" err="open /etc/resolv.conf: no such file or directory" pod="kube-system/kube-scheduler-kube-master01"
Oct 21 22:03:22 kube-master01 kubelet[122472]: E1021 22:03:22.828017  122472 pod_workers.go:1294] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"kube-scheduler-kube-master01_kube-system(5edcd228df6930e6ad0fc84c7e4ffa9d)\" with CreatePodSandboxError: \"Failed to generate sandbox config for pod \\\"kube-scheduler-kube-master01_kube-system(5edcd228df6930e6ad0fc84c7e4ffa9d)\\\": open /etc/resolv.conf: no such file or directory\"" pod="kube-system/kube-scheduler-kube-master01" podUID=5edcd228df6930e6ad0fc84c7e4ffa9d
Oct 21 22:03:22 kube-master01 kubelet[122472]: E1021 22:03:22.827868  122472 pod_workers.go:1294] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"kube-apiserver-kube-master01_kube-system(2f906b53df182def0b702f1f755589af)\" with CreatePodSandboxError: \"Failed to generate sandbox config for pod \\\"kube-apiserver-kube-master01_kube-system(2f906b53df182def0b702f1f755589af)\\\": open /etc/resolv.conf: no such file or directory\"" pod="kube-system/kube-apiserver-kube-master01" podUID=2f906b53df182def0b702f1f755589af
Oct 21 22:03:23 kube-master01 kubelet[122472]: E1021 22:03:23.753030  122472 event.go:289] Unable to write event: '&v1.Event{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta:v1.ObjectMeta{Name:"kube-master01.1790236d5617ede5", GenerateName:"", Namespace:"default", SelfLink:"", UID:"", ResourceVersion:"", Generation:0, CreationTimestamp:time.Date(1, time.January, 1, 0, 0, 0, 0, time.UTC), DeletionTimestamp:<nil>, DeletionGracePeriodSeconds:(*int64)(nil), Labels:map[string]string(nil), Annotations:map[string]string(nil), OwnerReferences:[]v1.OwnerReference(nil), Finalizers:[]string(nil), ManagedFields:[]v1.ManagedFieldsEntry(nil)}, InvolvedObject:v1.ObjectReference{Kind:"Node", Namespace:"", Name:"kube-master01", UID:"kube-master01", APIVersion:"", ResourceVersion:"", FieldPath:""}, Reason:"NodeAllocatableEnforced", Message:"Updated Node Allocatable limit across pods", Source:v1.EventSource{Component:"kubelet", Host:"kube-master01"}, FirstTimestamp:time.Date(2023, time.October, 21, 21, 46, 52, 21493221, time.Local), LastTimestamp:time.Date(2023, time.October, 21, 21, 46, 52, 21493221, time.Local), Count:1, Type:"Normal", EventTime:time.Date(1, time.January, 1, 0, 0, 0, 0, time.UTC), Series:(*v1.EventSeries)(nil), Action:"", Related:(*v1.ObjectReference)(nil), ReportingController:"", ReportingInstance:""}': 'Post "https://10.70.0.71:6443/api/v1/namespaces/default/events": dial tcp 10.70.0.71:6443: connect: connection refused'(may retry after sleeping)
Oct 21 22:03:23 kube-master01 kubelet[122472]: I1021 22:03:23.756711  122472 csi_plugin.go:913] Failed to contact API server when waiting for CSINode publishing: Get "https://10.70.0.71:6443/apis/storage.k8s.io/v1/csinodes/kube-master01": dial tcp 10.70.0.71:6443: connect: connection refused

```

没有 `/etc/resolve.conf` ,手动创建

```bash
echo "nameserver 8.8.8.8" >  /etc/resolv.conf
ansible all -m copy -a "src=/etc/resolv.conf dest=/etc/"
```

参考：

- [https://gitlab.openminds.be/mirror/kubespray/-/blob/master/docs/containerd.md](https://gitlab.openminds.be/mirror/kubespray/-/blob/master/docs/containerd.md)
- [https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- [https://github.com/kubespray-offline/kubespray-offline](https://github.com/kubespray-offline/kubespray-offline)
- [https://github.com/containerd/nerdctl/blob/main/docs/config.md](https://github.com/containerd/nerdctl/blob/main/docs/config.md)
