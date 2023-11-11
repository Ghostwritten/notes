
![在这里插入图片描述](https://img-blog.csdnimg.cn/76e3fe6a4e494c1a930c5328bf16a0a7.png)





## 1. 简介
[PostgreSQL](https://www.postgresql.org/)是一个可靠且健壮的[关系数据库系统](https://phoenixnap.com/kb/what-is-a-relational-database)，具有符合 [ACID 的事务](https://phoenixnap.com/kb/acid-vs-base)。它旨在处理各种规模的工作负载，非常适合个人使用和大规模部署，例如[数据仓库](https://phoenixnap.com/kb/data-warehouse-architecture-explained)、[大数据服务器](https://phoenixnap.com/kb/big-data-server)或 Web 服务。

这篇文章我尝试在 kubernetes 集群部署 Postgresql。

## 2. 条件

- 安装了kubectl的 Kubernetes 集群，我选择 [kind 部署 kubernetes](https://blog.csdn.net/xixihahalelehehe/article/details/121968488)
- 已[安装 Helm 3](https://blog.csdn.net/xixihahalelehehe/article/details/122866004)


## 3. helm 部署 posgresql
[Helm](https://helm.sh/) 为您提供了一种在 kubernetes 集群上部署 PostgreSQL 实例的快速简便的方法。

### 3.1 添加 Helm 存储库
在 [Artifact Hub](https://artifacthub.io/) 中搜索要使用的 PostgreSQL [Helm 图表](https://ghostwritten.blog.csdn.net/article/details/123662067)。

```bash
$ helm search hub postgresql
URL                                                     CHART VERSION   APP VERSION     DESCRIPTION

https://artifacthub.io/packages/helm/cetic/post...      0.2.3           11.5.0          PostgreSQL is an open-source object-relational ...
https://artifacthub.io/packages/helm/wiremind/p...      8.7.0           11.7.0          DEPRECATED Chart for PostgreSQL, an object-rela...
https://artifacthub.io/packages/helm/request-re...      10.1.1          11.10.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/design-cat...      10.1.1          11.10.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/taalhuizen...      10.1.1          11.10.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/riftbit/po...      10.11.0         11.13.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/kubesphere...      1.0.3           11.11.0         Chart for PostgreSQL with HA architecture.
https://artifacthub.io/packages/helm/cloudve/po...      4.0.0           10.7.0          Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/reportport...      10.9.4          11.13.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/notificati...      10.1.1          11.10.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/authorizat...      10.1.1          11.10.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/jfrog/post...      10.3.18         11.11.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/radar-base...      11.1.24         14.2.0          PostgreSQL (Postgres) is an open source object-...
https://artifacthub.io/packages/helm/bitnami/po...      12.1.7          15.1.0          PostgreSQL (Postgres) is an open source object-...
https://artifacthub.io/packages/helm/forms-cata...      10.1.1          11.10.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/bitnami-ak...      12.1.2          15.1.0          PostgreSQL (Postgres) is an open source object-...
https://artifacthub.io/packages/helm/cloudnativ...      5.0.0           11.3.0          Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/wallet-com...      10.1.1          11.10.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/choerodon/...      3.18.4          10.7.0          Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/graphql-hi...      12.1.2          15.1.0          PostgreSQL (Postgres) is an open source object-...
https://artifacthub.io/packages/helm/duyet/post...      9.3.3           11.9.0          Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/goauthenti...      10.16.2         11.14.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/kubegemsap...      10.5.1          11.12.0         Chart for PostgreSQL, an object-relational data...
https://artifacthub.io/packages/helm/camptocamp...      0.7.1                           Object-relational database management system (O...
https://artifacthub.io/packages/helm/openstack-...      0.1.8           v9.6            OpenStack-Helm PostgreSQL

https://artifacthub.io/packages/helm/inseefrlab...      3.4.0           1               An object-relational database management system...
https://artifacthub.io/packages/helm/truecharts...      11.0.18         14.6.0          PostgresSQL
```


将 charts 的存储库添加到本地 Helm 安装：

添加 `Bitnami Helm chart`，添加存储库后，更新本地存储库

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
查看
```bash
$ helm repo list
NAME    URL
bitnami https://charts.bitnami.com/bitnami
```

### 3.2 默认安装

```bash
$ helm install psql-test1 bitnami/postgresql
NAME: psql-test1
LAST DEPLOYED: Tue Jan 10 00:18:49 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 12.1.7
APP VERSION: 15.1.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    psql-test1-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default psql-test1-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run psql-test1-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:15.1.0-debian-11-r19 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host psql-test1-postgresql -U postgres -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/psql-test1-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
```

查看状态

```bash
$ kubectl get all
NAME                          READY   STATUS    RESTARTS   AGE
pod/psql-test1-postgresql-0   1/1     Running   0          82s

NAME                               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes                 ClusterIP   10.96.0.1      <none>        443/TCP    15d
service/psql-test1-postgresql      ClusterIP   10.96.80.174   <none>        5432/TCP   82s
service/psql-test1-postgresql-hl   ClusterIP   None           <none>        5432/TCP   82s

NAME                                     READY   AGE
statefulset.apps/psql-test1-postgresql   1/1     82s
```
已经部署好 postgresql，下一步是连接到数据库。

```bash
$ export POSTGRES_PASSWORD=$(kubectl get secret --namespace default psql-test1-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

$ kubectl run psql-test1-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:15.1.0-debian-11-r19 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
>       --command -- psql --host psql-test1-postgresql -U postgres -d postgres -p 5432
If you don't see a command prompt, try pressing enter.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "psql-test1-postgresql" (address "10.96.80.174") at port "5432".
```
成功连接。

### 3.3 选参安装
根据所选的helm chart，可用的配置选项可能会有所不同。`bitnami PostgreSQL chart` 提供了大量选项，从简单的用户创建到复杂的安全配置，如设置证书和策略。所有可用的选项都列在 [helm chart GitHub](https://github.com/bitnami/charts/tree/main/bitnami/postgresql) 页面上。

让我们看看一些常见的选项，例如更改用户名、密码、数据库名称和端口。这可以通过在安装 helm chart 时提供以下参数来完成。

```bash
$ helm install psql-test2 bitnami/postgresql --set global.postgresql.auth.username=testadmin --set global.postgresql.auth.password=testadmin123 --set global.postgresql.auth.database=testdb --set global.postgresql.service.ports.postgresql=5555
helm install psql-test2 bitnami/postgresql --set global.postgresql.auth.username=testadmin --set global.
postgresql.auth.password=testadmin123 --set global.postgresql.auth.database=testdb --set global.postgresql.service.ports.postgresql=5555
NAME: psql-test2
LAST DEPLOYED: Tue Jan 10 00:36:29 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 12.1.7
APP VERSION: 15.1.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5555 on the following DNS names from within your cluster:

    psql-test2-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_ADMIN_PASSWORD=$(kubectl get secret --namespace default psql-test2-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To get the password for "testadmin" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default psql-test2-postgresql -o jsonpath="{.data.password}" | base64 -d)

To connect to your database run the following command:

    kubectl run psql-test2-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:15.1.0-debian-11-r19 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host psql-test2-postgresql -U testadmin -d testdb -p 5555

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/psql-test2-postgresql 5555:5555 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U testadmin -d testdb -p 5555
```

导出“testadmin”用户的密码，它将返回用户定义的密码。

```bash
$ export POSTGRES_PASSWORD=$(kubectl get secret --namespace default psql-test2-postgresql -o jsonpath="{.data.password}" | base64 -d)
$ echo $POSTGRES_PASSWORD
testadmin123
```

### 3.4 持久存储安装
#### 3.4.1 创建 PersistentVolume
Postgres 数据库中的数据需要在 pod 重新启动后保持不变。

```bash
cat <<EOF> postgres-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgresql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
EOF
```

```bash
kubectl apply -f postgres-pv.yaml
```
#### 3.4.2 创建 PersistentVolumeClaim

```bash
cat <<EOF> postgres-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgresql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
EOF
```

```bash
 kubectl apply -f postgres-pvc.yaml
```
使用kubectl get检查PVC是否连接到PV成功：

```bash
$ kubectl get pvc
NAME                  STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgresql-pv-claim   Bound    postgresql-pv   10Gi       RWO            manual         46s
```
#### 3.4.3 安装 Helm Chart

执行：

```bash
$ helm install psql-test bitnami/postgresql --set persistence.existingClaim=postgresql-pv-claim --set volumePermissions.enabled=true
NAME: psql-test
LAST DEPLOYED: Mon Jan  9 18:21:18 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 12.1.7
APP VERSION: 15.1.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    psql-test-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default psql-test-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run psql-test-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:15.1.0-debian-11-r19 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host psql-test-postgresql -U postgres -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/psql-test-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
```

查看状态

```bash
$ k get pods -w
NAME                     READY   STATUS     RESTARTS   AGE
psql-test-postgresql-0   0/1     Init:0/1   0          7s
psql-test-postgresql-0   0/1     Init:0/1   0          27s
psql-test-postgresql-0   0/1     PodInitializing   0          28s
psql-test-postgresql-0   0/1     Running           0          55s
psql-test-postgresql-0   1/1     Running           0          66s
```

#### 3.4.5 连接到 PostgreSQL 客户端
导出POSTGRES_PASSWORD环境变量以便能够登录到 PostgreSQL 实例：

```bash
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default psql-test-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
```
打开另一个终端窗口并键入以下端口转发命令以转发 Postgres 端口：

```bash
$ kubectl port-forward --namespace default svc/psql-test-postgresql 5432:5432
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432
```
最小化端口转发窗口并返回上一个。键入连接到 PostgreSQL 客户端 psql 的命令：

```bash
PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
psql (9.6.22, server 15.1)
WARNING: psql major version 9.6, server major version 15.
         Some psql features might not work.
Type "help" for help.

postgres=#
```
如果没有安装psql，执行：`sudo apt install postgresql-client-12`

### 3.5 自定义配置 value.yaml 安装
创建 `PersistentVolume & PersistentVolumeClaim`，步骤同上。

```bash
$ vim value.yaml
# define default database user, name, and password for PostgreSQL deployment
auth:
  enablePostgresUser: true
  postgresPassword: "StrongPassword"
  username: "app1"
  password: "AppPassword"
  database: "app_db"

# The postgres helm chart deployment will be using PVC postgresql-data-claim
primary:
  persistence:
    enabled: true
    existingClaim: "postgresql-pv-claim"
```
配置完成后，执行：

```bash
$ helm install postgresql-dev -f values.yaml bitnami/postgresql
NAME: postgresql-dev
LAST DEPLOYED: Tue Jan 10 09:07:26 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 12.1.7
APP VERSION: 15.1.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    postgresql-dev.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_ADMIN_PASSWORD=$(kubectl get secret --namespace default postgresql-dev -o jsonpath="{.data.postgres-password}" | base64 -d)

To get the password for "app1" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresql-dev -o jsonpath="{.data.password}" | base64 -d)

To connect to your database run the following command:

    kubectl run postgresql-dev-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:15.1.0-debian-11-r19 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgresql-dev -U app1 -d app_db -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/postgresql-dev 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U app1 -d app_db -p 5432
```


## 4. 手动部署 postgresql

### 4.1 创建 configmap
提供数据库名称、用户名和登录 PostgreSQL 实例的密码。
```bash
$ vim  postgres-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  labels:
    app: postgres
data:
  POSTGRES_DB: postgresdb
  POSTGRES_USER: admin
  POSTGRES_PASSWORD: test123
```
执行：

```bash
kubectl apply -f postgres-configmap.yaml
```

### 4.2 创建 PersistentVolume & PersistentVolumeClaim

```bash
$ vim postgres-storage.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: postgres-pv-volume
  labels:
    type: local
    app: postgres
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/data"
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: postgres-pv-claim
  labels:
    app: postgres
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```
执行：

```bash
kubectl apply -f postgres-storage.yaml
```
```bash
$ kubectl get pvc
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pv-claim             Bound    postgres-pv-volume                         5Gi        RWX            manual         32s
```
### 4.3 创建 PostgreSQL Deployment

```bash
$ vim postgres-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:10.1
          imagePullPolicy: "IfNotPresent"
          ports:
            - containerPort: 5432
          envFrom:
            - configMapRef:
                name: postgres-config
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgredb
      volumes:
        - name: postgredb
          persistentVolumeClaim:
            claimName: postgres-pv-claim
```
执行：

```bash
kubectl apply -f postgres-deployment.yaml
```

### 4.4 创建 PostgreSQL Service

```bash
$ vim postgres-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  labels:
    app: postgres
spec:
  type: NodePort
  ports:
   - port: 5432
  selector:
   app: postgres
```
执行：

```bash
kubectl apply -f postgres-service.yaml
```
### 4.5 访问
访问部署清单

```bash
$ kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/postgres-5bb9d69b96-4lxr4   1/1     Running   0          15m

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          15d
service/postgres     NodePort    10.96.61.244   <none>        5432:30321/TCP   12m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/postgres   1/1     1            1           15m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/postgres-5bb9d69b96   1         1         1       15m
```
您可以看到类型为 `NodePort` 的 postgres 服务在 Kubernetes 主机上为 PostgreSQL 客户端连接公开端口 `30321`,您的端口可能不同，因为 NodePort 是为您的服务随机选择的端口。NodePort 服务会在 `30000-32767` 之间随机为你的服务选择端口。

访问日志

```bash
$ kubectl logs postgres-5bb9d69b96-4lxr4
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.utf8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/data ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok

syncing data to disk ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.
Success. You can now start the database server using:

    pg_ctl -D /var/lib/postgresql/data -l logfile start

waiting for server to start....2023-01-09 15:25:13.980 UTC [45] LOG:  listening on IPv6 address "::1", port 5432
2023-01-09 15:25:13.980 UTC [45] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2023-01-09 15:25:13.982 UTC [45] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-01-09 15:25:14.011 UTC [46] LOG:  database system was shut down at 2023-01-09 15:25:13 UTC
2023-01-09 15:25:14.021 UTC [45] LOG:  database system is ready to accept connections
 done
server started
CREATE DATABASE

CREATE ROLE


/usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*

2023-01-09 15:25:14.786 UTC [45] LOG:  received fast shutdown request
waiting for server to shut down....2023-01-09 15:25:14.787 UTC [45] LOG:  aborting any active transactions
2023-01-09 15:25:14.792 UTC [45] LOG:  worker process: logical replication launcher (PID 52) exited with exit code 1
2023-01-09 15:25:14.793 UTC [47] LOG:  shutting down
2023-01-09 15:25:14.811 UTC [45] LOG:  database system is shut down
 done
server stopped

PostgreSQL init process complete; ready for start up.

2023-01-09 15:25:14.913 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2023-01-09 15:25:14.913 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2023-01-09 15:25:14.915 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-01-09 15:25:14.948 UTC [72] LOG:  database system was shut down at 2023-01-09 15:25:14 UTC
2023-01-09 15:25:14.959 UTC [1] LOG:  database system is ready to accept connections
```

### 4.6 kubectl 连接 PostgreSQL
获取有关您与 PostgreSQL shell 的连接的信息
```bash
$ kubectl exec -ti postgres-5bb9d69b96-4lxr4 -- psql -h localhost -U admin --password -p 5432 postgresdb
Password for user admin:
psql (10.1)
Type "help" for help.

postgresdb=# \conninfo
You are connected to database "postgresdb" as user "admin" on host "localhost" at port "5432".
```
### 4.7 客户端 psql 连接 PostgreSQL
运行以下psql命令,将 IP 地址更改为`10.89.0.3`，一般为Kubernetes 主机 IP 地址，但kind部署的 kubernetes 集群分配给该集群另一个地址，，`30321`将端口更改为 NodePort 资源的端口。
```bash
$ kubectl get no -owide
NAME                 STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION                CONTAINER-RUNTIME
kind-control-plane   Ready    control-plane   15d   v1.25.3   10.89.0.3     <none>        Ubuntu 22.04.1 LTS   4.18.0-348.7.1.el8_5.x86_64   containerd://1.6.9

$ psql -h  10.89.0.3 -U admin --password -p 30321 postgresdb
Password for user admin:
psql (9.6.22, server 10.1)
WARNING: psql major version 9.6, server major version 10.
         Some psql features might not work.
Type "help" for help.

postgresdb=#  \conninfo
You are connected to database "postgresdb" as user "admin" on host "10.89.0.3" at port "30321".
```


参考：
- [Deploy and Manage PostgreSQL on Kubernetes](https://arctype.com/blog/deploy-postgres-kubernetes/)
- [How to Deploy Postgres to Kubernetes](https://adamtheautomator.com/postgres-to-kubernetes/)
