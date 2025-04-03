# Windows10把两张图片合并成一张图片

## 1. 背景
相比截图功能，在 Google 的 Chrome 浏览器上，整页截屏功能仍需要安装额外的插件才能完成，这一点 微软的 `bing` 要方便很多。

微软已经向 `Edge Canary` 推出 「完整网页截图」功能，用户现在可以轻松捕获整个网页的屏幕截图，而无需借助第三方扩展或截图工具。

要访问该功能，只需按 `Ctrl + Shift + S` 或从菜单中选择「网页捕获」。然后，您可以选择 “任意选择” 或 “整页”。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de13a2272432786baf72f1a8ad2915a0.png)


也可以点击设置“`页面捕捉`”。

另外还可以右击鼠标，选择“检查”，然后`ctrl + shift + p`， 搜索“`截图`”，可以看到四中截图方式
。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6869268f7f701e1140f9cf7d71f71443.png)
以上这几种关于截图还不能满足一种情况，那就是如何把滚动鼠标选中的范围进行截图呢。

也没仔细搜目前的第三方软件。

但windows 自带的软件`画图`可以实现多图拼接。

## 2. "画图"实现多图拼接
选中你需要合并的第一张图片，然后“右击->打开方式->画图”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e55512a8a5edc5a76f718ecf247fba04.png)
选中图片右下角边框（如下图，红色框内），拉取你合并图片的所需位置。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9b4a953e4a64a2beb56fbe79353582a1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/38e2c745e9c39269c7044483c71f8611.png)
点击“`粘贴`->`粘贴来源`”，选择第二张需要合并的图片。第二张图片不要放开，移动至你拉取的空位。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9ec97fc919ffce43ed8771e24f4c5e97.png)
选择你所需要的格式进行保存。

图片效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eaaf5859754d36fb502244d304727bf7.png)

