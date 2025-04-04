

「终端」，给我们大多数人最深刻的印象，可能就是某一次在搜索教程的时候，里面出现了「在 Spotlight 里面搜索终端，在出现的小黑框里面输入 XX 命令」这样的话语。没错，躺在应用列表中的「终端」，正是我们除了利用「图形界面」外，和计算机进行沟通交流的另一种途径。

终端并不是专业用户的专利，实际上，各类用户都可以利用终端。熟练的自动化玩家，可以利用终端进行快速的脚本执行；开发者在日常的开发工作中，则会使用终端进行复杂的命令操作；即使是初入 Mac 的新手玩家，也可以利用终端对系统隐藏的功能进行设置……
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5bb19adfaa8768b4a3cfe149130edfe3.png)
但是，macOS 自带的终端 Terminal.app 在功能上和一些专为 macOS 设计的第三方终端还是有所差距的。Mac 第三方终端中的佼佼者 —— [iTerm](https://iterm2.com/)，不仅受专业用户青睐、还有着强大的社区支持，即使是普通人也能快速上手。因此这篇文章，我想为大家介绍我心目中 macOS 上面最好、最强大的终端 —— iTerm。

## 到底什么是终端？
所谓的终端（Terminal）其实很好理解，它就是一个供输入、输出文本命令的窗口。无论是 macOS 自带的朴素的 Terminal 应用，还是作为本文主角的更加华丽的 iTerm，本质上都是一个窗口。

使用终端时还有一个容易与之混淆的概念「Shell」，这里我们借用另一篇文章[《告别 Windows 终端的难看难用，从改造 PowerShell 的外观开始》](https://sspai.com/post/52868)里的一个比喻来厘清两者关系：

Terminal 就像是一个人的衣服，可以有各种颜色、形状甚至功能；那么 Shell 则是这个人，你和他沟通来获得信息，并且他可以在与你沟通的过程中事先告诉你许多信息、提醒，甚至自动帮你补充你想说的话。

![终端是用户和电脑沟通的途径](https://i-blog.csdnimg.cn/blog_migrate/4be8423ca919d5f2483f32ed6fba8f09.png)
macOS 内置的 Terminal App、Windows 最近广为宣传的 Windows Terminal 以及 Linux 上面的各种「终端模拟器」等等，都是终端工具 —— Terminal。今天的主角 iTerm，则是一个专为 macOS 开发的第三方终端工具。

推荐阅读：[《微软为什么这么执着浏览器？3 个 Google I/O 上容易被忽视的亮点 | 奏折 21》](https://sspai.com/post/54652) 中有关 Windows Terminal 的介绍。

我们在 iTerm 等终端工具中，可以输入「脚本」、「命令」等，执行某种操作，来和计算机进行沟通。这是现代计算机中「终端工具」最大的功能和作用。

## 什么让 iTerm 脱颖而出？
### 强大的、专为 Mac 开发的富余功能
将 iTerm 和其他终端工具区分开来的，就是它丰富、强大、且专为 Mac 设计的功能。这些功能在 iTerm 的官网上足足介绍了有 30 余个。其中，最让我印象深刻的，也是我目前几乎离不开的功能就是：

- 全面的快捷键支持，上下左右任意分屏，以及通过快捷键建立 Quake 窗口
- 通过 Profile 配置文件保存多个窗口配置，能够一键套用现成配置
- 专为 macOS 编写的 Shell 集成脚本
- 利用 Metal 框架的 GPU 渲染加速
- 
其他诸如「剪贴板历史记录」、「图片显示」、「密码管理」等等，也都是 iTerm 所拥有的强大功能。这些为 Mac 开发设计，并为 Mac 优化的功能，让 iTerm 成为 macOS 上面终端工具的不二之选。在后文中我会对这些功能和使用场景进行具体详细的介绍。

### 丰富的社区主题支持
除了上面提到的 iTerm 本身强大的诸多功能以外，iTerm 还获得有社区强大的主题、配色支持，网上别人晒出的靓丽主题，你几乎都可以一键安装。目前 GitHub 上面最大的开源终端主题库，就是以 iTerm 命名的 [mbadolato/iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes)。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f352f7eee5ce80afda30dff5ea98a2e9.png)
参考：
- [macOS 最佳命令行客户端：iTerm | 使用详解](http://silenceallat.top/save_html/%E5%B0%91%E6%95%B0%E6%B4%BE/file/macOS%20%E6%9C%80%E4%BD%B3%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%AE%A2%E6%88%B7%E7%AB%AF%EF%BC%9AiTerm%20_%20%E4%BD%BF%E7%94%A8%E8%AF%A6%E8%A7%A3%20-%20%E5%B0%91%E6%95%B0%E6%B4%BE.html)
