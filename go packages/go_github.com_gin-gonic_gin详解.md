## gin介绍
`github.com/gin-gonic/gin`是一个轻量级的 WEB 框架，支持 RestFull 风格 API，支持 `GET，POST，PUT，PATCH，DELETE，OPTIONS` 等 http 方法，支持文件上传，分组路由，Multipart/Urlencoded FORM，以及支持 JsonP，参数处理等等功能，这些都和 WEB 紧密相关，通过提供这些功能，使开发人员更方便地处理 WEB 业务。
## 安装

```bash
go get -u github.com/gin-gonic/gin
```
如果卡或者慢，请参考：[配置代理加速](https://blog.csdn.net/xixihahalelehehe/article/details/105844145)

## 简单示例
ping.go  

```bash
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080 (for windows "localhost:8080")
}
```

```bash
$ go run ping.go
```
浏览器访问：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/47e10913b849e07201553a5a2e51949f.png)

