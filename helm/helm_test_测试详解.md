


----

## 1. 简介 
`helm chart` 中的测试`templates/`位于该目录下，并且是一个作业定义，它指定具有给定命令运行的容器。容器应该成功退出（退出 0），测试被认为是成功的。作业定义必须包含 `helm test hook` 注解：`helm.sh/hook: test.`

请注意，在 `Helm v3` 之前，作业定义需要包含以下 helm 测试挂钩注释之一：`helm.sh/hook: test-success`或`helm.sh/hook: test-failure`. `helm.sh/hook: test-success`仍然被接受为向后兼容的替代`helm.sh/hook: test`.

示例测试：

 - 验证 `values.yaml` 文件中的配置是否已正确注入。
 - 确保您的用户名和密码正确使用
 - 确保不正确的用户名和密码不起作用
 - 断言您的服务已启动并正确进行负载平衡
 - 等等。

您可以使用命令在 Helm 中运行预定义的测试`helm test <RELEASE_NAME>`。对于图表使用者来说，这是检查他们发布的图表（或应用程序）是否按预期工作的好方法。


## 2. demo
这是[bitnami wordpress](https://artifacthub.io/packages/helm/bitnami/wordpress) 图表中 helm test pod 定义的示例。如果您下载图表的副本，您可以在本地查看文件：

```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm pull bitnami/wordpress --untar
```

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

```bash
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
在版本上运行测试套件的步骤
首先，在集群上安装图表以创建发布。您可能必须等待所有 pod 都激活；如果您在此安装后立即进行测试，则可能会显示传递失败，您将需要重新测试。

```bash
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

 - 您可以在单个 yaml 文件中定义任意数量的测试，也可以分布在`templates/`目录中的多个 yaml 文件中。
 - 您的测试套件嵌套在tests/类似的目录下以`<chart-name>/templates/tests/`实现更多隔离。
 - 一个测试是一个Helm
   钩子，所以注解可以和测试资源一起使用`helm.sh/hook-weight`。`helm.sh/hook-delete-policy`


