main.go文件分析

```go
package main

import (
	_ "quickstart/routers"
	"github.com/astaxie/beego"
)

func main() {
	beego.Run()
}
```
们看到main函数是入口函数，但是我们知道Go的执行过程是如下图所示的方式：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020080300224780.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
这里我们就看到了我们引入了一个包_ "quickstart/routers",这个包只引入执行了里面的init函数，那么让我们看看这个里面做了什么事情：

```bash
package routers

import (
	"quickstart/controllers"
	"github.com/astaxie/beego"
)

func init() {
    beego.Router("/", &controllers.MainController{})
}
```

路由包里面我们看到执行了路由注册beego.Router, 这个函数的功能是映射URL到controller，第一个参数是URL(用户请求的地址)，这里我们注册的是 /，也就是我们访问的不带任何参数的URL，第二个参数是对应的 Controller，也就是我们即将把请求分发到那个控制器来执行相应的逻辑，我们可以执行类似的方式注册如下路由：

```bash
beego.Router("/user", &controllers.UserController{})	
```
这样用户就可以通过访问 /user 去执行 UserController 的逻辑。这就是我们所谓的路由，更多更复杂的路由规则请查询 beego 的路由设置
再回来看看main函数里面的 beego.Run， beego.Run 执行之后，我们看到的效果好像只是监听服务端口这个过程，但是它内部做了很多事情：

 - 解析配置文件
   
   beego 会自动在 conf 目录下面去解析相应的配置文件 `app.conf`，这样就可以通过配置文件配置一些例如开启的端口，是否开启session，应用名称等各种信息。
 - 执行用户的hookfunc
   
   beego会执行用户注册的`hookfunc`，默认的已经存在了注册mime，用户可以通过函数`AddAPPStartHook`注册自己的启动函数。
 - 是否开启 session
   
   会根据上面配置文件的分析之后判断是否开启 `session`，如果开启的话就初始化全局的 session。
 - 是否编译模板
   
   beego 会在启动的时候根据配置把 views 目录下的所有模板进行预编译，然后存在 map 里面，这样可以有效的提高模板运行的效率，无需进行多次编译。
 - 是否开启文档功能
   
   根据`EnableDocs`配置判断是否开启内置的文档路由功能
 - 是否启动管理模块
   
   beego 目前做了一个很帅的模块，应用内监控模块，会在 8088 端口做一个内部监听，我们可以通过这个端口查询到`QPS、CPU、内存、GC、goroutine、thread` 等各种信息。
 - 监听服务端口
   
   这是最后一步也就是我们看到的访问 8080 看到的网页端口，内部其实调用了 `ListenAndServe`，充分利用了 `goroutine`的优势


一旦 run 起来之后，我们的服务就监听在两个端口了，一个服务端口 8080 作为对外服务，另一个 8088 端口实行对内监。
参考资料：
[https://www.kancloud.cn/hello123/beego/126095](https://www.kancloud.cn/hello123/beego/126095)
