## consul-template介绍
consul-template是基于consul自动替换配置文件的应用。
github：[https://github.com/hashicorp/consul-template](https://github.com/hashicorp/consul-template)

## 安装
地址：[https://releases.hashicorp.com/consul-template/](https://releases.hashicorp.com/consul-template/)

```bash
$ wget https://releases.hashicorp.com/consul-template/0.25.0/consul-template_0.25.0_linux_amd64.zip
$ unzip consul-template_0.25.0_linux_amd64.zip 
$ mv consul-template /usr/local/bin/
$ consul-template -v
consul-template v0.25.0 (99efa642)
```
示例：
实验前准备：
启动一个consul集群，[详情请点击](https://blog.csdn.net/xixihahalelehehe/article/details/106036556)

```bash
$ consul agent -dev -ui -client 0.0.0.0
```

准备consul-template的配置文件tmpl.json，放在当前目录：

```bash
$ vim tmpl.json
```
```bash
consul = "127.0.0.1:8500"   //需要连接的consul

template {

source = "./print1.ctmpl"     //文件模板
destination = "./print1.py"   //需要生成的文件
command = "python ./print1.py"  //执行生成的文件

}
```
文件模板内容
```php
$ vim print1.ctmpl
```

```python
#!/usr/bin/python
#coding:utf-8
 
#bottle
iplist = [ {{range service "hello"}} "{{.Address}}",{{end}} ]
port = 8080
 
for ip in iplist:
    print ip
```
含义是：从consul拿到服务"hello"的ip，并打印出来

```powershell
$ consul-template -config ./tmpl.json -once
192.168.1.153
```

