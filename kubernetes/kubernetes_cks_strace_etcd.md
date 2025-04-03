
[strace](https://blog.csdn.net/xixihahalelehehe/article/details/118415249) 的全称是系统调用跟踪，意思是它是一个进程，在系统调用接口上就像一个边车，记录每个系统调用。[这篇 Medium 文章](https://medium.com/elements/diving-deeper-strace-9567ce531ee4)对大多数人如何在实践中使用 strace 给出了一个非常简单的解释，如果有人对细节感兴趣，[这篇文章](https://medium.com/@adminstoolbox/debugging-using-strace-efda7d65be1d)将给出一个彻底的演练。归根结底，您可以在 strace 中使用很多参数，而且都取决于具体情况。为了演示，我们将在调试 Kubernetes 操作时使用常用的。

一个例子是看看我们是否可以读取存储在 etcd 中的秘密。为了实现这一点，我们需要知道 etcd 正在运行的进程 ID


```bash
$ ps aux | grep etcd
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99a14972c8ad992315af24bf4426f252.png)

从上面的结果来看，第一个进程 ID 将是我们的目标，因为第二个进程是 kube-apiserver，第三个是我们刚刚执行的“grep”

```bash
sudo strace -p 4295
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/527baf3078d142c4eaf7ef18be59cd74.png)
我们应该看到很多操作被列出来。从这里，我们可以前往进程目录并查看其中包含的内容。

```bash
- sudo su
- cd /proc/4295/fd
- ls -l | grep 7
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37f28c8633947559b2e285daac7354b7.png)
此时，目录“7”似乎包含了 K8s 需要的信息。我们可以创建一个简单的秘密并尝试在其中找到值。

```bash
- kubectl create secret generic credit-card --from-literal ssecret=1111222233334444
#Make sure you are still in directory /proc/4295/fd. 
#"-A10" and "-B10" mean show 10 lines before and after the searching string.
- cat 7 | strings | grep 1111222233334444 -A10 -B10
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1522aa8eccbddddf8f5f65eb0ca78f81.png)

