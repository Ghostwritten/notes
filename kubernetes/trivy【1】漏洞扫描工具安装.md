#  trivy 漏洞扫描工具安装
tags: 安全,trivy
<!-- catalog: trivy 安装 :-->



---

{% youtube %}
https://www.youtube.com/watch?v=bgYrhQ6rTXA
{% endyoutube %}


##  1. 简介
[Trivy](https://aquasecurity.github.io/trivy/dev/) 是一个简单而全面的漏洞/错误配置/秘密扫描器，用于容器和其他工件。 检测操作系统包（Alpine、RHEL、CentOS 等）和特定语言包（Bundler、Composer、npm、yarn 等）的漏洞。此外，扫描Terraform 和 Kubernetes 等基础架构即代码 (IaC) 文件，以检测使您的部署面临攻击风险的潜在配置问题。 还扫描硬编码的秘密vyTrivyTrivyTrivy比如密码、API 密钥和令牌。 Trivy易于使用。只需安装二进制文件，您就可以扫描了。扫描所需要做的就是指定一个目标，例如容器的image名称

![在这里插入图片描述](https://img-blog.csdnimg.cn/562d6fc0627c4d9db4b32189b6d4b243.png)


Trivy 检测到两种类型的安全问题：

 - 漏洞
 - 错误配置

Trivy 可以扫描四种不同的工件：

 - 容器镜像
 - 文件系统和Rootfs
 - Git 存储库
 - Kubernetes

Trivy 可以在两种不同的模式下运行：

 - Standalone
 - Client/Server

Trivy 可以作为 Kubernetes Operator 运行：

 - Kubernetes Operator

##  2. 特征
**全面的漏洞检测**

 - [操作系统包](https://aquasecurity.github.io/trivy/v0.30.4/docs/vulnerability/detection/os/)（Alpine、Red Hat Universal Base Image、Red Hat Enterprise
   Linux、CentOS、AlmaLinux、Rocky Linux、CBL-Mariner、Oracle
   Linux、Debian、Ubuntu、Amazon Linux、openSUSE Leap、SUSE Enterprise
   Linux、Photon OS 和 Distroless）
 - [特定于语言的包](https://aquasecurity.github.io/trivy/v0.30.4/docs/vulnerability/detection/language/)（Bundler、Composer、Pipenv、Poetry、npm、yarn、pnpm、Cargo、NuGet、Maven
   和 Go）

**检测 IaC 错误配置**

 - 开箱即用地提供了多种[内置策略](https://aquasecurity.github.io/trivy/v0.30.4/docs/misconfiguration/policy/builtin/)：
   - Kubernetes
   - docker
   - Terraform

 - 支持自定义策略

**简单**
- 仅指定镜像名称、包含 IaC 配置的目录或工件名称

**快速**
- 第一次扫描将在 10 秒内完成（取决于您的网络）。随后的扫描将在几秒钟内完成。
- 与其他扫描程序在第一次运行时需要很长时间（约 10 分钟）获取漏洞信息并鼓励您维护持久的漏洞数据库不同，Trivy 是无状态的，不需要维护或准备。

**简易安装**
- apt-get install，yum install并且brew install是可能的（参见安装）
- 没有先决条件，例如安装数据库、库等。

**高准确率**
- 尤其是 Alpine Linux 和 RHEL/CentOS
- 其他操作系统也很高

**DevSecOps**
- 适用于Travis CI、CircleCI、Jenkins、GitLab CI 等 CI。
- 请参阅CI 示例

**支持多种格式**
- 容器图像
  - Docker Engine 中作为守护进程运行的本地映像
  - Podman (>=2.0) 中的本地图像暴露了一个套接字
  - Docker Registry 中的远程镜像，例如 Docker Hub、ECR、GCR 和 ACR
  - 存储在docker save/podman save格式文件中的 tar 存档
  - 符合OCI 图像格式的图像目录
- 本地文件系统和 rootfs
- 远程 git 仓库

**SBOM（软件物料清单）支持**
- CycloneDX
- SPDX
- GitHub Dependency Snapshots


##  3. 安装
###  3.1 RHEL/CentOS
yum
```bash
RELEASE_VERSION=$(grep -Po '(?<=VERSION_ID=")[0-9]' /etc/os-release)
cat << EOF | sudo tee -a /etc/yum.repos.d/trivy.repo
[trivy]
name=Trivy repository
baseurl=https://aquasecurity.github.io/trivy-repo/rpm/releases/$RELEASE_VERSION/\$basearch/
gpgcheck=0
enabled=1
EOF
sudo yum -y update
sudo yum -y install trivy
```
rpm

```bash
rpm -ivh https://github.com/aquasecurity/trivy/releases/download/v0.30.4/trivy_0.30.4_Linux-64bit.rpm
```
###  3.2 Debian/Ubuntu
apt源

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```
deb包

```bash
wget https://github.com/aquasecurity/trivy/releases/download/v0.30.4/trivy_0.30.4_Linux-64bit.deb
sudo dpkg -i trivy_0.30.4_Linux-64bit.deb
```
###  3.3 Arch Linux
pikaur
```bash
pikaur -Sy trivy-bin
```
yay

```bash
yay -Sy trivy-bin
```
###  3.4 Homebrew

```bash
brew install aquasecurity/trivy/trivy
```
###  3.5 脚本安装

```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.30.4

```

###  3.6 Docker

```bash
docker pull aquasec/trivy:0.30.4
docker pull ghcr.io/aquasecurity/trivy:0.30.4
docker pull public.ecr.aws/aquasecurity/trivy:0.30.4
```


linux

```bash
docker run --rm -v [YOUR_CACHE_DIR]:/root/.cache/ aquasec/trivy:0.30.4 image [YOUR_IMAGE_NAME]
```
macOS

```bash
docker run --rm -v $HOME/Library/Caches:/root/.cache/ aquasec/trivy:0.30.4 image [YOUR_IMAGE_NAME
```
实例

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
    -v $HOME/Library/Caches:/root/.cache/ aquasec/trivy:0.30.4 python:3.4-alpine
```

###  3.7 Helm

```bash
helm repo add aquasecurity https://aquasecurity.github.io/helm-charts/
helm repo update
helm search repo trivy
helm install my-trivy aquasecurity/trivy
```
使用发布名称安装图表my-release：


```bash
helm install my-release .
```

该命令以默认配置在 Kubernetes 集群上部署 Trivy。[参数](https://github.com/aquasecurity/trivy/tree/v0.30.4/helm/trivy) 部分列出了可以在安装期间配置的参数。

示例

```bash
$ helm install my-release . \
       --namespace my-namespace \
       --set "service.port=9090" \
       --set "trivy.vulnType=os\,library"
```
下一篇我们开始讲：[trivy【2】工具漏洞扫描](https://blog.csdn.net/xixihahalelehehe/article/details/126034395)

参考：

 - [trivy 官方](https://aquasecurity.github.io/trivy/dev/getting-started/installation/#__tabbed_4_2)

[
![在这里插入图片描述](https://img-blog.csdnimg.cn/02a8d300bbef4ada98fb415ce27e00d7.png)](https://zhuanlan.zhihu.com/p/539696760)
