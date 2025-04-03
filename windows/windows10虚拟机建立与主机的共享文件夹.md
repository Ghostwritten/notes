虚拟机一般分区和磁盘空间都非常有限，因此经常通过共享目录的方式与主机进行磁盘空间共享，也方便在两个系统间互换信息。本文记录如何在VMware上配置windows10共享目录。
## 1. 虚拟机安装VMware Tools
### 第一种方法
在虚拟机中
按快捷键【win】+r，打开【运行】
输入 `D:\setup.exe`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f72228e1127a062e4628dfe3d24d7465.png)
执行安装，点击下一步即可。

### 第二种方法
虚拟机与主机设置共享目录，在虚拟机中需要有VMware Tools支持。首先打开虚拟机，一般没有安装过VM ware Tools的都会遇到底部的提示，根据提示，点击【安装Tools】
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99555177d94081f19982505b8337fd77.png)
之后Tools的安装包会自动被载入到光驱中，如下图所示
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d2c4e97fe2ea75e739556d9690924b52.png)
双击光驱，会提示是否安装Tools，根据提示，选择【是】，进入VMware Tools安装过程：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2f2088e82163423abb3fdc548328d4bc.png)
之后VMware Tools的安装向导会被打开，如下图所示，此次类似于基本的windows软件安装，保持默认配置，一路点击下一步或者确定进行安装。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f975b8c6a407650315480c9a43ae0a2f.png)
安装完成后重启![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f1c83c998c1c44884f2a76cb31c5c7e4.png)
## 2 设置共享目录
在支持windows10的vmware软件上配置共享目录比以前的版本更加简单了，只需要在虚拟机上进行配置即可。下面将详细截图。首先将虚拟机关机（注意：不是挂起/暂停），才能开始配置.
右击虚拟机点击【设置】
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9880791e7c57d7d02b76b60f3b04f3bc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b0623417e79aadae27a2136221d66898.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e153affb725b6422ddc66eac0c5358a8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1379fdcd2f4030031d3de7fd206c4d84.png)

