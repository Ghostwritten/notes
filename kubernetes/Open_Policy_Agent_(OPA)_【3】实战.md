# Open Policy Agent (OPA) 实战
tags: OPA,策略
<!-- catalog: ~实战~ -->



![在这里插入图片描述](https://img-blog.csdnimg.cn/ca9435bb76cf4bdea2c5cf8055fc9803.png)








## 1. 安装 OPA
[gatekeeper.yaml 下载](https://github.com/killer-sh/cks-course-environment/blob/master/course-content/opa/gatekeeper.yaml)
```bash
root@master:~/cks/opa# k create -f gatekeeper.yaml 
namespace/gatekeeper-system created
Warning: apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
customresourcedefinition.apiextensions.k8s.io/configs.config.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constraintpodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplatepodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplates.templates.gatekeeper.sh created
serviceaccount/gatekeeper-admin created
role.rbac.authorization.k8s.io/gatekeeper-manager-role created
clusterrole.rbac.authorization.k8s.io/gatekeeper-manager-role created
rolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
secret/gatekeeper-webhook-server-cert created
service/gatekeeper-webhook-service created
deployment.apps/gatekeeper-audit created
deployment.apps/gatekeeper-controller-manager created
Warning: admissionregistration.k8s.io/v1beta1 ValidatingWebhookConfiguration is deprecated in v1.16+, unavailable in v1.22+; use admissionregistration.k8s.io/v1 ValidatingWebhookConfiguration
validatingwebhookconfiguration.admissionregistration.k8s.io/gatekeeper-validating-webhook-configuration created


root@master:~/cks/opa# k get ns
NAME                STATUS   AGE
default             Active   2d5h
gatekeeper-system   Active   48s
kube-node-lease     Active   2d5h
kube-public         Active   2d5h
kube-system         Active   2d5h

root@master:~/cks/opa# k get pod,svc -n gatekeeper-system
NAME                                                 READY   STATUS    RESTARTS   AGE
pod/gatekeeper-audit-65f658df68-jql6v                1/1     Running   0          74s
pod/gatekeeper-controller-manager-5fb6c9ff69-srvg7   1/1     Running   0          39m

NAME                                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/gatekeeper-webhook-service   ClusterIP   10.98.47.59   <none>        443/TCP   39m

```
参考：
[Dynamic Admission Control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)


## 2. Deny All Policy
任务
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210516231736609.png)

```c
root@master:~/cks/opa# k get crd 
NAME                                                  CREATED AT
bgpconfigurations.crd.projectcalico.org               2021-05-14T08:50:42Z
bgppeers.crd.projectcalico.org                        2021-05-14T08:50:42Z
blockaffinities.crd.projectcalico.org                 2021-05-14T08:50:42Z
clusterinformations.crd.projectcalico.org             2021-05-14T08:50:42Z
configs.config.gatekeeper.sh                          2021-05-16T14:33:44Z
constraintpodstatuses.status.gatekeeper.sh            2021-05-16T14:33:44Z
constrainttemplatepodstatuses.status.gatekeeper.sh    2021-05-16T14:33:44Z
constrainttemplates.templates.gatekeeper.sh           2021-05-16T14:33:44Z
felixconfigurations.crd.projectcalico.org             2021-05-14T08:50:42Z
globalnetworkpolicies.crd.projectcalico.org           2021-05-14T08:50:42Z
globalnetworksets.crd.projectcalico.org               2021-05-14T08:50:42Z
hostendpoints.crd.projectcalico.org                   2021-05-14T08:50:42Z
ipamblocks.crd.projectcalico.org                      2021-05-14T08:50:42Z
ipamconfigs.crd.projectcalico.org                     2021-05-14T08:50:42Z
ipamhandles.crd.projectcalico.org                     2021-05-14T08:50:42Z
ippools.crd.projectcalico.org                         2021-05-14T08:50:42Z
kubecontrollersconfigurations.crd.projectcalico.org   2021-05-14T08:50:42Z
networkpolicies.crd.projectcalico.org                 2021-05-14T08:50:43Z
networksets.crd.projectcalico.org                     2021-05-14T08:50:43Z


root@master:~/cks/opa# k get constrainttemplates
No resources found


root@master:~/cks/opa# vim template.yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8salwaysdeny
spec:
  crd:
    spec:
      names:
        kind: K8sAlwaysDeny
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            message:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8salwaysdeny

        violation[{"msg": msg}] {
          1 > 0
          msg := input.parameters.message
        }


root@master:~/cks/opa# k create -f template.yaml 
constrainttemplate.templates.gatekeeper.sh/k8salwaysdeny created
root@master:~/cks/opa# k get k8salwaysdeny
No resources found


root@master:~/cks/opa# k get constrainttemplates
NAME            AGE
k8salwaysdeny   90s

root@master:~/cks/opa# vim constraint.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAlwaysDeny
metadata:
  name: pod-always-deny
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    message: "ACCESS DENIED!"



root@master:~/cks/opa# k get k8sAlwaysDeny
NAME              AGE
pod-always-deny   19s

root@master:~/cks/opa# k run pod --image=nginx
Error from server ([denied by pod-always-deny] ACCESS DENIED!): admission webhook "validation.gatekeeper.sh" denied the request: [denied by pod-always-deny] ACCESS DENIED!


root@master:~/cks/opa# vim constraint.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAlwaysDeny
metadata:
  name: pod-always-deny
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    message: "NO WAY!"   #修改此行



root@master:~/cks/opa# k -f constraint.yaml replace
k8salwaysdeny.constraints.gatekeeper.sh/pod-always-deny replaced
root@master:~/cks/opa# k run pod --image=nginx
Error from server ([denied by pod-always-deny] NO WAY!): admission webhook "validation.gatekeeper.sh" denied the request: [denied by pod-always-deny] NO WAY!



root@master:~/cks/opa# k describe k8sAlwaysDeny pod-always-deny
Name:         pod-always-deny
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  constraints.gatekeeper.sh/v1beta1
Kind:         K8sAlwaysDeny
Metadata:
  Creation Timestamp:  2021-05-20T06:10:39Z
  Generation:          2
  Managed Fields:
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:match:
          .:
          f:kinds:
        f:parameters:
          .:
          f:message:
    Manager:      kubectl-create
    Operation:    Update
    Time:         2021-05-20T06:10:39Z
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        f:parameters:
          f:message:
    Manager:      kubectl-replace
    Operation:    Update
    Time:         2021-05-20T06:14:29Z
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        f:auditTimestamp:
        f:byPod:
        f:violations:
    Manager:         gatekeeper
    Operation:       Update
    Time:            2021-05-20T06:15:24Z
  Resource Version:  80590
  UID:               064073ac-19fd-4eaa-96d3-d97a640c4f57
Spec:
  Match:
    Kinds:
      API Groups:
        
      Kinds:
        Pod
  Parameters:
    Message:  NO WAY!
Status:
  Audit Timestamp:  2021-05-20T06:18:31Z
  By Pod:
    Constraint UID:       064073ac-19fd-4eaa-96d3-d97a640c4f57
    Enforced:             true
    Id:                   gatekeeper-audit-65f658df68-jql6v
    Observed Generation:  2
    Operations:
      audit
      status
    Constraint UID:       064073ac-19fd-4eaa-96d3-d97a640c4f57
    Enforced:             true
    Id:                   gatekeeper-controller-manager-5fb6c9ff69-srvg7
    Observed Generation:  2
    Operations:
      webhook
  Total Violations:  16
  Violations:
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                app
    Namespace:           default
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                gatekeeper-audit-65f658df68-jql6v
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                gatekeeper-controller-manager-5fb6c9ff69-srvg7
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                calico-kube-controllers-57fc9c76cc-t45js
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                calico-node-jkfm9
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                calico-node-lrkbp
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                calico-node-pdwqm
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                coredns-74ff55c5b-gmfsj
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                coredns-74ff55c5b-s6lt8
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                etcd-master
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                kube-apiserver-master
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                kube-controller-manager-master
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                kube-proxy-6m8q6
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                kube-proxy-8vprs
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                kube-proxy-b6r2s
    Namespace:           kube-system
    Enforcement Action:  deny
    Kind:                Pod
    Message:             NO WAY!
    Name:                kube-scheduler-master
    Namespace:           kube-system
Events:                  <none>



root@master:~/cks/opa# cat template.yaml 
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8salwaysdeny
spec:
  crd:
    spec:
      names:
        kind: K8sAlwaysDeny
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            message:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8salwaysdeny

        violation[{"msg": msg}] {
          1 > 0
          1 > 2   #添加此行
          msg := input.parameters.message
        }


root@master:~/cks/opa# k -f template.yaml  replace
constrainttemplate.templates.gatekeeper.sh/k8salwaysdeny replaced
root@master:~/cks/opa# k run pod --image=nginx
pod/pod created




root@master:~/cks/opa# k describe k8sAlwaysDeny pod-always-deny
Name:         pod-always-deny
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  constraints.gatekeeper.sh/v1beta1
Kind:         K8sAlwaysDeny
Metadata:
  Creation Timestamp:  2021-05-20T06:10:39Z
  Generation:          2
  Managed Fields:
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:match:
          .:
          f:kinds:
        f:parameters:
          .:
          f:message:
    Manager:      kubectl-create
    Operation:    Update
    Time:         2021-05-20T06:10:39Z
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        f:parameters:
          f:message:
    Manager:      kubectl-replace
    Operation:    Update
    Time:         2021-05-20T06:14:29Z
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        f:auditTimestamp:
        f:byPod:
        f:totalViolations:
    Manager:         gatekeeper
    Operation:       Update
    Time:            2021-05-20T06:21:42Z
  Resource Version:  80871
  UID:               064073ac-19fd-4eaa-96d3-d97a640c4f57
Spec:
  Match:
    Kinds:
      API Groups:
        
      Kinds:
        Pod
  Parameters:
    Message:  NO WAY!
Status:
  Audit Timestamp:  2021-05-20T06:21:40Z
  By Pod:
    Constraint UID:       064073ac-19fd-4eaa-96d3-d97a640c4f57
    Enforced:             true
    Id:                   gatekeeper-audit-65f658df68-jql6v
    Observed Generation:  2
    Operations:
      audit
      status
    Constraint UID:       064073ac-19fd-4eaa-96d3-d97a640c4f57
    Enforced:             true
    Id:                   gatekeeper-controller-manager-5fb6c9ff69-srvg7
    Observed Generation:  2
    Operations:
      webhook
  Total Violations:  0
Events:              <none>

```


## 3. Enforce Namespace Labels
![在这里插入图片描述](https://img-blog.csdnimg.cn/202105201423110.png)

```c
root@master:~/cks/opa# vim template_label.yaml 
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }



root@master:~/cks/opa# k -f template_label.yaml create
constrainttemplate.templates.gatekeeper.sh/k8srequiredlabels created




root@master:~/cks/opa# vim all_ns_must_have_cks.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-cks
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["cks"]



root@master:~/cks/opa# k -f all_ns_must_have_cks.yaml create
k8srequiredlabels.constraints.gatekeeper.sh/ns-must-have-cks created
root@master:~/cks/opa# k get K8sRequiredLabels
NAME               AGE
ns-must-have-cks   2m39s


root@master:~/cks/opa# k describe K8sRequiredLabels ns-must-have-cks
Name:         ns-must-have-cks
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  constraints.gatekeeper.sh/v1beta1
Kind:         K8sRequiredLabels
Metadata:
  Creation Timestamp:  2021-05-20T06:42:27Z
  Generation:          1
  Managed Fields:
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:match:
          .:
          f:kinds:
        f:parameters:
          .:
          f:labels:
    Manager:      kubectl-create
    Operation:    Update
    Time:         2021-05-20T06:42:27Z
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        .:
        f:auditTimestamp:
        f:byPod:
        f:totalViolations:
        f:violations:
    Manager:         gatekeeper
    Operation:       Update
    Time:            2021-05-20T06:42:38Z
  Resource Version:  85857
  UID:               4938d3e2-b926-4e60-ade6-ab957c92a8c2
Spec:
  Match:
    Kinds:
      API Groups:
        
      Kinds:
        Namespace
  Parameters:
    Labels:
      cks
Status:
  Audit Timestamp:  2021-05-20T07:20:11Z
  By Pod:
    Constraint UID:       4938d3e2-b926-4e60-ade6-ab957c92a8c2
    Enforced:             true
    Id:                   gatekeeper-audit-65f658df68-jql6v
    Observed Generation:  1
    Operations:
      audit
      status
    Constraint UID:       4938d3e2-b926-4e60-ade6-ab957c92a8c2
    Enforced:             true
    Id:                   gatekeeper-controller-manager-5fb6c9ff69-srvg7
    Observed Generation:  1
    Operations:
      webhook
  Total Violations:  5
  Violations:
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"cks"}
    Name:                default
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"cks"}
    Name:                gatekeeper-system
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"cks"}
    Name:                kube-node-lease
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"cks"}
    Name:                kube-public
    Enforcement Action:  deny
    Kind:                Namespace
    Message:             you must provide labels: {"cks"}
    Name:                kube-system
Events:                  <none>


root@master:~/cks/opa# k  edit ns default
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-05-14T08:37:43Z"
  labels:   # 添加此行
    cks: amazing   # 添加此行
  name: default
  resourceVersion: "86550"
  uid: 773025bb-2f60-40f9-a10d-924c21e45016
spec:
  finalizers:
  - kubernetes
status:
  phase: Active



root@master:~/cks/opa# k describe K8sRequiredLabels ns-must-have-cks
......................
  Total Violations:  4
......................



root@master:~/cks/opa# k create ns test
Error from server ([denied by ns-must-have-cks] you must provide labels: {"cks"}): admission webhook "validation.gatekeeper.sh" denied the request: [denied by ns-must-have-cks] you must provide labels: {"cks"}


root@master:~/cks/opa# vim all_ns_must_have_cks.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-cks
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["cks","team"]  #修改此行



root@master:~/cks/opa# k -f all_ns_must_have_cks.yaml replace
k8srequiredlabels.constraints.gatekeeper.sh/ns-must-have-cks replaced
root@master:~/cks/opa# k create ns test
Error from server ([denied by ns-must-have-cks] you must provide labels: {"cks", "team"}): admission webhook "validation.gatekeeper.sh" denied the request: [denied by ns-must-have-cks] you must provide labels: {"cks", "team"}



root@master:~/cks/opa# k create ns test -o yaml --dry-run=client
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: test
spec: {}
status: {}
root@master:~/cks/opa# k create ns test -o yaml --dry-run=client > ns.yaml

oot@master:~/cks/opa# vim ns.yaml 
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: test
  labels:    #添加此行
    cks: amazing  #添加此行
    team: killer-sh  #添加此行
spec: {}
status: {}

#创建namespace成功
root@master:~/cks/opa# k create -f ns.yaml 
namespace/test created

```
	
## 4. Enforce Deployment replica count
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210520153714413.png)

```c
root@master:~/cks/opa# cat template_replicas.yaml 
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8sminreplicacount
spec:
  crd:
    spec:
      names:
        kind: K8sMinReplicaCount
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            min:
              type: integer
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sminreplicacount

        violation[{"msg": msg, "details": {"missing_replicas": missing}}] {
          provided := input.review.object.spec.replicas
          required := input.parameters.min
          missing := required - provided
          missing > 0
          msg := sprintf("you must provide %v more replicas", [missing])
        }




root@master:~/cks/opa# k -f  template_replicas.yaml  create
constrainttemplate.templates.gatekeeper.sh/k8sminreplicacount created


root@master:~/cks/opa# cat all_deployment_must_have_min_replicacount.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sMinReplicaCount
metadata:
  name: deployment-must-have-min-replicas
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    min: 2


root@master:~/cks/opa# k -f  all_deployment_must_have_min_replicacount.yaml  create
k8sminreplicacount.constraints.gatekeeper.sh/deployment-must-have-min-replicas created
root@master:~/cks/opa# k get k8sminreplicacount
NAME                                AGE
deployment-must-have-min-replicas   8s



root@master:~/cks/opa# k describe k8sminreplicacount deployment-must-have-min-replicas
Name:         deployment-must-have-min-replicas
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  constraints.gatekeeper.sh/v1beta1
Kind:         K8sMinReplicaCount
Metadata:
  Creation Timestamp:  2021-05-20T07:44:35Z
  Generation:          1
  Managed Fields:
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:spec:
        .:
        f:match:
          .:
          f:kinds:
        f:parameters:
          .:
          f:min:
    Manager:      kubectl-create
    Operation:    Update
    Time:         2021-05-20T07:44:35Z
    API Version:  constraints.gatekeeper.sh/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:status:
        .:
        f:auditTimestamp:
        f:byPod:
        f:totalViolations:
        f:violations:
    Manager:         gatekeeper
    Operation:       Update
    Time:            2021-05-20T07:45:11Z
  Resource Version:  88195
  UID:               318fd47c-d885-4cdb-ab87-e827bd7154ae
Spec:
  Match:
    Kinds:
      API Groups:
        apps
      Kinds:
        Deployment
  Parameters:
    Min:  2
Status:
  Audit Timestamp:  2021-05-20T07:47:12Z
  By Pod:
    Constraint UID:       318fd47c-d885-4cdb-ab87-e827bd7154ae
    Enforced:             true
    Id:                   gatekeeper-audit-65f658df68-jql6v
    Observed Generation:  1
    Operations:
      audit
      status
    Constraint UID:       318fd47c-d885-4cdb-ab87-e827bd7154ae
    Enforced:             true
    Id:                   gatekeeper-controller-manager-5fb6c9ff69-srvg7
    Observed Generation:  1
    Operations:
      webhook
  Total Violations:  3
  Violations:
    Enforcement Action:  deny
    Kind:                Deployment
    Message:             you must provide 1 more replicas
    Name:                gatekeeper-audit
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Deployment
    Message:             you must provide 1 more replicas
    Name:                gatekeeper-controller-manager
    Namespace:           gatekeeper-system
    Enforcement Action:  deny
    Kind:                Deployment
    Message:             you must provide 1 more replicas
    Name:                calico-kube-controllers
    Namespace:           kube-system
Events:                  <none>


#无法创建
root@master:~/cks/opa# k create deploy test --image=nginx
error: failed to create deployment: admission webhook "validation.gatekeeper.sh" denied the request: [denied by deployment-must-have-min-replicas] you must provide 1 more replicas


root@master:~/cks/opa# k create deploy test --image=nginx -oyaml --dry-run=client > deploy.yaml
root@master:~/cks/opa# vim deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 2   #修改为2
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}

#创建成功
root@master:~/cks/opa# k -f deploy.yaml create
deployment.apps/test created

```
## 5. The Rego 练习
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210520162043625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
[rego playgroud](https://play.openpolicyagent.org/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210520162434756.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


参考：



 - [Open Policy Agent(OPA) 【1】介绍](https://blog.csdn.net/xixihahalelehehe/article/details/116905513?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164188925916780271953559%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164188925916780271953559&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-116905513.nonecase&utm_term=opa&spm=1018.2226.3001.4450)
 - [Open Policy Agent(OPA) 【2】rego语法](https://blog.csdn.net/xixihahalelehehe/article/details/116998878?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164188925916780271953559%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164188925916780271953559&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-116998878.nonecase&utm_term=opa&spm=1018.2226.3001.4450)
 - [Open Policy Agent(OPA) 【3】实战](https://blog.csdn.net/xixihahalelehehe/article/details/116904422?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164188925916780271953559%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164188925916780271953559&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-116904422.nonecase&utm_term=opa&spm=1018.2226.3001.4450)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)
 - [5 tips for using the Rego language for Open Policy Agent (OPA)](https://www.fugue.co/blog/5-tips-for-using-the-rego-language-for-open-policy-agent-opa)
 - [Policy Primer via Examples](https://www.openpolicyagent.org/docs/latest/kubernetes-primer/#detailed-admission-control-flow)
 - [Open Policy Agent: What Is OPA and How It Works (Examples)](https://spacelift.io/blog/what-is-open-policy-agent-and-how-it-works)
