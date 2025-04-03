
## 1. 简介

在 ESXi 环境中创建数据存储(Datastore)的步骤如下:

登录 vSphere Web Client

打开 Web 浏览器,输入 ESXi 主机或 vCenter Server 的 IP 地址,使用有权限的账户登录。

在 ESXi 环境中创建数据存储(Datastore)的步骤如下:

登录 vSphere Web Client

打开 Web 浏览器,输入 ESXi 主机或 vCenter Server 的 IP 地址,使用有权限的账户登录。

导航到 ESXi 主机

在导航窗口中,选择要添加数据存储的 ESXi 主机。

选择创建新数据存储

在主机摘要选项卡中,点击"配置">"存储">"新建数据存储"。

选择数据存储类型

选择要创建的数据存储类型,例如 VMFS、NFS 或 iSCSI。

配置数据存储

根据所选类型,配置数据存储所需的详细信息。比如对于 VMFS 数据存储,需要选择磁盘设备;对于 NFS 数据存储,需要提供 NFS 服务器地址和共享路径;对于 iSCSI 数据存储,需要提供 iSCSI 目标和 LUN 信息。

为数据存储命名

输入数据存储的名称。

完成创建过程

查看配置摘要并完成创建过程。新创建的数据存储会显示在"存储"视图中。


根据您的存储硬件和需求选择合适的数据存储类型。在创建过程中系统会提示您输入所需的配置参数。创建完成后,新数据存储就可以用于存储虚拟机文件和其他资源

## 2. 清空磁盘

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50e88b3cc7000d33ea61a83ffd2bcdda.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/81530581fe3ffae1abac66681d2460f2.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/772744e0a2a7e545f767ba717bd64006.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/46a5bcb11a8c5b47e00c52ad2c29c124.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2efa7c5d403d866f187bbadaf5bc9e82.png)
失败
## 3. 删除表
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/41ba816e2a3fa99890b586ec84a94a7b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1ae7d629f8d1b8240976f7ba351d448d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/950bc49ec3e82b1437e58fe334342632.png)
成功。

## 4. 创建database

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e2224570ce17a6dba16358681871ba1f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a729ad7ad4f50d934b22e215649c5c29.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc3007cbaf14509e8ea2fd1ab369bc83.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f63c46af0bd43617fca3a05285d0ebc9.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fecebeb9a4d8ae6232ca6927cb2f13a8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/396ac4b210208f269f02af577306caa3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f996971c544a4ed38430273b4ebc301.png)

# demo

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/69cc343a5c8222f33f02e9eba2c6fc86.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d9bc1884319e95a5549c7ecb05d07c63.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e427d9ea8f95d1bf2d07c1e9c06ab43.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/27677be9c6eb5688ce4dba744044d6ce.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c5f3552c46993ba918b756f368488242.png)

