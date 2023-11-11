之前我们谈到了网络报文的发送,以及这些报文对应了Nginx中的网络事件,比如像accept建立一条新连接,其实是收到了一条读事件,这么说大家可能觉得比较抽象,接下来我们通过抓包来分析建立三次握手时是怎么样让 Nginx 收到读事件，使用的抓包工具是 Wireshark

　　这里我们访问我们之前搭建的一个静态资源Web服务器;

　　我们可以访问它,这个IP对应着我们的Nginx服务器;我们访问这个服务器的时候对它抓包
　　![在这里插入图片描述](https://img-blog.csdnimg.cn/a9a698f91d6c41aa836b92261199a284.png)
首先,我们下载安装Wireshark这么一个网络分析器;然后对Nginx这个所在的ip和对应的端口进行抓包;
![在这里插入图片描述](https://img-blog.csdnimg.cn/9078f99845574c0c9386f86e05fb0033.png)
[Wireshark 使用入门 参考网络链接](https://www.cnblogs.com/cocowool/p/wireshark_tcp_http.html)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3b0c2456f650490987a270dbd2a887bf.png)
 192.168.1.114是我们客户端的ip

　　180.101.49.42 是我们nginx服务器端的ip,我们对它进行三次握手时,会首先向它发送一个SYN包;

　　 在 TCP 层主要说两件事情：

浏览器首先会打开这个页面，本地打开了一个 57852 端口，而 Nginx 启动的是 443 端口。

 - TCP 层主要做的是进程与进程之间通讯这件事。
 - IP 层主要解决机器与机器之间怎样互相找到的问题。

![在这里插入图片描述](https://img-blog.csdnimg.cn/5e1179095af840dda671490e11686e06.png)
**三次握手也就是 windows 先向 Nginx 发送了一次 [SYN]，那么相反的 Nginx 所在的服务器也会向 windows 发送一个 [SYN]，这个时候 Nginx 是没有感知到的，因为这个连接还是处于半打开的状态。直到这台 windows 服务器再次发送 [ACK] 到 Nginx 所在的服务器之上时，Nginx 所在的操作系统才会去通知 Nginx 我们收到了一个读事件，这个读事件对应是建立一个新连接，所以此时 Nginx 应该调用 Accept 方法去建立一个新的连接。**

以上我们通过 Wireshark 抓包演示了正常的三次握手是怎么样引发一个读事件来使得 Nginx 去处理这样一个读事件来建立新的连接的。

总结
这篇文章主要讲解了网络事件，并通过抓包来分析 Nginx 网络事件，这对我们理解 Nginx 异步处理框架是非常有帮助的，包括 OpenResty 也是强依赖于网络事件以及事件分发的。
