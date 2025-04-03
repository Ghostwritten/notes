
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c42744f1e73f46cef58f6551082b562d.png)

## 1.  Krew 是什么？
[Krew](https://github.com/kubernetes-sigs/krew) 是一个 Kubernetes 的插件管理器，它能够帮助用户轻松安装和管理 kubectl 插件。在使用 kubectl 命令时，插件可以为用户提供额外的功能和扩展，使用户更加高效地管理 Kubernetes 集群。。它类似于[apt](https://ghostwritten.blog.csdn.net/article/details/105641494)、[dnf](https://blog.csdn.net/xixihahalelehehe/article/details/123168620) 或 [brew](https://brew.sh/) 等工具。有很多非常有用的 kubectl 插件，如 kubectl-who-can、kubectl-tree、kubectl-ctx 等。但是手动安装和管理这些插件可能会比较麻烦。Krew 的出现就解决了这个问题。今天，Krew上有 [超过200个kubectl插件](https://krew.sigs.k8s.io/plugins/)。

Krew 功能：
- 发现Kubectl插件，
- 安装Kubectl插件
- 并使安装的插件保持最新。


## 2.  如何开发一个 kubectl 插件
写一个任务脚本：`kubectl-foo`

```bash
#!/bin/bash

# optional argument handling
if [[ "$1" == "version" ]]
then
    echo "1.0.0"
    exit 0
fi

# optional argument handling
if [[ "$1" == "config" ]]
then
    echo "$KUBECONFIG"
    exit 0
fi

echo "I am a plugin named kubectl-foo"
```
授执行权限

```bash
chmod +x ./kubectl-foo
mv ./kubectl-foo /usr/local/bin
```
测试

```bash
$ kubectl foo
I am a plugin named kubectl-foo

$ kubectl foo version
1.0.0

$ kubectl foo config

$ KUBECONFIG=/etc/kube/config kubectl foo config
/etc/kube/config

$  kubectl plugin list
The following compatible plugins are available:

/usr/local/bin/kubectl-foo
```

## 3. 安装 Krew 工具
- [官方安装 brew](https://krew.sigs.k8s.io/docs/user-guide/setup/install/) ，这里以 linux 系统为例：

1. 确保 [git 已经安装](https://ghostwritten.blog.csdn.net/article/details/125107061)

2. 通过以下命令安装 `krew`:

```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```
输出：

```bash
++ mktemp -d
+ cd /tmp/tmp.Ez9fDpOANb
++ tr '[:upper:]' '[:lower:]'
++ uname
+ OS=linux
++ sed -e s/x86_64/amd64/ -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/'
++ uname -m
+ ARCH=amd64
+ KREW=krew-linux_amd64
+ curl -fsSLO https://github.com/kubernetes-sigs/krew/releases/latest/download/krew-linux_amd64.tar.gz
+ tar zxvf krew-linux_amd64.tar.gz
./LICENSE
./krew-linux_amd64
+ ./krew-linux_amd64 install krew
Adding "default" plugin index from https://github.com/kubernetes-sigs/krew-index.git.
Updated the local copy of plugin index.
Installing plugin: krew
Installed plugin: krew
\
 | Use this plugin:
 | 	kubectl krew
 | Documentation:
 | 	https://krew.sigs.k8s.io/
 | Caveats:
 | \
 |  | krew is now installed! To start using kubectl plugins, you need to add
 |  | krew's installation directory to your PATH:
 |  |
 |  |   * macOS/Linux:
 |  |     - Add the following to your ~/.bashrc or ~/.zshrc:
 |  |         export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
 |  |     - Restart your shell.
 |  |
 |  |   * Windows: Add %USERPROFILE%\.krew\bin to your PATH environment variable
 |  |
 |  | To list krew commands and to get help, run:
 |  |   $ kubectl krew
 |  | For a full list of available plugins, run:
 |  |   $ kubectl krew search
 |  |
 |  | You can find documentation at
 |  |   https://krew.sigs.k8s.io/docs/user-guide/quickstart/.
 | /
/
```

3. 将`$HOME/.krew/bin`目录添加到PATH环境变量中。要做到这一点，更新你的`.bashrc`或`.zshrc`文件，并追加以下行:

```bash
cat <<EOF>> /$HOME/.bashrc
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
EOF
source /$HOME/.bashrc
```

4. 检查安装是否成功

```bash
$ kubectl krew
krew is the kubectl plugin manager.
You can invoke krew through kubectl: "kubectl krew [command]..."

Usage:
  kubectl krew [command]

Available Commands:
  completion  generate the autocompletion script for the specified shell
  help        Help about any command
  index       Manage custom plugin indexes
  info        Show information about an available plugin
  install     Install kubectl plugins
  list        List installed kubectl plugins
  search      Discover kubectl plugins
  uninstall   Uninstall plugins
  update      Update the local copy of the plugin index
  upgrade     Upgrade installed plugins to newer versions
  version     Show krew version and diagnostics

Flags:
  -h, --help      help for krew
  -v, --v Level   number for the log level verbosity

Use "kubectl krew [command] --help" for more information about a command.


$ kubectl krew version
OPTION            VALUE
GitTag            v0.4.3
GitCommit         dbfefa5
IndexURI          https://github.com/kubernetes-sigs/krew-index.git
BasePath          /root/.krew
IndexPath         /root/.krew/index/default
InstallPath       /root/.krew/store
BinPath           /root/.krew/bin
DetectedPlatform  linux/amd64
```

## 4. krew 常用命令

```bash
kubectl krew search：搜索可用的插件；
kubectl krew install：安装插件；
kubectl krew list：列出已经安装的插件；
kubectl krew uninstall：卸载插件；
kubectl krew upgrade：升级插件。
```

## 5. kubectl 插件实践场景

### 5.1 kubectl-who-can

```bash
$ kubectl krew search who-can

NAME                 DESCRIPTION                                       INSTALLED
kubectl-who-can      Discover `who` can perform `what` on Kubernetes.  No    
```

    
可以看到，kubectl-who-can 插件还没有被安装。我们可以使用 kubectl krew install 命令来安装：

```bash
$ kubectl krew install who-can
```
输出：

```bash
Updated the local copy of plugin index.
Installing plugin: who-can
Installed plugin: who-can
\
 | Use this plugin:
 |      kubectl who-can
 | Documentation:
 |      https://github.com/aquasecurity/kubectl-who-can
 | Caveats:
 | \
 |  | The plugin requires the rights to list (Cluster)Role and (Cluster)RoleBindings.
 | /
/
WARNING: You installed plugin "who-can" from the krew-index plugin repository.
   These plugins are not audited for security by the Krew maintainers.
   Run them at your own risk.
```

安装完成后，我们可以使用 kubectl who-can 命令来查询哪些用户或服务账号可以执行哪些操作：

```bash
$ kubectl who-can get pods
No subjects found with permissions to get pods assigned through RoleBindings

CLUSTERROLEBINDING                             SUBJECT                                 TYPE            SA-NAMESPACE
argocd-application-controller                  argocd-application-controller           ServiceAccount  argocd
argocd-server                                  argocd-server                           ServiceAccount  argocd
cluster-admin                                  system:masters                          Group
local-path-provisioner-bind                    local-path-provisioner-service-account  ServiceAccount  local-path-storage
loki-promtail                                  loki-promtail                           ServiceAccount  loki-stack
prometheus-ghostwritten-ku-prometheus          prometheus-ghostwritten-ku-prometheus   ServiceAccount  prometheus
system:controller:deployment-controller        deployment-controller                   ServiceAccount  kube-system
system:controller:endpoint-controller          endpoint-controller                     ServiceAccount  kube-system
system:controller:endpointslice-controller     endpointslice-controller                ServiceAccount  kube-system
system:controller:ephemeral-volume-controller  ephemeral-volume-controller             ServiceAccount  kube-system
system:controller:generic-garbage-collector    generic-garbage-collector               ServiceAccount  kube-system
system:controller:namespace-controller         namespace-controller                    ServiceAccount  kube-system
system:controller:persistent-volume-binder     persistent-volume-binder                ServiceAccount  kube-system
system:controller:pvc-protection-controller    pvc-protection-controller               ServiceAccount  kube-system
system:controller:statefulset-controller       statefulset-controller                  ServiceAccount  kube-system
system:kube-scheduler                          system:kube-scheduler                   User
```

### 5.2 ingress-nginx
安装 ingress-nginx
```bash
kubectl krew upgrade
kubectl krew search ingress-nginx
kubectl krew install ingress-nginx
```
#### 5.2.1 conf

```bash
$ kubectl ingress-nginx conf -n ingress-nginx

# Configuration checksum: 15967764457268127562

# setup custom paths that do not require root access
pid /tmp/nginx/nginx.pid;

daemon off;

worker_processes 4;

worker_rlimit_nofile 1047552;

worker_shutdown_timeout 240s ;

events {
        multi_accept        on;
        worker_connections  16384;
        use                 epoll;

}

http {
        lua_package_path "/etc/nginx/lua/?.lua;;";

        lua_shared_dict balancer_ewma 10M;
        lua_shared_dict balancer_ewma_last_touched_at 10M;
        lua_shared_dict balancer_ewma_locks 1M;
        lua_shared_dict certificate_data 20M;
        lua_shared_dict certificate_servers 5M;
        lua_shared_dict configuration_data 20M;
        lua_shared_dict global_throttle_cache 10M;
        lua_shared_dict ocsp_response_cache 5M;

        init_by_lua_block {
                collectgarbage("collect")

                -- init modules
                local ok, res

                ok, res = pcall(require, "lua_ingress"
```

或者

```bash
$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-tsgb7        0/1     Completed   0          40d
ingress-nginx-admission-patch-w8lz6         0/1     Completed   0          40d
ingress-nginx-controller-7466db4b45-q24cr   1/1     Running     0          30d

$ kubectl exec -n ingress-nginx ingress-nginx-controller-7466db4b45-q24cr -- cat /etc/nginx/nginx.conf | grep 'server_name'
        server_names_hash_max_size      1024;
        server_names_hash_bucket_size   64;
        server_name_in_redirect off;
                server_name _ ;
                server_name alertmanager.demo.com ;
                server_name argocd.example.com ;
                server_name grafana.demo.com ;
                server_name grafana.loki.com ;
                server_name log-example.com ;
                server_name progressive.auto ;
                server_name prometheus.demo.com ;
```

#### 5.2.2  exec

```bash
$ kubectl ingress-nginx exec -i -n ingress-nginx -- ls /etc/nginx
fastcgi.conf
fastcgi.conf.default
fastcgi_params
fastcgi_params.default
geoip
koi-utf
koi-win
lua
mime.types
mime.types.default
modsecurity
modules
nginx.conf
nginx.conf.default
opentracing.json
owasp-modsecurity-crs
scgi_params
scgi_params.default
template
uwsgi_params
uwsgi_params.default
win-utf
```
####  5.2.3 logs

```bash
$ kubectl ingress-nginx logs -n ingress-nginx
.........
2023/03/01 05:53:08 [error] 994#994: *2278835 [lua] balancer.lua:203: route_to_alternative_balancer(): no alternative balancer for backend: default-canary-demo-canary-http, client: 172.7.245.44, server: progressive.auto, request: "POST /color HTTP/1.1", host: "progressive.auto", referrer: "http://progressive.auto/"
2023/03/01 05:53:08 [error] 993#993: *2278889 [lua] balancer.lua:203: route_to_alternative_balancer(): no alternative balancer for backend: default-canary-demo-canary-http, client: 172.7.245.44, server: progressive.auto, request: "POST /color HTTP/1.1", host: "progressive.auto", referrer: "http://progressive.auto/"
172.7.245.44 - - [01/Mar/2023:05:53:08 +0000] "POST /color HTTP/1.1" 200 6 "http://progressive.auto/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36" 496 0.002 [default-canary-demo-http] [] 10.244.0.28:8080 6 0.002 200 319be0b7644e69de7163252781254fd0
172.7.245.44 - - [01/Mar/2023:05:53:08 +0000] "POST /color HTTP/1.1" 200 6 "http://progressive.auto/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36" 496 0.002 [default-canary-demo-http] [] 10.244.0.28:8080 6 0.002 200 1d00909fd3e95aad789658930060202d
.......
```
更多关于 `ingress-nginx` 插件使用方法可以[参考这里](https://kubernetes.github.io/ingress-nginx/kubectl-plugin/) 


参考：

- [https://krew.sigs.k8s.io/docs/](https://krew.sigs.k8s.io/docs/)
- [Extend kubectl with plugins](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/)
- [A New Kubectl Plugin for Kubernetes Ingress Controller ingress-nginx](https://shopify.engineering/kubectl-plugin-kubernetes-ingress-controller-ingress-nginx)
- [Kubectl plugins available](https://krew.sigs.k8s.io/plugins/)
