![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d4433b40f0a7631638b7077c922c9a87.png)



## 1. 背景
刚买mac min（2023年2月3日）不久，发现macOS的玩起来并不容易，勇习惯了windows系统的习惯，感觉 macOS 哪里都不习惯，好在这几天硬着头皮一点点摸索，慢慢将各个日常应用场景熟悉起来了。

比如：
- [macOS 各个快捷键使用](https://support.apple.com/zh-cn/HT201236)，比如：截图、输入法切换、快速查找、全屏、切换软件
- macOS 睡眠重新登录密码报错
- macOS 忘记密码如何重置密码
-  macOS 如何低价下载昂贵付费软件，例如：mainstage `49$` 、motion `328$`、 final cut pro `299$`、 compressor `49$` 、logic pro `199$` 、iboysofft `49.95$`
- macOS 如何干净的删除 dmg 安装软件
- macOS 如何美化配置 iterm2
- macOS 如何配置 clash 
- macOS 如何配置 Qv2ray

不过最近又发现了，一个问题，也是这篇文章的主题，我外接的西部数据与东芝硬盘小黑A3HDD硬盘无法写入，单单我的闪迪E61SSD 可以正常写入。这是为什么呢？脑袋条件反射的想苹果电脑这么辣鸡吗？

经过搜索发现原因是：移动硬盘默认的文件系统为NTFS格式，这种格式是微软Windows系统独有技术，在苹果Mac电脑上只能读取文件而无法往硬盘上写入。 

## 2. 让NTFS格式的移动硬盘正常读写方法

解决方法：

 1. 备份移动硬盘上所有的数据文件，将移动硬盘重新格式化为exFAT格式。
 2. 在不用第三方软件对移动硬盘进行一些魔法操作
 3. 下载第三方软件，例如：[iboysoft](https://iboysoft.com/)、[Paragon](https://www.paragon-software.com/home/ntfs-mac/) 、[Omi NTFS](https://apps.apple.com/us/app/ntfs-disk-by-omi-ntfs/id1585757563?mt=12)、Mounty、NTFS-3G、FUSE

最后，根据这篇 [7 Best Free NTFS for Mac Software - Roundup Review 2023](https://iboysoft.com/ntfs-for-mac/free-ntfs-for-mac.html)的评测，iboysoft 是不错的选择，我尝试了iboysoft ，淘宝五块买的破解版。由于我的macOS系统没有降低安全性，在“隐私与安全性”会出现一下提示：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b3a2490d70c1667b574fd692041cd2cf.png)
并且这个时候另一个现象是我的西部数据与东芝硬盘小黑A3HDD硬盘消失了，我还以为突然接触不良率，打开"磁盘工具"才发现是移动硬盘无法装载了。报错显示 `com apple diskmanagement disenter 29223`之类的报错。

使用“启动安全性实用工具”可确保 Mac 始终从您指定的启动磁盘以及合法的受信任操作系统启动。

那么如何降低系统安全性呢？

## 3. 打开“启动安全性实用工具”
1.将您的 Macbook 开机，然后在看到 Apple 标志后立即按住 Command (⌘)-R 键。Mac 会从 macOS 恢复功能启动。或者，是mac mini 的话一只按住启动键也可以进入恢复模式。

2.当系统要求您选择一个您知道相应密码的用户时，请选择这样的用户，点按“下一步”，然后输入用户的管理员密码。

3.当您看到“macOS 实用工具”窗口时，从菜单栏中选取“实用工具”>“启动安全性实用工具”。

4.当系统要求您进行认证时，点按“输入 macOS 密码”，然后选取管理员帐户并输入相应密码。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76c3a17e0d86533b91b01d7d4affe3ca.png)

## 4. 更改“安全启动”设置


我这里改为`降低安全性`。`允许用户管理来自被认可开发者的内核扩展`。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aa8e227a0b401c7e7bf852d83729f741.jpeg)
重新启动后。
外接移动硬盘，这时还没有成功挂载。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8752d68eb3e2487be3d7259f2be844f6.png)

点开“系统设置”——“隐私与安全性”，允许任何来源。

输入密码。就可以重新显示移动硬盘了。



