## 介绍
解析url
## 函数
### func PathEscape

```
func PathEscape(s string) string
```

PathEscape 会将字符串转义出来，以便将其安全地放置在 URL 路径段中。 

### func PathUnescape

```
func PathUnescape(s string) (string, error
```

PathUnescape 执行 PathEscape 的逆转换，将 ％AB 转换为字节 0xAB 。如果任何 ％ 之后没有两个十六进制数字，它将返回一个错误。

PathUnescape 与 QueryUnescape 相同，只是它不会将'+'改为''（空格）。

### func QueryEscape

```
func QueryEscape(s string) string
```

QueryEscape函数对s进行转码使之可以安全的用在URL查询里。

### func QueryUnescape

```
func QueryUnescape(s string) (string, error)
```

QueryUnescape函数用于将QueryEscape转码的字符串还原。它会把%AB改为字节0xAB，将'+'改为' '。如果有某个%后面未跟两个十六进制数字，本函数会返回错误。

示例：

```go
package main 
import(
    "fmt"
    "encoding/base64"
    "net/url"
    "crypto/rand"
    "io"
    "log"
)

//sessionId函数用来生成一个session ID，即session的唯一标识符
func sessionId() string {
    b := make([]byte, 32)
    //ReadFull从rand.Reader精确地读取len(b)字节数据填充进b
    //rand.Reader是一个全局、共享的密码用强随机数生成器
    if _, err := io.ReadFull(rand.Reader, b); err != nil { 
        return ""
    }
    fmt.Println(b) //[238 246 235 166 48 196 157 143 123 140 241 200 213 113 247 168 219 132 208 163 223 24 72 162 114 30 175 205 176 117 139 118]
    return base64.URLEncoding.EncodeToString(b)//将生成的随机数b编码后返回字符串,该值则作为session ID
}
func main() { 
    sessionId := sessionId() 
    fmt.Println(sessionId) //7vbrpjDEnY97jPHI1XH3qNuE0KPfGEiich6vzbB1i3Y=
    encodedSessionId := url.QueryEscape(sessionId) //对sessionId进行转码使之可以安全的用在URL查询里
    fmt.Println(encodedSessionId) //7vbrpjDEnY97jPHI1XH3qNuE0KPfGEiich6vzbB1i3Y%3D
    decodedSessionId, err := url.QueryUnescape(encodedSessionId) //将QueryEscape转码的字符串还原
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(decodedSessionId) //7vbrpjDEnY97jPHI1XH3qNuE0KPfGEiich6vzbB1i3Y=
}
```
### type URL

```go
type URL struct {
    Scheme   string    //具体指访问服务器上的资源使用的哪种协议
    Opaque   string    // 编码后的不透明数据
    User     *Userinfo // 用户名和密码信息,有些协议需要传入明文用户名和密码来获取资源，比如 FTP
    Host     string    // host或host:port，服务器地址，可以是 IP 地址，也可以是域名信息
    Path     string  //路径，使用"/"分隔
    RawQuery string // 编码后的查询字符串，没有'?'
    Fragment string // 引用的片段（文档位置），没有'#'
}
```
URL类型代表一个解析后的URL（或者说，一个URL参照）。URL基本格式如下：

```go
scheme://[userinfo@]host/path[?query][#fragment]
```

scheme后不是冒号加双斜线的URL被解释为如下格式：

```bash
scheme:opaque[?query][#fragment]
```
示例：

```go
package main
import (
        "fmt"
        "net/url"
        "strings"
)
func main() {
        Url := "https://root:123456@www.baidu.com:0000/login?name=xiaoming&name=xiaoqing&age=24&age1=23#fffffff"
        //Parse函数解析Url为一个URL结构体，Url可以是绝对地址，也可以是相对地址
        //        type URL struct {
        //    Scheme   string
        //    Opaque   string    // 编码后的不透明数据
        //    User     *Userinfo // 用户名和密码信息
        //    Host     string    // host或host:port
        //    Path     string
        //    RawQuery string // 编码后的查询字符串，没有'?'
        //    Fragment string // 引用的片段（文档位置），没有'#'
        //}
        u, err := url.Parse(Url)
        if err == nil {
                fmt.Println(u)
        }
        //ParseRequestURI函数解析Url为一个URL结构体，本函数会假设Url是在一个HTTP请求里，
        //        因此会假设该参数是一个绝对URL或者绝对路径，并会假设该URL没有#fragment后缀
        u1, err := url.ParseRequestURI(Url)
        if err == nil {
                fmt.Println(u1)
        }
        //得到Scheme
        fmt.Println(u.Scheme)
        //User 包含认证信息 username and password
        user := u.User
        fmt.Println(user)
        username := user.Username()
        fmt.Println(username)
        password, b := user.Password()
        fmt.Println(password, b)
        ////Host 包括主机名和端口信息，如过有端口，用strings.Split() 从Host中手动提取端口
        host := u.Host
        fmt.Println(host)
        ho := strings.Split(host, ":")
        fmt.Println("主机名：", ho[0])
        fmt.Println("端口号：", ho[1])
        //获取path
        path := u.Path
        fmt.Println(path)
        //获取参数 将查询参数解析为一个map。 map以查询字符串为键，对应值字符串切片为值。
        fmt.Println(u.RawQuery)
        urlParam := u.RawQuery
        fmt.Println("urlParam:", urlParam)
        m, err := url.ParseQuery(urlParam)
        if err == nil {
                fmt.Println(m)
                for k, v := range m {
                        fmt.Println(k, v)
                }
        }
        fmt.Println("****************************")
        //与ParseQuery功能一样，只是将上边的方法分装了一下
        m1 := u.Query()
        fmt.Println(m1)
        for k, v := range m1 {
                fmt.Println(k, v)
        }
        //得到查询片段信息
        fmt.Println(u.Fragment)
        ////生成参数形如：name=xiaoming&name=xiaoqing&age=24&age1=23
        v := url.Values{}
        //添加一个k-v值
        v.Set("name", "xiaoming")
        v.Add("name", "xiaoqing")
        v.Set("Age", "23")
        fmt.Println(v)
        fmt.Println(v.Get("name"))
        v.Del("name")
        fmt.Println(v)
        //把map编码成name=xiaoming&name=xiaoqing&age=24&age1=23的形式
        v.Set("name", "xiaoming")
        v.Add("name", "xiaoqing")
        fmt.Println(v.Encode())
}
```

输出结果：

```go
https://root:123456@www.baidu.com:0000/login?name=xiaoming&name=xiaoqing&age=24&age1=23#fffffff
https://root:123456@www.baidu.com:0000/login?name=xiaoming&name=xiaoqing&age=24&age1=23#fffffff
https
root:123456
root
123456 true
www.baidu.com:0000
主机名： www.baidu.com
端口号： 0000
/login
name=xiaoming&name=xiaoqing&age=24&age1=23
urlParam: name=xiaoming&name=xiaoqing&age=24&age1=23
map[name:[xiaoming xiaoqing] age:[24] age1:[23]]
name [xiaoming xiaoqing]
age [24]
age1 [23]
****************************
map[age:[24] age1:[23] name:[xiaoming xiaoqing]]
name [xiaoming xiaoqing]
age [24]
age1 [23]
fffffff
map[name:[xiaoming xiaoqing] Age:[23]]
xiaoming
map[Age:[23]]
Age=23&name=xiaoming&name=xiaoqing
```
参考资料：
[https://golang.org/pkg/net/url/](https://golang.org/pkg/net/url/)
