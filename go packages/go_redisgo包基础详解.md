

------


## 1 介绍
redigo是golang的一个操作redis的第三方库，之所以选择这个库，是因为它的文档十分丰富，操作起来也比较简单。
Redis是一个开源的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的Key-Value数据库。
## 2. 导入包

```go
import (
	"github.com/gomodule/redigo/redis"
)
```
## 3.源
github地址
[https://github.com/garyburd/redigo](https://github.com/garyburd/redigo)
文档地址：
[http://godoc.org/github.com/garyburd/redigo/redis](http://godoc.org/github.com/garyburd/redigo/redis)

## 4. 连接

```go
package main

import (
       "github.com/gomodule/redigo/redis"
       "log"
       "time"
)


func main() {
	setdb := redis.DialDatabase(1)  //库
	setPasswd := redis.DialPassword("111111")//密码
        timeout := redis.DialConnectTimeout(5*time.Second)//连接超时时间
        readTimeout := redis.DialReadTimeout(5*time.Second)//读超时时间
        writeTimeout := redis.DialWriteTimeout(5*time.Second)写超时时间
      
	c1, err := redis.Dial("tcp", "192.168.0.13:31016", setdb, setPasswd, timeout, readTimeout, writeTimeout)
	if err != nil {
		log.Fatalln(err)
	}
	defer c1.Close()
         
        //设置key
	set, err := redis.String(c1.Do("SET", "my_test", "redigo"))
	if err != nil {
		log.Println("set err")
	} else {
		log.Println("set key my_test sucessful",set)
	}

	//获取value并转成字符串
	get, err := redis.String(c1.Do("GET", "my_test"))
	if err != nil {
		log.Println("err while getting:", err)
	} else {
		log.Println("get key my_test sucessful",get)
	}

}
```

```go
[root@localhost redisgo]# go run redis2.go
2020/04/14 14:36:01 set key my_test sucessful OK
2020/04/14 14:36:01 get key my_test sucessful redigo
```
## 5. 使用连接池
### 5.1 源码
redigo中的线程池是一个对象
```python
type Pool struct {
    Dial func() (Conn, error)     // 连接redis的函数
    TestOnBorrow func(c Conn, t time.Time) error  // 测试空闲线程是否正常运行的函数
    MaxIdle int         //最大的空闲连接数，可以看作是最大连接数
    MaxActive int       //最大的活跃连接数，默认为0不限制
    IdleTimeout time.Duration  //连接超时时间，默认为0表示不做超时限制
    Wait bool         //当连接超出数量先之后，是否等待到空闲连接释放
}

// 连接池中的具体连接对象
type conn struct {
    //  锁
    mu      sync.Mutex
    pending int
    err     error
    // http 包中的conn对象
    conn    net.Conn

    // 读入过期时间
    readTimeout time.Duration
    // bufio reader对象 用于读取redis服务返回的结果
    br          *bufio.Reader

    // 写入过期时间
    writeTimeout time.Duration
    // bufio writer对象 带buf 用于往服务端写命令
    bw           *bufio.Writer

    // Scratch space for formatting argument length.
    // '*' or '$', length, "\r\n"
    lenScratch [32]byte

    // Scratch space for formatting integers and floats.
    numScratch [40]byte
}
我们可以看到，其中有几个关键
```

官方提供了一个NewPool()来创建连接池

```go
func NewPool(newFn func() (Conn, error), maxIdle int) *Pool {
 return &Pool{Dial: newFn, MaxIdle: maxIdle}
}
```

### 5.2 连接方法
方法1
使用连接池还是很简单的步骤：

 1. 创建连接池
 2. 简单设置连接池的最大链接数等参数
 3. 注入拨号函数（设置redis地址 端口号等）
 4. 调用pool.Get() 获取连接
 5. 使用连接Do函数请求redis
 6. 关闭连接

```go
package main

import (
    "log"
    "github.com/garyburd/redigo/redis"
    "time"
)

func main() {
    pool := &redis.Pool{
        MaxIdle:   4,
        MaxActive: 4,
        Dial: func() (redis.Conn, error) {
            rc, err := redis.Dial("tcp", "192.168.0.13:31016", redis.DialPassword("111111"))
            if err != nil {
                return nil, err
            }
            return rc, nil
        },
        IdleTimeout: time.Second,
        Wait:        true,
    }
    con := pool.Get()
    str, err := redis.String(con.Do("get", "test"))
    if err != nil {
       log.Println(err)
     }
    defer con.Close()
    log.Println("value: ", str)
}
```

```go
[root@localhost redisgo]# go run redis5.go
2020/04/14 15:48:38 value:  a
```


方法2
```go
package main
 
import (
 "github.com/garyburd/redigo/redis"
// "fmt"
// "flag"
 "time"
 "log"
)

func newPoolFunc()(redis.Conn, error){
      var (
	  timeout time.Duration
	  readTimeout time.Duration
	  writeTimeout time.Duration
      )
      timeout = 5 * time.Second
      readTimeout = 5 * time.Second
      writeTimeout = 5 * time.Second
      return redis.Dial("tcp", "192.168.0.13:31016",
                redis.DialPassword("111111"),
                redis.DialDatabase(1),
                redis.DialConnectTimeout(timeout),
                redis.DialReadTimeout(readTimeout),
                redis.DialWriteTimeout(writeTimeout))
}
 
func newPool()(* redis.Pool){
 return &redis.Pool{
 MaxIdle: 10,
 Dial: newPoolFunc,
 Wait: true,
 }
}

func main() {
     err1 := dbExec()
     if err1 != nil {
        log.Println("db insert error:", err1)
     }  
     log.Println("redis DB insert finished.")
     
     err2 := dbQuery() 
     if err2 != nil {
        log.Println("redis db query error:", err2)
     }  
     log.Println("redis DB query finished.")

}

func dbQuery() error {
     pool := newPool()
     conn := pool.Get()
     defer conn.Close()
     _, err := conn.Do("GET", "test")
     if err == nil {
        return err
     }
     conn.Close()
     return err
}

func dbExec() error {
     pool := newPool()
     conn := pool.Get()
     defer conn.Close()
     _, err := conn.Do("SET", "test", "a")
     if err == nil {
        return err
     }
     conn.Close()
     return err
}
```

```go
[root@localhost redisgo]# go run redis4.go 
2020/04/14 14:42:32 redis DB insert finished.
2020/04/14 14:42:32 redis DB query finished.
```

 - MaxActive 最大连接数，即最多的tcp连接数，一般建议往大的配置，但不要超过操作系统文件句柄个数（centos下可以ulimit
   -n查看）。
 - MaxIdle 最大空闲连接数，但过了超时时间也会关闭。
 - IdleTimeout 空闲连接超时时间，但应该设置比redis服务器超时时间短。否则服务端超时了，客户端保持着连接也没用。
 - Wait 如果超过最大连接，是报错，还是等待。

### 5.3 连接池报错
连接池消耗殆尽

```go
redigo: connection pool exhausted
```

redigo常常会有这样的报错。我们来从redigo源码上来分析这个问题。

```go
// Handle limit for p.Wait == false.
if !p.Wait && p.MaxActive > 0 && p.active >= p.MaxActive {
    p.mu.Unlock()
    return nil, ErrPoolExhausted
}  
```

当`Wait==false`，并且当前有效连接>=最大连接数里就报这个错了。要解决这个问题的话，可以修改这个参数：

 - MaxActive
   可以把MaxActive调大（一般设置为500，1000问题都不大。）但如果redis服务器负载已经很高了（可以看redis-server
   CPU占用），去调大MaxActive就没多大意义。还是需要根据实际情况来权衡。
 - Wait 可以把Wait设置为true。wait的话必然会加大响应，如果对响应时间要求较高的话，还得从别的途径来解决。
 - 还可以加从库通过读写分离来解决。特别是PHP，没有常驻的redis连接池，建议增加多个从库。

