
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bedce91014424344a3a2e536dff715b0.jpeg)



## 1. 什么是 nerdctl？
`nerdctl` 是一个与 `docker` 命令兼容的容器运行时 CLI，基于 `containerd`，可以在没有 `dockerd` 的情况下运行容器。它适用于 Kubernetes 和轻量级容器环境，特别适合与 `containerd` 和 `nerdctl` 结合使用的开发者和运维人员。

## 2. 配置文件位置
`nerdctl` 的默认配置文件路径如下：
- Linux/macOS: `~/.config/nerdctl/nerdctl.toml`
- Windows: `C:\Users\<用户名>\.config\nerdctl\nerdctl.toml`

你可以手动创建 `nerdctl.toml` 来修改默认行为。

## 3. 配置项解析
### 3.1 运行时（runtime）配置
```toml
[default]
cni_path = "/opt/cni/bin"
cni_config_path = "/etc/cni/net.d"
```
- `cni_path`：CNI 插件的可执行文件目录
- `cni_config_path`：CNI 网络配置文件路径

### 3.2 容器镜像设置
```toml
[registry]
  mirrors = { "docker.io" = { endpoint = ["https://mirror.gcr.io"] } }
```
- `mirrors`：配置加速镜像源，例如使用 GCR 代理 `mirror.gcr.io`

### 3.3 存储配置
```toml
[containerd]
snapshotter = "overlayfs"
```
- `snapshotter`：设置存储后端（如 `overlayfs`, `zfs`, `btrfs`）

### 3.4 nerdctl 命令别名
```toml
[alias]
  myrun = "run --rm -it"
```
- `alias`：定义自定义命令别名，例如 `nerdctl myrun ubuntu` 相当于 `nerdctl run --rm -it ubuntu`

## 4. 使用 `nerdctl config` 命令
`nerdctl config` 提供了一些管理配置的子命令：
- `nerdctl config info`：查看当前 nerdctl 配置
- `nerdctl config show-defaults`：显示默认配置
- `nerdctl config edit`：编辑配置文件

示例：
```sh
nerdctl config info
```

## 5. 配置示例：使用 nerdctl 运行私有仓库镜像
如果你使用的是私有镜像仓库（如 `harbor.example.com`），可以在 `nerdctl.toml` 中添加：
```toml
[registry]
  [registry."harbor.example.com"]
    http = true
    insecure = true
```
然后执行 `nerdctl pull harbor.example.com/my-image:v1` 以拉取镜像。


## 6. kubernetes默认 nerdctl 配置

- Rootful mode: /etc/nerdctl/nerdctl.toml
- Rootless mode: ~/.config/nerdctl/nerdctl.toml

```bash
$ vim /etc/nerdctl/nerdctl.toml 
debug             = false
debug_full        = false
address           = "unix:///var/run/containerd/containerd.sock"
namespace         = "k8s.io"
snapshotter       = "overlayfs"
cni_path          = "/opt/cni/bin"
cni_netconfpath   = "/etc/cni/net.d"
cgroup_manager    = "systemd"
hosts_dir         = ["/etc/containerd/certs.d"]
insecure_registry = true 
```
## 7. 结论
`nerdctl` 是一个强大且轻量的 `containerd` CLI，支持与 `docker` 类似的操作方式。通过配置 `nerdctl.toml`，可以优化网络、存储和镜像管理，使其更适合你的工作流。如果你正在使用 Kubernetes 或者希望摆脱 `dockerd`，`nerdctl` 绝对值得一试！






参考：

- [https://github.com/containerd/nerdctl/blob/main/docs/config.md](https://github.com/containerd/nerdctl/blob/main/docs/config.md)

