![](https://i-blog.csdnimg.cn/blog_migrate/f58dbe6be5af56e784284c753d76381f.jpeg#pic_center)






<font color=red>声明：</font>
- <font color=red>该文档仅用于测试开发环境，而非企业生产环境；</font>
- <font color=red>该文档为在线部署，而非离线环境；</font>

## 1. 简介
[Rancher](https://ranchermanager.docs.rancher.com/zh/) 是一个 Kubernetes 管理工具，让你能在任何地方和任何提供商上部署和运行集群。

Rancher 可以创建来自 Kubernetes 托管服务提供商的集群，创建节点并安装 Kubernetes，或者导入在任何地方运行的现有 Kubernetes 集群。

Rancher 基于 Kubernetes 添加了新的功能，包括统一所有集群的身份验证和 RBAC，让系统管理员从一个位置控制全部集群的访问。

此外，Rancher 可以为集群和资源提供更精细的监控和告警，将日志发送到外部提供商，并通过应用商店（Application Catalog）直接集成 Helm。如果你拥有外部 CI/CD 系统，你可以将其与 Rancher 对接。没有的话，你也可以使用 Rancher 提供的 Fleet 自动部署和升级工作负载。

Rancher 是一个 全栈式 的 Kubernetes 容器管理平台，为你提供在任何地方都能成功运行 Kubernetes 的工具。

## 2. 预备条件

- 所有支持的操作系统都使用 64-bit x86 架构。Rancher 兼容当前所有的主流 Linux 发行版。

- [查询 kubernetes 与 rancher 兼容性](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-7-9/)
- 请安装 ntp（Network Time Protocol），以防止在客户端和服务器之间由于时间不同步造成的证书验证错误。

- 某些 Linux 发行版的默认防火墙规则可能会阻止 Kubernetes 集群内的通信。从 Kubernetes v1.19 开始，你必须关闭 firewalld，因为它与 Kubernetes 网络插件冲突。
- [安装 kubernetes](https://ghostwritten.blog.csdn.net/article/details/134413829) ，这里我选择 rke2 方式
- 私有镜像仓库：你可以选择[安装 harbor](https://blog.csdn.net/xixihahalelehehe/article/details/127920005) 或者 [安装 registry](https://blog.csdn.net/xixihahalelehehe/article/details/105926147)


## 3. 推荐姿势
## 3.1 为什么是三个节点？​
在RKE集群中，Rancher服务器数据存储在etcd上。这个etcd数据库在所有三个节点上运行。
etcd数据库需要奇数个节点，这样它总是可以选出一个拥有大多数etcd集群的领导者。如果etcd数据库不能选出一个领导者，etcd可能会遭受分裂的大脑，需要从备份中恢复集群。如果三个etcd节点中的一个失败，剩下的两个节点可以选举一个领导者，因为它们拥有etcd节点总数的大多数。

## 3.2  为何采用自签名 SSL 证书无需cert-manager
来自官方文档推荐：[rancher 高可用安装指南](https://docs.rancher.cn/docs/rancher2.5/installation/install-rancher-on-k8s/_index)

![](https://i-blog.csdnimg.cn/blog_migrate/1910ac1c30cbe5206803b5e2e78b457b.png)

## 4. 下载介质

创建介质目录
```bash
mkdir rancher-2.8.2 && cd rancher-2.8.2
```

### 4.1 下载helm

```bash
wget https://get.helm.sh/helm-v3.14.2-linux-amd64.tar.gz
```


### 4.2 下载charts 包

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update
helm fetch rancher-stable/rancher --version=v2.8.2
```

### 4.3 下载镜像

#### 	4.3.1 获取rancher镜像列表

这里通过在线部署的rancher v2.8.2获取镜像列表命令
```bash
$ for i in `kubectl get ns  |grep cattle | awk '{print $1}'`;do kubectl get pods -o custom-columns='IMAGE:spec.containers[0].image' -n $i | grep -v IMAGE;done
```
安装 rancher 镜像列表：

```bash
rancher/fleet-agent:v0.9.0
rancher/fleet:v0.9.0
rancher/gitjob:v0.1.96
rancher/mirrored-cluster-api-controller:v1.4.4
rancher/rancher:v2.8.2
rancher/rancher-webhook:v0.4.2
```

#### 4.3.2 获取集群注册rancher-agent组件镜像列表
获取注册已有集群的rancher-agent组件镜像列表命令
 
```bash
$ for i in `kubectl get ns  |grep cattle | awk '{print $1}'`;do kubectl get pods -o custom-columns='IMAGE:spec.containers[0].image' -n $i | grep -v IMAGE;done
rancher/fleet-agent:v0.9.0
rancher/rancher-agent:v2.8.2
rancher/rancher-webhook:v0.4.2
```


#### 4.3.3 获取卸载rancher组件镜像列表

下载项目包
```bash
git clone https://github.com/rancher/rancher-cleanup.git
tar zcvf rancher-cleanup.tar.gz rancher-cleanup/
```
获取镜像列表
```bash
$ cat rancher-cleanup/deploy/* |grep image:| sort -n | uniq | awk '{print $2}'
rancher/rancher-cleanup:latest
```

#### 4.3.4 汇总所用镜像列表
镜像批量打包、拉取、入库脚本参考：[容器镜像搬运最佳脚本](https://blog.csdn.net/xixihahalelehehe/article/details/135082812)

所需镜像列表至rancher-image.txt

```bash
rancher/fleet-agent:v0.9.0
rancher/fleet:v0.9.0
rancher/gitjob:v0.1.96
rancher/mirrored-cluster-api-controller:v1.4.4
rancher/rancher:v2.8.2
rancher/rancher:v2.8.2
rancher/rancher:v2.8.2
rancher/rancher-webhook:v0.4.2
rancher/rancher-agent:v2.8.2
rancher/rancher-cleanup:latest
```

执行：
```bash
sh images.sh pull
sh images.sh save
```

### 4.4 生成证书脚本

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

### 4.5 安装 rancher 脚本

```bash
$ vim install_rancher.sh
helm install rancher ./rancher-2.8.2.tgz \
    --namespace cattle-system \
    --set hostname=rancher02.demo.com \
    --set rancherImage=harbor.ghostwritten.com/rancher/rancher \
    --set ingress.tls.source=secret \
    --set privateCA=true \
    --set systemDefaultRegistry=harbor.ghostwritten.com \
    --set useBundledSystemChart=true 
```






介质列表：
rancher-2.8.2/
├── certs
│   ├── create_certs.sh
│   └── create_self-signed-cert.sh
├── images
│   ├── rancher_fleet-agent_v0.9.0.tar
│   ├── rancher_fleet_v0.9.0.tar
│   ├── rancher_gitjob_v0.1.96.tar
│   ├── rancher_mirrored-cluster-api-controller_v1.4.4.tar
│   ├── rancher_rancher-agent_v2.8.2.tar
│   ├── rancher_rancher-cleanup_latest.tar
│   ├── rancher_rancher_v2.8.2.tar
│   └── rancher_rancher-webhook_v0.4.2.tar
├── images.sh
├── install_rancher.sh
├── rancher-2.8.2.tgz
├── rancher-cleanup.tar.gz
└── rancher-images.txt

2 directories, 15 files
### 4.6 介质打包

```bash
cd ..
tar zcvf rancher-2.8.2.tar.gz rancher-2.8.2
```

搬运rancher介质包至离线环境。

## 5. 安装
解压介质包

```bash
tar zxvf rancher-2.8.2.tar.gz 
cd rancher-2.8.2
```

### 5.1 安装 helm

```bash
tar -xzvf helm-v3.14.2-linux-amd64.tar.gz
cp linux-amd64/helm /usr/local/bin/
helm version
rm -rf linux-amd64
```

### 5.2 镜像入库
设置仓库与项目名
```bash
$ vim images.sh 
registry_name='harbor.ghostwritten.com'
project='rancher'

$ sh images.sh load
$ sh images.sh push
```


### 5.3 生成证书


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

### 5.4 安装 rancher v2.8.2

创建cattle-system命令空间
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
使用以下命令行安装 rancher：

```bash
helm install rancher ./rancher-2.8.2.tgz \
    --namespace cattle-system \
    --set hostname=rancher02.demo.com \
    --set rancherImage=harbor.ghostwritten.com/rancher/rancher \
    --set ingress.tls.source=secret \
    --set privateCA=true \
    --set systemDefaultRegistry=harbor.ghostwritten.com \
    --set useBundledSystemChart=true 


```
或者执行脚本：

```bash
sh install_rancher.sh
```

## 6. 卸载 rancher

```bash
cd rancher-cleanup
kubectl create -f deploy/rancher-cleanup.yaml
kubectl  -n kube-system logs -l job-name=cleanup-job  -f

kubectl create -f deploy/verify.yaml
kubectl  -n kube-system logs -l job-name=verify-job  -f
kubectl  -n kube-system logs -l job-name=verify-job  -f | grep -v "is deprecated"
```


结束。
![](https://i-blog.csdnimg.cn/blog_migrate/b399282d69147fdefea5c889ed852b99.jpeg#pic_center)




