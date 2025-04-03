
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f6f7873245a994e65b0e76484bd111ea.png)



##  1. 前言
体验一下最新的`rhel 9.0` 是什么感觉。它会飞吗？

Red Hat Enterprise Linux (RHEL) 9现已普遍可用 (GA)。该公告发布于 2022 年 5 月 18 日。最新版本旨在满足混合云环境的需求，可以轻松从边缘部署到云端。

RHEL 9可以在KVM和VMware等虚拟机管理程序、物理服务器、云上无缝配置为来宾计算机，或者作为从Red Hat Universal Base Images ( UBI ) 构建的容器运行。

RHEL 9的一些主要亮点:
##  2. 软件
RHEL 9.0提供了以下动态编程语言的新版本:

- PHP 8.0
- Node.JS 16
- Perl 5.32
- Python 3.9
- Ruby 3.0

它还提供以下版本控制系统:
- Git 2.31
- Subversion 1.14

对于数据库有:
- MySQL 8.0
- MariaDB 10.5
- PostgreSQL 13
- Redis 6.2

编译器和开发工具：

- GCC 11.2.1
- glibc 2.34
- binutils 2.35.2

编译器工具集：
- Go Toolset 1.17.7
- LLVM Toolset 13.0.1
- Rust Toolset 1.58.1

##  3. 支持的硬件架构
`Red Hat Enterprise Linux 9.0`附带 Linux kernel `5.14.0`，并提供对以下硬件架构的支持：

- Intel 64 位 (x86-64-v2) 和 AMD 架构。
- 64 位 ARM 架构 (ARMv8.0-A)。
- 64 位 IBM Z (z14)。
- IBM Power Systems，Little Endian (POWER9)。

## 4. GNOME更新到40版
RHEL 9提供了GNOME 40 ，这是与其前身RHEL 8提供的GNOME 3.28相比的巨大飞跃。GNOMe 40带有新外观的“活动概览”，在导航和启动应用程序时提供令人兴奋的用户体验。

其他增强功能包括：

- 带有精美图标的全新 UI。
- 重新设计的设置应用程序部分。
- 改进了远程桌面会话和屏幕共享。
- 改进的性能和资源使用。
- 暂停选项现在包含在“关机/注销”菜单和“重新启动”选项中。
- GNOME shell 扩展现在由Extensions应用程序管理，而不是Software。
- “请勿打扰”按钮现在包含在“通知”弹出窗口中。启用此按钮后，通知不会出现在屏幕上。
- 需要密码的系统对话框现在可以选择通过单击眼睛 (👁) 图标来显示密码文本。
- 分数缩放的可用性作为具有多个预配置分数比率的实验选项。

## 5. 安全和身份
RHEL 9.0提供了`OpenSSL 3.0.1`，这是继最新的 LTS 版本OpenSSL 3.0之后的最新版本。OpenSSL 3.0.1 带有提供者概念。提供者是一组算法实现。它还带有新的版本控制方案并增强了对 HTTPS 的支持。

此外，还调整了以下加密策略以提供增强的安全性。
- 弃用使用 SHA-1 的 TLS 和 SSH 算法，但在 HMAC（基于哈希的消息身份验证代码）中使用 SHA-1 除外。
- TLS 1.0、TLS 1.1、DSA、3DES、DTLS 1.0、Camellia、RC4 和 FFDHE-1024 已被弃用。
- LEGACY 中的最小 RSA 密钥和最小 Diffie-Hellman 参数大小已增加。

RHEL 9中的`SELinux`策略已更新。它现在包括新的类、权限和特性，它们也是内核的一部分。因此，它充分利用了内核允许的全部潜力。


## 6. 构建容器的通用基础镜像
[红帽通用基础镜像](https://access.redhat.com/articles/4238681)提供了一种基于红帽企业 Linux软件轻松构建、运行和管理容器镜像的方法。

Red Hat Enterprise Linux 9提供了 cgroups（控制组）和 podman 的改进版本，这是一个用于在 Linux 系统上构建和管理 OCI 容器的无守护引擎。

容器化应用程序可以在开箱即用的RHEL 9配置上进行测试。您可以了解有关如何在 [RHEL 上构建、运行和管理容器的更多信息](https://www.tecmint.com/manage-containers-using-podman-in-rhel/)。

带有 `Podman 4.0.2` 的 RHEL 9.0 中有很多很棒的新容器功能

通过 RHEL 9，我们继续提供对最新最好的 Podman、Buildah 和 Skopeo 的快速访问，但现在我们还允许您通过 EUS 访问稳定的流。

## 7. 改进了用于管理 RHEL 9 的 Cockpit Web 控制台
ed Hat Enterprise Linux 9提供了[Cockpit web](https://cockpit-project.org/) 控制台，它是一个基于 web 的监控工具，可以监控网络中的物理和虚拟 Linux 系统。

Cockpit使系统管理员能够直观地执行各种管理任务，例如：

- 创建和管理用户帐户。
- 管理用户帐户。
- 监控虚拟机和容器。
- 更新和管理软件包。
- 配置 SELinux。
- 监控指标，例如 CPU、磁盘和内存利用率以及网络统计数据等等。
- 管理订阅。


**更多关于新内容请查看**[Red Hat Enterprise Linux 9 的产品文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9)


参考：
- [What’s New in Red Hat Enterprise Linux (RHEL) 9](https://www.tecmint.com/rhel-9-download/)
