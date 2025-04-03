# Jenkins Pipeline & Kubernetes 如何创建 pod
![](https://i-blog.csdnimg.cn/blog_migrate/88f6cebf65caac96adb5d5da4880edf0.png)





关于`kubernetes & Jenkins` 部署可以参考：[minikube & helm 安装 jenkins](https://ghostwritten.blog.csdn.net/article/details/128166331)

##  1. 前言
在 [DevOps](https://www.atlassian.com/zh/devops) 的世界里，自动化是主要目标之一。针对 `CI/CD` 的最著名的开源工具之一就是自动化服务器 [Jenkins](https://www.jenkins.io/)。从简单的 CI 服务器到完整的 CD 集线器，Jenkins 都可以处理。

- `CI`：持续集成（continuous integration）是在源代码变更后自动检测、拉取、构建和（在大多数情况下）进行单元测试的过程。目标是快速确保开发人员新提交的变更是好的，并且适合在代码库中进一步使用

- `CD`：持续交付（continuous delivery）通常是指整个流程链（管道），它自动监测源代码变更并通过构建、测试、打包和相关操作运行它们以生成可部署的版本，基本上没有任何人为干预。持续交付在软件开发过程中的目标是自动化、效率、可靠性、可重复性和质量保障（通过持续测试）。
- `CD`: 持续部署（continuous deployment）是指能够自动提供持续交付管道中发布版本给最终用户使用的想法。根据用户的安装方式，可能是在云环境中自动部署、app 升级（如手机上的应用程序）、更新网站或只更新可用版本列表。
- `Pipeline`: 将源代码转换为可发布产品的多个不同的 任务(task)和 作业(job)通常串联成一个软件“管道”，一个自动流程成功完成后会启动管道中的下一个流程。这些管道有许多不同的叫法，例如持续交付管道、部署管道和软件开发管道。

![](https://i-blog.csdnimg.cn/blog_migrate/266c4482ae63d282d8c98fb9607e3f03.png)

<font color=#FFA500 size=3 face="楷体">"下面我们将实践如何利用 `Jenkins` 强大的 `CI/CD` 特性来练习如何部署 `kubernetes` 应用作为开始。"</font>
 


## 2. Jenkins 插件

### 2.1 安装 Kubernets Plugin
![](https://i-blog.csdnimg.cn/blog_migrate/85ffbc70017c5170fecbac9e534b8dc6.png)
![](https://i-blog.csdnimg.cn/blog_migrate/2e509990a6eb183e1ec588bf2ada0260.png)

### 2.2 安装 Docker Plugin
![](https://i-blog.csdnimg.cn/blog_migrate/58e89545bd2cab6e30beafde0c952506.png)

### 2.3 安装 Git Plugin
![](https://i-blog.csdnimg.cn/blog_migrate/a1c38fca3f29826b19af1835b304b241.png)
![](https://i-blog.csdnimg.cn/blog_migrate/deec42052e83b8d56b9de42118878c4c.png)

##  3. Jenkins 配置 kubernetes credentials


![](https://i-blog.csdnimg.cn/blog_migrate/e9f56768eaac16bf018256a0fca2fc18.png)

![](https://i-blog.csdnimg.cn/blog_migrate/529414cdb0f4b0826135ee2dcdc98c82.png)
![](https://i-blog.csdnimg.cn/blog_migrate/e9ef4a341482abeede3f2533b12aef6d.png)

![](https://i-blog.csdnimg.cn/blog_migrate/93b399ac996bdaa5cd39c7c1a5730db7.png)
![](https://i-blog.csdnimg.cn/blog_migrate/daeaf679c68d2ff5c4e75421875ac75a.png)
获取 `token`

```bash
 kubectl get secrets -n jenkins jenkins-token-6r26g -oyaml
apiVersion: v1
data:
  ca.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCakNDQWU2Z0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwdGFXNXAKYTNWaVpVTkJNQjRYRFRJeU1URXlPVEEyTURJd09Gb1hEVE15TVRFeU56QTJNREl3T0Zvd0ZURVRNQkVHQTFVRQpBeE1LYldsdWFXdDFZbVZEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTU8xCkpIMER5VU5DeGh2dHdtR05EaGxORTZFTXJWMzl0N1dmUjJmMTJEM3U4SVlpckFhdFBNb1RYZThpTDR4NXl5ckoKaUhRdWtIMXAzcjhqS0E4WVpaa2cwN3FIOE1mYXB2dG9qQjQ3QUROanBLYUNVcXh6UFlvY3l0VlU3UDA0dDhMVQpyRTZFTG9qcGlWcWNEdzZSakhEQ3p2R3NFU1NvTUIyZTlxVXN2dU9kMmdlMFVwMnJKdjRmTTY3aEdJN3FIYkJ3CjVFbE5tMzcySytxOS9nUFBtSW1kQU94Y2xFOENTcy9aYWxuV1AzcWdLbGVoNnQxbkFscnhpbXpaSVdkYnZvMzYKemxTRmwwaHlrZGNWL0RUdndaNFYvMkR1OVVtbWpsWFJ5cWdOYXF1R2lTYXVMRjhWRkpYWnROWmhSR2NWZW1tNQpRQ0gyWWFEN3lzMmJOYlM3YUtzQ0F3RUFBYU5oTUY4d0RnWURWUjBQQVFIL0JBUURBZ0trTUIwR0ExVWRKUVFXCk1CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC9NQjBHQTFVZERnUVcKQkJSVXg5bTFqVlIrR1llNXJMalZhRmpxaXljbjZEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUF3M3JpRkY1QgozTXZSZmFlRnB0d3pRRzYzVEZncVoxN3luaDBSV2xrUkM3M3c0L1BqczhUUmhXRFNYTy94elNROTNJNjNlaHkwCldIOTJsMDJhN3NNTXVvY1YrRG8vTUlpVmI3bVA2VmtKbjF2Vm43cE4zM2c5dDJFWkY2Yis0Q0JUUUo5YXlodGwKMHo5Y3hwMXVaMEt4SUZ1bzJuY3lPREFEZ293T3h0WHJKZ1h5ckE5MldFbi91NUl5VnJ1Q2ZlWjk0YTMwN3NVUAplMXpnTlBBam8xUHdTQVY4bE5tRkpZbG9FVzNhRkE1MDU3STZYZy81dEg0QzJ2c3NkNVFxS2R4RHM1UU5QK2VpClhZblp2S2dTTmJSVUtyL1NIM1Q5NkQ0azIrQXpsWEhBYVRwYk9uQURzYmt3YTYwcHUvWmJsa0dLajhFVE9KL2UKRldleGYyUmtzbkVWN1E9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  namespace: amVua2lucw==
  token: ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNkltdHpSM2xwUTJrd2FUUmxiR3gyYkMwMFIyVndOR0ZUZEZsVlVXUTRVaTFNUTBnMU9ISlBiRkJOWldNaWZRLmV5SnBjM01pT2lKcmRXSmxjbTVsZEdWekwzTmxjblpwWTJWaFkyTnZkVzUwSWl3aWEzVmlaWEp1WlhSbGN5NXBieTl6WlhKMmFXTmxZV05qYjNWdWRDOXVZVzFsYzNCaFkyVWlPaUpxWlc1cmFXNXpJaXdpYTNWaVpYSnVaWFJsY3k1cGJ5OXpaWEoyYVdObFlXTmpiM1Z1ZEM5elpXTnlaWFF1Ym1GdFpTSTZJbXBsYm10cGJuTXRkRzlyWlc0dE5uSXlObWNpTENKcmRXSmxjbTVsZEdWekxtbHZMM05sY25acFkyVmhZMk52ZFc1MEwzTmxjblpwWTJVdFlXTmpiM1Z1ZEM1dVlXMWxJam9pYW1WdWEybHVjeUlzSW10MVltVnlibVYwWlhNdWFXOHZjMlZ5ZG1salpXRmpZMjkxYm5RdmMyVnlkbWxqWlMxaFkyTnZkVzUwTG5WcFpDSTZJak01TnpjMU0yUmtMVEJtTTJZdE5ERmxNaTFoTnpOakxUaGhOVEV6TldRMlptRmhZU0lzSW5OMVlpSTZJbk41YzNSbGJUcHpaWEoyYVdObFlXTmpiM1Z1ZERwcVpXNXJhVzV6T21wbGJtdHBibk1pZlEuamx6SjV1bmRlZ3ZUMzVmVmViY3JsWnUxa09talAwTGpLenhsb2sxTHhINVowcm9jTmVsSmVJMXM4NWtnT1JXZGNuanFaTlg3VHpxano1eml1OS1peGZuY1UtTVRicTlRcTVMcnMyd1c1cGRTZTFwSlpWcmx5X2pGN2tOdXFSWENZZkdWVUFwbGFELXNES2ZJdFhmdEJ2dHpvWTgzV0QxdEdZMkJkSzRKM3NGdDRIdmw2YUVuUlZFejJ5WlRBTVBKZEJ1NUI4NjJCUnFOMEFicG13UFByM3FtellaZGZVQzNzdlBIVE9BWGxxUUpUcHRQbDVmMS05dEtKZUlGUGNLY1F5WTZSc0RXbUFfRW1sTHV6MWJWRkpxV2pxWnVqa3cyTndYeXhvS1VMNjBhVm9Lc2dQTVFpTU44TEg4Z2s0bTg5STQ5VjVvb2NpX3N3VTNYVy05cjNR
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: jenkins
    kubernetes.io/service-account.uid: 397753dd-0f3f-41e2-a73c-8a5135d6faaa
  creationTimestamp: "2022-12-03T13:49:22Z"
  name: jenkins-token-6r26g
  namespace: jenkins
  resourceVersion: "199550"
  uid: 0b896aba-1810-436a-9eb8-dd0502e9e79f
type: kubernetes.io/service-account-token
```
复制`token`内容至`secret`
![](https://i-blog.csdnimg.cn/blog_migrate/d7ccb9d59f630145c443cb08f50e143b.png)
![](https://i-blog.csdnimg.cn/blog_migrate/b3fe7350256b3e43f9be0c7426b6cbcc.png)
- 具体可[官方添加 credentials 步骤](https://www.jenkins.io/doc/book/using/using-credentials/)


## 4. Jenkins 连接 minikube 集群
我们需要配置 `manage Nodes and Clouds`

![](https://i-blog.csdnimg.cn/blog_migrate/648eda825de3fba59598c580a9da195e.png)
![](https://i-blog.csdnimg.cn/blog_migrate/a3549190d416fd13b78fbe95195adeb5.png)
![](https://i-blog.csdnimg.cn/blog_migrate/ce81816e7487c75a993e38fe5e7e9868.png)
![](https://i-blog.csdnimg.cn/blog_migrate/221036164d07dd88e335b4a0076ea86d.png)
![](https://i-blog.csdnimg.cn/blog_migrate/03026e64f28abb87d6267827a896b2a7.png)


- `name`: 定义集群名称即可。
- `kubernetes credentials` 选择下拉刚刚创建的 `minikube2`
- `Kubernetes URL` 与 通过命令 `kubectl  config view`获取
- 填完点击测试一下，再保存。
- 确保 `websocket` 勾选，否则，pod内无法运行`jnlp`代理。

```bash
$ kubectl  config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /root/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Wed, 30 Nov 2022 14:20:59 CST
        provider: minikube.sigs.k8s.io
        version: v1.28.0
      name: cluster_info
    server: https://192.168.10.26:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Wed, 30 Nov 2022 14:20:59 CST
        provider: minikube.sigs.k8s.io
        version: v1.28.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /root/.minikube/profiles/minikube/client.crt
    client-key: /root/.minikube/profiles/minikube/client.key
```
## 5. 配置参数说明
### 5.1 Pod template 参数 
- `cloud` ：Jenkins设置中定义的云的名称。默认为`kubernetes`
- `name`： pod名称. 
- `namespace` ：pod 命名空间.
- `label` ：节点标签。 这就是在通过节点步骤请求代理时可以引用 pod 模板的方式。 在管道中，建议省略此字段并依赖生成的标签，该标签可以使用 podTemplate 块中定义的 POD_LABEL 变量引用。
yaml Pod 的 yaml 表示，以允许设置任何不支持的值作为字段
- `yamlMergeStrategy` ：merge() or override(). 控制 yaml 定义是否覆盖或与从使用 inheritFrom 声明的 pod 模板继承的 yaml 定义合并。 默认为 override() （出于向后兼容性原因）。
- `containers`： container templates 部分
- `serviceAccount` ：pod 服务帐户.
- `nodeSelector` ：pod的节点选择器.
- `nodeUsageMode`： 要么NORMAL要么EXCLUSIVE，该参数控制的是只调度标签表达式匹配的作业还是尽可能使用节点.
- `volumes`： 挂载持久存储卷
  - `configMapVolume` : 挂载ConfigMap，只读.
  - `dynamicPVC()` : 动态管理的持久卷 pvc ，它与 pod 同时被删除.
  - `emptyDirVolume` (default): 空目录
  - `hostPathVolume()` : 挂载主机目录
  - `nfsVolume()` : 挂载NFS目录
  - `persistentVolumeClaim()` : 绑定pvc.
  - `secretVolume` : 挂载 secret，只读，适用于证书、用户密码.
- `envVars`： 应用于所有容器的环境变量.
  - `envVar`：个环境变量，其值是内联定义的.
  - `secretEnvVar`： 通过secret对象定义一个环境变量.
- `imagePullSecrets`： 提取 secret 名称, 从私有 Docker 仓库中提取镜像.
- `annotations`：pod注释
- `inheritFrom`： 要继承的一个或多个 pod 模板的列表
- `slaveConnectTimeout` ：agent 在线超时秒数
- `podRetention`： 控制保留代理 pod 的行为。 可以是 '`never()`'、'`onFailure()'`、'`always()`' 或 '`default()`' - 如果为空，将默认在 `activeDeadlineSeconds` 过后删除 pod。
- `activeDeadlineSeconds`： 如果 podRetention 设置为 never() 或 onFailure()，则 pod 将在截止日期过后删除。
- `idleMinutes`： 允许 pod 保持活动状态以供重用，直到自上一个步骤执行后配置的分钟数过去。
- `showRawYaml`：启用或禁用原始pod清单（manifest）的输出。默认为true
- `runAsUser`： 定义运行pod中所有容器的用户ID.
- `runAsGroup`： 定义在pod中运行所有容器的组ID.
- `hostNetwork`： 主机网络.
- `workspaceVolume`： 定义workspace卷的类型.
  - `dynamicPVC()` ：挂载动态管理的pvc，会与pod一同被删除.
  - `emptyDirWorkspaceVolume` (default): 定义主机上分配的空目录
  - `hostPathWorkspaceVolume()` : 挂载主机目录
  - `nfsWorkspaceVolume()` : 挂载nfs volume
  - `persistentVolumeClaimWorkspaceVolume()` : 绑定pvc.


### 5.2 Container template 参数
`Container template`可通过用户界面与pipeline配置。
- `name`： 容器名字.
- `image` ：镜像名称.
- `envVars` ：应用于容器的环境变量(补充和覆盖在pod已设置的环境变量).
  - `envVar`： 一个环境变量，其值是内联定义的.
  - `secretEnvVar` ：通过secret对象定义一个环境变量..
- `command`： 容器将执行的命令。 将覆盖 Docker 入口点。常用命令：sleep.
- `args` ：传递给命令的参数。例如：99999999 .
- `ttyEnabled`： 标志，以标记`tty`应该启用.
- `livenessProbe` ：探针（不支持 - httpGet liveness probes）
- `ports`： 暴露容器上的端口
- `alwaysPullImage`： 容器将在启动时拉取镜像.
- `runAsUser`： 定义运行容器的用户ID.
- `runAsGroup`： 定义运行容器的组ID.

## 6. Jenkins Piepline 部署 pod 实例

### 6.1 创建一个简单 pod
![](https://i-blog.csdnimg.cn/blog_migrate/e9536b3f4ccc95b9ecf8d585909a3c55.png)
![](https://i-blog.csdnimg.cn/blog_migrate/1f1260773c21a64717d101c639448a4d.png)
创建一个 包含 `docker` 容器的 `pod` ，并且带有`build`标签。创建完后，使用`docker version`命令执行查看容器内`docker` 的版本信息。
![](https://i-blog.csdnimg.cn/blog_migrate/c5034b58241f2ff2cad97d0914db43e4.png)
`Pipeline script`：

```bash
podTemplate(label: 'build', containers: [
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]
  ) {
    node('build') {
      container('docker') {
        sh 'docker version'
      }        
    }  
  }
```

点击执行“`build`”
![](https://i-blog.csdnimg.cn/blog_migrate/7369904d160a30e1e91f6fc698d24c66.png)
构建成功。
![](https://i-blog.csdnimg.cn/blog_migrate/eb38930aa4eead657368697ba9b701ff.png)
![](https://i-blog.csdnimg.cn/blog_migrate/baae85fa3537839b3ad8bf5f4d7d57b2.png)
`minikube` 集群查看部署结果：
```bash
$ k get pods -n jenkins
NAME                READY   STATUS    RESTARTS   AGE
build-cb6cf-twpw7   2/2     Running   0          6s
```
### 6.2 pod name 变化

创建pod，编写`podTemplate`，通过`yaml` 定义 pod内容。

```bash
podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    metadata:
      labels: 
        some-label: some-label-value
    spec:
      containers:
      - name: busybox
        image: busybox
        command:
        - sleep
        args:
        - 99d
    ''') {
    node(POD_LABEL) {
      container('busybox') {
        echo POD_CONTAINER // displays 'busybox'
        sh 'hostname'
      }
    }
}
```
第一次构建部署输出：

```bash
$ k get pods -n jenkins | grep busybox
busybox-1-zdnpq-mwqtd-0vfg0   2/2     Running   0          19s
```
- `busybox` pod前缀默认是项目名字
-  `1` 代表第一次构建
- `zdnpq-mwqtd-0vfg0`是随机码
- 
当我们第二次构建，pod 名字会发生变化，保持pod名字的唯一性。

```bash
$  k get pods -n jenkins | grep busybox
busybox-2-c7blp-840k1-s5c5d   2/2     Running   0          20s
```
我们尝试修改项目名称为`busybox2`
![](https://i-blog.csdnimg.cn/blog_migrate/81c54f24247744de6e65d6d736bc656a.png)
输出结果
```bash
$ k get pods -n jenkins | grep busybox
busybox2-3-2fc4l-8z8dp-2t0c7   2/2     Running   0          7s
```
原来的`busybox`变成`busybox2`，并且表明第三次构建。

我们尝试添加定义pod的`name`为`busybox3`

```bash
podTemplate(name: "busybox3",yaml: '''
    apiVersion: v1
    kind: Pod
    metadata:
      labels: 
        some-label: some-label-value
    spec:
      containers:
      - name: busybox
        image: busybox
        command:
        - sleep
        args:
        - 99d
    ''') {
    node(POD_LABEL) {
      container('busybox') {
        echo POD_CONTAINER // displays 'busybox'
        sh 'hostname'
      }
    }
}
```
当我们自定义pod 的`name`，构建次数会取消，项目名字不再是pod的前缀。
```bash
k get pods -n jenkins | grep busybox
busybox3-lzhfl-7hflx   2/2     Running   0          20s
```
### 6.3 指定 namespace
然后，我们尝试换一个已创建好的名字为`one`的`namespace`进行构建。

```bash
podTemplate(name: "busybox4",
    namespace: "one",
    yaml: '''
    ....
```

```bash
$ k get pods -n one  | grep busybox
busybox4-fxd38-5vpwf   2/2     Running   0          24s
```
### 6.4 volumes 挂载
挂载本地 `/var/run/docker.sock` 文件
```bash
podTemplate(name: "busybox5",
    namespace: "one",
    yaml: '''
    apiVersion: v1
    kind: Pod
    metadata:
      labels: 
        some-label: some-label-value
    spec:
      containers:
      - name: busybox
        image: busybox
        command:
        - sleep
        args:
        - 99d
    ''',
      volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
  ]
    ) {
    node(POD_LABEL) {
      container('busybox') {
        echo POD_CONTAINER // displays 'busybox'
        sh 'hostname'
      }
    }
}
```
### 6.5 Liveness Probe 探针

```bash
podTemplate(name: "busybox6",namespace: "one",
containers: [
containerTemplate(name: 'busybox', image: 'busybox', command: 'sleep', args: '99d',
                  livenessProbe: containerLivenessProbe(execArgs: 'ping 127.0.0.1', initialDelaySeconds: 30, timeoutSeconds: 1, failureThreshold: 3, periodSeconds: 10, successThreshold: 1)
)]
){
    node(POD_LABEL) {
      container('busybox') {
        echo POD_CONTAINER // displays 'busybox'
        sh 'hostname'
      }
    }
}
```
- 更多kubernetes探针内容：[Configure Liveness, Readiness and Startup Probes](https://ghostwritten.blog.csdn.net/article/details/108561740)

### 6.6 创建 pod 内多容器

```bash
podTemplate(name: "build1",namespace: "one",containers: [
    containerTemplate(name: 'maven', image: 'maven:3.8.1-jdk-8', command: 'sleep', args: '99d'),
    containerTemplate(name: 'golang', image: 'golang:1.16.5', command: 'sleep', args: '99d')
  ]) {

    node(POD_LABEL) {
        stage('Get a Maven project') {
            git 'https://github.com/jenkinsci/kubernetes-plugin.git'
            container('maven') {
                stage('Build a Maven project') {
                    sh 'mvn -B -ntp clean install'
                }
            }
        }

        stage('Get a Golang project') {
            git url: 'https://github.com/hashicorp/terraform.git', branch: 'main'
            container('golang') {
                stage('Build a Go project') {
                    sh '''
                    mkdir -p /go/src/github.com/hashicorp
                    ln -s `pwd` /go/src/github.com/hashicorp/terraform
                    cd /go/src/github.com/hashicorp/terraform && make
                    '''
                }
            }
        }

    }
}
```
查看创建的 pod

```bash
$ kubectl get pods -n one
NAME                 READY   STATUS    RESTARTS   AGE
build1-qc5pr-679mv   3/3     Running   0          17m
#三个容器
$ kubectl get pods -n one -o jsonpath="{.items[*].spec.containers[*].name}" | tr -s '[[:space:]]' '\n' |sort |uniq
golang
jnlp
maven
```

> 注意：`jnlp`容器是Jenkins代理,并且命令默认将在运行Jenkins代理的jnlp容器中执行。(jnlp的名称是历史的，为了兼容而保留。)，例如：

```bash
podTemplate {
    node(POD_LABEL) {
        stage('Run shell') {
            sh 'echo hello world'
        }
    }
}
```
我们还可以通过`yaml`定义格式：

```bash
podTemplate(name: "build1",namespace: "one",yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: maven
        image: maven:3.8.1-jdk-8
        command:
        - sleep
        args:
        - 99d
      - name: golang
        image: golang:1.16.5
        command:
        - sleep
        args:
        - 99d
''') {
  node(POD_LABEL) {
    stage('Get a Maven project') {
      git 'https://github.com/jenkinsci/kubernetes-plugin.git'
      container('maven') {
        stage('Build a Maven project') {
          sh 'mvn -B -ntp clean install'
        }
      }
    }

    stage('Get a Golang project') {
      git url: 'https://github.com/hashicorp/terraform-provider-google.git', branch: 'main'
      container('golang') {
        stage('Build a Go project') {
          sh '''
            mkdir -p /go/src/github.com/hashicorp
            ln -s `pwd` /go/src/github.com/hashicorp/terraform
            cd /go/src/github.com/hashicorp/terraform && make
          '''
        }
      }
    }

  }
}
```


### 6.7 继承
`pod template` 可以继承现有模板，也可以不继承。这意味着`pod template`将从它所继承的模板继承`node selector`, `service account`, `image pull secrets`, `container templates` and `volumes`。

- `yaml`根据`yamlMergeStrategy`的值进行合并。

我通过界面`Dashboard > Manage > Jenkins > Configure Clouds`创建 `pod template`
![](https://i-blog.csdnimg.cn/blog_migrate/0a80dcd8280fc1a781a2d09db3640d6a.png)

![](https://i-blog.csdnimg.cn/blog_migrate/35eeca90d7228cb0f0306c8f892a9b34.png)

这里我只想将`maven`的镜像版本由`3.8.1-jdk-8`升级为`3.8.1-jdk-11`，在项目编写`pipeline script`：

```bash
podTemplate(name: "build2",namespace: "one",inheritFrom: 'mypod', containers: [
    containerTemplate(name: 'maven', image: 'maven:3.8.1-jdk-11')
  ]) {
  node(POD_LABEL) {
    stage('Get a Maven project') {
      git 'https://github.com/jenkinsci/kubernetes-plugin.git'
      container('maven') {
        stage('Build a Maven project') {
          sh 'mvn -B -ntp clean install'
        }
      }
    }

    stage('Get a Golang project') {
      git url: 'https://github.com/hashicorp/terraform-provider-google.git', branch: 'main'
      container('golang') {
        stage('Build a Go project') {
          sh '''
            mkdir -p /go/src/github.com/hashicorp
            ln -s `pwd` /go/src/github.com/hashicorp/terraform
            cd /go/src/github.com/hashicorp/terraform && make
          '''
        }
      }
    }
  }
}
```

```bash
$ kubectl get pods -n one
NAME                 READY   STATUS    RESTARTS   AGE
build2-4p074-b1skx   3/3     Running   0          11m

$ kubectl get pods build2-4p074-b1skx -n one -o jsonpath="{.spec.containers[*].image}" | tr -s '[[:spa
ce:]]' '\n' |sort |uniq
golang:1.16.5
jenkins/inbound-agent:4.11-1-jdk11
maven:3.8.1-jdk-11
```
这里看到：通过`inheritFrom: 'mypod'`实现对模板的继承，并且实现`mypod`模板中的`maven`版本`v1.8.1-jdk-8`升级成为`3.8.1-jdk-11`，完成覆盖。

### 6.8 pod 嵌套
通过两个 `pod template` 方法合成一个包含两个容器的 `pod`实例。
```bash
podTemplate(name: 'docker', namespace: 'one', containers: [containerTemplate(image: 'docker', name: 'docker', command: 'cat', ttyEnabled: true)]) {
    podTemplate(name: 'maven', namespace: 'one', containers: [containerTemplate(image: 'maven:3.8.1-jdk-11', name: 'maven', command: 'cat', ttyEnabled: true)]) {
    node(POD_LABEL) {
      container('docker') {
        sh "echo hello from $POD_CONTAINER" // displays 'hello from docker'
      }
      container('maven') {
        sh "echo hello from $POD_CONTAINER" // displays 'hello from maven'
      }
     }
    }
}
```
构建结果：

```bash
$ k get pods -n one
NAME                READY   STATUS    RESTARTS   AGE
maven-9h33d-t39gn   3/3     Running   0          5s

#显示pod容器名称
$ kubectl get pods maven-9h33d-t39gn -n one -o jsonpath="{.spec.containers[*].name}" | tr -s '[[:space:]]' '\n' | sort |uniq
docker
jnlp
maven
```

`console output` 输出：

```bash
Started by user Jenkins Admin
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Created Pod: minikube2 one/maven-1bmzz-wv1sp
Agent maven-1bmzz-wv1sp is provisioned from template maven-1bmzz
---
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://jenkins.jenkins.svc.cluster.local:8080/job/busybox2/30/"
    runUrl: "job/busybox2/30/"
  labels:
    jenkins/jenkins-jenkins-agent: "true"
    jenkins/label-digest: "4ba5a4e248fa7baa06e229917703aa75dbced0ac"
    jenkins/label: "busybox2_30-lt9wl"
  name: "maven-1bmzz-wv1sp"
  namespace: "one"
spec:
  containers:
  - command:
    - "cat"
    image: "maven:3.8.1-jdk-11"
    imagePullPolicy: "IfNotPresent"
    name: "maven"
    resources:
      limits: {}
      requests: {}
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - command:
    - "cat"
    image: "docker"
    imagePullPolicy: "IfNotPresent"
    name: "docker"
    resources:
      limits: {}
      requests: {}
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_TUNNEL"
      value: "jenkins-agent.jenkins.svc.cluster.local:50000"
    - name: "JENKINS_AGENT_NAME"
      value: "maven-1bmzz-wv1sp"
    - name: "JENKINS_NAME"
      value: "maven-1bmzz-wv1sp"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.jenkins.svc.cluster.local:8080/"
    image: "jenkins/inbound-agent:4.11-1-jdk11"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on maven-1bmzz-wv1sp in /home/jenkins/agent/workspace/busybox2
[Pipeline] {
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ echo hello from docker
hello from docker
[Pipeline] }
[Pipeline] // container
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ echo hello from maven
hello from maven
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS
```

### 6.9 Pipeline script from SCM
github 仓库：[https://github.com/jenkinsci/kubernetes-plugin.git](https://github.com/jenkinsci/kubernetes-plugin.git)
![](https://i-blog.csdnimg.cn/blog_migrate/df49bcac798d391c2119687c8295957b.png)


- `examples/containerLog.groovy`
```bash
podTemplate(yaml: '''
              apiVersion: v1
              kind: Pod
              metadata:
                labels:
                  some-label: some-label-value
              spec:
                containers:
                - name: maven
                  image: maven:3.8.1-jdk-8
                  command:
                  - sleep
                  args:
                  - 99d
                  tty: true
                - name: mongo
                  image: mongo
''') {
  node(POD_LABEL) {
    stage('Integration Test') {
      try {
        container('maven') {
          sh 'nc -z localhost:27017 && echo "connected to mongo db"'
          // sh 'mvn -B clean failsafe:integration-test' // real integration test

          def mongoLog = containerLog(name: 'mongo', returnLog: true, tailingLines: 5, sinceSeconds: 20, limitBytes: 50000)
          assert mongoLog.contains('connection accepted from 127.0.0.1:')
          sh 'echo failing build; false'
        }
      } catch (Exception e) {
        containerLog 'mongo'
        throw e
      }
    }
  }
}
```
在`console output`可以看到输出日志：

```bash
[Pipeline] containerLog
.....
> start log of container 'mongo' in pod 'hello-34-p3hfk-8b868-bln0j'
{"t":{"$date":"2022-12-12T04:21:11.334+00:00"},"s":"I",  "c":"NETWORK",  "id":4915701, "ctx":"-","msg":"Initialized wire specification","attr":{"spec":{"incomingExternalClient":{"minWireVersion":0,"maxWireVersion":13},"incomingInternalClient":{"minWireVersion":0,"maxWireVersion":13},"outgoing":{"minWireVersion":0,"maxWireVersion":13},"isInternalClient":true}}}
{"t":{"$date":"2022-12-12T04:21:11.336+00:00"},"s":"I",  "c":"CONTROL",  "id":23285,   "ctx":"main","msg":"Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'"}
{"t":{"$date":"2022-12-12T04:21:11.336+00:00"},"s":"W",  "c":"ASIO",     "id":22601,   "ctx":"main","msg":"No TransportLayer configured during NetworkInterface startup"}
.....
{"t":{"$date":"2022-12-12T04:22:13.070+00:00"},"s":"I",  "c":"STORAGE",  "id":22430,   "ctx":"Checkpointer","msg":"WiredTiger message","attr":{"message":"[1670818933:69974][1:0x7f3fb7217700], WT_SESSION.checkpoint: [WT_VERB_CHECKPOINT_PROGRESS] saving checkpoint snapshot min: 34, snapshot max: 34 snapshot count: 0, oldest timestamp: (0, 0) , meta checkpoint timestamp: (0, 0) base write gen: 1"}}
> end log of container 'mongo' in pod 'hello-34-p3hfk-8b868-bln0j'
.......
```


### 6.10 配置共享库

示例：利用 [Ghostwritten/jenkins-shared-library](https://github.com/Ghostwritten/jenkins-shared-library) 作为我的共享库，点击配置`Dashboard` > `Manage Jenkins` > `Configure System`

![](https://i-blog.csdnimg.cn/blog_migrate/afa9de92a7a33af529a62e3d977de72f.png)


`pod` 模板位置：`jenkins-shared-library/src/com/kubernetes/PodTemplates.groovy`

```bash
package com.ghostwritten.PodTemplates

public void dockerTemplate(body) {
  podTemplate(
        containers: [containerTemplate(name: 'docker', image: 'docker:19.03.1', command: 'sleep', args: '99d')],
        volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]) {
    body.call()
}
}

public void mavenTemplate(body) {
  podTemplate(
        containers: [containerTemplate(name: 'maven', image: 'maven:3.8.1-jdk-8', command: 'sleep', args: '99d')]) {
    body.call()
}
}
```

注意，`POD_LABEL`将是生成的最里面的标签，以获得节点上所有外部pod都可用的节点，`Jenkins Pipeline script`代码：
```bash
@Library('my-shared-library') _
import com.ghostwritten.kubernetes.PodTemplates

podTemplates = new PodTemplates()

podTemplates.dockerTemplate {
  podTemplates.mavenTemplate {
    node(POD_LABEL) {
      container('docker') {
        sh "echo hello from $POD_CONTAINER" // displays 'hello from docker'
      }
      container('maven') {
        sh "echo hello from $POD_CONTAINER" // displays 'hello from maven'
      }
     }
  }
}
```
`console output`：

```bash
Started by user ghostwritten
Loading library my-shared-library@main
Examining Ghostwritten/jenkins-shared-library
Attempting to resolve main as a branch
Resolved main as branch main at revision ecf9353831650f5b7a2a97b316281407fd7de149
The recommended git tool is: NONE
using credential github-token
 > git rev-parse --resolve-git-dir /var/lib/jenkins/workspace/pod1@libs/41ff7c0a7b996db58dc106402d20945af1169af9339be67d96c4a407546ef059/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/Ghostwritten/jenkins-shared-library.git # timeout=10
Fetching without tags
Fetching upstream changes from https://github.com/Ghostwritten/jenkins-shared-library.git
 > git --version # timeout=10
 > git --version # 'git version 2.38.1'
using GIT_ASKPASS to set credentials github-token
 > git fetch --no-tags --force --progress -- https://github.com/Ghostwritten/jenkins-shared-library.git +refs/heads/main:refs/remotes/origin/main # timeout=10
Checking out Revision ecf9353831650f5b7a2a97b316281407fd7de149 (main)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f ecf9353831650f5b7a2a97b316281407fd7de149 # timeout=10
Commit message: "fix PodTemplate.groovy"
 > git rev-list --no-walk 3058774daa33ed501231e505eb1c61a20b6404d4 # timeout=10
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Created Pod: minikube1 default/pod1-10-0x0zw-81s8v-3jw0q
Agent pod1-10-0x0zw-81s8v-3jw0q is provisioned from template pod1_10-0x0zw-81s8v
---
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://192.168.10.90:8080/job/pod1/10/"
    runUrl: "job/pod1/10/"
  labels:
    jenkins: "slave"
    jenkins/label-digest: "71b3bbb3338de8a4c7c7a814e453cd35570e7604"
    jenkins/label: "pod1_10-0x0zw"
  name: "pod1-10-0x0zw-81s8v-3jw0q"
  namespace: "default"
spec:
  containers:
  - args:
    - "99d"
    command:
    - "sleep"
    image: "maven:3.8.1-jdk-8"
    imagePullPolicy: "IfNotPresent"
    name: "maven"
    resources:
      limits: {}
      requests: {}
    tty: false
    volumeMounts:
    - mountPath: "/var/run/docker.sock"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - args:
    - "99d"
    command:
    - "sleep"
    image: "docker:19.03.1"
    imagePullPolicy: "IfNotPresent"
    name: "docker"
    resources:
      limits: {}
      requests: {}
    tty: false
    volumeMounts:
    - mountPath: "/var/run/docker.sock"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "pod1-10-0x0zw-81s8v-3jw0q"
    - name: "JENKINS_WEB_SOCKET"
      value: "true"
    - name: "JENKINS_NAME"
      value: "pod1-10-0x0zw-81s8v-3jw0q"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://192.168.10.90:8080/"
    image: "jenkins/inbound-agent:4.11-1-jdk11"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/var/run/docker.sock"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - hostPath:
      path: "/var/run/docker.sock"
    name: "volume-0"
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on pod1-10-0x0zw-81s8v-3jw0q in /home/jenkins/agent/workspace/pod1
[Pipeline] {
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ echo hello from docker
hello from docker
[Pipeline] }
[Pipeline] // container
[Pipeline] container
[Pipeline] { (hide)
[Pipeline] sh
+ echo hello from maven
hello from maven
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS
```
有了共享库，我们就可以积累自己常用的`pipeline script demo`进行封装改为可调用配置。对提高编写`Jenkinsfile`的能力特别有帮助。

- [Extending with Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)

### 6.11 声明式 pipeline
声明式 `pipeline`可以很好地利用 kubernets 插件在kubernetes集群完成编译、打包、构建、测试、部署等一些流程。
这里先不展开细写，咱先记录几种 `pipeline script` 格式。
```bash
pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        metadata:
          labels:
            some-label: some-label-value
        spec:
          containers:
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
          - name: busybox
            image: busybox
            command:
            - cat
            tty: true
        '''
    }
  }
  stages {
    stage('Run maven') {
      steps {
        container('maven') {
          sh 'mvn -version'
        }
        container('busybox') {
          sh '/bin/busybox'
        }
      }
    }
  }
}
```

`console output`:

```bash
Started by user ghostwritten
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Created Pod: minikube1 default/pod2-8-nlldp-m2ghs-d7ms9
Agent pod2-8-nlldp-m2ghs-d7ms9 is provisioned from template pod2_8-nlldp-m2ghs
---
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://192.168.10.90:8080/job/pod2/8/"
    runUrl: "job/pod2/8/"
  labels:
    some-label: "some-label-value"
    jenkins: "slave"
    jenkins/label-digest: "90c685e1f5f0ea5bd4929c0bc949c47207784626"
    jenkins/label: "pod2_8-nlldp"
  name: "pod2-8-nlldp-m2ghs-d7ms9"
  namespace: "default"
spec:
  containers:
  - command:
    - "cat"
    image: "maven:alpine"
    name: "maven"
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - command:
    - "cat"
    image: "busybox"
    name: "busybox"
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "pod2-8-nlldp-m2ghs-d7ms9"
    - name: "JENKINS_WEB_SOCKET"
      value: "true"
    - name: "JENKINS_NAME"
      value: "pod2-8-nlldp-m2ghs-d7ms9"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://192.168.10.90:8080/"
    image: "jenkins/inbound-agent:4.11-1-jdk11"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on pod2-8-nlldp-m2ghs-d7ms9 in /home/jenkins/agent/workspace/pod2
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Run maven)
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ mvn -version
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-04T19:00:29Z)
Maven home: /usr/share/maven
Java version: 1.8.0_212, vendor: IcedTea, runtime: /usr/lib/jvm/java-1.8-openjdk/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "6.0.10-1.el7.elrepo.x86_64", arch: "amd64", family: "unix"
[Pipeline] }
[Pipeline] // container
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ echo hello world
hello world
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS
```

声明式模板不从父模板继承。由于在阶段级声明的代理可以覆盖全局代理，隐式继承导致了混乱。
如果需要，您需要使用字段`inheritFrom`显式地声明继承。在下面的例子中，`nested-pod`将只包含`maven`容器。


```bash
pipeline {
  agent {
    kubernetes {
      yaml '''
        spec:
        containers:
        - name: golang
            image: golang:1.16.5
            command:
            - sleep
            args:
            - 99d
        '''
    }
  }
  stages {
    stage('Run maven') {
        agent {
            kubernetes {
                yaml '''
                    spec:
                    containers:
                    - name: maven
                      image: maven:3.8.1-jdk-8
                      command:
                      - sleep
                      args:
                      - 99d
                    '''
            }
        }
      steps {
        …
      }
    }
  }
}
```


## 7 Jenkins Pipeline 构建镜像实例
###  7.1 git 拉取仓库 & 构建镜像
创建`docker` 容器，挂载`/var/run/docker.sock`文件即可获得在容器内构建镜像的条件。在`pipeline`中我从`github`拉取一个`demo`尝试构建镜像。
```bash
podTemplate(namespace: "default",yaml: '''
              apiVersion: v1
              kind: Pod
              spec:
                containers:
                - name: docker
                  image: docker:19.03.1
                  command:
                  - sleep
                  args:
                  - 99d
                  volumeMounts:
                  - name: dockersock
                    mountPath: /var/run/docker.sock
                volumes:
                - name: dockersock
                  hostPath:
                    path: /var/run/docker.sock
''') {
  node(POD_LABEL) {
    stage('Build Docker image') {
      git 'https://github.com/Ghostwritten/kaniko-python-docker.git'
      container('docker') {
        sh 'docker build -t ghostwritten/kaniko-python-docker:v1.0.0 .'
      }
    }
  }
}
```
`console output`：

```bash
Started by user Jenkins Admin
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Created Pod: kubernetes default/docker-6-mjk3l-z7k7p-zkt08
Agent docker-6-mjk3l-z7k7p-zkt08 is provisioned from template docker_6-mjk3l-z7k7p
---
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://jenkins.jenkins.svc.cluster.local:8080/job/docker/6/"
    runUrl: "job/docker/6/"
  labels:
    jenkins/jenkins-jenkins-agent: "true"
    jenkins/label-digest: "79a0f8e1675392aae2023abc0c58dbb40369e1eb"
    jenkins/label: "docker_6-mjk3l"
  name: "docker-6-mjk3l-z7k7p-zkt08"
  namespace: "default"
spec:
  containers:
  - args:
    - "99d"
    command:
    - "sleep"
    image: "docker:19.03.1"
    name: "docker"
    volumeMounts:
    - mountPath: "/var/run/docker.sock"
      name: "dockersock"
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_TUNNEL"
      value: "jenkins-agent.jenkins.svc.cluster.local:50000"
    - name: "JENKINS_AGENT_NAME"
      value: "docker-6-mjk3l-z7k7p-zkt08"
    - name: "JENKINS_NAME"
      value: "docker-6-mjk3l-z7k7p-zkt08"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.jenkins.svc.cluster.local:8080/"
    image: "jenkins/inbound-agent:4.11-1-jdk11"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - hostPath:
      path: "/var/run/docker.sock"
    name: "dockersock"
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on docker-6-mjk3l-z7k7p-zkt08 in /home/jenkins/agent/workspace/docker
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Build Docker image)
[Pipeline] git
The recommended git tool is: NONE
No credentials specified
Cloning the remote Git repository
Cloning repository https://github.com/Ghostwritten/kaniko-python-docker.git
 > git init /home/jenkins/agent/workspace/docker # timeout=10
Fetching upstream changes from https://github.com/Ghostwritten/kaniko-python-docker.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
 > git fetch --tags --force --progress -- https://github.com/Ghostwritten/kaniko-python-docker.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/Ghostwritten/kaniko-python-docker.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
Checking out Revision 925d4779eb7b4d41840d9daabb5ef5518d65ed1c (refs/remotes/origin/master)
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 925d4779eb7b4d41840d9daabb5ef5518d65ed1c # timeout=10
 > git branch -a -v --no-abbrev # timeout=10
 > git checkout -b master 925d4779eb7b4d41840d9daabb5ef5518d65ed1c # timeout=10
Commit message: "add kaniko python docker"
 > git rev-list --no-walk 925d4779eb7b4d41840d9daabb5ef5518d65ed1c # timeout=10
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ docker build -t ghostwritten/kaniko-python-docker:v1.0.0 .
Sending build context to Docker daemon  82.43kB

Step 1/6 : FROM python:3.8-slim-buster
 ---> 5cc8cb0c433a
Step 2/6 : WORKDIR /app
 ---> Using cache
 ---> 58118ab4f40e
Step 3/6 : COPY requirements.txt requirements.txt
 ---> Using cache
 ---> 638813138f19
Step 4/6 : RUN pip3 install -r requirements.txt
 ---> Using cache
 ---> 9dd11465b0d8
Step 5/6 : COPY . .
 ---> 4d92feff0580
Step 6/6 : CMD [ "python3", "app.py"]
 ---> Running in fca5c52c6378
Removing intermediate container fca5c52c6378
 ---> 048935596734
Successfully built 048935596734
Successfully tagged ghostwritten/kaniko-python-docker:v1.0.0
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS
```
构建成功。
###  7.2 编写 Dockerfile & 构建镜像
不过有时候我们直接在 `Pipeline Script` 中编写`Dockerfile`更加方便。

```bash
podTemplate(name: "testing", namespace: "default", yaml: '''
              apiVersion: v1
              kind: Pod
              spec:
                volumes:
                - name: docker-sock
                  hostPath: 
                    path: /var/run/docker.sock
                containers:
                - name: docker
                  image: docker:19.03.1
                  command:
                  - sleep
                  args:
                  - 99d
                  volumeMounts:
                  - name: docker-sock
                    mountPath: /var/run/docker.sock
                - name: docker-daemon
                  image: docker:19.03.1-dind
                  securityContext:
                    privileged: true
                  volumeMounts:
                  - name: docker-sock
                    mountPath: /var/run/docker.sock
''') {
  node(POD_LABEL) {
    writeFile file: 'Dockerfile', text: 'FROM scratch'
    container('docker') {
      sh 'docker version && DOCKER_BUILDKIT=1 docker build --progress plain -t testing .'
    }
  }
}
```
`Pipeline Steps`:

![](https://i-blog.csdnimg.cn/blog_migrate/5ebf96567f85feb5e93d89f46b77c4d5.png)
构建成功，太棒啦。😋

### 7.3  git 拉取仓库 & kaniko 构建镜像 & 推送入库
不过，以上这还不能满足我的需求，构建镜像就应该用专业的构建工具，来自`google`的项目：[kaniko](https://github.com/GoogleContainerTools/kaniko#--single-snapshot) 是一种在容器或 [Kubernetes](https://kubernetes.io/) 集群内从 `Dockerfile` 构建容器镜像的工具。它可以十分方便的在容器内安全地构建镜像并推送入库。如果你想了解更多它的细节，可以参考我的[这篇关于 kaniko 的实践](https://blog.csdn.net/xixihahalelehehe/article/details/121781164)。

这里我利用`secret-regcred.sh`脚本创建关于 `dockerhub`仓库 登陆认证的`secret`（regcred）。

```bash
#!/bin/bash
REGISTRY_SERVER=${1-'https://harbor.fumai.com/v2/'}
REGISTRY_USER=${2-'admin'}
REGISTRY_PASS=${3-'Harbor12345'}
REGISTRY_EMAIL=${4-'1zoxun1@gmail.com'}

SECRET_NAME=${5-'regcred'}
NAMESPACE=${6-'default'}

# 1. create secret method.
kubectl --namespace=$NAMESPACE create secret docker-registry $SECRET_NAME  --docker-server=$REGISTRY_SERVER --docker-username=$REGISTRY_USER --docker-password=$REGISTRY_PASS  --docker-email=$REGISTRY_EMAIL

# 2. create secret method too.
#kubectl --namespace=$NAMESPACE create secret   generic  $SECRET_NAME --from-file=.dockerconfigjson=config.json --type=kubernetes.io/dockerconfigjson

kubectl get secret $SECRET_NAME -n $NAMESPACE --output="jsonpath={.data.\.dockerconfigjson}" | base64 -d
```
创建：

```bash
$ bash secret-regcred.sh https://index.docker.io/v1/ <username> <password>
```
在`Pipeline Scrip`t写出：
```bash
podTemplate(name: 'kaniko-python-docker', namespace: 'default', yaml: '''
              kind: Pod
              spec:
                containers:
                - name: kaniko
                #  image: gcr.io/kaniko-project/executor:v1.6.0-debug
                  image: ghostwritten/kaniko-project-executor:v1.6.0-debug
                  imagePullPolicy: Always
                  command:
                  - sleep
                  args:
                  - 99d
                  volumeMounts:
                    - name: jenkins-docker-cfg
                      mountPath: /kaniko/.docker
                volumes:
                - name: jenkins-docker-cfg
                  secret:
                    secretName: regcred
                    items:
                      - key: .dockerconfigjson
                        path: config.json
'''
  ) {

  node(POD_LABEL) {
    stage('Build with Kaniko') {
      git 'https://github.com/Ghostwritten/kaniko-python-docker.git'
      container('kaniko') {
        sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=ghostwritten/kaniko-python-docker:v1.0.1'
      }
    }
  }
}
```
`console output`:

```bash
Started by user Jenkins Admin
[Pipeline] Start of Pipeline
[Pipeline] podTemplate
[Pipeline] {
[Pipeline] node
Created Pod: minikube2 default/kaniko-python-docker-frnd7-1wnb4
Still waiting to schedule task
‘kaniko-python-docker-frnd7-1wnb4’ is offline
Agent kaniko-python-docker-frnd7-1wnb4 is provisioned from template kaniko-python-docker-frnd7
---
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations:
    buildUrl: "http://192.168.10.26:32000/job/docker/21/"
    runUrl: "job/docker/21/"
  labels:
    jenkins/jenkins-jenkins-agent: "true"
    jenkins/label-digest: "117969f201f5b8afcc688a11799a14d3558b71c0"
    jenkins/label: "docker_21-mlxg4"
  name: "kaniko-python-docker-frnd7-1wnb4"
  namespace: "default"
spec:
  containers:
  - args:
    - "99d"
    command:
    - "sleep"
    image: "ghostwritten/kaniko-project-executor:v1.6.0-debug"
    imagePullPolicy: "Always"
    name: "kaniko"
    volumeMounts:
    - mountPath: "/kaniko/.docker"
      name: "jenkins-docker-cfg"
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_TUNNEL"
      value: "jenkins-agent.jenkins.svc.cluster.local:50000"
    - name: "JENKINS_AGENT_NAME"
      value: "kaniko-python-docker-frnd7-1wnb4"
    - name: "JENKINS_NAME"
      value: "kaniko-python-docker-frnd7-1wnb4"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://192.168.10.26:32000/"
    image: "jenkins/inbound-agent:4.11-1-jdk11"
    name: "jnlp"
    resources:
      limits: {}
      requests:
        memory: "256Mi"
        cpu: "100m"
    volumeMounts:
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
  nodeSelector:
    kubernetes.io/os: "linux"
  restartPolicy: "Never"
  volumes:
  - name: "jenkins-docker-cfg"
    secret:
      items:
      - key: ".dockerconfigjson"
        path: "config.json"
      secretName: "regcred"
  - emptyDir:
      medium: ""
    name: "workspace-volume"

Running on kaniko-python-docker-frnd7-1wnb4 in /home/jenkins/agent/workspace/docker
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Build with Kaniko)
[Pipeline] git
The recommended git tool is: NONE
No credentials specified
Cloning the remote Git repository
Cloning repository https://github.com/Ghostwritten/kaniko-python-docker.git
 > git init /home/jenkins/agent/workspace/docker # timeout=10
Fetching upstream changes from https://github.com/Ghostwritten/kaniko-python-docker.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
 > git fetch --tags --force --progress -- https://github.com/Ghostwritten/kaniko-python-docker.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/Ghostwritten/kaniko-python-docker.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
Checking out Revision 925d4779eb7b4d41840d9daabb5ef5518d65ed1c (refs/remotes/origin/master)
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 925d4779eb7b4d41840d9daabb5ef5518d65ed1c # timeout=10
 > git branch -a -v --no-abbrev # timeout=10
 > git checkout -b master 925d4779eb7b4d41840d9daabb5ef5518d65ed1c # timeout=10
Commit message: "add kaniko python docker"
First time build. Skipping changelog.
[Pipeline] container
[Pipeline] {
[Pipeline] sh
+ pwd
+ pwd
+ /kaniko/executor -f /home/jenkins/agent/workspace/docker/Dockerfile -c /home/jenkins/agent/workspace/docker '--cache=true' '--destination=ghostwritten/kaniko-python-docker:v1.0.1'
[36mINFO[0m[0002] Retrieving image manifest python:3.8-slim-buster 
[36mINFO[0m[0002] Retrieving image python:3.8-slim-buster from registry index.docker.io 
[36mINFO[0m[0004] Retrieving image manifest python:3.8-slim-buster 
[36mINFO[0m[0004] Returning cached image manifest              
[36mINFO[0m[0006] Built cross stage deps: map[]                
[36mINFO[0m[0006] Retrieving image manifest python:3.8-slim-buster 
[36mINFO[0m[0006] Returning cached image manifest              
[36mINFO[0m[0006] Retrieving image manifest python:3.8-slim-buster 
[36mINFO[0m[0006] Returning cached image manifest              
[36mINFO[0m[0006] Executing 0 build triggers                   
[36mINFO[0m[0006] Checking for cached layer index.docker.io/ghostwritten/kaniko-python-docker/cache:1c98e3b76b7eea791ef15553919754ea5ea0384db24733375b20585b5abf5c59... 
[36mINFO[0m[0008] No cached layer found for cmd RUN pip3 install -r requirements.txt 
[36mINFO[0m[0008] Unpacking rootfs as cmd COPY requirements.txt requirements.txt requires it. 
[36mINFO[0m[0025] WORKDIR /app                                 
[36mINFO[0m[0025] cmd: workdir                                 
[36mINFO[0m[0025] Changed working directory to /app            
[36mINFO[0m[0025] Creating directory /app                      
[36mINFO[0m[0025] Taking snapshot of files...                  
[36mINFO[0m[0025] COPY requirements.txt requirements.txt       
[36mINFO[0m[0025] Taking snapshot of files...                  
[36mINFO[0m[0025] RUN pip3 install -r requirements.txt         
[36mINFO[0m[0025] Taking snapshot of full filesystem...        
[36mINFO[0m[0027] cmd: /bin/sh                                 
[36mINFO[0m[0027] args: [-c pip3 install -r requirements.txt]  
[36mINFO[0m[0027] Running: [/bin/sh -c pip3 install -r requirements.txt] 
Collecting Flask==2.0.2
  Downloading Flask-2.0.2-py3-none-any.whl (95 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 95.2/95.2 KB 291.8 kB/s eta 0:00:00
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Collecting Jinja2>=3.0
  Downloading Jinja2-3.1.2-py3-none-any.whl (133 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.1/133.1 KB 119.1 kB/s eta 0:00:00
Collecting Werkzeug>=2.0
  Downloading Werkzeug-2.2.2-py3-none-any.whl (232 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 232.7/232.7 KB 54.5 kB/s eta 0:00:00
Collecting click>=7.1.2
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 KB 30.9 kB/s eta 0:00:00
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Installing collected packages: MarkupSafe, itsdangerous, click, Werkzeug, Jinja2, Flask
Successfully installed Flask-2.0.2 Jinja2-3.1.2 MarkupSafe-2.1.1 Werkzeug-2.2.2 click-8.1.3 itsdangerous-2.1.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 22.0.4; however, version 22.3.1 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
[36mINFO[0m[0053] Taking snapshot of full filesystem...        
[36mINFO[0m[0056] COPY . .                                     
[36mINFO[0m[0056] Taking snapshot of files...                  
[36mINFO[0m[0056] Pushing layer index.docker.io/ghostwritten/kaniko-python-docker/cache:1c98e3b76b7eea791ef15553919754ea5ea0384db24733375b20585b5abf5c59 to cache now 
[36mINFO[0m[0056] Pushing image to index.docker.io/ghostwritten/kaniko-python-docker/cache:1c98e3b76b7eea791ef15553919754ea5ea0384db24733375b20585b5abf5c59 
[36mINFO[0m[0056] CMD [ "python3", "app.py"]                   
[36mINFO[0m[0056] No files changed in this command, skipping snapshotting. 
[33mWARN[0m[0058] error uploading layer to cache: failed to push to destination index.docker.io/ghostwritten/kaniko-python-docker/cache:1c98e3b76b7eea791ef15553919754ea5ea0384db24733375b20585b5abf5c59: HEAD https://index.docker.io/v2/ghostwritten/kaniko-python-docker/cache/blobs/sha256:c3aa9870d3065edb2286cac744c95fb0d2f1c98b2d8a231257314eda7d3598b0: unexpected status code 401 Unauthorized (HEAD responses have no body, use GET for details) 
[36mINFO[0m[0058] Pushing image to ghostwritten/kaniko-python-docker:v1.0.1 
[36mINFO[0m[0066] Pushed image to 1 destinations               
[Pipeline] }
[Pipeline] // container
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] }
[Pipeline] // podTemplate
[Pipeline] End of Pipeline
Finished: SUCCESS
```
登陆 [dockerhub](https://hub.docker.com/repository/docker/ghostwritten/kaniko-python-docker) 查看新构建的镜像。
![](https://i-blog.csdnimg.cn/blog_migrate/bc7c927a989545854cf1951e5a99cf55.png)
大功告成。

关于在`Jenkins Pipeline` 中部署 `pod` 与构建镜像的实践到这里暂告一段落。但这一切还只是刚刚开始。`Jenkins Pipeline`可以实现许多功能，我们可以利用`jenkins`设置代码`push`的触发条件，当`push` 代码后自动触发审查、构建、推送、测试、部署等等。还可以结合关于触发 `ArgoCD` 的 Jenkins 管道，完成对`Kubernetes` 集群的部署，[Argo CD](https://argo-cd.readthedocs.io/en/stable/)是用于`Kubernetes`的声明性`GitOps`持续交付工具，`Argo CD`可在指定的目标环境中自动部署所需的应用程序状态，应用程序部署可以在Git提交时跟踪对分支，标签的更新，或固定到清单的特定版本。

拜！


##  问题
####  http://192.168.10.90:8080/tcpSlaveAgentListener/ is invalid: 404 Not Found
```bash
- jnlp -- terminated (255)
-----Logs-------------
Dec 09, 2022 2:02:42 PM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: hello-19-hcmkv-c0zgn-mjt8j
Dec 09, 2022 2:02:42 PM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
Dec 09, 2022 2:02:42 PM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 4.11
Dec 09, 2022 2:02:42 PM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
Dec 09, 2022 2:02:42 PM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
Dec 09, 2022 2:02:42 PM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://192.168.10.90:8080/]
Dec 09, 2022 2:02:42 PM hudson.remoting.jnlp.Main$CuiListener error
SEVERE: http://192.168.10.90:8080/tcpSlaveAgentListener/ is invalid: 404 Not Found
java.io.IOException: http://192.168.10.90:8080/tcpSlaveAgentListener/ is invalid: 404 Not Found
	at org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver.resolve(JnlpAgentEndpointResolver.java:219)
	at hudson.remoting.Engine.innerRun(Engine.java:724)
	at hudson.remoting.Engine.run(Engine.java:540)
```
解决方法：
- `http://192.168.10.90:8080/manage/configureSecurity/`

将`agents` 改为 `Random`

![](https://i-blog.csdnimg.cn/blog_migrate/bcafea66c610fce6affd20684d64c968.png)



参考：
- [Kubernetes plugin for Jenkins](https://plugins.jenkins.io/kubernetes/)
- [jenkinsci/kubernetes-plugin](https://github.com/jenkinsci/kubernetes-plugin)
- [jenkins pipeline 加载 groovy 脚本](https://unprobug.com/2022/03/08/cicd/jenkins/jenkins-load-groovy-script-md/)
- [Build & Push Docker Image using Jenkins Pipeline | Devops Integration Live Example Step By Step](https://www.youtube.com/watch?v=PKcGy9oPVXg)
- [Jenkins On Kubernetes Tutorial | How to setup Jenkins on kubernetes cluster | Thetips4you](https://www.youtube.com/watch?v=_r-C_FFDLmU)

