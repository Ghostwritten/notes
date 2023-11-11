#  Apparmor Overview
tags: apparmor,安全





{% youtube %}
https://www.youtube.com/watch?v=KYM-Dzivnjs
{% endyoutube %}

##  1. 简介
[AppArmor](https://launchpad.net/apparmor) 是类似于 Linux 的安全系统的强制访问控制 (MAC)。AppArmor 将单个程序限制在一组文件、功能、网络访问和限制中，统称为程序的 AppArmor 策略，或简称为配置文件。无需重新启动即可将新的或修改的策略应用于正在运行的系统。AppArmor 旨在通过以管理员友好的语言呈现其配置文件，使其易于理解和用于大多数常见需求。
AppArmor 的限制是有选择性的，因此系统上的某些程序可能会受到限制，而其他程序则不会。选择性限制允许管理员灵活地关闭有问题的配置文件以进行故障排除，同时限制系统的其他部分。
不受限制的程序在标准 Linux 自由访问控制 (DAC) 安全性下运行。AppArmor 增强了传统 DAC，因为首先在传统 DAC 下评估受限程序，如果 DAC 允许该行为，则咨询 AppArmor 策略。
AppArmor 支持按配置文件学习（投诉）模式，以帮助用户编写和维护策略。学习模式允许通过正常运行程序并学习其行为来创建配置文件。在 AppArmor 充分了解行为之后，配置文件可能会转为强制模式。虽然生成的配置文件可能比为特定环境和应用程序量身定制的手工配置文件更宽松，但学习模式可以大大减少使用 AppArmor 所需的工作量和知识，并增加重要的安全层。

##  2. 版本
AppArmor 有两个主要版本，即 `2.x` 系列（当前）和 `3.x` 系列（开发）。2.x 系列在其生命周期中看到了渐进式的改进，只是在语义兼容性方面出现了渐进式的中断。3.x 系列是 AppArmor 的重大扩展。两个主要系列都使用相同的基本策略语言，只有细微的语义差异。3.x 系列允许更多扩展的策略和细粒度的控制。

## 3. 包含 AppArmor 的发行版

 - [Annvix](https://annvix.org/)
 - [Arch Linux](https://archlinux.org/), documentation and Arch specific [notes](https://wiki.archlinux.org/title/AppArmor)
 - [CentOs](https://www.centos.org/), documentation and CentOS specific [notes](https://gitlab.com/apparmor/apparmor/-/wikis/Distro_CentOS)
 - [Debian](https://www.debian.org/), documentation and Debian specific [notes](https://gitlab.com/apparmor/apparmor/-/wikis/distro_debian)
 - [Gentoo](https://www.gentoo.org/)
 - [openSUSE](https://www.opensuse.org/) (integrated in default install), documentation and Suse specific [notes](https://gitlab.com/apparmor/apparmor/-/wikis/distro_suse)
 - [Pardus Linux](https://www.pardus.org.tr/en/home/)
 - [PLD](https://www.pld-linux.org/)
 - [Ubuntu](https://ubuntu.com/) (integrated in default install), documentation and Ubuntu specific [notes](https://gitlab.com/apparmor/apparmor/-/wikis/distro_ubuntu)

这些发行版的任何衍生产品也应该有 AppArmor 可用。[更新的 RPMS](http://download.opensuse.org/repositories/security:/apparmor/)可以在[openSUSE 构建服务](https://en.opensuse.org/Build_Service)中找到。这些不限于 SUSE 发行版。

##  4. 源代码
AppArmor 项目源代码分为内核模块（在 Linux 内核和 git 开发树中可用）和启动板中可用的用户空间工具。
###  4.1 Kernel
AppArmor 在 `2.6.36` 的上游内核中。内核模块 git 树中提供了早期版本：

 - [如何获取 AppArmor 内核源](https://gitlab.com/apparmor/apparmor/-/wikis/gittutorial)
 - 注意：master 分支不稳定，会时不时地rebase。发布分支将是稳定的，不会被重新定位。

`AppArmor v2.4` 兼容性补丁在稳定的内核分支中可用。例如 `v3.4-aa2.8` 或 `kernel-patches` 目录中的发行版 tarball。


###  4.2 Userspace

 - 当前稳定版本：[3.0.6](https://gitlab.com/apparmor/apparmor/-/wikis/Release_Notes_3.0.6)
 - 支持的版本：[2.13.6](https://gitlab.com/apparmor/apparmor/-/wikis/Release_Notes_2.13.6)
 - 支持的版本：[2.12.3](https://gitlab.com/apparmor/apparmor/-/wikis/Release_Notes_2.12.3)
 - 支持的版本：[2.11.3](https://gitlab.com/apparmor/apparmor/-/wikis/Release_Notes_2.11.3)
 - 生命周期结束版本：[2.10.6](https://gitlab.com/apparmor/apparmor/-/wikis/Release_Notes_2.10.6)
 - [用户空间工具](https://launchpad.net/apparmor)
[如何获取 AppArmor 用户空间工具](https://gitlab.com/apparmor/apparmor/-/wikis/launchpadtutorial)


##  5. AppArmor 配置文件
开发 AppArmor 配置文件位于 `Bazaar` 存储库中。安装发行版的 Bazaar 软件包后，可以使用以下命令下载它们：

```bash
 git clone git://git.launchpad.net/apparmor-profiles
```


找到与您的发行版和版本匹配的子目录，并在里面查找当前正在开发的各种配置文件。您可以通过将配置文件复制到 `/etc/apparmor.d` 并重新启动 AppArmor 来使用配置文件：

```bash
$ /etc/init.d/apparmor restart
 restart apparmor     # if upstart is being used, with initscripts that have been fully updated to support upstart
```
**如何创建或修改 AppArmor 配置文件**

 - [使用工具创建和修改 AppArmor 策略](https://gitlab.com/apparmor/apparmor/-/wikis/Profiling_with_tools)
 - [手动创建和修改 AppArmor 策略](https://gitlab.com/apparmor/apparmor/-/wikis/Profiling_by_hand)
 - [AppArmor 文档](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)




##  6. 命令
AppArmor [手册](https://gitlab.com/apparmor/apparmor/-/wikis/ManPages)
Ubuntu Profile enforcement:

 - [apparmor_parser](https://manpages.ubuntu.com/manpages/kinetic/en/man8/apparmor_parser.8.html)：将 AppArmor 配置文件加载到内核中
 - [aa-audit](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-audit.8.html)：将 AppArmor 安全配置文件设置为审核模式
 - [aa-enforce](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-enforce.8.html)：设置 AppArmor 安全配置文件以强制模式被禁用或抱怨模式。
 - [aa-complain](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-complain.8.html)：将 AppArmor 安全配置文件设置为抱怨模式

Ubuntu Monitoring tools:

 - [aa-status](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-status.8.html)：显示有关当前 AppArmor 策略的各种信息
 - [aa-notify](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-notify.8.html)：显示有关记录的 AppArmor 消息的信息
 - [aa-unconfined](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-unconfined.8.html)：输出带有 tcp 或 udp 端口​​但没有 AppArmor 的进程列表已加载配置文件

Ubuntu Profile development:

 - [aa-autodep](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-autodep.8.html)：猜测基本的 AppArmor 配置文件要求
 - [aa-logprof](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-logprof.8.html)：用于更新 AppArmor 安全配置文件的实用程序
 - [aa-genprof](https://manpages.ubuntu.com/manpages/kinetic/en/man8/aa-genprof.8.html)：AppArmor 的配置文件生成实用程序
 - [mod_apparmor](https://manpages.ubuntu.com/manpages/kinetic/en/man8/mod_apparmor.8.html)：Apache 的细粒度 AppArmor 限制
 - [aa_change_hat](https://manpages.ubuntu.com/manpages/kinetic/en/man2/aa_change_hat.2.html)
 - [aa_change_profile](https://manpages.ubuntu.com/manpages/kinetic/en/man2/aa_change_profile.2.html)：更改任务配置文件
 - [PAM plugin](https://gitlab.com/apparmor/apparmor/-/wikis/pam_apparmor)：可用于根据在用户或任务级别完成的身份验证附加配置文件


##  7. 教程

 - [使用工具创建和修改 AppArmor 策略](https://gitlab.com/apparmor/apparmor/-/wikis/Profiling_with_tools)
 - [手动创建和修改 AppArmor 策略](https://gitlab.com/apparmor/apparmor/-/wikis/Profiling_by_hand)
 - [将 mod_apparmor 与 Apache 一起使用来限制 Web 应用程序](https://gitlab.com/apparmor/apparmor/-/wikis/mod_apparmor_example)- DRAFT
 - [使用 AppAmor 和 libvirt 来限制虚拟机](https://gitlab.com/apparmor/apparmor/-/wikis/Libvirt)
 - [将 AppArmor 与 PAM 集成以实现基于登录的策略](https://gitlab.com/apparmor/apparmor/-/wikis/Pam_apparmor_example)
 - [使用 AppArmor 进行基于角色的访问控制 (RBAC)](https://gitlab.com/apparmor/apparmor/-/wikis/AppArmorRBAC) - 草稿
 - [使用 AppArmor 实现多级安全 (MLS)](https://gitlab.com/apparmor/apparmor/-/wikis/AppArmorMLS) - DRAFT
 - [使用 AppArmor 限制和控制通过 wine 运行的 Windows 应用程序](https://gitlab.com/apparmor/apparmor/-/wikis/AppArmorWine)- DRAFT
 - [使用完整的系统策略-](https://gitlab.com/apparmor/apparmor/-/wikis/FullSystemPolicy) DRAFT
 - [如何在 systemd 中使用 AppArmor](https://gitlab.com/apparmor/apparmor/-/wikis/AppArmorInSystemd) - DRAFT

参考：

 - [AppArmor](https://gitlab.com/apparmor/apparmor/-/wikis/home#reporting-bugs)
 - [AppArmor Documentation](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)

