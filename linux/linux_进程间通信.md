## 进程间通信发展
linux下的进程通信手段基本上是从Unix平台上的进程通信手段继承而来的。而对Unix发展做出重大贡献的两大主力AT&T的贝尔实验室及BSD（加州大学伯克利分校的伯克利软件发布中心）在进程间通信方面的侧重点有所不同。

前者对Unix早期的进程间通信手段进行了系统的改进和扩充，形成了`“system V IPC`”，通信进程局限在单个计算机内；

后者则跳过了该限制，形成了基于套接口（`socket`）的进程间通信机制。

Linux则把两者继承了下来

早期UNIX进程间通信

基于System V进程间通信

基于Socket进程间通信

POSIX进程间通信。

```bash
UNIX进程间通信方式包括：管道、FIFO、信号。

System V进程间通信方式包括：System V消息队列、System V信号灯、System V共享内存

POSIX进程间通信包括：posix消息队列、posix信号灯、posix共享内存。
```

由于Unix版本的多样性，电子电气工程协会（IEEE）开发了一个独立的Unix标准，这个新的ANSI Unix标准被称为计算机环境的可移植性操作系统界面（PSOIX）。现有大部分Unix和流行版本都是遵循POSIX标准的，而Linux从一开始就遵循POSIX标准；

BSD并不是没有涉足单机内的进程间通信（socket本身就可以用于单机内的进程间通信）。事实上，很多Unix版本的单机IPC留有BSD的痕迹，如4.4BSD支持的匿名内存映射、4.3+BSD对可靠信号语义的实现等等。


## 进程间通信定义

每个进程各自有不同的用户地址空间,任何一个进程的全局变量在另一个进程中都看不到，所以进程之间要交换数据必须通过内核,在内核中开辟一块缓冲区,进程A把数据从用户空间拷到内核缓冲区,进程B再从内核缓冲区把数据读走,内核提供的这种机制称为进程间通信（IPC）。
## 进程间通信目的

 - 数据传输 
 一个进程需要将它的数据发送给另一个进程，发送的数据量在一个字节到几M字节之间

 - 共享数据 
 多个进程想要操作共享数据，一个进程对共享数据
 
 - 通知事 
 一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要通知父进程）。
 
 - 资源共享 
 多个进程之间共享同样的资源。为了作到这一点，需要内核提供锁和同步机制。
 
 - 进程控制
   有些进程希望完全控制另一个进程的执行（如Debug进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

## 进程间通信方式

 1. 管道（pipe）,流管道(s_pipe)和有名管道（FIFO）
 2. 信号（signal）
 3. 消息队列
 4. 共享内存
 5. 信号量
 6. 套接字（socket)
### 匿名管道通信
匿名管道( pipe )：管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。
#### 1、管道具有以下两种局限性：
· 管道为半双工的；
· 管道只能在具有公共祖先的两个进程之间使用，通常，一个管道由一个进程创建，在进程调用fork之后，这个管道就能在父进程和子进程之间使用了。

#### 2、管道通过pipe函数创建：

```c
#include<unistd.h>
int pipe(int fd[2]);      //成功返回0，出错返回-1
```

经由参数fd返回两个文件描述符：fd[0]为读而打开，fd[1]为写而打开。

#### 3、 下图展示了父子进程之间的管道通信关系：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c0ce0cd177c8f0bd71f883a044a345fc.png)
父进程的fd[1]端的输出通过管道可以连接到子进程的fd[0]端，从而实现通信。
fork之后做什么取决于我们所需要的数据流方向。
例如对于父进程到子进程管道，父进程关闭管道的fd[0]端，子进程关闭fd[1]端，就形成了一条单工的通信方式，只能由父进程发送至子进程：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3000a81cecbd2bd9c819f95cc7887e9e.png)
常见的Linux命令 `"|"` 其实就是匿名管道，表示把一个进程的输出传输到另外一个进程，如：

```bash
echo "Happyjava" | awk -F 'j' '{print $2}'
# 输出 ava
```
### FIFO命名管道
FIFO即为命名管道，与无名管道不同的是，其可以在不相关的程序之间交换数据。

#### 1、FIFO其实是一种文件类型，创建FIFO类似于创建文件：

```c
#include<sys/stat.h>
int mkfifo(const char *path, mode_t mode);

int mkfifoat(int fd, const char *path, mode_t mode);    //成功返回0，失败返回-1
```

`mkfifoat`函数和`mkfifo`函数相似，但是mkfifoat函数可以被用来在fd文件描述符表示的目录相关的位置创建一个FIFO。

当我们用mkfifo或者mkfifoat创建FIFO时，都需要用open来打开它。

#### 2、FIFO有两种用途

· shell命令使用FIFO将数据从一条管道传送到另一条管道时，无须创建中间的临时文件。
· 客户进程-服务器进程应用程序中，FIFO用作汇聚点，在客户进程和服务器进程二者之间传递数据。

#### 3、FIFO应用实例

实例1 用FIFO复制输出流：
FIFO可以用于复制一系列shell命令中的输出流。这就防止了将数据写向中间磁盘文件。这类似于使用管道（|）来避免中间磁盘文件。

考虑这样一个过程，它需要对一个经过过滤的输入流进行两次处理。下图展示了这种安排：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b8a91cfe76b3a231d58faf50f83d38b0.png)
使用FIFO和UNIX程序tee就可以实现这样的过程而无需使用临时文件（tee程序将其标准输入同时复制到标准输出以及其命令行中命名的文件）：

```bash
mkfifo fifo1
prog3 < fifo1 &
prog1 < infile | tee fifo1 | prog2
```

创建FIFO，在后台启动prog3，从FIFO中读取数据。然后启动prog1，用tee将其输出发送到FIFO和prog2：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6033a4881f7879998c78559d4a754cc1.png)
实例2 使用FIFO进行客户进程-服务器进程通信：
如果有一个服务器进程，它与很多客户进程有关，每个客户进程都可以将其请求写到一个该服务器进程创建的“众所周知”的FIFO中（所有客户进程都知道该FIFO的路径名）。下图展示了这种安排：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2e799cd3dbdbdfa833d68139c7958589.png)
这类问题在于服务器进程如何将回答送回各个客户进程。不能使用单个FIFO，因为客户进程不知道该何时去读属于它们自己的相应。

一种解决办法就是：每个客户进程都在其请求中包含自身的进程ID。然后服务器进程为每个客户进程创建一个FIFO，用于回应：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f54b0837084f672ec5e649940e529bb.png)
实例3
 mkfifo <pipename> 命令创建一个命名管道，如：

```bash
mkfifo pipe
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/64d851a84436c1233d639b60bd066621.png)

## 消息队列

1、消息队列是消息的链接表，存储在内核中，有消息队列标识符标识。

`msgget`用于创建一个新的消息队列或打开一个现有队列。`msgsnd`用于将新消息添加到队列尾端。`msgrcv`用于从队列中去消息。：

```c
#include<sys/msg.h>
int msgget(key_t key, int flag);  
int msgsnd(int msqid, const void *ptr, size_t nbytes, int flag);  
ssize_t msgrcv(int msqid, void *ptr, size_t nbytes, long type, int flag);
```

我们并不一定以先进先出的顺序取消息，可以按照消息的类型字段取消息。

## 信号量

1、信号量与管道，FIFO以及消息队列不同。它是一个计数器，用于为多个进程提供共享数据对象的访问。

2、 为了获得共享资源，进程需要执行下列操作：

 - 1） 测试控制该资源的信号量；
 - 2） 若此信号量的值为正，则进程可以使用该资源。在这种情况下，进程会将信号量值减1，表示它使用了一个资源单位；
 - 3） 否则，若此信号量的值为0，则进程进入休眠状态，知道信号量的值大于0。进程被唤醒后，返回步骤1）。

当进程不在使用由一个信号量控制的共享资源时，该信号量值增1。如果有进程正在休眠等待此信号量，则唤醒它们。

3、 当我们想使用信号量时，首先通过函数semget来获取一个信号量ID：

```bash
#include<sys/sem.h>
int semget(key_t key, int nsems, int flag);
```

`semctl`函数包含了多种信号量操作：

```bash
#include<sys/sem.h>
int semctl(int semid, int semnum, int cmd, ... /* union semun arg*/);  
```

`semop`函数自动执行信号量集合上的操作数组：

```bash
#include<sys/sem.h>
int semop(int semid, struct sembuf semoparray[], size_t nops);
```

### 共享存储
1、共享存储允许两个或多个进程共享一个给定的存储区。

因为数据不需要在客户进程和服务器进程之间复制，所以这是最快的IPC。在多个进程同步访问一个给定存储区时，若服务器进程正在将数据放入存储区，则在它做完之前，客户进程不应该去取这些数据。

信号量用于同步共享存储访问。

2、shmget函数获得一个共享存储标识符：

```bash
#include<sys/shm.h>
int shmget(key_t key, size_t size, int flag);
```

`shmctl`函数对共享存储段执行多种操作：

```bash
#include<sys/shm.h>
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```

一旦创建了一个共享存储段，进程就可以调用shmat函数将其连接到它的地址空间中：

```bash
#include<sys/shm.h>
void *shmat(int shmid, const void *addr, int flag);
```

当对共享存储段的操作已经结束时，调用shmdt函数与该段分离，但这并不会删除标识符及其相关的数据结构。直至某个进程（一般是服务器进程）调用shmctl特地删除它为止：

```bash
#include<sys/shm.h>
int shmdt(const void *addr);
```

### socket

socket，即套接字是一种通信机制，凭借这种机制，客户/服务器（即要进行通信的进程）系统的开发工作既可以在本地单机上进行，也可以跨网络进行。也就是说它可以让不在同一台计算机但通过网络连接计算机上的进程进行通信。也因为这样，套接字明确地将客户端和服务器区分开来。

二、套接字的属性

套接字的特性由3个属性确定，它们分别是：域、类型和协议。

1、套接字的域

它指定套接字通信中使用的网络介质，最常见的套接字域是AF_INET，它指的是Internet网络。当客户使用套接字进行跨网络的连接时，它就需要用到服务器计算机的IP地址和端口来指定一台联网机器上的某个特定服务，所以在使用socket作为通信的终点，服务器应用程序必须在开始通信之前绑定一个端口，服务器在指定的端口等待客户的连接。另一个域AF_UNIX表示UNIX文件系统，它就是文件输入/输出，而它的地址就是文件名。

2、套接字类型

因特网提供了两种通信机制：流（stream）和数据报（datagram），因而套接字的类型也就分为流套接字和数据报套接字。这里主要讲流套接字。

流套接字由类型SOCK_STREAM指定，它们是在AF_INET域中通过TCP/IP连接实现，同时也是AF_UNIX中常用的套接字类型。流套接字提供的是一个有序、可靠、双向字节流的连接，因此发送的数据可以确保不会丢失、重复或乱序到达，而且它还有一定的出错后重新发送的机制。

与流套接字相对的是由类型SOCK_DGRAM指定的数据报套接字，它不需要建立连接和维持一个连接，它们在AF_INET中通常是通过UDP/IP协议实现的。它对可以发送的数据的长度有限制，数据报作为一个单独的网络消息被传输,它可能会丢失、复制或错乱到达，UDP不是一个可靠的协议，但是它的速度比较高，因为它并一需要总是要建立和维持一个连接。

3、套接字协议
只要底层的传输机制允许不止一个协议来提供要求的套接字类型，我们就可以为套接字选择一个特定的协议。通常只需要使用默认值。
参考链接：
[https://www.jianshu.com/p/4989c35c9475](https://www.jianshu.com/p/4989c35c9475)

[https://blog.csdn.net/gatieme/article/details/50908749](https://blog.csdn.net/gatieme/article/details/50908749)



