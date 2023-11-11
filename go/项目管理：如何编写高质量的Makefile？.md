
![](https://img-blog.csdnimg.cn/ff61b5901b10486eb385b2ff7d802a62.png)




##   背景
理解低质量的 Makefile 文件，例如：

```bash
build: clean vet
  @mkdir -p ./Role
  @export GOOS=linux && go build -v .

vet:
  go vet ./...

fmt:
  go fmt ./...

clean:
  rm -rf dashboard
```
上面这个 Makefile 存在不少问题。例如：功能简单，只能完成最基本的编译、格式化等操作，像构建镜像、自动生成代码等一些高阶的功能都没有；扩展性差，没法编译出可在 Mac 下运行的二进制文件；没有 Help 功能，使用难度高；单 Makefile 文件，结构单一，不适合添加一些复杂的管理功能。

所以，我们不光要编写 Makefile，还要编写高质量的 Makefile。那么如何编写一个高质量的 Makefile 呢？我觉得，可以通过以下 4 个方法来实现：
- 打好基础，也就是熟练掌握 Makefile 的语法。
- 做好准备工作，也就是提前规划 Makefile 要实现的功能。
- 进行规划，设计一个合理的 Makefile 结构。
- 掌握方法，用好 Makefile 的编写技巧。

##  熟练掌握 Makefile 语法
工欲善其事，必先利其器。编写高质量 Makefile 的第一步，便是熟练掌握 Makefile 的核心语法。因为 Makefile 的语法比较多，我把一些建议你重点掌握的语法放在了近期会更新的特别放送中，包括 Makefile 规则语法、伪目标、变量赋值、条件语句和 Makefile 常用函数等等。如果你想更深入、全面地学习 Makefile 的语法，我推荐你学习陈皓老师编写的[《跟我一起写 Makefile》 (PDF 重制版)](https://github.com/seisman/how-to-write-makefile)。

##  规划 Makefile 要实现的功能
接着，我们需要规划 Makefile 要实现的功能。提前规划好功能，有利于你设计 Makefile 的整体结构和实现方法。不同项目拥有不同的 Makefile 功能，这些功能中一小部分是通过目标文件来实现的，但更多的功能是通过伪目标来实现的。对于 Go 项目来说，虽然不同项目集成的功能不一样，但绝大部分项目都需要实现一些通用的功能。接下来，我们就来看看，在一个大型 Go 项目中 Makefile 通常可以实现的功能。

下面是 IAM 项目的 Makefile 所集成的功能，希望会对你日后设计 Makefile 有一些帮助。

```bash

$ make help

Usage: make <TARGETS> <OPTIONS> ...

Targets:
  # 代码生成类命令
  gen                Generate all necessary files, such as error code files.

  # 格式化类命令
  format             Gofmt (reformat) package sources (exclude vendor dir if existed).

  # 静态代码检查
  lint               Check syntax and styling of go sources.

  # 测试类命令
  test               Run unit test.
  cover              Run unit test and get test coverage.

  # 构建类命令
  build              Build source code for host platform.
  build.multiarch    Build source code for multiple platforms. See option PLATFORMS.

  # Docker镜像打包类命令
  image              Build docker images for host arch.
  image.multiarch    Build docker images for multiple platforms. See option PLATFORMS.
  push               Build docker images for host arch and push images to registry.
  push.multiarch     Build docker images for multiple platforms and push images to registry.

  # 部署类命令
  deploy             Deploy updated components to development env.

  # 清理类命令
  clean              Remove all files that are created by building.

  # 其他命令，不同项目会有区别
  release            Release iam
  verify-copyright   Verify the boilerplate headers for all files.
  ca                 Generate CA files for all iam components.
  install            Install iam system with all its components.
  swagger            Generate swagger document.
  tools              install dependent tools.

  # 帮助命令
  help               Show this help info.

# 选项
Options:
  DEBUG        Whether to generate debug symbols. Default is 0.
  BINS         The binaries to build. Default is all of cmd.
               This option is available when using: make build/build.multiarch
               Example: make build BINS="iam-apiserver iam-authz-server"
  ...
```
更详细的命令，你可以在 IAM 项目仓库根目录下执行`make help`查看。

通常而言，Go 项目的 Makefile 应该实现以下功能：格式化代码、静态代码检查、单元测试、代码构建、文件清理、帮助等等。如果通过 docker 部署，还需要有 docker 镜像打包功能。因为 Go 是跨平台的语言，所以构建和 docker 打包命令，还要能够支持不同的 CPU 架构和平台。为了能够更好地控制 Makefile 命令的行为，还需要支持 Options。为了方便查看 Makefile 集成了哪些功能，我们需要支持 help 命令。help 命令最好通过解析 Makefile 文件来输出集成的功能，例如：

```bash
## help: Show this help info.
.PHONY: help
help: Makefile
  @echo -e "\nUsage: make <TARGETS> <OPTIONS> ...\n\nTargets:"
  @sed -n 's/^##//p' $< | column -t -s ':' | sed -e 's/^/ /'
  @echo "$$USAGE_OPTIONS"
```
上面的 help 命令，通过解析 Makefile 文件中的##注释，获取支持的命令。通过这种方式，我们以后新加命令时，就不用再对 help 命令进行修改了。你可以参考上面的 Makefile 管理功能，结合自己项目的需求，整理出一个 Makefile 要实现的功能列表，并初步确定实现思路和方法。做完这些，你的编写前准备工作就基本完成了。

##  设计合理的 Makefile 结构
设计完 Makefile 需要实现的功能，接下来我们就进入 Makefile 编写阶段。编写阶段的第一步，就是设计一个合理的 Makefile 结构。对于大型项目来说，需要管理的内容很多，所有管理功能都集成在一个 Makefile 中，可能会导致 Makefile 很大，难以阅读和维护，**所以建议采用分层的设计方法，根目录下的 Makefile 聚合所有的 Makefile 命令，具体实现则按功能分类，放在另外的 Makefile 中**。

我们经常会在 Makefile 命令中集成 shell 脚本，但如果 shell 脚本过于复杂，也会导致 Makefile 内容过多，难以阅读和维护。并且在 Makefile 中集成复杂的 shell 脚本，编写体验也很差。**对于这种情况，可以将复杂的 shell 命令封装在 shell 脚本中，供 Makefile 直接调用，而一些简单的命令则可以直接集成在 Makefile 中**。

所以，最终我推荐的 Makefile 结构如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9efd5b63346b46d9901ec885db44b229.png)
在上面的 Makefile 组织方式中，根目录下的 Makefile 聚合了项目所有的管理功能，这些管理功能通过 Makefile 伪目标的方式实现。同时，还将这些伪目标进行分类，把相同类别的伪目标放在同一个 Makefile 中，这样可以使得 Makefile 更容易维护。对于复杂的命令，则编写成独立的 shell 脚本，并在 Makefile 命令中调用这些 shell 脚本。

举个例子，下面是 IAM 项目的 Makefile 组织结构：
```bash
├── Makefile
├── scripts
│   ├── gendoc.sh
│   ├── make-rules
│   │   ├── gen.mk
│   │   ├── golang.mk
│   │   ├── image.mk
│   │   └── ...
    └── ...
```
我们将相同类别的操作统一放在 `scripts/make-rules` 目录下的 `Makefile` 文件中。Makefile 的文件名参考分类命名，例如 `golang.mk`。最后，在 `/Makefile` 中 `include` 这些 `Makefile`。为了跟 `Makefile` 的层级相匹配，`golang.mk` 中的所有目标都按`go.xxx`这种方式命名。通过这种命名方式，我们可以很容易分辨出某个目标完成什么功能，放在什么文件里，这在复杂的 Makefile 中尤其有用。以下是 IAM 项目根目录下，Makefile 的内容摘录，你可以看一看，作为参考：

```bash
include scripts/make-rules/golang.mk
include scripts/make-rules/image.mk
include scripts/make-rules/gen.mk
include scripts/make-rules/...

## build: Build source code for host platform.
.PHONY: build
build:
  @$(MAKE) go.build

## build.multiarch: Build source code for multiple platforms. See option PLATFORMS.
.PHONY: build.multiarch
build.multiarch:
  @$(MAKE) go.build.multiarch

## image: Build docker images for host arch.
.PHONY: image
image:
  @$(MAKE) image.build

## push: Build docker images for host arch and push images to registry.
.PHONY: push
push:
  @$(MAKE) image.push

## ca: Generate CA files for all iam components.
.PHONY: ca
ca:
  @$(MAKE) gen.ca
```
另外，一个合理的 Makefile 结构应该具有前瞻性。也就是说，要在不改变现有结构的情况下，接纳后面的新功能。这就需要你整理好 Makefile 当前要实现的功能、即将要实现的功能和未来可能会实现的功能，然后基于这些功能，利用 Makefile 编程技巧，编写可扩展的 Makefile。

这里需要你注意：上面的 `Makefile` 通过 `.PHONY` 标识定义了大量的伪目标，定义伪目标一定要加 `.PHONY` 标识，否则当有同名的文件时，伪目标可能不会被执行。

##  掌握 Makefile 编写技巧
最后，在编写过程中，你还需要掌握一些 Makefile 的编写技巧，这些技巧可以使你编写的 Makefile 扩展性更强，功能更强大。接下来，我会把自己长期开发过程中积累的一些 Makefile 编写经验分享给你。这些技巧，你需要在实际编写中多加练习，并形成编写习惯。

###  技巧 1：善用通配符和自动变量
`Makefile` 允许对目标进行类似正则运算的匹配，主要用到的通配符是`%`。通过使用通配符，可以使不同的目标使用相同的规则，从而使 Makefile 扩展性更强，也更简洁。

我们的 IAM 实战项目中，就大量使用了通配符`%`，例如：`go.build.%`、`ca.gen.%`、`deploy.run.%`、`tools.verify.%`、`tools.install.%`等。

这里，我们来看一个具体的例子，`tools.verify.%`（位于`scripts/make-rules/tools.mk`文件中）定义如下：

```bash
tools.verify.%:
  @if ! which $* &>/dev/null; then $(MAKE) tools.install.$*; fi
```
`make tools.verify.swagger`, `make tools.verify.mockgen`等均可以使用上面定义的规则，`%`分别代表了`swagger`和`mockgen`。

如果不使用`%`，则我们需要分别为`tools.verify.swagger`和`tools.verify.mockgen`定义规则，很麻烦，后面修改也困难。

另外，这里也能看出`tools.verify.%`这种命名方式的好处：`tools` 说明依赖的定义位于`scripts/make-rules/tools.mk` Makefile 中；verify说明`tools.verify.%`伪目标属于 `verify` 分类，主要用来验证工具是否安装。通过这种命名方式，你可以很容易地知道目标位于哪个 Makefile 文件中，以及想要完成的功能。

另外，上面的定义中还用到了自动变量`$*`，用来指代被匹配的值`swagger`、`mockgen`。

###  技巧 2：善用函数
Makefile 自带的函数能够帮助我们实现很多强大的功能。所以，在我们编写 Makefile 的过程中，如果有功能需求，可以优先使用这些函数。我把常用的函数以及它们实现的功能整理在了 [Makefile 常用函数列表](https://github.com/marmotedu/geekbang-go/blob/master/makefile/Makefile%E5%B8%B8%E7%94%A8%E5%87%BD%E6%95%B0%E5%88%97%E8%A1%A8.md) 中，你可以参考下。

IAM 的 Makefile 文件中大量使用了上述函数，如果你想查看这些函数的具体使用方法和场景，可以参考 IAM 项目的 Makefile 文件 [make-rules](https://github.com/marmotedu/iam/tree/master/scripts/make-rules)。


###  技巧 3：依赖需要用到的工具
如果 `Makefile` 某个目标的命令中用到了某个工具，可以将该工具放在目标的依赖中。这样，当执行该目标时，就可以指定检查系统是否安装该工具，如果没有安装则自动安装，从而实现更高程度的自动化。例如，`Makefile` 文件中，`format` 伪目标，定义如下：

```bash

.PHONY: format
format: tools.verify.golines tools.verify.goimports
  @echo "===========> Formating codes"
  @$(FIND) -type f -name '*.go' | $(XARGS) gofmt -s -w
  @$(FIND) -type f -name '*.go' | $(XARGS) goimports -w -local $(ROOT_PACKAGE)
  @$(FIND) -type f -name '*.go' | $(XARGS) golines -w --max-len=120 --reformat-tags --shorten-comments --ignore-generated .
```
你可以看到，`format` 依赖`tools.verify.golines`, `tools.verify.goimports`。我们再来看下`tools.verify.golines`的定义：

```bash
tools.verify.%:
  @if ! which $* &>/dev/null; then $(MAKE) tools.install.$*; fi
```
再来看下`tools.install.$*`规则：

```bash
.PHONY: install.golines
install.golines:
  @$(GO) get -u github.com/segmentio/golines
```
通过`tools.verify.%`规则定义，我们可以知道，`tools.verify.%`会先检查工具是否安装，如果没有安装，就会执行`tools.install.$*`来安装。如此一来，当我们执行`tools.verify.%`目标时，如果系统没有安装 `golines` 命令，就会自动调用`go get`安装，提高了 `Makefile` 的自动化程度。


###  技巧 4：把常用功能放在 `/Makefile` 中，不常用的放在分类 Makefile 中
一个项目，尤其是大型项目，有很多需要管理的地方，其中大部分都可以通过 Makefile 实现自动化操作。不过，为了保持 `/Makefile` 文件的整洁性，我们不能把所有的命令都添加在 /Makefile 文件中。

一个比较好的建议是，将常用功能放在 /Makefile 中，不常用的放在分类 Makefile 中，并在 /Makefile 中 `include` 这些分类 Makefile。

例如，IAM 项目的 `/Makefile` 集成了`format`、`lint`、`test`、`build`等常用命令，而将`gen.errcode.code`、`gen.errcode.doc`这类不常用的功能放在 `scripts/make-rules/gen.mk` 文件中。当然，我们也可以直接执行 `make gen.errcode.code`来执行`gen.errcode.code`伪目标。通过这种方式，既可以保证 /Makefile 的简洁、易维护，又可以通过make命令来运行伪目标，更加灵活。

###  技巧 5：编写可扩展的 Makefile
什么叫可扩展的 `Makefile` 呢？在我看来，可扩展的 `Makefile` 包含两层含义：

1. 可以在不改变 Makefile 结构的情况下添加新功能。
2. 扩展项目时，新功能可以自动纳入到 Makefile 现有逻辑中。

其中的第一点，我们可以通过设计合理的 `Makefile` 结构来实现。要实现第二点，就需要我们在编写 Makefile 时采用一定的技巧，例如多用通配符、自动变量、函数等。这里我们来看一个例子，可以让你更好地理解。在我们 IAM 实战项目的`golang.mk`中，执行 `make go.build` 时能够构建 `cmd/` 目录下的所有组件，也就是说，当有新组件添加时， `make go.build` 仍然能够构建新增的组件，这就实现了上面说的第二点。

在我们 IAM 实战项目的`golang.mk`中，执行 `make go.build` 时能够构建 `cmd/` 目录下的所有组件，也就是说，当有新组件添加时， `make go.build` 仍然能够构建新增的组件，这就实现了上面说的第二点。具体实现方法如下：

```bash
COMMANDS ?= $(filter-out %.md, $(wildcard ${ROOT_DIR}/cmd/*))
BINS ?= $(foreach cmd,${COMMANDS},$(notdir ${cmd}))

.PHONY: go.build
go.build: go.build.verify $(addprefix go.build., $(addprefix $(PLATFORM)., $(BINS)))
.PHONY: go.build.%               

go.build.%:             
  $(eval COMMAND := $(word 2,$(subst ., ,$*)))
  $(eval PLATFORM := $(word 1,$(subst ., ,$*)))
  $(eval OS := $(word 1,$(subst _, ,$(PLATFORM))))           
  $(eval ARCH := $(word 2,$(subst _, ,$(PLATFORM))))                         
  @echo "===========> Building binary $(COMMAND) $(VERSION) for $(OS) $(ARCH)"
  @mkdir -p $(OUTPUT_DIR)/platforms/$(OS)/$(ARCH)
  @CGO_ENABLED=0 GOOS=$(OS) GOARCH=$(ARCH) $(GO) build $(GO_BUILD_FLAGS) -o $(OUTPUT_DIR)/platforms/$(OS)/$(ARCH)/$(COMMAND)$(GO_OUT_EXT) $(ROOT_PACKAGE)/cmd/$(COMMAND)
```

