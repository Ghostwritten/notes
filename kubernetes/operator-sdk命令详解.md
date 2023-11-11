
##格式

```bash
$ operator-sdk <command> [<subcommand>] [<argument>] [<flags>]
```
## 1. operator-sdk build
命令编译代码并生成可执行文件
```bash
operator-sdk build quay.io/example/operator:v0.0.1
```

## 2. operator-sdk completion
生成bash补全
```bash
operator-sdk completion
```
## 3. operator-sdk print-deps
命令显示操作员所需的最新Golang软件包和版本。默认情况下，它以列格式打印

```bash
operator-sdk print-deps --as-file
```
## 4. operator-sdk generate
命令将调用特定的生成器以根据需要生成代码

## 5. operator-sdk olm-catalog gen-csv

子命令将集群服务版本（CSV）清单和可选的自定义资源定义（CRD）文件写入 `deploy/olm-catalog/<operator_name>/<csv_version>`

```bash
$ operator-sdk olm-catalog gen-csv --csv-version 0.1.0 --update-crds
INFO[0000] Generating CSV manifest version 0.1.0
INFO[0000] Fill in the following required fields in file deploy/olm-catalog/operator-name/0.1.0/operator-name.v0.1.0.clusterserviceversion.yaml:
	spec.keywords
	spec.maintainers
	spec.provider
	spec.labels
INFO[0000] Created deploy/olm-catalog/operator-name/0.1.0/operator-name.v0.1.0.clusterserviceversion.yaml
```

## 6. operator-sdk new

```bash
$ mkdir $GOPATH/src/github.com/example.com/
$ cd $GOPATH/src/github.com/example.com/
$ operator-sdk new app-operator
$ operator-sdk new app-operator \
    --type=ansible \
    --api-version=app.example.com/v1alpha1 \
    --kind=AppService
```

## 7. operator-sdk add
命令将控制器或资源添加到项目中。该命令必须从Operator项目的根目录运行。
### 7.1 Example add api output
```bash
$ operator-sdk add api --api-version app.example.com/v1alpha1 --kind AppService
Create pkg/apis/app/v1alpha1/appservice_types.go
Create pkg/apis/addtoscheme_app_v1alpha1.go
Create pkg/apis/app/v1alpha1/register.go
Create pkg/apis/app/v1alpha1/doc.go
Create deploy/crds/app_v1alpha1_appservice_cr.yaml
Create deploy/crds/app_v1alpha1_appservice_crd.yaml
Running code-generation for Custom Resource (CR) group versions: [app:v1alpha1]
Generating deepcopy funcs

$ tree pkg/apis
pkg/apis/
├── addtoscheme_app_appservice.go
├── apis.go
└── app
	└── v1alpha1
		├── doc.go
		├── register.go
		├── types.go
```
### 7.2 Example add controller output

```bash
$ operator-sdk add controller --api-version app.example.com/v1alpha1 --kind AppService
Create pkg/controller/appservice/appservice_controller.go
Create pkg/controller/add_appservice.go

$ tree pkg/controller
pkg/controller/
├── add_appservice.go
├── appservice
│   └── appservice_controller.go
└── controller.go
```
### 7.3 Example add crd output

```bash
$ operator-sdk add crd --api-version app.example.com/v1alpha1 --kind AppService
Generating Custom Resource Definition (CRD) files
Create deploy/crds/app_v1alpha1_appservice_crd.yaml
Create deploy/crds/app_v1alpha1_appservice_cr.yaml
```
## 8. operator-sdk up local
该local子命令通过构建可以使用kubeconfig文件访问Kubernetes集群的操作员二进制文件在本地计算机上启动操作员 。

```bash
$ operator-sdk up local \
  --kubeconfig "mycluster.kubecfg" \
  --namespace "default" \
  --operator-flags "--flag1 value1 --flag2=value2"
$ operator-sdk up local --operator-flags "--resync-interval 10"
$ operator-sdk up local --namespace "testing"
```
参考链接：
[https://docs.openshift.com/container-platform/4.1/applications/operator_sdk/osdk-cli-reference.html](https://docs.openshift.com/container-platform/4.1/applications/operator_sdk/osdk-cli-reference.html)
