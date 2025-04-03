
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4cfc07eded241158fca2266d9dcdbb0b.jpeg#pic_center)




## 1. 预备条件
- [helm 在线部署 rancher & 使用自签名 SSL 证书无cert-manager](https://ghostwritten.blog.csdn.net/article/details/136233917)
- [helm 离线部署 rancher & 使用自签名 SSL 证书无cert-manager](https://ghostwritten.blog.csdn.net/article/details/136273466)


## 2. 准备全部集群的直连 kubeconfig 配置文件
在默认情况下， Rancher UI 上复制的 kubeconfig 通过cluster agent代理连接到 K8S 集群。变更 SSL 证书会导致cluster agent无法连接 Rancher Server，从而导致kubectl无法使用 Rancher UI 上复制的 kubeconfig 去操作 K8S 集群。用户需要通过kubectl命令行修改配置，解决这个问题。在执行域名或 IP 变更之前，请准备好所有集群的直连 kubeconfig 配置文件，详情考参考：[恢复 kubectl 配置文件](https://docs.rancher.cn/docs/rancher2.5/cluster-admin/restore-kubecfg/_index/)


## 3. 准备证书
SSL 证书与域名或IP之间存在绑定关系，客户端通过域名或IP访问 Server 端时，需要进行 SSL 证书校验。如果客户端访问的域名或IP与 SSL 证书中预先绑定的域名或IP不一致，那么 SSL 会认为这个 Server 端是伪造的，导致证书校验失败，客户端无法连接 Server 端。如果您更换了域名或 IP，而且在生成 SSL 证书时没有绑定新的域名或 IP，出现上述问题就是一个正常的现象。解决该问题的方法也非常直接明了：生成一个新的 SSL 证书，绑定新的域名或 IP，替换已有的证书。如果您已经有后续域名或 IP 变更的具体规划，也可以在生成 SSL 证书 时绑定这次和以后需要替换的域名或 IP，这样可以减轻下次变更的工作量。总而言之，如果更换了 Server 端的域名或IP，一般会涉及到 SSL 证书更换。请参考下文，替换自签名 SSL 证书或权威认证证书。

自签名 ssl 证书

复制以下代码另存为create_self-signed-cert.sh或者其他您喜欢的文件名。修改代码开头的CN(域名)，如果需要使用 ip 去访问 Rancher Server，那么需要给 ssl 证书添加扩展 IP，多个 IP 用逗号隔开。如果想实现多个域名访问 Rancher Server，则添加扩展域名(SSL_DNS),多个SSL_DNS用逗号隔开。

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

权威认证证书,生成证书命令如下：

```bash
bash create_self-signed-cert.sh --ssl-domain=rancher02.demo02.com --ssl-trusted-ip=192.168.23.80 --ssl-size=2048 --ssl-date=3650
```


## 4. 更新证书
提示：证书与域名或 IP 有绑定关系，一般情况更换域名或 IP 需更换证书。如果之前配置的证书是一个通配证书或者之前配置的证书已经包含了需要变更的域名或 IP，那么证书则可以不用更换。

### 4.1 Rancher 单节点运行（默认容器自动生成自签名 SSL 证书）#
默认情况，通过docker run运行的 Rancher Server 容器，会自动为 Rancher 生成 SSL 证书，这个证书会自动绑定 Rancher 系统设置中server-url配置的域名或IP。如果更换了域名或IP，证书会自动更新，无需单独操作。

### 4.2 Rancher 单节点运行（外置自签名 SSL 证书）#
注意：操作前先备份，备份和恢复。

如果是以映射证书文件的方式运行的单容器 Rancher Server，只需要停止原有 Rancher Server 容器，用新证书替换旧证书，保持文件名不变，然后重新运行容器即可。

### 4.3 Rancher HA 
注意：操作前先备份，[备份和恢复](https://docs.rancher.cn/docs/rancher2.5/backups/_index/)。

备份原有证书 YAML 文件

```bash
kubectl -n cattle-system get secret tls-rancher-ingress -o yaml > tls-rancher-ingress.yml
kubectl -n cattle-system get secret tls-ca -o yaml > tls-ca.yml
```


删除旧的 secret，然后创建新的 secret

```bash
kubectl  -n cattle-system delete secret tls-rancher-ingress
kubectl  -n cattle-system delete secret tls-ca
kubectl  -n cattle-system create secret tls tls-rancher-ingress --cert=./tls.crt --key=./tls.key
kubectl -n cattle-system create secret generic tls-ca --from-file=cacerts.pem
kubectl  -n cattle-system delete pod `kubectl --kubeconfig=$kubeconfig -n cattle-system get pod |grep -E "cattle-cluster-agent|cattle-node-agent|rancher" | awk '{print $1}'`
```

> 重要提示: 如果环境不是按照标准的 rancher 安装文档安装，secret名称可能不相同，请根据实际 secret 名称操作。


## 5. 修改 Rancher Server IP 或域名
依次访问全局 > 系统设置，页面往下翻找到server-url文件；
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f384880092c13c85d69f95fdbe2631f3.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/42c2ce78acda4ae4be251b686afa3dbf.png)

修改后

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ca474dfb9df910a12db354503793da02.png)
###  6. 更新 ingress 配置文件

```bash
$ kubectl get ingress -n cattle-system
NAME      CLASS    HOSTS                 ADDRESS                                                                                           PORTS     AGE
rancher   <none>   rancher.libydev.com   192.168.118.200,192.168.118.201,192.168.118.202,192.168.118.203,192.168.118.204,192.168.118.205   80, 443   29d
$ kubectl edit  ingress -n cattle-system
…..
spec:
  rules:
  - host: rancheruat.liby.com.cn
…..
$ kubectl get ingress -n cattle-system
NAME      CLASS    HOSTS                    ADDRESS                                                                                           PORTS     AGE
rancher   <none>   rancheruat.liby.com.cn   192.168.118.200,192.168.118.201,192.168.118.202,192.168.118.203,192.168.118.204,192.168.118.205   80, 443   29d
```

### 7. 更新 agent 配置文件
**通过新域名或IP登录 Rancher Server；**


访问：	`https://rancher02.demo02.com/g/clusters`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/edb4fa07b93145bd38acd12aaaacb0bf.png)
通过浏览器地址栏查询集群ID， c/后面以c开头的字段即为集群 ID；


点击 **rke2-81**

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9acfd356b47c063c2a68d17ecd2f5150.png)


rke2-81集群的ID：`c-m-fm6ms65h`


格式：访问https://<新的server_url>/v3/clusters/<集群ID>/clusterregistrationtokens页面；

https://rancher02.demo02.com/v3/clusters/c-m-fm6ms65h/clusterregistrationtokens

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/562e66b48b46f0a6417feeaf67476a0d.png)

打开clusterRegistrationTokens页面后，定位到`data`字段；

找到insecureCommand字段，复制 YAML 连接备用；
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cd74048c753d8aaf94619abded46048f.png)

可能会有多组"baseType": "clusterRegistrationToken"，如下图。这种情况以createdTS最大、时间最新的一组为准，一般是最后一组。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9fc4c1b3c582fa753be76f71460e0cc0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ae6ef587f45dbe172fd8d2ef6ff0b3a.png)


保存此命令

```bash
curl --insecure -sfL https://rancher02.demo02.com/v3/import/chwjtqjn4mvnbks9gw885mzdrcmrplz6k7fmp78qlnscmz8878x9b5_c-m-v8q2gb7x.yaml | kubectl apply -f -
```

```bash
$ kubectl logs   -n cattle-system cattle-cluster-agent-559978d9b5-2x8r6   -f
INFO: Environment: CATTLE_ADDRESS=10.42.0.97 CATTLE_CA_CHECKSUM=144376d315608e8bdec62ec40efe8f883e19c7c73a49f66a768b37edfadc3414 CATTLE_CLUSTER=true CATTLE_CLUSTER_AGENT_PORT=tcp://10.43.88.60:80 CATTLE_CLUSTER_AGENT_PORT_443_TCP=tcp://10.43.88.60:443 CATTLE_CLUSTER_AGENT_PORT_443_TCP_ADDR=10.43.88.60 CATTLE_CLUSTER_AGENT_PORT_443_TCP_PORT=443 CATTLE_CLUSTER_AGENT_PORT_443_TCP_PROTO=tcp CATTLE_CLUSTER_AGENT_PORT_80_TCP=tcp://10.43.88.60:80 CATTLE_CLUSTER_AGENT_PORT_80_TCP_ADDR=10.43.88.60 CATTLE_CLUSTER_AGENT_PORT_80_TCP_PORT=80 CATTLE_CLUSTER_AGENT_PORT_80_TCP_PROTO=tcp CATTLE_CLUSTER_AGENT_SERVICE_HOST=10.43.88.60 CATTLE_CLUSTER_AGENT_SERVICE_PORT=80 CATTLE_CLUSTER_AGENT_SERVICE_PORT_HTTP=80 CATTLE_CLUSTER_AGENT_SERVICE_PORT_HTTPS_INTERNAL=443 CATTLE_CLUSTER_REGISTRY= CATTLE_INGRESS_IP_DOMAIN=sslip.io CATTLE_INSTALL_UUID=3c900203-9a7f-43c5-8b95-eb68200a76f0 CATTLE_INTERNAL_ADDRESS= CATTLE_IS_RKE=false CATTLE_K8S_MANAGED=true CATTLE_NODE_NAME=cattle-cluster-agent-559978d9b5-2x8r6 CATTLE_RANCHER_WEBHOOK_VERSION=103.0.1+up0.4.2 CATTLE_SERVER=https://rancher02.demo02.com CATTLE_SERVER_VERSION=v2.8.2
INFO: Using resolv.conf: search cattle-system.svc.cluster.local svc.cluster.local cluster.local nameserver 10.43.0.10 options ndots:5
INFO: https://rancher02.demo02.com/ping is accessible
INFO: rancher02.demo02.com resolves to 192.168.23.80
INFO: Value from https://rancher02.demo02.com/v3/settings/cacerts is an x509 certificate
time="2024-02-22T09:33:32Z" level=info msg="Listening on /tmp/log.sock"
time="2024-02-22T09:33:32Z" level=info msg="Rancher agent version v2.8.2 is starting"
time="2024-02-22T09:33:32Z" level=info msg="Connecting to wss://rancher02.demo02.com/v3/connect/register with token starting with chwjtqjn4mvnbks9gw885mzdrcm"
time="2024-02-22T09:33:32Z" level=info msg="Connecting to proxy" url="wss://rancher02.demo02.com/v3/connect/register"

```

结束。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/308568ecaf66ad3342ad4186123248b1.jpeg#pic_center)

