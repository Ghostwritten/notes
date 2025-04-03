

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3edb5520a23727eed58664a8d4fc2ef9.png)


## 1. VMware 16 pro 特性
### 容器和Kubernetes支持

 - 使用vctl CLI 构建/运行/拉/推容器映像。
 - 支持在Workstation Pro顶部运行的KIND kubernetes集群。

> 注意：需要Windows 10 1809或更高版本

### 新来宾操作系统支持

 - RHEL 8.2
 - Debian的10.5
 - Fedora的32
 - CentOS的8.2
 - SLE 15 SP2 GA
 - FreeBSD 11.4
 - ESXi 7.0
 - 来宾中对DirectX 11和OpenGL 4.1的支持

### 沙盒图形
通过从vmx中删除图形渲染并将其作为单独的沙箱进程运行，可以增强虚拟机的安全性。

### USB 3.1控制器支持
虚拟机虚拟XHCI控制器从USB 3.0更改为USB 3.1，以支持10 Gbps。

### 更大的虚拟机

 - 32个虚拟CPU
 - 128 GB虚拟内存
 - 8 GB虚拟图形内存

> 注意：运行具有32个vCPU的虚拟机要求您的主机和客户机操作系统都支持32个逻辑处理器。



### 暗模式
Workstation 16 Pro支持暗模式，以优化用户体验。

> 注意：要求主机操作系统为Windows 10 1809或更高版本

```bash
File: C:\Users\"user"\AppData\Roaming\VMware\preferences.ini

Add line at end: wsFeatureDarkModeSupported = "TRUE"

Save file. Reload VMware Workstation app. Theme returns.
```

### vSphere 7.0支持
在Workstation 16中，您可以执行以下操作：

 - 连接到vSphere 7.0。
 - 将本地虚拟机上载到vSphere 7.0。
 - 将在vSphere 7.0上运行的远程虚拟机下载到本地桌面。

### 性能改进

 - 提高文件传输速度（拖放，复制和粘贴）
 - 改善了虚拟机关闭时间。
 - 改进的虚拟NVMe存储性能。
 - 改进的辅助功能支持
 - 添加了可访问性改进，因此Workstation Pro符合WCAG 2.1标准。

## 2. 升级方法

 - 旧 VMware 15 pro 不用卸载
 - 下载 [VMware 16 pro](https://www.vmware.com/hk/products/workstation-pro/workstation-pro-evaluation.html)
 - 打开安装（按照步骤来会自动检测到本地旧VMware）
 - 输入密钥


```bash
YF390-0HF8P-M81RQ-2DXQE-M2UT6
ZF3R0-FHED2-M80TY-8QYGC-NPKYF
ZF71R-DMX85-08DQY-8YMNC-PPHV8
```

 - 升级完成

