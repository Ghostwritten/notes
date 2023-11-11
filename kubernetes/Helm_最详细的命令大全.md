

##  1. helm repo
###  helm repo list
`Helm` 的 `Repo` 仓库和 `Docker Registry` 比较类似，Chart 库可以用来存储和共享打包 Chart 的位置，我们在安装了 Helm 后，默认的仓库地址是 google 的一个地址，这对于我们不能科学上网的同学就比较苦恼了，没办法访问到官方提供的 Chart 仓库，可以用`helm repo list`来查看当前的仓库配置：

```bash
$ helm repo list
NAME       URL
stable     https://kubernetes-charts.storage.googleapis.com/
local      http://127.0.0.1:8879/charts
```

我们可以看到除了一个默认的 `stable` 的仓库配置外，还有一个 `local` 的本地仓库，这是我们本地测试的一个仓库地址。其实要创建一个 Chart 仓库也是非常简单的，Chart 仓库其实就是一个带有index.yaml索引文件和任意个打包的 Chart 的 HTTP 服务器而已，比如我们想要分享一个 Chart 包的时候，将我们本地的 Chart 包上传到该服务器上面，别人就可以使用了，所以其实我们自己托管一个 Chart 仓库也是非常简单的，比如阿里云的 OSS、Github Pages，甚至自己创建的一个简单服务器都可以。

为了解决科学上网的问题，我这里建了一个 Github Pages 仓库，每天会自动和官方的仓库进行同步，地址是：`https://github.com/cnych/kube-charts-mirror`，这样我们就可以将我们的 Helm 默认仓库地址更改成我们自己的仓库地址了：

```bash
$ helm repo remove stable
"stable" has been removed from your repositories
$ helm repo add stable https://cnych.github.io/kube-charts-mirror/
"stable" has been added to your repositories
$ helm repo list
NAME       URL
stable     https://cnych.github.io/kube-charts-mirror/
local      http://127.0.0.1:8879/charts
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
```
仓库添加完成后，可以使用 update 命令进行仓库更新。当然如果要我们自己来创建一个 web 服务器来服务 `Helm Chart` 的话，只需要实现下面几个功能点就可以提供服务了：

 - 将索引和Chart置于服务器目录中
 - 确保索引文件`index.yaml`可以在没有认证要求的情况下访问
 - 确保 `yaml` 文件的正确内容类型（text/yaml 或 text/x-yaml）


如果你的 web 服务提供了上面几个功能，那么也就可以当做 Helm Chart 仓库来使用了。

##  2. helm search
Helm 将 Charts 包安装到 Kubernetes 集群中，一个安装实例就是一个新的 Release，要找到新的 Chart，我们可以通过搜索命令完成。

> 记住，如果不能科学上网，将默认的 stable 的仓库地址更换成上面我们创建的地址

直接运行`helm search`命令可以查看有哪些 Charts 是可用的：

```bash
$ helm search
NAME                                     CHART VERSION    APP VERSION                     DESCRIPTION
...
stable/minio                             1.6.3            RELEASE.2018-08-25T01-56-38Z    Minio is a high performance distributed object storage se...
stable/mission-control                   0.4.2            3.1.2                           A Helm chart for JFrog Mission Control
stable/mongodb                           4.2.2            4.0.2                           NoSQL document-oriented database that stores JSON-like do...
stable/mongodb-replicaset                3.5.6            3.6                             NoSQL document-oriented database that stores JSON-like do...
...
stable/zetcd                             0.1.9            0.0.3                           CoreOS zetcd Helm chart for Kubernetes
...
```
如果没有使用过滤条件，`helm search` 显示所有可用的 `charts`。可以通过使用过滤条件进行搜索来缩小搜索的结果范围：

```bash
$ helm search mysql
NAME                                CHART VERSION    APP VERSION    DESCRIPTION
...
stable/mysql                        0.10.1           5.7.14         Fast, reliable, scalable, and easy to use open-source rel...
stable/mysqldump                    0.1.0            5.7.21         A Helm chart to help backup MySQL databases using mysqldump
stable/prometheus-mysql-exporter    0.1.0            v0.10.0        A Helm chart for prometheus
stable/mariadb                      4.4.0            10.1.35        Fast, reliable, scalable, and easy to use open-source rel...
...
```

##  3. helm inspect
可以看到明显少了很多 charts 了，同样的，我们可以使用 inspect 命令来查看一个 chart 的详细信息：

```bash
$ helm inspect stable/mysql
appVersion: 5.7.14
description: Fast, reliable, scalable, and easy to use open-source relational database
  system.
engine: gotpl
home: https://www.mysql.com/
icon: https://www.mysql.com/common/logos/logo-mysql-170x115.png
keywords:
- mysql
- database
- sql
maintainers:
- email: o.with@sportradar.com
  name: olemarkus
- email: viglesias@google.com
  name: viglesiasce
name: mysql
sources:
- https://github.com/kubernetes/charts
- https://github.com/docker-library/mysql
version: 0.10.1

---
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.14"
...
```
使用 `inspect` 命令可以查看到该 chart 里面所有描述信息，包括运行方式、配置信息等等。

通过 `helm search` 命令可以找到我们想要的 chart 包，找到后就可以通过 `helm install` 命令来进行安装了。

###  3.1 helm inspect values

要查看 chart 上可配置的选项，使用`helm inspect values`命令即可，比如我们这里查看上面的 mysql 的配置选项：

```bash
$ helm inspect values stable/mysql
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.14"

## Specify password for root user
##
## Default: random 10 character string
# mysqlRootPassword: testing

## Create a database user
##
# mysqlUser:
## Default: random 10 character string
# mysqlPassword:

## Allow unauthenticated access, uncomment to enable
##
# mysqlAllowEmptyPassword: true

## Create a database
##
# mysqlDatabase:

## Specify an imagePullPolicy (Required)
## It's recommended to change this to 'Always' if the image tag is 'latest'
## ref: http://kubernetes.io/docs/user-guide/images/#updating-images
##
imagePullPolicy: IfNotPresent

extraVolumes: |
  # - name: extras
  #   emptyDir: {}

extraVolumeMounts: |
  # - name: extras
  #   mountPath: /usr/share/extras
  #   readOnly: true

extraInitContainers: |
  # - name: do-something
  #   image: busybox
  #   command: ['do', 'something']

# Optionally specify an array of imagePullSecrets.
# Secrets must be manually created in the namespace.
# ref: https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
# imagePullSecrets:
  # - name: myRegistryKeySecretName

## Node selector
## ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector
nodeSelector: {}

livenessProbe:
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3

readinessProbe:
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 1
  successThreshold: 1
  failureThreshold: 3

## Persist data to a persistent volume
persistence:
  enabled: true
  ## database data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  accessMode: ReadWriteOnce
  size: 8Gi
  annotations: {}

## Configure resource requests and limits
## ref: http://kubernetes.io/docs/user-guide/compute-resources/
##
resources:
  requests:
    memory: 256Mi
    cpu: 100m

# Custom mysql configuration files used to override default mysql settings
configurationFiles: {}
#  mysql.cnf: |-
#    [mysqld]
#    skip-name-resolve
#    ssl-ca=/ssl/ca.pem
#    ssl-cert=/ssl/server-cert.pem
#    ssl-key=/ssl/server-key.pem

# Custom mysql init SQL files used to initialize the database
initializationFiles: {}
#  first-db.sql: |-
#    CREATE DATABASE IF NOT EXISTS first DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
#  second-db.sql: |-
#    CREATE DATABASE IF NOT EXISTS second DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

metrics:
  enabled: false
  image: prom/mysqld-exporter
  imageTag: v0.10.0
  imagePullPolicy: IfNotPresent
  resources: {}
  annotations: {}
    # prometheus.io/scrape: "true"
    # prometheus.io/port: "9104"
  livenessProbe:
    initialDelaySeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    initialDelaySeconds: 5
    timeoutSeconds: 1

## Configure the service
## ref: http://kubernetes.io/docs/user-guide/services/
service:
  ## Specify a service type
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types
  type: ClusterIP
  port: 3306
  # nodePort: 32000

ssl:
  enabled: false
  secret: mysql-ssl-certs
  certificates:
#  - name: mysql-ssl-certs
#    ca: |-
#      -----BEGIN CERTIFICATE-----
#      ...
#      -----END CERTIFICATE-----
#    cert: |-
#      -----BEGIN CERTIFICATE-----
#      ...
#      -----END CERTIFICATE-----
#    key: |-
#      -----BEGIN RSA PRIVATE KEY-----
#      ...
#      -----END RSA PRIVATE KEY-----

## Populates the 'TZ' system timezone environment variable
## ref: https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html
##
## Default: nil (mysql will use image's default timezone, normally UTC)
## Example: 'Australia/Sydney'
# timezone:

# To be added to the database server pod(s)
podAnnotations: {}
```
然后，我们可以直接在 YAML 格式的文件中来覆盖上面的任何配置，在安装的时候直接使用该配置文件即可：(config.yaml)

```bash
mysqlUser: haimaxyUser
mysqlDatabase: haimaxyDB
service:
  type: NodePort
```
我们这里通过 `config.yaml` 文件定义了 `mysqlUser` 和 `mysqlDatabase`，并且把 `service` 的类型更改为了 `NodePort`


## 4. helm template

```bash
helm template stable/mysql
```


##  5. helm install
要安装新的软件包，直接使用 `helm install` 命令即可。最简单的情况下，它只需要一个 chart 的名称参数：

```bash
$ helm install stable/mysql
NAME:   mewing-squid
LAST DEPLOYED: Tue Sep  4 23:31:23 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/PersistentVolumeClaim
NAME                STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
mewing-squid-mysql  Pending  1s

==> v1/Service
NAME                TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
mewing-squid-mysql  ClusterIP  10.108.197.48  <none>       3306/TCP  1s

==> v1beta1/Deployment
NAME                DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
mewing-squid-mysql  1        0        0           0          1s

==> v1/Pod(related)
NAME                                 READY  STATUS   RESTARTS  AGE
mewing-squid-mysql-69f587bdf9-z7glv  0/1    Pending  0         0s

==> v1/Secret
NAME                TYPE    DATA  AGE
mewing-squid-mysql  Opaque  2     1s

==> v1/ConfigMap
NAME                     DATA  AGE
mewing-squid-mysql-test  1     1s


NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mewing-squid-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mewing-squid-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h mewing-squid-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/mewing-squid-mysql 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```
现在 `mysql chart` 已经安装上了，安装 chart 会创建一个新 `release` 对象。上面的 `release` 被命名为 `hmewing-squid`。如果你想使用你自己的 `release` 名称，只需使用`--name`参数指定即可，比如：

```bash
$ helm install stable/mysql --name mydb
```
在安装过程中，helm 客户端将打印有关创建哪些资源的有用信息，`release` 的状态以及其他有用的配置信息，比如这里的有访问 mysql 服务的方法、获取 root 用户的密码以及连接 mysql 的方法等信息。

> 值得注意的是 Helm 并不会一直等到所有资源都运行才退出。因为很多 charts
> 需要的镜像资源非常大，所以可能需要很长时间才能安装到集群中去。


### 5.1 helm install -f
上面的安装方式是使用 `chart` 的默认配置选项。但是在很多时候，我们都需要自定义 chart 以满足自身的需求，要自定义 chart，我们就需要知道我们使用的 chart 支持的可配置选项才行。

然后现在我们来安装的时候通过`-f` 直接指定该 yaml 文件：

```bash
$ helm install -f config.yaml stable/mysql --name mydb
NAME:   mydb
LAST DEPLOYED: Wed Sep  5 00:09:44 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME        TYPE    DATA  AGE
mydb-mysql  Opaque  2     1s

==> v1/ConfigMap
NAME             DATA  AGE
mydb-mysql-test  1     1s

==> v1/PersistentVolumeClaim
NAME        STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
mydb-mysql  Pending  1s

==> v1/Service
NAME        TYPE      CLUSTER-IP     EXTERNAL-IP  PORT(S)         AGE
mydb-mysql  NodePort  10.96.150.198  <none>       3306:32604/TCP  0s

==> v1beta1/Deployment
NAME        DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
mydb-mysql  1        1        1           0          0s

==> v1/Pod(related)
NAME                        READY  STATUS   RESTARTS  AGE
mydb-mysql-dfc999888-hbw5d  0/1    Pending  0         0s
...
```
我们可以看到当前 release 的名字已经变成 mydb 了。然后可以查看下 mydb 关联的 Service 是否变成 NodePort 类型的了：

```bash
$ kubectl get svc
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP          110d
mewing-squid-mysql   ClusterIP   10.108.197.48    <none>        3306/TCP         46m
mydb-mysql           NodePort    10.96.150.198    <none>        3306:32604/TCP   8m
```
看到服务 `mydb-mysql` 变成了 `NodePort` 类型的，二之前默认创建的 `mewing-squid-mysql` 是 `ClusterIP` 类型的，证明上面我们通过 YAML 文件来覆盖 values 是成功的。

接下来我们查看下 Pod 的状况：

```bash
$ kubectl get pods
NAME                                      READY     STATUS    RESTARTS   AGE
mewing-squid-mysql-69f587bdf9-z7glv       0/1       Pending   0          49m
mydb-mysql-dfc999888-hbw5d                0/1       Pending   0          11m
```
比较奇怪的是之前默认创建的和现在的 `mydb` 的 `release` 创建的 Pod 都是 `Pending` 状态，直接使用 `describe` 命令查看下：

```bash
$ kubectl describe pod mydb-mysql-dfc999888-hbw5d
Name:           mydb-mysql-dfc999888-hbw5d
Namespace:      default
Node:           <none>
Labels:         app=mydb-mysql
                pod-template-hash=897555444
...
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  2m (x37 over 12m)  default-scheduler  pod has unbound PersistentVolumeClaims (repeated 2 times)
```
我们可以发现两个 Pod 处于 `Pending` 状态的原因都是 PVC 没有被绑定上，所以这里我们可以通过 storageclass 或者手动创建一个合适的 PV 对象来解决这个问题。

另外为了说明 helm 更新的用法，我们这里来直接禁用掉数据持久化，可以在上面的 `config.yaml` 文件中设置：

```bash
persistence:
  enabled: false
```


###  5.2 helm install  --set
另外一种方法就是在安装过程中使用`--set`来覆盖对应的 value 值，比如禁用数据持久化，我们这里可以这样来覆盖：

```bash
$ helm install stable/mysql --set persistence.enabled=false --name mydb
```

###  5.3 helm install --dry-run --debug
我们可以通过 `helm install --dry-run --debug` 命令进行调试

```bash
helm install --dry-run --debug
```



##  6. helm status

要跟踪 `release` 状态或重新读取配置信息，可以使用 `helm status` 查看：

```bash
$  helm status mewing-squid
LAST DEPLOYED: Tue Sep  4 23:31:23 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
...
```

可以看到当前 `release` 的状态是`DEPLOYED`，下面还有一些安装的时候出现的信息。





##   7. helm ls

现在我们使用 `helm ls` 命令查看先当前的 `release`：

```bash
$ helm ls
NAME            REVISION    UPDATED                     STATUS      CHART           APP VERSION    NAMESPACE
mewing-squid    1           Tue Sep  4 23:31:23 2018    DEPLOYED    mysql-0.10.1    5.7.14         default
mydb            2           Wed Sep  5 00:38:33 2018    DEPLOYED    mysql-0.10.1    5.7.14  
```
可以看到 mydb 这个 `release` 的REVISION已经变成2了，这是因为 release 的版本是递增的，每次安装、升级或者回滚，版本号都会加1，第一个版本号始终为1.

##  8. helm history

同样我们可以使用 `helm history` 命令查看 `release` 的历史版本：

```bash
$ helm history mydb
REVISION    UPDATED                     STATUS        CHART           DESCRIPTION
1           Wed Sep  5 00:09:44 2018    SUPERSEDED    mysql-0.10.1    Install complete
2           Wed Sep  5 00:38:33 2018    DEPLOYED      mysql-0.10.1    Upgrade complete
```

##  9. helm rollback

当然如果我们要回滚到某一个版本的话，使用 helm rollback 命令即可，比如我们将 mydb 回滚到上一个版本：

```bash
$ helm rollback mydb 1
```

## 10. helm delete
上节课我们就学习了要删除一个 release 直接使用 `helm delete` 命令就 OK：

```bash
$ helm delete mewing-squid
release "mewing-squid" deleted
```
这将从集群中删除该 `release`，但是这并不代表就完全删除了.

##  11. helm list 

```bash
helm list
```

###  11.1 helm list --deleted
我们还可以通过`--deleted`参数来显示被删除掉 release:

```bash
$ helm list --deleted
NAME            REVISION    UPDATED                     STATUS     CHART           APP VERSION    NAMESPACE
mewing-squid    1           Tue Sep  4 23:31:23 2018    DELETED    mysql-0.10.1    5.7.14         default
$  helm list --all
NAME            REVISION    UPDATED                     STATUS      CHART           APP VERSION    NAMESPACE
mewing-squid    1           Tue Sep  4 23:31:23 2018    DELETED     mysql-0.10.1    5.7.14         default
mydb            2           Wed Sep  5 00:38:33 2018    DEPLOYED    mysql-0.10.1    5.7.14   
```
`helm list --all`则会显示所有的 `release`，包括已经被删除的
由于 Helm 保留已删除 release 的记录，因此不能重新使用 release 名称。（如果 确实 需要重新使用此 release 名称，则可以使用此 `--replace` 参数，但它只会重用现有 release 并替换其资源。）这点是不是和 `docker container` 的管理比较类似
请注意，因为 release 以这种方式保存，所以可以回滚已删除的资源并重新激活它。

如果要彻底删除 release，则需要加上`--purge`参数：

```bash
$ helm delete mewing-squid --purge
release "mewing-squid" deleted
```
##  12. helm create
[helm 创建 charts 应用示例](https://ghostwritten.blog.csdn.net/article/details/123662067)

```bash
$ helm create deis-workflow
Creating deis-workflow
```
现在有一个charts `./deis-workflow`。您可以编辑它并创建自己的模板。


##  13. helm lint
在我们去创建/维护，或者使用 [Helm chart](https://blog.csdn.net/xixihahalelehehe/article/details/123662067?spm=1001.2014.3001.5501) 进行应用部署的时候，有时候可能会遇到一些错误。

Helm chart 是通过 YAML 进行维护的，而 YAML 是缩进/语法敏感的。假如你的缩进或者语法有问题，都将会导致报错。最简单的检查办法是使用 helm lint 进行检查。

比如我们进行如下修改：

```yaml
$ diff --git a/values.yaml b/values.yaml
index 4a8b237..696a77d 100644
--- a/values.yaml
+++ b/values.yaml
@@ -5,7 +5,7 @@
 replicaCount: 1
 
 image:
-  repository: nginx
+ repository: nginx
   pullPolicy: IfNotPresent
   # Overrides the image tag whose default is the chart appVersion.
   tag: ""
```
将 `image.repository` 的缩进搞错，这时进行安装将看到如下报错：

```yaml
$ helm install foo .
Error: INSTALLATION FAILED: cannot load values.yaml: error converting YAML to JSON: yaml: line 9: mapping values are not allowed in this context
```
`helm lint`检查如下：

```yaml
$ helm lint .
==> Linting .
[INFO] Chart.yaml: icon is recommended
[ERROR] values.yaml: unable to parse YAML: error converting YAML to JSON: yaml: line 9: mapping values are not allowed in this context
[ERROR] templates/: cannot load values.yaml: error converting YAML to JSON: yaml: line 9: mapping values are not allowed in this context
[ERROR] : unable to load chart
        cannot load values.yaml: error converting YAML to JSON: yaml: line 9: mapping values are not allowed in this context

Error: 1 chart(s) linted, 1 chart(s) failed
```
我们将该内容恢复原样，并进行如下变更：

```yaml
$ diff --git a/values.yaml b/values.yaml
index 4a8b237..c86c0be 100644
--- a/values.yaml
+++ b/values.yaml
@@ -2,7 +2,7 @@
 # This is a YAML-formatted file.
 # Declare variables to be passed into your templates.
 
-replicaCount: 1
+replicaCount: "this should not be string"
 
 image:
   repository: nginx
```
将 `replicaCount` 修改成了一段字符串。这时候我们使用 `helm lint` 是无法检查出来的。

```yaml
$ helm lint .
==> Linting .
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```
因为从 `YAML` 语法上是无法检查出来类型的，并且这也是我们具体的业务逻辑（Kubernetes）限制的。

这时，如果进行安装将得到如下错误：

```yaml
$ helm install foo .
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: error validating "": error validating data: ValidationError(Deployment.spec.replicas): invalid type for io.k8s.api.apps.v1.DeploymentSpec.replicas: got "string", expected "integer"
```
以看到，对于这种类型错误是可以很直接的得到反馈的。

但还有一种情况，就是语法规则，类型均正常，但是不符合业务的实际预期。

比如我们进行如下变更：

```yaml
$ diff --git a/values.yaml b/values.yaml
index 4a8b237..8feedd6 100644
--- a/values.yaml                                                                                    
+++ b/values.yaml                                                                                    
@@ -8,7 +8,7 @@ image:
   repository: nginx
   pullPolicy: IfNotPresent
   # Overrides the image tag whose default is the chart appVersion.
-  tag: ""
+  tag: "1.20"
  
 imagePullSecrets: []
 nameOverride: ""
```
这时，可以安装成功，但可能我们预期想要安装的镜像是 1.20-alpine 。这种场景下，上述两种方式就都没有效果了。


##  14. helm upgrade
我们这里将数据持久化禁用掉来对上面的 mydb 进行升级：

```bash
$ echo config.yaml
mysqlUser: haimaxyUser
mysqlDatabase: haimaxyDB
service:
  type: NodePort
persistence:
  enabled: false
$ helm upgrade -f config.yaml mydb stable/mysql
helm upgrade -f config.yaml mydb stable/mysql
Release "mydb" has been upgraded. Happy Helming!
LAST DEPLOYED: Wed Sep  5 00:38:33 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
...
```
可以看到已经变成 `DEPLOYED` 状态了，现在我们再去看看 Pod 的状态呢：

```bash
$ kubectl get pods
NAME                                      READY     STATUS            RESTARTS   AGE
mewing-squid-mysql-69f587bdf9-z7glv       0/1       Pending           0          1h
mydb-mysql-6ffc84bbf6-lcn4d               0/1       PodInitializing   0          49s
...
```
我们看到 mydb 关联的 Pod 已经变成了 `PodInitializing` 的状态，已经不是 `Pending` 状态了，同样的，使用 `describe` 命令查看：

```bash
$ kubectl describe pod mydb-mysql-6ffc84bbf6-lcn4d
Name:           mydb-mysql-6ffc84bbf6-lcn4d
Namespace:      default
Node:           node02/10.151.30.63
Start Time:     Wed, 05 Sep 2018 00:38:33 +0800
Labels:         app=mydb-mysql
                pod-template-hash=2997406692
Annotations:    <none>
Status:         Pending
...
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  SuccessfulMountVolume  58s   kubelet, node02    MountVolume.SetUp succeeded for volume "data"
  Normal  SuccessfulMountVolume  58s   kubelet, node02    MountVolume.SetUp succeeded for volume "default-token-n9w2d"
  Normal  Scheduled              57s   default-scheduler  Successfully assigned mydb-mysql-6ffc84bbf6-lcn4d to node02
  Normal  Pulling                57s   kubelet, node02    pulling image "busybox:1.25.0"
  Normal  Pulled                 45s   kubelet, node02    Successfully pulled image "busybox:1.25.0"
  Normal  Created                44s   kubelet, node02    Created container
  Normal  Started                44s   kubelet, node02    Started container
  Normal  Pulling                41s   kubelet, node02    pulling image "mysql:5.7.14"
```
我们可以看到现在没有任何关于 PVC 的错误信息了，这是因为我们刚刚更新的版本中就是禁用掉了的数据持久化的，证明 `helm upgrade` 和 `--values` 是生效了的。

###  14.1 helm upgrade --install
既能安装
```bash
$ helm upgrade --install <release name> --values <values file> <chart directory>
```




##  15. helm package
当需要打包charts以进行分发时，您可以运行以下 `helm package`命令：

```bash
$ helm package deis-workflow
deis-workflow-0.1.0.tgz
```

现在可以通过`helm install`以下方式轻松安装该charts：

```bash
$ helm install ./deis-workflow-0.1.0.tgz

```

## 16. helm fetch	

```bash
helm fetch  jetstack/cert-manager
helm fetch ${repo_name}/${charts_name} --version ${version} -d ${Outputs_Dir}/${release}
```

##  17. helm test
`helm chart` 中的`test`位于该目录 `templates/`，并且是一个 `job`定义，它指定具有给定命令运行的容器。容器应该成功退出（退出 0），测试被认为是成功的。作业定义必须包含 `helm test hook` 注解：`helm.sh/hook: test`.

> 请注意，在 `Helm v3` 之前，作业定义需要包含以下 helm 测试挂钩注释之一：`helm.sh/hook: test-success`或`helm.sh/hook: test-failure`. `helm.sh/hook:test-success`仍然被接受为向后兼容的替代`helm.sh/hook: test`.

示例测试：

 - 验证 `values.yaml` 文件中的配置是否已正确注入。
 - 确保您的用户名和密码正确使用
 - 确保不正确的用户名和密码不起作用
 - 断言您的服务已启动并正确进行负载平衡
 - .................


这是 `bitnami wordpress` 图表中 `helm test pod` 定义的示例。如果您下载图表的副本，您可以在本地查看文件：







```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm pull bitnami/wordpress --untar
```
结构：

```bash
wordpress/
  Chart.yaml
  README.md
  values.yaml
  charts/
  templates/
  templates/tests/test-mariadb-connection.yaml
```

在`wordpress/templates/tests/test-mariadb-connection.yaml`中，您会看到一个可以尝试的测试：

```yaml
{{- if .Values.mariadb.enabled }}
apiVersion: v1
kind: Pod
metadata:
  name: "{{ .Release.Name }}-credentials-test"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: {{ .Release.Name }}-credentials-test
      image: {{ template "wordpress.image" . }}
      imagePullPolicy: {{ .Values.image.pullPolicy | quote }}
      {{- if .Values.securityContext.enabled }}
      securityContext:
        runAsUser: {{ .Values.securityContext.runAsUser }}
      {{- end }}
      env:
        - name: MARIADB_HOST
          value: {{ template "mariadb.fullname" . }}
        - name: MARIADB_PORT
          value: "3306"
        - name: WORDPRESS_DATABASE_NAME
          value: {{ default "" .Values.mariadb.db.name | quote }}
        - name: WORDPRESS_DATABASE_USER
          value: {{ default "" .Values.mariadb.db.user | quote }}
        - name: WORDPRESS_DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ template "mariadb.fullname" . }}
              key: mariadb-password
      command:
        - /bin/bash
        - -ec
        - |
                    mysql --host=$MARIADB_HOST --port=$MARIADB_PORT --user=$WORDPRESS_DATABASE_USER --password=$WORDPRESS_DATABASE_PASSWORD
  restartPolicy: Never
{{- end }}
```

首先，在集群上安装charts以创建发布。您可能必须等待所有 pod 都激活；如果您在此安装后立即进行测试，则可能会显示传递失败，您将需要重新测试。

```yaml
$ helm install quirky-walrus wordpress --namespace default
$ helm test quirky-walrus
Pod quirky-walrus-credentials-test pending
Pod quirky-walrus-credentials-test pending
Pod quirky-walrus-credentials-test pending
Pod quirky-walrus-credentials-test succeeded
Pod quirky-walrus-mariadb-test-dqas5 pending
Pod quirky-walrus-mariadb-test-dqas5 pending
Pod quirky-walrus-mariadb-test-dqas5 pending
Pod quirky-walrus-mariadb-test-dqas5 pending
Pod quirky-walrus-mariadb-test-dqas5 succeeded
NAME: quirky-walrus
LAST DEPLOYED: Mon Jun 22 17:24:31 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE:     quirky-walrus-mariadb-test-dqas5
Last Started:   Mon Jun 22 17:27:19 2020
Last Completed: Mon Jun 22 17:27:21 2020
Phase:          Succeeded
TEST SUITE:     quirky-walrus-credentials-test
Last Started:   Mon Jun 22 17:27:17 2020
Last Completed: Mon Jun 22 17:27:19 2020
Phase:          Succeeded
[...]
```
编写helm charts test要点：

 - 您可以在单个 yaml 文件中定义任意数量的测试，也可以分布在templates/目录中的多个 yaml 文件中。
 - 测试套件嵌套可以在`tests/`类似的目录 下以`<chart-name>/templates/tests/`实现更多隔离。
 - 一个测试是一个 `Helm  hook`，所以注解可以和测试资源一起使用`helm.sh/hook-weight`、`helm.sh/hook-delete-policy`


## 18. helm plugin
`[Helm plugin](https://helm.sh/zh/docs/topics/plugins/)`是与Helm无缝集成的附加工具。插件提供一种扩展Helm核心特性集的方法，但不需要每个新的特性都用Go编写并加入核心工具中。[helm plugin更多用法细节参阅这里](https://helm.sh/docs/topics/plugins/)。

Helm插件有以下特性：

 - 可以在不影响Helm核心工具的情况下添加和移除。
 - 可以用任意编程语言编写。
 - 与Helm集成，并展示在helm help和其他地方。

### 18.1 helm plugin install
格式：

```bash
$ helm plugin install <path|url>
```
demo：

```bash
$ helm plugin install https://github.com/adamreese/helm-env
```
如果是插件tar包，仅需解压插件到`$HELM_PLUGINS`目录。也可以用tar包的url直接安装：

```bash
 helm plugin install https://domain/path/to/plugin.tar.gz
```

###  18.2 helm plugin list

```bash
$ helm plugin list
NAME	VERSION	DESCRIPTION                    
env 	0.1.0  	Print out the helm environment.
```

### 18.3 helm plugin  uninstall

```bash
$ helm plugin uninstall env
Uninstalled plugin: env
```





✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)

-----------



