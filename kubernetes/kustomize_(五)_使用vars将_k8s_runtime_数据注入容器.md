![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d6b34263e07950d69008681266ca1da4.png#pic_center)

----

本教程将会介绍如何声明变量以及如何在容器中的命令使用变量。要注意的是，变量的查找和替换并不适用于任意字段，默认仅适用于容器的`env`，`args`和`command`。

运行WordPress，以下是必须的：

 - WordPress 连接 MySQL 数据库
 - MySQL 服务可以被 WordPress 容器访问

## 1 构建工作空间：

```bash
#!/bin/bash
export DEMO_HOME=$(mktemp -d)
export MYSQL_HOME=$DEMO_HOME/mysql
mkdir -p $MYSQL_HOME
export WORDPRESS_HOME=$DEMO_HOME/wordpress
mkdir -p $WORDPRESS_HOME
echo $MYSQL_HOME $WORDPRESS_HOME
```
执行：

```bash
$ bash   vars1.sh 
/tmp/tmp.OXhQnYJmhA/mysql /tmp/tmp.OXhQnYJmhA/wordpress
$ export DEMO_HOME=/tmp/tmp.OXhQnYJmhA
$ export MYSQL_HOME=/tmp/tmp.OXhQnYJmhA/mysql
$ export WORDPRESS_HOME=/tmp/tmp.OXhQnYJmhA/wordpress
```
## 2. 下载 resources
下载 WordPress 的 resources 和 kustomization.yaml 。

```bash
CONTENT="https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/examples/wordpress/wordpress"

curl -s -o "$WORDPRESS_HOME/#1.yaml"  "$CONTENT/{deployment,service,kustomization}.yaml"
```
若下载不了出现的问题:
1.[使用 Git 同步时出现gnutls_handshake() failed: Error in the pull function](https://ghostwritten.blog.csdn.net/article/details/111031250)
2.[raw.githubusercontent.com:443连接的OpenSSL SSL_ERROR_SYSCALL](https://ghostwritten.blog.csdn.net/article/details/111031475)
下载 MySQL 的 resources 和 kustomization.yaml 。

```bash
CONTENT="https://raw.githubusercontent.com/kubernetes-sigs/kustomize\/master/examples/wordpress/mysql"

curl -s -o "$MYSQL_HOME/#1.yaml"  "$CONTENT/{deployment,service,secret,kustomization}.yaml"
```
## 3. 创建 kustomization.yaml
基于 wordpress 和 mysql 的两个 bases 创建一个新的 `kustomization.yaml` ：

```bash
cat <<EOF >$DEMO_HOME/kustomization.yaml
resources:
 - wordpress
 - mysql
namePrefix: demo-
patchesStrategicMerge:
 - patch.yaml
EOF
```
## 4. 下载 WordPress 的 patchs
在新的 kustomization 中应用 WordPress Deployment 的 patch ，该 patch 包含：
 - 添加初始容器来显示mysql的服务名称
 - 添加允许 wordpress 查找到 mysql 数据库的环境变量

该 patch 内容如下：

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  template:
    spec:
      initContainers:
      - name: init-command
        image: debian
        command:
        - "echo $(WORDPRESS_SERVICE)"
        - "echo $(MYSQL_SERVICE)"
      containers:
      - name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: $(MYSQL_SERVICE)
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
```
初始化容器的命令需要依赖于k8s资源对象字段的信息，由占位符变量 `$(WORDPRESS_SERVICE)` 和 `$(MYSQL_SERVICE)` 表示。
将变量绑定到k8s对象字段

```bash
cat <<EOF >>$DEMO_HOME/kustomization.yaml
vars:
  - name: WORDPRESS_SERVICE
    objref:
      kind: Service
      name: wordpress
      apiVersion: v1
    fieldref:
      fieldpath: metadata.name
  - name: MYSQL_SERVICE
    objref:
      kind: Service
      name: mysql
      apiVersion: v1
EOF
```

`WORDPRESS_SERVICE` 来自 wordpress 服务的 `metadata.name` 字段。如果不指定 `fieldref` ，则使用默认的 `metadata.name` 。因此 `MYSQL_SERVICE` 来自 mysql 服务的 metadata.name 字段。

替换
运行命令查看替换结果：

```bash
kustomize build $DEMO_HOME
```

预期的输出为：

```bash
(truncated)
...
     initContainers:
     - command:
       - echo demo-wordpress
       - echo demo-mysql
       image: debian
       name: init-command
```
扩展阅读：

 - [kustomize (一) 管理yaml部署入门hello world](https://ghostwritten.blog.csdn.net/article/details/107925618)
 - [kustomize (二) ConfigMap的生成和滚动更新](https://ghostwritten.blog.csdn.net/article/details/110962982)
 - [kustomize (三) devops和开发配合管理配置数据behavior: merge、namePrefix、nameSuffix](https://ghostwritten.blog.csdn.net/article/details/110980010)
 - [kustomize (四) generatorOptions详解](https://ghostwritten.blog.csdn.net/article/details/110992002)
 - [kustomize (五) 使用vars将 k8s runtime数据注入容器](https://ghostwritten.blog.csdn.net/article/details/111029759)
 - [kustomize(六)命令行常用编排](https://ghostwritten.blog.csdn.net/article/details/111042577)
 - [kustomize (七)patches、patchesJson6902、patchesStrategicMerge详解](https://ghostwritten.blog.csdn.net/article/details/111188370)
 - [kustomize (八)生成secret](https://ghostwritten.blog.csdn.net/article/details/111211735)
 - [kustomize(九)使用终章](https://blog.csdn.net/xixihahalelehehe/article/details/111223923)
