


----
视频欣赏：[https://mp.weixin.qq.com/s/WeyXesSkt_-K3RH2kfoacw](https://mp.weixin.qq.com/s/WeyXesSkt_-K3RH2kfoacw)
原创：[https://www.youtube.com/watch?v=8vXoMqWgbQQ&t=855s](https://www.youtube.com/watch?v=8vXoMqWgbQQ&t=855s)
相关阅读：

 - [**docker圣经**](https://ghostwritten.blog.csdn.net/article/details/119462437)
 - [**云原生圣经**](https://ghostwritten.blog.csdn.net/article/details/108562082)
 - [Top 20 Dockerfile best practices](https://sysdig.com/blog/dockerfile-best-practices/)

---
## 1. 如何选择base image
假如我构建一个node.js
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63e53d334c47418f3fc7ff7ac19ab72f.png)
## 2. 如何使用latest tag
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d015dd9635120b3e5c94ad7f4f8dd0e.png)
##  3. 基于不同的操作镜像特性
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1ad998762780dcdce90565ca5fda1441.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3dde30298d32231e4997c13993520e4d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec5cee559f095b1d89cebe432732cea0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/105203d8f28e5ae8a7bf492f86df2dc7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/19be11769ac4f199404b4dd76d5ba4e6.png)
##  4. 如何优化的镜像层的缓存

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3d89bd273f3b889f69c25cdf9abddc5b.png)
镜像层的缓存在哪查看
dockerhub
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ceefaedfe93dc8c009fbbb67017921b6.png)
构建的时候
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ca5dd516bbe4e7c4d6d95579e075fbf8.png)
docker history 命令
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d2b49611f00e2143dec0f8e4101b5537.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/337e6ba50db8beae30cbd8978ee2c47e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f12c270452a4024b7e569cb787db1952.png)

如何利用好缓存关键在于我们要重用在没有修改的地方不需要重新构建。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/45488e68c6cd0f8640dfd5d12de6e9e9.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a00515b82da1495733cbd50daae66ad4.png)
docker build构建的时候可以看到缓存的command层
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5ea5ab9839a2bc5ce66dd397accb1fc7.png)

##  5 .dockerignore丢弃一些垃圾文件
当我们构建镜像的时候我们不需要项目的所有内容，来运行我们内部的应用程序，像目标或构建文件夹这样的自动生成的文件夹，我们不需要自述文件，因此，docker ignore文件是非常有必要的，
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f935e7298535202b2516d64c90a75ff6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e97f1e64091db7ec420fd69fe5f8db6e.png)
## 6. 多阶段构建（Multi-Stage Builds）
在运行容器的时候不需要测试依赖项、临时文件、开发环境工具、构建工具。但我们docker构建的时候却需要它们。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dd9d390be8e6e3088d184c7728143525.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec25facf376e7ffdeca8314f8e812b66.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c026a24dfcc7f258de43150cfcea339.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5aaba7984801311041e473e40b64631e.png)
我们分成build和stage两个阶段
build阶段是在以maven为base image并且取名为build的镜像构建打包
stage阶段是在以tomcat为base image的镜像拷贝build镜像里生成新的包，并运行应用程序，第一阶段的依赖工具被丢弃。


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0062fa4f21c0b78f6c8da87f1f1cf4f0.png)

##  7. 使用最小权限的用户
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/506b9f37aab1aed598544151cd8b81d5.png)
有的镜像中已经捆绑了一个默认的专用用户，例如node:10-alpine中有node用户。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/002ad137ef385e55796a71c046773340.png)
## 8. 构建镜像时要扫描安全漏洞
docker login 登陆仓库

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9691cc79d0afd65328d3064e24605c50.png)
docker scan参数：

```bash
Options: 
      --accept-license    接受使用第三方扫描提供商 
      --dependency-tree   显示带有扫描结果的依赖树 
      --exclude-base      从漏洞扫描中排除基础镜像 (requires --file) 
  -f, --file string       与image关联的Dockerfile，提供更详细的结果 
      --group-issues      聚合重复的漏洞并将其分组为1个漏洞 (requires --json) 
      --json              以json格式输出结果 
      --login             使用可选令牌(带有--token)向扫描提供程序进行身份验证，如果为空则使用web base令牌 
      --reject-license    拒绝使用第三方扫描提供商 
      --severity string   只报告提供级别或更高的漏洞(low|medium|high) 
      --token string      登录到第三方扫描提供程序的认证令牌 
      --version           显示扫描插件版本 
```

```bash
#指定Dockerfile
$ docker scan -f Dockerfile myapp:1.0

#不扫描该镜像的基础镜像
$ docker scan -f Dockerfile --exclude-base  myapp:1.0

#以json显示
docker scan --json  myapp:1.0 | jq

#聚合分组显示扫描信息
$ docker scan --json --group-issues myapp:1.0

#显示指定级别的漏洞，只有高于此级别的漏洞才会显示出来
$ docker scan --severity=medium  myapp:1.0
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3b7babd83f4d52f6b72254b1f143579.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88440cd1cd1cdc5d1fdc79d062d01068.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2e156b93f2d3c67cb7e95fd38c03ac86.png)

