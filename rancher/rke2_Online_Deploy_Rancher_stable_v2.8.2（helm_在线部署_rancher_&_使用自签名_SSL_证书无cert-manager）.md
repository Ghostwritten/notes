![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/36af470f116c909fe8ec8f6e9f02c185.jpeg#pic_center)






<font color=red>声明：</font>
- <font color=red>该文档仅用于测试开发环境，而非企业生产环境；</font>
- <font color=red>该文档为在线部署，而非离线环境；</font>


## 1. 简介
[Rancher](https://ranchermanager.docs.rancher.com/zh/) 是一个 Kubernetes 管理工具，让你能在任何地方和任何提供商上部署和运行集群。

Rancher 可以创建来自 Kubernetes 托管服务提供商的集群，创建节点并安装 Kubernetes，或者导入在任何地方运行的现有 Kubernetes 集群。

Rancher 基于 Kubernetes 添加了新的功能，包括统一所有集群的身份验证和 RBAC，让系统管理员从一个位置控制全部集群的访问。

此外，Rancher 可以为集群和资源提供更精细的监控和告警，将日志发送到外部提供商，并通过应用商店（Application Catalog）直接集成 Helm。如果你拥有外部 CI/CD 系统，你可以将其与 Rancher 对接。没有的话，你也可以使用 Rancher 提供的 Fleet 自动部署和升级工作负载。

Rancher 是一个 全栈式 的 Kubernetes 容器管理平台，为你提供在任何地方都能成功运行 Kubernetes 的工具。


## 2. 为何采用自签名 SSL 证书无需cert-manager
来自官方文档推荐：[rancher 高可用安装指南](https://docs.rancher.cn/docs/rancher2.5/installation/install-rancher-on-k8s/_index)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1910ac1c30cbe5206803b5e2e78b457b.png)

## 3. 预备条件
- [安装 kubernetes](https://ghostwritten.blog.csdn.net/article/details/134413829) ，这里我选择 rke2 方式
- [Ingress Controller](https://blog.csdn.net/xixihahalelehehe/article/details/112831123)（仅适用于托管 Kubernetes）

## 4. 安装 helm

```bash
wget https://get.helm.sh/helm-v3.14.2-linux-amd64.tar.gz
tar -xzvf helm-v3.14.2-linux-amd64.tar.gz
cp linux-amd64/helm /usr/local/bin/
helm version
rm -rf linux-amd64 helm-v3.14.2-linux-amd64.tar.gz
```

## 5. 生成证书
参考官方文档：[生成自签名 SSL 证书](https://docs.rancher.cn/docs/rancher2.5/installation/resources/advanced/self-signed-ssl/_index/#41-%E4%B8%80%E9%94%AE%E7%94%9F%E6%88%90-ssl-%E8%87%AA%E7%AD%BE%E5%90%8D%E8%AF%81%E4%B9%A6%E8%84%9A%E6%9C%AC)


> 使用自己的证书文件，安装时通过`ingress.tls.source=secret`来标识。

create_self-signed-cert.sh 内容如下：

```bash
#!/bin/bash

help ()
{
    echo  ' ================================================================ '
    echo  ' --ssl-domain: 生成ssl证书需要的主域名，如不指定则默认为www.rancher.local，如果是ip访问服务，则可忽略；'
    echo  ' --ssl-trusted-ip: 一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开；'
    echo  ' --ssl-trusted-domain: 如果想多个域名访问，则添加扩展域名（SSL_TRUSTED_DOMAIN）,多个扩展域名用逗号隔开；'
    echo  ' --ssl-size: ssl加密位数，默认2048；'
    echo  ' --ssl-cn: 国家代码(2个字母的代号),默认CN;'
    echo  ' --ca-cert-recreate: 是否重新创建 ca-cert，ca 证书默认有效期 10 年，创建的 ssl 证书有效期如果是一年需要续签，那么可以直接复用原来的 ca 证书，默认 false;'
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
        --ca-cert-recreate) CA_CERT_RECREATE=$value ;;
        --ca-key-recreate) CA_KEY_RECREATE=$value ;;
    esac
done

# CA相关配置
CA_KEY_RECREATE=${CA_KEY_RECREATE:-false}
CA_CERT_RECREATE=${CA_CERT_RECREATE:-false}

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

# 如果存在 ca-key， 并且需要重新创建 ca-key
if [[ -e ./${CA_KEY} ]] && [[ ${CA_KEY_RECREATE} == 'true' ]]; then

    # 先备份旧 ca-key，然后重新创建 ca-key
    echo -e "\033[32m ====> 1. 发现已存在 CA 私钥，备份 "${CA_KEY}" 为 "${CA_KEY}"-bak，然后重新创建 \033[0m"
    mv ${CA_KEY} "${CA_KEY}"-bak-$(date +"%Y%m%d%H%M")
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}

    # 如果存在 ca-cert，因为 ca-key 重新创建，则需要重新创建 ca-cert。先备份然后重新创建 ca-cert
    if [[ -e ./${CA_CERT} ]]; then
        echo -e "\033[32m ====> 2. 发现已存在 CA 证书，先备份 "${CA_CERT}" 为 "${CA_CERT}"-bak，然后重新创建 \033[0m"
        mv ${CA_CERT} "${CA_CERT}"-bak-$(date +"%Y%m%d%H%M")
        openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
    else
        # 如果不存在 ca-cert，直接创建 ca-cert
        echo -e "\033[32m ====> 2. 生成新的 CA 证书 ${CA_CERT} \033[0m"
        openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
    fi

# 如果存在 ca-key，并且不需要重新创建 ca-key
elif [[ -e ./${CA_KEY} ]] && [[ ${CA_KEY_RECREATE} == 'false' ]]; then

    # 存在旧 ca-key，不需要重新创建，直接复用
    echo -e "\033[32m ====> 1. 发现已存在 CA 私钥，直接复用 CA 私钥 "${CA_KEY}" \033[0m"

    # 如果存在 ca-cert，并且需要重新创建 ca-cert。先备份然后重新创建
    if [[ -e ./${CA_CERT} ]] && [[ ${CA_CERT_RECREATE} == 'true' ]]; then
        echo -e "\033[32m ====> 2. 发现已存在 CA 证书，先备份 "${CA_CERT}" 为 "${CA_CERT}"-bak，然后重新创建 \033[0m"
        mv ${CA_CERT} "${CA_CERT}"-bak-$(date +"%Y%m%d%H%M")
        openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"

    # 如果存在 ca-cert，并且不需要重新创建 ca-cert，直接复用
    elif [[ -e ./${CA_CERT} ]] && [[ ${CA_CERT_RECREATE} == 'false' ]]; then
        echo -e "\033[32m ====> 2. 发现已存在 CA 证书，直接复用 CA 证书 "${CA_CERT}" \033[0m"
    else
        # 如果不存在 ca-cert ，直接创建 ca-cert
        echo -e "\033[32m ====> 2. 生成新的 CA 证书 ${CA_CERT} \033[0m"
        openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
    fi

# 如果不存在 ca-key
else
    # ca-key 不存在，直接生成
    echo -e "\033[32m ====> 1. 生成新的 CA 私钥 ${CA_KEY} \033[0m"
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}

    # 如果存在旧的 ca-cert，先做备份，然后重新生成 ca-cert
    if [[ -e ./${CA_CERT} ]]; then
        echo -e "\033[32m ====> 2. 发现已存在 CA 证书，先备份 "${CA_CERT}" 为 "${CA_CERT}"-bak，然后重新创建 \033[0m"
        mv ${CA_CERT} "${CA_CERT}"-bak-$(date +"%Y%m%d%H%M")
        openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
    else
        # 不存在旧的 ca-cert，直接生成 ca-cert
        echo -e "\033[32m ====> 2. 生成新的 CA 证书 ${CA_CERT} \033[0m"
        openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
    fi

fi

echo -e "\033[32m ====> 3. 生成 Openssl 配置文件 ${SSL_CONFIG} \033[0m"
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

if [[ -n ${SSL_TRUSTED_IP} || -n ${SSL_TRUSTED_DOMAIN} || -n ${SSL_DOMAIN} ]]; then
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

echo -e "\033[32m ====> 4. 生成服务 SSL KEY ${SSL_KEY} \033[0m"
openssl genrsa -out ${SSL_KEY} ${SSL_SIZE}

echo -e "\033[32m ====> 5. 生成服务 SSL CSR ${SSL_CSR} \033[0m"
openssl req -sha256 -new -key ${SSL_KEY} -out ${SSL_CSR} -subj "/C=${CN}/CN=${SSL_DOMAIN}" -config ${SSL_CONFIG}

echo -e "\033[32m ====> 6. 生成服务 SSL CERT ${SSL_CERT} \033[0m"
openssl x509 -sha256 -req -in ${SSL_CSR} -CA ${CA_CERT} \
    -CAkey ${CA_KEY} -CAcreateserial -out ${SSL_CERT} \
    -days ${SSL_DATE} -extensions v3_req \
    -extfile ${SSL_CONFIG}

echo -e "\033[32m ====> 7. 证书制作完成 \033[0m"
echo
echo -e "\033[32m ====> 8. 以 YAML 格式输出结果 \033[0m"
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

echo -e "\033[32m ====> 9. 附加 CA 证书到 Cert 文件 \033[0m"
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
- `rancher02.demo.com`是自定义的测试域名。
- --ssl-domain: 生成ssl证书需要的主域名，如不指定则默认为www.rancher.local，如果是ip访问服务，则可忽略；
- --ssl-trusted-ip: 一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开；
- --ssl-trusted-domain: 如果想多个域名访问，则添加扩展域名（TRUSTED_DOMAIN）,多个TRUSTED_DOMAIN用逗号隔开；
- --ssl-size: ssl加密位数，默认2048；
- --ssl-cn: 国家代码(2个字母的代号),默认CN；

```bash
$ bash  create_self-signed-cert.sh --ssl-domain=rancher02.demo.com --ssl-trusted-ip=192.168.23.80 --ssl-size=2048 --ssl-date=3650

$ ls
ca-additional.pem  cacerts.srl  create_certs.sh             install_rancher.sh  rancher02.demo.com.crt  rancher02.demo.com.key  tls.key
cacerts.pem        cakey.pem    create_self-signed-cert.sh  openssl.cnf         rancher02.demo.com.csr  tls.crt
```
## 6. 安装 rancher

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```
查看库列表

```bash
$ helm repo list
NAME            URL                                                            
rancher-stable  https://releases.rancher.com/server-charts/stable
```

查询stable版本

```bash
$ helm search hub rancher
URL                                                     CHART VERSION           APP VERSION     DESCRIPTION                                       
https://artifacthub.io/packages/helm/rancher-la...      2.8.2                   v2.8.2          Install Rancher Server to manage Kubernetes clu...
https://artifacthub.io/packages/helm/rancher-st...      2.8.2                   v2.8.2          Install Rancher Server to manage Kubernetes clu...
https://artifacthub.io/packages/helm/wenerme/ra...      2.8.2                   v2.8.2          Install Rancher Server to manage Kubernetes clu...
https://artifacthub.io/packages/helm/wener/rancher      2.8.2                   v2.8.2          Install Rancher Server to manage Kubernetes clu...
https://artifacthub.io/packages/helm/rancher-au...      0.0.1                   0.0.1           A Helm chart for registering and importing a ne...
https://artifacthub.io/packages/helm/someblackm...      0.3.0                   0.3.0           grafana dashboards chart for rancher              
https://artifacthub.io/packages/helm/loft/vclus...      0.0.1                   0.0.1           vCluster.Pro plugin for Rancher                   
https://artifacthub.io/packages/helm/loft/vclus...      0.0.1                   0.0.1           vCluster.Pro plugin for Rancher                   
https://artifacthub.io/packages/helm/capsule/ca...      0.1.1                   v0.1.1          A Helm chart for Kubernetes                       
https://artifacthub.io/packages/helm/rke2-chart...      1.5.100                 1.26.1          vSphere Cloud Provider Interface (CPI)            
https://artifacthub.io/packages/helm/rke2-chart...      3.0.1-rancher101        3.0.1-rancher1  vSphere Cloud Storage Interface (CSI)             
https://artifacthub.io/packages/helm/supporttoo...      0.0.1-rc3               0.0.1-rc3       Rancher Systems Summary Report                    
https://artifacthub.io/packages/helm/cluster-te...      0.0.1                                   Cluster template for rke2                         
https://artifacthub.io/packages/helm/eugen/loca...      1.1.0                   0.0.26          Provisions the rancher local-path                 
https://artifacthub.io/packages/helm/kubegems/l...      0.0.22                  0.0.22          Rancher 开源的一款采用主机本地文件系统的Kuberne...
https://artifacthub.io/packages/helm/s3gw/s3gw          0.23.0                  latest          Easy-to-use Open Source and Cloud Native S3 ser...
```


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
helm install rancher rancher-stable/rancher --version 2.8.2 \
    --namespace cattle-system \
    --set hostname=rancher02.demo.com \
    --set ingress.tls.source=secret \
    --set privateCA=true
```
输出：

```bash
helm install rancher rancher-stable/rancher --version 2.8.2     --namespace cattle-system     --set hostname=rancher02.demo.com     --set ingress.tls.source=secret     --set privateCA=true
NAME: rancher
LAST DEPLOYED: Thu Feb 22 15:44:43 2024
NAMESPACE: cattle-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Rancher Server has been installed.

NOTE: Rancher may take several minutes to fully initialize. Please standby while Certificates are being issued, Containers are started and the Ingress rule comes up.

Check out our docs at https://rancher.com/docs/

If you provided your own bootstrap password during installation, browse to https://rancher02.demo.com to get started.

If this is the first time you installed Rancher, get started by running this command and clicking the URL it generates:


echo https://rancher02.demo.com/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')


To get just the bootstrap password on its own, run:


kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'



Happy Containering!
```

检查是否部署成功

```bash
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

界面初始化，随机生成密码：
- https://rancher02.demo.com
- admin
- fj54VYfx5VIfnN38

后访问首页

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a913365aa9aedde1caf999dd3e7b0f1.png)

结束。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ade7a0b09d1bf816c97ff9ae77ad45ce.jpeg#pic_center)


