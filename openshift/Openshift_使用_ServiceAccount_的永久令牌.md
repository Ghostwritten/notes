![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5582390f49734325a1bec51d057a21b6.png)



在 OpenShift 中，使用 Bearer Token 的方式进行认证时，如果希望实现永久验证，需采用以下策略来生成或管理一个长期有效的认证方式。

------

### **1. 使用 ServiceAccount 的永久令牌**

ServiceAccount 的默认 Secret 中会包含一个永久有效的 Token，你可以直接提取并使用：

#### **步骤：**

1. 找到相关 ServiceAccount：

   ```bash
   oc get sa -n openshift-monitoring
   ```

   默认的 ServiceAccount 可以是 `prometheus-k8s` 或其他特定名称。

2. 查看与 ServiceAccount 关联的 Secret：

 ```bash
   oc get secret -n openshift-monitoring | grep <serviceaccount-name>
   ```

3. 获取 Token：

 ```bash
   oc describe secret <secret-name> -n openshift-monitoring
   ```

   找到 `token` 字段的值，将其复制并替换到你的配置中：

 ```yaml

   secureJsonData:
     httpHeaderValue1: 'Bearer <your-serviceaccount-token>'
 ```

4. 更新数据源配置文件并应用。

------

### **2. 创建一个专用的 ServiceAccount**

如果不想使用默认的 ServiceAccount，可以创建一个新的专用 ServiceAccount，并赋予合适的权限。

#### **步骤：**

1. 创建 ServiceAccount：

   ```bash
   oc create sa grafana-datasource -n openshift-monitoring
   ```

2. 绑定权限（需要访问 Thanos 的权限）：

 ```
   bash
   oc adm policy add-cluster-role-to-user cluster-monitoring-view -z grafana-datasource -n openshift-monitoring
   ```

3. 获取该 ServiceAccount 的永久 Token：

   ```bash
   oc describe secret $(oc get secret -n openshift-monitoring | grep grafana-datasource | awk '{print $1}') -n openshift-monitoring
   ```

4. 将 Token 添加到数据源配置中。

------

### **3. 使用 OpenShift OAuth 的长期 Token**

如果你希望通过用户认证获取一个长期有效的 Token，可以创建一个基于 OpenShift OAuth 的访问令牌：

#### **步骤：**

1. 登录 OpenShift Web 控制台或使用 CLI：

   ```bash
   oc login -u <username>
   ```

2. 创建 OAuth Token：

   ```bash
   oc create oauthaccesstoken --user=<username>
   ```

3. 将生成的 Token 替换到数据源配置中：

   ```yaml
   secureJsonData:
     httpHeaderValue1: 'Bearer <your-oauth-token>'
   ```

------

### **4. 使用 API 网关或中间服务代理认证**

如果不希望直接暴露令牌，可以设置一个代理服务（如 API Gateway）来统一处理认证。Grafana 的数据源可以通过该代理访问，而不需要直接携带敏感令牌。

#### **示例配置：**

1. 创建一个中间代理，代理到 Thanos。

2. 在 Grafana 数据源中配置代理 URL：

   ```bash
   url: 'http://my-proxy-service.local/thanos'
   jsonData:
     access: proxy
   ```
