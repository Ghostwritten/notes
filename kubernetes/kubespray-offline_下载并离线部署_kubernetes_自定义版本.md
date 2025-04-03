
![](https://i-blog.csdnimg.cn/blog_migrate/73cee7a1829dfa353253c3ea4f7f4f04.png)



[kubespray-offline v2.21.0-0](https://github.com/tmurakam/kubespray-offline/tree/v2.21.0-0) 默认部署 kubernetes 版本为 `v1.25.6`

我们以自定义部署 kubernetes 版本 `v1.24.10`为例。

- 下载：[https://github.com/tmurakam/kubespray-offline/releases/tag/v2.21.0-0](https://github.com/tmurakam/kubespray-offline/releases/tag/v2.21.0-0)

```bash
unzip v2.21.0-0.zip
cd kubespray-offline-2.21.0-0
```

在下载介质之前需要安装：
- run `install-docker.sh` to install `Docker CE`.
- run `install-containerd.sh` to install `containerd` and `nerdctl.`
- Set docker environment variable to `/usr/local/bin/nerdctl` in config.sh.

docker 与 containerd 选择其一即可：

```bash
./install-docker.sh
```



安装成功后，下载介质执行：
暂时注释掉部分执行脚本

```bash
cat download-all.sh
#!/bin/bash

run() {
    echo "=> Running: $*"
    $* || {
        echo "Failed in : $*"
        exit 1
    }
}

source ./config.sh

run ./install-docker.sh
run ./precheck.sh
run ./prepare-pkgs.sh
run ./prepare-py.sh
run ./get-kubespray.sh
if $ansible_in_container; then
    run ./build-ansible-container.sh
else
    run ./pypi-mirror.sh
fi
#run ./download-kubespray-files.sh
#run ./download-additional-containers.sh
#run ./create-repo.sh
#run ./copy-target-scripts.sh

echo "Done."
```

```bash
./download-all.sh
```

执行结束后修改：

```bash
#修改下载k8s介质版本
$ vim cache/kubespray-2.21.0/roles/kubespray-defaults/defaults/main.yaml
....
kube_version: v1.25.6
.....


$ vim cache/kubespray-2.21.0/inventory/local/group_vars/k8s_cluster/k8s-cluster.yml
...
kube_version: v1.24.10
...
```
其他步骤参考：
- [Kubespray v2.21.0 离线部署 Kubernetes v1.25.6 集群](https://blog.csdn.net/xixihahalelehehe/article/details/130248294)

在执行部署修改如下：
```bash
#修改部署k8s介质版本
vi /root/kubespray-offline-2.21.0-0/outputs/kubespray-2.21.0、inventory/local/group_vars/k8s_cluster/k8s-cluster.yml
...
kube_version: v1.24.10
...
```

```bash
# Example  
$ ansible-playbook -i inventory/local/inventory.ini --become --become-user=root cluster.yml
```


> 注意：仅供参考

参考：
- github：[https://github.com/kubernetes-sigs/kubespray](https://github.com/kubernetes-sigs/kubespray)
- 官网：[https://kubespray.io/#/](https://kubespray.io/#/)
- 网友kubespray 学习：[https://github.com/wenwenxiong/book/tree/master/k8s/kubespray](https://github.com/wenwenxiong/book/tree/master/k8s/kubespray)
