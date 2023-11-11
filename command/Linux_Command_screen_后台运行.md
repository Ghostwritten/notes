#  Linux Command screen 后台运行
tags: 系统管理



----

##  1. 背景

系统管理员经常需要SSH 或者telent 远程登录到Linux 服务器，经常运行一些需要很长时间才能完成的任务，比如系统备份、ftp 传输等等。通常情况下我们都是为每一个这样的任务开一个远程终端窗口，因为它们执行的时间太长了。必须等待它们执行完毕，在此期间不能关掉窗口或者断开连接，否则这个任务就会被杀掉，一切半途而废了。
##  2. 简介

GNU Screen是一款由GNU计划开发的用于命令行终端切换的自由软件。用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。

GNU Screen可以看作是窗口管理器的命令行界面版本。它提供了统一的管理多个会话的界面和相应的功能。

 - 会话恢复
只要Screen本身没有终止，在其内部运行的会话都可以恢复。这一点对于远程登录的用户特别有用——即使网络连接中断，用户也不会失去对已经打开的命令行会话的控制。只要再次登录到主机上执行screen -r就可以恢复会话的运行。同样在暂时离开的时候，也可以执行分离命令detach，在保证里面的程序正常运行的情况下让Screen挂起（切换到后台）。这一点和图形界面下的VNC很相似。
 - 多窗口
在Screen环境下，所有的会话都独立的运行，并拥有各自的编号、输入、输出和窗口缓存。用户可以通过快捷键在不同的窗口下切换，并可以自由的重定向各个窗口的输入和输出。Screen实现了基本的文本操作，如复制粘贴等；还提供了类似滚动条的功能，可以查看窗口状况的历史记录。窗口还可以被分区和命名，还可以监视后台窗口的活动。
 - 会话共享
Screen可以让一个或多个用户从不同终端多次登录一个会话，并共享会话的所有特性（比如可以看到完全相同的输出）。它同时提供了窗口访问权限的机制，可以对窗口进行密码保护。

GNU's Screen 官方站点：[http://www.gnu.org/software/screen/](http://www.gnu.org/software/screen/)


## 3. 语法

```bash
 screen [-AmRvx -ls -wipe][-d <作业名称>][-h <行数>][-r <作业名称>][-s ][-S <作业名称>]
```

参数说明

 - -A 　将所有的视窗都调整为目前终端机的大小。
 - -d <作业名称> 　将指定的screen作业离线。
 - -h <行数> 　指定视窗的缓冲区行数。
 - -m 　即使目前已在作业中的screen作业，仍强制建立新的screen作业。
 - -r <作业名称> 　恢复离线的screen作业。
 - -R 　先试图恢复离线的作业。若找不到离线的作业，即建立新的screen作业。
 - -s 　指定建立新视窗时，所要执行的shell。
 - -S <作业名称> 　指定screen作业的名称。
 - -v 　显示版本信息。
 - -x 　恢复之前离线的screen作业。
 - -ls或--list 　显示目前所有的screen作业。
 - -wipe 　检查目前所有的screen作业，并删除已经无法使用的screen作业。

常用screen参数

```bash
screen -S yourname -> 新建一个叫yourname的session
screen -ls -> 列出当前所有的session
screen -r yourname -> 回到yourname这个session
screen -d yourname -> 远程detach某个session
screen -d -r yourname -> 结束当前session并回到yourname这个session
```

在每个screen session 下，所有命令都以 ctrl+a(C-a) 开始。
◈ Ctrl + a " - 列出所有会话
◈ Ctrl + a 0 - 切换到会话号 0
◈ Ctrl + a n - 切换到下一个会话
◈ Ctrl + a p - 切换到上一个会话
◈ Ctrl + a S - 将当前区域水平分割为两个区域
◈ Ctrl + a l - 将当前区域垂直分割为两个区域
◈ Ctrl + a Q - 关闭除当前会话之外的所有会话
◈ Ctrl + a X - 关闭当前会话
◈ Ctrl + a \ - 终止所有会话并终止 Screen
◈ Ctrl + a ? - 显示键绑定。要退出，请按回车  
 - C-a ? -> 显示所有键绑定信息
 - C-a c -> 创建一个新的运行shell的窗口并切换到该窗口
 - C-a n -> Next，切换到下一个 window
 - C-a p -> Previous，切换到前一个 window
 - C-a 0..9 -> 切换到第 0..9 个 window
 - Ctrl+a [Space] -> 由视窗0循序切换到视窗9
 - C-a C-a -> 在两个最近使用的 window 间切换
 - C-a x -> 锁住当前的 window，需用用户密码解锁
 - C-a d -> detach，暂时离开当前session，将目前的 screen session (可能含有多个 windows)丢到后台执行，并会回到还没进 screen 时的状态，此时在 screen session 里，每个 window 内运行的 process (无论是前台/后台)都在继续执行，即使 logout 也不影响。
 - C-a z -> 把当前session放到后台执行，用 shell 的 fg 命令则可回去。
 - C-a w -> 显示所有窗口列表
 - C-a t -> Time，显示当前时间，和系统的 load
 - C-a k -> kill window，强行关闭当前的 window
 - C-a [ -> 进入 copy mode，在 copy mode 下可以回滚、搜索、复制就像用使用 vi 一样
 - C-b Backward，PageUp
 - C-f Forward，PageDown
 - H(大写) High，将光标移至左上角
 - L Low，将光标移至左下角
 - 0 移到行首
 - $ 行末
 - w forward one word，以字为单位往前移
 - b backward one word，以字为单位往后移
 - Space 第一次按为标记区起点，第二次按为终点
 - Esc 结束 copy mode
 - C-a ] -> Paste，把刚刚在 copy mode 选定的内容贴上

##  4. 安装screen

```bash
要在 Arch Linux 上安装 GNU Screen，请运行：
$ sudo pacman -S screen
在 Debian、Ubuntu、Linux Mint 上：
$ sudo apt-get install screen
在 Fedora 上：
$ sudo dnf install screen
在 RHEL、CentOS 上：
$ sudo yum install screen
在 SUSE/openSUSE 上：
$ sudo zypper install screen
```
##  5. 示例

```bash
screen -S david 
```

screen启动后，会创建第一个窗口，也就是窗口No. 0，并在其中打开一个系统默认的shell，一般都会是bash。所以你敲入命令screen之后，会立刻又返回到命令提示符，仿佛什么也没有发生似的，其实你已经进入Screen的世界了。当然，也可以在screen命令之后加入你喜欢的参数，使之直接打开你指定的程序，例如：

```bash
 screen vi david.txt
```

screen创建一个执行vi david.txt的单窗口会话，退出vi 将退出该窗口/会话。

### 5.1 创建新的 Screen 会话
让我们创建一个新的 Screen 会话并连上它。为此，请在终端中键入以下命令：

```bash
screen
```

现在，在此会话中运行任何程序或进程，即使你与此会话断开连接，正在运行的进程或程序也将继续运行。
从 Screen 会话脱离
要从屏幕会话中脱离，请按 `Ctrl + a 和 d`。你无需同时按下两个组合键。首先按 Ctrl + a 然后按 d。从会话中脱离后，你将看到类似下面的输出。

```bash
[detached from 29149.pts-0.sk]
```

这里，`29149` 是 `Screen ID`，`pts-0.sk` 是屏幕会话的名称。你可以使用 `Screen ID` 或相应的会话名称来连上、脱离和终止屏幕会话。
### 5.2 创建命名会话
你还可以用你选择的任何自定义名称创建一个 Screen 会话，而不是默认用户名，如下所示。

```bash
screen -S ostechnix
```

上面的命令将创建一个名为 `xxxxx.ostechnix` 的新 `Screen` 会话，并立即连上它。要从当前会话中脱离，请按 `Ctrl + a`，然后按 `d`。
当你想要查找哪些进程在哪些会话上运行时，命名会话会很有用。例如，当在会话中设置 LAMP 系统时，你可以简单地将其命名为如下所示。

```bash
screen -S lampstack
```

### 5.3 创建脱离的会话
有时，你可能想要创建一个会话，但不希望自动连上该会话。在这种情况下，运行以下命令来创建名为senthil 的已脱离会话：

```bash
screen -S senthil -d -m
```

也可以缩短为：

```bash
screen -dmS senthil
```

上面的命令将创建一个名为 senthil 的会话，但不会连上它。
### 5.4 列出屏幕会话
要列出所有正在运行的会话（连上的或脱离的），请运行：

```bash
screen -ls
```

示例输出：

```bash
There are screens on:
    29700.senthil   (Detached)
    29415.ostechnix (Detached)
    29149.pts-0.sk  (Detached)
3 Sockets in /run/screens/S-sk.
```

如你所见，我有三个正在运行的会话，并且所有会话都已脱离。
### 5.5 连上 Screen 会话
如果你想连上会话，例如 `29415.ostechnix`，只需运行：

```bash
screen -r 29415.ostechnix
或：
screen -r ostechnix
或使用 Screen ID：
screen -r 29415
```


要验证我们是否连上到上述会话，只需列出打开的会话并检查。

```bash
screen -ls
示例输出：
There are screens on:
        29700.senthil   (Detached)
        29415.ostechnix (Attached)
        29149.pts-0.sk  (Detached)
3 Sockets in /run/screens/S-sk.
```

如你所见，在上面的输出中，我们目前已连上到 `29415.ostechnix` 会话。要退出当前会话，请按 `ctrl + a d`。
### 5.6 创建嵌套会话
当我们运行 screen 命令时，它将为我们创建一个会话。但是，我们可以创建嵌套会话（会话内的会话）。
首先，创建一个新会话或连上已打开的会话。然后我将创建一个名为 nested 的新会话。

```bash
screen -S nested
```

现在，在会话中按 `Ctrl + a 和 c` 创建另一个会话。只需重复此操作即可创建任意数量的嵌套 Screen 会话。每个会话都将分配一个号码。号码将从 0 开始。
你可以按 `Ctrl + n` 移动到下一个会话，然后按 `Ctrl + p` 移动到上一个会话。


###  5.7 终止 Screen 会话
如果不再需要会话，只需杀死它。要杀死名为 senthil 的脱离会话：

```bash
screen -r senthil -X quit
```

或：

```bash
screen -X -S senthil quit
```

或：

```bash
screen -X -S 29415 quit
```

如果没有打开的会话，你将看到以下输出：

```bash
$ screen -ls
```

No Sockets found in /run/screens/S-sk.
更多细节请参照 man 手册页：

```bash
$ man screen
```

还有一个名为 Tmux 的类似的命令行实用程序，它与 GNU Screen 执行相同的工作。要了解更多信息，请参阅以下指南。
