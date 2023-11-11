


--------------
##  1. 开启开发者工具

我们在 Chrome 浏览器中可以通过按下 `F12` 按钮或者右击页面，选择"`检查`"来开启开发者工具。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9b672453e2a4216b6e0f493017a43ec.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_9,color_FFFFFF,t_70,g_se,x_16)
也可以在右上角菜单栏选择 "更多工具"=》"开发者工具" 来开启：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c460cd96b07d4b2791e27f796c961688.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
##  2. Console 窗口调试 JavaScript 代码
打开开发者工具后，我们可以在 Console 窗口调试 `JavaScript`代码，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9f6710e6e3c4d4491b4eb89915f72c9.png)
上图中我们在 > 符号后输入我们要执行的代码 `console.log("runoob")`，按回车后执行。

我们也可以在其他地方复制一段代码过来执行，比如复制以下代码到 Console 窗口，按回车执行：

```bash
console.log("runoob-1")
console.log("runoob-2")
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/388ee9ab08ae4e5aa5f2709ad0cfeee8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_14,color_FFFFFF,t_70,g_se,x_16)
清空 Console 窗口到内容可以按以下按钮：
![在这里插入图片描述](https://img-blog.csdnimg.cn/5bbb005762e54c4b8172fff2795d743d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_15,color_FFFFFF,t_70,g_se,x_16)
##  3. Chrome snippets 小脚本
我们也可以在 Chrome 浏览器中创建一个脚本来执行，在开发者工具中点击 Sources 面板，选择 Snippets 选项卡，在导航器中右击鼠标，然后选择 `Creat new snippet` 来新建一个脚本文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c663b7997300475d929aa323e30f8d21.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
如果你没看到 `Snippets` ，可以点下面板上到 `>>` 就能看到了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f4b77e142d4415daa764308a32970af.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
点击 `Creat new snippet` 后，会自动创建一个文件，你只需在右侧窗口输入以下代码，然后按 `Command+S（Mac）`或 `Ctrl+S（Windows 和 Linux）`保存更改即可。

```bash
console.log("runoob-1")
console.log("runoob-2")
```
保存后，右击文件名，选择 "Run" 执行代码：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c0015158de4a44049fa76a5164688037.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
返回console：
![在这里插入图片描述](https://img-blog.csdnimg.cn/eab701b4583c4338826ee8a2865b4bd3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

参考链接：
[https://www.runoob.com/js/js-chrome.html](https://www.runoob.com/js/js-chrome.html)
