![在这里插入图片描述](https://img-blog.csdnimg.cn/e6c29e6efa6a4e5fb09b3b7ad1ba4ac8.png)




##  1. 简介
如果你在使用[Ubuntu 18.04/20.04 LTS](https://ubuntu.com/)版本的Ubuntu系统，会发现系统里面多了一个应用格式包——.snap包。[Snap包](https://snapcraft.io/)是Ubuntu 16.04 LTS发布时引入的新应用格式包。目前已流行在Ubuntu且在其他如Debian、Arch Linux、Fedora、Kaili Linux、openSUSE、Red Hat等Linux发行版上通过snapd来安装使用snap应用。较传统Linux的rpm，deb软件包，snap有什么特点和优势呢？下面将为你介绍snap软件包。

当你在安装完snap后，你会发现在在根目录下会出现如/dev/loop0的挂载点，这些挂载点正是snap软件包的目录。Snap使用了squashFS文件系统，一种开源的压缩，只读文件系统，基于GPL协议发行。一旦snap被安装后，其就有一个只读的文件系统和一个可写入的区域。应用自身的执行文件、库、依赖包都被放在这个只读目录，意味着该目录不能被随意篡改和写入。

squashFS文件系统的引入，使得snap的安全性要优于传统的Linux软件包。同时，每个snap默认都被严格限制（confined），即限制系统权限和资源访问。但是，可通过授予权限策略来获得对系统资源的访问。这也是安全性更好的表现。

Snap可包含一个或多个服务，支持cli（命令行）应用，GUI图形应用以及无单进程限制。因此，你可以单个snap下调用一个或多个服务。对于某些多服务的应用来说，非常方便。前面说到snap间相互隔离，那么怎么交换资源呢？答案是可以通过interface（接口）定义来做资源交换。interface被用于让snap可访问OpenGL加速，声卡播放、录制，网络和HOME目录。Interface由slot和plug组成即提供者和消费者。

## 2. 安装 snap
[这里可以查看更多系统环境下的安装](https://snapcraft.io/docs/installing-snapd)

这里以centos  为例

centos  8

```bash
$ sudo dnf install epel-release
$ sudo dnf upgrade
```

> 如果您有兴趣了解这些包是如何构建的，请参阅为[Red Hat Enterprise Linux (RHEL) 8构建一个快速RPM](https://snapcraft.io/docs/building-snap-rpms-on-rhel)。

centos 7

```bash
$ sudo yum install epel-release
$ sudo yum install snapd
$ sudo systemctl enable --now snapd.socket
$ sudo ln -s /var/lib/snapd/snap /snap
```

## 3. 命令 snap 
1.安装snap，可使用以下命令或图形界面的store通过鼠标点击操作：

```bash
sudo snap install code //安装code snap
```

2.卸载snap

```bash
sudo snap remove code
```

3.搜索snap

```bash
snap find code
```

4.查看snap信息

```bash
snap info code
```

5.查看已安装的snap

```bash
snap list
```

6.更新snap

```bash
sudo snap refresh code channel=latest/stable //channel来指定通道版本
```

## 4. 其他
Snap除了在Ubuntu 桌面和其他Linux发行版桌面系统上使用外，还能在[Ubuntu server](https://ubuntu.com/download/server)和[Ubuntu Core](https://ubuntu.com/core)上使用且为Ubuntu Core默认应用格式包，Ubuntu Core是迷你，与Ubuntu一致，专为物联网设备、嵌入式平台设计，更多内容请访问Ubuntu Core网页。

目前，Ubuntu的相关产品已以snap包的形式发布，例如Ubuntu [MAAS](https://maas.io/)，[Juju](https://juju.is/)，[Multipass](https://multipass.run/)，[MicroK8s](https://microk8s.io/)，[MicroStack](https://microstack.run/)等等。借助snap，你可以一键安装专为笔记本工作站打造的Kubernetes和OpenStack，省去了安装等待和繁琐配置过程。对于开发和测试团队来说无疑是一个更高效的方案，将更多的精力和资源投入到关键价值上。

## 5. Snap应用开发与Snapcraft
介绍了这么多snap相关内容，如何snap如何开发呢？Snap应用使用snapcraft来开发，并且snapcraft已打包为snap应用，一键安装：`sudo snap install snapcraft –classic` 即可开始构建你的snap应用。

参考：
- [什么是Snap应用？](https://cn.ubuntu.com/blog/what-is-snap-application)
- [https://snapcraft.io/](https://snapcraft.io/)

