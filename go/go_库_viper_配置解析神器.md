#  go 库 viper 配置解析神器
![在这里插入图片描述](https://img-blog.csdnimg.cn/6a510a88eef9492f9bae9b558f9c893f.png)






## 1. 简介
几乎所有的后端服务，都需要一些配置项来配置我们的服务，一些小型的项目，配置不是很多，可以选择只通过命令行参数来传递配置。但是大型项目配置很多，通过命令行参数传递就变得很麻烦，不好维护。标准的解决方案是将这些配置信息保存在配置文件中，由程序启动时加载和解析。Go 生态中有很多包可以加载并解析配置文件，目前最受欢迎的是 Viper 包。Viper 是 Go 应用程序现代化的、完整的解决方案，能够处理不同格式的配置文件，让我们在构建现代应用程序时，不必担心配置文件格式。Viper 也能够满足我们对应用配置的各种需求。

Viper 可以从不同的位置读取配置，不同位置的配置具有不同的优先级，高优先级的配置会覆盖低优先级相同的配置，按优先级从高到低排列如下：
- 通过 viper.Set 函数显示设置的配置
- 命令行参数
- 环境变量
- 配置文件
- Key/Value 存储
- 默认值

Viper 有很多功能，最重要的两类功能是读入配置和读取配置，Viper 提供不同的方式来实现这两类功能。

## 2. 安装

```bash
go get github.com/spf13/viper
```


## 3. 建立默认值
一个好的配置系统应该支持默认值。键不需要默认值，但如果没有通过配置文件、环境变量、远程配置或命令行标志（flag）设置键，则默认值非常有用。
代码  viper1.go
```bash
package main

import (
  "fmt"
  "log"

  "github.com/spf13/viper"
)
func main() {
	viper.SetConfigName("config1")
	viper.SetConfigType("toml")
	viper.AddConfigPath(".")
	viper.SetDefault("ContentDir", "content")
    viper.SetDefault("LayoutDir", "layouts")
    viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categories"})
	err := viper.ReadInConfig()
	if err != nil {
	  log.Fatal("read config failed: %v", err)
	}

	fmt.Println("ContentDir: ", viper.GetString("ContentDir"))
	fmt.Println("LayoutDir: ", viper.GetString("LayoutDir"))
	fmt.Println("Taxonomies: ", viper.Get("Taxonomies"))

}
```

执行：

```bash
$ go run viper4.go
ContentDir:  content
LayoutDir:  layouts
Taxonomies:  map[category:categories tag:tags]
```
## 4. 读取配置文件
Viper需要最少知道在哪里查找配置文件的配置。Viper支持JSON、TOML、YAML、HCL、envfile和Java properties格式的配置文件。Viper可以搜索多个路径，但目前单个Viper实例只支持单个配置文件。Viper不默认任何配置搜索路径，将默认决策留给应用程序。

下面是一个如何使用Viper搜索和读取配置文件的示例。不需要任何特定的路径，但是至少应该提供一个配置文件预期出现的路径。

```bash
viper.SetConfigFile("./config.yaml") // 指定配置文件路径
viper.SetConfigName("config") // 配置文件名称(无扩展名)
viper.SetConfigType("yaml") // 如果配置文件的名称中没有扩展名，则需要配置此项
viper.AddConfigPath("/etc/appname/")   // 查找配置文件所在的路径
viper.AddConfigPath("$HOME/.appname")  // 多次调用以添加多个搜索路径
viper.AddConfigPath(".")               // 还可以在工作目录中查找配置
err := viper.ReadInConfig() // 查找并读取配置文件
if err != nil { // 处理读取配置文件的错误
	panic(fmt.Errorf("Fatal error config file: %s \n", err))
}
```
在加载配置文件出错时，你可以像下面这样处理找不到配置文件的特定情况：

```bash
if err := viper.ReadInConfig(); err != nil {
    if _, ok := err.(viper.ConfigFileNotFoundError); ok {
        // 配置文件未找到错误；如果需要可以忽略
    } else {
        // 配置文件被找到，但产生了另外的错误
    }
}

// 配置文件找到并成功解析
```



## 5. 获取 key/value 方法
Viper 提供了如下方法来读取配置：

- Get(key string) : interface{}
- GetBool(key string) : bool
- GetFloat64(key string) : float64
- GetInt(key string) : int
- GetIntSlice(key string) : []int
- GetString(key string) : string
- GetStringMap(key string) : map[string]interface{}
- GetStringMapString(key string) : map[string]string
- GetStringSlice(key string) : []string
- GetTime(key string) : time.Time
- GetDuration(key string) : time.Duration
- IsSet(key string) : bool
- AllSettings() : map[string]interface{}

每一个 Get 方法在找不到值的时候都会返回零值。为了检查给定的键是否存在，可以使用 IsSet() 方法。<Type>可以是 Viper 支持的类型，首字母大写：Bool、Float64、Int、IntSlice、String、StringMap、StringMapString、StringSlice、Time、Duration。例如：`GetInt()`。

###  5.1 Get() 方法
配置文件：`config.toml`
```bash
app_name = "awesome web"

# possible values: DEBUG, INFO, WARNING, ERROR, FATAL
log_level = "DEBUG"

[mysql]
ip = "127.0.0.1"
port = 3306
user = "dj"
password = 123456
database = "awesome"

[redis]
ip = "127.0.0.1"
port = 7381
```
代码 `viper1.go`：

```bash
package main

import (
  "fmt"
  "log"

  "github.com/spf13/viper"
)

func main() {
  viper.SetConfigName("config")
  viper.SetConfigType("toml")
  viper.AddConfigPath(".")
  viper.SetDefault("redis.port", 6381)
  err := viper.ReadInConfig()
  if err != nil {
    log.Fatal("read config failed: %v", err)
  }

  fmt.Println(viper.Get("app_name"))
  fmt.Println(viper.Get("log_level"))

  fmt.Println("mysql ip: ", viper.Get("mysql.ip"))
  fmt.Println("mysql port: ", viper.Get("mysql.port"))
  fmt.Println("mysql user: ", viper.Get("mysql.user"))
  fmt.Println("mysql password: ", viper.Get("mysql.password"))
  fmt.Println("mysql database: ", viper.Get("mysql.database"))

  fmt.Println("redis ip: ", viper.Get("redis.ip"))
  fmt.Println("redis port: ", viper.Get("redis.port"))
}
```
设置文件名（`SetConfigName`）、配置类型（`SetConfigType`）和搜索路径（`AddConfigPath`），然后调用`ReadInConfig`。 viper会自动根据类型来读取配置。使用时调用`viper.Get`方法获取键值。
执行：

```bash
$ go run viper1.go
awesome web
DEBUG
mysql ip:  127.0.0.1
mysql port:  3306
mysql user:  dj
mysql password:  123456
mysql database:  awesome
redis ip:  127.0.0.1
redis port:  7381
```
有几点需要注意：

- 设置文件名时不要带后缀；
- 搜索路径可以设置多个，viper 会根据设置顺序依次查找；
- viper 获取值时使用`section.key`的形式，即传入嵌套的键名；
- 默认值可以调用`viper.SetDefault`设置。

### 5.2 IsSet()、GetStringMap()、GetStringMap() 方法
viper 提供了多种形式的读取方法。在上面的例子中，我们看到了Get方法的用法。Get方法返回一个`interface{}`的值，使用有所不便。
GetType系列方法可以返回指定类型的值。其中，Type 可以为Bool、Float64、Int、String、Time、Duration、IntSlice、StringSlice。但是请注意，如果指定的键不存在或类型不正确，GetType方法返回对应类型的零值。

如果要判断某个键是否存在，使用`IsSet`方法。另外，`GetStringMap`和`GetStringMapString`直接以 `map` 返回某个键下面所有的键值对，前者返回`map[string]interface{}`，后者返回`map[string]string`。`AllSettings`以`map[string]interface{}`返回所有设置。

配置文件`config.toml`

```bash
app_name = "awesome web"

# possible values: DEBUG, INFO, WARNING, ERROR, FATAL
log_level = "DEBUG"

[server]
protocols = ["http", "https", "port"]
ports = [10000, 10001, 10002]
timeout = '3s'

[mysql]
ip = "127.0.0.1"
port = 3306
user = "dj"
password = 123456
database = "awesome"

[redis]
ip = "127.0.0.1"
port = 7381
```
`viper2.go` 文件
```bash
package main

import (
  "fmt"
  "log"

  "github.com/spf13/viper"
)

func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("toml")
	viper.AddConfigPath(".")
	err := viper.ReadInConfig()
	if err != nil {
	  log.Fatal("read config failed: %v", err)
	}
  
	fmt.Println("protocols: ", viper.GetStringSlice("server.protocols"))
	fmt.Println("ports: ", viper.GetIntSlice("server.ports"))
	fmt.Println("timeout: ", viper.GetDuration("server.timeout"))
  
	fmt.Println("mysql ip: ", viper.GetString("mysql.ip"))
	fmt.Println("mysql port: ", viper.GetInt("mysql.port"))
  
	if viper.IsSet("redis.port") {
	  fmt.Println("redis.port is set")
	} else {
	  fmt.Println("redis.port is not set")
	}
  
	fmt.Println("mysql settings: ", viper.GetStringMap("mysql"))
	fmt.Println("redis settings: ", viper.GetStringMap("redis"))
	fmt.Println("all settings: ", viper.AllSettings())
  }
  
```
如果将配置中的`redis.port`注释掉，将输出`redis.port is not set`。

上面的示例中还演示了如何使用`time.Duration`类型，只要是`time.ParseDuration`接受的格式都可以，例如`3s`、`2min`、`1min30s`等。
执行：

```bash
$ go run viper2.go
protocols:  [http https port]
ports:  [10000 10001 10002]
timeout:  3s
mysql ip:  127.0.0.1
mysql port:  3306
redis.port is set
mysql settings:  map[database:awesome ip:127.0.0.1 password:123456 port:3306 user:dj]
redis settings:  map[ip:127.0.0.1 port:7381]
all settings:  map[app_name:awesome web log_level:DEBUG mysql:map[database:awesome ip:127.0.0.1 password:123456 port:3306 user:dj] redis:map[ip:127.0.0.1 port:7381] server:map[ports:[10000 10001 10002] protocols:[http https port] timeout:3s]]     
```


## 6. 命令行选项
如果一个键没有通过`viper.Set`显示设置值，那么获取时将尝试从命令行选项中读取。 如果有，优先使用。viper 使用 [pflag](https://blog.csdn.net/xixihahalelehehe/article/details/105961377) 库来解析选项。 我们首先在`init`方法中定义选项，并且调用`viper.BindPFlags`绑定选项到配置中：

代码 `viper.go`
```bash
package main

import (
	"fmt"
	"log"

	"github.com/spf13/pflag"
	"github.com/spf13/viper"
)

func init() {
	pflag.Int("redis.port", 8381, "Redis port to connect")

	// 绑定命令行
	viper.BindPFlags(pflag.CommandLine)
}

func main() {
	pflag.Parse()

	viper.SetConfigName("config")
	viper.SetConfigType("toml")
	viper.AddConfigPath(".")
	err := viper.ReadInConfig()
	if err != nil {
		log.Fatal("read config failed: %v", err)
	}

	fmt.Println(viper.Get("app_name"))
	fmt.Println(viper.Get("log_level"))

	fmt.Println("mysql ip: ", viper.Get("mysql.ip"))
	fmt.Println("mysql port: ", viper.Get("mysql.port"))
	fmt.Println("mysql user: ", viper.Get("mysql.user"))
	fmt.Println("mysql password: ", viper.Get("mysql.password"))
	fmt.Println("mysql database: ", viper.Get("mysql.database"))

	fmt.Println("redis ip: ", viper.Get("redis.ip"))
	fmt.Println("redis port: ", viper.Get("redis.port"))
}
```
执行：

```bash
$ go run viper8.go
awesome web
DEBUG
mysql ip:  127.0.0.1
mysql port:  3306
mysql user:  root
mysql password:  123456
mysql database:  awesome
redis ip:  127.0.0.1
redis port:  6381


$ go run viper8.go --redis.port 10831
awesome web
DEBUG
mysql ip:  127.0.0.1
mysql port:  3306
mysql user:  root
mysql password:  123456
mysql database:  awesome
redis ip:  127.0.0.1
redis port:  10831
```


## 7. 访问嵌套的键
例如，加载下面的 JSON 文件 `config.json`：

```bash
{
    "host": {
        "address": "localhost",
        "port": 5799
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}
```

`viper3.go`文件

```bash
package main

import (
  "fmt"
  "log"

  "github.com/spf13/viper"
)

func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("json")
	viper.AddConfigPath(".")
	err := viper.ReadInConfig()
	if err != nil {
	  log.Fatal("read config failed: %v", err)
	}

	fmt.Println("host: ", viper.GetString("datastore.metric.host"))
	
}
```
执行：

```bash
$ go run viper3.go
host:  127.0.0.1
```
这遵守上面建立的优先规则；搜索路径将遍历其余配置注册表，直到找到为止。(译注：因为Viper支持从多种配置来源，例如磁盘上的`配置文件 > 命令行标志位 > 环境变量 > 远程Key/Value存储 > 默认值`，我们在查找一个配置的时候如果在当前配置源中没找到，就会继续从后续的配置源查找，直到找到为止。)

例如，在给定此配置文件的情况下，`datastore.metric.host`和`datastore.metric.port`均已定义（并且可以被覆盖）。如果另外在默认值中定义了d`atastore.metric.protocol`，Viper也会找到它。

然而，如果`datastore.metric`被直接赋值覆盖（被flag，环境变量，set()方法等等…），那么`datastore.metric`的所有子键都将变为未定义状态，它们被高优先级配置级别“遮蔽”（shadowed）了。

最后，如果存在与分隔的键路径匹配的键，则返回其值。例如：

```bash
{
    "datastore.metric.host": "0.0.0.0",
    "host": {
        "address": "localhost",
        "port": 5799
    },
    "datastore": {
        "metric": {
            "host": "127.0.0.1",
            "port": 3099
        },
        "warehouse": {
            "host": "198.0.0.1",
            "port": 2112
        }
    }
}

GetString("datastore.metric.host") // 返回 "0.0.0.0"
```


## 8. 写入配置文件
从配置文件中读取配置文件是有用的，但是有时你想要存储在运行时所做的所有修改。为此，可以使用下面一组命令，每个命令都有自己的用途:

- `WriteConfig` - 将当前的viper配置写入预定义的路径并覆盖（如果存在的话）。如果没有预定义的路径，则报错。
- `SafeWriteConfig` - 将当前的viper配置写入预定义的路径。如果没有预定义的路径，则报错。如果存在，将不会覆盖当前的配置文件。
- `WriteConfigAs` - 将当前的viper配置写入给定的文件路径。将覆盖给定的文件(如果它存在的话)。
- `SafeWriteConfigAs` - 将当前的viper配置写入给定的文件路径。不会覆盖给定的文件(如果它存在的话)。

根据经验，标记为`safe`的所有方法都不会覆盖任何文件，而是直接创建（如果不存在），而默认行为是创建或截断。

```bash
viper.WriteConfig() // 将当前配置写入“viper.AddConfigPath()”和“viper.SetConfigName”设置的预定义路径
viper.SafeWriteConfig()
viper.WriteConfigAs("/path/to/my/.config")
viper.SafeWriteConfigAs("/path/to/my/.config") // 因为该配置文件写入过，所以会报错
viper.SafeWriteConfigAs("/path/to/my/.other_config")
```
代码：

```bash
package main

import (
  "log"

  "github.com/spf13/viper"
)

func main() {
  viper.SetConfigName("config")
  viper.SetConfigType("toml")
  viper.AddConfigPath(".")

  viper.Set("app_name", "awesome web")
  viper.Set("log_level", "DEBUG")
  viper.Set("mysql.ip", "127.0.0.1")
  viper.Set("mysql.port", 3306)
  viper.Set("mysql.user", "root")
  viper.Set("mysql.password", "123456")
  viper.Set("mysql.database", "awesome")

  viper.Set("redis.ip", "127.0.0.1")
  viper.Set("redis.port", 6381)

  err := viper.SafeWriteConfig()
  if err != nil {
    log.Fatal("write config failed: ", err)
  }
}
```
执行：

```bash
$ go run viper5.go
$ cat config.toml
app_name = 'awesome web'
log_level = 'DEBUG'

[mysql]
database = 'awesome'
ip = '127.0.0.1'
password = '123456'
port = 3306
user = 'root'

[redis]
ip = '127.0.0.1'
port = 6381
```


## 9. 监控并重新读取配置文件
Viper支持在运行时实时读取配置文件的功能。

需要重新启动服务器以使配置生效的日子已经一去不复返了，viper驱动的应用程序可以在运行时读取配置文件的更新，而不会错过任何消息。

只需告诉viper实例`watchConfig`。可选地，你可以为Viper提供一个回调函数，以便在每次发生更改时运行。

```bash
package main

import (
  "fmt"
  "log"
  "time"

  "github.com/spf13/viper"
)

func main() {
  viper.SetConfigName("config")
  viper.SetConfigType("toml")
  viper.AddConfigPath(".")
  err := viper.ReadInConfig()
  if err != nil {
    log.Fatal("read config failed: %v", err)
  }

  viper.WatchConfig()

  fmt.Println("redis port before sleep: ", viper.Get("redis.port"))
  time.Sleep(time.Second * 10)
  fmt.Println("redis port after sleep: ", viper.Get("redis.port"))
}
```
只需要调用`viper.WatchConfig`，viper 会自动监听配置修改。如果有修改，重新加载的配置。
上面程序中，我们先打印`redis.port`的值，然后`Sleep 10s`。在这期间修改配置中`redis.port`的值，Sleep结束后再次打印。
发现打印出修改后的值：

```bash
redis port before sleep:  7381
redis port after sleep:  73810
```
另外，还可以为配置修改增加一个回调：

```bash
viper.OnConfigChange(func(e fsnotify.Event) {
  fmt.Printf("Config file:%s Op:%s\n", e.Name, e.Op)
})
```
这样文件修改时会执行这个回调。

viper 使用[fsnotify](https://github.com/fsnotify/fsnotify)这个库来实现监听文件修改的功能。


## 10. 从io.Reader中读取
viper 支持从io.Reader中读取配置。这种形式很灵活，来源可以是文件，也可以是程序中生成的字符串，甚至可以从网络连接中读取的字节流。

```bash
package main

import (
  "bytes"
  "fmt"
  "log"

  "github.com/spf13/viper"
)

func main() {
  viper.SetConfigType("toml")
  tomlConfig := []byte(`
app_name = "awesome web"

# possible values: DEBUG, INFO, WARNING, ERROR, FATAL
log_level = "DEBUG"

[mysql]
ip = "127.0.0.1"
port = 3306
user = "dj"
password = 123456
database = "awesome"

[redis]
ip = "127.0.0.1"
port = 7381
`)
  err := viper.ReadConfig(bytes.NewBuffer(tomlConfig))
  if err != nil {
    log.Fatal("read config failed: %v", err)
  }

  fmt.Println("redis port: ", viper.GetInt("redis.port"))
}
```

## 11. Unmarshal
`viper` 支持将配置`Unmarshal`到一个结构体中，为结构体中的对应字段赋值。

```bash
package main

import (
  "fmt"
  "log"

  "github.com/spf13/viper"
)

type Config struct {
  AppName  string
  LogLevel string

  MySQL    MySQLConfig
  Redis    RedisConfig
}

type MySQLConfig struct {
  IP       string
  Port     int
  User     string
  Password string
  Database string
}

type RedisConfig struct {
  IP   string
  Port int
}

func main() {
  viper.SetConfigName("config")
  viper.SetConfigType("toml")
  viper.AddConfigPath(".")
  err := viper.ReadInConfig()
  if err != nil {
    log.Fatal("read config failed: %v", err)
  }

  var c Config
  viper.Unmarshal(&c)

  fmt.Println(c.MySQL)
}
```
执行：

```bash
$ go run viper7.go
{127.0.0.1 3306 root 123456 awesome}
```

## 12. 环境变量
如果前面都没有获取到键值，将尝试从环境变量中读取。我们既可以一个个绑定，也可以自动全部绑定。

在init方法中调用`AutomaticEnv`方法绑定全部环境变量：

```bash
func init() {
  // 绑定环境变量
  viper.AutomaticEnv()
}
```
为了验证是否绑定成功，通过 系统 -> 高级设置 -> 新建 创建一个名为redis.port的环境变量，值为 10381。 运行程序，输出的redis.port值为 10381，并且输出中有 GOPATH 信息。

```bash
package main

import (
  "fmt"

  "github.com/spf13/viper"
)

func init() {
	// 绑定环境变量
	viper.BindEnv("redis.port")
	viper.BindEnv("go.path", "GOPATH")
  }

func main() {
	// 省略部分代码
  
	fmt.Println("GOPATH: ", viper.Get("go.path"))
	fmt.Println("redis.port: ", viper.Get("redis.port"))
  }
```
执行：

```bash
$   go run viper9.go
GOPATH:  D:\goprojects
redis.port:  10381
```
用`BindEnv`方法，如果只传入一个参数，则这个参数既表示键名，又表示环境变量名。如果传入两个参数，则第一个参数表示键名，第二个参数表示环境变量名。还可以通过`viper.SetEnvPrefix`方法设置环境变量前缀，这样一来，通过`AutomaticEnv`和一个参数的`BindEnv`绑定的环境变量，在使用`Get`的时候，viper 会自动加上这个前缀再从环境变量中查找。如果对应的环境变量不存在，viper 会自动将键名全部转为大写再查找一次。所以，使用键名`gopath`也能读取环境变量`GOPATH`的值。


## 13. 远程Key/Value存储支持
在`Viper`中启用远程支持，需要在代码中匿名导入`viper/remote`这个包。

```bash
import _ "github.com/spf13/viper/remote"
```

`Viper`将读取从`Key/Value`存储（例如etcd或Consul）中的路径检索到的配置字符串（如JSON、TOML、YAML、HCL、envfile和Java properties格式）。这些值的优先级高于默认值，但是会被从磁盘、flag或环境变量检索到的配置值覆盖。（译注：也就是说Viper加载配置值的优先级为：磁盘上的配置文件>命令行标志位>环境变量>远程Key/Value存储>默认值。）

Viper使用[crypt](https://github.com/bketelsen/crypt)从K/V存储中检索配置，这意味着如果你有正确的gpg密匙，你可以将配置值加密存储并自动解密。加密是可选的。

你可以将远程配置与本地配置结合使用，也可以独立使用。

crypt有一个命令行助手，你可以使用它将配置放入K/V存储中。crypt默认使用在`http://127.0.0.1:4001`的`etcd`。

```bash
$ go get github.com/bketelsen/crypt/bin/crypt
$ crypt set -plaintext /config/hugo.json /Users/hugo/settings/config.json
```
确认值已经设置：

```bash
$ crypt get -plaintext /config/hugo.json
```
有关如何设置加密值或如何使用Consul的示例，请参见crypt文档。

### 13.1 远程Key/Value存储示例-未加密

- etcd

```bash
viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001","/config/hugo.json")
viper.SetConfigType("json") // 因为在字节流中没有文件扩展名，所以这里需要设置下类型。支持的扩展名有 "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"
err := viper.ReadRemoteConfig()
```
- Consul
你需要 Consul `Key/Value`存储中设置一个Key保存包含所需配置的JSON值。例如，创建一个key `MY_CONSUL_KEY`将下面的值存入Consul `key/value` 存储：

```bash
{
    "port": 8080,
    "hostname": "liwenzhou.com"
}
```

```bash
viper.AddRemoteProvider("consul", "localhost:8500", "MY_CONSUL_KEY")
viper.SetConfigType("json") // 需要显示设置成json
err := viper.ReadRemoteConfig()

fmt.Println(viper.Get("port")) // 8080
fmt.Println(viper.Get("hostname")) // liwenzhou.com
```
- Firestore

```bash
viper.AddRemoteProvider("firestore", "google-cloud-project-id", "collection/document")
viper.SetConfigType("json") // 配置的格式: "json", "toml", "yaml", "yml"
err := viper.ReadRemoteConfig()
```
当然，你也可以使用`SecureRemoteProvider`

### 13.2  远程Key/Value存储示例-加密

```bash
viper.AddSecureRemoteProvider("etcd","http://127.0.0.1:4001","/config/hugo.json","/etc/secrets/mykeyring.gpg")
viper.SetConfigType("json") // 因为在字节流中没有文件扩展名，所以这里需要设置下类型。支持的扩展名有 "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"
err := viper.ReadRemoteConfig()
```
### 13.3 监控etcd中的更改-未加密

```bash
// 或者你可以创建一个新的viper实例
var runtime_viper = viper.New()

runtime_viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001", "/config/hugo.yml")
runtime_viper.SetConfigType("yaml") // 因为在字节流中没有文件扩展名，所以这里需要设置下类型。支持的扩展名有 "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"

// 第一次从远程读取配置
err := runtime_viper.ReadRemoteConfig()

// 反序列化
runtime_viper.Unmarshal(&runtime_conf)

// 开启一个单独的goroutine一直监控远端的变更
go func(){
	for {
	    time.Sleep(time.Second * 5) // 每次请求后延迟一下

	    // 目前只测试了etcd支持
	    err := runtime_viper.WatchRemoteConfig()
	    if err != nil {
	        log.Errorf("unable to read remote config: %v", err)
	        continue
	    }

	    // 将新配置反序列化到我们运行时的配置结构体中。你还可以借助channel实现一个通知系统更改的信号
	    runtime_viper.Unmarshal(&runtime_conf)
	}
}()
```

参考：
- [李文周的博客：Go语言配置管理神器——Viper中文教程](https://www.liwenzhou.com/posts/Go/viper/)
- [Go 每日一库之 viper](https://juejin.cn/post/6844904051369312264)
- [go 应用构建三剑客：Pflag、Viper、Cobra 核心功能介绍](https://time.geekbang.org/column/article/395705)
