![](https://i-blog.csdnimg.cn/blog_migrate/768f06f96c3ec8366d548b5cef2d9de6.png)






## 1. 简介
[Rancher](https://ranchermanager.docs.rancher.com/) 是一个开源的企业级全栈化容器部署及管理平台。已有超过 1900 万次下载，4000+ 生产环境的应用。

简单的说，就是一个可以让你通过 web 界面管理 docker 容器的平台。定位上和 K8s 比较接近，都是通过 web 界面赋予完全的 docker 服务编排功能。

特色：

- 平台部署方便。管理 docker 的平台本身也基于 docker 部署。只要你有 docker ，一句命令就完成平台的部署了。

- 平台扩展方便。通过 agent 机制，一句 docker 命令完成 agent 部署，快速增加你的物理机。同时也支持 AWS 等云主机， 2.0 版本甚至还支持 K8s 。

- 服务部署方便。通过应用商店，2 步完成应用部署，而且还是像 docker-compose 那样各个中间件独立编排，可以随时扩容的哦。

- 自带账户权限。相比 K8s 没有账号管理，rancher 自带账号权限体系。账号可以独立创建，也可以很方便地接入 ldap 等账号体系。对于公司使用是一大利器
## 2. 预备条件

- [部署kubernetes cluster](https://ghostwritten.blog.csdn.net/article/details/131749277)

## 3. 创建存储目录

```bash
ansible all -m shell -a "mkdir -p /data/rancher"
```

## 4. 部署 rancher server

```bash
$ vim rancher-deploy.yaml
```

```bash
---
apiVersion: v1
kind: Namespace
metadata:
  name: rancher
  labels:
    app: rancher
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rancher
  namespace: rancher
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rancher
  template:
    metadata:
      labels:
        app: rancher
    spec:
      containers:
        - name: rancher
          image: rancher/rancher:v2.7.5
          imagePullPolicy: IfNotPresent
          resources:
            limits:
              cpu: "2"
              memory: 2Gi
            requests:
              cpu: 250m
              memory: 512Mi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          ports:
            - containerPort: 80
            - containerPort: 443
          volumeMounts:
            - name: rancher-data
              mountPath: /var/lib/rancher
          env:
            - name: CATTLE_SYSTEM_CATALOG
              value: "bundled"
          securityContext:
            privileged: true
      volumes:
        - name: rancher-data
          hostPath:
            path: /data/rancher

---
apiVersion: v1
kind: Service
metadata:
  name: rancher-service
  namespace: rancher
spec:
  type: NodePort
  selector:
    app: rancher
  ports:
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
      nodePort: 30010
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30011
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: null
  name: permissive-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: admin
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: kubelet
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: kube-system
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: default
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: rancher
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:serviceaccounts
```

执行：

```bash
kubectl apply -f rancher-deploy.yaml
```
查看
```bash
查看pod 状态
$ kubectl get pod -n rancher
NAME                       READY   STATUS    RESTARTS      AGE
rancher-85bc967c78-sr5wc   1/1     Running   4 (24m ago)   27m

查看自动创建的ns
[root@bastion01 rancher]# kubectl get ns |grep cattle
cattle-fleet-clusters-system   Active   22m
cattle-fleet-system            Active   24m
cattle-global-data             Active   25m
cattle-global-nt               Active   25m
cattle-impersonation-system    Active   25m
cattle-system                  Active   26m

查看每个命名空间的内容
[root@bastion01 rancher]# kubectl get all -n cattle-fleet-clusters-system 
No resources found in cattle-fleet-clusters-system namespace.
[root@bastion01 rancher]# kubectl get all -n cattle-global-data 
No resources found in cattle-global-data namespace.
[root@bastion01 rancher]# kubectl get all -n cattle-global-nt
No resources found in cattle-global-nt namespace.
[root@bastion01 rancher]# kubectl get all -n cattle-impersonation-system
No resources found in cattle-impersonation-system namespace.
[root@bastion01 rancher]# kubectl get all -n cattle-system
NAME                                  READY   STATUS      RESTARTS   AGE
pod/helm-operation-jjjhp              0/2     Completed   0          22m
pod/helm-operation-nq9ml              0/2     Completed   0          23m
pod/helm-operation-s8mzn              0/2     Completed   0          25m
pod/helm-operation-w26st              0/2     Completed   0          24m
pod/rancher-webhook-c85db8b75-fszxl   1/1     Running     0          22m

NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/rancher-webhook   ClusterIP   10.233.43.93    <none>        443/TCP   22m
service/webhook-service   ClusterIP   10.233.56.141   <none>        443/TCP   22m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rancher-webhook   1/1     1            1           22m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/rancher-webhook-c85db8b75   1         1         1       22m

```

每个命名空间的pod或许是执行任务类型，启动不久便消失了。

查看所需的镜像

```bash
[root@bastion01 rancher]# ansible all -m shell -a "crictl images |grep rancher"
kube-node02 | CHANGED | rc=0 >>
docker.io/rancher/fleet                                 v0.7.0              fdcac788e7415       66.7MB
docker.io/rancher/gitjob                                v0.1.54             4ea55b57283d4       90.5MB
docker.io/rancher/shell                                 v0.1.20             5411540943b48       135MB
kube-node01 | FAILED | rc=1 >>
non-zero return code
kube-master03 | FAILED | rc=1 >>
non-zero return code
kube-master02 | FAILED | rc=1 >>
non-zero return code
kube-master01 | FAILED | rc=1 >>
non-zero return code
bastion01 | FAILED | rc=1 >>
/bin/sh: crictl: command not foundnon-zero return code
kube-node04 | CHANGED | rc=0 >>
docker.io/rancher/shell                                 v0.1.20             5411540943b48       135MB
kube-node03 | CHANGED | rc=0 >>
docker.io/rancher/rancher-webhook                       v0.3.5              c83575d430914       25.3MB
docker.io/rancher/rancher                               v2.7.5              81ee0878ffcdc       689MB
docker.io/rancher/shell                                 v0.1.20             5411540943b48       135MB

```
查看创建的crd

```bash
[root@bastion01 rancher]# kubectl get crd |grep cattle
amazonec2configs.rke-machine-config.cattle.io                     2023-08-30T09:48:36Z
amazonec2machines.rke-machine.cattle.io                           2023-08-30T09:48:36Z
amazonec2machinetemplates.rke-machine.cattle.io                   2023-08-30T09:48:36Z
apiservices.management.cattle.io                                  2023-08-30T09:47:22Z
apprevisions.project.cattle.io                                    2023-08-30T09:47:46Z
apps.catalog.cattle.io                                            2023-08-30T09:47:23Z
apps.project.cattle.io                                            2023-08-30T09:47:46Z
authconfigs.management.cattle.io                                  2023-08-30T09:47:34Z
azureconfigs.rke-machine-config.cattle.io                         2023-08-30T09:48:35Z
azuremachines.rke-machine.cattle.io                               2023-08-30T09:48:35Z
azuremachinetemplates.rke-machine.cattle.io                       2023-08-30T09:48:35Z
bundledeployments.fleet.cattle.io                                 2023-08-30T09:51:00Z
bundlenamespacemappings.fleet.cattle.io                           2023-08-30T09:51:00Z
bundles.fleet.cattle.io                                           2023-08-30T09:47:23Z
catalogs.management.cattle.io                                     2023-08-30T09:47:46Z
catalogtemplates.management.cattle.io                             2023-08-30T09:47:46Z
catalogtemplateversions.management.cattle.io                      2023-08-30T09:47:46Z
clusteralertgroups.management.cattle.io                           2023-08-30T09:47:46Z
clusteralertrules.management.cattle.io                            2023-08-30T09:47:46Z
clusteralerts.management.cattle.io                                2023-08-30T09:47:46Z
clustercatalogs.management.cattle.io                              2023-08-30T09:47:46Z
clustergroups.fleet.cattle.io                                     2023-08-30T09:47:24Z
clustermonitorgraphs.management.cattle.io                         2023-08-30T09:47:46Z
clusterregistrations.fleet.cattle.io                              2023-08-30T09:51:02Z
clusterregistrationtokens.fleet.cattle.io                         2023-08-30T09:51:02Z
clusterregistrationtokens.management.cattle.io                    2023-08-30T09:47:22Z
clusterrepos.catalog.cattle.io                                    2023-08-30T09:47:23Z
clusterroletemplatebindings.management.cattle.io                  2023-08-30T09:47:46Z
clusters.fleet.cattle.io                                          2023-08-30T09:47:23Z
clusters.management.cattle.io                                     2023-08-30T09:47:22Z
clusters.provisioning.cattle.io                                   2023-08-30T09:47:24Z
clustertemplaterevisions.management.cattle.io                     2023-08-30T09:47:46Z
clustertemplates.management.cattle.io                             2023-08-30T09:47:46Z
composeconfigs.management.cattle.io                               2023-08-30T09:47:46Z
contents.fleet.cattle.io                                          2023-08-30T09:51:03Z
custommachines.rke.cattle.io                                      2023-08-30T09:47:26Z
digitaloceanconfigs.rke-machine-config.cattle.io                  2023-08-30T09:48:36Z
digitaloceanmachines.rke-machine.cattle.io                        2023-08-30T09:48:36Z
digitaloceanmachinetemplates.rke-machine.cattle.io                2023-08-30T09:48:36Z
dynamicschemas.management.cattle.io                               2023-08-30T09:47:46Z
etcdbackups.management.cattle.io                                  2023-08-30T09:47:46Z
etcdsnapshots.rke.cattle.io                                       2023-08-30T09:47:26Z
features.management.cattle.io                                     2023-08-30T09:47:13Z
fleetworkspaces.management.cattle.io                              2023-08-30T09:47:23Z
gitjobs.gitjob.cattle.io                                          2023-08-30T09:51:25Z
gitreporestrictions.fleet.cattle.io                               2023-08-30T09:51:03Z
gitrepos.fleet.cattle.io                                          2023-08-30T09:51:02Z
globaldnses.management.cattle.io                                  2023-08-30T09:47:46Z
globaldnsproviders.management.cattle.io                           2023-08-30T09:47:46Z
globalrolebindings.management.cattle.io                           2023-08-30T09:47:46Z
globalroles.management.cattle.io                                  2023-08-30T09:47:46Z
groupmembers.management.cattle.io                                 2023-08-30T09:47:34Z
groups.management.cattle.io                                       2023-08-30T09:47:34Z
harvesterconfigs.rke-machine-config.cattle.io                     2023-08-30T09:48:35Z
harvestermachines.rke-machine.cattle.io                           2023-08-30T09:48:35Z
harvestermachinetemplates.rke-machine.cattle.io                   2023-08-30T09:48:35Z
imagescans.fleet.cattle.io                                        2023-08-30T09:51:03Z
kontainerdrivers.management.cattle.io                             2023-08-30T09:47:46Z
linodeconfigs.rke-machine-config.cattle.io                        2023-08-30T09:48:35Z
linodemachines.rke-machine.cattle.io                              2023-08-30T09:48:35Z
linodemachinetemplates.rke-machine.cattle.io                      2023-08-30T09:48:35Z
managedcharts.management.cattle.io                                2023-08-30T09:47:24Z
monitormetrics.management.cattle.io                               2023-08-30T09:47:46Z
multiclusterapprevisions.management.cattle.io                     2023-08-30T09:47:46Z
multiclusterapps.management.cattle.io                             2023-08-30T09:47:46Z
navlinks.ui.cattle.io                                             2023-08-30T09:47:22Z
nodedrivers.management.cattle.io                                  2023-08-30T09:47:46Z
nodepools.management.cattle.io                                    2023-08-30T09:47:46Z
nodes.management.cattle.io                                        2023-08-30T09:47:46Z
nodetemplates.management.cattle.io                                2023-08-30T09:47:46Z
notifiers.management.cattle.io                                    2023-08-30T09:47:46Z
operations.catalog.cattle.io                                      2023-08-30T09:47:23Z
podsecurityadmissionconfigurationtemplates.management.cattle.io   2023-08-30T09:47:22Z
podsecuritypolicytemplateprojectbindings.management.cattle.io     2023-08-30T09:47:46Z
podsecuritypolicytemplates.management.cattle.io                   2023-08-30T09:47:46Z
preferences.management.cattle.io                                  2023-08-30T09:47:22Z
projectalertgroups.management.cattle.io                           2023-08-30T09:47:46Z
projectalertrules.management.cattle.io                            2023-08-30T09:47:46Z
projectalerts.management.cattle.io                                2023-08-30T09:47:46Z
projectcatalogs.management.cattle.io                              2023-08-30T09:47:46Z
projectmonitorgraphs.management.cattle.io                         2023-08-30T09:47:46Z
projectnetworkpolicies.management.cattle.io                       2023-08-30T09:47:46Z
projectroletemplatebindings.management.cattle.io                  2023-08-30T09:47:46Z
projects.management.cattle.io                                     2023-08-30T09:47:46Z
rancherusernotifications.management.cattle.io                     2023-08-30T09:47:46Z
rkeaddons.management.cattle.io                                    2023-08-30T09:47:46Z
rkebootstraps.rke.cattle.io                                       2023-08-30T09:47:25Z
rkebootstraptemplates.rke.cattle.io                               2023-08-30T09:47:25Z
rkeclusters.rke.cattle.io                                         2023-08-30T09:47:25Z
rkecontrolplanes.rke.cattle.io                                    2023-08-30T09:47:25Z
rkek8sserviceoptions.management.cattle.io                         2023-08-30T09:47:46Z
rkek8ssystemimages.management.cattle.io                           2023-08-30T09:47:46Z
roletemplates.management.cattle.io                                2023-08-30T09:47:46Z
samltokens.management.cattle.io                                   2023-08-30T09:47:46Z
settings.management.cattle.io                                     2023-08-30T09:47:22Z
templatecontents.management.cattle.io                             2023-08-30T09:47:46Z
templates.management.cattle.io                                    2023-08-30T09:47:46Z
templateversions.management.cattle.io                             2023-08-30T09:47:46Z
tokens.management.cattle.io                                       2023-08-30T09:47:34Z
userattributes.management.cattle.io                               2023-08-30T09:47:34Z
users.management.cattle.io                                        2023-08-30T09:47:34Z
vmwarevsphereconfigs.rke-machine-config.cattle.io                 2023-08-30T09:48:35Z
vmwarevspheremachines.rke-machine.cattle.io                       2023-08-30T09:48:35Z
vmwarevspheremachinetemplates.rke-machine.cattle.io               2023-08-30T09:48:35Z

```

查看日志
```bash
2023/08/30 09:47:12 [INFO] Rancher version v2.7.5 (eef55334b) is starting
2023/08/30 09:47:12 [INFO] Rancher arguments {ACMEDomains:[] AddLocal:true Embedded:false BindHost: HTTPListenPort:80 HTTPSListenPort:443 K8sMode:auto Debug:false Trace:false NoCACerts:false Audit
LogPath:/var/log/auditlog/rancher-api-audit.log AuditLogMaxage:10 AuditLogMaxsize:100 AuditLogMaxbackup:10 AuditLevel:0 Features: ClusterRegistry:}
2023/08/30 09:47:12 [INFO] Listening on /tmp/log.sock
2023/08/30 09:47:12 [INFO] Running in single server mode, will not peer connections
2023/08/30 09:47:13 [INFO] Applying CRD features.management.cattle.io
2023/08/30 09:47:13 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:14 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:15 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:15 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:16 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:16 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:17 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:17 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:18 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:18 [INFO] Waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:18 [INFO] Done waiting for CRD features.management.cattle.io to become available
2023/08/30 09:47:21 [INFO] Applying CRD navlinks.ui.cattle.io
2023/08/30 09:47:22 [INFO] Applying CRD podsecurityadmissionconfigurationtemplates.management.cattle.io
2023/08/30 09:47:22 [INFO] Applying CRD clusters.management.cattle.io
2023/08/30 09:47:22 [INFO] Applying CRD apiservices.management.cattle.io
2023/08/30 09:47:22 [INFO] Applying CRD clusterregistrationtokens.management.cattle.io
2023/08/30 09:47:22 [INFO] Applying CRD settings.management.cattle.io
2023/08/30 09:47:22 [INFO] Applying CRD preferences.management.cattle.io
2023/08/30 09:47:22 [INFO] Applying CRD features.management.cattle.io
2023/08/30 09:47:23 [INFO] Applying CRD clusterrepos.catalog.cattle.io
2023/08/30 09:47:23 [INFO] Applying CRD operations.catalog.cattle.io
2023/08/30 09:47:23 [INFO] Applying CRD apps.catalog.cattle.io
2023/08/30 09:47:23 [INFO] Applying CRD fleetworkspaces.management.cattle.io
2023/08/30 09:47:23 [INFO] Applying CRD bundles.fleet.cattle.io
2023/08/30 09:47:23 [INFO] Applying CRD clusters.fleet.cattle.io
2023/08/30 09:47:23 [INFO] Applying CRD clustergroups.fleet.cattle.io
2023/08/30 09:47:24 [INFO] Applying CRD managedcharts.management.cattle.io
2023/08/30 09:47:24 [INFO] Applying CRD clusters.provisioning.cattle.io
2023/08/30 09:47:24 [INFO] Applying CRD clusters.provisioning.cattle.io
2023/08/30 09:47:24 [INFO] Applying CRD rkeclusters.rke.cattle.io
2023/08/30 09:47:25 [INFO] Applying CRD rkecontrolplanes.rke.cattle.io
2023/08/30 09:47:25 [INFO] Applying CRD rkebootstraps.rke.cattle.io
2023/08/30 09:47:25 [INFO] Applying CRD rkebootstraptemplates.rke.cattle.io
2023/08/30 09:47:25 [INFO] Applying CRD rkecontrolplanes.rke.cattle.io
2023/08/30 09:47:25 [INFO] Applying CRD custommachines.rke.cattle.io
2023/08/30 09:47:26 [INFO] Applying CRD etcdsnapshots.rke.cattle.io
2023/08/30 09:47:26 [INFO] Applying CRD clusters.cluster.x-k8s.io
2023/08/30 09:47:26 [INFO] Applying CRD machinedeployments.cluster.x-k8s.io
2023/08/30 09:47:26 [INFO] Applying CRD machinehealthchecks.cluster.x-k8s.io
2023/08/30 09:47:26 [INFO] Applying CRD machines.cluster.x-k8s.io
2023/08/30 09:47:27 [INFO] Applying CRD machinesets.cluster.x-k8s.io
2023/08/30 09:47:28 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:29 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:29 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:30 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:30 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:31 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:31 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:32 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:32 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:33 [INFO] Waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:33 [INFO] Done waiting for CRD machinesets.cluster.x-k8s.io to become available
2023/08/30 09:47:33 [INFO] Waiting for CRD clusters.provisioning.cattle.io to become available
2023/08/30 09:47:33 [INFO] Done waiting for CRD clusters.provisioning.cattle.io to become available
2023/08/30 09:47:33 [INFO] Waiting for CRD etcdsnapshots.rke.cattle.io to become available
2023/08/30 09:47:34 [INFO] Done waiting for CRD etcdsnapshots.rke.cattle.io to become available
2023/08/30 09:47:34 [INFO] Creating CRD authconfigs.management.cattle.io
2023/08/30 09:47:34 [INFO] Creating CRD groupmembers.management.cattle.io
2023/08/30 09:47:34 [INFO] Creating CRD groups.management.cattle.io
2023/08/30 09:47:34 [INFO] Creating CRD tokens.management.cattle.io
2023/08/30 09:47:34 [INFO] Creating CRD userattributes.management.cattle.io
2023/08/30 09:47:34 [INFO] Creating CRD users.management.cattle.io
2023/08/30 09:47:35 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:36 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:36 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:37 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:37 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:38 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:38 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:39 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:39 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:40 [INFO] Waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:40 [INFO] Done waiting for CRD users.management.cattle.io to become available
2023/08/30 09:47:40 [INFO] Waiting for CRD authconfigs.management.cattle.io to become available
2023/08/30 09:47:40 [INFO] Done waiting for CRD authconfigs.management.cattle.io to become available
2023/08/30 09:47:40 [INFO] Waiting for CRD groupmembers.management.cattle.io to become available
2023/08/30 09:47:41 [INFO] Done waiting for CRD groupmembers.management.cattle.io to become available
2023/08/30 09:47:41 [INFO] Waiting for CRD groups.management.cattle.io to become available
2023/08/30 09:47:41 [INFO] Done waiting for CRD groups.management.cattle.io to become available
2023/08/30 09:47:41 [INFO] Waiting for CRD tokens.management.cattle.io to become available
2023/08/30 09:47:42 [INFO] Done waiting for CRD tokens.management.cattle.io to become available
2023/08/30 09:47:42 [INFO] Waiting for CRD userattributes.management.cattle.io to become available
2023/08/30 09:47:42 [INFO] Done waiting for CRD userattributes.management.cattle.io to become available
2023/08/30 09:47:46 [INFO] Creating CRD catalogs.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD catalogtemplates.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD catalogtemplateversions.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD apps.project.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD clusteralerts.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD apprevisions.project.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD clusterroletemplatebindings.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD clusteralertgroups.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD dynamicschemas.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD clustercatalogs.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD etcdbackups.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD globalrolebindings.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD clusteralertrules.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD globalroles.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD clustermonitorgraphs.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD kontainerdrivers.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD composeconfigs.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD nodedrivers.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD multiclusterapps.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD nodepools.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD multiclusterapprevisions.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD nodetemplates.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD monitormetrics.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD notifiers.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD nodes.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD podsecuritypolicytemplateprojectbindings.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD projectalerts.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD podsecuritypolicytemplates.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD projectnetworkpolicies.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD projectalertgroups.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD projectcatalogs.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD projectroletemplatebindings.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD projectalertrules.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD projects.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD projectmonitorgraphs.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD rkek8ssystemimages.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD templates.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD rkek8sserviceoptions.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD rkeaddons.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD templateversions.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD roletemplates.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD templatecontents.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD samltokens.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD globaldnses.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD globaldnsproviders.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD clustertemplates.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD clustertemplaterevisions.management.cattle.io
2023/08/30 09:47:46 [INFO] Creating CRD rancherusernotifications.management.cattle.io
2023/08/30 09:47:46 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:46 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:47 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:47 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:47 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:48 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:48 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:48 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:48 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:48 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:48 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:49 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:49 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:49 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:49 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:49 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:49 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:50 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:50 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:50 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:50 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:50 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:50 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:51 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:51 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:51 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:51 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:51 [INFO] Waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:51 [INFO] Done waiting for CRD apps.project.cattle.io to become available
2023/08/30 09:47:51 [INFO] Waiting for CRD apprevisions.project.cattle.io to become available
2023/08/30 09:47:51 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:52 [INFO] Waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:52 [INFO] Done waiting for CRD podsecuritypolicytemplates.management.cattle.io to become available
2023/08/30 09:47:52 [INFO] Waiting for CRD rkek8ssystemimages.management.cattle.io to become available
2023/08/30 09:47:52 [INFO] Done waiting for CRD apprevisions.project.cattle.io to become available
2023/08/30 09:47:52 [INFO] Waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:52 [INFO] Done waiting for CRD templates.management.cattle.io to become available
2023/08/30 09:47:52 [INFO] Waiting for CRD globaldnses.management.cattle.io to become available
2023/08/30 09:47:52 [INFO] Done waiting for CRD rkek8ssystemimages.management.cattle.io to become available
2023/08/30 09:47:52 [INFO] Waiting for CRD clustertemplates.management.cattle.io to become available
2023/08/30 09:47:53 [INFO] Done waiting for CRD globaldnses.management.cattle.io to become available
2023/08/30 09:47:53 [INFO] Waiting for CRD clusteralertgroups.management.cattle.io to become available
2023/08/30 09:47:53 [INFO] Done waiting for CRD clustertemplates.management.cattle.io to become available
2023/08/30 09:47:53 [INFO] Waiting for CRD nodepools.management.cattle.io to become available
2023/08/30 09:47:53 [INFO] Done waiting for CRD clusteralertgroups.management.cattle.io to become available
2023/08/30 09:47:53 [INFO] Waiting for CRD projectmonitorgraphs.management.cattle.io to become available
2023/08/30 09:47:53 [INFO] Done waiting for CRD nodepools.management.cattle.io to become available
2023/08/30 09:47:53 [INFO] Waiting for CRD kontainerdrivers.management.cattle.io to become available
2023/08/30 09:47:54 [INFO] Done waiting for CRD projectmonitorgraphs.management.cattle.io to become available
2023/08/30 09:47:54 [INFO] Waiting for CRD multiclusterapprevisions.management.cattle.io to become available
2023/08/30 09:47:54 [INFO] Done waiting for CRD kontainerdrivers.management.cattle.io to become available
2023/08/30 09:47:54 [INFO] Waiting for CRD rkeaddons.management.cattle.io to become available
2023/08/30 09:47:54 [INFO] Done waiting for CRD multiclusterapprevisions.management.cattle.io to become available
2023/08/30 09:47:54 [INFO] Waiting for CRD monitormetrics.management.cattle.io to become available
2023/08/30 09:47:54 [INFO] Done waiting for CRD rkeaddons.management.cattle.io to become available
2023/08/30 09:47:54 [INFO] Waiting for CRD nodes.management.cattle.io to become available
2023/08/30 09:47:55 [INFO] Done waiting for CRD monitormetrics.management.cattle.io to become available
2023/08/30 09:47:55 [INFO] Waiting for CRD projectcatalogs.management.cattle.io to become available
2023/08/30 09:47:55 [INFO] Done waiting for CRD nodes.management.cattle.io to become available
2023/08/30 09:47:55 [INFO] Waiting for CRD clustertemplaterevisions.management.cattle.io to become available
2023/08/30 09:47:55 [INFO] Done waiting for CRD projectcatalogs.management.cattle.io to become available
2023/08/30 09:47:55 [INFO] Waiting for CRD templatecontents.management.cattle.io to become available
2023/08/30 09:47:55 [INFO] Done waiting for CRD clustertemplaterevisions.management.cattle.io to become available
2023/08/30 09:47:55 [INFO] Waiting for CRD globalrolebindings.management.cattle.io to become available
2023/08/30 09:47:56 [INFO] Done waiting for CRD templatecontents.management.cattle.io to become available
2023/08/30 09:47:56 [INFO] Waiting for CRD clustermonitorgraphs.management.cattle.io to become available
2023/08/30 09:47:56 [INFO] Done waiting for CRD globalrolebindings.management.cattle.io to become available
2023/08/30 09:47:56 [INFO] Waiting for CRD podsecuritypolicytemplateprojectbindings.management.cattle.io to become available
2023/08/30 09:47:56 [INFO] Done waiting for CRD clustermonitorgraphs.management.cattle.io to become available
2023/08/30 09:47:56 [INFO] Waiting for CRD composeconfigs.management.cattle.io to become available
2023/08/30 09:47:56 [INFO] Done waiting for CRD podsecuritypolicytemplateprojectbindings.management.cattle.io to become available
2023/08/30 09:47:56 [INFO] Waiting for CRD etcdbackups.management.cattle.io to become available
2023/08/30 09:47:57 [INFO] Done waiting for CRD composeconfigs.management.cattle.io to become available
2023/08/30 09:47:57 [INFO] Waiting for CRD notifiers.management.cattle.io to become available
2023/08/30 09:47:57 [INFO] Done waiting for CRD etcdbackups.management.cattle.io to become available
2023/08/30 09:47:57 [INFO] Waiting for CRD nodetemplates.management.cattle.io to become available
2023/08/30 09:47:57 [INFO] Done waiting for CRD notifiers.management.cattle.io to become available
2023/08/30 09:47:57 [INFO] Waiting for CRD projectalertgroups.management.cattle.io to become available
2023/08/30 09:47:57 [INFO] Done waiting for CRD nodetemplates.management.cattle.io to become available
2023/08/30 09:47:57 [INFO] Waiting for CRD projectnetworkpolicies.management.cattle.io to become available
2023/08/30 09:47:58 [INFO] Done waiting for CRD projectalertgroups.management.cattle.io to become available
2023/08/30 09:47:58 [INFO] Waiting for CRD globaldnsproviders.management.cattle.io to become available
2023/08/30 09:47:58 [INFO] Done waiting for CRD projectnetworkpolicies.management.cattle.io to become available
2023/08/30 09:47:58 [INFO] Waiting for CRD nodedrivers.management.cattle.io to become available
2023/08/30 09:47:58 [INFO] Done waiting for CRD globaldnsproviders.management.cattle.io to become available
2023/08/30 09:47:58 [INFO] Waiting for CRD clusteralerts.management.cattle.io to become available
2023/08/30 09:47:58 [INFO] Done waiting for CRD nodedrivers.management.cattle.io to become available
2023/08/30 09:47:58 [INFO] Waiting for CRD clusterroletemplatebindings.management.cattle.io to become available
2023/08/30 09:47:59 [INFO] Done waiting for CRD clusteralerts.management.cattle.io to become available
2023/08/30 09:47:59 [INFO] Waiting for CRD multiclusterapps.management.cattle.io to become available
2023/08/30 09:47:59 [INFO] Done waiting for CRD clusterroletemplatebindings.management.cattle.io to become available
2023/08/30 09:47:59 [INFO] Waiting for CRD projectroletemplatebindings.management.cattle.io to become available
2023/08/30 09:47:59 [INFO] Done waiting for CRD multiclusterapps.management.cattle.io to become available
2023/08/30 09:47:59 [INFO] Waiting for CRD catalogtemplateversions.management.cattle.io to become available
2023/08/30 09:47:59 [INFO] Done waiting for CRD projectroletemplatebindings.management.cattle.io to become available
2023/08/30 09:47:59 [INFO] Waiting for CRD projects.management.cattle.io to become available
2023/08/30 09:48:00 [INFO] Done waiting for CRD catalogtemplateversions.management.cattle.io to become available
2023/08/30 09:48:00 [INFO] Waiting for CRD clustercatalogs.management.cattle.io to become available
2023/08/30 09:48:00 [INFO] Done waiting for CRD projects.management.cattle.io to become available
2023/08/30 09:48:00 [INFO] Waiting for CRD dynamicschemas.management.cattle.io to become available
2023/08/30 09:48:00 [INFO] Done waiting for CRD clustercatalogs.management.cattle.io to become available
2023/08/30 09:48:00 [INFO] Waiting for CRD clusteralertrules.management.cattle.io to become available
2023/08/30 09:48:00 [INFO] Done waiting for CRD dynamicschemas.management.cattle.io to become available
2023/08/30 09:48:00 [INFO] Waiting for CRD roletemplates.management.cattle.io to become available
2023/08/30 09:48:01 [INFO] Done waiting for CRD clusteralertrules.management.cattle.io to become available
2023/08/30 09:48:01 [INFO] Waiting for CRD projectalerts.management.cattle.io to become available
2023/08/30 09:48:01 [INFO] Done waiting for CRD roletemplates.management.cattle.io to become available
2023/08/30 09:48:01 [INFO] Waiting for CRD samltokens.management.cattle.io to become available
2023/08/30 09:48:01 [INFO] Done waiting for CRD projectalerts.management.cattle.io to become available
2023/08/30 09:48:01 [INFO] Waiting for CRD projectalertrules.management.cattle.io to become available
2023/08/30 09:48:01 [INFO] Done waiting for CRD samltokens.management.cattle.io to become available
2023/08/30 09:48:01 [INFO] Waiting for CRD globalroles.management.cattle.io to become available
2023/08/30 09:48:02 [INFO] Done waiting for CRD projectalertrules.management.cattle.io to become available
2023/08/30 09:48:02 [INFO] Waiting for CRD templateversions.management.cattle.io to become available
2023/08/30 09:48:02 [INFO] Done waiting for CRD globalroles.management.cattle.io to become available
2023/08/30 09:48:02 [INFO] Waiting for CRD rkek8sserviceoptions.management.cattle.io to become available
2023/08/30 09:48:02 [INFO] Done waiting for CRD templateversions.management.cattle.io to become available
2023/08/30 09:48:02 [INFO] Waiting for CRD catalogs.management.cattle.io to become available
2023/08/30 09:48:02 [INFO] Done waiting for CRD rkek8sserviceoptions.management.cattle.io to become available
2023/08/30 09:48:03 [INFO] Done waiting for CRD catalogs.management.cattle.io to become available
2023/08/30 09:48:03 [INFO] Waiting for CRD catalogtemplates.management.cattle.io to become available
2023/08/30 09:48:03 [INFO] Done waiting for CRD catalogtemplates.management.cattle.io to become available
2023/08/30 09:48:03 [INFO] Waiting for CRD rancherusernotifications.management.cattle.io to become available
2023/08/30 09:48:04 [INFO] Done waiting for CRD rancherusernotifications.management.cattle.io to become available
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Token controller
2023/08/30 09:48:18 [INFO] Starting rke.cattle.io/v1, Kind=RKEBootstrap controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=ProjectCatalog controller
2023/08/30 09:48:18 [INFO] Starting rbac.authorization.k8s.io/v1, Kind=ClusterRoleBinding controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=NodeTemplate controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=User controller
2023/08/30 09:48:18 [INFO] Starting rbac.authorization.k8s.io/v1, Kind=Role controller
2023/08/30 09:48:18 [INFO] Starting /v1, Kind=Namespace controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=GlobalDns controller
2023/08/30 09:48:18 [INFO] Starting catalog.cattle.io/v1, Kind=ClusterRepo controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=ClusterTemplate controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Feature controller
2023/08/30 09:48:18 [INFO] Starting API controllers
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=GlobalRole controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=ProjectRoleTemplateBinding controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=APIService controller
2023/08/30 09:48:18 [INFO] Starting provisioning.cattle.io/v1, Kind=Cluster controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Preference controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Setting controller
2023/08/30 09:48:18 [INFO] Starting rbac.authorization.k8s.io/v1, Kind=ClusterRole controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=CatalogTemplateVersion controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=PodSecurityPolicyTemplateProjectBinding controller
2023/08/30 09:48:18 [INFO] Starting rbac.authorization.k8s.io/v1, Kind=RoleBinding controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=CatalogTemplate controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=RkeAddon controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=GroupMember controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=DynamicSchema controller
2023/08/30 09:48:18 [INFO] Starting /v1, Kind=Secret controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=ClusterTemplateRevision controller
2023/08/30 09:48:18 [INFO] Starting /v1, Kind=ServiceAccount controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=KontainerDriver controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=NodePool controller
I0830 09:48:18.601002      33 leaderelection.go:248] attempting to acquire leader lease kube-system/cattle-controllers...
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Cluster controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Group controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=RkeK8sServiceOption controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=UserAttribute controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=MultiClusterAppRevision controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=MultiClusterApp controller
2023/08/30 09:48:18 [INFO] Starting /v1, Kind=ConfigMap controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=PodSecurityPolicyTemplate controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=ClusterRegistrationToken controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Node controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=RkeK8sSystemImage controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=ClusterRoleTemplateBinding controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=AuthConfig controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=NodeDriver controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=GlobalRoleBinding controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Catalog controller
2023/08/30 09:48:18 [INFO] Starting apiregistration.k8s.io/v1, Kind=APIService controller
2023/08/30 09:48:18 [INFO] Starting project.cattle.io/v3, Kind=App controller
2023/08/30 09:48:18 [INFO] Starting cluster.x-k8s.io/v1beta1, Kind=Machine controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=ClusterCatalog controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=RoleTemplate controller
2023/08/30 09:48:18 [INFO] Starting apiextensions.k8s.io/v1, Kind=CustomResourceDefinition controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Project controller
I0830 09:48:18.663554      33 leaderelection.go:258] successfully acquired lease kube-system/cattle-controllers
2023/08/30 09:48:18 [INFO] Reconciling GlobalRoles
2023/08/30 09:48:18 [INFO] Creating clusters-create
2023/08/30 09:48:18 [INFO] Creating users-manage
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Cluster controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=User controller
2023/08/30 09:48:18 [INFO] Starting /v1, Kind=Secret controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Token controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=UserAttribute controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=GroupMember controller
2023/08/30 09:48:18 [INFO] Starting management.cattle.io/v3, Kind=Group controller
2023/08/30 09:48:18 [INFO] Creating clustertemplaterevisions-create
2023/08/30 09:48:18 [INFO] Waiting for initial data to be populated
2023/08/30 09:48:18 [INFO] Creating admin
2023/08/30 09:48:18 [INFO] Creating view-rancher-metrics
2023/08/30 09:48:18 [INFO] Creating restricted-admin
2023/08/30 09:48:18 [INFO] Creating catalogs-manage
2023/08/30 09:48:18 [INFO] Creating catalogs-use
2023/08/30 09:48:18 [INFO] Creating roles-manage
2023/08/30 09:48:18 [INFO] Creating authn-manage
2023/08/30 09:48:18 [INFO] Creating features-manage
2023/08/30 09:48:18 [INFO] Creating podsecuritypolicytemplates-manage
2023/08/30 09:48:18 [INFO] Creating user-base
2023/08/30 09:48:18 [INFO] Creating settings-manage
2023/08/30 09:48:18 [INFO] Steve auth startup complete
2023/08/30 09:48:18 [INFO] Creating user
2023/08/30 09:48:18 [INFO] Creating nodedrivers-manage
2023/08/30 09:48:18 [INFO] Creating kontainerdrivers-manage
2023/08/30 09:48:19 [INFO] 
2023/08/30 09:48:19 [INFO] -----------------------------------------
2023/08/30 09:48:19 [INFO] Welcome to Rancher
2023/08/30 09:48:19 [INFO] A bootstrap password has been generated for your admin user.
2023/08/30 09:48:19 [INFO] 
2023/08/30 09:48:19 [INFO] Bootstrap Password: zhk2pqgrzbp2l68t8fztsrl5kkrhhcl5vcbpbgdjngznlxkzsckglh
2023/08/30 09:48:19 [INFO] 
2023/08/30 09:48:19 [INFO] Use https://10.233.94.196/dashboard/?setup=zhk2pqgrzbp2l68t8fztsrl5kkrhhcl5vcbpbgdjngznlxkzsckglh to complete setup in the UI
2023/08/30 09:48:19 [INFO] -----------------------------------------
2023/08/30 09:48:19 [INFO] 
2023/08/30 09:48:19 [INFO] Creating clustertemplates-create
2023/08/30 09:48:19 [INFO] Reconciling RoleTemplates
2023/08/30 09:48:19 [INFO] Created default admin user and binding
2023/08/30 09:48:19 [INFO] Creating ingress-view
2023/08/30 09:48:19 [INFO] Creating secrets-view
2023/08/30 09:48:19 [INFO] Creating configmaps-view
2023/08/30 09:48:19 [INFO] Creating serviceaccounts-manage
2023/08/30 09:48:19 [INFO] Creating nodes-view
2023/08/30 09:48:19 [INFO] Creating clustercatalogs-view
2023/08/30 09:48:19 [INFO] Creating project-owner
2023/08/30 09:48:19 [INFO] Creating project-member
2023/08/30 09:48:19 [INFO] Creating serviceaccounts-view
2023/08/30 09:48:19 [INFO] Creating view
2023/08/30 09:48:19 [INFO] Creating projects-view
2023/08/30 09:48:19 [INFO] Creating services-view
2023/08/30 09:48:19 [INFO] Creating edit
2023/08/30 09:48:19 [INFO] Creating persistentvolumeclaims-manage
2023/08/30 09:48:19 [INFO] Creating projectroletemplatebindings-manage
2023/08/30 09:48:19 [INFO] Creating cluster-admin
2023/08/30 09:48:19 [INFO] Creating projects-create
2023/08/30 09:48:19 [INFO] Creating secrets-manage
2023/08/30 09:48:19 [INFO] Creating projectroletemplatebindings-view
2023/08/30 09:48:19 [INFO] Creating projectcatalogs-view
2023/08/30 09:48:19 [INFO] Creating project-monitoring-readonly
2023/08/30 09:48:19 [INFO] Creating navlinks-view
2023/08/30 09:48:19 [INFO] Creating storage-manage
2023/08/30 09:48:19 [INFO] Creating workloads-view
2023/08/30 09:48:19 [INFO] Creating ingress-manage
2023/08/30 09:48:19 [INFO] Creating services-manage
2023/08/30 09:48:19 [INFO] Creating backups-manage
2023/08/30 09:48:19 [INFO] Creating workloads-manage
2023/08/30 09:48:19 [INFO] Registering namespaceHandler for adding labels 
2023/08/30 09:48:19 [INFO] Creating monitoring-ui-view
2023/08/30 09:48:19 [INFO] Creating clustercatalogs-manage
2023/08/30 09:48:19 [INFO] Creating read-only
2023/08/30 09:48:19 [INFO] Creating persistentvolumeclaims-view
2023/08/30 09:48:19 [INFO] Creating cluster-member
2023/08/30 09:48:19 [INFO] Creating nodes-manage
2023/08/30 09:48:19 [INFO] Creating clusterroletemplatebindings-manage
2023/08/30 09:48:19 [INFO] Creating clusterroletemplatebindings-view
2023/08/30 09:48:19 [INFO] Creating configmaps-manage
2023/08/30 09:48:19 [INFO] Creating projectcatalogs-manage
2023/08/30 09:48:19 [INFO] Creating admin
2023/08/30 09:48:19 [INFO] Creating cluster-owner
2023/08/30 09:48:20 [INFO] Creating navlinks-manage
2023/08/30 09:48:20 [INFO] Creating create-ns
2023/08/30 09:48:20 [INFO] Creating CRD prometheuses.monitoring.coreos.com
2023/08/30 09:48:20 [INFO] Creating CRD prometheusrules.monitoring.coreos.com
2023/08/30 09:48:20 [INFO] Creating CRD alertmanagers.monitoring.coreos.com
2023/08/30 09:48:20 [INFO] Creating CRD servicemonitors.monitoring.coreos.com
2023/08/30 09:48:20 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
2023/08/30 09:48:20 [INFO] Waiting for initial data to be populated
E0830 09:48:20.796378      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:20.909301      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:21.002169      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:21.196863      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:21.328311      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:21.419650      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:21.504811      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:21.595269      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:21.681811      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
E0830 09:48:21.741495      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
2023/08/30 09:48:21 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
E0830 09:48:21.800577      33 gvks.go:69] failed to sync schemas: unable to retrieve the complete list of server APIs: monitoring.coreos.com/v1: the server could not find the requested resource
2023/08/30 09:48:22 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
2023/08/30 09:48:22 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
2023/08/30 09:48:22 [INFO] Waiting for initial data to be populated
2023/08/30 09:48:23 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
2023/08/30 09:48:23 [INFO] adding kontainer driver rancherKubernetesEngine
2023/08/30 09:48:23 [INFO] adding kontainer driver googleKubernetesEngine
2023/08/30 09:48:23 [INFO] adding kontainer driver azureKubernetesService
2023/08/30 09:48:23 [INFO] adding kontainer driver amazonElasticContainerService
2023/08/30 09:48:23 [INFO] adding kontainer driver baiducloudcontainerengine
2023/08/30 09:48:23 [INFO] adding kontainer driver aliyunkubernetescontainerservice
2023/08/30 09:48:23 [INFO] adding kontainer driver tencentkubernetesengine
2023/08/30 09:48:23 [INFO] adding kontainer driver huaweicontainercloudengine
2023/08/30 09:48:23 [INFO] adding kontainer driver oraclecontainerengine
2023/08/30 09:48:23 [INFO] adding kontainer driver linodekubernetesengine
2023/08/30 09:48:23 [INFO] adding kontainer driver opentelekomcloudcontainerengine
2023/08/30 09:48:23 [INFO] Created cattle-global-data namespace
2023/08/30 09:48:23 [INFO] Created cattle-global-nt namespace
2023/08/30 09:48:23 [INFO] Creating node driver pinganyunecs
2023/08/30 09:48:23 [INFO] Creating node driver aliyunecs
2023/08/30 09:48:23 [INFO] Creating node driver amazonec2
2023/08/30 09:48:23 [INFO] Creating node driver azure
2023/08/30 09:48:23 [INFO] Creating node driver cloudca
2023/08/30 09:48:23 [INFO] Creating node driver cloudscale
2023/08/30 09:48:23 [INFO] Creating node driver digitalocean
2023/08/30 09:48:23 [INFO] Creating node driver exoscale
2023/08/30 09:48:23 [INFO] Creating node driver google
2023/08/30 09:48:23 [INFO] Creating node driver harvester
2023/08/30 09:48:23 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
2023/08/30 09:48:23 [INFO] Creating node driver linode
2023/08/30 09:48:23 [INFO] Creating node driver oci
2023/08/30 09:48:23 [INFO] Creating node driver openstack
2023/08/30 09:48:23 [INFO] Creating node driver otc
2023/08/30 09:48:23 [INFO] Creating node driver packet
2023/08/30 09:48:23 [INFO] Creating node driver pnap
2023/08/30 09:48:23 [INFO] Creating node driver rackspace
2023/08/30 09:48:23 [INFO] Creating node driver softlayer
2023/08/30 09:48:23 [INFO] Creating node driver nutanix
2023/08/30 09:48:23 [INFO] Creating node driver outscale
2023/08/30 09:48:23 [INFO] Creating node driver vmwarevsphere
2023/08/30 09:48:24 [ERROR] Failed to read API for groups map[monitoring.coreos.com/v1:the server could not find the requested resource]
2023/08/30 09:48:24 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
2023/08/30 09:48:24 [INFO] Starting catalog controller
2023/08/30 09:48:24 [INFO] Starting project-level catalog controller
2023/08/30 09:48:24 [INFO] Starting cluster-level catalog controller
2023/08/30 09:48:24 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
2023/08/30 09:48:24 [INFO] generated self-signed CA certificate CN=dynamiclistener-ca@1693388904,O=dynamiclistener-org: notBefore=2023-08-30 09:48:24.823873249 +0000 UTC notAfter=2033-08-27 09:48:
24.823873249 +0000 UTC
2023/08/30 09:48:24 [INFO] Listening on :443
2023/08/30 09:48:24 [INFO] Listening on :80
2023/08/30 09:48:24 [INFO] certificate CN=dynamic,O=dynamic signed by CN=dynamiclistener-ca@1693388904,O=dynamiclistener-org: notBefore=2023-08-30 09:48:24 +0000 UTC notAfter=2024-08-29 09:48:24 +
0000 UTC
2023/08/30 09:48:24 [WARNING] dynamiclistener [::]:443: no cached certificate available for preload - deferring certificate load until storage initialization or first client request
2023/08/30 09:48:24 [INFO] Creating new TLS secret for cattle-system/serving-cert (count: 4): map[listener.cattle.io/cn-10.233.94.196:10.233.94.196 listener.cattle.io/cn-127.0.0.1:127.0.0.1 listen
er.cattle.io/cn-localhost:localhost listener.cattle.io/cn-rancher.cattle-system:rancher.cattle-system listener.cattle.io/fingerprint:SHA1=CE8A71CC23656FB3D762CB5F858D0D6ED5636EF1]
2023/08/30 09:48:24 [INFO] Active TLS secret cattle-system/serving-cert (ver=384383) (count 4): map[listener.cattle.io/cn-10.233.94.196:10.233.94.196 listener.cattle.io/cn-127.0.0.1:127.0.0.1 list
ener.cattle.io/cn-localhost:localhost listener.cattle.io/cn-rancher.cattle-system:rancher.cattle-system listener.cattle.io/fingerprint:SHA1=CE8A71CC23656FB3D762CB5F858D0D6ED5636EF1]
2023/08/30 09:48:24 [INFO] generated self-signed CA certificate CN=dynamiclistener-ca@1693388904,O=dynamiclistener-org: notBefore=2023-08-30 09:48:24.956776969 +0000 UTC notAfter=2033-08-27 09:48:
24.956776969 +0000 UTC
2023/08/30 09:48:24 [INFO] Listening on :444
2023/08/30 09:48:24 [WARNING] dynamiclistener [::]:444: no cached certificate available for preload - deferring certificate load until storage initialization or first client request
E0830 09:48:25.102765      33 memcache.go:206] couldn't get resource list for monitoring.coreos.com/v1: the server could not find the requested resource
2023/08/30 09:48:25 [INFO] Starting /v1, Kind=Secret controller
2023/08/30 09:48:25 [INFO] Updating TLS secret for cattle-system/serving-cert (count: 4): map[listener.cattle.io/cn-10.233.94.196:10.233.94.196 listener.cattle.io/cn-127.0.0.1:127.0.0.1 listener.c
attle.io/cn-localhost:localhost listener.cattle.io/cn-rancher.cattle-system:rancher.cattle-system listener.cattle.io/fingerprint:SHA1=CE8A71CC23656FB3D762CB5F858D0D6ED5636EF1]
2023/08/30 09:48:25 [INFO] Waiting for CRD prometheuses.monitoring.coreos.com to become available
2023/08/30 09:48:25 [INFO] Refreshing driverMetadata in 1440 minutes
2023/08/30 09:48:25 [INFO] Starting rke.cattle.io/v1, Kind=RKECluster controller
2023/08/30 09:48:25 [INFO] Starting cluster.x-k8s.io/v1beta1, Kind=Cluster controller
2023/08/30 09:48:25 [INFO] Starting management.cattle.io/v3, Kind=GlobalDnsProvider controller
2023/08/30 09:48:25 [INFO] Starting networking.k8s.io/v1, Kind=Ingress controller
2023/08/30 09:48:25 [INFO] Starting /v1, Kind=Node controller
2023/08/30 09:48:25 [INFO] Starting management.cattle.io/v3, Kind=ManagedChart controller
2023/08/30 09:48:25 [INFO] Starting /v1, Kind=ReplicationController controller
2023/08/30 09:48:25 [INFO] Starting admissionregistration.k8s.io/v1, Kind=MutatingWebhookConfiguration controller
2023/08/30 09:48:25 [INFO] Starting fleet.cattle.io/v1alpha1, Kind=Cluster controller
2023/08/30 09:48:25 [INFO] Starting cluster.x-k8s.io/v1beta1, Kind=MachineSet controller
2023/08/30 09:48:25 [INFO] Starting admissionregistration.k8s.io/v1, Kind=ValidatingWebhookConfiguration controller
2023/08/30 09:48:25 [INFO] Starting catalog.cattle.io/v1, Kind=App controller
2023/08/30 09:48:25 [INFO] Starting management.cattle.io/v3, Kind=ComposeConfig controller
2023/08/30 09:48:25 [INFO] Starting management.cattle.io/v3, Kind=Notifier controller
```

##  5. 访问
查看svc

```bash
[root@bastion01 rancher]# kubectl get svc -n rancher
NAME              TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
rancher-service   NodePort   10.233.1.39   <none>        443:30010/TCP,80:30011/TCP   38m
```
界面访问：`https://192.168.10.41:30010/`

![](https://i-blog.csdnimg.cn/blog_migrate/892e32c8e2551289d3b6489a8d982142.png)
获取密码

```bash
[root@bastion01 rancher]# kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{"\n"}}'
zhk2pqgrzbp2l68t8fztsrl5kkrhhcl5vcbpbgdjngznlxkzsckglh
```
输入密码点击登陆后：
![](https://i-blog.csdnimg.cn/blog_migrate/025d969496a1f051a0fabbaa2f7641f3.png)
生成一个新密码：`dPUcGyO5pWjmPqeZ`

或者你可以自定义密码
进入首页的界面状态
![](https://i-blog.csdnimg.cn/blog_migrate/f8b8ae8c02bd4ef33fda417f0b0ee7f8.png)
![](https://i-blog.csdnimg.cn/blog_migrate/78eb0bc2fe7c2e305d6b2cbec8bebb40.png)

如果新创建集群对接云厂商。
![](https://i-blog.csdnimg.cn/blog_migrate/1e11f62957e9269423f7c91073bfd340.png)

如果导入已经存在的集群
![](https://i-blog.csdnimg.cn/blog_migrate/3a4ea4f719d869ae3db367ee415ddeef.png)

## 6. 加入集群

这里选择导入本地环境的另一套k8集群
![](https://i-blog.csdnimg.cn/blog_migrate/f12d8c7596b018cc521f5c430ec5f29b.png)
 设置导入的kubernetes集群的名字，这里我命名为kind1-29，然后直接创建 ![](https://i-blog.csdnimg.cn/blog_migrate/3101cce933c9410e25731657c710c4aa.png)
 执行一个命令：

```bash
 [root@kind2 ~]# kubectl apply -f https://192.168.10.41:30010/v3/import/vnscbt67wp6xlqjzjv9gjpf5v5b4pv65cv7pc8d756p6vpdkzb4tzs_c-m-5zv28n88.yaml
Unable to connect to the server: x509: certificate signed by unknown authority
```
按照提示执行第二个命令

```bash
[root@kind2 ~]# curl --insecure -sfL https://192.168.10.41:30010/v3/import/vnscbt67wp6xlqjzjv9gjpf5v5b4pv65cv7pc8d756p6vpdkzb4tzs_c-m-5zv28n88.yaml | kubectl apply -f -
clusterrole.rbac.authorization.k8s.io/proxy-clusterrole-kubeapiserver created
clusterrolebinding.rbac.authorization.k8s.io/proxy-role-binding-kubernetes-master created
Warning: resource namespaces/cattle-system is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
namespace/cattle-system configured
serviceaccount/cattle created
clusterrolebinding.rbac.authorization.k8s.io/cattle-admin-binding created
secret/cattle-credentials-10bf1d0 created
clusterrole.rbac.authorization.k8s.io/cattle-admin created
Warning: spec.template.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms[0].matchExpressions[0].key: beta.kubernetes.io/os is deprecated since v1.14; use "kubernetes.io/os" instead
deployment.apps/cattle-cluster-agent created
service/cattle-cluster-agent created

```
第三个命令
获取集群 user
```bash
[root@kind2 ~]# cat /root/.kube/config  |grep user:
    user: kind-kind
  user:
[root@kind2 ~]# kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user kind-kind
clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-binding created
```

