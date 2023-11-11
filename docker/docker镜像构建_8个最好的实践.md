


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
![在这里插入图片描述](https://img-blog.csdnimg.cn/74dcf334c3fb4cdaa9844ae03c9a007f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
## 2. 如何使用latest tag
![在这里插入图片描述](https://img-blog.csdnimg.cn/b709c66e477f4ab6899caacb108e8733.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
##  3. 基于不同的操作镜像特性
![在这里插入图片描述](https://img-blog.csdnimg.cn/8eafa0ffb4884b69ac5ad7cbafb5da52.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/84f2f84b3ac04f5787769e36de0f7e58.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2edd9a73b8647dcac8d81ff1146a8c7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6b88c65d2f774e7eb3574499b88e1289.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/27f6da126bb74b52bdbc99dc3141f45a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
##  4. 如何优化的镜像层的缓存

![在这里插入图片描述](https://img-blog.csdnimg.cn/d7f8fdf75a3d40cf9561c92c6a4267f6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
镜像层的缓存在哪查看
dockerhub
![在这里插入图片描述](https://img-blog.csdnimg.cn/e09a7b30770b448f93a997aa23aebf2d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
构建的时候
![在这里插入图片描述](https://img-blog.csdnimg.cn/5aa029e096e5486988b6094533331ccd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
docker history 命令
![在这里插入图片描述](https://img-blog.csdnimg.cn/1ccd2fdfae8b4be9b631d65e3e0f618a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/37c7d974d9e24ee9a5cc071ce7e2734d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4cf982b2fe1743aba7f459610a495bdb.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

如何利用好缓存关键在于我们要重用在没有修改的地方不需要重新构建。
![在这里插入图片描述](https://img-blog.csdnimg.cn/f97d4901d2964e0c83b44f87987d0d20.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/adab20219ebb46b6a92e8893c27bbb6d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
docker build构建的时候可以看到缓存的command层
![在这里插入图片描述](https://img-blog.csdnimg.cn/858321a14bc64686919d4f3fbe06f66a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

##  5 .dockerignore丢弃一些垃圾文件
当我们构建镜像的时候我们不需要项目的所有内容，来运行我们内部的应用程序，像目标或构建文件夹这样的自动生成的文件夹，我们不需要自述文件，因此，docker ignore文件是非常有必要的，
![在这里插入图片描述](https://img-blog.csdnimg.cn/3d4891926d0a4df184f1aacfd72e94c2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3fdb1977f24e4ebc90a37f6cdf4d88be.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
## 6. 多阶段构建（Multi-Stage Builds）
在运行容器的时候不需要测试依赖项、临时文件、开发环境工具、构建工具。但我们docker构建的时候却需要它们。
![在这里插入图片描述](https://img-blog.csdnimg.cn/bd1a1eb140634693b3fedc606d4cc5f2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/7fe2158e913f4716821cc1d545ee1efc.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/28cd652a71e84691b9a05521fdc9a872.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cac5289c295a4b628d19fae1e75c15f2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
我们分成build和stage两个阶段
build阶段是在以maven为base image并且取名为build的镜像构建打包
stage阶段是在以tomcat为base image的镜像拷贝build镜像里生成新的包，并运行应用程序，第一阶段的依赖工具被丢弃。


![在这里插入图片描述](https://img-blog.csdnimg.cn/9f8b9e641bed4f28a7cf280bf9695a68.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

##  7. 使用最小权限的用户
![在这里插入图片描述](https://img-blog.csdnimg.cn/b7175bcf32524a7cb1d365e9e43ca7c1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
有的镜像中已经捆绑了一个默认的专用用户，例如node:10-alpine中有node用户。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0fca548fa6234cb6997b486442622d93.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
## 8. 构建镜像时要扫描安全漏洞
docker login 登陆仓库

![在这里插入图片描述](https://img-blog.csdnimg.cn/0a873c3a475646d39945d93c1a4fd5e1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/dae84a539f3045c7bf7caf14e9ffd1c3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b7cb6727ef1d488d80a87a1d353d8127.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f2cce91309eb47f4806e08d3b2647634.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAZ2hvc3R3cml0dGVu,size_20,color_FFFFFF,t_70,g_se,x_16)

