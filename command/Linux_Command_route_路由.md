#  Linux Command route 路由
tags: 网络


大多数主机一般都是驻留在只连接一台路由器的网段上。由于只有一台路由器，因此不存在选择使用哪一台路由器将数据包发送到远程计算机上去的问题，该路由器的 IP 地址可作为该网段上所有计算机的缺省网关。

但是，当网络上拥有两个或多个路由器时，用户就不一定想只依赖缺省网关了。实际上可能想让某些远程 IP 地址通过某个特定的路由器来传递，而其他的远程 IP 则通过另一个路由器来传递。在这种情况下，用户需要相应的路由信息，这些信息储存在路由表中，每个主机和每个路由器都配有自己独一无二的路由表。大多数路由器使用专门的路由协议来交换和动态更新路由器之间的路由表。但在有些情况下，必须人工将项目添加到路由器和主机上的路由表中。route 命令就是用来显示、人工添加和修改路由表项目的。该命令可使用如下选项：

（1）route print：本命令用于显示路由表中的当前项目，在单个路由器网段上的输出结果如图所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/885c6672fc084eb1ab854a1fbaa694ca.png)

（2）route add：使用本命令，可以将路由项目添加给路由表。

例如，如果要设定一个到目的网络 209.99.32.33 的路由，其间要经过 5 个路由器网段，首先要经过本地网络上的一个路由器 IP 为 202.96.123.5，子网掩码为 255.255.255.224，那么用户应该输入以下命令：

```bash
route add 209.99.32.33 mask 255.255.255.224 202.96.123.5 metric 5
```

（3）route change：可以使用本命令来修改数据的传输路由，不过，用户不能使用本命令来改变数据的目的地。下面这个例子将上例路由改变采用一条包含 3 个网段的路径：

```bash
route add 209.99.32.33 mask 255.255.255.224 202.96.123.250  metric 3  
```

（4）route delete  使用本命令可以从路由表中删除路由。例如：route delete 209.99.32.33

参考：

 - [route命令（详细）](https://blog.csdn.net/u013485792/article/details/51700808)
 - [route command in Linux with Examples](https://www.geeksforgeeks.org/route-command-in-linux-with-examples/)
 - [route(8) — Linux manual page](https://man7.org/linux/man-pages/man8/route.8.html)
 - [7 Linux Route Command Examples (How to Add Route in Linux)](https://www.thegeekstuff.com/2012/04/route-examples/)
 - [An introduction to Linux network routing](https://opensource.com/business/16/8/introduction-linux-network-routing)

