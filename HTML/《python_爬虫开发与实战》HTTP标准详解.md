## 简介
HTTP协议（HyperText Transfer Protocol，超文本传输协议）是用于从WWW 服务器传输超文本到本地浏览器的传送协议。它可以使浏览器更加高效，减少网络 传输。它不仅保证计算机正确快速地传输超文本文档，还确定传输文档中的哪一部 分，以及哪部分内容首先显示（如文本先于图形）等。之后的Python爬虫开发，主 要就是和HTTP协议打交道。
## HTTP请求过程
HTTP协议采取的是请求响应模型，HTTP协议永远都是客户端发起请求，服务器 回送响应。模型如图2-8所示。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6dc9552977ffb383496ff873f98b14b5.png#pic_center)
HTTP协议是一个无状态的协议，同一个客户端的这次请求和上次请求没有对应 关系。一次HTTP操作称为一个事务，其执行过程可分为四步：

 - ·首先客户端与服务器需要建立连接，例如单击某个超链接，HTTP的工作就开始 了。
 - ·建立连接后，客户端发送一个请求给服务器，请求方式的格式为：统一资源标识符（URL）、协议版本号，后边是MIME信息，包括请求修饰符、客户机信息和可 能的内容。
 - ·服务器接到请求后，给予相应的响应信息，其格式为一个状态行，包括信息的协议版本号、一个成功或错误的代码，后边是MIME信息，包括服务器信息、实体信 息和可能的内容。
 - ·客户端接收服务器所返回的信息，通过浏览器将信息显示在用户的显示屏上， 然后客户端与服务器断开连接。

如果以上过程中的某一步出现错误，那么产生错误的信息将返回到客户端，在 显示屏输出，这些过程是由HTTP协议自己完成的。

## HTTP状态码含义
当浏览者访问一个网页时，浏览者的浏览器会向网页所在服务器发出请求。在 浏览器接收并显示网页前，此网页所在的服务器会返回一个包含HTTP状态码的信息 头（server header）用以响应浏览器的请求。HTTP状态码主要是是为了标识此次 HTTP请求的运行状态。下面是常见的HTTP状态码：

```c
·200——请求成功。
·301——资源（网页等）被永久转移到其他URL。
·404——请求的资源（网页等）不存在。
·500——内部服务器错误。
```

HTTP状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型。 HTTP状态码共分为5种类型，如表2-9所示。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3cd21b30f672cb24fbf38d110659a7af.png#pic_center)
## 　HTTP头部信息
HTTP头部信息由众多的头域组成，每个头域由一个域名、冒号（：）和域值三 部分组成。域名是大小写无关的，域值前可以添加任何数量的空格符，头域可以被 扩展为多行，在每行开始处，使用至少一个空格或制表符。
通过浏览器访问博客园首页时，使用F12打开开发者工具，里面可以监控整个 HTTP访问的过程。下面就以访问博客园的HTTP请求进行分析，首先是浏览器发出请 求，请求头的数据如下：

```c
GET / HTTP/1.1     
Host: www.cnblogs.com
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64; rv:49.0) Gecko/20100101 Firefox/49.0     
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8     
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3     
Accept-Encoding: gzip, deflate     
Connection: keep-alive     
If-Modified-Since: Sun, 30 Oct 2016 10:13:18 GMT
```
在请求头中包含以下内容：

 - `·GET`代表的是请求方式，HTTP/1.1表示使用HTTP 1.1协议标准。
 - ·`Host`头域，用于指定请求资源的Intenet主机和端口号，必须表示请求URL的原始服务器或网关的位置。HTTP/1.1请求必须包含主机头域，否则系统会以400状 态码返回。
 - ·`User-Agent`头域，里面包含发出请求的用户信息，其中有使用的浏览器型号、 版本和操作系统的信息。这个头域经常用来作为反爬虫的措施。
 - ·`Accept`请求报头域，用于指定客户端接受哪些类型的信息。例如：Accept image/gif，表明客户端希望接受GIF图象格式的资源；Accept：text/html，表 明客户端希望接受html文本。
 - ·Accept-Language请求报头域，类似于Accept，但是它用于指定一种自然语 言。例如：Accept-Language：zh-cn.如果请求消息中没有设置这个报头域，服务 器假定客户端对各种语言都可以接受。
 - ·`Accept-Encoding`请求报头域，类似于Accept，但是它用于指定可接受的内容编码。例如：Accept-Encoding：gzip.deflate。如果请求消息中没有设置这个域服务器假定客户端对各种内容编码都可以接受。
 - `·Connection`报头域允许发送用于指定连接的选项。例如指定连接的状态是连续，或者指定“close”选项，通知服务器，在响应完成后，关闭连接。
 - ·`If-Modified-Since`头域用于在发送HTTP请求时，把浏览器端缓存页面的最后修改时间一起发到服务器去，服务器会把这个时间与服务器上实际文件的最后修改时间进行比较。如果时间一致，那么返回HTTP状态码304（不返回文件内容），客户端收到之后，就直接把本地缓存文件显示到浏览器中。如果时间不一致，就返回HTTP状态码200和新的文件内容，客户端收到之后，会丢弃旧文件，把新文件缓存 起来，并显示到浏览器中。


请求发送成功后，服务器进行响应，接下来看一下响应的头信息，数据如下：

```bash
HTTP/1.1 200 OK     
Date: Sun, 30 Oct 2016 10:13:50 GMT     
Content-Type: text/html; charset=utf-8     
Transfer-Encoding: chunked     
Connection: keep-alive     
Vary: Accept-Encoding     
Cache-Control: public, max-age=3     
Expires: Sun, 30 Oct 2016 10:13:54 GMT    
Last-Modified: Sun, 30 Oct 2016 10:13:24 GMT     
Content-Encoding: gzip
```
响应头中包含以下内容：

 - ·`HTTP/1.1`表示使用HTTP 1.1协议标准，200OK说明请求成功。
 - ·`Date`表示消息产生的日期和时间。
 - ·`Content-Type`实体报头域用于指明发送给接收者的实体正文的媒体类型。text/html；charset=utf-8代表HTML文本文档，UTF-8编码。
 - ·`Transfer-Encoding`：chunked表示输出的内容长度不能确定。
 - ·`Connection`报头域允许发送用于指定连接的选项。例如指定连接的状态是连续，或者指定“close”选项，通知服务器，在响应完成后，关闭连接。
 - ·`Vary`头域指定了一些请求头域，这些请求头域用来决定当缓存中存在一个响应，并且该缓存没有过期失效时，是否被允许利用此响应去回复后续请求而不需要 重复验证。
 - ·`Cache-Control`用于指定缓存指令，缓存指令是单向的，且是独立的。请求时的缓存指令包括：no-cache（用于指示请求或响应消息不能缓存）、no-store、max-age、max-stale、min-fresh、only-if-cached；响应时的缓存指令包括：public、private、no-cache、no-store、no-transform、must-revalidate、proxy-revalidate、max-age、s-maxage。
 - ·`Expires`实体报头域给出响应过期的日期和时间。为了让代理服务器或浏览器在一段时间以后更新缓存中（再次访问曾访问过的页面时，直接从缓存中加载，缩短响应时间和降低服务器负载）的页面，我们可以使用Expires实体报头域指定页 面过期的时间。
 - ·`Last-Modified`实体报头域用于指示资源的最后修改日期和时间。
 - ·`Content-Encoding`实体报头域被用作媒体类型的修饰符，它的值指示了已经被应用到实体正文的附加内容的编码，因而要获得Content-Type报头域中所引用的 媒体类型，必须采用相应的解码机制。
总结：
HTTP消息报头主要包括普通报头、请求报头、响应报头、实体报头。具体如
下：
 - 1）在普通报头中，有少数报头域用于所有的请求和响应消息，但并不用于被传 输的实体，只用于传输的消息。
 - 2）请求报头允许客户端向服务器端传递请求的附加信息以及客户端自身的信 息。
 - 3）响应报头允许服务器传递不能放在状态行中的附加响应信息，以及关于服务
   器的信息和对Request-URI所标识的资源进行下一步访问的信息。
 - 4）请求和响应消息都可以传送一个实体。一个实体由实体报头域和实体正文组
   成，但并不是说实体报头域和实体正文要在一起发送，可以只发送实体报头域。实 体报头定义了关于实体正文和请求所标识的资源的元信息。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b34af0abb3db3f3e47beda57f60bea43.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6ecb324c71655307ac28f4141b2eabce.png#pic_center)
参考资料：
《python 爬虫开发与实战》
