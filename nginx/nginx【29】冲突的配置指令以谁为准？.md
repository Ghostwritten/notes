通过实例向大家演示HTTP模块,并结合以前讲解的知识;

　　nginx的模块非常多,包括官方模块和第三方模块;每一个模块又都有自己独特的指令;这些繁琐的指令是非常难以记忆的;

　　接下来带领大家以请求处理流程的方式进行把所有的常用的HTTP模块的指令梳理在一起;把HTTP模块已在Nginx设计架构中定义的11个阶段的方式依次的去讲解每一个模块的使用方法;

　　那么在nginx的11个阶段讲解完以后尼,我们还会讲到nginx的HTTP过滤模块;它会通过加工我们向客户端返回的响应来向客户端返回不一样的内容;

　　最后我们还需要介绍到nginx一个核心的概念叫做变量;因为nginx通过变量来实现非常复杂功能;


在讲解配置指令之前,先来看下HTTP配置指令的嵌套结构;

　　因为每一个二进制提供的指令,很多时候它可以出现在它的context也就是上下文,既可以在location中,又可以在server中,或者HTTP 中,甚至在if这样的配置快中;

　　**那么当一个指令出现在多个配置块中的时候,它们的值可能是冲突的,那么到底以谁为准尼;或者说在有些配置块下,我没有这条指令,但是在使用的过程中却发现它生效了,那么这是一种什么样的机制尼,还有很多第三方模块它们可能不是非常的遵循官方模块既定的一些规则,这个时候我们应该怎么样去判断配置指令到底怎么生效或者发生冲突的时候以谁为准的;**

　　我们先来看下一个典型的配置块的嵌套什么样的?

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad004aaa9552b7da8d12b0165fdfefda.png)
在`http{}`之前有一个`main`;如果看事件模块,配置进程一些它们的上下文都是在main中的;重点来看`http`下面的;可能会有`upstream`或者`split_clients`;这是某一个模块,包括`map`或者`geo`都是一些为了http模块它自己可以定义一些自己的配置块,http,server,location这三个是非常核心的,这是http的框架来定义的;因为我们要处理一个请求的时候需要先按照请求中的指示的**域名**,比如host找到相应的**server块**;然后再根据我们的**url**找到某一个**location**,根据这个location下这个具体的指令来处理请求,所以在这样一个典型的配置块的嵌套中尼,我们会发现很多冲突和奇怪的指令;

　　那么下面来看一看;首先什么叫指令的**context**;
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71d588cabac1ecf4709bd6d3eb08cdd7.png)
　我们在做下声明:比如`log`模块 我们记录`access.log` 模块;它有两个指令:
　![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/693e366ec245f0b93003904df2d3b532.png)
　　　　

 - (1):`log_format`:我们看到log_format的时候可以看到,它的`context`,也就是上下文能够出现的配置块是http;所以如果把log_format放在我们刚才所说的server,或者location中尼,我们nginx检查配置文件会报语法失败,压根就不会让我们启动;

 

　　　

 - (2):`access_log`:那么对于access.log这条指令尼,可以看到它可以出现在http,server,location,if in location,limit_except等等很多地方;那这就是它的上下　　　　　　  文;

当指令在多个块下存在的时候,它是可以合并的;并不是所有的指令都可以合并;

先讲下指令合并的总体规则:我们所有的指令会分为两类指令:

　　　　

 - (1):值指令:我们在解析配置的时候,它主要是对这个指令下我存储当时配置的值,那这种情况下是可以合并的; 比如说,向我们第一部分介绍的root,access.log,gzip它们存储的都是一些值,这些指令不同块下是可以合并的;


 - (2):动作类指令:不可以合并,因为它在指定我们的行为,比如说rewrite,proxy_pass;当这条指令出现的时候,请求处理到这条指令位置的时候;

　　　　　  我们必须立刻处理一种行为,那么生效阶段尼包括`server_rewrite`阶段,rewrite阶段,content阶段;这是我们以后讲解11个阶段的时候会讲到的;

　　　　那么很多同学会认为这个很难理解,我怎么会判断出一个指令到底是可以合并还是不可以合并尼?其实非常简单,就是我们刚刚谈到的**生效阶段**;

　　　　因为`server_rewrite`阶段和`rewrite`阶段只有我们的`http_result`模块才有可能提供;而`content`阶段一般是反向代理;或者其它我们在这一部分的课程中会介绍到五个content模块,像这些content模块它们提供了一些方法;只能是动作类指令的,当然我们通过源码也可以判断出来,但是相对来说,动作类指令并不是很多;

我们现在先来介绍下存储值的指令:它们有一些什么不同的地方?
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26301c96a01d812b7484d8e09a9329ed.png)
先我们看下这个`listen:8080`;listen这个指令 它只能出现在`server`这个`context`中,所以非常简单;

　第二个我们看下`alias`这个指令:虽然它可以出现在很多地方,比如说server中或者http中;但是在我们这个 配置中尼,我们只在了`location`中出现了,这个也非常容易理解;因为当一个请求匹配到  /dlib的时候,那么alias就产生作用了;

那么我们再来看下稍微需要我们值合并的场景;

比如说这里我们定义了一个:`root /home/geek/nginx/html`;这里我们定义了如果查到一个静态文件,从这个目录下查找,
但是我么在`location /{}` 去匹配所有的剩余的url的时候却没有定义`root`,这个时候尼,其实我们可以使用这个root的;

这里就是所谓的子配置不存的时候尼,直接使用 父配置块;root在location中并不存在,但是在server中是存在的,我们直接引用这里;这是nginx中一个通用的配置规则;那么所有的指令只要它写明了它能够存在http,server,location下的时候尼,当子的不存的时候尼,直接引用父的配置的值;我们再看第四种:比如在
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/778b8992b1d651663379e8e0019e454c.png)
这个location中尼?重新定义一个root和access.log
比如 

```bash
root /home/geek/nginx/test;
access.log logs/access.test.log main;
```

这就引出了我们的第二个规则:**当子配置和父配置同时存在的时候,直接覆盖父配置块**;也就是说当我们的请求匹配到 /test的时候,那么server下面的 root　　　　和access.log的值自动失效,我们开始使用子配置中的了;

　所有Nginx的官方HTTP模块或者OpenResty中的HTTP模块,它们都遵循上述的两个配置子指令的两个规则,但是有些第三方模块尼很可没有遵循这样的　　　　一套规则,这个时候尼,如果它相应的说明文档也不是很清楚地情况下,就需要我们通过源码来判断;

当它们的子指令出现冲突的时候尼,究竟以哪一个为准;那么怎么样通过它的源码来看尼?其实非常简单,主要抓住以下四个点就可以了;

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d42fc22b9ce34a266a0a33672396beaf.png)　

 - (1):第一个点我们需要判断这个指令是在哪一个块下生效的;生效:后面我们会讲到的11个阶段中;比如说有些指令可能会在server块下生效的;但大部分指令都是在location下生效的;
 - (2):指令允许出现在哪些块下? 比如access.log这条指令,可以看到它可以出现在http,server,location,if in location,limit_except等等很多地方

 - (3):在server块内生效,`从http`向`server`合并指令:当这个指令在server下生效的时候,它会定义一个方法,这个方法叫`merge_srv_conf`,也就是说这个指令出现在http块下,也出现在server块下,从http向server合并的时候;它会提供一个函数叫做`merge_srv_conf`;

```bash
char *(*merge_srv_conf)(ngx_conf_t *cf,void *prev,void *conf);
```

 - (4):配置缓存在内存 如果是在location下生效的尼,提供函数`merge_log_conf`;

```bash
char *(*merge_log_conf)(ngx_conf_t *cf,void *prev,void *conf);
```

