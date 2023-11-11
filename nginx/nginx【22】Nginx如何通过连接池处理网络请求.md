之前我们谈到了nginx的读写事件,这些网络读写事件究竟是怎么应用到nginx上的尼?

　　还有我们谈到nginx使用了一个连接池来增加它的资源的利用率,下面我们来看下nginx的连接池究竟是怎么来使用的?
![在这里插入图片描述](https://img-blog.csdnimg.cn/a6e4aa056e304452b9d04d4cf70361fd.png)
　我们来看下上图中的右边的图,每一个worker进程里面都有一个独立的`ngx_cycle_t`这样的一个数据结构;

现在不要对它里面的细节来纠结,这里有三个主要的数组需要看一下;

![在这里插入图片描述](https://img-blog.csdnimg.cn/33417698ffaf4c9287604982532d9ede.png)

其中`connections` 这就是我们所谓的连接池;它指向的我们这个数组有多大尼?我们可以看下有一个配置项:

　　 打开nginx的官网在文档中找到`Core functionality`

![在这里插入图片描述](https://img-blog.csdnimg.cn/7c3fa02e0a6e4edcb12493a37131c207.png)
也就是核心功能,这个核心功能有一个选项`worker_connections;`

![在这里插入图片描述](https://img-blog.csdnimg.cn/d29517eef4c844a7bd0944de5813dff1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c086f6715a1e454ca2909f6d0c7f2852.png)
默认会有一个512大小的数组,这里的每一个数组就是一个连接,可以看到512其实是非常小的,因为我们nginx动则处理万,十万,百万级的计算,所以我们往往是需要去修改的;而且官方提示很清楚,这个连接不止去用于客户端的连接,也用于面向上有服务器的连接,所以如果我们做反向代理的时候,每个客户端意味着消耗我们两个`connections`;

![在这里插入图片描述](https://img-blog.csdnimg.cn/c1ef427d7727499a9ffc192fbadb05fc.png)
每一个连接自动的对应一个读事件和一个写事件,所以在`ngx_cycle_t`中还有个`write_events`;它们指向的数组的大小也跟`worker_connections`这个配置是一样的;所以connections连接事件,读事件,写事件是通过序号对应起来的;所以我们在考虑nginx能够释放多大性能的时候,首先要保证`worker_connections`足够我们使用;但是`worker_connections`所指向的数组,同时影响了我们所打开的内存,当我们配置了更大的`worker_connections`的时候,也就意味着nginx使用了更大的内存;所以每一个connections连接到底使用了多大的内存尼?

　　我们可以看一下,每一个`ngx_connection_s`这样的一个结构体在64位操作系统中它占用的字节数大约是`232`字节;具体会因nginx版本不同,可能会有微小的差异;

　　每一个`ngx_connection_s`这样的一个结构体它对应着两个事件,一个读事件,一个写事件,我们之前谈到网络事件谈到了它的许多特性;

　　那么在nginx中每一个事件对应一个结构体叫做`ngx_event_s`;每个事件对应的结构体它所对应的字节数是96字节;

　　所以我们在使用一个连接的时候它大概消耗的字节为`(232+96)*2`;我们的`worker_connections`配置的越大,那么初始化的时候就会预分配这么多的内存;

　　我们再来看下`ngx_event_s`里面有哪些成员?

这里我们比较关注的是`handler`这是一个回调方法;也就是说很多第三方模块会把这个handler设置为自己的实现;这里还有个`timer`,也就是说当我们对http请求做读超时和写超时时候等等设置的时候,其实是在操作读事件和写事件中的tmer;这个timer其实就是nginx实现超时定时器,也就是基于`rbtree`中的红黑树去实现的结构体;这里定时器其实也是可配的,这里我们看下它的配置;

　　在官方文档`ngx_http_core_module` 模块中

![在这里插入图片描述](https://img-blog.csdnimg.cn/119caf36a0b744df93ba5822e0c4ed90.png)
比如`client_header_timeout`:
![在这里插入图片描述](https://img-blog.csdnimg.cn/2b226efde77b4da796e6267ffe2517e3.png)
默认设置为`60s`,其实这个60s也就是在我们刚刚某个连接上,在准备读它的`header`时,我们在它的读事件上添加个60s的定时器;

　　当多个事件形成队列的时候我们可以使用`ngx_queue_t`;
　　
　　我们再来看下`ngx_connections_s` 每个连接有些什么样的成员?

　　`read` 和 `write`分别是它的读写事件;

　　`recv`和s`end`是它的一个抽象的操作系统的底层方法;怎么样发送和接受;

　　这里还有一个变量叫`sent` (off_t:表示无符号的整型)它表示这个连接上已经发送了多少字节,也就是在配置中经常使用到的`$bytes_sent`

　　还是在`ngx_http_core_module` 模块中 我们先找到它的内置变量;
![在这里插入图片描述](https://img-blog.csdnimg.cn/03eeab6c0b2549ee9dee65c45cb1cf67.png)
`$bytes_sent`:它表示向客户端发送了多少字节;

　通常在`access.log`记录`nginx`处理了哪些请求中我们可以记录这么一个变量,我们在`ngixn.conf`配置中添加了这么一个变量
![在这里插入图片描述](https://img-blog.csdnimg.cn/bee5b80889494348a0f918b08edd32e1.png)
总结:当我们需要配置高并发的`nginx`的时候,必须把`connections`的数目配置的足够大,而每个`connections`将对应两个`event`,都会消耗一定的内存,还有nginx的许多结构体中,它们的一些成员和我们的内置变量是可以对应起来的;
