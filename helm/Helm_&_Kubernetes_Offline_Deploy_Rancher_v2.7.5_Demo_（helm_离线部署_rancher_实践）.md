
![](https://img-blog.csdnimg.cn/5ddd4c7532e44ba5a5c3c181366d31de.png)



## 1. 简介
[Rancher](https://ranchermanager.docs.rancher.com/) 是一个开源容器管理平台，专为在生产环境中部署容器的组织构建。Rancher可以轻松地在任何地方运行Kubernetes，满足IT需求，并为DevOps团队提供支持。


最新版本

Latest Release
- v2.7
  - Latest - v2.7.6 - rancher/rancher:v2.7.6 / rancher/rancher:latest - Read the full release [notes](https://github.com/rancher/rancher/releases/tag/v2.7.6).
  - Stable - v2.7.6 - rancher/rancher:v2.7.6 / rancher/rancher:stable - Read the full release [notes](https://github.com/rancher/rancher/releases/tag/v2.7.6).
- v2.6
  - Latest - v2.6.13 - rancher/rancher:v2.6.13 - Read the full release [notes](https://github.com/rancher/rancher/releases/tag/v2.6.13).
  - Stable - v2.6.13 - rancher/rancher:v2.6.13 - Read the full release [notes](https://github.com/rancher/rancher/releases/tag/v2.6.13).
- v2.5
  - Latest - v2.5.17 - rancher/rancher:v2.5.17 - Read the full release [notes](https://github.com/rancher/rancher/releases/tag/v2.5.17).
  - Stable - v2.5.17 - rancher/rancher:v2.5.17 - Read the full release [notes](https://github.com/rancher/rancher/releases/tag/v2.5.17).

## 2. 预备条件
- [Kubernetes Cluster](https://ghostwritten.blog.csdn.net/article/details/131749277)
- [Ingress Controller](https://blog.csdn.net/xixihahalelehehe/article/details/112831123)
- [helm](https://blog.csdn.net/xixihahalelehehe/article/details/128847114)


## 3. 选择 SSL 配置
Rancher Server 默认设计为安全的，并且需要 SSL/TLS 配置。

如果你在离线的 Kubernetes 集群中安装 Rancher，我们建议使用以下两种证书生成方式。
| 配置               | Chart 选项                   | 描述                                                      | 是否需要 cert-manager |
|------------------|----------------------------|---------------------------------------------------------|-------------------|
| Rancher 生成的自签名证书 | ingress.tls.source=rancher | 使用Rancher生成的CA签发的自签名证书。此项是默认选项。在渲染Helm模板的时候不需要传递此项。     | 是                 |
| 你已有的证书           | ingress.tls.source=secret  | 通过创建Kubernetes密文使用你自己的证书文件。在渲染 Rancher Helm 模板时必须传递此选项。 | 否                 |


## 4. 离线安装的 Helm Chart 选项
在配置 Rancher Helm 模板时，Helm Chart 中有几个专为离线安装设计的选项，如下表：
| Chart 选项              | Chart 值                        | 描述                                                                                                                                                                                             |
|-----------------------|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| certmanager.version   | \<version>                      | 根据运行的 cert-manager 版本配置适当的 Rancher TLS 颁发者。                                                                                                                                                    |
| systemDefaultRegistry | <REGISTRY.YOURDOMAIN.COM:PORT> | 将 Rancher Server 配置成在配置集群时，始终从私有镜像仓库中拉取镜像。                                                                                                                                                     |
| useBundledSystemChart | true                           | 配置 Rancher Server 使用打包的 Helm System Chart 副本。system charts 仓库包含所有 Monitoring，Logging，告警和全局 DNS 等功能所需的应用商店项目。这些 Helm Chart 位于 GitHub 中。但是由于你处在离线环境，因此使用 Rancher 内置的 Chart 会比设置 Git mirror 容易得多。 |


## 5. 下载介质
安装 helm

```bash
wget https://get.helm.sh/helm-v3.11.0-linux-amd64.tar.gz
tar zxvf helm-v3.11.0-linux-amd64.tar.gz 
cp linux-amd64/helm /usr/local/bin/
```
查看版本

```bash
$ helm version
version.BuildInfo{Version:"v3.11.0", GitCommit:"472c5736ab01133de504a826bd9ee12cbe4e7904", GitTreeState:"clean", GoVersion:"go1.18.10"}
```
下载 charts

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
helm fetch rancher-stable/rancher --version=v2.7.5
```

## 6. 生成证书
> 使用自己的证书文件，安装时通过`ingress.tls.source=secret`来标识。

create_self-signed-cert.sh 内容如下：

```bash
#!/bin/bash -e

help ()
{
    echo  ' ================================================================ '
    echo  ' --ssl-domain: 生成ssl证书需要的主域名，如不指定则默认为www.rancher.local，如果是ip访问服务，则可忽略；'
    echo  ' --ssl-trusted-ip: 一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开；'
    echo  ' --ssl-trusted-domain: 如果想多个域名访问，则添加扩展域名（SSL_TRUSTED_DOMAIN）,多个扩展域名用逗号隔开；'
    echo  ' --ssl-size: ssl加密位数，默认2048；'
    echo  ' --ssl-date: ssl有效期，默认10年；'
    echo  ' --ca-date: ca有效期，默认10年；'
    echo  ' --ssl-cn: 国家代码(2个字母的代号),默认CN;'
    echo  ' 使用示例:'
    echo  ' ./create_self-signed-cert.sh --ssl-domain=www.test.com --ssl-trusted-domain=www.test2.com \ '
    echo  ' --ssl-trusted-ip=1.1.1.1,2.2.2.2,3.3.3.3 --ssl-size=2048 --ssl-date=3650'
    echo  ' ================================================================'
}

case "$1" in
    -h|--help) help; exit;;
esac

if [[ $1 == '' ]];then
    help;
    exit;
fi

CMDOPTS="$*"
for OPTS in $CMDOPTS;
do
    key=$(echo ${OPTS} | awk -F"=" '{print $1}' )
    value=$(echo ${OPTS} | awk -F"=" '{print $2}' )
    case "$key" in
        --ssl-domain) SSL_DOMAIN=$value ;;
        --ssl-trusted-ip) SSL_TRUSTED_IP=$value ;;
        --ssl-trusted-domain) SSL_TRUSTED_DOMAIN=$value ;;
        --ssl-size) SSL_SIZE=$value ;;
        --ssl-date) SSL_DATE=$value ;;
        --ca-date) CA_DATE=$value ;;
        --ssl-cn) CN=$value ;;
    esac
done

# CA相关配置
CA_DATE=${CA_DATE:-3650}
CA_KEY=${CA_KEY:-cakey.pem}
CA_CERT=${CA_CERT:-cacerts.pem}
CA_DOMAIN=cattle-ca

# ssl相关配置
SSL_CONFIG=${SSL_CONFIG:-$PWD/openssl.cnf}
SSL_DOMAIN=${SSL_DOMAIN:-'www.rancher.local'}
SSL_DATE=${SSL_DATE:-3650}
SSL_SIZE=${SSL_SIZE:-2048}

## 国家代码(2个字母的代号),默认CN;
CN=${CN:-CN}

SSL_KEY=$SSL_DOMAIN.key
SSL_CSR=$SSL_DOMAIN.csr
SSL_CERT=$SSL_DOMAIN.crt

echo -e "\033[32m ---------------------------- \033[0m"
echo -e "\033[32m       | 生成 SSL Cert |       \033[0m"
echo -e "\033[32m ---------------------------- \033[0m"

if [[ -e ./${CA_KEY} ]]; then
    echo -e "\033[32m ====> 1. 发现已存在CA私钥，备份"${CA_KEY}"为"${CA_KEY}"-bak，然后重新创建 \033[0m"
    mv ${CA_KEY} "${CA_KEY}"-bak
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}
else
    echo -e "\033[32m ====> 1. 生成新的CA私钥 ${CA_KEY} \033[0m"
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}
fi

if [[ -e ./${CA_CERT} ]]; then
    echo -e "\033[32m ====> 2. 发现已存在CA证书，先备份"${CA_CERT}"为"${CA_CERT}"-bak，然后重新创建 \033[0m"
    mv ${CA_CERT} "${CA_CERT}"-bak
    openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
else
    echo -e "\033[32m ====> 2. 生成新的CA证书 ${CA_CERT} \033[0m"
    openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
fi

echo -e "\033[32m ====> 3. 生成Openssl配置文件 ${SSL_CONFIG} \033[0m"
cat > ${SSL_CONFIG} <<EOM
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, serverAuth
EOM

if [[ -n ${SSL_TRUSTED_IP} || -n ${SSL_TRUSTED_DOMAIN} ]]; then
    cat >> ${SSL_CONFIG} <<EOM
subjectAltName = @alt_names
[alt_names]
EOM
    IFS=","
    dns=(${SSL_TRUSTED_DOMAIN})
    dns+=(${SSL_DOMAIN})
    for i in "${!dns[@]}"; do
      echo DNS.$((i+1)) = ${dns[$i]} >> ${SSL_CONFIG}
    done

    if [[ -n ${SSL_TRUSTED_IP} ]]; then
        ip=(${SSL_TRUSTED_IP})
        for i in "${!ip[@]}"; do
          echo IP.$((i+1)) = ${ip[$i]} >> ${SSL_CONFIG}
        done
    fi
fi

echo -e "\033[32m ====> 4. 生成服务SSL KEY ${SSL_KEY} \033[0m"
openssl genrsa -out ${SSL_KEY} ${SSL_SIZE}

echo -e "\033[32m ====> 5. 生成服务SSL CSR ${SSL_CSR} \033[0m"
openssl req -sha256 -new -key ${SSL_KEY} -out ${SSL_CSR} -subj "/C=${CN}/CN=${SSL_DOMAIN}" -config ${SSL_CONFIG}

echo -e "\033[32m ====> 6. 生成服务SSL CERT ${SSL_CERT} \033[0m"
openssl x509 -sha256 -req -in ${SSL_CSR} -CA ${CA_CERT} \
    -CAkey ${CA_KEY} -CAcreateserial -out ${SSL_CERT} \
    -days ${SSL_DATE} -extensions v3_req \
    -extfile ${SSL_CONFIG}

echo -e "\033[32m ====> 7. 证书制作完成 \033[0m"
echo
echo -e "\033[32m ====> 8. 以YAML格式输出结果 \033[0m"
echo "----------------------------------------------------------"
echo "ca_key: |"
cat $CA_KEY | sed 's/^/  /'
echo
echo "ca_cert: |"
cat $CA_CERT | sed 's/^/  /'
echo
echo "ssl_key: |"
cat $SSL_KEY | sed 's/^/  /'
echo
echo "ssl_csr: |"
cat $SSL_CSR | sed 's/^/  /'
echo
echo "ssl_cert: |"
cat $SSL_CERT | sed 's/^/  /'
echo

echo -e "\033[32m ====> 9. 附加CA证书到Cert文件 \033[0m"
cat ${CA_CERT} >> ${SSL_CERT}
echo "ssl_cert: |"
cat $SSL_CERT | sed 's/^/  /'
echo

echo -e "\033[32m ====> 10. 重命名服务证书 \033[0m"
echo "cp ${SSL_DOMAIN}.key tls.key"
cp ${SSL_DOMAIN}.key tls.key
echo "cp ${SSL_DOMAIN}.crt tls.crt"
cp ${SSL_DOMAIN}.crt tls.crt
```

- `rancher.ghostwritten.com`是自定义的测试域名。
- --ssl-domain: 生成ssl证书需要的主域名，如不指定则默认为www.rancher.local，如果是ip访问服务，则可忽略；
- --ssl-trusted-ip: 一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开；
- --ssl-trusted-domain: 如果想多个域名访问，则添加扩展域名（TRUSTED_DOMAIN）,多个TRUSTED_DOMAIN用逗号隔开；
- --ssl-size: ssl加密位数，默认2048；
- --ssl-cn: 国家代码(2个字母的代号),默认CN；

```bash
$ bash  create_self-signed-cert.sh --ssl-domain=rancher.ghostwritten.com --ssl-trusted-ip=192.168.23.44,192.168.23.45,192.168.23.46 --ssl-size=2048 --ssl-date=3650

$ ls
cacerts.pem  cacerts.srl  cakey.pem  create_self-signed-cert.sh  openssl.cnf  rancher-2.7.5.tgz  rancher.ghostwritten.com.crt  rancher.ghostwritten.com.csr  rancher.ghostwritten.com.key  tls.crt  tls.key
```


## 7. 镜像入库

- [如何在 centos 7.9 部署 harbor 镜像仓库](https://blog.csdn.net/xixihahalelehehe/article/details/127920005)

```bash
docker pull docker.io/rancher/rancher:v2.7.5
docker pull docker.io/rancher/fleet:v0.7.0
docker pull docker.io/rancher/fleet-agent:v0.7.0
docker pull docker.io/rancher/gitjob:v0.1.54
docker pull docker.io/rancher/shell:v0.1.20
docker pull docker.io/rancher/rancher-webhook:v0.3.5 

docker tag docker.io/rancher/rancher:v2.7.5 harbor01.ghostwritten.com/rancher/rancher:v2.7.5
docker tag  docker.io/rancher/fleet:v0.7.0 harbor01.ghostwritten.com/rancher/fleet:v0.7.0
docker tag  docker.io/rancher/fleet-agent:v0.7.0 harbor01.ghostwritten.com/rancher/fleet-agent:v0.7.0
docker tag  docker.io/rancher/gitjob:v0.1.54 harbor01.ghostwritten.com/rancher/gitjob:v0.1.54
docker tag  docker.io/rancher/shell:v0.1.20 harbor01.ghostwritten.com/rancher/shell:v0.1.20
docker tag  docker.io/rancher/rancher-webhook:v0.3.5 harbor01.ghostwritten.com/rancher/rancher-webhook:v0.3.5

docker push harbor01.ghostwritten.com/rancher/rancher:v2.7.5
docker push harbor01.ghostwritten.com/rancher/fleet:v0.7.0
docker push harbor01.ghostwritten.com/rancher/fleet-agent:v0.7.0
docker push harbor01.ghostwritten.com/rancher/gitjob:v0.1.54
docker push harbor01.ghostwritten.com/rancher/shell:v0.1.20
docker push harbor01.ghostwritten.com/rancher/rancher-webhook:v0.3.5
```

## 8. 安装 rancher

```bash
kubectl create ns cattle-system
```
helm 渲染中 -`-set privateCA=true` 用到的证书：

```bash
kubectl -n cattle-system create secret generic tls-ca --from-file=cacerts.pem
```
helm 渲染中 `--set additionalTrustedCAs=true` 用到的证书：

```bash
cp cacerts.pem ca-additional.pem
kubectl -n cattle-system create secret generic tls-ca-additional --from-file=ca-additional.pem
```

helm 渲染中 `--set ingress.tls.source=secret` 用到的证书和密钥：

```bash
kubectl -n cattle-system create secret tls tls-rancher-ingress --cert=tls.crt --key=tls.key
```
使用以下命令创建 rancher：

```bash
   helm install rancher ./rancher-2.7.5.tgz \
    --namespace cattle-system \
    --set hostname=rancher.ghostwritten.com \
    --set rancherImage=harbor01.ghostwritten.com/rancher/rancher \
    --set ingress.tls.source=secret \
    --set privateCA=true \
    --set systemDefaultRegistry=harbor01.ghostwritten.com \
    --set useBundledSystemChart=true 
```
输出：

```bash
$   helm install rancher ./rancher-2.7.5.tgz \
>     --namespace cattle-system \
>     --set hostname=rancher.ghostwritten.com \
>     --set rancherImage=harbor01.ghostwritten.com/rancher/rancher \
>     --set ingress.tls.source=secret \
>     --set privateCA=true \
>     --set systemDefaultRegistry=harbor01.ghostwritten.com \
>     --set useBundledSystemChart=true 
NAME: rancher
LAST DEPLOYED: Sun Sep 10 21:08:01 2023
NAMESPACE: cattle-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Rancher Server has been installed.

NOTE: Rancher may take several minutes to fully initialize. Please standby while Certificates are being issued, Containers are started and the Ingress rule comes up.

Check out our docs at https://rancher.com/docs/

If you provided your own bootstrap password during installation, browse to https://rancher.ghostwritten.com to get started.

If this is the first time you installed Rancher, get started by running this command and clicking the URL it generates:


echo https://rancher.ghostwritten.com/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')


To get just the bootstrap password on its own, run:


kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'



Happy Containering!

```
查看 pod 创建状态
```bash
$ kubectl get all -n cattle-system
NAME                                  READY   STATUS      RESTARTS       AGE
pod/helm-operation-9b2ng              0/2     Completed   0              13m
pod/helm-operation-d6m4l              0/2     Completed   0              16m
pod/helm-operation-f7xgh              0/2     Completed   0              15m
pod/helm-operation-nvm49              0/2     Completed   0              13m
pod/rancher-574985fc95-8s9tq          1/1     Running     1 (6m7s ago)   20m
pod/rancher-574985fc95-mskmj          1/1     Running     0              20m
pod/rancher-574985fc95-q5c58          1/1     Running     0              20m
pod/rancher-webhook-db869ffdc-csp9m   1/1     Running     0              13m

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/rancher           ClusterIP   10.255.21.131    <none>        80/TCP,443/TCP   20m
service/rancher-webhook   ClusterIP   10.255.90.146    <none>        443/TCP          13m
service/webhook-service   ClusterIP   10.255.198.145   <none>        443/TCP          13m

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/rancher           3/3     3            3           20m
deployment.apps/rancher-webhook   1/1     1            1           13m

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/rancher-574985fc95          3         3         3       20m
replicaset.apps/rancher-webhook-db869ffdc   1         1         1       13m

```


注意：
有些pod需要手动删除

```bash
$ kubectl delete <pod名称> -n cattle-system
```
例如:

```bash
helm-operation-nbzz8              1/2     Error       0              12m
helm-operation-tsx5t              1/2     Error       0              13m
```


还有其他自动创建的命名空间，查看是否存在异常资源对象

```bash
$ kubectl get ns
NAME                                     STATUS   AGE
cattle-fleet-clusters-system             Active   14m
cattle-fleet-system                      Active   17m
cattle-global-data                       Active   17m
cattle-global-nt                         Active   17m
cattle-impersonation-system              Active   17m
cattle-system                            Active   49m
cluster-fleet-local-local-1a3d67d0a899   Active   14m
default                                  Active   24h
fleet-default                            Active   17m
fleet-local                              Active   18m
ingress-nginx                            Active   20h
kube-node-lease                          Active   24h
kube-public                              Active   24h
kube-system                              Active   24h
kubernetes-dashboard                     Active   21h
local                                    Active   17m
p-9qgwm                                  Active   17m
p-j8jn8                                  Active   17m
$ kubectl get pod -n cattle-fleet-clusters-system
No resources found in cattle-fleet-clusters-system namespace.
$  kubectl get all -n cattle-fleet-clusters-system
No resources found in cattle-fleet-clusters-system namespace.
$ kubectl get all -n cattle-fleet-system
NAME                                    READY   STATUS    RESTARTS   AGE
pod/fleet-controller-85c9b74cc4-vth7l   1/1     Running   0          15m
pod/gitjob-68858c7cc-578hl              1/1     Running   0          15m

NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/gitjob   ClusterIP   10.255.221.37   <none>        80/TCP    15m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/fleet-controller   1/1     1            1           15m
deployment.apps/gitjob             1/1     1            1           15m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/fleet-controller-85c9b74cc4   1         1         1       15m
replicaset.apps/gitjob-68858c7cc              1         1         1       15m
$ kubectl get all -n cattle-global-data
No resources found in cattle-global-data namespace.
$ kubectl get all -n cattle-global-nt 
No resources found in cattle-global-nt namespace.
$ kubectl get all -n cattle-impersonation-system
No resources found in cattle-impersonation-system namespace.
$ kubectl get all -n cluster-fleet-local-local-1a3d67d0a899
No resources found in cluster-fleet-local-local-1a3d67d0a899 namespace.
$ kubectl get all -n fleet-default
No resources found in fleet-default namespace.
$ kubectl get all -n fleet-local 
No resources found in fleet-local namespace.
$ kubectl get all -n local   
No resources found in local namespace.
$ kubectl get all -n p-9qgwm 
No resources found in p-9qgwm namespace.
$ kubectl get all -n p-j8jn8
No resources found in p-j8jn8 namespace.
```


##  9. 配置 nodeport 

```bash
$ kubectl get svc -n cattle-system
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
rancher           ClusterIP   10.255.21.131    <none>        80/TCP,443/TCP   29m
rancher-webhook   ClusterIP   10.255.90.146    <none>        443/TCP          22m
webhook-service   ClusterIP   10.255.198.145   <none>        443/TCP          22m

```
修改内容：

```bash
$ kubectl get svc -n cattle-system rancher -oyaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    field.cattle.io/publicEndpoints: '[{"port":30012,"protocol":"TCP","serviceName":"cattle-system:rancher","allNodes":true},{"port":30013,"protocol":"TCP","serviceName":"cattle-system:rancher","allNodes":true}]'
    meta.helm.sh/release-name: rancher
    meta.helm.sh/release-namespace: cattle-system
  creationTimestamp: "2023-09-10T13:08:03Z"
  labels:
    app: rancher
    app.kubernetes.io/managed-by: Helm
    chart: rancher-2.7.5
    heritage: Helm
    release: rancher
  name: rancher
  namespace: cattle-system
  resourceVersion: "179411"
  uid: c9f7031b-b913-4f60-8487-32ffcc274311
spec:
  clusterIP: 10.255.21.131  #删除，修改后会自动生成
  clusterIPs:    #删除，修改后会自动生成
  - 10.255.21.131   #删除，修改后会自动生成
  externalTrafficPolicy: Cluster  #修改后生成的
  internalTrafficPolicy: Cluster  #删除，修改后会自动生成
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    nodePort: 30012  #添加
    port: 80
    protocol: TCP
    targetPort: 80
  - name: https-internal
    nodePort: 30013    #添加
    port: 443
    protocol: TCP
    targetPort: 444
  selector:
    app: rancher
  sessionAffinity: None
  type: NodePort   #ClusterIP修改为NodePort
status:
  loadBalancer: {}

```
最后效果：

```bash
$ kubectl get svc -n cattle-system rancher
NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
rancher   NodePort   10.255.21.131   <none>        80:30012/TCP,443:30013/TCP   50m
```

## 10. 配置 ingress
需要添加如下配置，否则 ingress 无法获取 address

```bash
annotations:
  kubernetes.io/ingress.class: nginx  #添加
```


修改前

```bash
$ kubectl get ingress -n cattle-system 
NAME      CLASS    HOSTS              ADDRESS   PORTS     AGE
rancher   <none>   rancher.ghostwritten.com             80, 443   39m
```
当前，这样我们无法通过域名访问。

修改后

```bash
$ kubectl get ingress -n cattle-system 
NAME      CLASS    HOSTS              ADDRESS         PORTS     AGE
rancher   <none>   rancher.ghostwritten.com   192.168.23.45   80, 443   54m
```


## 11. 界面访问
配置域名解析：`C:\Windows\System32\drivers\etc\hosts`

- 1`92.168.23.45 rancher.ghostwritten.com`

![](https://img-blog.csdnimg.cn/adb9ca73e90244d6b33a5504b2688a29.png)

获取密码

```bash
$ kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{"\n"}}'
6xn88btlppbjgpktnvl5mf4mrvcqz498qv27q6kqkjq4jnddng4qqx
```
界面输入密码点击后，获取新密码。
![](https://img-blog.csdnimg.cn/a0714aeb7a3740b79fa359e5abe13bdb.png)

新密码：`gVCladEKFq9oViCa`
访问：`https://rancher.ghostwritten.com:30013`

### 11.1 首页预览
![](https://img-blog.csdnimg.cn/185716dee34f47b78349f1f56eaa342b.png)
### 11.2  查看集群信息

点击 local 集群，查看集群信息
![](https://img-blog.csdnimg.cn/d7e7b07bde414435b18846af9eaf824f.png)
### 11.3 查看项目空间
![](https://img-blog.csdnimg.cn/5e6f7cabf90f401fb1363d3778d43767.png)

###  11.4 查看节点信息

![](https://img-blog.csdnimg.cn/0f633c95e49045c0969a413b545f7006.png)
参考：
- [https://juejin.cn/post/6943494889564962823](https://juejin.cn/post/6943494889564962823)
- [离线安装 rancher](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/other-installation-methods/air-gapped-helm-cli-install/install-rancher-ha)
- [添加 TLS 密文](https://ranchermanager.docs.rancher.com/zh/getting-started/installation-and-upgrade/resources/add-tls-secrets)
- [Helm3部署Rancher2.6.3高可用集群](https://mp.weixin.qq.com/s/oncqV6e-ujPjXxWFsqP7cA)
- [Rancher2.5.5离线高可用安装示例](https://mp.weixin.qq.com/s/M_fT92wTLalfRb4R8v3tjg)
