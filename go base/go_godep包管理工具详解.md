## 背景
Golang 1.5 release版本的发布之前，只能通过设置多个GOPATH的方式来解决这个问题，例如：我两个工程都依赖了Beego，但A工程依赖的是Beego 1.1，B工程依赖的是Beego 1.7，我必须设置两个GOPATH来区分，并且在切换工程的时候GOPATH也得切换，无比痛苦。在Golang 1.5 release 开始支持除了GOROOT和GOPATH之外的依赖管理：vender，官方 wiki 推荐了多种支持这种特性的包管理工具，如：Godep、gv、gvt、glide、govendor和官方的dep等。


## godep简介
godep是解决包依赖的管理工具，原理是扫描记录版本控制的信息，并在go命令前加壳来做到依赖管理。godep早期版本并不依赖vendor，所以对go的版本要求很松，go 1.5之前的版本也可以用，只是行为上有所不同。在vendor推出以后，godep也改为使用vendor了。godep 建议在 golang 1.6 以后使用，且godep 依赖 `vendor` 。
godep的使用者众多，如docker，kubernetes， coreos等go项目很多都是使用godep来管理其依赖，当然原因可能是早期也没的工具可选

## 安装

```bash
go get -u -v github.com/tools/godep
```
## 编译和运行
项目用godep管理后，要编译和运行项目的时候再用go run和go build显然就不行了，因为go命令是直接到GOPATH目录下去找第三方库。 而使用godep下载的依赖库放到Godeps/workspace目录下的；

```bash
godep go run main.go 
godep go build 
godep go install 
godep go test
```

godep中的go命令，就是将原先的go命令加了一层壳，执行godep go的时候，会将当前项目的workspace目录加入GOPATH变量中；

## godep save 检出依赖
godep save将项目中使用到的第三方库复制到项目的Godeps目录下。

godep save 会自动扫描当前目录所属包中import的所有外部依赖库（非系统库），并查看其是否属于某个代码管理工具（比如git，hg）。若是，则把此库获取路径和当前对应的revision（commit id）记录到当前目录Godeps下的Godeps.json，同时，把不含代码管理信息（如.git目录）的代码拷贝到Godeps/_workspace/src下，用于后继godep go build等命令执行时固定查找依赖包的路径。

因此，godep save能否成功执行需要有两个要素： 当前或者需扫描的包均能够编译成功：因此所有依赖包事先都应该已经或go get或手工操作保存到当前GOPATH路径下 依赖包必须使用了某个代码管理工具（如git，hg）：这是因为godep需要记录revision

## godep restore
如果下载的项目中只有Godeps.json文件，而没有包含第三库则可以使用godep restore这个命令将所有的依赖库下来下来到GOPATH的src中。

```bash
godep restore
```

godep restore执行时，godep会按照Godeps/Godeps.json内列表，依次执行go get -d -v 来下载对应依赖包到GOPATH路径下，因此，如果某个原先的依赖包保存路径（GOPATH下的相对路径）与下载url路径不一致，比如kuberbetes在github上路径是github.com/kubernetes，而代码内import则是k8s.io，则会导致无法下载成功，也就是说godep restore不成功。这种只能手动，比如手动创建$GOPATH/k8s.io目录，然后git clone。

## golang自带包管理工具
自带工具：go get 
go get可以将依赖的第三方库下载本GOPATH目录，在代码中直接import相应的代码库就可以了。 与godep相比，如果项目引用的第三方库没有列入到项目里面，安装项目时，针对第三方库需要使用go get一个个下载，比较麻烦；

注：使用godep restore可能导致部分库无法下载下来；编译会报错： cmd/decode.go:16:2: cannot find package "github.com/CodisLabs/redis-port/pkg/libs/atomic2" in any of:

此时针对报错的特定库再go get一般都能下载：

```bash
 go get github.com/CodisLabs/redis-port/pkg/libs/atomic2
```

godep支持的命令

```bash
save     list and copy dependencies into Godeps
go       run the go tool with saved dependencies
get      download and install packages with specified dependencies
path     print GOPATH for dependency code
restore  check out listed dependency versions in GOPATH
update   update selected packages or the go version
diff     shows the diff between current and previously saved set of dependencies
version  show version info
```

