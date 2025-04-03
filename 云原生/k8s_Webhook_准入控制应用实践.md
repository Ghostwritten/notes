# k8s Webhook 准入控制应用实践
tags: 开发

![](https://i-blog.csdnimg.cn/blog_migrate/47303063086d31ff23ebc642023867de.png)



## 1. 背景
除了[内置的 admission 插件](https://ghostwritten.blog.csdn.net/article/details/112242163)， 准入插件可以作为扩展独立开发，并以运行时所配置的 Webhook 的形式运行。 此页面描述了如何构建、配置、使用和监视准入 Webhook。

## 2. 什么是准入 Webhook？
准入 Webhook 是一种用于接收准入请求并对其进行处理的 HTTP 回调机制。 可以定义两种类型的准入 webhook，即 `验证性质的准入 Webhook` 和 `修改性质的准入 Webhook`。 修改性质的准入 Webhook 会先被调用。它们可以更改发送到 API 服务器的对象以执行自定义的设置默认值操作。

> 说明： 如果准入 Webhook 需要保证它们所看到的是对象的最终状态以实施某种策略。 则应使用验证性质的准入Webhook，因为对象被修改性质 Webhook 看到之后仍然可能被修改。

## 3. 先决条件

 - 确保 Kubernetes 集群版本至少为 `v1.16`（以便使用 `admissionregistration.k8s.io/v1` API）或者 `v1.9` （以便使用 `admissionregistration.k8s.io/v1beta1` API）。
 - 确保启用 `MutatingAdmissionWebhook` 和 `ValidatingAdmissionWebhook` 控制器。 这里是一组推荐的 admission 控制器，通常可以启用。
 - 确保启用了 `admissionregistration.k8s.io/v1beta1` API

## 4. 编写一个准入 Webhook 服务器
请参阅 `Kubernetes e2e` 测试中的 [admission webhook 服务器](https://github.com/kubernetes/kubernetes/blob/v1.13.0/test/images/webhook/main.go) 的实现。webhook 处理由 apiserver 发送的 `AdmissionReview` 请求，并且将其决定作为 `AdmissionReview 对象`以相同版本发送回去。

示例准入 Webhook 服务器置 `ClientAuth` 字段为空，默认为 `NoClientCert` 。这意味着 webhook 服务器不会验证客户端的身份，认为其是 apiservers。 如果你需要双向 TLS 或其他方式来验证客户端，请参阅如何对 apiservers 进行身份认证。

## 5. 部署准入 Webhook 服务
e2e 测试中的 webhook 服务器通过 [deployment API 部署](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#deployment-v1-apps)在 Kubernetes 集群中。该测试还将创建一个 [service](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#service-v1-core) 作为 webhook 服务器的前端。[参见相关代码](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#service-v1-core)。

你也可以在集群外部署 webhook。这样做需要相应地更新你的 webhook 配置.

## 6. 即时配置准入 Webhook
你可以通过 [ValidatingWebhookConfiguration](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#validatingwebhookconfiguration-v1-admissionregistration-k8s-io) 或者 [MutatingWebhookConfiguration](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.20/#mutatingwebhookconfiguration-v1-admissionregistration-k8s-io) 动态配置哪些资源要被哪些准入 Webhook 处理。

以下是一个 ValidatingWebhookConfiguration 示例，mutating webhook 配置与此类似。

```bash
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  rules:
  - apiGroups:   [""]
    apiVersions: ["v1"]
    operations:  ["CREATE"]
    resources:   ["pods"]
    scope:       "Namespaced"
  clientConfig:
    service:
      namespace: "example-namespace"
      name: "example-service"
    caBundle: "Ci0tLS0tQk...<base64-encoded PEM bundle containing the CA that signed the webhook's serving certificate>...tLS0K"
  admissionReviewVersions: ["v1", "v1beta1"]
  sideEffects: None
  timeoutSeconds: 5
```

```bash
# 1.16 中被淘汰，推荐使用 admissionregistration.k8s.io/v1
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  rules:
  - apiGroups:   [""]
    apiVersions: ["v1"]
    operations:  ["CREATE"]
    resources:   ["pods"]
    scope:       "Namespaced"
  clientConfig:
    service:
      namespace: "example-namespace"
      name: "example-service"
    caBundle: "Ci0tLS0tQk...<base64-encoded PEM bundle containing the CA that signed the webhook's serving certificate>...tLS0K"
  admissionReviewVersions: ["v1beta1"]
  timeoutSeconds: 5
```
scope 字段指定是仅集群范围的资源（Cluster）还是名字空间范围的资源资源（Namespaced）将与此规则匹配。* 表示没有范围限制。

> 说明： 当使用 clientConfig.service 时，服务器证书必须对 <svc_name>.<svc_namespace>.svc有效。
> 

> 说明： 对于使用 admissionregistration.k8s.io/v1 创建的 webhook 而言，其 webhook调用的默认超时是 10 秒； 对于使用 admissionregistration.k8s.io/v1beta1 创建的 webhook而言，其默认超时是 30 秒。 从 kubernetes 1.14 开始，可以设置超时。建议对 webhooks 设置较短的超时时间。 如果webhook 调用超时，则根据 webhook 的失败策略处理请求。


当 `apiserver` 收到与 `rules` 相匹配的请求时，`apiserver` 按照 `clientConfig` 中指定的方式向 webhook 发送一个 `admissionReview` 请求。
创建 webhook 配置后，系统将花费几秒钟使新配置生效。

## 7. 对 apiservers 进行身份认证
如果你的 webhook 需要身份验证，则可以将 apiserver 配置为使用基本身份验证、持有者令牌或证书来向 webhook 提供身份证明。完成此配置需要三个步骤。

 - 启动 apiserver 时，通过 `--admission-control-config-file` 参数指定准入控制配置文件的位置。
 - 在准入控制配置文件中，指定 `MutatingAdmissionWebhook` 控制器和 `ValidatingAdmissionWebhook` 控制器应该读取凭据的位置。 凭证存储在 kubeConfig 文件中（是​​的，与 kubectl 使用的模式相同），因此字段名称为 kubeConfigFile。 以下是一个准入控制配置文件示例：

```bash
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ValidatingAdmissionWebhook
  configuration:
    apiVersion: apiserver.config.k8s.io/v1
    kind: WebhookAdmissionConfiguration
    kubeConfigFile: "<path-to-kubeconfig-file>"
- name: MutatingAdmissionWebhook
  configuration:
    apiVersion: apiserver.config.k8s.io/v1
    kind: WebhookAdmissionConfiguration
    kubeConfigFile: "<path-to-kubeconfig-file>"
```
有关 AdmissionConfiguration 的更多信息，请[参见 AdmissionConfiguration schema](https://github.com/kubernetes/kubernetes/blob/v1.17.0/staging/src/k8s.io/apiserver/pkg/apis/apiserver/v1/types.go#L27)。 有关每个配置字段的详细信息，请参见 webhook 配置部分。

在 kubeConfig 文件中，提供证书凭据：

```bash
apiVersion: v1
kind: Config
users:
# 名称应设置为服务的 DNS 名称或配置了 Webhook 的 URL 的主机名（包括端口）。
# 如果将非 443 端口用于服务，则在配置 1.16+ API 服务器时，该端口必须包含在名称中。
#
# 对于配置在默认端口（443）上与服务对话的 Webhook，请指定服务的 DNS 名称：
# - name: webhook1.ns1.svc
#   user: ...
#
# 对于配置在非默认端口（例如 8443）上与服务对话的 Webhook，请在 1.16+ 中指定服务的 DNS 名称和端口：
# - name: webhook1.ns1.svc:8443
#   user: ...
# 并可以选择仅使用服务的 DNS 名称来创建第二节，以与 1.15 API 服务器版本兼容：
# - name: webhook1.ns1.svc
#   user: ...
#
# 对于配置为使用 URL 的 webhook，请匹配在 webhook 的 URL 中指定的主机（和端口）。
# 带有 `url: https://www.example.com` 的 webhook：
# - name: www.example.com
#   user: ...
#
# 带有 `url: https://www.example.com:443` 的 webhook：
# - name: www.example.com:443
#   user: ...
#
# 带有 `url: https://www.example.com:8443` 的 webhook：
# - name: www.example.com:8443
#   user: ...
#
- name: 'webhook1.ns1.svc'
  user:
    client-certificate-data: "<pem encoded certificate>"
    client-key-data: "<pem encoded key>"
# `name` 支持使用 * 通配符匹配前缀段。
- name: '*.webhook-company.org'
  user:
    password: "<password>"
    username: "<name>"
# '*' 是默认匹配项。
- name: '*'
  user:
    token: "<token>"
```
当然，你需要设置 webhook 服务器来处理这些身份验证。

## 8. 请求
向 Webhook 发送 POST 请求时，请设置 `Content-Type: application/json` 并对 `admission.k8s.io API` 组中的 `AdmissionReview` 对象进行序列化，将所得到的 JSON 作为请求的主体。

Webhook 可以在配置中的 admissionReviewVersions 字段指定可接受的 AdmissionReview 对象版本：

```bash
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
...
webhooks:
- name: my-webhook.example.com
  admissionReviewVersions: ["v1", "v1beta1"]
  ...
```
创建 `admissionregistration.k8s.io/v1 webhook` 配置时，`admissionReviewVersions` 是必填字段。 Webhook 必须支持至少一个当前和以前的 apiserver 都可以解析的 AdmissionReview 版本。

API 服务器将发送的是 `admissionReviewVersions` 列表中所支持的第一个 AdmissionReview 版本。如果 API 服务器不支持列表中的任何版本，则不允许创建配置。

如果 API 服务器遇到以前创建的 Webhook 配置，并且不支持该 API 服务器知道如何发送的任何 AdmissionReview 版本，则调用 Webhook 的尝试将失败，并依据失败策略进行处理。

此示例显示了 AdmissionReview 对象中包含的数据，该数据用于请求更新 apps/v1 Deployment 的 scale 子资源：

```bash
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "request": {
    # 唯一标识此准入回调的随机 uid
    "uid": "705ab4f5-6393-11e8-b7cc-42010a800002",

    # 传入完全正确的 group/version/kind 对象
    "kind": {"group":"autoscaling","version":"v1","kind":"Scale"},
    # 修改 resource 的完全正确的的 group/version/kind
    "resource": {"group":"apps","version":"v1","resource":"deployments"},
    # subResource（如果请求是针对 subResource 的）
    "subResource": "scale",

    # 在对 API 服务器的原始请求中，传入对象的标准 group/version/kind
    # 仅当 webhook 指定 `matchPolicy: Equivalent` 且将对 API 服务器的原始请求转换为 webhook 注册的版本时，这才与 `kind` 不同。
    "requestKind": {"group":"autoscaling","version":"v1","kind":"Scale"},
    # 在对 API 服务器的原始请求中正在修改的资源的标准 group/version/kind
    # 仅当 webhook 指定了 `matchPolicy：Equivalent` 并且将对 API 服务器的原始请求转换为 webhook 注册的版本时，这才与 `resource` 不同。
    "requestResource": {"group":"apps","version":"v1","resource":"deployments"},
    # subResource（如果请求是针对 subResource 的）
    # 仅当 webhook 指定了 `matchPolicy：Equivalent` 并且将对 API 服务器的原始请求转换为该 webhook 注册的版本时，这才与 `subResource` 不同。
    "requestSubResource": "scale",

    # 被修改资源的名称
    "name": "my-deployment",
    # 如果资源是属于名字空间（或者是名字空间对象），则这是被修改的资源的名字空间
    "namespace": "my-namespace",

    # 操作可以是 CREATE、UPDATE、DELETE 或 CONNECT
    "operation": "UPDATE",

    "userInfo": {
      # 向 API 服务器发出请求的经过身份验证的用户的用户名
      "username": "admin",
      # 向 API 服务器发出请求的经过身份验证的用户的 UID
      "uid": "014fbff9a07c",
      # 向 API 服务器发出请求的经过身份验证的用户的组成员身份
      "groups": ["system:authenticated","my-admin-group"],
      # 向 API 服务器发出请求的用户相关的任意附加信息
      # 该字段由 API 服务器身份验证层填充，并且如果 webhook 执行了任何 SubjectAccessReview 检查，则应将其包括在内。
      "extra": {
        "some-key":["some-value1", "some-value2"]
      }
    },

    # object 是被接纳的新对象。
    # 对于 DELETE 操作，它为 null。
    "object": {"apiVersion":"autoscaling/v1","kind":"Scale",...},
    # oldObject 是现有对象。
    # 对于 CREATE 和 CONNECT 操作，它为 null。
    "oldObject": {"apiVersion":"autoscaling/v1","kind":"Scale",...},
    # options 包含要接受的操作的选项，例如 meta.k8s.io/v CreateOptions、UpdateOptions 或 DeleteOptions。
    # 对于 CONNECT 操作，它为 null。
    "options": {"apiVersion":"meta.k8s.io/v1","kind":"UpdateOptions",...},

    # dryRun 表示 API 请求正在以 `dryrun` 模式运行，并且将不会保留。
    # 带有副作用的 Webhook 应该避免在 dryRun 为 true 时激活这些副作用。
    # 有关更多详细信息，请参见 http://k8s.io/docs/reference/using-api/api-concepts/#make-a-dry-run-request
    "dryRun": false
  }
}
```
## 9. 响应
Webhook 使用 HTTP 200 状态码、`Content-Type: application/json` 和一个包含 `AdmissionReview` 对象的 JSON 序列化格式来发送响应。该 AdmissionReview 对象与发送的版本相同，且其中包含的 response 字段已被有效填充。

`response` 至少必须包含以下字段：

 - uid，从发送到 webhook 的 request.uid 中复制而来
 - allowed，设置为 true 或 false

Webhook 允许请求的最简单响应示例：

```bash
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true
  }
}
```
Webhook 禁止请求的最简单响应示例：

```bash
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": false
  }
}
```
当拒绝请求时，Webhook 可以使用 status 字段自定义 http 响应码和返回给用户的消息。 有关状态类型的详细信息，请参见 API 文档。 禁止请求的响应示例，它定制了向用户显示的 HTTP 状态码和消息：

```bash
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": false,
    "status": {
      "code": 403,
      "message": "You cannot do this because it is Tuesday and your name starts with A"
    }
  }
}
```
当允许请求时，mutating准入 Webhook 也可以选择修改传入的对象。 这是通过在响应中使用 `patch` 和 `patchType` 字段来完成的。 当前唯一支持的 patchType 是 JSONPatch。 有关更多详细信息，请参见 JSON patch。 对于 `patchType: JSONPatch`，patch 字段包含一个以 base64 编码的 JSON patch 操作数组。

例如，设置 spec.replicas 的单个补丁操作将是 `[{"op": "add", "path": "/spec/replicas", "value": 3}]`。

如果以 Base64 形式编码，结果将是 `W3sib3AiOiAiYWRkIiwgInBhdGgiOiAiL3NwZWMvcmVwbGljYXMiLCAidmFsdWUiOiAzfV0=`

因此，添加该标签的 webhook 响应为：

```bash
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true,
    "patchType": "JSONPatch",
    "patch": "W3sib3AiOiAiYWRkIiwgInBhdGgiOiAiL3NwZWMvcmVwbGljYXMiLCAidmFsdWUiOiAzfV0="
  }
```
相关性阅读：
[k8s 准入控制器【1】--介绍](https://ghostwritten.blog.csdn.net/article/details/112242163)
[k8s 准入控制器【2】--动态准入控制](https://ghostwritten.blog.csdn.net/article/details/112258669)
[k8s 准入控制器【3】--编写和部署准入控制器 Webhook](https://ghostwritten.blog.csdn.net/article/details/112280103)
