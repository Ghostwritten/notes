

---

## 1. 简介
 - Watches是查看指定数据信息的一种方法，比如查看nodes列表、键值对、健康检查。当监控到更新时，可以调用外部处理程序——可以自定义。比如，发现健康状态发生变化可以通知外部系统健康异常。
 - Watches在调用http api接口使用阻塞队列。Agent会自动调用合适的API接口俩监控数据的变化。
 - Watches可以作为Agent配置的一部分。在Agent初始化时就运行，并且支持重新载入配置——运行时新添加或删除配置。
 - 在任意情况下，watches的type都必须指定。Watch支持的每一个type需要的不同的参数，一些是必须的一些事非必须的。这些都是通过JSON来设置的。

## 2. docker安装consul

```bash
docker run -d --net=host --name=dev-consul consul agent -dev -client 0.0.0.0
```


### 3. Handle
Watch配置可以指定监控的数据。一旦数据发生变化，可以运行指定的处理程序——可以是任意可执行的程序。

 处理程序可以从标准输入中读取输入，也可以读取json数据。数据格式依赖于watch类型。Watch类型与Json格式是想对象的。因为watch是直接调用HTTP API，因此输入数据要格式化。

### 4. watch参数
 除了每一种类型支持的参数外，还有一些全局参数：

```bash
type - 监控的数据类型
key - 监控的键值数据的Key
handler_type - 通知类型，支持script和http
args - 配置通知类型为script的，执行命令，是一个数组，第一个元素是命令，后面第2个到第N个元素是命令的参数。
handler – 监控到数据变化后的调用程序。
```

http_handler_config参数说明：

```bash
path - 通知Url
method - http请求方法
header - 自定义Http请求头，没有可以忽略
timeout - 超时时间，10秒
tls_skip_verify - 是否跳过tls验证
```

### 5. Watches类型
不同的watch类型对应着不同HTTP API


 - Key – 监视指定K/V键值对
 - Keyprefix – Watch a prefix in the KV store
 - Services – 监视服务列表
 - nodes – 监控节点列表
 - service – 监视服务实例
 - checks- 监视健康检查的值
 - event – 监视用户事件
 
#### 5.1 key
 Key watch类型通常用来监视指定键值对的变化。要求提供key参数
##### 第一种方法：
**注意**: `watch json配置要放在和配置文件同一个目录下才会生效`
```bash
{
"watches": [ {
  "type": "key",
  "key": "foo/bar/baz",
  "handler": "/usr/bin/my-key-handler.sh hello"
}]
}
```

```bash
root@node1:~/consul# cat /root/consul/demo/my-service-handler.sh
#!/bin/sh
echo "This is a test  $1 !"
```
修改foo/bar/baz的值

```bash
consul kv putfoo/bar/baz 2
```
当watch发现`foo/bar/baz`的值发生变化，便会执行一次`/root/consul/demo/my-key-handler.sh`，

```bash
consul reload
#界面修改key：foo/bar/baz的值。，或者命令行修改 consul kv put foo/bar/baz 1
docker logs dev-consul -f --tail 200  #会看到执行/consul/demo/my-service-handler.sh输出
```

##### 第二种方法：

```bash
{

"watches": [{
		"type": "key",
		"key": "foo/bar/baz",
		"handler_type": "script",
		"args": ["/consul/demo/my-service-handler.sh", "-redis"]
	}]
}
```


##### 第三种方法
使用命令行：放在后台监控

```bash
$ consul watch -type key -key foo/bar/baz /usr/bin/my-service-handler.sh hello &
This is a test hello !
```
当然这会是命令行输出结果，而不是日志，当然你也可以自定义日志。



##### 第四种方法
和配置文件写在一起也是可以的

```bash
{
	"datacenter": "dc1", 
	"data_dir": "/consul/data", 
	"ui": true,
	"watches": [{
		"type": "key",
		"key": "foo/bar/baz",
		"handler_type": "script",
		"args": ["/usr/bin/my-service-handler.sh", "-redis"]
	}]
}
```
##### 第五种方法
通过http API接口通知

```bash
{
	"datacenter": "dc1",
	"data_dir": "/consul/data",
	"ui": true,
	"watches": [{
		"type": "key",
		"key": "foo/bar/baz",
		"handler_type": "http",
		"http_handler_config": {
			"path": "https://localhost:8000/watch",
			"method": "POST",
			"header": {
				"x-foo": ["bar", "baz"]
			},
			"timeout": "10s",
			"tls_skip_verify": false
		}
	}]
}
```
当然http API https://localhost:8000/watch是我们自定义开发的。
#### 5.2 keyprefix
第一种方法：
Keyprefix类型是用来监视KV存储中keys的前缀。必须提供prefix参数。
监视会返回匹配prefix的所有键值对。

```bash
{

"watches": [{
  "type": "keyprefix",
  "prefix": "foo/",
  "handler": "/usr/bin/my-prefix-handler.sh"
}]
}
```

```bash
$ consul watch -typekeyprefix -prefix foo/ /usr/bin/my-prefix-handler.sh
```
#### 5.3 services
监视一系列有效的service，无参数
内部接口/v1/catalog/services 

命令行输出信息如下：

```bash
{
  "consul": [],
  "redis": [],
  "web": []
}
```

#### 5.4 nodes
监视一系列有效的节点，无参数。
内部API：`/v1/catalog/nodes`

```bash
[
  {
    "Node": "nyc1-consul-1",
    "Address": "192.241.159.115"
  },
  {
    "Node": "nyc1-consul-2",
    "Address": "192.241.158.205"
  },
  {
    "Node": "nyc1-consul-3",
    "Address": "198.199.77.133"
  },
  {
    "Node": "nyc1-worker-1",
    "Address": "162.243.162.228"
  },
  {
    "Node": "nyc1-worker-2",
    "Address": "162.243.162.226"
  },
  {
    "Node": "nyc1-worker-3",
    "Address": "162.243.162.229"
  }
]
```
#### 5.5 service
监控指定的单个service。必须提供参数service。tag和passingonly参数可选。
第一种方法
内部接口：`/v1/health/service` 

```bash
{
  "type": "service",
  "service": "redis",
  "handler": "/usr/bin/my-service-handler.sh"
}
```
第二种方法
```bash
$ consul watch -typeservice -service redis /usr/bin/my-service-handler.sh
```



#### 5.6 check

```bash
{
  "watches": [
   {
        "type": "checks",
        "state": "critical",
        "args": ["/root/hello.sh", "test"],
        "token":"AN8uRFhY"
   },
   {
        "type": "checks",
        "state": "warning",
        "args": ["/root/hello2", "test2"],
        "token":"AN8uRFhY"
   }
  ]
}
```


