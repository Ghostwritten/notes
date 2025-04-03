

---
✈<font color=	#FF4500 size=3>推荐阅读：</font>✈

 - [docker 命令](https://blog.csdn.net/xixihahalelehehe/article/details/123378401?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681086016780271517687%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681086016780271517687&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-123378401.nonecase&utm_term=podman&spm=1018.2226.3001.4450)
 - [podman 命令](https://blog.csdn.net/xixihahalelehehe/article/details/121611523?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681086016780271517687%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681086016780271517687&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-121611523.nonecase&utm_term=podman&spm=1018.2226.3001.4450)
 -  [crictl 命令](https://blog.csdn.net/xixihahalelehehe/article/details/116591151?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681092916780271596159%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681092916780271596159&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-13-116591151.nonecase&utm_term=%E5%91%BD%E4%BB%A4&spm=1018.2226.3001.4450)



----
## 1. 介绍
[crictl](https://github.com/kubernetes-sigs/cri-tools) 是Kubelet容器接口（CRI）的CLI和验证工具。。 你可以使用它来检查和调试 Kubernetes 节点上的容器运行时和应用程序。 crictl 和它的源代码在 `cri-tools` 代码库。

它不会负责以下内容：
- 基于CRI构建新的kubelet容器运行时；
- 由终端用户管理CRI兼容运行时的pod/container，例如crictl创建的pod可能会被kubelet自动删除，因为在kube-apiserver上不存在。

## 版本兼容

| Kubernetes Version | cri-tools Version | cri-tools branch |
| ------------------ | ----------------- | ---------------- |
| ≥ 1.27.x           | ≥ 1.27.x          | master           |
| ≥ 1.16.x ≤ 1.26.x  | ≥ 1.16.x ≤ 1.26.x | master           |
| 1.15.X             | v1.15.0           | release-1.15     |
| 1.14.X             | v1.14.0           | release-1.14     |
| 1.13.X             | v1.13.0           | release-1.13     |
| 1.12.X             | v1.12.0           | release-1.12     |
| 1.11.X             | v1.11.1           | release-1.11     |
| 1.10.X             | v1.0.0-beta.2     | release-1.10     |
| 1.9.X              | v1.0.0-alpha.1    | release-1.9      |
| 1.8.X              | v0.2              | release-1.8      |
| 1.7.X              | v0.1              | release-1.7      |
## 2. 安装 crictl
下载指定版本：[https://github.com/kubernetes-sigs/cri-tools/releases](https://github.com/kubernetes-sigs/cri-tools/releases)

```bash
VERSION="v1.28.0"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

## 3. 配置
`crictl` 命令有几个子命令和运行时参数。 有关详细信息，请使用 crictl help 或 crictl <subcommand> help 获取帮助信息。

crictl 默认连接到 `unix:///var/run/dockershim.sock`。 对于其他的运行时，你可以用多种不同的方法设置端点：

- 通过设置参数 --runtime-endpoint 和 --image-endpoint
- 通过设置环境变量 CONTAINER_RUNTIME_ENDPOINT 和 IMAGE_SERVICE_ENDPOINT
- 通过在配置文件中设置端点 --config=/etc/crictl.yaml
- 你还可以在连接到服务器并启用或禁用调试时指定超时值，方法是在配置文件中指定 timeout 或 debug 值，或者使用 --timeout 和 --debug 命令行参数。

要查看或编辑当前配置，请查看或编辑 /etc/crictl.yaml 的内容。

```bash
cat /etc/crictl.yaml
runtime-endpoint: unix:///var/run/dockershim.sock
image-endpoint: unix:///var/run/dockershim.sock
timeout: 10
debug: true
```
## 4. 命令

语法

```bash
crictl [global options] command [command options] [arguments...]
```

COMMANDS:

- `attach`:             Attach to a running container
- `create`:             Create a new container
- `exec`:               Run a command in a running container
- `version`:            Display runtime version information
- `images, image, img`: List images
- `inspect`:            Display the status of one or more containers
- `inspecti`:           Return the status of one or more images
- `imagefsinfo`:        Return image filesystem info
- `inspectp`:           Display the status of one or more pods
- `logs`:               Fetch the logs of a container
- `port-forward`:       Forward local port to a pod
- `ps`:                 List containers
- `pull`:               Pull an image from a registry
- `run`:                Run a new container inside a sandbox
- `runp`:               Run a new pod
- `rm`:                 Remove one or more containers
- `rmi`:                Remove one or more images
- `rmp`:                Remove one or more pods
- `pods`:               List pods
- `start`:              Start one or more created containers
- `info`:               Display information of the container runtime
- `stop`:               Stop one or more running containers
- `stopp`:              Stop one or more running pods
- `update`:             Update one or more running containers
- `config`:             Get and set `crictl` client configuration options
- `stats`:              List container(s) resource usage statistics
- `statsp`:             List pod(s) resource usage statistics
- `completion`:         Output bash shell completion code
- `checkpoint`:         Checkpoint one or more running containers
- `events, event`:      Stream the events of containers
- `help, h`:            Shows a list of commands or help for one command

`crictl` by default connects on Unix to:

- `unix:///var/run/dockershim.sock` or
- `unix:///run/containerd/containerd.sock` or
- `unix:///run/crio/crio.sock` or
- `unix:///var/run/cri-dockerd.sock`

or on Windows to:

- `npipe:////./pipe/dockershim` or
- `npipe:////./pipe/containerd-containerd` or
- `npipe:////./pipe/cri-dockerd`

For other runtimes, use:

- [frakti](https://github.com/kubernetes/frakti): `unix:///var/run/frakti.sock`

The endpoint can be set in three ways:

- By setting global option flags `--runtime-endpoint` (`-r`) and `--image-endpoint` (`-i`)
- By setting environment variables `CONTAINER_RUNTIME_ENDPOINT` and `IMAGE_SERVICE_ENDPOINT`
- By setting the endpoint in the config file `--config=/etc/crictl.yaml`

If the endpoint is not set then it works as follows:

- If the runtime endpoint is not set, `crictl` will by default try to connect using:
  - dockershim
  - containerd
  - cri-o
  - cri-dockerd
- If the image endpoint is not set, `crictl` will by default use the runtime endpoint setting

> Note: The default endpoints are now deprecated and the runtime endpoint should always be set instead.
The performance maybe affected as each default connection attempt takes n-seconds to complete before timing out and going to the next in sequence.

Unix:

```sh
$ cat /etc/crictl.yaml
runtime-endpoint: unix:///var/run/dockershim.sock
image-endpoint: unix:///var/run/dockershim.sock
timeout: 2
debug: true
pull-image-on-create: false
```

Windows:

```cmd
C:\> type %USERPROFILE%\.crictl\crictl.yaml
runtime-endpoint: tcp://localhost:3735
image-endpoint: tcp://localhost:3735
timeout: 2
debug: true
pull-image-on-create: false
```

linux：containerd

```bash
cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10
debug: true
EOF
```

## Additional options

- `--timeout`, `-t`: Timeout of connecting to server in seconds (default: `2s`).
0 or less is interpreted as unset and converted to the default. There is no
option for no timeout value set and the smallest supported timeout is `1s`
- `--debug`, `-D`: Enable debug output
- `--help`, `-h`: show help
- `--version`, `-v`: print the version information of `crictl`
- `--config`, `-c`: Location of the client config file (default: `/etc/crictl.yaml`). Can be changed by setting `CRI_CONFIG_FILE` environment variable. If not specified and the default does not exist, the program's directory is searched as well

## Client Configuration Options

Use the `crictl` config command to get and set the `crictl` client configuration
options.

USAGE:

```sh
crictl config [command options] [<crictl options>]
```

For example `crictl config --set debug=true` will enable debug mode when giving subsequent `crictl` commands.

COMMAND OPTIONS:

- `--get value`: Show the option value
- `--set value`: Set option (can specify multiple or separate values with commas: opt1=val1,opt2=val2)
- `--help`, `-h`: Show help (default: `false`)

`crictl` OPTIONS:

- `runtime-endpoint`: Container runtime endpoint (no default value)
- `image-endpoint`: Image endpoint (no default value)
- `timeout`: Timeout of connecting to server (default: `2s`)
- `debug`: Enable debug output (default: `false`)
- `pull-image-on-create`: Enable pulling image on create requests (default: `false`)
- `disable-pull-on-run`: Disable pulling image on run requests (default: `false`)

> When enabled `pull-image-on-create` modifies the create container command to first pull the container's image.
This feature is used as a helper to make creating containers easier and faster.
Some users of `crictl` may desire to not pull the image necessary to create the container.
For example, the image may have already been pulled or otherwise loaded into the container runtime, or the user may be running without a network. For this reason the default for `pull-image-on-create` is `false`.

> By default the run command first pulls the container image, and `disable-pull-on-run` is `false`.
Some users of `crictl` may desire to set `disable-pull-on-run` to `true` to not pull the image by default when using the run command.

> To override these default pull configuration settings, `--no-pull` and `--with-pull` options are provided for the create and run commands.

## Examples

### Run pod sandbox with config file

```sh
$ cat pod-config.json
{
    "metadata": {
        "name": "nginx-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "log_directory": "/tmp",
    "linux": {
    }
}

$ crictl runp pod-config.json
f84dd361f8dc51518ed291fbadd6db537b0496536c1d2d6c05ff943ce8c9a54f
```

List pod sandboxes and check the sandbox is in Ready state:

```sh
$ crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT
f84dd361f8dc5       17 seconds ago      Ready               nginx-sandbox       default             1
```

### Run pod sandbox with runtime handler

Runtime handler requires runtime support. The following example shows running a pod sandbox with `runsc` handler on containerd runtime.

```sh
$ cat pod-config.json
{
    "metadata": {
        "name": "nginx-runsc-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "log_directory": "/tmp",
    "linux": {
    }
}

$ crictl runp --runtime=runsc pod-config.json
c112976cb6caa43a967293e2c62a2e0d9d8191d5109afef230f403411147548c

$ crictl inspectp c112976cb6caa43a967293e2c62a2e0d9d8191d5109afef230f403411147548c
...
    "runtime": {
      "runtimeType": "io.containerd.runtime.v1.linux",
      "runtimeEngine": "/usr/local/sbin/runsc",
      "runtimeRoot": "/run/containerd/runsc"
    },
...
```

### Pull a busybox image

```sh
$ crictl pull busybox
Image is up to date for busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
```

List images and check the busybox image has been pulled:

```sh
$ crictl images
IMAGE               TAG                 IMAGE ID            SIZE
busybox             latest              8c811b4aec35f       1.15MB
k8s.gcr.io/pause    3.1                 da86e6ba6ca19       742kB
```

### Create container in the pod sandbox with config file

```sh
$ cat pod-config.json
{
    "metadata": {
        "name": "nginx-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "log_directory": "/tmp",
    "linux": {
    }
}

$ cat container-config.json
{
  "metadata": {
      "name": "busybox"
  },
  "image":{
      "image": "busybox"
  },
  "command": [
      "top"
  ],
  "log_path":"busybox.0.log",
  "linux": {
  }
}

$ crictl create f84dd361f8dc51518ed291fbadd6db537b0496536c1d2d6c05ff943ce8c9a54f container-config.json pod-config.json
3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60
```

List containers and check the container is in Created state:

```sh
$ crictl ps -a
CONTAINER ID        IMAGE               CREATED             STATE               NAME                ATTEMPT
3e025dd50a72d       busybox             32 seconds ago      Created             busybox             0
```

### Start container

```sh
$ crictl start 3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60
3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60

$ crictl ps
CONTAINER ID        IMAGE               CREATED              STATE               NAME                ATTEMPT
3e025dd50a72d       busybox             About a minute ago   Running             busybox             0
```

### Exec a command in container

```sh
crictl exec -i -t 3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60 ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
```

### Create and start a container within one command

It is possible to start a container within a single command, whereas the image
will be pulled automatically, too:

```sh
$ cat pod-config.json
{
    "metadata": {
        "name": "nginx-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "log_directory": "/tmp",
    "linux": {
    }
}

$ cat container-config.json
{
  "metadata": {
      "name": "busybox"
  },
  "image":{
      "image": "busybox"
  },
  "command": [
      "top"
  ],
  "log_path":"busybox.0.log",
  "linux": {
  }
}

$ crictl run container-config.json pod-config.json
b25b4f26e342969eb40d05e98130eee0846557d667e93deac992471a3b8f1cf4
```

List containers and check the container is in Running state:

```sh
$ crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID
b25b4f26e3429       busybox:latest      14 seconds ago      Running             busybox             0                   158d7a6665ff3
```

### Checkpoint a running container

```sh
$ crictl checkpoint --export=/path/to/checkpoint.tar 39fcdd7a4f1d4
39fcdd7a4f1d4
$ ls /path/to/checkpoint.tar
/path/to/checkpoint.tar
```

## More information

* See the [Kubernetes.io Debugging Kubernetes nodes with crictl doc](https://kubernetes.io/docs/tasks/debug-application-cluster/crictl/)
* Visit [kubernetes-sigs/cri-tools](https://github.com/kubernetes-sigs/cri-tools) for more information.



如果使用 crictl 在正在运行的 Kubernetes 集群上创建 Pod 沙盒或容器， kubelet 最终将删除它们。 crictl 不是一个通用的工作流工具，而是一个对调试有用的工具。

```bash
$ crictl pods
POD ID              CREATED              STATE               NAME                         NAMESPACE           ATTEMPT
926f1b5a1d33a       About a minute ago   Ready               sh-84d7dcf559-4r2gq          default             0
4dccb216c4adb       About a minute ago   Ready               nginx-65899c769f-wv2gp       default             0
a86316e96fa89       17 hours ago         Ready               kube-proxy-gblk4             kube-system         0
919630b8f81f1       17 hours ago         Ready               nvidia-device-plugin-zgbbv   kube-system  

#打印某个固定pod
$ crictl pods --name nginx-65899c769f-wv2gp
POD ID              CREATED             STATE               NAME                     NAMESPACE           ATTEMPT
4dccb216c4adb       2 minutes ago       Ready               nginx-65899c769f-wv2gp   default             0


#根据标签筛选pod
$ crictl pods --label run=nginx
POD ID              CREATED             STATE               NAME                     NAMESPACE           ATTEMPT
4dccb216c4adb       2 minutes ago       Ready               nginx-65899c769f-wv2gp   default             0



#打印镜像
$ crictl images
IMAGE                                     TAG                 IMAGE ID            SIZE
busybox                                   latest              8c811b4aec35f       1.15MB
k8s-gcrio.azureedge.net/hyperkube-amd64   v1.10.3             e179bbfe5d238       665MB
k8s-gcrio.azureedge.net/pause-amd64       3.1                 da86e6ba6ca19       742kB
nginx                                     latest              cd5239a0906a6       109MB



#打印某个镜像
$ crictl images nginx
IMAGE               TAG                 IMAGE ID            SIZE
nginx               latest              cd5239a0906a6       109MB



#只打印镜像 ID：
$ crictl images -q
sha256:8c811b4aec35f259572d0f79207bc0678df4c736eeec50bc9fec37ed936a472a
sha256:e179bbfe5d238de6069f3b03fccbecc3fb4f2019af741bfff1233c4d7b2970c5
sha256:da86e6ba6ca197bf6bc5e9d900febd906b133eaa4750e6bed647b0fbe50ed43e
sha256:cd5239a0906a6ccf0562354852fae04bc5b52d72a2aff9a871ddb6bd57553569


#打印容器清单
$ crictl ps -a
CONTAINER ID        IMAGE                                                                                                             CREATED             STATE               NAME                       ATTEMPT
1f73f2d81bf98       busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47                                   7 minutes ago       Running             sh                         1
9c5951df22c78       busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47                                   8 minutes ago       Exited              sh                         0
87d3992f84f74       nginx@sha256:d0a8828cccb73397acb0073bf34f4d7d8aa315263f1e7806bf8c55d8ac139d5f                                     8 minutes ago       Running             nginx                      0
1941fb4da154f       k8s-gcrio.azureedge.net/hyperkube-amd64@sha256:00d814b1f7763f4ab5be80c58e98140dfc69df107f2



#打印正在运行的容器清单：
$ crictl ps
CONTAINER ID        IMAGE                                                                                                             CREATED             STATE               NAME                       ATTEMPT
1f73f2d81bf98       busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47                                   6 minutes ago       Running             sh                         1
87d3992f84f74       nginx@sha256:d0a8828cccb73397acb0073bf34f4d7d8aa315263f1e7806bf8c55d8ac139d5f                                     7 minutes ago       Running             nginx                      0
1941fb4da154f       k8s-gcrio.azureedge.net/hyperkube-amd64@sha256:00d814b1f7763f4ab5be80c58e98140dfc69df107f2


#容器上执行命令
$ crictl exec -i -t 1f73f2d81bf98 ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var



#获取容器的所有日志：
$ crictl logs 87d3992f84f74
10.240.0.96 - - [06/Jun/2018:02:45:49 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"
10.240.0.96 - - [06/Jun/2018:02:45:50 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"
10.240.0.96 - - [06/Jun/2018:02:45:51 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"



#获取最近的 N 行日志：
crictl logs --tail=1 87d3992f84f74
10.240.0.96 - - [06/Jun/2018:02:45:51 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"






```
#运行 Pod 沙盒
用 crictl 运行 Pod 沙盒对容器运行时排错很有帮助。 在运行的 Kubernetes 集群中，沙盒会随机地被 kubelet 停止和删除。

编写下面的 JSON 文件：

```bash
{
    "metadata": {
        "name": "nginx-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "logDirectory": "/tmp",
    "linux": {
    }
}
```

使用 crictl runp 命令应用 JSON 文件并运行沙盒。

```bash
$ crictl runp pod-config.json
```
创建容器
用 crictl 创建容器对容器运行时排错很有帮助。 在运行的 Kubernetes 集群中，沙盒会随机的被 kubelet 停止和删除。

拉取 busybox 镜像

```bash
crictl pull busybox
Image is up to date for busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
```

创建 Pod 和容器的配置：

Pod 配置：

```bash
{
    "metadata": {
        "name": "nginx-sandbox",
        "namespace": "default",
        "attempt": 1,
        "uid": "hdishd83djaidwnduwk28bcsb"
    },
    "log_directory": "/tmp",
    "linux": {
    }
}
```

容器配置：

```bash
{
  "metadata": {
      "name": "busybox"
  },
  "image":{
      "image": "busybox"
  },
  "command": [
      "top"
  ],
  "log_path":"busybox.log",
  "linux": {
  }
}
```

创建容器，传递先前创建的 Pod 的 ID、容器配置文件和 Pod 配置文件。返回容器的 ID。

```bash
crictl create f84dd361f8dc51518ed291fbadd6db537b0496536c1d2d6c05ff943ce8c9a54f container-config.json pod-config.json
```

查询所有容器并确认新创建的容器状态为 Created。

```bash
crictl ps -a
CONTAINER ID        IMAGE               CREATED             STATE               NAME                ATTEMPT
3e025dd50a72d       busybox             32 seconds ago      Created             busybox             0
```

启动容器
要启动容器，要将容器 ID 传给 crictl start：

```bash
crictl start 3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60
3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60
```

确认容器的状态为 Running。

```bash
crictl ps
CONTAINER ID        IMAGE               CREATED              STATE               NAME                ATTEMPT
3e025dd50a72d       busybox             About a minute ago   Running             busybox             0
```


##  新增

```bash
$ sudo crictl pull --creds admin:8cDcos@11 registry.paas/cmss/cni:v3.23.2
Image is up to date for sha256:a87d3f6f1b8fdc077e74ad6874301f57490f5b38cc731d5d6f5803f36837b4b1
```

