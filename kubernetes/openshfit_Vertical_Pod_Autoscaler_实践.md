

---


 1. [Red Hat OpenShift 4.8 环境集群搭建](https://ghostwritten.blog.csdn.net/article/details/123207497)
 2. [openshift 如何输出json日志](https://ghostwritten.blog.csdn.net/article/details/123335781)
 3. [openshfit Vertical Pod Autoscaler 实践](https://ghostwritten.blog.csdn.net/article/details/123335420)
 4. [openshift Certified Helm Charts 实践](https://ghostwritten.blog.csdn.net/article/details/123335635)
 5. [openshift 创建一个Serverless应用程序](https://ghostwritten.blog.csdn.net/article/details/123335299)
 6. [openshift gitops 实践](https://ghostwritten.blog.csdn.net/article/details/123336100)
 7. [openshift Tekton pipeline 实践](https://ghostwritten.blog.csdn.net/article/details/123375339)

---


## 1. 介绍
在本实验室中，您将了解垂直吊舱自动伸缩器(VPA)。
VPA学习并为与该工作负载对象关联的pod应用优化的CPU和内存资源。
要使用VPA Operator，您需要为集群中的工作负载对象创建VPA自定义资源(CR)。
您可以将VPA与deployment, stateful set, job, daemon set, replica set, or replication controller workload object.

> You can find the documentation for the VPA here:
> [https://docs.openshift.com/container-platform/4.8/nodes/pods/nodes-pods-vertical-autoscaler.html](https://docs.openshift.com/container-platform/4.8/nodes/pods/nodes-pods-vertical-autoscaler.html)

目标：

 - Describe the function and features of the `VPA`.
 - Set up a Vertical Pod Autoscaler for your Coffee Shop application.
 - Use the VPA and load generation to scale up a Pod.
 - Observe the JSON logs.

### 1.1   验证并安装 Vertical Pod Autoscaler Operator
Browse to your Red Hat® OpenShift® Container Platform web console, and log in as `admin`.

Instructions and credentials for this are in the provisioning email you received.

Use the perspective switcher to switch to the `Administrator` perspective.

In the navigation menu, navigate to `Workloads → Pods`

Click the `Project` drop-down list and select `openshift-vertical-pod-autoscaler.`

Expect to see that the four Pods running the VPA’s systems are running OK.

### 1.1 Vertical Pod Autoscaler 使用方法
The VPA CR must be in the same project as the Pods you want to monitor.

You use the VPA CR to associate a workload object and specify which mode the VPA operates in.

The VPA has several modes:

 - Auto
 - Recreate
 - Initial
 - Off

The Auto and Recreate modes automatically apply the VPA CPU and memory recommendations throughout the Pod lifetime. The VPA deletes any Pods in the project that are out of alignment with its recommendations. When redeployed by the workload object, the VPA updates the new Pods with its recommendations.

The Initial mode automatically applies VPA recommendations only at Pod creation.

The Off mode only provides recommended resource limits and requests, allowing you to manually apply the recommendations. The Off mode does not update Pods.

You can also use the CR to opt-out certain containers from VPA evaluation and updates.

> You must scale up your application to at least two replicas for the VPA to work in Auto or Recreate mode. This helps to reduce application downtime.

## 2. 扩容 Coffee Shop 应用程序
In this exercise, you use the OpenShift Container Platform web console to scale the Coffee Shop application to three Pods.

 1. Browse to the web console.
 2. From the `Administrator` perspective, select `Project`: `dev-coffeeshop`.
 3. From the navigation menu, select `Workloads → Deployments`.
 4. Locate the `coffee-shop` deployment, click options_menu_icon (`Options`) and select `Edit Pod count.`
 5. Click ⊕ (`Plus`) twice to scale up the deployment to three Pods.

Your development Coffee Shop application is now scaled up and ready for the VPA.

### 2.1. 检查 Current Resource Definitions（crd）
The Coffee Shop application was deployed without specific resource definitions. The VPA adds resource definitions once the VPA CR is created. In this exercise, you verify that there are no resource definitions.

 1. Select `Project: dev-coffeeshop` from the project drop-down list.
 2. On the navigation menu, select `Workloads → Deployments.`
 3. Click D `coffee-shop`.
 4. From the `Actions` drop-down list, select `Edit resource limits`.
 5. Observe that no values are filled in.
 6. Click `Cancel`.
 7. Examine the Pods that are deployed by clicking the `Pods` tab, and  clicking one of the `coffee-shop` Pods.
 8. On the `Pod details` page, scroll down until you see Containers and select the coffee-shop container.
On the Container details page, expect to see a section named `Resource requests` with no values:

Sample Output

```bash
Resource requests
-
```

This confirms that the application has been deployed without resource requests or limits.


## 3.  创建 VPA for Coffee Shop
In this exercise, you create a Vertical Pod Autoscaler and observe how the Pod is redeployed with recommended resource requests and limits. The VPA operator/controller is already installed. To use it for the Coffee Shop application, you must create a `VerticalPodAutoscaler` custom resource in the Coffee Shop project.

> ：vpa没有自定义的web界面。

### 3.1  创建 Custom Resource（cr）
In this section, you create a `VerticalPodAutoscaler` custom resource in the Coffee Shop project.

 1. From the toolbar at the top of your web console, click `ocp_web_console_add_icon (Add)`.
 2. Click the `Project` drop-down list and make sure that `dev-coffeeshop` is selected.
 3. Copy and paste the following YAML content into the text area:

```bash
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: coffee-shop-vpa-recommender
  namespace: dev-coffeeshop
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       coffee-shop
  updatePolicy:
    updateMode: "Auto"
```

4..Click `Create`.

### 3.2  验证重新部署的pod资源设置
In this section, you confirm that the Pod is redeployed with recommended resource requests and limits.

1.On the navigation menu, select `Workloads → Pods`.

> If you are quick, you can watch the Pods be redeployed. 

2.Click one of the new Pods.
They have a timestamp of `Just Now`.

3.On the `Pod details` page, scroll down to the `Containers` section and select the `coffee-shop` container.

On the `Container details` page, expect to see a section called Resource requests with some values:

Sample Output

```bash
Resource requests
cpu: 25m, memory: 297164212
```
The VPA has filled in its recommended Resource requests for your application. As utilization of your application changes, so too do these resource requests.

## 4.  删除 VPA for Coffee Shop Application
It is not strictly necessary to remove the VPA from the `dev-coffeeshop project`. It does not interfere with anything except a `HorizontalPodAutoscaler` and that is not part of this class.

 1. On the Navigation menu, select `Administration →CustomResourceDefinitions.`
 2. Scroll all the way to the bottom and click `VerticalPodAutoscalers`.
 3. In the `CustomResourceDefinition` details page for the  `VerticalPodAutoscalers`, select the `Instances` tab.
 4. Locate the VPA `coffee-shop-vpa-recommender,` click `options_menu_icon (Options)` and select Delete `Vertical Pod Autoscaler.`
 5. Click `Delete.`

你已经完成了垂直吊舱自动缩放实验。您可能希望尝试其他VPA模式，如重新创建(recreates)，它只在部署被扩展时应用建议。
当您完成之后，继续到[openshift 创建一个Serverless应用程序](https://ghostwritten.blog.csdn.net/article/details/123335299)




