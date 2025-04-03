


# 离线安装 OpenShift 的完整指南
在企业级环境中，许多场景需要在隔离的网络中部署 OpenShift 集群。为此，准备离线安装介质是关键步骤。本文将详细介绍如何下载和准备 OpenShift 4.16 的离线安装介质，包括工具、镜像、安装文件等，确保您能够顺利完成离线环境中的 OpenShift 集群部署。

## 1. 前置准备
### 1.2 获取 RHEL ISO 镜像
在部署 OpenShift 之前，需要一台运行 Red Hat Enterprise Linux (RHEL) 的服务器作为基础。您可以通过以下链接下载对应版本的 RHEL ISO 镜像：

- [下载 RHEL](https://access.redhat.com/downloads/content/rhel)

### 1.2 安装必要工具
在 RHEL 环境中安装以下依赖工具，确保具备基本的运行环境：



```bash
sudo yum install -y wget tar
```

## 2. 下载所需工具
以下工具是部署 OpenShift 必备的组件。

### 2.1 下载 oc 客户端工具
oc 是 OpenShift 提供的命令行工具，用于管理集群资源。


```bash
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.16.10/openshift-client-linux-amd64-rhel9-4.16.10.tar.gz
tar -xvzf openshift-client-linux-amd64-rhel9-4.16.10.tar.gz -C /usr/local/bin
```

### 2.2 下载 oc-mirror 工具
oc-mirror 用于从公开注册表下载和镜像 OpenShift 的安装介质。

```bash
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.16.10/oc-mirror.rhel9.tar.gz
tar -xvzf oc-mirror.rhel9.tar.gz -C /usr/local/bin
```

备用：

```bash
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/oc-mirror.tar.gz
```

### 3.3 下载 openshift-install
openshift-install 是 OpenShift 的集群安装工具，负责集群的初始化部署。


```bash
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.16.10/openshift-install-rhel9-amd64.tar.gz
tar -xvzf openshift-install-rhel9-amd64.tar.gz -C /usr/local/bin
```

备用镜像：

```bash
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
```

### 2.4 下载 opm
opm 是用于管理 Operator 镜像的命令行工具。

```bash
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.16.10/opm-linux-rhel9-4.16.10.tar.gz
tar -xvzf opm-linux-rhel9-4.16.10.tar.gz -C /usr/local/bin
```

## 3. 下载部署所需镜像
在离线环境中，需要提前将 OpenShift 所有必要的镜像下载到本地镜像仓库。以下是完整的操作步骤。

1. 配置镜像下载配置文件
创建一个配置文件 ImageSetConfig-ocp4.16.10.yaml，内容如下：

```bash
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v1alpha2
storageConfig:
  registry:
    imageURL: registry.ocp.local:8443/mirror/oc-mirror-metadata # 本地镜像仓库地址
    skipTLS: true
mirror:
  platform:
    channels:
      - name: stable-4.16    # OpenShift 版本
        minVersion: 4.16.10  # 最低版本
        maxVersion: 4.16.10  # 最高版本
        shortestPath: true
        type: ocp
    graph: true   # 如果需要通过 OSUS operator 更新，建议启用
  additionalImages:
    - name: registry.redhat.io/ubi8/ubi:latest
  helm: {}
```

2. 执行镜像下载命令
运行以下命令，将镜像拉取到本地仓库：

```bash
oc mirror --config=ImageSetConfig-ocp4.16.10.yaml docker://registry.ocp.local:8443
```

该命令将会从官方注册表下载所有指定的镜像，并推送到本地仓库 registry.ocp.local:8443。

3. 验证镜像下载结果
下载完成后，可以验证本地仓库是否包含所需镜像：

```bash
curl -u <username>:<password> -X GET https://registry.ocp.local:8443/v2/_catalog
```

如果返回的结果包含所有 OpenShift 所需的镜像，则镜像同步完成。

## 4. 小结
本文详细介绍了离线下载 OpenShift 安装介质的完整步骤，包括工具下载、镜像同步以及本地仓库的配置。通过上述操作，您可以在隔离网络中成功部署 OpenShift 4.16 集群。

如在部署中遇到问题，可以根据以下几点排查：

确保镜像仓库服务正常运行，并允许从集群节点访问。
验证下载的工具版本与 OpenShift 版本是否匹配。
检查镜像配置文件是否正确。
