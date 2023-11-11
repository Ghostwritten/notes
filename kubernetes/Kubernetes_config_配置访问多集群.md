#  Kubernetes config Overivew
tags: config,对象
<!-- catalog: ~config~ -->



![在这里插入图片描述](https://img-blog.csdnimg.cn/6eb6db2df289460ea4e964fa82d0127e.png)


##  1. 方法

###  1.1 定义集群、用户和上下文
假设用户有两个集群，一个用于正式开发工作，一个用于其它临时用途（scratch）。 在 `development` 集群中，前端开发者在名为 `frontend` 的名字空间下工作， 存储开发者在名为 `storage` 的名字空间下工作。在 `scratch` 集群中， 开发人员可能在默认名字空间下工作，也可能视情况创建附加的名字空间。 访问开发集群需要通过证书进行认证。 访问其它临时用途的集群需要通过用户名和密码进行认证。

创建名为 `config-exercise` 的目录。在 `config-exercise` 目录中，创建名为 `config-demo` 的文件，其内容为：

```bash
apiVersion: v1
kind: Config
preferences: {}

clusters:
- cluster:
  name: development
- cluster:
  name: scratch

users:
- name: developer
- name: experimenter

contexts:
- context:
  name: dev-frontend
- context:
  name: dev-storage
- context:
  name: exp-scratch
```


配置文件描述了集群、用户名和上下文。`config-demo` 文件中含有描述两个集群、 两个用户和三个上下文的框架。

### 1.2 集群配置

进入 `config-exercise` 目录。输入以下命令，将集群详细信息添加到配置文件中：

```bash
kubectl config --kubeconfig=config-demo set-cluster development --server=https://1.2.3.4 --certificate-authority=fake-ca-file
kubectl config --kubeconfig=config-demo set-cluster scratch --server=https://5.6.7.8 --insecure-skip-tls-verify
```
###  1.3 用户配置
将用户详细信息添加到配置文件中：

```bash
kubectl config --kubeconfig=config-demo set-credentials developer --client-certificate=fake-cert-file --client-key=fake-key-seefile
kubectl config --kubeconfig=config-demo set-credentials experimenter --username=exp --password=some-password
```

###  1.4 上下文配置

```bash
kubectl config --kubeconfig=config-demo set-context dev-frontend --cluster=development --namespace=frontend --user=developer
kubectl config --kubeconfig=config-demo set-context dev-storage --cluster=development --namespace=storage --user=developer
kubectl config --kubeconfig=config-demo set-context exp-scratch --cluster=scratch --namespace=default --user=experimenter
```

###  1.5 查看配置
打开 `config-demo` 文件查看添加的详细信息。也可以使用 `config view` 命令进行查看：

```bash
kubectl config --kubeconfig=config-demo view
```
输出：

```bash
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
- cluster:
    insecure-skip-tls-verify: true
    server: https://5.6.7.8
  name: scratch
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: scratch
    namespace: default
    user: experimenter
  name: exp-scratch
current-context: ""
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
- name: experimenter
  user:
    password: some-password
    username: exp
```

其中的 `fake-ca-file`、`fake-cert-file` 和 `fake-key-file` 是证书文件路径名的占位符。 你需要更改这些值，使之对应你的环境中证书文件的实际路径名。

有时你可能希望在这里使用 `BASE64` 编码的数据而不是一个个独立的证书文件。 如果是这样，你需要在键名上添加 `-data` 后缀。例如， `certificate-authority-data`、`client-certificate-data` 和 `client-key-data`。

每个上下文包含三部分（集群、用户和名字空间），例如， `dev-frontend` 上下文表明：使用 `developer` 用户的凭证来访问 development 集群的 `frontend` 名字空间。

###  1.6 配置当前上下文

```bash
kubectl config --kubeconfig=config-demo use-context dev-frontend
```
现在当输入 `kubectl` 命令时，相应动作会应用于 `dev-frontend` 上下文中所列的集群和名字空间， 同时，命令会使用 `dev-frontend` 上下文中所列用户的凭证。

使用 `--minify` 参数，来查看与当前上下文相关联的配置信息。

```bash
kubectl config --kubeconfig=config-demo view --minify
```
`dev-frontend` 上下文相关的配置信息：

```bash
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
current-context: dev-frontend
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
```

###  1.7 切换用户上下文
现在假设用户希望在其它临时用途集群中工作一段时间。

将当前上下文更改为 `exp-scratch`：

```bash
kubectl config --kubeconfig=config-demo use-context exp-scratch
```
现在你发出的所有 `kubectl` 命令都将应用于 `scratch` 集群的默认名字空间。 同时，命令会使用 `exp-scratch` 上下文中所列用户的凭证。

查看更新后的当前上下文 `exp-scratch` 相关的配置：

```bash
kubectl config --kubeconfig=config-demo view --minify
```
最后，假设用户希望在 `development` 集群中的 `storage` 名字空间下工作一段时间。

将当前上下文更改为 `dev-storage`：

```bash
kubectl config --kubeconfig=config-demo use-context dev-storage
```
查看更新后的当前上下文 dev-storage 相关的配置：

```bash
kubectl config --kubeconfig=config-demo view --minify
```

###  1.8 设置 KUBECONFIG 环境变量
查看是否有名为 `KUBECONFIG` 的环境变量。 如有，保存 KUBECONFIG 环境变量当前的值，以便稍后恢复。 例如：

```bash
export KUBECONFIG_SAVED="$KUBECONFIG"
```
临时添加两条路径到 KUBECONFIG 环境变量中。例如：

```bash
export KUBECONFIG="${KUBECONFIG}:config-demo:config-demo-2"
```

在 `config-exercise` 目录中输入以下命令：

```bash
kubectl config view
```
输出config-demo与config-demo-2的上下文（context）：

```bash
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: ramp
    user: developer
  name: dev-ramp-up
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: scratch
    namespace: default
    user: experimenter
  name: exp-scratch
```

将 `$HOME/.kube/config` 追加到 KUBECONFIG 环境变量中

```bash
export KUBECONFIG="${KUBECONFIG}:$HOME/.kube/config"
```

清理
将 `KUBECONFIG` 环境变量还原为原始值。例如：

```bash
export KUBECONFIG="$KUBECONFIG_SAVED"
```

###  1.9 删除

 - 要删除用户，可以运行 `kubectl --kubeconfig=config-demo config unset users.<name>`
 - 要删除集群，可以运行 `kubectl --kubeconfig=config-demo config unset clusters.<name>`
 - 要删除上下文，可以运行 `kubectl --kubeconfig=config-demo config unset contexts.<name>`


##  2. 常用练习

```bash
kubectl config view # 显示合并的 kubeconfig 配置。

# 同时使用多个 kubeconfig 文件并查看合并的配置
KUBECONFIG=~/.kube/config:~/.kube/kubconfig2

kubectl config view

# 获取 e2e 用户的密码
kubectl config view -o jsonpath='{.users[?(@.name == "e2e")].user.password}'

kubectl config view -o jsonpath='{.users[].name}'    # 显示第一个用户
kubectl config view -o jsonpath='{.users[*].name}'   # 获取用户列表
kubectl config get-contexts                          # 显示上下文列表
kubectl config current-context                       # 展示当前所处的上下文
kubectl config use-context my-cluster-name           # 设置默认的上下文为 my-cluster-name

kubectl config set-cluster my-cluster-name           # 在 kubeconfig 中设置集群条目

# 在 kubeconfig 中配置代理服务器的 URL，以用于该客户端的请求
kubectl config set-cluster my-cluster-name --proxy-url=my-proxy-url

# 添加新的用户配置到 kubeconf 中，使用 basic auth 进行身份认证
kubectl config set-credentials kubeuser/foo.kubernetes.com --username=kubeuser --password=kubepassword

# 在指定上下文中持久性地保存名字空间，供所有后续 kubectl 命令使用
kubectl config set-context --current --namespace=ggckad-s2

# 使用特定的用户名和名字空间设置上下文
kubectl config set-context gce --user=cluster-admin --namespace=foo \
  && kubectl config use-context gce

kubectl config unset users.foo                       # 删除用户 foo

# 设置或显示 context / namespace 的短别名
# （仅适用于 bash 和 bash 兼容的 shell，在使用 kn 设置命名空间之前要先设置 current-context）
alias kx='f() { [ "$1" ] && kubectl config use-context $1 || kubectl config current-context ; } ; f'
alias kn='f() { [ "$1" ] && kubectl config set-context --current --namespace $1 || kubectl config view --minify | grep namespace | cut -d" " -f6 ; } ; f'
```

参考：

 - [kubectl 备忘单](https://kubernetes.io/zh-cn/docs/reference/kubectl/cheatsheet/)
 - [配置对多集群的访问](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)
 - [kubectl config 命令](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config)
