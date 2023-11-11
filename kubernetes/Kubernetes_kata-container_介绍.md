


-------
参考链接：
[https://blog.csdn.net/zhonglinzhang/article/details/86489695](https://blog.csdn.net/zhonglinzhang/article/details/86489695)


## 1. 为什么用kata-container
弥补了传统容器技术安全性的缺点，Kata Containers通过使用硬件虚拟化来达到容器隔离的目的。每一个container/pod 都是基于一个独立的kernel实例来作为一个轻量级的虚拟机。自从每一个container/pod运行与独立的虚拟机上，他们不再从宿主机内核上获取相应所有的权限。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510150304549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. 什么是kata-container
`kata containers`是由`OpenStack`基金会管理的容器项目。kata containers整合了Intel的 `Clear Containers` 和 `Hyper.sh` 的 `runV`，能够支持不同平台的硬件，并符合`Open Container Initiative`规范，同时还可以兼容k8s的 `CRI`（Container Runtime Interface）接口规范。项目包含几个配套组件，即`Runtime`，`Agent`， `Proxy`，`Shim`等。项目已于6月份`release`了1.0版本。
      从docker架构上看，kata-container和原来的runc是平级的。大家知道docker只是管理容器生命周期的框架，真正启动容器最早用的是LXC，然后是runc，现在也可以换成kata了。所以说kata-container可以当做docker的一个插件，启动kata-container可以通过docker命令。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510151406249.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 3. kata container组件
###  3.1 Agent

 Kata-agent运行在guest负责管理容器。Kata-agent的执行单元是定义了一系列命名空间的沙盒。每个VM可以运行多个容器，支持k8s一个pod运行多个容器的需求。不过目前docker中，kata-runtime只能一个pod一个容器。Kata-agent通过gRPC和其他kata组件通信。Kata-agent使用libcontainer管理容器的生命周期

### 3.2 Runtime
kata-runtime是一个OCI兼容的容器运行时，负责处理OCI运行时规范指定的所有命令并启动kata-shim实例

 配置文件是`/usr/share/defaults/kata-containers/configuration.toml`

```bash
# XXX: WARNING: this file is auto-generated.
# XXX:
# XXX: Source file: "cli/config/configuration.toml.in"
# XXX: Project:
# XXX:   Name: Kata Containers
# XXX:   Type: kata
 
[hypervisor.qemu]
path = "/usr/bin/qemu-lite-system-x86_64"
kernel = "/usr/share/kata-containers/vmlinuz.container"
image = "/usr/share/kata-containers/kata-containers.img"
machine_type = "pc"
 
# Optional space-separated list of options to pass to the guest kernel.
# For example, use `kernel_params = "vsyscall=emulate"` if you are having
# trouble running pre-2.15 glibc.
#
# WARNING: - any parameter specified here will take priority over the default
# parameter value of the same name used to start the virtual machine.
# Do not set values here unless you understand the impact of doing so as you
# may stop the virtual machine from booting.
# To see the list of default parameters, enable hypervisor debug, create a
# container and look for 'default-kernel-parameters' log entries.
kernel_params = ""
 
# Path to the firmware.
# If you want that qemu uses the default firmware leave this option empty
firmware = ""
```

### 3.3 Proxy
默认使用virtio-serial和VM通信。VM可以运行多个容器进程。在使用virtio-serial的情况下，与每个进程相关联的I/O流需要在主机上多路复用和解复用。
        Kata-proxy给多个`kata-shim`和`kata-runtime`客户端提供对kata-agent提供访问，它的主要作用是在每个kata-shim和kata-agent之间路由I/O流和信号。Kata-proxy连接到kata-agent的unix域套接字上，这个套接字是kata-proxy启动时kata-runtime提供的

### 3.4 Shim
runtime运行在宿主机上，不能直接监控运行在虚拟机里的进程，最多只能看到QEMU进程。kata-shim监控容器进程，处理容器的所有I/O流，包括stdout、stdin和stderr，以及转发所有的要发送出去的信号。
 Kata-shim还有其他功能：

通过一个UNIX域套接字连接到kata-proxy。这个套接字在kata-runtime启动kata-shim的时候，由kata-runtime传给kata-shim，同时带上了containerID和execID，两个ID用来识别shim管理的是哪个容器。
读取VM内部容器进程的输出流和错误流
使用SignalProcessRequest API转发从reaper到kata-proxy的信号
监控终端修改，并使用grpc TtyWinResize API转发到kata-proxy
 

kata-runtime kata-env

```bash
[Meta]
  Version = "1.0.18"

[Runtime]
  Debug = false
  Path = "/usr/bin/kata-runtime"
  [Runtime.Version]
    Semver = "1.3.1"
    Commit = "258eae0"
    OCI = "1.0.1"
  [Runtime.Config]
    Path = "/etc/kata-containers/configuration.toml"

[Hypervisor]
  MachineType = "pc"
  Version = "QEMU emulator version 2.11.0\nCopyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers"
  Path = "/usr/bin/qemu-lite-system-x86_64"
  BlockDeviceDriver = "virtio-scsi"
  EntropySource = "/dev/urandom"
  Msize9p = 8192
  MemorySlots = 10
  Debug = false
  UseVSock = false

[Image]
  Path = "/usr/share/kata-containers/kata-containers-image_clearlinux_1.3.1_agent_c7fdd324cda.img"

[Kernel]
  Path = "/usr/share/kata-containers/vmlinuz-4.14.67.16-4.4.container"
  Parameters = ""

[Initrd]
  Path = ""

[Proxy]
  Type = "kataProxy"
  Version = "kata-proxy version 1.3.1-d364b2e"
  Path = "/usr/libexec/kata-containers/kata-proxy"
  Debug = false

[Shim]
  Type = "kataShim"
  Version = "kata-shim version 1.3.1-58f757d"
  Path = "/usr/libexec/kata-containers/kata-shim"
  Debug = false

[Agent]
  Type = "kata"

[Host]
  Kernel = "3.10.0-957.el7.x86_64"
  Architecture = "amd64"
  VMContainerCapable = true
  SupportVSocks = false
  [Host.Distro]
    Name = "CentOS Linux"
    Version = "7"
  [Host.CPU]
    Vendor = "GenuineIntel"
    Model = "Intel(R) Core(TM) i7-6500U CPU @ 2.50GHz"

[Netmon]
  Version = "kata-netmon version 1.3.1"
  Path = "/usr/libexec/kata-containers/kata-netmon"
  Debug = false
  Enable = false
```

