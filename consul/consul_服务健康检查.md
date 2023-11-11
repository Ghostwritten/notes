

-----
## 简介
服务注册 - 服务进程在注册中心注册自己的位置。它通常注册自己的主机和端口号，有时还有身份验证信息，协议，版本号，以及运行环境的详细资料。

服务发现 - 客户端应用进程向注册中心发起查询，来获取服务的位置。服务发现的一个重要作用就是提供一个可用的服务列表。

服务定义的格式类似如下：

```bash
/ # cat /consul/config/prometheus.json 
{  
  "service":{  
    "id": "promtheus",  
    "name": "prometheus",  
    "address": "192.168.1.120",  
    "port": 9090,  
    "tags": ["dev"],  
    "checks": [  
        {  
            "http": "http://192.168.1.120:9090",  
            "interval": "5s"  
        }  
    ]  
  }  
}
```
重新载入配置

```bash
consul reload
```
界面查看，服务载入成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200822113744492.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
其中，check是用来做服务的健康检查的，可以有多个，也可以没有，支持多种方式的检查。check定义在配置文件中，或运行时通过HTTP接口添加。Check是通过HTTP与节点保持一致

## check方法
有五种check方法：

**check必须是script或者TTL类型的，如果是script类型，则script和interval变量必须被提供，如果是TTL类型，则ttl变量必须被提供**

**script是consul主动去检查服务的健康状况，ttl是服务主动向consul报告自己的健康状况。**

以下是几种配置方式

Check必须是`Script、HTTP、TCP、TTL`四种类型中的一种。Script类型需要提供Script脚本和interval变量。HTTP类型必须提供http和Interval字段。TCP类型需要提供tcp和Interval字段，TTL类型秩序提供ttl。
Check的name字段是自动通过service:<service-id>生成，如果有多个service，则由service:<service-id>:<num>生成。

### Script check（Script+ Interval）

通过执行外部应用进行健康检查：这种外部程序具有退出代码，并可能产生一些输出；脚本按照指预置时间间隔来调用（比如，每30秒调用一次），类似于Nagios插件系统，脚本输出限制在4K以内，输出大于4K将截断。默认情况下，脚本超时时间为30秒——可通过timeout来配置。
示例1
```bash
{
	"service": {
		"name": "web",
		"tags": ["rails"],
		"port": 80,
		"checks": [{
			"id": "mem-util", // 检查项id
			"name": "Memory utilization", // 检查项名字
			"args": ["/usr/local/bin/check_mem.py", "-limit", "256MB"], // 这里是我们要执行的命令，第一个参数是命令或者脚本名，后面跟着任意个参数
			"interval": "10s", // 每10秒执行一次命令
			"timeout": "1s" // 命令执行超时时间
		}]
	}
}
```

示例2：

```bash
{
	"service": {
		"name": "web",
		"tags": ["rails"],
		"port": 80,
		"checks": [{
			"args": ["curl", "localhost"], // 执行curl命令，数组第2到第N个元素，代表命令参数
			"interval": "10s" // 每10秒执行一次命令
		}]
	}
}
```

为了安全考虑，如果健康检查使用执行命令方式，在启动consul的时候支持下面两种参数：

 - -enable-script-checks 允许通过配置文件和http api注册的服务，执行命令检查健康状态
 - -enable-local-script-checks 禁止通过http api注册的服务，执行命令检查健康状态，只允许通过配置文件注册的服务，执行命令。

建议生产环境使用`-enable-local-script-checks`参数启动consul agent。

例子：

```bash
consul agent -dev -enable-local-script-checks -config-dir=/etc/consul.d
```

本地测试，增加-dev参数，代表开发模式，生产环境，去掉-dev参数。

因为只有consul的client才会执行健康检查任务，可以在client设置这个参数就可以。


### 基于HTTP请求
定时以GET请求方式，请求指定url，http请求返回状态码200表示正常，其他状态代表异常。这种检查将按照预设的时间间隔创建一个HTTP “get”请求。HTTP响应代码来标示服务所处状态：任何2xx代码视为正常，429表示警告——有很多请求；其他值表示失败。

这种类型的检查应使用curl或外部程序来处理HTTP操作。默认情况下，HTTP Checks中，请求超时时间等于调用请求的间隔时间，最大10秒。有可能使用客制的HTTP check，可以自由配置timeout时间，输出限制在4K以内，输出大于4K将截断

```bash
{
	"service": {
		"name": "web",
		"tags": ["rails"],
		"port": 80,
		"checks": [{
			"id": "api", // 健康检查项的id，唯一
			"name": "HTTP API on port 5000", // 检查项的名字
			"http": "https://localhost:5000/health", // 定期访问的Url,通过这个url请求结果确定服务是否正常
			"tls_skip_verify": false, // 关闭tls验证
			"method": "POST", // 设置http请求方式，默认是GET
			"header": { // 可以自定义请求头，可以不配置
				"x-foo": ["bar", "baz"]
			},
			"interval": "10s", // 定期检查的时间间隔，这里是10秒
			"timeout": "1s" // 请求超时时间，1秒
		}]
	}
}
```
### 基于tcp请求
基于tcp请求方式，就是定时，向指定的地址，建立tcp连接，连接成功就代表服务正常，否则代表异常。
将按照预设的时间间隔与指定的IP/Hostname和端口创建一个TCP连接。服务的状态依赖于TCP连接是否成功——如果连接成功，则状态是“success”；否则状态是“critical”。如果一个Hostname解析为一个IPv4和一个IPv6，将尝试连接这两个地址，第一次连接成功则服务状态是“success”。

如果希望通过这种方式利用外部脚本执行健康检查，那么脚本应该采用“netcat”或者简单的socket操作。

默认情况下，TCP checks中，请求超时时间等于调用请求的间隔时间，最大10秒。也是可以自由配置的。

```bash
{
	"service": {
		"name": "web",
		"tags": ["rails"],
		"port": 80,
		"checks": [{
			"id": "ssh", // 检查项目id
			"name": "SSH TCP on port 22", // 检查项名字
			"tcp": "localhost:22", // tcp连接地址，ip+port
			"interval": "10s", // 定义建立连接的时间间隔是10秒
			"timeout": "1s" // 超时时间是1秒
		}]
	}
}
```
### 基于grpc请求
如果微服务是基于grpc协议，可以使用grpc协议检测服务是否正常。

```bash
{
	"service": {
		"name": "web",
		"tags": ["rails"],
		"port": 80,
		"checks": [{
			"id": "mem-util", // 检查项目id
			"name": "Service health status", // 检查项名字
			"grpc": "127.0.0.1:12345", // grpc地址，ip+port
			"grpc_use_tls": true,
			"interval": "10s" // 10秒检测一次
		}]
	}
}
```
### Docker
这种检查依赖于调用封装在docker容器内的外部程序。运行的docker通过docker Exec API来触发外部应用。
我们期望，consul Agent用户访问Docker HTTP API或UNIX套接字。Consul使用$DOCKER_HOST来确定Docker API端点。应用程序将运行，并对在容器内运行的服务执行健康检查，并返回适当的退出代码。Check按照指定的时间间隔调用。

如果在同一个host主机上有多重shell，那么同样需要配置shell参数。

输出限制在4K以内，输出大于4K将截断。

```bash
{
"check": {
    "id": "mem-util",
    "name": "Memoryutilization",
    "docker_container_id": "f972c95ebf0e",
    "shell": "/bin/bash",
    "script": "/usr/local/bin/check_mem.py",
    "interval": "10s"
  }
}
```
 
 每一种check都必须包含name，id和notes两个是可选的。如果没有提供id，那么id会被设置为name。在一个节点中，check的ID都必须是唯一的。如果名字是冲突的，那么ID就应该设置。

字段Notes主要是增强checks的可读性。Script check中，notes字段可以由脚本生成。同样，适用HTTP接口更新TTL check的外部程序一样可以设置notes字段。

参考：
[https://www.cnblogs.com/duanxz/p/9662862.html](https://www.cnblogs.com/duanxz/p/9662862.html)
[https://www.tizi365.com/archives/514.html](https://www.tizi365.com/archives/514.html)
