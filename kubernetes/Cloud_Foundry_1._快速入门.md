

##  概述

在本模块中，我们开始使用云铸造。我们引入了CLI或命令行接口，用于与云计算实例交互。我们将在平台上部署我们的第一个应用程序，来体验一下接下来会发生什么:

 - 我们将向您介绍CLI，这是一个用于与Cloud Foundry交互的工具。
 - 我们将解释如何使用不同的云计算实例。
 - 您将开始将您的第一个应用程序部署到Cloud Foundry。本节的目的是强调Cloud
   Foundry为复杂的开发人员工作流带来的简单性和能力。


##  Command Line Interface (CLI)
命令行接口(或CLI)用于通过[REST A](https://www.codecademy.com/article/what-is-rest)PI与云计算(CF)实例交互。cf CLI向平台用户公开了人性化的命令，并且是通用核心，这意味着它可以与所有的Cloud Foundry发行版一起工作。


##  The CF API

cf CLI正在与[Cloud Foundry API](https://v3-apidocs.cloudfoundry.org/version/3.101.0/index.html)通信，使开发人员能够以用户友好的方式向API发出请求。

您也可以通过cf curl直接向API发出请求，但这往往更复杂。我们稍后会更多地讨论cf旋度。

在本课程中，我们将使用cf CLI ([v7](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html))的最新版本，它由最新版本的cf API ([v3](https://v3-apidocs.cloudfoundry.org/version/3.101.0/index.html))支持。确保你使用的是版本7+是很重要的，因为我们将在整个课程中使用v3的特性。

按照命令行版本7的安装说明[安装命令行](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)。

通过运行cf version，确保运行的是CLI的version 7。

##  文档
CLI具有大量的自文档，因此熟悉它的导航非常有帮助。

作为一个总结:

 - `Cf help`显示最常用命令的帮助信息
 - `Cf help` -显示所有可用命令的帮助信息
 - `Cf -h`(或——help)显示执行特定命令的详细帮助。

如果执行cf <命令>，而不带必要参数，则CLI将输出详细的帮助文本，就像执行cf <命令>——help一样。然而，省略——help并假设CLI将始终输出帮助文本是有风险的，因为不是所有命令都需要额外的参数。例如，如果您要运行cf push，希望看到帮助文本和当前目录中存在的清单，您可能无意中推一个应用程序。因此，熟悉cf <命令>——help工作流是很有帮助的。

尝试运行`cf help -a`来查看所有可用的cf命令，然后在几个命令上运行——help标志，以便熟悉帮助输出。

如果你正在准备Cloud Foundry Certified Developer考试，你可以在考试期间访问cf help，因为cf CLI安装在考试系统中。


##  Aliases
许多命令都有别名(例如，cf m作为cf市场的快捷方式)。别名在帮助描述中列出。例如，在cf帮助中的Services integration:部分，你应该看到:

```bash
marketplace,m
```

逗号后的m是快捷方式。随着您对更常见的CLI命令的熟悉，快捷方式可以节省您的时间。

##  国际化
The CLI supports displaying text and descriptions in multiple languages. The name of the commands won't change, but help text and command descriptions show in the language of your choice.

For the most up-to-date information on supported languages, see the Cloud Foundry docs.

To change the locale in use, you can run:

```bash
cf config --locale <LOCALE>
```

For example, you can change to German using:

```bash
cf config --locale de-DE
```

You can rerun cf help commands to see the effect.

Using a locale of CLEAR will remove any value set previously:

```bash
cf config --locale CLEAR
```

We will discuss more advanced CF CLI concepts towards the end of this course.

## API Endpoints

cf命令行用于与任何云计算实例交互。它对正在使用的特定实例的CF API端点进行[RESTful](https://www.codecademy.com/articles/what-is-rest)调用。

##  Setting the Endpoint
Before you interact with a Cloud Foundry instance using the cf CLI, you need set the API endpoint. When you set an API endpoint, you are selecting an instance of Cloud Foundry to use. You may hear some vendors refer to deployments as "foundations" or "instances", which mean the same thing. They are a running Cloud Foundry.

You can set the API endpoint either with the cf api command, or as part of the cf login command (you will see this below). Go ahead and set your API endpoint using:

```bash
cf api <API_ENDPOINT>
```

You should be able to see the endpoint you set by running:

```bash
cf api
```

##  Authentication
After setting the API endpoint, you need to authenticate with the Cloud Foundry instance you are using. There are numerous options for authentication (used in different circumstances), but for now, we will interactively authenticate using cf login.

The cf login command has features and flags (view these by running cf login --help). The options allow you to authenticate interactively or in non-interactive mode, which we discuss on the next pages. The -a flag, is used to specify the API endpoint when logging in (rather than using the cf api command separately).

Be careful when using non-interactive authentication, as credentials can end up in your terminal history.

##  Authentication: Interactive Terminal Login
To authenticate interactively in the terminal, run cf login without any flags. The CLI will prompt you to provide input. In this mode, your sensitive credentials are not stored in the command shell history. If you scroll through the command shell history in your terminal using the up-arrow key, you will see the `cf login` command, but will not see your username or password.

##  Authentication: Interactive Browser Login
The cf login command also supports interactive login through your browser via the -sso (single sign-on) flag. Interactive browser login works well with browser-integrated password managers like 1Password, LastPass, or KeePass. Because you are authenticating via a browser, you have mitigated the risk of your password ending up in your shell history.

You can login interactively via:

```bash
cf login --sso
```

The CLI will output a URL. Copy and paste this in a browser, and authenticate in exchange for a passcode. You can copy and paste the passcode provided into the terminal window to complete the authentication flow.

CFCD Exam Tip: The one-time passcode flow requires you to open pages in a browser. For this reason, you should not use cf login --sso in the exam, as it may take you to a non-approved URL.


##  Authentication: Logging Out
When you authenticate with Cloud Foundry, the CLI caches a token locally. When you make requests (like pushing an app), the CLI includes this token in each request. Therefore, you do not have to re-authenticate on every request and continue to work until your token expires (and cannot be refreshed). At this time, you would need to re-authenticate. However, this also means you need to be mindful to log out when appropriate to ensure a malicious actor does not gain access to your Cloud Foundry.

If you run cf target, you will see that you are authenticated. You can then log out using:

```bash
cf logout
```

If you re-run cf target, you will see you are no longer logged in.


##  cf push
使用cf push 命令将应用部署到Cloud Foundry。



我们将部署的应用程序没有什么特别之处——它是一个简单的静态应用程序，命名为`first-push`，用于培训目的。您将在课程资源的`/applications`目录中找到第一个推送应用程序(您可以从Menu中的resources选项卡下载它们)。



在你的终端，更改目录，使你在第一个push目录:



```bash
cd applications/first-push
```


在这个目录中，与应用程序一起的是一个名为manifest (manifest.yml)的文件。manifest为Cloud Foundry提供了如何运行应用程序的指令。



你现在可以在这个目录下通过运行以下命令来部署`first-push`应用:



```bash
cf push
```



默认情况下，push命令查找名为manifest的文件。并使用它来部署应用程序。



您可以查看用于部署应用程序的清单。让我们看看`manifest.yml`中的指令:

 - name: first-push
 在Cloud Foundry中应用程序的名称。名称应该是供人类使用的描述性名称，可以是任何您想要的名称。
 - instances: 1
 The number of instances of the application Cloud Foundry should create.
 - memory: 32M
The amount of memory allocated to the container for each application instance.
 - disk_quota: 64M
The amount of disk allocated to the container for each application instance.
 - random-route: true
Specifies that Cloud Foundry should generate a random route for accessing the app.
 - buildpacks:
The buildpack(s) used to containerize your application.
staticfile_buildpack - In this case, we only need the staticfile buildpack.

##  Checking Your Work
The push should take about a minute. If everything is successful, you should see output for your running application similar to:

```bash
name:              first-push
requested state:   started
routes:            first-push-active-quokka-dy.<my-domain>
last uploaded:     Mon 24 May 11:52:30 BST 2021
stack:             cflinuxfs3
buildpacks:
  name                  version  detect output  buildpack name
  staticfile_buildpack  1.5.17   staticfile     staticfile

type: web
sidecars:
instances: 1/1
memory usage: 32M
start command: $HOME/boot.sh
    state    since                 cpu   memory    disk      details
#0  running  2021-05-24T10:52:41Z  0.0%  0 of 32M  0 of 64M
```

The application has a user interface that shows some details about the application. You can copy the URL of your application from your terminal output under routes and open it in a tab in your browser.

##  Wait, What Just Happened?
In short, you deployed an app using a single command! It would have looked the same regardless of the language of the app. The CLI on your machine uploaded the app bits to Cloud Foundry and provided some information about what should happen.

Cloud Foundry took your app bits and used a buildpack to create a container image with your app, plus any necessary runtime dependencies. The resulting image was then run in a container. Once your app was running, Cloud Foundry began to automatically route traffic to it (hence your ability to access it in a browser). We discuss this process in more detail later in the course.

##  And What Didn't Happen?
You didn't build an image with a tool like Docker (though Cloud Foundry supports Docker, too). You didn't install a runtime or manipulate any filesystems. You didn't manually provision any resources like compute and storage. You didn't create and deploy a load balancer or reserve any ports. You didn't update routing tables or do any health checks. You just typed cf push. Pretty cool, right?

The rest of the course will build on these experiences.

![在这里插入图片描述](https://img-blog.csdnimg.cn/8992e97b8a904330a3a8a59baf95f6e7.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e30d840f8acc4ce287eb44b57af30cd5.png)

