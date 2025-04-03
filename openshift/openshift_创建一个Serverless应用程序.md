


---


 1. [Red Hat OpenShift 4.8 环境集群搭建](https://ghostwritten.blog.csdn.net/article/details/123207497)
 2. [openshift 如何输出json日志](https://ghostwritten.blog.csdn.net/article/details/123335781)
 3. [openshfit Vertical Pod Autoscaler 实践](https://ghostwritten.blog.csdn.net/article/details/123335420)
 4. [openshift Certified Helm Charts 实践](https://ghostwritten.blog.csdn.net/article/details/123335635)
 5. [openshift 创建一个Serverless应用程序](https://ghostwritten.blog.csdn.net/article/details/123335299)
 6. [openshift gitops 实践](https://ghostwritten.blog.csdn.net/article/details/123336100)
 7. [openshift Tekton pipeline 实践](https://ghostwritten.blog.csdn.net/article/details/123375339)

---
##  1. Serverless Lab 
使用`Knative Serverless`，您可以轻松地在Kubernetes上运行无服务器容器。Knative负责网络、自动伸缩(甚至为零)和修订跟踪的细节。这让你能够专注于你的核心逻辑。
在本实验室中，您将学习如何将无服务器应用程序部署为“服务”应用程序。

Serverless服务
减少资源消耗的一种好方法是确定哪些服务需要24/7运行。示例咖啡店应用程序有三个组件: a database, a front-end coffee shop, and a service barista that makes drinks。似乎只有在咖啡订单实际提交时才需要咖啡师服务，而咖啡店前端应该始终运行。因此，有必要将咖啡师服务转换为无服务器服务，在不使用时可以扩展到零副本。

目标：

 - 将咖啡师应用程序转换为无服务器服务
 - 观察日志的变化

## 2. 将Barista应用程序转换为无服务器服务
咖啡师应用程序不需要一直运行。它只需要在创建新订单时运行。因此，当咖啡店没有流量时，应用程序不需要消耗资源。

###  2.1 Delete Barista Deployment and Service

 1. Go to your Red Hat® OpenShift® Container Platform web console and log in as `admin`.
 2. Use the perspective switcher to switch to the `Developer` perspective.
 3. If a pop-up window appears, click `Cancel`.
 4. In the navigation menu, click `Topology.`

A graphical representation of your application appears, displaying the app components, plus a cron job that orders drinks every minute:
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/126c947945b35afb1356aaee87fb952a.png)

 1. Click `D barista` to display the details panel.
 2. You need to delete this barista OpenShift service and the baristadeployment before you can deploy a Knative service by the same name.
 3. Click `S barista` to open the Service details page.
 4. Select `Actions → Delete` Service and confirm.
 5. Go back to `Topology` and click `D barista` again to verify that the service was deleted.
 6. Select `Actions → Delete deployment` to delete the `barista` deployment.

Confirm the deletion.

###  2.2 Deploy Barista Component as Knative Service
在本节中，您将把咖啡师容器映像部署为Knative无服务器服务。Knative服务的API包不同于Kubernetes服务，扮演着不同的角色。

> Kubernetes Service service.k8s.io/v1: Service is a named abstraction of software service (for example, mysql) consisting of a local port (for example 3306) that the proxy listens on, and the selector that determines which pods will answer requests sent through the proxy. [https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/](https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/)

> Knative Service serving.knative.dev/v1: A complete application: [https://knative.dev/docs/reference/api/serving-api/#serving.knative.dev](https://knative.dev/docs/reference/api/serving-api/#serving.knative.dev)

 1. On the navigation menu, click `Add`.
 2. Click `Container Images`.
 3. Fill in the `Image stream from internal registry` fields as follows:
Project: `dev-coffeeshop`
Image Stream: `barista`
Tag: `latest`
 4. Complete the remaining fields as follows:
`Runtime icon`: select the Knative icon
`Application`: coffee-shop
`Name`: barista
`Resources`: select the Knative Service radio button
 `Create a route to the Applicatio`n: check this box
 5. Click `Create`
Expect the `Topology` view to appear with the new KSVC barista service along with its revision number.

##  3 更新咖啡店ConfigMap访问Knative咖啡师服务
The route to the barista part of the application has changed, so the coffee shop component cannot find it.

The old service URL was `http://barista:8080/processes`. The new service URL (Knative route) is `http://barista.dev-coffeeshop.svc.cluster.local/processes`. Knative使用项目名而不是服务名，并且端口现在由Knative处理。这将断开咖啡店应用程序和咖啡师应用程序之间的连接。

In this exercise, you reconnect them.

###  3.1 Fix ConfigMap that Associates `coffee-shop` and `barista`
The `coffee-shop` application gets its `BARISTA_URL` from a ConfigMap.

 1. In the navigation menu, click `ConfigMaps` and then click `CM coffee-shop`.
 2. Scroll down to the `Data` section and the `BARISTA_URL` of `http://barista:8080/processes`.
 3. Select `Actions → Edit ConfigMap`.
 4. In the YAML listing that opens, copy and paste the following URL  into the `BARISTA_URL` value:
    `http://barista.dev-coffeeshop.svc.cluster.local/processes`.

###  3.2 Restart coffee-shop Application
在本节中，您将向下扩展咖啡店部署并备份以获取ConfigMap更改。

 1. Return to Topology and click `D coffee-shop`.
 2. In the panel that appears, click the `Details` tab.
Expect to see three running pods.
 3.使用箭头按钮来缩小到0，然后返回到3

###  3.3 Access coffee-shop Web Application
要确保`coffee-shop`应用程序工作，请在coffee-shop应用程序上单击Open URL，而不是`barista`应用程序:
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16d015b725afc9ef0a44af1079efb545.png)
预计会出现一两行JSON输出。

 2. Add `/index.html` to the end of the URL to access the application.

> 如果看到类似`RESTEASY003210`的错误…你走的是Knative 	`barista`的路线。这是错误的路线。使用`coffee-shop`路线代替。 

 3. 点一杯饮料，并验证咖啡师应用程序可以扩展到处理请求

Expect to see some beverages already ordered by the order-drinks cron job. Your order is at the bottom of the list.

The `order-drinks` cron job orders two new drinks every minute from the `coffee-shop` application in the `dev-coffeeshop` namespace.

## 4 Examine Logs
如果您返回Kibana接口，您会看到结构化类型中现在有更多的字段。它们大多是当您从Kubernetes部署更改为Knative应用程序时，应用程序捕获的应用程序异常。

 1. 返回Kibana web界面，从左侧导航面板单击Discovery。
 2. 从“可用字段”一节中，将`*t* structured.error.message`添加到日志发现中。
 3. 使用顶部的时间滑块来关注带有错误消息的事件。
希望很容易找到错误消息，例如`“RESTEASY004655`: Unable to invoke r .”
java.net.NoRouteToHostException: No route to host (Host unreachable)".

您使用Knative将一个无服务器应用程序部署为一个服务应用程序。
现在，在下一个模块中，我们将继续学习GitOps。使用GitOps，您可以将咖啡馆应用程序的部署自动化到生产环境，所有这些都来自一个Git存储库。



