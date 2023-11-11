虚拟机一般分区和磁盘空间都非常有限，因此经常通过共享目录的方式与主机进行磁盘空间共享，也方便在两个系统间互换信息。本文记录如何在VMware上配置windows10共享目录。
## 1. 虚拟机安装VMware Tools
### 第一种方法
在虚拟机中
按快捷键【win】+r，打开【运行】
输入 `D:\setup.exe`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421233541992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
执行安装，点击下一步即可。

### 第二种方法
虚拟机与主机设置共享目录，在虚拟机中需要有VMware Tools支持。首先打开虚拟机，一般没有安装过VM ware Tools的都会遇到底部的提示，根据提示，点击【安装Tools】
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421232655552.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
之后Tools的安装包会自动被载入到光驱中，如下图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421232731803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
双击光驱，会提示是否安装Tools，根据提示，选择【是】，进入VMware Tools安装过程：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421232817272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
之后VMware Tools的安装向导会被打开，如下图所示，此次类似于基本的windows软件安装，保持默认配置，一路点击下一步或者确定进行安装。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421232848501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
安装完成后重启![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421232903565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2 设置共享目录
在支持windows10的vmware软件上配置共享目录比以前的版本更加简单了，只需要在虚拟机上进行配置即可。下面将详细截图。首先将虚拟机关机（注意：不是挂起/暂停），才能开始配置.
右击虚拟机点击【设置】
![在这里插入图片描述](https://img-blog.csdnimg.cn/202004212331424.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421233303835.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421233802401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200421233920868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

