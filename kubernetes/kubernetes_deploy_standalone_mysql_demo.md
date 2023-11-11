
![](https://img-blog.csdnimg.cn/42002312e7a64b17bc5ed9ab0ddd8c2f.png)


## deployment mysql standalone

kubernetes 集群内部署 单节点 mysql

```bash
ansible all -m shell -a "mkdir -p /mnt/mysql/data"
```

cat mysql-pv-pvc.yaml

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/mysql/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

cat mysql-deploy.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
  - port: 3306
    nodePort: 30006
    targetPort: 3306
    protocol: TCP
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```


> 注意：如果使用 image: mysql:8.0 以上 需要 添加`MYSQL_SERVICE_DB_PARAM`修改`characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=Asia/Shanghai`

start:

```bash
kubectl apply -f mysql-pv-pvc.yaml
kubectl apply -f mysql-deploy.yaml
```
check

```bash
[root@kube-master01 mysql]# kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
mysql-794c6d56c6-xnfhb   1/1     Running   0          10h
[root@kube-master01 mysql]# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP          23d
mysql        NodePort    10.43.17.172   <none>        3306:30006/TCP   10h
mysql        NodePort    10.43.17.172   <none>        3306:30006/TCP   10h
[root@kube-master01 mysql]# mysql -h 192.168.23.31 -P 30006 -u root -p'password'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 5.7.43 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| clusterpedia       |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.05 sec)

mysql>
```

## statefulset mysql standalone
在kube-node01 创建目录/mnt/mysql/data

```bash
mkdir -p /mnt/mysql/data
```
创建 pv & pvc

```bash
$ vim mysql-pv-pvc.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: mysql.standalone.node
          operator: In
          values:
          - enable
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/mysql/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  namespace: mysql
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

```
创建 statefulset

```bash
$ vim mysql-deploy.yaml 
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: mysql
spec:
  type: NodePort
  ports:
  - port: 3306
    nodePort: 30006
    targetPort: 3306
    protocol: TCP
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: mysql.standalone.node
                operator: In 
                values:
                - enable
      containers:
      - image: mysql:5.7
        name: mysql
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim

```

执行：

```bash
kubectl apply -f mysql-pv-pvc.yaml 
kubectl apply -f mysql-deploy.yaml
```

检查

```bash
$ kubectl get all -n mysql
NAME          READY   STATUS    RESTARTS   AGE
pod/mysql-0   1/1     Running   0          5m16s

NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/mysql   NodePort   10.102.59.69   <none>        3306:30006/TCP   4m47s

NAME                     READY   AGE
statefulset.apps/mysql   1/1     5m16s
```
连接 mysql

```bash
[root@kube-master01 mysql-yaml]# mysql -h 192.168.23.21 -u root  -P 30006  -p'password'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.36 MySQL Community Server (GPL)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)


```

##  helm install mysql standalone
- [https://github.com/bitnami/charts/tree/main/bitnami/mysql](https://github.com/bitnami/charts/tree/main/bitnami/mysql)
- [https://github.com/upmio/infini-scale-install/tree/main/addons/mysql-standalone](https://github.com/upmio/infini-scale-install/tree/main/addons/mysql-standalone)


### 1. 设置必要的环境变量
- DB_USER：登录 MySQL 用户名。

- DB_PWD：登录 MySQL 用户密码。

- DB_STORAGECLASS_NAME：指定Storageclass名称, 使用 kubectl get storageclasses 获取可用的 Storageclass 名称。

 - DB_PVC_SIZE_G：指定持久化卷的大小，单位为Gi。

- DB_NODE_NAMES：指定安装MySQL pod的节点名称，节点名称可以使用","作为分隔符，表示多个节点名称，安装程序会对节点进行label固定安装节点。

```bash
export DB_USER="admin"
export DB_PWD='password'
export DB_STORAGECLASS_NAME="topolvm-provisioner"
export DB_PVC_SIZE_G="50"
export DB_NODE_NAMES="db-node01,db-node02,db-node03"
```

### 2. 运行安装脚本
注意⚠️：如果找不到 Helm3，将自动安装。

注意⚠️：安装脚本会对指定节点进行添加label的操作。

运行安装脚本

```bash
curl -sSL https://raw.githubusercontent.com/upmio/infini-scale-install/main/addons/mysql/install_el7.sh | sh -
```

等几分钟。 如果所有 mysql pod 都在运行，则 mysql 将成功安装。

```bash
$ kubectl get all  -n mysql 
NAME          READY   STATUS    RESTARTS   AGE
pod/mysql-0   1/1     Running   0          113m

NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/mysql   ClusterIP   None         <none>        3306/TCP   113m

NAME                     READY   AGE
statefulset.apps/mysql   1/1     113m

```


参考：

- [Run a Single-Instance Stateful Application](https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/)


