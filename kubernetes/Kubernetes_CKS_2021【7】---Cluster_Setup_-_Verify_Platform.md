

---

 - [Kubernetes安全专家认证 (CKS)考试动员](https://ghostwritten.blog.csdn.net/article/details/112358241)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)

----



## 1. Practice - Download and verify K8s release

```bash
root@master:~# k get nodes
NAME     STATUS   ROLES                  AGE   VERSION
master   Ready    control-plane,master   95d   v1.20.1
node1    Ready    <none>                 95d   v1.20.1
node2    Ready    <none>                 95d   v1.20.1
```
下载kubernets库
[https://github.com/kubernetes/kubernetes/releases/tag/v1.20.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.20.1)
[https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#v1201](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#v1201)
下载：`kubernetes-server-linux-arm64.tar.gz`

```bash
root@master:~/k8slib# sha512sum kubernetes-server-linux-arm64.tar.gz 
b93857e8c38e433f3edd1ea5727c64b79e1898bcfb8b31a823024c06c2dc66b047482f28d8e89db5c1aae99532a7820dc0212b2aa5a51de3b9c94aa88514b372  kubernetes-server-linux-arm64.tar.gz

root@master:~/k8slib# sha512sum kubernetes-server-linux-arm64.tar.gz  > compare
root@master:~/k8slib# vim compare
root@master:~/k8slib# cat compare
b93857e8c38e433f3edd1ea5727c64b79e1898bcfb8b31a823024c06c2dc66b047482f28d8e89db5c1aae99532a7820dc0212b2aa5a51de3b9c94aa88514b372
```
复制此sha512 hash至`compare`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210425153228226.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~/k8slib# cat compare
b93857e8c38e433f3edd1ea5727c64b79e1898bcfb8b31a823024c06c2dc66b047482f28d8e89db5c1aae99532a7820dc0212b2aa5a51de3b9c94aa88514b372
b93857e8c38e433f3edd1ea5727c64b79e1898bcfb8b31a823024c06c2dc66b047482f28d8e89db5c1aae99532a7820dc0212b2aa5a51de3b9c94aa88514b372
root@master:~/k8slib# cat compare | uniq   #确认二者相同
b93857e8c38e433f3edd1ea5727c64b79e1898bcfb8b31a823024c06c2dc66b047482f28d8e89db5c1aae99532a7820dc0212b2aa5a51de3b9c94aa88514b372
```

##  2. Practice - Verify apiserver binary running in our cluster

```c
root@master:~/k8slib# ls
compare  kubernetes-server-linux-arm64.tar.gz
root@master:~/k8slib# tar zxf kubernetes-server-linux-arm64.tar.gz 
root@master:~/k8slib# ls
compare  kubernetes  kubernetes-server-linux-arm64.tar.gz
root@master:~/k8slib# ls kubernetes/server/bin/kube-apiserver
kubernetes/server/bin/kube-apiserver
root@master:~/k8slib# sha512sum kubernetes/server/bin/kube-apiserver
4d7b3752148a56e457621b1e163f7ef28732c7748f188ca282aed4d540f6e1ec1d48a510ef6d31467d8756ba3f827e5637eb793b89e14eafd9127d6d5ab8424e  kubernetes/server/bin/kube-apiserver
root@master:~/k8slib# sha512sum kubernetes/server/bin/kube-apiserver > compare
root@master:~/k8slib# k -n kube-system get pod | grep api
kube-apiserver-master                      1/1     Running   2          96d
root@master:~/k8slib# k -n kube-system get pod kube-apiserver-master -o yaml | grep image
            f:image: {}
            f:imagePullPolicy: {}
    image: k8s.gcr.io/kube-apiserver:v1.20.1
    imagePullPolicy: IfNotPresent
    image: k8s.gcr.io/kube-apiserver:v1.20.1
    imageID: docker://sha256:ca9843d3b545457f24b012d6d579ba85f132f2406aa171ad84d53caa55e5de99



root@master:~/k8slib# k -n kube-system exec kube-apiserver-master -- sh
OCI runtime exec failed: exec failed: container_linux.go:346: starting container process caused "exec: \"sh\": executable file not found in $PATH": unknown
command terminated with exit code 126
root@master:~/k8slib# k -n kube-system exec kube-apiserver-master -- bash
OCI runtime exec failed: exec failed: container_linux.go:346: starting container process caused "exec: \"bash\": executable file not found in $PATH": unknown
command terminated with exit code 126

root@master:~/k8slib# docker ps |grep apiserver
0fb5321dfd57        ca9843d3b545           "kube-apiserver --ad…"   2 months ago        Up 2 months                             k8s_kube-apiserver_kube-apiserver-master_kube-system_ee31a01764366141f7c85e23f94828f8_2
82760b2dffdc        k8s.gcr.io/pause:3.2   "/pause"                 2 months ago        Up 2 months                             k8s_POD_kube-apiserver-master_kube-system_ee31a01764366141f7c85e23f94828f8_2

root@master:~/k8slib# docker cp 0fb5321dfd57:/ container-fs
root@master:~/k8slib# ls container-fs/
bin  boot  dev  etc  go-runner  home  lib  proc  root  run  sbin  sys  tmp  usr  var

root@master:~/k8slib# find container-fs/ | grep kube-apiserver
container-fs/usr/local/bin/kube-apiserver

root@master:~/k8slib# sha512sum container-fs/usr/local/bin/kube-apiserver
4d7b3752148a56e457621b1e163f7ef28732c7748f188ca282aed4d540f6e1ec1d48a510ef6d31467d8756ba3f827e5637eb793b89e14eafd9127d6d5ab8424e  container-fs/usr/local/bin/kube-apiserver
root@master:~/k8slib# sha512sum container-fs/usr/local/bin/kube-apiserver > compare
root@master:~/k8slib#  vim compare

root@master:~/k8slib# cat compare 
4d7b3752148a56e457621b1e163f7ef28732c7748f188ca282aed4d540f6e1ec1d48a510ef6d31467d8756ba3f827e5637eb793b89e14eafd9127d6d5ab8424e
4d7b3752148a56e457621b1e163f7ef28732c7748f188ca282aed4d540f6e1ec1d48a510ef6d31467d8756ba3f827e5637eb793b89e14eafd9127d6d5ab8424e

root@master:~/k8slib# cat compare | uniq  #确认版本一致
4d7b3752148a56e457621b1e163f7ef28732c7748f188ca282aed4d540f6e1ec1d48a510ef6d31467d8756ba3f827e5637eb793b89e14eafd9127d6d5ab8424e


```

##  3. 官方验证
```bash
kubectl version --short --client

#download checksum for kubectl for linux - (change version)
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

#download old version
curl -LO "https://dl.k8s.io/v1.22.1/bin/linux/amd64/kubectl.sha256"

#download checksum for kubectl for mac - (change version)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl.sha256"

#insall coreutils (for mac)
brew install coreutils

#verify kubectl binary (for linux)
echo "$(<kubectl.sha256) /usr/bin/kubectl" | sha256sum --check
#verify kubectl binary (for mac)
echo "$(<kubectl.sha256) /usr/local/bin/kubectl" | sha256sum -c
```

 - Ref:   [https://github.com/kubernetes/kubernetes/releases](https://github.com/kubernetes/kubernetes/releases)
 - Ref:[https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG#changelogs](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG#changelogs)
 - Ref: [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

