

---
## 1. hosts配置格式
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210205111355652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

> 注意： 您可以使用逗号（,）或冒号（:）分隔主机列表。处理范围和IPv6地址时，首选逗号。


```bash
#定位到“ webservers”和“ dbservers”组中也属于“ staging”组的所有计算机，但“ phoenix”组中的所有计算机除外。
webservers:dbservers:&staging:!phoenix
```
您可以将通配符模式与FQDN或IP地址一起使用，只要主机在清单中按FQDN或IP地址命名即可：

```bash
192.0.\*
\*.example.com
\*.com
```
您可以同时混合使用通配符模式和组：

```bash
one*.com:dbservers
```
**第一种分组匹配：**

```bash
[webservers]
cobweb
webbing
weber
```
您可以使用下标在Webservers组中选择单个主机或范围：

```bash
webservers[0]       # == cobweb
webservers[-1]      # == weber
webservers[0:2]     # == webservers[0],webservers[1]
                    # == cobweb,webbing
webservers[1:]      # == webbing,weber
webservers[:3]      # == cobweb,webbing,weber
```

**第二种分组匹配**

```bash
atlanta:
  host1:
    http_port: 80
    maxRequestsPerChild: 808
    host: 127.0.0.2
```

```bash
all:
  hosts:
    node0001:
      ansible_host: 10.250.47.135
      ip: 10.250.47.135
      access_ip: 10.250.47.135
    node0002:
      ansible_host: 10.250.47.136
      ip: 10.250.47.136
      access_ip: 10.250.47.136
    node0003:
      ansible_host: 10.250.47.137
      ip: 10.250.47.137
      access_ip: 10.250.47.137
  children:
    kube-master:
      hosts:
        node0001:
        node0002:
        node0003:
    kube-node:
      hosts:
        node0001:
        node0002:
        node0003:
    etcd:
      hosts:
        node0001:
        node0002:
        node0003:
    k8s-cluster:
      children:
        kube-master:
        kube-node:
    calico-rr:
      hosts: {}

```

在模式中使用变量
您可以使用变量通过`-eansible-playbook`的参数来启用传递组说明符：

```bash
webservers:!{{ excluded }}:&{{ required }}
```

## 2. 命令使用格式
```bash
ansible <pattern> -m <module_name> -a "<module options>"
```
例如：

```bash
ansible webservers -m service -a "name=httpd state=restarted"
```

## 3. yaml文件格式

```bash
- name: <play_name>
  hosts: <pattern>
```
例如：

```bash
---
- name: update webservers
  hosts: webservers
  remote_user: admin

  tasks:
  - name: thing to do first in this playbook
  . . .
```

```bash
ansible-playbook site.yml --limit datacenter2
```

最后，您可以--limit通过在文件名前添加前缀来从文件中读取主机列表@：

```bash
ansible-playbook site.yml --limit @retry_hosts.txt
```


如果`RETRY_FILES_ENABLED`设置为`True`，`.retry`则`ansible-playbook`运行后将创建一个文件，其中包含所有播放的失败主机列表。每次ansible-playook完成运行都会覆盖此文件。

```bash
ansible-playbook site.yml –limit @ site.retry
```

## hosts tempalte

- vim /home/deployer/apps/hosts
```bash
[all:children]
k8s
es

[k8s]
192.168.20.1
192.168.20.2
192.168.20.3
192.168.20.4
192.168.20.5
192.168.20.6
192.168.20.7
192.168.20.8
192.168.20.9
192.168.20.10

[es]
192.168.20.6
192.168.20.7
192.168.20.8

[all:vars]
ansible_connection=ssh
ansible_user=deployer
ansible_ssh_pass=12345678
```

配置别名

```bash
$ vim /home/deployer/.bashrc
alias asb='ansible -i  /home/deployer/apps/hosts'

$ source /home/deployer/.bashrc
```

