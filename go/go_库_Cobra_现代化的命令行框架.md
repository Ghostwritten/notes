#  go 库 Cobra 现代化的命令行框架

![](https://i-blog.csdnimg.cn/blog_migrate/cecb9963d89543970af3ca4262b75ad3.png)






## 1. 简介
[Cobra](https://github.com/spf13/cobra) 既是一个可以创建强大的现代 CLI 应用程序的库，也是一个可以生成应用和命令文件的程序。有许多大型项目都是用 Cobra 来构建应用程序的，例如 `Kubernetes`、`Docker`、`etcd`、`Rkt`、`Hugo` 等。

Cobra 建立在 `commands`、arguments 和 flags 结构之上。
- commands 代表命令，
- arguments 代表非选项参数，
- flags 代表选项参数（也叫标志）。

一个好的应用程序应该是易懂的，用户可以清晰地知道如何去使用这个应用程序。应用程序通常遵循如下模式：`APPNAME VERB NOUN --ADJECTIVE`或者`APPNAME COMMAND ARG --FLAG`，例如：

```bash
# server是 commands，port 是 flag
$ hugo server --port=1313

# clone 是 commands，URL 是 arguments，brae 是 flag
$ git clone URL --bare # clone 是一个命令，URL是一个非选项参数，bare是一个选项参数


$ docker info --help 
Usage:  docker info [OPTIONS]

Display system-wide information

Options:
  -f, --format string   Format the output using the given Go template
```
这里，`VERB` 代表动词，`NOUN` 代表名词，`ADJECTIVE` 代表形容词。

Cobra 提供了两种方式来创建命令：Cobra 命令和 Cobra 库。Cobra 命令可以生成一个 Cobra 命令模板，而命令模板也是通过引用 Cobra 库来构建命令的。

## 2. 主要功能
cobra 的主要功能如下，可以说每一项都很实用：

- 简易的子命令行模式，如 app server， app fetch 等等
- 完全兼容 posix 命令行模式
- 嵌套子命令 subcommand
- 支持全局，局部，串联 flags
- 使用 cobra 很容易的生成应用程序和命令，使用 cobra create appname 和 cobra add cmdname
- 如果命令输入错误，将提供智能建议，如 app srver，将提示 srver 没有，是不是 app server
- 自动生成 commands 和 flags 的帮助信息
- 自动生成详细的 help 信息，如 app help
- 自动识别帮助 flag -h，--help
- 自动生成应用程序在 bash 下命令自动完成功能
- 自动生成应用程序的 man 手册
- 命令行别名
- 自定义 help 和 usage 信息
- 可选的与 viper apps 的紧密集成


## 3. 应用举例
obra 被用于许多 Go 项目中，例如：[Kubernetes](https://kubernetes.io/)、[Hugo](https://gohugo.io/)和[Github CLI](https://github.com/cli/cli)等，更多广泛使用的项目有：
- [Allero](https://github.com/allero-io/allero)
- [Arduino CLI](https://github.com/arduino/arduino-cli)
- [Bleve](https://blevesearch.com/)
- [Cilium](https://cilium.io/)
- [CloudQuery](https://github.com/cloudquery/cloudquery)
- [CockroachDB](https://www.cockroachlabs.com/)
- [Constellation](https://github.com/edgelesssys/constellation)
- [Cosmos SDK](https://github.com/cosmos/cosmos-sdk)
- [Datree](https://github.com/datreeio/datree)
- [Delve](https://github.com/derekparker/delve)
- [Docker (distribution)](https://github.com/docker/distribution)
- [Etcd](https://etcd.io/)
- [Gardener](https://github.com/gardener/gardenctl)
- [Giant Swarm's gsctl](https://github.com/giantswarm/gsctl)
- [Git Bump](https://github.com/erdaltsksn/git-bump)
- [GitHub CLI](https://github.com/cli/cli)
- [GitHub Labeler](https://github.com/erdaltsksn/gh-label)
- [Golangci-lint](https://golangci-lint.run)
- [GopherJS](https://github.com/gopherjs/gopherjs)
- [GoReleaser](https://goreleaser.com)
- [Helm](https://helm.sh)
- [Hugo](https://gohugo.io)
- [Infracost](https://github.com/infracost/infracost)
- [Istio](https://istio.io)
- [Kool](https://github.com/kool-dev/kool)
- [Kubernetes](https://kubernetes.io/)
- [Kubescape](https://github.com/kubescape/kubescape)
- [KubeVirt](https://github.com/kubevirt/kubevirt)
- [Linkerd](https://linkerd.io/)
- [Mattermost-server](https://github.com/mattermost/mattermost-server)
- [Mercure](https://mercure.rocks/)
- [Meroxa CLI](https://github.com/meroxa/cli)
- [Metal Stack CLI](https://github.com/metal-stack/metalctl)
- [Moby (former Docker)](https://github.com/moby/moby)
- [Moldy](https://github.com/Moldy-Community/moldy)
- [Multi-gitter](https://github.com/lindell/multi-gitter)
- [Nanobox](https://github.com/nanobox-io/nanobox)/[Nanopack](https://github.com/nanopack)
- [nFPM](https://nfpm.goreleaser.com)
- [Okteto](https://github.com/okteto/okteto)
- [OpenShift](https://www.openshift.com/)
- [Ory Hydra](https://github.com/ory/hydra)
- [Ory Kratos](https://github.com/ory/kratos)
- [Pixie](https://github.com/pixie-io/pixie)
- [Polygon Edge](https://github.com/0xPolygon/polygon-edge)
- [Pouch](https://github.com/alibaba/pouch)
- [ProjectAtomic (enterprise)](https://www.projectatomic.io/)
- [Prototool](https://github.com/uber/prototool)
- [Pulumi](https://www.pulumi.com)
- [QRcp](https://github.com/claudiodangelis/qrcp)
- [Random](https://github.com/erdaltsksn/random)
- [Rclone](https://rclone.org/)
- [Scaleway CLI](https://github.com/scaleway/scaleway-cli)
- [Sia](https://github.com/SiaFoundation/siad)
- [Skaffold](https://skaffold.dev/)
- [Tendermint](https://github.com/tendermint/tendermint)
- [Twitch CLI](https://github.com/twitchdev/twitch-cli)
- [UpCloud CLI (`upctl`)](https://github.com/UpCloudLtd/upcloud-cli)
- VMware's [Tanzu Community Edition](https://github.com/vmware-tanzu/community-edition) & [Tanzu Framework](https://github.com/vmware-tanzu/tanzu-framework)
- [Werf](https://werf.io/)
- [ZITADEL](https://github.com/zitadel/zitadel)


## 4. Cobra 安装
使用Cobra很简单。首先，使用go get安装最新版本的库。
```bash
$ go get -u github.com/spf13/cobra@latest
go: downloading github.com/inconshreveable/mousetrap v1.0.1
go: downloading github.com/inconshreveable/mousetrap v1.1.0
go: added github.com/inconshreveable/mousetrap v1.1.0
go: added github.com/spf13/cobra v1.6.1
go: added github.com/spf13/pflag v1.0.5
```
接下来 ,导入 Cobra:

```bash
import "github.com/spf13/cobra"
```

## 5. 使用 Cobra 库创建命令
如果要用 Cobra 库编码实现一个应用程序，需要首先创建一个空的 `main.go` 文件和一个 `rootCmd` 文件，之后可以根据需要添加其他命令。具体步骤如下：
### 5.1 创建 rootCmd

```bash
$ mkdir -p newApp2 && cd newApp2
```
通常情况下，我们会将 `rootCmd` 放在文件 `cmd/root.go` 中。

```bash
var rootCmd = &cobra.Command{
  Use:   "hugo",
  Short: "Hugo is a very fast static site generator",
  Long: `A Fast and Flexible Static Site Generator built with
                love by spf13 and friends in Go.
                Complete documentation is available at http://hugo.spf13.com`,
  Run: func(cmd *cobra.Command, args []string) {
    // Do Stuff Here
  },
}

func Execute() {
  if err := rootCmd.Execute(); err != nil {
    fmt.Println(err)
    os.Exit(1)
  }
}
```
还可以在 `init()` 函数中定义标志和处理配置，例如 `cmd/root.go`。

```bash
import (
  "fmt"
  "os"

  homedir "github.com/mitchellh/go-homedir"
  "github.com/spf13/cobra"
  "github.com/spf13/viper"
)

var (
    cfgFile     string
    projectBase string
    userLicense string
)

func init() {
  cobra.OnInitialize(initConfig)
  rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra.yaml)")
  rootCmd.PersistentFlags().StringVarP(&projectBase, "projectbase", "b", "", "base project directory eg. github.com/spf13/")
  rootCmd.PersistentFlags().StringP("author", "a", "YOUR NAME", "Author name for copyright attribution")
  rootCmd.PersistentFlags().StringVarP(&userLicense, "license", "l", "", "Name of license for the project (can provide `licensetext` in config)")
  rootCmd.PersistentFlags().Bool("viper", true, "Use Viper for configuration")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
  viper.BindPFlag("projectbase", rootCmd.PersistentFlags().Lookup("projectbase"))
  viper.BindPFlag("useViper", rootCmd.PersistentFlags().Lookup("viper"))
  viper.SetDefault("author", "NAME HERE <EMAIL ADDRESS>")
  viper.SetDefault("license", "apache")
}

func initConfig() {
  // Don't forget to read config either from cfgFile or from home directory!
  if cfgFile != "" {
    // Use config file from the flag.
    viper.SetConfigFile(cfgFile)
  } else {
    // Find home directory.
    home, err := homedir.Dir()
    if err != nil {
      fmt.Println(err)
      os.Exit(1)
    }

    // Search config in home directory with name ".cobra" (without extension).
    viper.AddConfigPath(home)
    viper.SetConfigName(".cobra")
  }

  if err := viper.ReadInConfig(); err != nil {
    fmt.Println("Can't read config:", err)
    os.Exit(1)
  }
}
```
###  5.2 创建 main.go
我们还需要一个 `main` 函数来调用 `rootCmd`，通常我们会创建一个 `main.go` 文件，在 `main.go` 中调用 `rootCmd.Execute()` 来执行命令：

```bash
package main

import (
  "newApp2/cmd/cmd"
)

func main() {
  cmd.Execute()
}
```
需要注意，`main.go` 中不建议放很多代码，通常只需要调用 `cmd.Execute()` 即可。

### 5.3 添加命令
除了 `rootCmd`，我们还可以调用 `AddCommand` 添加其他命令，通常情况下，我们会把其他命令的源码文件放在 `cmd/` 目录下，例如，我们添加一个 `version` 命令，可以创建 `cmd/version.go` 文件，内容为：

```bash
package cmd

import (
  "fmt"

  "github.com/spf13/cobra"
)

func init() {
  rootCmd.AddCommand(versionCmd)
}

var versionCmd = &cobra.Command{
  Use:   "version",
  Short: "Print the version number of Hugo",
  Long:  `All software has versions. This is Hugo's`,
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hugo Static Site Generator v0.9 -- HEAD")
  },
}
```
本示例中，我们通过调用`rootCmd.AddCommand(versionCmd)`给 `rootCmd` 命令添加了一个 `versionCmd` 命令。

### 5.4 编译并运行
创建`.cobra.yaml`文件

```bash
$ cat  /c/Users/XH/.cobra.yaml
author: Steve Francia <spf@spf13.com>
license: MIT
useViper: true
```

将 `main.go` 中`{pathToYourApp}`替换为对应的路径，例如本示例中 `pathToYourApp` 为`github.com/marmotedu/gopractise-demo/cobra/newApp2`

```bash
$ cd /d/goprojects/src/github.com/marmotedu/gopractise-demo/cobra/newApp2
$ go mod init newApp2
$ go build -v .
$ ./newApp2 -h
A Fast and Flexible Static Site Generator built with
love by spf13 and friends in Go.
Complete documentation is available at http://hugo.spf13.com
 
Usage:
hugo [flags]
hugo [command]
 
Available Commands:
help Help about any command
version Print the version number of Hugo
 
Flags:
-a, --author string Author name for copyright attribution (default "YOUR NAME")
--config string config file (default is $HOME/.cobra.yaml)
-h, --help help for hugo
-l, --license licensetext Name of license for the project (can provide licensetext in config)
-b, --projectbase string base project directory eg. github.com/spf13/
--viper Use Viper for configuration (default true)
 
Use "hugo [command] --help" for more information about a command.


$ $ ./newApp2.exe version
Hugo Static Site Generator v0.9 -- HEAD
```

## 6. 特性
### 6.1 使用标志

Cobra 可以跟 [Pflag](https://blog.csdn.net/xixihahalelehehe/article/details/105961377) 结合使用，实现强大的标志功能。使用步骤如下：

1. 使用持久化的标志
标志可以是“持久的”，这意味着该标志可用于它所分配的命令以及该命令下的每个子命令。可以在 `rootCmd` 上定义持久标志：

```bash
rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```
2. 使用本地标志
也可以分配一个本地标志，本地标志只能在它所绑定的命令上使用：

```bash
rootCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
```
`--source`标志只能在 rootCmd 上引用，而不能在 rootCmd 的子命令上引用。

3. 将标志绑定到 Viper
我们可以将标志绑定到 `Viper`，这样就可以使用 `viper.Get()` 获取标志的值。

```bash
var author string

func init() {
  rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
}
```
4. 设置标志为必选
默认情况下，标志是可选的，我们也可以设置标志为必选，当设置标志为必选，但是没有提供标志时，Cobra 会报错。

```bash
rootCmd.Flags().StringVarP(&Region, "region", "r", "", "AWS region (required)")
rootCmd.MarkFlagRequired("region")
```

### 6.2 非选项参数验证
在命令的过程中，经常会传入非选项参数，并且需要对这些非选项参数进行验证，Cobra 提供了机制来对非选项参数进行验证。可以使用 Command 的 Args 字段来验证非选项参数。Cobra 也内置了一些验证函数：

- `NoArgs`：如果存在任何非选项参数，该命令将报错。
- `ArbitraryArgs`：该命令将接受任何非选项参数。
- `OnlyValidArgs`：如果有任何非选项参数不在 Command 的 ValidArgs 字段中，该命令将报错。
- `MinimumNArgs(int)`：如果没有至少 N 个非选项参数，该命令将报错。
- `MaximumNArgs(int)`：如果有多于 N 个非选项参数，该命令将报错。
- `ExactArgs(int)`：如果非选项参数个数不为 N，该命令将报错。
- `ExactValidArgs(int)`：如果非选项参数的个数不为 N，或者非选项参数不在 Command 的 ValidArgs 字段中，该命令将报错。
- `RangeArgs(min, max)`：如果非选项参数的个数不在 min 和 max 之间，该命令将报错。

使用预定义验证函数，示例如下：

```bash
var cmd = &cobra.Command{
  Short: "hello",
  Args: cobra.MinimumNArgs(1), // 使用内置的验证函数
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hello, World!")
  },
}
```
当然你也可以自定义验证函数，示例如下：

```bash
var cmd = &cobra.Command{
  Short: "hello",
  // Args: cobra.MinimumNArgs(10), // 使用内置的验证函数
  Args: func(cmd *cobra.Command, args []string) error { // 自定义验证函数
    if len(args) < 1 {
      return errors.New("requires at least one arg")
    }
    if myapp.IsValidColor(args[0]) {
      return nil
    }
    return fmt.Errorf("invalid color specified: %s", args[0])
  },
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hello, World!")
  },
}
```
### 6.3 PreRun and PostRun Hooks
在运行 Run 函数时，我们可以运行一些钩子函数，比如 `PersistentPreRun` 和 `PreRun` 函数在 Run 函数之前执行，`PersistentPostRun` 和 `PostRun` 在 Run 函数之后执行。如果子命令没有指定Persistent*Run函数，则子命令将会继承父命令的Persistent*Run函数。这些函数的运行顺序如下：

- PersistentPreRun
- PreRun
- Run
- PostRun
- PersistentPostRun

注意，父级的 `PreRun` 只会在父级命令运行时调用，子命令是不会调用的。

Cobra 还支持很多其他有用的特性，比如：
- 自定义 `Help` 命令；
- 可以自动添加`--version`标志，输出程序版本信息；
- 当用户提供无效标志或无效命令时，Cobra 可以打印出 `usage` 信息；
- 当我们输入的命令有误时，Cobra 会根据注册的命令，推算出可能的命令，等等。

## 7. cobra-cli 命令

###  7.1 安装
`cobra-cli`是一个命令行程序，用于生成cobra应用程序和命令文件。它将引导您的应用程序脚手架，以快速开发基于cobra的应用程序。这是将Cobra合并到应用程序中最简单的方法。

可以通过以下命令安装:

```bash
$ go install github.com/spf13/cobra-cli@latest
go: downloading github.com/spf13/cobra-cli v1.3.0
go: downloading github.com/spf13/cobra v1.3.0
go: downloading github.com/spf13/viper v1.10.1
go: downloading github.com/fsnotify/fsnotify v1.5.1
go: downloading github.com/mitchellh/mapstructure v1.4.3
go: downloading github.com/magiconair/properties v1.8.5
go: downloading github.com/spf13/afero v1.6.0
go: downloading github.com/spf13/cast v1.4.1
go: downloading gopkg.in/ini.v1 v1.66.2
go: downloading github.com/pelletier/go-toml v1.9.4
go: downloading golang.org/x/text v0.3.7
```
`cobra-cli`命令自动将它安装在你的`$GOPATH/bin`目录中

```bash
$ ls $GOPATH/bin/cobra-cli.exe 
'D:\goprojects/bin/cobra-cli.exe'
```

### 7.2 cobra-cli 初始化
该`cobra-cli init [app]`命令将为您创建初始应用程序代码。这是一个非常强大的应用程序，可以用正确的结构填充您的程序，这样您就可以立即享受 Cobra 的所有好处。它还可以将您指定的许可证应用于您的应用程序。

随着 Go 模块的引入，Cobra 生成器已被简化以利用模块。Cobra 生成器在 Go 模块中工作。


如果你想初始化一个新的 Go 模块：

```bash
cd $HOME/code 
mkdir myapp
cd myapp
go mod init github.com/spf13/myapp
cobra-cli init
go run main.go
```
`cobra-cli init` 也可以从子目录运行，例如[cobra 生成器本身的组织方式](https://github.com/spf13/cobra)。如果您想将应用程序代码与库代码分开，这将很有用。

可选标志：
- 您可以为它提供您的作者姓名和`--author`标志。例如`cobra-cli init --author "Steve Francia spf@spf13.com"`

- 您可以提供使用许可，`--license` 例如`cobra-cli init --license apache`

- 使用`--viper`标志自动设置`viper`，Viper 是 Cobra 的辅助工具，旨在提供对环境变量和配置文件的轻松处理，并将它们无缝连接到应用程序标志。

### 7.3  向项目添加命令
初始化 Cobra 应用程序后，您可以继续使用 Cobra 生成器向您的应用程序添加其他命令。执行此操作的命令是cobra-cli add。

假设您创建了一个应用程序，并希望为它执行以下命令：

- 应用服务
- 应用配置
- 应用程序配置创建

在您的项目目录（您的 `main.go` 文件所在的位置）中，您将运行以下命令：

```bash
cobra-cli add serve
cobra-cli add config
cobra-cli add create -p 'configCmd'
```
您会注意到这个最终命令有一个`-p`标志。这用于将父命令分配给新添加的命令。在这种情况下，我们要将“`create`”命令分配给“`config`”命令。如果未指定，所有命令都具有 `rootCmd` 的默认父级。

默认情况下`cobra-cli`将附加Cmd到提供的名称并将此名称用作内部变量名称。指定父级时，一定要与代码中使用的变量名相匹配。

> 注意：命令名称使用驼峰命名法（不是 `snake_case/kebab-case`）。否则，您将遇到错误。例如，`cobra-cli add add-user`不正确，但`cobra-cli add addUser`有效。

运行这三个命令后，您将拥有类似于以下的应用程序结构：

```bash
  ▾ app/
    ▾ cmd/
        config.go
        create.go
        serve.go
        root.go
      main.go
```

此时您可以运行`go run main.go`，它会运行您的应用程序。`go run main.go serve`, `go run main.go config`,`go run main.go config create`连同`go run main.go help serve`, 等都可以。

您现在已经启动并运行了一个基于 `Cobra` 的基本应用程序。

参考：
- [Cobra 快速入门 - 专为命令行程序而生](https://xcbeyond.cn/blog/golang/cobra-quick-start/)
- [Golang : cobra 包简介](https://www.cnblogs.com/sparkdev/p/10856077.html)
