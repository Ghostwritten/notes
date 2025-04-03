

---

 1. [Red Hat OpenShift 4.8 环境集群搭建](https://ghostwritten.blog.csdn.net/article/details/123207497)
 2. [openshift 如何输出json日志](https://ghostwritten.blog.csdn.net/article/details/123335781)
 3. [openshfit Vertical Pod Autoscaler 实践](https://ghostwritten.blog.csdn.net/article/details/123335420)
 4. [openshift Certified Helm Charts 实践](https://ghostwritten.blog.csdn.net/article/details/123335635)
 5. [openshift 创建一个Serverless应用程序](https://ghostwritten.blog.csdn.net/article/details/123335299)

---

##  1. 知识点
主题涵盖OpenShift容器平台4.8特性:

 - Single-node OpenShift Container Platform cluster JSON logging
 - Certified Helm repositories
 - Vertical Pod Autoscaler (VPA)
 - Serverless
 - GitOps
 - Pipelines

## 2. 集群节点

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/486c2e7f744bf9d41055ac17cfeada4b.png)

 - 一个用于命令行访问集群的bastion主机
 - 一个单节点OpenShift容器平台集群，部署OpenShift容器平台和应用程序
 - 一个Elasticsearch专用节点

##  3. 环境示例应用程序
部署在两个namespaces中的咖啡店应用程序示例:

 - dev-coffeeshop
 - prod-coffeeshop

据应用程序包括:

 - PostgreSQL数据库服务器
 - coffee-shop front-end component
 - 咖啡师service component，您可以将其从Deployment转换为Knative service
OpenShift CronJob在两个命名空间中生成应用程序活动

##  4. 部署一个集群	
略

## 5. 环境预配置
在本节中，您将提供实验室环境，以提供对执行实验室所需的所有组件的访问。实验室环境是一个基于云的环境，因此您可以从任何地方通过Internet访问它。但是，不要期望性能与专用环境相匹配。
在本课程的实验中，你使用OpenShift容器平台web控制台和命令行来完成大多数任务。	

###  5.1  Deploy Environment
> If you previously set up an environment for this class, you can skip this section and go directly to the Install Operator SDK section.

在本节中，您将提供实验室环境，以提供对执行实验室所需的所有组件的访问。实验室环境是一个基于云的环境，因此您可以从任何地方通过Internet访问它。但是，不要期望性能与专用环境相匹配。

1. Go to the [OPENTLC lab portal](https://labs.opentlc.com/) and use your OPENTLC credentials to log in.

> 如果您不记得自己的密码，请进入[OPENTLC帐户管理页面重置密码](https://account.opentlc.com/account/)。

2. Navigate to `Services` → `Catalogs` → `All Services` → `OPENTLC OpenShift 4 Labs`.

3. On the left, select OpenShift 4 Operators Lab.

4. On the right, click `Order.`

5. If you are not in North America, select your region from the Region list.

6. At the bottom right, click `Submit`.

> Do not select `App Control` → `Start` after ordering the lab.


几分钟后，你将收到三封电子邮件，其中包括如何连接到环境的说明:

 - 第一封电子邮件通知您配置已经开始。
 - 当请求虚拟机成功时，触发第二封邮件。
 - 最后一封电子邮件声明配置已经完成，并提供了启动和使用实验室环境的重要信息。

等待直到您收到最后一封电子邮件，然后再尝试连接到您的环境。
记下您的GUID——您环境中唯一的4个字符的标识符。


###  5.1 关机后启动环境(Reference)
为了节省资源，实验室环境在8小时后自动关闭。本节提供在本课程的实验环境自动关闭后重新启动的说明。
Go to the OPENTLC lab portal and use your OPENTLC credentials to log in.
Expect to see a list of your Active Services when you first log in—if not, select Services → My Services.
在您的服务列表中，选择您的实验室环境。
选择App Control→Start启动您的实验室环境。
在“Are you sure?”提示。
At the bottom right, click Submit.
几分钟后，您将收到一封电子邮件，通知您实验室环境已经启动

### 5.2 Access Bastion Host
该堡垒管理主机作为进入环境的接入点，而不是Red Hat®OpenShift®容器平台环境的一部分。
使用邮件中提供的OPENTLC帐户名、主机名和密码连接到您的bastion主机:


```bash
ssh opentlc-username_@bastion.GUID.BASEDOMAIN
```

确认您当前的身份是`system:admin`:


```bash
oc whoami
```
Sample Output

```bash
system:admin
```

##  6. 测试连接OpenShift Container Platform
1.在您收到的准备邮件中，单击OpenShift容器平台web控制台的URL。
2.使用电子邮件中提供的凭据登录。
###  6.1 Test Server Connections
使用您在配置邮件中收到的命令和密码连接到您的bastion主机:

```bash
ssh <opentlc-username>@bastion.<$GUID>.<$BASEDOMAIN>
```
验证GUID变量是否为您的环境正确设置:

```bash
echo $GUID
```
Sample Output

```bash
c3po
```

> 您的GUID可以是一个4或5个字符的字母数字字符串。

###  6.2 连接到OpenShift容器平台集群
Your User ID is **labuser**.

In the provisioning email you r	eceived, note the following:

The OpenShift Container Platform password

The URL for the API of the cluster—for example, `https://api.<some-custom-identifier>.opentlc.com:6443`

The URL for the web console of the cluster—for example, h`ttps://console-openshift-console.apps.<some-custom-identifier>.opentlc.com`

Use the `oc login` command to log in to the cluster:

```bash
oc login --insecure-skip-tls-verify=true -u <opentlc-username> -p ${OCP_PASSWORD} ${OCP_API}
```
Sample Output

```bash
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>
```
您的OpenShift 4.8环境已经可以使用了。
进入下一个模块，开始使用JSON日志记录。



