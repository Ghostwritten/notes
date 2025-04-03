![](https://i-blog.csdnimg.cn/blog_migrate/848e6039af1ae916dd21225ae6e07071.jpeg#pic_center)


## 现象
执行kubectl get node 无法获取集群状态。日志截图：

![image.png](https://i-blog.csdnimg.cn/blog_migrate/50352bc058e63e082c15b71b69f03dc7.png)
查看 message日志，发现报错存在OOM，并与应用测试的容器相关，截图如下：

![image.png](https://i-blog.csdnimg.cn/blog_migrate/fa45d43ead5971668d8719b188f04419.png)

## 分析
首先，定位最初的oom发生的时间点，是2023年12月15日，如图

![image.png](https://i-blog.csdnimg.cn/blog_migrate/60b97a08c137453c82acbdf42f43e033.png)


**按照正常逻辑来讲，应用实例做了 limit 限制，如果应用超出内存限制，应该被杀掉并且进行重新调度。
进一步发现，集群的 kube-apiserver 在2023年12月17日是挂掉了。当kube-apiserver挂掉时，Kubernetes的调度器和控制器无法与API服务器通信，这并不会导致Pod使用内存超出限制而被终止。
kubelet会采取以下行动之一：
l  OOM（Out of Memory）Killing：如果容器无法分配更多的内存，并且超出了限制，kubelet可能会触发OOM Killer，终止该容器。这是Linux内核中的一项功能，用于防止系统内存耗尽。
l  重启容器：kubelet也可以选择重启超出内存限制的容器，以尝试解决内存问题。重启容器可以释放内存并清除可能导致内存泄漏或过度消耗的状态。**

Kube-apiserver 挂掉的原因在日志中也明显可以看到是由于镜像没有正常拉取。

跟踪到最初镜像没有正常被拉取的时间。


**综合分析，由于containerd 更改了存储目录和镜像仓库访问认证，镜像仓库的访问认证错误修改，导致集群无法正常拉取镜像。Kube-apiserver启动异常。导致当应用实例占用内存超出限制时，并没有被杀掉，kubelet触发OOM Killer。**


参考：

- [https://stackoverflow.com/questions/9199731/understanding-the-linux-oom-killers-logs](https://stackoverflow.com/questions/9199731/understanding-the-linux-oom-killers-logs)
- [Linux OOM机制分析](https://www.cnblogs.com/MrLiuZF/p/15229868.html)
