运行一个容器

```bash
$ sudo docker run -itd ubuntu:14.04  --name ubuntu /bin/bash  
```

 - -d, --detach=false         指定容器运行于前台还是后台，默认为false
 - -i, --interactive=false   打开STDIN，用于控制台交互
 - -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false

## 1. docker attach
```bash
$ sudo docker attach ubuntu
```
**注意**：

 - `exit`，会导致容器的停止
 - `ctrl + p`, `ctrl+q`  退出容器，容器继续运行

## 2. docker exec

```bash
$ docker exec -it ubuntu bash
$ docker exec -it ubuntu ls
$ docker exec -it ubuntu touch test.txt
```

## 3. nsenter

```bash
$ sudo docker inspect -f {{.State.Pid}} ubuntu   #获取容器pid
$ sudo nsenter --target <pid>  --mount --uts --ipc --net --pid  
$ nsenter -t <pid> -m -p -n -i -u
$ nsenter -t <pid> -m -p -n -i -u <cmd>
#脚本中 
$ cat nsenter.sh
#!/bin/bash
PID=$(docker inspect --format "{{ .State.Pid}}"  <container id>)
nsenter --target    $PID   --mount --uts   --ipc   --net   --pid ls 


nsenter -n -t 896949
```


