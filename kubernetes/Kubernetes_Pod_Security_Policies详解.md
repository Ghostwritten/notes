

## 1. 版本
功能状态： `Kubernetes v1.21 [deprecated]`
`PodSecurityPolicy` 自 `Kubernetes v1.21` 起已弃用，并将在 v1.25 中删除。

Pod 安全策略启用对 Pod 创建和更新的细粒度授权

## 2. 介绍
PodSecurityPolicy对象定义一组条件，一个pod必须以被接受进入系统，以及用于相关字段默认值运行。它们允许管理员控制以下内容：
| 运行特权容器                    | privileged                                                           |
|---------------------------|----------------------------------------------------------------------|
| 主机命名空间的使用                 | hostPID, hostIPC                                                     |
| 主机网络和端口的使用                | hostNetwork, hostPorts                                               |
| 卷类型的使用                    | volumes                                                              |
| 主机文件系统的使用                 | allowedHostPaths                                                     |
| 允许特定的 FlexVolume 驱动程序     | allowedFlexVolumes                                                   |
| 分配一个拥有 Pod 卷的 FSGroup     | fsGroup                                                              |
| 需要使用只读根文件系统               | readOnlyRootFilesystem                                               |
| 容器的用户和组 ID                | runAsUser, runAsGroup,supplementalGroups                             |
| 限制升级到 root 权限             | allowPrivilegeEscalation, defaultAllowPrivilegeEscalation            |
| Linux 功能                  | defaultAddCapabilities, requiredDropCapabilities,allowedCapabilities |
| 容器的 SELinux 上下文           | seLinux                                                              |
| 容器的 Allowed Proc Mount 类型 | allowedProcMountTypes                                                |
| 容器使用的 AppArmor 配置文件       | 注释                                                                   |
| 容器使用的 seccomp 配置文件        | 注释                                                                   |
| 容器使用的 sysctl 配置文件         | forbiddenSysctls,allowedUnsafeSysctls                                |


Pod 安全策略控制作为可选（但推荐）的 准入控制器实现。PodSecurityPolicies 是通过启用准入控制器来强制执行的，但是在没有授权任何策略的情况下这样做会阻止在集群中创建任何 pod。

由于 Pod 安全策略 API ( policy/v1beta1/podsecuritypolicy) 是独立于准入控制器启用的，对于现有集群，建议在启用准入控制器之前添加和授权策略。

## 3. 授权政策
创建 PodSecurityPolicy 资源时，它什么都不做。为了使用它，请求用户或目标 Pod 的服务帐户必须被授权使用该策略，方法是允许use策略上的动词。

大多数 Kubernetes pod 不是由用户直接创建的。相反，它们通常作为Deployment、 ReplicaSet或其他模板化控制器的一部分通过控制器管理器间接创建 。授予控制器访问策略的权限将授予该控制器创建的所有pod 的访问权限，因此授权策略的首选方法是授予对 pod 服务帐户的访问权限
### 3.1 通过 RBAC
RBAC是一种标准的 Kubernetes 授权模式，可以很容易地用于授权使用策略。

首先， a RoleorClusterRole需要授予use对所需策略的访问权限。授予访问权限的规则如下所示：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: <role name>
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - <list of policies to authorize>
```
然后(Cluster)Role绑定到授权用户：

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: <binding name>
roleRef:
  kind: ClusterRole
  name: <role name>
  apiGroup: rbac.authorization.k8s.io
subjects:
# Authorize specific service accounts:
- kind: ServiceAccount
  name: <authorized service account name>
  namespace: <authorized pod namespace>
# Authorize specific users (not recommended):
- kind: User
  apiGroup: rbac.authorization.k8s.io
  name: <authorized user name>
```
如果使用 `a RoleBinding`（不是 a ClusterRoleBinding），它只会授予在与绑定相同的命名空间中运行的 pod 的使用权。这可以与系统组配对以授予对在命名空间中运行的所有 pod 的访问权限：
