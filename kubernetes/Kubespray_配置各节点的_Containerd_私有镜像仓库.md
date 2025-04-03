
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/dd6ac3108ea94366a52442deee76984e.png)





在 Kubernetes 集群中，为了提高镜像拉取效率并保证安全性，我们常常配置私有镜像仓库。本文以使用 Kubespray 配置 Containerd 的私有镜像仓库为例，详细介绍操作步骤和注意事项。

##  1. 前提条件

1. 已部署一个 Kubernetes 集群，且集群通过 Kubespray 管理。
2. 已配置好私有镜像仓库（如 Harbor），本文以 `http://192.168.21.9` 为例，用户名为 `admin`，密码为 `Harbor12345`。
3. 在变更配置前确保不影响业务。

## 2. 注意事项

- **评估业务影响**：在更新 Containerd 配置前，需检查节点当前运行的 Pod 是否依赖于现有的镜像拉取方式。如果变更可能导致服务不可用，建议先调整工作负载策略（如设置 Pod 的镜像拉取策略为 `IfNotPresent` 或 `Never`）。
- **备份现有配置**：在更新任何配置前，备份相关文件以便回滚。

## 3. 配置步骤

### 3.1 确认 Containerd 当前配置

登录任意 Kubernetes 节点，查看 Containerd 配置文件（通常位于 `/etc/containerd/config.toml`）：

```
cat /etc/containerd/config.toml
```

检查 `[plugins."io.containerd.grpc.v1.cri".registry.mirrors]` 配置部分是否已有私有镜像仓库的配置。

### 3.2 备份配置文件

在所有节点执行以下命令，备份 Containerd 配置文件：

```
cp /etc/containerd/config.toml /etc/containerd/config.toml.bak
```

### 3.2 更新 Ansible 变量文件

在 Kubespray 的 `inventory` 目录下找到集群的变量文件，例如：

```
inventory/mycluster/group_vars/all/containerd.yml
```

添加或更新以下内容：

```
containerd_insecure_registries:
  - "192.168.21.9"
containerd_registry_mirrors:
  "192.168.21.9":
    endpoint:
      - "http://192.168.21.9"
containerd_registry_auths:
  "192.168.21.9":
    username: "admin"
    password: "Harbor12345"
```

此配置将：

- 设置 `192.168.21.9` 为不安全的镜像仓库（非 HTTPS）。
- 配置镜像加速器。
- 添加私有仓库的认证信息。

### 3.3 执行更新操作

运行 Kubespray 的 Ansible 脚本更新节点配置：

```
ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml --become --tags=containerd
```

此操作会重启 Containerd 并应用新的配置。

### 3.4 验证配置

在任意节点上，验证 Containerd 是否正确配置私有镜像仓库：

```
cat /etc/containerd/config.toml | grep -A 5 "registry.mirrors"
```

测试从私有镜像仓库拉取镜像：

```
crictl pull 192.168.21.9/library/test-image:latest
```

##  4. 更新前的业务影响评估

1. 检查现有节点上运行的 Pod：

```
kubectl get pods -o wide --all-namespaces
```

1. 检查 Pod 的镜像拉取策略：

```
kubectl get pods -n <namespace> -o json | jq '.items[].spec.containers[].imagePullPolicy'
```

1. 若发现业务关键 Pod 使用了 `Always` 策略，建议：
   - 手动将关键镜像预拉取到节点。
   - 临时更改镜像拉取策略为 `IfNotPresent`。

##  5. 总结

通过 Kubespray 配置 Containerd 的私有镜像仓库是一种高效的方法，能够保证集群内镜像拉取的安全性和速度。在实施前需要充分评估对业务的影响，并做好备份和验证工作。
