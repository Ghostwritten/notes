


----

 1. [Red Hat OpenShift 4.8 环境集群搭建](https://ghostwritten.blog.csdn.net/article/details/123207497)
 2. [openshift 如何输出json日志](https://ghostwritten.blog.csdn.net/article/details/123335781)
 3. [openshfit Vertical Pod Autoscaler 实践](https://ghostwritten.blog.csdn.net/article/details/123335420)
 4. [openshift Certified Helm Charts 实践](https://ghostwritten.blog.csdn.net/article/details/123335635)
 5. [openshift 创建一个Serverless应用程序](https://ghostwritten.blog.csdn.net/article/details/123335299)
 6. [openshift gitops 实践](https://ghostwritten.blog.csdn.net/article/details/123336100)
 7. [openshift Tekton pipeline 实践](https://ghostwritten.blog.csdn.net/article/details/123375339)

---

##  1. 介绍

Red Hat®OpenShift®pipeline是一个基于Kubernetes资源的云本地持续集成和持续交付(CI/CD)解决方案。
它使用Tekton构建块，通过抽象底层实现细节，实现跨多个平台的自动化部署。
Tekton引入了许多标准的自定义资源定义(crd)，用于定义在Kubernetes发行版之间可移植的CI/CD管道。


OpenShift管道操作符已经为您部署，所以您可以立即启动。
在本实验中，您将创建一个新项目，并部署一个具有“hello world”任务的简单管道，然后部署构建和部署一个两层Golang应用程序所需的各种资源。
目标：

 - 部署一个简单的pipeline
 - 部署一个构建pipeline应用程序

##  2. 部署一个简单的pipeline
在本练习中，您将创建一个名为`trivial-pipeline`的新项目。

 1. 在OpenShift容器平台web控制台中，选择`Administrator`透视图。
 2. 使用`Project`:下拉列表创建一个名为`trivial-pipeline`的新项目。
 3. 导入这个YAML内容来创建一个新任务:
a. 在导航菜单上，选择`Pipelines → Tasks`。
b. 在“`Create`…”下拉列表中，选择“`Task`”。
c. 将YAML的默认定义替换为以下内容:

```bash
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
  namespace: trivial-pipeline
spec:
  steps:
    - name: say-hello
      image: registry.access.redhat.com/ubi8/ubi
      command:
        - /bin/bash
      args: ['-c', 'echo Hello World']
```

 4. Click `Create`.

###  2.1 使用TaskRun运行“hello”样例任务
要运行Task，需要创建`TaskRun`对象。您可以使用命令行tkn实用程序，或者导入以下YAML内容。

 1. 在导航菜单上，选择`Pipelines → Tasks`
 2. 从Create下拉列表中，选择`TaskRun`
 3. 将YAML的默认定义替换为以下内容:

```bash
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  namespace: trivial-pipeline
  generateName: hello-run-
  labels:
    app.kubernetes.io/managed-by: tekton-pipelines
    tekton.dev/task: hello
spec:
  resources: {}
  serviceAccountName: pipeline
  taskRef:
    kind: Task
    name: hello
```

 4. 单击Logs选项卡查看TaskRun的输出。
期望它显示`“Hello World”`。

##  3 部署 Two-Tier Golang Application
### 3.1 创建任务来定义资源定义

接下来，您将构建和部署一个应用程序，其中包含一些`Tasks`, `ClusterTasks`, and `custom Tasks`。
首先，为这项工作创建一个名称空间/项目。
从“项目:”下拉列表中选择“`Create Project`”，并在“ `Name`”字段中输入`pipelinez -vote`。
在导航菜单上，选择`Pipelines → Tasks`。
在“`create`”下拉列表中，选择“Tasks”。
将YAML的默认定义替换为以下内容:


```bash
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: apply-manifests
  namespace: pipelines-vote
spec:
  workspaces:
 - name: source #1
  params:  #2
    - name: manifest_dir
      description: The directory in source that contains yaml manifests
      type: string
      default: "k8s"
  steps: #3
    - name: apply
      image: quay.io/openshift/origin-cli:latest
      workingDir: /workspace/source
      command: ["/bin/bash", "-c"]
      args:
        - |-
          echo Applying manifests in $(inputs.params.manifest_dir) directory  #4
          oc apply -f $(inputs.params.manifest_dir)
          echo -----------------------------------
```
 - 1.工作区指的是持久化卷声明(Persistent Volume Claim, PVC)，其中存储了参数和任务输出;这个叫做`source`。
 - 2.此任务接受的单个参数。在本例中，它用于指示可以在其中找到部署应用程序的YAML清单的目录。
 - 3.本任务中的单个步骤。它实际上通过在参数中定义的manifest_dir中的所有文件上运行`oc apply -f`来创建OpenShift对象。
 - 4.注意在语法`$(input .params.)`的步骤中参数是如何被引用的。

Click `Create`.

> 参数被发送到使用它们的任务之前在哪里定义的?在任务运行。您可以像上面在`trivial-pipeline`中所做的那样创建单独的任务运行，或者让PipelineRun提供这些值，如您在下一节中看到的那样。


### 3.2 Create Task to Update Name of Image Deployed
由于管道在每次新应用程序构建时都要构建一个新的容器映像，因此新容器映像将获得一个不同的标记或哈希ref。
您需要在Pipeline中添加一个Task，以确保在重新部署豆荚时使用了适当的容器映像。

 1. 再次单击Create，然后选择Task。
 2. 将YAML的默认定义替换为以下内容:

```bash
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: update-deployment
  namespace: pipelines-vote
spec:
  params:
  - description: The name of the deployment patch the image   #1
    name: deployment
    type: string
  - description: Location of image to be patched with   #1
    name: IMAGE
    type: string
  steps:
  - args:
    - |-
      oc patch deployment $(inputs.params.deployment) --patch='{"spec":{"template":{"spec":{
        "containers":[{
          "name": "$(inputs.params.deployment)",
          "image":"$(inputs.params.IMAGE)"
        }]
      }}}}'
    command:
    - /bin/bash
    - -c
    image: quay.io/openshift/origin-cli:latest   #2
    name: patch
    resources: {}
```

> 1.这些是Task期望从`TaskRun`接收的参数。.
> 2.此任务使用专用的容器 OpenShift命令行工具, `oc`.

### 3.3 创建PVC来存储工作区数据
询问参数和合成输出由泰克顿自动存储在专用pvc。你可以通过`PipelineRun`让任意数量的工作空间与你的任务相关联。此外，工作区可以跨越一个或多个任务，以提供一个共享区域，任务可以在其中访问彼此的数据。这些是普通的pvc。

 1. 从web控制台顶部的工具栏中，单击ocp_web_console_add_icon (`Add`)来创建一个PVC来支持工作区
或者，您可以使用此方法应用定义并创建PVC资源:`Storage→PersistentVolumeClaims→create`。

2. Use the following YAML definition:

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: source-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
Click `Create`.

### 3.4 为应用程序创建 Build-and-Deploy Pipeline 
这个管道有三个主要部分和四个任务。各节内容如下:

 - Workspaces
在这里定义，为下面使用它们的任务提供上下文。
 - Params
管道期望从PipelineRun得到的输入，这些输入对任务是可用的。
 - Tasks
这个任务数组定义了哪些任务被执行，对它们可用的工作区，以及传递给它们的参数。

> 在Pipeline定义中出现的任务顺序不适用。有些步骤具有`runAfter`值，它指示当前步骤应该在其之后运行的特定任务。这是必需的，因为Tekton默认为并行运行所有步骤。请记住，这是区别于其他持续集成系统的一个重要方面。

 1. 在导航菜单上，点击管道，然后 `Create → Pipeline`。
 2. 单击🔘YAML `view`单选按钮，查看将管道定义粘贴到其中的文本区域。
 3. 将默认定义替换为以下YAML内容:

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-and-deploy
spec:
  workspaces:   #1
  - name: shared-workspace
  params:    #2
  - name: deployment-name
    type: string
    description: name of the deployment to be patched
  - name: git-url
    type: string
    description: url of the git repo for the code of deployment
  - name: git-revision
    type: string
    description: revision to be used from repo of the code for deployment
    default: "master"
  - name: IMAGE
    type: string
    description: image to be build from the code
  tasks:    #3
  - name: fetch-repository
    taskRef:
      name: git-clone
      kind: ClusterTask
    workspaces:
    - name: output   #4
      workspace: shared-workspace
    params:
    - name: url
      value: $(params.git-url)
    - name: subdirectory
      value: ""
    - name: deleteExisting
      value: "true"
    - name: revision
      value: $(params.git-revision)
  - name: build-image
    taskRef:
      name: buildah
      kind: ClusterTask
    params:
    - name: TLSVERIFY
      value: "false"
    - name: IMAGE
      value: $(params.IMAGE)
    workspaces:
    - name: source 
      workspace: shared-workspace
    runAfter:   #5
    - fetch-repository
  - name: apply-manifests
    taskRef:
      name: apply-manifests
    workspaces:
    - name: source
      workspace: shared-workspace
    runAfter:
    - build-image
  - name: update-deployment
    taskRef:
      name: update-deployment
    workspaces:
    - name: source
      workspace: shared-workspace
    params:
    - name: deployment
      value: $(params.deployment-name)
    - name: IMAGE
      value: $(params.IMAGE)
    runAfter:
    - apply-manifests
```

---

 1. 这定义了与下面的单个任务共享的PVC。
 2. Pipeline期望从PipelineRun获得的各种参数。
 3. 任务的数组。它们不是按照在此列表中出现的顺序执行的。
 4. 工作区的详细信息:这两个设置指出了工作区文件系统中不同的子目录(`output and source`)。Tekton自动组织这些数据，如果需要，它们可以访问彼此的数据。如前所述，这是通过$(input.)完成的。
 5. 一个`runAfter`:节出现在这个和所有后续的task中，定义了task的一个有序的执行列表


Click `Create`.

### 3.5 Run Pipeline for Back-end API of Voting Application

在本节中，您将为投票应用程序的后端部分执行管道。

 1. 在导航菜单上，点击`pipeline`，然后点击`Create→PipelineRun`。
 2. 将YAML的默认定义替换为以下内容:


```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  labels:
    tekton.dev/pipeline: build-and-deploy
  generatename: build-and-deploy-run-
  namespace: pipelines-vote
spec:
  params:   #1
  - name: IMAGE
    value: image-registry.openshift-image-registry.svc:5000/pipelines-vote/vote-api
  - name: deployment-name
    value: pipelines-vote-api
  - name: git-url
    value: https://github.com/openshift/pipelines-vote-api.git
  pipelineRef:   #2
    name: build-and-deploy
  serviceAccountName: pipeline
  timeout: 1h0m0s
  workspaces:   #3
  - name: shared-workspace
    persistentVolumeClaim:
      claimName: source-pvc
```

--

 1. 这些是由`PipelineRun`创建的TaskRuns传递给task的实际字符串值。
 2. 这是对您在前一节中创建的Pipeline的引用。
 3. 这就是工作区的定义。这就是PVC和工作区名称绑定在一起的地方。

点击`Create`并查看管道的运行!
这将部署应用程序的一部分。接下来创建应用程序的前端部分。

### 3.6 Run Pipeline for Front-End UI of Voting Application
在本节中，将全面部署应用程序。
如上所述，使用下面的PipelineRun定义来执行构建和部署应用程序:

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  generateName: build-and-deploy-run-
  labels:
    tekton.dev/pipeline: build-and-deploy
  namespace: pipelines-vote
spec:
  params:
  - name: IMAGE
    value: image-registry.openshift-image-registry.svc:5000/pipelines-vote/vote-ui   #1
  - name: deployment-name
    value: pipelines-vote-ui
  - name: git-url
    value: https://github.com/openshift/pipelines-vote-ui.git   #2
  pipelineRef:
    name: build-and-deploy
  serviceAccountName: pipeline
  timeout: 1h0m0s
  workspaces:
  - name: shared-workspace
    persistentVolumeClaim:
      claimName: source-pvc
```

--

 1. 请注意，不同的映像名称将写入构建，并从其中部署pod。
 2. 请注意应用程序前端的不同存储库。在这个案子里，a monorepo是怎么用的?

单击Create并查看管道运行情况。

### 3.7 Access Application
一旦构建完成，你可以获得投票应用UI的URL:

 1. 在导航菜单上，单击Networking查看到UI的路由。
 2. Click the `route` and `vote`.


## 4. 问题

 1. 这是一个Flask应用程序https://github.com/openshift/pipelines-vote-ui.git
 2. 如何修改应用程序，使其以cat而不是wolf为特征?
 3. 如何打开JSON日志以便更健壮地查询这些日志—例如，查看有多少人在为猫和狗投票?
 4. 您是否可以获得代码、修改代码，然后修改构建管道以获取代码修改?
 5. 您需要创建自己的源代码存储库吗?
 6. 您可以使用本地pvc来托管代码吗?


管道是复杂的，但是任务是非常强大的，并且很容易用`taskrun`进行测试。考虑如何将容器图像扫描添加到管道中，以便及早发现安全问题。
