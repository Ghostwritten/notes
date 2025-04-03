![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc82c903ec46e9eb92ba68e3a62fa81e.png)




## 介绍
[Nacos](https://nacos.io/zh-cn/docs/what-is-nacos.html) /nɑ:kəʊs/ 是 Dynamic Naming and Configuration Service的首字母简称，一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

## 准备条件
- Kubernetes 1.10+
- Helm v3
- PV provisioner support in the underlying infrastructure
  - [部署 openebs-lvmlocalpv](https://github.com/upmio/infini-scale-install/tree/main/addons/openebs-lvmlocalpv)
  - [Kubernetes 使用 helm 部署 NFS Provisioner](https://blog.csdn.net/xixihahalelehehe/article/details/131756156)
- mysql
  - [kubernetes deploy standalone mysql demo](https://blog.csdn.net/xixihahalelehehe/article/details/132530842) 

创建数据库、用户名、导入表结构

```bash
$  mysql -h 192.168.23.21 -u root  -P 30006  -p'password'
mysql> create database nacos_config character set utf8;
mysql> create user 'nacos'@'%' identified by 'nacos';
mysql> grant all privileges ON nacos_config.* TO 'nacos'@'%';
mysql>use nacos_config;	

$ wget https://raw.githubusercontent.com/alibaba/nacos/develop/distribution/conf/mysql-schema.sql
$  mysql -h 192.168.23.21 -u root  -P 30006  -p'password' -D nacos_config < mysql-schema.sql
$  mysql -h 192.168.23.21 -u root  -P 30006  -p'password' -D nacos_config
mysql> show tables;
+------------------------+
| Tables_in_nacos_config |
+------------------------+
| config_info            |
| config_info_aggr       |
| config_info_beta       |
| config_info_tag        |
| config_tags_relation   |
| group_capacity         |
| his_config_info        |
| permissions            |
| roles                  |
| tenant_capacity        |
| tenant_info            |
| users                  |
+------------------------+
12 rows in set (0.00 sec)

```

## 定制 values.yaml 

```bash
git clone https://github.com/nacos-group/nacos-k8s.git
cd nacos-k8s/helm
vim values.yaml
```

```bash
# Default values for nacos.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

global:
  mode: standalone
#  mode: cluster

############################nacos###########################
namespace: default
nacos:
  image:
    repository: nacos/nacos-server
    tag: latest
    pullPolicy: IfNotPresent
  plugin:
    enable: true
    image:
      repository: nacos/nacos-peer-finder-plugin
      tag: 1.1
      pullPolicy: IfNotPresent
  replicaCount: 1
  podManagementPolicy: Parallel
  domainName: cluster.local
  preferhostmode: hostname
  serverPort: 8848
  health:
    enabled: false
  storage:
    type: embedded
#    type: mysql
#    db:
#      host: localhost
#      name: nacos
#      port: 3306
#      username: usernmae
#      password: password
#      param: characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false

persistence:
  enabled: false
  data:
    accessModes:
      - ReadWriteOnce
    storageClassName: manual
    resources:
      requests:
        storage: 5Gi


service:
  #type: ClusterIP
  type: NodePort
  port: 8848
  nodePort: 30000


ingress:
  enabled: false
  # apiVersion: extensions/v1beta1
  apiVersion: networking.k8s.io/v1
  annotations: { }
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
    # For Kubernetes >= 1.18 you should specify the ingress-controller via the field ingressClassName
    # See https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/#specifying-the-class-of-an-ingress
    # ingressClassName: nginx
  ingressClassName: "nginx"
  hosts:
    - host: nacos.example.com
      #paths: [ ]

  tls: [ ]
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources:
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  requests:
    cpu: 500m
    memory: 2Gi
annotations: { }

nodeSelector: { }

tolerations: [ ]

affinity: { }

```

部署

```bash
helm  install  nacos  ./
```
输出：

```bash
NAME: nacos
LAST DEPLOYED: Mon Sep  4 20:10:59 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services  nacos-cs)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT/nacos
2. MODE:
   standalone: you need to modify replicaCount in the values.yaml, .Values.replicaCount=1
   cluster: kubectl scale sts default-nacos --replicas=3

```

## 检查

```bash
$ kubectl get all
NAME          READY   STATUS    RESTARTS   AGE
pod/nacos-0   1/1     Running   0          8m36s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                       AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP                                                       48d
service/nacos-cs     NodePort    10.99.163.228   <none>        8848:32575/TCP,9848:30113/TCP,9849:30338/TCP,7848:30000/TCP   8m36s

NAME                     READY   AGE
statefulset.apps/nacos   1/1     8m37s
```
## 登陆
执行：

```bash
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services  nacos-cs)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT/nacos
```

访问：`https://192.168.23.14`: 
用户：nacos
密码：nacos
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/843622bfac91fe02834623b05e735b07.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/965ddf1bb3a99dc5c5cce578be19851f.png)

## 默认配置

```bash
$ kubectl exec -ti nacos-0 -- sh
sh-4.2# ls
LICENSE  NOTICE  bin  conf  data  derby.log  logs  plugins  start.out  target  work
sh-4.2# cat conf/
1.4.0-ipv6_support-update.sql  announcement.conf              application.properties         derby-schema.sql               mysql-schema.sql               nacos-logback.xml
sh-4.2# cat conf/application.properties 
# spring
server.servlet.contextPath=${SERVER_SERVLET_CONTEXTPATH:/nacos}
server.contextPath=/nacos
server.port=${NACOS_APPLICATION_PORT:8848}
server.tomcat.accesslog.max-days=30
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i %{Request-Source}i
server.tomcat.accesslog.enabled=${TOMCAT_ACCESSLOG_ENABLED:false}
server.error.include-message=ALWAYS
# default current work dir
server.tomcat.basedir=file:.
#*************** Config Module Related Configurations ***************#
### Deprecated configuration property, it is recommended to use `spring.sql.init.platform` replaced.
#spring.datasource.platform=${SPRING_DATASOURCE_PLATFORM:}
spring.sql.init.platform=${SPRING_DATASOURCE_PLATFORM:}
nacos.cmdb.dumpTaskInterval=3600
nacos.cmdb.eventTaskInterval=10
nacos.cmdb.labelTaskInterval=300
nacos.cmdb.loadDataAtStart=false
db.num=${MYSQL_DATABASE_NUM:1}
db.url.0=jdbc:mysql://${MYSQL_SERVICE_HOST}:${MYSQL_SERVICE_PORT:3306}/${MYSQL_SERVICE_DB_NAME}?${MYSQL_SERVICE_DB_PARAM:characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false}
db.user.0=${MYSQL_SERVICE_USER}
db.password.0=${MYSQL_SERVICE_PASSWORD}
### The auth system to use, currently only 'nacos' and 'ldap' is supported:
nacos.core.auth.system.type=${NACOS_AUTH_SYSTEM_TYPE:nacos}
### worked when nacos.core.auth.system.type=nacos
### The token expiration in seconds:
nacos.core.auth.plugin.nacos.token.expire.seconds=${NACOS_AUTH_TOKEN_EXPIRE_SECONDS:18000}
### The default token:
nacos.core.auth.plugin.nacos.token.secret.key=${NACOS_AUTH_TOKEN:}
### Turn on/off caching of auth information. By turning on this switch, the update of auth information would have a 15 seconds delay.
nacos.core.auth.caching.enabled=${NACOS_AUTH_CACHE_ENABLE:false}
nacos.core.auth.enable.userAgentAuthWhite=${NACOS_AUTH_USER_AGENT_AUTH_WHITE_ENABLE:false}
nacos.core.auth.server.identity.key=${NACOS_AUTH_IDENTITY_KEY:}
nacos.core.auth.server.identity.value=${NACOS_AUTH_IDENTITY_VALUE:}
## spring security config
### turn off security
nacos.security.ignore.urls=${NACOS_SECURITY_IGNORE_URLS:/,/error,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-fe/public/**,/v1/auth/**,/v1/console/health/**,/actuator/**,/v1/console/server/**}
# metrics for elastic search
management.metrics.export.elastic.enabled=false
management.metrics.export.influx.enabled=false
nacos.naming.distro.taskDispatchThreadCount=10
nacos.naming.distro.taskDispatchPeriod=200
nacos.naming.distro.batchSyncKeyCount=1000
nacos.naming.distro.initDataRatio=0.9
nacos.naming.distro.syncRetryDelay=5000
nacos.naming.data.warmup=true


```

