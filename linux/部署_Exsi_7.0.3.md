
##  1. 下载介质
下载 `VMware-VMvisor-Installer-7.0U3l-21424296.x86_64.iso` 安装 EXSI 7.0.3


可参考:
- [https://www.dinghui.org/vmware-iso-download.html](https://www.dinghui.org/vmware-iso-download.html)

## 2. u盘引导安装启动盘
- 工具 [https://www.ventoy.net/cn/](https://www.ventoy.net/cn/)
- 30G u盘
## 3. 硬件连接  

- 一个启动u盘
- 一个显示屏
- 一个鼠标
- 一块键盘
- 三台服务器部署 EXSI 7.0.3：

  - 192.168.21.91
  - 192.168.21.91
  - 192.168.21.91

## 4. 安装 EXSI 7.0.3
`F2` 进入 `BIOS`

设置 USB 启动顺序

![请添加图片描述](https://i-blog.csdnimg.cn/blog_migrate/9cb21aedffbf38605cf93e6b864b19cf.jpeg)

选择 “`VMware-VMvisor-Installer-7.0U3l-21424296.x86_64.iso`”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec67353956e2660f4c14b3b9d5b58691.jpeg)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/08ba9a5e9089a62d119d1f00458b6d3e.jpeg)
按 “ENTER”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/96e082bbbe4151e3e765a76e829a29b8.png)
按“F11”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1b55efdf21a1d0a345b24273ea96d60.png)

选择 ssd1 （465.76.GB）
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a69cd4551f2126603efa31935952d96.png)
选择“US” default
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8aec233a7d4a95c892514de2a880a41.png)

设置密码 `2023bsg.`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2c4cae8cd32ffa66898dc1ba9b97a7e0.png)
按 “F11” 安装
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48345aa122bd700687bbd6e41949b914.png)

安装中....


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a6f10dd960cb4d7870703d42950dbcd.png)

安装完毕，进行重启
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc788f066a0db6cca1ca7f835d5d64d7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f8d2e9fdb3483e770781b0e2e210d283.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ef690e15797a17a53be0a25c45ba6322.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18b50f83e41e6225514fc5cc972eee6d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e4facf630745d2343ed966afa0b75c3a.png)
按 “F2” ，进入系统配置。设置静态地址
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8f9fd1fa7200acf4aeaabf3a38ddd05.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/db6fa8628691a28250564d20cfbcf70c.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c5a117638af1e672a921a7337e4b881d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/57a75f329234f264144a43ae1437d0ae.png)
