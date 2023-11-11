#  falco default macros
tags: falco,安全
<!-- catalog:falco default macros:-->



上篇我们学习[falco的规则](https://ghostwritten.blog.csdn.net/article/details/126101266)运用，其中宏（macro）是指可重用方式定义规则的公共子部分的方法。
 Falco 规则集定义了许多宏，可以更轻松地开始编写规则。这些宏为许多常见场景提供了快捷方式，并且可以在任何用户定义的规则集中使用。Falco 还提供了应该由用户覆盖的宏，以提供特定于用户环境的设置。提供的宏也可以附加到本地规则文件中。


###  为写入而打开的文件

```bash
- macro: open_write
  condition: (evt.type=open or evt.type=openat) and evt.is_open_write=true and fd.typechar='f' and fd.num>=0
```
###  打开文件以供阅读

```bash
- macro: open_read
  condition: (evt.type=open or evt.type=openat) and evt.is_open_read=true and fd.typechar='f' and fd.num>=0
```
###  从不真实 

```bash
- macro: never_true
  condition: (evt.num=0)
```
###  永远真实

```bash
- macro: always_true
  condition: (evt.num=>0)
```
###  进程名称已设置

```bash
- macro: proc_name_exists
  condition: (proc.name!="<NA>")
```

###  文件系统对象重命名

```bash
- macro: proc_name_exists
  condition: (proc.name!="<NA>")
```

###  已创建新目录

```bash
- macro: mkdir
  condition: evt.type = mkdir
```

### 文件系统对象已删除

```bash
- macro: remove
  condition: evt.type in (rmdir, unlink, unlinkat)
```

### 文件系统对象已修改 

```bash
- macro: modify
  condition: rename or remove
```

### 新进程产生

```bash
- macro: spawned_process
  condition: evt.type = execve and evt.dir=<
```

### 二进制文件的公共目录 

```bash
- macro: bin_dir
  condition: fd.directory in (/bin, /sbin, /usr/bin, /usr/sbin)
```

### Shell 已启动

```bash
- macro: shell_procs
  condition: (proc.name in (shell_binaries))
```

### 已知敏感文件

```bash
- macro: sensitive_files
  condition: >
    fd.name startswith /etc and
    (fd.name in (sensitive_file_names)
     or fd.directory in (/etc/sudoers.d, /etc/pam.d))
```

### 新创建的进程

```bash
- macro: proc_is_new
  condition: proc.duration <= 5000000000
Inbound Network Connections
- macro: inbound
  condition: >
    (((evt.type in (accept,listen) and evt.dir=<)) or
     (fd.typechar = 4 or fd.typechar = 6) and
     (fd.ip != "0.0.0.0" and fd.net != "127.0.0.0/8") and (evt.rawres >= 0 or evt.res = EINPROGRESS))
```

### 出站网络连接

```bash
- macro: outbound
  condition: >
    (((evt.type = connect and evt.dir=<)) or
     (fd.typechar = 4 or fd.typechar = 6) and
     (fd.ip != "0.0.0.0" and fd.net != "127.0.0.0/8") and (evt.rawres >= 0 or evt.res = EINPROGRESS))
```

### 入站网络连接

```bash
- macro: inbound_outbound
  condition: >
    (((evt.type in (accept,listen,connect) and evt.dir=<)) or
     (fd.typechar = 4 or fd.typechar = 6) and
     (fd.ip != "0.0.0.0" and fd.net != "127.0.0.0/8") and (evt.rawres >= 0 or evt.res = EINPROGRESS))
```

### 对象是一个容器

```bash
- macro: container
  condition: container.id != host

```

### 交互过程产生

```bash
- macro: interactive
  condition: >
    ((proc.aname=sshd and proc.name != sshd) or
    proc.name=systemd-logind or proc.name=login)
```


### 通用 SSH 端口
覆盖此宏以反映环境中提供 SSH 服务的端口。

```bash
- macro: ssh_port
  condition: fd.sport=22
```

### 允许的 SSH 主机
覆盖此宏以反映可以连接到已知 SSH 端口（即堡垒或跳转框）的主机。

```bash
- macro: allowed_ssh_hosts
  condition: ssh_port
```

### 用户列入白名单的容器
允许在特权模式下运行的白名单容器。

```bash
- macro: user_trusted_containers
  condition: (container.image startswith sysdig/agent)
```

### 允许生成shell的容器
将允许生成 shell 的容器列入白名单，如果在 CI/CD 管道中使用容器，则可能需要这样做。

```bash
- macro: user_shell_container_exclusions
  condition: (never_true)
```

### 允许与 EC2 元数据服务通信的容器
将允许与 EC2 元数据服务通信的容器列入白名单。默认值：任何容器。

```bash
- macro: ec2_metadata_containers
  condition: container
```

### Kubernetes API 服务器
在此处设置 Kubernetes API 服务的 IP。

```bash
- macro: k8s_api_server
  condition: (fd.sip="1.2.3.4" and fd.sport=8080)
```

### 允许与 Kubernetes API 通信的容器
将允许与 Kubernetes API 服务通信的容器列入白名单。需要设置 k8s_api_server。

```bash
- macro: k8s_containers
  condition: >
    (container.image startswith gcr.io/google_containers/hyperkube-amd64 or
    container.image startswith gcr.io/google_containers/kube2sky or
    container.image startswith sysdig/agent or
    container.image startswith sysdig/falco or
    container.image startswith sysdig/sysdig)
```

### 允许与 Kubernetes 服务节点端口通信的容器

```bash
- macro: nodeport_containers
  condition: container
```

参考：

 - [faloc Default Macros](https://falco.org/docs/rules/default-macros/)

