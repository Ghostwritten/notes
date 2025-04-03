


---


✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>



 - [docker 命令](https://blog.csdn.net/xixihahalelehehe/article/details/123378401?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681086016780271517687%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681086016780271517687&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-123378401.nonecase&utm_term=podman&spm=1018.2226.3001.4450)
 - [podman 命令](https://blog.csdn.net/xixihahalelehehe/article/details/121611523?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681086016780271517687%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681086016780271517687&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-121611523.nonecase&utm_term=podman&spm=1018.2226.3001.4450)
 -  [crictl 命令](https://blog.csdn.net/xixihahalelehehe/article/details/116591151?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681092916780271596159%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681092916780271596159&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-13-116591151.nonecase&utm_term=%E5%91%BD%E4%BB%A4&spm=1018.2226.3001.4450)
 - [operator-sdk 命令](https://blog.csdn.net/xixihahalelehehe/article/details/112024963?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164681502916780255218754%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164681502916780255218754&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-112024963.nonecase&utm_term=operator-sdk&spm=1018.2226.3001.4450)
 - [operator-sdk官网](https://sdk.operatorframework.io/)

---
## 1. 主题

 - Operator SDK
 - Build Preparation
 - Go-Based Operator
 - Ansible® Operator
 - Helm Operator
 - Operator Capabilities


##  2. Operator SDK

 - 使用Kubernetes API库用编程语言编写的Operator
 - Default: Go  可能的其他编程语言：java
 - 需要编写整个控制器逻辑
 - 需要了解informers、shared informers、对象缓存的工作队列和事件处理


| Feature | Use |
|--|--|
|高级api和抽象 | 更直观地编写运算逻辑 |
|脚手架和代码生成工具 |快速启动新项目|
|扩展|覆盖常用的Operator用例|


### 2.1 安装
1. 安装容器运行时和构建环境

 - docker
 - buildah

2. 从Operator SDK GitHub主页下载Operator SDK可执行文件
3. 将operator-sdk二进制可执行文件移动到PATH

参考：
完整的安装说明：[Operator SDK home page](https://sdk.operatorframework.io/)

Example Installation

```c
$ sudo wget https://github.com/operator-framework/operator-sdk/releases/download/v0.17.0/operator-sdk-v0.17.0-x86_64-linux-gnu -O /usr/local/bin/operator-sdk
$ sudo chmod +x /usr/local/bin/operator-sdk
```
`operator-sdk` executable located in `/usr/local/bin`

##  2.2 Build 准备
安装工具链

 - 构建Operator容器映像所需的容器运行时
 - Podman和其他工具优于Docker
 - RHEL 7.6及更高版本:普通用户可以构建Operator容器映像
 - Install build tools:

```bash
 sudo yum -y install podman buildah skopeo
```

##  2.3 Go-Based Operator
创建新的基于go的项目

```bash
operator-sdk new app-operator \
  --api-version=app.example.com/v1alpha1 \
  --kind=App
```
Creates:

 - Gopkg.lock, Gopkg.toml
 - cmd, config, deploy, pkg
 - tmp, vendor, version

Next step: Implement

参考：
[Operator SDK home page](https://sdk.operatorframework.io/)

##  3. Ansible Operator
###  3.1 概述

 - 提供 base Operator image
 - 执行Ansible Playbook或role
 -  - 确认对象在那里
 -  - 重新创建/删除对象发生了改变
 -  - Playbook or role必须是幂等的
- 允许在OpenShift中创建，但不允许删除对象
- 向所有创建的对象添加`ownerReferences`值

参考：

 - [Ansible User Guide for Operator SDK](https://github.com/operator-framework/operator-sdk/blob/v0.17.x/doc/ansible/user-guide.md)

### 3.2 Workflow

 - Ansible Operator SDK创建所需的基本结构
 - 更新Ansible内容
 - 要在`role/playbook`中创建对象，使用`k8s Ansible`模块
 - 使用提供的Dockerfile构建 Operator容器image
 - Push to image registry (e.g., Quay.io)
 - Update created YAML files

### 3.3 创建新项目
命令帮助
```bash
operator-sdk --help
```
为集群范围的Gitea Operator创建新的shell项目:

```bash
operator-sdk new gitea-operator \
  --api-version=gpte.opentlc.com/v1alpha1 \
  --kind=Gitea --type=ansible \
  --generate-playbook
```
###  3.4 项目概况
watches.yaml:

```bash
---
 - version: v1alpha1
  group: gpte.opentlc.com
  kind: Gitea
  playbook: /opt/ansible/playbook.yml
```
 - Ansible Operator的配置文件
 - Use playbook or role

生成Dockerfile
如果不需要，删除复制`playbook/roles`

```bash
FROM quay.io/operator-framework/ansible-operator:v0.17.0

COPY requirements.yml ${HOME}/requirements.yml
RUN ansible-galaxy collection install -r ${HOME}/requirements.yml \
 && chmod -R ug+rwx ${HOME}/.ansible

COPY watches.yaml ${HOME}/watches.yaml

COPY roles/ ${HOME}/roles/
COPY playbook.yml ${HOME}/playbook.yml
```
### 3.5 YAML Files
其他创建文件:

```bash
ls -R build deploy
```

Sample output:

```bash
build:
Dockerfile  test-framework/

build/test-framework:
ansible-test.sh  Dockerfile

deploy:
crds/  operator.yaml  role_binding.yaml  role.yaml  service_account.yaml

deploy/crds:
gpte.opentlc.com_giteas_crd.yaml  gpte.opentlc.com_v1alpha1_gitea_cr.yaml
```
###  3.6 Build Container Image
构建Operator容器镜像，使用`Operator -sdk`

 - 有正确名称和版本的tag
 - 指定想要的`builder` (docker, podman, buildah)
 - 包括`registry`以接收推送

```bash
operator-sdk build quay.io/wkulhanek/gitea-operator:v0.0.1 --image-builder podman
```

###  3.7 Push to Registry

 - 将构建的`image`推送到容器`registry`
 - Use `podman push`, skopeo, or `docker push` (when using Docker):

```bash
$ podman push quay.io/wkulhanek/gitea-operator:v0.0.1
```
###  3.8 Update Deployment

 - 更新`deploy/operator.yaml`中的占位符。实际容器图像位置的
 - 更新`imagePullPolicy`以匹配您的首选项e.g. Always for `development`:

```bash
[...]
        - name: gitea-operator
          # Replace this with the built image name
          image: quay.io/wkulhanek/gitea-operator:v0.0.1
          ports:
          - containerPort: 60000
            name: metrics
          imagePullPolicy: Always
[...]
```

###  3.9 安装基于ansible的操作系统

 - 像部署任何其他Operator一样部署基于ansible的Operator
 - 创建CRD
 - 需要 `cluster-admin`
 - Create project
 - Create service account
 - Create role or ClusterRole
 - Create RoleBinding or ClusterRoleBinding
 - Create deployment

###  3.10 集群测试外
一旦创建了所有先决条件，SDK就可以在集群外运行Operator了

替换deploy/operator.yaml的部署

仍然关注cr创建的事件

避免每次都需要重新构建/推送/重新部署容器映像

要在本地运行Operator:

登录OpenShift，在“Operator”目录下执行命令:

```bash
operator-sdk up local
```
##  4. Helm Operator
### 4.1 概述

 - 将 Helm charts改变Operator
 - 集群上不需要Tiller
 - Helm charts 需要openshift兼容
 - 不以root用户运行
 - 没有其他的先决条件

参考：
[Helm User Guide for Operator SDK](https://github.com/operator-framework/operator-sdk/blob/v0.17.x/doc/helm/user-guide.md)

###  4.2 设置  Helm Operator

创建式helm项目

```bash
operator-sdk new nginx-operator --api-version=example.com/v1alpha1 --kind=Nginx --type=helm
```
参考：

 - [Helm User Guide for Operator SDK](https://github.com/operator-framework/operator-sdk/blob/v0.17.x/doc/helm/user-guide.md)


##  5. Operator 功能
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8091a466eb8f25521e5b8f9a26a84ec2.png)

