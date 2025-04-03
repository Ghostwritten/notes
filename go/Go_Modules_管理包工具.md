#  Go Modules 管理包工具

![](https://i-blog.csdnimg.cn/blog_migrate/ab96175207f4f201a78816896d601622.png)





---
## 1. 背景
发展历史：

在 1.5 版本之前，所有的依赖包都是存放在 GOPATH 下，没有版本控制。这个类似 Google 使用单一仓库来管理代码的方式。这种方式的最大的弊端就是无法实现包的多版本控制，比如项目 A 和项目 B 依赖于不同版本的 package，如果 package 没有做到完全的向前兼容，往往会导致一些问题。

 - 1.5 版本推出了 vendor 机制。所谓 vendor 机制，就是每个项目的根目录下可以有一个 vendor 目录，里面存放了该项目的依赖的 package。go build 的时候会先去 vendor 目录查找依赖，如果没有找到会再去GOPATH 目录下查找。
 - 1.9 版本推出了实验性质的包管理工具 dep，这里把 dep 归结为 Golang 官方的包管理方式可能有一些不太准确。关于 dep 的争议颇多，比如为什么官方后来没有直接使用 dep 而是弄了一个新的 modules，具体细节这里不太方便展开。
 - 1.11 版本推出 modules 机制，简称 mod。modules 的原型其实是 vgo，关于 vgo，可以自行搜索。

除此之外，社区也一直在有几个活跃的包管理工具，使用广泛且具有代表性的主要有下面几个：

```bash
godep
glide
govendor
```

## 2. 简介
在 Java 的项目中，有 Maven 和 Gradle 这些很好用的依赖版本管理工具，简直不要太方便了，但是在 Golang 的项目中，之前的 Golang 官方并没有提供版本管理工具，我们以前用 go get 获取依赖其实是有潜在危险的，因为我们不确定最新版依赖是否会破坏掉我们项目对依赖包的使用方式，即当前项目可能会出现不兼容最新依赖包的问题。之后官方出了一个 vendor 机制，将项目依赖的包都放在该目录中，但这也并没有很好地管理依赖的版本。之后官方出了一个准官方版本管理工具 go dep，这也算是 go modules 的前身了吧。

### 2.1 设置 GO111MODULE
Go Modules 在 Go 1.11 及 Go 1.12 中有三个模式，根据环境变量 GO111MODULE 定义：

 - 默认模式（未设置该环境变量或 `GO111MODULE=auto`）：Go 命令行工具在同时满足以下两个条件时使用 Go Modules：当前目录不在 `GOPATH/src/` 下；在当前目录或上层目录中存在 go.mod 文件。
 - `GOPATH` 模式（`GO111MODULE=off`）：Go 命令行工具从不使用 Go Modules。相反，它查找 `vendor` 目录和`GOPATH` 以查找依赖项。
 - `Go Modules` 模式（GO111MODULE=on）：Go 命令行工具只使用 Go Modules，从不咨询GOPATH。GOPATH不再作为导入目录，但它仍然存储下载的依赖项（GOPATH/pkg/mod/）和已安装的命令（GOPATH/bin/），只移除了 GOPATH/src/。

Go 1.13 默认使用 Go Modules 模式，所以以上内容在 Go 1.13 发布并在生产环境中使用后都可以忽略。

### 2.2 Mod Cache 路径
默认在$GOPATH/pkg 下面：

```bash
$ echo $GOPATH/pkg/mod
/usr/local/gopath/pkg/mod
```
项目下载下来的文件形式

```bash
$ ls /usr/local/gopath/pkg/mod/cache/download/github.com/go-sql-driver/mysql/@v/
list  list.lock  v1.5.0.info  v1.5.0.lock  v1.5.0.mod  v1.5.0.zip  v1.5.0.ziphash
```

### 2.3 Go Mod 命令

```bash
$ go mod
Usage:

	go mod <command> [arguments]

The commands are:

download    download modules to local cache (下载依赖的module到本地cache))
edit        edit go.mod from tools or scripts (编辑go.mod文件)
graph       print module requirement graph (打印模块依赖图))
init        initialize new module in current directory (再当前文件夹下初始化一个新的module, 创建go.mod文件))
tidy        add missing and remove unused modules (增加丢失的module，去掉未用的module)
vendor      make vendored copy of dependencies (将依赖复制到vendor下)
verify      verify dependencies have expected content (校验依赖)
why         explain why packages or modules are needed (解释为什么需要依赖)
```

## 3. 创建 module
### 3.1 创建项目
在默认情况下，$GOPATH 默认情况下是不支持 go mudules 的，我们需要在项目目录下手动执行以下命令：

```bash
$ export GO111MODULE=on
```
为了配合 go modules 机制，我们 $GOPATH 以外的目录创建一个 testmod 的包：

```bash
$ mkdir testmod
$ cd testmod
$ go mod init github.com/ghostwritten/testmod
go: creating new go.mod: module github.com/ghostwritten/testmod
```
### 3.2 添加依赖项
```go
$ vim main.go
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    r.Run() // listen and serve on 0.0.0.0:8080
}

$ ls
go.mod  main.go
$ cat go.mod 
module github.com/ghostwritten/testmod

go 1.13
$ go build 
go: finding github.com/gin-gonic/gin v1.6.3
go: downloading github.com/gin-gonic/gin v1.6.3
go: extracting github.com/gin-gonic/gin v1.6.3
$ cat go.mod 
module github.com/ghostwritten/testmod

go 1.13

require github.com/gin-gonic/gin v1.6.3  #新依赖包
```
### 3.3 升级依赖项
#### 3.3.1 依赖列表 
```bash
$ go list -m all  #第一种方式
github.com/ghostwritten/testmod
github.com/davecgh/go-spew v1.1.1
github.com/gin-contrib/sse v0.1.0
github.com/gin-gonic/gin v1.6.3
github.com/go-playground/assert/v2 v2.0.1
......
$ go list -m -json   #第二种方式
{
	"Path": "github.com/ghostwritten/testmod",
	"Main": true,
	"Dir": "/root/go/testmod",
	"GoMod": "/root/go/testmod/go.mod",
	"GoVersion": "1.13"
}
```
#### 3.3.2 查看依赖的版本历史
```bash
go list -m -versions github.com/gin-gonic/gin
github.com/gin-gonic/gin v1.1.1 v1.1.2 v1.1.3 v1.1.4 v1.3.0 v1.4.0 v1.5.0 v1.6.0 v1.6.1 v1.6.2 v1.6.3
```
#### 3.3.3 更新到上个版本

```bash
$ go get github.com/gin-gonic/gin@v1.1.4 
$ go list -m all
// 看到了版本变化
github.com/gin-gonic/gin v1.1.4
```
#### 3.3.4 或者可以使用 go mod 来进行版本的切换

```bash
$ go mod edit -require="github.com/gin-gonic/gin@v1.1.4" // 修改 go.mod 文件
$ go tidy //下载更新依赖
```
#### 3.3.5 删除未使用的依赖项

```bash
 go mod tidy
```
## 4. 推送仓库

```bash
$ git init
$ git add *
$ git commit -am "First commit"
4 git git remote add origin http://gitlab.com/ghostwritten/testmod.git
$ git push -u origin master
```
### 4.1 发布版本
发布一下该项目的版本
```bash
$ git tag v1.0.0
$ git push --tags
```
如果更新创建一条 v1 分支，以便我们在其它分支写代码不会影响到 v1.0.0 版本
```bash
$ git checkout -b v1
$ git push -u origin v1
```

![](https://i-blog.csdnimg.cn/blog_migrate/0cbb2a96763963b909b4c0daf80556b9.png)
### 4.2 语义化版本
什么是语义化版本？语义化版本是一套由 Gravatars 创办者兼 GitHub 共同创办者 Tom Preston-Werner 所建立的约定。在这套约定下，语义化版本号及其更新方式包含了很多有用的信息。
语义化版本号格式为：X.Y.Z（主版本号.次版本号.修订号），使用方法如下：

 - 进行不向下兼容的修改时，递增主版本号。
 - API 保持向下兼容的新增及修改时，递增次版本号。
 - 修复问题但不影响 API 时，递增修订号。


## 5. vendor 目录
### 5.1 vendor 目录有两个目的：

 - 可以使用依赖项的确切版本用来构建。
 - 即使原始副本消失，也能保证这些依赖项是可用的。

而模块现在有了更好的机制来实现这两个目的：

 - 通过在 go.mod 文件中指定依赖项的确切版本。
 - 可用性则由缓存代理（$GOPROXY）实现。

### 5.2 生成 vendor

```bash
$ go mod vendor
```


参考：
- [https://www.cnblogs.com/sunsky303/p/10710637.html](https://www.cnblogs.com/sunsky303/p/10710637.html)
- [https://objcoding.com/2018/09/13/go-modules/](https://objcoding.com/2018/09/13/go-modules/)
