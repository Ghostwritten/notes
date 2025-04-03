接下来我们来介绍reload重载配置文件的真相;

　　当我们更改了nginx配置文件的时候,我们都会执行`nginx -s reload`;那么我们执行这条命令的原因是希望nginx不能停止服务,始终还在处理新的请求的同时,把nginx的配置文件平滑的从旧的`nginx.conf`更新为新的nginx.conf;这样的一个功能对nginx来说非常的有必要,但是我们往往会发现,在我们执行之后,会发现nginx的worker进程的数量变多了;这是因为老的nginx配置的worker进程它长时间没有退出;当我们使用`stream`做四层反向代理的时候;可能这种场景会更多;下面我们通过来分析下nginx的reload流程来看一看nginx到底做了些什么?所谓优雅的退出和立即退出有怎么样的差别?
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fe31896766d226f35bc15edc281aa9a8.png)
reload流程

　　　

 - 第一步:在我们已经修改好`nginx.conf`后,需要向master进程发送`HUP`信号(实际上与我们在命令行执行`nginx -s reload`命令效果是一样的)；
 - 第二步:master进程在收到HUP信号以后尼,master进程会检验配置语法是否正确;也就是说我们并不一定需要在`nginx -s reload`之前先执行nginx -t 检验下配置文件是否正确;因为在第二步nginx的master进程一定会做这件事情;

 - 第三步:在配置语法完全正确以后,这个时候nginx的master进程就会打开新的监听端口;为什么要在master进程中打开新的监听端口尼?因为我们可能在nginx的conf文件中引入了新的之前没有打开过的监听端口;而所有的worker进程是master进程的子进程;子进程会继承父进程所有已经打开的端口;这是Linux操作系统所定义的;

 - 第四步:接下来,master进程会用新的nginx配置文件启动新的`worker`子进程;

 - 第五步:在启动新的worker子进程以后,`master`进程会向老的`worker`子进程发送QUIT信号;这个时候我们会发现QUIT信号和`TERM`,INT信号是不一样的,`QUIT`信号是请优雅的关闭子进程;这个时候我们要注意先后顺序,因为nginx要保证平滑;所以它一定要先启动新的worker子进程,再向老的worker子进程发送QUIT信号;

 - 第六步:老的worker子进程在收到QUIT信号以后尼,它首先会关闭监听句柄,处理完当前连接后就会结束进程;这个时候新的连接只会到新的worker子进程;虽然新的worker子进程开启,老worker子进程的关闭它们之间有个时间差,但是时间是非常快速的;

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3e779396c66b3c7be35dbfb0a7264728.png)
比如原先有四个绿色的worker子进程;它们使用了老的配置,当我们更改了`nginx.conf`配置文件以后;向master父进程发送`SIGHUP`信号或者执行`nginx -s reload`;master父进程会用新的配置文件启动四个黄色的worker子进程;所以此时四个新的黄色的worker子进程与四个旧的worker子进程是并存的;老的worker子进程在正常的情况下在处理完老的请求连接以后会关闭这个连接和老的worker子进程;但是在异常的情况下,比如说一些请求出问题了,客户端长时间没有处理,就会导致这个请求长时间占用在这个worker子进程上面,而这个worker子进程会一直存在,当然一些新的连接已经跑在了黄色的worker子进程中了,所以影响并不会很大,唯一会导致绿色的woker子进程会长时间存在,但也只会影响已经存在的连接而不会影响新的连接;这个时候有没有什么办法来处理尼？实际上nginx新的版本中提供了一个新的配置项`worker_shutdown_timeout` 表示最长会等待多长时间,也就是说master进程在启用黄色的worker子进程以后尼,它会加一个定时器worker_shutdown_timeout 定时器到期了以后尼如果worker子进程还没有退出尼那就立刻强制把老的worker子进程给退出掉;

以上就是nginx平滑启动新的nginx配置文件的流程;

补充阅读:

之前我们讲解 Nginx 命令行的时候，可以看到 Nginx 停止有两种方式，分别是 `nginx -s quit` 和 `nginx -s stop`，其中 stop 是指立即停止 Nginx，而 quit 是指优雅的关闭 Nginx，对应的信号也是同样的，还有我们之前提到的 reload 和热升级这样的过程中都涉及到了优雅的停止 Nginx。

那所谓的优雅的停止 Nginx 究竟是怎样一个过程呢，接下来让我一起来学习下吧。

所谓的优雅的关闭，是针对 worker 进程而言的，因为只有 worker 进程 才会处理请求。如果我们在处理一个连接的时候，不管连接此时对于请求是怎样一个作用，直接去关闭链接会导致用户收到错误，所以优雅地关闭就是指 Nginx 的 worker 进程 可以识别出当前连接没有正在处理请求，这个时候再把连接进行关闭。

对于某些请求 Nginx 无法做到优雅地关闭 worker 进程，比如当 Nginx 代理 websocket 协议的时候，在 websocket 后面进行通讯的 `frame` 桢里面，Nginx 是不解析他的桢的；Nginx 做 TCP 层或者 UDP 层反向代理的时候，也没有办法识别一个请求需要经历多少报文才算是结束；但是对于 HTTP 请求，Nginx 可以做到，所以优雅地关闭主要针对的是 HTTP 请求。

接下来我们去看一下优雅地关闭 worker 进程都有哪些流程。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/40916e419b64e85d44a1102d8f1366db.png)
首先第一步会设置一个定时器，在 nginx.conf 中可以配置一个 `worker_shutdown_timeout`，配置完 `worker_shutdown_timeout` 之后，会加一个标志位，表示进入优雅关闭流程了。

第二步会先关闭监听句柄，要保证所在的 worker 进程不会再去处理新的连接。

接下来会先去看连接池，因为 Nginx 为了保证对资源的利用是最大化的，经常会保存一些空闲的连接，但是没有断开，这时候会首先关闭空闲连接。

第四步是可能非常耗时的一步，因为 Nginx 不是主动的立刻关闭，是通过第一步添加的标志位，然后在循环中每当发现一个请求处理完毕，就会把这个请求使用的连接关掉，所以在循环中等待关闭所有的时间可能会很长。当设置了 worker_shutdown_timeout 的时候，即使请求还没处理完，当时间到了之后这些请求都会被强制关闭，也就是说优雅地关闭只完成了一半，有一部分连接是立即停止的。

因此在以下两个条件：当所有循环中连接被优雅地关闭，或者达到了 worker_shutdown_timeout 时间定时器以后，worker 进程都会立即退出。

这篇文章主要讲解了 worker 进程优雅关闭的一个过程，很多时候我们都会用到 Nginx 优雅关闭这样一个特性，那么在这一个特性失效的时候，我们需要考虑 Nginx 有没有能力去判定一个连接此时应当被正确的关掉；或者说如果出现了错误、有些模块或者有些客户端不能正常的处理请求时，Nginx 需要有一些例外的措施，比如 worker_shutdown_timeout 来保证 Nginx 老的 worker 进程可以正常的退出掉。
