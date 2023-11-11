#  Linux Command Nano 编辑器
tags: 编辑器


##  1. 简介

[GNU](https://www.gnu.org/) nano是一个简单的基于终端的文本编辑器。虽然不如 Emacs 或 Vim 强大，但它易于学习和使用。Nano 非常适合对现有配置文件进行小幅更改或编写简短的纯文本文件。它最初是作为非免费 Pico 编辑器的免费替代品而创建的。Pico 是华盛顿大学 Pine 电子邮件套件中使用的基于终端的编辑器。

Nano 可用于终端窗口或系统控制台。

##  2. 安装
如 `macOS` 或 Linux 发行版，可能已经预装了 Nano 文本编辑器。

要检查，只需使用以下命令：`nano --version`

###  2.1 Debian/Ubuntu 

```bash
sudo apt update
sudo apt-get install nano
```
### 2.2 CentOS/RHEL 

```bash
yum install nano
```
##  3. 使用
### 3.1 创建并打开一个新文件

```bash
nano new_filename
```
每当您打开重要的配置文件时，建议使用`-w`选项。它将以标准格式打开文件。

```bash
sudo nano -w /etc/apache2/apache2.conf
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/3c8ff82a6283459a879fa2ed504295a6.gif#pic_center)

###  3.2 Ctrl+o 保存文件
按`Ctrl+o`
![在这里插入图片描述](https://img-blog.csdnimg.cn/b40f0dbdc756481aafc7b105bfeda139.gif#pic_center)

###  3.3 Ctrl+k 剪切 Ctrl+u 粘贴



 - 剪切和粘贴一整行。移动到要剪切的行，然后按`Ctrl+k`。现在该行已移动到剪贴板，要粘贴它，请转到要粘贴的位置，然后按`Ctrl+u`

![在这里插入图片描述](https://img-blog.csdnimg.cn/1ecc4b6796074e03a3d456df67260626.gif#pic_center)


 1. 剪切和粘贴选定的文本。选择要剪切的文本，然后按`Ctrl+k`。现在文本被移动到剪贴板。要粘贴它，请转到要粘贴的位置，然后按`Ctrl+u`。
![在这里插入图片描述](https://img-blog.csdnimg.cn/334bc3363cf049158247fa5f9f964d8c.gif#pic_center)

###  3.4 Ctrl+w 搜索文件中的单词
按 Ctrl+w,它将询问要搜索的单词。输入单词
它将搜索单词并将光标放在单词第一次出现的第一个字母上。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c8be300be84e4d62ad25657dec91c7c8.gif#pic_center)

###  3.5 Ctrl+t 替换
在 nano 中启用拼写检查。首先，安装拼写检查包。

```bash
sudo apt install spell
```
然后它会要求输入密码，然后输入密码。然后按y，然后按回车。

要进行拼写检查，请先按 `Ctrl+t`
现在它会要求您替换不正确的单词
在此处输入要替换的单词
只要你按下回车键
![在这里插入图片描述](https://img-blog.csdnimg.cn/b44b6da8cd95483ebb93deed4febc42c.gif#pic_center)

###  3.6 CTRL + X 保存退出
要退出编辑器，请按CTRL + X。如果有更改，它会询问您是否保存它们。输入Y表示Y es或N表示No，然后按Enter。但如果没有更改，您将立即退出编辑器。

参考：

 - [How to Install and Use Nano Text Editor: A Beginner’s Tutorial](https://www.hostinger.com/tutorials/how-to-install-and-use-nano-text-editor)
 - [Nano Text Editor in Linux](https://www.geeksforgeeks.org/nano-text-editor-in-linux/)

