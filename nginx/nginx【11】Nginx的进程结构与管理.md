接下来我们来看下Nginx的进程结构

　　Nginx其实有**两种进程结构**,一种是**单进程结构**,一种是**多进程结构**;单进程结构,其实不适用于生产环境,只适合我们做开发;因为在生产环境中我们需要保证Nginx足够健壮,以及Nginx可以利用**多核**的特性;而单进程的Nginx是做不到这一点的;所以默认的配置中都是打开多进程的nginx;我们来看下多进程的Nginx中;它的进程模型是什么样的?

　　会有一个父进程叫**master process**,它会有许多子进程;

　　子进程会分为两类:一类是**cache进程**,一类是**worker进程**;

　　为什么Nginx采用的是多进程结构,而不是多线程结构?

　　这要从Nginx最核心的一个目的,**Nginx要保证它的高可用性和可靠性,而当Nginx在使用了多线程的时候,因为线程是共享同一个地址空间的;所以当某一个第三方模块引发了一个地址空间导致的段错误时,在地址越界出现时会导致整个Ngxin进程全部挂掉;而当我们使用多进程这样的一个Nginx进程模型时,往往不会出现这样的一个问题;**

　　所以Nginx在做它的进程设计时,同样遵循了高可用,高可靠性这样的一个目的;比如说在maste进程中,通常第三方模块是不会在master进程这里加上自己的功能代码的;虽然Nginx在设计的时候是允许第三方模块在master进程中添加自己独有的自定义的一些方法;但是通常没有第三方模块会这么做;那么master进程它被设计用来的目的就是做worker进程的管理的;也就是说所有的worker进程是用来处理真正请求的,而master进程负责监控每个worker进程是不是在正常的工作;需不需要做重新载入配置文件,需不需要做热部署;而我们的cache在做缓存的时候,缓存需要在多个worker进程间共享的,而且缓存不光要被worker进程使用,还要被`Cache Manager`和`Cache Loader`进程使用;Cache Manager和Cache Loader也是为反向代理时后端发送的动态请求做缓存所使用的;那么Cache Loader 做缓存的载入Cache Manager做缓存的管理,实际上每个请求使用的缓存还是由worker进程来进行的;那么这些进程间的通讯都是使用共享内存来解决的;

　　那么有一个问题,为什么worker进程会很多?

　　我们从下图可以看到,Cache Manager和Cache Loader各有一个进程;master进程是父进程,肯定只有一个进程;那么为什么worker进程会很多？主要是因为Nginx采用**事件驱动模型**以后,它希望每一个woker进程从头到尾占有一颗CPU,所以往往我们不止要把woker进程的数量配置与我们服务器上的CPU核数一样以外;我们还需要把每一个worker进程与某一颗CPU核绑定在一起;**这样可以更好的使用CPU核上的CPU缓存,来减少缓存失效的命中率;**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2f82db832b063f06c800b8903f4a3304.png)
Nginx的多进程模型,由master作为父进程启动许多子进程,Nginx的父子进程之间是通过信号管理的,现在我们通过一个简单的演示,来直观地看下这些父子进程以及信号之间是怎么样工作的?

在Nginx配置当中,我启动了两个worker进程;

　　使用ps命令  `ps -ef |grep nginx` 查看nginx 进程
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/da7abe18d6d4563317dab783afa3538c.png)
有一个master进程是root用户启动的,进程的id号为 `13517`;使用ps命令可以查看到当前进程的id和父进程的id;

　　我们执行nginx的一个命令行命令就是 `nginx -s reload` 它会把之前的woker进程,包括cache进程,包括url的退出,再使用新的配置型去启动新的worker进程;
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37b89aa7ce43aaa47afa1f9c8b67d355.png)
　我们可以看到之前的两个子进程 3718,3719已经不见了,新出的两个子进程为3729,3730

　　**reload 和HUP信号它们的作用是相同的**;现在如果我们像Nginx的master 进程 `3714` 发送HUP信号;是不是会发生相同的结果尼?
　　![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f7d97250f8a7e845029d040e376f8f8.png)
我们可以看到发送信号HUP以后,之前的两个子进程都不在了,新启用的两个子进程;

　　像quit,stop 对应的也有信号,那么如果我像一个worker进程发送一个退出的信号,那么这个worker进程就会退出;但是进程在退出时,会自动的向它的父进程(master进程)发送一个默认行为SIGCHLD信号;

　　master进程收到了这样的一个信号以后,master进程就知道它的子进程退出了,它会新起一个worker进程,维持开启了的子进程个数的结构;
　　![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3281f6dd54400e32a8ca848e74167fb.png)
SIGTERM    发送到进程的 TERM 信号用于要求进程终止。

##  使用信号管理Nginx的父子进程
Nginx是一个多进程的程序,多进程之间进行通讯可以使用共享内存,信号等,但是我们在做进程间的管理时,通常只使用信号;下面我们来看下nginx的信号是怎样使用的?

　　能够发送和处理信号的有master进程,worker进程和nginx命令行;

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99d181be9f43fbd25ab04f2007f7d42c.png)
我们先来看master进程,因为master进程会启动worker进程,所以它管理worker进程的方式首先是监控worker进程有没有发送CHLD信号,因为nginx操作系统中规定,当子进程终止的时候会向父进程CHLD信号;所以如果worker进程由于一些模块出现BUG,导致worker进程意外的终止掉;那么master进程可以立刻通过CHLD发现这样一个事件;重新把worker进程拉起;那么master进程还会通过接收一些信号;来管理worker进程;

　　　　**master进程可以接收哪些信号尼?** TERM,INT,QUIT,HUP,USR1,USR2,WINCH;

 - TERM，INT  立刻停止
 - QUIT  优雅停止
 - HUP  重载配置文件
 - USR1 重新打开日志文件，做切割

　　　　TERM,INT,QUIT,HUP,USR1 可以通过nginx命令行 加上特定的命令向master进程发送命令的;

　　　　USR2,WINCH 只能通过kill Linux命令行直接向master进程发送信号;也就是说我们需要先找到master进程所在的pid,对这个pid 直接发送USR2和WINCH;

　　　　

　　　　**worker进程可以接收哪些信号尼?** TERM,INT,QUIT,HUP,USR1,WINCH;

　　　　为什么我们通常是不对worker进程发送信号尼?

　　　　因为我们希望由master进程来管理worker进程;如果直接对worker进程发送信号,也会在worker进程产生同样的结果;所以我们往往是对master进程进行管理;master进程收到信号以后,会去再把信号发送给worker进程;

　　　　**nginx命令行尼?**

　　　　当我们启动了nginx以后,nginx会把它的pid记录在一个文件中,通常是记录在nginx的安装目录中logs文件夹下的一个nginx.pid文件中,这个pid文件会记录master进程的进程pid;那这个时候我们再执行`nginx -s` 这样的命令行的时候,那么nginx这个工具命令行它就会读取pid文件中的master进程的pid,向这个pid同样去发生HUP,USR1,TERM,QUIT这样的信号;而这样的信号对应的我们命令就是reload,reopen,stop,quit;所以我们可以看到调用nginx命令行发生相应的命令和直接用kill发送信号产生的效果是一样的;
