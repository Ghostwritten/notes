##  linux
阿里云
```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

或者

```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

## kubernets
### Debian / Ubuntu
阿里云
```bash
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

### CentOS / RHEL / Fedora
阿里云

- [http://mirrors.aliyun.com/repo/](http://mirrors.aliyun.com/repo/) 

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
腾讯

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg  
EOF
```


查看kubelet kubeadm kubectl版本

```bash
yum list kubelet kubeadm kubectl  --showduplicates|sort -r
```
安装指定版本

```bash
yum install -y kubelet-1.13.5 kubeadm-1.13.5 kubectl-1.13.5
```
查看安装的版本

```bash
kubeadm version
kubectl version
kubelet --version
```
国内下载k8s组件镜像
[https://www.cnblogs.com/xinfang520/p/11698404.html](https://www.cnblogs.com/xinfang520/p/11698404.html)

------
图片鉴赏：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fbfad00ea36a35d7f010c0815c51324d.jpeg#pic_center)

