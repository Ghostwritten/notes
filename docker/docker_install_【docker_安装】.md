#  docker 安装

![在这里插入图片描述](https://img-blog.csdnimg.cn/7a0a8052a60b4809bac2a5a65b1e66fa.png)




---



## 1. centos 安装 (yum) docker

旧版本要卸载：

```bash
sudo yum remove -y docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

```

yum-utils提供了yum-config-manager 效用，并device-mapper-persistent-data和lvm2由需要 devicemapper存储驱动程序
```bash
yum install -y yum-utils 
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
可选：启用边缘和测试存储库。这些存储库包含在docker.repo上面的文件中，但默认情况下处于禁用状态。您可以将它们与稳定的存储库一起启用。

```bash
sudo yum-config-manager --enable docker-ce-edge
sudo yum-config-manager --enable docker-ce-test
```
禁用

```bash
$ sudo yum-config-manager --disable docker-ce-edge
```


按版本号排序结果

```bash
yum list docker-ce --showduplicates | sort -r
```
安装 docker
```bash
sudo yum -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
安装指定版本：

```bash
sudo yum -y install docker-ce-VERSION_STRING docker-ce-cli-VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin

```


启动docker并保证docker重启自动拉起
```bash
sudo systemctl start docker && systemctl enable docker  && systemctl status docker
```
运行一个容器
```bash
sudo docker run hello-world
```

###  1.1 卸载
卸载Docker包

```bash
sudo yum remove -y docker-ce
```
卸载旧版本
```bash
sudo yum remove -y docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

不会自动删除主机上的图像，容器，卷或自定义配置文件。删除所有图像，容器和卷：

```bash
sudo yum remove -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

```

## 2. docker 1.13 安装
导入安装源:
  

```bash
rpm --import "https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e"
```

  如果没有响应使用：pgp.mit.edu 或keyserver.ubuntu.com

```bash
  yum-config-manager --add-repo https://packages.docker.com/1.13/yum/repo/main/centos/7
```

安装：

```bash
  yum makecache fast
  yum install -y docker-engine
```

注意：安装其他cs版本

```bash
  yum list docker-engine.x86_64  --showduplicates |sort -r
  yum install docker-engine-<version>
```

## 3. docker 17.09.0-ce 安装

```bash
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
 rpm -ivh epel-release-latest-7.noarch.rpm

wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-17.09.0.ce-1.el7.centos.x86_64.rpm
yum -y install  docker-engine-selinux
yum -y remove  selinux-policy-targeted-3.13.1-192.0.5.el7_5.6.noarch
yum -y install  docker-engine-selinux

yum -y install container-selinux-2.68-1.el7.noarch.rpm
yumdownloader --resolve container-selinux
```

## 4. oracle 7.4 安装 docker

```bash
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
 rpm -ivh epel-release-latest-7.noarch.rpm
yum -y install  docker-engine-selinux
yum -y remove  selinux-policy-targeted-3.13.1-192.0.5.el7_5.6.noarch
yum -y install  docker-engine-selinux
yum -y install container-selinux-2.68-1.el7.noarch.rpm
yumdownloader --resolve container-selinux
yum -y install docker-ce
```

## 5.  ubuntu 安装（apt-get）docker
### 5.1 设置存储库
1. 更新apt包索引并安装包，以允许apt通过HTTPS使用存储库

```bash
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
2. 添加Docker的官方GPG密钥

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
3. 设置存储库

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
### 5.2 安装 Docker Engine

```bash
sudo apt-get update
```

> 默认的umask可能配置错误，导致无法检测到存储库公钥文件。在更新包索引之前，尝试为Docker公钥文件授予读权限:
> `sudo chmod a+r /etc/apt/keyrings/docker.gpg`
> `sudo apt-get update`

默认安装

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
安装指定版本

```bash
# 列出可用的版本
$ apt-cache madison docker-ce | awk '{ print $3 }'

5:20.10.16~3-0~ubuntu-jammy
5:20.10.15~3-0~ubuntu-jammy
5:20.10.14~3-0~ubuntu-jammy
5:20.10.13~3-0~ubuntu-jammy
```
安装指定版本

```bash
VERSION_STRING=5:20.10.13~3-0~ubuntu-jammy
sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-compose-plugin
```

### 5.3 卸载 docker

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
rm -rf /var/lib/docker
```

##  6. Fedora 安装（dnf）docker
### 6.1 配置源
```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
```
### 6.2 安装
安装默认版本（最新）

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
安装指定版本

```bash
$ dnf list docker-ce  --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.fc28                 docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.fc28                 docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.fc28                docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.fc28                docker-ce-stable

$ sudo dnf -y install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io docker-compose-plugin
```
### 6.3 卸载

```bash
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
rm -rf /var/lib/docker/
```


## 7. 二进制安装
下载 docker 指定版本：[https://download.docker.com/linux/static/stable/x86_64/](https://download.docker.com/linux/static/stable/x86_64/)

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


在配置文件中新增本地镜像仓库下载地址

```bash
cat << EOF > /etc/docker/daemon.json
{
"insecure-registries": ["harbor.demo.com"],
"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF



systemctl daemon-reload;systemctl start docker;systemctl enable docker && systemctl status docker
docker login -u admin -p xxxx harbor.demo.com
docker pull harbor.demo.com/library/busybox:1.28
```


参考：

 - [官方安装 docker](https://docs.docker.com/install/)
 - [下载 docker ](https://download.docker.com/)


