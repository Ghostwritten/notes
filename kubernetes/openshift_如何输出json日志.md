

----


 1. [Red Hat OpenShift 4.8 环境集群搭建](https://ghostwritten.blog.csdn.net/article/details/123207497)
 2. [openshift 如何输出json日志](https://ghostwritten.blog.csdn.net/article/details/123335781)
 3. [openshfit Vertical Pod Autoscaler 实践](https://ghostwritten.blog.csdn.net/article/details/123335420)
 4. [openshift Certified Helm Charts 实践](https://ghostwritten.blog.csdn.net/article/details/123335635)
 5. [openshift 创建一个Serverless应用程序](https://ghostwritten.blog.csdn.net/article/details/123335299)
 6. [openshift gitops 实践](https://ghostwritten.blog.csdn.net/article/details/123336100)
 7. [openshift Tekton pipeline 实践](https://ghostwritten.blog.csdn.net/article/details/123375339)

---
##  1. JSON Logging 
Red Hat®OpenShift®容器平台的4.8版包含了JSON日志记录支持，现在这又回到了日志解决方案中。
客户现在可以精确地控制哪些容器日志是json格式的。他们可以标记通用的JSON模式，以便日志管理系统(Red Hat管理的Elasticsearch系统或第三方系统)确切地知道如何处理这些日志。

当一个`parse: json`字段被添加到`ClusterLogForwarder`自定义资源的管道中时，顶级字段被添加并以`structured`作为前缀。
当使用它们在集成的Elasticsearch集群中存储日志数据时，将为每个新的管道创建新的Elasticsearch索引。进入翻译页面

> 集群的本地Elasticsearch服务器在创建每一个新索引时都会面临显著的性能影响

实现目标：

 - 演示结构化JSON日志记录
 - 检查一个为dev-咖啡店项目定制的ClusterLogging资源
 - 使用Kibana显示结构化日志
 - 说明如何将结构化日志发送到其他日志采集器

## 2.  Add Logging for dev-coffeeshop Namespace
对于本练习，您将使用以下`ClusterLogForwarder`配置，该配置已经部署在您的集群中。
要查看ClusterLogForwarder配置，请执行以下操作:

在OpenShift容器平台web控制台中，选择“**Project: All Projects**”。
使用透视图切换器切换到**Administrator**透视图并单击**Search**。
在“**Resources**”下拉列表中，选择“**ClusterLogForwarder**”。
单击CLF**instance**，然后在出现的页面上单击YAML。
查看这个配置，看看创建JSON日志时的新特性:

```bash
apiVersion: logging.openshift.io/v1
kind: ClusterLogForwarder
metadata:
  name: instance
  namespace: openshift-logging
spec:
  inputs:
 - application:
      namespaces:
      - dev-coffeeshop 
    name: dev-coffeeshop-input-example
  outputDefaults: 
    elasticsearch:
      structuredTypeKey:  kubernetes.namespace_name 
        # OR
      structuredTypeName: dev-coffeeshop-index-name 
  pipelines:
 - inputRefs: 
    - dev-coffeeshop-input-example
    name: pipeline-dev-coffeeshop
    outputRefs:
    - default
    parse: json
 - inputRefs: 
    - infrastructure
    - application
    - audit
    outputRefs:
    - default
```
 - 从`dev-coffeeshop`名称空间引导日志的定制输入。
 - 默认输出是集群的Elasticsearch，而不是远程服务器。
 - 这将基于名称空间在Elasticsearch中设置一个结构化日志索引。
 - 一个用户定义的字符串，它命名了索引，如果存在的话，前面会有一个`structuredTypeKey`。
 - 启用JSON解析的管道。
 - 没有JSON解析的所有公共输入的管道。

> 每个用于Elasticsearch的新JSON日志管道都会在Elasticsearch中创建一个新索引。从命令行，你可以得到一个ES索引的快速列表:`oc exec $es_pod -c elasticsearch -- indices`

稍后，我们将在dev- coffeshop名称空间中看到咖啡馆应用程序可用的结构化日志。

在Kibana中写一个查询，只从`structured.message`中获取日志消息。

## 3.  在Kibana中检查结构化日志
在这个练习中，您通过`openshift-logging`命名空间中的Kibana路由访问Kibana web控制台，并查询结构化日志。

### 3.1 Open Kibana Route
1.从OpenShift容器平台web控制台**Administrator** 的角度找到Kibana URL作为路由，并确保选中了`Project: OpenShift -logging`。
2.在导航菜单中，导航到“`Network → Routes`”。
期待看到Kibana服务的位置。
3.单击该位置以打开它。

### 3.2 建立索引模式

 1. In the Kibana web console, click `Management.`
 2. Click `Index Patterns`.
 3. Click `Create index pattern`.
 4. In the `Index pattern` field, enter `app-dev-coffeeshop-*`, then click `Next Step`.
 5. In the `Time Filter` field, select `@timestamp`.
 6. Click `Create index pattern`.

### 3.3 使用Kibana查询Elasticsearch

 - Click `Discover`.
 - 如果还没有选中`app-dev-coffeeshop-*`，请使用左侧的下拉列表选择`app-dev-coffeeshop-*`。
 - 在找到的第一个日志条目上，单击右箭头▶查看日志详细信息。
 - 在列表的底部，找到`structured`的条目。 `elements,` including `structured.hostName`
   `structured.message,` and `structured.level`

![在这里插入图片描述](https://img-blog.csdnimg.cn/e71bc8f5373a4acf97ea80569ffc193d.png)
每种具有不同JSON格式的日志输入类型都在Elasticsearch数据库中创建一个新索引，以处理不同的`structured.*`数据。通过“Apache”、“Google”等标准JSON格式组织日志来节约资源。

## 4. 清理日志查询，方便故障处理
应用程序不断地发出消息，因此您需要一种方法来清除混乱，只查看对您的情况有意义的消息和查询结果。结构化JSON日志记录使这变得更容易。

首先，查看一个杂乱的消息，其中的数据不明显，无法搜索:

 - 在`Available Field`下找到`*t* message`并单击`add`。
 - 观察日志输出如何更容易阅读，但是消息仍然很模糊——这是查看任何输出时的典型情况

整理的信息:

 - Under `Selected fields` find `*t* message` and click `remove`.
 - Under `Available fields` find and click `add` for *`t* structured.message`, `*t* structured.origin`, and `*t* structured.type`
 - 注意，这里只显示了您做决策所需的关键数据

如果有某种类型的`orderID`，可以将起源和类型的Order Details与`FINISHED`和`COLLECTED`的Order Status关联起来，那就太好了。为了实现这个目标，你会要求开发人员进行哪些代码更改?

在本课程的后面，您将把prod- coffeshop名称空间添加到这个日志管道中，并更新查询。
现在，我们来看看OpenShift容器平台4.8的另一个重要功能:Certified Helm Charts。



