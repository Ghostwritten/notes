一个请求进入Nginx开始处理之前尼？

我们首先要增添端口,以使得Nginx可以和客户端建立起一个TCP连接,那么监听端口的这个指令尼,叫listen;它是放在我们的server配置块下的;通过监听的端口或者地址;我们其实已经可以决定有哪些匹配上我们TCP四元簇的地址连接对应的server块的相关指令去处理相应的请求;

下面我们来简单看下listen指令提供的简单功能;
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a3170da9f114abb92d4a20068d85c7f.png)
我们可以看到listen指令它的语法主要有三类:

(1):监听一个地址加上相应的端口:`listen address[:port]`
因为我们Nginx所在的机器上可能会有多块网卡,包括内网外网,那么这个时候尼,我可以通过选择地址,来确定我相应的server块指出以向这个地址建立连接的请求;
(2):也可以只指定端口:`listen port`
这个比较好理解:比如说我们监听80 8080 443 端口;

(3):也可以通过监听一个unix:path: `listen unix:path`，也就是一个**unix socket地址**;这个只能应用于本机通讯;我们通过listen address[:port] 要走内核的完整的网络栈的,而listen unix:path是不需要的所以它的性能应该会更好,而它的context只能出现在server这个配置块下,比如以下示例

```bash
(1):listen unix:/var/run/nginx.sock   我们监听一个unix socket的地址;
(2):listen 127.0.0.1:8000;    监听一个地址加端口,因为我们nginx可能有多个地址
(3):listen 127.0.0.1;      监听只指向一个地址,那么这个时候尼,  我们会默认使用80端口;       
(4):listen 8000;     只监听了一个端口,但是并没有指明地址;
(5):listen *:8000;      和第四种情况效果是一样的,什么情况下会不同尼?
```

