


![](https://img-blog.csdnimg.cn/direct/413ccd06adcd4392b39b68081382f08c.png)


## containerd

### 简介
[containerd](https://containerd.io/) 是一个高级容器运行时，又名容器管理器。简单来说，它是一个守护进程，在单个主机上管理完整的容器生命周期：创建、启动、停止容器、拉取和存储镜像、配置挂载、网络等。Containerd被设计成可以很容易地嵌入到更大的系统中。Docker使用containerd来运行容器。Kubernetes可以通过CRI使用containerd来管理单个节点上的容器。

![](https://i-blog.csdnimg.cn/blog_migrate/2933c26261fd320f68b5a598106f24f8.png)

## nerdctl
### 简介
[Nerdctl](https://github.com/containerd/nerdctl) 是一个相对较新的containerd命令行客户端。与ctr不同，nerdctl的目标是用户友好和docker兼容。在某种程度上，nerdctl + containerd可以无缝地替代docker + dockerd。nerdctl的目标是促进试验Docker中没有的最前沿的容器特性。这些特性包括但不限于lazy-pulling(stargz)和镜像加密(ocicrypt)。
从基本用法的角度来看，与ctr相比，nerdctl支持:

- 与Docker相同的UI/UX，熟悉docker(或podman) CLI，你就已经熟悉nerdctl；
- 使用nerdctl构建镜像
- 容器网络管理
- 支持Docker Compose（nerdctl compose up）
- 支持 [rootless mode, without slirp overhead (bypass4netns)](https://github.com/containerd/nerdctl/blob/main/docs/rootless.md)


###  安装
下载地址：[containerd/nerdctl](https://github.com/containerd/nerdctl/releases),中下载最新的可执行文件，每一个版本都有两种可用的发行版

- 精简版 (nerdctl-<version>-linux-amd64.tar.gz): 只包含nerdctl
- 完整版 (nerdctl-full-<version>-linux-amd64.tar.gz): 包含 containerd, runc, and CNI等依赖

如果你已经安装了 Containerd，只需要选择前一个发行版，否则就选择完整版。

#### 精简 Minimal 安装
将归档文件解压到 /usr/local/bin

```bash
$ sudo tar Cxzvvf /usr/local/bin nerdctl-1.7.1-linux-amd64.tar.gz
-rwxr-xr-x root/root  24805376 2023-11-30 16:13 nerdctl
-rwxr-xr-x root/root     21618 2023-11-30 16:13 containerd-rootless-setuptool.sh
-rwxr-xr-x root/root      7187 2023-11-30 16:13 containerd-rootless.sh
```

#### 完整Full 安装

```bash
$ sudo tar Cxzvvf /usr/local nerdctl-full-1.7.1-linux-amd64.tar.gz
drwxr-xr-x 0/0               0 2023-11-30 16:20 bin/
-rwxr-xr-x 0/0        27639916 2015-10-21 08:00 bin/buildctl
-rwxr-xr-x 0/0        23724032 2022-09-05 17:52 bin/buildg
-rwxr-xr-x 0/0        53383522 2015-10-21 08:00 bin/buildkitd
-rwxr-xr-x 0/0         3784248 2023-11-30 16:18 bin/bypass4netns
-rwxr-xr-x 0/0         5279744 2023-11-30 16:18 bin/bypass4netnsd
-rwxr-xr-x 0/0        38819192 2023-11-30 16:19 bin/containerd
-rwxr-xr-x 0/0         9474048 2023-11-03 01:34 bin/containerd-fuse-overlayfs-grpc
-rwxr-xr-x 0/0           21618 2023-11-30 16:18 bin/containerd-rootless-setuptool.sh
-rwxr-xr-x 0/0            7187 2023-11-30 16:18 bin/containerd-rootless.sh
-rwxr-xr-x 0/0        12066816 2023-11-30 16:20 bin/containerd-shim-runc-v2
-rwxr-xr-x 0/0        45903872 2023-10-31 16:57 bin/containerd-stargz-grpc
-rwxr-xr-x 0/0        20494228 2023-11-30 16:20 bin/ctd-decoder
-rwxr-xr-x 0/0        18726912 2023-11-30 16:19 bin/ctr
-rwxr-xr-x 0/0        29444894 2023-11-30 16:20 bin/ctr-enc
-rwxr-xr-x 0/0        19931136 2023-10-31 16:58 bin/ctr-remote
-rwxr-xr-x 0/0         1785448 2023-11-30 16:20 bin/fuse-overlayfs
-rwxr-xr-x 0/0        68890491 2023-11-30 16:19 bin/ipfs
-rwxr-xr-x 0/0        24776704 2023-11-30 16:18 bin/nerdctl
-rwxr-xr-x 0/0        10113536 2023-05-30 14:31 bin/rootlessctl
-rwxr-xr-x 0/0        11600435 2023-05-30 14:31 bin/rootlesskit
-rwxr-xr-x 0/0        15028088 2023-11-30 16:18 bin/runc
-rwxr-xr-x 0/0         2346328 2023-11-30 16:20 bin/slirp4netns
-rwxr-xr-x 0/0          870496 2023-11-30 16:20 bin/tini
drwxr-xr-x 0/0               0 2023-11-30 16:20 lib/
drwxr-xr-x 0/0               0 2023-11-30 16:20 lib/systemd/
drwxr-xr-x 0/0               0 2023-11-30 16:20 lib/systemd/system/
-rw-r--r-- 0/0            1475 2023-11-30 16:20 lib/systemd/system/buildkit.service
-rw-r--r-- 0/0            1414 2023-11-30 16:17 lib/systemd/system/containerd.service
-rw-r--r-- 0/0             312 2023-11-30 16:20 lib/systemd/system/stargz-snapshotter.service
drwxr-xr-x 0/0               0 2023-11-30 16:20 libexec/
drwxrwxr-x 0/0               0 2023-11-30 16:20 libexec/cni/
-rwxr-xr-x 0/0         4016001 2023-05-10 03:53 libexec/cni/bandwidth
-rwxr-xr-x 0/0         4531309 2023-05-10 03:53 libexec/cni/bridge
-rwxr-xr-x 0/0        10816051 2023-05-10 03:53 libexec/cni/dhcp
-rwxr-xr-x 0/0         4171248 2023-05-10 03:53 libexec/cni/dummy
-rwxr-xr-x 0/0         4649749 2023-05-10 03:53 libexec/cni/firewall
-rwxr-xr-x 0/0         4059321 2023-05-10 03:53 libexec/cni/host-device
-rwxr-xr-x 0/0         3444776 2023-05-10 03:53 libexec/cni/host-local
-rwxr-xr-x 0/0         4193323 2023-05-10 03:53 libexec/cni/ipvlan
-rwxr-xr-x 0/0         3514598 2023-05-10 03:53 libexec/cni/loopback
-rwxr-xr-x 0/0         4227193 2023-05-10 03:53 libexec/cni/macvlan
-rwxr-xr-x 0/0         3955775 2023-05-10 03:53 libexec/cni/portmap
-rwxr-xr-x 0/0         4348835 2023-05-10 03:53 libexec/cni/ptp
-rwxr-xr-x 0/0         3716095 2023-05-10 03:53 libexec/cni/sbr
-rwxr-xr-x 0/0         2984504 2023-05-10 03:53 libexec/cni/static
-rwxr-xr-x 0/0         4258344 2023-05-10 03:53 libexec/cni/tap
-rwxr-xr-x 0/0         3603365 2023-05-10 03:53 libexec/cni/tuning
-rwxr-xr-x 0/0         4187498 2023-05-10 03:53 libexec/cni/vlan
-rwxr-xr-x 0/0         3754911 2023-05-10 03:53 libexec/cni/vrf
drwxr-xr-x 0/0               0 2023-11-30 16:18 share/
drwxr-xr-x 0/0               0 2023-11-30 16:18 share/doc/
drwxr-xr-x 0/0               0 2023-11-30 16:18 share/doc/nerdctl/
-rw-r--r-- 0/0           12480 2023-11-30 16:13 share/doc/nerdctl/README.md
drwxr-xr-x 0/0               0 2023-11-30 16:13 share/doc/nerdctl/docs/
-rw-r--r-- 0/0            3953 2023-11-30 16:13 share/doc/nerdctl/docs/build.md
-rw-r--r-- 0/0            2570 2023-11-30 16:13 share/doc/nerdctl/docs/builder-debug.md
-rw-r--r-- 0/0            3996 2023-11-30 16:13 share/doc/nerdctl/docs/cni.md
-rw-r--r-- 0/0           74114 2023-11-30 16:13 share/doc/nerdctl/docs/command-reference.md
-rw-r--r-- 0/0            1846 2023-11-30 16:13 share/doc/nerdctl/docs/compose.md
-rw-r--r-- 0/0            5329 2023-11-30 16:13 share/doc/nerdctl/docs/config.md
-rw-r--r-- 0/0            9128 2023-11-30 16:13 share/doc/nerdctl/docs/cosign.md
-rw-r--r-- 0/0            5660 2023-11-30 16:13 share/doc/nerdctl/docs/cvmfs.md
-rw-r--r-- 0/0            2435 2023-11-30 16:13 share/doc/nerdctl/docs/dir.md
-rw-r--r-- 0/0             906 2023-11-30 16:13 share/doc/nerdctl/docs/experimental.md
-rw-r--r-- 0/0           14217 2023-11-30 16:13 share/doc/nerdctl/docs/faq.md
-rw-r--r-- 0/0             884 2023-11-30 16:13 share/doc/nerdctl/docs/freebsd.md
-rw-r--r-- 0/0            2439 2023-11-30 16:13 share/doc/nerdctl/docs/gpu.md
-rw-r--r-- 0/0           14463 2023-11-30 16:13 share/doc/nerdctl/docs/ipfs.md
-rw-r--r-- 0/0            1748 2023-11-30 16:13 share/doc/nerdctl/docs/multi-platform.md
-rw-r--r-- 0/0            2936 2023-11-30 16:13 share/doc/nerdctl/docs/notation.md
-rw-r--r-- 0/0            2596 2023-11-30 16:13 share/doc/nerdctl/docs/nydus.md
-rw-r--r-- 0/0            3277 2023-11-30 16:13 share/doc/nerdctl/docs/ocicrypt.md
-rw-r--r-- 0/0            1876 2023-11-30 16:13 share/doc/nerdctl/docs/overlaybd.md
-rw-r--r-- 0/0           15657 2023-11-30 16:13 share/doc/nerdctl/docs/registry.md
-rw-r--r-- 0/0            5088 2023-11-30 16:13 share/doc/nerdctl/docs/rootless.md
-rw-r--r-- 0/0            2015 2023-11-30 16:13 share/doc/nerdctl/docs/soci.md
-rw-r--r-- 0/0           10312 2023-11-30 16:13 share/doc/nerdctl/docs/stargz.md
drwxr-xr-x 0/0               0 2023-11-30 16:20 share/doc/nerdctl-full/
-rw-r--r-- 0/0            1153 2023-11-30 16:20 share/doc/nerdctl-full/README.md
-rw-r--r-- 0/0            6400 2023-11-30 16:20 share/doc/nerdctl-full/SHA256SUMS
```
#### 启动服务

```bash
$ sudo ls /usr/local/lib/systemd/system/
buildkit.service  containerd.service  stargz-snapshotter.service
$ sudo cp /usr/local/lib/systemd/system/*.service /etc/systemd/system/

$ sudo systemctl enable  buildkit containerd --now
Created symlink from /etc/systemd/system/multi-user.target.wants/buildkit.service to /etc/systemd/system/buildkit.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/containerd.service to /etc/systemd/system/containerd.service.

$ sudo systemctl status buildkit containerd
● buildkit.service
   Loaded: loaded (/etc/systemd/system/buildkit.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-12-08 14:08:19 CST; 11s ago
  Process: 2526 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
 Main PID: 2532 (buildkitd)
    Tasks: 9
   Memory: 9.1M
   CGroup: /system.slice/buildkit.service
           └─2532 /usr/local/bin/buildkitd

Dec 08 14:08:18 localhost.localdomain systemd[1]: Starting buildkit.service...
Dec 08 14:08:18 localhost.localdomain buildkitd[2532]: time="2023-12-08T14:08:18+08:00" level=info msg="auto snapshotter: using overlayfs"
Dec 08 14:08:18 localhost.localdomain buildkitd[2532]: time="2023-12-08T14:08:18+08:00" level=warning msg="using host network as the default"
Dec 08 14:08:19 localhost.localdomain buildkitd[2532]: time="2023-12-08T14:08:19+08:00" level=info msg="found worker \"t62vn24loeqdovh6vc02s05ky\", labels=map[org.mobyproject.buildkit.wor...
Dec 08 14:08:19 localhost.localdomain buildkitd[2532]: time="2023-12-08T14:08:19+08:00" level=warning msg="skipping containerd worker, as \"/run/containerd/containerd.sock\" does not exist"
Dec 08 14:08:19 localhost.localdomain buildkitd[2532]: time="2023-12-08T14:08:19+08:00" level=info msg="found 1 workers, default=\"t62vn24loeqdovh6vc02s05ky\""
Dec 08 14:08:19 localhost.localdomain buildkitd[2532]: time="2023-12-08T14:08:19+08:00" level=warning msg="currently, only the default worker can be used."
Dec 08 14:08:19 localhost.localdomain systemd[1]: Started buildkit.service.
Dec 08 14:08:19 localhost.localdomain buildkitd[2532]: time="2023-12-08T14:08:19+08:00" level=info msg="running server on /run/buildkit/buildkitd.sock"

● containerd.service - containerd container runtime
   Loaded: loaded (/etc/systemd/system/containerd.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2023-12-08 14:08:19 CST; 11s ago
     Docs: https://containerd.io
  Process: 2533 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
 Main PID: 2540 (containerd)
    Tasks: 10
   Memory: 13.5M
   CGroup: /system.slice/containerd.service
           └─2540 /usr/local/bin/containerd

Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.088359777+08:00" level=error msg="failed to load cni during init, please check CRI plugin stat...cni config"
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.088872786+08:00" level=info msg="Start subscribing containerd event"
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.089252930+08:00" level=info msg="Start recovering state"
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.089620862+08:00" level=info msg="Start event monitor"
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.089713828+08:00" level=info msg="Start snapshots syncer"
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.089987186+08:00" level=info msg="Start cni network conf syncer for default"
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.090421286+08:00" level=info msg="Start streaming server"
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.090151447+08:00" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.090906355+08:00" level=info msg=serving... address=/run/containerd/containerd.sock
Dec 08 14:08:19 localhost.localdomain containerd[2540]: time="2023-12-08T14:08:19.091380138+08:00" level=info ms
```

> 注意：对于非root用户以上命令需要加上sudo，而且还要额外运行 `containerd-rootless-setuptool.sh installz`，这个rootless-setuptool目前不支持centOS 7及其以下。




### 命令参数
可选的参数使用和 docker run 基本一直，比如 -i、-t、--cpus、--memory 等选项，可以使用 nerdctl run --help 获取可使用的命令选项：
```bash
$ sudo nerdctl -h
nerdctl is a command line interface for containerd

Config file ($NERDCTL_TOML): /etc/nerdctl/nerdctl.toml

Usage: nerdctl [flags]

Management commands:
  apparmor   Manage AppArmor profiles
  builder    Manage builds
  container  Manage containers
  image      Manage images
  ipfs       Distributing images on IPFS
  namespace  Manage containerd namespaces
  network    Manage networks
  system     Manage containerd
  volume     Manage volumes

Commands:
  attach      Attach stdin, stdout, and stderr to a running container.
  build       Build an image from a Dockerfile. Needs buildkitd to be running.
  commit      Create a new image from a container's changes
  completion  Generate the autocompletion script for the specified shell
  compose     Compose
  cp          Copy files/folders between a running container and the local filesystem.
  create      Create a new container. Optionally specify "ipfs://" or "ipns://" scheme to pull image from IPFS.
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  help        Help about any command
  history     Show the history of an image
  images      List images
  info        Display system-wide information
  inspect     Return low-level information on objects.
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a container registry
  logout      Log out from a container registry
  logs        Fetch the logs of a container. Expected to be used with 'nerdctl run -d'.
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image from a registry. Optionally specify "ipfs://" or "ipns://" scheme to pull image from IPFS.
  push        Push an image or a repository to a registry. Optionally specify "ipfs://" or "ipns://" scheme to push image to IPFS.
  rename      rename a container
  restart     Restart one or more running containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container. Optionally specify "ipfs://" or "ipns://" scheme to pull image from IPFS.
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  start       Start one or more running containers
  stats       Display a live stream of container(s) resource usage statistics.
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update one or more running containers
  version     Show the nerdctl version information
  wait        Block until one or more containers stop, then print their exit codes.

Flags:
  -H, --H string                 Alias of --address (default "/run/containerd/containerd.sock")
  -a, --a string                 Alias of --address (default "/run/containerd/containerd.sock")
      --address string           containerd address, optionally with "unix://" prefix [$CONTAINERD_ADDRESS] (default "/run/containerd/containerd.sock")
      --cgroup-manager string    Cgroup manager to use ("cgroupfs"|"systemd") (default "cgroupfs")
      --cni-netconfpath string   cni config directory [$NETCONFPATH] (default "/etc/cni/net.d")
      --cni-path string          cni plugins binary directory [$CNI_PATH] (default "/opt/cni/bin")
      --data-root string         Root directory of persistent nerdctl state (managed by nerdctl, not by containerd) (default "/var/lib/nerdctl")
      --debug                    debug mode
      --debug-full               debug mode (with full output)
      --experimental             Control experimental: https://github.com/containerd/nerdctl/blob/main/docs/experimental.md [$NERDCTL_EXPERIMENTAL] (default true)
  -h, --help                     help for nerdctl
      --host string              Alias of --address (default "/run/containerd/containerd.sock")
      --host-gateway-ip string   IP address that the special 'host-gateway' string in --add-host resolves to. Defaults to the IP address of the host. It has no effect without setting --add-host [$NERDCTL_HOST_GATEWAY_IP] (default "192.168.10.31")
      --hosts-dir strings        A directory that contains <HOST:PORT>/hosts.toml (containerd style) or <HOST:PORT>/{ca.cert, cert.pem, key.pem} (docker style) (default [/etc/containerd/certs.d,/etc/docker/certs.d])
      --insecure-registry        skips verifying HTTPS certs, and allows falling back to plain HTTP
  -n, --n string                 Alias of --namespace (default "default")
      --namespace string         containerd namespace, such as "moby" for Docker, "k8s.io" for Kubernetes [$CONTAINERD_NAMESPACE] (default "default")
      --snapshotter string       containerd snapshotter [$CONTAINERD_SNAPSHOTTER] (default "overlayfs")
      --storage-driver string    Alias of --snapshotter (default "overlayfs")
  -v, --version                  version for nerdctl

Run 'nerdctl COMMAND --help' for more information on a command.

```
### 容器运行
默认CNI网络（10.4.0.0/24）

```bash
$ sudo nerdctl run -it --rm alpine
docker.io/library/alpine:latest:                                                  resolved       |++++++++++++++++++++++++++++++++++++++| 
index-sha256:51b67269f354137895d43f3b3d810bfacd3945438e94dc5ac55fdac340352f48:    done           |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:13b7e62e8df80264dbb747995705a986aa530415763a6c58f84a3ca8af9a5bcd: done           |++++++++++++++++++++++++++++++++++++++| 
config-sha256:f8c20f8bbcb684055b4fea470fdd169c86e87786940b3262335b12ec3adef418:   done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:661ff4d9561e3fd050929ee5097067c34bafc523ee60f5294a37fd08056a73ca:    done           |++++++++++++++++++++++++++++++++++++++| 
elapsed: 8.8 s                                                                    total:  3.3 Mi (378.6 KiB/s)                                     
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if4: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 06:6f:28:ad:82:ec brd ff:ff:ff:ff:ff:ff
    inet 10.4.0.2/24 brd 10.4.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::46f:28ff:fead:82ec/64 scope link 
       valid_lft forever preferred_lft forever
```

创建容器后台运行

```bash
$ sudo nerdctl run -t -d --name alpine1  alpine
874f917ba1e1113d6674e07f8d734dbb9d3b07c1a377feb4e8121eb7cb4ad669
```
映射端口并，设置自动重启

```bash
sudo nerdctl run -d -p 80:80 --name=nginx --restart=always nginx:alpine
```

### 容器列出

```bash
$ sudo nerdctl ps
CONTAINER ID    IMAGE                              COMMAND                   CREATED          STATUS    PORTS                 NAMES
0b07012f8ce3    docker.io/library/nginx:alpine     "/docker-entrypoint.…"    6 seconds ago    Up        0.0.0.0:80->80/tcp    nginx
874f917ba1e1    docker.io/library/alpine:latest    "/bin/sh"                 7 minutes ago    Up                              alpine1
```
### 容器详情

```bash
sudo nerdctl inspect   nginx
```

### 容器日志

```bash
sudo nerdctl logs nginx
sudo nerdctl logs -f nginx
sudo nerdctl logs -f --tail 20 nginx
```

### 容器进入

```bash
$ sudo nerdctl exec -ti alpine1 sh
/ # ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
```

### 容器停止

```bash
$ sudo nerdctl ps -a
CONTAINER ID    IMAGE                              COMMAND                   CREATED           STATUS    PORTS                 NAMES
0b07012f8ce3    docker.io/library/nginx:alpine     "/docker-entrypoint.…"    2 minutes ago     Up        0.0.0.0:80->80/tcp    nginx
874f917ba1e1    docker.io/library/alpine:latest    "/bin/sh"                 10 minutes ago    Up                              alpine1
$ nerdctl stop alpine1
alpine1
$ sudo nerdctl ps -a
CONTAINER ID    IMAGE                              COMMAND                   CREATED           STATUS                        PORTS                 NAMES
0b07012f8ce3    docker.io/library/nginx:alpine     "/docker-entrypoint.…"    2 minutes ago     Up                            0.0.0.0:80->80/tcp    nginx
874f917ba1e1    docker.io/library/alpine:latest    "/bin/sh"                 10 minutes ago    Exited (137) 3 seconds ago                          alpine1
```

或者强制停止`nerdctl kill`

```bash
sudo nerdctl start alpine1
sudo nerdctl kill alpine1
或者
sudo nerdctl kill 874f917ba1e1 
```
### 容器删除

```bash
sudo nerdctl rm alpine1
```

### 镜像列表

```bash
$ sudo nerdctl images
REPOSITORY    TAG       IMAGE ID        CREATED           PLATFORM       SIZE        BLOB SIZE
alpine        latest    51b67269f354    51 minutes ago    linux/amd64    7.3 MiB     3.3 MiB
nginx         alpine    3923f8de8d22    7 minutes ago     linux/amd64    44.2 MiB    17.1 MiB
```
### 镜像拉取

```bash
$ sudo nerdctl pull busybox:latest
docker.io/library/busybox:latest:                                                 resolved       |++++++++++++++++++++++++++++++++++++++| 
index-sha256:1ceb872bcc68a8fcd34c97952658b58086affdcb604c90c1dee2735bde5edc2f:    done           |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:1780cb47b7dfbcbf1e511be1cdb62722bd0ce208b996ea199689b56892e15af9: done           |++++++++++++++++++++++++++++++++++++++| 
config-sha256:fa152d3b2cb95c5b351d0192c52dfb996d4080b62cd4b0c4cf30b0f08ad1ea69:   done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:6672f60b6ba813a11925e795a0742f6c4c6313b64db544f60f1077619a616967:    done           |++++++++++++++++++++++++++++++++++++++| 
elapsed: 10.4s                                                                    total:  2.1 Mi (208.8 KiB/s)       
```

### 镜像标签

```bash
$ sudo nerdctl images|grep busybox
busybox       latest    1ceb872bcc68    About a minute ago    linux/amd64    4.1 MiB     2.1 MiB
$ nerdctl tag busybox:latest registry.demo:5000/busybox:v1
$ sudo nerdctl images|grep busybox
busybox                       latest    1ceb872bcc68    About a minute ago    linux/amd64    4.1 MiB     2.1 MiB
registry.demo:5000/busybox    v1        1ceb872bcc68    3 seconds ago         linux/amd64    4.1 MiB     2.1 MiB
```

### 镜像导出

```bash
$ sudo nerdctl save -o busybox.tar.gz busybox:latest
$ ls -lh busybox.tar.gz 
-rw-r--r-- 1 root root 2.2M Dec  8 15:12 busybox.tar.gz
```
### 镜像导入

```bash
$ sudo nerdctl load -i busybox.tar.gz
unpacking docker.io/library/busybox:latest (sha256:1ceb872bcc68a8fcd34c97952658b58086affdcb604c90c1dee2735bde5edc2f)...
Loaded image: busybox:latest
```
### 镜像删除

```bash
sudo nerdctl rmi busybox:latest
```

### 镜像构建
镜像构建是平时我们非常重要的一个需求，我们知道 ctr 并没有构建镜像的命令，而现在我们又不使用 Docker 了，那么如何进行镜像构建了，幸运的是 nerdctl 就提供了 nerdctl build 这样的镜像构建命令。

构建镜像需要我们安装 buildctl 并运行 buildkitd，这是因为 nerdctl build 需要依赖 buildkit 工具。
buildkit 项目也是 Docker 公司开源的一个构建工具包，支持 OCI 标准的镜像构建。它主要包含以下部分:

- 服务端 buildkitd：当前支持 runc 和 containerd 作为 worker，默认是 runc，我们这里使用 containerd
- 客户端 buildctl：负责解析 Dockerfile，并向服务端 buildkitd 发出构建请求

buildkit 工具以及 buildkitd服务配置包含在安装nerdctl完整Full过程中，此处不在再赘述。

以定制一个 nginx 镜像为例，新建一个如下所示的 Dockerfile 文件：

```bash
mkdir -p demo
cat > demo/Dockerfile <<EOF
FROM nginx:alpine
RUN echo 'Hello Nerdctl From Containerd' > /usr/share/nginx/html/index.html
EOF
```
然后在文件所在目录执行镜像构建命令：

```bash
$ sudo nerdctl build -t nginx:nerctl -f demo/Dockerfile demo
[+] Building 15.1s (6/6)                                                                                                                                                                      
[+] Building 15.2s (6/6) FINISHED                                                                                                                                                             
 => [internal] load build definition from Dockerfile                                                                                                                                     0.1s
 => => transferring dockerfile: 131B                                                                                                                                                     0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine                                                                                                                          4.9s
 => [internal] load .dockerignore                                                                                                                                                        0.1s
 => => transferring context: 2B                                                                                                                                                          0.0s
 => [1/2] FROM docker.io/library/nginx:alpine@sha256:3923f8de8d2214b9490e68fd6ae63ea604deddd166df2755b788bef04848b9bc                                                                    7.4s
 => => resolve docker.io/library/nginx:alpine@sha256:3923f8de8d2214b9490e68fd6ae63ea604deddd166df2755b788bef04848b9bc                                                                    0.0s
 => => sha256:d7fb62c2e1cc7510e9c63402d02061002604b6ab79deab339ee8abf9f7452fde 12.65MB / 12.65MB                                                                                         2.4s
 => => sha256:f9302969eafdbfd15516462ec6f9a8c8b537abb385d938e2fb154c23998c3851 1.40kB / 1.40kB                                                                                           1.2s
 => => sha256:5ea1ba8ab969c385f95c844167644f56aca56cc947548764033c92654d60a304 958B / 958B                                                                                               1.3s
 => => sha256:c99555e79d522323ea54a2e9e5c56c0bc5ed2fd7ffa16fa9cf06e5c231c15db8 1.21kB / 1.21kB                                                                                           1.4s
 => => sha256:47df6ca4b6bc8e8c42f5fcb7ce4d37737d68cb5fb5056a54605deb2b0d33415b 628B / 628B                                                                                               0.4s
 => => sha256:6a4b140a5e7cbbec14bdbc3d9e7eced3b5f87652515c1cb65af5abeb53fc9fa8 371B / 371B                                                                                               0.4s
 => => sha256:eb2797aa8e799e16f2a041cb7d709dc913519995a8a7dd22509d33c662612c5e 1.90MB / 1.90MB                                                                                           1.4s
 => => sha256:c926b61bad3b94ae7351bafd0c184c159ebf0643b085f7ef1d47ecdc7316833c 3.40MB / 3.40MB                                                                                           1.3s
 => => extracting sha256:c926b61bad3b94ae7351bafd0c184c159ebf0643b085f7ef1d47ecdc7316833c                                                                                                0.6s
 => => extracting sha256:eb2797aa8e799e16f2a041cb7d709dc913519995a8a7dd22509d33c662612c5e                                                                                                1.0s
 => => extracting sha256:47df6ca4b6bc8e8c42f5fcb7ce4d37737d68cb5fb5056a54605deb2b0d33415b                                                                                                0.0s
 => => extracting sha256:5ea1ba8ab969c385f95c844167644f56aca56cc947548764033c92654d60a304                                                                                                0.0s
 => => extracting sha256:6a4b140a5e7cbbec14bdbc3d9e7eced3b5f87652515c1cb65af5abeb53fc9fa8                                                                                                0.0s
 => => extracting sha256:c99555e79d522323ea54a2e9e5c56c0bc5ed2fd7ffa16fa9cf06e5c231c15db8                                                                                                0.0s
 => => extracting sha256:f9302969eafdbfd15516462ec6f9a8c8b537abb385d938e2fb154c23998c3851                                                                                                0.0s
 => => extracting sha256:d7fb62c2e1cc7510e9c63402d02061002604b6ab79deab339ee8abf9f7452fde                                                                                                2.2s
 => [2/2] RUN echo 'Hello Nerdctl From Containerd' > /usr/share/nginx/html/index.html                                                                                                    0.8s
 => exporting to docker image format                                                                                                                                                     1.7s
 => => exporting layers                                                                                                                                                                  0.2s
 => => exporting manifest sha256:91a740ed53294f50160476c566b407f8c53781e8cb84f35b9560a6bce998ea25                                                                                        0.0s
 => => exporting config sha256:717f4bf47513e48ae8f29093530d52e353ad69f49af55d0e9c26fe8457e57024                                                                                          0.0s
 => => sending tarball                                                                                                                                                                   1.4s
Loaded image: docker.io/library/nginx:nerctl
```
> 注意：也可以加上这个`–no-cache`选项,在每次构建去掉缓存层

###  配置tab键

```bash
$ sudo vim /etc/profile
source <(nerdctl completion bash)
$ sudo source /etc/profile
```

### 配置加速

```bash
$ sudo cat /etc/containerd/config.toml 
disabled_plugins = ["restart"]
[plugins]
   [plugins.cri.registry.mirrors."docker.io"]
     endpoint = ["https://frz7i079.mirror.aliyuncs.com"]
....
$ sudo systemctl restart containerd
```
### 配置仓库
#### http方式
无证书

```bash
$ mkdir -p /etc/docker/certs.d/registry.demo:5000
#或者创建,不过建议/etc/containerd目录创建
$ mkdir -p /etc/containerd/certs.d/registry.demo:5000
$ cat  /etc/containerd/certs.d/registry.demo\:5000/hosts.toml
server = "https://registry.demo:5000"
[host."http://registry.demo:5000"]
  capabilities = ["pull","resolve"]

$ sudo nerdctl pull  --all-platforms  registry.demo:5000/library/registry:latest
registry.demo:5000/library/registry:latest:                                    resolved       |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:36cb5b157911061fb610d8884dc09e0b0300a767a350563cbfd88b4b85324ce4: exists         |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:3790aef225b922bc97aaba099fe762f7b115aec55a0083824b548a6a1e610719:    exists         |++++++++++++++++++++++++++++++++++++++| 
config-sha256:b8604a3fe8543c9e6afc29550de05b36cd162a97aa9b2833864ea8a5be11f3e2:   exists         |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:79e9f2f55bf5465a02ee6a6170e66005b20c7aa6b115af6fcd04fad706ea651a:    exists         |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:0d96da54f60b86a4d869d44b44cfca69d71c4776b81d361bc057d6666ec0d878:    exists         |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:5b27040df4a23c90c3837d926f633fb327fb3af9ac4fa5d5bc3520ad578acb10:    exists         |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:e2ead8259a04d39492c25c9548078200c5ec429f628dcf7b7535137954cc2df0:    exists         |++++++++++++++++++++++++++++++++++++++| 
elapsed: 0.2 s                                                                    total:   0.0 B (0.0 B/s)
```


#### https方式
有证书

```bash
$ mkdir -p /etc/docker/certs.d/registry.demo:5000
#或者创建,不过建议/etc/containerd目录创建
$ mkdir -p /etc/containerd/certs.d/registry.demo:5000
$ cat  /etc/containerd/certs.d/registry.demo\:5000/hosts.toml
server = "https://registry.demo:5000"
[host."http://registry.demo:5000"]
  capabilities = ["pull","resolve"]
  ca = ["ca.crt"]
....
#把镜像仓库证书拷贝到 /etc/containerd/certs.d/registry.demo\:5000目录
$ ls  /etc/containerd/certs.d/registry.demo\:5000
ca.crt  hosts.toml

#登陆
$ echo 123456 | sudo nerdctl login --username "admin" --password-stdin  registry.demo:5000
#拉取镜像
$ sudo nerdctl pull  --all-platforms  registry.demo:5000/library/registry:latest
```

## ctr
### 简介
ctr是作为 containerd 项目的一部分提供的命令行客户端。如果您在一台机器上运行了 containerd，那么ctr二进制文件很可能也在那里。该ctr界面 [显然] 与 Docker CLI不兼容，乍一看，可能看起来不太用户友好。显然，它的主要受众是测试守护进程的容器开发人员。但是，由于它最接近实际的 containerd API，因此它可以作为一种很好的探索手段——通过检查可用命令，您可以大致了解 containerd可以做什么和不可以做什么。ctr也非常适合学习的能力低级别[OCI]容器的运行时间，因为ctr + containerd是更接近实际的容器比docker + dockerd。

### 命令参数

```bash
$ sudo  ctr --help
NAME:
   ctr - 
        __
  _____/ /______
 / ___/ __/ ___/
/ /__/ /_/ /
\___/\__/_/

containerd CLI


USAGE:
   ctr [global options] command [command options] [arguments...]

VERSION:
   v1.7.5

DESCRIPTION:
   
ctr is an unsupported debug and administrative client for interacting
with the containerd daemon. Because it is unsupported, the commands,
options, and operations are not guaranteed to be backward compatible or
stable from release to release of the containerd project.

COMMANDS:
   plugins, plugin            Provides information about containerd plugins
   version                    Print the client and server versions
   containers, c, container   Manage containers
   content                    Manage content
   events, event              Display containerd events
   images, image, i           Manage images
   leases                     Manage leases
   namespaces, namespace, ns  Manage namespaces
   pprof                      Provide golang pprof outputs for containerd
   run                        Run a container
   snapshots, snapshot        Manage snapshots
   tasks, t, task             Manage tasks
   install                    Install a new package
   oci                        OCI tools
   sandboxes, sandbox, sb, s  Manage sandboxes
   info                       Print the server info
   shim                       Interact with a shim directly
   help, h                    Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug                      Enable debug output in logs
   --address value, -a value    Address for containerd's GRPC server (default: "/run/containerd/containerd.sock") [$CONTAINERD_ADDRESS]
   --timeout value              Total timeout for ctr commands (default: 0s)
   --connect-timeout value      Timeout for connecting to containerd (default: 0s)
   --namespace value, -n value  Namespace to use with commands (default: "default") [$CONTAINERD_NAMESPACE]
   --help, -h                   show help
   --version, -v                print the version
```


### 镜像拉取

```bash
$ sudo ctr i pull docker.io/library/nginx:latest
docker.io/library/nginx:latest:                                                   resolved       |++++++++++++++++++++++++++++++++++++++| 
index-sha256:10d1f5b58f74683ad34eb29287e07dab1e90f10af243f151bb50aa5dbb4d62ee:    done           |++++++++++++++++++++++++++++++++++++++| 
manifest-sha256:3c4c1f42a89e343c7b050c5e5d6f670a0e0b82e70e0e7d023f10092a04bbb5a7: done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:9b16c94bb68628753a94b89ddf26abc0974cd35a96f785895ab011d9b5042ee5:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:51735783196785d1f604dc7711ea70fb3fab3cd9d99eaeff991c5afbfa0f20e8:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:9ea27b074f71d5766a59cdbfaa15f4cd3d17bffb83fed066373eb287326abbd3:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:9a59d19f9c5bb1ebdfef2255496b1bb5d658fdccc300c4c1f0d18c73f1bb14b5:    done           |++++++++++++++++++++++++++++++++++++++| 
config-sha256:a6bd71f48f6839d9faae1f29d3babef831e76bc213107682c5cc80f0cbb30866:   done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:c6edf33e2524b241a0b191d0a0d2ca3d8d4ae7470333b059dd97ba30e663a1a3:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:1f7ce2fa46ab3942feabee654933948821303a5a821789dddab2d8c3df59e227:    done           |++++++++++++++++++++++++++++++++++++++| 
layer-sha256:84b1ff10387b26e2952f006c0a4fe4c6f3c0743cb08ee448bb7157220ad2fc8f:    done           |++++++++++++++++++++++++++++++++++++++| 
elapsed: 80.7s                                                                    total:  67.3 M (853.7 KiB/s)                                     
unpacking linux/amd64 sha256:10d1f5b58f74683ad34eb29287e07dab1e90f10af243f151bb50aa5dbb4d62ee...
done: 12.744883787s
```

### 镜像压缩与解压
你可以解压来自docker 构建的镜像
```bash
sudo docker build -t my-app .
sudo docker save -o my-app.tar my-app
sudo ctr images import my-app.tar
```
将本地 nginx 镜像打包

```bash
$ sudo ctr i export nginx-latest.tar docker.io/library/nginx:latest
$ ls
nginx-latest.tar
```

### 镜像打标签

```bash
$ sudo ctr images ls
REF                            TYPE                                                      DIGEST                                                                  SIZE     PLATFORMS                                                                                               LABELS 
docker.io/library/nginx:latest application/vnd.docker.distribution.manifest.list.v2+json sha256:10d1f5b58f74683ad34eb29287e07dab1e90f10af243f151bb50aa5dbb4d62ee 67.3 MiB linux/386,linux/amd64,linux/arm/v5,linux/arm/v7,linux/arm64/v8,linux/mips64le,linux/ppc64le,linux/s390x -      

$ ctr images tag docker.io/library/nginx:latest docker.io/library/nginx:v1
docker.io/library/nginx:v1

$ sudo ctr images ls
REF                            TYPE                                                      DIGEST                                                                  SIZE     PLATFORMS                                                                                               LABELS 
docker.io/library/nginx:latest application/vnd.docker.distribution.manifest.list.v2+json sha256:10d1f5b58f74683ad34eb29287e07dab1e90f10af243f151bb50aa5dbb4d62ee 67.3 MiB linux/386,linux/amd64,linux/arm/v5,linux/arm/v7,linux/arm64/v8,linux/mips64le,linux/ppc64le,linux/s390x -      
docker.io/library/nginx:v1     application/vnd.docker.distribution.manifest.list.v2+json sha256:10d1f5b58f74683ad34eb29287e07dab1e90f10af243f151bb50aa5dbb4d62ee 67.3 MiB linux/386,linux/amd64,linux/arm/v5,linux/arm/v7,linux/arm64/v8,linux/mips64le,linux/ppc64le,linux/s390x -  
```

### 镜像挂载

```bash
$ mkdir /tmp/nginx
$ sudo ctr images mount docker.io/nginx:latest /tmp/nginx
 
$ ls -l /tmp/nginx
total 80
drwxr-xr-x 2 root root 4096 Oct 18  2018 bin
drwxr-xr-x 2 root root 4096 Apr 24  2018 boot
drwxr-xr-x 4 root root 4096 Oct 18  2018 dev
drwxr-xr-x 1 root root 4096 Oct 24  2018 etc
drwxr-xr-x 2 root root 4096 Apr 24  2018 home
drwxr-xr-x 3 root root 4096 Oct 24  2018 nginx
...
 
$ sudo ctr images unmount /tmp/nginx
```
### 容器运行
有了一个本地镜像，你可以通过ctr运行<image-ref> <container-id>来运行一个容器。例如: 
启动容器直接输出日志，停止便自动删除
```bash
$ sudo ctr run --rm -t docker.io/library/nginx:latest nginx1
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/12/07 08:05:16 [notice] 1#1: using the "epoll" event method
2023/12/07 08:05:16 [notice] 1#1: nginx/1.25.3
2023/12/07 08:05:16 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2023/12/07 08:05:16 [notice] 1#1: OS: Linux 6.5.5-1.el7.elrepo.x86_64
2023/12/07 08:05:16 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1024:1024
2023/12/07 08:05:16 [notice] 1#1: start worker processes
2023/12/07 08:05:16 [notice] 1#1: start worker process 30
2023/12/07 08:05:16 [notice] 1#1: start worker process 31
2023/12/07 08:05:16 [notice] 1#1: start worker process 32
2023/12/07 08:05:16 [notice] 1#1: start worker process 33
```
>注意: 与友好的docker运行生成唯一的容器ID不同，使用ctr，你必须自己提供唯一的容器ID。ctr运行命令也只支持一些常见的docker运行标志: --env， -t，--tty， -d，--detach，--rm，等等。但是没有端口发布或使用--restart总是开箱即用的自动容器重新启动。

一般我们会将容器进程防止后台启动，这样启动容器

```bash
$ sudo ctr run  -t -d  docker.io/library/nginx:latest nginx1
#查看容器
$ sudo ctr c ls
CONTAINER    IMAGE                             RUNTIME                  
nginx1       docker.io/library/nginx:latest    io.containerd.runc.v2   
```

### 容器创建与启动
ctrl运行命令:

```bash
$ sudo ctr container create -t docker.io/library/nginx:latest nginx
$ sudo ctr container ls
CONTAINER    IMAGE                              RUNTIME
nginx      docker.io/library/nginx:latest     io.containerd.runc.v2
 
$ sudo ctr task ls
TASK    PID    STATUS        # Empty!
 
$ sudo ctr task start -d nginx  # -d for --detach
$ sudo ctr task list
TASK     PID      STATUS
nginx  10074    RUNNING
```

### 容器进入
使用ctr任务连接，你可以重新连接到一个在容器中运行的现有任务的stdio流:

```bash
#当退出会自动停掉容器
sudo ctr task attach nginx 
#当退出不会停掉容器
sudo ctr task exec -t --exec-id $RANDOM nginx bash 
 
```
### 	容器停止

```bash
$ sudo ctr task kill  nginx
$ sudo ctr task ls
TASK     PID     STATUS    
nginx    2355    STOPPED
```
### 容器删除
delete, del, remove, rm 参数都可以

```bash
$ sudo ctr task rm  nginx
$ sudo ctr task ls
TASK    PID    STATUS    
```

##  crictl
### 简介
[crictl](https://github.com/kubernetes-sigs/cri-tools) 是Kubelet容器接口（CRI）的CLI和验证工具。你可以使用它来检查和调试 Kubernetes 节点上的容器运行时和应用程序。 crictl 和它的源代码在 `cri-tools` 代码库。

它不会负责以下内容：
- 基于CRI构建新的kubelet容器运行时；
- 由终端用户管理CRI兼容运行时的pod/container，例如crictl创建的pod可能会被kubelet自动删除，因为在kube-apiserver上不存在；
- crictl 不是 docker 替代者。

 版本兼容

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


### 安装

下载指定版本：[https://github.com/kubernetes-sigs/cri-tools/releases](https://github.com/kubernetes-sigs/cri-tools/releases)

```bash
export VERSION="v1.28.0"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```

### 配置
`crictl` 命令有几个子命令和运行时参数。 有关详细信息，请使用 crictl help 或 crictl <subcommand> help 获取帮助信息。

crictl 默认连接到 `unix:///var/run/dockershim.sock`。 对于其他的运行时，你可以用多种不同的方法设置端点：

- 通过设置参数 `--runtime-endpoint` 和 `--image-endpoint`；
- 通过设置环境变量 `CONTAINER_RUNTIME_ENDPOINT` 和 `IMAGE_SERVICE_ENDPOINT`；
- 通过在配置文件中设置端点 `--config=/etc/crictl.yaml`；
- 你还可以在连接到服务器并启用或禁用调试时指定超时值，方法是在配置文件中指定 timeout 或 debug 值，或者使用 --timeout 和 --debug 命令行参数。

要查看或编辑当前配置，请查看或编辑 /etc/crictl.yaml 的内容。

```bash
sudo cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 10
debug: true
EOF
```
### 命令参数

```bash
$ sudo crictl -h
NAME:
   crictl - client for CRI

USAGE:
   crictl [global options] command [command options] [arguments...]

VERSION:
   v1.24.0

COMMANDS:
   attach              Attach to a running container
   create              Create a new container
   exec                Run a command in a running container
   version             Display runtime version information
   images, image, img  List images
   inspect             Display the status of one or more containers
   inspecti            Return the status of one or more images
   imagefsinfo         Return image filesystem info
   inspectp            Display the status of one or more pods
   logs                Fetch the logs of a container
   port-forward        Forward local port to a pod
   ps                  List containers
   pull                Pull an image from a registry
   run                 Run a new container inside a sandbox
   runp                Run a new pod
   rm                  Remove one or more containers
   rmi                 Remove one or more images
   rmp                 Remove one or more pods
   pods                List pods
   start               Start one or more created containers
   info                Display information of the container runtime
   stop                Stop one or more running containers
   stopp               Stop one or more running pods
   update              Update one or more running containers
   config              Get and set crictl client configuration options
   stats               List container(s) resource usage statistics
   statsp              List pod resource usage statistics
   completion          Output shell completion code
   help, h             Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config value, -c value            Location of the client config file. If not specified and the default does not exist, the program's directory is searched as well (default: "/etc/crictl.yaml") [$CRI_CONFIG_FILE]
   --debug, -D                         Enable debug mode (default: false)
   --image-endpoint value, -i value    Endpoint of CRI image manager service (default: uses 'runtime-endpoint' setting) [$IMAGE_SERVICE_ENDPOINT]
   --runtime-endpoint value, -r value  Endpoint of CRI container runtime service (default: uses in order the first successful one of [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]). Default is now deprecated and the endpoint should be set instead. [$CONTAINER_RUNTIME_ENDPOINT]
   --timeout value, -t value           Timeout of connecting to the server in seconds (e.g. 2s, 20s.). 0 or less is set to default (default: 2s)
   --help, -h                          show help (default: false)
   --version, -v                       print the version (default: false)
```

### 容器列出

```bash
sudo crictl ps 
sudo crictl ps  -a
```

### 容器启动

```sh
$ sudo crictl start 3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60
3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60

$ sudo crictl ps
CONTAINER ID        IMAGE               CREATED              STATE               NAME                ATTEMPT
3e025dd50a72d       busybox             About a minute ago   Running             busybox             0
```

### 容器进入

```sh
$ sudo crictl exec -i -t 3e025dd50a72d956c4f14881fbb5b1080c9275674e95fb67f965f6478a957d60 ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
```

### pod 查看

```bash
$ sudo crictl pods
POD ID              CREATED             STATE               NAME                                        NAMESPACE           ATTEMPT             RUNTIME
f4c4b9dcaf48d       5 hours ago         Ready               prometheus-prometheus-node-exporter-4w4qt   prometheus          0                   (default)
b654f6084870c       5 weeks ago         Ready               kube-proxy-9n99f                            kube-system         0                   (default)
2ec8090dd8bbb       8 weeks ago         Ready               coredns-5867d9544c-xqlq2                    kube-system         15                  (default)
f20b0354b4e15       8 weeks ago         Ready               dns-autoscaler-59b8867c86-qdb5q             kube-system         15                  (default)
8d123201668ca       8 weeks ago         Ready               calico-node-5nzsq                           kube-system         15                  (default)

$ sudo crictl pods  --name kube-proxy-9n99f
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME
b654f6084870c       5 weeks ago         Ready               kube-proxy-9n99f    kube-system         0                   (default)
```

### 拉取镜像

```bash
$ sudo crictl pull busybox:latest
Image is up to date for sha256:fa152d3b2cb95c5b351d0192c52dfb996d4080b62cd4b0c4cf30b0f08ad1ea69


$ sudo crictl pull --creds admin:123456 registry.paas/cmss/cni:v3.23.2
Image is up to date for sha256:a87d3f6f1b8fdc077e74ad6874301f57490f5b38cc731d5d6f5803f36837b4b1
```

